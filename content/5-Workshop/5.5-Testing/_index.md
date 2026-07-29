---
title : "DevSecOps Security Testing"
date : 2026-07-06 
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

# 5.5 Comprehensive DevSecOps Security Testing on AWS

---

## Overview

To verify the accuracy and effectiveness of the `devsecops-factory` pipeline, the team designed **[tetris-app](https://github.com/lamelihuynh/tetris-app.git)** (ReactJS + Nginx multi-stage build) as an intentional vulnerability target. The test target features multi-layered security vulnerabilities across source code, third-party libraries, container images, and AWS infrastructure configurations (Terraform, ECS Task Definitions).

This section documents detailed testing results across **6 Security Gates** in the 11-stage Jenkins CI/CD pipeline, centralized security report storage on **Amazon S3**, automated incident notification via **AWS Lambda**, and production-ready deployment on **Amazon ECS Fargate** monitored by **AWS CloudWatch Container Insights**.

---

## Table of Contents

1. [5.5.1 Testing Strategy Overview](5.5.1-Overview/)
2. [5.5.2 Stage 3 – Hardcoded Secrets Scanning (Gitleaks)](5.5.2-Secrets-Scan/)
3. [5.5.3 Stage 4 – Dependency Vulnerability Audit (SCA Scan)](5.5.3-SCA-Scan/)
4. [5.5.4 Stage 5 – Static Code Analysis (SAST Scan)](5.5.4-SAST-Scan/)
5. [5.5.5 Stage 6 – Infrastructure as Code Scan (IaC Scan - Checkov)](5.5.5-IaC-Scan/)
6. [5.5.6 Stage 8 – Container Image Scan on Amazon ECR (Trivy Image)](5.5.6-Container-Scan/)
7. [5.5.7 Comprehensive Pipeline Verification & AWS ECS Fargate Deployment](5.5.7-Summary/)

