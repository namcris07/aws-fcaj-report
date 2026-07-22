---
title: "Proposal"
date: 2026-07-06
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# DevSecOps Factory on AWS
## Building an End-to-End CI/CD DevSecOps System for a React Web Application on Amazon ECS Fargate

### 1. Executive Summary
DevSecOps Factory is a complete CI/CD system that integrates security into every stage of the software development lifecycle (Shift-Left Security). The project builds an automated pipeline from code commit to application deployment on **Amazon ECS Fargate** for staging and production environments, using Jenkins as the CI engine, Argo CD (local k3d) for GitOps, Amazon S3 for centralized security scan report storage, AWS Lambda for finding aggregation, and Amazon CloudWatch as the centralized observability platform. The system adopts a **Local-first, Cloud-after** strategy — developing and testing everything locally with k3d before deploying to AWS cloud infrastructure, maximizing cost efficiency and minimizing operational risks.

The pipeline integrates 6 automated Security Gates: Secrets Scan (Gitleaks), SCA (Trivy Filesystem), SAST (SonarQube), IaC Scan (Checkov), Container Image Scan (Trivy Image), and DAST (OWASP ZAP), ensuring every build is validated and scan reports are archived in Amazon S3 prior to deployment.

### 2. Problem Statement

*Current Challenges*

In software development practice, many teams still deploy manually, lack automated security checks, and have no observability after applications run in production. Specifically:
- **Manual Deployments:** Copying files and SSH-ing into servers to deploy, leading to human error and inconsistency across environments.
- **Lack of Security Checks:** Vulnerabilities in source code, dependencies, and container images are only discovered after the application is live, creating significant risk.
- **No Centralized Report Archiving:** Security scan reports are scattered across build agents, making auditability and trend tracking difficult.
- **High Cloud Infrastructure Expenses:** Maintaining static EKS Control Planes ($72/month) is cost-prohibitive for student projects and cost-sensitive teams.
- **Configuration Drift:** Actual deployment states diverge from declared Git configurations, causing hard-to-reproduce errors.

*The Solution*

The project builds an end-to-end CI/CD DevSecOps pipeline on AWS featuring:
- **Jenkins** orchestrates a 12-stage pipeline, automating builds, 6 security scans, Docker image packaging, Amazon ECR pushing, and report uploading to Amazon S3.
- **6 Security Gates** integrated directly into the pipeline: Gitleaks (secrets), Trivy (SCA + container scan), SonarQube (SAST), Checkov (IaC scan), OWASP ZAP (DAST).
- **Amazon S3 & AWS Lambda Aggregator:** Centralized JSON/HTML scan report storage on S3 (`reports/secrets/`, `reports/sca/`, `reports/sast/`, `reports/container/`, `reports/dast/`); AWS Lambda automatically aggregates findings and manages risk thresholds.
- **Amazon ECS Fargate:** Serverless Container Runtime for staging and production environments. Zero cluster management fees, easily scaling down to 0 (`desired-count 0`) after live demos to optimize budget.
- **Argo CD & Kustomize:** Local K8s (k3d) configuration management and automated GitOps sync for staging, enforcing a manual approval gate for production.
- **Amazon CloudWatch Container Insights & Prometheus/Grafana:** Centralized monitoring for logs, metrics, container health, and AWS Budget alerts.

*Benefits and Return on Investment (ROI)*

- Reduces deployment time from hours (manual) to minutes (automated).
- Detects security vulnerabilities early during CI (Shift-Left Security), automatically archiving audit reports in S3.
- Saves > 85% in AWS infrastructure costs by transitioning from EKS to ECS Fargate Serverless.
- Provides comprehensive observability for rapid incident detection and response.
- Delivers a standardized, reusable DevSecOps framework for organizational software projects.

### 3. Solution Architecture

The system is designed following a microservices architecture, adopting a Local-first (k3d) and Cloud-after (Amazon ECS Fargate) strategy. The end-to-end workflow is as follows:

```text
React App → Jenkins Security Gates → Amazon ECR → ECS Fargate (Staging) → Scan Reports → Amazon S3 → AWS Lambda (Aggregator) → ECS Fargate (Production) → CloudWatch
```

