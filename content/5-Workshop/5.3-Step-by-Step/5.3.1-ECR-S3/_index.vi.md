---
title : "Khởi tạo ECR & S3 Bucket"
date : 2026-07-06 
weight : 1
chapter : false
pre : " <b> 5.3.1. </b> "
---

# 5.3.1 Khởi tạo Amazon ECR Repository & S3 Report Bucket

---

### 1. Khởi tạo Amazon ECR Repository

Chạy lệnh AWS CLI để tạo ECR Repository chứa Docker image của ứng dụng với tính năng tự động quét khi push:

```bash
# Đặt tên region mặc định
export AWS_REGION="ap-southeast-1"

aws ecr create-repository \
    --repository-name devsecops/tetris \
    --image-scanning-configuration scanOnPush=true \
    --region ${AWS_REGION}
```

---

![Amazon ECR Repository](/images/5-Workshop/5.3-Step-by-Step/aws_ecr_repository.png)
*Hình 5.3.1a: Giao diện Amazon ECR Private Repository `devsecops/tetris` với tính năng Mutable Tag và mã hóa AES-256.*

---

### 2. Khởi tạo Amazon S3 Bucket lưu trữ báo cáo

Tạo S3 Bucket với tên duy nhất theo AWS Account ID và kích hoạt mã hóa mặc định Server-Side Encryption (SSE-S3):

```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
BUCKET_NAME="devsecops-reports-${ACCOUNT_ID}"

aws s3api create-bucket \
    --bucket ${BUCKET_NAME} \
    --region ${AWS_REGION} \
    --create-bucket-configuration LocationConstraint=${AWS_REGION}

# Bật mã hóa SSE-S3
aws s3api put-bucket-encryption \
    --bucket ${BUCKET_NAME} \
    --server-side-encryption-configuration '{"Rules": [{"ApplyServerSideEncryptionByDefault": {"SSEAlgorithm": "AES256"}}]}'
```

---

![Amazon S3 Bucket](/images/5-Workshop/5.3-Step-by-Step/aws_s3_bucket.png)
*Hình 5.3.1b: Giao diện danh sách Amazon S3 Buckets lưu trữ các báo cáo kiểm thử an ninh tập trung (`devsecops-reports-*`).*
