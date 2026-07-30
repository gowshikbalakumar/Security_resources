---
title: Weak Password Policy
aliases:
  - Weak Password Policy
  - Weak Passwords
  - Password Security
tags:
  - WebSecurity
  - Authentication
  - PasswordSecurity
  - OWASP
  - BugBounty
  - CEH
created: 2026-07-30
---

# 🔑 Weak Password Policy

> [!info]
> A **Weak Password Policy** is a security misconfiguration where an application allows users to create **easy-to-guess, predictable, or insecure passwords**.
>
> Weak passwords significantly increase the risk of account compromise through password-based attacks.

---

# 📖 What is a Weak Password Policy?

A password policy defines the rules users must follow when creating passwords.

A **weak password policy** means the application **does not enforce strong password requirements**, allowing users to choose passwords that attackers can easily guess.

---

# 🏗 Authentication Flow

```text
User Creates Password
        │
        ▼
Application Validates Password
        │
        ▼
Weak Policy?
        │
   ┌────┴────┐
   │         │
 Yes        No
   │         │
Weak      Strong
Password  Password
   │         │
Higher     Better
Risk      Security
```

---

# 🎓 Real-World Analogy

Imagine your ATM PIN.

Would you choose:

```text
1234
```

or

```text
0000
```

Probably not.

But many websites still allow passwords like:

```text
password
123456
admin
qwerty
welcome
```

If a website accepts these passwords,

it has a **Weak Password Policy**.

---

# Example

Registration Form

```text
Username

john

Password

123456
```

Website Response

```text
✔ Account Created Successfully
```

The application accepted an extremely weak password.

---

# What Makes a Password Weak?

Common weak passwords include:

```text
123456
password
password123
admin
admin123
welcome
qwerty
111111
abc123
letmein
```

Predictable passwords are also weak.

Example

Username

```text
john
```

Password

```text
john123
```

---

# Why Is This Dangerous?

Attackers often don't need to exploit the server.

Instead,

they simply **guess common passwords**.

Example

```text
Username

john@gmail.com
```

Possible guesses

```text
123456
password
john123
john@123
welcome
```

Eventually,

one may work.

---

# Common Password Attacks

## 1️⃣ Brute Force Attack

Attempts **every possible password combination**.

Example

```text
aaaaaa
aaaaab
aaaaac
...
zzzzzz
```

Characteristics

- Tries every combination
- Slow
- Guaranteed if no limits exist

---

## 2️⃣ Dictionary Attack

Uses a list of common passwords.

Example

```text
password
123456
football
india123
admin123
```

Characteristics

- Much faster
- Uses real passwords people commonly choose

---

## 3️⃣ Credential Stuffing

Uses usernames and passwords leaked from previous data breaches.

Example

```text
Facebook

john@gmail.com
Password

hello123
```

Attacker tries the same credentials on

- Netflix
- Amazon
- Instagram
- GitHub

If the user reused the password,

the attacker may log in successfully.

---

## 4️⃣ Password Spraying

Instead of attacking one account with many passwords,

the attacker tries **one common password against many accounts**.

Example

```text
john      Password1!
alice     Password1!
bob       Password1!
charlie   Password1!
```

Benefits for attackers

- Avoids account lockouts
- Exploits common passwords

---

# Password Attack Comparison

| Attack | Description |
|---------|-------------|
| Brute Force | Tries every possible password |
| Dictionary | Uses common password lists |
| Credential Stuffing | Uses leaked credentials |
| Password Spraying | Uses one password against many users |

---

# Real-World Example

Dropbox experienced account compromises over the years because many users **reused passwords that had already been exposed in other breaches**.

This demonstrates why:

- Weak passwords
- Password reuse

are both dangerous.

---

# Tools Used by Security Professionals

> [!warning]
> These tools should only be used during authorized security assessments.

Common tools include:

- Hydra
- Burp Suite Intruder
- Medusa
- Ncrack
- Hashcat (offline password cracking)
- John the Ripper

These tools automate password testing and auditing.

---

# Example Login Flow

```http
POST /login HTTP/1.1

username=admin
password=admin
```

Response

```text
Invalid Credentials
```

Another attempt

```http
POST /login HTTP/1.1

username=admin
password=Password123
```

Eventually

```http
POST /login HTTP/1.1

username=admin
password=Welcome123
```

Response

```text
Login Successful
```

The account is compromised because the password was weak.

---

# Signs of a Weak Password Policy

During a security assessment,

check whether the application:

