---
title : "6 Vulnerability Categories & Threat Modeling"
date : 2026-07-06 
weight : 1
chapter : false
pre : " <b> 5.4.1. </b> "
---

# 5.4.1 Vulnerability Categories & Threat Modeling in the Project

---

Security is an essential requirement across all cloud-native architectures. The `devsecops-factory` framework and `tetris-app` demonstration application were deliberately designed with 6 distinct vulnerability categories to thoroughly test the defensive capabilities of the CI/CD pipeline.

---

### 1. Hardcoded Secrets Leakage

Under development velocity pressure, engineers frequently commit credentials directly into source code: AWS Access Keys, API Tokens, Database Passwords, Private Keys, or GitHub Personal Access Tokens.

- **Exploitation Mechanics:** Automated scanners continuously crawl public and misconfigured private repositories. Secret discovery occurs within seconds post-push.
- **Business Impact:** Attackers leverage leaked AWS Access Keys to compromise AWS Accounts, spin up unauthorized GPU instances for cryptomining, incurring thousands of dollars in AWS bills within hours.
- **Pipeline Test Target:** `app/src/config.js` is seeded with `AKIAIOSFODNN7EXAMPLEFAKE` and a GitHub PAT → **Gate 1 (Gitleaks)** detects the secrets and aborts the pipeline immediately.

---

### 2. Vulnerable Software Dependencies (SCA)

Modern React applications rely on hundreds of third-party npm packages, forming a complex dependency graph:
- **Direct Dependencies:** Declared explicitly in `package.json`.
- **Transitive Dependencies:** Sub-packages imported by dependencies, difficult to monitor manually.
- **Supply Chain Attacks:** Malicious actors publish typosquatted packages or compromise maintainer accounts to inject malware into popular libraries.
- **Pipeline Test Target:** `package-lock.json` pins `react-scripts 4.0.3`, `nth-check 1.0.2` (ReDoS HIGH), and `serialize-javascript 4.0.0` (XSS HIGH) → **Gate 2 (Trivy FS)** flags 4 HIGH CVEs.

---

### 3. Static Source Code Flaws (SAST & OWASP Code Vulnerabilities)

Static application security testing (SAST) of React/JavaScript identifies:
- **Security Hotspots:** Sensitive code patterns requiring manual review (usage of `eval()`, permissive CORS settings).
- **Vulnerabilities:** Exploitable bugs such as Cross-Site Scripting (XSS), SQL Injection, and Insecure Deserialization.
- **Pipeline Test Target:** `app/src/App.js` employs `dangerouslySetInnerHTML` without proper sanitization → **Gate 3 (SonarQube)** sets the Quality Gate status to FAILED.

---

### 4. Infrastructure-as-Code Misconfigurations (IaC)

The `tetris-app` manifests, Dockerfile, and Terraform files incorporate known configuration flaws:
- **`[CKV_DOCKER_3]`:** Missing `USER` instruction in Dockerfile → container defaults to running as `root`.
- **`[CKV_DOCKER_2]`:** Missing `HEALTHCHECK` directive in Dockerfile.
- **`[CKV_K8S_8]`:** Missing `livenessProbe` in Kubernetes Deployment spec.
- **`[CKV_K8S_15]`:** Container image tag uses `:latest` instead of pinning immutable digests.
- **`[CKV_AWS_130]`:** Security Group permits unrestricted ingress `0.0.0.0/0`.
- **Pipeline Test Target:** **Gate 4 (Checkov)** discovers 34 configuration violations with Soft-fail handling.

---

### 5. Container Image Vulnerabilities (OS Base Image CVEs)

Utilizing bloated base images like `nginx:1.18.0` (Debian Buster) inherits dozens of OS-level CVEs:

```bash
# nginx:1.18.0 (Debian) - contains numerous CRITICAL CVEs
$ trivy image nginx:1.18.0 --severity CRITICAL,HIGH
Total: 40+ CVEs (CRITICAL: 15, HIGH: 25+)

# nginxinc/nginx-unprivileged:alpine - minimal and hardened
$ trivy image nginxinc/nginx-unprivileged:alpine
Total: 0 CVEs (CRITICAL: 0, HIGH: 0)
```

- **Pipeline Test Target:** **Gate 5 (Trivy Image)** scans the ECR image, flags 15 CRITICAL CVEs on `nginx:1.18.0`, and **FAILS immediately at Stage 9**, blocking vulnerable image deployment to Amazon ECR.

---

### 6. OWASP Top 10 Flaws & Dynamic Testing (DAST)

OWASP ZAP dynamic application security testing evaluates the running Staging ALB endpoint to uncover runtime web application flaws:

| OWASP ID | Vulnerability Category | Severity | Detection Scanner in Pipeline |
|---|---|---|---|
| **A01:2021** | Broken Access Control | Critical | DAST (OWASP ZAP) |
| **A03:2021** | Injection (XSS, SQLi) | High | SAST (SonarQube) + DAST |
| **A05:2021** | Security Misconfiguration | Medium | IaC (Checkov) + DAST |
| **A06:2021** | Vulnerable Software Components | High | SCA (Trivy FS) + Container Scan |
| **A09:2021** | Security Logging Failures | Medium | Manual Review + CloudWatch Insights |