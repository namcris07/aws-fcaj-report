---
title : "Kiểm thử quy trình DevSecOps"
date : 2026-07-06 
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

# 5.5 Kiểm thử toàn diện quy trình bảo mật DevSecOps

---

## Tổng quan

Để xác minh tính đúng đắn và hiệu quả của hệ thống `devsecops-factory`, nhóm đã thiết kế ứng dụng `tetris-app` làm mục tiêu kiểm thử có chủ ý. Ứng dụng được cài đặt sẵn các lỗ hổng ở nhiều tầng khác nhau, bao gồm mã nguồn, thư viện phụ thuộc, cấu hình container và cấu hình hạ tầng Kubernetes.

Mục này phân tích chi tiết kết quả kiểm thử tại từng stage bảo mật trong pipeline 11 stage.

---

## Danh mục các mục kiểm thử

1. [5.5.1 Tổng quan về chiến lược kiểm thử](5.5.1-Overview/)
2. [5.5.2 Stage 3 – Quét key bí mật mã hoá cứng (Secrets Scan)](5.5.2-Secrets-Scan/)
3. [5.5.3 Stage 4 – Quét thành phần phụ thuộc (SCA Scan)](5.5.3-SCA-Scan/)
4. [5.5.4 Stage 5 – Phân tích tĩnh mã nguồn (SAST Scan)](5.5.4-SAST-Scan/)
5. [5.5.5 Stage 6 – Quét cấu hình Hạ tầng (IaC Scan)](5.5.5-IaC-Scan/)
6. [5.5.6 Stage 8 – Quét Image Container (Container Scan)](5.5.6-Container-Scan/)
7. [5.5.7 Kết quả tổng hợp](5.5.7-Summary/)
