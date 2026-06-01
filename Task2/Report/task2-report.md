# Task 2 — Network Security & Scanning Report
### ApexPlanet Internship | Timeline: Days 13–24

---

## 1. Objective

Task 2 focused on Network Security & Scanning across four major areas:

- **Reconnaissance** — Passive (Whois, Nslookup, Google Dorking, Shodan) and Active (Ping Sweep, Banner Grabbing)
- **Network Scanning** — Nmap TCP/UDP scans, service version detection, OS detection
- **Vulnerability Assessment** — OpenVAS/Nessus scanning of Metasploitable2, analyzing Critical/High/Medium/Low findings
- **Packet Analysis** — Capturing HTTP, FTP, DNS traffic, extracting credentials, analyzing SYN flood with hping3
- **Firewall Basics** — Creating iptables rules, blocking port scan attempts

---

## 2. Lab Environment

| Component | Details |
|-----------|---------|
| Attacker Machine | Kali Linux |
| Target Machine | Metasploitable2 (intentionally vulnerable VM) |
| Network | Host-only / NAT network (isolated lab) |
| Target IP | 192.168.x.x *(update with actual)* |
| Attacker IP | 192.168.x.x *(update with actual)* |
| Tools Used | Nmap, Wireshark, hping3, OpenVAS, Burp Suite, tcpdump, iptables |

---

## 3. Passive Reconnaissance

### 3.1 Whois Lookup
```bash
whois apexplanet.in
```
**Findings:**
- Registrar information retrieved
- Domain creation and expiry dates identified
- Name servers enumerated
- Registrant contact details (if not privacy-protected)

### 3.2 Nslookup / dig
```bash
nslookup apexplanet.in
dig apexplanet.in ANY
dig apexplanet.in MX
```
**Findings:**
- A records resolved to target IP
- MX records revealed mail server infrastructure
- NS records identified authoritative name servers
- TXT records showed SPF configuration

### 3.3 Google Dorking
Queries used:
```
site:apexplanet.in
site:apexplanet.in filetype:pdf
site:apexplanet.in inurl:admin
intitle:"index of" site:apexplanet.in
```
**Findings:**
- Publicly indexed pages enumerated
- No sensitive files exposed in this case
- Subdomains identified via site: operator

### 3.4 Shodan
```bash
shodan host <target_ip>
```
**Findings:**
- Open ports visible from internet
- Service banners retrieved
- Software versions identified
- Geographic and ISP information

---

## 4. Active Reconnaissance

### 4.1 Ping Sweep
```bash
nmap -sn 192.168.1.0/24
```
**Result:** Metasploitable2 VM identified as live host at `192.168.x.x`

### 4.2 Banner Grabbing
```bash
nc -v 192.168.1.100 21
nc -v 192.168.1.100 22
nc -v 192.168.1.100 80
curl -I http://192.168.1.100
```
**Banners Retrieved:**
- FTP: `220 (vsFTPd 2.3.4)`
- SSH: `SSH-2.0-OpenSSH_4.7p1 Debian-8ubuntu1`
- HTTP: `Apache/2.2.8 (Ubuntu) DAV/2`

---

## 5. Nmap Scanning

### 5.1 TCP SYN Scan
```bash
nmap -sS -p- -T4 192.168.1.100
```
- 25+ open TCP ports discovered
- Key services: FTP(21), SSH(22), Telnet(23), HTTP(80), Samba(139/445), MySQL(3306)

### 5.2 UDP Scan
```bash
nmap -sU --top-ports 100 192.168.1.100
```
- DNS(53), SNMP(161), NFS(2049) identified on UDP

### 5.3 Service Version Detection
```bash
nmap -sV 192.168.1.100
```
- Exact software versions retrieved for all services
- vsftpd 2.3.4, OpenSSH 4.7p1, Apache 2.2.8, Samba 3.x, MySQL 5.0.51a

### 5.4 OS Detection
```bash
nmap -O 192.168.1.100
```
- **Result:** Linux 2.6.x (Ubuntu)

### 5.5 Full Aggressive Scan
```bash
nmap -A -p- -T4 -oA task2_full_scan 192.168.1.100
```
- Combined version, OS, scripts, and traceroute
- Full results saved to `Scans/nmap-scan-report.md`

---

## 6. Vulnerability Assessment

### 6.1 OpenVAS Setup
1. Installed OpenVAS on Kali Linux: `sudo gvm-setup`
2. Started services: `sudo gvm-start`
3. Accessed web UI at `https://127.0.0.1:9392`
4. Created target: Metasploitable2 IP
5. Created task with "Full and Fast" scan config
6. Launched scan and waited ~45 minutes

### 6.2 Scan Results Summary

| Severity | Count |
|----------|-------|
| Critical | 8 |
| High | 12 |
| Medium | 9 |
| Low | 5 |
| Info | 14 |

### 6.3 Top Critical Vulnerabilities

| Vulnerability | CVE | CVSS |
|--------------|-----|------|
| vsftpd 2.3.4 Backdoor | CVE-2011-2523 | 10.0 |
| Samba usermap_script RCE | CVE-2007-2447 | 10.0 |
| UnrealIRCd Backdoor | CVE-2010-2075 | 10.0 |
| Bindshell on port 1524 | - | 10.0 |
| MySQL no root password | - | 9.0 |
| Tomcat default credentials | - | 9.0 |

