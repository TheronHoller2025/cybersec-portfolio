# Debian Homelab Box — Dell OptiPlex 3040 (camel)

**Date:** April 28th, 2026

---

## Overview

Dell OptiPlex 3040 ("camel") was previously run as a Proxmox VE hypervisor.
After issues with SPICE display lag, RustDesk crashes, and the overhead of
managing a full hypervisor for day-to-day access, Proxmox was abandoned.
The machine was wiped and rebuilt as a clean Debian box.

camel now serves as an always-on Debian server: SSH remote access,
Wake-on-LAN, WireGuard VPN server, Mullvad exit node, DDNS, Pi-hole
for network-wide DNS filtering, nftables firewall, automated monitoring,
rsync backup target, and a hardening target. All VMs moved to the
ThinkPad under QEMU/KVM.

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
[wireguard-ddns-setup.md](wireguard-ddns-setup.md). eyeoftheneedle.dev
via Cloudflare DDNS script.

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

## Phase 6 — Pi-hole HTTPS

Let's Encrypt certificate issued for `pihole.eyeoftheneedle.dev` using
certbot with a Cloudflare DNS challenge. Pi-hole admin interface now
accessible via HTTPS using a valid certificate — no browser warnings,
accessible from anywhere the WireGuard tunnel is up.

| Setting | Value |
|---|---|
| Domain | pihole.eyeoftheneedle.dev |
| Certificate | Let's Encrypt — Cloudflare DNS challenge |
| Access | Via WireGuard tunnel from any network |

---

## Phase 7 — SSH Hardening

SSH configuration hardened on camel:

| Change | Details |
|---|---|
| Password authentication | Disabled — key-only |
| Root login | Disabled |
| Port 22 | Removed from sshd_config — SSH accessible only via WireGuard tunnel |

---

## Phase 8 — Service Reduction and Auto-Updates

Audited all running systemd services and disabled those not needed for
camel's actual role. Reduced active service count significantly. Only
services required for WireGuard, Pi-hole, Unbound, DDNS, monitoring,
and SSH remain enabled.

unattended-upgrades configured to automatically apply security patches
without manual intervention.

---

## Phase 9 — nftables Hardening

nftables configured with a default-drop policy on the input chain.
The external-facing interface accepts only WireGuard traffic and ICMP
from whitelisted external monitoring service IPs — nothing else reaches
camel from outside. SSH is inaccessible from the LAN directly and only
reachable through the WireGuard tunnel.

| Rule | Effect |
|---|---|
| Default drop on input | All inbound traffic denied unless explicitly allowed |
| WireGuard port accepted | VPN clients can connect |
| ICMP from monitoring IPs | External uptime checks allowed — scoped, not blanket |
| WireGuard interface accepted | All decrypted tunnel traffic allowed inbound |
| SSH blocked on LAN interface | Accessible only through WireGuard tunnel |
| Forward chain | WireGuard client traffic forwarded — established/related accepted |
| NAT masquerade | Outbound traffic from WireGuard clients NATed to camel's IP |

---

## Phase 10 — Full Tunnel and Automatic Endpoint Switching

ThinkPad WireGuard reconfigured from split tunnel (homelab subnets
only) to full tunnel — all traffic routes through camel rather than
only homelab-destined traffic. This enables the Mullvad exit node
built in Phase 11.

A NetworkManager dispatcher script handles automatic endpoint switching
at connection time — no manual commands needed:

| Scenario | Behavior |
|---|---|
| Home network | Dispatcher sets WireGuard to connect via LAN directly |
| Any other network | Dispatcher resolves camel.eyeoftheneedle.dev and connects remotely |
| DNS | Static resolv.conf — Pi-hole primary, public DNS fallback |

On any network change, the dispatcher runs, determines the correct
endpoint, and updates the WireGuard peer. DNS routes through Pi-hole
over the tunnel regardless of network.

---

## Phase 11 — Mullvad Exit Node

camel configured as a VPN gateway: all WireGuard client traffic exits
through a Mullvad WireGuard tunnel (wg-mlvd) rather than camel's own
uplink. camel's local traffic bypasses Mullvad and uses the normal
internet connection directly.

