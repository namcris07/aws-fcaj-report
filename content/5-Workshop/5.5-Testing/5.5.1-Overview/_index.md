---
title : "Testing Strategy Overview"
date : 2026-07-06 
weight : 1
chapter : false
pre : " <b> 5.5.1. </b> "
---

# 5.5.1 Testing Strategy Overview

---

The `devsecops-factory` system is designed following the **"Shift Left Security"** principle, placing security validation as early as possible in the CI/CD pipeline. To verify the pipeline, the team designed `tetris-app` as an intentional vulnerability target.

![DevSecOps 11-stage Pipeline Flow](/images/5-Workshop/5.5-Testing/pipeline_stages.png)

*Figure 5.5.1: Full DevSecOps 11-stage pipeline architecture running on Jenkins.*

#### Table 5.5.1: Security Pipeline Testing Summary

| Stage | Scan Type | Tool | Findings | Severity | Result |
|---|---|---|---|---|---|
| **Stage 3** | Secrets Scan | Gitleaks | 2 secrets | MEDIUM – HIGH | **FAIL (exit code 1)** |
| **Stage 4** | SCA Scan | Trivy FS | 4 CVEs (HIGH) | HIGH | **FAIL** |
| **Stage 5** | SAST Scan | SonarQube | 4+ code issues | MEDIUM – HIGH | **FAIL** |
| **Stage 6** | IaC Scan | Checkov | 34 failures | MEDIUM – HIGH | **FAIL (Soft-fail)** |
| **Stage 8** | Container Image Scan | Trivy Image | 0 CVEs (Alpine) / 40+ CVEs (Debian) | CRITICAL – HIGH | **PASS (Alpine) / FAIL (Debian)** |
