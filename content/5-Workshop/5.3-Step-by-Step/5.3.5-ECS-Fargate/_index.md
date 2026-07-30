---
title : "Deploy to Amazon ECS Fargate"
date : 2026-07-06 
weight : 5
chapter : false
pre : " <b> 5.3.5. </b> "
---

# 5.3.5 Deploy Application to Amazon ECS Fargate

---

### 1. Configure ECS Task Definition (`tetris-app`)

The Task Definition is configured with strict security controls (Container Hardening) within `infrastructure/terraform/main.tf` and `ecs-task-def.json`:
- **CPU / Memory Allocation:** 256 CPU (.25 vCPU) / 512 MiB RAM.
- **Compute Provider:** `FARGATE` (Production) and `FARGATE_SPOT` (Staging — saving 70% in compute costs).
- **Container Port:** 8080 (Nginx Unprivileged non-root user 101).
- **Security Context:** `user: "101"`, `readonlyRootFilesystem: true`.
- **Init Container Pattern:** Uses `volume-permissions` init container (running root 0) to grant read/write permissions on `/var/cache/nginx` and `/tmp` before main `tetris` container starts.
- **CloudWatch Logging Integration:** Streams container stdout/stderr logs directly to CloudWatch Log Group `/ecs/devsecops-factory`.
- **ECS Cluster:** `devsecops-factory-cluster`.
- **Services:** `tetris-staging` (`FARGATE_SPOT`) and `tetris-production` (`FARGATE`).

---

### 2. Update ECS Staging Service

Automatically registers the updated Task Definition and triggers a deployment to the ECS Staging Service (Pipeline Stage 14):

```bash
# Stage 14: Deploy to ECS Staging Service
aws ecs update-service \
    --cluster devsecops-factory-cluster \
    --service tetris-staging \
    --force-new-deployment \
    --region ap-southeast-1
```

---

### 3. Manual Approval Gate & Production Promotion

Following successful DAST scanning on the Staging ALB endpoint, the pipeline pauses at an interactive approval checkpoint (**Stage 19 — Manual Approval Gate**) on the Jenkins UI. Upon selecting **Proceed**, the pipeline releases deployment to Production:

```bash
# Stage 21: Deploy to ECS Production Service
aws ecs update-service \
    --cluster devsecops-factory-cluster \
    --service tetris-production \
    --force-new-deployment \
    --region ap-southeast-1
```

> **Approval Gate Timeout:** Set to **30 minutes**. If unapproved within the timeframe, the pipeline automatically aborts execution.

---

### 4. Manage Task Scaling (Cost Optimization)

Utilize the helper script to scale container task counts on demand:

```bash
# Scale tasks up for testing/demo
scripts/scale-ecs.sh up

# Scale tasks down to 0 to eliminate Fargate compute costs
scripts/scale-ecs.sh down
```

---

### Actual Screenshots: Amazon ECS Fargate Cluster & Services

![Amazon ECS Fargate Cluster](/images/5-Workshop/5.3-Step-by-Step/aws_ecs_fargate_services.png)
*Figure 5.3.5a: Amazon ECS Cluster `devsecops-factory-cluster` interface displaying active services `tetris-production` and `tetris-staging`.*

![ECS Services Running](/images/5-Workshop/5.3-Step-by-Step/ecs_services_running.png)
*Figure 5.3.5b: Amazon ECS Console showing both `tetris-staging` and `tetris-production` services in RUNNING status with desired count = 1.*

![ECS Task Detail](/images/5-Workshop/5.3-Step-by-Step/ecs_task_detail.png)
*Figure 5.3.5c: ECS Task detail view — CPU 256, Memory 512MB, Network Mode `awsvpc`, non-root user 101.*

![Tetris Application Staging](/images/5-Workshop/5.3-Step-by-Step/app_staging_browser.png)
*Figure 5.3.5d: Tetris web application running successfully on ECS Fargate Staging environment, accessible via ALB URL.*

![Tetris Application Production](/images/5-Workshop/5.3-Step-by-Step/app_production_browser.png)
*Figure 5.3.5e: Tetris web application live on ECS Fargate Production environment following Manual Approval Gate promotion.*
