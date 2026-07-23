---
title : "Triển khai Amazon ECS Fargate"
date : 2026-07-06 
weight : 5
chapter : false
pre : " <b> 5.3.5. </b> "
---

# 5.3.5 Triển khai Ứng dụng lên Amazon ECS Fargate

---

### 1. Cập nhật ECS Service Staging
Tự động cập nhật Task Definition mới lên ECS Staging Service:

```bash
aws ecs update-service \
    --cluster devsecops-cluster \
    --service devsecops-staging-service \
    --force-new-deployment \
    --region ap-southeast-1
```

---

### 2. Manual Approval Gate & Production Promotion
Sau khi DAST scan thành công trên Staging, quản trị viên phê duyệt trên Jenkins UI để kích hoạt lệnh triển khai lên Production ECS Service.
