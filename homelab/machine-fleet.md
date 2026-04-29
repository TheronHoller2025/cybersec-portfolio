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
| VPN | Mullvad (app) |

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
| VPN | Manual WireGuard config when needed / Mullvad on Kali |
| VMs | None |

---

### Dell Inspiron 3501
| Field | Details |
|---|---|
| OS | Linux Mint 22.3 (Cinnamon/X11) |
| CPU | Intel i5-1035G1 |
| RAM | 8GB |
| Purpose | General use |
| VMs | Devuan (QEMU/KVM) |

---

### Lenovo ThinkBook 21KK
| Field | Details |
|---|---|
| OS | Windows 11 Pro |
| CPU | AMD Ryzen 5 7430U |
| RAM | 32GB |
| Purpose | Windows administration practice |
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
| VMs | None |

---

## Lab Servers

### Dell OptiPlex 3040
| Field | Details |
|---|---|
| OS | Debian (KDE Plasma) |
| CPU | Intel i7-6700 |
| RAM | 16GB |
| Purpose | Debian server, backup target, Pi-hole host, hardening target |
| Access | Workstation (KDE Plasma) + SSH from ThinkPad + tablet |

---

## Mobile Devices

| Device | OS | Purpose |
|---|---|---|
| Samsung Galaxy S23 Ultra | Android | Primary phone |
| Samsung Galaxy S10 FE | Android | Tablet |

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
| DNS | Pi-hole (camel) → Unbound upstream — network-wide ad/malware blocking, DNSSEC |
| VPN Server (camel) | WireGuard on Linux host — UDP 443 — remote access tunnel |
| DDNS | eyeoftheneedle.dev (Cloudflare) — camel.eyeoftheneedle.dev → home IP, script-driven on camel — see [wireguard-ddns-setup.md](wireguard-ddns-setup.md) |
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
- Samsung Galaxy S23 Ultra added as WireGuard peer on camel's tunnel (UDP 443) — April 28th, 2026

---

*Last updated: April 28th, 2026*
