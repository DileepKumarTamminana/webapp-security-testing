# 🔍 Web Application Security Testing

![OWASP ZAP](https://img.shields.io/badge/OWASP%20ZAP-DAST-red?logo=owasp&logoColor=white)
![Burp Suite](https://img.shields.io/badge/Burp%20Suite-Manual%20Testing-orange?logo=burpsuite&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Containerized%20Scanning-2496ED?logo=docker&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-Automated%20DAST-2088FF?logo=githubactions&logoColor=white)
![OWASP Top 10](https://img.shields.io/badge/OWASP-Top%2010%202021-black?logo=owasp&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-yellow.svg)

A hands-on, portfolio-grade demonstration of **Dynamic Application Security Testing (DAST)** methodology and automation. This repository does **not** ship a vulnerable web application — instead, it documents how to responsibly scan and manually test one, using the industry-standard **OWASP Juice Shop** as a legal, purpose-built practice target.

The goal is to show a repeatable, automatable web application security testing workflow that a security engineer would run against real, in-scope targets: recon, automated baseline scanning with OWASP ZAP, manual verification, and reporting mapped to the **OWASP Top 10 (2021)**.

---

## Overview

| | |
|---|---|
| **Focus** | Web Application Security Testing (DAST + methodology) |
| **Target used for demonstration** | [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/) (intentionally vulnerable, legal to test) |
| **Primary tool** | [OWASP ZAP](https://www.zaproxy.org/) — Baseline Scan |
| **Automation** | GitHub Actions workflow (scheduled + on-demand) |
| **Standard referenced** | [OWASP Top 10 2021](https://owasp.org/Top10/) and the [OWASP Web Security Testing Guide (WSTG)](https://owasp.org/www-project-web-security-testing-guide/) |
| **Author** | [Dileep Kumar Tamminana](https://github.com/DileepKumarTamminana) |

This project is intentionally structured as **methodology + automation + documentation** rather than as an application. It reflects how a security engineer plans, automates, and reports on web application security assessments in a real engagement — while using only a legally sanctioned, deliberately vulnerable target for demonstration.

---

## What This Project Demonstrates

- **DAST with OWASP ZAP (Baseline Scan)** — Configuring and running `zap-baseline.py` (via the official Docker image / GitHub Action) against a target URL, passively crawling and spidering the application, and generating a machine- and human-readable report.
- **Methodology aligned to the OWASP Testing Guide** — A structured approach (recon → mapping → automated scanning → manual testing → reporting) documented in [`docs/methodology.md`](docs/methodology.md), rather than ad-hoc testing.
- **Findings mapped to the OWASP Top 10 (2021)** — Every documented finding is tied back to a Top 10 category (e.g., A03:2021 – Injection, A05:2021 – Security Misconfiguration) so results are meaningful to both technical and non-technical stakeholders. See [`docs/findings-template.md`](docs/findings-template.md) for the reusable report format and worked examples.
- **CI/CD security automation** — A GitHub Actions workflow ([`.github/workflows/zap-baseline.yml`](.github/workflows/zap-baseline.yml)) that runs the ZAP baseline scan on a schedule or on demand, uploading the resulting report as a build artifact — the same pattern used to gate deployments with automated security checks (DevSecOps).
- **Tunable scan behavior** — A [`.zap/rules.tsv`](.zap/rules.tsv) file showing how individual ZAP rule IDs can be set to `WARN`, `IGNORE`, or `FAIL`, so noisy or accepted-risk findings don't block a pipeline while genuine issues do.
- **Use of OWASP Juice Shop as a legal practice target** — Juice Shop is an official OWASP project built specifically to contain (and teach) vulnerabilities across the OWASP Top 10. It is designed to be attacked, making it a safe and legal stand-in for a real client engagement in a portfolio context.

---

## Scope & Authorization Note

> **⚠️ Only ever test applications you own or are explicitly authorized to test.**

Scanning or attacking systems without permission is illegal in most jurisdictions (e.g., under the U.S. Computer Fraud and Abuse Act, UK Computer Misuse Act, and equivalent laws elsewhere) and violates professional codes of ethics for security practitioners.

This repository exclusively targets **[OWASP Juice Shop](https://owasp.org/www-project-juice-shop/)**, which is:

- An official OWASP flagship project, **intentionally built with vulnerabilities** for training and benchmarking security tools.
- **Explicitly designed and licensed to be attacked** — no authorization request is required to test an instance you deploy yourself (e.g., locally via Docker) or a public demo instance provided for that purpose.
- Used here purely to **demonstrate methodology and tooling**, not to claim any real-world compromise.

If you fork or adapt this project, update the target URL only to applications you own, have a signed engagement/authorization letter for, or that are explicitly published as legal testing targets (e.g., OWASP Juice Shop, OWASP WebGoat, PortSwigger Web Security Academy labs).

---

## How the Automated ZAP Workflow Works

The workflow at [`.github/workflows/zap-baseline.yml`](.github/workflows/zap-baseline.yml):

1. **Triggers** on a manual `workflow_dispatch` (with an optional `target_url` input) and on a **weekly schedule** (`cron`), so scans run continuously without manual effort.
2. **Runs the official [`zaproxy/action-baseline`](https://github.com/zaproxy/action-baseline) action**, which wraps the ZAP `zap-baseline.py` script — a passive, spider-based scan safe enough to run regularly without risking damage to the target.
3. **Targets a configurable URL** (defaulting to a public Juice Shop demo instance) via the `target_url` input / `TARGET_URL` environment variable, so the same workflow can be pointed at any authorized target.
4. **Applies scan tuning** from [`.zap/rules.tsv`](.zap/rules.tsv) to control which rule IDs should `WARN`, `IGNORE`, or `FAIL` the build.
5. **Publishes the generated report** (HTML/Markdown/JSON, produced by ZAP) as a downloadable **GitHub Actions artifact**, so findings can be reviewed, archived, or attached to a written report (see [`docs/findings-template.md`](docs/findings-template.md)).

This mirrors how DAST is embedded into a CI/CD pipeline in a DevSecOps environment — automated, repeatable, and producing evidence for every run.

---

## How to Run ZAP Locally

You can reproduce the same baseline scan locally using Docker — no installation required beyond Docker itself.

```bash
# Pull the official OWASP ZAP stable image
docker pull zaproxy/zap-stable

# Run a baseline scan against an authorized target (example: a local Juice Shop instance)
docker run --rm -v $(pwd)/reports:/zap/wrk/:rw \
  -t zaproxy/zap-stable zap-baseline.py \
  -t https://demo.owasp-juice.shop \
  -g gen.conf \
  -r zap-baseline-report.html \
  -c rules.tsv
```

Flag reference:

| Flag | Purpose |
|---|---|
| `-t` | Target URL to scan (must be owned/authorized) |
| `-g` | Generate a default rules config file if one doesn't exist |
| `-r` | Output report filename (written to the mounted `/zap/wrk/` volume) |
| `-c` | Path to a rules config file (e.g., [`.zap/rules.tsv`](.zap/rules.tsv)) controlling per-rule WARN/IGNORE/FAIL behavior |

### Spinning up a local Juice Shop target

```bash
docker run --rm -d -p 3000:3000 bkimminich/juice-shop
```

Then run the ZAP baseline scan above against `http://localhost:3000`.

---

## Repository Structure

```
webapp-security-testing/
├── .github/
│   └── workflows/
│       └── zap-baseline.yml       # Automated ZAP baseline scan (scheduled + manual)
├── .zap/
│   └── rules.tsv                  # ZAP rule tuning (WARN/IGNORE/FAIL per rule ID)
├── docs/
│   ├── methodology.md             # Web app pentest methodology (recon → reporting)
│   └── findings-template.md       # Reusable finding report template + worked examples
├── .gitignore
├── LICENSE
└── README.md
```

---

## Tools & Technologies

- **OWASP ZAP** — Automated DAST scanning (spidering, passive scanning, baseline analysis)
- **Burp Suite** — Manual testing, request/response manipulation, and Repeater/Intruder-based verification of automated findings
- **Docker** — Consistent, disposable environments for both the scanner and the vulnerable target
- **GitHub Actions** — CI/CD automation for scheduled and on-demand security scanning
- **OWASP Juice Shop** — Legal, intentionally vulnerable practice target
- **OWASP Top 10 (2021)** & **OWASP WSTG** — Frameworks used to structure testing and reporting

---

## Author

**Dileep Kumar Tamminana** — Cybersecurity Engineer

GitHub: [github.com/DileepKumarTamminana](https://github.com/DileepKumarTamminana)

---

## License

This project is licensed under the [MIT License](LICENSE).
