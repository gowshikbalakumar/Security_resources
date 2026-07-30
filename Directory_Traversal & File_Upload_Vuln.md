---
title: Directory Traversal & File Upload Vulnerability
tags:
  - WebSecurity
  - OWASP
  - DirectoryTraversal
  - PathTraversal
  - FileUpload
  - BugBounty
  - CEH
  - PortSwigger
  - LFI
created: 2026-07-30
---

# 📁 Directory Traversal & File Upload Vulnerability

> [!info]
> These notes are intended for learning, bug bounty, CEH preparation, and legal security testing.

---

# 📂 Directory Traversal (Path Traversal)

## Definition

Directory Traversal (also called **Path Traversal**) is a web vulnerability where an attacker manipulates a file path to access files outside the intended directory.

---

## 📖 How it Works

Suppose a web application stores downloadable files inside:

```text
/var/www/files/
```

Users normally request:

```http
GET /download?file=report.pdf
```

Backend code:

```python
open("/var/www/files/" + file)
```

If user input is not validated, an attacker can manipulate the file path.

---

## Attack Example

Instead of requesting:

```text
report.pdf
```

An attacker supplies:

```text
../../../../etc/passwd
```

### What does `../` mean?

`../` means **move up one directory**.

```text
/var/www/files/
        │
        ▼
../
        │
        ▼
/var/www/
        │
        ▼
../
        │
        ▼
/var/
        │
        ▼
../
        │
        ▼
/
        │
        ▼
etc/passwd
```

Final path:

```text
/etc/passwd
```

---

## ⚠️ Why is it Dangerous?

Attackers may access:

- `/etc/passwd`
- `/etc/shadow`
- Windows `win.ini`
- Application configuration files
- Database credentials
- API keys
- Source code
- Backup files
- Environment variables (`.env`)
- SSH keys
- Log files

---

## Common Payloads

### Linux

```text
../../../etc/passwd
```

### Windows

```text
..\..\..\windows\win.ini
```

### Filter Bypass

```text
....//....//etc/passwd
```

### URL Encoded

```text
..%2f..%2fetc/passwd
```

```
%2f = /
%5c = \
```

---

## Finding Directory Traversal

Look for parameters such as:

```text
file=
path=
page=
template=
download=
image=
filename=
document=
```

Example:

```http
GET /view?file=invoice.pdf
```

---

## PortSwigger Example

```http
GET /image?filename=218.png
```

Testing request:

```http
GET /image?filename=../../../etc/passwd
```

Possible indicator:

```text
root:x:0:0:root:/root:/bin/bash
```

---

## Why Filters Fail

Bad filtering:

```python
filename = filename.replace("../","")
```

Possible bypasses:

```text
....//
..%2f
../\
```

---

# 🛡 Prevention

## ✅ Allowlist

```python
ALLOWED = {
    "report.pdf",
    "invoice.pdf"
}

if filename not in ALLOWED:
    deny()
```

## ✅ Resolve Canonical Path

```python
from pathlib import Path

base = Path("/var/www/files").resolve()
target = (base / filename).resolve()

if not str(target).startswith(str(base)):
    deny()
```

## ✅ Avoid User-Controlled Paths

Instead of

```text
/download?file=secret.txt
```

Use

```text
/download?id=123
```

---

# 📊 Quick Comparison

| Concept | Meaning |
|---------|---------|
| `../` | Move up one directory |
| `../../` | Move up two directories |
| Directory Traversal | Read files outside intended directory |
| LFI | Local File Inclusion |

---

# 📤 File Upload Vulnerability

## Definition

A File Upload Vulnerability occurs when an application improperly validates uploaded files, allowing attackers to upload unexpected or potentially dangerous content.

---

## Safe Testing Workflow

### 1. Locate Upload Features

- Profile Picture
- Resume Upload
- Attachments
- Image Gallery
- Support Ticket

### 2. Upload a Normal File

Examples:

```text
test.jpg
test.png
test.pdf
```

Observe:

- Allowed extensions
- Maximum size
- File URL
- File renaming

### 3. Check Validation

Examples:

```text
test.txt
test.jpg
```

Rename a text file to `.jpg`.

Weak validation may rely only on the filename extension.

### 4. Verify Storage

Check whether:

- Files are public
- URL is returned
- Files are inside the web root
- Files are served directly

---

# 🚩 Weak Upload Indicators

- Accepts unexpected file types
- Trusts file extension only
- No MIME validation
- No magic byte verification
- Stores uploads inside web root
- Direct access to uploaded files

---

# 🛡 Secure Upload Checklist

- [x] Extension allowlist
- [x] MIME validation
- [x] Magic byte verification
- [x] Rename uploaded files
- [x] Store outside web root
- [x] Download handler
- [x] Remove metadata
- [x] Antivirus scanning
- [x] File size limits

---

# 📝 Bug Bounty Checklist

- [ ] Find upload endpoint
- [ ] Upload valid file
- [ ] Record accepted extensions
- [ ] Observe renaming
- [ ] Check public accessibility
- [ ] Verify validation method
- [ ] Check storage location
- [ ] Collect evidence
- [ ] Report responsibly

---

# 🔬 Practice Labs

## PortSwigger

- Path Traversal Labs
- File Upload Labs

## DVWA

- File Inclusion
- File Upload

---

# 🎯 Key Takeaways

> Directory Traversal abuses file path manipulation to access unintended files.

> Secure file uploads require validating the actual file content, not just the extension.

> Always test only systems you own or are authorized to assess.