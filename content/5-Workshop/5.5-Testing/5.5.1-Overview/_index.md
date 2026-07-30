---
title: "Testing Overview & Pipeline Architecture"
date: 2026-07-06
weight: 1
chapter: false
pre: " <b> 5.5.1. </b> "
---

# 5.5.1 Testing Overview & Security Verification Strategy

---

The `devsecops-factory` system is architected under the **"Shift Left Security"** paradigm, bringing security verification as early as possible into the development pipeline — from the moment a developer commits code to the final deployment of container images onto **Amazon ECS Fargate**. To validate system effectiveness, the team designed the **tetris-app** (ReactJS + Nginx multi-stage build located under `app/`) as an intentional security testing target. The application incorporates seeded vulnerabilities across multiple layers, including source code, software dependencies, Dockerfile container manifests, and AWS cloud infrastructure configurations (Terraform, ECS Task Definitions). This design enables rigorous verification of each scanner's detection efficacy within the pipeline.

---

### 1. Pipeline Architecture & Defense-in-Depth

The `devsecops-factory` pipeline consists of 22 automated stages running on Jenkins, featuring 6 core Security Gates focused on automated analysis and verification. The architecture enforces the **"Defense in Depth"** model — each layer guarding against a distinct threat vector, ensuring zero blind spots across the software supply chain.

![Jenkins Build Overview 22 Stages](/images/5-Workshop/5.5-Testing/jenkins_build_overview.png)

*Figure 5.5.1a: Overview of the 22-stage automated Jenkins build pipeline, highlighting security checkpoints and deployment stages.*

#### Table 5.5.1: Summary of End-to-End Security Gate Scan Results

| Stage | Scan Type | Security Tool | Discovered Vulnerabilities | Severity | Gate Status |
|---|---|---|---|---|---|
| **Stage 4** | Secrets Scan | Gitleaks | 2 secrets (GitHub PAT + AWS Access Key) | HIGH | **FAIL (exit code 1)** |
| **Stage 5** | SCA Scan | Trivy FS | 4 CVEs (HIGH) | HIGH | **FAIL** |
| **Stage 6** | SAST Scan | SonarQube | 2 Vulnerabilities + 2 Hotspots | MEDIUM–HIGH | **FAIL** |
| **Stage 7** | IaC Scan | Checkov | 34 violations | MEDIUM–HIGH | **SOFT-FAIL** |
| **Stage 9** | Container Scan | Trivy Image | 40+ CVEs (15 CRITICAL) on `nginx:1.18.0` | CRITICAL | **FAIL** |
| **Stage 15** | DAST Scan | OWASP ZAP | Missing security headers | MEDIUM | **report-only** |

---

### 2. Structure of the `tetris-app` Test Target

```text
tetris-app/
├── Jenkinsfile              # CI/CD Pipeline definition (22 stages)
├── app/
│   ├── Dockerfile           # Multi-stage build (Node:16 + Nginx:1.18.0)
│   ├── package.json         # Dependency manifest (contains vulnerable pinned versions)
│   └── package-lock.json    # Lock file (SCA scan target)
├── src/                     # React source code (contains SAST flaws & hardcoded secrets)
├── infrastructure/
│   └── terraform/           # IaC Terraform manifests for ECS Fargate, ECR, S3, IAM
└── ci/
    └── stages/              # 6 Security Gate integration scripts for Jenkins
```

---

### Verification Screenshots: App Build & Docker Testing

![Build React App](/images/5-Workshop/5.3-Step-by-Step/app-01-run_build_app.jpg)
*Figure 5.5.1b: Compiling the React Tetris web application (`npm run build`).*

![Build Docker Image](/images/5-Workshop/5.3-Step-by-Step/app-02-build_docker.jpg)
*Figure 5.5.1c: Packaging the Multi-stage Docker container image for the application.*

![App Local UI](/images/5-Workshop/5.3-Step-by-Step/app-02-docker_app.jpg)
*Figure 5.5.1d: Tetris game web UI running locally via Docker container for preliminary verification.*

![AWS CloudWatch Logs Insights](/images/5-Workshop/5.3-Step-by-Step/aws_cloudwatch_insights.png)
*Figure 5.5.1e: Real-time container log analysis via AWS CloudWatch Logs Insights (`/ecs/devsecops-factory`).*

![Jenkins Approval Gate](/images/5-Workshop/5.3-Step-by-Step/jenkins_approval_gate.png)
*Figure 5.5.1f: Stage 19 — Manual Approval Gate on Jenkins UI — requiring engineer approval to proceed with Production deployment.*