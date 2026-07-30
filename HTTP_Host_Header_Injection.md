---
title: HTTP Host Header Injection
aliases:
  - Host Header Injection
  - Host Header Attack
  - X-Forwarded-Host Injection
tags:
  - WebSecurity
  - OWASP
  - BugBounty
  - BurpSuite
  - HTTP
  - HostHeader
  - ReverseProxy
created: 2026-07-30
---

# 🌐 HTTP Host Header Injection

> [!info]
> **HTTP Host Header Injection** occurs when a web application **trusts client-controlled Host-related headers** (such as `Host` or `X-Forwarded-Host`) for security-sensitive operations without proper validation.
>
> This can lead to vulnerabilities like:
>
> - Password Reset Poisoning
> - Web Cache Poisoning
> - Open Redirects
> - Routing Confusion
> - Authentication Issues
> - Multi-tenant Misconfiguration

---

# 📖 What is the Host Header?

Every HTTP/1.1 request contains a **Host** header.

Example:

```http
GET /dashboard HTTP/1.1
Host: app.example.com
```

The **Host** header tells the web server:

> "Which website is this request intended for?"

Many servers host multiple websites on the same IP address.

Without the Host header, the server would not know which website to serve.

---

# 🏗 Example Architecture

```text
                Internet
                    │
                    ▼
              Browser
                    │
                    ▼
        Reverse Proxy / Load Balancer
                    │
                    ▼
             Backend Application
```

Sometimes the reverse proxy forwards additional headers to the backend.

---

# Common Forwarded Headers

```http
Host:
X-Forwarded-Host:
X-Forwarded-For:
X-Forwarded-Proto:
Forwarded:
```

---

# What does each header mean?

| Header | Purpose |
|----------|----------|
| Host | Current destination hostname |
| X-Forwarded-Host | Original hostname requested by the client |
| X-Forwarded-For | Original client IP |
| X-Forwarded-Proto | Original protocol (HTTP/HTTPS) |
| Forwarded | Standardized replacement for X-Forwarded-* headers |

---

# Real Example

Browser request

```http
GET /profile HTTP/1.1
Host: app.example.com
```

Backend receives

```http
Host: internal-lb.local
X-Forwarded-Host: app.example.com
```

The backend may trust

```text
X-Forwarded-Host
```

instead of

```text
Host
```

---

# Why do Security Testers Modify Host?

When using Burp Suite, testers change Host-related headers to determine whether the application improperly trusts client-supplied values.

Example:

Original

```http
GET / HTTP/1.1
Host: app.example.com
```

Modified

```http
GET / HTTP/1.1
Host: attacker.com
```

The goal is to observe whether the application changes its behavior.

---

# Testing X-Forwarded-Host

Instead of changing Host,

a tester may add

```http
Host: app.example.com
X-Forwarded-Host: attacker.com
```

Some applications validate the Host header but accidentally trust the X-Forwarded-Host header.

---

# What Should You Observe?

When modifying Host headers, observe whether the application changes:

- Redirect URLs
- Absolute URLs
- Password reset links
- Email links
- Canonical URLs
- API responses
- OAuth callback URLs
- HTML links

---

# Example Response

Normal response

```html
<a href="https://app.example.com/reset">
Reset Password
</a>
```

Modified response

```html
<a href="https://attacker.com/reset">
Reset Password
</a>
```

If the server generated the attacker-controlled URL,

this may indicate a Host Header vulnerability.

---

# Reflection vs Vulnerability

## Reflection

Response simply echoes your value.

Example

```html
<a href="https://attacker.com/home">
```

but nothing security-sensitive happens.

Usually **not reportable**.

---

## Real Vulnerability

Your supplied Host value influences:

- Password reset emails
- Login links
- OAuth redirects
- Cache keys
- API endpoints
- Security workflows

These can become valid security findings.

---

# Common Vulnerabilities

## 1. Password Reset Poisoning

Application creates reset links using:

```text
Host
```

or

```text
X-Forwarded-Host
```

Example:

```text
https://attacker.com/reset?token=XXXX
```

instead of

```text
https://app.example.com/reset?token=XXXX
```

---

## 2. Web Cache Poisoning

A cache stores content generated using a manipulated Host header.

