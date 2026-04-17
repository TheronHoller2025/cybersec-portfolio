# QEMU/KVM Virtualization Lab
## EndeavourOS Lenovo ThinkPad E16 Gen 2 — 2026

---

## Overview

Set up a QEMU/KVM virtualization environment on my primary workstation
to run dedicated security VMs without dual booting. The goal was a
proper isolated attack platform separate from the host system.

Getting the network bridge working correctly took the most
troubleshooting time.

---

## Host System

| Field | Details |
|---|---|
| Machine | Lenovo ThinkPad E16 Gen 2 |
| OS | EndeavourOS (systemd, KDE Plasma 6/Wayland) |
| CPU | AMD Ryzen 7 7735HS |
| RAM | 64GB |
| Hypervisor | QEMU/KVM with libvirt |

---

## Virtual Machines

| VM | Purpose | Status |
|---|---|---|
| Kali Linux 2026.1 | Primary penetration testing platform | Active |
| REMnux | Malware analysis sandbox | Active |

---

## Packages Installed

- qemu
- libvirt
- virt-manager
- dnsmasq

---

## What Worked

- QEMU/KVM installed and running
- Kali Linux 2026.1 VM deployed and booting
- REMnux VM deployed and booting
- virt-manager GUI working for VM management

---

## Network Issue — Resolved

Kali VM's eth0 was getting a 169.254.x.x (APIPA) address instead
of a proper DHCP lease from virbr0 when Mullvad VPN was active.

**Root cause:** Mullvad VPN interfering with the virtual network
bridge when active before the VM starts.

**Fix:**
- Start the VM BEFORE enabling Mullvad, OR
- Turn Mullvad off and back on after the VM is running
- Patience — DHCP lease sometimes takes a moment to assign

Issue fully resolved. VM connects correctly when startup
order is followed.

---

## What I Learned

- How libvirt manages virtual networks
- How virbr0 NAT bridge works
- What APIPA addresses mean and why they happen
- Difference between NAT, bridged, and host-only networking
- How to use virt-manager for VM lifecycle management

---

## References

- QEMU/KVM Arch Wiki: https://wiki.archlinux.org/title/QEMU
- libvirt networking: https://wiki.libvirt.org/VirtualNetworking.html
- Kali Linux: https://www.kali.org/
- REMnux: https://remnux.org/
- REMnux KVM/QEMU setup: [remnux-kvm-setup.md](remnux-kvm-setup.md)

---

*Setup: Early 2026 — updated April 2026*
