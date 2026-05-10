# 🛡️ Hands-On Security Writeups

![Type](https://img.shields.io/badge/type-hands--on%20lab-blue)
![Writeups](https://img.shields.io/badge/writeups-5-brightgreen)
![Tools](https://img.shields.io/badge/tools-Nmap%20%7C%20Nessus%20%7C%20Wireshark%20%7C%20UFW-orange)
![Status](https://img.shields.io/badge/status-active-brightgreen)
![Focus](https://img.shields.io/badge/focus-blue%20team%20%2B%20recon-blueviolet)

> A collection of practical cybersecurity lab writeups covering network scanning, firewall hardening, phishing analysis, traffic analysis, and vulnerability assessment — all performed in real environments with documented evidence.

---

## Overview

This repository contains hands-on technical writeups from real lab exercises. Each writeup includes the objective, methodology, tools used, actual scan/capture files, screenshots, and security analysis — not just theory, but real execution with documented results.

---

## Writeups

| # | Topic | Tools | Key Finding |
|---|-------|-------|------------|
| 1 | [Network Scanning](#1-network-scanning) | Nmap, Wireshark | SMB + MySQL exposed across NAT and Bridged networks |
| 2 | [Firewall Hardening](#2-firewall-hardening) | UFW, Windows Defender | Telnet (port 23) blocked on Linux + Windows |
| 3 | [Phishing Analysis](#3-phishing-analysis) | MessageHeader, URLVoid, IPVoid | 2 phishing campaigns dissected — Wells Fargo + Microsoft |
| 4 | [Traffic Analysis](#4-traffic-analysis) | Wireshark, tcpdump | Unencrypted HTTP traffic and DNS recon patterns identified |
| 5 | [Vulnerability Assessment](#5-vulnerability-assessment) | Nessus Essentials | Critical Node.js CVEs (CVSS 9.8) on localhost + local machine |

---

## 1. Network Scanning

**Folder:** `Network-Scanning/`

### Objective
Perform network reconnaissance using TCP SYN scanning across NAT and Bridged environments to identify active hosts, open ports, and exposed services.

### Methodology
- Tool: Nmap 7.95
- Scan type: TCP SYN Scan (`-sS`)
- Networks: NAT `10.0.2.0/24` and Bridged `192.168.1.0/24`
- Traffic captured with Wireshark for analysis

### Key Findings

**NAT Network (10.0.2.0/24)**

| Host | Port | Service | Risk |
|------|------|---------|------|
| 10.0.2.2 | 135/tcp | MSRPC | Medium |
| 10.0.2.2 | 445/tcp | SMB | High |
| 10.0.2.2 | 3306/tcp | MySQL | High |
| 10.0.2.2 | 9080/tcp | Web Service | Medium |
| 10.0.2.3 | 53/tcp | DNS | Low |

**Bridged Network (192.168.1.0/24)**

| Host | Port | Service | Risk |
|------|------|---------|------|
| 192.168.1.1 | 53/tcp | DNS | Low |
| 192.168.1.1 | 80/tcp | HTTP | Medium |
| 192.168.1.1 | 443/tcp | HTTPS | Low |
| 192.168.1.9 | 135/tcp | MSRPC | Medium |
| 192.168.1.9 | 139/tcp | NetBIOS | High |
| 192.168.1.9 | 445/tcp | SMB | High |
| 192.168.1.9 | 3306/tcp | MySQL | High |

### Security Analysis
- **SMB (445)** — susceptible to lateral movement and exploitation (EternalBlue, WannaCry)
- **MySQL (3306)** — exposed database port risks unauthorized access if not properly secured
- **HTTP (80)** — unencrypted communication, vulnerable to interception and MITM
- **DNS (53)** — can be abused for reconnaissance and DNS-based data exfiltration

### Files Included
```
nat_scan.txt        → NAT network raw Nmap output
nat_scan.xml        → NAT network XML format
bridged_scan.txt    → Bridged network raw Nmap output
nmap_scan.pcapng    → Wireshark packet capture of scan traffic
```

### Tools Used
- Nmap 7.95
- Wireshark

---

## 2. Firewall Hardening

**Folder:** `Firewall-Hardening/`

### Objective
Enhance system security by restricting insecure network services and validating firewall configurations across Linux (Kali) and Windows environments.

### Methodology
- Blocked Telnet (port 23) on both platforms
- Allowed SSH (port 22) on Linux
- Validated rules by attempting Telnet connections

### Linux — UFW (Kali)

```bash
sudo ufw enable                    # Activate firewall
sudo ufw status numbered           # View active rules
sudo ufw deny 23                   # Block Telnet
sudo ufw allow 22                  # Allow SSH
```

**Validation result:**
```
telnet localhost 23
→ Connection failed: Connection refused  ✅
```

### Windows — Windows Defender Firewall
- Created new Inbound Rule → Block port 23 (Telnet)
- Rule name: "Block Telnet"
- Applied to all profiles (Public, Private, Domain)

**Validation result:**
```
C:\Users\kisha> telnet localhost 23
Connecting To localhost...Could not open connection to the host, 
on port 23: Connect failed  ✅
```

### Key Findings
- Telnet (port 23) successfully blocked in both environments
- SSH (port 22) remained accessible after hardening
- Firewall rules effectively prevented unauthorized plaintext access

### Security Analysis
- Telnet transmits all data including credentials in plaintext — blocking it is critical
- Replacing Telnet with SSH eliminates credential exposure risk
- Proper firewall configuration directly reduces the attack surface

### Files Included
```
firewall-ufw-commands    → UFW commands reference
Windows-firewall         → Windows Firewall configuration notes
ufw.png                  → UFW terminal output (rules active)
telnet-localhost23.png   → Telnet connection refused on Linux
block_telnet_.png        → Windows Defender block rule configured
windows-inbound.png      → Windows inbound rules list
```

### Tools Used
- UFW (Uncomplicated Firewall)
- Windows Defender Firewall
- Telnet (for validation testing)

---

## 3. Phishing Analysis

**Folder:** `Phishing-Analysis/`

### Objective
Analyze phishing emails by examining headers, sender details, embedded links, and social engineering techniques to identify malicious indicators.

### Methodology
- Collected two phishing email samples
- Extracted and analyzed email headers
- Investigated sender domains and reply-to addresses
- Traced origin IPs using MessageHeader analyzer
- Validated malicious domains and IPs using URLVoid and IPVoid

---

### Case 1 — Wells Fargo Phishing

**Email subject:** *"Secure your WellsFargo Online Key"*

**Header Analysis:**
```
From:        "Wells Fargo Alerts" <support@wellsfargo-support.net>
Return-Path: bounces@phishing-site.com
Received:    from shadyhosting.biz (91.234.56.78)
X-Mailer:    Outlook Express (fake)
Reply-To:    security@wellsfargo-secure.xyz
```

**Phishing Indicators:**

| Indicator | Detail |
|-----------|--------|
| Domain mismatch | `wellsfargo-support.net` — not official Wells Fargo domain |
| Malicious link | `http://cabinetkignima.com/Wellsfargo_keys_account5/page2.html` |
| Insecure link | HTTP instead of HTTPS |
| Origin server | `shadyhosting.biz` — IP confirmed blacklisted via IPVoid |
| Urgency tactic | "Your security key has expired" |
| Generic greeting | "Dear James" — scraped/random name |
| Grammar errors | Multiple structural mistakes |

---

### Case 2 — Microsoft Account Phishing

**Email subject:** *"Unusual sign-in activity"*

**Header Analysis:**
```
From:        "Microsoft Team" <no-reply.msteam2@outlook.com>
Return-Path: bounces@scam-server.ru
Received:    from mail.scamhost.net (185.143.223.17)
X-Mailer:    Some Phishing Kit v2.0
Reply-To:    support@microsoft-security.xyz
```

**Phishing Indicators:**

| Indicator | Detail |
|-----------|--------|
| Free email domain | `outlook.com` — not an official Microsoft domain |
| Phishing kit exposed | `X-Mailer: Some Phishing Kit v2.0` visible in header |
| Invalid IP in body | `293.09.101.9` — first octet exceeds valid IP range |
| Fake phone number | `1-800-816-0380` — not a verified Microsoft number |
| Urgency tactic | "Unusual sign-in from Russia" |
| Suspicious reply-to | `microsoft-security.xyz` |
| Hidden malicious CTA | "Review recent activity" button links to phishing page |

---

### Security Analysis
- Attackers use domain spoofing to impersonate trusted brands
- Social engineering (urgency + fear) drives victims to act without verifying
- Header analysis reveals true origin even when display name looks legitimate
- `X-Mailer` fields can expose phishing kit signatures

### Files Included
```
wellsfargo.txt           → Wells Fargo email body + header + indicators
Microsoft-phishing.txt   → Microsoft email body + header + indicators
wellsfargo.png           → MessageHeader analysis screenshot
windows-phishing.png     → Microsoft phishing header analysis screenshot
```

### Tools Used
- MessageHeader Analyzer
- URLVoid
- IPVoid

---

## 4. Traffic Analysis

**Folder:** `Traffic-Analysis/`

### Objective
Analyze captured network traffic using Wireshark to identify protocols, inspect packet flows, and detect potential suspicious or malicious activity.

### Methodology
- Captured live network traffic using Wireshark and tcpdump
- Analyzed PCAP files for protocol distribution
- Applied display filters to isolate specific traffic types
- Followed TCP streams to reconstruct full communication sessions

### Filters Applied
```
http    → HTTP request/response inspection
dns     → DNS query and response analysis
tcp     → Full TCP handshake and stream analysis
```

### Key Findings
- **TCP** — Multiple active connections identified with full handshake visibility
- **HTTP** — Unencrypted web traffic observed, exposing request content
- **DNS** — Domain resolution queries visible, including timing and response data

### Security Analysis
- Unencrypted HTTP traffic exposes sensitive data to any network observer
- DNS traffic reveals which domains a host is communicating with — useful for C2 detection
- TCP stream reconstruction enables full session replay for incident investigation
- Packet-level visibility is essential for anomaly detection and forensic analysis

### Files Included
```
traffic-cap      → tcpdump capture reference
traffic1.pcapng  → Wireshark packet capture file
```

### Tools Used
- Wireshark
- tcpdump

---

## 5. Vulnerability Assessment

**Folder:** `Vulnerability-Assessment/`

### Objective
Identify, analyze, and validate system vulnerabilities using Tenable Nessus by performing local and network-based scans on systems running outdated software.

### Methodology
- Tool: Nessus Essentials (CVSS v3.0 scoring)
- Scan type: Basic Network Scan
- Targets scanned:
  - **Localhost** — `127.0.0.1`
  - **Local Machine** — `192.168.1.10`

### Scan Results Summary

| Target | Total Vulns | Critical | High | Medium | Info |
|--------|------------|---------|------|--------|------|
| 127.0.0.1 | 61 | 4 | 2 | — | 75 |
| 192.168.1.10 | 59 | 1 | 3 | 1 | 1 |

### Critical Finding — Node.js Multiple Vulnerabilities

```
Plugin ID : 190856
Severity  : CRITICAL
CVSS Score: 9.8
Affected  : Node.js < 18.19.1 / < 20.11.1 / < 21.6.2
Installed : Node.js 20.11.0
```

**CVEs Identified:**

| CVE | Description | Impact |
|-----|-------------|--------|
| CVE-2024-21892 | Improper environment variable handling with elevated privileges | Privilege Escalation |
| CVE-2024-22019 | HTTP chunked encoding → unbounded memory read | DoS |
| CVE-2024-21896 | Buffer.from() path traversal via monkey-patching | Path Traversal |
| CVE-2024-22017 | setuid() bypass — privileged operations without dropping privileges | Privilege Escalation |
| CVE-2023-46809 | PKCS#1 timing side-channel in privateDecrypt() | RSA Key Recovery |
| CVE-2024-21891 | Wildcard path traversal via --allow-fs-read | File System Access |

### Remediation
```
Upgrade Node.js to:
  → 18.19.1 or later
  → 20.11.1 or later
  → 21.6.2 or later
```

### Security Analysis
- Critical vulnerabilities present on **both** targets — same outdated Node.js installation
- CVSS 9.8 means near-maximum exploitability with network access
- Privilege escalation CVEs are particularly dangerous on multi-user systems
- RSA timing attack (CVE-2023-46809) enables remote key recovery against API endpoints

### Files Included
```
Local_Machine_scan_gi4cv0.nessus  → Full Nessus scan export
local-machine                     → Local machine scan notes
localhost                         → Localhost scan notes
localhost1.png                    → Localhost scan overview
localhost2.png                    → Localhost critical finding detail
localmachine1.png                 → Local machine vulnerability list
localmachine2.png                 → Local machine critical CVE detail
```

### Tools Used
- Nessus Essentials
- CVSS v3.0 Scoring System

---

## Skills Demonstrated

| Skill | Writeup |
|-------|---------|
| Network reconnaissance | Network Scanning |
| Firewall rule management | Firewall Hardening |
| Email header forensics | Phishing Analysis |
| Social engineering identification | Phishing Analysis |
| Packet capture and analysis | Traffic Analysis |
| Vulnerability identification and CVE analysis | Vulnerability Assessment |
| Remediation guidance | Vulnerability Assessment |

---

## Tools Used Across All Writeups

- **Nmap** — network scanning and host discovery
- **Wireshark / tcpdump** — packet capture and traffic analysis
- **UFW** — Linux firewall management
- **Windows Defender Firewall** — Windows firewall management
- **Nessus Essentials** — vulnerability scanning and CVE identification
- **MessageHeader Analyzer** — email header tracing
- **URLVoid / IPVoid** — malicious domain and IP verification

---

## Author

**Kishan N**
Offensive Security Engineer | Blue Team Practitioner

Hands-on lab exercises covering core defensive and offensive security skills — from network recon to vulnerability assessment, all documented with real evidence.

---

*Security is learned by doing — not just reading.*
