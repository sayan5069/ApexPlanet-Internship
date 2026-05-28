# Task 1 — Internship Report
### ApexPlanet Internship | Date: May 28, 2026

---

## 1. Objective

The goal of Task 1 was to gain hands-on familiarity with the foundational pillars of cybersecurity:

- **Linux** — command-line proficiency, filesystem navigation, permissions, and scripting
- **Networking** — protocols, IP addressing, packet analysis, and network scanning
- **Cryptography** — encryption algorithms, hashing, digital signatures, and crypto tools

This task involved setting up a lab environment, using industry-standard security tools, and documenting all findings.

---

## 2. Environment Setup

| Component | Details |
|-----------|---------|
| OS | Kali Linux |
| Virtualization | VirtualBox / VMware |
| Primary Tools | Nmap, Burp Suite, Wireshark, Metasploit, Hashcat |
| Network | Local lab network / loopback |

---

## 3. Linux

### 3.1 Filesystem Navigation
Explored the Linux directory hierarchy and understood the purpose of key directories (`/etc`, `/var`, `/home`, `/proc`, `/dev`).

### 3.2 Commands Practiced
- File operations: `ls`, `cd`, `cp`, `mv`, `rm`, `mkdir`, `touch`
- Permissions: `chmod`, `chown`, `umask`
- Process management: `ps aux`, `top`, `kill`, `htop`
- Searching: `grep`, `find`, `locate`, `which`
- Text processing: `awk`, `sed`, `cut`, `sort`, `uniq`

### 3.3 Shell Scripting
Wrote basic Bash scripts using variables, conditionals (`if/else`), loops (`for`, `while`), and functions.

### 3.4 Key Takeaways
- Linux permissions follow the `rwx` model for owner, group, and others
- Everything in Linux is a file — including devices and processes
- Shell scripting automates repetitive tasks efficiently

---

## 4. Networking

### 4.1 Concepts Covered
- OSI Model (7 layers) and TCP/IP Model (4 layers)
- IPv4 addressing, subnetting, and CIDR notation
- TCP vs UDP — connection-oriented vs connectionless
- Common protocols: HTTP, HTTPS, DNS, DHCP, FTP, SSH, SMTP

### 4.2 Network Scanning — Nmap
Used Nmap to discover hosts and enumerate open ports and services.

```
nmap -sV -A 192.168.1.0/24
```

**Findings:**
- Identified active hosts on the local network
- Detected open ports and associated services
- Retrieved OS fingerprinting information

### 4.3 Packet Analysis — Wireshark
Captured live traffic and analyzed packets at the protocol level.

- Filtered HTTP traffic and inspected request/response headers
- Followed TCP streams to reconstruct full conversations
- Identified DNS queries and responses in real time

### 4.4 Key Takeaways
- Nmap is essential for initial reconnaissance in any pentest
- Wireshark reveals exactly what data is transmitted over the network
- Unencrypted protocols (HTTP, Telnet, FTP) expose data in plaintext

---

## 5. Cryptography

### 5.1 Concepts Covered
- Symmetric encryption (AES, DES) — same key for encrypt/decrypt
- Asymmetric encryption (RSA, ECC) — public/private key pairs
- Hashing (MD5, SHA-1, SHA-256) — one-way, fixed-length output
- Digital signatures — authenticity and integrity verification
- PKI, SSL/TLS, and certificate chains

### 5.2 Practical Work

**Key Generation & Encryption (OpenSSL):**
```bash
openssl genrsa -out private.pem 2048
openssl enc -aes-256-cbc -in plaintext.txt -out encrypted.enc
```

**Hashing:**
```bash
echo -n "hello" | sha256sum
# Output: 2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824
```

**Password Cracking (Hashcat):**
```bash
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt
```
- Successfully cracked weak MD5 hashes using dictionary attack
- Demonstrated why MD5 is unsuitable for password storage

### 5.3 Key Takeaways
- MD5 and SHA-1 are broken — SHA-256 or stronger should be used
- Salting prevents rainbow table attacks on password hashes
- Asymmetric encryption solves the key distribution problem
- TLS 1.3 is the current standard for secure communication

---

## 6. Web Application Testing

### 6.1 Burp Suite
- Configured browser to route traffic through Burp proxy (`127.0.0.1:8080`)
- Intercepted and inspected HTTP GET and POST requests
- Used Repeater to manually modify parameters and observe responses
- Used Decoder to encode/decode Base64 and URL-encoded strings

### 6.2 Key Takeaways
- HTTP parameters can be manipulated if not properly validated server-side
- Burp Suite is indispensable for manual web application testing
- Always test both GET and POST parameters for injection points

---

## 7. Tools Summary

| Tool | Used For | Outcome |
|------|----------|---------|
| Nmap | Port scanning, service detection | Identified open ports and services |
| Wireshark | Packet capture and analysis | Analyzed live network traffic |
| Burp Suite | Web app proxy and testing | Intercepted and modified HTTP requests |
| Hashcat | Password hash cracking | Cracked MD5 hashes via dictionary attack |
| John the Ripper | Password cracking | Cracked Linux shadow file hashes |
| OpenSSL | Encryption, key gen, hashing | Generated keys, encrypted files |
| CyberChef | Encoding/decoding | Decoded Base64 and hex strings |
| GPG | File encryption and signing | Encrypted and signed files |
| Metasploit | Exploitation framework | Explored exploit/payload structure |
| Gobuster | Directory brute-forcing | Enumerated hidden web directories |

---

## 8. Challenges Faced

| Challenge | Resolution |
|-----------|-----------|
| Understanding CIDR subnetting | Practiced with subnet calculators and manual calculation |
| Wireshark filter syntax | Referred to official Wireshark display filter documentation |
| Hashcat GPU driver issues | Switched to CPU mode (`--force`) for lab purposes |
| Burp Suite SSL certificate | Installed Burp's CA certificate in browser to intercept HTTPS |

---

## 9. Results

- ✅ Gained command-line proficiency in Linux
- ✅ Successfully scanned and enumerated a network with Nmap
- ✅ Captured and analyzed network packets with Wireshark
- ✅ Performed encryption, decryption, and hashing with OpenSSL
- ✅ Cracked weak password hashes using Hashcat
- ✅ Intercepted and modified HTTP traffic with Burp Suite
- ✅ Documented all findings in structured notes

---

## 10. Conclusion

Task 1 established a strong foundation in the three core areas of cybersecurity. The hands-on approach — using real tools in a lab environment — made abstract concepts tangible and immediately applicable.

Linux proficiency is non-negotiable in security work; nearly every tool and server runs on it. Networking knowledge allows a security professional to understand the attack surface and trace malicious activity. Cryptography is the mechanism that protects data, and understanding its strengths and weaknesses is critical for both offense and defense.

The skills and tools explored in this task — Nmap, Wireshark, Burp Suite, Hashcat, and OpenSSL — are used daily by security professionals worldwide. This task has prepared a solid base for more advanced penetration testing and security analysis work in subsequent tasks.

---

## 11. References

- [Nmap Official Documentation](https://nmap.org/docs.html)
- [Wireshark User Guide](https://www.wireshark.org/docs/wsug_html_chunked/)
- [Burp Suite Documentation](https://portswigger.net/burp/documentation)
- [OpenSSL Manual](https://www.openssl.org/docs/man3.0/)
- [Hashcat Wiki](https://hashcat.net/wiki/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [TryHackMe](https://tryhackme.com)
- [HackTheBox](https://hackthebox.com)

---

*Report prepared by: Sayan*
*Internship: ApexPlanet | Task 1*
*Date: May 28, 2026*
