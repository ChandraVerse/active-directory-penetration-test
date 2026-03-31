# 🛡️ Active Directory Penetration Testing

> A comprehensive, hands-on lab guide and toolkit for performing Active Directory (AD) penetration testing — from reconnaissance to full domain compromise.

---

## 📌 Overview

Active Directory is the backbone of most enterprise networks. This repository covers the **end-to-end methodology** for legally testing and auditing AD environments, aligned with real-world red team operations and MITRE ATT&CK framework tactics.

**Target Audience:** Cybersecurity students, SOC Analysts, Ethical Hackers, and Penetration Testers.

---

## 🗂️ Repository Structure

```
active-directory-penetration-test/
├── 01-recon/               # Enumeration & reconnaissance techniques
├── 02-initial-access/      # Getting initial foothold (phishing, spray, etc.)
├── 03-enumeration/         # Post-compromise AD enumeration
├── 04-privilege-escalation/ # Local & domain privilege escalation
├── 05-lateral-movement/    # Pass-the-Hash, Pass-the-Ticket, etc.
├── 06-persistence/         # Backdoors, Golden Ticket, DCSync
├── 07-credential-dumping/  # LSASS, NTDS.dit, Kerberoasting
├── 08-defense-evasion/     # AV bypass, log clearing
├── 09-reporting/           # Report templates and documentation
├── tools/                  # Scripts and tool references
├── lab-setup/              # Lab environment setup guide
└── README.md
```

---

## 🧪 Lab Environment

| Component        | Details                          |
|-----------------|----------------------------------|
| Domain Controller | Windows Server 2019/2022        |
| Workstations     | Windows 10/11 (domain-joined)   |
| Attacker Machine | Kali Linux / Parrot OS           |
| Virtualization   | VirtualBox / VMware Workstation  |
| Network          | NAT / Host-Only Adapter          |

See [`lab-setup/`](./lab-setup/) for full configuration guide.

---

## ⚔️ Attack Phases Covered

1. **Reconnaissance** — OSINT, DNS, LDAP enumeration
2. **Initial Access** — Password spraying, AS-REP Roasting
3. **Internal Enumeration** — BloodHound, PowerView, ADRecon
4. **Privilege Escalation** — ACL abuse, Token impersonation
5. **Lateral Movement** — PtH, PtT, WMI, PSRemoting
6. **Persistence** — Golden Ticket, Silver Ticket, AdminSDHolder
7. **Credential Dumping** — Mimikatz, SecretsDump, Kerberoasting
8. **Defense Evasion** — AMSI bypass, OPSEC considerations

---

## 🔧 Key Tools Used

| Tool           | Purpose                             |
|---------------|--------------------------------------|
| BloodHound     | AD attack path visualization        |
| Impacket       | Remote protocol exploitation        |
| Mimikatz       | Credential extraction               |
| CrackMapExec   | SMB/AD lateral movement             |
| Rubeus         | Kerberos attacks                    |
| PowerView      | AD enumeration via PowerShell       |
| Kerbrute       | Kerberos username/password spray    |
| Nmap           | Network & port scanning             |

---

## 📋 MITRE ATT&CK Mapping

All techniques are mapped to the [MITRE ATT&CK Enterprise Matrix](https://attack.mitre.org/matrices/enterprise/). Each phase folder includes a `mitre-mapping.md` file.

---

## ⚠️ Legal Disclaimer

> **This repository is strictly for educational purposes and authorized security testing only.**
> Unauthorized access to computer systems is illegal. Always obtain explicit written permission before testing any environment. The author is not responsible for misuse.

---

## 👤 Author

**Chandra Sekhar Chakraborty**  
B.Tech CSE | Cybersecurity Enthusiast | SOC Analyst (Aspiring)  
📍 West Bengal, India  
🔗 [GitHub: ChandraVerse](https://github.com/ChandraVerse)

---

## 📄 License

MIT License — see [LICENSE](./LICENSE) for details.
