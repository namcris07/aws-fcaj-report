---
title : "Stage 9 – Quét Image Container (Trivy)"
date : 2026-07-06 
weight : 6
chapter : false
pre : " <b> 5.5.6. </b> "
---

# 5.5.6 Stage 9 — Quét Image Container (Container Scan — Trivy Image)

---

### 1. Kịch bản kiểm thử so sánh Base Image

Dự án sử dụng `nginx:1.18.0` (Debian) cho ứng dụng kiểm thử (cố ý) để kích hoạt FAIL, và `nginxinc/nginx-unprivileged:alpine` cho môi trường production thực tế.

#### Bảng 5.5.6: So sánh kết quả Container Scan giữa hai Base Images

| Base Image | OS nền | C Library | CVE Critical | CVE High | Kết quả |
|---|---|---|---|---|---|
| `nginx:1.18.0` (Debian Buster) | Debian 10 | glibc | **15** | **25+** | **FAIL** |
| `nginxinc/nginx-unprivileged:alpine` | Alpine 3.23 | musl libc | **0** | **0** | **PASS** |

---

### 2. Kịch bản 1: Base Image Alpine (PASS)

Khi sử dụng `nginxinc/nginx-unprivileged:alpine`:

```text
Target: 585572506644.dkr.ecr.ap-southeast-1.amazonaws.com/devsecops/tetris
Type: alpine (alpine 3.23.4)
Total: 0 (CRITICAL: 0, HIGH: 0)
Status: Clean — image được push lên ECR thành công
```

---

### 3. Kịch bản 2: Base Image `nginx:1.18.0` (FAIL — 15 CRITICAL CVEs)

Trivy phát hiện 40+ CVEs nghiêm trọng:

```text
nginx:1.18.0 (debian 10.13)
Total: 40 (CRITICAL: 15, HIGH: 25)

CVE-2023-5363  CRITICAL  openssl    Incorrect cipher key/IV lengths
CVE-2023-0215  HIGH      openssl    BIO_new_NDEF use after free
CVE-2022-2097  HIGH      openssl    AES OCB fails to encrypt some bytes
```

Với `SECURITY_MODE=enforce` và `SECURITY_BLOCK_SEVERITIES=CRITICAL`, pipeline **dừng ngay tại Stage 9** — image **không được push lên ECR**, bảo vệ môi trường Production khỏi các lỗ hổng hệ điều hành nghiêm trọng.

---

### 4. Ảnh minh chứng thực tế: Trivy Image Scan

![Trivy 15 CRITICAL CVEs](/images/5-Workshop/5.5-Testing/trivy_critical_cves.png)
*Hình 5.5.6a: Jenkins console log hiển thị chi tiết báo cáo Trivy phát hiện 15 CRITICAL CVEs và 25 HIGH CVEs trong base image nginx:1.18.0 (Debian Buster).*

![So sánh lỗ hổng Container Scan giữa hai base image](/images/5-Workshop/5.5-Testing/base_image_comparison.png)
*Hình 5.5.6b: Biểu đồ so sánh số lượng lỗ hổng: Alpine có 0 lỗ hổng HIGH/CRITICAL, Debian nginx:1.18.0 có 40+ CVEs tích lũy.*