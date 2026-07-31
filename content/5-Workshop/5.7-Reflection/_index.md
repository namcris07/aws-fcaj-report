---
title : "Reflection & Lessons Learned"
date : 2026-07-06 
weight : 7
chapter : false
pre : " <b> 5.7. </b> "
---

# 5.7 Reflection & Lessons Learned

---

## Overview

This section documents honest reflections on the technical challenges encountered during the 9-week **DevSecOps Factory** project, the approaches used to resolve them, and the long-term lessons carried forward.

---

## 1. Key Challenges Encountered

### 1.1 ECS Fargate Health Check Failures
**Problem:** The initial ECS Task Definition configured the React app container to run as `root` on port 80 (standard Nginx). However, the team adopted `nginxinc/nginx-unprivileged` (non-root user 101) which binds to port **8080** — causing the ALB Health Check to report `unhealthy` and the ECS service to enter a continuous restart loop.

**Resolution:**
- Updated the ALB Target Group health check path from `/` on port `80` to port `8080`.
- Added an `init` container (`volume-permissions`) running as root to pre-grant `/var/cache/nginx` and `/tmp` write permissions before the main `tetris` container starts.
- Enforced `readonlyRootFilesystem: true` on the main container for additional security hardening.

**Lesson Learned:** Always align the **container port**, **Task Definition `containerPort`**, and **ALB Target Group port** before first deployment. Non-root base images require explicit volume permission initialization patterns.

---

### 1.2 AWS Lambda ASFF Schema Validation Errors
**Problem:** The initial Lambda Aggregator function produced `InvalidInput` errors when calling `BatchImportFindings` because the ASFF (Amazon Security Finding Format) JSON schema required several mandatory fields (`SchemaVersion`, `ProductArn`, `GeneratorId`, `Types`, `CreatedAt`, `UpdatedAt`, `Severity`, `Title`, `Description`, `Resources`) that were missing from the raw Trivy/Gitleaks JSON output.

**Resolution:**
- Built a dedicated `normalize_report.py` script that maps raw scanner output fields to the required ASFF schema structure.
- Added a pipeline stage (`Normalize Reports & ASFF Conversion`) before the S3 upload step to guarantee schema-compliant `securityhub-asff.json` files.
- Validated the schema locally using `aws securityhub batch-import-findings --dry-run` before enabling the live Lambda trigger.

**Lesson Learned:** Never assume raw scanner output matches the AWS Security Hub ASFF schema. Build a normalization layer early in the pipeline design phase.

---

### 1.3 Jenkins Pipeline Credential Leakage Risk
**Problem:** During early development, AWS Access Keys were temporarily stored in the `.env` file and volume-mounted directly into the Jenkins container — creating a risk that credentials would accidentally be committed to Git or exposed in Docker inspect output.

**Resolution:**
- Migrated all credentials to the **Jenkins Credentials Store** (Secret Text type).
- Injected credentials via `withCredentials([string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID')])` blocks within the Jenkinsfile — ensuring credentials never appear in build logs or Docker environment variables.
- Added `.env` to `.gitignore` immediately and ran `git filter-branch` to purge any accidental credential commits from Git history.
- Configured Gitleaks in Stage 3 to catch any future accidental credential commits.

**Lesson Learned:** Credential hygiene must be established on Day 1. Treat Jenkins Credentials Store as mandatory, not optional. Gitleaks as a pipeline gate prevents leaks from ever reaching remote repositories.

---

### 1.4 FARGATE_SPOT Capacity Interruption During Demo
**Problem:** During Week 6 demo preparation, the Staging ECS service running on `FARGATE_SPOT` was interrupted mid-demo as Spot capacity was reclaimed by AWS. The application became unreachable for approximately 3 minutes while ECS launched a replacement task.

**Resolution:**
- Implemented a **Capacity Provider Strategy** on the Staging service: primary `FARGATE_SPOT` (weight 1) with `FARGATE` (weight 0) as fallback — ensuring AWS automatically switches to standard Fargate when Spot is unavailable.
- Added `make demo-scale` script to quickly scale Production (standard FARGATE) up for critical demo windows.
- Prepared static screenshot backup slides of every pipeline stage for offline fallback demonstrations.

**Lesson Learned:** `FARGATE_SPOT` is ideal for non-critical workloads and cost savings but should never be the sole capacity provider for demo or production environments without a fallback strategy.

---

### 1.5 Checkov IaC Scan Blocking the Pipeline Prematurely
**Problem:** With `--exit-code 1` enforced, Checkov reported 34 IaC failures across Terraform, Dockerfile, and Kubernetes manifests — blocking the pipeline entirely during Week 4, preventing any cloud deployment testing.

**Resolution:**
- Adopted a phased gate enforcement strategy:
  - **Phase 1 (Weeks 1–5):** `--soft-fail` mode — reports failures but does not block the pipeline.
  - **Phase 2 (Weeks 6–9):** Selectively enable `--exit-code 1` only for **HIGH** severity checks; suppress accepted LOW/MEDIUM findings with a `checkov.yaml` skip configuration.
- Documented all accepted risk suppressions in `checkov.yaml` with justification comments.

**Lesson Learned:** Enforce security gates gradually. Start with `soft-fail` reporting to build a baseline, then progressively tighten gate criteria as the codebase matures.

---

## 2. What Worked Well

| Area | Outcome |
|---|---|
| **Terraform IaC** | 50 AWS resources provisioned in a single `terraform apply` — zero manual console clicks required |
| **Makefile Automation** | `make bootstrap`, `make status`, `make demo-reset` reduced onboarding time for new team members from hours to minutes |
| **Local-first Strategy (k3d)** | Enabled full pipeline testing offline, eliminating cloud costs during the 5-week development phase |
| **FARGATE_SPOT for Staging** | Achieved ~70% compute cost reduction on staging workloads |
| **Lambda Aggregator** | Operated entirely within AWS Free Tier — zero Lambda cost across 9 weeks |
| **6 Security Gates** | Successfully detected all intentionally seeded vulnerabilities: 2 secrets, 4 CVEs, 40+ container CVEs, 2 SAST findings, 34 IaC misconfigurations |

---

## 3. Future Development Directions

- **Enforce DAST in Blocking Mode:** Upgrade OWASP ZAP from `report-only` to `--exit-code 1` for FAIL-grade alerts (e.g., missing `X-Frame-Options`, `Content-Security-Policy` headers).
- **Add SonarQube Quality Gate:** Integrate SonarQube Quality Gate API polling in the pipeline to block builds that fall below defined code coverage or issue thresholds.
- **Multi-Region Deployment:** Extend Terraform to provision a secondary ECS Fargate environment in `us-east-1` for disaster recovery validation.
- **Automated Cost Anomaly Detection:** Replace manual Budget Alerts with **AWS Cost Anomaly Detection** for ML-driven spending spike notifications.
- **Security Hub Custom Insights:** Build custom Security Hub Insights dashboards filtering CRITICAL/HIGH findings by pipeline build ID for faster triage.
- **GitHub Actions Migration:** Evaluate migrating from Jenkins to GitHub Actions native runners for cloud-native CI/CD to further reduce infrastructure overhead.
