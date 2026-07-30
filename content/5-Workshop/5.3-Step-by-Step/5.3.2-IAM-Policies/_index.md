---
title : "Configure IAM Roles & Security Policies"
date : 2026-07-06 
weight : 2
chapter : false
pre : " <b> 5.3.2. </b> "
---

# 5.3.2 Configure IAM Roles & Security Policies for Pipeline

---

The project establishes 4 IAM principals adhering strictly to the principle of **Least Privilege**, ensuring each service component (Jenkins, ECS Task, Lambda) holds only the minimal necessary permissions required for operation without granting unnecessary administrative rights.

---

### 1. IAM Policy for Jenkins Build Agent (jenkins-ci)

Create the `devsecops-factory-jenkins-ci` IAM policy enforcing **Least Privilege** and attach it to the pipeline execution IAM User or Role:

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

- **ECS Task Execution Role (ecs_execution_role):** Attached with standard AWS managed policy `AmazonECSTaskExecutionRolePolicy` to pull images from ECR and write logs to CloudWatch Log Group (`/ecs/devsecops-factory`).

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

- **ECS Task Role (ecs_task_role):** Grants runtime permissions directly to running application containers. If the app does not interact with other AWS services, this role maintains a minimal Trust Policy allowing `ecs-tasks.amazonaws.com`.

---

### 3. IAM Role for Lambda Security Hub Importer

The `devsecops-factory-securityhub-importer` role grants Lambda permission to read ASFF JSON files from S3, emit CloudWatch logs, and invoke `securityhub:BatchImportFindings`:

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

### 4. Summary of 4 Project IAM Principals

| IAM Principal | Applied To | Core Granted Permissions |
|---|---|---|
| **ECS Execution Role** | All ECS Tasks | ECR image pull + CloudWatch logs ingestion |
| **ECS Task Role** | All ECS Tasks | Trust ECS tasks (no managed policy — minimal runtime privilege) |
| **Lambda Importer Role** | Lambda function | `s3:GetObject` + `securityhub:BatchImportFindings` + CloudWatch logs |
| **Jenkins CI IAM Policy** | IAM User `jenkins-ci` | `ecr:Push` + `ecs:Deploy` + `s3:PutObject` + `iam:PassRole` |

> **Note:** All IAM Roles and Policies are automatically managed via Terraform (`infrastructure/terraform/main.tf`). When applying Terraform with `create_local_jenkins_user = true`, Terraform provisions the IAM User for Jenkins and attaches corresponding policies.

---

### Actual Screenshot: AWS IAM Roles

![AWS IAM Roles](/images/5-Workshop/5.3-Step-by-Step/aws_iam_roles.png)
*Figure 5.3.2: AWS IAM Console displaying project IAM Roles (`devsecops-factory-ecs-execution-role`, `devsecops-factory-ecs-task-role`, `devsecops-securityhub-importer-role`).*
