# Arch Linux Monthly Maintenance Script
## HP ProBook 450 G7 (archlaptop) — Arch Linux

---

## Overview

Wrote a custom bash script to handle routine Arch Linux system maintenance
in one shot. The idea was to stop running individual commands one at a time
every month and instead have a single script that covers everything: package
updates, AUR updates, orphan cleanup, package cache cleanup, and log
management.

The script lives at `/usr/local/bin/arch-maintenance` and is launched from
a `.desktop` file on the desktop called **Arch Maintenance**. Running it
monthly keeps the system clean and up to date.

---

## System

| Field | Details |
|---|---|
| Machine | HP ProBook 450 G7 (archlaptop) |
| OS | Arch Linux (KDE Plasma, Zen kernel) |
| Script path | `/usr/local/bin/arch-maintenance` |
| AUR helper | yay |
| Run frequency | Monthly |

---

## What the Script Does

### 1. System Update — pacman
```bash
sudo pacman -Syu
```

Updates all official repository packages. `-S` sync, `-y` refresh the
package database, `-u` upgrade all out-of-date packages. This covers
everything from the kernel to system libraries.

---

### 2. AUR Update — yay
```bash
yay -Syu
```

Updates AUR packages alongside official packages. yay wraps pacman and
handles AUR packages at the same time — one command covers everything.

---

### 3. Remove Orphaned Packages
```bash
sudo pacman -Rns $(pacman -Qdtq)
```

**Command breakdown:**
- `pacman -Qdtq` — **Q**uery installed packages, **d** = installed as
  dependencies, **t** = no longer required by anything, **q** = quiet
  (names only). Produces a list of orphaned packages.
- `pacman -Rns` — **R**emove packages, **n** = also remove config files,
  **s** = also remove unneeded dependencies
- `$()` — command substitution: feeds the output of the inner command
  as arguments to the outer one

Orphaned packages are packages that were installed as dependencies for
something that has since been removed. They serve no purpose and take
up space. This cleans them out.

> Note: If `pacman -Qdtq` returns nothing (no orphans found), `pacman -Rns`
> is skipped automatically to avoid an error. The script handles this case.

---

### 4. Clean Package Cache — paccache
```bash
paccache -r
```

Pacman caches downloaded packages in `/var/cache/pacman/pkg/`. By default
it keeps every version ever downloaded — this grows large over time.
`paccache -r` removes all but the three most recent versions of each package.
`paccache` is part of the `pacman-contrib` package.

---

### 5. Clean Journal Logs
```bash
sudo journalctl --vacuum-time=2weeks
```

Removes systemd journal log entries older than two weeks. Keeps recent logs
for troubleshooting while preventing the journal from growing indefinitely.

---

## Desktop Launcher

A `.desktop` file makes the script launchable from the KDE desktop without
opening a terminal manually.

Created `~/Desktop/Arch Maintenance.desktop`:

```ini
[Desktop Entry]
Name=Arch Maintenance
Comment=Monthly Arch Linux system maintenance
Exec=konsole -e bash -c "sudo /usr/local/bin/arch-maintenance; read -p 'Press Enter to close...'"
Icon=utilities-terminal
Terminal=false
Type=Application
Categories=System;
```

**How it works:**
- Opens a Konsole terminal window
- Runs the maintenance script inside it
- Keeps the terminal open after the script finishes so the output can be
  reviewed before closing
- `read -p 'Press Enter to close...'` prevents the window from disappearing
  instantly when the script completes

---

## Script Installation

The script was made executable and placed in `/usr/local/bin/` so it's
available system-wide:

```bash
sudo chmod +x /usr/local/bin/arch-maintenance
```

`/usr/local/bin/` is the conventional location for custom system-wide
scripts on Linux — it's in `$PATH` by default and separate from packages
managed by pacman.

---

## What I Learned

- How `pacman -Qdtq` + `pacman -Rns` works to find and remove orphaned
  packages — a two-step command that took some research to understand fully
- What `paccache` does and why keeping every downloaded package version is
  a bad default on a rolling release system
- Why `/usr/local/bin/` is the right place for custom scripts
- How to use `Exec=` in a `.desktop` file to run a script inside a terminal
  window and keep it open after completion

---

## References

- Arch Wiki — System Maintenance: https://wiki.archlinux.org/title/System_maintenance
- Arch Wiki — pacman tips: https://wiki.archlinux.org/title/Pacman/Tips_and_tricks
- paccache: https://wiki.archlinux.org/title/Pacman#Cleaning_the_package_cache
- journalctl: https://wiki.archlinux.org/title/Systemd/Journal

---

*Written: April 2026*
