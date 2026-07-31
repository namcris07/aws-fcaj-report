---
title : "Đánh giá & Bài học kinh nghiệm"
date : 2026-07-06 
weight : 7
chapter : false
pre : " <b> 5.7. </b> "
---

# 5.7 Đánh giá & Bài học kinh nghiệm

---

## Tổng quan

Phần này ghi lại trung thực những khó khăn kỹ thuật gặp phải trong suốt 9 tuần triển khai dự án **DevSecOps Factory**, cách giải quyết từng vấn đề, và những bài học quan trọng rút ra để ứng dụng cho các dự án tiếp theo.

---

## 1. Những khó khăn chính đã gặp phải

### 1.1 ECS Fargate Health Check thất bại liên tục
**Vấn đề:** Task Definition ban đầu cấu hình container React chạy với user `root` trên cổng 80 (Nginx tiêu chuẩn). Tuy nhiên, nhóm đã chuyển sang sử dụng `nginxinc/nginx-unprivileged` (user non-root 101) vốn bind vào cổng **8080** — khiến ALB Health Check báo `unhealthy` và ECS service rơi vào vòng lặp restart liên tục.

**Cách giải quyết:**
- Cập nhật ALB Target Group health check từ cổng `80` sang cổng `8080`.
- Thêm `init container` (`volume-permissions`) chạy với root để cấp quyền ghi lên `/var/cache/nginx` và `/tmp` trước khi container chính `tetris` khởi động.
- Thiết lập `readonlyRootFilesystem: true` trên container chính để tăng cường bảo mật.

**Bài học rút ra:** Luôn đảm bảo **container port**, **`containerPort` trong Task Definition**, và **cổng ALB Target Group** khớp nhau trước lần deploy đầu tiên. Base image non-root cần có pattern khởi tạo quyền volume riêng.

---

### 1.2 Lambda Aggregator lỗi ASFF Schema Validation
**Vấn đề:** Hàm Lambda Aggregator ban đầu trả về lỗi `InvalidInput` khi gọi `BatchImportFindings` vì schema ASFF (Amazon Security Finding Format) yêu cầu nhiều trường bắt buộc (`SchemaVersion`, `ProductArn`, `GeneratorId`, `Types`, `CreatedAt`, `UpdatedAt`, `Severity`, `Title`, `Description`, `Resources`) không có trong output thô của Trivy/Gitleaks.

**Cách giải quyết:**
- Xây dựng script `normalize_report.py` để ánh xạ (map) các trường từ output scanner thô sang cấu trúc ASFF chuẩn.
- Thêm một pipeline stage chuyên biệt (`Normalize Reports & ASFF Conversion`) trước bước upload S3, đảm bảo file `securityhub-asff.json` luôn hợp lệ theo schema.
- Kiểm tra schema cục bộ bằng `aws securityhub batch-import-findings --dry-run` trước khi bật Lambda trigger thực tế.

**Bài học rút ra:** Không bao giờ giả định output thô của scanner tương thích với schema ASFF của AWS Security Hub. Cần xây dựng lớp chuẩn hóa (normalization layer) ngay từ giai đoạn thiết kế pipeline.

---

### 1.3 Rủi ro lộ lọt Credentials trong Jenkins Pipeline
**Vấn đề:** Trong giai đoạn đầu phát triển, AWS Access Keys được lưu tạm trong file `.env` và mount trực tiếp vào container Jenkins — tạo nguy cơ credentials vô tình bị commit lên Git hoặc lộ lọt qua output `docker inspect`.

**Cách giải quyết:**
- Chuyển toàn bộ credentials sang **Jenkins Credentials Store** (kiểu Secret Text).
- Inject credentials qua block `withCredentials([string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID')])` trong Jenkinsfile — đảm bảo credentials không bao giờ xuất hiện trong build logs hay biến môi trường Docker.
- Thêm `.env` vào `.gitignore` ngay lập tức và chạy `git filter-branch` để xóa sạch các commit credentials đã lỡ push.
- Cấu hình Gitleaks (Stage 3) để bắt mọi rủi ro lộ lọt secrets trong các commit tiếp theo.

**Bài học rút ra:** Quản lý credentials phải được thiết lập từ ngày đầu. Jenkins Credentials Store là bắt buộc, không phải tùy chọn. Gitleaks như một pipeline gate ngăn credentials rò rỉ đến remote repository.

---

### 1.4 FARGATE_SPOT bị thu hồi Capacity khi Demo
**Vấn đề:** Trong tuần 6 khi chuẩn bị demo, ECS Staging service đang chạy trên `FARGATE_SPOT` bị gián đoạn giữa chừng do AWS thu hồi Spot capacity. Ứng dụng mất kết nối khoảng 3 phút trong khi ECS khởi động lại task mới.

