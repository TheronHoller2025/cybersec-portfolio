# Hardware-Agnostic Arch Linux USB

**Date:** April 18th–19th, 2026

---

## Overview

Built a portable, hardware-agnostic Arch Linux USB drive from scratch.
The goal was a self-contained system that boots and runs on any
x86_64 UEFI machine in the fleet without modification — AMD or Intel,
whatever hardware it lands on.

---

## Hardware

| Field | Details |
|---|---|
| Device | SanDisk Cruzer Glide 57.3GB |
| Kernel | linux-zen |
| Desktop | KDE Plasma + SDDM |
| Bootloader | GRUB (--removable) |

**Partition layout:**

| Partition | Size | Filesystem | Purpose |
|---|---|---|---|
| sda1 | 512MB | FAT32 | EFI System Partition |
| sda2 | 55GB | ext4 | Root / |
| sda3 | 1.8GB | FAT32 | Data exchange |

---

## Key Design Decisions

**GRUB --removable:**
Installed GRUB to the generic EFI path (`/EFI/BOOT/BOOTX64.EFI`)
rather than a machine-specific NVRAM entry. The USB boots on any
UEFI machine without touching its EFI variables.

**Hardware-agnostic initramfs:**
Removed `autodetect` from the mkinitcpio HOOKS array. Standard Arch
builds use `autodetect` to limit the initramfs to drivers for the
current hardware — correct for a fixed install, wrong for a portable
one. Without it, the initramfs includes drivers for all hardware in
the fleet.

```
MODULES=(amdgpu i915 ahci nvme xhci_hcd usb_storage)
HOOKS=(base udev microcode modconf kms keyboard keymap consolefont block filesystems fsck)
```

**Dual microcode:**
Both `amd-ucode` and `intel-ucode` installed. GRUB loads the correct
one at boot based on the CPU it finds.

**ext4 over F2FS:**
Chose ext4 for the root partition — better compatibility across the
fleet and no dependency on F2FS kernel support being present.

**No swap:**
No swap partition. The USB uses whatever RAM the host machine has.

---

## Build Process

The build is fully scripted. Scripts live in `~/archusbproj/` on the
internal disk:

| Script | Purpose |
|---|---|
| `partition.sh` | sgdisk GPT partitioning + mkfs formatting |
| `pacstrap.sh` | Full package list passed to pacstrap |
| `mount_usb.sh` | Mounts root and EFI partitions |
| `run_chroot.sh` | Runs genfstab, copies scripts, enters arch-chroot |
| `chroot_config.sh` | Full chroot configuration (locale, hostname, mkinitcpio, GRUB, user) |
| `fix_fstab.sh` | Removes host swap entry genfstab picks up from the build machine |
| `umount_usb.sh` | Clean unmount |

---

## Validation

First boot tested on the ThinkBook (Lenovo ThinkBook 21KK, AMD Ryzen
5 7430U) — different hardware from the build machine (ThinkPad E16
Gen 2, AMD Ryzen 7 7735HS). KDE Plasma came up cleanly. NetworkManager
connected via nmtui. Confirms the hardware-agnostic design works in
practice, not just in theory.

---

*Last updated: April 22nd, 2026*
