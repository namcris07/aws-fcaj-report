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
make setup-env
# Edit .env file and change default secret values
```

---

#### 1. Tool & Environment Requirements

Ensure the following tools are installed on your build machine / Jenkins Agent:

- **AWS CLI v2:** Configured with IAM credentials (`aws configure`).
- **Terraform (v1.0+):** Used for automated AWS infrastructure provisioning.
- **Docker Engine & Docker Compose:** Used to build images and execute security scanners.
- **Node.js (v16+) & npm:** Used for local React Web testing.
- **Git & GitHub Account:** Source code repository and Jenkins Webhook integration.
- **Jenkins CI Server:** Local or EC2 server.

---

#### 2. Configure IAM Policy for Pipeline

Attach `devsecops-factory-jenkins-ci` policy to your execution IAM Role/User:

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

### Genuine Screenshot: AWS Budgets Overview

![AWS Budgets](/images/5-Workshop/5.2-Prerequisite/aws_budgets.png)
*Figure 5.2: AWS Budgets management console (`devsecops-factory-monthly-budget`) controlling monthly project spending.*

---

#### 3. Create Amazon ECR Repository

```bash
aws ecr create-repository \
    --repository-name devsecops/tetris \
    --image-scanning-configuration scanOnPush=true \
    --region ap-southeast-1
```

---

#### 4. Provision S3 Report Storage Bucket

```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
BUCKET_NAME="devsecops-reports-${ACCOUNT_ID}"

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