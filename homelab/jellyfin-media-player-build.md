# Jellyfin Media Player — AUR Build
## HP ProBook 450 G7 (archlaptop) — Arch Linux

---

## Overview

Built and installed `jellyfin-media-player` from the AUR on the ProBook.
Jellyfin Media Player is a standalone desktop client for Jellyfin — a
full native player rather than accessing Jellyfin through a browser. It
provides better performance and a proper media player interface.

The build required pulling and compiling the package and its dependencies
via yay. The significant dependency is `qt5-webengine` — a full Chromium
engine embedded in Qt — which makes the build take considerably longer
than a typical AUR package.

---

## Build Info

| Field | Details |
|---|---|
| Machine | HP ProBook 450 G7 (archlaptop) |
| OS | Arch Linux (KDE Plasma, Zen kernel) |
| Package | jellyfin-media-player |
| Version | 1.12.0-6 |
| Built | March 20, 2026 |
| AUR helper | yay |
| Key dependency | qt5-webengine |

---

## What Is yay

yay (Yet Another Yogurt) is an AUR helper for Arch Linux. The AUR (Arch
User Repository) contains community-maintained package build scripts
(`PKGBUILD` files) for software not in the official repositories. yay
automates the process of downloading, building, and installing these packages.

yay wraps pacman — it handles official packages the same way pacman does,
plus AUR packages transparently in the same command.

---

## Build Process

### Install Command

```bash
yay -S jellyfin-media-player
```

yay fetched the `PKGBUILD` from the AUR, resolved dependencies, and began
the build process.

---

### Dependency — qt5-webengine

The build pulled in `qt5-webengine` as a required dependency.
`qt5-webengine` embeds a full Chromium-based browser engine into Qt
applications — Jellyfin Media Player uses it to render parts of the
Jellyfin web interface inside the native player.

`qt5-webengine` is a large compile. It is one of the heavier AUR
dependencies on Arch and significantly extends the build time compared
to packages with lighter dependencies. On the ProBook the full build
took a while.

---

### Build Output

```
==> Making package: jellyfin-media-player 1.12.0-6
==> Checking runtime dependencies...
==> Checking buildtime dependencies...
==> Retrieving sources...
==> Building and installing package
```

Build completed successfully. Version 1.12.0-6 installed.

---

### Launch

After installation, Jellyfin Media Player appeared in the KDE application
menu and could be launched directly. It connected to the Jellyfin server
using the server's local address.

---

## Why a Native Player Instead of Browser

- Browser access works but is a general-purpose tab — not optimized for media
- Jellyfin Media Player provides a proper 10-foot interface
- Better hardware decoding support
- No browser security prompts for media playback
- Cleaner, dedicated window for the media server

---

## What I Learned

- How the AUR and yay work together — yay automates the fetch, build,
  and install steps that would otherwise be manual with `makepkg`
- What `qt5-webengine` is and why packages that embed a browser engine
  have long build times — you're compiling part of Chromium
- How AUR package versions work — the `1.12.0-6` format means upstream
  version 1.12.0, package revision 6 in the AUR
- The difference between installing from the AUR and installing from
  official repos — AUR packages are built locally from source, not
  downloaded as pre-compiled binaries

---

## References

- Jellyfin Media Player AUR: https://aur.archlinux.org/packages/jellyfin-media-player
- yay GitHub: https://github.com/Jguer/yay
- Arch Wiki — AUR: https://wiki.archlinux.org/title/Arch_User_Repository
- qt5-webengine: https://archlinux.org/packages/extra/x86_64/qt5-webengine/

---

*Built: March 20, 2026*
