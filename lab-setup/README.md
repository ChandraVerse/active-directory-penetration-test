# 🧪 Lab Environment Setup

This guide walks you through building a fully functional Active Directory lab for penetration testing practice.

---

## Requirements

| Resource       | Minimum Specs                    |
|---------------|----------------------------------|
| RAM            | 16 GB (recommended 32 GB)       |
| Storage        | 100 GB free disk space           |
| Hypervisor     | VirtualBox 7.x or VMware 17.x   |
| Host OS        | Windows 10/11 or Linux           |

---

## Virtual Machines

### 1. Domain Controller (DC01)
- **OS:** Windows Server 2019/2022 (Evaluation ISO)
- **RAM:** 4 GB | **CPU:** 2 cores | **Disk:** 40 GB
- **Role:** Active Directory Domain Services (AD DS), DNS
- **Domain Name:** `corp.local` (or your choice)

### 2. Windows Workstation (WS01)
- **OS:** Windows 10/11 Enterprise Evaluation
- **RAM:** 2–4 GB | **CPU:** 2 cores | **Disk:** 40 GB
- **Role:** Domain-joined workstation (simulates user machine)

### 3. Kali Linux (Attacker)
- **OS:** Kali Linux 2024.x
- **RAM:** 4 GB | **CPU:** 2 cores | **Disk:** 40 GB
- **Role:** Attack platform

---

## Network Configuration

```
Network Adapter:  Host-Only (all VMs on same subnet)
Subnet:           192.168.56.0/24
DC01 IP:          192.168.56.10  (Static)
WS01 IP:          192.168.56.20  (Static or DHCP)
Kali IP:          192.168.56.5   (Static)
DNS:              192.168.56.10  (point to DC)
```

---

## Step-by-Step Setup

### Step 1: Install Windows Server & Promote to DC

```powershell
# Run in PowerShell as Administrator on Windows Server
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
Import-Module ADDSDeployment
Install-ADDSForest `
  -CreateDnsDelegation:$false `
  -DatabasePath "C:\Windows\NTDS" `
  -DomainMode "WinThreshold" `
  -DomainName "corp.local" `
  -DomainNetbiosName "CORP" `
  -ForestMode "WinThreshold" `
  -InstallDns:$true `
  -LogPath "C:\Windows\NTDS" `
  -NoRebootOnCompletion:$false `
  -SysvolPath "C:\Windows\SYSVOL" `
  -Force:$true
```

### Step 2: Create Vulnerable AD Users & Groups

```powershell
# Create Organizational Units
New-ADOrganizationalUnit -Name "Corp Users" -Path "DC=corp,DC=local"
New-ADOrganizationalUnit -Name "Corp Admins" -Path "DC=corp,DC=local"

# Create regular users
New-ADUser -Name "John Smith" -SamAccountName "jsmith" -AccountPassword (ConvertTo-SecureString "Password123!" -AsPlainText -Force) -Enabled $true -Path "OU=Corp Users,DC=corp,DC=local"
New-ADUser -Name "Jane Doe" -SamAccountName "jdoe" -AccountPassword (ConvertTo-SecureString "Summer2024!" -AsPlainText -Force) -Enabled $true -Path "OU=Corp Users,DC=corp,DC=local"

# Create service account (for Kerberoasting demo)
New-ADUser -Name "svc-sql" -SamAccountName "svc-sql" -AccountPassword (ConvertTo-SecureString "Sql@Service1" -AsPlainText -Force) -Enabled $true
Set-ADUser -Identity "svc-sql" -ServicePrincipalNames @{Add="MSSQLSvc/dc01.corp.local:1433"}

# Create Domain Admin
New-ADUser -Name "Admin User" -SamAccountName "adm-user" -AccountPassword (ConvertTo-SecureString "Admin@2024!" -AsPlainText -Force) -Enabled $true -Path "OU=Corp Admins,DC=corp,DC=local"
Add-ADGroupMember -Identity "Domain Admins" -Members "adm-user"
```

### Step 3: Join Workstation to Domain

```powershell
# Run on WS01 - set DNS to DC IP first
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.56.10

# Join domain
Add-Computer -DomainName "corp.local" -Credential (Get-Credential) -Restart
```

### Step 4: Configure Kali Linux

```bash
# Update Kali
sudo apt update && sudo apt upgrade -y

# Install essential tools
sudo apt install -y bloodhound neo4j impacket-scripts crackmapexec kerbrute evil-winrm
pip3 install impacket

# Set static IP
sudo nano /etc/network/interfaces
# Add:
# iface eth0 inet static
#   address 192.168.56.5
#   netmask 255.255.255.0
#   gateway 192.168.56.1
```

---

## ✅ Verification Checklist

- [ ] DC01 is accessible at `192.168.56.10`
- [ ] DNS resolves `corp.local` from all VMs
- [ ] WS01 is domain-joined (visible in AD Users & Computers)
- [ ] Kali can ping DC01 and WS01
- [ ] BloodHound/Neo4j starts without errors
- [ ] CrackMapExec can enumerate SMB shares on DC01

---

> ⚠️ This lab is for **educational purposes only**. Use only in isolated environments.
