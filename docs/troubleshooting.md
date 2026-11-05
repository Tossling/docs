# Troubleshooting

## The apps

### Nothing arrives

On a Mac, `tossling status` shows whether the helper is connected, and `tossling log` shows what it did and why it
skipped something. A paused Mac does not send but still receives, `tossling resume` ends the pause.

Remember what is never sent by itself: password managers' copies, files copied in Finder, text over 1 MB. See
[What goes by itself](everyday-use.md#what-goes-by-itself).

If events arrive only in bursts or after a long delay, the reverse proxy is buffering responses. See
[Reverse proxy](server/reverse-proxy.md).

### "The server no longer accepts this computer's token"

The server answered 401 to this computer's token, so the token was deleted or has expired. Tossling stops asking the
server with that token. It tries again after 1 minute, then 2, 4, 8, 16 and every 30 minutes, so the server does not
block the whole network for failed sign-ins. Other devices cannot be disconnected from this computer meanwhile.

To get it going again:

- choose New Room in the menu (also under Devices). It asks for the server token from the setup page or from Security
  in the [web panel](server/web-panel.md), creates a new room and shows how to add the other devices;
- or choose Join Another Room and paste a code from another computer, or run `tossling join` on a Mac.

When the server answers 429 (too many requests from the network), Tossling waits the same way for every token.

### What I copy on an iPhone or iPad does not reach the phone

Tossling ignores Apple's Universal Clipboard on purpose and does not wake it. Copy on a device that runs Tossling.

### Files do not arrive

The server keeps files for 3 hours. A device that was offline longer cannot fetch them.

If sending a large file fails, the reverse proxy probably limits the request size. It must accept bodies up to
600 MB. In nginx that is `client_max_body_size 600m;`.

### The phone gets new items only when the app opens

The server has no [instant delivery](server/instant-delivery.md) set up. On Android turn on Keep a connection, which
the app does by itself when the server reports `"push": false`.

### The hotkey does nothing on a Mac

Allow Tossling in Accessibility, in the Privacy & Security section of System Settings. After switching from a source
build to the app, remove the old Tossling from that list with "−" and enable the new one. The old entry looks enabled
but no longer matches.

⇧⌘C is also taken by Chrome, Finder, Android Studio and Xcode, so the default is ⌃⌥⌘C.

### The hotkey does nothing on Linux

Wayland has no global shortcuts for apps. Bind `/opt/tossling/bin/Tossling --send-clipboard` to a key in the system
settings.

### No "Send via Tossling" in Finder

Enable Tossling among the Finder extensions in Login Items & Extensions, in the General section of System Settings.

### No notifications on a Mac

Allow notifications for Tossling in the Notifications section of System Settings.

### The helper does not start after login

Allow it in Login Items & Extensions and run `tossling on`.

### No Tossling icon in the Linux panel

GNOME needs the AppIndicator extension, which Ubuntu has built in. If the panel has no place for the icon, Tossling
opens a window that explains how to get one.

### Windows does not want to start the installer

The installers are not code-signed yet. Choose More info, then Run anyway.

### Disconnecting a device is refused

Another device in the room has not sent its public key yet: an older version, or one not opened since it joined. Open
that device or update it, then try again. See [Disconnect a device](rooms.md#disconnect-a-device).

## The server

### The setup link does not work

It works for 24 hours or until Done is pressed on its page. Make a new one:

```bash
docker compose exec tossling-server tossling-server setup-link
```

Security in the web panel also makes one.

### The server does not start

Read the log with `docker compose logs tossling-server`.

- `-base-url or TOSSLING_BASE_URL is required`: set `TOSSLING_BASE_URL` in `docker-compose.yml`.
- `another tossling-server already uses …`: a second server runs on the same data directory. Stop the other one.
- `cannot hand /data to …`: the server could not give the data directory to the user from `TOSSLING_RUN_AS`.
  Check that the directory is on a file system that allows changing the owner.

### All devices behind one router get 429

ntfy answers 429 to every request from an address after 30 failed sign-ins. Behind a reverse proxy, without
`TOSSLING_BEHIND_PROXY: "true"`, every client looks like the proxy's address, so one device with a dead token blocks
everyone. Set the variable and make sure the proxy sends `X-Forwarded-For`.

### Creating a room or adding a device fails with 409

The server keeps at most 60 tokens for devices and refuses to issue more. Remove the room tokens nobody uses on the
Security page of the [web panel](server/web-panel.md).

### Instant delivery does not work

`curl https://tossling.example.com/v1/tossling/health` must say `"push": true`. That only means
`TOSSLING_FIREBASE_KEY_FILE` is set. Also check that it points at the mounted file, that user 10001 can read it, and
that the server log shows no error about it. The key has to belong to the Firebase project the apps are built with.
See [Instant delivery](server/instant-delivery.md).
