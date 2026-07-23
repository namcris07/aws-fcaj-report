---
title : "Stage 4 – SCA Scan (Trivy FS)"
date : 2026-07-06 
weight : 3
chapter : false
pre : " <b> 5.5.3. </b> "
---

# 5.5.3 Stage 4 – Dependency Vulnerability Audit (SCA Scan)

---

### 1. Mechanics

**Trivy Filesystem Mode** parses `package-lock.json` to build complete transitive dependency trees and queries CVE databases.

---

### 2. Vulnerability Findings

Trivy detects **4 HIGH severity vulnerabilities** (`nth-check`, `serialize-javascript`, `underscore`), all with available security patches (`Status: fixed`).

![SCA Vulnerabilities Pie Chart](/images/5-Workshop/5.5-Testing/sca_pie_chart.png)

*Figure 5.5.3: SCA Vulnerabilities breakdown pie chart.*

---

### 3. Remediation

Run automated audit resolution via `npm audit fix`.
