# Access Control Vulnerabilities

## Overview

Access Control vulnerabilities occur when an application fails to properly enforce restrictions on authenticated or unauthenticated users. These flaws allow attackers to gain unauthorized access to resources, perform privileged actions, or access sensitive information beyond their intended permissions.

Access control mechanisms ensure that users can only access data and functionality appropriate to their role or privilege level.

---

# Skills Practiced

- Authorization Testing
- HTTP Request Manipulation
- IDOR Testing
- Privilege Escalation Analysis
- Forced Browsing
- Burp Suite Repeater Usage
- Response Analysis
- Web Application Security Testing

---

# Tools Used

- Burp Suite Community Edition
  - Proxy
  - Repeater
  - Intruder
- Browser Developer Tools

---

# Common Access Control Vulnerabilities

| Vulnerability Type | Description |
|---|---|
| Vertical Privilege Escalation | User gains higher-level privileges |
| Horizontal Privilege Escalation | User accesses another user's resources |
| IDOR | Direct access to objects without authorization checks |
| Forced Browsing | Accessing hidden or unlinked resources |
| Parameter-Based Access Control | Authorization controlled via manipulable parameters |

---

# Labs Covered

1. Unprotected Admin Panel
2. User Role Controlled by Request
3. User ID Controlled by Request Parameter
4. URL-Based Access Control Bypass

---

---

# Lab 1 — Unprotected Admin Panel

## Objective

Identify and access an exposed administrative panel without proper authorization controls.

---

## Vulnerability Overview

The application exposed administrative functionality through predictable URLs without implementing proper server-side authorization checks.

Example endpoint:

```http
GET /admin HTTP/1.1
```

---

## Reconnaissance

The application functionality was explored manually to identify:
- Hidden endpoints
- Administrative functionality
- Sensitive resources

The `/robots.txt` file revealed potential administrative directories.

---

## Exploitation Steps

### Step 1 — Access robots.txt

```http
GET /robots.txt HTTP/1.1
Host: vulnerable-website.com
```

---

### Step 2 — Identify Administrative Path

The response disclosed:

```text
Disallow: /administrator-panel
```

---

### Step 3 — Access Administrative Endpoint

```http
GET /administrator-panel HTTP/1.1
Host: vulnerable-website.com
```

The application granted unauthorized access to the administrative interface.

---

## Result

Administrative functionality was successfully accessed without authentication or authorization.

---

## Impact

- Unauthorized administrative access
- Data manipulation
- User management abuse
- Potential complete application compromise

---

## Root Cause

The application failed to implement proper authorization checks for sensitive administrative endpoints.

---

## Remediation

- Enforce server-side authorization checks
- Restrict administrative functionality
- Deny access by default
- Remove sensitive paths from publicly accessible files

---

## Key Learnings

- Hidden endpoints are not secure controls
- robots.txt can expose sensitive paths
- Administrative functionality must always enforce authorization

---

---

# Lab 2 — User Role Controlled by Request

## Objective

Manipulate user-controlled parameters to escalate privileges and gain administrative access.

---

## Vulnerability Overview

The application relied on a client-side parameter to determine user roles.

Example request:

```http
GET /admin HTTP/1.1
Cookie: Admin=false
```

The server trusted the client-supplied role value without proper validation.

---

## Exploitation Steps

### Step 1 — Intercept Request

The request was intercepted using Burp Suite Proxy.

Original request:

```http
GET /admin HTTP/1.1
Cookie: Admin=false
```

---

### Step 2 — Modify Role Parameter

The cookie value was modified:

```http
Cookie: Admin=true
```

---

### Step 3 — Forward Modified Request

The modified request was forwarded to the server.

---

## Result

The application granted administrative access after the cookie value was manipulated.

---

## Payload Used

```text
Admin=true
```

---

## Impact

- Privilege escalation
- Unauthorized administrative access
- Security control bypass

---

## Root Cause

The application trusted client-controlled authorization data instead of enforcing server-side validation.

---

## Remediation

- Never trust client-side role values
- Store authorization information server-side
- Validate user privileges on every request

