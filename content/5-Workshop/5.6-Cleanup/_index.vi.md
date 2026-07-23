---
title : "Dọn dẹp tài nguyên"
date : 2026-07-06 
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

#### Dọn dẹp tài nguyên (Clean-up)

Sau khi hoàn tất việc kiểm thử và báo cáo workshop, bạn cần thực hiện dọn dẹp các tài nguyên đã tạo trên AWS nhằm tránh phát sinh chi phí ngoài ý muốn.

---

#### 1. Scale số lượng ECS Fargate Tasks về 0 (Tạm dừng container)

Để ngắt ngay lập tức chi phí tính toán Fargate mà không mất cấu hình dịch vụ, hãy giảm `desired-count` về 0 cho cả 2 môi trường Staging và Production:

```bash
# 1. Scale Staging Service về 0
aws ecs update-service \
    --cluster devsecops-cluster \
    --service devsecops-staging-service \
    --desired-count 0 \
    --region ap-southeast-1

# 2. Scale Production Service về 0
aws ecs update-service \
    --cluster devsecops-cluster \
    --service devsecops-prod-service \
    --desired-count 0 \
    --region ap-southeast-1
```

---

#### 2. Xóa ECS Services & Cluster (Nếu hủy hoàn toàn)

Nếu không còn nhu cầu sử dụng lại workshop:

```bash
# Xóa Services
aws ecs delete-service --cluster devsecops-cluster --service devsecops-staging-service --force
aws ecs delete-service --cluster devsecops-cluster --service devsecops-prod-service --force

# Xóa Cluster
aws ecs delete-cluster --cluster devsecops-cluster
```

---

#### 3. Dọn dẹp Amazon ECR Repository

Xóa toàn bộ Docker images và repository trên ECR:

```bash
aws ecr delete-repository \
    --repository-name devsecops-react-app \
    --force \
    --region ap-southeast-1
```

---

#### 4. Dọn dẹp Amazon S3 Report Bucket

Xóa toàn bộ các tệp báo cáo bảo mật và bucket S3:

```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
BUCKET_NAME="devsecops-reports-bucket-${ACCOUNT_ID}"

# Xóa toàn bộ objects trong bucket
aws s3 rm s3://${BUCKET_NAME} --recursive

# Xóa S3 Bucket
aws s3api delete-bucket --bucket ${BUCKET_NAME} --region ap-southeast-1
```

---

#### 5. Xóa AWS Lambda Function & CloudWatch Log Groups

Dọn dẹp Lambda function tổng hợp báo cáo và các nhóm log:

```bash
# Xóa Lambda Aggregator Function
aws lambda delete-function --function-name devsecops-report-aggregator

# Xóa CloudWatch Log Groups
aws logs delete-log-group --log-group-name /ecs/devsecops-react-app
aws logs delete-log-group --log-group-name /aws/lambda/devsecops-report-aggregator
```

> **Lưu ý:** Việc thực hiện đầy đủ các bước trên sẽ đảm bảo tài khoản AWS của bạn trở về trạng thái an toàn và chi phí phát sinh là **$0.00 USD**.