```text
Developer pushes code to GitHub
    ↓
GitHub webhook → Jenkins pipeline
    ↓
Jenkins Jenkinsfile 12 stages:
  ① Secrets scan        (Gitleaks → S3)
  ② Build + unit tests  (npm build)
  ③ SCA                 (Trivy filesystem → S3)
  ④ SAST                (SonarQube → S3)
  ⑤ Docker build        (multi-stage Dockerfile)
  ⑥ Container scan      (Trivy image → S3)
  ⑦ IaC scan            (Checkov → S3)
  ⑧ Push image → Amazon ECR
  ⑨ Deploy Staging     (ECS Fargate / Argo CD auto-sync)
  ⑩ DAST                (OWASP ZAP vs staging URL → S3)
  ⑪ Lambda Aggregator   (Aggregate findings from S3)
  ⑫ Manual approval gate → Promote Production (ECS Fargate)
    ↓
CloudWatch Container Insights & Logs (logs, metrics, alarms)
```

![DevSecOps CI/CD Pipeline Architecture on AWS](/images/2-Proposal/devsecops_pipeline_architecture.png)

*AWS Services Used*

| AWS Service | Role in System |
|---|---|
| **Amazon ECS Fargate** | Serverless Container Engine deploying React web applications for Staging and Production with no cluster fees and auto-scaling. |
| **Amazon ECR** | Secure Docker image registry with Scan on Push and Git Commit SHA tagging. |
| **Amazon S3** | Centralized security scan report storage (`reports/secrets/`, `reports/sca/`, `reports/sast/`, `reports/container/`, `reports/dast/`) with SSE-S3 encryption and 30-day lifecycle expiration. |
| **AWS Lambda** | Event-driven function triggered by new S3 report uploads, aggregating vulnerability data and classifying risk levels. |
| **Amazon CloudWatch** | Centralized monitoring: Container Insights for ECS Task metrics, Logs Insights for queries, and Budget Alerts for cost management. |
| **AWS IAM** | Fine-grained Least Privilege permissions for Jenkins agents, ECS Task Execution Roles, and Lambda execution roles. |
| **Application Load Balancer (ALB)** | Layer 7 traffic routing for Staging and Production environments on ECS Fargate. |

*Supporting DevOps Tools*

| Tool | Role |
|---|---|
| **Jenkins** | CI engine orchestrating the 12-stage pipeline; Configuration as Code (CasC). |
| **Argo CD** | GitOps CD engine on local k3d cluster, pull-based deployment, auto-sync staging. |
| **Docker** | Packages the React app into a multi-stage container image with non-root runtime (Nginx Unprivileged). |
| **Kustomize** | Manages base/overlay configurations for staging and production. |
| **k3d** | Local Kubernetes cluster for offline development and testing prior to cloud deployment. |

*Security Gate Tools*

| Gate | Tool | Purpose | Report Destination |
|---|---|---|---|
| Secrets Scan | Gitleaks | Detects hardcoded secrets, tokens, and passwords committed to source code. | Amazon S3 |
| SCA | Trivy Filesystem | Scans CVE vulnerabilities in dependencies (npm packages). | Amazon S3 |
| SAST | SonarQube | Static code analysis to detect code injection, bugs, and code smells. | Amazon S3 |
| IaC Scan | Checkov | Validates Kubernetes manifests, Dockerfiles, and Terraform configs against security benchmarks. | Amazon S3 |
| Container Scan | Trivy Image | Scans CVE vulnerabilities in Docker base images and layers. | Amazon S3 |
| DAST | OWASP ZAP | Dynamic vulnerability scanning against web apps running on staging (security headers, XSS, injection). | Amazon S3 |

*Component Design*

- **Application Layer:** Tetris web app built with React, packaged via multi-stage Dockerfile (Node 16 build → Nginx Unprivileged runtime), exposing port 8080 with health check endpoint `/`.
- **CI Pipeline (Jenkins):** Jenkinsfile declaring 12 sequential stages with standardized environment variables (`AWS_REGION`, `REGISTRY`, `IMAGE_TAG`, `SCAN_REPORT_DIR`), automatically uploading scan reports to S3.
- **Security & Aggregation:** S3 Bucket for centralized scan report storage; Lambda Aggregator automatically analyzes severity levels and determines pipeline pass/fail status.
- **CD & Runtime (ECS Fargate / Argo CD):** Staging automatically updates to new image tags; Production enforces a Manual Approval Gate prior to initiating new ECS Tasks.
- **Observability:** CloudWatch Container Insights collects ECS task metrics/logs, Prometheus/Grafana monitors local containers, AWS Budget alerts prevent unexpected costs.

