---
title : "Stage 3 – Secrets Scan (Gitleaks)"
date : 2026-07-06 
weight : 2
chapter : false
pre : " <b> 5.5.2. </b> "
---

# 5.5.2 Stage 3 – Hardcoded Secrets Scanning (Gitleaks)

---

### 1. Mechanics & Exit Code Logic

**Gitleaks** audits git history and source files against 150+ built-in rules. Exit code 1 indicates hardcoded secrets detected, while exit code >= 2 indicates system/execution errors.

![Gitleaks Exit Code Flow](/images/5-Workshop/5.5-Testing/gitleaks_flowchart.png)

*Figure 5.5.2: Gitleaks exit code processing flow.*

---

### 2. Test Scenario & Results

Injecting two dummy secrets into `app/src/config.js`:

```javascript
const API_CONFIG = {
  awsAccessKeyId: "AKIAIOSFODNN7EXAMPLE",
  awsSecretAccessKey: "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",
  googleMapsApiKey: "AIzaSyD1234567890abcdefghijklmnopqrstuvwx",
  backendUrl: process.env.REACT_APP_API_URL || "http://localhost:3001",
};

export default API_CONFIG;
```

**Result:** Gitleaks flags both secrets and aborts execution immediately with **exit code 1 (FAIL)**.
