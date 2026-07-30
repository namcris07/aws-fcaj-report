---
title : "Kiểm thử quy trình DevSecOps"
date : 2026-07-06 
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

# 5.5 Kiểm thử toàn diện quy trình bảo mật DevSecOps trên AWS

---

## Tổng quan

Để xác minh tính đúng đắn và hiệu quả của hệ thống `devsecops-factory`, nhóm đã thiết kế ứng dụng **tetris-app** (React + Nginx multi-stage build, trong `app/`) làm **mục tiêu kiểm thử có chủ ý**. Ứng dụng được cài đặt sẵn các lỗ hổng ở nhiều tầng:
- **Mã nguồn:** secrets hardcode (`app/src/config.js`), XSS vulnerability (dangerouslySetInnerHTML)
- **Phụ thuộc:** npm packages cũ có CVE (`react-scripts 4.0.3`, `nth-check 1.0.2`, `serialize-javascript 2.1.1`)
- **Container:** Base image `nginx:1.18.0` (Debian) có 40+ CVEs thay vì nginx-unprivileged:alpine
- **Hạ tầng:** Kubernetes/Dockerfile thiếu hardening (không có USER directive, HEALTHCHECK)

Mục này phân tích chi tiết kết quả kiểm thử qua **6 Security Gates** trong pipeline Jenkins **22 stages**, quá trình tập trung báo cáo lên **Amazon S3**, chuẩn hóa ASFF và nhập tự động qua **AWS Lambda** vào **AWS Security Hub**, và triển khai ứng dụng an toàn lên **Amazon ECS Fargate** kết hợp với **AWS CloudWatch Container Insights**.

---

## Bảng tổng hợp kết quả kiểm thử

| Stage | Gate | Công cụ | Phát hiện thực tế | Mức độ | Kết quả |
|---|---|---|---|---|---|
| **Stage 4** | Secrets Scan | Gitleaks | 2 secrets (GitHub PAT + AWS Access Key) | HIGH | **FAIL** |
| **Stage 5** | SCA Scan | Trivy FS | 4 CVEs (nth-check, serialize-javascript) | HIGH | **FAIL** |
| **Stage 6** | SAST Scan | SonarQube | 2 Vulnerabilities + 2 Security Hotspots | MEDIUM–HIGH | **FAIL** |
| **Stage 7** | IaC Scan | Checkov | 34 failures (Dockerfile + K8s + Terraform) | MEDIUM–HIGH | **SOFT-FAIL** |
| **Stage 9** | Container Scan | Trivy Image | 40+ CVEs (15 CRITICAL, 25 HIGH) nginx:1.18.0 | CRITICAL | **FAIL** |
| **Stage 15** | DAST | OWASP ZAP | Missing security headers (không có FAIL alerts) | MEDIUM | **report-only** |

**Kết quả tổng thể:**
- ✓ Phát hiện đúng 5/6 loại lỗ hổng với enforce mode
- ✓ Pipeline dừng đúng điểm (Stage 9 — CRITICAL CVEs)
- ✓ Báo cáo lưu tập trung trên Amazon S3
- ✓ AWS Security Hub nhận findings qua Lambda trong vòng 60 giây

---

## Danh mục các mục kiểm thử

1. [5.5.1 Tổng quan về chiến lược kiểm thử](5.5.1-Overview/)
2. [5.5.2 Stage 4 — Quét Secrets cứng hóa (Gitleaks)](5.5.2-Secrets-Scan/)
3. [5.5.3 Stage 5 — Quét phụ thuộc SCA (Trivy FS)](5.5.3-SCA-Scan/)
4. [5.5.4 Stage 6 — Phân tích tĩnh SAST (SonarQube)](5.5.4-SAST-Scan/)
5. [5.5.5 Stage 7 — Quét cấu hình IaC (Checkov)](5.5.5-IaC-Scan/)
6. [5.5.6 Stage 9 — Quét Container Image (Trivy Image)](5.5.6-Container-Scan/)
7. [5.5.7 Kết quả tổng hợp & Triển khai ECS Fargate](5.5.7-Summary/)
