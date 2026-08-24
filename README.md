# Burp Suite Web App Testing Lab

Intercepting and exploiting SQL Injection and Reflected XSS in DVWA (Damn Vulnerable Web Application) using Burp Suite Community — proxying browser traffic and manipulating requests directly.

## Findings

| Vulnerability | Location | Result |
|---|---|---|
| SQL Injection | `/vulnerabilities/sqli/` | `' OR 1=1#` returned all 5 user records — no input sanitization |
| Reflected XSS | `/vulnerabilities/xss_r/` | `<script>alert('XSS')</script>` executed unencoded |

`[SCREENSHOT: Burp Suite intercepting the SQLi request]`
`[SCREENSHOT: XSS alert firing in-browser]`

## Fixes for each

- **SQLi:** parameterized queries, least-privilege DB accounts, no raw SQL errors returned to the client
- **XSS:** output encoding, a Content-Security-Policy header, HttpOnly cookies

## MITRE ATT&CK

T1190 (Exploit Public-Facing Application), T1552.001 (Unsecured Credentials in Database), T1059.007 (Client-Side Script Execution)

## Tools

Burp Suite Community Edition, DVWA, Kali Linux
