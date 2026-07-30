---
title : "Stage 7 – Quét Hạ tầng (IaC Scan)"
date : 2026-07-06 
weight : 5
chapter : false
pre : " <b> 5.5.5. </b> "
---

# 5.5.5 Stage 7 — Quét cấu hình Hạ tầng (IaC Scan — Checkov)

---

### 1. Kết quả tổng quan

Công cụ **Checkov** được tích hợp tại Stage 7 (`ci/stages/iac-scan.sh`) để kiểm tra tĩnh toàn bộ mã nguồn định nghĩa hạ tầng dưới dạng mã (Infrastructure as Code - IaC) bao gồm các tệp **Terraform (`infrastructure/terraform/*.tf`)**, **Dockerfile (`app/Dockerfile`)** và **Amazon ECS Task Definitions (`infrastructure/task-definition.json`)**.

Checkov kiểm tra hạ tầng theo các tiêu chuẩn an ninh AWS Well-Architected Framework và CIS Benchmarks:

```text
terraform scan results:
Passed checks: 18, Failed checks: 4, Skipped checks: 0

dockerfile scan results:
Passed checks: 12, Failed checks: 2, Skipped checks: 0

ecs task definition scan results:
Passed checks: 14, Failed checks: 3, Skipped checks: 0
```

![Biểu đồ so sánh số lượng checks passed/failed theo loại file hạ tầng](/images/5-Workshop/5.5-Testing/iac_bar_chart.png)

*Hình 5.5.5: Biểu đồ so sánh số lượng checks passed/failed theo loại file hạ tầng AWS và Dockerfile.*

---

### 2. Nhóm lỗi Dockerfile Container

- **CKV_DOCKER_3 – Không tạo user riêng cho container:** Container chạy mặc định với quyền `root`.  
  *Giải pháp:* Thêm `USER 101` hoặc dùng base image `nginxinc/nginx-unprivileged:alpine`.
- **CKV_DOCKER_2 – Thiếu HEALTHCHECK:** Thiếu chỉ thị kiểm tra sức khỏe của container.  
  *Giải pháp:* Thêm chỉ thị `HEALTHCHECK --interval=30s --timeout=3s CMD wget --quiet --tries=1 --spider http://localhost:8080/ || exit 1`.

---

### 3. Nhóm lỗi Hạ tầng AWS Terraform & ECS Task Definition

#### Bảng 5.5.5: Các lỗi IaC Scan phát hiện trong cấu hình hạ tầng AWS

| Check ID | Mô tả lỗi | File ảnh hưởng | Giải pháp gia cố (Hardening) |
|---|---|---|---|
| **CKV_AWS_24** | Security Group mở cổng Ingress `0.0.0.0/0` | `terraform/main.tf` | Giới hạn Ingress IP CIDR cụ thể cho ALB |
| **CKV_AWS_18** | S3 Report Bucket chưa bật Access Logging | `terraform/s3.tf` | Thêm `logging` block hướng về S3 log bucket |
| **CKV_AWS_145** | S3 Bucket thiếu mã hóa SSE-KMS | `terraform/s3.tf` | Đảm bảo mã hóa SSE-KMS hoặc AES-256 enabled |
| **CKV_AWS_130** | ECS Task Definition không bật `readonlyRootFilesystem` | `task-definition.json` | Cấu hình `"readonlyRootFilesystem": true` |
| **CKV_AWS_336** | ECS Task Definition cho phép privileged mode | `task-definition.json` | Đặt `"privileged": false` |
| **CKV_AWS_55** | ECR Repository chưa bật KMS Key CMK | `terraform/ecr.tf` | Cấu hình encryption_configuration với AWS KMS |

---

> **Ghi chú quan trọng:** Checkov chạy với `--soft-fail` flag trong pipeline — pipeline tiếp tục nhưng báo cáo được ghi nhận đầy đủ. Kết quả kiểm thử thực tế phát hiện **34 FAILED, 12 PASSED**.

Chi tiết các lỗi chính phát hiện trong ứng dụng `tetris-app`:
- **CKV_DOCKER_3**: Thiếu chỉ thị `USER` → container chạy dưới quyền root (`app/Dockerfile`).
- **CKV_DOCKER_2**: Thiếu chỉ thị `HEALTHCHECK` trong Dockerfile.
- **CKV_K8S_8**: Thiếu `livenessProbe` trong Kubernetes Deployment.
- **CKV_K8S_15**: Container Image không chỉ định tag cố định (dùng `:latest`).
- **CKV_AWS_130**: VPC subnet gán IP công khai (cố ý để ECS Fargate pull images từ ECR).