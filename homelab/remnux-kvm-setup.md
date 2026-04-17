# REMnux VM Setup on QEMU/KVM (EndeavourOS)

**Author:** Theron Holler  
**Date:** April 5, 2026  
**Host Machine:** Lenovo ThinkPad E16 Gen 2 — EndeavourOS (systemd), KDE Plasma/Wayland, Ryzen 7 7735HS, 64GB RAM  
**Hypervisor:** QEMU/KVM with virt-manager  
**Guest:** REMnux Noble (Ubuntu 24.04-based malware analysis distro)

---

## Overview

This document covers the full process of importing, configuring, and setting up a REMnux virtual machine in QEMU/KVM on EndeavourOS. It includes every command attempted — successful and failed — along with the reasoning behind each decision and the troubleshooting steps taken along the way.

REMnux replaced an AthenaOS VM that was decommissioned on the same date after a failed kernel update left it unbootable. That incident is documented below before the REMnux setup steps.

---

## AthenaOS Decommission — April 5, 2026

### Background

AthenaOS was a security-focused Arch-based Linux distribution running as a VM in QEMU/KVM on the Lenovo ThinkPad. It was used for general security tooling and exploration. On April 5, 2026, a routine system update via `pacman` broke the VM's ability to boot, ultimately leading to the decision to decommission it and replace it with REMnux.

---

### Failure Mode

After the update, the AthenaOS VM failed to boot. The error presented at boot was:

```
Failed to mount /efi
Dependency failed for local file systems
```

The system dropped to a minimal rescue CLI rather than a full desktop environment. This indicated that systemd could not mount the EFI partition defined in `/etc/fstab`, which caused a cascade of failures for anything that depended on local filesystems being mounted.

---

### Troubleshooting Steps

#### Step 1 — Identify the failing unit

```bash
journalctl -xb | grep -i efi
```

**Command breakdown:**
- `journalctl` — reads the systemd journal (system log)
- `-x` — adds explanatory text to log entries
- `-b` — shows logs from the current boot only
- `| grep -i efi` — filters output for lines mentioning EFI (case-insensitive)

**Result:** Confirmed `efi.mount` as the failing unit. Output also showed the systemd dependency chain error — everything that depended on `/efi` being mounted had failed as a result.

---

#### Step 2 — Verify the fstab EFI entry

```bash
cat /etc/fstab
```

**Result:** The EFI entry was present and read:

```
UUID=CE3A-B9CA  /efi  vfat  rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=ascii,shortname=mixed,utf8,errors=remount-ro  0 2
```

---

#### Step 3 — Verify the UUID exists on the system

```bash
blkid | grep -i efi
```

**Command breakdown:**
- `blkid` — lists all block devices and their UUIDs and filesystem types
- `| grep -i efi` — filters for EFI-related entries

**Result:** UUID `CE3A-B9CA` matched `/dev/vda1` with type `vfat`. The UUID in fstab was correct and the partition existed.

---

#### Step 4 — Attempt manual mount

```bash
mount /dev/vda1 /efi
```

**Result:**

```
unknown filesystem type 'vfat' — dmesg may have more info after failed mount system call
```

The kernel could not mount the partition because the `vfat` filesystem module was not loaded. This was unexpected — `vfat` is a standard module that should always be available.

---

#### Step 5 — Attempt to load the vfat module

```bash
modprobe vfat
```

**Command breakdown:**
- `modprobe` — loads a kernel module into the running kernel
- `vfat` — the FAT32/EFI filesystem module

**Result:**

```
modprobe: FATAL: Module vfat not found in directory /lib/modules/6.18.20-1-lts
```

The module did not exist for the running kernel version. This revealed the root cause.

---

#### Step 6 — Identify the kernel mismatch

```bash
uname -r
```

**Result:** `6.18.20-1-lts` (currently running kernel)

```bash
ls /lib/modules
```

