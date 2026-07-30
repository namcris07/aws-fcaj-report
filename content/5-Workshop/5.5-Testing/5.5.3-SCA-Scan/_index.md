---
title : "Stage 5 – SCA Dependency Scan (Trivy)"
date : 2026-07-06 
weight : 3
chapter : false
pre : " <b> 5.5.3. </b> "
---

# 5.5.3 Stage 5 — Software Composition Analysis (SCA Scan — Trivy FS)

---

### 1. Tooling & Execution Mechanics

**Trivy** is invoked in `fs` (filesystem scan) mode, parsing `package-lock.json` to construct a complete dependency graph and cross-referencing each package against vulnerability databases (NVD, GitHub Security Advisory):

```bash
trivy fs --scanners vuln --severity HIGH,CRITICAL --quiet "${SCAN_DIR}"
```

---

### 2. Dependency Vulnerability Injection Scenario

```text
styled-components@5.0.1 ──► css-select@2.x ──────► nth-check@1.0.2 [CVE-2021-3803 ReDoS]
react-spring@8.0.27     ──► webpack (react-scripts) ──► serialize-js@4.0.0 [GHSA-5c6j RCE]
lodash@4.17.21          ──► underscore@1.13.6  ──► underscore@1.13.6 [CVE-2026-27601 DoS]
```

---

### 3. Verification Findings & Scan Results

Trivy uncovers **4 HIGH severity vulnerabilities** within `app/package-lock.json`:

#### Table 5.5.3: Discovered SCA Vulnerability Details (Trivy FS)

| Package Name | Vulnerability / CVE ID | Severity | Currently Pinned Version | Fixed Patched Version |
|---|---|---|---|---|
| **nth-check** | CVE-2021-3803 | HIGH | 1.0.2 | 2.0.1 |
| **serialize-javascript** | GHSA-5c6j-r48x-rmvq | HIGH | 4.0.0 / 6.0.2 | 7.0.3 |
| **underscore** | CVE-2026-27601 | HIGH | 1.13.6 | 1.13.8 |

![SCA Vulnerability Breakdown](/images/5-Workshop/5.5-Testing/sca_pie_chart.png)

*Figure 5.5.3a: Distribution of SCA package vulnerabilities (4 HIGH severity findings).*

---

### 4. Remediation Strategy

Perform automated dependency updates via `npm audit fix` or bump transitive dependency overrides in `package.json`.

---

### Actual Screenshots: OWASP ZAP DAST Report & S3 Storage

![ZAP HTML Report](/images/5-Workshop/5.5-Testing/zap_html_report.png)
*Figure 5.5.3b: OWASP ZAP HTML report archived in Jenkins Artifacts — displaying WARN alerts regarding missing security response headers.*

![S3 DAST Reports](/images/5-Workshop/5.5-Testing/s3_dast_reports.png)
*Figure 5.5.3c: Amazon S3 Console displaying ZAP DAST scan reports automatically uploaded to `reports/dast/` prefix.*