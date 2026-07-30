# Cross-Site Request Forgery (CSRF)

**Tags:** #WebSecurity #CSRF #OWASP #Authentication #BugBounty

---

# What is CSRF?

**Cross-Site Request Forgery (CSRF)** is an attack that tricks a **logged-in user's browser** into sending an unwanted request to a trusted website where the user is already authenticated.

Since the browser automatically includes the user's **session cookie**, the server believes the request came from the legitimate user.

> [!NOTE]
> CSRF **forces actions** on behalf of the victim but **cannot read the server's response**.

---

# Simple Example

```
Victim logs into bank.com
        │
        ▼
Browser stores session cookie
        │
        ▼
Victim visits attacker.com
        │
        ▼
Attacker silently sends request
        │
        ▼
Browser automatically includes cookie
        │
        ▼
Bank performs the action
```

---

# Why Does CSRF Work?

Browsers automatically send:

- Session Cookies
- Authentication Cookies
- Remember-Me Cookies

The server trusts these cookies and processes the request as if it came from the user.

---

# Requirements for CSRF

A successful CSRF attack usually requires:

- Victim is logged in.
- Authentication uses cookies.
- No CSRF protection exists.
- Victim visits a malicious website.

---

# Common Targets

- Change Password
- Change Email
- Transfer Money
- Update Profile
- Delete Account
- Add Administrator
- Purchase Products

---

# Example Attack

**Victim Request**

```http
POST /transfer
Cookie: SESSIONID=abc123

amount=1000
account=987654321
```

**Malicious HTML**

```html
<form action="https://bank.com/transfer" method="POST">
    <input type="hidden" name="amount" value="1000">
    <input type="hidden" name="account" value="attacker">
</form>

<script>
document.forms[0].submit();
</script>
```

When the victim opens the page, the browser automatically sends the request with the victim's session cookie.

---

# Types of CSRF

### GET-Based CSRF

```http
GET /delete?id=15
```

Example:

```html
<img src="https://example.com/delete?id=15">
```

> [!WARNING]
> Never use **GET** requests for actions that modify data.

---

### POST-Based CSRF

A hidden HTML form automatically submits sensitive requests.

---

### AJAX-Based CSRF

Possible when applications rely only on cookies and have weak CORS or cookie protections. Modern browsers reduce this risk using **SameSite cookies** and proper **CORS** policies.

---

# Impact

- Financial Loss
- Password Changes
- Account Modification
- Unauthorized Purchases
- Data Deletion
- Privilege Abuse

---

# CSRF vs XSS

| Feature | CSRF | XSS |
|---------|------|-----|
| Requires Victim Login | ✅ | ❌ |
| Executes JavaScript | ❌ | ✅ |
| Uses Victim Session | ✅ | ✅ |
| Reads Response | ❌ | ✅ |
| Main Goal | Force Actions | Execute Malicious Scripts |

---

# Prevention

### ✅ CSRF Tokens

```html
<input type="hidden"
name="csrf_token"
value="8fd7ab92391d">
```

Reject requests with:

- Missing Token
- Invalid Token
- Expired Token

---

### ✅ SameSite Cookies

```http
Set-Cookie:
SESSIONID=abc123;
Secure;
HttpOnly;
SameSite=Strict
```

| Value | Description |
|--------|-------------|
| Strict | Strongest protection |
| Lax | Good balance of security and usability |
| None | Allows cross-site cookies (must use Secure) |

---

### Other Defenses

- Use unpredictable CSRF tokens.
- Validate `Origin` or `Referer` headers.
- Avoid state-changing **GET** requests.
- Require re-authentication for sensitive actions.
- Enable MFA for critical operations.
- Use custom headers for AJAX APIs.

---

# Testing for CSRF

1. Log in to the application.
2. Capture a state-changing request using Burp Suite.
3. Check for a CSRF token.
4. Remove or modify the token.
5. Create a PoC HTML page.
6. Open the PoC while logged in.
7. Verify whether the action succeeds.

> [!TIP]
> If a sensitive action works **without a valid CSRF token**, the application may be vulnerable.

---

# Key Terms

| Term | Meaning |
|------|---------|
| CSRF | Forces a victim's browser to perform unwanted actions |
| CSRF Token | Random value that verifies legitimate requests |
| SameSite | Cookie attribute that limits cross-site requests |
| Origin Header | Indicates where the request originated |
| Referer Header | Shows the previous page URL |

---

# Summary

CSRF is an attack that **forces a logged-in user's browser** to perform unintended actions on a trusted website by abusing automatically sent cookies. The most effective defenses are **CSRF tokens**, **SameSite cookies**, **Origin/Referer validation**, and avoiding state-changing **GET** requests.

> [!TIP]
> **Remember:** CSRF **forces actions**, while **XSS steals data or executes scripts**.