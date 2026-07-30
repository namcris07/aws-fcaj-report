---
title : "Dọn dẹp tài nguyên"
date : 2026-07-06 
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

# 5.6 Dọn dẹp tài nguyên (Clean-up)

---

Sau khi hoàn tất việc kiểm thử và báo cáo workshop, việc dọn dẹp tài nguyên AWS kịp thời là **bắt buộc** để tránh phát sinh chi phí không cần thiết. Dự án cung cấp **2 mức độ** dọn dẹp riêng biệt:

---

## Mức 1: Tạm dừng — Scale ECS về 0 (Demo Reset)

Cách nhanh nhất để ngừng tốn phí Fargate là đặt `desired-count = 0` cho cả hai ECS services. ALB và các tài nguyên khác vẫn tồn tại nhưng chi phí Fargate task sẽ về $0 ngay lập tức. Phù hợp khi muốn demo lại sau đó.

```bash
# Cách nhanh: dùng Makefile target
AWS_PROFILE_NAME=devsecops-factory make demo-reset

# Hoặc scale thủ công từng service:
# Scale Staging về 0
aws ecs update-service \
  --profile devsecops-factory \
  --cluster devsecops-factory-cluster \
  --service tetris-staging \
  --desired-count 0 \
  --region ap-southeast-1

# Scale Production về 0
aws ecs update-service \
  --profile devsecops-factory \
  --cluster devsecops-factory-cluster \
  --service tetris-production \
  --desired-count 0 \
  --region ap-southeast-1
```

Sau khi scale về 0, dừng local platform để giải phóng tài nguyên máy:

```bash
# Dừng Docker Compose stack (không xóa volumes — giữ Jenkins history, SonarQube data)
make down

# Dừng k3d cluster (giữ state, có thể restart lại)
k3d cluster stop devsecops

# Xác nhận tất cả containers đã dừng
docker compose ps
```

> ⚠️ **Lưu ý về ALB:** Hai Application Load Balancers vẫn tồn tại sau khi ECS scale về 0 và có thể phát sinh phí nhỏ (~$0.008/giờ mỗi ALB). Để hoàn toàn về $0, cần thực hiện Mức 2 (Terraform destroy).

---

## Mức 2: Xóa triệt để toàn bộ hạ tầng AWS (`cleanup-aws.sh`)

> ⚠️ **Cảnh báo quan trọng:** Quy trình này **xóa vĩnh viễn** toàn bộ tài nguyên AWS của dự án bao gồm: ECS Cluster, ALBs, ECR images, S3 reports, Lambda, Security Hub integration, CloudWatch log groups, VPC và IAM resources. **Dữ liệu bị xóa không thể khôi phục.**

### Bước 1: Dừng toàn bộ local platform

Đảm bảo không có Jenkins build đang chạy/chờ approval trước khi destroy:

```bash
make down
k3d cluster stop devsecops
```

### Bước 2: Xác nhận đúng AWS Account

Kiểm tra kỹ đang đăng nhập đúng account trước khi destroy:

```bash
aws sso login --profile devsecops-factory
aws sts get-caller-identity --profile devsecops-factory
# Expected output:
# {
#   "Account": "585572506644",  <-- Phải là account đúng
#   "Arn": "arn:aws:iam::585572506644:..."
# }
# KHÔNG tiếp tục nếu Account ID không khớp!
```

### Bước 3: Chạy Script Cleanup với Safety Gate

Script `scripts/cleanup-aws.sh` yêu cầu ba biến môi trường xác nhận trước khi destroy — đây là safety gate ngăn chặn xóa nhầm:

```bash
AWS_PROFILE=devsecops-factory \
EXPECTED_AWS_ACCOUNT_ID=585572506644 \
CONFIRM_AWS_CLEANUP=devsecops-factory \
DESTROY_TERRAFORM=true \
./scripts/cleanup-aws.sh
```

**Quy trình tự động của `cleanup-aws.sh`:**
1. **Xác minh tài khoản:** Kiểm tra Account ID khớp với `EXPECTED_AWS_ACCOUNT_ID` để tránh xóa nhầm môi trường.
2. **Scale ECS về 0:** Đưa `desired-count` về `0` cho cả hai ECS services.
3. **Terraform Destroy:** Tự động thực thi `terraform destroy` xóa VPC, Subnets, ALB, Target Groups, Security Groups, IAM Roles, S3 Bucket (`force_destroy = true`), ECR Repository (`force_delete = true`), Lambda, Security Hub integration.
4. **Deregister Task Definitions:** Hủy đăng ký toàn bộ các revision active.
5. **Kiểm tra State:** Xác nhận Terraform state đã trở về trạng thái rỗng.

> Xem destroy plan và chỉ nhập `yes` khi phạm vi đúng.

### Bước 4: Xác nhận Terraform State trống

```bash
# Kiểm tra state đã rỗng
terraform -chdir=infrastructure/terraform state list
# Expected: (không có output - state trống)

# Kiểm tra trên AWS Console không còn ECS cluster
aws ecs describe-clusters \
  --profile devsecops-factory \
  --clusters devsecops-factory-cluster \
  --query 'clusters[].status'
# Expected: [] hoặc "INACTIVE"
```

---

## Chi phí theo từng trạng thái cleanup

| Tài nguyên | Sau Scale-to-zero | Sau Terraform Destroy |
|---|---|---|
| ECS Fargate Tasks | $0 (không có task chạy) | $0 |
| Application Load Balancer | ~$0.008/giờ/ALB (2 ALB) | $0 |
| Amazon ECR | ~$0.10/GB/tháng | $0 |
| Amazon S3 | ~$0.023/GB/tháng | $0 |
| AWS Lambda | $0 (không invoke) | $0 |
| CloudWatch Logs | Nhỏ (~$0.50/GB) | $0 |
| **Tổng** | **~$0.32/ngày (2 ALB)** | **$0.00** |

---

## 3. Tắt các container local stack (nếu không dùng `make down`)

Tắt từng thành phần local nếu cần kiểm soát chi tiết:

```bash
docker compose -f docker-compose.infra.yml down --remove-orphans
docker compose -f docker-compose.security.yml down --remove-orphans
docker compose -f docker-compose.obs.yml down --remove-orphans
docker compose down --remove-orphans

# Xóa cụm Kubernetes local k3d (nếu muốn xóa hẳn)
k3d cluster delete devsecops
# hoặc
make k3d-delete
```

> `make down` không xóa named volumes. `make clean` xóa container local và cluster k3d nhưng không destroy AWS.

---

### Ảnh chụp màn hình thực tế: Kết quả dọn dẹp triệt để

![Terraform Destroy Complete](/images/5-Workshop/5.6-Cleanup/destroy_complete.png)
*Hình 5.6: Kết quả dọn dẹp triệt để toàn bộ tài nguyên AWS trên Terminal (`Destroy complete! Resources: X destroyed`).*

---

> **Tối ưu chi phí trong quá trình phát triển:**
> - Sau mỗi buổi demo, luôn chạy `make demo-reset` để scale ECS về 0
> - Sử dụng `make down` để dừng local platform, giải phóng RAM
> - Chỉ deploy hạ tầng AWS khi cần demo đầy đủ
> - AWS Budget Alert ở 50%/80%/100% giúp cảnh báo sớm nếu chi phí vượt ngưỡng
> - AWS Cost Explorer có độ trễ 24–48 giờ — destroy ngăn tài nguyên tạo chi phí mới nhưng không xóa chi phí đã phát sinh