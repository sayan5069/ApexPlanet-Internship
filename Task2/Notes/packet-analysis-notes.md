# Packet Analysis & Traffic Capture Notes — Task 2

## Overview
Packet analysis involves capturing and inspecting network traffic to understand communication patterns, detect anomalies, extract credentials from unencrypted protocols, and analyze attacks like SYN floods.

---

## 1. Wireshark — Full Reference

### Starting a Capture
1. Open Wireshark
2. Select network interface (eth0, wlan0, lo)
3. Click the blue shark fin to start
4. Click the red square to stop
5. Save as `.pcap` file

### Display Filters (Most Important)

**By Protocol:**
```
http
ftp
dns
tcp
udp
icmp
arp
ssl
ssh
smtp
```

**By IP:**
```
ip.addr == 192.168.1.10
ip.src == 192.168.1.10
ip.dst == 192.168.1.10
ip.addr == 192.168.1.0/24
```

**By Port:**
```
tcp.port == 80
tcp.port == 21
udp.port == 53
tcp.dstport == 443
tcp.srcport == 4444
```

**By TCP Flags:**
```
tcp.flags.syn == 1
tcp.flags.ack == 1
tcp.flags.rst == 1
tcp.flags.syn == 1 && tcp.flags.ack == 0    # SYN only (new connections)
tcp.flags == 0x002                           # SYN packets
```

**Combined Filters:**
```
http && ip.addr == 192.168.1.10
ftp && ip.src == 192.168.1.100
!(arp || dns || icmp)                        # Remove noise
http.request.method == "POST"
http contains "password"
ftp contains "PASS"
```

### Following Streams
- Right-click a packet → **Follow → TCP Stream**
- Reconstructs the full conversation in readable form
- Essential for extracting FTP credentials, HTTP data

### Key Wireshark Features
| Feature | Location | Use |
|---------|----------|-----|
| Follow TCP Stream | Right-click → Follow | See full conversation |
| Export Objects | File → Export Objects | Extract files from HTTP/FTP |
| Statistics → Protocol Hierarchy | Statistics menu | See traffic breakdown |
| Statistics → Conversations | Statistics menu | Top talkers |
| Statistics → IO Graph | Statistics menu | Traffic over time |
| Colorize Traffic | View → Coloring Rules | Visual protocol identification |

---

## 2. Capturing HTTP Traffic

HTTP is unencrypted — all data including credentials visible in plaintext.

```bash
# Capture HTTP with tcpdump
tcpdump -i eth0 port 80 -w http_capture.pcap

# Capture and display in terminal
tcpdump -i eth0 port 80 -A

# Filter for POST requests (often contain credentials)
tcpdump -i eth0 port 80 -A | grep -i "POST\|password\|user\|login"
```

**Wireshark filter for HTTP credentials:**
```
http.request.method == "POST"
http contains "username"
http contains "password"
```

**What to look for in HTTP:**
- GET/POST request parameters
- Cookie values (session tokens)
- Authorization headers (Basic Auth = Base64 encoded)
- Form data in POST body

**Decoding Basic Auth:**
```bash
echo "dXNlcjpwYXNzd29yZA==" | base64 -d
# Output: user:password
```

---

## 3. Capturing FTP Traffic

FTP transmits credentials in **plaintext** — username and password fully visible.

```bash
# Capture FTP traffic
tcpdump -i eth0 port 21 -w ftp_capture.pcap

# Display FTP traffic in terminal
tcpdump -i eth0 port 21 -A
```

**FTP Command Sequence:**
```
220 (vsFTPd 2.3.4)           ← Server banner
USER msfadmin                ← Username sent in cleartext
331 Please specify password
PASS msfadmin                ← Password sent in cleartext
230 Login successful
```

**Wireshark filter to extract FTP credentials:**
```
ftp
ftp.request.command == "USER"
ftp.request.command == "PASS"
```

**Steps to extract FTP credentials in Wireshark:**
1. Apply filter: `ftp`
2. Look for packets with `USER` and `PASS` commands
3. Check the Info column for the actual values
4. Or: Right-click → Follow TCP Stream to see full session

