---
title : "DevSecOps Security Testing"
date : 2026-07-06 
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

# 5.5 Comprehensive DevSecOps Security Process Testing

---

## Overview

To verify the accuracy and effectiveness of the `devsecops-factory` pipeline, the team designed `tetris-app` as an intentional vulnerability target. The test target features multi-layered security vulnerabilities across source code, third-party libraries, container images, and Kubernetes manifests.

This section documents detailed testing results across each security stage.

---

## Table of Contents

1. [5.5.1 Testing Strategy Overview](5.5.1-Overview/)
2. [5.5.2 Stage 3 – Hardcoded Secrets Scanning (Gitleaks)](5.5.2-Secrets-Scan/)
3. [5.5.3 Stage 4 – Dependency Vulnerability Audit (SCA Scan)](5.5.3-SCA-Scan/)
4. [5.5.4 Stage 5 – Static Code Analysis (SAST Scan)](5.5.4-SAST-Scan/)
5. [5.5.5 Stage 6 – Infrastructure as Code Scan (IaC Scan)](5.5.5-IaC-Scan/)
6. [5.5.6 Stage 8 – Container Image Scan (Trivy Image)](5.5.6-Container-Scan/)
7. [5.5.7 Comprehensive Pipeline Verification Summary](5.5.7-Summary/)
