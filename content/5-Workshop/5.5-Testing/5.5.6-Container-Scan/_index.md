---
title : "Stage 9 – Container Image Scan (Trivy)"
date : 2026-07-06 
weight : 6
chapter : false
pre : " <b> 5.5.6. </b> "
---

# 5.5.6 Stage 9 — Container Image Security Scan (Trivy Image)

---

### 1. Base Image Vulnerability Comparison Test

The project evaluates vulnerability posture between `nginx:1.18.0` (Debian Buster - intentionally seeded target) and `nginxinc/nginx-unprivileged:alpine` (Alpine Linux - hardened production base image).

#### Table 5.5.6: Container Scan Vulnerability Comparison Between Base Images

| Base Image | Operating System | C Library | Critical CVEs | High CVEs | Gate Result |
|---|---|---|---|---|---|
| `nginx:1.18.0` (Debian Buster) | Debian 10 | glibc | **15** | **25+** | **FAIL** |
| `nginxinc/nginx-unprivileged:alpine` | Alpine 3.23 | musl libc | **0** | **0** | **PASS** |

---

### 2. Scenario 1: Alpine Base Image (PASS)

When executing with `nginxinc/nginx-unprivileged:alpine`:

```text
Target: 585572506644.dkr.ecr.ap-southeast-1.amazonaws.com/devsecops/tetris
Type: alpine (alpine 3.23.4)
Total: 0 (CRITICAL: 0, HIGH: 0)
Status: Clean — Container image pushed to Amazon ECR successfully
```

---

### 3. Scenario 2: `nginx:1.18.0` Base Image (FAIL — 15 CRITICAL CVEs)

Trivy detects 40+ critical vulnerabilities across OS packages:

```text
nginx:1.18.0 (debian 10.13)
Total: 40 (CRITICAL: 15, HIGH: 25)

CVE-2023-5363  CRITICAL  openssl    Incorrect cipher key/IV lengths
CVE-2023-0215  HIGH      openssl    BIO_new_NDEF use after free
CVE-2022-2097  HIGH      openssl    AES OCB fails to encrypt some bytes
```

With `SECURITY_MODE=enforce` and `SECURITY_BLOCK_SEVERITIES=CRITICAL`, the pipeline **halts immediately at Stage 9** — preventing vulnerable container images from being **pushed to Amazon ECR**.

---

### 4. Actual Screenshots: Trivy Image Scan

![Trivy 15 CRITICAL CVEs](/images/5-Workshop/5.5-Testing/trivy_critical_cves.png)
*Figure 5.5.6a: Jenkins console output showing Trivy detecting 15 CRITICAL and 25 HIGH CVEs in Debian `nginx:1.18.0`.*

![Base Image Comparison](/images/5-Workshop/5.5-Testing/base_image_comparison.png)
*Figure 5.5.6b: Vulnerability comparison chart between Alpine (0 CVEs) and Debian `nginx:1.18.0` (40+ accumulated CVEs).*