### 4. Technical Implementation

*Implementation Phases*

The project spans 9 internship weeks (15/06/2026 – 14/08/2026), structured into 3 main phases:

1. **Phase 1 — Foundation & Analysis (Weeks 1–3, 15/06 – 05/07):** Study core AWS services (VPC, IAM, EC2, ECS Fargate), analyze the existing system, create a report outline, and define the project scope. Test building the React app, Dockerfile, and Kubernetes/ECS manifests locally.

2. **Phase 2 — Pipeline, Security Gates & Deployment (Weeks 4–6, 06/07 – 26/07):** Standardize the Jenkins pipeline, integrate 6 security scan scripts, create ECR repository, S3 report bucket, and Lambda aggregator, push images tagged with commit SHAs, provision ECS Fargate cluster + ALB, deploy apps via Argo CD / ECS Services, and finalize GitOps staging/production workflows.

3. **Phase 3 — Monitoring, Documentation, Acceptance & Submission (Weeks 7–9, 27/07 – 14/08):** Enable CloudWatch Container Insights, consolidate security findings, design presentation slides, build the bilingual Hugo workshop website, write step-by-step Lab guides, write 3 blog posts, gather events/self-evaluation/feedback, audit bilingual quality, cross-check against grading criteria, submit final deliverables, scale ECS services to 0, and clean up AWS resources.

*Technical Requirements*

- **AWS Infrastructure:** AWS account with permissions for ECS Fargate, ECR, S3, Lambda, IAM, CloudWatch; region `ap-southeast-1`; AWS Budget alerts configured.
- **CI/CD:** Jenkins (Docker Compose local or EC2), Docker, Argo CD (local k3d).
- **Security:** Gitleaks, Trivy, SonarQube, Checkov, OWASP ZAP, AWS Lambda Aggregator.
- **Container & Orchestration:** Docker, Kustomize, k3d (local), Amazon ECS Fargate.
- **Documentation:** Hugo (workshop website), Markdown, Git (source code and content management).

*Team Roles*

| Member | Primary Role | Focus Areas | Email |
|---|---|---|---|
| Vu Hai An | AWS Infrastructure & Platform | AWS account, IAM, ECR, ECS Fargate, S3 (reports), networking. | 23520035@gm.uit.edu.vn |
| Huynh Nhat Linh | CI/CD & GitOps | Jenkinsfile, pipeline, push image ECR, deploy ECS Fargate, Argo CD, promotion. | linh.huynhnhat@hcmut.edu.vn |
| Bui Huu Loi | DevSecOps Security | Secrets scan, SCA, SAST, IaC scan, container scan, DAST, S3 reports, Lambda aggregator. | loi.bui2311972@hcmut.edu.vn |
| Nguyen Van Hao | Application, Docker, K8s & ECS Task | React app, Dockerfile, health check, K8s base/overlays, ECS Task Definition. | 23520448@gm.uit.edu.vn |
| Nguyen Chu Hai Nam | Observability, QA, Documentation & Demo | CloudWatch, Prometheus/Grafana, testing, workshop website, demo slides. | nam.nguyennamcris7@hcmut.edu.vn |

### 5. Timeline & Milestones

| Phase | Timeframe | Key Activities | Deliverables |
|---|---|---|---|
| Foundation & Analysis | Weeks 1–3 (15/06 – 05/07) | Study AWS, analyze system, build app/Docker locally, implement secrets scan, create report outline. | App builds successfully, Dockerfile runs locally, pipeline draft, report outline. |
| Pipeline, Security & Deployment | Weeks 4–6 (06/07 – 26/07) | Create ECR & S3 buckets, push images, configure ECS Fargate & ALB, set up Lambda aggregator, complete 6 security gates, deploy staging/production via ECS/Argo CD. | End-to-end pipeline, ECR with images, S3 storing reports, ECS Fargate running, Argo CD Synced & Healthy. |
| Monitoring, Documentation & Acceptance | Weeks 7–9 (27/07 – 14/08) | Enable CloudWatch, consolidate findings, design slides, build Hugo website, write workshop labs, write 3 blogs, gather events/evaluation, audit bilingual content, submit deliverables, scale ECS to 0 & clean up AWS. | CloudWatch dashboard, slide deck, complete workshop website, 3 blogs published, deliverables submitted, AWS resources cleaned up/scaled to 0. |

