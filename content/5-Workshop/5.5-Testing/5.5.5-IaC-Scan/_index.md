---
title : "Stage 6 – IaC Scan (Checkov)"
date : 2026-07-06 
weight : 5
chapter : false
pre : " <b> 5.5.5. </b> "
---

# 5.5.5 Stage 6 – Infrastructure as Code Scan (IaC Scan - Checkov)

---

Checkov audits Dockerfiles and Kubernetes manifests, recording **34 total failures**:
- **Dockerfile:** `CKV_DOCKER_3` (Missing non-root USER), `CKV_DOCKER_2` (Missing HEALTHCHECK).
- **Kubernetes:** `CKV_K8S_21` (Default namespace), `CKV_K8S_31` (Missing seccomp profile), `CKV_K8S_40` (Low UID), `CKV2_K8S_6` (Missing NetworkPolicy).

![IaC Scan Results Bar Chart](/images/5-Workshop/5.5-Testing/iac_bar_chart.png)

*Figure 5.5.5: IaC scan results comparison bar chart.*
