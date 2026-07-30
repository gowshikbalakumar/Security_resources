# OWASP Top 10:2025 Progress Tracker

**Tags:** #OWASP #WebSecurity #BugBounty #Learning

---

# Progress Overview

| OWASP Category | Status | Topics Covered |
|---------------|:------:|----------------|
| 🟢 **A01 – Broken Access Control** | 🟡 Partial | CSRF ✅, IDOR ✅, Privilege Escalation ✅, SSRF ⏳, BOLA ⏳ |
| 🟢 **A02 – Security Misconfiguration** | ✅ Good | CORS, Host Header, Cookie Attributes |
| 🔴 **A03 – Software Supply Chain Failures** | ❌ Not Started | Dependency Security, CI/CD, Third-party Libraries |
| 🔴 **A04 – Cryptographic Failures** | ❌ Not Started | HTTPS, Encryption, Password Hashing |
| 🟢 **A05 – Injection** | ✅ Strong | SQLi, Blind SQLi, Command Injection, LFI, RFI, Directory Traversal |
| 🟡 **A06 – Insecure Design** | 🟡 Partial | Race Condition ✅, Business Logic Flaws ⏳ |
| 🟢 **A07 – Authentication Failures** | ✅ Strong | Sessions, JWT, Replay, Password Policy |
| 🔴 **A08 – Software or Data Integrity Failures** | ❌ Not Started | Code Signing, Integrity Verification |
| 🔴 **A09 – Security Logging & Alerting Failures** | ❌ Not Started | Logging, Monitoring, Alerts |
| 🔴 **A10 – Mishandling of Exceptional Conditions** | ❌ Not Started | Error Handling, Fail Securely |

---

# Vulnerability Mapping

| Vulnerability / Attack | OWASP 2025 | Status |
|-------------------------|------------|:------:|
| SQL Injection (SQLi) | A05 – Injection | ✅ |
| Blind SQL Injection | A05 – Injection | ✅ |
| OS Command Injection | A05 – Injection | ✅ |
| Local File Inclusion (LFI) | A05 – Injection | ✅ |
| Remote File Inclusion (RFI) | A05 – Injection | ✅ |
| Directory Traversal | A05 – Injection | ✅ |
| Host Header Injection | A02 – Security Misconfiguration | ✅ |
| CORS Misconfiguration | A02 – Security Misconfiguration | ✅ |
| Cookie Attribute Misconfiguration | A02 – Security Misconfiguration | ✅ |
| Weak Password Policy | A07 – Authentication Failures | ✅ |
| Session Hijacking | A07 – Authentication Failures | ✅ |
| Session Fixation | A07 – Authentication Failures | ✅ |
| Improper Session Implementation | A07 – Authentication Failures | ✅ |
| Replay Attack | A07 – Authentication Failures | ✅ |
| JWT Security | A07 – Authentication Failures | ✅ |
| JWT Weak Secret / Brute Force | A07 – Authentication Failures | ✅ |
| Race Condition | A06 – Insecure Design | ✅ |
| Cross-Site Request Forgery (CSRF) | A01 – Broken Access Control | ✅ |
| IDOR | A01 – Broken Access Control | ✅ |
| Privilege Escalation | A01 – Broken Access Control | ✅ |
| Broken Object Level Authorization (BOLA) | A01 – Broken Access Control | ⏳ |
| Server-Side Request Forgery (SSRF) | A01 – Broken Access Control | ⏳ |
| Cryptographic Failures | A04 – Cryptographic Failures | ⏳ |
| Business Logic Flaws | A06 – Insecure Design | ⏳ |
| Software Supply Chain Failures | A03 – Software Supply Chain Failures | ⏳ |
| Software/Data Integrity Failures | A08 – Software or Data Integrity Failures | ⏳ |
| Security Logging & Alerting Failures | A09 – Security Logging & Alerting Failures | ⏳ |
| Mishandling of Exceptional Conditions | A10 – Mishandling of Exceptional Conditions | ⏳ |

---

# Recommended Learning Order

1. ✅ HTTP Fundamentals
2. ✅ Authentication & Sessions
3. ✅ SQL Injection
4. ✅ Command Injection
5. ✅ LFI / RFI
6. ✅ Directory Traversal
7. ✅ Host Header Injection
8. ✅ CORS
9. ✅ Session Security
10. ✅ JWT
11. ✅ Race Condition
12. ✅ CSRF
13. ✅ IDOR
14. ✅ Privilege Escalation
15. ⏳ BOLA
16. ⏳ SSRF
17. ⏳ Cryptographic Failures
18. ⏳ Business Logic Flaws
19. ⏳ Software Supply Chain
20. ⏳ Software/Data Integrity
21. ⏳ Logging & Alerting
22. ⏳ Exceptional Conditions

---

# Overall Progress

```
A01 ████████░░ 80%
A02 ██████████ 100%
A03 ░░░░░░░░░░ 0%
A04 ░░░░░░░░░░ 0%
A05 ██████████ 100%
A06 █████░░░░░ 50%
A07 ██████████ 100%
A08 ░░░░░░░░░░ 0%
A09 ░░░░░░░░░░ 0%
A10 ░░░░░░░░░░ 0%
```

> [!TIP]
> You have completed most of the practical **Injection**, **Authentication**, and **Security Misconfiguration** topics. Next, focus on **BOLA**, **SSRF**, **Cryptographic Failures**, and **Business Logic Flaws** to cover the highest-impact bug bounty vulnerabilities.