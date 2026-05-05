# Home Lab Network Topology
## Logical Network Map — 2026

![Network Topology](network-topology.svg)

---

## Overview

Documents the logical network layout of my home lab including
all machines, network segments, VPN configuration, and planned
additions. A full SVG network diagram is included above.

---

## Network Infrastructure

| Device | Role | Details |
|---|---|---|
| TP-Link Archer BE700 | Primary Router | WiFi 7, WPA3 |
| ISP Gateway | ISP Modem | Double NAT resolved via DMZ |
| virbr0 | Virtual Bridge | libvirt NAT bridge for QEMU/KVM VMs on Lenovo ThinkPad |

---

## Network Segments

### Main LAN
All primary workstations and laptops

### IoT VLAN
- Dedicated isolated network for smart devices
- Client isolation enabled
- Internet access only — cannot reach main LAN

### Guest Network
- Dedicated guest SSIDs with AP isolation
- Internet access only — confirmed: guests cannot reach LAN devices
- Router admin panel locked to specific MACs

### Virtual Network (virbr0)
- NAT bridge managed by libvirt on Lenovo ThinkPad
- Confirmed active via libvirtd and dnsmasq
- Serves DHCP to Kali and REMnux VMs
- Start VMs before enabling Mullvad to avoid
  DHCP conflicts on eth0

---

## Machine Network Roles

| Machine | Segment | Notes |
|---|---|---|
| Lenovo ThinkPad E16 Gen 2 | Main LAN | Primary workstation, QEMU/KVM host |
| HP ProBook 450 G7 | Main LAN | Secondary lab, attack platform |
| BigDell Inspiron 5680 | Main LAN | Desktop workstation |
| OptiPlex 3040 "camel" | Main LAN | Always-on server — WireGuard, DNS, Mullvad exit |
| Inspiron 3501 | Main LAN | General use |
| ThinkBook 21KK | Main LAN | Windows environment |
| Kali Linux 2026.1 VM | virbr0 (NAT) | Penetration testing |
| REMnux VM | virbr0 (NAT) | Malware analysis |
| Kali Linux VM | Local (VirtualBox) | Penetration testing |
| Devuan VM | virbr0 (NAT) | General use |
| Galaxy S23 Ultra | WiFi | Primary phone |
| Galaxy S10 FE | WiFi | Tablet |
| LG NanoCell 55" | IoT VLAN | Smart TV |

---

## VPN Configuration

| Machine | VPN | Notes |
|---|---|---|
| camel (OptiPlex 3040) | WireGuard server + Mullvad exit node | 9-machine fleet mesh — all client traffic exits through Mullvad |
| ThinkPad E16 Gen 2 | Full tunnel → camel → Mullvad | NM dispatcher, auto endpoint switching |
| HP ProBook 450 G7 (Arch + Kali) | Full tunnel → camel → Mullvad | Both boot entries — separate fleet peers |
| Dell Inspiron 3501 | Full tunnel → camel → Mullvad | NM dispatcher, auto endpoint switching |
| Dell Inspiron 5680 (BigDell) | Full tunnel → camel → Mullvad | systemd, auto-start |
| Lenovo ThinkBook 21KK | Full tunnel → camel → Mullvad | Windows service, AUTO_START |
| Galaxy S23 Ultra | Full tunnel → camel → Mullvad | WireGuard app |
| Galaxy S10 FE | Full tunnel → camel → Mullvad | WireGuard app |

Full setup documented in [wireguard-ddns-setup.md](wireguard-ddns-setup.md).

---

## DNS

| Setting | Value |
|---|---|
| LAN DNS | Pi-hole on camel — served via router DHCP |
| Pi-hole upstream | Unbound → DNS-over-TLS to Quad9 (encrypted, bypasses Mullvad DNS interception) |
| Pi-hole admin | HTTPS at pihole.eyeoftheneedle.dev — valid Let's Encrypt certificate |
| DNS leak prevention | Unbound queries routed through Mullvad via UID routing |
| Fleet local DNS | All fleet hostnames resolve by name through Pi-hole |
| Blocklists | Custom blocklist |
| DDNS | eyeoftheneedle.dev (Cloudflare) — camel.eyeoftheneedle.dev → home IP, updated by script on camel |

---

## Planned Additions

| Addition | Status |
|---|---|
| NAS/Storage server | Aspirational |
| Dedicated IDS (Suricata) | Aspirational |
| Dedicated firewall device | Aspirational |
| Web server | Aspirational |

---

*Last updated: May 5th, 2026*
