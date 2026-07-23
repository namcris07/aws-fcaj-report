---
title : "Clean up"
date : 2026-07-06 
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

#### Resource Clean-up

After completing all workshop verification tasks and report documentation, perform the following steps to clean up all AWS resources and prevent unintended cloud charges.

---

#### 1. Scale ECS Fargate Tasks to 0 (Pause Running Containers)

To immediately stop Fargate compute charges while preserving service definitions, reduce the `desired-count` to 0 for both Staging and Production environments:

```bash
# 1. Scale Staging Service to 0
aws ecs update-service \
    --cluster devsecops-cluster \
    --service devsecops-staging-service \
    --desired-count 0 \
    --region ap-southeast-1

# 2. Scale Production Service to 0
aws ecs update-service \
    --cluster devsecops-cluster \
    --service devsecops-prod-service \
    --desired-count 0 \
    --region ap-southeast-1
```

---

#### 2. Delete ECS Services & Cluster

If you do not plan to reuse the workshop environment:

```bash
# Delete Services
aws ecs delete-service --cluster devsecops-cluster --service devsecops-staging-service --force
aws ecs delete-service --cluster devsecops-cluster --service devsecops-prod-service --force

# Delete Cluster
aws ecs delete-cluster --cluster devsecops-cluster
```

---

#### 3. Delete Amazon ECR Repository

Remove all Docker images and the ECR repository:

```bash
aws ecr delete-repository \
    --repository-name devsecops-react-app \
    --force \
    --region ap-southeast-1
```

---

#### 4. Clean up Amazon S3 Report Bucket

Purge all stored security reports and remove the S3 bucket:

```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
BUCKET_NAME="devsecops-reports-bucket-${ACCOUNT_ID}"

aws s3 rm s3://${BUCKET_NAME} --recursive
aws s3api delete-bucket --bucket ${BUCKET_NAME} --region ap-southeast-1
```

---

#### 5. Delete AWS Lambda Function & CloudWatch Log Groups

Clean up the aggregator function and log resources:

```bash
# Delete Lambda Function
aws lambda delete-function --function-name devsecops-report-aggregator

# Delete CloudWatch Log Groups
aws logs delete-log-group --log-group-name /ecs/devsecops-react-app
aws logs delete-log-group --log-group-name /aws/lambda/devsecops-report-aggregator
```

> **Note:** Completing all steps above ensures your AWS account incurs **$0.00 USD** ongoing operational costs.