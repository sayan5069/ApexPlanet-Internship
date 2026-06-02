# Task 3 — Web Application Security
### ApexPlanet Internship | Timeline: Days 25–36

---

## Objective

Task 3 covers web application security testing against DVWA (Damn Vulnerable Web Application):

- **SQL Injection** — Extract credentials; demonstrate prepared statement prevention
- **XSS (Stored & Reflected)** — Inject scripts; mitigate with encoding and CSP
- **CSRF** — Forge password change requests; protect with CSRF tokens
- **File Inclusion (LFI/RFI)** — Read sensitive files; execute remote code
- **Burp Suite Advanced** — Intercept login requests; fuzz with Intruder
- **Web Security Headers** — Analyze and fix missing HTTP security headers

---

## Tools Used

| Tool | Purpose |
|------|---------|
| **DVWA** | Intentionally vulnerable web app target |
| **Burp Suite** | HTTP proxy, request interception, fuzzing |
| **SQLMap** | Automated SQL injection testing |
| **Firefox + FoxyProxy** | Browser with proxy switching |
| **Apache2** | Web server for DVWA and header configuration |
| **MariaDB / MySQL** | Backend database |
| **securityheaders.com** | Analyze HTTP security headers |
| **Python HTTP Server** | Host CSRF attack page and RFI shell |
| **curl** | Verify headers and manual HTTP requests |

---

## Steps Performed

1. Installed and configured DVWA on Kali Linux
2. Performed Union-based SQL Injection to extract all usernames and passwords
3. Cracked extracted MD5 password hashes
4. Demonstrated prevention using prepared statements (PDO)
5. Performed Stored XSS — injected persistent script in guestbook
6. Performed Reflected XSS — crafted malicious URL with script payload
7. Implemented mitigation — `htmlspecialchars()` + Content Security Policy
8. Created CSRF attack page that silently changes admin password
9. Demonstrated token-based CSRF protection (DVWA High level)
10. Exploited LFI to read `/etc/passwd` and other sensitive files
11. Exploited RFI to execute remote code via hosted PHP shell
12. Applied file inclusion mitigations (whitelist, `allow_url_include = Off`)
13. Intercepted and modified login requests with Burp Suite Proxy
14. Used Burp Intruder to brute-force DVWA credentials
15. Analyzed DVWA security headers — initial grade F
16. Added all security headers in Apache config — final grade A

---

## Key Findings

| Vulnerability | Impact | Status |
|--------------|--------|--------|
| SQL Injection | All user credentials extracted | Mitigated with prepared statements |
| Stored XSS | Session hijacking for all visitors | Mitigated with encoding + CSP |
| Reflected XSS | Session theft via crafted link | Mitigated with output encoding |
| CSRF | Admin password changed silently | Protected with CSRF tokens |
| LFI | Read /etc/passwd, config files | Mitigated with whitelist |
| RFI | Remote code execution | Mitigated with allow_url_include=Off |
| Missing Headers | Grade F on securityheaders.com | Fixed — Grade A after headers added |

---

## Deliverables

- [x] Notes — SQLi, XSS, CSRF, File Inclusion, Burp Suite, Security Headers
- [x] Security Testing Report → `Report/task3-report.md`
- [x] README with full summary
- [ ] Screenshots → `Screenshots/` *(add during practical)*
- [ ] 8-min Demo Video → `Videos/` *(add after recording)*

---

## Folder Structure

```
Task3/
├── README.md
├── Notes/
│   ├── dvwa-setup-notes.md
│   ├── sql-injection-notes.md
│   ├── xss-notes.md
│   ├── csrf-notes.md
│   ├── file-inclusion-notes.md
│   ├── burpsuite-advanced-notes.md
│   └── web-security-headers-notes.md
├── Report/
│   └── task3-report.md
├── Screenshots/
└── Videos/
```

---

*ApexPlanet Internship — Task 3 | Web Application Security | Days 25–36*
