---
title: "Project Proposal"
date: 2026-07-06
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# DevSecOps Factory on AWS

## Building an End-to-End DevSecOps CI/CD Pipeline for a React Web App on Amazon ECS Fargate

### 1. Executive Summary

DevSecOps Factory is a complete CI/CD system integrating security into every phase of the software development lifecycle (**Shift-Left Security**). The project provisions an automated pipeline from the developer push event to application deployment on staging and production environments on **Amazon ECS Fargate**. The architecture leverages Jenkins as the CI orchestrator, Argo CD (local k3d) for GitOps, Amazon S3 for centralized security report storage, AWS Lambda for finding aggregation, and Amazon CloudWatch for centralized observability. The system enforces a **Local-first, Cloud-after** strategy — developing and testing completely on local workstations using k3d before promoting to AWS cloud infrastructure, eliminating risk and minimizing operational costs.

The pipeline incorporates 6 automated security control checkpoints (Security Gates): Secrets Scan (Gitleaks), SCA (Trivy Filesystem), SAST (SonarQube), IaC Scan (Checkov), Container Image Scan (Trivy Image), and DAST (OWASP ZAP). Every build is verified and archived centrally to Amazon S3 prior to release.

### 2. Problem Statement

*Current Challenges*

In software engineering practices, teams frequently suffer from manual deployments, lack of automated security controls, and absent post-deployment observability in production:

- **Manual Deployments:** Manual file copying and SSH server access lead to human errors and environment drift.
- **Missing Security Checks:** Vulnerabilities in source code, third-party libraries, and container images are uncovered post-production release, causing severe security risks.
- **Scattered Security Artifacts:** Scan reports remain fragmented across worker nodes, hindering historical tracking.
- **High Cloud Infrastructure Expenses:** Maintaining static control planes (such as $72/month EKS fees) burdens smaller project budgets.
- **Configuration Drift:** Actual cloud environments diverge from Git declarations, causing non-reproducible bugs.

*Proposed Solution*

The project builds an end-to-end DevSecOps CI/CD pipeline on AWS comprising core components:

- **Jenkins Engine:** Orchestrates a 22-stage automated pipeline covering code build, 6 security scans, Docker multi-stage packaging, ECR image push, local registry mirroring, ECS Fargate staging deployment, DAST scanning, ASFF schema conversion to S3/Lambda/Security Hub, and production approval gates.
- **6 Security Gates:** Directly integrated into the pipeline: Gitleaks (secrets), Trivy (SCA + container image), SonarQube (SAST), Checkov (IaC scan), OWASP ZAP (DAST).
- **Amazon S3 & AWS Lambda Aggregator:** Centralizes JSON/HTML scan reports on S3 (`reports/secrets/`, `reports/sca/`, `reports/sast/`, `reports/container/`, `reports/dast/`); AWS Lambda automatically ingests findings into Security Hub.
- **Amazon ECS Fargate:** Serverless Container Runtime for staging and production. Incurs zero cluster base fees and easily scales task desired counts to zero (`desired-count 0`) when idle.
- **Argo CD & Kustomize:** Manages local Kubernetes configurations (k3d) with GitOps auto-sync for staging and interactive manual approval gates for production.
- **Amazon CloudWatch & Prometheus/Grafana:** Delivers centralized monitoring for logs, metrics, container health, and AWS Budget alerts.

*Benefits & Return on Investment (ROI)*

- Accelerates deployment cycles from hours (manual) down to minutes (automated).
- Identifies security vulnerabilities early during CI (Shift-Left Security) with immutable report logging on S3.
- Reduces AWS infrastructure baseline costs by 85% by selecting ECS Fargate Serverless over EKS.
- Provides comprehensive end-to-end observability for rapid troubleshooting.
- Establishes a standardized, reusable DevSecOps factory blueprint for enterprise projects.

### 3. Solution Architecture

The solution adheres to a microservices architecture employing a Local-first (k3d) and Cloud-after (Amazon ECS Fargate) paradigm. The end-to-end workflow operates as follows:

```text
React App → Jenkins Security Gates → Amazon ECR → ECS Fargate (Staging) → Scan Reports → Amazon S3 → AWS Lambda (Aggregator) → ECS Fargate (Production) → CloudWatch
```

