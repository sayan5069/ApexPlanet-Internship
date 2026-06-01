# Firewall & iptables Notes — Task 2

## Overview
A firewall controls incoming and outgoing network traffic based on predefined rules. `iptables` is the standard Linux firewall tool that interacts directly with the kernel's netfilter framework.

---

## 1. iptables Basics

### How iptables Works
- Traffic passes through **chains** of rules
- Each rule has a **match** (criteria) and a **target** (action)
- Rules are evaluated top-to-bottom — first match wins

### Tables
| Table | Purpose |
|-------|---------|
| `filter` | Default — packet filtering (INPUT, OUTPUT, FORWARD) |
| `nat` | Network Address Translation |
| `mangle` | Packet modification |
| `raw` | Connection tracking bypass |

### Chains (in filter table)
| Chain | Traffic |
|-------|---------|
| `INPUT` | Incoming to this machine |
| `OUTPUT` | Outgoing from this machine |
| `FORWARD` | Passing through this machine (routing) |

### Targets (Actions)
| Target | Action |
|--------|--------|
| `ACCEPT` | Allow the packet |
| `DROP` | Silently discard |
| `REJECT` | Discard and send error back |
| `LOG` | Log the packet |

---

## 2. Basic iptables Commands

### View Rules
```bash
sudo iptables -L                    # List all rules
sudo iptables -L -v                 # Verbose (with packet counts)
sudo iptables -L -n                 # Numeric (no DNS resolution)
sudo iptables -L -n -v --line-numbers  # With line numbers
sudo iptables -L INPUT -n -v        # Specific chain
```

### Flush / Reset Rules
```bash
sudo iptables -F                    # Flush all rules
sudo iptables -F INPUT              # Flush INPUT chain only
sudo iptables -X                    # Delete custom chains
sudo iptables -Z                    # Zero packet/byte counters
```

### Set Default Policies
```bash
sudo iptables -P INPUT DROP         # Drop all incoming by default
sudo iptables -P OUTPUT ACCEPT      # Allow all outgoing by default
sudo iptables -P FORWARD DROP       # Drop all forwarded by default
```

---

## 3. Allow / Deny Specific Ports

### Allow Specific Ports
```bash
# Allow SSH (port 22)
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow HTTP (port 80)
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# Allow HTTPS (port 443)
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow DNS (port 53)
sudo iptables -A INPUT -p udp --dport 53 -j ACCEPT

# Allow from specific IP only
sudo iptables -A INPUT -p tcp --dport 22 -s 192.168.1.5 -j ACCEPT

# Allow established/related connections (important!)
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow loopback interface
sudo iptables -A INPUT -i lo -j ACCEPT
```

### Deny / Block Specific Ports
```bash
# Block Telnet (port 23)
sudo iptables -A INPUT -p tcp --dport 23 -j DROP

# Block FTP (port 21)
sudo iptables -A INPUT -p tcp --dport 21 -j DROP

# Block from specific IP
sudo iptables -A INPUT -s 192.168.1.50 -j DROP

# Block entire subnet
sudo iptables -A INPUT -s 10.0.0.0/8 -j DROP

# Block outgoing to specific IP
sudo iptables -A OUTPUT -d 192.168.1.50 -j DROP

# Reject with ICMP error (instead of silent drop)
sudo iptables -A INPUT -p tcp --dport 23 -j REJECT --reject-with tcp-reset
```

### Delete a Rule
```bash
# By rule number
sudo iptables -D INPUT 3

# By matching the rule
sudo iptables -D INPUT -p tcp --dport 23 -j DROP
```

---

## 4. Blocking Port Scan Attempts

