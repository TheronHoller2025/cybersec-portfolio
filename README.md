# Theron's Cybersecurity Portfolio

## About Me

I'm an aspiring cybersecurity professional building toward a career in ethical hacking and penetration testing. I graduated from our local IT training program in Summer 2025 and am currently pursuing CompTIA certifications (Tech+ → A+ → Network+ → Security+) through night classes at a local community college.

My long-term goal is remote cybersecurity work — specifically SOC analyst and eventually penetration testing roles. I'm motivated by a genuine belief that protecting people and systems is meaningful work, not just a career path.

## 📁 Quick Navigation

- [Homelab Documentation](homelab/)
- [Penetration Tests](pentests/)
- [Certifications](certs/)
- [Hardening Notes](hardening/)
- [Study Notes](notes/)
- [Portfolio Guide](PORTFOLIO-GUIDE.md)
---

## Certifications & Education

- Google IT Support Professional Certificate ✅
- CompTIA Tech+ — in progress
- CompTIA A+ — upcoming
- CompTIA Network+ — upcoming
- CompTIA Security+ — target
- Local IT training program — graduate, summer 2025
- Local community college — current student

**Long-term path:** eJPT → PNPT → OSCP

---

## Projects

### Hands-On Labs — Local IT Training Program
Completed structured hands-on labs covering core networking and Windows Server administration, culminating in a Pi-hole capstone project:
- Cisco Packet Tracer: multi-subnet network design with VLSM (April 2025)
- Windows Server 2016: AD DS, DHCP, DNS, GPO, and role-based network shares (April 2025)
- Pi-hole on Raspberry Pi 4: network-wide DNS filtering capstone (Summer 2025)

**Full documentation:** [it-training.md](it-training.md) | [Pi-hole build notes](pihole-raspi-build.md)
---

### Authorized Penetration Test — Jellyfin Media Server
**Tools:** Kali Linux, nmap, curl, whois, traceroute, Shodan, NVD CVE database  
Conducted a two-day authorized penetration test against a friend's self-hosted Jellyfin media server. Produced a formal pentest report documenting findings and remediation recommendations.

**Key Findings:**
- No HTTPS configured — traffic transmitted in cleartext
- No brute-force lockout mechanism on login
- Internal IP address leakage in server responses

**Deliverable:** Full written pentest report (professional format)

---

### Home Network Hardening — TP-Link Archer BE700 (WiFi 7)
Configured and hardened a WiFi 7 router from scratch:
- WPA3/WPA2 unified SSID with MLO (Multi-Link Operation)
- Quad9 DNS for malware blocking
- Dedicated IoT VLAN with client isolation
- WireGuard VPN server tested over mobile data
- DMZ configuration to resolve double NAT with ISP gateway

---

### Linux System Hardening — Lynis Audit
Performed full Lynis security audit and hardening session on Arch Linux system:
- rkhunter baseline established
- SSH hardened (key-based auth, disabled root login)
- NTP configured and secured
- File permission and SUID binary review
- Full audit report reviewed and acted upon

---

### QEMU/KVM Virtualization Lab
Set up professional virtualization environment on EndeavourOS (Lenovo ThinkPad E16):
- QEMU/KVM with libvirt configured
- Kali Linux 2026.1 VM deployed
- REMnux VM deployed (malware analysis sandbox)
- Devuan VM deployed
- Network bridge configuration via virbr0
- Dedicated penetration testing environment isolated from host

---

### REMnux Malware Analysis VM — QEMU/KVM
Deployed REMnux Noble (Ubuntu 24.04-based) as a dedicated malware analysis sandbox in QEMU/KVM:
- Converted General OVA to QCOW2 format for QEMU/KVM compatibility
- Diagnosed and resolved persistent networking issue caused by REMnux's intentional `managed=false` NetworkManager default
- Configured dynamic display scaling via SPICE and spice-vdagent
- Ran `remnux install` — 1024/1029 states succeeded; 5 failures traced to a known upstream Speakeasy checksum bug
- Snapshot taken at clean baseline post-install
- AthenaOS VM decommissioned same session after kernel update left it unbootable — full incident documented

**Full documentation:** [remnux-kvm-setup.md](homelab/remnux-kvm-setup.md)

---

### Network Topology Documentation
Built complete SVG network topology diagram documenting full home lab machine fleet — 6 machines across multiple Linux distributions with network relationships mapped.

---

### Sandboxed Web App — Firejail + Chromium (Facebook Messenger)
**Tools:** Firejail, Chromium  
Wrapped Facebook Messenger as a sandboxed desktop application on the ProBook using Firejail and Chromium's app mode. A `.desktop` file on the KDE desktop launches Messenger in an isolated Firejail process — no browser chrome, no tab bar, process-level sandboxing via Linux namespaces and seccomp-bpf.

**Full documentation:** [chromium-messenger-firejail.md](homelab/chromium-messenger-firejail.md)

---

### Arch Linux Monthly Maintenance Script — HP ProBook
Custom bash script at `/usr/local/bin/arch-maintenance` covering full monthly system upkeep on the ProBook: official and AUR package updates, orphaned package removal, package cache cleanup, and journal log vacuuming. Launched from an **Arch Maintenance** `.desktop` file on the KDE desktop.

**Full documentation:** [arch-maintenance-script.md](homelab/arch-maintenance-script.md)

---

### Jellyfin Media Player — AUR Build
**Tools:** yay  
Built `jellyfin-media-player` version 1.12.0-6 from the AUR on the ProBook. Key dependency `qt5-webengine` (embedded Chromium engine) extended the compile time significantly. Installed March 20, 2026.

**Full documentation:** [jellyfin-media-player-build.md](homelab/jellyfin-media-player-build.md)

---

## Machine Fleet

| Machine | OS | Purpose |
|---|---|---|
| Lenovo ThinkPad E16 Gen 2 | EndeavourOS + KDE Plasma 6 | Primary workstation, QEMU/KVM host |
| HP ProBook 450 G7 | Arch Linux / Kali dual-boot | Secondary lab machine |
| Dell Inspiron 5680 | CachyOS | Desktop workstation, 4-monitor setup |
| Dell OptiPlex 3040 | Debian 13 Trixie | Server/lab machine |
| Dell Inspiron 3501 | Linux Mint 22.3 | General use, QEMU/KVM host |
| Lenovo ThinkBook | Windows 11 Pro | Windows environment / VirtualBox |

---

## Currently Learning

- CompTIA Tech+ exam preparation
- TryHackMe — SOC Level 1 path
- Malware analysis and sandbox techniques
- Network+ exam preparation

---

## Contact

- Location: Massachusetts (open to remote work)
- Local community college student — Hardware Fundamentals + Security courses
- Available for remote roles — targeting Philippines-based remote work

---

*This portfolio is actively updated as I complete new projects and certifications.*
