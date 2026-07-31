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

**End-to-end flow (12 steps):**

1. **Commit/Push** — Developer pushes code to Git SCM.
2. **Trigger/Poll SCM** — On-premise Jenkins polls SCM and receives a webhook to trigger the pipeline.
3. **Run Security Gates** — Jenkins executes 7 sequential security checks:
   - 3.a **Secret Scan** (git-secret) → detect hardcoded secrets
   - 3.b **SCA Scan** (Trivy Filesystem) → detect CVEs in dependencies
   - 3.c **SAST Scan** (SonarQube) → static source code analysis
   - 3.d **IaC Scan** (Checkov) → validate Dockerfile and K8s manifests
   - 3.e **Build Image** (Docker) → package the React app into a container image
   - 3.f **Container Scan** (Trivy Image) → scan the container image for vulnerabilities
   - 3.g **DAST Scan** (OWASP ZAP) → dynamic security scan against the running app
4. **Push Image** — Push Docker image to Amazon ECR (tagged by commit SHA).
5. **Deploy to ECS Staging** — Deploy the new image to ECS Fargate Staging (AZ 1) via ALB test traffic.
6. **Pull Image** — ECS Fargate pulls the image from Amazon ECR.
7. **DAST Endpoint Test** — OWASP ZAP scans the live staging endpoint.
8. **Promote** — After DAST passes & Manual Approval, promote to ECS Fargate Production (AZ 2).
9. **Upload Reports/ASFF** — Jenkins uploads all JSON/HTML reports and ASFF files to Amazon S3.
10. **S3 Event Trigger** — S3 automatically triggers AWS Lambda when a new file arrives.
11. **BatchImportFindings** — Lambda converts reports to ASFF format and pushes them into AWS Security Hub.
12. **Logs/Metrics** — ECS Fargate streams logs and metrics to Amazon CloudWatch.

---

### 2. AWS Service Selection & Comparison Rationale

#### 2.1 Identity, Audit & Cost Guardrails Layer

| AWS Service | Role in the System |
|---|---|
| **AWS IAM Roles** | Enforces least-privilege access for Jenkins Agent, ECS Task Execution Role, and Lambda Execution Role. No static keys — all access is role-based. |
| **AWS CloudTrail** | Records all API calls across the AWS account for audit and security incident investigation. |
| **AWS Config** | Continuously evaluates AWS resource configuration state against compliance rules and detects configuration drift. |
| **AWS Budgets** | Sets cost threshold alerts at 50%, 80%, and 100% to keep infrastructure spending under control throughout the internship. |

#### 2.2 Artifact & Report Storage Layer

| AWS Service | Selection Rationale & Solution Comparison |
|---|---|
| **Amazon ECR** | **Secure Private Container Registry:** Stores private Docker images with native Server-Side Encryption (SSE), Immutable Tag enforcement, and built-in Scan-on-Push features. Images are tagged by Git Commit SHA for full traceability. |
| **Amazon S3** | **Centralized Security Report Storage:** Offers 99.999999999% (11 9s) durability, SSE-S3 encryption, and S3 Lifecycle Policies to automatically purge aged artifacts after 30 days. Stores all raw reports and normalized ASFF files. |

#### 2.3 Application Runtime - VPC Layer

| AWS Service | Selection Rationale & Solution Comparison |
|---|---|
| **Application Load Balancer (ALB)** | Layer 7 traffic routing: splits test traffic to ECS Fargate Staging (AZ 1) and production traffic to ECS Fargate Production (AZ 2). |
| **Amazon ECS Fargate** | **Serverless Container Runtime:** No fixed cluster management cost ($72/month EKS control plane fee avoided), billed strictly per task execution second. Easily scales `desired-count` to 0 post-demo to optimize budget. Deployed across 2 Availability Zones: **Staging (AZ 1)** and **Production (AZ 2)**. |

#### 2.4 Security Findings Processing Layer

| AWS Service | Selection Rationale & Solution Comparison |
|---|---|
| **AWS Lambda** | **Event-Driven Security Ingestion:** Triggered automatically via S3 Event Notifications upon report upload, converts findings to ASFF format, and invokes `BatchImportFindings` to push data into Security Hub. Falls within the AWS Free Tier (1M requests/month). |
| **AWS Security Hub** | **Central Findings Dashboard:** Aggregates and normalizes all security findings from 6 security gates in ASFF format. Single pane of glass for all security detections. |
| **Amazon CloudWatch** | **Observability & Budget Control:** Container Insights ingests logs and metrics from ECS Fargate Tasks; Logs Insights enables application log queries; Budget Alerts notify on cost threshold breaches. |

---

### 3. IAM Security & Operational Scalability

- **Least-Privilege IAM Scoping:** Every service component (ECS, Lambda, Jenkins) is granted only the minimum required IAM actions (`ecr:PutImage`, `s3:PutObject`, `ecs:UpdateService`, `securityhub:BatchImportFindings`).
- **Continuous Audit & Compliance:** AWS CloudTrail logs every API call; AWS Config evaluates configuration drift in real time.
- **Proactive Cost Control:** AWS Budgets sends email alerts at 50%/80%/100% of the budget threshold to prevent uncontrolled spending.
- **Auto-Scaling & Event-Driven Pipeline:** ECS Fargate scales task replicas dynamically based on HTTP traffic through the ALB; vulnerability report processing operates entirely asynchronously via S3 Event → Lambda → Security Hub.
- **Credential Hygiene:** Prevents hardcoded secrets by leveraging IAM Roles for Tasks and AWS Secrets Manager. Jenkins credentials are managed via the Jenkins Credentials Store.
- **High Availability:** The application is deployed across 2 Availability Zones (Staging at AZ 1, Production at AZ 2) with automatic ALB load balancing.
- **Auto-Scaling & Event-Driven Pipeline:** ECS Fargate scales task replicas dynamically based on HTTP traffic, while vulnerability report processing operates entirely asynchronously via S3 Event Notifications to Lambda.
- **Credential Hygiene:** Prevents hardcoded secrets by leveraging IAM Roles for Tasks, AWS Secrets Manager, and Environment Variables.