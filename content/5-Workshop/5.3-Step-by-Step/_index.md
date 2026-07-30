---
title : "Step-by-Step Lab Guide"
date : 2026-07-06 
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

# 5.3 Step-by-Step Hands-on Lab Guide

---

## AWS Infrastructure Provisioning Overview (Terraform Infrastructure)

The entire AWS infrastructure supporting the **DevSecOps Factory** project is defined as Infrastructure as Code (IaC) within `infrastructure/terraform/main.tf` comprising 883 lines of HCL code. The cloud stack provisions 6 core automated resource modules:

1. **AWS-01: Budget Guardrails & IAM Scoping:** Provisions AWS Budget Alerts (notifying at 50%/80%/100% thresholds), ECS Execution Roles, ECS Task Roles, and Jenkins CI IAM Policies enforcing Least Privilege.
2. **AWS-02: Amazon ECR:** Provisions private container repository `devsecops/tetris` with SSE-AES256 encryption and automated Scan-on-Push enabled (`scan_on_push = true`).
3. **AWS-03: Amazon ECS Fargate:** Provisions ECS Cluster `devsecops-factory-cluster` with CloudWatch Container Insights enabled, running 2 services: `tetris-staging` (`FARGATE_SPOT` yielding 70% cost savings) and `tetris-production` (`FARGATE` high availability).
4. **AWS-04: Amazon S3:** Provisions centralized S3 bucket `devsecops-reports-*` storing outputs from 6 Security Gates with SSE-S3 encryption, Versioning, S3 Public Access Blocked, and 30-day Lifecycle policies.
5. **AWS-05: AWS Lambda Aggregator:** Provisions Lambda function `devsecops-factory-securityhub-importer` (Python 3.12) automatically parsing ASFF JSON reports from S3 and importing findings into **AWS Security Hub**.
6. **Networking (VPC & ALB):** Configures dedicated VPC (`10.0.0.0/16`), 2 Public Subnets (`10.0.1.0/24` & `10.0.2.0/24`), along with 2 Application Load Balancers (ALB Staging & ALB Production).

---

## Hands-on Lab Step Directory

1. [5.3.1 Provision Amazon ECR Repository & S3 Report Bucket](5.3.1-ECR-S3/)
2. [5.3.2 Configure IAM Roles & Security Policies](5.3.2-IAM-Policies/)
3. [5.3.3 Configure 6 Security Gates in Jenkinsfile](5.3.3-Security-Gates/)
4. [5.3.4 Provision AWS Lambda Security Report Aggregator](5.3.4-Lambda-Aggregator/)
5. [5.3.5 Deploy Application to Amazon ECS Fargate](5.3.5-ECS-Fargate/)
6. [5.3.6 Post-Deployment Verification & Testing](5.3.6-Verification/)
