# DVWA Setup Notes — Task 3

## What is DVWA?
**Damn Vulnerable Web Application (DVWA)** is a PHP/MySQL web application that is intentionally vulnerable. It is designed for security professionals to practice common web vulnerabilities in a legal, controlled environment.

**Official Repo:** https://github.com/digininja/DVWA

---

## Installation on Kali Linux

### Method 1 — Using apt (Easiest)
```bash
sudo apt update
sudo apt install dvwa -y
sudo dvwa-start        # Start DVWA service
```
Access at: `http://127.0.0.1:42001`

---

### Method 2 — Manual Installation

#### Step 1: Install Apache, PHP, MySQL
```bash
sudo apt update
sudo apt install apache2 php php-mysqli php-gd mariadb-server git -y
```

#### Step 2: Clone DVWA
```bash
cd /var/www/html
sudo git clone https://github.com/digininja/DVWA.git dvwa
```

#### Step 3: Configure DVWA
```bash
cd /var/www/html/dvwa/config
sudo cp config.inc.php.dist config.inc.php
sudo nano config.inc.php
```

Edit these lines:
```php
$_DVWA['db_user']     = 'dvwa';
$_DVWA['db_password'] = 'p@ssw0rd';
$_DVWA['db_database'] = 'dvwa';
```

#### Step 4: Set Up MySQL Database
```bash
sudo systemctl start mariadb
sudo mysql -u root -p
```
```sql
CREATE DATABASE dvwa;
CREATE USER 'dvwa'@'localhost' IDENTIFIED BY 'p@ssw0rd';
GRANT ALL PRIVILEGES ON dvwa.* TO 'dvwa'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

#### Step 5: Set File Permissions
```bash
sudo chmod -R 777 /var/www/html/dvwa/hackable/uploads/
sudo chmod -R 777 /var/www/html/dvwa/config/
```

#### Step 6: Configure PHP
```bash
sudo nano /etc/php/*/apache2/php.ini
```
Set:
```ini
allow_url_fopen = On
allow_url_include = On
display_errors = Off
```

#### Step 7: Start Apache
```bash
sudo systemctl start apache2
sudo systemctl enable apache2
```

#### Step 8: Initialize DVWA
- Open browser: `http://127.0.0.1/dvwa/setup.php`
- Click **"Create / Reset Database"**
- Login: `admin` / `password`

---

## DVWA Security Levels

| Level | Description |
|-------|-------------|
| **Low** | No protection — all vulnerabilities exploitable |
| **Medium** | Partial protection — basic filters applied |
| **High** | Strong protection — hard to exploit |
| **Impossible** | Fully secure — for comparison only |

Change security level: **DVWA Security** tab → select level → Submit

---

## DVWA Vulnerability Modules

| Module | Vulnerability Type |
|--------|--------------------|
| Brute Force | Password brute forcing |
| Command Injection | OS command injection |
| CSRF | Cross-Site Request Forgery |
| File Inclusion | LFI / RFI |
| File Upload | Malicious file upload |
| Insecure CAPTCHA | CAPTCHA bypass |
| SQL Injection | Classic SQLi |
| SQL Injection (Blind) | Blind SQLi |
| Weak Session IDs | Session prediction |
| XSS (DOM) | DOM-based XSS |
| XSS (Reflected) | Reflected XSS |
| XSS (Stored) | Stored XSS |

---

## Default Credentials

| Username | Password |
|----------|---------|
| admin | password |
| gordonb | abc123 |
| 1337 | charley |
| pablo | letmein |
| smithy | password |

---

## Key Notes
- Always run DVWA in an **isolated VM** — never on a public network
- Set security level to **Low** when starting each exercise
- Use Burp Suite proxy (`127.0.0.1:8080`) for intercepting requests
- DVWA requires PHP `allow_url_include = On` for RFI exercises
