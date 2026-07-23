---
title : "Stage 8 – Quét Image Container"
date : 2026-07-06 
weight : 6
chapter : false
pre : " <b> 5.5.6. </b> "
---

# 5.5.6 Stage 8 – Quét Image Container (Container Scan)

---

### 1. Kịch bản kiểm thử so sánh Base Image

#### Bảng 5.5.6: So sánh kết quả Container Scan giữa hai Base Images

| Base Image | Hệ điều hành nền | C Library | Lỗ hổng phát hiện (HIGH/CRITICAL) | Kết quả |
|---|---|---|---|---|
| `nginxinc/nginx-unprivileged:alpine` | Alpine Linux | musl libc | **0** | **PASS** |
| `nginx:1.18.0` | Debian Buster | glibc (GNU) | **Hàng chục CVEs** | **FAIL** |

---

### 2. Kịch bản 1: Base Image Alpine (PASS)

Khi sử dụng `nginxinc/nginx-unprivileged:alpine`:

```text
Target: localhost:5001/devsecops/tetris:build-30
Type: alpine (alpine 3.23.4)
Vulnerabilities: 0
Status: Clean (no security findings detected)
```

---

### 3. Kịch bản 2: Base Image Debian `nginx:1.18.0` (FAIL)

Trivy phát hiện nhiều CVEs nghiêm trọng trong `nginx:1.18.0` (glibc CVE-2021-35942, libcurl4 CVE-2022-32221, openssl CVE-2023-0286).

![So sánh lỗ hổng Container Scan giữa hai base image](/images/5-Workshop/5.5-Testing/base_image_comparison.png)

*Hình 5.5.6: Alpine có 0 lỗ hổng HIGH/CRITICAL, Debian nginx:1.18.0 có hàng chục CVE tích lũy.*
