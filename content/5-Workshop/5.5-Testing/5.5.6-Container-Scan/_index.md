---
title : "Stage 8 – Container Scan (Trivy Image)"
date : 2026-07-06 
weight : 6
chapter : false
pre : " <b> 5.5.6. </b> "
---

# 5.5.6 Stage 8 – Container Image Scan (Trivy Image)

---

### Base Image Comparison:
- `nginxinc/nginx-unprivileged:alpine`: **0 HIGH/CRITICAL CVEs (PASS)**.
- `nginx:1.18.0` (Debian Buster): **40+ HIGH/CRITICAL CVEs (FAIL)** (including `glibc` CVE-2021-35942, `libcurl4` CVE-2022-32221, `openssl` CVE-2023-0286).

![Base Image Vulnerability Comparison](/images/5-Workshop/5.5-Testing/base_image_comparison.png)

*Figure 5.5.6: Container image CVE comparison between Alpine and Debian.*
