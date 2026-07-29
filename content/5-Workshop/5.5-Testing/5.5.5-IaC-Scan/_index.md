---
title : "Stage 6 – IaC Scan (Checkov)"
date : 2026-07-06 
weight : 5
chapter : false
pre : " <b> 5.5.5. </b> "
---

# 5.5.5 Stage 6 – Infrastructure as Code Scan (IaC Scan - Checkov)

---

Checkov audits Terraform scripts (`infrastructure/terraform/*.tf`), Dockerfiles (`app/Dockerfile`), and Amazon ECS Task Definitions (`infrastructure/task-definition.json`), reporting findings across AWS infrastructure:
- **Dockerfile:** `CKV_DOCKER_3` (Missing non-root USER), `CKV_DOCKER_2` (Missing HEALTHCHECK).
- **AWS Infrastructure & ECS:** `CKV_AWS_24` (Overly permissive Security Group Ingress `0.0.0.0/0`), `CKV_AWS_18` (S3 Report Bucket missing Access Logging), `CKV_AWS_130` (ECS Task missing `readonlyRootFilesystem`), `CKV_AWS_55` (ECR Repository missing KMS CMK encryption).

![IaC Scan Results Bar Chart](/images/5-Workshop/5.5-Testing/iac_bar_chart.png)

*Figure 5.5.5: IaC scan results comparison bar chart for AWS Infrastructure and Dockerfile.*

