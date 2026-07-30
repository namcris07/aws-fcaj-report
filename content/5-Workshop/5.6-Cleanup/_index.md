---
title : "Clean Up Resources"
date : 2026-07-06 
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

# 5.6 Clean Up Resources

---

After completing testing and workshop reporting, cleaning up AWS resources promptly is **mandatory** to avoid incurring unnecessary costs. The project provides **2 distinct cleanup levels**:

---

## Level 1: Pause — Scale ECS to 0 (Demo Reset)

The fastest way to stop incurring Fargate costs is to set `desired-count = 0` for both ECS services. ALBs and other resources will still exist, but Fargate task compute costs will immediately drop to $0. This is ideal when planning to re-demo later.

```bash
# Quick method: use Makefile target
AWS_PROFILE_NAME=devsecops-factory make demo-reset

# Or manually scale each service to zero:
# Scale Staging to 0
aws ecs update-service \
  --profile devsecops-factory \
  --cluster devsecops-factory-cluster \
  --service tetris-staging \
  --desired-count 0 \
  --region ap-southeast-1

# Scale Production to 0
aws ecs update-service \
  --profile devsecops-factory \
  --cluster devsecops-factory-cluster \
  --service tetris-production \
  --desired-count 0 \
  --region ap-southeast-1
```

After scaling tasks to 0, stop the local platform to free up workstation compute resources:

```bash
# Stop Docker Compose stack (preserves volumes — keeps Jenkins history & SonarQube data)
make down

# Stop k3d cluster (preserves state; can be restarted anytime)
k3d cluster stop devsecops

# Confirm all containers have stopped
docker compose ps
```

> ⚠️ **Note regarding ALBs:** The two Application Load Balancers will remain active after scaling ECS tasks to 0 and may incur minor baseline charges (~$0.008/hour per ALB). To completely achieve $0 cost, execute Level 2 (Terraform destroy).

---

## Level 2: Complete AWS Infrastructure Teardown (`cleanup-aws.sh`)

> ⚠️ **Important Warning:** This process **permanently destroys** all AWS resources for the project, including: ECS Cluster, ALBs, ECR images, S3 report buckets, Lambda functions, Security Hub integration, CloudWatch log groups, VPC, and IAM resources. **Deleted data cannot be recovered.**

### Step 1: Stop Entire Local Platform

Ensure no Jenkins builds are running or awaiting manual approval prior to executing destroy:

```bash
make down
k3d cluster stop devsecops
```

### Step 2: Confirm Target AWS Account Identity

Double-check that CLI credentials point to the correct AWS Account ID before running destroy:

```bash
aws sso login --profile devsecops-factory
aws sts get-caller-identity --profile devsecops-factory
# Expected output:
# {
#   "Account": "585572506644",  <-- Must match expected account ID
#   "Arn": "arn:aws:iam::585572506644:..."
# }
# DO NOT proceed if Account ID does not match!
```

### Step 3: Execute Cleanup Script with Safety Gate

The script `scripts/cleanup-aws.sh` requires three explicit confirmation environment variables before triggering destroy — serving as a safety gate against accidental teardown:

```bash
AWS_PROFILE=devsecops-factory \
EXPECTED_AWS_ACCOUNT_ID=585572506644 \
CONFIRM_AWS_CLEANUP=devsecops-factory \
DESTROY_TERRAFORM=true \
./scripts/cleanup-aws.sh
```

**Automated Workflow Executed by `cleanup-aws.sh`:**
1. **Account Verification:** Checks that CLI Account ID matches `EXPECTED_AWS_ACCOUNT_ID` to prevent accidental deletion of wrong environments.
2. **Scale ECS to Zero:** Sets `desired-count` to `0` for both ECS staging and production services.
3. **Terraform Destroy:** Automatically executes `terraform destroy` to delete VPC, Subnets, ALBs, Target Groups, Security Groups, IAM Roles, S3 Bucket (`force_destroy = true`), ECR Repository (`force_delete = true`), Lambda, and Security Hub integration.
4. **Deregister Task Definitions:** Deregisters all active task definition revisions.
5. **State Verification:** Confirms Terraform state file is empty.

> Review the destroy plan carefully and enter `yes` only when resource scope is verified.

### Step 4: Verify Empty Terraform State

```bash
# Verify state file is empty
terraform -chdir=infrastructure/terraform state list
# Expected output: (empty output)

# Confirm on AWS Console that ECS cluster is removed
aws ecs describe-clusters \
  --profile devsecops-factory \
  --clusters devsecops-factory-cluster \
  --query 'clusters[].status'
# Expected output: [] or "INACTIVE"
```

---

## Cost Comparison by Cleanup State

| Resource | Post Scale-to-Zero | Post Terraform Destroy |
|---|---|---|
| ECS Fargate Tasks | $0 (no running tasks) | $0 |
| Application Load Balancer | ~$0.008/hr/ALB (2 ALBs) | $0 |
| Amazon ECR | ~$0.10/GB/month | $0 |
| Amazon S3 | ~$0.023/GB/month | $0 |
| AWS Lambda | $0 (no invocations) | $0 |
| CloudWatch Logs | Minimal (~$0.50/GB) | $0 |
| **Total** | **~$0.32/day (2 ALBs)** | **$0.00** |

---

## 3. Stop Local Stack Containers (Without `make down`)

To manually shut down individual local components:

```bash
docker compose -f docker-compose.infra.yml down --remove-orphans
docker compose -f docker-compose.security.yml down --remove-orphans
docker compose -f docker-compose.obs.yml down --remove-orphans
docker compose down --remove-orphans

# Delete local k3d Kubernetes cluster (if full reset desired)
k3d cluster delete devsecops
# or
make k3d-delete
```

> `make down` preserves named volumes. `make clean` removes local containers and k3d cluster without destroying AWS resources.

---

### Actual Screenshot: Complete Teardown Result

![Terraform Destroy Complete](/images/5-Workshop/5.6-Cleanup/destroy_complete.png)
*Figure 5.6: Complete AWS resource cleanup output in Terminal (`Destroy complete! Resources: X destroyed`).*

---

> **Cost Optimization Best Practices During Development:**
> - After each demo session, always run `make demo-reset` to scale ECS tasks to zero.
> - Use `make down` to stop local containers and free system memory (RAM).
> - Deploy full AWS cloud infrastructure only when end-to-end cloud validation is required.
> - AWS Budget Alerts set at 50%/80%/100% provide early warnings if spending approaches thresholds.
> - AWS Cost Explorer has a 24–48 hour reporting latency — running destroy stops future billing immediately but does not purge previously accumulated charges.