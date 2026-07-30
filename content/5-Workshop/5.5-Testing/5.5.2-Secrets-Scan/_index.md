---
title : "Stage 4 – Secrets Scan (Gitleaks)"
date : 2026-07-06 
weight : 2
chapter : false
pre : " <b> 5.5.2. </b> "
---

# 5.5.2 Stage 4 – Hardcoded Secrets Scan (Gitleaks)

---

### 1. Tooling & Execution Mechanics

**Gitleaks** is integrated into Stage 4 to scan the complete repository source code and Git commit history, uncovering hardcoded sensitive information such as API keys, passwords, authentication tokens, private keys, database connection strings, or any pattern matching Gitleaks' rule database (featuring 150+ built-in detection rules).

A key architectural design feature is exit code differentiation: **exit code 1** indicates successful execution where Gitleaks detected hardcoded secrets, whereas **any non-zero, non-one exit code** signifies a system script execution failure.

![Gitleaks Exit Code Handling Flowchart](/images/5-Workshop/5.5-Testing/gitleaks_flowchart.png)

*Figure 5.5.2a: Gitleaks exit code handling logic within the Jenkins pipeline — distinguishing security findings from system execution errors.*

---

### 2. Test Scenario & Verification Findings

To test this stage, two fake credential secrets were seeded into `app/src/config.js`:

```javascript
// app/src/config.js - Application configuration
const API_CONFIG = {
  // [HARDCODED SECRET] GitHub Personal Access Token
  githubToken: "ghp_XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",

  // [HARDCODED SECRET] AWS Access Key ID
  awsAccessKeyId: "AKIAIOSFODNN7EXAMPLEFAKE",
};

export default API_CONFIG;
```

Upon pipeline execution, Gitleaks immediately uncovers both secrets and generates a detailed report pinpointing the exact file location and line numbers:

```json
[
  {
    "Description": "GitHub Personal Access Token",
    "StartLine": 2,
    "File": "app/src/config.js",
    "Secret": "ghp_XXXXXX...",
    "RuleID": "github-pat"
  },
  {
    "Description": "AWS Access Key ID",
    "StartLine": 3,
    "File": "app/src/config.js",
    "Secret": "AKIAIOSFODNN7EXAMPLEFAKE",
    "RuleID": "aws-access-key-id"
  }
]
```

**Result:** The pipeline emits **exit code 1 (FAIL)** and terminates immediately at Stage 4 within **30 seconds**, preventing execution from advancing to Stage 5 (SCA Scan) or Stage 8 (Build Docker Image).

---

### 3. Risk Analysis & Remediation Strategy

- **Security Risk:** Once secrets are committed to Git, they persist permanently in the repository object history even if deleted in subsequent commits.
- **Remediation:** Secrets must be ingested via environment variables (`process.env`) and injected securely at runtime via Jenkins Credential Stores or AWS Secrets Manager.
- **Custom Rules & Allowlist:** The `.gitleaks.toml` file allows defining custom regex rules and whitelisting benign false positives (e.g., public demo credentials in documentation).