---
title : "Các bước chuẩn bị"
date : 2026-07-06 
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

#### 5.2 Các bước chuẩn bị

#### 0. Tải về Mã nguồn Dự án (Clone Repository)

Tải mã nguồn dự án mẫu **CICD-DevSecOps-using-AWS-services** từ kho lưu trữ GitHub chính thức của nhóm:

```bash
git clone https://github.com/loi-bui0703/CICD-DevSecOps-using-AWS-services.git
cd CICD-DevSecOps-using-AWS-services
```

---

#### 1. Yêu cầu công cụ và môi trường

Trước khi bắt đầu thực hành workshop, hãy đảm bảo máy tính cá nhân hoặc máy chủ build (Jenkins Agent) đã cài đặt đầy đủ các công cụ sau:

- **AWS CLI v2:** Đã cấu hình IAM user có quyền khởi tạo tài nguyên (`aws configure`).
- **Docker Engine & Docker Compose:** Dùng để build image và chạy các công cụ quét container.
- **Node.js (v16+) & npm:** Dùng để kiểm thử ứng dụng Web React local.
- **Git & GitHub Account:** Để lưu trữ mã nguồn và cấu hình webhook cho Jenkins.
- **Jenkins CI Server:** Cài đặt local hoặc trên máy chủ EC2.

---

#### 2. Cấu hình IAM Policy cho Pipeline

Gắn IAM Policy sau vào IAM Role/User thực thi Pipeline (`DevSecOpsPipelinePolicy`) để đảm bảo nguyên tắc phân quyền tối thiểu (Least Privilege):

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ECRManagement",
            "Effect": "Allow",
            "Action": [
                "ecr:GetAuthorizationToken",
                "ecr:BatchCheckLayerAvailability",
                "ecr:GetDownloadUrlForLayer",
                "ecr:PutImage",
                "ecr:InitiateLayerUpload",
                "ecr:UploadLayerPart",
                "ecr:CompleteLayerUpload"
            ],
            "Resource": "*"
        },
        {
            "Sid": "S3ReportStorage",
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObject",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::devsecops-reports-bucket-*",
                "arn:aws:s3:::devsecops-reports-bucket-*/*"
            ]
        },
        {
            "Sid": "ECSFargateDeployment",
            "Effect": "Allow",
            "Action": [
                "ecs:UpdateService",
                "ecs:DescribeServices",
                "ecs:RegisterTaskDefinition",
                "ecs:RunTask"
            ],
            "Resource": "*"
        },
        {
            "Sid": "CloudWatchLogs",
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "*"
        }
    ]
}
```

---

#### 3. Khởi tạo Amazon ECR Repository

Chạy lệnh AWS CLI sau để tạo ECR Repository với tính năng mã hóa và quét tự động khi push image:

```bash
aws ecr create-repository \
    --repository-name devsecops-react-app \
    --image-scanning-configuration scanOnPush=true \
    --region ap-southeast-1
```

---

#### 4. Khởi tạo Amazon S3 Bucket cho Báo cáo Bảo mật

Tạo S3 bucket tập trung để lưu trữ kết quả rà quét từ 6 Security Gates:

```bash
# Đặt tên bucket duy nhất theo Account ID của bạn
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
BUCKET_NAME="devsecops-reports-bucket-${ACCOUNT_ID}"

aws s3api create-bucket \
    --bucket ${BUCKET_NAME} \
    --region ap-southeast-1 \
    --create-bucket-configuration LocationConstraint=ap-southeast-1

# Bật mã hóa mặc định Server-Side Encryption (SSE-S3)
aws s3api put-bucket-encryption \
    --bucket ${BUCKET_NAME} \
    --server-side-encryption-configuration '{"Rules": [{"ApplyServerSideEncryptionByDefault": {"SSEAlgorithm": "AES256"}}]}'
```

---

#### 5. Cài đặt các Security Tools trên Jenkins Agent

Thực hiện cài đặt các công cụ quét bảo mật tự động trên Jenkins agent container:

```bash
# 1. Gitleaks (Secrets Scan)
wget https://github.com/gitleaks/gitleaks/releases/download/v8.18.1/gitleaks_8.18.1_linux_x64.tar.gz
tar -zxvf gitleaks_8.18.1_linux_x64.tar.gz
sudo mv gitleaks /usr/local/bin/

# 2. Trivy (SCA & Container Scan)
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update && sudo apt-get install trivy

# 3. Checkov (IaC Scan)
pip3 install checkov
```