# WireGuard Port Migration, Remote Access & DDNS
## camel (OptiPlex 3040) + eyeoftheneedle.dev

**Date:** April 25th, 2026

---

## Overview

This documents two related improvements to the home lab's remote access
infrastructure: a WireGuard port migration that resolved carrier-level
blocking on UDP 51820, and a transition from a router-hosted DDNS service
to a custom domain with a script-driven Cloudflare DDNS solution running
on camel.

The result is a remote access setup that reaches the full homelab —
including camel and the full home network — from anywhere on any
network, with automatic DNS updates when the home IP changes.

Related documentation:

- Router build and original WireGuard server setup: [router-hardening-tp-link.md](router-hardening-tp-link.md)
- Debian box (camel): [deb-box.md](deb-box.md)
- Full network topology: [network-topology.md](network-topology.md)

---

## Background

This document covers camel's WireGuard server — the production remote
access tunnel used to reach camel and the full home network from outside.
Peers: Lenovo ThinkPad E16 Gen 2, Samsung Galaxy S10 FE (tablet), and
Samsung Galaxy S23 Ultra.

---

## Problem — Carrier Blocking UDP 51820

camel's WireGuard server was originally running on UDP 51820, the WireGuard
default. The tunnel worked on local Wi-Fi and on other networks but failed
completely over mobile data. After confirming the port forward and server
config were correct, the diagnosis was carrier-level blocking — UDP 51820
is silently dropped.

---

## Fix — Port Migration to UDP 443

UDP 443 is the QUIC/HTTP3 port. Mobile carriers treat it as routine web
traffic and do not block it. Migrating WireGuard to UDP 443 resolved the
issue immediately and confirmed the diagnosis.

| Component | Before | After |
|---|---|---|
| camel WireGuard ListenPort | UDP 51820 | UDP 443 |
| TP-Link port forward | UDP 51820 → camel | UDP 443 → camel |
| Client endpoint | [LAN-IP]:51820 | camel.eyeoftheneedle.dev:443 |

---

## SSH Config Fix

The SSH client config on the ThinkPad previously pointed to camel's LAN IP
as the `HostName` for the `camel` host alias. That worked on the local
network but failed remotely.

The fix was to update `HostName` to camel's WireGuard peer IP. Since
`ssh camel` now routes through the tunnel regardless of whether
the connection originates from the LAN or remotely, the same command works
everywhere the tunnel is up. No manual switching between LAN and remote
SSH configs.

---

## Shell Aliases — Endpoint Switching

The BE700 does not support hairpin NAT on port 443 — connecting to the
public-facing endpoint from inside the LAN would loop through the router
and break. Two bash aliases in `~/.bashrc` on the ThinkPad handle this.

`wg-home` tears down any active WireGuard tunnel and reconnects using
camel's LAN IP as the endpoint. This is used when on the home network —
it routes tunnel traffic directly on the local segment rather than out
through the internet and back in.

`wg-away` tears down and reconnects using `camel.eyeoftheneedle.dev:443`
as the endpoint. This is used over mobile data or any external network.
The hostname resolves at connection time, so it always picks up the
current DNS record.

---

## Dynamic DNS — eyeoftheneedle.dev

### Problem

A home connection with a dynamic public IP means hard-coded IP addresses
in client configs eventually break without warning. The previous setup
used a router-hosted DDNS client pointed at a third-party free service —
it worked, but relied on a free service, used a hostname under a shared
domain, and gave no control over update timing or logic.

### Domain Registration

Registered `eyeoftheneedle.dev` through Cloudflare Registrar — two years,
WHOIS privacy enabled. DNS is managed entirely through Cloudflare.

### DNS Record

| Record | Type | Proxy |
|---|---|---|
| camel.eyeoftheneedle.dev | A → home IP | DNS only (no Cloudflare proxy) |

The Cloudflare proxy is intentionally disabled. WireGuard and SSH are not
HTTP traffic — routing them through Cloudflare's reverse proxy would break
the tunnel. DNS only means the record resolves directly to the home IP.

### DDNS Update Script

A bash script at `/usr/local/bin/ddns-update.sh` runs on camel every five
minutes via cron. It queries a public IP detection endpoint, compares the
result to the current value of the Cloudflare A record via the Cloudflare
API, and updates the record only if the IP has changed. The API token is
scoped to DNS edit permissions on the `eyeoftheneedle.dev` zone only —
no account-level access. No credentials are stored in this repository.

On a stable connection the script runs silently on most checks and fires
the API call only when the IP actually rotates.

---

## Current State

| Item | Status |
|---|---|
| WireGuard port | UDP 443 |
| Port forward (BE700) | UDP 443 → camel |
| DDNS domain | eyeoftheneedle.dev (Cloudflare, 2-year registration) |
| DNS record | camel.eyeoftheneedle.dev → home IP (A, DNS only) |
| DDNS update | /usr/local/bin/ddns-update.sh — cron every 5 min on camel |
| SSH alias | `ssh camel` routes via WireGuard IP — works anywhere tunnel is up |
| wg-home / wg-away | Endpoint switching aliases in ~/.bashrc on ThinkPad |
| Remote access | Confirmed from mobile data |

---

## Skills This Built

- WireGuard troubleshooting and port migration
- Carrier-level network blocking diagnosis
- Cloudflare DNS management and API token scoping
- Bash DDNS automation with API calls
- Remote access infrastructure design

---

## References

- Router build: [router-hardening-tp-link.md](router-hardening-tp-link.md)
- Debian box build: [deb-box.md](deb-box.md)
- Network topology: [network-topology.md](network-topology.md)
- WireGuard documentation: https://www.wireguard.com/quickstart/
- Cloudflare DNS API: https://developers.cloudflare.com/api/

---

*Last updated: April 28th, 2026*