---

## 4. Capturing DNS Traffic

DNS queries reveal what domains a host is resolving — useful for understanding behavior.

```bash
# Capture DNS
tcpdump -i eth0 port 53 -w dns_capture.pcap

# Display DNS queries in terminal
tcpdump -i eth0 port 53 -n
```

**Wireshark DNS filters:**
```
dns
dns.qry.name contains "google"
dns.flags.response == 0        # DNS queries only
dns.flags.response == 1        # DNS responses only
dns.qry.type == 1              # A record queries
dns.qry.type == 28             # AAAA record queries
```

**What DNS traffic reveals:**
- Domains being visited
- C2 (Command & Control) communication patterns
- DNS tunneling (large TXT record responses)
- DNS poisoning attempts

---

## 5. SYN Flood Attack Analysis

### What is a SYN Flood?
A SYN flood is a **Denial of Service (DoS)** attack that exploits the TCP 3-way handshake:
1. Attacker sends thousands of SYN packets with spoofed source IPs
2. Server responds with SYN-ACK and waits for ACK
3. ACK never comes → server's connection table fills up
4. Legitimate connections are refused

### Simulating with hping3
```bash
# Basic SYN flood
sudo hping3 -S --flood -V -p 80 192.168.1.10

# SYN flood with random source IPs (spoofed)
sudo hping3 -S --flood --rand-source -p 80 192.168.1.10

# Controlled rate SYN flood
sudo hping3 -S -p 80 --faster 192.168.1.10

# SYN flood on specific port
sudo hping3 -S -p 443 --flood 192.168.1.10
```

**hping3 Flags:**
| Flag | Meaning |
|------|---------|
| `-S` | Set SYN flag |
| `--flood` | Send packets as fast as possible |
| `--rand-source` | Randomize source IP |
| `-V` | Verbose |
| `-p` | Destination port |
| `-c` | Packet count |
| `-i u1000` | Interval in microseconds |

### Identifying SYN Flood in Wireshark

**Filter:**
```
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

**Indicators of SYN Flood:**
- Massive number of SYN packets in short time
- Many different source IPs (if spoofed)
- All targeting same destination port
- No corresponding ACK packets completing handshake
- Server sending many SYN-ACK packets with no response

**Statistics → IO Graph** — shows spike in traffic volume

**Statistics → Conversations** — shows huge number of half-open connections

### Defending Against SYN Floods
```bash
# Enable SYN cookies (Linux)
echo 1 > /proc/sys/net/ipv4/tcp_syncookies

# Limit SYN rate with iptables
iptables -A INPUT -p tcp --syn -m limit --limit 1/s --limit-burst 3 -j ACCEPT
iptables -A INPUT -p tcp --syn -j DROP
```

---

## 6. tcpdump Quick Reference

```bash
tcpdump -i eth0                          # Capture on interface
tcpdump -i eth0 -c 100                   # Capture 100 packets
tcpdump -i eth0 -w file.pcap             # Write to file
tcpdump -r file.pcap                     # Read from file
tcpdump -i eth0 port 80                  # Filter by port
tcpdump -i eth0 host 192.168.1.10        # Filter by host
tcpdump -i eth0 src 192.168.1.10         # Filter by source
tcpdump -i eth0 dst 192.168.1.10         # Filter by destination
tcpdump -i eth0 -A                       # Print ASCII
tcpdump -i eth0 -X                       # Print hex + ASCII
tcpdump -i eth0 -n                       # No DNS resolution
tcpdump -i eth0 -v                       # Verbose
tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0'   # SYN packets only
```

---

## Key Takeaways
- HTTP, FTP, Telnet transmit data in **plaintext** — never use on production
- FTP credentials are trivially captured with Wireshark
- SYN floods are easy to simulate with hping3 and clearly visible in Wireshark
- Always filter traffic to reduce noise — use specific protocol/IP/port filters
- Follow TCP Stream is the fastest way to extract readable data from captures
- Save captures as `.pcap` for documentation and later analysis
