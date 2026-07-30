---
title : "Stage 6 – SAST Code Scan (SonarQube)"
date : 2026-07-06 
weight : 4
chapter : false
pre : " <b> 5.5.4. </b> "
---

# 5.5.4 Stage 6 — Static Application Security Testing (SAST Scan — SonarQube)

---

### 1. Tooling & Execution Mechanics

Static Application Security Testing (SAST) inspects custom-written source code logic without executing the program. **SonarQube** is integrated via `sonar-scanner-cli`, transmitting analysis results back to the SonarQube Server (operating locally at `http://localhost:9000`).

Scan findings are stored locally in `scan-reports/sonar-issues.json` and uploaded to S3 under `reports/sast/`.

---

### 2. Seeded Vulnerabilities in React Source Code

The file `app/src/App.js` includes an intentional XSS flaw for SonarQube detection:

```jsx
// BAD: dangerouslySetInnerHTML without sanitizing input → SonarQube S5247
function GameInfo({ playerName }) {
  return (
    <div
      dangerouslySetInnerHTML={{
        __html: `Welcome ${playerName}!`  // [S5247] XSS risk
      }}
    />
  );
}
```

#### Table 5.5.4: Discovered SAST Code Findings (SonarQube)

| Rule ID | Finding Description | Severity | Finding Category |
|---|---|---|---|
| **S5247** | `dangerouslySetInnerHTML` XSS vulnerability | HIGH | Vulnerability |
| **S2703** | `eval()` usage without input validation | HIGH | Security Hotspot |
| **S1763** | Unreachable code after `return` statement | MEDIUM | Code Smell |
| **S3776** | Cognitive Complexity = 18 > 15 threshold | MEDIUM | Code Smell |

**Result:** SonarQube Quality Gate status evaluates to **FAILED** — 2 Vulnerabilities exceed the security threshold.

---

### 3. Actual Screenshots: SonarQube Dashboard & Quality Gate

![SonarQube Vulnerabilities Dashboard](/images/5-Workshop/5.5-Testing/sonarqube_vulnerabilities.png)
*Figure 5.5.4a: SonarQube Dashboard displaying detected code vulnerabilities in React source files.*

![SonarQube Quality Gate FAILED](/images/5-Workshop/5.5-Testing/sonarqube_quality_gate_fail.png)
*Figure 5.5.4b: SonarQube Quality Gate status FAILED due to policy threshold breaches — halting the pipeline at Stage 6.*

---

### 4. Remediation Strategy

- **S5247 (XSS):** Replace `dangerouslySetInnerHTML` with standard text rendering or sanitize using `DOMPurify`.
- **S2703 (eval):** Remove `eval()` entirely, replacing with `JSON.parse()` or safe conditional mappings.
- **S1763:** Clean up unreachable dead code behind return statements.
- **S3776:** Refactor complex functions into smaller modular components, keeping cognitive complexity under 15.