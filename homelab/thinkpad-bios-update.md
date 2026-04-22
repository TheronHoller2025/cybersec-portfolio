# ThinkPad E16 Gen 2 — BIOS Update and Automated Version Checker

**Date:** April 18th, 2026

---

## Overview

Updated the ThinkPad E16 Gen 2 BIOS from v1.26 (R2KET37W) to v1.27
(R2KET38W) and built an automated version checker to handle future
updates. Lenovo does not publish LVFS capsules for this model, so
fwupd is not an option — the entire process required a manual
workaround.

---

## Part 1 — Manual BIOS Update

**The problem with the standard approach:**
`fwupd` and LVFS do not serve firmware for the ThinkPad E16 Gen 2.
The only update path Lenovo provides is a "Bootable CD" ISO — a
bootable image wrapped in an ISO container that can't be written
directly to a USB drive.

**Process:**

1. Downloaded the BIOS Update Bootable CD ISO (`r2kur38w.iso`) from
   Lenovo support.
2. Installed `geteltorito` from the AUR (`yay -S geteltorito`) to
   extract the bootable disk image from inside the ISO.
3. Extracted the image:
   ```bash
   geteltorito.pl -o ~/Downloads/bios.img ~/Downloads/r2kur38w.iso
   ```
4. Wrote the image to a USB drive:
   ```bash
   sudo dd if=~/Downloads/bios.img of=/dev/sdX bs=1M status=progress
   ```
5. Booted from the USB — flash ran automatically and completed
   successfully.

**Result:** BIOS updated from v1.26 to v1.27.

---

## Part 2 — Automated BIOS Version Checker

**The problem:**
With no LVFS support, there is no native way to get notified when
Lenovo releases a new BIOS version. Lenovo's support portal
(`pcsupport.lenovo.com`) is bot-protected and unusable via scripting.

**The solution:**
Lenovo publishes a machine-type catalog XML at a separate endpoint
that has no bot protection:

```
https://download.lenovo.com/catalog/21M5_Win11.xml
```

This catalog links to a signed package XML containing the current
BIOS version. Wrote a Python script (`~/.local/bin/lenovo-bios-check`)
using only the standard library that fetches and parses this XML,
compares against the installed version, and reports whether an update
is available.

**Automation:**
Created a systemd user service and weekly timer (fires Mondays) to
run the check automatically.

```bash
systemctl --user enable --now lenovo-bios-check.timer
```

Confirmed working — test run returned: `BIOS up to date: 1.27`

---

*Last updated: April 22nd, 2026*
