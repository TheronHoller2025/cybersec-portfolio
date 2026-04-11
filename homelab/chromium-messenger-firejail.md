# Chromium Web Wrapper — Facebook Messenger (Firejail)
## HP ProBook 450 G7 (archlaptop) — Arch Linux

---

## Overview

Set up Facebook Messenger as a sandboxed desktop application on the ProBook
using Firejail and Chromium in app mode. The goal was to have Messenger
accessible without opening a full browser tab — a clean, isolated window that
behaves like a native app — while keeping it sandboxed away from the rest
of the system using Firejail.

The result is a clickable desktop icon that opens Messenger in its own
Chromium window, contained inside a Firejail sandbox, with no browser chrome
or tab bar. It behaves like a standalone app but is just a sandboxed web wrapper.

---

## System

| Field | Details |
|---|---|
| Machine | HP ProBook 450 G7 (archlaptop) |
| OS | Arch Linux (KDE Plasma, Zen kernel) |
| Browser | Chromium |
| Sandbox | Firejail |
| Target URL | https://www.messenger.com |

---

## How It Works

### Firejail
Firejail is a Linux sandboxing tool that uses Linux namespaces and
seccomp-bpf to restrict what a process can access. When Chromium is
launched inside Firejail, it runs in an isolated environment — restricted
filesystem access, isolated network namespace, limited system call access.
If the browser or a malicious page tries to reach outside its sandbox,
Firejail blocks it.

### Chromium App Mode
Chromium's `--app=URL` flag launches a URL in a stripped-down window
with no address bar, no tab bar, and no browser UI. The window looks and
behaves like a standalone desktop application. Combined with Firejail,
it's a sandboxed native-feeling app wrapper built from standard tools.

---

## Setup

### Step 1 — Install Firejail and Chromium

```bash
sudo pacman -S firejail chromium
```

Both are in the official Arch repositories.

---

### Step 2 — Test the Launch Command

Before creating a desktop file, the launch command was tested manually
from a terminal to confirm it worked:

```bash
firejail chromium --app=https://www.messenger.com
```

**Command breakdown:**
- `firejail` — wraps the following command in a Firejail sandbox
- `chromium` — the browser being sandboxed
- `--app=https://www.messenger.com` — launches Chromium in app mode, loading
  Messenger directly with no browser chrome or tab bar

Messenger opened in a clean standalone-style window. Login and normal
use worked correctly.

---

### Step 3 — Create the .desktop File on the Desktop

A `.desktop` file is a standard Linux desktop entry — it tells KDE (and
other desktop environments) how to launch an application, what icon to use,
and what category it belongs to.

The file was created directly on the desktop at `~/Desktop/messenger.desktop`:

```ini
[Desktop Entry]
Name=Messenger
Comment=Facebook Messenger (Firejail + Chromium)
Exec=firejail chromium --app=https://www.messenger.com
Icon=chromium
Terminal=false
Type=Application
Categories=Network;InstantMessaging;
```

**Key fields:**
- `Exec=` — the exact command launched when you open the app
- `Icon=chromium` — reuses Chromium's existing icon
- `Terminal=false` — no terminal window opens when launched
- `Categories=` — places it correctly in application menus

In KDE, right-clicking the desktop entry and selecting "Allow Launching"
marks it as trusted so it can be double-clicked to launch.

---

## Result

Double-clicking the desktop icon opens Messenger in a sandboxed Chromium
app-mode window. It looks like a native app. The window has no browser
controls, and the process runs inside a Firejail sandbox with restricted
system access.

---

## What I Learned

- How Firejail uses Linux namespaces to create isolated process environments
- How Chromium's `--app=` flag turns any URL into a standalone-looking
  desktop application
- How `.desktop` files work — the standard format Linux desktop environments
  use to register and launch applications
- How `.desktop` files placed on the KDE desktop work and how to mark them as trusted for launching
- Why sandboxing a browser app matters — even a messaging app running in
  a browser can access the filesystem unless explicitly restricted

---

## References

- Firejail documentation: https://firejail.wordpress.com/documentation-2/
- Firejail Arch Wiki: https://wiki.archlinux.org/title/Firejail
- .desktop file spec: https://specifications.freedesktop.org/desktop-entry-spec/latest/
- Chromium app mode: `chromium --help`

---

*Setup: April 2026*
