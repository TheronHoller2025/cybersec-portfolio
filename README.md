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
- [Malware Analysis](malware-analysis/)
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
Completed structured hands-on labs covering core networking and Windows Server administration, including a Pi-hole capstone project:
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
- Pi-hole for network-wide DNS filtering and malware blocking
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
- Network bridge configuration via virbr0
- Dedicated penetration testing environment isolated from host

**Full documentation:** [qemu-kvm-kali-setup.md](homelab/qemu-kvm-kali-setup.md)

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

### Debian Homelab Box — Dell OptiPlex 3040 (camel)
Wiped Proxmox after persistent display and stability issues and rebuilt camel as a clean Debian box:
- SSH key authentication (ed25519) deployed from ThinkPad and tablet (Termux)
- Wake-on-LAN configured and confirmed working across NIC, NetworkManager, and BIOS
- WireGuard and DDNS restored — same config, same eyeoftheneedle.dev endpoint
- Pi-hole deployed as network-wide DNS filter — Unbound recursive upstream with DNSSEC, custom blocklist, router DHCP updated
- Pi-hole HTTPS configured — valid Let's Encrypt certificate via Cloudflare DNS challenge
- SSH hardened — key-only auth, password auth disabled, port 22 removed
- nftables firewall — default-drop policy, SSH accessible only via WireGuard tunnel
- Mullvad exit node — all WireGuard client traffic policy-routed through Mullvad; DNS leak prevention via Unbound UID routing
- Automated monitoring — camel-monitor.sh checks all critical services every five minutes, email alerts on state change

**Full documentation:** [deb-box.md](homelab/deb-box.md)

---

### WireGuard Port Migration & Remote Access — camel + eyeoftheneedle.dev
Diagnosed carrier-level blocking of UDP 51820 and migrated camel's WireGuard server to UDP 443. Registered `eyeoftheneedle.dev` on Cloudflare and deployed a DDNS update script on camel — checks the public IP every five minutes via cron and calls the Cloudflare API only on change. Two bash aliases (`wg-home` / `wg-away`) handle endpoint switching between LAN and remote without editing any config files. SSH config updated to route through the WireGuard IP so `ssh camel` works from anywhere the tunnel is up.

- WireGuard migrated from UDP 51820 → UDP 443 on camel; port forward updated on BE700
- `eyeoftheneedle.dev` registered on Cloudflare (2 years, WHOIS privacy); `camel.eyeoftheneedle.dev` A record, DNS only
- `/usr/local/bin/ddns-update.sh` on camel — Cloudflare API, scoped token, cron every 5 min
- `wg-home` / `wg-away` aliases for endpoint switching; `ssh camel` routes via WireGuard IP

**Full documentation:** [wireguard-ddns-setup.md](homelab/wireguard-ddns-setup.md)

---

### ThinkPad BIOS Update + Automated Version Checker
Updated the ThinkPad E16 Gen 2 BIOS manually — Lenovo does not publish LVFS capsules for this model:
- Used `geteltorito` (AUR) to extract a bootable image from Lenovo's Bootable CD ISO, flashed via `dd`
- Built a Python script that queries Lenovo's undocumented catalog XML endpoint to check for new BIOS versions
- Automated with a systemd user service and weekly timer

**Full documentation:** [thinkpad-bios-update.md](homelab/thinkpad-bios-update.md)

---

### Hardware-Agnostic Arch Linux USB
Built a portable Arch Linux USB that boots on any x86_64 UEFI machine in the fleet:
- GRUB installed with `--removable` — boots from generic EFI path, no NVRAM dependency
- `autodetect` removed from mkinitcpio — hardware-agnostic initramfs covering all fleet hardware
- Both `amd-ucode` and `intel-ucode` installed — correct microcode loaded at boot regardless of CPU
- Full build scripted across seven shell scripts
- Validated on different hardware than the build machine

**Full documentation:** [arch-linux-usb-build.md](homelab/arch-linux-usb-build.md)

---

### Malware Analysis — WannaCry Static Analysis
**Tools:** REMnux Noble, `file`, `strings`, `sha256sum`, VirusTotal  
First analysis exercise using the REMnux sandbox. Obtained a WannaCry sample from theZoo (public malware research repository) and performed static analysis without execution.

**Key findings:**
- Confirmed PE32 Windows executable — no extension spoofing
- `diskpart.exe` internal PE filename — masquerade technique consistent with documented WannaCry VSS destruction behavior
- 67/74 AV engine detections on VirusTotal; classified as ransomware + worm
- SSH-related strings flagged for follow-up in dynamic analysis

**Full documentation:** [wannacry-static-analysis.md](malware-analysis/wannacry-static-analysis.md)

---

### Network Topology Documentation
Built complete SVG network topology diagram documenting full home lab machine fleet — 6 machines across multiple Linux distributions with network relationships mapped.

---

### Arch Linux Monthly Maintenance Script — HP ProBook
Custom bash script at `/usr/local/bin/arch-maintenance` covering full monthly system upkeep on the ProBook: official and AUR package updates, orphaned package removal, package cache cleanup, and journal log vacuuming. Launched from an **Arch Maintenance** `.desktop` file on the KDE desktop.

**Full documentation:** [arch-maintenance-script.md](homelab/arch-maintenance-script.md)

---

## Machine Fleet

| Machine | OS | Purpose |
|---|---|---|
| Lenovo ThinkPad E16 Gen 2 | EndeavourOS + KDE Plasma 6 | Primary workstation, QEMU/KVM host |
| HP ProBook 450 G7 | Arch Linux / Kali dual-boot | Secondary lab machine |
| Dell Inspiron 5680 | CachyOS | Desktop workstation, 4-monitor setup |
| Dell OptiPlex 3040 | Debian | Server — WireGuard, Pi-hole/Unbound, Mullvad exit, nftables |
| Dell Inspiron 3501 | Linux Mint 22.3 | General use, QEMU/KVM host |
| Lenovo ThinkBook | Windows 11 Pro | Windows environment / VirtualBox |

---

## Currently Learning

- CompTIA Tech+ (exam May 16th, 2026)
- CompTIA A+ (classes starting May 5th, 2026)
- Malware analysis and sandbox techniques

---

## Contact

- Location: U.S.-based (open to remote work)
- Local community college student — Hardware Fundamentals + Security courses
- Available for remote roles

---

*This portfolio is actively updated as I complete new projects and certifications.*
