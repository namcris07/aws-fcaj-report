---
title : "Kết quả tổng hợp"
date : 2026-07-06 
weight : 7
chapter : false
pre : " <b> 5.5.7. </b> "
---

# 5.5.7 Kết quả tổng hợp & Kiểm thử triển khai trên AWS ECS Fargate

---

### 1. Tổng kết 6 Security Gates trong Pipeline CI/CD

Quá trình kiểm thử ứng dụng **tetris-app** đã xác minh thành công khả năng phát hiện chính xác các lỗ hổng ở tất cả các tầng của pipeline `devsecops-factory`:

| Stage | Gate | Công cụ | Phát hiện thực tế | Mức độ | Kết quả |
|---|---|---|---|---|---|
| **Stage 4** | Secrets Scan | Gitleaks | 2 secrets (GitHub PAT + AWS Access Key) | HIGH | **FAIL** |
| **Stage 5** | SCA Scan | Trivy FS | 4 CVEs (nth-check HIGH, serialize-javascript HIGH) | HIGH | **FAIL** |
| **Stage 6** | SAST Scan | SonarQube | 2 Vulnerabilities (XSS) + 2 Security Hotspots | MEDIUM–HIGH | **FAIL** |
| **Stage 7** | IaC Scan | Checkov | 34 failures (Dockerfile + K8s YAML + Terraform) | MEDIUM–HIGH | **SOFT-FAIL** |
| **Stage 9** | Container Scan | Trivy Image | 40+ CVEs (15 CRITICAL, 25 HIGH) trên nginx:1.18.0 | CRITICAL | **FAIL** |
| **Stage 15** | DAST Scan | OWASP ZAP | Missing security headers (không có FAIL alerts) | MEDIUM | **report-only** |

**Kết quả tổng thể:**
- ✓ Phát hiện đúng 5/6 loại lỗ hổng với `SECURITY_MODE=enforce`
- ✓ Pipeline dừng đúng điểm (Stage 9 — CRITICAL CVEs), chặn đẩy container lỗi lên Amazon ECR
- ✓ Báo cáo tập trung được tự động lưu trữ trên **Amazon S3**
- ✓ **AWS Security Hub** tiếp nhận ASFF findings thông qua **AWS Lambda** trong vòng 60 giây

---

### 2. Luồng lưu trữ báo cáo S3 & AWS Lambda Aggregator

Tất cả kết quả quét từ 6 Security Gates được tự động upload lên **Amazon S3** (`devsecops-reports-*`) trong Stage 18. AWS Lambda kích hoạt ngay khi file ASFF mới xuất hiện trong `reports/asff/`.

![S3 tất cả reports](/images/5-Workshop/5.5-Testing/s3_all_reports.png)
*Hình 5.5.7a: AWS S3 Console lưu trữ toàn bộ báo cáo bảo mật theo cấu trúc thư mục (secrets/, sca/, sast/, container/, dast/, iac/, asff/, normalized/).*

![AWS Security Hub Dashboard](/images/5-Workshop/5.5-Testing/securityhub_dashboard.png)
*Hình 5.5.7b: AWS Security Hub Findings Dashboard — findings được nhóm theo mức độ nghiêm trọng (Critical, High, Medium, Low).*

---

### 3. Kết quả triển khai ứng dụng lên Amazon ECS Fargate

Sau khi gia cố bảo mật (dùng `nginxinc/nginx-unprivileged:alpine` thay vì `nginx:1.18.0`) và vượt qua tất cả Security Gates:

- Pipeline tự động push image đã quét sạch (0 CRITICAL CVEs) lên **Amazon ECR**
- Cập nhật GitOps Staging → Argo CD auto-sync → ECS Task Definition mới → deploy `tetris-staging`
- Manual Approval Gate (Stage 19) → deploy `tetris-production`
- Hệ thống được giám sát thời gian thực qua **AWS CloudWatch Container Insights**

![Jenkins Pipeline SUCCESS](/images/5-Workshop/5.5-Testing/jenkins_pipeline_success.png)
*Hình 5.5.7c: Jenkins Pipeline hoàn thành SUCCESS với đủ 22 stages và toàn bộ artifacts trong scan-reports/.*

![Ứng dụng Tetris Staging trên browser](/images/5-Workshop/5.5-Testing/app_staging_browser.png)
*Hình 5.5.7d: Ứng dụng Tetris đang chạy trên môi trường ECS Fargate Staging, truy cập qua ALB URL.*

![Ứng dụng Tetris Production trên browser](/images/5-Workshop/5.5-Testing/app_production_browser.png)
*Hình 5.5.7e: Ứng dụng Tetris đang chạy trên môi trường ECS Fargate Production sau khi Manual Approval Gate được phê duyệt.*