```text
Developer pushes code to GitHub
    ↓
GitHub Webhook triggers Jenkins pipeline
    ↓
Jenkinsfile 22-stage pipeline execution:
  ① Environment Preflight & Checkout
  ② Static Validation (scripts/validate.sh: shell, Python, Kustomize, Docker, Terraform)
  ③ Secrets Scan (Gitleaks → gitleaks-report.json)
  ④ SCA Scan (Trivy filesystem → trivy-sca-report.json/html)
  ⑤ SAST Scan (SonarQube code analysis → sonar-issues.json)
  ⑥ IaC Scan (Checkov → checkov_report.json)
  ⑦ Application Build (React app npm build)
  ⑧ Container Image Build (Multi-stage Dockerfile Node 16 → Nginx)
  ⑨ Container Image Scan (Trivy image → container-scan-report.json)
  ⑩ Push Image → Amazon ECR (tag commit SHA & latest)
  ⑪ Local Registry Mirror (Registry localhost:5001)
  ⑫ Deploy Local GitOps Staging (Argo CD auto-sync)
  ⑬ Deploy AWS ECS Fargate Staging (tetris-staging service)
  ⑭ DAST Scan (OWASP ZAP vs staging ALB URL)
  ⑮ Normalize Reports & ASFF Conversion (securityhub-asff.json)
  ⑯ Upload Security Reports → Amazon S3 (s3://bucket/reports/)
  ⑰ AWS Lambda Importer → Ingest Findings → AWS Security Hub
  ⑱ Observability Verification (CloudWatch Container Insights & Prometheus/Grafana)
  ⑲ Production Manual Approval Gate (Interactive Jenkins proceed/abort)
  ⑳ Deploy Local GitOps Production (Argo CD sync)
  ㉑ Deploy AWS ECS Fargate Production (tetris-production service)
  ㉒ Summary & Artifact Archiving (scan-reports/ & final evidence recording)
    ↓
CloudWatch Container Insights & Logs (logs, metrics, alarms)
```

![DevSecOps CI/CD Pipeline Architecture on AWS](/images/2-Proposal/devsecops_pipeline_architecture.png)

*AWS Services Utilized*

| AWS Service | Core System Role |
|---|---|
| **Amazon ECS Fargate** | Serverless Container Engine deploying React web applications for Staging and Production with zero cluster management fees. |
| **Amazon ECR** | Secure private container registry featuring Scan-on-Push and Git Commit SHA image tag immutability. |
| **Amazon S3** | Centralized security report storage (`reports/secrets/`, `reports/sca/`, `reports/sast/`, `reports/container/`, `reports/dast/`) with SSE-S3 encryption and 30-day lifecycle rules. |
| **AWS Lambda** | Event-driven function triggered on S3 report uploads, aggregating vulnerability findings into ASFF schema. |
| **Amazon CloudWatch** | Centralized monitoring: Container Insights ingests ECS task metrics, Logs Insights queries app logs, Budget Alerts enforce cost guardrails. |
| **AWS IAM** | Granular least-privilege scoping across Jenkins agent, ECS Task Execution Roles, and Lambda execution roles. |
| **Application Load Balancer (ALB)** | Layer 7 traffic routing for ECS Fargate Staging and Production environments. |

*DevOps Supporting Tooling*

| Tooling | Role & Function |
|---|---|
| **Jenkins** | CI orchestration engine executing the **22-stage** pipeline; managed via Configuration as Code (CasC). |
| **Argo CD** | GitOps CD engine running on local k3d cluster with pull-based automated staging sync. |
| **Docker** | Packages React web app into multi-stage, non-root runtime container images (Nginx Unprivileged). |
| **Kustomize** | Manages Kubernetes base/overlay manifests for staging and production environments. |
| **k3d** | Local Kubernetes cluster enabling offline development prior to cloud deployment. |

*Security Control Gates*

| Gate | Scanner Tool | Target Vulnerability Vector | Report Storage Target |
|---|---|---|---|
| Secrets Scan | Gitleaks | Hardcoded API keys, tokens, passwords in git history | Amazon S3 |
| SCA | Trivy Filesystem | Dependency CVE vulnerabilities in npm packages | Amazon S3 |
| SAST | SonarQube | Static code analysis detecting XSS, injection, code smells | Amazon S3 |
| IaC Scan | Checkov | Infrastructure misconfigurations in Terraform, Dockerfile, K8s | Amazon S3 |
| Container Scan | Trivy Image | OS base image CVE vulnerabilities in container layers | Amazon S3 |
| DAST | OWASP ZAP | Runtime web application flaws on running staging endpoints | Amazon S3 |

---

### 4. Technical Implementation & Roadmap

*Project Phases*

The project is structured across 9 internship weeks (June 15, 2026 – August 14, 2026) divided into 3 primary phases:

1. **Phase 1 — Foundations & System Analysis (Weeks 1–3, June 15 – July 05):** Study AWS fundamentals (VPC, IAM, EC2, ECS Fargate), analyze legacy workflow, draft project outline, and construct local React app, Dockerfile, and Kubernetes/ECS manifests.
2. **Phase 2 — Pipeline Construction, Security Gates & Cloud Deployment (Weeks 4–6, July 06 – July 26):** Standardize Jenkins pipeline, integrate 6 security scan scripts, provision ECR & S3 buckets, deploy Lambda aggregator, enforce SHA commit tagging, configure ECS Fargate & ALBs, and achieve GitOps deployment.
3. **Phase 3 — Observability, Quality Assurance, Documentation & Teardown (Weeks 7–9, July 27 – August 14):** Enable CloudWatch Container Insights, aggregate security findings, construct Hugo workshop documentation site, publish 3 technical blog posts, complete bilingual quality audits, submit final deliverables, scale ECS tasks to 0, and tear down cloud resources.

