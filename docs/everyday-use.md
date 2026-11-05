# Sending files and everyday use

## What goes by itself

Text and images you copy reach the other devices of the room by themselves. If you prefer, they go only when you
press the hotkey. On a Mac run `tossling auto off` for that.

Some things are never sent by themselves:

- passwords and other items password managers mark as secret (see
  [Secrets from password managers](rooms.md#secrets-from-password-managers));
- Apple's Universal Clipboard;
- files copied in Finder;
- text over 1 MB;
- whatever arrived from another device a minute ago, so a copy does not bounce back.

Text and images older than 15 minutes do not go into the clipboard, for example when the Mac was asleep. The Android
app still lists them in its history.

## The hotkey

The hotkey copies the selection and sends it.

| Platform | Hotkey |
|---|---|
| macOS | ⌃⌥⌘C, or ⇧⌘C, chosen in the menu. Needs Tossling allowed in Accessibility. |
| Windows | Win+Shift+C, or Ctrl+Alt+Shift+C when another app has taken the first one |
| Linux on X11 | Super+Shift+C, or Ctrl+Alt+Shift+C |
| Linux on Wayland | none built in, see below |

⇧⌘C is also taken by Chrome, Finder, Android Studio and Xcode, so the Mac default is ⌃⌥⌘C.

Wayland has no global shortcuts for apps. Bind this command to a key in the system settings instead:

```bash
/opt/tossling/bin/Tossling --send-clipboard
```

On Wayland Tossling runs through XWayland and still sees what you copy.

## Send files and folders

Files and folders go when you ask, up to 500 MB. A folder goes as a `.zip`.

| Where | How |
|---|---|
| macOS | "Send via Tossling" in Finder, the hotkey, the menu bar, or `tossling send <file>` |
| Windows | "Send via Tossling" in Explorer |
| Linux | "Send via Tossling" in Nautilus, Caja, Nemo, Dolphin or Thunar |
| Android | Share, then To Mac, from any app |
| iPhone and iPad | Tossling in the share sheet, for text, links, photos and files from any app |

On iOS the Shortcuts action Send to Tossling sends what the previous step gives it. Put Get Clipboard before it and
the shortcut sends the clipboard. Back Tap can run that shortcut.

While a file is on its way, the menu shows its progress with a Cancel item.

### Where received files go

| Device | Folder |
|---|---|
| macOS | `~/Downloads/Tossling`, and onto the clipboard, ready to paste in Finder |
| Windows and Linux | `Tossling` in Downloads, and onto the clipboard |
| Android | Downloads/Tossling |
| iPhone and iPad | Files, On My iPhone, Tossling |

The server keeps files for 3 hours. A device that comes online later than that cannot fetch them any more.

A received file is marked as downloaded from the internet, as browsers do: `com.apple.quarantine` on a Mac, a
`Zone.Identifier` stream on Windows. So Gatekeeper and SmartScreen check an app or a script from another device before
it runs.

## Screenshots

On a Mac, screenshots can go to the phone the moment you take them.

## History

The Mac menu bar keeps the last 30 items in both directions, together with the devices of the room, pause and
settings. The Android app has a history with search and pins, and a home screen widget.

## Notifications from projects

Events that your services publish to [projects](server/projects.md) show up as notifications on every device, and in
the Mac menu. Each device chooses which projects to listen to.

## The `tossling` command

The `tossling` command comes with the Mac app (see [Install](install.md#the-tossling-command)).

| Command | What it does |
|---|---|
| `tossling` | a menu of actions in the terminal |
| `tossling status` | connection, devices in the room, what was sent and received |
| `tossling send <file or text>` | sends to every device. `--to <name>` sends to one, `--pick` asks in a window, `… | tossling send` sends piped text |
| `tossling pause [min]` / `tossling resume` | stops sending this Mac's clipboard for a while, receiving goes on |
| `tossling auto on|off` | sends copied text and images by themselves, or only on the hotkey |
| `tossling hotkey on|off` | the hotkey (⌃⌥⌘C or ⇧⌘C, chosen in the menu) copies the selection and sends it |
| `tossling images on|off` | sends images too, or text only |
| `tossling rename <name>` | renames this Mac for the whole room. `tossling rename <device> <name>` names another device on this Mac only |
| `tossling pair [--new]` | a QR code for a phone. `--new` starts a new room |
| `tossling invite` / `tossling join <code>` | adds another computer |
| `tossling log` | what was sent and why something was skipped |
| `tossling on` / `off` / `remove` | starts, stops, or removes the helper and its settings |
| `tossling project [list]` | projects of the server and their publishers |
| `tossling project add <channel> [name] [--publisher <name>]` | a new project on a Tossling Server. Prints the publisher token once with a `curl` example |
| `tossling project remove <channel>` | deletes a project, and its token stops working |
| `tossling language en|ru|auto` | the language of menus and messages |
| `tossling config --json` | server, token and project channels for other programs |

Some examples:

```bash
tossling send ~/Desktop/report.pdf
tossling send --to "Pixel" "see you at 6"
git log -1 | tossling send
tossling pause 30
```

## Files on a Mac

| Path | What is there |
|---|---|
| `~/.config/tossling/config.json` | settings, room key, the room token and this Mac's own token (mode 600) |
| `~/.cache/tossling/` | state, history, the device list for Finder, the server's refusals (`refusals.json`) |
| `~/Library/Logs/Tossling.log` | the helper's log |
| `/Applications/Tossling.app` | the app with the helper, the Finder extension and the `tossling` command inside |

On Windows the room and the history are in `%APPDATA%\Tossling`.
