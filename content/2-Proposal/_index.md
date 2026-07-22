---
title: "Proposal"
date: 2026-07-06
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

In this section, you need to summarize the contents of the workshop that you **plan** to conduct.

# DevSecOps Factory on AWS
## Building an End-to-End CI/CD DevSecOps System for a React Web Application on Amazon EKS

### 1. Executive Summary
DevSecOps Factory is a complete CI/CD system that integrates security into every stage of the software development lifecycle (Shift-Left Security). The project builds an automated pipeline from the moment a developer pushes code until the application is deployed to staging and production environments on Amazon EKS, using Jenkins as the CI engine, Argo CD as the GitOps CD engine, and Amazon CloudWatch as the monitoring platform. The system adopts a **Local-first, Cloud-after** strategy — developing and testing everything on local machines with k3d before migrating to real AWS infrastructure, saving costs and reducing risks during development.

The pipeline integrates 6 automated Security Gates: Secrets Scan, SCA (Software Composition Analysis), SAST (Static Application Security Testing), IaC Scan, Container Image Scan, and DAST (Dynamic Application Security Testing), ensuring every build is validated before going live.

### 2. Problem Statement

*Current Challenges*

In software development practice, many teams still deploy manually, lack automated security checks, and have no observability after applications run in production. Specifically:
- **Manual Deployments:** Copying files, SSH-ing into servers to deploy, leading to human error and inconsistency across environments.
- **Lack of Security Checks:** Vulnerabilities in source code, dependencies, and container images are only discovered after the application is live, creating significant risk.
- **No Observability:** When applications fail in production, teams have no dashboards, centralized logs, or alerts to detect and respond in time.
- **Configuration Drift:** The actual state on Kubernetes clusters gradually diverges from the declared configuration in Git, causing hard-to-reproduce errors.

*The Solution*

The project builds an end-to-end CI/CD DevSecOps pipeline on AWS with key components:
- **Jenkins** orchestrates a 12-stage pipeline, automating build, security scanning, Docker image packaging, and pushing to Amazon ECR.
- **6 Security Gates** integrated directly into the pipeline: Gitleaks (secrets), Trivy (SCA + container scan), SonarQube (SAST), Checkov (IaC scan), OWASP ZAP (DAST).
- **Argo CD** applies pull-based GitOps deployment to automatically synchronize manifests from the Git repository to Amazon EKS, ensuring the cluster state always matches the declared configuration.
- **Amazon CloudWatch Container Insights** provides centralized monitoring for logs, metrics, and performance alerts.

*Benefits and Return on Investment (ROI)*

- Reduces deployment time from hours (manual) to minutes (automated).
- Detects security vulnerabilities early during CI, before code reaches production.
- Prevents configuration drift through GitOps — all infrastructure changes are version-controlled via Git.
- Provides comprehensive observability for rapid incident detection and response.
- Creates a reusable foundation for future software projects within the organization.

### 3. Solution Architecture

The system is designed using a microservices architecture on Kubernetes, utilizing Kustomize to manage configurations for multiple environments (staging/production). The end-to-end workflow is as follows:

```text
React App → Docker Image → Jenkins Security Gates → Amazon ECR → Argo CD → Amazon EKS → CloudWatch
```

```text
Developer pushes code to GitHub
    ↓
GitHub webhook → Jenkins pipeline
    ↓
Jenkins Jenkinsfile 12 stages:
  ① Secrets scan        (Gitleaks)
  ② Build + unit tests  (npm build)
  ③ SCA                 (Trivy filesystem)
  ④ SAST                (SonarQube)
  ⑤ Docker build        (multi-stage Dockerfile)
  ⑥ Container scan      (Trivy image)
  ⑦ IaC scan            (Checkov)
  ⑧ Push image → Amazon ECR
  ⑨ Update image tag → Argo CD auto-sync staging
  ⑩ DAST                (OWASP ZAP vs staging URL)
  ⑪ Manual approval gate
  ⑫ Promote → production
    ↓
CloudWatch Container Insights (logs, metrics, alarms)
```

![DevSecOps CI/CD Pipeline Architecture on AWS](/images/2-Proposal/devsecops_pipeline_architecture.png)

*AWS Services Used*

| AWS Service | Role in the System |
|---|---|
| **Amazon EKS** | Kubernetes container orchestration for the web application, managed Node Group with `t3.small` instance type, multi-namespace support (staging/production). |
| **Amazon ECR** | Secure Docker image storage with Scan on Push, image tagging by Git Commit SHA. |
| **Amazon CloudWatch** | Centralized monitoring: Container Insights for Node/Pod metrics, Logs Insights for application log queries, Budget Alerts for cost warnings. |
| **AWS IAM** | Fine-grained permissions following Least Privilege; IRSA (IAM Roles for Service Accounts) for EKS workloads. |
| **AWS Load Balancer Controller** | Auto-provisions Application Load Balancers (ALB) from Kubernetes Ingress configurations, Layer 7 routing. |

