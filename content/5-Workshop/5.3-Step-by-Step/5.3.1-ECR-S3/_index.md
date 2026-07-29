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
    --repository-name devsecops/tetris \
    --image-scanning-configuration scanOnPush=true \
    --region ${AWS_REGION}
```

---

![Amazon ECR Repository](/images/5-Workshop/5.3-Step-by-Step/aws_ecr_repository.png)
*Figure 5.3.1a: Amazon ECR Private Repository `devsecops/tetris` with Mutable tags and AES-256 encryption.*

---

### 2. Create Amazon S3 Report Bucket
Provision an S3 Bucket with default Server-Side Encryption (SSE-S3):

```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
BUCKET_NAME="devsecops-reports-${ACCOUNT_ID}"

aws s3api create-bucket \
    --bucket ${BUCKET_NAME} \
    --region ${AWS_REGION} \
    --create-bucket-configuration LocationConstraint=${AWS_REGION}

aws s3api put-bucket-encryption \
    --bucket ${BUCKET_NAME} \
    --server-side-encryption-configuration '{"Rules": [{"ApplyServerSideEncryptionByDefault": {"SSEAlgorithm": "AES256"}}]}'
```

---

![Amazon S3 Bucket](/images/5-Workshop/5.3-Step-by-Step/aws_s3_bucket.png)
*Figure 5.3.1b: Amazon S3 Buckets console listing report storage buckets (`devsecops-reports-*`).*
