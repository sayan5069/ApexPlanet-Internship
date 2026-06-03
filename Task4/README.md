# Task 4 — Penetration Testing
### ApexPlanet Internship | Timeline: Days 37+

---

## Objective

Task 4 covers a full penetration testing engagement against Metasploitable2 following professional methodology, plus additional topics in password attacks, phishing, malware analysis, and system hardening.

- **Pentest Methodology** — Recon → Scanning → Exploitation → Post-Exploitation → Reporting
- **Exploitation with Metasploit** — vsftpd backdoor, Samba RCE, reverse shell, sysinfo, hashdump
- **Password Attacks** — SSH brute force with Hydra, hash cracking with John the Ripper
- **Phishing Simulation** — Create phishing page, awareness training
- **Malware Basics** — Static vs dynamic analysis, sandbox analysis
- **System Hardening** — Patching, firewall rules, disabling unused services

---

## Tools Used

| Tool | Purpose |
|------|---------|
| **Nmap** | Reconnaissance and scanning |
| **Metasploit** | Exploitation and post-exploitation |
| **Meterpreter** | Advanced post-exploitation shell |
| **msfvenom** | Payload generation |
| **Hydra** | Online brute force (SSH, FTP, HTTP) |
| **John the Ripper** | Offline password hash cracking |
| **Hashcat** | GPU-accelerated hash cracking |
| **SET (Social Engineering Toolkit)** | Phishing simulation |
| **GoPhish** | Phishing campaign management |
| **Wireshark** | Network traffic analysis |
| **ClamAV** | Malware scanning |
| **strace / ltrace** | Dynamic malware analysis |
| **strings / xxd** | Static malware analysis |
| **iptables** | Firewall hardening |
| **Any.run / VirusTotal** | Sandbox analysis |

---

## Steps Performed

1. **Reconnaissance** — Host discovery, banner grabbing, service enumeration
2. **Scanning** — Full Nmap scan (TCP/UDP, version, OS, scripts)
3. **Exploitation** — vsftpd 2.3.4 backdoor via Metasploit (CVE-2011-2523)
4. **Exploitation** — Samba usermap_script RCE (CVE-2007-2447)
5. **Reverse Shell** — Established Meterpreter reverse shell
6. **Post-Exploitation** — sysinfo, hashdump, credential extraction
7. **SSH Brute Force** — Cracked SSH credentials with Hydra + rockyou.txt
8. **Hash Cracking** — Cracked /etc/shadow hashes with John the Ripper
9. **Phishing Page** — Created convincing phishing simulation with SET
10. **Awareness Training** — Documented phishing detection techniques
11. **Malware Analysis** — Static (strings, file, xxd) and dynamic (strace, sandbox)
12. **Sandbox Analysis** — Submitted sample to Any.run, extracted IOCs
13. **Patching** — Applied all available security updates
14. **Firewall** — Configured iptables to block port scans and malicious traffic
15. **Services** — Disabled vsftpd, telnet, Samba, IRC, r-services
16. **SSH Hardening** — Disabled password auth, root login, set idle timeout

---

## Key Findings

| Vulnerability | Severity | Result |
|--------------|----------|--------|
| vsftpd 2.3.4 Backdoor | Critical | Root shell in < 10 seconds |
| Samba RCE | Critical | Root shell via username injection |
| UnrealIRCd Backdoor | Critical | Root shell via IRC |
| MySQL No Root Password | Critical | Full database access |
| SSH Weak Credentials | High | SSH login with msfadmin:msfadmin |
| Telnet Cleartext | High | Credentials visible in traffic |

---

## Deliverables

- [x] Notes — Methodology, Metasploit, Password Attacks, Phishing, Malware, Hardening
- [x] Penetration Testing Report → `Report/task4-pentest-report.md`
- [x] README with full summary
- [ ] Screenshots → `Screenshots/` *(add during practical)*
- [ ] 10-min Demo Video → `Videos/` *(add after recording)*

---

## Folder Structure

```
Task4/
├── README.md
├── Notes/
│   ├── pentest-methodology-notes.md
│   ├── metasploit-exploitation-notes.md
│   ├── password-attacks-notes.md
│   ├── phishing-notes.md
│   ├── malware-basics-notes.md
│   └── system-hardening-notes.md
├── Report/
│   └── task4-pentest-report.md
├── Screenshots/
└── Videos/
```

---

*ApexPlanet Internship — Task 4 | Penetration Testing*