*Supporting DevOps Tools*

| Tool | Role |
|---|---|
| **Jenkins** | CI engine orchestrating the 12-stage pipeline; Configuration as Code (CasC). |
| **Argo CD** | GitOps CD engine, pull-based deployment, auto-sync staging, manual approval for production. |
| **Docker** | Packages the React application into a multi-stage container image with non-root runtime (Nginx Unprivileged). |
| **Kustomize** | Manages Kubernetes base/overlays configurations for staging and production. |
| **k3d** | Local Kubernetes cluster for offline development and testing before cloud migration. |

*Security Gate Tools*

| Gate | Tool | Purpose |
|---|---|---|
| Secrets Scan | Gitleaks | Detects secrets, tokens, and passwords committed to source code. |
| SCA | Trivy Filesystem | Scans CVE vulnerabilities in dependencies (npm packages). |
| SAST | SonarQube | Static code analysis to detect code injection, bugs, and code smells. |
| IaC Scan | Checkov | Validates Kubernetes manifests and Dockerfiles against security benchmarks. |
| Container Scan | Trivy Image | Scans CVE vulnerabilities in Docker base images and layers. |
| DAST | OWASP ZAP | Scans running web applications on staging (security headers, XSS, injection). |

*Component Design*

- **Application Layer:** A Tetris web application built with React, packaged via multi-stage Dockerfile (Node 16 build → Nginx Unprivileged runtime), exposing port 8080 with health check endpoint `/`.
- **CI Pipeline (Jenkins):** Jenkinsfile declaring 12 sequential stages, standardized environment variables (`AWS_REGION`, `REGISTRY`, `IMAGE_TAG`), security reports archived as artifacts.
- **CD GitOps (Argo CD):** Application CRDs for staging (auto-sync) and production (manual sync), tracking image tag changes in `kustomization.yaml`.
- **Kubernetes Infrastructure (EKS):** Deployments with Liveness/Readiness Probes, SecurityContext (runAsNonRoot, drop ALL capabilities, readOnlyRootFilesystem), Services and Ingress ALB.
- **Observability:** CloudWatch Container Insights with DaemonSet agent, Fluent Bit for log collection, CloudWatch Logs Insights for queries, AWS Budget for cost alerts.

### 4. Technical Implementation

*Implementation Phases*

The project spans 9 internship weeks (15/06/2026 – 14/08/2026), structured into 3 main phases:

1. **Phase 1 — Foundation & Analysis (Weeks 1–3, 15/06 – 05/07):** Study core AWS services (VPC, IAM, EC2, EKS), analyze the existing system, create a report outline, and define the project scope. Test building the React app, Dockerfile, and Kubernetes manifests locally.

2. **Phase 2 — Pipeline, Security Gates & Deployment (Weeks 4–6, 06/07 – 26/07):** Standardize the Jenkins pipeline, integrate 6 security scan scripts, create the ECR repository, push images tagged with commit SHAs, provision the EKS cluster, install AWS Load Balancer Controller, deploy applications via Argo CD, and finalize the GitOps staging/production workflow.

3. **Phase 3 — Monitoring, Documentation, Acceptance & Submission (Weeks 7–9, 27/07 – 14/08):** Enable CloudWatch Container Insights, consolidate security findings, design presentation slides, build the bilingual Hugo workshop website, write step-by-step Lab guides, write 3 blog posts, gather events/self-evaluation/feedback, audit bilingual quality, cross-check against grading criteria, submit the final report, and clean up AWS resources.

*Technical Requirements*

- **AWS Infrastructure:** AWS account with permissions for EKS, ECR, IAM, CloudWatch; region `ap-southeast-1`; AWS Budget alerts configured.
- **CI/CD:** Jenkins (Docker Compose local or EC2), Docker, Argo CD installed on EKS via Helm.
- **Security:** Gitleaks, Trivy, SonarQube (Docker Compose), Checkov, OWASP ZAP.
- **Kubernetes:** kubectl, Kustomize, k3d (local), Helm (AWS Load Balancer Controller, Argo CD).
- **Documentation:** Hugo (workshop website), Markdown, Git (source code and content management).

*Team Roles*

| Member | Primary Role | Focus Areas |
|---|---|---|
| Member 1 | AWS Infrastructure & Kubernetes Platform | AWS account, IAM, EKS, networking, ECR, deployment platform. |
| Member 2 | CI/CD & GitOps | Jenkinsfile, pipeline, push image, Argo CD, staging/production promotion. |
| Member 3 | DevSecOps Security | Secrets scan, SCA, SAST, IaC scan, container scan, DAST, security policies. |
| Member 4 | Application, Docker & K8s Manifests | React app, Dockerfile, health check, Kubernetes base/overlays. |
| Member 5 | Observability, QA, Documentation & Demo | CloudWatch, testing, workshop website, reports, demo slides. |

### 5. Timeline & Milestones

