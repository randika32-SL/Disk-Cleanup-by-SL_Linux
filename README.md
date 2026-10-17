<p align="center">
  <img src="assets/linux-disk-cleanup.svg" alt="Disk Cleanup logo" width="128">
</p>

<h1 align="center">Disk Cleanup - by SL_Linux</h1>

<p align="center">
  A Windows-style <b>Disk Cleanup</b> for Linux. Find temporary files, caches and old logs,
  see exactly where they are and how big they are, tick what you want gone, and clean up safely.
</p>

<p align="center">
  <img alt="Version" src="https://img.shields.io/badge/version-1.3.0-1C8FD6">
  <img alt="License" src="https://img.shields.io/badge/license-MIT-green">
  <img alt="Python" src="https://img.shields.io/badge/python-3.8%2B-blue">
  <img alt="GTK" src="https://img.shields.io/badge/GTK-3-lightgrey">
</p>

<!-- Add a screenshot of the main window here, for example: ![Main window](docs/screenshot.png) -->

## Why

Linux keeps temporary data in many places (`~/.cache`, `/tmp`, `/var/log`, the APT cache, the Trash...).
Cleaning them by hand means typing `rm` commands as root and hoping you got the path right.
**Disk Cleanup** does what Windows Disk Cleanup does: it lists everything it can remove, shows locations and
sizes, lets you select or deselect each item, asks for confirmation, and only asks for your administrator
password when a system file really needs it.

> The Linux kernel itself does not store temporary files or logs. They live in ordinary folders defined by the
> Linux filesystem layout, and those folders are what this app cleans.

## Features

- **Windows-style list** - categories you can expand to see every location, with its size and a checkbox
- **Select / deselect** each item, a whole category, everything, nothing, or just the "Recommended" safe set
- **Confirmation before any delete**, listing counts and sizes
- **Administrator password only when needed** (through the standard polkit prompt; no password is ever stored)
- **Preview mode** - see what would be removed and delete nothing
- **Exclusions** - protect folders or names (for example `firefox`) so they are never listed or touched
- **Open-programs check** - warns you to save and close browsers, terminals and other apps before cleaning
- **Skips files in use** - open or memory-mapped files are left alone; a running browser's cache is never touched
- **Re-validates every item right before deleting it** (it must still be what the scan showed)
- **Kernel-safe package cleanup** - the running kernel and its neighbouring versions are always kept
- **Large old files finder** - lists big files you have not touched for a long time (it never deletes them)
- **Optional cleanup history** - dates, sizes and counts only, never file names (off by default)
- **Optional monthly reminder** - a notification only, nothing is ever deleted automatically (off by default)
- Keyboard shortcuts, dark-theme friendly, translation-ready

## What it cleans

| Category | Location | Needs admin |
|---|---|:---:|
| Thumbnail cache | `~/.cache/thumbnails` | |
| Application caches | `~/.cache` | |
| Web browser caches | Firefox, Chrome, Chromium, Brave, Edge, Opera (incl. Snap) | |
| Temporary files | your files in `/tmp` older than 2 days and not in use | |
| Flatpak app caches | `~/.var/app/*/cache` | |
| Old session error logs | `~/.xsession-errors.old` | |
| Trash | `~/.local/share/Trash` | |
| Developer caches | pip, npm, Yarn, Go build cache | |
| Downloaded package files | `/var/cache/apt/archives` | yes |
| Old system logs | rotated logs in `/var/log` (`.gz`, `.1`, `.old`) | yes |
| Crash reports and core dumps | `/var/crash`, `/var/lib/systemd/coredump` | yes |
| System temporary files | `/var/tmp` older than 30 days | yes |
| Old Snap versions | disabled Snap revisions | yes |
| **Advanced:** system journal, unused packages, unused Flatpak runtimes, Docker leftovers | | yes / no |

Advanced items need an extra "I have reviewed this" tick before they can be removed.

## What it never touches

- Active (non-rotated) logs, your documents, settings, bookmarks, passwords or browser history
- Anything outside its fixed allow-list of cleanup locations (every delete is checked against it)
- Files that a running program has open
- The running kernel, or the newest older/newer kernel
- Symlink targets - symlinks are removed, never followed

## Requirements

- Linux with **Python 3.8+**, **GTK 3** (`python3-gi`, `gir1.2-gtk-3.0`) and **pkexec** (polkit)
- Optional: `wmctrl` (window-accurate open-programs check on X11), `libnotify-bin` (reminders),
  `librsvg2-common` (SVG logo), `gio` (Trash option)