### 6. Budget Estimation

*Monthly AWS Infrastructure Costs (Estimated)*

| Service | Estimated Cost | Notes |
|---|---|---|
| Amazon ECS Fargate | ~$5.00/month | Pay per task execution time during demos; scaled to `desired-count 0` when idle. |
| Amazon ECR | ~$1.00/month | < 5 GB image storage. |
| Amazon S3 (Security Reports) | ~$0.50/month | Stores JSON/HTML reports with 30-day lifecycle policies. |
| AWS Lambda (Aggregator) | ~$0.00/month | Covered under AWS Free Tier (1M requests/month). |
| Amazon CloudWatch | ~$3.00/month | Container Insights + Logs Insights. |
| Elastic Load Balancer (ALB) | ~$10.00/month | 1 ALB shared between Staging & Production. |
| Data Transfer | ~$1.00/month | Outbound traffic. |
| **Total Estimated** | **~$20.50/month** | **Saves > 85% compared to EKS (~$131/month)** |

> **Note:** By utilizing **Amazon ECS Fargate Serverless**, the project eliminates the $73/month fixed cost of EKS Control Planes. The actual real AWS expenditure for the entire internship is estimated at only **$20 – $35 USD**.

*Cost Optimization Strategies*

- Use ECS Fargate Serverless, automatically scaling service desired count to 0 immediately following testing and demo sessions.
- Develop and test offline on local k3d clusters (free).
- Configure S3 Lifecycle Policies to automatically purge scan reports after 30 days.
- Set AWS Budget alerts at 50%, 80%, and 100% threshold levels.

### 7. Risk Assessment

*Risk Matrix*

| Risk | Impact | Probability | Mitigation Strategy |
|---|---|---|---|
| AWS Budget Overrun | Low | Low | Scale ECS Fargate to 0; AWS Budget alerts; S3 lifecycle policies. |
| ECS Fargate Task Failure During Demo | High | Low | Prepare static demo slides (screenshot backups); verify ALB health checks prior to presentation. |
| Security Scan Blocking Pipeline | Medium | Medium | Use `--soft-fail` during early phases; enable `--exit-code 1` during live demos to showcase active security enforcement. |
| Code Merge Conflicts Between Members | Medium | Medium | Dedicated branch per member (`aws-infra`, `cicd-gitops`, `security`, `app-k8s`, `observability-docs`); Pull Requests require code reviews. |
| Network Loss During Live Demo | Medium | Low | Demo offline on local k3d; screenshot backups for each pipeline step. |

*Contingency Plans*

- If ECS Fargate encounters issues: Switch to demo on local k3d cluster with local Docker registry.
- If Jenkins fails: Present via static demo slides with screenshots of successful pipeline execution logs.
- Immediately after live demo: Execute `aws ecs update-service --desired-count 0` to pause container tasks and halt charges.

### 8. Expected Outcomes

*Technical Improvements*
- Successfully build an end-to-end CI/CD DevSecOps pipeline over 9 weeks on Amazon ECS Fargate, S3, and Lambda.
- Integrate 6 automated security gates, centralizing scan report storage in S3 and aggregating findings via Lambda.
- Implement automated GitOps staging sync and production controls via manual approval gates.
- Comprehensive monitoring via CloudWatch Container Insights and Prometheus/Grafana at optimized costs (~$20/month).

*Deliverables*
- Bilingual workshop website (Vietnamese – English) following FCAJ template standards with step-by-step Lab guides.
- Project source code on GitHub with clear branch structures and detailed `tasks.md` specifications.
- 3 blog posts published on AWS Study Group.
- Project presentation slides and an operational screenshot evidence package.
- Consolidated security findings table with remediation recommendations.
- Automated AWS cleanup and scale-to-0 execution scripts.

*Long-term Value*
- Highly viable, cost-optimized Serverless DevSecOps architecture model suitable for SMB enterprises.
- The workshop website serves as a premium reference resource for future internship cohorts.
- Extensive hands-on experience across DevSecOps, ECS Fargate, S3/Lambda, GitOps, and CloudWatch for all team members.