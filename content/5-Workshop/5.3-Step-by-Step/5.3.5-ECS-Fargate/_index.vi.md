---
title : "Triển khai Amazon ECS Fargate"
date : 2026-07-06 
weight : 5
chapter : false
pre : " <b> 5.3.5. </b> "
---

# 5.3.5 Triển khai Ứng dụng lên Amazon ECS Fargate

---

### 1. Cấu hình ECS Task Definition (`tetris-app`)

Task Definition được cấu hình bảo mật nghiêm ngặt (Container Hardening):
- **CPU / Memory:** 256 CPU (.25 vCPU) / 512 MiB RAM.
- **Security Context:** `user: "101"` (chạy non-root user Nginx), `readonlyRootFilesystem: true`.
- **Mount Volumes:** Gắn tạm `tmpfs` volume vào `/tmp` và `/var/cache/nginx`.
- **CloudWatch Logging:** Đẩy logs trực tiếp lên Log Group `/ecs/devsecops-factory`.

---

### 2. Cập nhật ECS Service Staging

Tự động cập nhật Task Definition mới và yêu cầu deployment lên ECS Staging Service:

```bash
aws ecs update-service \
    --cluster devsecops-factory-cluster \
    --service devsecops-factory-staging \
    --force-new-deployment \
    --region ap-southeast-1
```

---

### 3. Manual Approval Gate & Production Promotion

Sau khi DAST scan thành công trên Staging ALB, pipeline dừng lại chờ nút phê duyệt thủ công (Manual Approval Gate) trên Jenkins UI trước khi triển khai lên Production ECS Service:

```bash
aws ecs update-service \
    --cluster devsecops-factory-cluster \
    --service devsecops-factory-prod \
    --force-new-deployment \
    --region ap-southeast-1
```

---

### 4. Quản lý Scale Tasks (Tối ưu Chi phí)

Sử dụng script có sẵn để thay đổi số lượng task chạy nhanh chóng:

```bash
# Bật tasks để test/demo
scripts/scale-ecs.sh up

# Tắt tasks (scale về 0) để ngắt phí Fargate
scripts/scale-ecs.sh down
```

---

### Ảnh chụp màn hình thực tế: Amazon ECS Fargate Cluster & Services

![Amazon ECS Fargate Cluster](/images/5-Workshop/5.3-Step-by-Step/aws_ecs_fargate_services.png)
*Hình 5.3.5: Giao diện Amazon ECS Cluster `devsecops-factory-cluster` hiển thị các Active Services `tetris-production` và `tetris-staging`.*
