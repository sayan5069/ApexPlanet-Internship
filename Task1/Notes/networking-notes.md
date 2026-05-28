# Networking Notes — Task 1

## Overview
Networking is the practice of connecting computers and devices to share resources and communicate. Understanding networking fundamentals is essential for cybersecurity and system administration.

---

## OSI Model (7 Layers)

| Layer | Name         | Protocol Examples       | Function |
|-------|--------------|-------------------------|----------|
| 7     | Application  | HTTP, FTP, DNS, SMTP    | User-facing services |
| 6     | Presentation | SSL/TLS, JPEG, ASCII    | Data formatting/encryption |
| 5     | Session      | NetBIOS, RPC            | Session management |
| 4     | Transport    | TCP, UDP                | End-to-end communication |
| 3     | Network      | IP, ICMP, ARP           | Routing and addressing |
| 2     | Data Link    | Ethernet, MAC, PPP      | Node-to-node transfer |
| 1     | Physical     | Cables, Wi-Fi, Hubs     | Raw bit transmission |

> **Mnemonic**: "All People Seem To Need Data Processing"

---

## TCP/IP Model (4 Layers)

| Layer       | Equivalent OSI Layers | Protocols |
|-------------|----------------------|-----------|
| Application | 5, 6, 7              | HTTP, DNS, FTP, SMTP |
| Transport   | 4                    | TCP, UDP |
| Internet    | 3                    | IP, ICMP |
| Network Access | 1, 2              | Ethernet, ARP |

---

## IP Addressing

### IPv4
- 32-bit address, written as 4 octets: `192.168.1.1`
- Range: `0.0.0.0` to `255.255.255.255`

### IPv4 Classes
| Class | Range                     | Default Subnet Mask | Use |
|-------|---------------------------|---------------------|-----|
| A     | 1.0.0.0 – 126.255.255.255 | 255.0.0.0 (/8)      | Large networks |
| B     | 128.0.0.0 – 191.255.255.255 | 255.255.0.0 (/16) | Medium networks |
| C     | 192.0.0.0 – 223.255.255.255 | 255.255.255.0 (/24) | Small networks |

### Private IP Ranges (RFC 1918)
- `10.0.0.0/8`
- `172.16.0.0/12`
- `192.168.0.0/16`

### IPv6
- 128-bit address: `2001:0db8:85a3::8a2e:0370:7334`
- Solves IPv4 exhaustion

### Subnetting
- **CIDR notation**: `192.168.1.0/24` — `/24` means 24 bits for network
- **Subnet mask**: `/24` = `255.255.255.0` → 256 addresses, 254 usable
- **Formula**: Usable hosts = 2^(32 - prefix) - 2

---

## Key Protocols

### TCP vs UDP
| Feature     | TCP                        | UDP                     |
|-------------|----------------------------|-------------------------|
| Connection  | Connection-oriented        | Connectionless          |
| Reliability | Guaranteed delivery        | No guarantee            |
| Speed       | Slower (overhead)          | Faster                  |
| Use cases   | HTTP, FTP, SSH, Email      | DNS, VoIP, Streaming    |

### Common Ports
| Port | Protocol | Service |
|------|----------|---------|
| 20/21 | TCP | FTP |
| 22   | TCP | SSH |
| 23   | TCP | Telnet |
| 25   | TCP | SMTP |
| 53   | TCP/UDP | DNS |
| 80   | TCP | HTTP |
| 110  | TCP | POP3 |
| 143  | TCP | IMAP |
| 443  | TCP | HTTPS |
| 3306 | TCP | MySQL |
| 3389 | TCP | RDP |
| 8080 | TCP | HTTP Alternate |

---

## DNS (Domain Name System)
- Translates domain names → IP addresses
- **Record types**:
  - `A` — IPv4 address
  - `AAAA` — IPv6 address
  - `MX` — Mail server
  - `CNAME` — Alias/canonical name
  - `NS` — Name server
  - `TXT` — Text records (SPF, DKIM)
  - `PTR` — Reverse DNS lookup

```bash
nslookup google.com       # DNS lookup
dig google.com            # Detailed DNS query
host google.com           # Simple DNS lookup
```

---

## DHCP (Dynamic Host Configuration Protocol)
- Automatically assigns IP addresses to devices
- **DORA process**: Discover → Offer → Request → Acknowledge
- Port: UDP 67 (server), UDP 68 (client)

---

## Networking Tools

```bash
ping 8.8.8.8              # Test reachability (ICMP)
traceroute 8.8.8.8        # Trace packet path (Linux)
tracert 8.8.8.8           # Trace packet path (Windows)
nmap -sV 192.168.1.1      # Port scan with service detection
netstat -tulnp            # Active connections
ss -tulnp                 # Socket statistics
arp -a                    # ARP table
route -n                  # Routing table
ip route show             # Modern routing table
tcpdump -i eth0           # Packet capture
wireshark                 # GUI packet analyzer
```

---

## Firewalls & NAT

- **Firewall**: Filters traffic based on rules (iptables, ufw, pf)
- **NAT (Network Address Translation)**: Maps private IPs to a public IP
- **PAT (Port Address Translation)**: Multiple private IPs share one public IP using different ports

```bash
ufw status                # Check firewall status
ufw allow 22/tcp          # Allow SSH
ufw deny 23/tcp           # Block Telnet
iptables -L               # List iptables rules
```

---

## Key Concepts

- **MAC Address**: Hardware address, 48-bit, unique per NIC (e.g., `AA:BB:CC:DD:EE:FF`)
- **ARP**: Resolves IP → MAC address on local network
- **Default Gateway**: Router IP that forwards traffic outside local network
- **VLAN**: Virtual LAN — logically segments a network
- **VPN**: Encrypted tunnel over public network
- **Proxy**: Intermediary between client and server
- **Load Balancer**: Distributes traffic across multiple servers
