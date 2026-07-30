---
title: JSON Web Token (JWT)
aliases:
  - JWT
  - JSON Web Token
  - Bearer Token
tags:
  - WebSecurity
  - Authentication
  - API
  - JWT
  - OWASP
  - BugBounty
  - CEH
created: 2026-07-30
---

# 🔑 JSON Web Token (JWT)

> [!info]
> **JSON Web Token (JWT)** is a compact, URL-safe token used to securely transmit information between two parties. It is commonly used for **authentication** and **authorization** in web applications and APIs.

---

# 📖 What is JWT?

JWT is a **token-based authentication mechanism**.

Instead of storing session information on the server, the server issues a **signed token** to the client.

The client stores this token and sends it with every request.

The server verifies the token before granting access.

---

# Why JWT?

Traditional Authentication

```text
User
   │
Login
   │
   ▼
Server
   │
Creates Session
   │
Stores Session in Database
```

JWT Authentication

```text
User
   │
Login
   │
   ▼
Server
   │
Creates JWT
   │
Returns Token
   │
   ▼
Client Stores Token
```

No server-side session storage is required.

---

# Authentication Flow

```text
+---------+
|  User   |
+---------+
     │
     │ Login
     ▼
+------------+
|  Server    |
+------------+
     │
Creates JWT
     │
     ▼
JWT Token
     │
     ▼
+------------+
|  Browser   |
+------------+
     │
Sends JWT in every request
     │
     ▼
+------------+
|  Server    |
+------------+
Verifies Signature
     │
     ▼
Authorized
```

---

# JWT Structure

A JWT has **three parts** separated by dots (`.`).

```text
Header.Payload.Signature
```

Example

```text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJ1c2VySWQiOjEyMywicm9sZSI6InVzZXIifQ.
KxP6mM9L...
```

---

# JWT Format

```text
Header
    .
Payload
    .
Signature
```

---

# 1️⃣ Header

The header contains metadata.

Example

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

Meaning

| Field | Description |
|--------|-------------|
| alg | Signing algorithm |
| typ | Token type (JWT) |

---

# 2️⃣ Payload

The payload contains **claims**.

Example

```json
{
  "sub": "12345",
  "name": "Tuxor",
  "role": "user",
  "iat": 1753840000,
  "exp": 1753843600
}
```

Claims are **not encrypted**.

Anyone holding the token can decode them.

---

# Types of Claims

## Registered Claims

Standard JWT fields.

| Claim | Meaning |
|---------|---------|
| iss | Issuer |
| sub | Subject |
| aud | Audience |
| exp | Expiration Time |
| nbf | Not Before |
| iat | Issued At |
| jti | JWT ID |

---

## Public Claims

Application-defined claims.

Example

```json
{
  "role":"admin",
  "department":"IT"
}
```

---

## Private Claims

Custom claims between specific applications.

Example

```json
{
  "employeeId":"EMP001"
}
```

---

# 3️⃣ Signature

The signature ensures that the token has **not been modified**.

Example (conceptually)

```text
Signature =
HMACSHA256(
Base64(Header) + "." + Base64(Payload),
SecretKey
)
```

If the payload changes,

the signature becomes invalid.

---

# Complete JWT

```text
Header
↓

{
 "alg":"HS256",
 "typ":"JWT"
}

↓

Payload

{
 "user":"alice",
 "role":"user"
}

↓

Signature

Generated using Secret Key

↓

Header.Payload.Signature
```

---

# How JWT Authentication Works

## Step 1

User logs in.

```text
Username
Password
```

---

## Step 2

Server validates credentials.

---

## Step 3

Server generates JWT.

```text
JWT
↓

Header.Payload.Signature
```

---

## Step 4

Server returns JWT.

```http
HTTP/1.1 200 OK

{
 "token":"eyJhbGc..."
}
```

---

## Step 5

Client stores token.

Usually in:

- Memory
- Secure Cookies
- Local Storage *(not recommended for sensitive tokens)*

---

## Step 6

Client sends JWT.

```http
Authorization: Bearer eyJhbGc...
```

---

## Step 7

Server verifies:

- Signature
- Expiration
- Claims

If valid,

Access Granted.

---

# Authorization Header

Most APIs use

```http
Authorization: Bearer <JWT>
```

Example

```http
GET /api/profile HTTP/1.1

Authorization: Bearer eyJhbGc...
```

---

# Common Signing Algorithms

| Algorithm | Type | Description |
|------------|------|-------------|
| HS256 | Symmetric | Most common |
| HS384 | Symmetric | Stronger SHA-384 |
| HS512 | Symmetric | Stronger SHA-512 |
| RS256 | Asymmetric | RSA public/private key |
| ES256 | Asymmetric | Elliptic Curve |

