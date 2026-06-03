# System Hardening Notes — Task 4

## What is System Hardening?
System hardening is the process of reducing a system's attack surface by applying security patches, configuring firewalls, disabling unnecessary services, and enforcing security policies.

---

## 1. Apply Security Patches

### Why Patching Matters
Most exploits target known vulnerabilities with public CVEs. Keeping systems patched eliminates the majority of exploitation risk.

### Linux (Debian/Ubuntu)
```bash
# Update package list
sudo apt update

# Upgrade all packages
sudo apt upgrade -y

# Full distribution upgrade
sudo apt dist-upgrade -y

# Remove unused packages
sudo apt autoremove -y

# Check for specific security updates only
sudo apt list --upgradable 2>/dev/null | grep -i security

# Enable automatic security updates
sudo apt install unattended-upgrades
sudo dpkg-reconfigure unattended-upgrades
```

### Verify Kernel Version
```bash
uname -r                   # Current kernel version
apt-cache search linux-image   # Available kernel versions
```

### Check Installed Package Versions
```bash
dpkg -l | grep apache2
dpkg -l | grep php
dpkg -l | grep openssh
```

---

## 2. Configure Firewall to Block Malicious Traffic

### iptables — Complete Hardening Ruleset

```bash
#!/bin/bash
# System hardening firewall rules

# Flush existing rules
iptables -F
iptables -X
iptables -t nat -F

# Default policies — deny everything
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Allow loopback
iptables -A INPUT -i lo -j ACCEPT

# Allow established/related connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow SSH from specific IP or subnet only
iptables -A INPUT -p tcp --dport 22 -s 192.168.1.0/24 -j ACCEPT

# Allow HTTP/HTTPS (if web server)
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Block known malicious/unnecessary services
iptables -A INPUT -p tcp --dport 23 -j DROP     # Telnet
iptables -A INPUT -p tcp --dport 21 -j DROP     # FTP
iptables -A INPUT -p tcp --dport 512 -j DROP    # rexec
iptables -A INPUT -p tcp --dport 513 -j DROP    # rlogin
iptables -A INPUT -p tcp --dport 514 -j DROP    # rsh
iptables -A INPUT -p tcp --dport 6667 -j DROP   # IRC
iptables -A INPUT -p tcp --dport 1524 -j DROP   # bindshell backdoor

# Block port scans
iptables -A INPUT -p tcp --tcp-flags ALL NONE -j DROP    # NULL scan
iptables -A INPUT -p tcp --tcp-flags ALL ALL -j DROP     # XMAS scan
iptables -A INPUT -p tcp --tcp-flags SYN,FIN SYN,FIN -j DROP

# SYN flood protection
iptables -A INPUT -p tcp --syn -m limit --limit 1/s --limit-burst 3 -j ACCEPT
iptables -A INPUT -p tcp --syn -j DROP

# ICMP rate limiting (allow ping but prevent ICMP flood)
iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 1/s -j ACCEPT
iptables -A INPUT -p icmp -j DROP

# Log and drop everything else
iptables -A INPUT -j LOG --log-prefix "DROPPED: " --log-level 4
iptables -A INPUT -j DROP

# Save rules
iptables-save > /etc/iptables/rules.v4
echo "Firewall rules applied and saved."
```

### Verify Rules Applied
```bash
iptables -L -n -v --line-numbers
```

### ufw (Simplified Firewall for Ubuntu)
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw deny 23/tcp
sudo ufw enable
sudo ufw status verbose
```

---

## 3. Disable Unused Services

Unused services are attack vectors. Every open port is a potential entry point.

### List Running Services
```bash
systemctl list-units --type=service --state=running
ss -tulnp                  # All listening ports with services
netstat -tulnp             # Alternative
```

### Disable Vulnerable Services on Metasploitable2
```bash
# Disable Telnet
sudo systemctl stop telnet
sudo systemctl disable telnet
sudo update-rc.d telnet remove

# Disable FTP (vsftpd)
sudo systemctl stop vsftpd
sudo systemctl disable vsftpd

