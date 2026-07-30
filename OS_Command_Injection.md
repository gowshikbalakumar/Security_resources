---
title: OS Command Injection (Shell Injection)
aliases:
  - Shell Injection
  - Command Injection
  - OSCI
tags:
  - WebSecurity
  - OWASP
  - CommandInjection
  - BugBounty
  - CEH
  - Linux
  - Windows
created: 2026-07-30
---

# 💻 OS Command Injection (Shell Injection)

> [!info]
> **OS Command Injection** (also called **Shell Injection**) is a web security vulnerability where an attacker can inject **Operating System commands** into an application that executes system commands.
>
> It occurs when an application passes **user-controlled input** directly to the operating system shell **without proper validation or sanitization**.

---

# 📖 What is OS Command Injection?

Many web applications need to interact with the operating system.

Examples include:

- Network diagnostics
- Image conversion
- File compression
- Database backups
- System monitoring

Instead of performing these tasks using built-in programming libraries, some developers execute **operating system commands**.

If user input becomes part of those commands without proper protection, unintended commands may be executed.

---

# 🏗 Simple Architecture

```text
            User
              │
              ▼
      Web Application
              │
              │ Executes System Command
              ▼
     Operating System Shell
              │
              ▼
        Command Output
```

---

# 🎓 Real-World Analogy

Imagine a receptionist who receives written instructions.

Normal instruction:

> "Print today's sales report."

The receptionist prints the report.

Now imagine someone secretly adds extra instructions to the note.

Instead of performing only the intended task, the receptionist follows **all** instructions.

OS Command Injection works similarly.

The application executes more than what the developer originally intended.

---

# Why Developers Use OS Commands

Applications may execute operating system commands to:

- Ping servers
- Convert images
- Compress files
- Generate backups
- Retrieve system information

Example (PHP)

```php
$ip = $_GET['ip'];

system("ping -c 4 " . $ip);
```

If the user enters

```text
192.168.1.1
```

the application executes

```text
ping -c 4 192.168.1.1
```

This is the intended behavior.

---

# Where the Vulnerability Comes From

The vulnerability appears when the application **directly appends user input** to an operating system command.

Example:

```php
system("ping -c 4 " . $_GET['ip']);
```

If the application performs **no validation**, the shell may interpret additional user-controlled input as more than just an IP address.

Instead of executing only the intended command, the operating system may execute **additional unintended commands**.

The exact behavior depends on:

- Operating system
- Shell
- Programming language
- Application implementation

---

# Example Scenario

A website provides a network diagnostic tool.

```
Enter IP Address

_____________________

[ Ping ]
```

Backend code

```php
system("ping -c 4 " . $_GET['ip']);
```

Normal input

```text
8.8.8.8
```

Executed command

```text
ping -c 4 8.8.8.8
```

Everything works correctly.

If user input is not validated, unexpected shell syntax may alter what the operating system executes.

---

# Visual Flow

```text
User Input
      │
      ▼
Web Application
      │
      │ Builds OS Command
      ▼
Operating System Shell
      │
      ▼
Command Executes
```

---

# Why is it Dangerous?

If successful, an attacker may be able to:

- Execute unintended operating system commands
- Read accessible files
- Modify or delete data
- Gather information about the server
- Install software (depending on privileges)
- Move laterally within the environment if additional weaknesses exist

The impact depends entirely on the permissions of the account running the application.

---

# Common Causes

## 1. Unsanitized User Input

Example:

```php
system("ping " . $_GET['ip']);
```

The application trusts user input without validation.

---

## 2. Dangerous Functions

### PHP

```php
system()
exec()
shell_exec()
passthru()
```

---

### Python

```python
os.system()

subprocess.run()

subprocess.Popen()
```

---

### Node.js

```javascript
child_process.exec()

child_process.execSync()
```

These functions become risky when called with untrusted input.

---

# Secure Coding Practices

## 1. Avoid Shell Commands

Whenever possible,

use programming language libraries instead of external commands.

Examples:

- File APIs
- Networking libraries
- Compression libraries