Other users may later receive the poisoned response.

---

## 3. Open Redirect

Redirects use the Host value supplied by the client.

Example:

```http
Location:
https://attacker.com/login
```

---

## 4. Routing Confusion

Reverse proxies may route requests incorrectly when Host validation is weak.

---

## 5. Multi-Tenant Issues

Cloud applications hosting multiple customers may expose data across tenants if Host routing is insecure.

---

# Burp Suite Workflow

## Step 1

Intercept the request.

```http
GET / HTTP/1.1
Host: app.example.com
```

---

## Step 2

Send it to Repeater.

---

## Step 3

Modify only

```http
Host:
```

Observe the response.

---

## Step 4

Restore Host.

Add

```http
X-Forwarded-Host:
```

Observe the response again.

---

## Step 5

Compare:

- HTML
- Redirects
- JSON
- Email previews
- API responses

---

# Important Note

> [!warning]

Changing the HTML response inside Burp **does not prove a vulnerability.**

Example:

Original response

```html
<a href="https://app.example.com/reset">
```

You manually edit it to

```html
<a href="https://attacker.com/reset">
```

Only **your browser** sees that modified response.

The server is **not vulnerable** simply because Burp allowed you to edit the HTML.

---

# When is it a Real Vulnerability?

The server itself must generate attacker-controlled values.

Example

Request

```http
Host: app.example.com
X-Forwarded-Host: attacker.com
```

Server response

```html
<a href="https://attacker.com/reset">
```

The application trusted your header.

This is potentially reportable.

---

# Match and Replace

Burp Suite allows automatic modification of requests.

Location:

```text
Proxy
    └── Options
            └── Match and Replace
```

Useful for repeatedly changing:

- Host
- X-Forwarded-Host
- X-Forwarded-For
- URLs
- Cookies
- Headers

---

# Common Testing Indicators

Look for changes in:

- HTTP Location header
- HTML links
- Password reset URLs
- Canonical URLs
- JSON absolute URLs
- Email previews
- OAuth callbacks

---

# Host vs X-Forwarded-Host

| Host | X-Forwarded-Host |
|-------|------------------|
| Current destination | Original hostname before proxy |
| Used by web server | Often used by backend |
| Always present in HTTP/1.1 | Usually added by reverse proxies |

---

# Easy Memory Trick

> [!tip]

**Host**

> "Where is this request going?"

**X-Forwarded-Host**

> "What hostname did the client originally request before the proxy?"

---

# Vulnerability Mapping

| Header Change | Possible Finding |
|---------------|------------------|
| Host | Host Header Injection |
| X-Forwarded-Host | Proxy Trust Issue |
| Redirect changes | Open Redirect |
| Password reset link changes | Password Reset Poisoning |
| Cache affected | Web Cache Poisoning |
| Backend routing changes | Routing Confusion |

---

# Impact

| Impact | Severity |
|----------|----------|
| Password Reset Poisoning | 🔴 High |
| Cache Poisoning | 🔴 High |
| Open Redirect | 🟠 Medium |
| Multi-Tenant Routing Issues | 🔴 High |
| Authentication Confusion | 🔴 High |

---

# Secure Implementation

Developers should:

- Validate Host headers against an allowlist.
- Trust forwarded headers only from trusted reverse proxies.
- Never generate security-sensitive URLs directly from client-controlled headers.
- Configure reverse proxies to strip untrusted forwarded headers.
- Use fixed application base URLs for emails and redirects.

---

# Related Notes

- [[HTTP Request]]
- [[Reverse Proxy]]
- [[Burp Suite]]
- [[Password Reset Poisoning]]
- [[Open Redirect]]
- [[Web Cache Poisoning]]
- [[SSRF]]
- [[OWASP Top 10 2025]]

---

# Key Takeaways

> [!summary]

- The **Host** header identifies the destination website.
- **X-Forwarded-Host** is commonly added by reverse proxies to indicate the original hostname.
- Applications should not blindly trust client-controlled Host-related headers.
- Simply seeing your value reflected in a response is **not enough** to confirm a vulnerability.
- A reportable issue exists when manipulated Host-related headers influence security-sensitive behavior, such as generating password reset links, redirects, cache entries, or routing decisions.