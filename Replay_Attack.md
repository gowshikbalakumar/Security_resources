---
title: Replay Attack
aliases:
  - Replay Attack
  - Replay Attack Prevention
tags:
  - WebSecurity
  - Authentication
  - SessionManagement
  - NetworkSecurity
  - OWASP
  - BugBounty
  - CEH
created: 2026-07-30
---

# 🔁 Replay Attack

> [!info]
> A **Replay Attack** is a cyberattack in which an attacker captures a **valid communication** (such as a login request, authentication token, API request, or payment transaction) and **retransmits (replays)** it later to trick the server into accepting it as a legitimate request.
>
> The attacker usually **does not need to decrypt or modify the message**. Simply replaying a previously valid request may be enough if the application does not verify that the request is **fresh** or **unique**.

---

# 📖 What is a Replay Attack?

During communication between a client and a server, every request contains authentication information.

Normally:

```text
User
   │
   ▼
Authenticated Request
   │
   ▼
Server
```

An attacker may capture this request.

Later,

the attacker sends **the exact same request again**.

If the server cannot detect that the request has already been used,

it may process it again.

---

# Simple Architecture

```text
User
   │
   ▼
Server
   ▲
   │
Authenticated Request
   │
Attacker Records
   │
   ▼
Later Replays Same Request
   │
   ▼
Server Accepts Again ❌
```

---

# Real-World Analogy

Imagine entering a movie theater.

You buy a ticket.

The security guard scans it and lets you in.

A dishonest person secretly photographs your valid ticket.

Later,

they show the **same ticket** to another security guard.

If the theater has no way to detect that the ticket has already been used,

the attacker gets inside.

A Replay Attack works the same way.

---

# How a Replay Attack Works

## Step 1

A legitimate user sends an authenticated request.

```text
Alice
   │
   ▼
Login Request
   │
   ▼
Server
```

---

## Step 2

An attacker captures the request.

```text
Alice

↓

Login Request

↓

Attacker Records It
```

---

## Step 3

Later,

the attacker resends **the exact same request**.

```text
Attacker

↓

Replay Request

↓

Server
```

---

## Step 4

If the server cannot detect that the request has already been used,

it accepts it.

```text
HTTP/1.1 200 OK
```

Unauthorized access or repeated actions occur.

---

# Visual Flow

```text
Legitimate User
       │
       ▼
Authenticated Request
       │
       ▼
Server
       ▲
       │
Attacker Records Request
       │
       ▼
Later Replays Same Request
       │
       ▼
Server Accepts Request Again ❌
```

---

# Example Scenario

Alice logs into a website.

The browser sends:

```http
POST /login HTTP/1.1

Authorization: Bearer eyJhb...
```

The attacker captures this request over an insecure or compromised network.

Later,

the attacker sends the same authentication token again.

If the server does not verify that the token is still valid and unique,

the attacker may gain access as Alice.

---

# Where Replay Attacks Occur

Replay attacks are commonly seen in:

- Authentication systems
- Payment systems
- REST APIs
- IoT devices
- Wireless communication
- RFID/NFC systems
- Smart card systems

---

# Common Targets

- Login requests
- Session tokens
- JWT tokens
- API keys
- Payment requests
- Password reset requests
- One-Time Password (OTP) implementations without replay protection

---

# Why is it Dangerous?

Replay attacks may allow attackers to:

- Access another user's account
- Repeat financial transactions
- Reuse authentication tokens
- Bypass authentication mechanisms
- Perform unauthorized actions

---

# Replay Attack vs Session Hijacking

| Replay Attack | Session Hijacking |
|--------------|-------------------|
| Reuses a previously captured request | Steals and uses a valid session |
| May not require stealing the session permanently | Uses an active session cookie/token |
| Focuses on replaying messages | Focuses on taking over an authenticated session |

---

# Replay Attack vs Man-in-the-Middle (MITM)

| Replay Attack | MITM Attack |
|--------------|-------------|
| Records and reuses a message later | Intercepts communication in real time |
| Usually does not modify messages | Can read, modify, or inject messages |
| Passive after capture | Active between client and server |

---

# Replay Attack vs Session Fixation

