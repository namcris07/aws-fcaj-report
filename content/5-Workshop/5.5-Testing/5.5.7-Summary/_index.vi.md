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

Quá trình kiểm thử ứng dụng **[tetris-app](https://github.com/lamelihuynh/tetris-app.git)** đã xác minh thành công khả năng phát hiện chính xác các lỗ hổng ở tất cả các tầng của pipeline `devsecops-factory`:

1. **Gate 1 - Gitleaks (Secrets Scan):** Chặn các secret bị hardcode (`AKIA...`, API Keys).
2. **Gate 2 - Trivy FS (SCA Scan):** Phát hiện 4 CVEs mức HIGH trong `package-lock.json`.
3. **Gate 3 - SonarQube (SAST Scan):** Phát hiện các lỗi logic và độ phức tạp mã nguồn (Cognitive Complexity > 15).
4. **Gate 4 - Checkov (IaC Scan):** Quét hạ tầng Terraform, Dockerfile và ECS Task Definitions.
5. **Gate 5 - Trivy Image (Container Scan):** Quét image container trên Amazon ECR, xác nhận 0 CVEs đối với Base Image Alpine.
6. **Gate 6 - OWASP ZAP (DAST Scan):** Quét động địa chỉ Application Load Balancer (ALB) của môi trường ECS Fargate Staging.

---

### 2. Luồng lưu trữ báo cáo S3 & AWS Lambda Aggregator

- Tất cả kết quả quét từ 6 Security Gates được tự động tải lên **Amazon S3** (`devsecops-reports-*`).
- **AWS Lambda Function** tự động parse các báo cáo JSON, tổng hợp số lượng lỗ hổng CRITICAL/HIGH và đẩy log cảnh báo về **AWS CloudWatch Logs**.

---

### 3. Kết quả triển khai ứng dụng lên Amazon ECS Fargate

Sau khi gia cố bảo mật và vượt qua 6 Security Gates:
- Pipeline tự động push image đã quét sạch lên **Amazon ECR**.
- Cập nhật ECS Task Definition và triển khai thành công lên 2 ECS Services: `tetris-staging` và `tetris-production`.
- Hệ thống được giám sát thời gian thực qua **AWS CloudWatch Container Insights**.

![Sơ đồ tổng kết kết quả kiểm thử từng stage bảo mật trong pipeline](/images/5-Workshop/5.5-Testing/summary_stages.png)

_Hình 5.5.7: Sơ đồ tổng kết kết quả kiểm thử từng stage bảo mật trong pipeline devsecops-factory._

