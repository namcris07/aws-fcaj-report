---
title : "Architecture & AWS Services"
date : 2026-07-06 
weight : 2
chapter : false
pre : " <b> 5.1.2. </b> "
---

# 5.1.2 Architecture Diagram & AWS Service Rationale

---

### 1. Solution Architecture Diagram

![DevSecOps Architecture](/images/2-Proposal/devsecops_pipeline_architecture.png)

*Figure 5.1.2: Overall architecture diagram of the DevSecOps Factory on AWS.*

---

### 2. AWS Service Selection & Comparison Rationale

| AWS Service | Selection Rationale & Solution Comparison |
|---|---|
| **Amazon ECS Fargate** | **Serverless Container Runtime:** Auto-scales containers without managing EC2 instances, charging strictly per task execution second. Avoids fixed control plane costs ($72/month EKS cluster fee) and easily scales task desired counts to 0 when idle to optimize costs. |
| **Amazon ECR** | **Secure Private Container Registry:** Stores private Docker images with native Server-Side Encryption (SSE), Immutable Tag enforcement, and built-in Scan-on-Push features. |
| **Amazon S3** | **Centralized Security Report Storage:** Offers 99.999999999% (11 9s) durability, supports SSE-S3 encryption, and applies S3 Lifecycle Policies to automatically purge aged scan artifacts after 30 days. |
| **AWS Lambda** | **Event-Driven Security Ingestion:** Triggered automatically via S3 Event Notifications upon report upload, transforms security output to ASFF format, and invokes `batch_import_findings` to push findings into Security Hub. |
| **AWS Security Hub** | **Centralized CSPM & Findings Management:** Aggregates, normalizes, and manages infrastructure and application security findings in standardized ASFF format. |
| **Amazon CloudWatch** | **Observability & Budget Control:** Ingests container logs and metrics from ECS Fargate Tasks via Container Insights and triggers AWS Budget Alerts upon cost threshold breaches. |
| **AWS IAM** | **Security & Access Control:** Enforces granular, least-privilege policies across Jenkins Build Agent, ECS Task Execution Roles, and Lambda Execution Roles. |

---

### 3. IAM Security & Operational Scalability

- **Least-Privilege IAM Scoping:** Every service component (ECS, Lambda, Jenkins) is granted only the minimum required IAM actions (`ecr:PutImage`, `s3:PutObject`, `ecs:UpdateService`).
- **Auto-Scaling & Event-Driven Pipeline:** ECS Fargate scales task replicas dynamically based on HTTP traffic, while vulnerability report processing operates entirely asynchronously via S3 Event Notifications to Lambda.
- **Credential Hygiene:** Prevents hardcoded secrets by leveraging IAM Roles for Tasks, AWS Secrets Manager, and Environment Variables.