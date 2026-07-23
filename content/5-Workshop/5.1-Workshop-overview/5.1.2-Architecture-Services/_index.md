---
title : "Architecture & AWS Services"
date : 2026-07-06 
weight : 2
chapter : false
pre : " <b> 5.1.2. </b> "
---

# 5.1.2 Architecture Diagram & AWS Service Selection

---

### 1. System Architecture Diagram

![DevSecOps Architecture](/images/2-Proposal/devsecops_pipeline_architecture.png)

*Figure 5.1.2: Overall DevSecOps Factory architecture on AWS.*

---

### 2. AWS Service Selection Rationale

| AWS Service | Technical Rationale & Comparison |
|---|---|
| **Amazon ECS Fargate** | **Serverless Container Runtime:** Scales automatically and charges strictly by task execution time. Eliminates fixed cluster management fees ($72/month for EKS), allowing tasks to scale to 0 when idle. |
| **Amazon ECR** | **Secure Private Registry:** Private Docker image registry supporting Server-Side Encryption, Tag Immutability, and Scan-on-Push. |
| **Amazon S3** | **Centralized Report Storage:** 99.999999999% (11 9's) durability, supporting SSE-S3 encryption and S3 Lifecycle Policies for cost optimization. |
| **AWS Lambda** | **Event-Driven Processing:** Triggers automatically on S3 report uploads via S3 Event Notifications, parsing vulnerability metrics without requiring persistent servers. |
| **Amazon CloudWatch** | **Observability & Budget Control:** Collects telemetry via Container Insights and manages cost alarms via AWS Budget Alerts. |
| **AWS IAM** | **Security & Access Control:** Enforces Least-Privilege permissions across Jenkins Build Agents, ECS Execution Roles, and Lambda Functions. |

---

### 3. IAM Security & Operational Scalability

- **Least-Privilege IAM Policies:** Restricting service actions to minimal required permissions (`ecr:PutImage`, `s3:PutObject`, `ecs:UpdateService`).
- **Auto Scaling & Event-Driven Flow:** ECS Fargate scales dynamically; security report processing is 100% event-driven (S3 -> Lambda).
- **No Hardcoded Credentials:** Utilizing IAM Roles for Tasks and Systems Manager Parameter Store.
