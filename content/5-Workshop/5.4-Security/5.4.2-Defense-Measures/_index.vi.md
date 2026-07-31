---
title : "Các biện pháp bảo mật áp dụng"
date : 2026-07-06 
weight : 2
chapter : false
pre : " <b> 5.4.2. </b> "
---

# 5.4.2 Biện pháp Bảo mật Theo Mô hình Defense in Depth

---

Dự án áp dụng mô hình **Defense in Depth (Phòng thủ Đa tầng)** với **7 tầng bảo vệ**. Mỗi tầng đảm nhiệm bảo vệ một loại rủi ro riêng biệt, đảm bảo không có điểm mù nào trong toàn bộ chuỗi cung ứng phần mềm.

---

### Mô hình 7 Tầng Bảo vệ (Defense in Depth)

| Tầng | Tên Tầng Bảo vệ | Công cụ / Giải pháp Triển khai | Trạm Kiểm soát (Gate) |
|---|---|---|---|
| **Tầng 1** | **Mã nguồn** | SonarQube SAST, Static Analysis | Gate 3: SAST Scan (Stage 6) |
| **Tầng 2** | **Thành phần phụ thuộc** | Trivy FS SCA (npm packages CVEs) | Gate 2: SCA Scan (Stage 5) |
| **Tầng 3** | **Container Image** | Trivy Image Scan, ECR Scan-on-Push | Gate 5: Container Scan (Stage 9) |
| **Tầng 4** | **Hạ tầng dưới dạng mã** | Checkov IaC Scan (Terraform, Dockerfile) | Gate 4: IaC Scan (Stage 7) |
| **Tầng 5** | **Secrets & Credentials** | Gitleaks (Git commit history) | Gate 1: Secrets Scan (Stage 4) |
| **Tầng 6** | **Runtime Application** | OWASP ZAP DAST Baseline Scan | Gate 6: DAST Scan (Stage 15) |
| **Tầng 7** | **AWS Cloud Control** | IAM Least Privilege, S3 SSE, Security Hub, VPC | AWS Native Security Layer |

---

### 1. Tầng 1: Bảo mật Mã nguồn (SonarQube SAST)

SonarQube thực hiện phân tích tĩnh mã nguồn React (JavaScript) với script `ci/stages/sast-scan.sh`:

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

### 2. Tầng 2: Phân tích Phụ thuộc (Trivy SCA)

Trivy quét `package-lock.json` để tìm lỗ hổng CVE với script `ci/stages/sca-scan.sh`:

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

### 3. Tầng 5: Phát hiện Secrets (Gitleaks) & Whitelist Rules

Gitleaks quét toàn bộ lịch sử commit Git với script `ci/stages/secrets-scan.sh` và file `.gitleaks.toml`:

```toml
# .gitleaks.toml - Cấu hình custom rules và allowlist
useDefault = true

rules = [
  { id = "custom-aws-key", description = "AWS Access Key ID", regex = "AKIA[0-9A-Z]{16}", severity = "CRITICAL" }
]

allowlist = { description = "Whitelist cho demo credentials", regexes = ["AKIAIOSFODNN7EXAMPLE"] }
```

---

### 4. Tầng 6: Kiểm thử Động Runtime (OWASP ZAP DAST)

Sau khi ứng dụng deploy lên ECS Staging, OWASP ZAP tự động thực hiện Baseline Scan:

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

### 5. Tầng 7: Bảo mật AWS Cloud Control Layer

Bảo mật hạ tầng AWS được đảm bảo thông qua 10 cơ chế cốt lõi:

| Cơ chế Bảo mật AWS | Mô tả Triển khai Chi tiết |
|---|---|
| **IAM Least Privilege** | 4 IAM Principals riêng biệt; Jenkins chỉ có quyền `ecr:Push`, `ecs:Deploy`, `s3:PutObject`. |
| **Không Hardcode Credentials** | Jenkins sử dụng IAM Access Key được lưu trong Jenkins Credentials Store mã hóa. |
| **Network Isolation** | VPC riêng biệt (10.0.0.0/16); Security Groups chỉ cho phép ECS tasks nhận traffic từ ALB. |
| **S3 Encryption** | SSE-S3 (AES256) mã hóa dữ liệu at-rest cho toàn bộ báo cáo bảo mật. |
| **S3 Public Block** | Bật cả 4 cấu hình Block Public Access trên S3 Report Bucket. |
| **AWS Budget Alert** | Cảnh báo email khi chi phí đạt ngưỡng 50%, 80% và 100% ngân sách tháng. |
| **CloudWatch Alarms** | Cảnh báo ở cấp ứng dụng: ECS Task dừng đột ngột, Lambda báo lỗi, Security Hub nhận finding CRITICAL. |
| **AWS Security Hub** | Quản lý tập trung thông số an ninh, tiếp nhận findings từ Lambda. |
| **ECR Scan-on-Push** | Tự động quét CVE ngay khi Docker image được đẩy lên ECR. |
| **Readonly Root Filesystem** | ECS Task Definition bật `"readonlyRootFilesystem": true` cho container. |
| **Non-root Container User** | Container Tetris chạy dưới `user: 101` (Nginx unprivileged user), không dùng root. |

