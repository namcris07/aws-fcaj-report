---
title : "Configure 6 Security Gates"
date : 2026-07-06 
weight : 3
chapter : false
pre : " <b> 5.3.3. </b> "
---

# 5.3.3 Configure 6 Security Gates in Jenkinsfile

---

Integrating 6 security control checkpoints (Security Gates) following the **Defense in Depth** model into `ci/Jenkinsfile`. The pipeline comprises **22 stages** supporting 3 configuration presets:

| Preset | Registry Target | Security Mode | Deployment Purpose |
|---|---|---|---|
| `CUSTOM` | Local or ECR | `stub` | Rapid local development |
| `FULL_AWS_DEMO` | ECR | `enforce` | Pure AWS cloud demo |
| `FULL_PROJECT_DEMO` | ECR + Local | `enforce` | Full end-to-end demo (AWS + Local GitOps) |

---

### 22-Stage Jenkins Pipeline Flow

```groovy
pipeline {
    agent any
    environment {
        AWS_REGION        = 'ap-southeast-1'
        REGISTRY_TARGET   = 'ecr'
        IMAGE_NAME        = 'devsecops/tetris'
        SECURITY_MODE     = 'enforce'
        ENABLE_ECS_DEPLOY = 'true'
        ENABLE_DAST       = 'true'
        ENABLE_S3_UPLOAD  = 'true'
    }
    stages {
        stage('01. Checkout Monorepo')               { steps { checkout scm } }
        stage('02. Validate Inputs & Metadata')      { steps { sh './scripts/validate.sh' } }
        stage('03. Security Contract')               { steps { /* init scan env */ } }
        stage('04. Secrets Scan (Gitleaks)')         { steps { sh './ci/stages/secrets-scan.sh' } }
        stage('05. SCA Scan (Trivy FS)')             { steps { sh './ci/stages/sca-scan.sh' } }
        stage('06. SAST Scan (SonarQube)')           { steps { sh './ci/stages/sast-scan.sh' } }
        stage('07. IaC Scan (Checkov)')              { steps { sh './ci/stages/iac-scan.sh' } }
        stage('08. Build Docker Image')              { steps { sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -f app/Dockerfile app' } }
        stage('09. Container Scan (Trivy Image)')    { steps { sh './ci/stages/container-scan.sh' } }
        stage('10. ECR Login (AWS Auth)')            { steps { sh 'aws ecr get-login-password | docker login' } }
        stage('11. Push Image (ECR + :latest)')      { steps { sh 'docker push ${ECR_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}' } }
        stage('12. Mirror Image (Local GitOps)')     { steps { sh 'docker push localhost:5001/${IMAGE_NAME}:${IMAGE_TAG}' } }
        stage('13. Update GitOps Staging')           { steps { sh 'kustomize edit set image ... && git commit && git push' } }
        stage('14. Deploy ECS Staging')              { steps { sh 'aws ecs update-service --cluster devsecops-factory-cluster --service tetris-staging --force-new-deployment' } }
        stage('15. DAST Staging (OWASP ZAP)')        { steps { sh './ci/stages/dast-scan.sh' } }
        stage('16. Normalize Security Reports')      { steps { sh 'python3 ci/stages/normalize-reports.py' } }
        stage('17. Generate ASFF')                   { steps { sh 'python3 ci/stages/generate-asff.py' } }
        stage('18. Upload Reports to S3')            { steps { sh './ci/stages/upload-reports.sh' } }
        stage('19. Production Approval Gate')        { steps { input message: 'Approve deployment to ECS Production?' } }
        stage('20. Update GitOps Production')        { steps { sh 'kustomize edit set image ... && git commit && git push' } }
        stage('21. Deploy ECS Production')           { steps { sh 'aws ecs update-service --cluster devsecops-factory-cluster --service tetris-production --force-new-deployment' } }
        stage('22. Summary & Archive')               { steps { archiveArtifacts artifacts: 'scan-reports/**' } }
    }
}
```