Implemented via nftables packet marking and Linux policy routing:

| Component | Role |
|---|---|
| wg-mlvd | Mullvad WireGuard tunnel on camel — Boston endpoint |
| nftables fwmark | Marks all inbound WireGuard client traffic at arrival |
| Policy routing table | Routes marked traffic out through Mullvad |
| UID-based routing | Unbound's DNS queries also exit through Mullvad |

The UID routing for Unbound closes a DNS leak that would otherwise
exist: without it, Unbound's outbound DNS queries would bypass the
Mullvad tunnel and exit through camel's normal uplink.

**Result:** Any device connected to camel's WireGuard server gets
Mullvad exit, Pi-hole filtering, and DNSSEC — with no DNS leaks.

---

## Phase 12 — Monitoring Stack

Automated monitoring deployed to detect and alert on failures.

**camel-monitor.sh** — runs every five minutes via cron. Sends email
alerts on state transitions only (not every run). Checks:

| Check | What It Detects |
|---|---|
| Internet connectivity | camel cannot reach the internet |
| Mullvad tunnel health | wg-mlvd handshake stale or absent |
| WireGuard server | wg-quick@wg0 inactive |
| Pi-hole DNS | Pi-hole not resolving queries |
| Unbound | Unbound not resolving on local port |
| DDNS | camel.eyeoftheneedle.dev fails to resolve |
| TLS certificate | Let's Encrypt cert expiring within 30 days |
| Disk space | Root partition above threshold |
| Critical services | pihole-FTL, unbound, nftables inactive |

**External monitoring:**
- HTTPS health endpoint on camel for external uptime monitoring
- External monitoring service performs periodic checks from outside
  the home network — provides visibility into outages camel itself
  cannot self-report

---

## Current State

| Item | Status |
|---|---|
| OS | Debian (KDE Plasma) |
| SSH | Passwordless ed25519 — ThinkPad, tablet, S23 Ultra. Password auth disabled. WireGuard-only access. |
| Wake-on-LAN | Working — nmcli + BIOS, tested end-to-end |
| WireGuard | Remote access tunnel — see [wireguard-ddns-setup.md](wireguard-ddns-setup.md) |
| Full tunnel | All ThinkPad traffic routes through camel |
| Mullvad exit | WireGuard client traffic exits via Mullvad — no DNS leaks |
| DDNS | camel.eyeoftheneedle.dev → home IP, cron every 5 min |
| nftables | Default-drop — external access via WireGuard only |
| Pi-hole | HTTPS at pihole.eyeoftheneedle.dev — large blocklist, network-wide via router DHCP, Unbound upstream |
| Unbound | Running — recursive resolver on 127.0.0.1#5335, DNSSEC validated, queries routed through Mullvad |
| Backup | rsync target from ThinkPad via ~/backup.sh |
| Tablet SSH | Termux on Samsung Galaxy S10 FE — confirmed working |
| Monitoring | camel-monitor.sh (cron, email alerts), external uptime monitoring, HTTPS health endpoint |
| Auto-updates | unattended-upgrades — security patches applied automatically |

---

## Skills This Built

- Debian 13 server install and post-install configuration
- SSH key deployment and hardened auth (key-only, password auth disabled, root login disabled)
- Wake-on-LAN across NIC, NetworkManager, and BIOS layers
- Pi-hole DNS filtering with custom blocklists and Unbound recursive upstream
- Unbound recursive DNS resolver with DNSSEC validation
- Network-wide DNS via router DHCP
- Let's Encrypt TLS certificate issuance via Cloudflare DNS challenge
- nftables packet filtering — default-drop policy, scoped rules, NAT masquerade
- Policy-based routing with packet marking and routing tables
- WireGuard full-tunnel configuration
- Mullvad WireGuard exit node on a Linux server
- DNS leak prevention via UID-based routing
- NetworkManager dispatcher scripts for automatic VPN endpoint management
- Automated monitoring with bash, msmtp, and cron
- External uptime monitoring integration
- Cross-device SSH with Termux and ed25519 keys

---

*Last updated: May 3rd, 2026*
