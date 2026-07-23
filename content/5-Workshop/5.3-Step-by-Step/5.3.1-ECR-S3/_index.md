---
title : "Initialize ECR & S3 Bucket"
date : 2026-07-06 
weight : 1
chapter : false
pre : " <b> 5.3.1. </b> "
---

# 5.3.1 Initialize Amazon ECR Repository & S3 Report Bucket

---

### 1. Create Amazon ECR Repository
Run AWS CLI commands to provision a private container repository with scan-on-push enabled:

```bash
export AWS_REGION="ap-southeast-1"

aws ecr create-repository \
    --repository-name devsecops-react-app \
    --image-scanning-configuration scanOnPush=true \
    --region ${AWS_REGION}
```

---

### 2. Create Amazon S3 Report Bucket
Provision an S3 Bucket with default Server-Side Encryption (SSE-S3):

```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
BUCKET_NAME="devsecops-reports-bucket-${ACCOUNT_ID}"

aws s3api create-bucket \
    --bucket ${BUCKET_NAME} \
    --region ${AWS_REGION} \
    --create-bucket-configuration LocationConstraint=${AWS_REGION}

aws s3api put-bucket-encryption \
    --bucket ${BUCKET_NAME} \
    --server-side-encryption-configuration '{"Rules": [{"ApplyServerSideEncryptionByDefault": {"SSEAlgorithm": "AES256"}}]}'
```