| Replay Attack | Session Fixation |
|--------------|------------------|
| Reuses captured authentication data | Forces a victim to use a known session ID |
| Happens after communication is captured | Happens before authentication |

---

# How Developers Prevent Replay Attacks

## 1️⃣ Nonces

A **nonce** is a random value used **only once**.

```text
Request

↓

Nonce = 8F72AB

↓

Server Stores It

↓

Same Nonce Again

↓

Rejected
```

---

## 2️⃣ Timestamps

Every request contains a timestamp.

Example

```text
10:30:01
```

If the request arrives too late,

the server rejects it.

---

## 3️⃣ Session IDs

Servers should invalidate old session IDs when:

- User logs out
- Session expires
- Session is regenerated

---

## 4️⃣ One-Time Tokens

Each token is valid for only one request.

Once used,

it cannot be reused.

---

## 5️⃣ Message Authentication Codes (MACs)

MACs verify:

- Message integrity
- Authentication

When freshness information (such as a nonce or timestamp) is included,

replayed messages can be detected.

---

## 6️⃣ Digital Signatures

Digital signatures combined with timestamps or nonces ensure that replayed messages are rejected.

---

## 7️⃣ HTTPS / TLS

HTTPS encrypts communication,

making it much harder for attackers to capture requests.

However,

HTTPS **alone does not completely prevent replay attacks**.

Applications must still verify request freshness.

---

# Replay Protection Flow

```text
Client
   │
   ▼
Request + Nonce + Timestamp
   │
   ▼
Server
   │
Checks:
✔ Timestamp
✔ Nonce
✔ Session
   │
   ▼
Accept or Reject
```

---

# Burp Suite Perspective

> [!warning]
> Only test applications you own or have explicit authorization to assess.

Replay testing typically involves:

1. Capture a legitimate authenticated request.
2. Send it to **Repeater**.
3. Resend the same request without modification.
4. Observe whether the application accepts or rejects the repeated request.
5. Determine whether sensitive actions can be repeated without appropriate freshness checks.

---

# Indicators of a Replay Vulnerability

- The same authenticated request succeeds multiple times.
- Payment requests can be repeated.
- API requests execute repeatedly using the same token.
- One-time tokens can be reused.
- Duplicate transactions are accepted.

---

# Impact Assessment

| Impact | Severity |
|----------|----------|
| Unauthorized Access | 🔴 High |
| Duplicate Transactions | 🔴 Critical |
| Authentication Bypass | 🔴 High |
| Financial Loss | 🔴 Critical |
| Account Misuse | 🔴 High |

---

# Secure Development Checklist

- [x] Use HTTPS/TLS
- [x] Use nonces
- [x] Validate timestamps
- [x] Issue one-time tokens
- [x] Expire session IDs properly
- [x] Detect duplicate requests
- [x] Use MACs or digital signatures with freshness information

---

# CEH Exam Definition

> **Replay Attack** is a cyberattack in which an attacker captures a legitimate authenticated communication and later retransmits it to the server in an attempt to gain unauthorized access or repeat an action. Replay attacks succeed when the system fails to verify that each request is unique and recent.

---

# Easy Memory Trick

> [!tip]

Imagine using a movie ticket.

- 🎟 Original Ticket = Legitimate Request
- 📸 Attacker Takes a Photo = Captures the Request
- 🔁 Uses the Same Ticket Again = Replay Attack
- 🚫 Ticket Scanner Marks It as Already Used = Replay Protection

If the system cannot tell that the ticket has already been used,

the attacker gets in.

---

# Related Notes

- [[Session Hijacking]]
- [[Improper Session Management]]
- [[Session Fixation]]
- [[Cookie Attributes]]
- [[JWT]]
- [[Cross-Origin Resource Sharing]]
- [[Authentication]]
- [[OWASP Top 10 2025]]

---

# Key Takeaways

> [!summary]

- Replay attacks **reuse valid communications** rather than creating new ones.
- The attacker often **does not need to decrypt or modify** the message.
- Authentication systems, APIs, payment systems, and IoT devices are common targets.
- Replay attacks differ from MITM because they involve **retransmitting** previously captured messages rather than actively modifying live communications.
- Strong replay protection includes **nonces, timestamps, one-time tokens, session management, and freshness verification**, in addition to HTTPS/TLS.