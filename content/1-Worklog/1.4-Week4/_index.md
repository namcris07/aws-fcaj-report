---
title: "Week 4 Worklog"
date: 2026-07-06
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Week 4 Objectives:

* Study and understand Docker Image storage on Amazon ECR.
* Coordinate the creation of image delivery flow and tag management documentation.
* Capture ECR screenshots for reporting.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - **Amazon ECR Research & Authentication:** <br>&emsp; + Learn about Amazon ECR (Elastic Container Registry) storage capabilities on AWS. <br>&emsp; + Differentiate between Public and Private Repositories; study automated Scan on Push security features. <br>&emsp; + Study secure IAM authentication methods using `aws ecr get-login-password` via AWS CLI to log in the docker client. | 06/07/2026 | 06/07/2026 | [ECR Module](https://github.com/namcris07/aws-fcaj-learning/blob/main/Worklogs/Tu%E1%BA%A7n-04-Amazon-ECR.md) |
| 3 | - **Packaging & Storage Coordination:** <br>&emsp; + Assist team members in aligning Dockerfile directory structures and Docker registry settings. <br>&emsp; + Coordinate with Member 1 and Member 4 to draft the automated Image Delivery Flow (from Jenkins CI, through Trivy security scans, to Amazon ECR). <br>&emsp; + Define required environment variables: `REGISTRY`, `REPO_NAME`, `IMAGE_NAME`, `IMAGE_TAG`. | 07/07/2026 | 07/07/2026 | [ECR Module](https://github.com/namcris07/aws-fcaj-learning/blob/main/Worklogs/Tu%E1%BA%A7n-04-Amazon-ECR.md) |
| 4 | - **CI/CD ECR Documentation:** <br>&emsp; + Document ECR configurations and draft image tagging instructions based on dynamic Git Commit SHAs to prevent overwriting old builds. <br>&emsp; + Detail configuration parameters in `ci/Jenkinsfile` and explain ECR credential integration inside Jenkins Credentials Provider. | 08/07/2026 | 08/07/2026 | [ECR Module](https://github.com/namcris07/aws-fcaj-learning/blob/main/Worklogs/Tu%E1%BA%A7n-04-Amazon-ECR.md) |
| 5 | - **Report Updates & Capture:** <br>&emsp; + Capture screenshots of ECR repository operations in the AWS Console. <br>&emsp; + Log ECR image scan findings and vulnerability summaries. <br>&emsp; + Update the "AWS Image Registry" section in the project documentation; update project folder structures related to `ci/` and ECR. | 09/07/2026 | 09/07/2026 | [ECR Module](https://github.com/namcris07/aws-fcaj-learning/blob/main/Worklogs/Tu%E1%BA%A7n-04-Amazon-ECR.md) |
| 6 | - **ECR Checklist & Review:** <br>&emsp; + Formulate a test checklist for verifying ECR registry readiness (testing login, pull, push from local and Jenkins agent). <br>&emsp; + Review and evaluate Week 4 deliverables, preparing for the EKS deployment phase. | 10/07/2026 | 10/07/2026 | [ECR Module](https://github.com/namcris07/aws-fcaj-learning/blob/main/Worklogs/Tu%E1%BA%A7n-04-Amazon-ECR.md) |

### Week 4 Achievements:

* **Mastered Amazon ECR Service & Permission Management:**
  * Acquired in-depth knowledge of Amazon ECR operations, access permissions (IAM Policies, ECR Repository Policies), and secure image storage.
  * Clarified the difference between IAM Identity Center SSO authentication and temporary token generation via AWS CLI for Docker daemon login.
  
* **Finalized Documentation & Automation Workflows:**
  * Collaboratively designed the automated image delivery flow diagram from Jenkins CI to Amazon ECR.
  * Aligned environment variables and image paths in the `Jenkinsfile` to block image overwrite risks in ECR.
  * Completed documentation and captured screenshots demonstrating ECR Repository operations with image tags mapped to Git Commit SHAs.
