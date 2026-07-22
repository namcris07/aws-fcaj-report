---
title: "Week 6 Worklog"
date: 2026-07-20
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Week 6 Objectives:

* Understand GitOps principles and Argo CD operations.
* Establish verification processes for GitOps auto-sync and manual approval gates.
* Capture Argo CD staging/production application status screenshots.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - **GitOps & Argo CD Research:** <br>&emsp; + Learn GitOps principles and pull-based deployments on Kubernetes. <br>&emsp; + Study Argo CD Application CRD (Custom Resource Definition) specifications for multi-environment configurations. <br>&emsp; + Compare deployment sync strategies (Manual Sync vs Automated Sync). | 20/07/2026 | 20/07/2026 | [GitOps Guides](https://000062.awsstudygroup.com/) |
| 3 | - **Staging Auto-Sync QA Design:** <br>&emsp; + Design test cases for validating Argo CD automated synchronization (Auto-sync) from Git to EKS staging namespace when Jenkins pushes new image tags. <br>&emsp; + Establish criteria for Synced (manifest matching) and Healthy (running without crashes) application status. | 21/07/2026 | 21/07/2026 | [Argo CD Documentation](https://argoproj.github.io/cd/) |
| 4 | - **Manual Approval Gate Workflow:** <br>&emsp; + Document verification procedures for the manual approval gate in the Jenkins pipeline to promote builds to Production. <br>&emsp; + Establish safety verification test scenarios to block untested code from production. | 22/07/2026 | 22/07/2026 | [Jenkins Approval](file:///d:/AWS%20FCAJ/CICD-DevSecOps-using-AWS-services/tasks.md) |
| 5 | - **Gather GitOps Capture:** <br>&emsp; + Capture screenshots of Argo CD application status (Synced & Healthy) for both environments. <br>&emsp; + Record synchronization logs from the Argo CD server and capture Git bump actions inside Jenkins. | 23/07/2026 | 23/07/2026 | [Argo CD UI](file:///d:/AWS%20FCAJ/CICD-DevSecOps-using-AWS-services/README.md) |
| 6 | - **QA Summary & Weekly Review:** <br>&emsp; + Consolidate the GitOps testing checklist. <br>&emsp; + Review the security configurations of Git credentials (SSH Keys/Personal Access Tokens) used in Jenkins. <br>&emsp; + Review and evaluate Week 6 deliverables. | 24/07/2026 | 24/07/2026 | [Test Checklist](file:///d:/AWS%20FCAJ/CICD-DevSecOps-using-AWS-services/tasks.md) |

### Week 6 Achievements:

* **Mastered GitOps Continuous Delivery Workflows:**
  * Acquired in-depth knowledge of pull-based deployment paradigms and manifest synchronization via Argo CD on EKS.
  * Evaluated GitOps advantages over traditional push CD in preventing Configuration Drift.

* **Established Safety & QA Controls:**
  * Successfully designed and verified test cases for Staging Auto-Sync and Production Manual Approval Gates.
  * Captured clear screenshots of Argo CD displaying Synced and Healthy states for both environments, ready for the final report.
  * Ensured secure storage of Git credentials in Jenkins Credentials Provider without leaking secrets.
