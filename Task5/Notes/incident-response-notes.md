# Incident Response Notes — Task 5

## Overview
Incident Response (IR) is a structured approach to handling and managing the aftermath of a security breach or attack. The goal is to limit damage, reduce recovery time, and prevent future incidents.

---

## IR Framework — NIST SP 800-61

```
Preparation → Detection → Containment → Eradication → Recovery → Lessons Learned
```

---

## 1. Preparation

Before an incident occurs:
- Deploy logging (auth.log, syslog, application logs)
- Set up SIEM or log aggregation
- Define IR roles and responsibilities
- Create communication plan
- Maintain clean backup images

---

## 2. Detection & Analysis

### Log Sources
```bash
# Authentication log (Linux)
cat /var/log/auth.log
tail -f /var/log/auth.log

# System log
cat /var/log/syslog

# Apache access log
cat /var/log/apache2/access.log

# Failed login attempts
grep "Failed password" /var/log/auth.log

# Successful logins
grep "Accepted password" /var/log/auth.log

# SSH connection events
grep "sshd" /var/log/auth.log

# User switching (su/sudo)
grep "su\[" /var/log/auth.log
grep "sudo" /var/log/auth.log
```

### Indicators of Compromise (IOCs) in Logs
```
Authentication failures:
  Dec 15 22:14:33 host sshd[1234]: Failed password for msfadmin from 192.168.1.5 port 43210 ssh2

Multiple failures from same IP = brute force:
  Dec 15 22:14:33 Failed password from 192.168.1.5
  Dec 15 22:14:34 Failed password from 192.168.1.5
  Dec 15 22:14:35 Failed password from 192.168.1.5

Successful login after failures = compromise:
  Dec 15 22:14:40 Accepted password for msfadmin from 192.168.1.5

Unusual su/sudo events:
  Dec 15 22:15:01 su[1235]: Successful su for root by msfadmin

Off-hours activity:
  Dec 15 03:22:11 sshd: Connection from 192.168.1.5
```

---

## 3. Containment

Short-term containment (stop the bleeding):
```bash
# Block attacker IP immediately
iptables -A INPUT -s 192.168.1.5 -j DROP

# Kill active suspicious session
who                          # Find active sessions
pkill -KILL -u msfadmin      # Kill all processes for user

# Disable compromised account
passwd -l msfadmin           # Lock account
usermod -L msfadmin          # Alternative lock
```

Long-term containment:
```bash
# Restrict SSH to specific IPs
iptables -A INPUT -p tcp --dport 22 -s 192.168.1.0/24 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j DROP

# Monitor exposed services
netstat -tulnp | grep LISTEN

# Enable fail2ban
sudo systemctl start fail2ban
```

---

## 4. Eradication

Remove the threat:
```bash
# Patch vulnerable software
sudo apt update && sudo apt upgrade -y

# Remove unused services
sudo systemctl disable vsftpd
sudo systemctl disable telnet
sudo systemctl disable smbd

# Check for unauthorized files
find /tmp /var/tmp -type f -newer /etc/passwd 2>/dev/null
find / -name "*.php" -newer /etc/passwd 2>/dev/null

# Check for unauthorized users
cat /etc/passwd | grep -v nologin
last                         # Login history
lastb                        # Failed login history

# Remove malicious files/backdoors
# Audit crontabs for persistence
crontab -l
cat /etc/cron.d/*
```

---

## 5. Recovery

Restore normal operations:
```bash
# Restart clean services
sudo systemctl restart apache2
sudo systemctl restart ssh

# Reset compromised credentials
passwd msfadmin

# Verify firewall rules active
iptables -L -n -v

# Validate access controls
sudo -l                      # Verify sudo rules
cat /etc/sudoers

# Test services are working correctly
curl http://localhost
ssh user@localhost
```

---

## 6. Lessons Learned

Document what happened and what to improve:

```
What was detected?
  - Authentication failures observed in auth.log
  - SSH brute force from 192.168.1.5
  - Successful login after brute force

What worked?
  - Logging captured the attack sequence
  - Firewall block was applied quickly

What failed?
  - No rate limiting on SSH initially
  - Weak passwords allowed brute force to succeed
  - No real-time alerting on auth failures

Improvements:
  - Deploy fail2ban for automatic SSH blocking
  - Enforce key-based authentication only
  - Set up log monitoring with alerting
  - Regular password audits
  - Continuous monitoring is necessary
```

---

## Key Takeaways
- auth.log is the primary source for detecting SSH and authentication attacks
- Brute force = many failures in short time from same IP
- Containment must happen fast — isolate before full analysis
- Eradication means removing the root cause, not just the symptom
- Every incident improves the organization's security posture
- Continuous monitoring is the most important lesson learned
