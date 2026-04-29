# Theron's Cybersecurity Master Notes
*Compiled from learning sessions — updated regularly*

---

## 1. LINUX FUNDAMENTALS

### Kernel vs OS
- **Kernel** = the core engine. Talks directly to hardware. Manages memory, processes, devices. Linux IS just the kernel.
- **OS** = kernel + everything built on top (shell, desktop environment, applications)
- GNU/Linux = GNU software + Linux kernel combined
- Think of it as: kernel = building's electrical/plumbing system. OS = the entire building.

### Key Commands
- `ps aux` — List all running processes (a=all users, u=user format, x=no terminal required)
- `ps aux | grep root` — Filter processes to show only root-owned ones
- `lsmod` — List loaded kernel modules
- `lsof` — List open files (everything in Linux is a file — network connections, devices, etc.)
- `ss` — Modern replacement for netstat. Shows active network connections
- `netstat` — Older tool showing network connections
- `strace` — Traces system calls a process makes. Watches what a program asks the kernel to do.
- `chmod` — Change file permissions
- `kill` / `pkill` — Terminate processes
- `more` / `cat` / `less` — Read file contents in terminal
- `ls -a` — List directory contents including hidden files (dot files)
- `ping -c 1` — Send one ping to test if a host is alive

### Piping ( | )
- The `|` symbol takes output from one command and feeds it as input to the next
- Example: `ps aux | grep root | awk` = list all processes → keep only root ones → format output
- Chain as many pipes as needed to slice data precisely

### File Permissions (chmod)
- `chmod 600 /path/to/file` = only owner can read and write, nobody else
- `-R` flag = recursive, applies to all files inside directory too
- `chmod 600 /root/fsociety/` = locks directory so only you have access

### Kernel Modules
- Linux kernel is modular — loads/unloads "plugins" called modules without rebooting
- WiFi drivers, GPU drivers = kernel modules
- View with `lsmod`
- Kernel rootkits inject themselves AS modules, gaining same trust level as kernel itself

---

## 2. NETWORKING

### IP Addresses
- Valid IPv4 format: four numbers 0-255 separated by dots (e.g. 192.168.1.1)
- Numbers above 255 in any section = invalid/fake
- 169.254.x.x = self-assigned address, means DHCP failed (no lease received)

### Network Topology
- Network maps show logical connections between devices, not necessarily physical cables
- Blue lines on maps = network pathways/connections
- Red lines = degraded or high-traffic connections (like Google Maps red = congested road)
- Cloud shapes = network segments or upstream connections
- Colored nodes = server health states (red=down/compromised, green=healthy)

### Network Commands
- `ping` = tests if a host is reachable and measures response time
- "Destination unreachable [time out]" = host not responding, either down or blocked
- "Response: 1.0s packet loss: 1%" = host IS alive but degraded connection — key indicator

### Key Concepts
- **DHCP** = Dynamic Host Configuration Protocol — automatically assigns IP addresses
- **Gateway** = the router/device that connects your network to other networks
- **Packet loss** = percentage of data packets that don't reach destination
- **Lateral movement** = using one compromised system as a stepping stone to reach others
- **Pivoting** = moving from one compromised system to others on same network

---

## 3. SECURITY CONCEPTS

### Types of Attacks
- **DDOS** = Distributed Denial of Service — overwhelming a server with traffic to take it offline
- **Social Engineering** = manipulating humans rather than hacking systems technically
- **Pretexting** = creating a fake scenario to extract information (e.g. "I'm from bank fraud dept")
- **Vishing** = Voice phishing — using phone calls to extract sensitive info
- **Insider Threat** = attack from someone with legitimate access (most dangerous type)

### OSINT (Open Source Intelligence)
- Gathering information from publicly available sources
- Social media profiles, dating profiles, public records = all valid OSINT sources
- Personal details from profiles (pet names, sports teams) = common password components
- OSINT → Wordlist → Brute Force = complete real-world attack chain

