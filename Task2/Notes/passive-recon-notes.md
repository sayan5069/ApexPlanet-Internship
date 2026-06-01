# Passive Reconnaissance Notes — Task 2

## Overview
Passive recon involves gathering information about a target **without directly interacting** with it. No packets are sent to the target — all data comes from public sources.

---

## 1. Whois

Whois queries domain registration databases to find ownership, registrar, and contact info.

```bash
whois example.com
whois 8.8.8.8
```

**Information Retrieved:**
- Registrant name, organization, email
- Registrar details
- Domain creation, expiry dates
- Name servers
- IP block owner (for IP lookups)

**Online Tools:**
- https://who.is
- https://whois.domaintools.com
- https://lookup.icann.org

**Example Output Fields:**
```
Domain Name: EXAMPLE.COM
Registrar: IANA
Creation Date: 1995-08-14
Expiry Date: 2025-08-13
Name Server: A.IANA-SERVERS.NET
```

---

## 2. Nslookup

Queries DNS servers to resolve domain names and retrieve DNS records.

```bash
nslookup example.com                    # Basic A record lookup
nslookup -type=MX example.com          # Mail server records
nslookup -type=NS example.com          # Name server records
nslookup -type=TXT example.com         # TXT records (SPF, DKIM)
nslookup -type=AAAA example.com        # IPv6 records
nslookup -type=CNAME www.example.com   # Canonical name
nslookup 8.8.8.8                       # Reverse DNS lookup
```

**Alternative — dig:**
```bash
dig example.com
dig example.com MX
dig example.com ANY
dig +short example.com
dig -x 8.8.8.8          # Reverse lookup
```

**DNS Record Types:**
| Record | Purpose |
|--------|---------|
| A | IPv4 address |
| AAAA | IPv6 address |
| MX | Mail server |
| NS | Name server |
| CNAME | Alias |
| TXT | Text (SPF, DKIM, verification) |
| PTR | Reverse DNS |
| SOA | Start of Authority |

---

## 3. Google Dorking

Using advanced Google search operators to find sensitive information indexed publicly.

**Common Operators:**
| Operator | Example | Purpose |
|----------|---------|---------|
| `site:` | `site:example.com` | Limit to specific domain |
| `filetype:` | `filetype:pdf site:example.com` | Find specific file types |
| `intitle:` | `intitle:"index of"` | Search in page title |
| `inurl:` | `inurl:admin` | Search in URL |
| `intext:` | `intext:"password"` | Search in page body |
| `cache:` | `cache:example.com` | Google's cached version |
| `link:` | `link:example.com` | Pages linking to target |

**Useful Dorks:**
```
site:example.com filetype:pdf
site:example.com inurl:login
site:example.com intitle:"index of"
inurl:"/wp-admin" site:example.com
filetype:sql "password" site:example.com
intitle:"phpMyAdmin" inurl:"/phpmyadmin/"
intext:"username" intext:"password" filetype:log
```

**Google Hacking Database (GHDB):**
- https://www.exploit-db.com/google-hacking-database

> **Legal Note**: Only use Google Dorking on targets you have permission to test.

---

## 4. Shodan

Shodan is a search engine for internet-connected devices — servers, routers, cameras, IoT devices, industrial systems.

**Website:** https://shodan.io

**CLI Usage:**
```bash
shodan init <API_KEY>
shodan search "apache 2.4"
shodan search "port:22 country:IN"
shodan host 8.8.8.8
shodan count "nginx"
```

**Common Search Filters:**
| Filter | Example | Purpose |
|--------|---------|---------|
| `port:` | `port:22` | Filter by port |
| `country:` | `country:US` | Filter by country |
| `city:` | `city:Mumbai` | Filter by city |
| `org:` | `org:"Amazon"` | Filter by organization |
| `os:` | `os:"Windows"` | Filter by OS |
| `product:` | `product:"Apache"` | Filter by product |
| `version:` | `version:"2.4.49"` | Filter by version |
| `hostname:` | `hostname:example.com` | Filter by hostname |
| `net:` | `net:192.168.1.0/24` | Filter by IP range |

**Useful Shodan Queries:**
```
apache port:80 country:IN
"default password" port:23
product:"vsftpd" port:21
"MongoDB Server Information" port:27017
webcamXP port:8080
```

**Information Shodan Reveals:**
- Open ports and services
- Software versions (useful for finding CVEs)
- SSL certificate details
- Geographic location
- ISP / Organization
- Banners and service responses

---

## Key Takeaways
- Passive recon leaves **no trace** on the target
- Whois + DNS records reveal infrastructure details
- Google Dorking can expose sensitive files and login pages
- Shodan reveals exposed services and vulnerable software versions
- Always document findings with timestamps and sources
