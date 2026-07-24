---
title: "Week 7 Worklog"
date: 2026-07-27
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Week 7 Objectives:

* Set up and review logs/metrics monitoring for the EKS cluster using CloudWatch Container Insights.
* Gather monitoring screenshots, cost alarms (AWS Budget), and draft the Observability documentation.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - **CloudWatch Observability Research:** <br>&emsp; + Learn about Amazon CloudWatch capabilities and Container Insights features on AWS. <br>&emsp; + Study log forwarding using Fluent Bit daemon and resource metric ingestion via the CloudWatch agent on EKS nodes. | 27/07/2026 | 27/07/2026 | [CloudWatch Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights.html) |
| 3 | - **Container Insights Configuration:** <br>&emsp; + Coordinate with Member 1 to configure and deploy the CloudWatch Observability add-on on the EKS cluster. <br>&emsp; + Confirm EKS log groups (cluster, node, pod logs) are successfully streamed to the AWS console. | 28/07/2026 | 28/07/2026 | [Observability Config](file:///d:/AWS%20FCAJ/CICD-DevSecOps-using-AWS-services/tasks.md) |
| 4 | - **Log Queries & Dashboards:** <br>&emsp; + Build CloudWatch Logs Insights queries to filter and analyze application-specific container logs (e.g. error tracking, HTTP status aggregation). <br>&emsp; + Assist in building visualizations for CPU/Memory and network throughput metrics. | 29/07/2026 | 29/07/2026 | [Observability Module](https://github.com/namcris07/aws-fcaj-learning/blob/main/Worklogs/Tu%E1%BA%A7n-07-CloudWatch-Observability.md) |
| 5 | - **Gather Monitoring Capture:** <br>&emsp; + Capture screenshots of active Container Insights dashboards showing EKS node and pod metrics. <br>&emsp; + Capture screenshots of AWS Budget configuration and active cost warnings set at 50%, 80%, and 100% thresholds. | 30/07/2026 | 30/07/2026 | [Observability Dashboard](file:///d:/AWS%20FCAJ/CICD-DevSecOps-using-AWS-services/README.md) |
| 6 | - **Observability Documentation:** <br>&emsp; + Write the detailed Observability section (monitoring mechanisms, screenshots) in the core project documentation. <br>&emsp; + Formulate cloud resources cost optimization and decommissioning procedures post-project. <br>&emsp; + Review and evaluate Week 7 progress. | 31/07/2026 | 31/07/2026 | [Observability Section](file:///d:/AWS%20FCAJ/CICD-DevSecOps-using-AWS-services/tasks.md) |

### Week 7 Achievements:

* **Established CloudWatch Observability Framework:**
  * Enabled and configured CloudWatch Container Insights on EKS for performance monitoring of worker nodes and pods.
  * Designed custom Logs Insights queries and dashboard widgets for CPU/Memory metrics.

* **Completed Observability Artifacts & Cost Governance:**
  * Collected screenshots of CloudWatch dashboards, EKS log streams, and AWS Budget alerts.
  * Finalized the Observability segment of the project report and established clean-up steps to avoid billing overhead.
