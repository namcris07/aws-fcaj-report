---
title: "Testing Strategy Overview"
date: 2026-07-06
weight: 1
chapter: false
pre: " <b> 5.5.1. </b> "
---

# 5.5.1 Testing Strategy Overview

---

The `devsecops-factory` system is designed following the **"Shift Left Security"** principle, placing security validation as early as possible in the CI/CD pipeline. To verify the pipeline, the team designed **[tetris-app](https://github.com/lamelihuynh/tetris-app.git)** (ReactJS + Nginx multi-stage build) as an intentional vulnerability target.

![DevSecOps 22-stage Pipeline Flow](/images/5-Workshop/5.5-Testing/pipeline_stages.png)

_Figure 5.5.1: Full DevSecOps 22-stage automated pipeline architecture running on Jenkins._

# STAGE 2: Deploy (Serve static files via Nginx)

# FROM nginxinc/nginx-unprivileged:alpine <- USED TO PASS container scan

FROM nginx:1.18.0 # <- CURRENTLY USED - Triggers CRITICAL CVEs
COPY --from=builder /app/build /usr/share/nginx/html
EXPOSE 8080
CMD ["nginx", "-g", "daemon off;"]

# [CKV_DOCKER_3]: Missing USER -> runs as root

# [CKV_DOCKER_2]: Missing HEALTHCHECK

---

### App & Docker Build Verification Screenshots

![Build React App](/images/5-Workshop/5.3-Step-by-Step/app-01-run_build_app.jpg)
_Figure 5.5.1b: React Tetris web application build compilation (`npm run build`)._

![Build Docker Image](/images/5-Workshop/5.3-Step-by-Step/app-02-build_docker.jpg)
_Figure 5.5.1c: Packaging application into Multi-stage Docker image._

![Local App GUI](/images/5-Workshop/5.3-Step-by-Step/app-02-docker_app.jpg)
_Figure 5.5.1d: Tetris web game GUI running locally via Docker container._

![AWS CloudWatch Logs Insights](/images/5-Workshop/5.3-Step-by-Step/aws_cloudwatch_insights.png)
_Figure 5.5.1e: Real-time container log analysis using AWS CloudWatch Logs Insights (`/ecs/devsecops-factory`)._

#### Table 5.5.1: Security Pipeline Testing Summary

| Stage       | Scan Type            | Tool        | Findings                            | Severity        | Result                            |
| ----------- | -------------------- | ----------- | ----------------------------------- | --------------- | --------------------------------- |
| **Stage 3** | Secrets Scan         | Gitleaks    | 2 secrets                           | MEDIUM – HIGH   | **FAIL (exit code 1)**            |
| **Stage 4** | SCA Scan             | Trivy FS    | 4 CVEs (HIGH)                       | HIGH            | **FAIL**                          |
| **Stage 5** | SAST Scan            | SonarQube   | 4+ code issues                      | MEDIUM – HIGH   | **FAIL**                          |
| **Stage 6** | IaC Scan             | Checkov     | 34 failures                         | MEDIUM – HIGH   | **FAIL (Soft-fail)**              |
| **Stage 8** | Container Image Scan | Trivy Image | 0 CVEs (Alpine) / 40+ CVEs (Debian) | CRITICAL – HIGH | **PASS (Alpine) / FAIL (Debian)** |
