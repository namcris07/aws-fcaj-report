---
title : "Problem Statement & Goals"
date : 2026-07-06 
weight : 1
chapter : false
pre : " <b> 5.1.1. </b> "
---

# 5.1.1 Problem Statement & Project Objectives

---

### 1. Context & Background
In modern software development, Continuous Integration and Continuous Deployment (CI/CD) pipelines execute at rapid speed. However, security checks are often deferred to late-stage manual audits prior to production release. This causes delayed vulnerability discovery, ballooning remediation costs, and elevated risk of production data breaches.

### 2. Target Audience
- Software Developers, Security Engineers, and DevOps/SRE teams.
- Small and Medium Enterprises (SMEs) seeking an automated, cost-efficient cloud DevSecOps pipeline on AWS.

### 3. Key Challenges Solved
- **Manual & Inconsistent Deployments:** Lack of automated container build and environment synchronization.
- **Credential Leakage:** Inadvertent hardcoding of AWS Access Keys or API credentials into Git repositories.
- **Vulnerable Dependencies & Base Images:** Integration of open-source packages and Docker base images with unpatched CVEs.
- **High Cloud Infrastructure Costs:** Maintenance of static Kubernetes control planes (EKS) incurring fixed costs (~$72/month).

---

### 4. Deliverables & Outputs
- **12-Stage CI/CD Pipeline:** Automated security auditing across 6 Security Gates (Gitleaks, Trivy SCA, SonarQube, Checkov, Trivy Image, OWASP ZAP).
- **Centralized Security Report Aggregation:** JSON/HTML report archiving on **Amazon S3** and automated parsing via **AWS Lambda Aggregator**.
- **Serverless Container Runtime:** Automated Staging deployment on **Amazon ECS Fargate** with Manual Approval gates for Production.
- **Centralized Observability:** Operational telemetry via **Amazon CloudWatch Container Insights** and AWS Budget Alarms.

---

### 5. Success Evaluation Criteria
- 100% automated security scan coverage before application deployment.
- Instant pipeline failure (FAIL-FAST) upon detection of hardcoded secrets or Critical/High CVEs.
- Total monthly AWS infrastructure cost kept under **$20.50 USD/month** (> 85% savings over EKS).
