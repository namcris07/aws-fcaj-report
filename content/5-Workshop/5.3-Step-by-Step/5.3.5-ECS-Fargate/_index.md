---
title : "Deploy to Amazon ECS Fargate"
date : 2026-07-06 
weight : 5
chapter : false
pre : " <b> 5.3.5. </b> "
---

# 5.3.5 Deploy Application to Amazon ECS Fargate

---

### 1. ECS Task Definition Specs (`tetris-app`)

The Task Definition incorporates container hardening practices:
- **CPU / Memory:** 256 CPU (.25 vCPU) / 512 MiB RAM.
- **Security Context:** `user: "101"` (non-root Nginx execution), `readonlyRootFilesystem: true`.
- **Volume Mounts:** In-memory `tmpfs` mounts for `/tmp` and `/var/cache/nginx`.
- **CloudWatch Logging:** Log streaming to Log Group `/ecs/devsecops-factory`.

---

### 2. Update ECS Service Staging

Deploy updated task definition to Staging ECS Service:

```bash
aws ecs update-service \
    --cluster devsecops-factory-cluster \
    --service devsecops-factory-staging \
    --force-new-deployment \
    --region ap-southeast-1
```

---

### 3. Manual Approval Gate & Production Promotion

After DAST scan succeeds against Staging ALB, pause execution awaiting manual approval on Jenkins UI before promoting to Production ECS Service:

```bash
aws ecs update-service \
    --cluster devsecops-factory-cluster \
    --service devsecops-factory-prod \
    --force-new-deployment \
    --region ap-southeast-1
```

---

### 4. Task Scale Management (Cost Control)

Utilize helper scripts to scale container tasks on demand:

```bash
# Scale tasks up for active testing/demo
scripts/scale-ecs.sh up

# Scale tasks down to 0 to prevent idle Fargate charges
scripts/scale-ecs.sh down
```

---

### Genuine Screenshot: Amazon ECS Fargate Cluster & Active Services

![Amazon ECS Fargate Cluster](/images/5-Workshop/5.3-Step-by-Step/aws_ecs_fargate_services.png)
*Figure 5.3.5: Amazon ECS Cluster `devsecops-factory-cluster` console displaying active `tetris-production` and `tetris-staging` Fargate services.*
