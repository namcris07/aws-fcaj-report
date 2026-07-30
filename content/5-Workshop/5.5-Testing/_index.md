---
title : "DevSecOps Security Testing"
date : 2026-07-06 
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

# 5.5 End-to-End DevSecOps Security Testing & Verification on AWS

---

## Overview

To validate the security posture and detection capabilities of the `devsecops-factory` system, the team engineered the **tetris-app** (React + Nginx multi-stage build under `app/`) as an **intentional security test target**. The application incorporates seeded vulnerabilities across multiple layers:
- **Source Code:** Hardcoded credentials (`app/src/config.js`), XSS vulnerability (`dangerouslySetInnerHTML`)
- **Software Dependencies:** Legacy npm packages carrying known CVEs (`react-scripts 4.0.3`, `nth-check 1.0.2`, `serialize-javascript 2.1.1`)
- **Container Base Image:** Base image `nginx:1.18.0` (Debian) harboring 40+ CVEs instead of `nginxinc/nginx-unprivileged:alpine`
- **Infrastructure Code:** Dockerfile and Kubernetes manifests lacking hardening (missing `USER` instruction, `HEALTHCHECK`)

This section analyzes detailed verification findings across **6 Security Gates** within the **22-stage** Jenkins pipeline, centralized report archival on **Amazon S3**, ASFF normalization and automated ingestion via **AWS Lambda** into **AWS Security Hub**, and secure application deployment to **Amazon ECS Fargate** integrated with **AWS CloudWatch Container Insights**.

---

## Summary Table of End-to-End Verification Results

| Stage | Security Gate | Tool | Verification Findings | Severity | Gate Status |
|---|---|---|---|---|---|
| **Stage 4** | Secrets Scan | Gitleaks | 2 secrets (GitHub PAT + AWS Access Key) | HIGH | **FAIL** |
| **Stage 5** | SCA Scan | Trivy FS | 4 CVEs (`nth-check`, `serialize-javascript`) | HIGH | **FAIL** |
| **Stage 6** | SAST Scan | SonarQube | 2 Vulnerabilities + 2 Security Hotspots | MEDIUM–HIGH | **FAIL** |
| **Stage 7** | IaC Scan | Checkov | 34 failures (Dockerfile + K8s + Terraform) | MEDIUM–HIGH | **SOFT-FAIL** |
| **Stage 9** | Container Scan | Trivy Image | 40+ CVEs (15 CRITICAL, 25 HIGH) on `nginx:1.18.0` | CRITICAL | **FAIL** |
| **Stage 15** | DAST | OWASP ZAP | Missing security headers (0 FAIL alerts) | MEDIUM | **report-only** |

**Overall Verification Summary:**
- ✓ Accurately detected 5 out of 6 vulnerability categories in `enforce` mode
- ✓ Pipeline aborted at exact target stage (Stage 9 — CRITICAL CVEs)
- ✓ Security reports archived centrally on Amazon S3
- ✓ AWS Security Hub ingested ASFF findings via Lambda within 60 seconds

---

## Verification Test Sections

1. [5.5.1 Testing Overview & Pipeline Architecture](5.5.1-Overview/)
2. [5.5.2 Stage 4 — Hardcoded Secrets Scan (Gitleaks)](5.5.2-Secrets-Scan/)
3. [5.5.3 Stage 5 — Software Composition Analysis (Trivy FS)](5.5.3-SCA-Scan/)
4. [5.5.4 Stage 6 — Static Application Security Testing (SonarQube)](5.5.4-SAST-Scan/)
5. [5.5.5 Stage 7 — Infrastructure as Code Security Scan (Checkov)](5.5.5-IaC-Scan/)
6. [5.5.6 Stage 9 — Container Image Security Scan (Trivy Image)](5.5.6-Container-Scan/)
7. [5.5.7 Summary Findings & ECS Fargate Deployment](5.5.7-Summary/)
