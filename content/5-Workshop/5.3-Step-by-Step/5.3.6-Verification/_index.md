---
title : "Post-Deployment Verification"
date : 2026-07-06 
weight : 6
chapter : false
pre : " <b> 5.3.6. </b> "
---

# 5.3.6 Post-Deployment Verification

---

After the 22-stage Jenkins pipeline completes, executing post-deployment verification across both AWS Cloud and Local Platform is mandatory to ensure application health and security configuration.

---

### 1. Verify AWS ECS Services Status

Use AWS CLI to verify the status of both ECS Fargate services (`tetris-staging` and `tetris-production`):

```bash
aws ecs describe-services \
  --profile devsecops-factory \
  --region ap-southeast-1 \
  --cluster devsecops-factory-cluster \
  --services tetris-staging tetris-production \
  --query 'services[].{service:serviceName,desired:desiredCount,running:runningCount,status:status}'
```

**Expected Output:**
- `tetris-staging`: `desiredCount = 1`, `runningCount = 1`, `status = ACTIVE` (running on FARGATE_SPOT)
- `tetris-production`: `desiredCount = 1`, `runningCount = 1`, `status = ACTIVE` (running on FARGATE)

---

### 2. Access Application via ALB Public DNS

Retrieve the public Application Load Balancer DNS endpoints from Terraform output and test HTTP response:

```bash
# Get ALB DNS endpoints from Terraform
STAGING_URL="$(terraform -chdir=infrastructure/terraform output -raw alb_dns_staging)"
PROD_URL="$(terraform -chdir=infrastructure/terraform output -raw alb_dns_production)"

echo "Staging URL: http://${STAGING_URL}"
echo "Production URL: http://${PROD_URL}"

# Check HTTP response header
curl -I "http://${STAGING_URL}"     # Expected: HTTP/1.1 200 OK
curl -I "http://${PROD_URL}"        # Expected: HTTP/1.1 200 OK
```

---

### 3. Verify Security Reports on Amazon S3

Confirm that all security reports from the 6 Security Gates have been uploaded to S3:

```bash
BUCKET_NAME="$(terraform -chdir=infrastructure/terraform output -raw security_report_bucket)"

aws s3 ls "s3://${BUCKET_NAME}/reports/" --recursive
```

**Expected Reports List:**
- `reports/secrets/.../gitleaks-report.json`
- `reports/sca/.../trivy-sca-report.json`
- `reports/sast/.../sonar-issues.json`
- `reports/container/.../container-scan-report.json`
- `reports/dast/.../zap-report.json`
- `reports/iac/.../checkov_report.json`
- `reports/asff/.../securityhub-asff.json`
- `reports/normalized/.../findings.json`

---

### 4. Verify AWS Security Hub & Lambda Ingestion

Inspect CloudWatch Logs of the Lambda importer function to confirm ASFF findings were ingested:

```bash
aws logs tail /aws/lambda/devsecops-factory-securityhub-importer \
  --profile devsecops-factory \
  --region ap-southeast-1 \
  --since 1h
```

**Expected Output:** Lambda log confirms `importedFindings > 0` and `failedFindings = 0`.

---

### 5. Verify Local GitOps (k3d Cluster & Argo CD)

For `FULL_PROJECT_DEMO` scenario, verify Argo CD synchronization on the local k3d cluster:

```bash
export KUBECONFIG="$HOME/.kube/devsecops-local.kubeconfig"

# Check Argo CD Applications
kubectl get applications -n argocd

# Check Pods in staging and production namespaces
kubectl get pods -n staging
kubectl get pods -n production

# Test local endpoints
curl -I http://tetris-staging.localhost    # Expected: HTTP 200
curl -I http://tetris-production.localhost # Expected: HTTP 200
```

---

### Verification Checklist Table

| Acceptance Criteria | Expected State | Verification Method |
|---|---|---|
| **Jenkins Build** | SUCCESS (22 stages) | Jenkins UI Dashboard |
| **Artifacts** | All 6 JSON reports present | `scan-reports/` folder |
| **Amazon ECR** | Image tagged with 12-char SHA | `aws ecr list-images` |
| **Amazon ECS Staging** | 1 Task RUNNING (SPOT) | `aws ecs describe-services` |
| **Amazon ECS Production**| 1 Task RUNNING (FARGATE) | `aws ecs describe-services` |
| **Amazon S3** | Full reports uploaded | `aws s3 ls` |
| **AWS Security Hub** | Findings imported | AWS Console Security Hub |
| **Local GitOps** | Synced & Healthy | Argo CD Dashboard |