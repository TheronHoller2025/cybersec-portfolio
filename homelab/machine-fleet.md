# Home Lab — Machine Fleet
## Full Documentation — 2026

---

## Overview

A personal multi-machine Linux lab built for cybersecurity learning,
hardening practice, and penetration testing. Every machine serves a
specific purpose and runs a deliberately chosen OS.

---

## Primary Workstation

### Lenovo ThinkPad E16 Gen 2
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

## Laptops

### HP ProBook 450 G7 — archlaptop
| Field | Details |
|---|---|
| OS | Arch Linux (Zen kernel, KDE) / Kali Linux (KDE) dual-boot  |
| CPU | Intel i7-10510U |
| RAM | 32GB |
| Purpose | Secondary lab machine, attack platform |
| VPN | Manual WireGuard config when needed / Mullvad on Kali |
| VMs | None |

**ProBook Projects:**
| Project | Documentation |
|---|---|
| Sandboxed Messenger web wrapper (Firejail + Chromium) | [chromium-messenger-firejail.md](chromium-messenger-firejail.md) |
| Monthly Arch maintenance script | [arch-maintenance-script.md](arch-maintenance-script.md) |
| Jellyfin Media Player AUR build (v1.12.0-6) | [jellyfin-media-player-build.md](jellyfin-media-player-build.md) |

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

## Desktop Workstations

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
| OS | Debian 13 Trixie (Xfce4/X11) |
| CPU | Intel i7-6700 |
| RAM | 16GB |
| Display | 2x HP 24" 1080p |
| Purpose | Lab server, hardening target, enumeration exercises |
| VMs | None |
| Planned | Pi-hole DNS server (direct install, not VM) |

---

## Windows Environment

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

## Mobile Devices

| Device | OS | Purpose |
|---|---|---|
| Samsung Galaxy S23 Ultra | Android | Primary phone |
| Samsung Galaxy S10 FE | Android | Tablet |

---

## Portable Media

| Device | OS | Status |
|---|---|---|
| Arch Linux USB | Arch Linux | Active — used regularly for diagnostics |
| Slackware USB | Slackware | Installer — attempted install, still learning |

---

## Network Infrastructure

| Device | Details |
|---|---|
| Router | TP-Link Archer BE700 (WiFi 7) |
| DNS | Quad9 (9.9.9.9) — malware blocking |
| VPN Server | WireGuard server on TP-Link router |
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

---

*Last updated: April 2026*
