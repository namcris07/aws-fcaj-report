---
title : "Step 1 – Amazon ECR & S3 Setup"
date : 2026-07-06 
weight : 1
chapter : false
pre : " <b> 5.3.1. </b> "
---

# 5.3.1 Provision Amazon ECR Repository & S3 Report Bucket

---

The entire AWS infrastructure for the project is defined in `infrastructure/terraform/main.tf` and managed via Terraform. This step provisions the Amazon ECR Repository (storing container images) and the Amazon S3 Bucket (storing centralized security reports).

---

### 1. Provision Amazon ECR Repository

The Amazon ECR Repository is declared with native security features:

**Option 1: Using Terraform (Recommended — managed alongside full cloud infrastructure)**

```hcl
# infrastructure/terraform/main.tf
resource "aws_ecr_repository" "tetris" {
  name                 = "devsecops/tetris"
  image_tag_mutability = "MUTABLE"
  force_delete         = true

  # Enable automatic vulnerability scanning on image push
  image_scanning_configuration {
    scan_on_push = true
  }

  # Server-side encryption (AES256)
  encryption_configuration {
    encryption_type = "AES256"
  }

  tags = {
    Name        = "devsecops-factory-ecr"
    Environment = "shared"
  }
}
```

**Option 2: Using AWS CLI (Quick standalone setup)**

```bash
# Set default AWS region
export AWS_REGION="ap-southeast-1"

aws ecr create-repository \
    --repository-name devsecops/tetris \
    --image-scanning-configuration scanOnPush=true \
    --encryption-configuration encryptionType=AES256 \
    --region ${AWS_REGION}
```

**Configuration Rationale:**
- **`scan_on_push = true`**: Every pushed container image is automatically scanned for CVEs via Amazon ECR Enhanced Scanning, ensuring vulnerability detection even outside CI/CD runs.
- **`image_tag_mutability = "MUTABLE"`**: Permits updating the `:latest` tag to streamline local testing. Enterprise production environments should enforce `IMMUTABLE`.
- **`encryption_type = "AES256"`**: Protects data at rest using AWS KMS-managed encryption keys.

**Jenkins Image Push Workflow to ECR (Stages 10–11):**

```bash
# Stage 10: ECR Authentication Login
aws sts get-caller-identity --output json > scan-reports/aws-caller-identity.json
aws ecr get-login-password --region ap-southeast-1 \
  | docker login --username AWS --password-stdin \
    "<ACCOUNT_ID>.dkr.ecr.ap-southeast-1.amazonaws.com"

# Stage 11: Push Image with Immutable 12-char SHA Tag
docker push "${IMAGE_URI}"    # tag: abc123def456
docker push "${LATEST_IMAGE_URI}"  # tag: latest
```

The image tag utilizes the **first 12 characters of the Git commit SHA**, ensuring image immutability and accurate commit traceability.

---

### 2. Provision Amazon S3 Bucket for Centralized Security Reports

The S3 Bucket is configured with **4 defense-in-depth layers** and automated lifecycle management:

**Option 1: Using Terraform (Recommended)**

```hcl
# Create S3 bucket with unique random suffix
resource "aws_s3_bucket" "security_reports" {
  bucket        = "devsecops-reports-${random_string.suffix.result}"
  force_destroy = true
}

# Enable Versioning
resource "aws_s3_bucket_versioning" "reports_versioning" {
  bucket = aws_s3_bucket.security_reports.id
  versioning_configuration { status = "Enabled" }
}

# Server-Side Encryption (SSE-S3 = AES256)
resource "aws_s3_bucket_server_side_encryption_configuration" "reports_encryption" {
  bucket = aws_s3_bucket.security_reports.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

# Block all Public Access
resource "aws_s3_bucket_public_access_block" "reports_public_block" {
  bucket                  = aws_s3_bucket.security_reports.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

# S3 Lifecycle Rule: Automatically delete artifacts after 30 days
resource "aws_s3_bucket_lifecycle_configuration" "reports_lifecycle" {
  bucket = aws_s3_bucket.security_reports.id
  rule {
    id     = "delete-after-30-days"
    status = "Enabled"
    filter {}
    expiration { days = 30 }
    noncurrent_version_expiration { noncurrent_days = 30 }
  }
}
```

**Option 2: Using AWS CLI (Quick creation)**

```bash
# Create bucket with unique name containing AWS Account ID
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
BUCKET_NAME="devsecops-reports-${ACCOUNT_ID}"

aws s3api create-bucket \
    --bucket ${BUCKET_NAME} \
    --region ap-southeast-1 \
    --create-bucket-configuration LocationConstraint=ap-southeast-1

# Enable SSE-S3 Encryption
aws s3api put-bucket-encryption \
    --bucket ${BUCKET_NAME} \
    --server-side-encryption-configuration \
    '{"Rules": [{"ApplyServerSideEncryptionByDefault": {"SSEAlgorithm": "AES256"}}]}'

# Block Public Access
aws s3api put-public-access-block \
    --bucket ${BUCKET_NAME} \
    --public-access-block-configuration \
    "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"
```

---

### Actual Screenshots

![50 Resources Added by Terraform](/images/5-Workshop/5.3-Step-by-Step/50-resources-added.png)
*Figure 5.3.1a: `terraform apply` output showing successful creation of 50 AWS infrastructure resources (ECS Cluster, ALBs, ECR, S3, Lambda, IAM, VPC).*

![Amazon ECR Repository](/images/5-Workshop/5.3-Step-by-Step/aws_ecr_repository.png)
*Figure 5.3.1b: Amazon ECR Private Repository UI for `devsecops/tetris` holding container images tagged with 12-character Git commit SHAs.*

![Amazon S3 Bucket Reports](/images/5-Workshop/5.3-Step-by-Step/aws_s3_bucket.png)
*Figure 5.3.1c: Amazon S3 Console displaying centralized security report bucket (`devsecops-reports-*`).*

![ECR Images with SHA tag](/images/5-Workshop/5.3-Step-by-Step/ecr_images.png)
*Figure 5.3.1d: Amazon ECR Private Repository showing detailed image list.*

![S3 All Reports](/images/5-Workshop/5.3-Step-by-Step/s3_all_reports.png)
*Figure 5.3.1e: Centralized S3 bucket displaying security scan reports from 6 security gates organized by prefix (`secrets/`, `sca/`, `sast/`, `container/`, `dast/`, `iac/`, `asff/`, `normalized/`).*