# Disable Samba (if not needed)
sudo systemctl stop smbd nmbd
sudo systemctl disable smbd nmbd

# Disable IRC
sudo systemctl stop unrealircd
sudo systemctl disable unrealircd

# Disable rservices (rexec, rlogin, rsh)
sudo update-inetd --disable exec
sudo update-inetd --disable login
sudo update-inetd --disable shell

# Disable distccd
sudo systemctl stop distccd
sudo systemctl disable distccd

# Disable VNC
sudo systemctl stop vncserver
sudo systemctl disable vncserver
```

### Verify Services After Disabling
```bash
nmap -sS -p- 127.0.0.1    # Scan yourself to verify ports closed
ss -tulnp                  # Check remaining listening services
```

---

## 4. SSH Hardening

```bash
sudo nano /etc/ssh/sshd_config
```

```ini
# Disable root login
PermitRootLogin no

# Disable password authentication (use keys only)
PasswordAuthentication no
ChallengeResponseAuthentication no

# Use SSH protocol 2 only
Protocol 2

# Limit login attempts
MaxAuthTries 3

# Idle timeout (5 minutes)
ClientAliveInterval 300
ClientAliveCountMax 0

# Restrict users who can SSH
AllowUsers yourusername

# Change default port (security through obscurity)
Port 2222

# Disable X11 forwarding if not needed
X11Forwarding no

# Disable empty passwords
PermitEmptyPasswords no
```

```bash
sudo systemctl restart ssh
```

---

## 5. User Account Hardening

```bash
# Lock root account (use sudo instead)
sudo passwd -l root

# List users with shells
cat /etc/passwd | grep -v nologin | grep -v false

# Remove unnecessary users
sudo userdel -r olduser

# Set password expiry policy
sudo chage -M 90 username    # Expire password every 90 days
sudo chage -l username       # Check current policy

# Enforce strong passwords (PAM)
sudo apt install libpam-pwquality
sudo nano /etc/pam.d/common-password
# Add: minlen=12 dcredit=-1 ucredit=-1 ocredit=-1 lcredit=-1

# Restrict sudo access
sudo visudo
# Only allow specific users: username ALL=(ALL) NOPASSWD:SPECIFIC_COMMAND
```

---

## 6. File System Hardening

```bash
# Find SUID files (potential privilege escalation)
find / -perm -4000 -type f 2>/dev/null

# Find world-writable files
find / -perm -o+w -type f 2>/dev/null | grep -v proc

# Remove SUID from unused binaries
sudo chmod u-s /usr/bin/unnecessary_binary

# Set correct permissions on sensitive files
sudo chmod 640 /etc/shadow
sudo chmod 644 /etc/passwd
sudo chmod 600 /root/.ssh/authorized_keys

# Mount /tmp with noexec (prevent execution from /tmp)
# Add to /etc/fstab:
# tmpfs /tmp tmpfs defaults,noexec,nosuid 0 0
```

---

## 7. Hardening Checklist

```
PATCHES:
[ ] OS fully patched
[ ] All services/packages updated
[ ] Kernel updated

SERVICES:
[ ] Telnet disabled
[ ] FTP replaced with SFTP
[ ] r-services removed
[ ] Unnecessary daemons stopped

FIREWALL:
[ ] Default deny INPUT policy
[ ] Only required ports open
[ ] Port scan protection enabled
[ ] SYN flood protection enabled

SSH:
[ ] Root login disabled
[ ] Password auth disabled (keys only)
[ ] MaxAuthTries = 3
[ ] Idle timeout configured

ACCOUNTS:
[ ] No unnecessary user accounts
[ ] Password complexity enforced
[ ] Root account locked
[ ] Sudo configured correctly

FILES:
[ ] SUID binaries audited
[ ] World-writable files removed
[ ] Sensitive file permissions correct
```

---

## Key Takeaways
- Patching eliminates the majority of known exploit risk
- Every open port is a potential attack vector — close what you don't need
- Default deny firewall policy is the correct posture
- SSH keys are far more secure than passwords
- Disable root login — use sudo for privileged operations
- Regular audits are essential — hardening is not a one-time activity
