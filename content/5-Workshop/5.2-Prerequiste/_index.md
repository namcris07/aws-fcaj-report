---
title : "Prerequisites"
date : 2026-07-06 
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

#### 5.2 Prerequisites

#### 0. Clone Project Source Code

Clone the official team repository `CICD-DevSecOps-using-AWS-services`:

```bash
git clone https://github.com/loi-bui0703/CICD-DevSecOps-using-AWS-services.git
cd CICD-DevSecOps-using-AWS-services
```

---

#### 1. Tool & Environment Requirements

Ensure the following tools are installed on your build machine / Jenkins Agent:

- **AWS CLI v2:** Configured with IAM credentials (`aws configure`).
- **Docker Engine & Docker Compose:** Used to build images and execute security scanners.
- **Node.js (v16+) & npm:** Used for local React Web testing.
- **Git & GitHub Account:** Source code repository and Jenkins Webhook integration.
- **Jenkins CI Server:** Local or EC2 server.

---

#### 2. Configure IAM Policy for Pipeline

Attach `DevSecOpsPipelinePolicy` to your execution IAM Role/User:

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

#### 3. Create Amazon ECR Repository

```bash
aws ecr create-repository \
    --repository-name devsecops-react-app \
    --image-scanning-configuration scanOnPush=true \
    --region ap-southeast-1
```

---

#### 4. Provision S3 Report Storage Bucket

```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
BUCKET_NAME="devsecops-reports-bucket-${ACCOUNT_ID}"

aws s3api create-bucket \
    --bucket ${BUCKET_NAME} \
    --region ap-southeast-1 \
    --create-bucket-configuration LocationConstraint=ap-southeast-1

aws s3api put-bucket-encryption \
    --bucket ${BUCKET_NAME} \
    --server-side-encryption-configuration '{"Rules": [{"ApplyServerSideEncryptionByDefault": {"SSEAlgorithm": "AES256"}}]}'
```

---

#### 5. Install Security Scanning Tools on Jenkins Agent

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