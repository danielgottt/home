---
layout: post
title: "FortiJump to PEERSTITCH: One FortiManager, 500 Firewalls and trust you should hate"
subtitle: "A pointer to my latest Unit 42 writeup on an edge device intrusion, a memory-only FortiGate implant and certificate renewal as C2"
date: 2026-10-15
header_image: /assets/images/peerstitch.png
tags: ["CVE-2024-47575", "CVE-2025-61624", "CVE-2024-26009", "CVE-2024-26008", "Fortinet", "FortiManager", "FortiGate", "PEERSTITCH", "forensics", "vulnerability", "original_research"]
vendor: "Fortinet"
affected_product: "FortiManager, Fortinet, FortiGate"
stamp_label: "Never trust your edge devices"
toc: 
  - "The Short Version"
  - "The CVEs"
  - "The Fun Parts"
  - "Go Read It"
---

Quick one. I just published a recent intrusion on Unit 42: [Unit 42 Uncovers PEERSTITCH, a Stealthy FortiGate Implant](https://unit42.paloaltonetworks.com/fortimanager-cve-2024-47575-peerstitch-malware/).

## The Short Version

CL-STA-0954 (aka UNC5820), a patient crew whose tradecraft overlaps with suspected nation-state activity, hit a telecom organization's FortiManager and walked off with configs, credentials and trust material for 500+ FortiGates. Then they sat on it. Long dwell, legitimate credentials, everything dressed up as normal management traffic. When you hold the keys to 500 firewalls, there is no rush.

## The CVEs

- **CVE-2024-47575 (exploited):** FortiJump. Unauthenticated FortiManager access, the front door.
- **CVE-2025-61624 (exploited):** Path traversal in FortiOS auto-script names, turning super_admin into actual root.
- **CVE-2024-26009 (possible):** FGFM auth bypass, one way they may have impersonated the FortiManager.
- **CVE-2024-26008 (possible):** Unauthenticated FGFM resets, a likely fit for the telemetry drops.

## The Fun Parts

**PEERSTITCH:** A one-shot ARM64 injector that ptraces into `miglogd` and `iked`, plants hooks in RWX memory and wires the two daemons together through `/proc/<pid>/mem`. No file on disk, no listening port, no outbound connection.

**Apache mod_md persistence:** The attackers pointed the FortiGate's built-in ACME client at their own server, mapped the DNS-01 challenge to `/bin/node` and smuggled a reverse shell into the domain field. Set renewal to always, and Apache happily re-runs it forever. Certificate renewal as C2. Rude, but impressive.

**Anti-forensics:** Logging daemons killed, crashlogs wiped, config changes rolled back, the implant deleting itself, and a decoy `md-acme.conf` that looks perfectly boring. Anything they wrote down was either erased or faked.

## Go Read It

The full post has the technical breakdown, hunting guidance, IOCs and a YARA rule.

[Unit 42 Uncovers PEERSTITCH, a Stealthy FortiGate Implant](https://unit42.paloaltonetworks.com/fortimanager-cve-2024-47575-peerstitch-malware/)
