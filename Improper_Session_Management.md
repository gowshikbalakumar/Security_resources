---
title: Improper Session Management
aliases:
  - Improper Session Invalidation
  - Broken Session Management
  - Session Reuse After Logout
tags:
  - WebSecurity
  - OWASP
  - Authentication
  - SessionManagement
  - BurpSuite
  - BugBounty
  - CEH
created: 2026-07-30
---

# 🔐 Improper Session Management

> [!info]
> **Improper Session Management** is a security vulnerability where a web application fails to correctly create, validate, rotate, or destroy user sessions.
>
> One of the most common examples is **Improper Session Invalidation**, where a session remains valid even after the user logs out.

---

# 📖 What is a Session?

When a user logs in, the server creates a **session** to remember that the user has been authenticated.

Instead of asking for the username and password on every request, the browser sends a **Session ID** (usually stored in a cookie).

Example:

```http
POST /login HTTP/1.1
```

Server Response:

```http
Set-Cookie: PHPSESSID=ABC123
```

Every future request contains:

```http
GET /dashboard HTTP/1.1
Cookie: PHPSESSID=ABC123
```

The server checks the Session ID and grants access to protected resources.

---

# 🎓 Real-World Analogy

Imagine entering your college.

- Login = Security gives you an ID card.
- Session Cookie = Your ID card.
- Every time you enter a classroom, you show your ID card.
- Logout = Security should cancel your ID card.

If the ID card still works after cancellation, anyone with that card can continue entering the college.

That is exactly what **Improper Session Invalidation** looks like.

---

# ✅ Secure Session Flow

```text
User Login
      │
      ▼
Server Creates Session
      │
      ▼
Browser Stores Session Cookie
      │
      ▼
User Accesses Protected Pages
      │
      ▼
User Clicks Logout
      │
      ▼
Server Deletes Session
      │
      ▼
Old Session Cookie
      │
      ▼
401 Unauthorized ✅
```

---

# ❌ Vulnerable Session Flow

```text
User Login
      │
      ▼
Server Creates Session
      │
      ▼
Browser Stores Session Cookie
      │
      ▼
User Accesses Protected Pages
      │
      ▼
User Clicks Logout
      │
      ▼
Server DOES NOT Delete Session
      │
      ▼
Old Session Cookie
      │
      ▼
200 OK ❌
Protected Page Still Accessible
```

---

# 🧪 Burp Suite Testing Workflow

## Step 1 – Login

Login to the application.

Example:

```text
Username: alice
Password: ********
```

The server returns:

```http
Set-Cookie: PHPSESSID=ABC123
```

---

## Step 2 – Capture an Authenticated Request

Enable **Burp Intercept**.

Capture a request such as:

```http
GET /profile HTTP/1.1
Cookie: PHPSESSID=ABC123
```

Send it to **Repeater**.

---

## Step 3 – Logout

Logout from the website normally.

A secure application should invalidate the session.

---

## Step 4 – Replay the Request

Send the previously captured request from Burp Repeater.

```http
GET /profile HTTP/1.1
Cookie: PHPSESSID=ABC123
```

---

# 🔍 Expected Results

## ✅ Secure Application

```http
HTTP/1.1 401 Unauthorized
```

or

```http
HTTP/1.1 302 Found
Location: /login
```

The session has been invalidated.

---

## ❌ Vulnerable Application

```http
HTTP/1.1 200 OK
```

```html
<h1>Welcome Alice</h1>
```

The old session is still valid after logout.

---

# 🚨 Why Is This Dangerous?

If an attacker steals a user's session cookie:

```text
PHPSESSID=ABC123
```

they can continue using that session even after the legitimate user logs out.

Possible consequences include:

- Account takeover
- Viewing personal information
- Performing unauthorized actions
- Changing account settings
- Financial fraud
- Bypassing logout protection

---

# 🔄 Session Replay vs Session Hijacking

| Concept | Description |
|----------|-------------|
| **Session Replay** | Reusing a previously captured authenticated request. |
| **Session Hijacking** | Stealing another user's valid session and using it. |
| **Improper Session Invalidation** | The server fails to invalidate the session after logout. |

> [!note]
> During testing with Burp Repeater, you are performing **Session Replay** to verify whether the application suffers from **Improper Session Invalidation**.

---

# 🍪 Cookie Manipulation

Sometimes testers modify or remove cookies.

Examples:

Original:

```http
Cookie: role=user
```

Modified:

```http
Cookie: role=admin
```

If the server trusts the modified cookie:

- Cookie Poisoning
- Privilege Escalation

---

If deleting a security-related cookie still allows access:

```http
Cookie:
```

but the server still responds:

```http
HTTP/1.1 200 OK
```

this may indicate:

- Broken Authentication
- Weak Session Validation
- Improper Session Management

---

# 🧠 Common Vulnerability Names

| Situation | Vulnerability |
|------------|---------------|
| Session works after logout | Improper Session Invalidation |
| Logout doesn't destroy session | Broken Session Management |
| Old request still succeeds | Session Replay Vulnerability |
| Modified cookie is trusted | Cookie Poisoning |
| Cookie changes user role | Privilege Escalation |
| JWT modified and accepted | JWT Signature Validation Bypass |

---

# 📋 Bug Bounty Report Title

Good report titles include:

- Improper Session Invalidation Allows Access After Logout
- Authenticated Session Remains Valid After Logout
- Logout Does Not Invalidate Server-Side Session

---

# ✅ Verification Checklist

- [ ] Login successfully
- [ ] Capture an authenticated request
- [ ] Send request to Burp Repeater
- [ ] Logout normally
- [ ] Replay the captured request
- [ ] Check HTTP status code
- [ ] Verify whether protected content is returned
- [ ] Confirm the old session cookie is still accepted

---

# 🛡 Developer Mitigations

A secure application should:

- Destroy the server-side session on logout.
- Expire the session cookie.
- Rotate Session IDs after login.
- Implement session expiration.
- Invalidate sessions across devices if required.
- Prevent reuse of old Session IDs.

---

## PHP Example

```php
<?php
session_start();

$_SESSION = [];

session_destroy();
?>
```

---

# 🎯 Key Indicators

| Secure | Vulnerable |
|---------|------------|
| 401 Unauthorized after logout | 200 OK after logout |
| Redirects to login | Protected page still loads |
| Session destroyed | Session still active |
| New login required | Old cookie continues working |

---

# 💡 CEH Exam Definition

> **Improper Session Invalidation** is a vulnerability where a web application fails to terminate a user's server-side session after logout, allowing previously issued session identifiers (cookies or tokens) to continue accessing authenticated resources.

---

# 🔗 Related Notes

- [[Session Hijacking]]
- [[Session Fixation]]
- [[Cookie Attributes]]
- [[Cookie Poisoning]]
- [[JWT]]
- [[Broken Authentication]]
- [[RBAC]]
- [[Privilege Escalation]]
- [[OWASP Top 10 2025]]

---

# 📝 Key Takeaways

> [!summary]

- A session is created after successful authentication.
- The Session ID is stored in a cookie.
- Logging out **must** invalidate the server-side session.
- If an old session still works after logout, the application is vulnerable to **Improper Session Invalidation**.
- Burp Suite Repeater is commonly used to verify this behavior.
- This issue falls under **Broken Authentication / Session Management** in OWASP.