---

### 5. Estimated AWS Operating Cost (< $20.50/month)

| AWS Cloud Service | Estimated Monthly Charge | Usage Specification |
|---|---|---|
| Amazon ECS Fargate | ~$5.00 USD/month | Pay-per-execution second during demos; scaled to `desired-count 0` when idle. |
| Amazon ECR | ~$1.00 USD/month | Private registry storage (< 5 GB image layers). |
| Amazon S3 (Security Reports) | ~$0.50 USD/month | Stores JSON/HTML scan reports under 30-day lifecycle policies. |
| AWS Lambda (Aggregator) | ~$0.00 USD/month | Operates within AWS Free Tier allowance (1M requests/month). |
| Amazon CloudWatch | ~$3.00 USD/month | Ingests Container Insights metrics & Logs Insights data. |
| Elastic Load Balancer (ALB) | ~$10.00 USD/month | Shared Application Load Balancer routing Staging and Production. |
| Data Transfer Out | ~$1.00 USD/month | Outbound HTTP demo traffic. |
| **Total Estimated Expense** | **~$20.50 USD/month** | **> 85% cost savings compared to EKS (~$131/month)** |

---

### 6. Risk Assessment Matrix

| Risk Factor | Impact Level | Likelihood | Mitigation Strategy |
|---|---|---|---|
| AWS Cost Overruns | Low | Low | Enforce ECS Fargate scale-to-zero; set AWS Budget alerts at 50%/80%/100%. |
| ECS Fargate Demo Outage | High | Low | Prepare static backup screenshots; verify ALB health check endpoints prior to defense. |
| Security Scan Blocks Pipeline | Medium | Medium | Use `--soft-fail` initially; enable strict `--exit-code 1` enforcement during final verification. |
| Code Merge Conflicts | Medium | Medium | Work on isolated feature branches (`aws-infra`, `cicd-gitops`, `security`); enforce mandatory PR reviews. |
| Network Instability | Medium | Low | Maintain offline k3d local demo environment with cached images. |

---

### 7. References

The solution is engineered according to official security standards published by AWS, NIST, and OWASP:

1. **Amazon Web Services (2024)**, *Amazon ECS Developer Guide*, [AWS Documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/).
2. **Amazon Web Services (2024)**, *Amazon ECR User Guide*, [AWS Documentation](https://docs.aws.amazon.com/AmazonECR/latest/userguide/).
3. **Amazon Web Services (2024)**, *AWS Security Hub User Guide & Security Finding Format (ASFF)*, [AWS Documentation](https://docs.aws.amazon.com/securityhub/latest/userguide/).
4. **HashiCorp (2024)**, *Terraform AWS Provider Documentation*, [Terraform Registry](https://registry.terraform.io/providers/hashicorp/aws/latest/docs).
5. **NIST (2017)**, *Special Publication 800-190: Application Container Security Guide*, National Institute of Standards and Technology.
6. **OWASP Foundation (2021)**, *OWASP Top Ten 2021 & OWASP ZAP Documentation*, [OWASP Official Site](https://owasp.org/www-project-top-ten/).
7. **Aqua Security (2024)**, *Trivy -- Vulnerability Scanner for Containers*, [Aqua Security Site](https://aquasecurity.github.io/trivy/).
8. **Zachary Rice (2024)**, *Gitleaks -- SAST Tool for Detecting Hardcoded Secrets*, [GitHub Repository](https://github.com/gitleaks/gitleaks).
9. **SonarSource (2024)**, *SonarQube Documentation -- Code Quality and Security*, [SonarSource Site](https://docs.sonarsource.com/sonarqube/).
10. **Bridgecrew / Palo Alto Networks (2024)**, *Checkov -- Infrastructure as Code Static Analysis*, [Checkov Site](https://www.checkov.io/).
11. **Jenkins Project (2024)**, *Jenkins Declarative Pipeline Documentation*, [Jenkins Official Site](https://www.jenkins.io/doc/).
12. **Argo Project (2024)**, *Argo CD -- Declarative GitOps CD for Kubernetes*, [Argo CD Documentation](https://argo-cd.readthedocs.io/).
13. **Gene Kim, Jez Humble, Patrick Debois, John Willis (2016)**, *The DevOps Handbook*, IT Revolution Press.
14. **Nicole Forsgren, Jez Humble, Gene Kim (2018)**, *Accelerate: The Science of Lean Software and DevOps*, IT Revolution Press.