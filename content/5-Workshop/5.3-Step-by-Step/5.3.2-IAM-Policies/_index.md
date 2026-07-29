---
title : "Configure IAM Roles & Policies"
date : 2026-07-06 
weight : 2
chapter : false
pre : " <b> 5.3.2. </b> "
---

# 5.3.2 Configure IAM Roles & Security Policies

---

### 1. IAM Policy for Jenkins Build Agent (`jenkins_ci`)

Create `devsecops-factory-jenkins-ci` IAM policy adhering to the **Least Privilege** principle:

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

### 2. IAM Roles for Amazon ECS Fargate Tasks

- **ECS Task Execution Role (`ecs_execution_role`):** Granted standard `AmazonECSTaskExecutionRolePolicy` to pull images from ECR and send logs to CloudWatch (`/ecs/devsecops-factory`).
- **ECS Task Role (`ecs_task_role`):** Provides runtime permissions for active containers.

---

### 3. IAM Role for Lambda Security Hub Importer

Role `devsecops-factory-securityhub-importer` enables Lambda to read ASFF JSON reports from S3, write logs, and invoke `securityhub:BatchImportFindings`:

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

### Genuine Screenshot: AWS IAM Roles Overview

![AWS IAM Roles](/images/5-Workshop/5.3-Step-by-Step/aws_iam_roles.png)
*Figure 5.3.2: AWS IAM Roles management console (`devsecops-factory-ecs-execution-role`, `devsecops-factory-ecs-task-role`, `devsecops-securityhub-importer-role`).*