### Social Engineering Principles
- Works by exploiting trust, authority, urgency, and fear
- Target's stress level matters — panicked people make poor decisions
- Trusted experts' words are rarely questioned in crisis situations
- Reframing a situation makes targets think they're making their own choice
- The warning "this sounds like protecting the company" bypasses critical thinking

### Indicators of Compromise (IOCs)
- System running slow unexpectedly
- Network traffic at unusual hours
- Files appearing or disappearing
- Login attempts in logs
- Commands behaving strangely
- Processes with unusual names running as root
- Any active host responding when everything else is dead silent

---

## 4. MALWARE & ROOTKITS

### What is a Rootkit
- Malware designed to hide itself from the OS
- Maintains persistent access even after reboots
- Masks other malicious processes
- Named "root" kit because it targets root (admin) level access

### Rootkit Levels (from least to most dangerous)

**Level 1 — User Space**
- Hides in regular applications
- Replaces commands like ps, ls, netstat with fake versions
- Easiest to detect and remove

**Level 2 — Kernel Level**
- Inserts as a kernel module
- Lies to the OS itself, not just commands
- Security tools get lied to because they depend on kernel
- How Darlene's rootkit in Mr. Robot worked

**Level 3 — Bootkit**
- Embeds in bootloader — runs BEFORE OS loads
- Survives OS reinstall
- OS boots already compromised

**Level 4 — Firmware/UEFI**
- Hides inside hardware firmware (motherboard, hard drive, network card)
- Wiping OS does nothing
- NSA's ANT Catalog contained real tools at this level (IRATEMONK, DEITYBOUNCE)
- Requires firmware reflash to remove

**Level 5 — Hypervisor (theoretical/nation-state)**
- Creates fake virtualization layer beneath OS
- Entire OS runs inside it unknowingly
- Essentially undetectable from inside OS

### Detecting Rootkits
- `rkhunter` / `chkrootkit` — rootkit detection tools
- `strace` — watches system calls, can reveal hidden behavior
- `lsof` — reveals files rootkit is keeping open
- `ss` / `netstat` — reveals hidden network connections
- Comparing firmware hashes against manufacturer known-good hashes
- **CHIPSEC** (Intel tool) — firmware integrity verification
- **Tripwire** — monitors filesystem for any changes from baseline

### Limitation: strace and lsof can NOT catch firmware rootkits
- Both run inside the OS
- Firmware rootkit sits BELOW the kernel
- Loaded and in control before kernel starts
- Corrupted foundation reporting false data to everything above it

### Dead Man's Switch (advanced rootkits)
- If process is killed: phones home to attacker, triggers data exfiltration, wipes evidence, spawns child process to relaunch
- Correct order: isolate network FIRST, snapshot/image system, THEN kill process

### Incident Response Workflow
1. Scan — find the live host among dead ones
2. Enumerate — ps aux | grep root to find suspicious processes
3. Trace — strace equivalent to watch malware behavior
4. Follow — malware reveals its own directory by accessing files while traced
5. Investigate — ls -a to see contents, more/cat to read files
6. Decision — kill it or study it? (Always image first)

---

## 5. MALWARE ANALYSIS & SANDBOXING

### Sandbox
- Isolated environment where malware can run freely without harming real systems
- Malware thinks it's in a real system — you watch through the glass taking notes

### Tools
- **REMnux** — Linux distro built for malware analysis, pre-loaded with hundreds of tools. Active VM running on Lenovo ThinkPad via QEMU/KVM.
- **Cuckoo Sandbox** — automated sandbox, throws suspicious file at it, produces full report
- **Ghidra** — free reverse engineering tool (made by NSA), reads compiled malware code
- **Binary Ninja** — alternative reverse engineering tool

