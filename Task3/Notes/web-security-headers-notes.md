# Web Security Headers Notes — Task 3

## Overview
HTTP security headers are response headers that instruct the browser how to behave when handling a site's content. They are one of the easiest and most impactful security improvements for any web application.

---

## 1. Analyzing Headers with securityheaders.com

**Website:** https://securityheaders.com

### How to Use
1. Go to https://securityheaders.com
2. Enter the target URL (e.g., `http://127.0.0.1/dvwa/`)
3. Click **Scan**
4. Review the grade (A+ to F) and missing headers

### What it Checks
- Content-Security-Policy
- X-Frame-Options
- X-Content-Type-Options
- Referrer-Policy
- Permissions-Policy
- Strict-Transport-Security (HSTS)

### Common Findings on DVWA (Default)
```
Missing: Content-Security-Policy
Missing: X-Frame-Options
Missing: X-Content-Type-Options
Missing: Referrer-Policy
Missing: Permissions-Policy
Grade: F
```

---

## 2. Adding Security Headers in Apache

Edit Apache config or use `.htaccess`:

```bash
# Edit main Apache config
sudo nano /etc/apache2/apache2.conf

# Or create/edit .htaccess in web root
sudo nano /var/www/html/dvwa/.htaccess

# Enable headers module
sudo a2enmod headers
sudo systemctl restart apache2
```

---

## 3. Key Security Headers

### Content-Security-Policy (CSP)
Prevents XSS by controlling which resources the browser is allowed to load.

```apache
Header set Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data:; object-src 'none'; frame-ancestors 'none';"
```

**CSP Directives:**
| Directive | Purpose |
|-----------|---------|
| `default-src 'self'` | Only load resources from same origin |
| `script-src 'self'` | Only run scripts from same origin |
| `style-src 'self'` | Only load styles from same origin |
| `img-src 'self' data:` | Images from same origin + data URIs |
| `object-src 'none'` | Block all plugins (Flash, Java) |
| `frame-ancestors 'none'` | Prevent framing (blocks clickjacking) |
| `upgrade-insecure-requests` | Force HTTPS for all requests |

**Strict CSP with Nonce:**
```apache
# Generate nonce per request in application code
Header set Content-Security-Policy "script-src 'nonce-RANDOM_VALUE' 'strict-dynamic';"
```

---

### X-Frame-Options
Prevents clickjacking by controlling whether the page can be embedded in a frame.

```apache
Header set X-Frame-Options "DENY"
# or
Header set X-Frame-Options "SAMEORIGIN"
```

| Value | Meaning |
|-------|---------|
| `DENY` | Never allow framing |
| `SAMEORIGIN` | Only allow framing from same origin |
| `ALLOW-FROM uri` | Allow framing from specific URI (deprecated) |

> Note: `frame-ancestors` in CSP supersedes this header in modern browsers.

---

### X-Content-Type-Options
Prevents MIME type sniffing — browser must use the declared Content-Type.

```apache
Header set X-Content-Type-Options "nosniff"
```

**Attack it prevents:** Browser sniffing a text file as JavaScript and executing it.

---

### Strict-Transport-Security (HSTS)
Forces browser to use HTTPS only, even if HTTP is requested.

```apache
Header set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
```

| Directive | Meaning |
|-----------|---------|
| `max-age=31536000` | Enforce for 1 year |
| `includeSubDomains` | Apply to all subdomains |
| `preload` | Include in browser HSTS preload list |

> Only use on HTTPS sites — applying to HTTP breaks the site.

---

### Referrer-Policy
Controls how much referrer information is included in requests.

```apache
Header set Referrer-Policy "strict-origin-when-cross-origin"
```

| Value | Behavior |
|-------|---------|
| `no-referrer` | Never send referrer |
| `same-origin` | Send referrer only for same-origin requests |
| `strict-origin` | Send only origin (not path) to HTTPS destinations |
| `strict-origin-when-cross-origin` | Full URL same-origin, origin-only cross-origin |

---

### Permissions-Policy (formerly Feature-Policy)
Controls which browser features and APIs the page can use.

```apache
Header set Permissions-Policy "geolocation=(), microphone=(), camera=(), payment=(), usb=()"
```

This disables geolocation, microphone, camera, payment APIs, and USB access for the page.

---

### X-XSS-Protection (Legacy)
Legacy header — enables browser's built-in XSS filter (mostly deprecated in modern browsers, replaced by CSP).

```apache
Header set X-XSS-Protection "1; mode=block"
```

Still useful for older browsers (IE, older Chrome).

---

### Cache-Control for Sensitive Pages
Prevent sensitive pages from being cached.

```apache
Header set Cache-Control "no-store, no-cache, must-revalidate"
Header set Pragma "no-cache"
```

---

## 4. Complete Apache Security Headers Config

Add to `/etc/apache2/conf-available/security-headers.conf`:

```apache
<IfModule mod_headers.c>
    # Prevent clickjacking
    Header set X-Frame-Options "DENY"

    # Prevent MIME sniffing
    Header set X-Content-Type-Options "nosniff"

    # XSS protection (legacy browsers)
    Header set X-XSS-Protection "1; mode=block"

    # Control referrer info
    Header set Referrer-Policy "strict-origin-when-cross-origin"

    # Disable browser features not needed
    Header set Permissions-Policy "geolocation=(), microphone=(), camera=()"

    # Content Security Policy
    Header set Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data:; object-src 'none'; frame-ancestors 'none';"

    # HSTS (only on HTTPS)
    # Header set Strict-Transport-Security "max-age=31536000; includeSubDomains"

    # Prevent caching of sensitive responses
    Header set Cache-Control "no-store"
</IfModule>
```

Enable and apply:
```bash
sudo a2enconf security-headers
sudo a2enmod headers
sudo systemctl restart apache2
```

---

## 5. Verify Headers with curl

```bash
# Check response headers
curl -I http://127.0.0.1/dvwa/

# Check specific header
curl -I http://127.0.0.1/dvwa/ | grep -i "content-security-policy"
curl -I http://127.0.0.1/dvwa/ | grep -i "x-frame-options"

# Full header dump
curl -v http://127.0.0.1/dvwa/ 2>&1 | grep "^<"
```

---

## 6. Security Header Grade Targets

| Grade | Requirements |
|-------|-------------|
| **A+** | All headers present + HSTS preload |
| **A** | All major headers present |
| **B** | Most headers present |
| **C/D** | Several headers missing |
| **F** | Most headers missing (DVWA default) |

---

## 7. Quick Reference — Header vs Attack

| Header | Attack Prevented |
|--------|-----------------|
| Content-Security-Policy | XSS, data injection |
| X-Frame-Options | Clickjacking |
| X-Content-Type-Options | MIME sniffing attacks |
| Strict-Transport-Security | SSL stripping, protocol downgrade |
| Referrer-Policy | Sensitive URL leakage |
| Permissions-Policy | Feature abuse (camera, mic, GPS) |
| Cache-Control | Sensitive data in browser cache |

---

## Key Takeaways
- Security headers are free, easy to add, and have immediate impact
- CSP is the most powerful — it mitigates XSS at the browser level
- HSTS prevents protocol downgrade and SSL stripping attacks
- Use securityheaders.com to test before and after adding headers
- X-Frame-Options + CSP frame-ancestors together cover all browsers
- Always test that CSP does not break legitimate site functionality