- Developed for **Linux Mint (Cinnamon)**; also targets Ubuntu and Debian. Automated tests run on Ubuntu 24.04.

## Install

### From the `.deb` (recommended)

Download `linux-disk-cleanup_<version>_all.deb` from the **Releases** page, then:

```bash
sudo apt install ./linux-disk-cleanup_1.3.0_all.deb
```

Open **Disk Cleanup - by SL_Linux** from the menu, or run `linux-disk-cleanup`.

### From source

```bash
git clone https://github.com/<your-username>/linux-disk-cleanup.git
cd linux-disk-cleanup
sudo ./install.sh        # installs system-wide (root-owned files)
```

To try it without installing: `python3 ldc_gui.py`. This cleans **your own files only**; system cleanup needs
the installed version (see "How the admin step stays safe").

### Remove

```bash
sudo apt remove linux-disk-cleanup      # if installed from the .deb
sudo ./uninstall.sh                     # if installed with install.sh
```

Your settings in `~/.config/linux-disk-cleanup` are left in place.

## Usage

1. Start the app - it scans automatically (no password needed).
2. Expand a category to see every location and size. Tick or untick what you want to remove.
3. Press **Clean up...**. Save your work and close any programs the app lists, then confirm.
4. If system files are selected, enter your administrator password in the standard prompt.

| Shortcut | Action |
|---|---|
| `Ctrl+R` | Rescan |
| `Ctrl+A` / `Ctrl+Shift+A` | Select all / select none |
| `Ctrl+Enter` | Clean up |
| `Ctrl+,` | Options |
| `Esc` | Cancel a scan or cleanup |
| `F1` | About |

Double-click an item to open its folder.

### Command line

```bash
linux-disk-cleanup --selfcheck       # checks your setup (GTK, pkexec, helper, polkit rule...)
linux-disk-cleanup --version
python3 ldc_core.py                  # text preview of what could be cleaned - changes nothing
```

## How the admin step stays safe

- System items are removed by a small helper (`ldc-helper`) started through `pkexec` with its own polkit rule.
- The helper **refuses to run** unless it lives in a root-owned folder that normal users cannot modify, so it
  cannot be swapped for malicious code.
- The helper **never trusts paths sent by the interface**: it re-scans the system itself and only touches items
  that are still valid candidates *and* were selected. It also honours your exclusions.
- Package changes wait while `apt`/`dpkg` is busy.

## Files the app creates

| File | Purpose |
|---|---|
| `~/.config/linux-disk-cleanup/settings.json` | your options and exclusions |
| `~/.local/share/linux-disk-cleanup/history.jsonl` | cleanup history (only if you turn it on; no file names) |
| `~/.local/state/linux-disk-cleanup/reminder.json` | time of the last reminder check |

## Project layout

```
ldc_core.py        scanning, safety rules, deletion engine (also the root helper's logic)
ldc_gui.py         GTK 3 interface
ldc-helper         tiny root helper started by pkexec
install.sh         system-wide installer          uninstall.sh   remover
assets/            logo and icons
extra/             polkit rule and systemd reminder units
po/                translation template
tests/             safety tests
```

## Tests

```bash
python3 -m unittest discover -s tests -v
```

The tests cover the safety rules: refusing paths outside the allow-list, not following symlinks, leaving
open files alone, honouring exclusions, preview mode, re-validation, helper refusal, kernel protection and
history privacy. Some tests need root and are skipped otherwise.

## Translations

Strings are wrapped for `gettext`. A template is in `po/linux-disk-cleanup.pot`; translations are loaded from
`/usr/share/locale/<lang>/LC_MESSAGES/linux-disk-cleanup.mo`.

## Known limitations

- The Flatpak and Docker cleanup relies on those tools' command-line output and has had less real-world testing.
- On Wayland the open-programs check cannot use `wmctrl` and falls back to a basic mode.
- Once the administrator step has started, it cannot be cancelled.
- The admin password prompt, monthly reminder timer and notifications depend on your desktop session.

## Disclaimer

This software deletes files. It is built to be careful, and **Preview mode** lets you see exactly what would
be removed first, but you use it at your own risk. See the license for the full terms.

## Contributing

Issues and pull requests are welcome. Please run the tests before sending a change, and keep new cleanup
locations on the safe side: a new category must have a fixed allow-list, skip files in use, and be covered by a test.

## License

[MIT](LICENSE) - Copyright (c) 2026 SL_Linux