### Learning Path for Malware
1. Now: Understand how malware BEHAVES — what traces it leaves (rkhunter, chkrootkit)
2. Next: Analyze malware in sandbox (REMnux, Cuckoo)
3. After Network+/Security+: Reverse engineering with Ghidra
4. PNPT/OSCP level: Write proof of concept malware for controlled lab environments

---

## 6. TOOLS REFERENCE

### Kali Linux Tools
- **nmap** — network scanning and enumeration
- **chkrootkit** — rootkit detection
- **rkhunter** — rootkit hunter
- **strace** — system call tracer
- **lsof** — list open files

### Useful Linux Tools
- **htop** — interactive process viewer
- **Tripwire** — filesystem integrity monitor
- **CHIPSEC** — firmware security checker
- **Firejail** — Linux sandboxing tool using namespaces and seccomp-bpf to isolate processes. Wraps any application to restrict filesystem access, network access, and system calls. Used to sandbox Chromium/Messenger on archlaptop.
- **paccache** — Arch Linux package cache cleaner (part of pacman-contrib). Keeps only the N most recent versions of each cached package.
- **yay** — AUR helper for Arch Linux. Wraps pacman and handles AUR package builds transparently alongside official packages.

### Password Tools
- **Wordlists** — lists of potential passwords to try
- **Brute force** — systematically trying every combination
- Custom wordlists built from OSINT = much more effective than generic lists

---

## 7. CERTIFICATION PATH

### My Path
Tech+ → A+ → Network+ → Security+ → eJPT → PNPT → OSCP

### Status
- Google IT Support Professional Certificate ✅
- Tech+ — in progress
- A+ — upcoming
- Network+ — upcoming  
- Security+ — target for SOC Tier 1 and analyst roles
- eJPT → PNPT → OSCP — long term

### Key Point
Security+ opens doors for SOC Tier 1 Analyst roles and remote work opportunities.

---

## 8. MR. ROBOT TECHNICAL BREAKDOWN (S1E1)

### Real Commands Shown
- `ps aux | grep root` — find root processes on compromised server
- `strace` equivalent (astu trace) — attach to process to watch system calls
- `ls -a /root/fsociety/` — list hidden files in directory
- `more readme.txt` — read file contents
- `chmod -R 600 /root/fsociety/` — lock directory to owner-only access
- `sudo kill [PID]` — kill a process by ID
- `ping -c 1` — single ping to test host

### How Elliot Found the Infected Server
- Scanned all servers — everything returned "destination unreachable [time out]"
- One server responded with "response: 1.0s packet loss: 1%"
- Live host among dead ones = immediate red flag

### How Elliot Found the fsociety Directory
- Attached strace to malicious process (PID 244)
- Watched what files the malware accessed in real time
- Malware accessed its own files in /root/fsociety/
- Malware ratted itself out by going home while being watched

### Process Names That Revealed the Rootkit
- evilscorpwbthread — named to look like legitimate Evil Corp processes (camouflage)
- evilscorpwbworker — same tactic
- cpuset running ./01dat — unusual process running from current directory

### The chmod Decision
- Read "LEAVE ME HERE" in readme.txt
- Started delete command, answered N to cancel
- Instead ran chmod 600 to lock directory — only he has access
- Went from incident responder to co-conspirator in one command

### Michael Hansen OSINT Attack Chain
1. Borrowed Michael's phone, called himself = got his number
2. Saw Bank of E app on Michael's phone = identified his bank
3. Found Michael's dating profile = extracted personal details
4. Built custom wordlist from profile (Yankees, Flipper the dog)
5. Brute forced bank login — FAILED
6. Failure revealed the profile was fake = Michael Hansen is an alias
7. Confronted him with all evidence + fabricated 15 year old threat (bluff)
8. Michael folded, gave up Flipper the dog, agreed to break up with Krista

---

## 9. HOMELAB & MACHINES

