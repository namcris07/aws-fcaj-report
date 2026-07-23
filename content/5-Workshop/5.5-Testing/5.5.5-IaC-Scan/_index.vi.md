---
title : "Stage 6 – Quét Hạ tầng (IaC Scan)"
date : 2026-07-06 
weight : 5
chapter : false
pre : " <b> 5.5.5. </b> "
---

# 5.5.5 Stage 6 – Quét cấu hình Hạ tầng (IaC Scan)

---

### 1. Kết quả tổng quan

Checkov phát hiện tổng cộng **34 failures**:

```text
kubernetes scan results:
Passed checks: 85, Failed checks: 10, Skipped checks: 0

dockerfile scan results:
Passed checks: 40, Failed checks: 2, Skipped checks: 0

kustomize scan results:
Passed checks: 260, Failed checks: 22, Skipped checks: 0
```

![Biểu đồ so sánh số lượng checks passed/failed theo loại file hạ tầng](/images/5-Workshop/5.5-Testing/iac_bar_chart.png)

*Hình 5.5.5: Biểu đồ so sánh số lượng checks passed/failed theo loại file hạ tầng.*

---

### 2. Nhóm lỗi Dockerfile

- **CKV_DOCKER_3 – Không tạo user riêng cho container:** Thêm `USER 101` trước câu lệnh `CMD`.
- **CKV_DOCKER_2 – Thiếu HEALTHCHECK:** Thêm hướng dẫn `HEALTHCHECK`.

---

### 3. Nhóm lỗi Kubernetes

#### Bảng 5.5.5: Các lỗi IaC Scan phát hiện trong cấu hình Kubernetes

| Check ID | Mô tả lỗi | File | Giải pháp khắc phục |
|---|---|---|---|
| **CKV_K8S_21** | Default namespace | `service.yaml`, `deployment.yaml` | Thêm `namespace: tetris-prod` |
| **CKV_K8S_31** | Thiếu seccomp profile | `deployment.yaml` | Thêm `seccompProfile: RuntimeDefault` |
| **CKV_K8S_14** | Image tag dùng `latest` | `deployment.yaml` | Dùng tag cố định theo Git Commit SHA |
| **CKV_K8S_40** | Container chạy UID thấp (< 10000) | `deployment.yaml` | Đặt `runAsUser: 10001` |
| **CKV_K8S_38** | SA Token tự động mount | `deployment.yaml` | Đặt `automountServiceAccountToken: false` |
| **CKV2_K8S_6** | Pod thiếu NetworkPolicy | `deployment.yaml` | Định nghĩa NetworkPolicy kiểm soát traffic |
