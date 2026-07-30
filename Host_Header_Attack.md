**Tags:** #web-security #host-header #owasp #ceh #bugbounty #http

---

## What is a Host Header Attack?

A **Host Header Attack** occurs when a web application **trusts the HTTP `Host` header** supplied by the client and uses it in security-sensitive functionality.

### Example Request

```http
GET / HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
```

The `Host` header tells the server **which domain the client wants to access**.

---

## Why Does the Host Header Exist?

HTTP/1.1 allows **multiple websites to share the same IP address** (virtual hosting).

### Example

|Domain|IP Address|
|---|---|
|`shop.example.com`|`203.0.113.10`|
|`blog.example.com`|`203.0.113.10`|
|`api.example.com`|`203.0.113.10`|

The web server uses the `Host` header to decide **which site configuration to serve**.

---

# The Core Concept

> [!info] Core Idea  
> The application assumes:
> 
> **“If the request says `Host: example.com`, it must really be for example.com.”**

That assumption is dangerous because **clients can modify HTTP headers**.

---

# How the Vulnerability Happens

## Vulnerable Logic

```python
# Pseudocode
host = request.headers["Host"]
reset_link = "https://" + host + "/reset?token=" + token
send_email(reset_link)
```

The application uses the **user-controlled Host header** to generate a password reset URL.

---

# Real-World Scenario — Password Reset Poisoning

## Normal Request

```http
POST /forgot-password HTTP/1.1
Host: app.company.com
```

The email contains:

```text
https://app.company.com/reset?token=abc123
```

---

## Malicious Request

```http
POST /forgot-password HTTP/1.1
Host: attacker.com
```

The application may generate:

```text
https://attacker.com/reset?token=abc123
```

### What Happens Next?

1. Victim receives the email.
    
2. Victim clicks the link.
    
3. The browser goes to `attacker.com`.
    
4. The token may be exposed to the attacker.
    

> [!danger] Impact  
> This can lead to **account takeover**.

---

# Visual Attack Flow

```text
Attacker
   |
   | POST /forgot-password
   | Host: attacker.com
   v
Application
   |
   | Generates reset link using Host header
   v
Email Sent to Victim
   |
   | https://attacker.com/reset?token=XYZ
   v
Victim clicks link
   |
   v
Attacker captures token
```

---

# Common Exploitation Categories

## Password Reset Poisoning

The most common and high-impact category.

### Indicators

- Password reset emails
    
- Account activation emails
    
- Invitation links
    
- Magic login links
    

---

## Web Cache Poisoning

If a reverse proxy or CDN caches responses based on the Host header, attackers may poison cached content.

### Example

```http
GET / HTTP/1.1
Host: evil.example
```

The cached response may later be served to legitimate users.

---

## URL Generation Attacks

Applications often build absolute URLs:

```php
$url = 'https://' . $_SERVER['HTTP_HOST'] . '/profile';
```

Affected features include:

- Canonical URLs
    
- Redirects
    
- Email links
    
- API documentation
    
- OAuth callbacks
    

---

## SSRF-Assisted Scenarios

Some internal services trust the Host header for routing, which can contribute to **Server-Side Request Forgery (SSRF)** chains.

---

# Why Developers Introduce This Bug

## Convenience

Frameworks provide easy access to the host:

### PHP

```php
$_SERVER['HTTP_HOST']
```

### Django

```python
request.get_host()
```

### Express.js

```javascript
req.headers.host
```

Developers use these values without validating them.

---

# Vulnerable Patterns

## Direct Use in Email

```php
$link = "https://" . $_SERVER['HTTP_HOST'] . "/verify/" . $token;
```

## Redirect Logic

```php
header("Location: https://" . $_SERVER['HTTP_HOST'] . "/dashboard");
```

## Multi-Tenant Routing

```python
tenant = request.headers['Host'].split('.')[0]
```

---

# Real Historical Incidents

Host header vulnerabilities have affected:

- Password reset systems
    
- E-commerce platforms
    
- SaaS applications
    
- CMS installations
    
- Custom enterprise portals
    

OWASP classifies these issues under **Improper Input Validation** and related web application risks.

