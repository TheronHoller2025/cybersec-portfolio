# Proxmox VE Homelab Server — Dell OptiPlex 3040

**Date:** April 20th, 2026

---

## Overview

Repurposed a Dell OptiPlex 3040 as a dedicated Proxmox VE hypervisor
and backup server. The machine previously ran Debian 13 Trixie as a
hardening target. This build establishes a headless, remotely managed
hypervisor on the local network with dedicated backup storage.

---

## Hardware

| Field | Details |
|---|---|
| Machine | Dell OptiPlex 3040 |
| CPU | Intel i7-6700 |
| RAM | 16GB |
| Storage | 1TB HDD |
| Network | Gigabit Ethernet |
| Operation | Headless — no monitor, keyboard, or mouse |

---

## Phase 1 — Remote Management Infrastructure

Before Proxmox was installed, the OptiPlex needed to be fully
manageable without physical access.

**SSH key authentication:**
Generated an ed25519 keypair on the ThinkPad and deployed the public
key to the OptiPlex. All access is passwordless over SSH.

**DHCP reservation:**
Assigned a permanent IP reservation to the OptiPlex MAC address on
the TP-Link Archer BE700 router so the machine always gets the same
address on the local network.

**Wake-on-LAN:**
Configured WoL across all three required layers:

| Layer | Action |
|---|---|
| NIC | Confirmed WoL support via `ethtool`, enabled with `ethtool -s enp2s0 wol g` |
| NetworkManager | Set `wake-on-lan=magic` in the connection profile for persistence across reboots |
| BIOS | Enabled Wake-on-LAN in OptiPlex BIOS settings |

Tested: ThinkPad sends magic packet → OptiPlex powers on from off.

**Update:** Wake-on-LAN was later found to be a hardware dead end on this machine. The Realtek RTL8168h NIC has no standby power in S5 on the OptiPlex 3040 — magic packets arrive but the NIC cannot act on them when fully powered off. No BIOS setting resolves this. The machine is now left on permanently with AC Recovery set to Power On, so it automatically boots after any power outage.

**UFW:**
Diagnosed and fixed a UFW bug on Debian where SSH was being blocked
despite an allow rule existing. Re-enabled UFW after the fix.

---

## Phase 2 — Proxmox VE Installation

Downloaded Proxmox VE 9.1-1 ISO to a Ventoy USB and installed,
wiping Debian 13. Configured partition layout to reserve approximately
300GB in the LVM volume group for backup storage.

**Repository configuration:**
Disabled the enterprise and Ceph repositories (require a paid
subscription, return 401 errors). Used the no-subscription repository.
Ran a full `apt upgrade` after fixing the repos.

---

## Phase 3 — Backup Storage Volume

Created a dedicated LVM logical volume for backup storage:

```bash
lvcreate -L 300G -n backups pve
mkfs.ext4 /dev/pve/backups
mkdir /backups
```

Added to `/etc/fstab` for automatic mount at boot. Provides
approximately 295GB usable space.

---

## Current State

| Item | Status |
|---|---|
| Proxmox VE 9.1 | Running, fully updated |
| Remote access | SSH + Proxmox web UI via WireGuard — see [wireguard-ddns-setup.md](wireguard-ddns-setup.md) |
| Backup volume (/backups) | Mounted, ~295GB usable |
| VMs | Kali Linux 2026.1, REMnux |

---

*Last updated: April 22nd, 2026*
