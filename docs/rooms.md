# Rooms and devices

A room is the group of devices that share a clipboard: any number of computers and phones. On the server a room is
one channel. Its encryption key lives only on the devices, so the server cannot read what goes through it.

Every device also has its own X25519 key pair. When a device is disconnected, the others use these keys to move to a
new room key that the disconnected device never sees.

The device that creates the room is its owner. The owner cannot be disconnected by the others.

## Create a room

The first computer creates the room on your server.

- On a Mac, open the app and choose Connect my server in the first-run window. Enter the address and the token from
  the server's [setup page](server/index.md#the-setup-link).
- On Windows and Linux, choose to create a room on your server and enter the same address and token.

The computer then shows a QR code for the phone.

To start over in a new room later, choose New Room in the menu (also under Devices) or run `tossling pair --new` on a
Mac. It asks for the server token again.

## Add a phone

1. On a computer in the room, open Devices, then Add a Device. On a Mac `tossling pair` shows the same QR code. On
   Windows and Linux the item is Devices, then Connect a Phone.
2. In the phone app tap Pair with a computer and scan the code.

The QR code carries the room key and the token. Do not share it and do not take a screenshot of it.

A room can hold several phones. Each one pairs the same way.

### Reinstalling the Android app

With Google backup on, the welcome screen of a reinstalled app offers to return to the room as the same device, with
no QR code. The room key stays on the phone and is not backed up to the cloud.

## Add a computer

Computers join with a one-time invite code.

1. On a computer in the room, open Devices, then Add a Device, or run `tossling invite` on a Mac. It shows something
   like `tossling join tossling.example.com/7KQ2M-9XDRT`.
2. Within 10 minutes, on the new computer:
    - on a Mac, run that command, or choose Join another computer's room in the first-run window and enter the code;
    - on Windows and Linux, choose Join Another Room and paste `tossling.example.com/7KQ2M-9XDRT`.

The invite is never stored on the server. The inviting computer keeps sending it while it waits, and the room key
inside is encrypted with the code (PBKDF2 and AES-GCM). Codes have 10 characters. The 8-character codes of older
versions still work for joining, but new apps never make them.

## Device names

`tossling rename <name>` on a Mac renames it for the whole room. `tossling rename <device> <name>` gives another device
a name that only this Mac shows.

## Disconnect a device

Open Devices on any device in the room and disconnect the one you want out, for example a lost phone. The rest of the
room then:

1. moves to a new channel and a new room key, sealed for each remaining device with its own public key (X25519,
   HKDF-SHA256 and AES-GCM);
2. gets a new server token that does not expire. If the new key does not reach the room, the new token is deleted and
   the room stays as it was;
3. deletes the old token a day later, unless it is the server's own token from the setup page.

On a server that gives devices tokens of their own, the disconnected device's own token is deleted right after the new
key is out. The disconnected device can no longer read the room, even if it still has a token that works on the
server.

Two checks run before anything is sent:

- A device that has not sent its key yet (an older version, or one not opened since it joined) blocks the
  disconnection. Open it or update it, and try again.
- A device whose key changed is marked in Devices until you accept the new key there. Disconnecting warns about such
  devices and goes on only after you confirm.

A key changes when a device joins again, for example after a reinstall. Any member of the room could announce a new
key in another device's name, so the change is never accepted silently.

## Device tokens

On Tossling Server 0.5 and later (except demo servers) every device gets a token of its own from the server and talks
to it with that token. The token from the setup page stays the room token: it goes into QR codes and invites, and a
disconnection replaces it.

`tossling status` on a Mac says which token it uses. In `~/.config/tossling/config.json` the device's own token is
`own`. Joining another room, `tossling setup --new` and `tossling remove` delete the Mac's own token on the server
first.

Devices of older versions keep using the shared room token, so one room can mix both.

## Secrets from password managers

Tossling never sends a password manager's copy by itself. On a Mac these are the items password managers mark as
concealed (`org.nspasteboard.ConcealedType` and similar). On Linux it is a copy marked with KDE's
`x-kde-passwordManagerHint: secret`, which KeePassXC and others set. Windows skips the passwords marked by password
managers too.

To send such a copy on purpose, use the hotkey or Send the Clipboard Now. It then goes as a secret. A device that
receives a secret:

- puts it on the clipboard marked so that clipboard history, cloud clipboards and previews leave it out (macOS
  concealed type, Windows clipboard history and cloud clipboard, KDE's Klipper, Android's sensitive flag, iOS local
  only with an expiration date);
- keeps it out of its own history;
- does not show it in the notification.

Apple's Universal Clipboard is never sent either. What you copy on an iPhone or iPad without Tossling does not reach
the room. Copy on a device that runs Tossling.
