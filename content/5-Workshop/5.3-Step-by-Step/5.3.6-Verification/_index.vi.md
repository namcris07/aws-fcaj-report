---
title : "Xác minh sau Triển khai (Verification)"
date : 2026-07-06 
weight : 6
chapter : false
pre : " <b> 5.3.6. </b> "
---

# 5.3.6 Xác minh sau Triển khai (Post-Deployment Verification)

---

Sau khi Jenkins pipeline 22 stages hoàn thành, việc thực hiện kiểm tra xác minh toàn diện trên cả môi trường AWS Cloud và Local Platform là bắt buộc để đảm bảo ứng dụng vận hành an toàn và đúng cấu hình.

---

### 1. Kiểm tra trạng thái ECS Services trên AWS

Sử dụng AWS CLI để kiểm tra trạng thái hoạt động của hai dịch vụ ECS Fargate 	etris-staging và 	etris-production:

```bash
aws ecs describe-services \
  --profile devsecops-factory \
  --region ap-southeast-1 \
  --cluster devsecops-factory-cluster \
  --services tetris-staging tetris-production \
  --query 'services[].{service:serviceName,desired:desiredCount,running:runningCount,status:status}'
```

**Kết quả mong đợi:**
- 	tetris-staging: desiredCount = 1, 
runningCount = 1, status = ACTIVE (chạy trên FARGATE_SPOT)
- 	tetris-production: desiredCount = 1, 
runningCount = 1, status = ACTIVE (chạy trên FARGATE)

---

### 2. Truy cập Ứng dụng qua ALB DNS

Lấy địa chỉ Application Load Balancer (ALB) công khai từ Terraform output và kiểm tra phản hồi HTTP:

```bash
# Lấy ALB DNS từ Terraform
STAGING_URL=""
PROD_URL=""

echo "Staging URL: http://"
echo "Production URL: http://"

# Kiểm tra HTTP response header
curl -I "http://"     # Expected: HTTP/1.1 200 OK
curl -I "http://"        # Expected: HTTP/1.1 200 OK
```

---

### 3. Kiểm tra Báo cáo Bảo mật trên Amazon S3

Xác nhận toàn bộ báo cáo từ 6 Security Gates đã được upload đúng cấu hình prefix:

```bash
BUCKET_NAME=""

aws s3 ls "s3:///reports/" --recursive
```

**Danh sách tệp mong đợi:**
- reports/secrets/.../gitleaks-report.json
- reports/sca/.../trivy-sca-report.json
- reports/sast/.../sonar-issues.json
- reports/container/.../container-scan-report.json
- reports/dast/.../zap-report.json
- reports/iac/.../checkov_report.json
- reports/asff/.../securityhub-asff.json
- reports/normalized/.../findings.json

---

### 4. Kiểm tra AWS Security Hub & Lambda Ingestion

Kiểm tra nhật ký AWS CloudWatch Logs của Lambda để đảm bảo các ASFF findings đã được import thành công:

```bash
aws logs tail /aws/lambda/devsecops-factory-securityhub-importer \
  --profile devsecops-factory \
  --region ap-southeast-1 \
  --since 1h
```

**Kết quả mong đợi:** Lambda log ghi nhận `importedFindings > 0` và `failedFindings = 0`.

---

### 5. Kiểm tra Local GitOps (k3d Cluster & Argo CD)

Đối với kịch bản FULL_PROJECT_DEMO, kiểm tra tính đồng bộ của Argo CD trên k3d cluster:

```bash
export KUBECONFIG="C:\Users\DELL/.kube/devsecops-local.kubeconfig"

# Kiểm tra Argo CD Applications
kubectl get applications -n argocd

# Kiểm tra Pods trong staging và production namespace
kubectl get pods -n staging
kubectl get pods -n production

# Kiểm tra endpoint local
curl -I http://tetris-staging.localhost    # Expected: HTTP 200
curl -I http://tetris-production.localhost # Expected: HTTP 200
```

---

### Bảng Kiểm thử Nghiệm thu Thành công (Checklist)

| TIêu chí nghiệm thu | Trạng thái mong đợi | Phương pháp xác minh |
|---|---|---|
| **Jenkins Build** | SUCCESS (22 stages) | Jenkins UI Dashboard |
| **Artifacts** | Đủ 6 JSON reports | scan-reports/ folder |
| **Amazon ECR** | Image tag 12-char SHA | aws ecr list-images |
| **Amazon ECS Staging** | 1 Task RUNNING (SPOT) | aws ecs describe-services |
| **Amazon ECS Production**| 1 Task RUNNING (FARGATE) | aws ecs describe-services |
| **Amazon S3** | Full reports uploaded | aws s3 ls |
| **AWS Security Hub** | Findings imported | AWS Console Security Hub |
| **Local GitOps** | Synced & Healthy | Argo CD Dashboard |