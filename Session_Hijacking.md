
## Overview

When a user logs in to a web application, the server typically creates a **session** and assigns the browser a **session identifier (Session ID)**. This identifier is commonly stored in a cookie such as:

- `PHPSESSID`
    
- `JSESSIONID`
    
- `ASP.NET_SessionId`
    
- `connect.sid`
    

### Simple Analogy

|Component|Analogy|
|---|---|
|Username + Password|Showing an ID card once|
|Session Cookie|Temporary access badge|

As long as the session cookie remains valid, the server trusts the user without requiring the password again.

---

# Definition

> **Session Hijacking** is an attack in which an attacker obtains or abuses another user's valid session token and uses it to act as that user without knowing the password.

---

# Normal Login Flow

## Step 1 — User Authentication

### Request

```http
POST /login HTTP/1.1
Host: bank.com

username=alice&password=Secret123
```

### Server Response

```http
HTTP/1.1 200 OK
Set-Cookie: SESSIONID=abc123xyz; HttpOnly; Secure
```

---

## Step 2 — Authenticated Requests

```http
GET /account HTTP/1.1
Host: bank.com
Cookie: SESSIONID=abc123xyz
```

The server checks the session ID and associates it with the authenticated user.

**Server interpretation:**

```text
SESSIONID=abc123xyz → Alice
```

---

# Historical Real-World Example

## Public Wi‑Fi Session Theft

### Victim

- Connects to a coffee shop Wi‑Fi network.
    
- Logs into a social media website.
    

### Attacker

- Connects to the same network.
    
- Captures unencrypted HTTP traffic.
    

### Result

- The attacker extracts the victim's session cookie.
    
- The cookie is imported into the attacker's browser.
    
- The attacker becomes authenticated as the victim.
    

This technique was famously demonstrated by tools such as **Firesheep**.

> **Important:** Modern websites using **HTTPS everywhere** significantly reduce this specific attack vector.

---

# Types of Session Hijacking

---

# 1. Cookie Theft

## How It Happens

A common route is **Cross-Site Scripting (XSS)**.

### Example Payload

If the session cookie is **not** marked `HttpOnly`, JavaScript can read it.

---

## Attack Chain

<List.Item>Website contains an XSS vulnerability.</List.Item><List.Item>An administrator visits the malicious page.</List.Item><List.Item>The administrator's session cookie is stolen.</List.Item><List.Item>The attacker uses the cookie to access the admin account.</List.Item>

> **Key Point:** XSS is often considered critical because it can directly lead to session hijacking.

---

# 2. Session Fixation

This attack is frequently asked in **CEH, Security+, and web security interviews**.

## Goal

The attacker wants the victim to authenticate using a session ID that the attacker already knows.

---

## Attack Flow

### Step 1 — Attacker Obtains a Session

```http
Set-Cookie: SESSIONID=KNOWN123
```

### Step 2 — Victim Receives the Session

The attacker sends a crafted link or otherwise causes the victim to use `KNOWN123`.

### Step 3 — Victim Logs In

The application authenticates the user **without changing the session ID**.

### Step 4 — Session Becomes Authenticated

```text
KNOWN123 → Authenticated Victim Session
```

### Step 5 — Attacker Reuses the Same Session

The attacker uses `KNOWN123` and gains access to the victim's account.

---

## Root Cause

> The application does **not regenerate the session ID after successful authentication**.

---

## Proper Mitigation

Immediately create a new session after login.

### Secure Flow

```text
Before login: SESSIONID=KNOWN123
After login : SESSIONID=RANDOM_NEW_456
```

---

# 3. Predictable Session IDs

## Insecure Example

```text
SESSION-1001
SESSION-1002
SESSION-1003
```

An attacker can enumerate likely values.

---

## Secure Example

Modern frameworks generate cryptographically random tokens such as:

```text
f7a9c1e4b8d24c7f9a1e5d3c6b8f2a1d
```

These values are computationally infeasible to guess.

---

# Replay vs. Session Hijacking

This distinction is extremely important.

## Your Burp Suite Scenario

### Steps Performed

<List.Item>Log in to the application.</List.Item><List.Item>Capture an authenticated request in Burp.</List.Item><List.Item>Log out of the application.</List.Item><List.Item>Resend the old request from Repeater.</List.Item>

---

## Possible Outcomes

### Outcome 1 — Protected Data Is Returned

**Meaning**

- Logout did not invalidate the server-side session.
    
- The token is still accepted.
    

**Likely Finding**

- Broken logout
    
- Session invalidation failure
    
- Insufficient session termination
    

**Typical Severity**

Medium to High, depending on the impact.

---

### Outcome 2 — Only Cached HTML Appears

Burp may simply display the previously received response.

**This is not a vulnerability.**

---

### Outcome 3 — The Request Still Performs Actions

Examples:

- Money transfer
    
- Profile update
    
- Password change
    
- Admin operation
    

**This is a serious session management vulnerability.**

---

# Is It Session Hijacking?

### Replay

You reuse **your own** previously captured request.

### Hijacking

You use **another user's** valid session to access their account.

> Your Burp test is primarily a **session replay / broken logout** test, not automatically a session hijacking attack.

---

# Real Enterprise Example

## SaaS Admin Panel

### Scenario

- An employee logs into `admin.company.com`.
    
- The employee leaves the company.
    
- Security forces a logout.
    
- The old JWT or session remains valid for several hours.
    

### Impact

The former employee can continue accessing customer data using the retained token.

### Common Report Titles

- Insufficient Session Expiration
    
- Failure to Invalidate Session
    
- Broken Authentication
    
- Improper Session Management
    

---

# How Attackers Obtain Session Tokens

## Common Sources

### XSS Vulnerabilities

Most common web-application route to cookie theft.

### Public or Compromised Networks

Particularly when traffic is not fully protected.

### Phishing

Fake login pages capture credentials or tokens.

### Malware and Malicious Browser Extensions

Local compromise exposes browser storage.

### Server-Side Information Leaks

Examples:

- Debug endpoints
    
- Application logs
    
- Backups
    
- Misconfigured monitoring systems
    

### Shared or Unrevoked Sessions

Tokens remain active across devices or shared browsers.

---

# Cookie Security Attributes

## HttpOnly

### Without HttpOnly

JavaScript can read the cookie.

### With HttpOnly

```http
Set-Cookie: SESSIONID=abc; HttpOnly
```

- Browser sends the cookie automatically.
    
- JavaScript cannot access it.
    

**Primary protection:** XSS-based cookie theft.

---

## Secure

```http
Set-Cookie: SESSIONID=abc; Secure
```

The cookie is transmitted **only over HTTPS**.

**Primary protection:** Accidental transmission over unencrypted HTTP.

---

# Session Hijacking vs. Session Replay

### Examples

- **Replay:** You resend your own captured request.
    
- **Hijacking:** You use someone else's session cookie to access their account.
    

---
# Key Takeaway

> **Session hijacking is fundamentally a session management problem.** The attacker does not need the victim's password if they can obtain a valid session token. Strong session generation, HTTPS, `HttpOnly`, `Secure`, session regeneration after login, and immediate invalidation on logout are the core defenses against this class of attacks.