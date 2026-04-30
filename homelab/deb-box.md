# Debian Homelab Box — Dell OptiPlex 3040 (camel)

**Date:** April 28th, 2026

---

## Overview

Dell OptiPlex 3040 ("camel") was previously run as a Proxmox VE hypervisor.
After issues with SPICE display lag, RustDesk crashes, and the overhead of
managing a full hypervisor for day-to-day access, Proxmox was abandoned.
The machine was wiped and rebuilt as a clean Debian box.

camel now serves as an always-on Debian server with SSH remote access,
Wake-on-LAN, WireGuard VPN, DDNS, rsync backup target, Pi-hole for
network-wide DNS filtering, and a hardening target. All VMs moved to
the ThinkPad under QEMU/KVM.

---

## Hardware

| Field | Details |
|---|---|
| Machine | Dell OptiPlex 3040 |
| CPU | Intel i7-6700 |
| RAM | 16GB |
| Storage | 1TB HDD |
| Network | Gigabit Ethernet |
| Operation | Workstation (KDE Plasma) + SSH from ThinkPad + tablet |

---

## Why Proxmox Was Abandoned

Proxmox VE was installed with the goal of running Kali Linux and REMnux
as VMs remotely. The implementation ran into persistent issues:

- **SPICE lag:** Display over the Proxmox web console had too much latency
  for practical use — unusable for an interactive Kali session
- **RustDesk crashes:** Attempted as an alternative remote desktop;
  crashed repeatedly and was never stable
- **Wrong tool:** Enterprise hypervisor management overhead was excessive
  for the actual need — two VMs that mostly ran locally on the ThinkPad anyway

The VMs moved to the ThinkPad (QEMU/KVM), where they already lived. camel
was rebuilt for the things it was actually useful for: always-on SSH,
backup storage, Pi-hole, and network infrastructure.

---

## Phase 1 — SSH and DHCP Reservation

Generated an ed25519 keypair on the ThinkPad and deployed the public
key to camel. All access is passwordless over SSH.

Assigned a permanent DHCP reservation to camel's MAC address on the
TP-Link Archer BE700 — camel always gets the same address on the local
network.

---

## Phase 2 — Wake-on-LAN

WOL configured and confirmed working. Previous documentation incorrectly
noted this as a dead end — that was based on S5 (fully powered off) behavior.
With AC Recovery set to Power On in BIOS, camel auto-boots after power
loss and WOL works correctly in S3/suspend.

| Layer | Action |
|---|---|
| NIC | `ethtool` — Wake-on: g confirmed |
| NetworkManager | `nmcli` — wake-on-lan magic set for persistence across reboots |
| BIOS | Wake-on-LAN enabled in OptiPlex BIOS settings |

Tested end-to-end: ThinkPad sends magic packet → camel powers on.

---

## Phase 3 — WireGuard and DDNS

WireGuard server and DDNS configuration were restored from the ThinkPad
backup — same config as documented in
[wireguard-ddns-setup.md](wireguard-ddns-setup.md). Port 443 (UDP),
eyeoftheneedle.dev via Cloudflare DDNS script, wg-home/wg-away aliases.

WireGuard peers on camel's server:

| Peer | Role |
|---|---|
| Lenovo ThinkPad E16 Gen 2 | Primary workstation |
| Samsung Galaxy S10 FE | Tablet |
| Samsung Galaxy S23 Ultra | Primary phone — added April 28th, 2026 |

Connectivity tested from three different WiFi networks on April 30th, 2026 —
WireGuard re-established the tunnel automatically each time and SSH to camel
over the VPN IP worked immediately on every network change.

---

## Phase 4 — Pi-hole

Pi-hole installed as the primary DNS resolver for the home network.

| Setting | Value |
|---|---|
| Upstream DNS | Unbound (local recursive resolver — see below) |
| Blocklists | Custom blocklist |
| Admin dashboard | Web interface with real-time query logs and block statistics |
| Router DHCP | Updated to push camel as DNS server to all LAN clients |

All LAN devices now route DNS through Pi-hole without any manual
per-device configuration. The router's DHCP server distributes
camel as the DNS server.

---

## Phase 4a — Unbound Recursive DNS Resolver

Unbound installed alongside Pi-hole as a local recursive DNS resolver.
Pi-hole's upstream is now Unbound rather than Cloudflare.

| Setting | Value |
|---|---|
| Unbound listen address | 127.0.0.1#5335 |
| Pi-hole upstream | 127.0.0.1#5335 (Unbound) |
| DNSSEC | Enabled and validated |

DNS chain: LAN clients → Pi-hole (filtering) → Unbound (recursive
resolver) → root servers directly.

Unbound resolves queries by walking the DNS tree from the root servers
down — no upstream resolver like Cloudflare or Google in the path.
DNSSEC validation confirmed working end-to-end.

---

## Phase 5 — Tablet SSH

SSH access configured from a Samsung Galaxy S10 FE tablet running
Termux.

| Step | Details |
|---|---|
| Keypair | ed25519 generated in Termux on tablet |
| Deployment | Public key added to camel's authorized_keys (root and tman) |
| Aliases | `ssh-camel-tman` and `ssh-camel-root` in Termux SSH config |
| Access method | Via WireGuard tunnel (wg-away) |

Both aliases confirmed working from the tablet over WireGuard.

---

## Current State

| Item | Status |
|---|---|
| OS | Debian (KDE Plasma) |
| SSH | Passwordless ed25519 — ThinkPad and tablet |
| Wake-on-LAN | Working — nmcli + BIOS, tested end-to-end |
| WireGuard | UDP 443 — see [wireguard-ddns-setup.md](wireguard-ddns-setup.md) |
| DDNS | camel.eyeoftheneedle.dev → home IP, cron every 5 min |
| Backup | rsync target from ThinkPad via ~/backup.sh |
| Pi-hole | Running — large blocklist, network-wide via router DHCP, Unbound upstream |
| Unbound | Running — recursive resolver on 127.0.0.1#5335, DNSSEC validated |
| Tablet SSH | Termux on Samsung Galaxy S10 FE — confirmed working |

---

## Skills This Built

- Debian 13 server install and post-install configuration
- SSH key deployment and hardened auth
- Wake-on-LAN across NIC, NetworkManager, and BIOS layers
- Pi-hole DNS filtering with custom blocklists and Unbound recursive upstream
- Unbound recursive DNS resolver with DNSSEC validation
- Network-wide DNS via router DHCP
- Cross-device SSH with Termux and ed25519 keys

---

*Last updated: April 28th, 2026*
