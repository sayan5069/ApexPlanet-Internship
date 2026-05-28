# Task 1 — ApexPlanet Internship

## Objective

The objective of Task 1 was to build a foundational understanding of core cybersecurity concepts by exploring three key domains:

- **Linux** — navigating the filesystem, using essential commands, managing permissions, and writing basic shell scripts
- **Networking** — understanding the OSI/TCP-IP models, IP addressing, subnetting, protocols, and using network diagnostic tools
- **Cryptography** — learning symmetric/asymmetric encryption, hashing algorithms, digital signatures, and applying crypto tools in practice

The goal was to get hands-on experience with industry-standard tools used in penetration testing and security analysis, and document findings for future reference.

---

## Tools Used

| Tool | Category | Purpose |
|------|----------|---------|
| **Nmap** | Network Scanning | Port scanning, service/OS detection |
| **Burp Suite** | Web App Testing | Intercept HTTP traffic, test for web vulnerabilities |
| **Wireshark** | Packet Analysis | Capture and inspect network traffic |
| **Metasploit** | Exploitation | Exploit known vulnerabilities |
| **Hashcat** | Password Cracking | Crack password hashes using wordlists/brute-force |
| **John the Ripper** | Password Cracking | CPU-based hash cracking |
| **Gobuster** | Web Enumeration | Brute-force hidden directories and subdomains |
| **Nikto** | Web Scanning | Detect web server misconfigurations |
| **SQLMap** | SQL Injection | Automated SQL injection testing |
| **OpenSSL** | Cryptography | Encryption, key generation, certificate management |
| **GPG** | Encryption | File encryption and digital signing |
| **CyberChef** | Encoding/Crypto | Encoding, decoding, and data transformation |
| **tcpdump** | Packet Capture | CLI-based packet capture |
| **netcat (nc)** | Networking | TCP/UDP connections, port listening |
| **theHarvester** | OSINT | Email and subdomain harvesting |
| **Kali Linux** | OS / Lab | Penetration testing environment |

---

## Steps Performed

### Step 1 — Environment Setup
- Set up Kali Linux as the primary working environment (VM or native)
- Verified tool availability: Nmap, Burp Suite, Wireshark, Metasploit, Hashcat
- Configured browser proxy settings for Burp Suite (`127.0.0.1:8080`)

### Step 2 — Linux Fundamentals
- Explored the Linux filesystem hierarchy (`/etc`, `/var`, `/home`, `/proc`, etc.)
- Practiced essential commands: file navigation, permissions (`chmod`, `chown`), process management (`ps`, `kill`, `top`)
- Used `grep`, `find`, `awk`, and `sed` for text processing and file searching
- Wrote and executed basic shell scripts

### Step 3 — Network Scanning with Nmap
- Performed host discovery on the local network
- Ran service and version detection scans (`-sV`)
- Used aggressive scan mode (`-A`) to detect OS and run default scripts
- Identified open ports and running services on target machines

### Step 4 — Packet Analysis with Wireshark
- Captured live network traffic on the active interface
- Applied display filters to isolate HTTP, DNS, and TCP traffic
- Followed TCP streams to reconstruct HTTP conversations
- Analyzed packet headers and identified protocols in use

### Step 5 — Web Application Testing with Burp Suite
- Configured Burp Suite proxy and intercepted HTTP requests
- Used Repeater to manually modify and resend requests
- Explored Intruder for parameter fuzzing
- Used Decoder module to encode/decode Base64 and URL-encoded data

### Step 6 — Cryptography Practice
- Generated RSA key pairs using OpenSSL
- Encrypted and decrypted files using AES-256-CBC
- Computed MD5, SHA-1, and SHA-256 hashes of files
- Used CyberChef for Base64 encoding/decoding and hash analysis
- Explored GPG for file encryption and signing

### Step 7 — Password Cracking
- Used Hashcat to crack MD5 and SHA-256 hashes with the `rockyou.txt` wordlist
- Used John the Ripper on Linux shadow file hashes
- Understood the role of salting in preventing rainbow table attacks

### Step 8 — Documentation
- Created structured notes for Linux, Networking, and Cryptography
- Compiled a tools reference document
- Captured screenshots of key steps and outputs

