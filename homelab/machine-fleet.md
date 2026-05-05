# Home Lab — Machine Fleet
## Full Documentation — 2026

---

## Overview

A personal multi-machine Linux lab built for cybersecurity learning,
hardening practice, and penetration testing. Every machine serves a
specific purpose and runs a deliberately chosen OS.

---

## Laptops

### Lenovo ThinkPad E16 Gen 2 — Primary Workstation
| Field | Details |
|---|---|
| OS | EndeavourOS (KDE) |
| CPU | AMD Ryzen 7 7735HS |
| RAM | 64GB |
| Purpose | Primary workstation, QEMU/KVM virtualization host |
| VPN | WireGuard full tunnel → camel → Mullvad |

**Virtual Machines (QEMU/KVM):**
| VM | Purpose |
|---|---|
| Kali Linux 2026.1 | Primary penetration testing platform |
| REMnux | Malware analysis sandbox |

---

### HP ProBook 450 G7 — archlaptop
| Field | Details |
|---|---|
| OS | Arch Linux (Zen kernel, KDE) / Kali Linux (KDE) dual-boot |
| CPU | Intel i7-10510U |
| RAM | 32GB |
| Purpose | Secondary lab machine, attack platform |
| VPN | WireGuard full tunnel → camel → Mullvad (both boot entries) |
| VMs | None |

---

### Dell Inspiron 3501
| Field | Details |
|---|---|
| OS | Linux Mint 22.3 (Cinnamon/X11) |
| CPU | Intel i5-1035G1 |
| RAM | 8GB |
| Purpose | General use |
| VPN | WireGuard full tunnel → camel → Mullvad |
| VMs | Devuan (QEMU/KVM) |

---

### Lenovo ThinkBook 21KK
| Field | Details |
|---|---|
| OS | Windows 11 Pro |
| CPU | AMD Ryzen 5 7430U |
| RAM | 32GB |
| Purpose | Windows administration practice |
| VPN | WireGuard full tunnel → camel → Mullvad (Windows service, AUTO_START) |
| Tools | VirtualBox, Sysinternals Suite, PowerShell |
| VMs | Kali |

---

## Desktops

