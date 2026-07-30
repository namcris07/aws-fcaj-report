---
title : "Context & Goals"
date : 2026-07-06 
weight : 1
chapter : false
pre : " <b> 5.1.1. </b> "
---

# 5.1.1 Context, Problem Statement & Project Objectives

---

### 1. Project Background
In modern software engineering organizations, continuous integration and deployment (CI/CD) pipelines operate at unprecedented velocity. However, security verification is frequently deprioritized or performed manually near the end of release cycles (Late-stage Security Gate). This causes delayed vulnerability discovery, astronomical remediation costs, and elevated risks of production data breaches.

### 2. Target Audience & Stakeholders
- Software Engineers (Developers), Cloud Security Engineers, and DevOps/SRE Operations Teams.
- Small and Medium Enterprises (SMEs) seeking an automated, cost-optimized DevSecOps CI/CD workflow on AWS cloud infrastructure.

### 3. Problem Statement
- **Manual Deployments & Inconsistency:** Lack of automated container packaging and environment synchronization.
- **Hardcoded Credential Leaks:** Engineers accidentally committing AWS Access Keys, API Tokens, or database credentials into Git repositories.
- **Vulnerable Dependencies & Base Images:** Ingesting unvetted open-source packages and bloated container base images harboring known CVEs.
- **Excessive Infrastructure Baseline Costs:** Maintaining static Kubernetes control planes (EKS) incurring heavy fixed costs (~$72/month for control plane alone).

---

### 4. Project Objectives & Target Deliverables
- **22-Stage CI/CD Pipeline:** Automated verification across 6 Security Gates (Gitleaks, Trivy SCA, SonarQube, Checkov, Trivy Image, OWASP ZAP), converting findings to ASFF for direct ingestion into AWS Security Hub.
- **Centralized Security Report Aggregation:** Archiving JSON/HTML scan artifacts on **Amazon S3** and automated parsing via an **AWS Lambda Aggregator**.
- **Serverless Container Deployment:** Automated deployment to **Amazon ECS Fargate** for Staging and interactive Manual Approval Gate enforcement for Production.
- **Centralized Observability:** Real-time log and metrics visualization via **Amazon CloudWatch Container Insights** paired with cost guardrail alerts.

---

### 5. Key Performance Indicators (KPIs) & Success Criteria
- 100% of source code commits and container images automatically verified prior to cloud deployment.
- Enforce FAIL-FAST pipeline aborts upon discovering hardcoded secrets or Critical/High CVEs.
- Optimize monthly AWS cloud infrastructure baseline costs to **< $20.50 USD/month** (> 85% cost savings compared to EKS).