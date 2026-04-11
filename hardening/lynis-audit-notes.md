# Linux System Hardening — Lynis Audit
## Arch Linux — HP ProBook 450 G7 (archlaptop)

---

## Overview

Performed a full Lynis security audit on my Arch Linux system and
worked through the recommendations systematically. Lynis is a
battle-tested open source security auditing tool for Linux systems.
It scans the system and produces a hardening index score along with
detailed recommendations.

The goal was to understand what a real security audit looks like
from the inside and actually implement the fixes rather than just
reading about them.

---

## Tool Used

| Field | Details |
|---|---|
| Tool | Lynis |
| Version | Current as of early 2026 |
| Target | Arch Linux, HP ProBook 450 G7 |
| Kernel | Zen kernel |

---

## What Lynis Checks

- Boot and filesystem security
- Kernel hardening parameters
- Authentication and password policies
- SSH configuration
- File permissions and SUID binaries
- Network configuration
- Logging and auditing
- Installed software and packages
- Malware scanners present
- Firewall status

---

## What I Actually Did

### SSH Hardening
- Disabled root login via SSH
- Configured key-based authentication only
- Disabled password authentication
- Changed default SSH port

### rkhunter Baseline
- Installed rkhunter (rootkit hunter)
- Ran initial baseline scan
- Stored baseline for future comparison
- Understanding: if rkhunter finds changes later
  it means something may have modified system files

### File Permission Review
- Reviewed SUID binaries — programs that run with
  elevated privileges regardless of who runs them
- Removed unnecessary SUID bits where safe to do so
- Reviewed world-writable files and directories

### NTP Configuration
- Configured and secured NTP (Network Time Protocol)
- Accurate system time matters for log integrity
  and certificate validation

### Kernel Parameters
- Reviewed /etc/sysctl.conf hardening recommendations
- Applied several network hardening parameters

---

## What I Learned

- How professional security audits are structured
- Why each hardening recommendation exists —
  not just what to do but WHY
- How SUID binaries work and why they're a risk
- How SSH key authentication works vs passwords
- Why system time accuracy matters in security
- How rootkit detection baselines work
- The difference between fixing something and
  understanding why it needed fixing

---

## Lynis Hardening Index

Score improved after implementing recommendations.
Exact scores not recorded but measurable improvement
was confirmed on re-scan.

---

## Tools Used

| Tool | Purpose |
|---|---|
| Lynis | Full system security audit |
| rkhunter | Rootkit detection and baseline |
| ssh | Secure remote access configuration |
| sysctl | Kernel parameter tuning |
| chmod | File permission hardening |

---

## References

- Lynis official docs: https://cisofy.com/documentation/lynis/
- Arch Wiki SSH hardening: https://wiki.archlinux.org/title/OpenSSH
- Arch Wiki sysctl: https://wiki.archlinux.org/title/Sysctl
- rkhunter: https://rkhunter.sourceforge.net/

---

*Completed: Early 2026*
