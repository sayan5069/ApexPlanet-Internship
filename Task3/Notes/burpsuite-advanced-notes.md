# Burp Suite Advanced Notes — Task 3

## Overview
Burp Suite is the industry-standard web application security testing tool. This document covers advanced usage — intercepting login requests, fuzzing with Intruder, and analyzing web app vulnerabilities.

---

## 1. Setup & Configuration

### Proxy Configuration
1. Open Burp Suite → Proxy → Options
2. Listener running on `127.0.0.1:8080` (default)
3. In browser (Firefox): Settings → Network Settings → Manual Proxy
   - HTTP Proxy: `127.0.0.1` Port: `8080`
4. Install Burp CA Certificate:
   - Visit `http://burpsuite` or `http://127.0.0.1:8080`
   - Download CA cert → Import into browser trusted certs

### FoxyProxy (Recommended Browser Extension)
- Install FoxyProxy Standard
- Add proxy: `127.0.0.1:8080`
- Toggle on/off with one click

---

## 2. Intercepting Login Requests

### Step 1 — Enable Intercept
Proxy tab → Intercept → **Intercept is ON**

### Step 2 — Submit Login Form
Go to DVWA login: `http://127.0.0.1/dvwa/login.php`
Enter credentials and click Login.

### Step 3 — Capture the Request
Burp intercepts the POST request:
```
POST /dvwa/login.php HTTP/1.1
Host: 127.0.0.1
Content-Type: application/x-www-form-urlencoded
Cookie: PHPSESSID=abc123; security=low

username=admin&password=password&Login=Login
```

### Step 4 — Modify the Request
- Change `password=password` to `password=wrongpass` → Forward → observe response
- Change `username=admin` to `username=admin' OR '1'='1` → test for SQLi
- Add/remove headers → observe server behavior

### Step 5 — Forward or Drop
- **Forward** — send modified request to server
- **Drop** — discard the request entirely
- **Action → Send to Repeater** — for manual repeated testing

---

## 3. Burp Repeater — Manual Testing

Repeater lets you manually resend and modify a request as many times as needed.

### Workflow
1. Intercept a request → Right-click → **Send to Repeater** (Ctrl+R)
2. Go to **Repeater** tab
3. Modify the request
4. Click **Send**
5. Analyze the response on the right panel

### Use Cases
```
Testing SQL injection payloads:
  id=1
  id=1'
  id=1' OR '1'='1'-- -
  id=1' UNION SELECT 1,2-- -

Testing XSS payloads:
  name=test
  name=<script>alert(1)</script>
  name=<img src=x onerror=alert(1)>

Testing authentication bypass:
  username=admin'-- -&password=anything
  username=' OR 1=1-- -&password=x
```

---

## 4. Burp Intruder — Fuzzing & Brute Force

Intruder automates sending many modified requests — for brute force, fuzzing, and parameter manipulation.

### Step 1 — Send to Intruder
Intercept request → Right-click → **Send to Intruder** (Ctrl+I)

### Step 2 — Set Attack Positions
Go to **Intruder → Positions** tab.
Burp auto-marks parameters with `§markers§`.

**Attack Types:**
| Type | Use Case |
|------|----------|
| **Sniper** | Single payload list, one position at a time |
| **Battering Ram** | Same payload in all positions simultaneously |
| **Pitchfork** | Multiple lists, one item per position in parallel |
| **Cluster Bomb** | All combinations of multiple lists |

### Step 3 — Brute Force Login Example

Request in Positions tab:
```
POST /dvwa/login.php HTTP/1.1
...
username=§admin§&password=§password§&Login=Login
```

Attack type: **Cluster Bomb**

Payload Set 1 (usernames):
```
admin
administrator
root
user
```

Payload Set 2 (passwords):
```
password
admin
123456
letmein
```

### Step 4 — Configure Payload
Intruder → **Payloads** tab:
- Select payload set number
- Payload type: **Simple list** (paste wordlist)
- Or: **Runtime file** → load `/usr/share/wordlists/rockyou.txt`

### Step 5 — Grep for Success
Intruder → **Options** tab → **Grep - Match**:
- Add string: `Welcome` or `Login successful`
- Responses with this string = successful login

### Step 6 — Start Attack
Click **Start Attack** → sort by **Length** or **Status** to find anomalies.

### Fuzzing Parameters
```
# Fuzz SQL injection
Payload: SQL injection list from SecLists
/usr/share/seclists/Fuzzing/SQLi/Generic-SQLi.txt

# Fuzz XSS
/usr/share/seclists/Fuzzing/XSS/XSS-Jhaddix.txt

# Fuzz file paths (LFI)
/usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt

# Fuzz directories
/usr/share/seclists/Discovery/Web-Content/common.txt
```

---

## 5. Burp Decoder

Quickly encode/decode data in various formats.

```
Base64 encode/decode
URL encode/decode
HTML encode/decode
Hex encode/decode
ASCII hex
Gzip compress/decompress
```

**Example — Decode session token:**
1. Copy cookie value
2. Decoder tab → paste → Decode as Base64
3. Reveals: `{"user":"admin","role":"user"}`

---

## 6. Burp Scanner (Pro) / Active Scan

For Burp Suite Professional:
```
Right-click any request → Do active scan
Or: Target → Site map → right-click host → Scan
```

For Community (manual equivalent):
- Use Repeater with manual payloads
- Use Intruder with vulnerability-specific lists

---

## 7. Useful Burp Suite Shortcuts

| Shortcut | Action |
|----------|--------|
| Ctrl+R | Send to Repeater |
| Ctrl+I | Send to Intruder |
| Ctrl+S | Send to Scanner |
| Ctrl+F | Forward intercepted request |
| Ctrl+T | New tab in Repeater |
| Ctrl+Z | Undo in editor |
| F12 | Focus on message editor |

---

## 8. Common Burp Workflow for DVWA

```
1. Set proxy, open DVWA, login
2. Turn Intercept ON
3. Navigate to vulnerable page (SQLi, XSS, CSRF, etc.)
4. Submit form / trigger request
5. Burp captures request
6. Send to Repeater for manual testing
7. Send to Intruder for automated fuzzing
8. Document findings with screenshots
```

---

## Key Takeaways
- Burp Proxy is the foundation — everything flows through it
- Repeater is for manual, precise testing of individual requests
- Intruder automates brute force and fuzzing at scale
- Decoder is invaluable for understanding encoded data (JWT, cookies, params)
- Community edition is fully capable for manual testing — Pro adds automation
- Always check response **length** differences — they reveal behavioral changes
