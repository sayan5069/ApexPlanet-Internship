# Project Plan — Task 5 Capstone
## Vulnerability Assessment of Test Network

---

## 1. Project Overview

**Project Title:** Vulnerability Assessment of Test Network
**Type:** Capstone Project — Option 2
**Intern:** Sayan
**Organization:** ApexPlanet Internship

---

## 2. Objectives

- Perform a complete vulnerability assessment of the lab network
- Identify, document, and categorize all vulnerabilities by severity
- Provide evidence-backed findings with tool outputs and screenshots
- Suggest actionable mitigation strategies for each finding
- Simulate an incident response scenario based on observed log activity
- Deliver a professional report suitable for a real-world client

---

## 3. Scope

| Item | Details |
|------|---------|
| Target Systems | Metasploitable2 VM, Kali Linux attacker VM |
| Network Range | 192.168.x.x/24 (isolated lab) |
| Test Type | Grey-box vulnerability assessment |
| In Scope | All services on target VM |
| Out of Scope | Production systems, external networks |
| Authorization | Internal lab — authorized testing |

---

## 4. Tools

| Tool | Purpose |
|------|---------|
| Nmap | Network discovery, port scanning, service detection |
| Netdiscover | Host discovery |
| Metasploit | Vulnerability validation |
| OpenVAS | Automated vulnerability scanning |
| Wireshark | Traffic analysis |
| Hydra | Authentication testing |
| John the Ripper | Password hash analysis |
| auth.log | Log-based incident detection |
| ELK Stack (reference) | SIEM concept |

---

## 5. Timeline

| Phase | Activity | Duration |
|-------|----------|----------|
| Phase 1 | Project Planning & Network Diagram | Day 1 |
| Phase 2 | Reconnaissance & Host Discovery | Day 2 |
| Phase 3 | Vulnerability Scanning & Enumeration | Day 3-4 |
| Phase 4 | Exploitation & Evidence Collection | Day 5-6 |
| Phase 5 | Incident Response Simulation | Day 7 |
| Phase 6 | Report Writing & Documentation | Day 8-9 |
| Phase 7 | Video Recording & Submission | Day 10 |

---

## 6. Network Diagram

```
+------------------+          +----------------------+
|   Kali Linux     |          |   Metasploitable2    |
|   (Attacker)     |          |   (Target)           |
|                  |          |                      |
|  192.168.x.5     +----------+  192.168.x.100       |
|                  |  LAN     |                      |
|  Tools:          |          |  Services:           |
|  - Nmap          |          |  - FTP  (21)         |
|  - Metasploit    |          |  - SSH  (22)         |
|  - Hydra         |          |  - HTTP (80)         |
|  - Wireshark     |          |  - SMB  (139/445)    |
|  - OpenVAS       |          |  - MySQL(3306)       |
+------------------+          +----------------------+
         |
         | Host-only / NAT Network
         |
+------------------+
|  VirtualBox /    |
|  VMware          |
|  Hypervisor      |
+------------------+
```

---

## 7. Methodology

```
Phase 1: Planning
  └── Define scope, tools, timeline, network diagram

Phase 2: Reconnaissance
  └── Netdiscover host discovery
  └── Nmap ping sweep
  └── Banner grabbing

Phase 3: Scanning & Enumeration
  └── Nmap full port scan (-sS -sV -O)
  └── OpenVAS automated scan
  └── Metasploit auxiliary scanners
  └── Service-specific enumeration

Phase 4: Vulnerability Assessment
  └── Map services to CVEs
  └── Validate with Metasploit search
  └── Document risk ratings
  └── Collect tool output evidence

Phase 5: Incident Response
  └── Analyze auth.log for suspicious events
  └── Detection → Containment → Eradication → Recovery

Phase 6: Reporting
  └── Executive Summary
  └── Findings with evidence
  └── Mitigations
  └── Post-Incident Report
```

---

## 8. Deliverables

- [x] Project Plan & Network Diagram
- [x] Vulnerability Assessment Report (Markdown)
- [x] Incident Response Report
- [ ] Professional PDF Report
- [ ] GitHub Repository
- [ ] 12-min Final Video
