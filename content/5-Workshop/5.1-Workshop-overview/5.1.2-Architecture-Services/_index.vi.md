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

---

### 2. Giải thích lý do lựa chọn Dịch vụ AWS

| Dịch vụ AWS | Lý do lựa chọn & So sánh giải pháp |
|---|---|
| **Amazon ECS Fargate** | **Serverless Container Runtime:** Tự động mở rộng và chỉ tính phí theo thời gian task chạy. Không mất phí cố định duy trì cluster tĩnh ($72/tháng của EKS), dễ dàng scale số lượng task về 0 khi không sử dụng để tối ưu chi phí. |
| **Amazon ECR** | **Private Registry Bảo mật:** Đăng ký lưu trữ Docker Image riêng tư, hỗ trợ mã hóa Server-Side, tính năng Immutable Tag và tính năng Scan-on-Push tích hợp sẵn. |
| **Amazon S3** | **Lưu trữ Báo cáo Tập trung:** Độ tin cậy 99.999999999% (11 số 9), hỗ trợ SSE-S3 encryption và S3 Lifecycle Policy tự động dọn dẹp báo cáo sau 30 ngày để tối ưu chi phí. |
| **AWS Lambda** | **Event-Driven Processing:** Tự động kích hoạt khi có báo cáo mới trên S3 (S3 Event Notification), phân tích kết quả rà quét mà không cần duy trì máy chủ chạy thường trực. |
| **Amazon CloudWatch** | **Observability & Budget Control:** Thu thập logs/metrics từ ECS Fargate Tasks thông qua Container Insights và phát thông báo cảnh báo chi phí (AWS Budget Alerts). |
| **AWS IAM** | **Security & Access Control:** Quản lý phân quyền hạt mịn theo nguyên tắc Least Privilege cho Jenkins Build Agent, ECS Task Execution Role và Lambda Execution Role. |

---

### 3. Bảo mật IAM & Khả năng mở rộng Vận hành

- **Phân quyền IAM Least Privilege:** Mỗi dịch vụ (ECS, Lambda, Jenkins) chỉ được cấp đúng các action tối thiểu cần thiết (`ecr:PutImage`, `s3:PutObject`, `ecs:UpdateService`).
- **Khả năng Auto Scaling & Event-Driven:** ECS Fargate tự động scale task theo lưu lượng traffic; luồng phân tích lỗ hổng hoàn toàn hướng sự kiện (Event-Driven) qua S3 -> Lambda.
- **Kiểm soát không Hardcode Credentials:** Sử dụng IAM Roles for Tasks và Systems Manager Parameter Store / Secrets Manager thay cho static keys.