### Detect and Block Nmap SYN Scans
```bash
# Limit new TCP connections (rate limiting)
sudo iptables -A INPUT -p tcp --syn -m limit --limit 1/s --limit-burst 3 -j ACCEPT
sudo iptables -A INPUT -p tcp --syn -j DROP

# Log and drop port scans
sudo iptables -A INPUT -p tcp --tcp-flags ALL NONE -j LOG --log-prefix "NULL_SCAN: "
sudo iptables -A INPUT -p tcp --tcp-flags ALL NONE -j DROP

sudo iptables -A INPUT -p tcp --tcp-flags ALL ALL -j LOG --log-prefix "XMAS_SCAN: "
sudo iptables -A INPUT -p tcp --tcp-flags ALL ALL -j DROP

sudo iptables -A INPUT -p tcp --tcp-flags ALL FIN,URG,PSH -j LOG --log-prefix "XMAS_SCAN: "
sudo iptables -A INPUT -p tcp --tcp-flags ALL FIN,URG,PSH -j DROP

sudo iptables -A INPUT -p tcp --tcp-flags SYN,RST SYN,RST -j DROP
sudo iptables -A INPUT -p tcp --tcp-flags SYN,FIN SYN,FIN -j DROP
```

### Block Nmap OS Detection Probes
```bash
# Block invalid TCP flag combinations used by Nmap
sudo iptables -A INPUT -p tcp --tcp-flags ALL FIN -j DROP
sudo iptables -A INPUT -p tcp --tcp-flags SYN,FIN SYN,FIN -j DROP
sudo iptables -A INPUT -p tcp --tcp-flags SYN,RST SYN,RST -j DROP
```

### Block SYN Flood
```bash
# SYN flood protection
sudo iptables -A INPUT -p tcp --syn -m limit --limit 1/s -j ACCEPT
sudo iptables -A INPUT -p tcp --syn -j DROP

# Or using hashlimit for per-IP rate limiting
sudo iptables -A INPUT -p tcp --syn -m hashlimit \
  --hashlimit-name synflood \
  --hashlimit-above 200/sec \
  --hashlimit-burst 3 \
  --hashlimit-mode srcip \
  -j DROP
```

---

## 5. Complete Firewall Setup Example

```bash
#!/bin/bash
# Basic secure firewall setup

# Flush existing rules
iptables -F
iptables -X

# Default policies — drop everything
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Allow loopback
iptables -A INPUT -i lo -j ACCEPT

# Allow established/related connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow SSH from specific subnet only
iptables -A INPUT -p tcp --dport 22 -s 192.168.1.0/24 -j ACCEPT

# Allow HTTP and HTTPS
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Block Telnet
iptables -A INPUT -p tcp --dport 23 -j DROP

# Block FTP
iptables -A INPUT -p tcp --dport 21 -j DROP

# SYN flood protection
iptables -A INPUT -p tcp --syn -m limit --limit 1/s --limit-burst 3 -j ACCEPT
iptables -A INPUT -p tcp --syn -j DROP

# Log dropped packets
iptables -A INPUT -j LOG --log-prefix "DROPPED: " --log-level 4

echo "Firewall rules applied."
```

---

## 6. Save & Restore iptables Rules

```bash
# Save rules (Debian/Ubuntu)
sudo iptables-save > /etc/iptables/rules.v4

# Restore rules
sudo iptables-restore < /etc/iptables/rules.v4

# Install persistence package
sudo apt install iptables-persistent -y
```

---

## 7. ufw — Simplified Firewall (Ubuntu)

```bash
sudo ufw enable                     # Enable firewall
sudo ufw disable                    # Disable firewall
sudo ufw status verbose             # Check status

sudo ufw allow 22/tcp               # Allow SSH
sudo ufw allow 80/tcp               # Allow HTTP
sudo ufw deny 23/tcp                # Block Telnet
sudo ufw allow from 192.168.1.0/24  # Allow subnet
sudo ufw delete allow 80/tcp        # Remove rule

sudo ufw default deny incoming      # Block all incoming by default
sudo ufw default allow outgoing     # Allow all outgoing
```

---

## Key Takeaways
- Default policy should be `DROP` — only allow what's needed
- Always allow `ESTABLISHED,RELATED` to not break existing connections
- Always allow loopback (`lo`) interface
- Rate limiting with `--limit` prevents SYN floods and brute force
- Log before DROP to capture evidence of blocked attempts
- Test rules carefully — a wrong INPUT DROP rule can lock you out of SSH
- Use `iptables-save` to persist rules across reboots
