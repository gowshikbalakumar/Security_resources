# Privilege Escalation

**Tags:** #WebSecurity #PrivilegeEscalation #OWASP #Authorization #BugBounty

---

# What is Privilege Escalation?

**Privilege Escalation** is a vulnerability where an attacker gains **more permissions than they are supposed to have**.

In web applications, it occurs when a user can perform actions or access resources reserved for other users or administrators.

> [!NOTE]
> Authentication answers **"Who are you?"**  
> Authorization answers **"What are you allowed to do?"**  
> Privilege Escalation happens when **authorization fails**.

---

# Types of Privilege Escalation

## 1. Vertical Privilege Escalation

A low-privileged user gains **higher privileges**.

### Example

```
User
   │
   ▼
Admin Panel ❌

Attacker modifies request

   │
   ▼
Admin Panel ✅
```

Example Request:

```http
GET /admin/dashboard
```

A normal user should receive:

```http
403 Forbidden
```

A vulnerable application returns:

```http
200 OK
```

---

## 2. Horizontal Privilege Escalation

A user accesses **another user's data** without becoming an admin.

Example:

```
User A

/profile?id=100

        │

Change ID

        ▼

/profile?id=101

        ▼

User B's Profile
```

This is also known as **IDOR (Insecure Direct Object Reference)**.

---

# Common Causes

- Missing authorization checks
- Insecure Direct Object References (IDOR)
- Broken Object Level Authorization (BOLA)
- Misconfigured permissions
- Trusting client-side input
- Unpatched software
- Credential theft or reuse

---

# Impact

- Unauthorized access
- Account takeover
- Admin access
- Data theft
- Financial fraud
- System compromise

---

# Prevention

- Enforce **server-side authorization**
- Follow the **Principle of Least Privilege**
- Implement **Role-Based Access Control (RBAC)**
- Validate object ownership
- Never trust client-side controls
- Regularly audit permissions
- Enable MFA for privileged accounts

---

# Testing Checklist

- Try accessing **admin pages** as a normal user.
- Modify user IDs in requests.
- Change API parameters (`userId`, `role`, `accountId`).
- Test hidden endpoints.
- Verify that every sensitive action performs a server-side authorization check.

---

# Key Terms

| Term | Meaning |
|------|---------|
| Privilege Escalation | Gaining permissions beyond those intended |
| Vertical Escalation | User becomes a higher-privileged user (e.g., Admin) |
| Horizontal Escalation | User accesses another user's resources |
| RBAC | Role-Based Access Control |
| Least Privilege | Grant only the minimum permissions required |

---

# Summary

Privilege Escalation occurs when an application **fails to enforce authorization**, allowing users to perform actions or access resources beyond their intended permissions.

- **Vertical Privilege Escalation:** User → Admin
- **Horizontal Privilege Escalation:** User A → User B's data

> [!TIP]
> In **OWASP Top 10:2025**, Privilege Escalation is part of **A01 – Broken Access Control** and is one of the most common vulnerabilities found in bug bounty programs.