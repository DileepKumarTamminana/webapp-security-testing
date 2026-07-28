# ▶️ How to Run — Web Application Security Testing

Automated DAST with **OWASP ZAP** baseline scanning, plus a documented methodology and reusable finding templates mapped to the OWASP Top 10.

> ⚠️ **Authorization required.** Only scan applications you **own** or are **explicitly authorized** to test. The default target is **OWASP Juice Shop**, which is intentionally vulnerable and legal to test.

## Prerequisites
- Docker (for a local ZAP scan), **or** just use the GitHub Actions workflow

## Run a ZAP baseline scan locally
```bash
docker run -t ghcr.io/zaproxy/zaproxy:stable zap-baseline.py \
  -t https://demo.owasp-juice.shop \
  -r zap-report.html
```
- `-t` — target URL (change to your authorized target)
- `-r` — output HTML report

To use the rule tuning file in this repo, mount it:
```bash
docker run -t -v "$(pwd)/.zap:/zap/wrk:rw" ghcr.io/zaproxy/zaproxy:stable zap-baseline.py \
  -t https://demo.owasp-juice.shop -c rules.tsv -r zap-report.html
```

## Run via GitHub Actions (recommended)
Repo → **Actions** tab → **ZAP Baseline** workflow → **Run workflow**:
- Optionally set `target_url` (defaults to the Juice Shop demo instance)
- The HTML report is uploaded as a build **artifact**

The workflow is **manual-trigger only** (`workflow_dispatch`) plus a weekly schedule, so it never scans a target without your action.

## Documentation
- `docs/methodology.md` — the testing methodology (recon → mapping → automated scan → manual testing per OWASP Top 10 → reporting)
- `docs/findings-template.md` — a reusable finding report template with two worked examples (Reflected XSS, SQL Injection)
- `.zap/rules.tsv` — ZAP baseline rule tuning (WARN/IGNORE/FAIL per rule id)
