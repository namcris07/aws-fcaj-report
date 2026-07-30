---
title : "Các bước chuẩn bị"
date : 2026-07-06 
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

#### 5.2 Các bước chuẩn bị

Chương này mô tả toàn bộ những bước cần thực hiện trước khi khởi chạy hệ thống **DevSecOps Factory**. Việc chuẩn bị đúng và đầy đủ từ đầu giúp tránh các lỗi phát sinh trong quá trình chạy demo.

---

#### 0. Tải về Mã nguồn Dự án (Clone Repository)

Mã nguồn dự án được lưu trữ tập trung tại kho GitHub chính thức. Clone repository và chuyển sang branch GitOps:

```bash
git clone https://github.com/loi-bui0703/CICD-DevSecOps-using-AWS-services.git
cd CICD-DevSecOps-using-AWS-services

# Chuyển sang branch GitOps (Argo CD local theo dõi branch này)
git switch cicd-gitops

# Tạo file .env từ template
make setup-env
# Mở file .env và điền các giá trị bắt buộc
```

> **Lưu ý về branch:** Các Argo CD Application local được cấu hình theo dõi branch cicd-gitops. Nếu cần dùng branch khác, cần cập nhật tham số GITOPS_BRANCH trong Jenkins pipeline trước khi chạy.

---

#### 1. Yêu cầu Phần cứng và Hệ điều hành

| Yêu cầu | Mô tả |
|---|---|
| **Hệ điều hành** | macOS hoặc Linux (WSL2 trên Windows). Đã kiểm chứng trên macOS với Docker Desktop. |
| **RAM** | Tối thiểu 10–12 GB cho Docker (khuyến nghị 16 GB). Jenkins, SonarQube, k3d và OWASP ZAP cần nhiều bộ nhớ. |
| **Disk** | Ít nhất 25 GB trống cho Docker images và volumes. |
| **Mạng** | Kết nối Internet ổn định cho lần tải đầu tiên: Jenkins image, SonarQube, k3d, ZAP (~3–5 GB). |

Các cổng TCP cần trống trên máy host:

| Cổng | Dịch vụ | Mô tả |
|---|---|---|
| 80 | ingress-nginx | HTTP vào các app local (*.localhost) |
| 443 | ingress-nginx | HTTPS |
| 3000 | Grafana | Dashboard monitoring |
| 5001 | Local Registry | Docker registry nội bộ |
| 6443 | k3d API Server | Kubernetes API |
| 8080 | Jenkins | CI/CD UI |
| 8443 | Argo CD | GitOps UI (port-forward) |
| 9000 | SonarQube | SAST scanner UI |
| 9090 | Prometheus | Metrics scraping |
| 9418 | Git daemon | Local git-server cho Jenkins |

---

#### 2. Yêu cầu Công cụ và Cài đặt

Cài đặt và kiểm tra phiên bản các công cụ bắt buộc:

```bash
docker version          # >= 24.0
git --version           # >= 2.40
make --version          # GNU Make >= 4.0
kubectl version --client # >= 1.28
helm version             # >= 3.12
k3d version              # >= 5.6
terraform version        # >= 1.0
aws --version            # AWS CLI v2 >= 2.13
jq --version             # >= 1.6
python3 --version        # >= 3.10
```

---

#### 3. Cấu hình Biến Môi trường (.env)

Dự án sử dụng file .env để quản lý tất cả credentials và cấu hình. File này **không được commit** vào Git.

```dotenv
# === LOCAL PLATFORM ===
JENKINS_ADMIN_PASS=<mật-khẩu-mạnh-8+-ký-tự>
GRAFANA_ADMIN_PASSWORD=<mật-khẩu-mạnh>

# === SAST (SonarQube) ===
SONAR_TOKEN=<token-từ-SonarQube-UI>

# === AWS (chỉ cần cho FULL_PROJECT_DEMO) ===
AWS_ACCESS_KEY_ID=<access-key-của-IAM-jenkins-ci>
AWS_SECRET_ACCESS_KEY=<secret-key>
```

---

#### 4. Cấu hình IAM Policy cho Jenkins CI

IAM User (hoặc Role) dùng cho Jenkins cần được gán policy với các quyền tối thiểu theo nguyên tắc **Least Privilege**. Trong dự án này, IAM Policy và User được quản lý hoàn toàn bởi Terraform (infrastructure/terraform/main.tf).

---

#### 5. Khởi tạo Hạ tầng AWS bằng Terraform

```bash
# Đăng nhập AWS SSO
aws sso login --profile devsecops-factory
export AWS_PROFILE=devsecops-factory

# Kiểm tra đăng nhập đúng account
aws sts get-caller-identity

# Khởi tạo và apply Terraform
terraform -chdir=infrastructure/terraform init
terraform -chdir=infrastructure/terraform apply -auto-approve
```

---

#### 6. Khởi động Nền tảng Local

```bash
# Khởi động toàn bộ (Jenkins, SonarQube, Prometheus, Grafana, k3d, Argo CD)
make bootstrap

# Kiểm tra trạng thái
make status
```

| Dịch vụ | URL | Thông tin đăng nhập |
|---|---|---|
| Jenkins | http://localhost:8080 | admin /  |
| SonarQube | http://localhost:9000 | admin / (thiết lập lần đầu) |
| Prometheus | http://localhost:9090 | Không yêu cầu |
| Grafana | http://localhost:3000 | admin /  |
| Argo CD | http://localhost:8443 | admin / (lấy từ K8s secret) |
| Tetris Staging | http://tetris-staging.localhost | — |
| Tetris Production | http://tetris-production.localhost | — |

---

### Ảnh chụp màn hình thực tế: Nền tảng sau bootstrap

![Platform Status](/images/5-Workshop/5.2-Prerequisite/platform_status.png)
*Hình 5.2a: Giao diện make status hiển thị toàn bộ nền tảng local hoạt động bình thường.*

![Jenkins UI Home](/images/5-Workshop/5.2-Prerequisite/jenkins_ui_home.png)
*Hình 5.2b: Giao diện trang chủ Jenkins sau khi đăng nhập.*

![Argo CD Apps](/images/5-Workshop/5.2-Prerequisite/argocd_apps.png)
*Hình 5.2c: Giao diện Argo CD hiển thị 2 Applications (staging và production) ở trạng thái Synced & Healthy.*

![Make Status Success](/images/5-Workshop/5.2-Prerequisite/make_status_success.png)
*Hình 5.2d: Terminal hiển thị kết quả make status sau khi bootstrap thành công — tất cả containers đều healthy.*

---

#### 7. Lỗi thường gặp khi chuẩn bị

| Hiện tượng | Kiểm tra/Xử lý |
|---|---|
| permission denied với Docker socket | Mở Docker Desktop và chờ engine sẵn sàng |
| Cổng đã được sử dụng | lsof -nP -iTCP:<port> -sTCP:LISTEN, dừng dịch vụ xung đột |
| Jenkins chưa healthy | make logs SVC=jenkins; lần đầu tải image mất vài phút |
| SonarQube chưa sẵn sàng | make logs SVC=sonarqube; kiểm tra RAM và SONAR_TOKEN trong .env |
| demo-trigger báo working tree bẩn | Commit thay đổi trước; Jenkins chỉ thấy code đã commit |
| AWS credential lỗi trong Jenkins | Cập nhật .env, kiểm tra IAM key, chạy docker compose restart jenkins |
| Argo CD app OutOfSync | make gitops-seed, make argocd-apps |
| Pod ImagePullBackOff | curl http://localhost:5001/v2/_catalog; chạy pipeline để mirror image |