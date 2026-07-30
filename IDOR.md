# Insecure Direct Object Reference (IDOR)

**Tags:** #WebSecurity #IDOR #BOLA #OWASP #BugBounty #Authorization

---

# What is IDOR?

**Insecure Direct Object Reference (IDOR)** is a **Broken Access Control** vulnerability where an application allows a user to **access or modify another user's resource** by changing an object identifier (ID, UUID, filename, etc.) **without proper authorization checks**.

> [!NOTE]
> **OWASP Top 10:2025:** **A01 – Broken Access Control**

---

# Simple Definition

Imagine you have the key to **Room 101**.

Instead of checking **which room belongs to you**, the security guard only checks that **you have a key**.

You ask for **Room 102**, and the guard opens it.

➡️ **That's IDOR.**

---

# Authentication vs Authorization

| Authentication | Authorization |
|---------------|---------------|
| Who are you? | What are you allowed to access? |
| Login | Permission Check |

> [!WARNING]
> IDOR happens because **authorization is missing**, **not authentication**.

---

# How IDOR Works

```
Alice logs in
      │
      ▼
GET /profile?id=101
      │
      ▼
Alice's Profile ✅

Attacker changes

GET /profile?id=102
      │
      ▼
Bob's Profile ❌
```

The server checks:

✅ User is logged in

But forgets to check:

❌ Does Profile 102 belong to Alice?

---

# Common Places to Find IDOR

- URL Parameters (`?id=100`)
- REST APIs (`/api/users/100`)
- JSON Body (`"user_id":100`)
- Hidden Form Fields
- File Downloads
- Mobile APIs
- GraphQL Object IDs

---

# Examples

## User Profile

```http
GET /profile?id=1
```

Change to:

```http
GET /profile?id=2
```

If another user's profile is returned → **IDOR**

---

## Bank Account

```http
GET /account?id=500
```

Change:

```http
GET /account?id=501
```

Another customer's balance is displayed.

---

## File Download

```http
/download?file=invoice101.pdf
```

Change:

```http
/download?file=invoice102.pdf
```

Another customer's invoice is downloaded.

---

## Delete Request

```http
POST /delete

id=45
```

Change:

```http
id=46
```

Deletes another user's resource.

---

# Types of IDOR

## Read IDOR

View another user's information.

Example:

```http
GET /profile?id=25
```

**Impact**

- Data Disclosure
- Privacy Violation

---

## Write IDOR

Modify another user's data.

Example:

```http
POST /changeEmail

user=25
email=hacker@example.com
```

**Impact**

- Account Manipulation
- Account Takeover

---

## Delete IDOR

Delete another user's resource.

Example:

```http
POST /delete?id=25
```

---

# IDOR in APIs (BOLA)

APIs often call IDOR **Broken Object Level Authorization (BOLA).**

Example:

```http
GET /api/users/100/orders
```

Change:

```http
GET /api/users/101/orders
```

If another user's orders are returned → **BOLA**

---

# Common Parameters to Test

- id
- user_id
- uid
- account_id
- order_id
- invoice_id
- customer_id
- filename
- UUID
- slug

---

# Impact

- Sensitive Data Disclosure
- Data Modification
- Data Deletion
- Financial Fraud
- Privacy Violations
- Account Takeover (some cases)
- Privilege Escalation

---

# Prevention

### ✅ Server-Side Authorization

Always verify ownership.

Example:

```sql
SELECT *
FROM orders
WHERE id = ?
AND owner_id = current_user;
```

---

### ✅ Never Trust Client IDs

Never trust values like:

```json
"user_id":25
```

Determine the user's identity from the authenticated session or token.

---

### ✅ Use UUIDs

Instead of:

```
1
2
3
```

Use:

```
550e8400-e29b-41d4-a716-446655440000
```

> [!TIP]
> UUIDs reduce guessability but **do not replace authorization checks**.

---

### ✅ Implement RBAC

Ensure users only access resources permitted by their role.

---

# Testing Checklist

- Does the request contain an object ID?
- Can the ID be modified?
- Can another user's data be viewed?
- Can another user's data be edited?
- Can another user's resource be deleted?
- Does the server verify ownership?

---

# OWASP Mapping

| Vulnerability | OWASP Top 10:2025 |
|--------------|-------------------|
| IDOR | A01 – Broken Access Control |
| BOLA | A01 – Broken Access Control |
| Privilege Escalation | A01 – Broken Access Control |

---

# Key Terms

| Term | Meaning |
|------|---------|
| IDOR | Insecure Direct Object Reference |
| BOLA | Broken Object Level Authorization |
| Object | Resource such as a user, order, invoice, or file |
| Authorization | Verifies whether a user can access a resource |
| RBAC | Role-Based Access Control |

---

# Summary

IDOR is a **Broken Access Control** vulnerability where an application trusts **user-supplied object identifiers** without verifying ownership or permissions. Simply being authenticated **does not** mean a user should be able to access every object.

> [!TIP]
> **Remember:** Authentication proves **who you are**. Authorization proves **what you are allowed to access**. IDOR exists when the server skips the authorization check.