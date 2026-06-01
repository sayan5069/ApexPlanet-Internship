# Active Reconnaissance & Nmap Notes — Task 2

## Overview
Active recon involves **directly interacting** with the target system. Packets are sent to the target, which may trigger IDS/IPS alerts. Always ensure you have written authorization before performing active recon.

---

## 1. Ping Sweep

Identifies live hosts on a network by sending ICMP echo requests.

```bash
# Using ping (single host)
ping -c 4 192.168.1.1

# Ping sweep with nmap
nmap -sn 192.168.1.0/24

# Ping sweep with fping
fping -a -g 192.168.1.0/24 2>/dev/null

# Using for loop in bash
for ip in $(seq 1 254); do
    ping -c 1 -W 1 192.168.1.$ip &>/dev/null && echo "192.168.1.$ip is UP"
done
```

**Nmap Host Discovery Options:**
| Flag | Description |
|------|-------------|
| `-sn` | Ping scan (no port scan) |
| `-PE` | ICMP echo request |
| `-PS` | TCP SYN ping |
| `-PA` | TCP ACK ping |
| `-PU` | UDP ping |
| `-Pn` | Skip host discovery (assume up) |

---

## 2. Banner Grabbing

Retrieves service banners to identify software name and version running on open ports.

```bash
# Using netcat
nc -v 192.168.1.10 80
nc -v 192.168.1.10 21
nc -v 192.168.1.10 22

# Using telnet
telnet 192.168.1.10 80

# Using curl (HTTP banner)
curl -I http://192.168.1.10

# Using nmap scripts
nmap -sV --script=banner 192.168.1.10

# Using wget
wget -S http://192.168.1.10 2>&1 | head
```

**What Banners Reveal:**
- Web server type and version (Apache 2.4.49, nginx 1.18)
- FTP server (vsftpd 2.3.4, ProFTPD)
- SSH version (OpenSSH 7.4)
- SMTP server info
- Custom application headers

---

## 3. Nmap — Full Reference

### Basic Syntax
```bash
nmap [scan type] [options] [target]
```

### Scan Types

#### TCP SYN Scan (-sS) — Stealth Scan
```bash
nmap -sS 192.168.1.10
```
- Sends SYN packet, waits for SYN-ACK (open) or RST (closed)
- Never completes the 3-way handshake → "half-open" scan
- Faster and stealthier than full connect scan
- Requires root/admin privileges

#### TCP Connect Scan (-sT)
```bash
nmap -sT 192.168.1.10
```
- Completes full 3-way TCP handshake
- Slower, more detectable
- Does NOT require root privileges

#### UDP Scan (-sU)
```bash
nmap -sU 192.168.1.10
nmap -sU -p 53,67,68,161 192.168.1.10
```
- Scans UDP ports (DNS:53, DHCP:67/68, SNMP:161, etc.)
- Much slower than TCP scans
- Open port = no response or UDP response
- Closed port = ICMP port unreachable

#### Combined TCP + UDP
```bash
nmap -sS -sU 192.168.1.10
```

### Service & Version Detection (-sV)
```bash
nmap -sV 192.168.1.10
nmap -sV --version-intensity 9 192.168.1.10   # Max intensity
```
- Probes open ports to determine service/version
- Intensity 0-9 (default: 7)

### OS Detection (-O)
```bash
nmap -O 192.168.1.10
nmap -O --osscan-guess 192.168.1.10    # Aggressive guess
```
- Analyzes TCP/IP stack fingerprint to guess OS
- Requires at least one open and one closed port
- Requires root privileges

### Aggressive Scan (-A)
```bash
nmap -A 192.168.1.10
```
Combines: `-sV` + `-O` + `-sC` (default scripts) + traceroute

### Script Scanning (-sC / --script)
```bash
nmap -sC 192.168.1.10                          # Default scripts
nmap --script=vuln 192.168.1.10                # Vulnerability scripts
nmap --script=http-enum 192.168.1.10           # HTTP enumeration
nmap --script=ftp-anon 192.168.1.10            # FTP anonymous login
nmap --script=smb-vuln-ms17-010 192.168.1.10   # EternalBlue check
nmap --script=ssh-brute 192.168.1.10           # SSH brute force
```

### Port Specification
```bash
nmap -p 80 192.168.1.10              # Single port
nmap -p 80,443,8080 192.168.1.10     # Multiple ports
nmap -p 1-1000 192.168.1.10          # Port range
nmap -p- 192.168.1.10                # All 65535 ports
nmap --top-ports 100 192.168.1.10    # Top 100 common ports
```

### Output Formats
```bash
nmap -oN scan.txt 192.168.1.10       # Normal text output
nmap -oX scan.xml 192.168.1.10       # XML output
nmap -oG scan.gnmap 192.168.1.10     # Grepable output
nmap -oA scan 192.168.1.10           # All formats at once
```

### Timing Templates (-T)
| Template | Speed | Use Case |
|----------|-------|----------|
| `-T0` | Paranoid | IDS evasion |
| `-T1` | Sneaky | IDS evasion |
| `-T2` | Polite | Low bandwidth |
| `-T3` | Normal | Default |
| `-T4` | Aggressive | Fast networks |
| `-T5` | Insane | Very fast, may miss results |

### Full Scan Example (Metasploitable2)
```bash
nmap -sS -sV -O -sC -p- -T4 -oA metasploitable_scan 192.168.1.100
```

---

## 4. Nmap Scan Report Template

```
Target IP     : 192.168.x.x
Hostname      : metasploitable2
Scan Date     : YYYY-MM-DD
Scan Type     : SYN + UDP + Version + OS Detection
Command Used  : nmap -sS -sU -sV -O -A -p- -oA report 192.168.x.x

OPEN PORTS:
PORT     STATE  SERVICE    VERSION
21/tcp   open   ftp        vsftpd 2.3.4
22/tcp   open   ssh        OpenSSH 4.7p1
23/tcp   open   telnet     Linux telnetd
25/tcp   open   smtp       Postfix smtpd
80/tcp   open   http       Apache 2.2.8
139/tcp  open   netbios    Samba 3.x
445/tcp  open   smb        Samba 3.x
3306/tcp open   mysql      MySQL 5.0.51a
5432/tcp open   postgresql PostgreSQL 8.3
8180/tcp open   http       Apache Tomcat 5.5

OS DETECTION:
OS: Linux 2.6.x (Ubuntu)

NOTES:
- vsftpd 2.3.4 has a known backdoor (CVE-2011-2523)
- Samba is vulnerable to MS08-067 equivalent
- Multiple outdated services detected
```

---

## Key Takeaways
- `-sS` is the most common scan — fast and stealthy
- `-sU` is slow but essential for finding UDP services (DNS, SNMP)
- `-sV` reveals exact software versions → map to CVEs
- `-O` OS detection helps tailor exploits
- Always save output with `-oA` for reporting
- Metasploitable2 is intentionally vulnerable — ideal for practice
