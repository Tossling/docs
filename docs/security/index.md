# Security

## What is encrypted

Everything a room carries is encrypted on the devices with the room key, AES-256-GCM. Short text travels inside the
message itself. Longer text, images and files travel as attachments encrypted in 1 MiB chunks, so large files are
never read into memory as a whole.

The room key never reaches the server. It goes from device to device inside QR codes, invite codes encrypted with the
code itself, and new keys sealed for each device after a disconnection.

Projects are the exception. Events that services publish to [projects](../server/projects.md) are ordinary ntfy
messages and are not encrypted.

## What the server knows

The server never sees the clipboard. It knows:

- which room channels are used, and when;
- the size of each message and attachment;
- which token sent a request, and from which address.

Whoever runs the server can see this metadata. Running your own server keeps it with you.

## Tokens

Every request to the server carries a token. There are three kinds:

| Token | Who has it | What it can do |
|---|---|---|
| The room token | every device of the room, inside QR codes and invites | read and write rooms, read projects |
| A device's own token | one device, on Tossling Server 0.5 and later | the same, until the device is disconnected |
| A publisher token | a service that sends events | write to the channels of its publisher, nothing else |

The first room token is the server token from the setup page, which cannot be deleted. A disconnection gives the room
a new one.

Devices can read every channel on the server, but write only to their rooms and the channels `mac` and `claude`.
Room tokens do not expire.

When a device keeps knocking with a token the server no longer accepts, the server answers 401 for an hour without
passing that token to ntfy. So a dead token does not use up ntfy's limit of failed sign-ins for the whole network. The
devices also wait longer and longer between retries, at most every 30 minutes.

## Disconnecting a device

A disconnected device loses the room for good. The others move to a new channel and key that it never sees, sealed
for each remaining device with its own X25519 key. Its own token on the server is deleted, and the old shared token a
day later. See [Disconnect a device](../rooms.md#disconnect-a-device) for the checks that run before.

## Replays and old content

A device keeps the ids of the messages it handled in the last 3 hours and skips any that come again. Content older
than 15 minutes is not put on the clipboard. The age is taken from both the server's time and the time the sender
sealed the message, so an old message sent again stays old.

## Received files

A received file is marked as downloaded from the internet: `com.apple.quarantine` on a Mac, a `Zone.Identifier`
stream on Windows. Gatekeeper and SmartScreen check it before it runs.

## Updates

- The Mac app is signed with a Developer ID and notarized. Sparkle updates are signed with Tossling's update key.
- Windows and Linux install an update only after its Ed25519 signature checks out against the key built into the app.
- The Windows installers are not code-signed yet. They are built from the
  [tossling-desktop](https://github.com/tossling/tossling-desktop) repository by its "Windows and Linux" workflow on a
  version tag.

## What the apps send

The clipboard goes, encrypted on the device, to the server of your room, which you choose at setup. The apps also
check for updates: the Mac reads the Sparkle appcast, Windows and Linux read feeds like
`https://monoroh.com/tossling/windows.json`, and new versions download from GitHub Releases. Nothing else is sent, and
there is no analytics or telemetry.

## Reporting a vulnerability

Do not open a public issue. Use private vulnerability reporting, the Report a vulnerability button on the Security tab
of the repository where the problem is:

- [tossling-server](https://github.com/tossling/tossling-server/security/advisories/new) for the setup page, the web
  panel, the API, the access rules and the Docker image;
- [tossling-desktop](https://github.com/tossling/tossling-desktop/security/advisories/new) for the Mac, Windows and
  Linux apps;
- [tossling-mobile](https://github.com/tossling/tossling-mobile/security/advisories/new) for the Android and iOS apps.

Include what you found, how to reproduce it and what an attacker gains. You should get an answer within a week.
Problems in ntfy itself go to
[binwiederhier/ntfy](https://github.com/binwiederhier/ntfy/security). Only the latest release gets security fixes.
