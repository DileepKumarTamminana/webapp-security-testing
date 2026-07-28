# Finding Report Template

Use this template to document each security finding discovered during automated scanning (OWASP ZAP) or manual testing. Copy the template section for each new finding. Two worked examples (illustrative only, based on well-known OWASP Juice Shop challenge classes) follow at the end.

> Findings below are illustrative examples written for portfolio/demonstration purposes to show reporting format and OWASP Top 10 mapping. They are not a claim of a specific undisclosed vulnerability against a live system.

---

## Finding Template

```markdown
### [Finding Title]

**Severity / CVSS:** [Critical | High | Medium | Low | Informational] — CVSS v3.1: [Score] ([Vector string])

**OWASP Top 10 (2021) Category:** [A0X:2021 – Category Name]

**Affected URL / Parameter:** `[https://target/path?parameter=]`

**Description:**
[Clear, concise explanation of the vulnerability — what it is and why it exists.]

**Steps to Reproduce:**
1. [Step one]
2. [Step two]
3. [Step three — include the exact payload/request used]

**Impact:**
[What an attacker could achieve by exploiting this — data exposure, account takeover, financial loss, reputational damage, etc.]

**Remediation:**
[Specific, actionable guidance to fix the root cause — not just "sanitize input," but the concrete control to implement.]

**References:**
- [OWASP Cheat Sheet / CWE / vendor doc link]
```

---

## Example Finding 1: Reflected Cross-Site Scripting (XSS)

**Severity / CVSS:** High — CVSS v3.1: 6.1 (AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N)

**OWASP Top 10 (2021) Category:** A03:2021 – Injection

**Affected URL / Parameter:** `https://demo.owasp-juice.shop/#/search?q=<parameter>`

**Description:**
The search functionality reflects the `q` query parameter back into the page's DOM without adequate output encoding or sanitization. An attacker-supplied script is executed in the context of the victim's browser session when a crafted link is followed, allowing arbitrary JavaScript execution.

**Steps to Reproduce:**
1. Navigate to the application's product search feature.
2. Submit the search query: `<script>alert(document.cookie)</script>` (or an equivalent payload confirmed via ZAP's passive/active scan alert).
3. Observe the payload executes in the browser rather than being rendered as literal text, confirming the input is not properly encoded before being rendered.
4. Craft a URL containing the payload and confirm it executes when opened in a fresh browser session (reflected XSS via a shared link).

**Impact:**
An attacker could use this to steal session cookies/tokens, perform actions on behalf of the victim (session riding), redirect users to phishing pages, or deliver further client-side exploits — particularly damaging if delivered via a mass-distributed link (e.g., email, chat, forum post).

**Remediation:**
- Apply context-aware output encoding (HTML entity encoding) to all user-supplied data rendered into HTML.
- Adopt a templating framework that auto-escapes output by default (e.g., React/Angular's built-in escaping) and avoid raw DOM injection (`innerHTML`, `dangerouslySetInnerHTML`) with untrusted input.
- Implement a strict Content-Security-Policy (CSP) header to reduce the impact of any XSS that does occur.
- Validate and encode input server-side in addition to client-side controls.

**References:**
- [OWASP XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [CWE-79: Improper Neutralization of Input During Web Page Generation](https://cwe.mitre.org/data/definitions/79.html)

---

## Example Finding 2: SQL Injection in Login Form

**Severity / CVSS:** Critical — CVSS v3.1: 9.8 (AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H)

**OWASP Top 10 (2021) Category:** A03:2021 – Injection

**Affected URL / Parameter:** `https://demo.owasp-juice.shop/rest/user/login` — `email` parameter (JSON body)

**Description:**
The login endpoint concatenates the `email` field directly into a backend database query without parameterization. Supplying a crafted value alters the intended query logic, allowing authentication bypass without valid credentials.

**Steps to Reproduce:**
1. Intercept the login request in a proxy tool (e.g., Burp Suite).
2. Replace the `email` field value with a classic authentication-bypass payload, e.g.: `' OR 1=1--`
3. Submit the request with any value in the `password` field.
4. Observe the response returns a valid authenticated session (e.g., a JWT / session token) for an arbitrary account (frequently the first user record, such as an administrator), confirming the query logic was altered.

**Impact:**
Complete authentication bypass allows an unauthenticated attacker to log in as any user, including administrative accounts — leading to full account takeover, exposure of all user data reachable via the application, and potential further database compromise (data exfiltration, modification, or deletion) depending on database privileges.

**Remediation:**
- Use parameterized queries / prepared statements (or a vetted ORM with parameter binding) for all database access — never build queries via string concatenation with user input.
- Apply least-privilege database account permissions for the application's database user.
- Add server-side input validation as defense-in-depth (not as the primary control).
- Deploy a Web Application Firewall (WAF) as an additional detection/mitigation layer, not a substitute for fixing the root cause.

**References:**
- [OWASP SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
- [CWE-89: Improper Neutralization of Special Elements used in an SQL Command](https://cwe.mitre.org/data/definitions/89.html)
