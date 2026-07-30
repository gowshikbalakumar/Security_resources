# Role-Based Access Control (RBAC)

**Tags:** #WebSecurity #RBAC #Authorization #OWASP #BugBounty

---

# What is RBAC?

**Role-Based Access Control (RBAC)** is an **authorization model** where permissions are assigned to **roles**, and users inherit those permissions through their assigned role.

> [!NOTE]
> **Authentication:** Who are you?  
> **Authorization (RBAC):** What are you allowed to do?

---

# Simple Concept

Instead of assigning permissions to every user:

```
Alice → Read
Alice → Write
Alice → Delete
```

Assign permissions to a role:

```
Admin
 ├── Read
 ├── Write
 └── Delete

Alice → Admin
```

---

# Real-World Example

| Role | Permissions |
|------|-------------|
| Doctor | View records, prescribe medicine |
| Nurse | View records, update vitals |
| Receptionist | Book appointments |
| Patient | View own records only |

The system checks the **user's role** before allowing an action.

---

# RBAC Workflow

```text
User Login
     │
     ▼
Authentication
     │
     ▼
Retrieve User Role
     │
     ▼
Check Permissions
     │
 ┌───┴───┐
 │       │
Allow   Deny
```

---

# Example

### User Roles

| User | Role |
|------|------|
| Alice | Admin |
| Bob | Manager |
| Charlie | User |

### Request

```http
GET /admin/dashboard
```

Server Check:

```javascript
if (user.role == "Admin")
    Allow;
else
    Deny;
```

---

# Common RBAC Vulnerabilities

## 1. Privilege Escalation

A normal user gains admin access.

```
User
 │
 ▼
/admin/dashboard
 │
 ▼
Access Granted ❌
```

---

## 2. Broken Access Control

The admin button is hidden in the UI, but the backend never checks permissions.

```
Hidden Button
      │
User manually visits
/admin
      │
      ▼
Access Granted ❌
```

---

## 3. IDOR / BOLA

```
GET /api/users/5
```

Attacker changes:

```
GET /api/users/6
```

If ownership isn't verified, another user's data is exposed.

---

## 4. Role Manipulation

Client sends:

```json
{
  "username":"john",
  "role":"Admin"
}
```

If the server trusts this value, the attacker becomes an admin.

---

## 5. JWT Role Tampering

Original Payload:

```json
{
  "username":"john",
  "role":"user"
}
```

Modified Payload:

```json
{
  "username":"john",
  "role":"admin"
}
```

If the JWT signature isn't properly verified, the attacker gains elevated privileges.

---

## 6. Unprotected APIs

```
POST /api/admin/deleteUser
```

Even if the UI hides the admin menu, the API must still verify authorization.

---

# How to Test RBAC

- Create accounts with different roles.
- Capture requests using Burp Suite.
- Replay admin requests as a normal user.
- Modify IDs, endpoints, and HTTP methods.
- Test APIs directly.
- Verify server-side authorization.
- Attempt role manipulation.
- Check JWTs, cookies, and request bodies for role information.

---

# Common Developer Mistakes

- Trusting client-side role values.
- Checking authentication but not authorization.
- Missing server-side permission checks.
- Exposing admin APIs.
- Relying on hidden UI elements.
- Forgetting to protect REST or GraphQL APIs.

---

# Prevention

- Enforce authorization on every request.
- Follow the Principle of Least Privilege.
- Store roles and permissions on the server.
- Validate JWT signatures.
- Log authorization failures.
- Regularly review access controls.

---

# RBAC vs ABAC

| Feature | RBAC | ABAC |
|---------|------|------|
| Based On | Roles | Attributes |
| Example | Admin, User | Department, Time, Location |
| Complexity | Low | High |
| Best For | Most Applications | Large Enterprises |

---

# OWASP Mapping

| Vulnerability | OWASP Top 10:2025 |
|--------------|-------------------|
| Broken RBAC | A01 – Broken Access Control |
| Privilege Escalation | A01 – Broken Access Control |
| IDOR / BOLA | A01 – Broken Access Control |
| Forced Browsing | A01 – Broken Access Control |
| Role Manipulation | A01 – Broken Access Control |

---

# Key Terms

| Term | Meaning |
|------|---------|
| RBAC | Role-Based Access Control |
| Role | Collection of permissions |
| Permission | Allowed action |
| Authorization | Verifies what a user can do |
| Least Privilege | Grant only the minimum required permissions |

---

# Summary

RBAC is an authorization model that assigns **permissions to roles** instead of individual users. Users inherit permissions through their assigned role. Most RBAC vulnerabilities occur because applications **fail to enforce server-side authorization**, allowing attackers to access resources or perform actions beyond their intended permissions.

> [!TIP]
> **Authentication verifies identity. Authorization verifies permissions.**  
> In bug bounty hunting, always test whether a lower-privileged user can access or perform actions reserved for higher-privileged roles.