---
title: "Week 6 Worklog"
date: 2026-07-20
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Week 6 Objectives:

* Understand GitOps principles and Argo CD operations.
* Build QA testing procedures for GitOps auto-sync and the manual approval gate.
* Capture visual evidence of Argo CD staging and production applications.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - **GitOps & Argo CD Research:** <br>&emsp; + Study pull-based GitOps principles on Kubernetes using Argo CD. <br>&emsp; + Analyze Argo CD Application CRD (Custom Resource Definition) structure for environment management. <br>&emsp; + Compare synchronization strategies (Manual Sync vs Automated Sync). | 20/07/2026 | 20/07/2026 | [GitOps Module](https://github.com/namcris07/aws-fcaj-learning/blob/main/Worklogs/Tu%E1%BA%A7n-06-GitOps-ArgoCD.md) |
| 3 | - **Design Staging Auto-Sync QA:** <br>&emsp; + Design test cases for automated sync from Git manifest repo to EKS staging namespace when Jenkins pushes a new commit SHA tag to Git. <br>&emsp; + Define Synced (manifest matches Git) and Healthy (application operational) status criteria. | 21/07/2026 | 21/07/2026 | [Argo CD Documentation](https://argoproj.github.io/cd/) |
| 4 | - **Manual Approval Gate Workflow:** <br>&emsp; + Document verification procedures for the manual approval gate in the Jenkins pipeline to promote builds to Production. <br>&emsp; + Establish safety verification test scenarios to block untested code from production. | 22/07/2026 | 22/07/2026 | [Jenkins Approval](https://github.com/loi-bui0703/CICD-DevSecOps-using-AWS-services/blob/main/tasks.md) |
| 5 | - **Gather GitOps Capture:** <br>&emsp; + Capture screenshots of Argo CD application status (Synced & Healthy) for both environments. <br>&emsp; + Record synchronization logs from the Argo CD server and capture Git bump actions inside Jenkins. | 23/07/2026 | 23/07/2026 | [Argo CD UI](https://github.com/loi-bui0703/CICD-DevSecOps-using-AWS-services/blob/main/README.md) |
| 6 | - **QA Summary & Weekly Review:** <br>&emsp; + Consolidate the GitOps testing checklist. <br>&emsp; + Review the security configurations of Git credentials (SSH Keys/Personal Access Tokens) used in Jenkins. <br>&emsp; + Review and evaluate Week 6 deliverables. | 24/07/2026 | 24/07/2026 | [Test Checklist](https://github.com/loi-bui0703/CICD-DevSecOps-using-AWS-services/blob/main/tasks.md) |

### Week 6 Achievements:

* **Mastered Continuous Delivery with GitOps:**
  * Gained deep understanding of pull-based GitOps deployment and Argo CD manifest sync from Git repositories to Kubernetes.
  * Evaluated GitOps advantages over traditional CD pipelines in preventing Configuration Drift.

* **Built Quality Control & Release Promotion Workflow:**
  * Designed and validated test cases for Staging Auto-Sync and Production Manual Approval Gate workflows.
  * Captured evidence of Argo CD UI displaying Synced and Healthy application states.
