---
title : "Deploy to Amazon ECS Fargate"
date : 2026-07-06 
weight : 5
chapter : false
pre : " <b> 5.3.5. </b> "
---

# 5.3.5 Deploy Application to Amazon ECS Fargate

---

### 1. Update ECS Service Staging
Deploy updated task definition to Staging ECS Service:

```bash
aws ecs update-service \
    --cluster devsecops-cluster \
    --service devsecops-staging-service \
    --force-new-deployment \
    --region ap-southeast-1
```

---

### 2. Manual Approval Gate & Production Promotion
Pause pipeline execution awaiting manual review before promoting build to Production ECS Service.
