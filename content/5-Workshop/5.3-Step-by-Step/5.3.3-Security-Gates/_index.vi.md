---
title : "Cấu hình 6 Security Gates"
date : 2026-07-06 
weight : 3
chapter : false
pre : " <b> 5.3.3. </b> "
---

# 5.3.3 Cấu hình 6 Security Gates trong Jenkinsfile

---

Tích hợp 6 trạm kiểm soát bảo mật (Security Gates) theo mô hình **Defense in Depth** cùng luồng triển khai ECS vào `ci/Jenkinsfile`. Pipeline gồm **22 stages** với 3 preset cấu hình:

| Preset | Registry | Security Mode | Mục đích |
|---|---|---|---|
| `CUSTOM` | local hoặc ECR | stub | Phát triển local nhanh |
| `FULL_AWS_DEMO` | ECR | enforce | Demo AWS thuần túy |
| `FULL_PROJECT_DEMO` | ECR + local | enforce | Demo đầy đủ (AWS + GitOps) |

---

### Sơ đồ 22 Stages Jenkins Pipeline

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
        stage('19. Production Approval Gate')        { steps { input message: 'Phê duyệt deploy lên ECS Production?' } }
        stage('20. Update GitOps Production')        { steps { sh 'kustomize edit set image ... && git commit && git push' } }
        stage('21. Deploy ECS Production')           { steps { sh 'aws ecs update-service --cluster devsecops-factory-cluster --service tetris-production --force-new-deployment' } }
        stage('22. Summary & Archive')               { steps { archiveArtifacts artifacts: 'scan-reports/**' } }
    }
}
```

---

### Security Mode — 3 chế độ bảo mật

| SECURITY_MODE | Hành vi khi scan thất bại | Mục đích |
|---|---|---|
| `stub` | Bỏ qua hoàn toàn (không chạy script) | Local dev nhanh |
| `report-only` | Build tiếp, đánh dấu stage UNSTABLE | Thu thập metrics |
| `enforce` | Build FAIL ngay lập tức (exit code 1) | Production gate — Demo đầy đủ |

Cơ chế `runSecurityScript()` dùng chung cho tất cả 6 Security Gates trong `ci/Jenkinsfile`:

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

### Kết quả thực tế từ kiểm thử với ứng dụng tetris-app

| Stage | Gate | Công cụ | Phát hiện thực tế | Mức độ | Kết quả |
|---|---|---|---|---|---|
| Stage 4 | Secrets Scan | Gitleaks | 2 secrets (GitHub PAT + AWS Access Key) | HIGH | **FAIL** |
| Stage 5 | SCA Scan | Trivy FS | 4 CVEs (nth-check HIGH, serialize-javascript HIGH) | HIGH | **FAIL** |
| Stage 6 | SAST Scan | SonarQube | 2 Vulnerabilities + 2 Security Hotspots (XSS) | MEDIUM–HIGH | **FAIL** |
| Stage 7 | IaC Scan | Checkov | 34 failures (Dockerfile + K8s YAML + Terraform) | MEDIUM–HIGH | **SOFT-FAIL** |
| Stage 9 | Container Scan | Trivy Image | 40+ CVEs (15 CRITICAL, 25 HIGH) trên nginx:1.18.0 | CRITICAL | **FAIL** |
| Stage 15 | DAST | OWASP ZAP | Missing security headers (WARN, không có FAIL alerts) | MEDIUM | **report-only** |

> **Nguyên tắc fail-fast:** Với `SECURITY_MODE=enforce` và `SECURITY_BLOCK_SEVERITIES=CRITICAL`, container image nginx:1.18.0 bị chặn tại Stage 9 — image **không được push lên ECR**, bảo vệ production khỏi CVE CRITICAL.

---

### Ảnh minh chứng luồng Jenkins Pipeline & Security Artifacts

![Jenkins Pipeline Build Overview](/images/5-Workshop/5.3-Step-by-Step/jenkins_build_overview.png)
*Hình 5.3.3a: Tổng quan quá trình build Jenkins với các stages (xanh/đỏ) rõ ràng.*

![Trivy 15 CRITICAL CVEs](/images/5-Workshop/5.3-Step-by-Step/trivy_critical_cves.png)
*Hình 5.3.3b: Jenkins Stage 9 Container Scan (Trivy Image) phát hiện 15 CRITICAL CVEs trên base image nginx:1.18.0 (Debian Buster).*

![Jenkins Manual Approval Gate](/images/5-Workshop/5.3-Step-by-Step/jenkins_approval_gate.png)
*Hình 5.3.3c: Jenkins UI hiển thị Manual Approval Gate (Stage 19) — kỹ sư cần chọn Proceed để deploy Production.*

![Jenkins Pipeline Success](/images/5-Workshop/5.3-Step-by-Step/jenkins_pipeline_success.png)
*Hình 5.3.3d: Pipeline hoàn thành SUCCESS với đủ 22 stages và artifacts trong scan-reports/.*