**Result:**
```
6.18.20-1-lts
6.18.21-1-lts
6.19.10-hardened1-1-hardened
6.19.9-hardened1-1-hardened
```

**Root cause identified:** The update had installed kernel `6.18.21-1-lts` but the system booted into the old `6.18.20-1-lts`. The modules for `6.18.20` were incomplete or missing — `vfat` was gone. The new `6.18.21` kernel had a full module set, but the system never booted into it. This is a classic partial update failure on Arch-based systems: the new kernel was installed, the old kernel's modules were partially removed, but the bootloader still pointed to the old kernel.

---

#### Step 7 — Attempt to load vfat from a different kernel's modules (failed)

Since `/lib/modules/6.18.21-1-lts/` existed and contained `vfat`, an attempt was made to load it manually:

```bash
insmod /lib/modules/6.18.21-1-lts/kernel/fs/fat/vfat.ko.zst
```

**Command breakdown:**
- `insmod` — manually inserts a kernel module, bypassing dependency resolution
- Path points to the `vfat` module from the `6.18.21` kernel

**Result:**

```
insmod: ERROR: could not insert module — Invalid module format
```

Expected. Kernel modules are compiled against a specific kernel version and cannot be loaded into a different running kernel. This confirmed the only real fix was to boot into `6.18.21-1-lts`.

---

#### Step 8 — Attempt kexec boot into working kernel (failed — not available)

```bash
kexec -l /boot/vmlinuz-linux-lts --initrd=/boot/initramfs-linux-lts.img --reuse-cmdline
```

**Command breakdown:**
- `kexec` — loads a new kernel into memory and executes it without a full hardware reboot, bypassing the bootloader entirely
- `-l` — loads the kernel without immediately executing
- `--reuse-cmdline` — reuses the current boot parameters

**Result:** `command not found`

`kexec` was not available in the rescue CLI environment. This eliminated the fastest recovery path.

---

#### Step 9 — Attempt to mask the failing mount and boot normally

The EFI partition was not strictly needed for normal system operation — it only contained bootloader files. Masking it would prevent systemd from trying to mount it and allow the system to boot fully.

```bash
systemctl mask efi.mount
```

**Command breakdown:**
- `systemctl mask` — completely prevents a unit from starting, even if another unit requests it
- `efi.mount` — the failing EFI mount unit

Then rebooted:

```bash
systemctl reboot
```

**Result:** The system attempted to boot but hung for approximately 5 minutes before the decision was made to hard-reset the VM. The mask did suppress the EFI mount failure, but the system stalled on other units downstream and never reached a login prompt. It was not determined exactly which unit caused the hang.

---

### Decision: Decommission

After exhausting available recovery paths without access to a live ISO or `kexec`, the decision was made to decommission the VM entirely. The reasoning:

- AthenaOS is a niche Arch-based security distro with a history of update instability
- The VM was used for general exploration, not a critical workflow
- Recovery would require mounting a live ISO, chrooting in, and reinstalling the LTS kernel and regenerating initramfs — a straightforward process, but the decision was made to replace the VM with REMnux instead
- REMnux had already been identified as the replacement target, offering a more stable Ubuntu 24.04 base with a purpose-built malware analysis toolset

**The VM was deleted from virt-manager with storage also deleted.** This was done by right-clicking the AthenaOS VM in virt-manager, selecting Delete, and checking the "Delete associated storage" option.

---

### Lessons From the AthenaOS Incident

- Arch-based systems can enter an unbootable state after updates if the kernel and its modules fall out of sync — this is a known risk of rolling release distributions
- Always snapshot VMs before running system updates, especially on Arch-based guests
- `kexec` is a powerful recovery tool but must be installed proactively — it cannot be installed after a boot failure
- Having a live ISO pre-attached or readily available to mount into a VM eliminates most recovery dead ends
- When a VM has no clean baseline and the recovery cost exceeds the value of the environment, nuking and rebuilding is a legitimate engineering decision — not a failure

---