---

### Security Mode Execution Behaviors

| SECURITY_MODE | Failure Handling Behavior | Purpose |
|---|---|---|
| `stub` | Completely bypassed (scripts skipped) | Quick local development |
| `report-only` | Pipeline continues; marks stage `UNSTABLE` | Metrics & telemetry collection |
| `enforce` | Pipeline terminates with `FAIL` immediately (exit code 1) | Production gate — Enforcement demo |

The reusable `runSecurityScript()` helper function manages execution for all 6 Security Gates in `ci/Jenkinsfile`:

```groovy
def runSecurityScript(Map config) {
  String scanMode = (config.mode ?: params.SECURITY_MODE ?: 'stub') as String
  if (scanMode == 'stub') { echo "[STUB] ${config.name} disabled."; return }
  if (scanMode == 'enforce') {
    withEnv(scanEnvironment) {
      sh(label: config.name, script: "#!/usr/bin/env bash\nset -euo pipefail\n'${config.script}'\n")
    }
  } else {
    catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
      withEnv(scanEnvironment) {
        sh(label: config.name, script: "#!/usr/bin/env bash\nset -euo pipefail\n'${config.script}'\n")
      }
    }
  }
}
```

---

### Real-World Scan Results on `tetris-app`

| Stage | Security Gate | Scanner Tool | Discovered Vulnerabilities | Severity | Gate Status |
|---|---|---|---|---|---|
| Stage 4 | Secrets Scan | Gitleaks | 2 secrets (GitHub PAT + AWS Access Key) | HIGH | **FAIL** |
| Stage 5 | SCA Scan | Trivy FS | 4 CVEs (`nth-check` HIGH, `serialize-javascript` HIGH) | HIGH | **FAIL** |
| Stage 6 | SAST Scan | SonarQube | 2 Vulnerabilities + 2 Security Hotspots (XSS) | MEDIUM–HIGH | **FAIL** |
| Stage 7 | IaC Scan | Checkov | 34 violations (Dockerfile + K8s Manifests + Terraform) | MEDIUM–HIGH | **SOFT-FAIL** |
| Stage 9 | Container Scan | Trivy Image | 40+ CVEs (15 CRITICAL, 25 HIGH) on `nginx:1.18.0` | CRITICAL | **FAIL** |
| Stage 15 | DAST Scan | OWASP ZAP | Missing security headers (WARN, 0 FAIL alerts) | MEDIUM | **report-only** |

> **Fail-Fast Defense Principle:** When `SECURITY_MODE=enforce` and `SECURITY_BLOCK_SEVERITIES=CRITICAL` are set, container images built on `nginx:1.18.0` are blocked at Stage 9 — preventing vulnerable images from being **pushed to Amazon ECR**.

---

### Actual Screenshots: Pipeline & Security Artifacts

![Jenkins Pipeline Build Overview](/images/5-Workshop/5.3-Step-by-Step/jenkins_build_overview.png)
*Figure 5.3.3a: Jenkins build overview detailing 22 stages and execution status.*

![Trivy 15 CRITICAL CVEs](/images/5-Workshop/5.3-Step-by-Step/trivy_critical_cves.png)
*Figure 5.3.3b: Jenkins Stage 9 Container Scan (Trivy Image) output identifying 15 CRITICAL CVEs on base image `nginx:1.18.0` (Debian Buster).*

![Jenkins Manual Approval Gate](/images/5-Workshop/5.3-Step-by-Step/jenkins_approval_gate.png)
*Figure 5.3.3c: Jenkins UI presenting Stage 19 Manual Approval Gate — requiring engineer approval before deploying to ECS Production.*

![Jenkins Pipeline Success](/images/5-Workshop/5.3-Step-by-Step/jenkins_pipeline_success.png)
*Figure 5.3.3d: Pipeline SUCCESS run across all 22 stages with artifacts archived in `scan-reports/`.*