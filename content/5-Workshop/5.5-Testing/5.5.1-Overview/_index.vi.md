---
title: "Tổng quan về chiến lược kiểm thử"
date: 2026-07-06
weight: 1
chapter: false
pre: " <b> 5.5.1. </b> "
---

# 5.5.1 Tổng quan về chiến lược kiểm thử

---

Hệ thống `devsecops-factory` được thiết kế theo nguyên tắc **"Shift Left Security"** giúp đưa kiểm tra bảo mật về sớm nhất có thể trong pipeline, từ lúc lập trình viên commit code cho đến khi image container sẵn sàng deploy lên **Amazon ECS Fargate**. Để xác minh tính đúng đắn và hiệu quả của hệ thống, nhóm đã thiết kế ứng dụng **tetris-app** (ReactJS + Nginx multi-stage build trong `app/`) làm mục tiêu kiểm thử có chủ ý. Ứng dụng được cài đặt sẵn các lỗ hổng ở nhiều tầng khác nhau, bao gồm mã nguồn, thư viện phụ thuộc, cấu hình container Dockerfile và cấu hình hạ tầng AWS (Terraform, ECS Task Definitions). Cách thiết kế này giúp kiểm chứng khả năng phát hiện của từng công cụ quét trong pipeline.

---

### 1. Kiến trúc pipeline và phân tầng bảo mật

Pipeline `devsecops-factory` được cấu thành từ 22 stages tự động hóa chạy trên Jenkins, trong đó 6 Security Gates cốt lõi tập trung vào bảo mật và kiểm định. Kiến trúc này áp dụng mô hình **"Defense in Depth"** – mỗi tầng bảo vệ một loại rủi ro riêng biệt, đảm bảo không có điểm mù nào trong toàn bộ chuỗi cung ứng phần mềm.

![Tổng quan Jenkins Build Overview 22 Stages](/images/5-Workshop/5.5-Testing/jenkins_build_overview.png)

*Hình 5.5.1a: Tổng quan quá trình build Jenkins với 22 stages tự động hóa phân chia rõ ràng giữa các trạm kiểm soát bảo mật và luồng triển khai.*

#### Bảng 5.5.1: Tóm tắt kết quả kiểm thử bảo mật toàn pipeline

| Stage | Loại quét | Công cụ | Lỗi phát hiện | Mức độ | Kết quả |
|---|---|---|---|---|---|
| **Stage 4** | Secrets Scan | Gitleaks | 2 secrets (GitHub PAT + AWS Key) | HIGH | **FAIL (exit code 1)** |
| **Stage 5** | SCA Scan | Trivy FS | 4 CVEs (HIGH) | HIGH | **FAIL** |
| **Stage 6** | SAST Scan | SonarQube | 2 Vulnerabilities + 2 Hotspots | MEDIUM–HIGH | **FAIL** |
| **Stage 7** | IaC Scan | Checkov | 34 failures | MEDIUM–HIGH | **SOFT-FAIL** |
| **Stage 9** | Container Scan | Trivy Image | 40+ CVEs (15 CRITICAL) trên nginx:1.18.0 | CRITICAL | **FAIL** |
| **Stage 15** | DAST Scan | OWASP ZAP | Missing security headers | MEDIUM | **report-only** |

---

### 2. Cấu trúc ứng dụng kiểm thử tetris-app

```text
tetris-app/
├── Jenkinsfile              # Pipeline CI/CD (22 stages)
├── app/
│   ├── Dockerfile           # Multi-stage build (Node:16 + Nginx:1.18.0)
│   ├── package.json         # Khai báo dependency (có version cũ lỗ hổng)
│   └── package-lock.json    # Lock file (nguồn scan SCA)
├── src/                     # Source code React (có lỗi SAST & Secrets)
├── infrastructure/
│   └── terraform/           # IaC Terraform cấu hình ECS Fargate, ECR, S3, IAM
└── ci/
    └── stages/              # 6 Security Gates scripts tích hợp Jenkins
```

---

### Ảnh minh chứng kiểm thử App & Docker build

![Build React App](/images/5-Workshop/5.3-Step-by-Step/app-01-run_build_app.jpg)
*Hình 5.5.1b: Biên dịch ứng dụng web React Tetris (`npm run build`).*

![Build Docker Image](/images/5-Workshop/5.3-Step-by-Step/app-02-build_docker.jpg)
*Hình 5.5.1c: Đóng gói Multi-stage Docker image cho ứng dụng.*

![Giao diện App local](/images/5-Workshop/5.3-Step-by-Step/app-02-docker_app.jpg)
*Hình 5.5.1d: Giao diện game Tetris chạy kiểm thử local qua Docker container.*

![AWS CloudWatch Logs Insights](/images/5-Workshop/5.3-Step-by-Step/aws_cloudwatch_insights.png)
*Hình 5.5.1e: Màn hình truy vấn nhật ký ứng dụng thực tế trên AWS CloudWatch Logs Insights (`/ecs/devsecops-factory`).*

![Jenkins Approval Gate](/images/5-Workshop/5.3-Step-by-Step/jenkins_approval_gate.png)
*Hình 5.5.1f: Stage 19 — Manual Approval Gate trên Jenkins UI — kỹ sư cần chọn Proceed để deploy Production.*