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
    --repository-name devsecops-react-app \
    --image-scanning-configuration scanOnPush=true \
    --region ${AWS_REGION}
```

---

### 2. Khởi tạo Amazon S3 Bucket lưu trữ báo cáo
Tạo S3 Bucket với tên duy nhất theo AWS Account ID và kích hoạt mã hóa mặc định Server-Side Encryption (SSE-S3):

```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
BUCKET_NAME="devsecops-reports-bucket-${ACCOUNT_ID}"

aws s3api create-bucket \
    --bucket ${BUCKET_NAME} \
    --region ${AWS_REGION} \
    --create-bucket-configuration LocationConstraint=${AWS_REGION}

# Bật mã hóa SSE-S3
aws s3api put-bucket-encryption \
    --bucket ${BUCKET_NAME} \
    --server-side-encryption-configuration '{"Rules": [{"ApplyServerSideEncryptionByDefault": {"SSEAlgorithm": "AES256"}}]}'
```