**Cách giải quyết:**
- Triển khai **Capacity Provider Strategy** cho Staging service: ưu tiên `FARGATE_SPOT` (weight 1) với `FARGATE` làm dự phòng (weight 0) — AWS tự động chuyển sang Fargate tiêu chuẩn khi Spot không khả dụng.
- Thêm script `make demo-scale` để nhanh chóng scale Production (FARGATE tiêu chuẩn) lên cho các buổi demo quan trọng.
- Chuẩn bị bộ slide ảnh chụp backup cho từng giai đoạn pipeline để demo offline khi cần.

**Bài học rút ra:** `FARGATE_SPOT` lý tưởng để tiết kiệm chi phí cho workload không quan trọng, nhưng không bao giờ nên là capacity provider duy nhất cho demo hoặc môi trường production mà không có chiến lược dự phòng.

---

### 1.5 Checkov IaC Scan chặn Pipeline quá sớm
**Vấn đề:** Với `--exit-code 1` được bật, Checkov báo cáo 34 lỗi IaC trên Terraform, Dockerfile và Kubernetes manifests — chặn pipeline hoàn toàn trong tuần 4, ngăn cản mọi thử nghiệm deploy lên cloud.

**Cách giải quyết:**
- Áp dụng chiến lược bật gate theo từng giai đoạn:
  - **Giai đoạn 1 (Tuần 1–5):** Chế độ `--soft-fail` — chỉ báo cáo lỗi, không chặn pipeline.
  - **Giai đoạn 2 (Tuần 6–9):** Bật `--exit-code 1` có chọn lọc chỉ cho các kiểm tra mức **HIGH**; bỏ qua các phát hiện LOW/MEDIUM đã được chấp nhận bằng file cấu hình `checkov.yaml`.
- Ghi lại toàn bộ lý do bỏ qua (skip justification) trong `checkov.yaml` kèm comment giải thích.

**Bài học rút ra:** Áp dụng security gate theo từng bước. Bắt đầu với chế độ `soft-fail` để xây dựng baseline, sau đó dần dần siết chặt tiêu chí khi codebase trưởng thành hơn.

---

## 2. Những gì hoạt động tốt

| Lĩnh vực | Kết quả |
|---|---|
| **Terraform IaC** | 50 tài nguyên AWS được provisioned chỉ bằng một lệnh `terraform apply` — không cần thao tác thủ công trên Console |
| **Makefile Automation** | `make bootstrap`, `make status`, `make demo-reset` giảm thời gian onboarding thành viên mới từ vài giờ xuống còn vài phút |
| **Chiến lược Local-first (k3d)** | Cho phép kiểm thử toàn bộ pipeline offline, tiết kiệm hoàn toàn chi phí cloud trong 5 tuần phát triển |
| **FARGATE_SPOT cho Staging** | Tiết kiệm ~70% chi phí compute cho workload staging |
| **Lambda Aggregator** | Hoạt động hoàn toàn trong phạm vi AWS Free Tier — chi phí Lambda = $0 trong suốt 9 tuần |
| **6 Security Gates** | Phát hiện thành công tất cả lỗ hổng đã cài đặt có chủ ý: 2 secrets, 4 CVEs, 40+ container CVEs, 2 SAST findings, 34 IaC misconfigurations |

---

## 3. Hướng phát triển trong tương lai

- **Bật DAST ở chế độ Blocking:** Nâng cấp OWASP ZAP từ `report-only` sang `--exit-code 1` cho các cảnh báo FAIL (VD: thiếu header `X-Frame-Options`, `Content-Security-Policy`).
- **Tích hợp SonarQube Quality Gate:** Thêm bước polling SonarQube Quality Gate API trong pipeline để chặn build không đạt ngưỡng coverage hoặc số lượng issue cho phép.
- **Triển khai đa vùng (Multi-Region):** Mở rộng Terraform để provisioning môi trường ECS Fargate thứ hai tại `us-east-1` phục vụ kiểm thử Disaster Recovery.
- **Cost Anomaly Detection tự động:** Thay thế Budget Alert thủ công bằng **AWS Cost Anomaly Detection** để nhận thông báo thông minh (ML-driven) khi chi phí tăng bất thường.
- **Security Hub Custom Insights:** Xây dựng dashboard Security Hub Insights lọc findings CRITICAL/HIGH theo build ID pipeline để triage nhanh hơn.
- **Đánh giá chuyển sang GitHub Actions:** Xem xét migrate từ Jenkins sang GitHub Actions native runners để giảm thêm chi phí hạ tầng CI/CD.
