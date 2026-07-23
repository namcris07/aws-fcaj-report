---
title : "Tổng quan về chiến lược kiểm thử"
date : 2026-07-06 
weight : 1
chapter : false
pre : " <b> 5.5.1. </b> "
---

# 5.5.1 Tổng quan về chiến lược kiểm thử

---

Hệ thống `devsecops-factory` được thiết kế theo nguyên tắc **"Shift Left Security"** giúp đưa kiểm tra bảo mật về sớm nhất có thể trong pipeline, từ lúc lập trình viên commit code cho đến khi image container sẵn sàng deploy. Để xác minh tính đúng đắn và hiệu quả của hệ thống, nhóm đã thiết kế ứng dụng `tetris-app` làm mục tiêu kiểm thử có chủ ý. Đây là một game Tetris web xây dựng bằng React và phục vụ file tĩnh qua Nginx. Ứng dụng được cài đặt sẵn các lỗ hổng ở nhiều tầng khác nhau, bao gồm mã nguồn, thư viện phụ thuộc, cấu hình container và cấu hình hạ tầng Kubernetes. Cách thiết kế này giúp kiểm chứng khả năng phát hiện của từng công cụ quét trong pipeline.

---

### 1. Kiến trúc pipeline và phân tầng bảo mật

Pipeline `devsecops-factory` được cấu thành từ 11 stage chạy tuần tự trên Jenkins, trong đó 5 stage cốt lõi tập trung vào bảo mật. Kiến trúc này áp dụng mô hình **"Defense in Depth"** – mỗi tầng bảo vệ một loại rủi ro riêng biệt, đảm bảo không có điểm mù nào trong toàn bộ chuỗi cung ứng phần mềm.

![Toàn cảnh pipeline DevSecOps 11 stage của devsecops-factory chạy trên Jenkins](/images/5-Workshop/5.5-Testing/pipeline_stages.png)

*Hình 5.5.1: Toàn cảnh pipeline DevSecOps 11 stage của devsecops-factory chạy trên Jenkins – các stage bảo mật (màu đỏ) xen kẽ với các stage hạ tầng (màu xanh).*

#### Bảng 5.5.1: Tóm tắt kết quả kiểm thử bảo mật toàn pipeline

| Stage | Loại quét | Công cụ | Lỗi phát hiện | Mức độ | Kết quả |
|---|---|---|---|---|---|
| **Stage 3** | Secrets Scan | Gitleaks | 2 secrets | MEDIUM – HIGH | **FAIL (exit code 1)** |
| **Stage 4** | SCA Scan | Trivy FS | 4 CVEs (HIGH) | HIGH | **FAIL** |
| **Stage 5** | SAST Scan | SonarQube | 4+ code issues | MEDIUM – HIGH | **FAIL** |
| **Stage 6** | IaC Scan | Checkov | 34 failures | MEDIUM – HIGH | **FAIL (Soft-fail)** |
| **Stage 8** | Container Image Scan | Trivy Image | 0 CVEs (Alpine) / 40+ CVEs (Debian) | CRITICAL – HIGH | **PASS (Alpine) / FAIL (Debian)** |

---

### 2. Cấu trúc ứng dụng kiểm thử `tetris-app`

Ứng dụng `tetris-app` có kiến trúc điển hình của một ứng dụng web hiện đại được container hóa:

```text
tetris-app/
├── Jenkinsfile              # Pipeline CI/CD (11 stages)
├── app/
│   ├── Dockerfile           # Multi-stage build (Node:16 + Nginx)
│   ├── package.json         # Khai báo dependency (có version cũ lỗ hổng)
│   └── package-lock.json    # Lock file (nguồn scan SCA)
├── src/                     # Source code React (có lỗi SAST)
├── kubernetes/
│   ├── base/
│   │   ├── deployment.yaml  # K8s Deployment (thiếu hardening)
│   │   └── service.yaml     # K8s Service (dùng namespace default)
│   └── overlays/
│       ├── staging/         # Kustomize overlay staging
│       └── production/      # Kustomize overlay production
└── ci/
    └── stages/              # Gọi sang scripts từ devsecops-factory
```

Dockerfile của ứng dụng sử dụng kiến trúc multi-stage build:

```dockerfile
# STAGE 1: Build (Compile React code)
FROM node:16 AS builder

WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# STAGE 2: Deploy (Serve static files via Nginx)
# FROM nginxinc/nginx-unprivileged:alpine <- DÙNG ĐỂ PASS container scan
FROM nginx:1.18.0    # <- ĐANG DÙNG - Kích hoạt nhiều CVE CRITICAL
COPY --from=builder /app/build /usr/share/nginx/html
EXPOSE 8080
CMD ["nginx", "-g", "daemon off;"]
# [LỖI CKV_DOCKER_3]: Thiếu USER -> chạy với quyền root
# [LỖI CKV_DOCKER_2]: Thiếu HEALTHCHECK
```
