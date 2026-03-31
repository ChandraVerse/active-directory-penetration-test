# 📝 Phase 9: Reporting

A penetration test is only as valuable as its report. Clear, professional documentation is essential.

---

## Report Structure

1. **Executive Summary** — Non-technical overview for management
2. **Scope & Methodology** — What was tested, how, and when
3. **Attack Narrative** — Step-by-step story of the engagement
4. **Findings** — Categorized vulnerabilities with severity ratings
5. **Remediation Recommendations** — Actionable fixes
6. **Appendices** — Screenshots, tool outputs, raw data

---

## Finding Severity Ratings

| Severity | CVSS Score | Example |
|----------|-----------|--------|
| Critical | 9.0–10.0 | Domain compromise, DCSync |
| High | 7.0–8.9 | Kerberoastable DA service account |
| Medium | 4.0–6.9 | Password spraying success |
| Low | 0.1–3.9 | Null session enumeration |
| Informational | N/A | Outdated OS versions |

---

## Sample Finding Template

```markdown
### FINDING-001: Kerberoastable Service Accounts

**Severity:** High  
**CVSS Score:** 7.5  
**MITRE ATT&CK:** T1558.003  

**Description:**  
Multiple service accounts were found with SPNs set, allowing any domain user 
to request TGS tickets and crack them offline.

**Affected Accounts:**  
- svc-sql (MSSQLSvc/dc01.corp.local:1433)

**Evidence:**  
[Screenshot of GetUserSPNs output and cracked hash]

**Impact:**  
If cracked, attacker gains credentials for the service account, potentially 
leading to lateral movement or privilege escalation.

**Remediation:**  
1. Use strong, random passwords (25+ chars) for all service accounts  
2. Implement Managed Service Accounts (gMSA)  
3. Audit accounts with SPNs using: Get-DomainUser -SPN  
4. Enable AES encryption for Kerberos
```

---

## Useful References

- [PTES - Penetration Testing Execution Standard](http://www.pentest-standard.org/)
- [MITRE ATT&CK Enterprise Matrix](https://attack.mitre.org/matrices/enterprise/)
- [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
