---
title : "Stage 4 – Quét thành phần phụ thuộc (SCA)"
date : 2026-07-06 
weight : 3
chapter : false
pre : " <b> 5.5.3. </b> "
---

# 5.5.3 Stage 4 – Quét thành phần phụ thuộc (SCA Scan)

---

### 1. Công cụ và cơ chế hoạt động

Trivy được sử dụng ở chế độ `fs` (filesystem scan), phân tích file `package-lock.json` để xây dựng đồ thị phụ thuộc đầy đủ (dependency graph), sau đó tra cứu từng package trong cơ sở dữ liệu CVE.

```bash
trivy fs --scanners vuln --severity HIGH,CRITICAL --quiet "${SCAN_DIR}"
```

---

### 2. Kịch bản tiêm lỗ hổng

```text
styled-components@5.0.1 ──► css-select@2.x ──────► nth-check@1.0.2 [CVE-2021-3803 ReDoS]
react-spring@8.0.27     ──► webpack (react-scripts) ──► serialize-js@4.0.0 [GHSA-5c6j RCE]
lodash@4.17.21          ──► underscore@1.13.6  ──► underscore@1.13.6 [CVE-2026-27601 DoS]
```

---

### 3. Kết quả phát hiện

Trivy phát hiện **4 lỗ hổng mức HIGH** trong `app/package-lock.json`:

#### Bảng 5.5.3: Chi tiết các lỗ hổng phát hiện bởi SCA Scan (Trivy)

| Thư viện | CVE / Vulnerability ID | Severity | Phiên bản đang dùng | Phiên bản đã vá |
|---|---|---|---|---|
| **nth-check** | CVE-2021-3803 | HIGH | 1.0.2 | 2.0.1 |
| **serialize-javascript** | GHSA-5c6j-r48x-rmvq | HIGH | 4.0.0 / 6.0.2 | 7.0.3 |
| **underscore** | CVE-2026-27601 | HIGH | 1.13.6 | 1.13.8 |

![Phân bố lỗ hổng SCA theo thư viện](/images/5-Workshop/5.5-Testing/sca_pie_chart.png)

*Hình 5.5.3: Phân bố lỗ hổng SCA theo thư viện (tổng 4 lỗi HIGH).*

---

### 4. Cách khắc phục

Thực hiện nâng cấp tự động qua lệnh `npm audit fix`.