- Accepts passwords shorter than **8 characters**
- Accepts passwords like `12345678` or `password`
- Allows passwords containing only letters
- Allows passwords containing only numbers
- Allows passwords matching the username or email
- Does not check against common or breached password lists
- Allows unlimited login attempts
- Allows reuse of old passwords
- Does not encourage or require MFA for sensitive accounts

> [!note]
> Allowing weak passwords is often considered a **security weakness** rather than a high-severity vulnerability by itself unless it creates a realistic exploitation path or violates the application's stated security requirements.

---

# Strong Password Policy

A secure password policy should:

- Require **12–16 characters** minimum
- Encourage long passphrases
- Block common passwords
- Block breached passwords
- Prevent passwords matching usernames or email addresses
- Prevent password reuse
- Support password managers
- Encourage or require MFA for sensitive accounts

---

# Strong Password Examples

Good passphrases

```text
Coffee!River-Train2026
```

```text
correct-horse-battery-staple
```

These are much harder to guess than

```text
Password123
```

---

# Weak vs Strong Password

| Weak Password | Strong Password |
|---------------|-----------------|
| `123456` | `Coffee!River-Train2026` |
| `password` | `correct-horse-battery-staple` |
| `admin123` | `Rain!Forest@Mountain2026` |
| `john123` | Long unique passphrase |

---

# How Developers Improve Password Security

## 1️⃣ Enforce Minimum Length

Require at least

```text
12–16 Characters
```

---

## 2️⃣ Block Common Passwords

Reject passwords such as

```text
123456
password
admin
qwerty
```

---

## 3️⃣ Check Against Breached Password Lists

Reject passwords that have appeared in known data breaches.

---

## 4️⃣ Prevent Password Reuse

Users should not be able to reuse recently used passwords.

---

## 5️⃣ Encourage Password Managers

Password managers generate long,

random,

unique passwords.

---

## 6️⃣ Enable Multi-Factor Authentication (MFA)

Even if a password is compromised,

an attacker still needs another authentication factor.

---

# Burp Suite / Pentesting Perspective

> [!warning]
> Test only systems you own or have authorization to assess.

During testing:

1. Register a new account.
2. Attempt to create simple passwords.
3. Observe whether the application enforces password requirements.
4. Test whether password reuse is allowed.
5. Review login protections such as account lockout, rate limiting, and MFA.

---

# Impact Assessment

| Weakness | Risk |
|-----------|------|
| Weak Password | 🟠 Medium |
| Password Reuse | 🔴 High |
| No MFA | 🔴 High |
| Unlimited Login Attempts | 🔴 High |
| No Breached Password Check | 🟠 Medium |

---

# Secure Password Checklist

- [x] Minimum 12–16 characters
- [x] Encourage passphrases
- [x] Block common passwords
- [x] Block breached passwords
- [x] Prevent password reuse
- [x] Support password managers
- [x] Enable MFA
- [x] Implement account lockout or rate limiting

---

# CEH Exam Definition

> **Weak Password Policy** is a security weakness where an application allows users to create predictable or easily guessed passwords, increasing the likelihood of brute-force attacks, dictionary attacks, password spraying, and credential stuffing. A strong password policy enforces sufficient length, blocks common and breached passwords, prevents password reuse, and encourages or requires multi-factor authentication.

---

# Easy Memory Trick

> [!tip]

Imagine an apartment building.

- 🔑 Password = Apartment key
- 🚪 Weak key = Many people have a copy
- 🔒 Strong key = Unique and difficult to duplicate
- 🛡 MFA = Security guard checking your identity after the key

A weak key alone makes unauthorized entry much easier.

---

# Related Notes

- [[Authentication]]
- [[Session Hijacking]]
- [[Session Fixation]]
- [[Replay Attack]]
- [[Cookie Attributes]]
- [[JWT]]
- [[Multi-Factor Authentication]]
- [[OWASP Top 10 2025]]

---

# Key Takeaways

> [!summary]

- Weak passwords are easy to guess and significantly increase the risk of account compromise.
- Common attacks include **Brute Force**, **Dictionary Attacks**, **Credential Stuffing**, and **Password Spraying**.
- Password reuse increases the impact of credential leaks across multiple services.
- A strong password policy should prioritize **length**, **uniqueness**, **breached-password detection**, **password reuse prevention**, and **MFA**.
- During security assessments, evaluate password creation rules alongside login protections such as **rate limiting**, **account lockout**, and **multi-factor authentication**.