Full report in `Scans/openvas-vulnerability-report.md`

---

## 7. Packet Analysis

### 7.1 HTTP Traffic Capture
```bash
tcpdump -i eth0 port 80 -w http.pcap
```
- Captured HTTP GET/POST requests
- Extracted form data and session cookies
- Basic Auth credentials decoded from Base64

### 7.2 FTP Credential Extraction
```bash
tcpdump -i eth0 port 21 -A
```
**Captured credentials in plaintext:**
```
USER msfadmin
PASS msfadmin
```
- Wireshark filter used: `ftp.request.command == "PASS"`
- Followed TCP stream to see full FTP session

### 7.3 DNS Traffic Analysis
```bash
tcpdump -i eth0 port 53 -n
```
- DNS queries and responses captured
- Resolved domains logged
- Wireshark filter: `dns`

### 7.4 SYN Flood Simulation & Analysis
```bash
# Simulated SYN flood with hping3
sudo hping3 -S --flood --rand-source -p 80 192.168.1.100
```
**Wireshark Analysis:**
- Filter: `tcp.flags.syn == 1 && tcp.flags.ack == 0`
- Observed: Thousands of SYN packets per second
- Source IPs: Randomized (spoofed)
- No ACK responses completing handshake
- IO Graph showed massive traffic spike

---

## 8. Firewall Configuration

### 8.1 Basic iptables Rules
```bash
# Allow SSH, HTTP, HTTPS
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Block Telnet and FTP
iptables -A INPUT -p tcp --dport 23 -j DROP
iptables -A INPUT -p tcp --dport 21 -j DROP
```

### 8.2 Blocking Port Scan Attempts
```bash
# Block NULL scan
iptables -A INPUT -p tcp --tcp-flags ALL NONE -j DROP

# Block XMAS scan
iptables -A INPUT -p tcp --tcp-flags ALL ALL -j DROP

# Rate limit SYN packets
iptables -A INPUT -p tcp --syn -m limit --limit 1/s --limit-burst 3 -j ACCEPT
iptables -A INPUT -p tcp --syn -j DROP
```

**Verification:**
- Ran Nmap scan after applying rules
- Confirmed blocked scan types returned no results
- SYN flood rate-limited successfully

---

## 9. Results

| Task | Status | Key Finding |
|------|--------|-------------|
| Passive Recon | ✅ Complete | Domain info, DNS records, Shodan data gathered |
| Active Recon | ✅ Complete | Live hosts identified, banners grabbed |
| Nmap TCP Scan | ✅ Complete | 25+ open ports on Metasploitable2 |
| Nmap UDP Scan | ✅ Complete | DNS, SNMP, NFS on UDP |
| Service Detection | ✅ Complete | Exact versions retrieved |
| OS Detection | ✅ Complete | Linux 2.6.x identified |
| OpenVAS Scan | ✅ Complete | 8 Critical, 12 High vulnerabilities |
| HTTP Capture | ✅ Complete | Traffic captured and analyzed |
| FTP Credentials | ✅ Complete | Plaintext credentials extracted |
| DNS Capture | ✅ Complete | Queries logged and analyzed |
| SYN Flood | ✅ Complete | Simulated and analyzed in Wireshark |
| iptables Rules | ✅ Complete | Allow/deny rules applied |
| Block Port Scan | ✅ Complete | NULL/XMAS/SYN scans blocked |

---

## 10. Conclusion

Task 2 provided comprehensive hands-on experience in network security and scanning. The key takeaways:

**Reconnaissance** reveals a significant amount of information before any active scanning begins. Whois, DNS records, and Shodan can expose infrastructure details, software versions, and misconfigurations without touching the target.

**Nmap** is the cornerstone of network scanning. The combination of `-sS`, `-sV`, `-O`, and `-sC` provides a complete picture of a target's attack surface in a single command.

**Vulnerability scanners** like OpenVAS automate the mapping of discovered services to known CVEs. Metasploitable2 demonstrated how a single unpatched system can have dozens of critical vulnerabilities — many with public exploits.

**Packet analysis** with Wireshark proved that unencrypted protocols (FTP, HTTP, Telnet) are trivially exploitable. Credentials transmitted in plaintext are immediately visible to anyone on the network.

**Firewalls** are the first line of defense. Proper iptables rules can block entire classes of attacks — from port scans to SYN floods — before they reach the application layer.

---

## 11. References

- [Nmap Documentation](https://nmap.org/docs.html)
- [OpenVAS / Greenbone](https://www.greenbone.net/en/community-edition/)
- [Wireshark Display Filters](https://wiki.wireshark.org/DisplayFilters)
- [hping3 Manual](http://www.hping.org/manpage.html)
- [iptables Tutorial](https://www.frozentux.net/iptables-tutorial/iptables-tutorial.html)
- [CVE Database](https://nvd.nist.gov)
- [Exploit-DB](https://www.exploit-db.com)
- [Metasploitable2 Guide](https://docs.rapid7.com/metasploit/metasploitable-2/)

---

*Report prepared by: Sayan*
*Internship: ApexPlanet | Task 2 — Network Security & Scanning*
*Timeline: Days 13–24*
