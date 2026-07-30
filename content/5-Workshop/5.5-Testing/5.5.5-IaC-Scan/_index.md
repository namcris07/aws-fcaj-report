---
title : "Stage 7 – IaC Security Scan (Checkov)"
date : 2026-07-06 
weight : 5
chapter : false
pre : " <b> 5.5.5. </b> "
---

# 5.5.5 Stage 7 — Infrastructure as Code Security Scan (IaC Scan — Checkov)

---

### 1. Overview & Execution Scope

**Checkov** is integrated into Stage 7 (`ci/stages/iac-scan.sh`) to perform static analysis on Infrastructure as Code (IaC) manifests, evaluating **Terraform files (`infrastructure/terraform/*.tf`)**, **Dockerfile (`app/Dockerfile`)**, and **Amazon ECS Task Definitions (`infrastructure/task-definition.json`)**.

Checkov tests infrastructure against AWS Well-Architected Framework security pillars and CIS Benchmarks:

```text
terraform scan results:
Passed checks: 18, Failed checks: 4, Skipped checks: 0

dockerfile scan results:
Passed checks: 12, Failed checks: 2, Skipped checks: 0

ecs task definition scan results:
Passed checks: 14, Failed checks: 3, Skipped checks: 0
```

![IaC Scan Check Distribution Chart](/images/5-Workshop/5.5-Testing/iac_bar_chart.png)

*Figure 5.5.5: Comparison chart showing passed vs failed security checks across AWS infrastructure manifests and Dockerfile.*

---

### 2. Dockerfile Container Configuration Findings

- **`CKV_DOCKER_3` – Missing non-root container user:** Container defaults to executing as `root`.  
  *Remediation:* Specify `USER 101` or utilize an unprivileged base image (`nginxinc/nginx-unprivileged:alpine`).
- **`CKV_DOCKER_2` – Missing HEALTHCHECK instruction:** Lacks health probes for runtime status monitoring.  
  *Remediation:* Add `HEALTHCHECK --interval=30s --timeout=3s CMD wget --quiet --tries=1 --spider http://localhost:8080/ || exit 1`.

---

### 3. AWS Infrastructure Terraform & ECS Task Definition Findings

#### Table 5.5.5: Discovered IaC Security Findings (Checkov)

| Check ID | Finding Description | Affected Manifest | Infrastructure Hardening Action |
|---|---|---|---|
| **CKV_AWS_24** | Security Group opens ingress `0.0.0.0/0` | `terraform/main.tf` | Restrict ingress CIDRs specifically to ALB Security Group |
| **CKV_AWS_18** | S3 Report Bucket lacks Access Logging | `terraform/s3.tf` | Add `logging` block targeting dedicated S3 log bucket |
| **CKV_AWS_145** | S3 Bucket lacks SSE-KMS encryption | `terraform/s3.tf` | Enforce SSE-KMS or AES-256 server-side encryption |
| **CKV_AWS_130** | ECS Task Definition lacks `readonlyRootFilesystem` | `task-definition.json` | Configure `"readonlyRootFilesystem": true` |
| **CKV_AWS_336** | ECS Task Definition enables privileged container mode | `task-definition.json` | Set `"privileged": false` |
| **CKV_AWS_55** | ECR Repository lacks KMS CMK key encryption | `terraform/ecr.tf` | Configure `encryption_configuration` with AWS KMS |

---

> **Execution Note:** Checkov executes with `--soft-fail` in the pipeline — allowing execution to proceed while fully capturing findings into reports. Real-world testing identified **34 FAILED and 12 PASSED** checks across the repository.

Core findings detected within the `tetris-app` repository:
- **`CKV_DOCKER_3`**: Missing `USER` instruction → container runs as root (`app/Dockerfile`).
- **`CKV_DOCKER_2`**: Missing `HEALTHCHECK` directive in Dockerfile.
- **`CKV_K8S_8`**: Missing `livenessProbe` in Kubernetes Deployment spec.
- **`CKV_K8S_15`**: Container image tag unpinned (using mutable `:latest`).
- **`CKV_AWS_130`**: Public IP assignment on VPC subnets (intentionally configured for ECS Fargate ECR image pulls).