---

## Screenshots

Screenshots are located in the `Screenshots/` folder. Below is a summary of what each captures:

| File | Description |
|------|-------------|
| `Screenshot 2026-05-28 203849.png` | Initial environment/tool setup |
| `Screenshot 2026-05-28 204044.png` | Linux command practice |
| `Screenshot 2026-05-28 204058.png` | Linux command practice |
| `Screenshot 2026-05-28 204243.png` | Linux filesystem exploration |
| `Screenshot 2026-05-28 204611.png` | Nmap scan output |
| `Screenshot 2026-05-28 204647.png` | Nmap service detection |
| `Screenshot 2026-05-28 204728.png` | Nmap scan results |
| `Screenshot 2026-05-28 210610.png` | Wireshark packet capture |
| `Screenshot 2026-05-28 223731.png` | Burp Suite proxy intercept |
| `Screenshot 2026-05-28 223819.png` | Burp Suite Repeater |
| `Screenshot 2026-05-28 223954.png` | Web application testing |
| `Screenshot 2026-05-28 224253.png` | Cryptography / OpenSSL |
| `Screenshot 2026-05-28 224540.png` | Hash generation |
| `Screenshot 2026-05-28 224547.png` | Hash cracking with Hashcat |
| `Screenshot 2026-05-28 224922.png` | Password cracking output |
| `Screenshot 2026-05-28 224934.png` | John the Ripper results |
| `Screenshot 2026-05-28 225029.png` | CyberChef encoding |
| `Screenshot 2026-05-28 225213.png` | GPG encryption |
| `Screenshot 2026-05-28 230227.png` | Network analysis |
| `Screenshot 2026-05-28 230342.png` | Final tool output |
| `Screenshot 2026-05-28 230353.png` | Summary / wrap-up |

> **Note**: Screenshot descriptions are approximate. Drag the images into the chat to get exact descriptions updated here.

---

## Results

### Linux
- Successfully navigated the Linux filesystem and executed core commands
- Managed file permissions and ownership using `chmod` and `chown`
- Wrote functional shell scripts with variables, conditionals, and loops
- Used `grep`, `find`, and `awk` for effective file and text searching

### Networking
- Identified open ports and running services on target hosts using Nmap
- Captured and analyzed live network traffic using Wireshark
- Understood TCP/IP communication by following packet streams
- Verified DNS resolution, ARP tables, and routing information

### Cryptography
- Generated RSA key pairs and performed file encryption/decryption with OpenSSL
- Computed and verified file hashes (MD5, SHA-256)
- Successfully cracked weak MD5 hashes using Hashcat with `rockyou.txt`
- Understood the importance of salting and strong hashing algorithms

### Web Application Testing
- Intercepted and modified HTTP requests using Burp Suite
- Identified how parameters are passed and how they can be manipulated
- Explored directory enumeration concepts with Gobuster

---

## Conclusion

Task 1 provided a solid foundation across three critical areas of cybersecurity:

**Linux** is the backbone of most security tools and server environments. Comfort with the command line, file permissions, and scripting is essential for any security professional.

**Networking** knowledge is fundamental to understanding how attacks and defenses work. Tools like Nmap and Wireshark make it possible to see exactly what's happening on a network at the packet level.

**Cryptography** underpins the security of nearly every modern system. Understanding the difference between hashing, symmetric, and asymmetric encryption — and knowing their weaknesses — is critical for both attacking and defending systems.

The hands-on use of tools like Burp Suite, Metasploit, Hashcat, and Wireshark bridged the gap between theory and real-world application. These skills form the foundation for more advanced tasks in penetration testing and security analysis.

---

## Folder Structure

```
Task1/
├── README.md
├── Notes/
│   ├── linux-notes.md
│   ├── networking-notes.md
│   ├── crypto-notes.md
│   └── tools-used.md
├── Report/
│   └── task1-report.md
├── Screenshots/
│   ├── Screenshot 2026-05-28 203849.png
│   ├── Screenshot 2026-05-28 204044.png
│   └── ... (21 screenshots total)
└── Videos/
```

---

*ApexPlanet Internship — Task 1 | Date: May 28, 2026*