### Machine Fleet
| Machine | OS | Key Specs |
|---|---|---|
| Lenovo ThinkPad E16 Gen 2 (Lenovo-EOS) | EndeavourOS (KDE) | Ryzen 7 7735HS, 64GB, QEMU/KVM, Kali 2026.1 VM + REMnux VM |
| HP ProBook 450 G7 (archlaptop) | Arch Linux (KDE) / Kali Linux (KDE) — dual boot | i7-10510U, 32GB |
| Dell Inspiron 5680 (BigDell) | CachyOS (KDE) | i7-8700, GTX 1060, 16GB, 4-monitor setup |
| Dell OptiPlex 3040 | Debian | i7-6700, 16GB, Pi-hole host, backup target |
| Dell Inspiron 3501 | Linux Mint 22.3 (Cinnamon) | i5-1035G1, 8GB, QEMU/KVM, Devuan VM |
| Lenovo ThinkBook | Windows 11 Pro | Ryzen 5 7430U, VirtualBox, Kali VM |

### Completed Projects
- Local IT training program labs: Cisco Packet Tracer multi-subnet design, Windows Server 2016 (AD DS, DHCP, DNS, GPO, network shares), Pi-hole Raspberry Pi 4 capstone — documented in it-training.md
- TP-Link Archer BE700 router hardened from scratch (WPA3, WireGuard VPN, IoT VLAN) — DNS now via Pi-hole on camel
- Lynis audit + full system hardening on HP ProBook 450 G7 (archlaptop)
- Authorized pentest against Jellyfin server (formal report written)
- QEMU/KVM setup with Kali 2026.1 VM and REMnux VM on EndeavourOS
- AthenaOS VM decommissioned April 2026 after kernel update left it unbootable — replaced with REMnux
- Firejail + Chromium sandboxed Messenger web wrapper on archlaptop — .desktop file on KDE desktop
- Custom monthly Arch maintenance script at /usr/local/bin/arch-maintenance on archlaptop — launched via desktop .desktop file
- Jellyfin Media Player v1.12.0-6 built from AUR via yay on archlaptop (March 20th, 2026)
- WireGuard port migration (UDP 51820 → 443) on camel — resolved carrier-level blocking; DDNS migrated to eyeoftheneedle.dev via Cloudflare API with script on camel
- camel (OptiPlex 3040) rebuilt as Debian box — SSH, WOL, WireGuard, DDNS, Pi-hole, backup target (April 2026)
- Pi-hole deployed on camel — network-wide DNS filtering, Unbound upstream, DNSSEC, custom blocklist, router DHCP updated
- Unbound deployed on camel as recursive DNS resolver (127.0.0.1#5335) — Pi-hole upstream now points to Unbound, DNSSEC validated (April 28th, 2026)
- Tablet SSH configured (Termux on Samsung Galaxy S10 FE) — ed25519 keys, root and tman aliases via WireGuard
- Samsung Galaxy S23 Ultra added as WireGuard peer on camel's tunnel (UDP 443) — April 28th, 2026

---

## 10. CAREER & REMOTE WORK PATH

### Target Role
Remote cybersecurity work — SOC Tier 1 Analyst or IT Support — open to international remote teams

### Portfolio Building Blocks
- Jellyfin pentest report
- Pi-hole Raspberry Pi capstone (local IT training program)
- Network topology SVG diagram (built with Claude)
- Lynis hardening audit documentation
- TP-Link router hardening build
- Kali VM setup on QEMU/KVM

### TryHackMe Path
Start here before HackTheBox:
1. Pre-Security path (free, networking basics)
2. SOC Level 1 path (directly relevant to SOC career)
3. Jr Penetration Tester path (leads to eJPT)

### Remote Work Platforms to Target
- HackerOne / Bugcrowd — bug bounty (location independent)
- LinkedIn remote SOC roles
- MSSPs (Managed Security Service Providers) hiring remote Tier 1 analysts

---

*Last updated: April 28th, 2026 — ongoing*