---

### 6. CloudWatch Alarms — Giám sát Cấp Ứng dụng

Ngoài AWS Budget Alerts kiểm soát chi phí, dự án cấu hình thêm **3 CloudWatch Alarms** theo dõi trực tiếp sức khỏe của ECS task, lỗi Lambda, và findings bảo mật nghiêm trọng từ Security Hub:

```hcl
# infrastructure/terraform/main.tf

# Alarm 1: ECS Task dừng đột ngột (số task chạy < 1)
resource "aws_cloudwatch_metric_alarm" "ecs_task_stopped" {
  alarm_name          = "devsecops-ecs-task-stopped"
  comparison_operator = "LessThanThreshold"
  evaluation_periods  = 1
  metric_name         = "RunningTaskCount"
  namespace           = "ECS/ContainerInsights"
  period              = 60
  statistic           = "Average"
  threshold           = 1
  alarm_description   = "Cảnh báo khi số task đang chạy xuống dưới 1 (task crash hoặc OOM)"
  alarm_actions       = [aws_sns_topic.alerts.arn]
  dimensions = {
    ClusterName = "devsecops-factory-cluster"
    ServiceName = "tetris-production"
  }
}

# Alarm 2: Lambda Aggregator xảy ra lỗi thực thi
resource "aws_cloudwatch_metric_alarm" "lambda_errors" {
  alarm_name          = "devsecops-lambda-aggregator-errors"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 1
  metric_name         = "Errors"
  namespace           = "AWS/Lambda"
  period              = 60
  statistic           = "Sum"
  threshold           = 0
  alarm_description   = "Cảnh báo khi Lambda Aggregator xảy ra lỗi (nhập ASFF thất bại)"
  alarm_actions       = [aws_sns_topic.alerts.arn]
  dimensions = {
    FunctionName = "devsecops-factory-securityhub-importer"
  }
}

# Alarm 3: Security Hub nhận finding mức CRITICAL
resource "aws_cloudwatch_metric_alarm" "securityhub_critical" {
  alarm_name          = "devsecops-securityhub-critical-findings"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 1
  metric_name         = "Findings"
  namespace           = "AWS/SecurityHub"
  period              = 300
  statistic           = "Sum"
  threshold           = 0
  alarm_description   = "Cảnh báo khi có bất kỳ finding mức CRITICAL nào được nhập vào Security Hub"
  alarm_actions       = [aws_sns_topic.alerts.arn]
  dimensions = {
    ComplianceStatus = "FAILED"
    SeverityLabel    = "CRITICAL"
  }
}

# SNS Topic để nhận email thông báo
resource "aws_sns_topic" "alerts" {
  name = "devsecops-factory-alerts"
}
```

**Kiểm tra trạng thái các Alarm bằng AWS CLI:**

```bash
aws cloudwatch describe-alarms \
  --alarm-names \
    "devsecops-ecs-task-stopped" \
    "devsecops-lambda-aggregator-errors" \
    "devsecops-securityhub-critical-findings" \
  --profile devsecops-factory \
  --region ap-southeast-1 \
  --query 'MetricAlarms[].{Name:AlarmName,State:StateValue}'
```

| Alarm | Điều kiện kích hoạt | Kênh thông báo |
|---|---|---|
| `devsecops-ecs-task-stopped` | Số ECS Production task đang chạy < 1 | SNS → Email |
| `devsecops-lambda-aggregator-errors` | Lambda báo lỗi > 0 lần | SNS → Email |
| `devsecops-securityhub-critical-findings` | Security Hub nhận finding CRITICAL | SNS → Email |