---

# JWT vs Session Authentication

| JWT | Session |
|------|----------|
| Stateless | Stateful |
| No server session | Server stores session |
| Good for APIs | Good for traditional web apps |
| Sent as Bearer Token | Sent as Session Cookie |
| Easily scalable | Requires session storage |

---

# Advantages

- Stateless authentication
- Easy API integration
- Works well with mobile apps
- Cross-platform
- Supports microservices
- No session database required

---

# Disadvantages

- Difficult to revoke before expiration
- Payload is visible (not encrypted)
- Large tokens increase request size
- Secret key compromise affects all tokens
- Poor implementation can introduce vulnerabilities

---

# Common JWT Vulnerabilities

## 1️⃣ Weak Secret Key

Using predictable secrets.

Example

```text
secret
password
123456
admin
```

Attackers may guess the signing key and forge tokens.

---

## 2️⃣ Missing Signature Verification

If the server fails to verify the signature,

attackers may modify claims.

---

## 3️⃣ Accepting `alg:none`

Older implementations accepted:

```json
{
 "alg":"none"
}
```

No signature verification.

Modern libraries should reject this.

---

## 4️⃣ Information Disclosure

JWT payload is only Base64URL encoded.

Example

```json
{
 "email":"user@example.com",
 "role":"admin"
}
```

Anyone with the token can decode it.

Never store:

- Passwords
- API secrets
- Credit card numbers
- Private keys

inside JWTs.

---

## 5️⃣ Long Expiration Time

Example

```text
exp = 30 days
```

If stolen,

the attacker can use the token for a long period.

---

# JWT Security Best Practices

## Use Strong Algorithms

Recommended:

- HS256 with a strong secret
- RS256
- ES256

---

## Strong Secret Keys

Use long,

random,

cryptographically secure secrets.

---

## Short Expiration

Example

```text
15 minutes
```

Use refresh tokens for longer sessions.

---

## Validate Everything

Verify

- Signature
- Expiration (`exp`)
- Issuer (`iss`)
- Audience (`aud`)
- Not Before (`nbf`)

---

## Never Trust Payload Alone

Always verify the signature before trusting claims.

---

## Use HTTPS

Never transmit JWT over HTTP.

---

## Store Tokens Securely

Preferred:

- HttpOnly Cookies

Avoid storing sensitive tokens in Local Storage when stronger alternatives are available.

---

# Burp Suite Perspective

During an assessment:

Check:

- Weak algorithms
- Weak secrets
- Missing signature validation
- Expired token acceptance
- Sensitive data inside payload
- Missing expiration
- Missing issuer validation

> [!warning]
> Test JWT implementations only on systems you own or have explicit authorization to assess.

---

# JWT Claims Summary

| Claim | Purpose |
|---------|----------|
| iss | Issuer |
| sub | Subject |
| aud | Audience |
| exp | Expiration |
| nbf | Not Before |
| iat | Issued At |
| jti | Token ID |

---

# JWT vs Cookie

| JWT | Cookie |
|------|--------|
| Authentication Token | Browser Storage Mechanism |
| Usually in Authorization Header | Usually sent automatically |
| Stateless | Often used with sessions |
| Used heavily by APIs | Used by web applications |

---

# CEH Exam Definition

> **JSON Web Token (JWT)** is a compact, URL-safe token used to securely transmit signed claims between two parties for authentication and authorization. A JWT consists of a **Header**, **Payload**, and **Signature**, where the signature protects the integrity of the token and allows the server to verify that it has not been altered.

---

# Easy Memory Trick

> [!tip]

Think of a JWT as a **sealed ID card**.

- 🪪 Header = Card type
- 👤 Payload = User information
- 🔏 Signature = Official security seal

Anyone can **read** the information on the card,

but **only the issuer can create a valid seal**.

If someone changes the information,

the seal no longer matches,

and the server rejects the token.

---

# Related Notes

- [[Authentication]]
- [[Cookie Attributes]]
- [[Session Management]]
- [[Session Hijacking]]
- [[Replay Attack]]
- [[OAuth 2.0]]
- [[API Security]]
- [[OWASP Top 10 2025]]

---

# Key Takeaways

> [!summary]

- JWT is a **stateless authentication mechanism** widely used in APIs.
- Every JWT has **Header**, **Payload**, and **Signature**.
- The **Payload is encoded, not encrypted**, so do not store sensitive information inside it.
- The **Signature protects the token's integrity** and must always be verified by the server.
- Use **strong signing algorithms**, **short expiration times**, **HTTPS**, and **secure storage** to reduce the risk of token compromise.
```