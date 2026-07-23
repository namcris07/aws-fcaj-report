---
title : "Khai báo IAM Roles & Policies"
date : 2026-07-06 
weight : 2
chapter : false
pre : " <b> 5.3.2. </b> "
---

# 5.3.2 Khai báo IAM Roles & Security Policies cho Pipeline

---

Tạo IAM Policy `DevSecOpsPipelinePolicy` tuân thủ nguyên tắc **Least Privilege** và gán cho Role thực thi Pipeline:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ECRAuthAndPush",
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
            "Sid": "S3ReportUpload",
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
            "Sid": "ECSFargateManagement",
            "Effect": "Allow",
            "Action": [
                "ecs:UpdateService",
                "ecs:DescribeServices",
                "ecs:RegisterTaskDefinition",
                "ecs:RunTask"
            ],
            "Resource": "*"
        }
    ]
}
```
