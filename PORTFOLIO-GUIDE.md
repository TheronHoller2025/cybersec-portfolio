# Portfolio Guide
## Structure, Navigation & Growth Plan

---

## Repository Structure
cybersec-portfolio/
│
├── README.md                          ← Main profile page
├── PORTFOLIO-GUIDE.md                 ← This file
├── it-training.md                     ← IT training program documentation
├── pihole-raspi-build.md              ← IT training program capstone
│
├── notes/
│   └── master-notes.md               ← Full cybersecurity learning notes
│
├── homelab/
│   ├── machine-fleet.md              ← Full machine documentation
│   ├── network-topology.md           ← Network map and diagram
│   ├── qemu-kvm-kali-setup.md        ← QEMU/KVM setup documentation
│   ├── remnux-kvm-setup.md           ← REMnux VM setup + AthenaOS decommission
│   ├── router-hardening-tp-link.md   ← BE700 router build
│   ├── chromium-messenger-firejail.md ← Sandboxed Messenger web wrapper
│   ├── arch-maintenance-script.md    ← Monthly Arch maintenance script
│   ├── network-topology.svg          ← Network diagram
│   ├── deb-box.md                    ← Debian box build on OptiPlex (camel)
│   ├── thinkpad-bios-update.md       ← BIOS update + automated version checker
│   ├── arch-linux-usb-build.md       ← Hardware-agnostic Arch Linux USB build
│   └── wireguard-ddns-setup.md       ← WireGuard port migration (443), Cloudflare DDNS on camel
│
├── images/                            ← Screenshots and diagrams
│
├── pentests/
│   └── jellyfin-pentest-report.md    ← Authorized pentest report
│
├── hardening/
│   └── lynis-audit-notes.md          ← System hardening documentation
│
├── certs/
│   ├── progress-tracker.md           ← Certification path and progress
│   └── google-it-support-cert.pdf    ← Google IT Support certificate
│
└── malware-analysis/
    └── wannacry-static-analysis.md   ← WannaCry static analysis (REMnux)

A note on folder organization:
The homelab/ directory covers network and infrastructure builds — routers, VMs, topology, and device configuration. The hardening/ directory covers OS-level security hardening on individual machines, such as Lynis audits and system lockdown procedures. Some overlap exists by nature (e.g. router hardening lives in homelab/ because it is primarily an infrastructure build), but the distinction keeps infrastructure work separate from host-level security work. This structure may be consolidated in a future revision.

---

## How to Navigate This Portfolio

### On Mobile or Tablet
GitHub shows README by default. To see everything else:
- Scroll up and click **Code** tab
- Or use the links below

### Quick Links
- [Homelab Documentation](homelab/)
- [Penetration Tests](pentests/)
- [Certifications](certs/)
- [Hardening Notes](hardening/)
- [Study Notes](notes/)
- [Malware Analysis](malware-analysis/)

---

## How This Portfolio Grows

After every completed activity I add documentation:

| Activity | Where It Goes |
|---|---|
| TryHackMe room completed | notes/ or pentests/ |
| New cert earned | certs/ |
| New lab project | homelab/ |
| Mr. Robot breakdown | notes/ |
| New machine added | homelab/machine-fleet.md |
| Hardening session | hardening/ |
| Desktop app setup / system script | homelab/ |
| AUR software build | homelab/ |
| Malware analysis exercise | malware-analysis/ |

---


Built and maintained by Theron Holler — targeting remote cybersecurity work

---

*Last updated: April 28th, 2026*