### Dell Inspiron 5680 — BigDell
| Field | Details |
|---|---|
| OS | CachyOS (KDE) |
| CPU | Intel i7-8700 |
| GPU | NVIDIA GTX 1060 3GB |
| RAM | 16GB |
| Display | 4-monitor setup (55" LG 4K, 32" Samsung portrait, 2x HP 24" 1080p) |
| Purpose | Desktop workstation |
| VPN | WireGuard full tunnel → camel → Mullvad |
| VMs | None |

---

## Lab Servers

### Dell OptiPlex 3040 — camel
| Field | Details |
|---|---|
| OS | Debian 13 Trixie |
| CPU | Intel i7-6700 |
| RAM | 16GB |
| Purpose | Always-on server — WireGuard, Pi-hole/Unbound/DoT DNS, Mullvad exit node, nftables firewall, monitoring, backup target, hardening target |
| Access | SSH on a non-standard port via WireGuard tunnel only — full fleet |

---

## Mobile Devices

| Device | OS | Purpose |
|---|---|---|
| Samsung Galaxy S23 Ultra | Android | Primary phone — WireGuard full tunnel |
| Samsung Galaxy S10 FE | Android | Tablet — WireGuard full tunnel |

---

## Portable Media

| Device | OS | Status |
|---|---|---|
| Arch Linux USB | Arch Linux (Zen kernel) | Active — hardware-agnostic, boots on any machine |
| Slackware USB | Slackware | Installer — attempted install, still learning |

---

## Network Infrastructure

| Device | Details |
|---|---|
| Router | TP-Link Archer BE700 (WiFi 7) |
| DNS | Pi-hole → Unbound → DNS-over-TLS (Quad9, encrypted) — network-wide filtering, DNSSEC, no DNS leaks |
| VPN Mesh | WireGuard full-tunnel mesh — 9 machines, all traffic exits via Mullvad |
| DDNS | eyeoftheneedle.dev (Cloudflare) — camel.eyeoftheneedle.dev → home IP, script-driven on camel — see [wireguard-ddns-setup.md](wireguard-ddns-setup.md) |
| Guest Network | Dedicated guest SSIDs with AP isolation — confirmed: guests cannot reach LAN devices |
| IoT Network | Dedicated VLAN with client isolation |

---

## Completed Lab Work

- Full Lynis security audit and hardening on Arch Linux
- rkhunter baseline established
- SSH hardening across multiple machines
- nmap enumeration exercises on OptiPlex
- SUID binary and world-writable file audits
- CVE research and patch verification on Windows
- Authorized penetration test against Jellyfin server
- QEMU/KVM virtualization setup with Kali 2026.1 and REMnux VMs
- AthenaOS VM decommissioned April 2026 after kernel update left it unbootable — replaced with REMnux
- REMnux Noble (Ubuntu 24.04) deployed as dedicated malware analysis sandbox
- Oracle VirtualBox virtualization setup with Kali
- TP-Link BE700 router hardened from scratch
- Pi-hole capstone at local IT training program (Summer 2025)
- Clonezilla full-system backup of ThinkPad NVMe (67GB compressed image on camel)
- Hardware-agnostic Arch Linux USB built from scratch (Zen kernel, GRUB --removable)
- WireGuard port migration (UDP 51820 → 443) on camel — resolved carrier-level blocking; DDNS migrated to eyeoftheneedle.dev via Cloudflare API with script on camel
- Proxmox VE abandoned April 2026 — enterprise hypervisor wrong tool for actual need; VMs moved to ThinkPad
- camel (OptiPlex 3040) rebuilt as Debian box — SSH, WOL, WireGuard, DDNS, Pi-hole, backup target
- Pi-hole deployed on camel — network-wide DNS ad/malware blocking, router DHCP updated
- Unbound deployed on camel as recursive DNS resolver — Pi-hole upstream now points to Unbound, DNSSEC validated (April 28th, 2026)
- Tablet SSH configured (Termux on Samsung Galaxy S10 FE) — ed25519 keys, root and tman aliases via WireGuard
- Samsung Galaxy S23 Ultra added as WireGuard peer on camel's tunnel — April 28th, 2026
- Pi-hole HTTPS — Let's Encrypt certificate issued via Cloudflare DNS challenge, admin interface accessible at pihole.eyeoftheneedle.dev
- SSH hardening on camel — password auth disabled, root login disabled, port 22 removed
- Service reduction on camel — disabled unnecessary services, only required services remain
- nftables hardening on camel — default-drop policy, SSH accessible only via WireGuard tunnel, ICMP scoped to monitoring service IPs
- Full-tunnel WireGuard on ThinkPad — all traffic routes through camel, automatic endpoint switching via NetworkManager dispatcher
- Mullvad exit node on camel — WireGuard client traffic policy-routed through Mullvad tunnel; DNS leak prevention via Unbound UID routing
- Automated monitoring — camel-monitor.sh checks all critical services every five minutes, email alerts on state change; external uptime monitoring integrated
- unattended-upgrades configured on camel — security patches applied automatically
- WireGuard fleet mesh expanded to all 9 machines — archlaptop, BigDell, Inspiron, ThinkBook, Kali all on full tunnel through camel → Mullvad (May 2026)
- SSH mesh deployed across full fleet — passwordless ed25519, all directions, all machines (May 2026)
- Kali Linux added as bare-metal boot entry on HP ProBook 450 G7 — WireGuard peer, boot-persistent (May 2026)
- ThinkBook WireGuard configured on Windows 11 — OpenSSH Server deployed, Windows tunnel service AUTO_START (May 2026)
- DNS-over-TLS deployed on camel — Unbound forwards to Quad9 over encrypted channel to bypass Mullvad DNS interception (May 2026)
- Fleet firmware audited via fwupd/LVFS — Dell Inspiron 3501 BIOS updated, critical security update (May 2026)
- Guest network isolation verified — Kali tested on guest SSID, LAN devices unreachable; router admin locked to specific MACs (May 2026)
- Pi-hole local DNS — all fleet hostnames resolve by name through Pi-hole across the mesh (May 2026)

---

*Last updated: May 5th, 2026*
