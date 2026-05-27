# Finding ID: F-05 — Security Misconfiguration

## Severity
Medium

## OWASP Category
OWASP A05:2021 — Security Misconfiguration

---

## Overview

During security assessment activities across DVWA and OWASP Juice Shop, multiple security misconfiguration issues were identified involving verbose error exposure, permissive response configurations, and missing browser security hardening controls.

The observed issues increase the application's exposure to information disclosure and client-side attack scenarios.

---

## Affected Components

- DVWA — SQL Injection Module
- OWASP Juice Shop — API Responses
- Environment: Local Docker Deployment

---

## Testing Approach

Testing focused on identifying insecure default configurations, verbose system behavior, and missing security hardening mechanisms.

The assessment included:
- malformed input testing
- response header analysis
- browser security header validation
- API response inspection

---

## Verbose Error Exposure

### Test Performed

Malformed SQL payloads were submitted into the DVWA SQL Injection module.

Example payload:

```sql
'
```

---

### Observation

The application returned verbose backend error behavior after receiving malformed input.

The response exposed internal application behavior and database-related processing details.

Verbose error responses may assist attackers in understanding backend query structure and application logic.

---

## Response Header Analysis

### Observed Response Headers

```http
Access-Control-Allow-Origin: *
X-Frame-Options: SAMEORIGIN
X-Content-Type-Options: nosniff
X-Recruiting: /#/jobs
```

---

## Observations

### 1. Permissive CORS Configuration

The application returned:

```http
Access-Control-Allow-Origin: *
```

This wildcard configuration allows requests from any origin and may increase exposure risk in certain deployment scenarios.

---

### 2. Missing Content Security Policy

No `Content-Security-Policy` header was observed during testing.

Missing CSP protections may increase exposure to client-side injection attacks such as Cross-Site Scripting.

---

### 3. Missing Strict Transport Security

No `Strict-Transport-Security` header was identified during testing.

This indicates lack of enforced HTTPS transport hardening policies.

---

### 4. Information Disclosure Through Headers

The application exposed additional non-essential response information:

```http
X-Recruiting: /#/jobs
```

Although low risk individually, unnecessary header disclosure may contribute to application fingerprinting activities.

---

## Evidence Collected

| Screenshot | Description |
|---|---|
| misconfig-verbose-sql-error.png | Verbose backend SQL error exposure |
| misconfig-response-headers.png | Response headers showing permissive CORS configuration and missing security headers |

---

## Technical Root Cause

The identified issues appear to stem from insecure default configurations, incomplete security hardening, and excessive backend response verbosity.

Several browser-side security protections were either missing or insufficiently restricted.

---

## Potential Business Impact

If present within a production financial services environment, these weaknesses could contribute to:

- increased attack surface exposure
- backend technology fingerprinting
- information disclosure
- increased susceptibility to client-side attacks
- cross-origin abuse scenarios

While individual misconfigurations may appear low impact independently, combined weaknesses may significantly assist attackers during reconnaissance and exploitation phases.

---

## Risk Assessment

| Category | Rating |
|---|---|
| Likelihood | Medium |
| Impact | Medium |
| Overall Risk | Medium |

---

## Recommended Remediation

The following remediation measures are recommended:

- disable verbose error exposure in production environments
- implement strict Content Security Policy headers
- enforce HTTPS using HSTS
- restrict CORS origins to trusted domains only
- minimize unnecessary response header disclosure
- apply secure production hardening standards

---

## Secure Configuration Recommendations

### Recommended CORS Configuration

```http
Access-Control-Allow-Origin: https://trusted-domain.com
```

### Recommended Security Headers

```http
Content-Security-Policy: default-src 'self'
Strict-Transport-Security: max-age=31536000
X-Frame-Options: DENY
```

---

## Conclusion

The assessment identified multiple security misconfiguration issues involving verbose error handling, permissive response configurations, and missing browser security protections.

Although several individual issues were moderate in severity, combined weaknesses may increase overall application exposure and assist attackers during reconnaissance and exploitation activities.
