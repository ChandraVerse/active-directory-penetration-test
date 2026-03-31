# Active Directory Penetration Testing

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Platform](https://img.shields.io/badge/Platform-Windows%20Server%202019%2F2022-0078D4?logo=windows)
![ATT&CK](https://img.shields.io/badge/MITRE-ATT%26CK%20v14-red)
![Language](https://img.shields.io/badge/Python-3.10%2B-yellow?logo=python)
![Phases](https://img.shields.io/badge/Attack%20Phases-8-brightgreen)
![Team](https://img.shields.io/badge/Team-Red%20Team-red)

> A production-grade Active Directory penetration testing lab — covering the full red team kill chain from unauthenticated reconnaissance to domain compromise, persistence, and credential dumping. Fully mapped to MITRE ATT&CK v14 with a reproducible lab environment on `corp.local`.

---

## Attack Screenshots

> Real attack output from the lab environment — Kali Linux terminal, BloodHound CE, Impacket, and Mimikatz captured during a simulated red team campaign against `corp.local`. Full analysis in [`docs/attack-findings.md`](docs/attack-findings.md).

### 1 · Phase 1 — Nmap Recon Scan on DC01
![Phase 1 — Nmap Recon](docs/screenshots/screenshot1_nmap_recon.png)
*Nmap `-sV -sC` scan against DC01 (192.168.56.10). Key findings: LDAP port 389 open, Kerberos port 88 open, SMB message signing enabled but **NOT REQUIRED** (critical — enables SMB Relay attacks), OS identified as Windows Server 2019 Standard 17763, domain FQDN `DC01.corp.local` leaked without credentials. MITRE: T1046, T1595.*

### 2 · Phase 2 — AS-REP Roasting → Hash Capture & Crack
![Phase 2 — AS-REP Roasting](docs/screenshots/screenshot2_asrep_roasting.png)
*Impacket `GetNPUsers.py` targeting 50 domain accounts with zero credentials. Two accounts (`jdoe`, `svc-monitor`) had `UF_DONT_REQUIRE_PREAUTH` set — AS-REP hashes captured silently (no failed login events). Hashcat cracked both hashes against `rockyou.txt` in **27 seconds**: `jdoe → Summer2024!` and `svc-monitor → Monitor@2023`. MITRE: T1558.004.*

### 3 · Phase 3 — BloodHound Attack Path to Domain Admin
![Phase 3 — BloodHound Attack Path](docs/screenshots/screenshot3_bloodhound_path.png)
*BloodHound CE attack path graph after SharpHound `All` collection on `corp.local`. Shortest path from compromised `jdoe` to Domain Admin: `jdoe → HasSession → WS01$ → AdminTo → adm-user → MemberOf → Domain Admins → DCSync → DC01`. **4 hops, CRITICAL severity** — full domain compromise achievable from a single low-privilege account. MITRE: T1087.002, T1484.*

### 4 · Phase 7 — Mimikatz LSASS Dump (WDigest Plaintext)
![Phase 7 — Mimikatz LSASS Dump](docs/screenshots/screenshot4_mimikatz_dump.png)
*Mimikatz 2.2.0 `sekurlsa::logonpasswords` on DC01 after Pass-the-Hash lateral movement. WDigest caching enabled — **plaintext passwords stored in LSASS memory**. `adm-user → Admin@2024!` (NTLM: `e19ccf75...`) and `jsmith → Password123!` both extracted. 4 domain accounts compromised. NTLM hashes usable for PtH without cracking. MITRE: T1003.001.*

### 5 · Full Campaign — Attack Timeline (23 Minutes to Domain Compromise)
![Attack Timeline](docs/screenshots/screenshot5_attack_timeline.png)
*Severity-scored timeline of all 8 attack events during the `corp.local` red team campaign. Campaign progressed from a severity-2 Nmap scan at 07:22 UTC to dual severity-10 events (Golden Ticket + LSASS dump) at 07:41–07:45 UTC. Total time from zero credentials to full domain compromise: **23 minutes**.*

> 📄 **Full professional findings report with CVSS scores, MITRE mappings, and remediation:** [`docs/attack-findings.md`](docs/attack-findings.md)

---

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Attack Phases](#attack-phases)
- [MITRE ATT\&CK Coverage](#mitre-attck-coverage)
- [Tools Reference](#tools-reference)
- [Lab Setup](#lab-setup)
- [Project Structure](#project-structure)
- [Author](#author)

---

## Overview

This lab replicates a realistic enterprise Active Directory environment and walks through an **end-to-end penetration testing engagement** — the same methodology used by professional red teams and OSCP/CEH practitioners.

The engagement follows the **8-phase red team kill chain**:

1. **Reconnaissance** — Network scanning, LDAP/DNS enumeration, username harvesting
2. **Initial Access** — Password spraying, AS-REP Roasting, LLMNR poisoning, SMB Relay
3. **Post-Compromise Enumeration** — BloodHound, PowerView, LDAP domain dump
4. **Privilege Escalation** — Kerberoasting, ACL/ACE abuse, Token impersonation
5. **Lateral Movement** — Pass-the-Hash, Pass-the-Ticket, WMI, Evil-WinRM
6. **Persistence** — Golden Ticket, Silver Ticket, DCSync, AdminSDHolder
7. **Credential Dumping** — LSASS, SAM/LSA, NTDS.dit, Mimikatz
8. **Defense Evasion** — AMSI bypass, Event log clearing, LOLBins

### Key Highlights

| Metric | Value |
|--------|-------|
| Attack Phases | 8 |
| ATT&CK Techniques Covered | 25+ |
| Tools Demonstrated | 10+ |
| Lab Environment | Windows Server 2019 + Kali Linux |
| Domain | `corp.local` |
| MITRE ATT&CK Version | v14 |
| Findings Documented | 4 (2 Critical, 1 High, 1 Medium) |

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      ATTACKER MACHINE                           │
│             Kali Linux  ·  192.168.56.5                         │
│   Impacket · BloodHound · CrackMapExec · Mimikatz · Kerbrute    │
└──────────────────────────┬──────────────────────────────────────┘
                           │  Host-Only Network (192.168.56.0/24)
           ┌───────────────┴───────────────┐
           ▼                               ▼
┌─────────────────────┐         ┌─────────────────────┐
│   DOMAIN CONTROLLER │         │     WORKSTATION      │
│   DC01 · 192.168.56.10│       │   WS01 · 192.168.56.20│
│   Windows Server 2019│        │   Windows 10/11      │
│   AD DS · DNS · LDAP │        │   Domain-Joined       │
│   Kerberos · SMB     │        │   Simulated User      │
└─────────────────────┘         └─────────────────────┘
           │                               │
           └──────────── corp.local ───────┘
                    (AD Forest Root)
```

See [`lab-setup/`](lab-setup/) for full VM provisioning and network configuration guide.

---

## Attack Phases

Each phase has a dedicated folder with **step-by-step commands**, **tool usage**, and a **MITRE ATT&CK mapping table**.

| Phase | Folder | Key Techniques | ATT&CK IDs |
|-------|--------|----------------|------------|
| 1 · Reconnaissance | [`01-recon/`](01-recon/) | Nmap, LDAP enum, Kerbrute, SMB null session | T1595, T1046, T1087.002 |
| 2 · Initial Access | [`02-initial-access/`](02-initial-access/) | Password spray, AS-REP Roasting, Responder | T1110.003, T1558.004, T1557.001 |
| 3 · Enumeration | [`03-enumeration/`](03-enumeration/) | BloodHound, PowerView, LDAP dump | T1087.002, T1018, T1135 |
| 4 · Privilege Escalation | [`04-privilege-escalation/`](04-privilege-escalation/) | Kerberoasting, ACL abuse, Token impersonation | T1558.003, T1548, T1484 |
| 5 · Lateral Movement | [`05-lateral-movement/`](05-lateral-movement/) | Pass-the-Hash, Pass-the-Ticket, WMI, WinRM | T1550.002, T1550.003, T1021.006 |
| 6 · Persistence | [`06-persistence/`](06-persistence/) | Golden Ticket, Silver Ticket, DCSync | T1558.001, T1558.002, T1003.006 |
| 7 · Credential Dumping | [`07-credential-dumping/`](07-credential-dumping/) | LSASS, SAM, NTDS.dit, Mimikatz | T1003.001, T1003.002, T1003.003 |
| 8 · Defense Evasion | [`08-defense-evasion/`](08-defense-evasion/) | AMSI bypass, Log clearing, LOLBins | T1562.001, T1070.001, T1218 |

---

## MITRE ATT&CK Coverage

All techniques are mapped to the [MITRE ATT&CK Enterprise Matrix v14](https://attack.mitre.org/matrices/enterprise/). Each phase folder includes a `mitre-mapping.md` with per-technique detail.

| Tactic | Techniques Covered |
|--------|--------------------|
| Reconnaissance | T1595, T1046 |
| Initial Access | T1566, T1110.003 |
| Execution | T1047, T1059.001 |
| Persistence | T1053.005, T1547.001, T1558.001 |
| Privilege Escalation | T1548, T1134, T1484 |
| Defense Evasion | T1562.001, T1070.001, T1218, T1140 |
| Credential Access | T1003.001, T1003.002, T1003.003, T1558.003, T1558.004 |
| Discovery | T1087.002, T1018, T1135 |
| Lateral Movement | T1550.002, T1550.003, T1021.006, T1021.002 |
| Command & Control | T1557.001 |

---

## Tools Reference

All tools demonstrated in this lab with installation and key usage commands in [`tools/`](tools/).

| Tool | Purpose | Phase |
|------|---------|-------|
| [BloodHound](https://github.com/BloodHoundAD/BloodHound) | AD attack path visualization | Enumeration |
| [Impacket](https://github.com/fortra/impacket) | Remote protocol exploitation suite | All phases |
| [Mimikatz](https://github.com/gentilkiwi/mimikatz) | Credential extraction from memory | Credential Dumping |
| [CrackMapExec](https://github.com/mpgn/CrackMapExec) | SMB/AD lateral movement & spraying | Initial Access, Lateral Movement |
| [Rubeus](https://github.com/GhostPack/Rubeus) | Kerberos attack toolkit | Priv Esc, Persistence |
| [PowerView](https://github.com/PowerShellMafia/PowerSploit) | AD enumeration via PowerShell | Enumeration |
| [Kerbrute](https://github.com/ropnop/kerbrute) | Kerberos username/password spray | Recon, Initial Access |
| [Evil-WinRM](https://github.com/Hackplayers/evil-winrm) | WinRM shell with PtH support | Lateral Movement |
| [Responder](https://github.com/lgandx/Responder) | LLMNR/NBT-NS poisoning & capture | Initial Access |
| [Nmap](https://nmap.org) | Network & port scanning | Reconnaissance |

---

## Lab Setup

> Full provisioning guide in [`lab-setup/`](lab-setup/)

### Prerequisites

- **Host Machine**: 16 GB RAM minimum, 100 GB free disk
- **Hypervisor**: VirtualBox 7.x or VMware Workstation 17.x
- **Windows Server 2019** Evaluation ISO (free from Microsoft)
- **Windows 10/11** Enterprise Evaluation ISO
- **Kali Linux** 2024.x
- **Python**: 3.10+

### Quick Start

```bash
# Clone this repository
git clone https://github.com/ChandraVerse/active-directory-penetration-test.git
cd active-directory-penetration-test

# Review lab setup guide
cat lab-setup/README.md

# Install Kali dependencies
sudo apt install -y bloodhound neo4j impacket-scripts crackmapexec kerbrute evil-winrm
pip3 install impacket

# Provision AD users on DC01 (run on Windows Server)
# See lab-setup/README.md Step 2 for full PowerShell script
```

### Lab Network

| VM | OS | IP | Role |
|----|----|----|------|
| DC01 | Windows Server 2019 | 192.168.56.10 | Domain Controller, DNS |
| WS01 | Windows 10/11 | 192.168.56.20 | Domain-joined workstation |
| Kali | Kali Linux 2024.x | 192.168.56.5 | Attacker machine |

---

## Project Structure

```
active-directory-penetration-test/
├── 01-recon/
│   └── README.md           # Nmap, LDAP, DNS, SMB null session, Kerbrute
├── 02-initial-access/
│   └── README.md           # Password spray, AS-REP, LLMNR, SMB Relay
├── 03-enumeration/
│   └── README.md           # BloodHound, PowerView, ldapdomaindump
├── 04-privilege-escalation/
│   └── README.md           # Kerberoasting, ACL abuse, token impersonation
├── 05-lateral-movement/
│   └── README.md           # PtH, PtT, WMI, Evil-WinRM, PSRemoting
├── 06-persistence/
│   └── README.md           # Golden Ticket, Silver Ticket, DCSync
├── 07-credential-dumping/
│   └── README.md           # LSASS, SAM, NTDS.dit extraction
├── 08-defense-evasion/
│   └── README.md           # AMSI bypass, log clearing, LOLBins
├── 09-reporting/
│   └── README.md           # Report structure, severity ratings, finding template
├── docs/
│   ├── attack-findings.md  # 🔴 Professional pentest report (4 findings, CVSS, remediation)
│   └── screenshots/        # Attack evidence screenshots
│       ├── screenshot1_nmap_recon.png
│       ├── screenshot2_asrep_roasting.png
│       ├── screenshot3_bloodhound_path.png
│       ├── screenshot4_mimikatz_dump.png
│       └── screenshot5_attack_timeline.png
├── tools/
│   └── README.md           # All tool installation and quick-start commands
├── lab-setup/
│   └── README.md           # VM setup, network config, AD provisioning scripts
├── LICENSE
└── README.md
```

---

## Author

**Chandra Sekhar Chakraborty**
Cybersecurity Analyst | Red Team Enthusiast | SOC Analyst (Aspiring)
📍 West Bengal, India
🔗 [GitHub](https://github.com/ChandraVerse)

---

> *"Penetration testing is not about breaking things — it's about understanding how defenders think, finding the gaps before adversaries do, and making the enterprise resilient."*

---

> ⚠️ **Legal Disclaimer:** This repository is strictly for **educational purposes and authorized security testing only.** Unauthorized access to computer systems is illegal under the IT Act, 2000 (India) and equivalent laws globally. Always obtain explicit written permission before testing any environment. The author bears no responsibility for misuse.
