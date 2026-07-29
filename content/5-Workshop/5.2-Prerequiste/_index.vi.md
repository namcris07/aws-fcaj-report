---
title : "Các bước chuẩn bị"
date : 2026-07-06 
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

#### 5.2 Các bước chuẩn bị

#### 5.2 Các bước chuẩn bị

#### 0. Tải về Mã nguồn Dự án (Clone Repository)

Tải mã nguồn dự án mẫu **CICD-DevSecOps-using-AWS-services** từ kho lưu trữ GitHub chính thức của nhóm:

```bash
git clone https://github.com/loi-bui0703/CICD-DevSecOps-using-AWS-services.git
cd CICD-DevSecOps-using-AWS-services
make setup-env
# Chỉnh sửa file .env và thay đổi các giá trị bí mật mặc định
```

---

#### 1. Yêu cầu công cụ và môi trường

Trước khi bắt đầu thực hành workshop, hãy đảm bảo máy tính cá nhân hoặc máy chủ build (Jenkins Agent) đã cài đặt đầy đủ các công cụ sau:

- **AWS CLI v2:** Đã cấu hình IAM user có quyền khởi tạo tài nguyên (`aws configure`).
- **Terraform (v1.0+):** Dùng để tự động hóa khởi tạo toàn bộ hạ tầng AWS.
- **Docker Engine & Docker Compose:** Dùng để build image và chạy các công cụ quét container.
- **Node.js (v16+) & npm:** Dùng để kiểm thử ứng dụng Web React local.
- **Git & GitHub Account:** Để lưu trữ mã nguồn và cấu hình webhook cho Jenkins.
- **Jenkins CI Server:** Cài đặt local hoặc trên máy chủ EC2.

---

#### 2. Cấu hình IAM Policy cho Pipeline

Gắn IAM Policy sau (`devsecops-factory-jenkins-ci`) vào IAM User/Role thực thi Pipeline để đảm bảo nguyên tắc phân quyền tối thiểu (Least Privilege):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "EcrLogin",
      "Effect": "Allow",
      "Action": [
        "ecr:GetAuthorizationToken"
      ],
      "Resource": "*"
    },
    {
      "Sid": "EcrPush",
      "Effect": "Allow",
      "Action": [
        "ecr:BatchCheckLayerAvailability",
        "ecr:CompleteLayerUpload",
        "ecr:InitiateLayerUpload",
        "ecr:PutImage",
        "ecr:UploadLayerPart"
      ],
      "Resource": "arn:aws:ecr:ap-southeast-1:*:repository/devsecops/tetris"
    },
    {
      "Sid": "EcsDeploy",
      "Effect": "Allow",
      "Action": [
        "ecs:DescribeServices",
        "ecs:DescribeTaskDefinition",
        "ecs:RegisterTaskDefinition",
        "ecs:UpdateService"
      ],
      "Resource": "*"
    },
    {
      "Sid": "PassEcsRoles",
      "Effect": "Allow",
      "Action": [
        "iam:PassRole"
      ],
      "Resource": [
        "arn:aws:iam::*:role/devsecops-factory-ecs-execution-role",
        "arn:aws:iam::*:role/devsecops-factory-ecs-task-role"
      ]
    },
    {
      "Sid": "UploadReports",
      "Effect": "Allow",
      "Action": [
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::devsecops-reports-*/reports/*"
    }
  ]
}
```

---

### Ảnh chụp màn hình thực tế: AWS Budgets

![AWS Budgets](/images/5-Workshop/5.2-Prerequisite/aws_budgets.png)
*Hình 5.2: Giao diện quản lý ngân sách AWS Budgets (`devsecops-factory-monthly-budget`) kiểm soát chi phí hàng tháng.*

---

#### 3. Khởi tạo Amazon ECR Repository

Chạy lệnh AWS CLI sau (hoặc dùng Terraform `main.tf`) để tạo ECR Repository với tính năng mã hóa và quét tự động khi push image:

```bash
aws ecr create-repository \
    --repository-name devsecops/tetris \
    --image-scanning-configuration scanOnPush=true \
    --region ap-southeast-1
```

---

#### 4. Khởi tạo Amazon S3 Bucket cho Báo cáo Bảo mật

Tạo S3 bucket tập trung để lưu trữ kết quả rà quét từ 6 Security Gates:

```bash
# Đặt tên bucket duy nhất theo Account ID hoặc chuỗi ngẫu nhiên
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
BUCKET_NAME="devsecops-reports-${ACCOUNT_ID}"

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