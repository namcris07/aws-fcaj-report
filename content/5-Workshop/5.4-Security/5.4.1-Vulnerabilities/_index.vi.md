---
title : "Các lỗ hổng bảo mật thường gặp"
date : 2026-07-06 
weight : 1
chapter : false
pre : " <b> 5.4.1. </b> "
---

# 5.4.1 Các lỗ hổng bảo mật thường gặp và mô hình kiểm thử trong dự án

---

Hệ thống `devsecops-factory` và ứng dụng `tetris-app` được thiết kế có chủ ý chứa 6 nhóm lỗ hổng bảo mật điển hình tại các tầng khác nhau nhằm thử nghiệm toàn diện khả năng phát hiện của đường ống CI/CD:

---

### 1. Lọt lộ thông tin nhạy cảm (Hardcoded Secrets) - Target: Gate 1 (Gitleaks)

- **Mô tả trong dự án:** Lập trình viên vô tình commit các chuỗi thông tin bí mật như AWS Access Keys (`AKIA...`), Private Keys hoặc JWT secret trực tiếp vào file cấu hình hoặc lịch sử Git.
- **Rủi ro:** Kẻ tấn công dùng bot rà quét công khai trên GitHub để chiếm đoạt tài khoản AWS hoặc quyền truy cập hệ thống chỉ trong vài giây.
- **Công cụ phát hiện:** **Gitleaks** được tích hợp tại Stage 1 (`ci/stages/01-secrets-scan.sh`), tự động quét từng commit diff và toàn bộ lịch sử git.

---

### 2. Lỗ hổng thư viện phụ thuộc (SCA Vulnerabilities) - Target: Gate 2 (Trivy FS)

- **Mô tả trong dự án:** Tệp `package.json` và `package-lock.json` của `tetris-app` sử dụng các thư viện Node.js phiên bản cũ chứa các CVEs đã công bố (ví dụ: các lỗ hổng HIGH/CRITICAL trong `express`, `lodash` hoặc `react-scripts`).
- **Rủi ro:** Tin tặc khai thác lỗ hổng trong thư viện bên thứ ba để thực thi mã từ xa (RCE) hoặc gây từ chối dịch vụ (DoS).
- **Công cụ phát hiện:** **Trivy FileSystem** (`ci/stages/02-sca-scan.sh`) quét mã nguồn và cây phụ thuộc gián tiếp trước khi build image.

---

### 3. Lỗi chất lượng mã nguồn tĩnh (SAST Code Flaws) - Target: Gate 3 (SonarQube)

- **Mô tả trong dự án:** Các đoạn mã nguồn React/JavaScript chứa lỗi logic, thiếu xác thực dữ liệu đầu vào (Unsanitized Inputs), hoặc vi phạm chuẩn mã sạch (Code Smells / Vulnerabilities).
- **Rủi ro:** Tạo nguy cơ tấn công Cross-Site Scripting (XSS) hoặc làm rò rỉ dữ liệu phiên làm việc của người dùng.
- **Công cụ phát hiện:** **SonarQube Scanner** (`ci/stages/03-sast-scan.sh`) phân tích cú pháp mã nguồn tĩnh và đối chiếu với bộ quy tắc an toàn.

---

### 4. Cấu hình sai hạ tầng mã hóa (IaC Misconfigurations) - Target: Gate 4 (Checkov)

- **Mô tả trong dự án:** Các tệp Terraform (`infrastructure/terraform/main.tf`) và Kubernetes/Docker manifests (`app/Dockerfile`, `kubernetes/base/deployment.yaml`) chứa các lỗi cấu hình:
  - Dockerfile thiếu chỉ thị `USER` (chạy với quyền root - lỗi `CKV_DOCKER_3`).
  - Dockerfile thiếu kiểm tra trạng thái sức khỏe `HEALTHCHECK` (`CKV_DOCKER_2`).
  - Security Group trong Terraform cho phép truy cập ingress mở rộng `0.0.0.0/0`.
- **Công cụ phát hiện:** **Checkov** (`ci/stages/04-iac-scan.sh`) quét tĩnh hạ tầng Terraform và file Dockerfile/K8s với chế độ Soft-fail (không block build nhưng ghi nhận báo cáo).

---

### 5. Lỗ hổng Container Image (Base Image CVEs) - Target: Gate 5 (Trivy Image)

- **Mô tả trong dự án:** Dockerfile phiên bản chưa gia cố sử dụng base image `nginx:1.18.0` (Debian-based) chứa hơn 40+ lỗ hổng hệ điều hành cấp độ CRITICAL và HIGH.
- **Giải pháp gia cố (Hardening):** Chuyển sang sử dụng base image siêu nhẹ và gia cố `nginxinc/nginx-unprivileged:alpine` chạy dưới user non-root `user: 101` với `readonlyRootFilesystem: true`.
- **Công cụ phát hiện:** **Trivy Image** (`ci/stages/05-container-scan.sh`) quét trực tiếp container image sau khi đóng gói.

---

### 6. Lỗ hổng ứng dụng web động (DAST Web Flaws) - Target: Gate 6 (OWASP ZAP)

- **Mô tả trong dự án:** Môi trường staging sau khi triển khai lên Amazon ECS Fargate được quét động từ bên ngoài bằng **OWASP ZAP** (`ci/stages/06-dast-scan.sh`) để phát hiện các lỗ hổng như thiếu HTTP Security Headers (X-Frame-Options, CSP, HSTS), hoặc rò rỉ thông tin máy chủ Nginx.