instead of shell utilities.

---

## 2. Validate User Input

Accept only expected formats.

Examples:

- IPv4 addresses
- Hostnames
- Numeric IDs

Reject everything else.

---

## 3. Pass Arguments Safely

If executing external programs is unavoidable,

use APIs that safely pass arguments without constructing one shell command string.

---

## 4. Least Privilege

Run the application using an account that has only the permissions required.

This limits the damage if a vulnerability exists.

---

## 5. Use Safe APIs

Many programming languages provide APIs that invoke programs **without invoking a shell**.

These APIs reduce the risk of shell metacharacters being interpreted.

---

# Dangerous vs Secure Design

## ❌ Vulnerable Design

```text
User Input
      │
      ▼
Application
      │
      ▼
Shell Command
      │
      ▼
Operating System
```

---

## ✅ Secure Design

```text
User Input
      │
      ▼
Validation
      │
      ▼
Safe Library/API
      │
      ▼
Operating System
```

---

# OS Command Injection vs Code Injection

| OS Command Injection | Code Injection |
|----------------------|----------------|
| Targets Operating System | Targets Application Runtime |
| Executes OS Commands | Executes Application Code |
| Uses Shell/System Interfaces | Uses Language Interpreter |
| Affects Host Operating System | Affects Application Logic |

---

# Common Programming Languages

| Language | Risky Functions |
|----------|-----------------|
| PHP | `system()`, `exec()`, `shell_exec()`, `passthru()` |
| Python | `os.system()`, `subprocess.run()`, `subprocess.Popen()` |
| Node.js | `child_process.exec()`, `execSync()` |

---

# Bug Bounty Testing Methodology

> [!warning]
> Test only systems you own or have explicit authorization to assess.

General workflow:

1. Identify functionality that invokes operating system commands (such as network diagnostics or file conversion).
2. Determine whether user input influences those commands.
3. Observe how the application validates and processes that input.
4. Use safe, non-destructive techniques appropriate to the engagement to determine whether unintended command execution is possible.
5. Collect evidence and report findings responsibly if a vulnerability is confirmed.

---

# Impact Assessment

| Impact | Severity |
|----------|----------|
| Information Disclosure | 🟠 Medium |
| File Modification | 🔴 High |
| Data Destruction | 🔴 High |
| Server Compromise | 🔴 Critical |
| Lateral Movement | 🔴 Critical |

---

# Secure Development Checklist

- [x] Avoid shell commands where possible
- [x] Validate all user input
- [x] Use language libraries instead of shell utilities
- [x] Use APIs that safely pass arguments
- [x] Run applications with least privilege
- [x] Review code for dangerous system-call functions

---

# CEH Exam Definition

> **OS Command Injection (Shell Injection)** is a vulnerability where an application executes operating system commands using untrusted user input without proper validation or safeguards, potentially allowing unintended command execution with the privileges of the application.

---

# Easy Memory Trick

> [!tip]

Think of a restaurant chef.

- Customer places an order.
- Chef should cook **only** the requested dish.
- If someone secretly adds extra instructions to the order slip and the chef follows them too, unintended actions occur.

Similarly:

- **User Input** = Order Slip
- **Application** = Waiter
- **Operating System** = Chef

If the waiter forwards the entire note without checking it, the chef may perform actions that were never intended.

---

# Related Notes

- [[File Upload Vulnerability]]
- [[Directory Traversal]]
- [[SQL Injection]]
- [[Server-Side Request Forgery]]
- [[Remote Code Execution]]
- [[Command Injection]]
- [[OWASP Top 10 2025]]

---

# Key Takeaways

> [!summary]

- OS Command Injection occurs when untrusted input is incorporated into operating system command execution.
- It commonly arises from invoking shell commands without proper validation or safe argument handling.
- The impact depends on the privileges of the account running the application.
- Prefer built-in language libraries over shell commands whenever possible.
- If external commands are necessary, validate input strictly, use safe APIs that avoid invoking a shell, and follow the principle of least privilege.