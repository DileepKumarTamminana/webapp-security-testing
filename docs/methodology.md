# Web Application Security Testing Methodology

This document describes the methodology followed in this project when assessing a web application, using the phases of the **OWASP Web Security Testing Guide (WSTG)** as a foundation and reporting findings against the **OWASP Top 10 (2021)**.

> **Authorization first:** every phase below assumes the target is owned by the tester or covered by explicit written authorization (e.g., a signed engagement letter, or a published legal target such as OWASP Juice Shop). No step in this methodology should ever be performed against a system without that authorization.

---

## 1. Reconnaissance

- Identify the target application, its purpose, and its exposed attack surface (URLs, subdomains, APIs).
- Confirm the target is in scope and authorized (client engagement letter, or a known legal practice target such as OWASP Juice Shop).
- Passive information gathering: technology fingerprinting (frameworks, server headers, cookies), robots.txt / sitemap.xml review, exposed JS bundles.
- Identify authentication mechanisms, session handling, and user roles present in the application.
- Document assumptions, testing window, and rules of engagement.

## 2. Mapping the Application

- Manually walk through the application to build a functional map (pages, forms, API endpoints, user roles/privilege levels).
- Use a proxy (e.g., Burp Suite) to capture and catalog all requests made by the application, including hidden/asynchronous API calls.
- Identify all input points: query parameters, form fields, headers, cookies, file uploads, JSON/XML bodies.
- Note client-side technologies (SPA framework, client-side routing) that may hide additional attack surface.
- Group discovered endpoints by functional area (auth, search, checkout, admin, etc.) to prioritize testing.

## 3. Automated Scanning (DAST)

- Run an **OWASP ZAP baseline scan** against the mapped target to catch common, low-risk-to-test issues quickly (passive scanning + spidering, no active/aggressive attacks).
- For deeper coverage, run a **ZAP full/active scan** in a safe, non-production, authorized environment only (active scans send attack payloads and can be disruptive).
- Feed the automated scan a authenticated session/context where applicable, so scanning covers logged-in functionality, not just public pages.
- Tune scan rules (see [`.zap/rules.tsv`](../.zap/rules.tsv)) to suppress known false positives or accepted-risk findings (`IGNORE`), flag findings for review (`WARN`), or hard-fail a pipeline on critical categories (`FAIL`).
- Automate the scan in CI/CD (see [`.github/workflows/zap-baseline.yml`](../.github/workflows/zap-baseline.yml)) so regressions are caught continuously, not just during periodic manual assessments.
- Export scan output (HTML/JSON/Markdown) as the starting evidence base for manual verification.

## 4. Manual Testing per OWASP Top 10 (2021)

Automated tools catch known patterns but miss business-logic and context-dependent issues. Each category below is manually verified against the mapped application:

- **A01:2021 – Broken Access Control**
  Test for horizontal/vertical privilege escalation, insecure direct object references (IDOR), forced browsing to admin/restricted endpoints, and missing function-level access checks.

- **A02:2021 – Cryptographic Failures**
  Check for sensitive data transmitted or stored without proper encryption (HTTP instead of HTTPS, weak TLS configuration, plaintext secrets, weak password hashing).

- **A03:2021 – Injection**
  Manually test all input points for SQL injection, NoSQL injection, command injection, and reflected/stored/DOM-based Cross-Site Scripting (XSS), including bypasses of any client-side validation.

- **A04:2021 – Insecure Design**
  Review business logic flows (e.g., checkout, password reset, coupon codes) for design-level flaws that no amount of secure coding can fix — race conditions, missing rate limiting, trust boundary violations.

- **A05:2021 – Security Misconfiguration**
  Check for verbose error messages/stack traces, default credentials, unnecessary exposed services/endpoints, missing security headers (CSP, HSTS, X-Frame-Options), and directory listing.

- **A06:2021 – Vulnerable and Outdated Components**
  Fingerprint libraries/frameworks and versions in use (via headers, JS bundles, error pages) and cross-reference against known CVEs.

- **A07:2021 – Identification and Authentication Failures**
  Test login flows for weak password policies, missing account lockout/rate limiting, session fixation, predictable session tokens, and insecure "remember me"/password reset flows.

- **A08:2021 – Software and Data Integrity Failures**
  Look for insecure deserialization, unsigned/unverified auto-update mechanisms, and reliance on untrusted CI/CD dependencies or plugins.

- **A09:2021 – Security Logging and Monitoring Failures**
  Assess whether authentication failures, access-control violations, and high-value transactions are logged in a way that would support detection and incident response.

- **A10:2021 – Server-Side Request Forgery (SSRF)**
  Test any functionality that fetches a URL/resource on the server's behalf (webhooks, PDF generation, image import, URL preview) for SSRF against internal/cloud-metadata endpoints.

## 5. Reporting

- Consolidate automated (ZAP) and manual findings into a single findings register using the standard template in [`docs/findings-template.md`](findings-template.md).
- For each finding, record: title, severity/CVSS, OWASP Top 10 mapping, affected URL/parameter, description, reproduction steps, business impact, and remediation guidance.
- Prioritize findings by risk (severity × exploitability × business impact), not just by scanner-assigned confidence.
- Include an executive summary suitable for non-technical stakeholders alongside the detailed technical findings.
- Provide clear, actionable remediation guidance and, where relevant, links to authoritative references (OWASP Cheat Sheets, CWE, vendor documentation).
- Retain scan artifacts (ZAP HTML/JSON reports) as evidence, and re-test after remediation to confirm fixes.
