# Protocol

This is an overview of what Tossling apps send to each other, for anyone who wants to check the design or write a new
client. The full rules are in
[PROTOCOL.md](https://github.com/tossling/tossling-server/blob/main/docs/PROTOCOL.md) in the server repository. Its
[test vectors](https://github.com/tossling/tossling-server/blob/main/docs/vectors.json) pin down every byte that is
derived or encrypted, and the Mac, Android, iOS, Windows and Linux apps all test against the same file. A new client
should pass all of them.

Some names still say `tossy`. The protocol was born as Tossy, and the labels that go into hashes and key derivation
keep that name forever, because changing them would break every existing room.

## Transport

Tossling Server is ntfy with a small API next to it, so most of the transport is plain ntfy. Every request carries
`Authorization: Bearer <token>`.

| What | Request |
|---|---|
| Publish a short message | `POST /<topic>`, the body is the message |
| Publish with an attachment | `PUT /<topic>`, the body is the attachment, `X-Message: <message>`, `X-Filename: clip.bin` |
| Listen | `GET /<topic>[,<topic>…]/json?since=<id or duration>`: one JSON event per line, keep-alive events in between |
| Fetch what was missed | `GET /<topic>/json?poll=1&since=<id or duration>` |
| Check the server | `GET /v1/tossling/health`, no token |
| Check the token | `GET /v1/account`: 200 means it works, 401 or 403 that it does not |

The Tossling API lives under `/v1/tossling`. It manages projects (`/v1/tossling/projects`) and the devices' own tokens
(`POST /v1/tossling/devices` with `{"id", "name"}`, `DELETE /v1/tossling/devices/<id>`). `/v1/tossy` is the same API
under its old name.

Attachments up to 520 MiB are kept for 3 hours. ntfy keeps a body of up to 4,096 bytes as the message and turns a
longer one into an attachment. An attachment URL comes with the event, outside the encryption, so a client sends its
token only to the room's own server.

## Rooms

A room is one ntfy topic plus a 32-byte AES key.

- The topic is `tossling-` and 24 lowercase hex characters (12 random bytes). Rooms made on servers before 0.3 start
  with `tossy-`. Both are rooms for every client.
- The key is 32 random bytes, sent around as standard Base64 with padding.
- Every device has an id of 16 lowercase hex characters, kept across rooms, and an X25519 key pair.
- The owner is the device that created the room. It cannot be removed.

There is no member list on the server. Every device builds its own from the messages it sees, and counts a device
seen within 5 minutes as online.

## Messages

Every message in a room is an envelope: a JSON object, the meta, encrypted with the room key.

```text
envelope = Base64( nonce[12] || AES-256-GCM(room key, nonce, meta JSON) || tag[16] )
```

A fresh random nonce for every message, no associated data. Content that does not fit into the meta travels as the
ntfy attachment, encrypted the same way with the same key, as raw bytes.

The main keys of the meta:

| Key | Meaning |
|---|---|
| `k` | the kind: `text`, `image`, `file`, `hello`, `ping`, `bye`, `rekey`, `kick`, `move` |
| `m` | the MIME type of the content |
| `id`, `n`, `src` | the sender's device id, name and platform |
| `to` | optional list of device ids that should act on the message |
| `v` | inline text |
| `f`, `s`, `x` | file name, size before encryption, and `2` for the streamed format |
| `pk` | the sender's X25519 public key, on control messages |
| `ts`, `mid` | when the sender sealed the message, and its own random id |
| `sens` | `true` on content marked as secret, such as a password manager's copy |

Readers ignore keys they do not know. Text up to 2,400 bytes goes inline, longer text and every image go as an
attachment.

A receiver skips its own messages and those not addressed to it, skips a `mid` it handled in the last 3 hours, and
does not put content older than 15 minutes on the clipboard. It also ignores the same content coming back within 60
seconds, so a copy does not bounce between devices.

## Files

Files use a chunked stream, so neither side holds the whole file in memory.

```text
header  = "TSY2" || chunk size (uint32, big endian) || prefix[7] || 0x00          16 bytes
chunk i = AES-256-GCM(room key, nonce_i, plaintext_i, aad = header) || tag[16]
nonce_i = prefix[7] || i (uint32, big endian) || last (0x01 for the last chunk, else 0x00)
```

The chunk size is 1 MiB when sending. The last chunk holds the rest and can be empty. The sealed size is known in
advance, `16 + size + 16 × (size / chunk size + 1)`, so the upload carries `Content-Length`.

## Joining

A phone joins by scanning a QR code. Its payload is JSON with the server, the token, the room topic and key, and the
computer's id, name and public key.

A computer joins with a one-time invite code of 10 characters, such as `7KQ2M-9XDRT`, and nothing is stored on the
server:

- `seed = PBKDF2-HMAC-SHA256(code, "tossling-invite-v2", 600,000 iterations, 32 bytes)`;
- the key and the invite topic are derived from the seed with HKDF-SHA256, so seeing the topic name does not help
  guess the code faster than running PBKDF2 for every guess;
- the inviter publishes the sealed invite with `X-Cache: no` every 15 seconds for up to 10 minutes, so ntfy delivers
  it only to whoever is listening at that moment;
- the joiner opens it, saves the room and says `hello` there, which tells the inviter to stop.

## Removing a device

Removing a device moves everyone else to a new room the removed device does not know.

1. Make a new topic and key and issue a new shared token.
2. Publish `kick` to the removed device.
3. Publish `rekey`, with the new topic, key and token sealed for each remaining device:

    ```text
    shared = X25519(ephemeral private, recipient public)
    KEK    = HKDF-SHA256(ikm = shared, salt = ephemeral public || recipient public, info = "tossy-rekey-v1", 32 bytes)
    box    = Base64( nonce[12] || AES-256-GCM(KEK, nonce, secret, aad = recipient id) || tag[16] )
    ```

4. Everyone switches to the new room and says `hello` there. The removed device's own tokens are deleted, the old
   shared token a day later.

Before anything is sent, the removal is refused if a remaining device has no public key yet, and warns about devices
whose key changed.

## Moving the server

The health answer carries `url`, the server's main address. When it differs from the address a device uses, the
device checks that the new address is `https` (or the same scheme as now), answers as a Tossling server and accepts
the token, and then switches to it, keeping rooms, keys and ids.

## Push

With Firebase set up, ntfy wakes the phone through a Firebase topic named after the room. The push carries no
content: the phone then fetches the room with `poll=1`.
