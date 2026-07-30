---
title: Cross-Origin Resource Sharing (CORS)
aliases:
  - CORS
tags:
  - WebSecurity
  - OWASP
  - CORS
  - BrowserSecurity
  - BugBounty
  - CEH
created: 2026-07-30
---

# 🌐 Cross-Origin Resource Sharing (CORS)

> [!info]
> **Cross-Origin Resource Sharing (CORS)** is a browser security mechanism that allows a website to request resources from a different origin **only if the target server explicitly permits it**.

---

# 📖 What is an Origin?

An **Origin** is defined by three components:

```text
Protocol + Domain + Port
```

Example:

```
https://example.com:443
│       │           │
│       │           └── Port
│       └────────────── Domain
└────────────────────── Protocol
```

All three must match for two URLs to have the same origin.

---

# Same-Origin Policy (SOP)

Before understanding CORS, you must understand the **Same-Origin Policy (SOP)**.

> [!note]
> SOP is a browser security feature that **prevents JavaScript running on one origin from reading data from another origin** unless explicitly allowed.

Example:

```
Website A
https://shop.example.com

tries to read

https://bank.example.com
```

The browser blocks the request because the origins are different.

---

# Why Do We Need CORS?

Modern websites often communicate with:

- APIs
- CDNs
- Image servers
- Payment gateways
- Authentication services

Without CORS, browsers would block these legitimate requests.

Example:

```
Frontend

https://app.example.com

↓

API

https://api.example.com
```

Different subdomains = Different origins

The browser requires permission before allowing access.

---

# How CORS Works

### Step 1

Browser sends a request.

```http
GET /profile HTTP/1.1
Origin: https://app.example.com
```

Notice the **Origin** header.

---

### Step 2

Server checks whether the origin is allowed.

---

### Step 3

Server responds.

```http
Access-Control-Allow-Origin:
https://app.example.com
```

The browser now allows JavaScript to access the response.

---

# CORS Request Flow

```text
JavaScript
      │
      ▼
Browser
      │
      ▼
Origin Header Added
      │
      ▼
Server
      │
      ▼
Access-Control-Allow-Origin
      │
      ▼
Browser Checks Policy
      │
      ▼
Allow or Block Response
```

---

# Important CORS Headers

## Origin

Sent by the browser.

Example:

```http
Origin: https://app.example.com
```

---

## Access-Control-Allow-Origin

Server response.

Example:

```http
Access-Control-Allow-Origin:
https://app.example.com
```

Allows only one origin.

---

## Wildcard

```http
Access-Control-Allow-Origin: *
```

Allows every origin.

Safe only for **public resources**.

---

## Credentials

```http
Access-Control-Allow-Credentials: true
```

Allows cookies or authentication credentials.

---

## Allowed Methods

```http
Access-Control-Allow-Methods:

GET,
POST,
PUT,
DELETE
```

---

## Allowed Headers

```http
Access-Control-Allow-Headers:

Authorization,
Content-Type
```

---

# Simple Requests

No preflight request is required.

Examples:

- GET
- POST
- HEAD

Using simple content types like:

```
text/plain
application/x-www-form-urlencoded
multipart/form-data
```

---

# Preflight Requests (OPTIONS)

Some requests require permission **before** they are sent.

The browser first sends:

```http
OPTIONS /api/profile HTTP/1.1

Origin:
https://app.example.com

Access-Control-Request-Method: PUT

Access-Control-Request-Headers:
Authorization
```

The server replies:

```http
HTTP/1.1 204 No Content

Access-Control-Allow-Origin:
https://app.example.com

Access-Control-Allow-Methods:
PUT

Access-Control-Allow-Headers:
Authorization
```

Only then does the browser send the actual request.

---

# Visual Flow

```text
Browser
    │
    ▼
OPTIONS Request
    │
    ▼
Server
    │
    ▼
Permission Granted?
    │
 ┌──┴──┐
 │     │
Yes    No
 │     │
 ▼     ▼
Real  Browser
Request Blocks
```

---

# Common CORS Misconfigurations

