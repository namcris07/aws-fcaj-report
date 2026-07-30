---
title : "Triển khai Amazon ECS Fargate"
date : 2026-07-06 
weight : 5
chapter : false
pre : " <b> 5.3.5. </b> "
---

# 5.3.5 Triển khai Ứng dụng lên Amazon ECS Fargate

---

### 1. Cấu hình ECS Task Definition (	etris-app)

Task Definition được cấu hình bảo mật nghiêm ngặt (Container Hardening) trong infrastructure/terraform/main.tf và ecs-task-def.json:
- **CPU / Memory:** 256 CPU (.25 vCPU) / 512 MiB RAM.
- **Runtime:** FARGATE (Production) và FARGATE_SPOT (Staging — tiết kiệm chi phí 70%).
- **Container Port:** 8080 (Nginx Unprivileged non-root user 101).
- **Security Context:** user: "101", eadonlyRootFilesystem: true.
- **Init Container Pattern:** Sử dụng container olume-permissions (chạy root 0) để cấp quyền trên /var/cache/nginx và /tmp trước khi main container 	etris khởi động.
- **CloudWatch Logging:** Đẩy logs trực tiếp lên Log Group /ecs/devsecops-factory.
- **ECS Cluster:** devsecops-factory-cluster.
- **Services:** 	etris-staging (FARGATE_SPOT) và 	etris-production (FARGATE).

---

### 2. Cập nhật ECS Service Staging

Tự động cập nhật Task Definition mới và yêu cầu deployment lên ECS Staging Service (Stage 14 của pipeline):

```bash
# Stage 14: Deploy ECS Staging
aws ecs update-service \
    --cluster devsecops-factory-cluster \
    --service tetris-staging \
    --force-new-deployment \
    --region ap-southeast-1
```

---

### 3. Manual Approval Gate & Production Promotion

Sau khi DAST scan thành công trên Staging ALB, pipeline dừng lại chờ nút phê duyệt thủ công (**Stage 19 — Manual Approval Gate**) trên Jenkins UI. Khi kỹ sư chọn **Proceed**, pipeline tiếp tục deploy lên Production:

```bash
# Stage 21: Deploy ECS Production
aws ecs update-service \
    --cluster devsecops-factory-cluster \
    --service tetris-production \
    --force-new-deployment \
    --region ap-southeast-1
```

> Approval Gate có timeout **30 phút**. Nếu không có phê duyệt, pipeline sẽ tự động Abort.

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
*Hình 5.3.5a: Giao diện Amazon ECS Cluster devsecops-factory-cluster hiển thị các Active Services 	etris-production và 	etris-staging.*

![ECS Services Running](/images/5-Workshop/5.3-Step-by-Step/ecs_services_running.png)
*Hình 5.3.5b: Amazon ECS Console hiển thị cả hai services 	etris-staging và 	etris-production ở trạng thái RUNNING với desired count = 1.*

![ECS Task Detail](/images/5-Workshop/5.3-Step-by-Step/ecs_task_detail.png)
*Hình 5.3.5c: Chi tiết ECS Task — CPU 256, Memory 512MB, Network Mode awsvpc, non-root user 101.*

![Ứng dụng Tetris Staging](/images/5-Workshop/5.3-Step-by-Step/app_staging_browser.png)
*Hình 5.3.5d: Ứng dụng Tetris đang chạy thành công trên môi trường ECS Fargate Staging, truy cập qua ALB URL.*

![Ứng dụng Tetris Production](/images/5-Workshop/5.3-Step-by-Step/app_production_browser.png)
*Hình 5.3.5e: Ứng dụng Tetris đang chạy trên ECS Fargate Production sau khi Manual Approval Gate được phê duyệt.*