**Tags:** #web-security #session-fixation #owasp #ceh #bugbounty #session-management

---

## Overview

**Session Fixation** is a web security vulnerability in which an attacker forces or tricks a victim into using a session ID that the attacker already knows. If the application does not generate a new session ID after successful authentication, the attacker can reuse the same session ID to access the victim’s authenticated session.

> **One-line Definition:** The session is fixed before login, and the server fails to generate a new session after successful authentication.

---

## Simple Analogy

Imagine a hotel:

<List.Item>The receptionist gives **Room Key #123**.</List.Item><List.Item>An attacker secretly obtains a copy of **Key #123**.</List.Item><List.Item>The victim checks into the hotel and continues using **Room #123**.</List.Item><List.Item>Because the key was never changed, the attacker can also enter the room.</List.Item>

|Real World|Web Application|
|---|---|
|Room Key|Session ID|
|Hotel Room|User Session|
|Guest|Authenticated User|
|Attacker with copied key|Attacker with known session ID|

---
# Vulnerable Session Fixation Flow

## Attack Steps

### Step 1 — Attacker Obtains a Valid Session ID

The attacker visits the application and receives:

---

### Step 2 — Attacker Sends the Session to the Victim

Example malicious link:

Possible delivery methods:

- Phishing email
    
- Malicious website
    
- Cookie injection
    
- HTTP response splitting
    
- Open redirect abuse
    

---

### Step 3 — Victim Uses the Fixed Session

The victim opens the link and the browser starts using:

---

### Step 4 — Victim Logs In

The server authenticates the victim **but does not regenerate the session ID**.

Session remains:

---

### Step 5 — Attacker Reuses the Same Session

The attacker sends requests with:

The server treats the attacker as the authenticated victim.

---

# Root Cause

> **The application accepts an existing session ID and keeps it unchanged after authentication.**

This is a **session management flaw** caused by improper session handling during the authentication process.

Common causes:

- No session regeneration after login
    
- Reusing anonymous sessions
    
- Trusting client-supplied session identifiers
    
- Poor session lifecycle management
    

---

# Real HTTP Example

## Before Login

Server response:

---

## User Logs In

<CodeBlock language="http" content="POST /login HTTP/1.1  
Host: vulnerable.com  
Cookie: PHPSESSID=FIXED999

username=alice&password=Password123"/>

---

## Vulnerable Response

⚠️ **Notice:** There is **no `Set-Cookie` header** issuing a new session.

The session ID is still:

This confirms the vulnerability.

---

# Session Fixation vs Session Hijacking

### Memory Trick

- **Fixation** = _Fix the session first_
    
- **Hijacking** = _Hijack an existing session_
    

---

# Impact Assessment

## 1. Account Takeover

The attacker gains access to the victim’s account without knowing the password.

**Example:**

- Victim logs into a banking portal.
    
- Attacker uses the same fixed session.
    
- Attacker accesses the authenticated account.
    

---

## 2. Unauthorized Actions

The attacker can perform any action available to the victim.

Possible actions:

- Change password
    
- Change email address
    
- Transfer money
    
- Delete data
    
- Access private files
    
- Create API keys
    
- Download sensitive reports
    

---

## 3. Privilege Escalation

If an administrator logs in using the fixed session, the attacker inherits administrative privileges.

This is often the **highest severity scenario**.

---

## 4. Authentication Bypass

Even if the login mechanism is secure (MFA, strong passwords, rate limiting), the attacker can still access the authenticated session because the **session itself is compromised**.

---

# Severity

Typical bug bounty / CVSS assessment:

|Scenario|Severity|
|---|---|
|Normal user account takeover|**High**|
|Access to sensitive personal data|**High**|
|Financial transactions possible|**High / Critical**|
|Administrator account takeover|**Critical**|
|Full application compromise|**Critical**|

---

# Detection & Testing Methodology

## Safe Testing Workflow (Burp Suite)

<List.Item>Open the login page.</List.Item><List.Item>Capture the request in **Burp Proxy**.</List.Item><List.Item>Note the session cookie value.</List.Item><List.Item>Send the request to **Repeater**.</List.Item><List.Item>Keep the same cookie.</List.Item><List.Item>Log in through the browser.</List.Item><List.Item>Observe whether the session cookie changes after authentication.</List.Item>

---

## Vulnerable Indicator

### Before Login

### After Login

❌ **Same value = Potential Session Fixation**

---

## Secure Indicator

### Before Login

### After Login

✅ **New session generated = Secure behavior**

---

# Important Clarification (Your Earlier Burp Observation)

You observed:

> Capture request → Log out → Resend old request → Page still renders.

That behavior is **not classic Session Fixation**.

It is more closely related to:

- Improper session invalidation
    
- Session persistence after logout
    
- Session hijacking
    
- Broken access control
    
- Server-side cache issues
    

### Why?

Classic fixation specifically requires:

> **Attacker knows the session BEFORE the victim logs in.**

If the session was already authenticated and remains usable after logout, the issue is usually **improper logout/session invalidation**, not fixation.

---

# Mitigation Strategies

## 1. Regenerate Session ID After Login (Most Important)

### PHP

### Java (Servlet)

### ASP.NET

ASP.NET automatically issues a new authentication ticket, but developers should ensure session renewal occurs during authentication and privilege changes.

---

## 2. Destroy the Old Session

Invalidate any pre-authentication session.

---

## 3. Use Secure Cookie Attributes

### Why These Matter

---

## 4. Enforce HTTPS Everywhere

Without HTTPS, attackers may inject or manipulate cookies on the network.

Recommended controls:

- HTTPS
    
- HSTS
    
- Redirect HTTP → HTTPS
    
- Disable mixed content
    

---

## 5. Invalidate Session on Logout

Also:

- Remove the cookie from the browser.
    
- Expire the cookie immediately.
    
- Clear server-side session storage.
    

---

## 6. Rotate Session After Privilege Changes

Generate a new session whenever:

- User logs in
    
- User completes MFA
    
- User becomes an administrator
    
- Password is changed
    
- Email is changed
    
- Recovery flow is completed
    

---

# Secure Authentication Lifecycle

## Golden Rule

> **An anonymous session ID must never become the authenticated session ID.**

---

# Indicators of a Well-Implemented Application

A secure application should:

<List.Item>Issue a new session after login.</List.Item>
<List.Item>Invalidate old sessions.</List.Item>
<List.Item>Use secure cookie flags.</List.Item>
<List.Item>Enforce HTTPS.</List.Item>
<List.Item>Expire sessions on logout.</List.Item>
<List.Item>Rotate sessions during privilege changes.</List.Item>
<List.Item>Maintain short session lifetimes.</List.Item>

---

# CEH / Interview Quick Answer

## What is Session Fixation?

**Session Fixation is a vulnerability where an attacker forces a victim to use a known session ID before authentication. If the application does not regenerate the session ID after login, the attacker can reuse that session ID to access the victim’s authenticated session.**

---

## Impact

- Account takeover
    
- Unauthorized access
    
- Privilege escalation
    
- Sensitive data exposure
    
- Administrative compromise
    

---

## Primary Solution

> **Regenerate the session ID immediately after successful authentication and invalidate the old session.**

This is the most important defense against **Session Fixation attacks**.

---

# Key Takeaway

## Remember This in One Sentence

> **Session Fixation occurs when a web application allows a user to log in using a pre-existing session ID and fails to generate a new session after authentication, enabling an attacker who knows that session ID to take over the authenticated session.**