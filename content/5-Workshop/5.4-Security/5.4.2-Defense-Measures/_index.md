---
title : "Defense-in-Depth Measures Implemented"
date : 2026-07-06 
weight : 2
chapter : false
pre : " <b> 5.4.2. </b> "
---

# 5.4.2 Defense-in-Depth Implementation Model

---

The project implements a comprehensive **Defense in Depth (Multi-layered Defense)** model featuring **7 Security Layers**. Each layer mitigates distinct risk vectors, ensuring zero blind spots across the software supply chain.

---

### The 7 Security Layers (Defense-in-Depth Model)

| Layer | Security Layer Name | Implemented Tooling / Solution | Security Checkpoint (Gate) |
|---|---|---|---|
| **Layer 1** | **Source Code Security** | SonarQube SAST, Static Analysis | Gate 3: SAST Scan (Stage 6) |
| **Layer 2** | **Software Dependencies** | Trivy FS SCA (npm package CVEs) | Gate 2: SCA Scan (Stage 5) |
| **Layer 3** | **Container Image** | Trivy Image Scan, ECR Scan-on-Push | Gate 5: Container Scan (Stage 9) |
| **Layer 4** | **Infrastructure as Code** | Checkov IaC Scan (Terraform, Dockerfile) | Gate 4: IaC Scan (Stage 7) |
| **Layer 5** | **Secrets & Credentials** | Gitleaks (Git commit history analysis) | Gate 1: Secrets Scan (Stage 4) |
| **Layer 6** | **Runtime Application** | OWASP ZAP DAST Baseline Scan | Gate 6: DAST Scan (Stage 15) |
| **Layer 7** | **AWS Cloud Control** | IAM Least Privilege, S3 SSE, Security Hub, VPC | AWS Native Security Layer |

---

### 1. Layer 1: Source Code Security (SonarQube SAST)

SonarQube performs static application analysis on React JavaScript code using `ci/stages/sast-scan.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

sonar-scanner \
  -Dsonar.projectKey=devsecops-tetris \
  -Dsonar.sources=app/src \
  -Dsonar.host.url="${SONAR_HOST}" \
  -Dsonar.login="${SONAR_TOKEN}" \
  -Dsonar.projectVersion="${IMAGE_TAG}" \
  -Dsonar.qualitygate.wait=true

curl -u "${SONAR_TOKEN}:" \
  "${SONAR_HOST}/api/issues/search?componentKeys=devsecops-tetris&types=VULNERABILITY,BUG" \
  -o "${SCAN_REPORT_DIR}/sonar-issues.json"
```

---

### 2. Layer 2: Dependency Analysis (Trivy SCA)

Trivy scans `package-lock.json` to uncover dependency CVEs via `ci/stages/sca-scan.sh`:

```bash
#!/usr/bin/env bash
trivy fs \
  --format json \
  --output "${SCAN_REPORT_DIR}/trivy-sca-report.json" \
  --severity "${SECURITY_BLOCK_SEVERITIES}" \
  --exit-code "$([ "${SECURITY_MODE}" = 'enforce' ] && echo 1 || echo 0)" \
  "${SCAN_DIR}"
```

---

### 3. Layer 5: Secrets Detection (Gitleaks) & Whitelisting Rules

Gitleaks scans git history using `ci/stages/secrets-scan.sh` guided by `.gitleaks.toml`:

```toml
# .gitleaks.toml - Custom rules and allowlist configuration
useDefault = true

rules = [
  { id = "custom-aws-key", description = "AWS Access Key ID", regex = "AKIA[0-9A-Z]{16}", severity = "CRITICAL" }
]

allowlist = { description = "Allowlist for demo credentials", regexes = ["AKIAIOSFODNN7EXAMPLE"] }
```

---

### 4. Layer 6: Dynamic Runtime Application Testing (OWASP ZAP DAST)

Following deployment to the ECS Staging environment, OWASP ZAP automatically runs an unauthenticated Baseline Scan:

```bash
#!/usr/bin/env bash
docker run --rm \
  -v "${REPORT_DIR}:/zap/wrk" \
  --network host \
  ghcr.io/zaproxy/zaproxy:stable \
  zap-baseline.py \
  -t "${TARGET_URL}" \
  -J "${REPORT_DIR}/zap-report.json" \
  -r "${REPORT_DIR}/zap-report.html" \
  -I
```

---

### 5. Layer 7: AWS Cloud Control Security Layer

Cloud infrastructure posture is guarded by 10 core controls:

| AWS Security Control | Implementation & Enforcement Specification |
|---|---|
| **IAM Least Privilege** | 4 isolated IAM Principals; Jenkins restricted to `ecr:Push`, `ecs:Deploy`, `s3:PutObject`. |
| **No Hardcoded Credentials** | Jenkins authenticates using IAM Access Keys stored in encrypted Jenkins Credential Stores. |
| **Network Isolation** | Dedicated VPC (`10.0.0.0/16`); Security Groups enforce inbound traffic strictly from ALBs to ECS tasks. |
| **S3 Encryption** | SSE-S3 (AES256) encrypts data at rest across all stored security reports. |
| **S3 Public Access Block** | All 4 S3 Block Public Access flags explicitly enabled on the report bucket. |
| **AWS Budget Alerts** | Automated email alerts emitted upon reaching 50%, 80%, and 100% monthly budget limits. |
| **AWS Security Hub** | Centralized security dashboard aggregating findings automatically dispatched by Lambda. |
| **ECR Scan-on-Push** | Immediate CVE scanning triggered on every container image push to ECR. |
| **Read-Only Root Filesystem** | ECS Task Definition enforces `"readonlyRootFilesystem": true` for runtime containers. |
| **Non-Root Container User** | Tetris application container executes under `user: 101` (Nginx unprivileged), barring root permissions. |