## Why QEMU/KVM Instead of VirtualBox

QEMU/KVM was chosen because:

- The host machine was already running QEMU/KVM with a working Kali 2026.1 VM
- KVM uses kernel-level virtualization, communicating directly with the CPU for better performance
- Running two hypervisors simultaneously (VirtualBox + KVM) causes conflicts over `/dev/kvm`
- VirtualBox and KVM cannot both run simultaneously on the same host

The REMnux OVA is distributed as a "General OVA" format intended for most hypervisors. virt-manager does not natively support OVA imports, so a manual conversion process was required.

---

## Previous Attempt

A previous attempt to set up this same REMnux VM was made the night before and ultimately ended with the VM being nuked and rebuilt from scratch.

During that attempt, `spice-vdagent` was installed inside the VM but dynamic display scaling never worked after reboot. The VM was left in an unknown partial state — some things configured, some not, and no clean baseline to revert to. Rather than continuing to troubleshoot an uncertain environment, the decision was made to nuke it and start fresh with a better plan.

The exact cause of the failure in that session is unknown. This attempt changed several things simultaneously compared to the previous one:

- Shared memory was enabled in virt-manager before first boot
- Virtio + Spice display settings were verified before first boot by comparing against the working Kali VM
- Networking was fixed before installing spice-vdagent
- `sudo apt update` was run before `sudo apt install spice-vdagent`
- The correct order of operations was followed throughout

The result was a fully working setup. However, it cannot be determined which of these changes, if any individually, was responsible for the fix. Multiple variables changed at once and no single cause was isolated. This is an honest limitation of the troubleshooting process and worth noting for anyone trying to reproduce these results.

---

## Step 1: OVA Extraction

The downloaded `remnux-noble-amd64.ova` file was already extracted (an OVA is a tar archive). The relevant extracted file was confirmed present:

```
~/Downloads/remnux-noble-amd64-disk1.vmdk
```

---

## Step 2: Convert VMDK to QCOW2

QEMU/KVM uses QCOW2 (QEMU Copy-On-Write version 2) as its native disk format. The VMDK (VMware Virtual Machine Disk) file from the OVA was converted using `qemu-img`:

```bash
qemu-img convert -f vmdk -O qcow2 ~/Downloads/remnux-noble-amd64-disk1.vmdk ~/Downloads/remnux-noble-amd64.qcow2
```