| Phase | Timeframe | Key Activities | Deliverables |
|---|---|---|---|
| Foundation & Analysis | Weeks 1–3 (15/06 – 05/07) | Study AWS, analyze the system, build app/Docker locally, implement secrets scan, create report outline. | App builds successfully, Dockerfile runs locally, pipeline draft, report outline. |
| Pipeline, Security & Deployment | Weeks 4–6 (06/07 – 26/07) | Create ECR, push images, provision EKS cluster, install ALB Controller, complete 6 security gates, deploy staging/production via Argo CD. | End-to-end pipeline, ECR with images, EKS nodes Ready, Argo CD Synced & Healthy. |
| Monitoring, Documentation & Acceptance | Weeks 7–9 (27/07 – 14/08) | Enable CloudWatch, consolidate findings, design slides, build Hugo website, write workshop labs, write 3 blogs, gather events/evaluation, audit bilingual content, submit deliverables, clean up AWS. | CloudWatch dashboard, slide deck, complete workshop website, 3 blogs published, deliverables submitted, AWS cleaned up. |

### 6. Budget Estimation

*Monthly AWS Infrastructure Costs (Estimated)*

| Service | Estimated Cost | Notes |
|---|---|---|
| Amazon EKS (Control Plane) | ~$73.00/month | 1 cluster. |
| EC2 Managed Node Group (t3.small × 2) | ~$30.00/month | 2 nodes, On-Demand pricing. |
| Amazon ECR | ~$1.00/month | < 5 GB image storage. |
| Amazon CloudWatch | ~$5.00/month | Container Insights + Logs Insights. |
| Elastic Load Balancer (ALB) | ~$20.00/month | 1 ALB for staging + production. |
| Data Transfer | ~$2.00/month | Outbound traffic. |
| **Total Estimated** | **~$131/month** | |

> **Note:** The primary costs come from the EKS Control Plane and EC2 nodes. To minimize costs during development, the team adopts a **Local-first** strategy with k3d — only provisioning the EKS cluster on AWS for actual demos, and cleaning up immediately afterward. The estimated total real AWS cost for the entire project is approximately **$150–$250** (as the cloud environment runs for only 1–2 weeks).

*Cost Optimization Strategies*

- Develop and test on local k3d clusters (free), only migrating to EKS when needed.
- Use AWS Budget alerts at 50%, 80%, and 100% thresholds for early warnings.
- Thoroughly clean up all resources (EKS, ALB, ECR, CloudWatch) immediately after demos.
- Use cost-effective instance types (`t3.small`) and maintain minimum node counts.

### 7. Risk Assessment

*Risk Matrix*

| Risk | Impact | Probability | Mitigation Strategy |
|---|---|---|---|
| AWS Budget Overrun | High | Medium | AWS Budget alerts; Local-first strategy; immediate cleanup after demo; use t3.small instances. |
| EKS Cluster Failure During Demo | High | Low | Prepare static demo slides (screenshot backups); thoroughly test before defense; use local k3d as fallback. |
| Security Scan Blocking Pipeline | Medium | Medium | Use `--soft-fail` during early phases; enable `--exit-code 1` during demo to showcase real security gates. |
| Code Merge Conflicts Between Members | Medium | Medium | Each member works on dedicated branches; Pull Requests require reviews; CODEOWNERS enforced. |
| Network Loss During Live Demo | Medium | Low | Demo offline on local k3d; screenshot backups for each pipeline step. |
| Outdated Dependencies Causing Build Failures | Low | Medium | Document dependency issues in `VULNERABILITIES.md`; accept some warnings for security scan demo purposes. |

*Contingency Plans*

- If EKS is unavailable: Switch to demo on local k3d cluster with local Docker registry.
- If Jenkins fails: Present via static demo slides with screenshots of successful pipeline logs.
- If budget is exceeded: Immediately terminate the EKS cluster, revert to local-only; use CloudFormation/Terraform for automated cleanup.

### 8. Expected Outcomes

*Technical Improvements*
- Successfully build an end-to-end CI/CD DevSecOps pipeline within 9 weeks, running completely from code push to production deployment.
- Integrate 6 automated security gates that detect and report vulnerabilities before code reaches production.
- Implement GitOps with Argo CD, auto-syncing staging and controlling production via manual approval.
- Comprehensive monitoring via CloudWatch Container Insights with dashboards, log queries, and cost alerts.

*Deliverables*
- Bilingual workshop website (Vietnamese – English) following FCAJ template standards, with step-by-step Lab guides.
- Project source code on GitHub with clear structure and complete documentation.
- 3 blog posts published on AWS Study Group.
- Project presentation slides and a set of operational evidence screenshots.
- Consolidated security findings table with remediation recommendations.
- AWS cleanup checklist to prevent post-project cost overruns.

*Long-term Value*
- The pipeline model can be reused for other software projects within the organization.
- The workshop website becomes a reference resource for future internship cohorts.
- Hands-on experience with DevSecOps, Kubernetes, GitOps, and AWS services for all team members.