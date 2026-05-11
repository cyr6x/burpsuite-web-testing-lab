# Burp Suite Web Application Testing Lab
**Web App Pentesting | Traffic Interception | SQLi | XSS | Defender Analysis**

> A hands-on web application penetration testing lab using Burp Suite Community Edition to intercept HTTP traffic and exploit SQL Injection and Cross-Site Scripting (XSS) vulnerabilities in DVWA — simulating techniques used by real attackers and mapping them to MITRE ATT&CK.

---

## Why This Matters

Web application attacks are among the most common attack vectors in the wild. SQL Injection and XSS consistently appear in the OWASP Top 10. This lab demonstrates how attackers intercept and manipulate HTTP requests, bypass authentication, extract database contents, and inject malicious scripts — paired with the controls that prevent each attack.

---

## Lab Environment

| Component | Details |
|---|---|
| **Testing Tool** | Burp Suite Community Edition (pre-installed on Kali) |
| **Target Application** | DVWA (Damn Vulnerable Web Application) |
| **Platform** | Kali Linux (VM) |
| **DVWA URL** | http://127.0.0.1/dvwa |
| **Network** | Localhost only — isolated, no external exposure |

> ⚠️ All testing performed against a locally hosted intentionally vulnerable application. No live systems were targeted.

---

## Objectives

- Configure Burp Suite as a proxy to intercept browser traffic
- Capture and inspect raw HTTP requests and responses
- Exploit SQL Injection to bypass authentication and extract data
- Exploit Reflected XSS to inject and execute a script
- Document findings and map to MITRE ATT&CK
- Identify developer and SOC controls that prevent each attack

---

## Setup

### Install & Start DVWA on Kali
```bash
sudo apt install dvwa -y
sudo dvwa-start
```
Open `http://127.0.0.1/dvwa` in Firefox on Kali.

Default credentials:
- **Username:** admin
- **Password:** password

Set DVWA Security Level to **Low** (DVWA Security tab).

### Configure Burp Suite Proxy
1. Open Burp Suite Community → **Proxy > Intercept**
2. In Firefox: Settings → Network Settings → Manual proxy → `127.0.0.1:8080`
3. Browse to DVWA — Burp will intercept all requests

---

## Attack 1 — SQL Injection

### What it is
SQL Injection occurs when user input is inserted directly into a database query without sanitisation, allowing an attacker to manipulate the query logic.

### Steps
1. In DVWA, go to **SQL Injection**
2. In Burp Suite, enable **Intercept**
3. Submit any ID in the DVWA input field (e.g. `1`)
4. In Burp, observe the raw GET request
5. Forward the request, then test the following payloads directly in the DVWA input:

```sql
-- Bypass: always-true condition
1' OR '1'='1

-- Extract all users from the database
1' OR '1'='1' --

-- UNION-based: extract database version
1' UNION SELECT null, version() --

-- UNION-based: extract all usernames and passwords
1' UNION SELECT user, password FROM users --
```

### Expected Result
- Authentication bypass: all user records returned
- Database version disclosed
- Plaintext (MD5-hashed) credentials extracted from the `users` table

---

## Attack 2 — Reflected XSS

### What it is
Cross-Site Scripting (XSS) occurs when unsanitised user input is reflected back in the HTML response and executed as JavaScript in the victim’s browser.

### Steps
1. In DVWA, go to **XSS (Reflected)**
2. In Burp Suite, enable **Intercept**
3. Submit the following payload in the name field:

```html
<!-- Basic alert pop-up (proof of concept) -->
<script>alert('XSS')</script>

<!-- Cookie theft simulation -->
<script>alert(document.cookie)</script>
```

4. Observe the intercepted GET request in Burp — note the payload in the URL parameter
5. Forward the request — the script executes in the browser

### Expected Result
- Alert box appears confirming script execution
- Cookie value displayed — demonstrating session hijack potential

---

## Findings

| Attack | Vulnerability | Location | Severity | Result |
|---|---|---|---|---|
| SQL Injection | Unsanitised SQL query | DVWA `/vulnerabilities/sqli/` | Critical | All user credentials extracted |
| Reflected XSS | Unsanitised input reflected in HTML | DVWA `/vulnerabilities/xss_r/` | High | Script executed, cookie exposed |

---

## MITRE ATT&CK Mapping

| Tactic | Technique | ID |
|---|---|---|
| Initial Access | Exploit Public-Facing Application | T1190 |
| Credential Access | Exploitation for Credential Access | T1212 |
| Credential Access | Unsecured Credentials in Database | T1552.001 |
| Collection | Data from Information Repositories | T1213 |
| Execution | XSS — Client-Side Script Execution | T1059.007 |
| Collection | Session Hijacking via Cookie Theft | T1539 |

---

## Screenshots

```
screenshots/
├── 01-burp-intercept.png        # Burp Suite intercepting a DVWA HTTP request
├── 02-sqli-payload.png          # SQLi payload in DVWA input field
├── 03-sqli-results.png          # Extracted user credentials from database
├── 04-xss-payload.png           # XSS payload submitted in name field
└── 05-xss-alert.png             # Alert box executing in browser
```

---

## Defender Takeaways

### SQL Injection — Prevention
| Control | How It Helps |
|---|---|
| **Parameterised Queries / Prepared Statements** | Input never concatenated into SQL — eliminates SQLi entirely |
| **Input Validation** | Reject special characters (`'`, `--`, `;`) at the application layer |
| **Least Privilege DB accounts** | App DB user should only have SELECT on required tables — not read `users` |
| **WAF (Web Application Firewall)** | Detects and blocks SQLi patterns in HTTP requests |
| **Error Handling** | Never return raw SQL errors to the user — reveals DB structure |

### XSS — Prevention
| Control | How It Helps |
|---|---|
| **Output Encoding** | Encode `<`, `>`, `"`, `'` before rendering user input as HTML |
| **Content Security Policy (CSP)** | HTTP header that blocks inline script execution |
| **HttpOnly Cookie Flag** | Prevents JavaScript from reading session cookies — blocks cookie theft |
| **Input Sanitisation** | Strip or reject HTML tags from user input |
| **WAF** | Detects `<script>` patterns and XSS payloads in HTTP parameters |

### SOC Detection
| Signal | Detection Logic |
|---|---|
| **SQLi patterns in logs** | Web server logs containing `'`, `UNION SELECT`, `OR 1=1` in URL params |
| **XSS patterns in logs** | `<script>`, `alert(`, `document.cookie` in HTTP request params |
| **Abnormal response sizes** | SQLi dumping full tables returns much larger responses than normal |
| **WAF alerts** | ModSecurity / AWS WAF rules fire on both attack signatures |

---

## Academic & Professional Context

- **OWASP Top 10** — A03: Injection, A07: Cross-Site Scripting
- **CompTIA Security+ SY0-701** — Domain 2.3: Application vulnerabilities
- **CompTIA PenTest+ PT0-002** — Domain 3: Web application attacks
- **eWPT / BSCP** — Core web app pentesting methodology

---

## Status

- [ ] DVWA installed and running on Kali
- [ ] Burp Suite proxy configured
- [ ] HTTP request intercepted and inspected
- [ ] SQL Injection payloads tested — credentials extracted
- [ ] XSS payload executed — cookie exposed
- [ ] Screenshots captured and pushed
- [ ] Findings table completed
- [ ] MITRE ATT&CK mapping finalised
