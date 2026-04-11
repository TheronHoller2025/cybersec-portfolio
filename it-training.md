# Hands-On Labs — Local IT Training Program

Hands-on lab work completed as part of coursework in a local IT training program. Labs cover core networking and Windows Server administration skills.

---

## Lab 1 — Cisco Packet Tracer: Multi-Subnet Network Design

**Date Completed:** April 1, 2025  
**Tool:** Cisco Packet Tracer

### Overview

Designed and built a multi-subnet network from scratch using Cisco Packet Tracer. The topology consisted of four standard subnets and one point-to-point link using Variable Length Subnet Masking (VLSM).

### Topology

- Two routers interconnected via a VLSM point-to-point subnet
- Four subnets, each with a dedicated switch and two end-user PCs
- All devices connected using copper straight-through Ethernet cables
- Network IDs and default gateways labeled per subnet

### Network Addressing

| Subnet | Network ID        | Default Gateway | Hosts                          |
|--------|-------------------|-----------------|--------------------------------|
| SN1    | 192.168.1.0/24    | 192.168.1.1     | 192.168.1.2, 192.168.1.3       |
| SN2    | 192.168.2.0/24    | 192.168.2.1     | 192.168.2.2, 192.168.2.3       |
| SN3    | 192.168.3.0/24    | 192.168.3.1     | 192.168.3.2, 192.168.3.3       |
| SN4    | 192.168.4.0/24    | 192.168.4.1     | 192.168.4.2, 192.168.4.3       |
| SN5 (VLSM) | 192.168.252.252/30 | —          | 192.168.252.253, 192.168.252.254 |

SN5 serves as the point-to-point link between the two routers. A /30 subnet was used to conserve address space — only two usable host addresses are needed for a router-to-router connection.

### Results

All ping tests completed successfully. Every device on every subnet was able to reach every other device across the full topology.

### Skills Demonstrated

- Subnetting and IP address planning
- VLSM for point-to-point links
- Router and switch configuration in Packet Tracer
- Network topology design and documentation
- End-to-end connectivity verification via ICMP ping

---

## Lab 2 — Windows Server 2016: Active Directory, DHCP, DNS, GPO, and Network Shares

**Date Completed:** April 21, 2025  
**Platform:** Oracle VirtualBox  
**OS:** Windows Server 2016 Standard (Desktop Experience)

### Overview

Built and configured a Windows Server 2016 environment from scratch inside a virtual machine. Set up core server roles and simulated a small enterprise network with user accounts, security groups, shared network folders, domain-joined clients, and Group Policy enforcement.

### VM Specifications

- 2 CPU cores
- 4 GB RAM
- 32 GB HDD
- Network adapter set to Host-Only

### Server Configuration

- Assigned static IP: `192.168.56.2`
- Default gateway: `192.168.56.1`
- Preferred DNS pointed to self (`192.168.56.2`)
- Server renamed and promoted to domain controller
- Domain: `LAB.local`

### Roles Installed

- **Active Directory Domain Services (AD DS)** — promoted server to domain controller, created new forest
- **DHCP** — configured scope with IP exclusions at both ends of the range for static assignment reservation; 14-day lease duration
- **DNS** — auto-configured during AD DS promotion

### DHCP Scope

- Range: `192.168.56.1` – `192.168.56.254`
- Exclusions: `.1`–`.5` and `.250`–`.254` reserved for static assignments
- Default gateway set to `192.168.56.1`

### Active Directory Configuration

**User Accounts**  
Created multiple user accounts with descriptive fields (title, department, email). Used a consistent naming convention of first initial + last name in lowercase.

**Security Groups**  
Created department-based security groups: Leadership, IT, Marketing, Engineering, and Human Resources. Also created a Distribution Group for cross-department communication.

**Domain Admin Access via Group Membership**  
Configured the IT security group as a member of Domain Admins. Any user added to the IT group automatically inherits domain-level administrator rights — no manual per-user elevation required.

**Network Shares**  
Created a shared root folder (`LAB_Share`) with department subfolders. Set NTFS and share permissions so that each department group can only access their own folder. Removed the default `Everyone` permission and replaced it with explicit group-based access control.

### Domain Join

Joined a Windows 10 VM to the domain by pointing its DNS to the domain controller and running `sysdm.cpl`. Verified that domain users could authenticate and access only their authorized network share folders.

### Group Policy (GPO)

Created and linked a GPO to restrict Control Panel access for non-IT users. Configured a deny rule on the `Apply group policy` permission for non-IT accounts. Verified the policy:

- IT department user: Control Panel accessible
- Non-IT user: Control Panel blocked

A wallpaper restriction GPO was also applied as part of the same policy.

### Skills Demonstrated

- Windows Server 2016 installation and configuration in a virtualized environment
- Active Directory Domain Services setup and forest creation
- DHCP scope configuration with exclusion ranges and lease management
- DNS integration with AD DS
- User account and security group management in Active Directory
- Role-based access control via group membership and NTFS permissions
- Network share creation with department-level permission segmentation
- Domain joining Windows 10 clients
- Group Policy Object (GPO) creation, linking, and testing
---

## Lab 3 — Pi-hole on Raspberry Pi 4: Network-Wide DNS Filtering

**Date Completed:** Summer 2025  
**Device:** Raspberry Pi 4  
**Platform:** Raspberry Pi OS Lite (64-bit)  
**Full Documentation:** [pihole-raspi-build.md](pihole-raspi-build.md)

### Overview

Deployed Pi-hole on a Raspberry Pi 4 as a capstone project, creating a network-wide DNS sinkhole capable of blocking ads, trackers, and malicious domains for all devices on the network.

### What Was Built

- Flashed Raspberry Pi OS Lite to a microSD card using Raspberry Pi Imager
- Configured hostname, credentials, and SSH during imaging
- Booted the Pi, identified the network interface IP, and set a static IP via `dhcpcd.conf`
- SSH'd in remotely using PuTTY from a Windows machine
- Installed Pi-hole via the official install script
- Configured Cloudflare as upstream DNS with DNSSEC enabled
- Added Hagezi Pro DNS blocklist on top of the default StevenBlack list
- Verified blocking via canyoublockit.com extreme test — passed ✅

### Troubleshooting

SD card imaging required three attempts. Resolved a drive letter conflict by running GParted on a secondary machine to wipe and recreate the partition before re-imaging.

### Skills Demonstrated

- Raspberry Pi hardware setup and OS imaging
- Linux CLI — nano, apt, ip, SSH
- Static IP configuration via dhcpcd.conf
- Remote administration via SSH (PuTTY)
- DNS fundamentals and network-wide filtering concepts
- Pi-hole installation, configuration, and blocklist management
- Real-world troubleshooting under a project deadline