---

## Key Learnings

- Client-side controls are insecure
- Authorization decisions must always occur server-side
- Burp Suite Repeater is effective for privilege escalation testing

---

---

# Lab 3 — User ID Controlled by Request Parameter

## Objective

Exploit insecure direct object references (IDOR) to access another user's account information.

---

## Vulnerability Overview

The application exposed user account functionality through predictable user identifiers.

Example request:

```http
GET /my-account?id=wiener HTTP/1.1
```

The application failed to verify whether the authenticated user was authorized to access the requested account.

---

## Exploitation Steps

### Step 1 — Login as Standard User

Authenticated using standard user credentials.

---

### Step 2 — Intercept Account Request

Captured request using Burp Suite Proxy.

Original request:

```http
GET /my-account?id=wiener HTTP/1.1
```

---

### Step 3 — Modify User Identifier

Modified request parameter:

```http
GET /my-account?id=administrator HTTP/1.1
```

---

### Step 4 — Analyze Response

The server returned unauthorized account information belonging to another user.

---

## Result

Sensitive account data was accessed without proper authorization.

---

## Payload Used

```text
id=administrator
```

---

## Impact

- Unauthorized data disclosure
- Account information exposure
- Horizontal privilege escalation

---

## Root Cause

The application failed to validate ownership of requested resources.

---

## Remediation

- Validate object ownership server-side
- Use indirect object references
- Implement proper authorization checks

---

## Key Learnings

- Predictable identifiers increase IDOR risk
- Authentication alone does not guarantee authorization
- Resource ownership validation is critical

---

---

# Lab 4 — URL-Based Access Control Bypass

## Objective

Bypass URL-based access restrictions using alternative paths or headers.

---

## Vulnerability Overview

The application attempted to restrict access based on URL patterns but failed to properly validate alternative request formats.

Example blocked request:

```http
GET /admin HTTP/1.1
```

---

## Exploitation Steps

### Step 1 — Identify Restricted Endpoint

Attempted to access restricted administrative functionality.

---

### Step 2 — Test Alternative Headers

Modified the request using rewrite-related headers.

Modified request:

```http
GET / HTTP/1.1
X-Original-URL: /admin
```

---

### Step 3 — Forward Request

The server processed the request and exposed administrative functionality.

---

## Result

Access restrictions were bypassed using alternative request headers.

---

## Payload Used

```http
X-Original-URL: /admin
```

---

## Impact

- Access control bypass
- Unauthorized administrative access
- Security filter evasion

---

## Root Cause

The application relied on weak URL-based filtering without validating rewritten request paths.

---

## Remediation

- Validate authorization after URL rewriting
- Avoid relying solely on frontend access restrictions
- Implement centralized authorization controls

---

## Key Learnings

- URL filtering alone is insufficient
- Reverse proxy misconfigurations can introduce vulnerabilities
- Alternative request headers may bypass protections

---

# Detection Techniques

## Manual Testing

- Forced browsing
- Parameter manipulation
- ID enumeration
- Header manipulation
- Role testing

---

## Burp Suite Modules Used

- Proxy
- Repeater
- Intruder

---

# General Remediation Strategies

## 1. Enforce Server-Side Authorization

Authorization must always be validated on the server side.

---

## 2. Implement Role-Based Access Control (RBAC)

Restrict functionality according to properly validated user roles.

---

## 3. Deny Access by Default

Sensitive functionality should remain inaccessible unless explicitly authorized.

---

## 4. Validate Every Request

Authorization checks should occur for every sensitive action.

---

## 5. Avoid Predictable Identifiers

Use indirect object references where possible.

---

# References

- OWASP Access Control
- PortSwigger Web Security Academy
- CWE-284: Improper Access Control

---

# Conclusion

These labs demonstrated multiple access control weaknesses resulting from insufficient authorization enforcement and improper trust in user-controlled input. By manipulating requests, modifying identifiers, and testing alternative access methods, unauthorized access to restricted functionality and sensitive data was achieved. Proper server-side authorization validation remains essential for securing modern web applications.
