---
title : "Khai báo IAM Roles & Policies"
date : 2026-07-06 
weight : 2
chapter : false
pre : " <b> 5.3.2. </b> "
---

# 5.3.2 Khai báo IAM Roles & Security Policies cho Pipeline

---

### 1. IAM Policy cho Jenkins Build Agent (`jenkins_ci`)

Tạo IAM Policy `devsecops-factory-jenkins-ci` tuân thủ nguyên tắc **Least Privilege** và gán cho IAM User/Role thực thi Pipeline:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "EcrLogin",
      "Effect": "Allow",
      "Action": ["ecr:GetAuthorizationToken"],
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
      "Action": ["iam:PassRole"],
      "Resource": [
        "arn:aws:iam::*:role/devsecops-factory-ecs-execution-role",
        "arn:aws:iam::*:role/devsecops-factory-ecs-task-role"
      ]
    },
    {
      "Sid": "UploadReports",
      "Effect": "Allow",
      "Action": ["s3:PutObject"],
      "Resource": "arn:aws:s3:::devsecops-reports-*/reports/*"
    }
  ]
}
```

---

### 2. IAM Roles cho Amazon ECS Fargate Tasks

- **ECS Task Execution Role (`ecs_execution_role`):** Được cấp policy chuẩn `AmazonECSTaskExecutionRolePolicy` để kéo image từ ECR và đẩy logs lên CloudWatch (`/ecs/devsecops-factory`).
- **ECS Task Role (`ecs_task_role`):** Cấp quyền tối thiểu tại runtime cho container khi đang hoạt động.

---

### 3. IAM Role cho Lambda Security Hub Importer

Role `devsecops-factory-securityhub-importer` cho phép Lambda đọc tệp ASFF JSON từ S3, ghi log CloudWatch và gọi `securityhub:BatchImportFindings`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadSecurityReports",
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": "arn:aws:s3:::devsecops-reports-*/*"
    },
    {
      "Sid": "ImportSecurityHubFindings",
      "Effect": "Allow",
      "Action": ["securityhub:BatchImportFindings"],
      "Resource": "*"
    },
    {
      "Sid": "WriteLambdaLogs",
      "Effect": "Allow",
      "Action": ["logs:CreateLogStream", "logs:PutLogEvents"],
      "Resource": "*"
    }
  ]
}
```

---

### Ảnh chụp màn hình thực tế: AWS IAM Roles

![AWS IAM Roles](/images/5-Workshop/5.3-Step-by-Step/aws_iam_roles.png)
*Hình 5.3.2: Màn hình quản lý IAM Roles (`devsecops-factory-ecs-execution-role`, `devsecops-factory-ecs-task-role`, `devsecops-securityhub-importer-role`).*
