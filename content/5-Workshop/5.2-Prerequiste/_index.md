---
title : "Prerequisites & Setup"
date : 2026-07-06 
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

#### 5.2 Prerequisites & Environment Setup

This chapter details all requirements and setup steps needed before launching the **DevSecOps Factory** system. Completing these preparations upfront ensures a smooth setup and avoids errors during execution.

---

#### 0. Clone Repository & Environment Initialization

The project source code is centrally hosted on GitHub. Clone the official repository and switch to the GitOps branch:

```bash
git clone https://github.com/loi-bui0703/CICD-DevSecOps-using-AWS-services.git
cd CICD-DevSecOps-using-AWS-services

# Switch to the GitOps branch (local Argo CD monitors this branch)
git switch cicd-gitops

# Generate .env configuration file from template
make setup-env
# Open .env and fill in required values
```

> **Branch Note:** The local Argo CD applications are configured to monitor the `cicd-gitops` branch. If using a different branch, update the `GITOPS_BRANCH` parameter in the Jenkins pipeline before running builds.

---

#### 1. Hardware & Operating System Requirements

| Requirement | Description |
|---|---|
| **Operating System** | macOS or Linux (WSL2 on Windows). Tested on macOS with Docker Desktop. |
| **RAM** | Minimum 10–12 GB allocated to Docker (16 GB recommended). Jenkins, SonarQube, k3d, and OWASP ZAP are memory-intensive. |
| **Disk** | At least 25 GB free disk space for Docker images and volumes. |
| **Network** | Stable Internet connection for initial image downloads: Jenkins, SonarQube, k3d, ZAP (~3–5 GB). |

Required TCP ports on host machine:

| Port | Service | Description |
|---|---|---|
| 80 | ingress-nginx | HTTP ingress to local apps (*.localhost) |
| 443 | ingress-nginx | HTTPS |
| 3000 | Grafana | Monitoring dashboard |
| 5001 | Local Registry | Internal Docker registry |
| 6443 | k3d API Server | Kubernetes API |
| 8080 | Jenkins | CI/CD Engine UI |
| 8443 | Argo CD | GitOps UI (port-forward) |
| 9000 | SonarQube | SAST scanner UI |
| 9090 | Prometheus | Metrics scraping |
| 9418 | Git daemon | Local git-server for Jenkins |

---

#### 2. Tools & Dependencies Verification

Install and verify mandatory CLI tools:

```bash
docker version          # >= 24.0
git --version           # >= 2.40
make --version          # GNU Make >= 4.0
kubectl version --client # >= 1.28
helm version             # >= 3.12
k3d version              # >= 5.6
terraform version        # >= 1.0
aws --version            # AWS CLI v2 >= 2.13
jq --version             # >= 1.6
python3 --version        # >= 3.10
```

---

#### 3. Environment Variables Configuration (.env)

The project relies on a `.env` file for credentials and settings. This file is **git-ignored** and must never be committed.

```dotenv
# === LOCAL PLATFORM ===
JENKINS_ADMIN_PASS=<strong-password-8+-chars>
GRAFANA_ADMIN_PASSWORD=<strong-password>

# === SAST (SonarQube) ===
SONAR_TOKEN=<token-from-SonarQube-UI>

# === AWS (required for FULL_PROJECT_DEMO) ===
AWS_ACCESS_KEY_ID=<jenkins-ci-iam-user-access-key>
AWS_SECRET_ACCESS_KEY=<secret-key>
```

---

#### 4. Configure IAM Policy for Jenkins CI

The IAM User (or Role) used by Jenkins must be granted minimum privileges according to the principle of **Least Privilege**. In this project, IAM Policies and Users are fully managed by Terraform (`infrastructure/terraform/main.tf`).

---

#### 5. Provision AWS Cloud Infrastructure with Terraform

```bash
# Log in via AWS SSO
aws sso login --profile devsecops-factory
export AWS_PROFILE=devsecops-factory

# Verify correct account identity
aws sts get-caller-identity

# Initialize and apply Terraform manifests
terraform -chdir=infrastructure/terraform init
terraform -chdir=infrastructure/terraform apply -auto-approve
```

---

#### 6. Launch Local Platform Services

```bash
# Bootstrap local stack (Jenkins, SonarQube, Prometheus, Grafana, k3d, Argo CD)
make bootstrap

# Check system health status
make status
```

| Service | URL | Default Credentials |
|---|---|---|
| Jenkins | http://localhost:8080 | admin / `<JENKINS_ADMIN_PASS>` |
| SonarQube | http://localhost:9000 | admin / `<change on first login>` |
| Prometheus | http://localhost:9090 | N/A |
| Grafana | http://localhost:3000 | admin / `<GRAFANA_ADMIN_PASSWORD>` |
| Argo CD | http://localhost:8443 | admin / `<retrieved from K8s secret>` |
| Tetris Staging | http://tetris-staging.localhost | — |
| Tetris Production | http://tetris-production.localhost | — |

---

### Screenshots: Platform Status Post-Bootstrap

![Platform Status](/images/5-Workshop/5.2-Prerequisite/platform_status.png)
*Figure 5.2a: `make status` interface displaying all local platform services operating normally.*

![Jenkins UI Home](/images/5-Workshop/5.2-Prerequisite/jenkins_ui_home.png)
*Figure 5.2b: Jenkins homepage UI after initial login.*

![Argo CD Apps](/images/5-Workshop/5.2-Prerequisite/argocd_apps.png)
*Figure 5.2c: Argo CD UI displaying 2 Applications (staging and production) in Synced & Healthy status.*

![Make Status Success](/images/5-Workshop/5.2-Prerequisite/make_status_success.png)
*Figure 5.2d: Terminal output showing successful `make status` results after bootstrap — all containers healthy.*

---

#### 7. Common Setup Troubleshooting

| Issue | Resolution / Action |
|---|---|
| Permission denied on Docker socket | Ensure Docker Desktop is running and engine is ready |
| Port conflict on host machine | Run `lsof -nP -iTCP:<port> -sTCP:LISTEN` and stop conflicting service |
| Jenkins container unhealthy | Run `make logs SVC=jenkins`; initial image download takes several minutes |
| SonarQube not ready | Run `make logs SVC=sonarqube`; check allocated RAM and `SONAR_TOKEN` in `.env` |
| `demo-trigger` reports dirty working tree | Commit local changes first; Jenkins only builds committed code |
| AWS credentials error in Jenkins | Update `.env`, verify IAM access keys, run `docker compose restart jenkins` |
| Argo CD app `OutOfSync` | Execute `make gitops-seed` and `make argocd-apps` |
| Pod in `ImagePullBackOff` state | Check local registry (`curl http://localhost:5001/v2/_catalog`); run pipeline to mirror images |