## 1. Wildcard with Sensitive Data

Server:

```http
Access-Control-Allow-Origin: *
```

If sensitive information is returned to everyone,

this is dangerous.

---

## 2. Reflection

Request:

```http
Origin:
https://attacker.com
```

Response:

```http
Access-Control-Allow-Origin:
https://attacker.com
```

The server reflects any origin without validation.

---

## 3. Credentials Allowed with Weak Origin Validation

Example:

```http
Access-Control-Allow-Origin:
https://attacker.com

Access-Control-Allow-Credentials:
true
```

The browser may send cookies to an attacker-controlled origin if the application incorrectly trusts it.

---

## 4. Origin Validation Bypass

Poor validation such as:

```
example.com.attacker.com
```

being accepted because it merely contains the trusted domain name.

Developers should validate the complete origin rather than perform simple substring checks.

---

# CORS ≠ CSRF

| CORS | CSRF |
|------|------|
| Browser security policy | Authentication attack |
| Controls JavaScript access | Tricks authenticated users into sending requests |
| Uses Origin header | Exploits existing user sessions |
| Prevents unauthorized reading of responses | Causes unwanted actions |

---

# Burp Suite Testing Methodology

> [!warning]
> Test only systems you own or have authorization to assess.

### Step 1

Capture a request.

---

### Step 2

Look for:

```http
Origin:
```

---

### Step 3

Send the request to Repeater.

---

### Step 4

Change the Origin value.

Example:

```http
Origin:
https://example-test.com
```

---

### Step 5

Observe the response.

Look for:

```http
Access-Control-Allow-Origin:
```

and

```http
Access-Control-Allow-Credentials:
```

---

### Step 6

Determine whether the application properly validates the supplied origin.

---

# Indicators of Weak Configuration

- Reflects arbitrary Origin values.
- Allows credentials for untrusted origins.
- Uses wildcard for sensitive endpoints.
- Inconsistent validation between endpoints.

---

# Real-World Example

Frontend:

```
https://shop.example.com
```

API:

```
https://api.example.com
```

Browser request:

```http
Origin:
https://shop.example.com
```

Server:

```http
Access-Control-Allow-Origin:
https://shop.example.com
```

Browser allows JavaScript to access the API response.

---

# Impact

| Issue | Severity |
|--------|----------|
| Public data with wildcard | 🟢 Low |
| Reflection of arbitrary origins | 🟠 Medium |
| Credentials exposed to untrusted origins | 🔴 High |
| Sensitive API accessible through weak CORS | 🔴 High |

---

# Secure Configuration

Developers should:

- Use an explicit allowlist of trusted origins.
- Validate the complete origin.
- Avoid using `*` for sensitive endpoints.
- Return `Access-Control-Allow-Credentials: true` only when necessary and only for trusted origins.
- Limit allowed methods and headers to those required.

---

# CEH Exam Definition

> **Cross-Origin Resource Sharing (CORS)** is a browser mechanism that allows controlled access to resources on a different origin through HTTP response headers. Incorrect configuration can expose sensitive data by allowing untrusted origins to access protected resources.

---

# Easy Memory Trick

> [!tip]

Imagine two neighboring apartments.

- **Same-Origin Policy** = The security guard does **not** allow one apartment to enter another.
- **CORS** = The apartment owner gives the guard a guest list.
- Only visitors on the guest list are allowed inside.

---

# Related Notes

- [[Same-Origin Policy]]
- [[CSRF]]
- [[HTTP Request]]
- [[Burp Suite]]
- [[Cookie Attributes]]
- [[JWT]]
- [[OWASP Top 10 2025]]

---

# Key Takeaways

> [!summary]

- An **origin** consists of the protocol, domain, and port.
- Browsers enforce the **Same-Origin Policy** by default.
- CORS relaxes this policy through specific HTTP response headers.
- Preflight (`OPTIONS`) requests are used for certain cross-origin requests.
- Misconfigured CORS can expose sensitive data to untrusted origins.
- During testing, focus on how the application validates the `Origin` header and whether it grants cross-origin access only to trusted origins.