---
title : "Khai báo IAM Roles & Policies"
date : 2026-07-06 
weight : 2
chapter : false
pre : " <b> 5.3.2. </b> "
---

# 5.3.2 Khai báo IAM Roles & Security Policies cho Pipeline

---

Dự án sử dụng 4 IAM principal với quyền tối thiểu (Least Privilege), đảm bảo từng thành phần (Jenkins, ECS Task, Lambda) chỉ được cấp đúng các quyền cần thiết để hoạt động, không cấp thừa quyền administrator.

---

### 1. IAM Policy cho Jenkins Build Agent (jenkins-ci)

Tạo IAM Policy devsecops-factory-jenkins-ci tuân thủ nguyên tắc **Least Privilege** và gán cho IAM User/Role thực thi Pipeline:

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

- **ECS Task Execution Role (ecs_execution_role):** Được cấp policy chuẩn AmazonECSTaskExecutionRolePolicy để kéo image từ ECR và đẩy logs lên CloudWatch Log Group (/ecs/devsecops-factory).

```hcl
resource "aws_iam_role" "ecs_execution_role" {
  name = "devsecops-factory-ecs-execution-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action    = "sts:AssumeRole"
      Effect    = "Allow"
      Principal = { Service = "ecs-tasks.amazonaws.com" }
    }]
  })
}

resource "aws_iam_role_policy_attachment" "ecs_execution_policy" {
  role       = aws_iam_role.ecs_execution_role.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy"
}
```

- **ECS Task Role (ecs_task_role):** Cấp quyền tối thiểu tại runtime cho container khi đang hoạt động. Nếu ứng dụng không cần tương tác dịch vụ AWS khác, role này chỉ chứa Trust Policy chấp nhận ecs-tasks.amazonaws.com.

---

### 3. IAM Role cho Lambda Security Hub Importer

Role devsecops-factory-securityhub-importer cho phép Lambda đọc tệp ASFF JSON từ S3, ghi log CloudWatch và gọi securityhub:BatchImportFindings:

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

### 4. Tóm tắt 4 IAM Principals trong dự án

| IAM Principal | Được dùng bởi | Quyền chính |
|---|---|---|
| **ECS Execution Role** | ECS Tasks (tất cả) | Kéo images từ ECR + ghi CloudWatch Logs |
| **ECS Task Role** | ECS Tasks (tất cả) | Trust ECS tasks (không có policy — minimal runtime privilege) |
| **Lambda Importer Role** | Lambda function | s3:GetObject + securityhub:BatchImportFindings + CloudWatch Logs |
| **Jenkins CI IAM Policy** | IAM User jenkins-ci | ecr:Push + ecs:Deploy + s3:PutObject + iam:PassRole |

> **Ghi chú:** Trong dự án này, toàn bộ IAM Roles và Policy được quản lý tự động bởi Terraform (infrastructure/terraform/main.tf). Khi triển khai 	erraform apply với create_local_jenkins_user = true, Terraform sẽ khởi tạo IAM User cho Jenkins và gán policy tương ứng.

---

### Ảnh chụp màn hình thực tế: AWS IAM Roles

![AWS IAM Roles](/images/5-Workshop/5.3-Step-by-Step/aws_iam_roles.png)
*Hình 5.3.2: Màn hình quản lý IAM Roles (devsecops-factory-ecs-execution-role, devsecops-factory-ecs-task-role, devsecops-securityhub-importer-role).*