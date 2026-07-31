---
title : "Kiến trúc & Dịch vụ AWS"
date : 2026-07-06 
weight : 2
chapter : false
pre : " <b> 5.1.2. </b> "
---

# 5.1.2 Sơ đồ Kiến trúc & Lựa chọn Dịch vụ AWS

---

### 1. Sơ đồ kiến trúc giải pháp

![DevSecOps Architecture](/images/2-Proposal/devsecops_pipeline_architecture.png)

*Hình 5.1.2: Kiến trúc tổng thể hệ thống DevSecOps Factory trên AWS.*

**Luồng hoạt động end-to-end (12 bước):**

1. **Commit/Push** — Developer push code lên Git SCM.
2. **Trigger/Poll SCM** — Jenkins On-premise poll SCM, nhận webhook kích hoạt pipeline.
3. **Run Security Gates** — Jenkins thực thi tuần tự 7 bước kiểm định bảo mật:
   - 3.a **Secret Scan** (git-secret) → phát hiện secrets bị hardcode
   - 3.b **SCA Scan** (Trivy Filesystem) → phát hiện CVE trong dependency
   - 3.c **SAST Scan** (SonarQube) → phân tích tĩnh mã nguồn
   - 3.d **IaC Scan** (Checkov) → kiểm tra cấu hình Dockerfile, K8s manifests
   - 3.e **Build Image** (Docker) → đóng gói ứng dụng React thành container image
   - 3.f **Container Scan** (Trivy Image) → quét lỗ hổng trong container image
   - 3.g **DAST Scan** (OWASP ZAP) → quét bảo mật ứng dụng đang chạy
4. **Push Image** — Đẩy Docker image lên Amazon ECR (tag theo commit SHA).
5. **Deploy to ECS Staging** — Triển khai image mới lên ECS Fargate Staging (AZ 1) qua ALB test traffic.
6. **Pull Image** — ECS Fargate kéo image từ Amazon ECR.
7. **DAST Endpoint Test** — OWASP ZAP quét bảo mật endpoint staging đang chạy thực tế.
8. **Promote** — Sau khi DAST pass & Manual Approval, promote lên ECS Fargate Production (AZ 2).
9. **Upload Reports/ASFF** — Jenkins đẩy toàn bộ báo cáo JSON/HTML và file ASFF lên Amazon S3.
10. **S3 Event Trigger** — S3 kích hoạt AWS Lambda tự động khi có file mới.
11. **BatchImportFindings** — Lambda chuyển đổi báo cáo sang chuẩn ASFF và đẩy vào AWS Security Hub.
12. **Logs/Metrics** — ECS Fargate gửi logs và metrics sang Amazon CloudWatch.

---

### 2. Giải thích lý do lựa chọn Dịch vụ AWS

#### 2.1 Lớp Identity, Audit & Cost Guardrails

| Dịch vụ AWS | Vai trò trong hệ thống |
|---|---|
| **AWS IAM Roles** | Phân quyền hạt mịn (Least Privilege) cho Jenkins Agent, ECS Task Execution Role và Lambda Execution Role. Không sử dụng static keys — toàn bộ truy cập qua IAM Role. |
| **AWS CloudTrail** | Ghi lại toàn bộ API calls trên tài khoản AWS, phục vụ audit và điều tra sự cố bảo mật. |
| **AWS Config** | Đánh giá liên tục trạng thái cấu hình tài nguyên AWS theo các quy tắc compliance, phát hiện configuration drift. |
| **AWS Budgets** | Thiết lập cảnh báo ngân sách ở các mức 50%, 80%, 100% để kiểm soát chi phí hạ tầng trong suốt đợt thực tập. |

#### 2.2 Lớp Artifact & Report Storage

| Dịch vụ AWS | Lý do lựa chọn & So sánh giải pháp |
|---|---|
| **Amazon ECR** | **Private Registry Bảo mật:** Lưu trữ Docker Image riêng tư, hỗ trợ mã hóa Server-Side, Immutable Tag và Scan-on-Push tích hợp sẵn. Image được tag theo Git Commit SHA để truy xuất nguồn gốc. |
| **Amazon S3** | **Lưu trữ Báo cáo Tập trung:** Độ tin cậy 99.999999999% (11 số 9), SSE-S3 encryption, S3 Lifecycle Policy tự động dọn dẹp báo cáo sau 30 ngày. Lưu toàn bộ raw reports và file ASFF chuẩn hóa. |

#### 2.3 Lớp Application Runtime - VPC

| Dịch vụ AWS | Lý do lựa chọn & So sánh giải pháp |
|---|---|
| **Application Load Balancer (ALB)** | Định tuyến traffic Layer 7: phân tách test traffic sang ECS Fargate Staging (AZ 1) và production traffic sang ECS Fargate Production (AZ 2). |
| **Amazon ECS Fargate** | **Serverless Container Runtime:** Không mất phí cố định duy trì cluster ($72/tháng của EKS), chỉ tính phí theo thời gian task chạy. Dễ dàng scale về `desired-count 0` sau demo để tối ưu ngân sách. Triển khai tại 2 Availability Zone: **Staging (AZ 1)** và **Production (AZ 2)**. |

#### 2.4 Lớp Security Findings Processing

| Dịch vụ AWS | Lý do lựa chọn & So sánh giải pháp |
|---|---|
| **AWS Lambda** | **Event-Driven Processing:** Tự động kích hoạt qua S3 Event Notification khi có báo cáo mới, chuyển đổi sang chuẩn ASFF và gọi `BatchImportFindings` đẩy vào Security Hub. Nằm trong Free Tier (1M request/tháng). |
| **AWS Security Hub** | **Central Findings Dashboard:** Tổng hợp và chuẩn hóa toàn bộ security findings từ 6 security gates theo định dạng ASFF. Dashboard duy nhất cho toàn bộ phát hiện bảo mật. |
| **Amazon CloudWatch** | **Observability & Budget Control:** Container Insights thu thập logs/metrics từ ECS Fargate Tasks, Logs Insights truy vấn log ứng dụng, Budget Alerts cảnh báo chi phí. |

---

### 3. Bảo mật IAM & Khả năng mở rộng Vận hành

- **Phân quyền IAM Least Privilege:** Mỗi dịch vụ (ECS, Lambda, Jenkins) chỉ được cấp đúng các action tối thiểu cần thiết (`ecr:PutImage`, `s3:PutObject`, `ecs:UpdateService`, `securityhub:BatchImportFindings`).
- **Audit & Compliance liên tục:** AWS CloudTrail ghi lại mọi API call; AWS Config đánh giá configuration drift theo thời gian thực.
- **Kiểm soát chi phí chủ động:** AWS Budgets gửi cảnh báo email ở mức 50%/80%/100% ngân sách để tránh chi phí vượt kiểm soát.
- **Khả năng Auto Scaling & Event-Driven:** ECS Fargate tự động scale task theo lưu lượng traffic qua ALB; luồng phân tích lỗ hổng hoàn toàn hướng sự kiện (Event-Driven) qua S3 Event → Lambda → Security Hub.
- **Kiểm soát không Hardcode Credentials:** Sử dụng IAM Roles for Tasks và AWS Secrets Manager thay cho static keys. Jenkins credentials được quản lý qua Jenkins Credentials Store.
- **High Availability:** Ứng dụng triển khai trên 2 Availability Zones (Staging tại AZ 1, Production tại AZ 2) với ALB phân tải tự động.
