# Task 2 — Network Security & Scanning
### ApexPlanet Internship | Timeline: Days 13–24

---

## Objective

Task 2 covers end-to-end network security and scanning skills:

- **Passive Recon** — Whois, Nslookup, Google Dorking, Shodan
- **Active Recon** — Ping Sweep, Banner Grabbing
- **Nmap Scanning** — TCP SYN (`-sS`), UDP (`-sU`), Version (`-sV`), OS Detection (`-O`)
- **Vulnerability Assessment** — OpenVAS / Nessus Essentials on Metasploitable2
- **Packet Analysis** — HTTP, FTP, DNS traffic capture; SYN flood with hping3; Wireshark analysis
- **Firewall Basics** — iptables rules, blocking port scan attempts

---

## Tools Used

| Tool | Purpose |
|------|---------|
| **Nmap** | Port scanning, service/version/OS detection |
| **Wireshark** | Packet capture and traffic analysis |
| **hping3** | SYN flood simulation |
| **tcpdump** | CLI packet capture |
| **OpenVAS** | Vulnerability scanning |
| **Nessus Essentials** | Vulnerability scanning (alternative) |
| **Whois** | Domain registration lookup |
| **nslookup / dig** | DNS record enumeration |
| **Shodan** | Internet-connected device search |
| **netcat (nc)** | Banner grabbing |
| **iptables** | Firewall rule management |
| **Metasploitable2** | Intentionally vulnerable target VM |

---

## Steps Performed

1. **Passive Recon** — Whois, DNS lookups, Google Dorking, Shodan queries
2. **Active Recon** — Ping sweep to find live hosts, banner grabbing on open ports
3. **Nmap TCP SYN Scan** — Full port scan with `-sS -p-`
4. **Nmap UDP Scan** — Top UDP ports with `-sU`
5. **Service Version Detection** — `-sV` on all open ports
6. **OS Detection** — `-O` fingerprinting
7. **OpenVAS Setup** — Installed, configured, and ran full scan against Metasploitable2
8. **Vulnerability Analysis** — Categorized findings by Critical/High/Medium/Low
9. **HTTP Traffic Capture** — Captured and analyzed with Wireshark/tcpdump
10. **FTP Credential Extraction** — Filtered plaintext credentials from FTP traffic
11. **DNS Traffic Analysis** — Captured and analyzed DNS queries/responses
12. **SYN Flood Simulation** — Used hping3, analyzed in Wireshark
13. **iptables Rules** — Created allow/deny rules for specific ports
14. **Block Port Scans** — Applied rules to block NULL, XMAS, and SYN flood scans

---

## Key Findings

- Metasploitable2 has **8 Critical** vulnerabilities including backdoors in vsftpd, Samba, and UnrealIRCd
- FTP transmits credentials in **plaintext** — captured `msfadmin:msfadmin` directly from traffic
- SYN flood is clearly visible in Wireshark as a massive spike of SYN-only packets
- iptables rate limiting effectively mitigates SYN flood attacks
- Passive recon alone reveals significant infrastructure details without touching the target

---

## Deliverables

- [x] Nmap Scan Report → `Scans/nmap-scan-report.md`
- [x] OpenVAS Vulnerability Report → `Scans/openvas-vulnerability-report.md`
- [x] Full Task Report → `Report/task2-report.md`
- [x] Notes (Passive Recon, Active Recon/Nmap, Vuln Scanning, Packet Analysis, Firewall)
- [ ] Screenshots → `Screenshots/` *(add during practical)*
- [ ] 5-min Demo Video → `Videos/` *(add after recording)*

---

## Folder Structure

```
Task2/
├── README.md
├── Notes/
│   ├── passive-recon-notes.md
│   ├── active-recon-and-nmap-notes.md
│   ├── vulnerability-scanning-notes.md
│   ├── packet-analysis-notes.md
│   └── firewall-notes.md
├── Scans/
│   ├── nmap-scan-report.md
│   └── openvas-vulnerability-report.md
├── Report/
│   └── task2-report.md
├── Screenshots/
└── Videos/
```

---

*ApexPlanet Internship — Task 2 | Network Security & Scanning | Days 13–24*
