---
title : "Các lỗ hổng bảo mật thường gặp"
date : 2026-07-06 
weight : 1
chapter : false
pre : " <b> 5.4.1. </b> "
---

# 5.4.1 Các lỗ hổng bảo mật thường gặp và mô hình kiểm thử trong dự án

---

Bảo mật là yêu cầu không thể thiếu trong mọi hệ thống phần mềm vận hành trên môi trường đám mây. Dự án devsecops-factory và ứng dụng kiểm thử 	etris-app được thiết kế có chủ ý chứa 6 nhóm lỗ hổng bảo mật điển hình tại các tầng nhằm thử nghiệm toàn diện khả năng phòng thủ của đường ống CI/CD.

---

### 1. Lộ lọt Thông tin Nhạy cảm (Hardcoded Secrets)

Trong áp lực tiến độ, lập trình viên thường vô tình hardcode thông tin xác thực trực tiếp vào mã nguồn: AWS Access Keys, API Keys, Database Passwords, Private Keys hoặc GitHub Personal Access Tokens.

- **Cơ chế khai thác:** Các bot quét tự động công khai liên tục lùng sục repository public và private bị lộ. Việc phát hiện có thể diễn ra trong vòng vài giây sau khi push code.
- **Hậu quả:** Kẻ tấn công có thể sử dụng Access Key để chiếm quyền AWS Account, khởi tạo EC2 instances đào tiền ảo, làm phát sinh hóa đơn AWS hàng nghìn đô la trong vài giờ.
- **Mục tiêu kiểm thử:** `app/src/config.js` tiêm sẵn `AKIAIOSFODNN7EXAMPLEFAKE` và GitHub PAT → **Gate 1 (Gitleaks)** phát hiện và dừng pipeline lập tức.

---

### 2. Lỗ hổng Thành phần Phụ thuộc (Vulnerable Dependencies - SCA)

Ứng dụng web React hiện đại phụ thuộc vào hàng trăm thư viện npm, tạo thành một cây phụ thuộc phức tạp (dependency tree):
- **Direct dependencies:** Khai báo trực tiếp trong package.json.
- **Transitive dependencies:** Thư viện con của thư viện con, rất khó kiểm soát thủ công.
- **Supply Chain Attack:** Kẻ tấn công tạo package giả mạo tên gần giống (*typosquatting*) hoặc chèn mã độc vào package phổ biến.
- **Mục tiêu kiểm thử:** package-lock.json sử dụng `react-scripts 4.0.3`, `nth-check 1.0.2` (ReDoS HIGH), `serialize-javascript 4.0.0` (XSS HIGH) → **Gate 2 (Trivy FS)** phát hiện 4 CVEs mức HIGH.

---

### 3. Lỗ hổng Mã nguồn Tĩnh (SAST & OWASP Code Vulnerabilities)

Phân tích tĩnh mã nguồn React/JavaScript phát hiện:
- **Security Hotspots:** Các điểm nguy hiểm tiềm ẩn cần review thủ công (sử dụng eval(), cấu hình CORS lỏng lẻo).
- **Vulnerabilities:** Lỗ hổng có thể khai thác như Cross-Site Scripting (XSS), SQL Injection, Insecure Deserialization.
- **Mục tiêu kiểm thử:** `app/src/App.js` sử dụng dangerouslySetInnerHTML không qua sanitize → **Gate 3 (SonarQube)** đánh dấu Quality Gate FAILED.

---

### 4. Cấu hình Sai Hạ tầng (IaC Misconfigurations)

Ứng dụng `tetris-app` và hạ tầng Terraform/Kubernetes được thiết kế sẵn các lỗi cấu hình:
- **[CKV_DOCKER_3]:** Thiếu chỉ thị USER trong Dockerfile → container chạy mặc định với quyền root.
- **[CKV_DOCKER_2]:** Thiếu chỉ thị HEALTHCHECK trong Dockerfile.
- **[CKV_K8S_8]:** Thiếu livenessProbe trong Kubernetes Deployment.
- **[CKV_K8S_15]:** Container Image sử dụng :latest tag thay vì pin phiên bản cố định.
- **[CKV_AWS_130]:** Security Group cho phép Ingress IP 0.0.0.0/0.
- **Mục tiêu kiểm thử:** **Gate 4 (Checkov)** phát hiện 34 lỗi cấu hình với chế độ Soft-fail.

---

### 5. Lỗ hổng Container Image (CVEs trong Base Image)

Sử dụng base image lớn như nginx:1.18.0 (Debian Buster) kéo theo hàng chục lỗ hổng CVE của hệ điều hành nền:

```bash
# nginx:1.18.0 (Debian) - chứa nhiều CVE CRITICAL
$ trivy image nginx:1.18.0 --severity CRITICAL,HIGH
Total: 40+ CVEs (CRITICAL: 15, HIGH: 25+)

# nginxinc/nginx-unprivileged:alpine - sạch sẽ và an toàn
$ trivy image nginxinc/nginx-unprivileged:alpine
Total: 0 CVEs (CRITICAL: 0, HIGH: 0)
```

- **Mục tiêu kiểm thử:** **Gate 5 (Trivy Image)** quét Docker image trên ECR, phát hiện 15 CRITICAL CVEs trên nginx:1.18.0 và **FAIL ngay tại Stage 9**, không cho phép push image lỗi lên ECR.

---

### 6. Lỗ hổng OWASP Top 10 & DAST Scan

OWASP ZAP DAST scan thực hiện kiểm thử động trên URL Staging đang chạy để phát hiện các lỗ hổng tầng Web:

| OWASP ID | Tên Lỗ hổng | Mức độ | Công cụ phát hiện trong Pipeline |
|---|---|---|---|
| **A01:2021** | Broken Access Control | Critical | DAST (OWASP ZAP) |
| **A03:2021** | Injection (XSS, SQLi) | High | SAST (SonarQube) + DAST |
| **A05:2021** | Security Misconfiguration | Medium | IaC (Checkov) + DAST |
| **A06:2021** | Vulnerable Components | High | SCA (Trivy FS) + Container Scan |
| **A09:2021** | Security Logging Failures | Medium | Manual Review + CloudWatch Insights |