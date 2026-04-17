# Router Hardening — TP-Link Archer BE700
## WiFi 7 Home Network Security Build — 2026

---

## Overview

I built this network from scratch — meaning I started from factory 
defaults and went through every setting manually rather than just 
accepting whatever the router came with out of the box. This was 
part of my learning process for network security. Some of it was 
straightforward, some of it took research and troubleshooting to 
figure out. The double NAT situation with the ISP gateway was 
probably the most frustrating part — took me a while to realize 
DMZ was the fix.

I used official documentation and guides throughout — listed in 
the references section at the bottom.

---

## Hardware

| Field | Details |
|---|---|
| Device | TP-Link Archer BE700 |
| Standard | WiFi 7 (802.11be) |
| ISP Gateway | ISP (double NAT resolved via DMZ) |
| Purpose | Primary home router, security hardened |

---

## What I Configured and Why

### Wireless Security
- WPA3/WPA2 unified SSID — WPA3 for devices that support it,
  WPA2 as fallback for older devices
- MLO (Multi-Link Operation) enabled — WiFi 7 feature that bonds
  multiple frequency bands simultaneously for better performance
- Removed all default credentials immediately

### DNS Security
- Replaced ISP default DNS with Quad9 (9.9.9.9)
- Quad9 blocks known malware and phishing domains at the DNS level
- Every device on the network gets this protection automatically
  without any per-device configuration needed

### Network Segmentation
- Created a dedicated IoT VLAN for smart devices
- Enabled client isolation on the IoT network so IoT devices
  can reach the internet but cannot communicate with my main machines
- The idea here is limiting blast radius — if an IoT device gets
  compromised it can't pivot to my other systems

### WireGuard VPN Server
- Configured WireGuard VPN server directly on the router itself
- Tested the tunnel successfully over mobile data from outside
  the network — it worked
- This was one of the more challenging parts of the whole build
  and required the most research

### NAT and DMZ
- Having both the ISP gateway and the TP-Link router created
  a double NAT situation
- This was breaking WireGuard connections from outside
- Solution was placing the TP-Link in the ISP gateway's DMZ
- After that single NAT worked correctly and WireGuard connected

---

## What Went Wrong / What I Learned the Hard Way

- Double NAT was not obvious at first — spent time troubleshooting
  WireGuard before realizing the real problem was at the NAT layer
- MLO required understanding which bands were being combined and why
- IoT VLAN setup required learning how VLANs tag and separate traffic
- Some settings required router reboots that weren't obvious upfront

---

## Skills This Built

- Router hardening and DNS security config
- VLAN segmentation and WireGuard VPN setup
- NAT, DMZ, and port forwarding troubleshooting
- WiFi 7 hands-on

---

## Still On My To-Do List

- DNS-over-TLS configuration
- DDNS setup via No-IP for dynamic IP management
- Pi-hole + Unbound integration on the Debian box

---

## References & Documentation Used

- TP-Link Archer BE700 official documentation
- WireGuard official docs — https://www.wireguard.com/quickstart/
- Quad9 setup guide — https://www.quad9.net/support/set-up-guides
- Mullvad WireGuard config — https://mullvad.net/en/help/wireguard-and-mullvad-vpn/
- General VLAN configuration guides via TP-Link support

---

*Build completed: Early 2026*
