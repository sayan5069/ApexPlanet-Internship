# Task 5 — Capstone Project
## Vulnerability Assessment of Test Network
### ApexPlanet Internship | Final Submission

---

## Project Selection

**Chosen Project:** Vulnerability Assessment of Test Network

---

## Objective

Conduct a complete, professional-grade vulnerability assessment of an isolated test network (Metasploitable2), document all findings with evidence and mitigations, simulate an incident response scenario using auth logs, and deliver a comprehensive final report.

---

## Methodology

```
Phase 1 → Project Planning & Network Diagram
Phase 2 → Reconnaissance — Netdiscover, Nmap ping sweep
Phase 3 → Scanning — Full Nmap, Metasploit auxiliary, OpenVAS
Phase 4 → Vulnerability Assessment — CVE mapping, risk rating
Phase 5 → Incident Response — auth.log analysis, IR simulation
Phase 6 → Documentation & Professional Report
Phase 7 → Final Video & Submission
```

---

## Findings Summary

| # | Finding | Evidence | Risk | Mitigation |
|---|---------|----------|------|------------|
| 1 | Vulnerable FTP service (vsftpd 2.3.4) | Metasploit vsftpd search | 🔴 High | Disable outdated FTP |
| 2 | Samba-related vulnerabilities | Metasploit module search | 🔴 High | Update Samba packages |
| 3 | Host discovery successful | Netdiscover output | 🟠 Medium | Restrict unnecessary visibility |
| 4 | Authentication events in logs | auth.log | 🟠 Medium | Monitor logs regularly |
| 5 | Network connectivity confirmed | Ping results | 🔵 Low | Maintain segmentation |

---

## Incident Response Summary

**Detection:** Suspicious authentication events observed in auth.log — SSH brute force followed by successful login and su to root

**Containment:** Blocked attacker IP, terminated active sessions, locked compromised account

**Eradication:** Patched outdated software, removed unused services, audited for backdoors

**Recovery:** Restarted services, validated access controls, reset credentials

**Lessons Learned:** Continuous monitoring is necessary — deploy fail2ban, enforce key-based SSH, implement SIEM alerting

---

## Tools Used

| Tool | Purpose |
|------|---------|
| Netdiscover | Host discovery |
| Nmap | Port scanning, service detection |
| Metasploit | Vulnerability validation |
| OpenVAS | Automated vulnerability scanning |
| Wireshark | Traffic analysis |
| auth.log | Incident detection and response |
| iptables | Firewall hardening |
| fail2ban | Automated brute force protection |

---

## Deliverables

- [x] Project Plan & Network Diagram → `Notes/project-plan.md`
- [x] Incident Response Notes → `Notes/incident-response-notes.md`
- [x] Vulnerability Assessment Report → `Report/vulnerability-assessment-report.md`
- [x] Screenshots → `Screenshots/`
- [ ] 12-min Final Video → `Videos/`

---

## Folder Structure

```
Task5/
├── README.md
├── Notes/
│   ├── project-plan.md
│   └── incident-response-notes.md
├── Report/
│   └── vulnerability-assessment-report.md
├── Screenshots/
└── Videos/
```

---

## Final Submission

| Item | Status |
|------|--------|
| Capstone Report (Markdown) | ✅ Complete |
| GitHub Repository | ✅ Pushed |
| 12-min Video | ⏳ Pending |

---

*ApexPlanet Internship — Task 5 | Capstone Project*
*Vulnerability Assessment of Test Network*
