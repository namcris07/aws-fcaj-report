---
title : "Khởi tạo ECR & S3 Bucket"
date : 2026-07-06 
weight : 1
chapter : false
pre : " <b> 5.3.1. </b> "
---

# 5.3.1 Khởi tạo Amazon ECR Repository & S3 Report Bucket

---

Toàn bộ hạ tầng AWS của dự án được định nghĩa trong `infrastructure/terraform/main.tf` và được quản lý bằng Terraform. Bước này bao gồm tạo ECR Repository (lưu Docker images) và S3 Bucket (lưu security reports tập trung).

---

### 1. Khởi tạo Amazon ECR Repository

Amazon ECR Repository được khai báo với các tính năng bảo mật tích hợp sẵn:

**Cách 1: Dùng Terraform (khuyến nghị — được quản lý cùng toàn bộ hạ tầng)**

```hcl
# infrastructure/terraform/main.tf
resource "aws_ecr_repository" "tetris" {
  name                 = "devsecops/tetris"
  image_tag_mutability = "MUTABLE"
  force_delete         = true

  # Kích hoạt tự động quét image khi push
  image_scanning_configuration {
    scan_on_push = true
  }

  # Mã hóa phía server (AES256)
  encryption_configuration {
    encryption_type = "AES256"
  }

  tags = {
    Name        = "devsecops-factory-ecr"
    Environment = "shared"
  }
}
```

**Cách 2: Dùng AWS CLI (tạo nhanh độc lập)**

```bash
# Đặt region mặc định
export AWS_REGION="ap-southeast-1"

aws ecr create-repository \
    --repository-name devsecops/tetris \
    --image-scanning-configuration scanOnPush=true \
    --encryption-configuration encryptionType=AES256 \
    --region ${AWS_REGION}
```

**Lý do chọn cấu hình này:**
- **`scan_on_push = true`**: Mọi image được đẩy lên sẽ tự động được quét CVE bởi Amazon ECR Enhanced Scanning, giúp phát hiện lỗ hổng ngay cả khi không chạy pipeline đầy đủ.
- **`image_tag_mutability = "MUTABLE"`**: Cho phép cập nhật tag `:latest` để đơn giản hóa local development. Trong môi trường production enterprise nên dùng `IMMUTABLE`.
- **`encryption_type = "AES256"`**: Bảo vệ dữ liệu at-rest theo tiêu chuẩn AWS KMS-managed key.

**Quy trình Jenkins push image lên ECR (Stage 10–11):**

```bash
# Stage 10: ECR Login
aws sts get-caller-identity --output json > scan-reports/aws-caller-identity.json
aws ecr get-login-password --region ap-southeast-1 \
  | docker login --username AWS --password-stdin \
    "<ACCOUNT_ID>.dkr.ecr.ap-southeast-1.amazonaws.com"

# Stage 11: Push Image với immutable SHA tag (12 ký tự)
docker push "${IMAGE_URI}"    # tag: abc123def456
docker push "${LATEST_IMAGE_URI}"  # tag: latest
```

Image tag sử dụng **12 ký tự đầu của Git commit SHA**, đảm bảo mỗi image là immutable và có thể truy vết nguồn gốc chính xác.

---

### 2. Khởi tạo Amazon S3 Bucket lưu trữ báo cáo bảo mật

S3 Bucket được cấu hình với **4 lớp bảo mật** và quản lý vòng đời dữ liệu tự động:

**Cách 1: Dùng Terraform (khuyến nghị)**

```hcl
# Tạo S3 bucket với tên unique (random suffix)
resource "aws_s3_bucket" "security_reports" {
  bucket        = "devsecops-reports-${random_string.suffix.result}"
  force_destroy = true
}

# Bật Versioning
resource "aws_s3_bucket_versioning" "reports_versioning" {
  bucket = aws_s3_bucket.security_reports.id
  versioning_configuration { status = "Enabled" }
}

# Mã hóa Server-Side (SSE-S3 = AES256)
resource "aws_s3_bucket_server_side_encryption_configuration" "reports_encryption" {
  bucket = aws_s3_bucket.security_reports.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

# Chặn tất cả public access
resource "aws_s3_bucket_public_access_block" "reports_public_block" {
  bucket                  = aws_s3_bucket.security_reports.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

# Lifecycle: Tự động xóa sau 30 ngày
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

**Cách 2: Dùng AWS CLI (tạo nhanh)**

```bash
# Tạo bucket với tên duy nhất theo Account ID
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
BUCKET_NAME="devsecops-reports-${ACCOUNT_ID}"

aws s3api create-bucket \
    --bucket ${BUCKET_NAME} \
    --region ap-southeast-1 \
    --create-bucket-configuration LocationConstraint=ap-southeast-1

# Bật mã hóa SSE-S3
aws s3api put-bucket-encryption \
    --bucket ${BUCKET_NAME} \
    --server-side-encryption-configuration \
    '{"Rules": [{"ApplyServerSideEncryptionByDefault": {"SSEAlgorithm": "AES256"}}]}'

# Chặn public access
aws s3api put-public-access-block \
    --bucket ${BUCKET_NAME} \
    --public-access-block-configuration \
    "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"
```

---

### Ảnh chụp màn hình thực tế

![50 Resources Added by Terraform](/images/5-Workshop/5.3-Step-by-Step/50-resources-added.png)
*Hình 5.3.1a: Kết quả `terraform apply` tạo thành công toàn bộ 50 tài nguyên hạ tầng AWS (ECS Cluster, ALBs, ECR, S3, Lambda, IAM, VPC).*

![Amazon ECR Repository](/images/5-Workshop/5.3-Step-by-Step/aws_ecr_repository.png)
*Hình 5.3.1b: Giao diện Amazon ECR Private Repository `devsecops/tetris` với Docker images có 12-char SHA tag.*

![Amazon S3 Bucket Reports](/images/5-Workshop/5.3-Step-by-Step/aws_s3_bucket.png)
*Hình 5.3.1c: Giao diện Amazon S3 Console lưu trữ các báo cáo kiểm thử an ninh tập trung (`devsecops-reports-*`).*

![ECR Images với SHA tag](/images/5-Workshop/5.3-Step-by-Step/ecr_images.png)
*Hình 5.3.1d: Amazon ECR Private Repository `devsecops/tetris` hiển thị danh sách images chi tiết.*

![S3 All Reports](/images/5-Workshop/5.3-Step-by-Step/s3_all_reports.png)
*Hình 5.3.1e: Toàn bộ báo cáo bảo mật từ 6 Security Gates được tổ chức theo prefix trong S3 (secrets/, sca/, sast/, container/, dast/, iac/, asff/, normalized/).*