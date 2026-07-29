---
title : "Clean up"
date : 2026-07-06 
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

# 5.6 Resource Clean-up

---

After completing all workshop verification tasks and report documentation, perform resource clean-up to prevent unintended cloud charges. The repository provides 2 distinct clean-up tiers:

---

### Tier 1: Temporary Pause (Demo Reset)

To immediately stop Fargate compute charges **without destroying** Terraform infrastructure definitions (ideal when preparing for another demo):

```bash
# Option 1: Using make command
AWS_PROFILE_NAME=devsecops-factory make demo-reset

# Option 2: Using scale-ecs.sh script
scripts/scale-ecs.sh down
```

The command above sets the `desired-count` to `0` for both `devsecops-factory-staging` and `devsecops-factory-prod` ECS services.

---

### Tier 2: Complete Infrastructure Destruction (`cleanup-aws.sh`)

This method destroys all AWS cloud resources including VPC, ALBs, ECR repositories, S3 report buckets, CloudWatch log groups, IAM users, and Terraform state.

```bash
# 1. Login and verify expected AWS Account ID (Account: 585572506644)
aws sts get-caller-identity --profile devsecops-factory

# 2. Execute automated teardown script
AWS_PROFILE=devsecops-factory \
EXPECTED_AWS_ACCOUNT_ID=585572506644 \
CONFIRM_AWS_CLEANUP=devsecops-factory \
DESTROY_TERRAFORM=true \
./scripts/cleanup-aws.sh
```

**Automated Workflow of `cleanup-aws.sh`:**
1. **Account Verification:** Confirms active Account ID matches target to avoid destroying unintended environments.
2. **ECS Scaling:** Scales desired count for all ECS services to `0`.
3. **Terraform Destroy:** Executes `terraform destroy` to tear down VPC, Subnets, ALBs, Target Groups, Security Groups, IAM Roles, S3 Buckets (`force_destroy = true`), and ECR Repositories (`force_delete = true`).
4. **Deregister Task Definitions:** Deregisters all active revisions of `tetris-app` Task Definition.
5. **State File Reset:** Confirms Terraform state file is emptied.

---

### Genuine Screenshot: Complete AWS Resource Cleanup (`terraform destroy`)

![Terraform Destroy Complete](/images/5-Workshop/5.6-Cleanup/destroy_complete.png)
*Figure 5.6: Terminal execution output confirming all 42 AWS resources destroyed successfully (`Destroy complete! Resources: 42 destroyed`).*

---

### 3. Stop Local Stack Containers

Tear down local development services (Jenkins, SonarQube, Prometheus, Grafana, Argo CD):

```bash
docker compose -f docker-compose.infra.yml down --remove-orphans
docker compose -f docker-compose.security.yml down --remove-orphans
docker compose -f docker-compose.obs.yml down --remove-orphans
docker compose down --remove-orphans

# Delete local k3d Kubernetes cluster (if created)
make k3d-delete
```

> **Note:** Completing all steps above ensures your AWS account incurs **$0.00 USD** ongoing operational costs.