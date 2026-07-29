---
title : "Dọn dẹp tài nguyên"
date : 2026-07-06 
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

# 5.6 Dọn dẹp tài nguyên (Clean-up)

---

Sau khi hoàn tất việc kiểm thử và báo cáo workshop, bạn cần thực hiện dọn dẹp các tài nguyên đã tạo trên AWS nhằm tránh phát sinh chi phí ngoài ý muốn. Dự án cung cấp 2 mức độ dọn dẹp riêng biệt:

---

### Mức độ 1: Tạm dừng để tiết kiệm chi phí (Demo Reset)

Phương pháp này giúp ngắt ngay lập tức chi phí tính toán Fargate mà **không xóa** cấu hình hạ tầng Terraform (phù hợp khi chuẩn bị demo lại):

```bash
# Lựa chọn 1: Dùng make command
AWS_PROFILE_NAME=devsecops-factory make demo-reset

# Lựa chọn 2: Dùng script scale-ecs.sh
scripts/scale-ecs.sh down
```

Lệnh trên sẽ đưa `desired-count` về `0` cho cả 2 dịch vụ ECS `devsecops-factory-staging` và `devsecops-factory-prod`.

---

### Mức độ 2: Xóa triệt để toàn bộ hạ tầng AWS (`cleanup-aws.sh`)

Phương pháp này hủy hoàn toàn các tài nguyên trên AWS bao gồm VPC, ALB, ECR repository, S3 report bucket, CloudWatch log groups, IAM user và Terraform state.

```bash
# 1. Đăng nhập và xác minh đúng AWS Account ID (Account: 585572506644)
aws sts get-caller-identity --profile devsecops-factory

# 2. Khởi chạy script dọn dẹp tự động
AWS_PROFILE=devsecops-factory \
EXPECTED_AWS_ACCOUNT_ID=585572506644 \
CONFIRM_AWS_CLEANUP=devsecops-factory \
DESTROY_TERRAFORM=true \
./scripts/cleanup-aws.sh
```

**Quy trình dọn dẹp tự động của `cleanup-aws.sh`:**
1. **Xác minh tài khoản:** Kiểm tra Account ID khớp với giá trị kỳ vọng để tránh xóa nhầm môi trường khác.
2. **Scale ECS:** Đưa desired count của các dịch vụ ECS về `0`.
3. **Terraform Destroy:** Tự động thực thi `terraform destroy` xóa VPC, Subnets, ALB, Target Groups, Security Groups, IAM Roles, S3 Bucket (`force_destroy = true`), ECR Repository (`force_delete = true`).
4. **Deregister Task Definitions:** Tự động hủy đăng ký toàn bộ các revision active của Task Definition `tetris-app`.
5. **Kiểm tra trạng thái State:** Xác nhận file Terraform state đã trở về trạng thái rỗng.

---

### Ảnh chụp màn hình thực tế: Kết quả dọn dẹp triệt để hạ tầng AWS (`terraform destroy`)

![Terraform Destroy Complete](/images/5-Workshop/5.6-Cleanup/destroy_complete.png)
*Hình 5.6: Kết quả dọn dẹp triệt để toàn bộ 42 tài nguyên AWS trên Terminal (`Destroy complete! Resources: 42 destroyed`).*

---

### 3. Tắt các container local stack

Tắt toàn bộ các dịch vụ bổ trợ chạy local (Jenkins, SonarQube, Prometheus, Grafana, Argo CD):

```bash
docker compose -f docker-compose.infra.yml down --remove-orphans
docker compose -f docker-compose.security.yml down --remove-orphans
docker compose -f docker-compose.obs.yml down --remove-orphans
docker compose down --remove-orphans

# Xóa cụm Kubernetes local k3d (nếu có)
make k3d-delete
```

> **Lưu ý:** Sau khi hoàn tất đầy đủ các bước trên, tài khoản AWS của bạn sẽ hoàn toàn trở về trạng thái rỗng và không còn bất kỳ chi phí phát sinh nào từ các tài nguyên của dự án.