---

# How Security Testers Detect It

> [!important] Authorized Testing Only  
> Perform these checks only in environments you are authorized to test.

## Step 1 — Observe Normal Behavior

Send a normal request and note:

- Redirects
    
- Absolute URLs
    
- Email-generated links
    
- Response headers
    

## Step 2 — Modify the Host Header

Example:

```http
GET / HTTP/1.1
Host: test.invalid
```

## Step 3 — Look for Reflection

Check whether the response contains the supplied host.

### Example Response

```html
<link rel="canonical" href="https://test.invalid/">
```

This suggests the application is using the Host header.

---

# High-Value Testing Targets

|Feature|Why|
|---|---|
|Password reset|Token exposure|
|Email verification|Account activation abuse|
|Invite systems|Unauthorized onboarding|
|OAuth login|Callback manipulation|
|SSO portals|Identity flow disruption|
|CDN-backed apps|Cache poisoning|

---

# Host vs X-Forwarded-Host

Modern deployments often sit behind proxies.

## Original Request

```http
Host: app.company.com
```

## Proxy Adds

```http
X-Forwarded-Host: app.company.com
```

If the application trusts `X-Forwarded-Host` without validation, the same vulnerability may occur.

> [!warning] Proxy Trust Issue  
> Misconfigured reverse proxies can make `X-Forwarded-Host` even more dangerous than `Host`.

---

# Impact Severity

|Impact|Severity|
|---|---|
|Open redirect|Low–Medium|
|URL poisoning|Medium|
|Cache poisoning|Medium–High|
|Password reset poisoning|High|
|Account takeover|Critical|

---

# Secure Design

## ❌ Unsafe

```php
$host = $_SERVER['HTTP_HOST'];
$url = "https://$host/reset/$token";
```

---

## ✅ Safe — Hardcoded Trusted Domain

```php
$trustedHost = 'app.company.com';
$url = "https://$trustedHost/reset/$token";
```

### Why This Is Safe

The domain is **not taken from user input**.

---

# Framework-Level Defenses

## Django

```python
ALLOWED_HOSTS = ['app.company.com']
```

Requests with unexpected hosts are rejected.

---

## Express + Helmet

Validate the host against an allowlist.

```javascript
const allowed = ['app.company.com'];
if (!allowed.includes(req.hostname)) {
  return res.status(400).send('Invalid host');
}
```

---

## Nginx

```nginx
server {
    server_name app.company.com;
    if ($host != 'app.company.com') {
        return 444;
    }
}
```

---

# Additional Mitigations

- Configure **strict virtual hosts**.
    
- Reject unknown Host headers at the **web server** layer.
    
- Use **absolute URLs from configuration**, not from requests.
    
- Validate `X-Forwarded-Host` only from trusted proxies.
    
- Monitor logs for unusual host values.
    
- Implement **rate limiting** on password reset endpoints.
    

---

# Detection Checklist

-  Can I change the Host header?
    
-  Is the Host reflected in the response?
    
-  Are emails generated using the Host?
    
-  Do redirects use the supplied host?
    
-  Is a CDN or cache present?
    
-  Is `X-Forwarded-Host` trusted?
    

---

# Quick Interview Answer

> **A Host Header Attack occurs when a web application trusts the client-supplied `Host` header and uses it in security-sensitive operations such as password reset links, redirects, or URL generation. An attacker can manipulate the header to poison emails, perform cache poisoning, or potentially achieve account takeover. The primary defense is to validate the host against a strict allowlist and generate sensitive URLs from server-side configuration rather than user-controlled headers.**

---

# Related Notes

- [[HTTP Fundamentals]]
    
- [[Password Reset Poisoning]]
    
- [[Web Cache Poisoning]]
    
- [[SSRF]]
    
- [[OWASP Top 10]]
    
- [[Reverse Proxy Security]]
    
- [[Nginx Hardening]]
    

---

# Exam Memory Trick

## “HOST”

- **H** → Header is user-controlled
    
- **O** → Open redirects and poisoned URLs
    
- **S** → Sensitive links (reset, verify, invite)
    
- **T** → Trusted domain must be hardcoded or allowlisted