**Command breakdown:**
- `qemu-img` — QEMU disk image utility for creating, converting, and inspecting disk images
- `convert` — subcommand to convert between formats
- `-f vmdk` — specifies the **f**ormat of the source file (VMware's disk format)
- `-O qcow2` — specifies the **O**utput format (QEMU's native format, supports snapshots)
- Source path — the extracted VMDK file
- Destination path — the QCOW2 file to be created

This process takes several minutes and produces no progress output. A blinking cursor is normal.

---

## Step 3: Import into virt-manager

virt-manager does not have a native OVA import option. The import was done via **File → New Virtual Machine → Import existing disk image**, pointing at the converted QCOW2 file.

**Settings chosen during import wizard:**

| Setting | Value |
|---|---|
| OS | Ubuntu 24.04 LTS |
| Memory | 8192 MiB (8GB) |
| CPUs | 4 |
| Storage | remnux-noble-amd64.qcow2 (100GB) |
| Name | REMnux |
| Network | Virtual network 'default' : NAT |

The **"Customize configuration before install"** checkbox was enabled to review and adjust hardware settings before first boot.

---

## Step 4: Pre-Boot Hardware Configuration

Before booting the VM for the first time, the following hardware settings were reviewed and confirmed in virt-manager's hardware details:

**Video:** Virtio  
**Display:** Spice server  
**Shared Memory:** Enabled (under Memory settings)

> **Why this matters:** The Kali 2026.1 VM on the same host uses Virtio video + Spice display and has perfect dynamic display scaling. Matching these settings was the key lesson from a previous failed attempt where the wrong display stack prevented `spice-vdagent` from working even after installation.

> **Note from official REMnux docs (KVM/QEMU section):** "Use Standard VGA display for the first boot. After booting, run `remnux install` to automatically install spice-vdagent and other KVM guest tools. You can then switch to Spice for better graphics." In practice, Virtio + Spice was already set by default and worked without needing to start on Standard VGA.

**OpenGL** and **3D acceleration** were left at their defaults and not touched.

---

## Step 5: First Boot

On first boot, REMnux logged in automatically to the GNOME desktop. The window was maximized and restored to get a sense of the display. The resolution appeared to be **1280x800** — significantly better than the 640x480 experienced in the previous failed attempt, suggesting Spice was at least partially working. No scaling settings were changed at this point. Network troubleshooting was the next priority.

---

## Step 6: Network Troubleshooting

### Initial test — no connectivity

```bash
ping -c 4 google.com
```

**Result:** `ping: google.com: Temporary failure in name resolution`

The VM had no network connectivity. The following attempts were made before finding the correct fix:

### Attempt 1 — dhclient (failed)

```bash
sudo dhclient
```

**Result:** `command not found`

REMnux Noble does not include `dhclient`. It uses `dhcpcd` instead.

### Attempt 2 — dhcpcd without interface (partial)

```bash
sudo dhcpcd
```

**Result:** `Dropped protocol specifier '.link' from 'enp1so.link'. Using 'enp1s0 (ifindex=2). no interfaces have a carrier`

The interface name was misread. The correct interface was `enp1s0`.

### Checking the interface

```bash
ip link show
```

**Result:** Interface `enp1s0` was visible but had no IP address — only a MAC address.

### Attempt 3 — dhcpcd with explicit interface (worked temporarily)

```bash
sudo dhcpcd enp1s0
```

**Result:** `sending commands to dhcpcd process` — connectivity established.

```bash
ping -c 4 google.com
```

**Result:** 4 packets transmitted, 0% loss. Network confirmed working.

**However:** This fix did not survive a reboot. `dhcpcd enp1s0` had to be run manually every boot.

---

## Step 7: Install spice-vdagent for Dynamic Display Scaling

With networking up, `spice-vdagent` (the Spice guest agent that enables dynamic display resizing and clipboard sharing between host and VM) was installed.

### Attempt 1 — install without updating package list (failed)

```bash
sudo apt install spice-vdagent -y
```

**Result:** `E: Unable to locate package spice-vdagent`

The local package list was stale and did not yet know the package existed.

### Fix — update package list first

```bash
sudo apt update
```

This refreshed the local package list from the internet repositories.

### Attempt 2 — install after update (succeeded)

```bash
sudo apt install spice-vdagent -y
```

**Result:** Package installed successfully.

> **Lesson:** On Debian/Ubuntu-based systems, always run `sudo apt update` before `sudo apt install`. Without it, apt works from a cached package list that may not include recently available packages.

### Reboot to activate spice-vdagent

```bash
sudo reboot
```

After reboot, **View → Autoresize VM with window** was enabled in virt-manager for the first time. The VM display filled the screen and resized dynamically with the window — full dynamic scaling working, matching Kali's behavior exactly. Copy/paste between host and VM also worked immediately.

---

## Step 8: Making Network Persistent (Permanent Fix)

After the reboot that activated `spice-vdagent`, network connectivity was lost again as expected. A permanent fix was needed.

### Diagnosing what manages networking

```bash
systemctl list-units --type=service | grep -i net
```

**Result:** Showed `NetworkManager.service` as active and running.

### Checking NetworkManager connection profiles

```bash
sudo nmcli con show
```

**Result:** Blank output — NetworkManager had no connections configured at all.

### Root cause — managed=false in NetworkManager config

```bash
cat /etc/NetworkManager/NetworkManager.conf
```

**Result:**
```
[main]
plugins=ifupdown,keyfile
[ifupdown]
managed=false
[device]
wifi.scan-rand-mac-address=no
```

The `managed=false` line tells NetworkManager to leave all network interfaces alone, deferring to the older `ifupdown` system. However, `ifupdown` was also not configured to manage `enp1s0`. The interface fell into a gap where nothing was managing it automatically.

> **Why REMnux ships this way:** This is likely intentional. REMnux is a malware analysis distro designed to be air-gapped during analysis sessions. Having networking off by default forces analysts to deliberately enable it, reducing the risk of accidentally allowing malware to phone home. For a lab setup where persistent networking is acceptable, this can be changed.

### Fix — tell NetworkManager to take control

```bash
sudo sed -i 's/managed=false/managed=true/' /etc/NetworkManager/NetworkManager.conf
```

**Command breakdown:**
- `sudo` — administrator privileges
- `sed` — Stream EDitor for modifying text in files
- `-i` — edit the file **i**n place
- `'s/managed=false/managed=true/'` — substitution: replace `managed=false` with `managed=true`

### Restart NetworkManager to apply the change

```bash
sudo systemctl restart NetworkManager
```

### Create a connection profile for enp1s0

```bash
sudo nmcli con add type ethernet ifname enp1s0
```

**Result:** `Connection 'ethernet-enp1s0' successfully added.`

### Attempt to bring up the connection (first try — failed)

```bash
sudo nmcli con up ethernet-enp1s0
```

**Result:** `Error: Connection activation failed: No suitable device found for this connection (device lo not available because device is strictly unmanaged).`

NetworkManager grabbed `lo` (the loopback interface) instead of `enp1s0`. Adding the interface name explicitly fixed this.

### Attempt to bring up with explicit interface (second try — failed)

```bash
sudo nmcli con up ethernet-enp1s0 ifname enp1s0
```

**Result:** `Error: Connection activation failed: Connection 'ethernet-enp1s0' is not available on device enp1s0 because device is strictly unmanaged`

At this point `managed=false` was still in effect. The config file change and NetworkManager restart had not yet been applied in the correct order. After confirming the config change and restarting NetworkManager, the command succeeded.

### Bring up the connection (succeeded after restart)

```bash
sudo nmcli con up ethernet-enp1s0
```

**Result:** `Connection successfully activated`

### Verify connectivity

```bash
ping -c 4 google.com
```

**Result:** 4 packets, 0% loss. Network confirmed.

### Verify persistence after reboot

```bash
sudo reboot
```

After reboot, `ping -c 4 google.com` succeeded without any manual intervention. Network now auto-connects on every boot.

---

## Step 9: Optional — Disable Auto-Connect for Malware Analysis Sessions

For sessions where the VM should be air-gapped (e.g., detonating malware samples), auto-connect can be toggled:

**Disable auto-connect (air-gapped mode):**
```bash
sudo nmcli con modify ethernet-enp1s0 connection.autoconnect no
```

**Re-enable auto-connect:**
```bash
sudo nmcli con modify ethernet-enp1s0 connection.autoconnect yes
```

**Manually connect when needed (after disabling auto-connect):**
```bash
sudo nmcli con up ethernet-enp1s0
```

---

## Step 10: Running remnux install

### What the official docs say

The official REMnux documentation specifies `remnux install` for both initial setup and ongoing updates — including for KVM/QEMU environments specifically. Initially `remnux upgrade` was run instead, which produced identical results. Both commands invoke the same underlying process.

> **Note:** Ran `remnux upgrade` first instead of `remnux install` — both produce identical results. `remnux install` is the officially documented command.

### Running the installer

```bash
remnux install
```

This process ran approximately 1029 states (installation steps), covering hundreds of malware analysis tools. It took approximately 30–45 minutes.

### Result

```
Successful states: 1024
Failed states: 5
```

### Investigating the failures

```bash
remnux results
```

**Root cause:** A single failure in the Speakeasy Windows emulator tool:

```
Specified sha256 checksum for https://github.com/mandiant/speakeasy/raw/master/examples/emu_dll.py
(fd220ee9f484d071418f060f65119e4b16ec37c0ce9a014e991546d92d5a1d96) does not match actual checksum
(898160b3b01c46b66600d44f5c32d068a8050b2c55a5a18a40ef2c4f3263465c).
```

4 additional failures cascaded from this single root cause.

### Root cause analysis

This failure is a known upstream bug in REMnux's salt-states repository. Mandiant updated `emu_dll.py` on their GitHub, but REMnux's `speakeasy.sls` state file still contains the old expected checksum. This is verifiable by examining the file directly at:

`https://github.com/REMnux/salt-states/blob/master/remnux/python3-packages/speakeasy.sls`

The hardcoded hash in that file matches the outdated value in the error message exactly. This same type of Speakeasy checksum mismatch has occurred previously and is documented in REMnux's GitHub issue tracker. There is no user-side fix — the REMnux team must update their salt state file to match the new hash.

**1024 out of 1029 states succeeded.** All major tools installed correctly.

---

## Step 11: Snapshot

After confirming the installation results, a snapshot was taken in virt-manager using the snapshot manager (the stacked monitors icon in the toolbar).

**Snapshot name:** `REMnux-clean-baseline`  
**Snapshot type:** Internal  
**State:** Post-install, networking persistent, spice-vdagent working, display scaling functional

> **Note:** virt-manager displayed a non-critical UI error (`'NoneType' object has no attribute 'get_width'`) after snapshot creation. The snapshot appeared correctly in the list and is valid. This is a known virt-manager display glitch, not a snapshot failure.

---

## Summary of Key Lessons

- Always run `sudo apt update` before `sudo apt install` on Debian/Ubuntu systems
- REMnux sets `managed=false` in NetworkManager by default — likely intentional for air-gapped malware analysis workflows
- Match the host VM display settings (Virtio video + Spice) to a known working VM before first boot
- `spice-vdagent` must be installed inside the guest VM for dynamic display scaling to work
- `remnux install` and `remnux upgrade` produce identical results; `remnux install` is the officially documented command
- Speakeasy checksum failures are an upstream REMnux/Mandiant issue and will resolve when REMnux updates their salt states
- Snapshot VMs before updates — especially on rolling release distros
- Always have a live ISO available to mount into a VM for recovery purposes
- When a non-critical VM reaches a recovery dead end, rebuilding is a valid engineering decision

---

## Tools Notable in This REMnux Install

During the installation, the following tools were observed being configured (partial list):

| Tool | Purpose |
|---|---|
| FakeNet-NG | Simulates internet services to capture malware network traffic without real connectivity |
| Radare2 | Full reverse engineering framework for binary disassembly and analysis |
| Malcat | GUI-based binary analysis and reverse engineering tool |
| LIEF | Library for parsing and modifying PE, ELF, and Mach-O executables |
| PyInstxtractor-NG | Unpacks PyInstaller-bundled executables back to Python source |
| Burp Suite Community | Web application penetration testing proxy |
| BXor (brxor.py) | XOR decryption tool for deobfuscating malware strings |
| Wireshark | Network packet capture and analysis |
| Wine | Windows application compatibility layer for running PE samples |
| Speakeasy | Windows emulator for dynamic malware analysis (failed — upstream bug) |
| spice-vdagent | Spice guest agent for display scaling and clipboard (installed manually) |

---

---

## Post-Setup Incident: Display Scaling Regression — April 12, 2026

### Background

After the clean baseline snapshot was taken and the VM was not in use for several weeks, the display scaling issue returned. The VM was again stuck at 640x480 and the mouse cursor was invisible in the virt-manager window. Running `sudo apt update && sudo apt full-upgrade -y` followed by a reboot did not fix it. `spice-vdagent` was already installed and `spice-vdagentd` was showing as active.

This was the third time the display issue had appeared across REMnux VM attempts. Each previous attempt had treated it as a `spice-vdagent` problem. This time the root cause was actually identified.

---

### Diagnosis

The qemu-guest-agent channel was connected, which allowed running diagnostic commands inside the VM from the host without needing a usable display.

**Step 1 — Verified spice-vdagentd was running:**

```bash
systemctl status spice-vdagentd
```

Result: active (running). Package `spice-vdagent` 0.22.1-4build3 installed. This ruled out the daemon as the problem.

**Step 2 — Checked the user-session agent:**

```bash
# run via guest-agent as root, checking the remnux user's systemd session
systemctl --user -M remnux@ status spice-vdagent
```

Result: `inactive (dead)`. The user-level `spice-vdagent.service` was not running. The autostart desktop entry at `/etc/xdg/autostart/spice-vdagent.desktop` existed and was correctly configured — but it was never triggering.

**Step 3 — Checked the session type:**

```bash
cat /proc/$(pgrep gnome-shell | head -1)/environ | tr '\0' '\n' | grep XDG_SESSION_TYPE
```

Result:

```
XDG_SESSION_TYPE=wayland
```

**Root cause identified:** The guest was running GNOME on Wayland. `spice-vdagent` only works on X11. In a Wayland session it does nothing — the user-session agent silently fails to connect and the SPICE channel has no mechanism to negotiate resolution changes. Every previous fix attempt (reinstall, restart, `apt upgrade`) was targeting the wrong layer entirely.

---

### Fix

`/etc/gdm3/custom.conf` already contained the correct fix as a commented-out line:

```
#WaylandEnable=false
```

Uncommented it:

```bash
sudo sed -i 's/#WaylandEnable=false/WaylandEnable=false/' /etc/gdm3/custom.conf
```

Rebooted. GDM started an X11 session. `spice-vdagent` connected immediately. Resolution auto-resized with the virt-manager window and the cursor appeared.

---

### Why This Kept Happening

REMnux Noble (Ubuntu 24.04-based) defaults to GNOME on Wayland via GDM. The Kali 2026.1 VM on the same host had previously had its display issue fixed by *switching to Wayland* (X11 → Wayland) — the opposite fix. Both fixes are correct for their respective environments: Kali's GNOME/Wayland session handles SPICE display through Mutter's native compositor path, while REMnux's setup required X11 for `spice-vdagent` to function.

The deeper lesson: display scaling in QEMU/KVM+SPICE is not just about `spice-vdagent`. It depends on the interaction between the SPICE protocol, the guest's display server (X11 vs Wayland), and whether the compositor handles SPICE resize hints natively. Treating every instance as a `spice-vdagent` reinstall problem missed this entirely.

---

### Permanence

`WaylandEnable=false` in `/etc/gdm3/custom.conf` is a persistent config file. It survives reboots and `apt full-upgrade`. The only way it reverts is if a future GDM package upgrade ships a new `custom.conf` and overwrites it — apt prompts on config file conflicts in that case.

---

---

## Post-Setup Incident: Display Still Stuck at 640x480 After Wayland Fix — April 12, 2026

### Background

After the Wayland fix (above) forced GDM to start an X11 session, the mouse cursor became visible — confirming X11 was active and `spice-vdagent` was running. However, the display was still stuck at 640x480 and did not auto-resize with the virt-manager window. The `spice-vdagent` package was installed and both the system daemon and the user session agent were running.

---

### Diagnosis

Diagnostic commands were run inside the VM via `virsh qemu-agent-command` from the host, since the QEMU guest agent channel (`org.qemu.guest_agent.0`) was connected and usable.

**Step 1 — Confirmed VM XML was correct:**

```bash
virsh -c qemu:///system dumpxml REmnux | grep -A5 -E 'graphics|channel|controller|serial'
```

Result: `virtio-serial` controller present, `spicevmc` channel with `state='connected'`, SPICE graphics on port 5900. The VM configuration was not the problem.

**Step 2 — Confirmed both spice-vdagent processes were running:**

```bash
ps aux | grep spice-vdagent
ls /run/spice-vdagentd/
```

Result: Both `spice-vdagentd` (root, PID 1480) and `spice-vdagent` (remnux user, PID 1456) were running. The socket `/run/spice-vdagentd/spice-vdagent-sock` existed. No problem here.

**Step 3 — Checked xrandr:**

```bash
DISPLAY=:0 XAUTHORITY=/run/user/1000/gdm/Xauthority xrandr
```

Result:

```
Screen 0: minimum 320 x 200, current 640 x 480, maximum 4096 x 4096
None-1 connected primary 640x480+0+0 (normal left inverted right x axis y axis) 0mm x 0mm
   640x480       60.00*+
```

The output name `None-1` is abnormal. In a working KMS/DRM setup with virtio-gpu, outputs are named `Virtual-1` or similar. `None-1` means X11 found no real DRM device and fell back to a framebuffer with no resize capability. `spice-vdagent` cannot resize what it can't negotiate with xrandr.

**Step 4 — Checked the Xorg log:**

```bash
cat /home/remnux/.local/share/xorg/Xorg.0.log | head -80
```

Result (critical line):

```
[14.299] Kernel command line: BOOT_IMAGE=/boot/vmlinuz-6.8.0-107-generic root=UUID=... ro nomodeset
```

**Root cause identified:** The kernel was booting with `nomodeset`. This parameter disables KMS (Kernel Mode Setting), which prevents the `virtio-gpu` DRM driver from initializing. Without a DRM device, X11's `modesetting` driver has no KMS backend to work with — it falls back to a dumb framebuffer at the BIOS-level 640x480 resolution. The display output gets named `None-1` because there is no real device behind it. `spice-vdagent` was running correctly but had nothing it could resize.

---

### Fix

**Read the current GRUB config:**

```bash
cat /etc/default/grub
```

Result confirmed:

```
GRUB_CMDLINE_LINUX_DEFAULT=" nomodeset"
```

**Remove `nomodeset`:**

```bash
sed -i 's/GRUB_CMDLINE_LINUX_DEFAULT=" nomodeset"/GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"/' /etc/default/grub
```

**Regenerate GRUB config:**

```bash
update-grub
```

**Reboot:**

```bash
reboot
```

After reboot, xrandr shows `Virtual-1` with multiple resolution modes available, the VM window auto-resizes correctly, and display scaling works as expected.

---

### Why nomodeset Was Set

`nomodeset` is sometimes added to the kernel command line as a workaround for display issues during initial VM setup — particularly when a VM first boots with a generic VGA framebuffer before GPU drivers are established. It prevents the GPU driver from taking over the display during boot, which can resolve black screens or corrupt display output in certain configurations. In this case it appears to have been set at some point as a troubleshooting step and never removed, causing the opposite problem: the GPU driver never initialized at all.

---

### Permanence

The change is permanent. `/etc/default/grub` is a config file that persists across reboots. `update-grub` regenerated `/boot/grub/grub.cfg` from it. The `nomodeset` parameter is gone from every future boot unless manually re-added.

---

### Key Lesson

`nomodeset` disables KMS globally. In a QEMU/KVM+SPICE VM with `virtio-vga`, removing KMS means the virtio-gpu DRM driver never initializes — X11 falls back to a basic framebuffer, xrandr output shows as `None-1` with only 640x480, and SPICE display negotiation through `spice-vdagent` stops working entirely. If dynamic display scaling stops working after what should be a working `spice-vdagent` setup, check `dmesg` or the Xorg log for `nomodeset` in the kernel command line before assuming the agent is broken.

---

*Part of the cybersecurity homelab portfolio at [github.com/TheronHoller2025/cybersec-portfolio](https://github.com/TheronHoller2025/cybersec-portfolio)*
