---
title : "Hướng dẫn Thực hành Step-by-Step"
date : 2026-07-06 
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

# 5.3 Hướng dẫn Thực hành Step-by-Step (Hands-on Lab Guide)

---

## Tổng quan Hạ tầng AWS (Terraform Overview)

Toàn bộ hạ tầng AWS của dự án **DevSecOps Factory** được định nghĩa bằng hạ tầng dưới dạng mã (Infrastructure as Code - IaC) trong thư mục infrastructure/terraform/main.tf với 883 dòng mã HCL. Hạ tầng bao gồm 6 nhóm tài nguyên chính được tổ chức tự động hóa:

1. **AWS-01: Budget & IAM:** Quản lý AWS Budget Alert (cảnh báo 50%/80%/100%), ECS Execution Role, ECS Task Role, và Jenkins CI IAM Policy theo nguyên tắc phân quyền tối thiểu (Least Privilege).
2. **AWS-02: Amazon ECR:** Repository devsecops/tetris lưu trữ Docker container images với mã hóa SSE-AES256 và tự động quét lỗ hổng scan_on_push = true.
3. **AWS-03: Amazon ECS Fargate:** ECS Cluster devsecops-factory-cluster kích hoạt CloudWatch Container Insights, chạy 2 services: 	etris-staging (FARGATE_SPOT giảm 70% chi phí) và 	etris-production (FARGATE độ tin cậy cao).
4. **AWS-04: Amazon S3:** Bucket tập trung devsecops-reports-* lưu trữ báo cáo từ 6 Security Gates với mã hóa SSE-S3, Versioning, Public Access Block và Lifecycle 30 ngày.
5. **AWS-05: AWS Lambda:** Function devsecops-factory-securityhub-importer (Python 3.12) tự động đọc tệp ASFF JSON từ S3 và đẩy trực tiếp vào **AWS Security Hub**.
6. **Networking (VPC & ALB):** Cấu hình VPC (10.0.0.0/16), 2 Public Subnets (10.0.1.0/24 & 10.0.2.0/24), cùng 2 Application Load Balancers (ALB Staging & ALB Production).

---

## Danh mục các bước thực hành

1. [5.3.1 Khởi tạo Amazon ECR Repository & S3 Report Bucket](5.3.1-ecr-s3/)
2. [5.3.2 Khai báo IAM Roles & Security Policies](5.3.2-iam-policies/)
3. [5.3.3 Cấu hình 6 Security Gates trong Jenkinsfile](5.3.3-security-gates/)
4. [5.3.4 Khởi tạo AWS Lambda Security Report Aggregator](5.3.4-lambda-aggregator/)
5. [5.3.5 Triển khai Ứng dụng lên Amazon ECS Fargate](5.3.5-ecs-fargate/)
6. [5.3.6 Xác minh sau Triển khai (Post-Deployment Verification)](5.3.6-verification/)