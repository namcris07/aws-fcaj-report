---
title: "Week 5 Worklog"
date: 2026-07-13
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Week 5 Objectives:

* Understand application deployment on Kubernetes/EKS.
* Create test cases for the deployment pipeline (Pod status, Ingress routing).
* Gather EKS infrastructure configuration details from Member 1.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - **EKS & ALB Controller Research:** <br>&emsp; + Learn about Amazon Elastic Kubernetes Service (EKS) container orchestration architecture. <br>&emsp; + Study OIDC Provider mechanisms for EKS Service Accounts. <br>&emsp; + Study AWS Load Balancer Controller operations integrated with ALB (Application Load Balancer) to auto-provision Layer 7 Ingress. | 13/07/2026 | 13/07/2026 | [EKS & K8s](https://000062.awsstudygroup.com/) |
| 3 | - **EKS Deployment QA Design:** <br>&emsp; + Design detailed test case documents for deploying the web application on Kubernetes/EKS. <br>&emsp; + List testing scenarios for Pod states (Running, Ready, CrashLoopBackOff), self-healing capabilities (Liveness/Readiness Probes), Service networking, and Ingress routing. | 14/07/2026 | 14/07/2026 | [QA Guides](https://github.com/namcris07/aws-fcaj-learning/) |
| 4 | - **EKS Infrastructure Coordination:** <br>&emsp; + Coordinate with Member 1 to record EKS node group settings (instance type, scaling limits), VPC subnets, and IAM role parameters. <br>&emsp; + Document Kubernetes infrastructure configurations to match actual environments created via Terraform/AWS CLI. | 15/07/2026 | 15/07/2026 | [EKS Configuration](file:///d:/AWS%20FCAJ/CICD-DevSecOps-using-AWS-services/tasks.md) |
| 5 | - **Gather Deployment Capture:** <br>&emsp; + Capture screenshots of Ingress ALB and Services running on EKS. <br>&emsp; + Run and capture terminal commands like `kubectl get nodes` and `kubectl get pods -A`. <br>&emsp; + Log errors during staging deployment (e.g. mismatched ports, pull image errors, or missing Ingress annotations). | 16/07/2026 | 16/07/2026 | [EKS Deployment](file:///d:/AWS%20FCAJ/CICD-DevSecOps-using-AWS-services/README.md) |
| 6 | - **QA Sign-off & Weekly Review:** <br>&emsp; + Finalize the test checklist and deployment verification reports on EKS staging. <br>&emsp; + Cross-check container ports and Service/Ingress port declarations (8080 vs 80). <br>&emsp; + Review and evaluate Week 5 progress. | 17/07/2026 | 17/07/2026 | [Test Case Templates](file:///d:/AWS%20FCAJ/CICD-DevSecOps-using-AWS-services/tasks.md) |

### Week 5 Achievements:

* **Mastered EKS Infrastructure Operations:**
  * Acquired in-depth knowledge of EKS cluster networking, node groups, and fine-grained permissions via IAM Roles for Service Accounts (IRSA).
  * Mastered Ingress Controller mechanics and how the AWS Load Balancer Controller translates K8s Ingress objects to actual Application Load Balancer resources.

* **Established Infrastructure QA Workflows:**
  * Developed a comprehensive test case suite for validating web applications deployed on EKS.
  * Resolved deployment port conflicts (mismatched 8080 app port vs 80 service port) in collaboration with the team.
  * Captured screenshots showing Ready EKS Nodes and successful ALB creation on AWS Console.
