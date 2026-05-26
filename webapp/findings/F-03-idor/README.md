# Finding ID: F-03 — Insecure Direct Object Reference (IDOR)

## Severity
High

## OWASP Category
OWASP A01:2021 — Broken Access Control

---

## Overview

During testing of OWASP Juice Shop APIs, an Insecure Direct Object Reference (IDOR) vulnerability was identified involving basket resource access.

The application exposed predictable numeric object identifiers through API endpoints. By modifying the basket identifier within authenticated requests, it was possible to access basket data associated with another user account.

The vulnerability demonstrated insufficient object-level authorization validation on backend API requests.

---

## Affected Component

- Application: OWASP Juice Shop
- Endpoint: `/rest/basket/{id}`
- Environment: Local Docker Deployment

---

## Testing Approach

The assessment focused on identifying predictable object references exposed through authenticated API requests.

Testing was performed using:
- browser developer tools
- captured authenticated API requests
- manual identifier manipulation
- cURL-based request replay

---

## Initial Observation

During normal application interaction, authenticated requests to basket resources were observed in browser network traffic.

Example endpoint:

```http
/rest/basket/6
```

The numeric identifier suggested direct object referencing behavior.

---

## Exploitation Steps

### 1. Captured Authenticated Request

An authenticated API request was captured using browser developer tools.

The request included:
- valid bearer token
- basket resource identifier

---

### 2. Identifier Manipulation

The basket identifier was manually modified.

Original request:

```http
/rest/basket/6
```

Modified request:

```http
/rest/basket/4
```

---

### 3. Replay Using cURL

The modified request was replayed using the original authenticated bearer token.

Example request:

```bash
curl 'http://localhost:3000/rest/basket/4' \
-X GET \
-H 'Authorization: Bearer <token>'
```

---

## Observation

The application returned basket information associated with another user account.

Returned response data included:
- basket identifier
- user identifier
- product details
- basket item information

Observed response excerpt:

```json
{
  "id": 4,
  "UserId": 11,
  "Products": [
    {
      "name": "Raspberry Juice (1000ml)"
    }
  ]
}
```

This confirmed that backend authorization validation was insufficient for object-level resource access.

---

## Evidence Collected

| Screenshot | Description |
|---|---|
| idor-original-basket.png | Original authenticated basket request |
| idor-modified-basket.png | Modified basket identifier request |
| idor-unauthorized-response.png | Initial unauthorized access attempt without valid authorization header |
| idor-authenticated-basket-access.png | Successful authenticated access to another user's basket data |

---

## Technical Root Cause

The application relied on predictable object identifiers without enforcing strict ownership validation for requested resources.

Although authentication was present, authorization checks were insufficient to verify whether the authenticated user actually owned the requested basket resource.

---

## Potential Business Impact

If exploited within a production financial services environment, this vulnerability could allow attackers to:

- access other users' sensitive information
- enumerate customer resources
- retrieve unauthorized transactional data
- perform horizontal privilege escalation
- compromise data confidentiality

In multi-user systems, Broken Access Control vulnerabilities can lead to large-scale unauthorized data exposure.

---

## Risk Assessment

| Category | Rating |
|---|---|
| Likelihood | High |
| Impact | High |
| Overall Risk | Critical |

---

## Recommended Remediation

The following remediation measures are recommended:

- enforce strict server-side ownership validation
- avoid exposing predictable object identifiers
- implement centralized authorization middleware
- validate resource ownership on every request
- monitor abnormal resource access patterns

---

## Secure Coding Recommendation

### Vulnerable Access Pattern

```javascript
GET /rest/basket/:id
```

### Recommended Secure Validation

```javascript
if (basket.UserId !== authenticatedUser.id) {
    return res.status(403).send('Forbidden');
}
```

---

## Conclusion

The assessment confirmed the presence of an Insecure Direct Object Reference vulnerability caused by insufficient object-level authorization validation.

The issue allowed authenticated access to basket resources associated with another user account through simple identifier manipulation.

The findings highlight the importance of strict authorization enforcement and secure resource ownership validation within API-driven applications.
