# Finding ID: F-02 — Cross-Site Scripting (XSS)

## Severity
High

## OWASP Category
OWASP A03:2021 — Injection

---

## Overview

During testing of the DVWA XSS modules, multiple Cross-Site Scripting vulnerabilities were identified due to insufficient input sanitization and unsafe rendering of user-controlled content within the browser.

Both reflected and stored XSS behaviors were successfully demonstrated using JavaScript payload execution.

The testing confirmed that attacker-controlled scripts could execute within the application context.

---

## Affected Components

- DVWA — XSS (Reflected)
- DVWA — XSS (Stored)
- Environment: Local Docker Deployment
- Security Level: Low

---

## Testing Approach

Testing was performed by submitting crafted HTML and JavaScript payloads into application input fields and observing client-side execution behavior.

The assessment included:
- reflected XSS validation
- alternate payload execution
- stored XSS persistence testing

---

## Payloads Executed and Observations

### 1. Reflected XSS Validation

Payload used:

```html
<script>alert('XSS')</script>
```

### Observation

The application reflected user-controlled input directly into the response without sanitization.

The injected JavaScript executed successfully within the browser context and triggered a popup alert.

This confirmed the presence of reflected Cross-Site Scripting.

---

### 2. Alternate Payload Execution

Payload used:

```html
<img src=x onerror=alert('XSS')>
```

### Observation

The payload successfully triggered JavaScript execution using the image onerror event handler.

This demonstrated that multiple payload formats could bypass application handling and execute within the client browser.

---

### 3. Stored XSS Validation

Payload used:

```html
<script>alert('Stored XSS')</script>
```

### Observation

The payload was stored by the application and executed automatically when the affected page was accessed again.

Unlike reflected XSS, the payload persisted within the application and could impact multiple users interacting with the vulnerable page.

This significantly increases the overall risk severity.

---

## Evidence Collected

| Screenshot | Description |
|---|---|
| xss-reflected-normal.png | Normal application behavior with standard input |
| xss-reflected-alert.png | Successful reflected XSS payload execution |
| xss-reflected-img-onerror.png | Alternate XSS payload execution using image event handler |
| xss-stored-alert.png | Stored XSS payload executing after persistence in application |

---

## Technical Root Cause

The vulnerabilities exist because the application fails to properly sanitize or encode user-controlled input before rendering it inside HTML responses.

The application directly processes untrusted content within the browser without applying secure output encoding mechanisms.

---

## Potential Business Impact

If exploited within a production financial services environment, Cross-Site Scripting vulnerabilities could allow attackers to:

- hijack authenticated user sessions
- steal session cookies
- redirect users to malicious pages
- manipulate application content
- perform actions on behalf of authenticated users

Stored XSS vulnerabilities are particularly dangerous because malicious payloads may automatically execute for multiple users accessing affected pages.

---

## Risk Assessment

| Category | Rating |
|---|---|
| Likelihood | High |
| Impact | High |
| Overall Risk | High |

---

## Recommended Remediation

The following remediation measures are recommended:

- implement proper output encoding
- sanitize all user-controlled input
- apply Content Security Policy (CSP)
- restrict unsafe inline script execution
- validate input server-side before storage

---

## Secure Coding Recommendation

### Vulnerable Pattern

```php
echo $_GET['name'];
```

### Recommended Secure Approach

```php
echo htmlspecialchars($_GET['name'], ENT_QUOTES, 'UTF-8');
```

---

## Conclusion

The assessment confirmed multiple Cross-Site Scripting vulnerabilities within the application due to insufficient input sanitization and unsafe rendering behavior.

Both reflected and stored XSS payloads were successfully executed during testing.

The findings demonstrate the importance of secure output encoding and strict handling of user-controlled content within web applications.
