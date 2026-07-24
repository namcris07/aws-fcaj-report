---
title: "Week 7 Worklog"
date: 2026-07-27
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Week 7 Objectives:

* Configure and inspect logs/metrics for the **Amazon ECS Fargate** cluster using **CloudWatch Container Insights**.
* Aggregate monitoring screenshots, cost alert setups (AWS Budgets), and compile the Observability documentation section.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - **CloudWatch Observability Research for ECS:** <br>&emsp; + Learn about Amazon CloudWatch capabilities and Container Insights features for Amazon ECS Fargate. <br>&emsp; + Study automatic container log streaming via the `awslogs` log driver directly to CloudWatch Log Groups. | 27/07/2026 | 27/07/2026 | [CloudWatch Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights.html) |
| 3 | - **Container Insights Configuration for ECS Cluster:** <br>&emsp; + Coordinate with Member 1 to enable Container Insights on ECS Cluster `devsecops-factory` via `aws ecs update-cluster-settings`. <br>&emsp; + Confirm ECS task, service, and container log groups are successfully streamed to the AWS console. | 28/07/2026 | 28/07/2026 | [Observability Config](file:///d:/AWS%20FCAJ/CICD-DevSecOps-using-AWS-services/tasks.md) |
| 4 | - **Log Queries & Dashboards:** <br>&emsp; + Build CloudWatch Logs Insights queries to filter and analyze application-specific container logs (e.g. error tracking, HTTP status aggregation). <br>&emsp; + Assist in building visualizations for ECS Task CPU/Memory and network throughput metrics. | 29/07/2026 | 29/07/2026 | [Observability Module](https://github.com/namcris07/aws-fcaj-learning/blob/main/Worklogs/Tu%E1%BA%A7n-07-CloudWatch-Observability.md) |
| 5 | - **Gather Monitoring Capture:** <br>&emsp; + Capture screenshots of active Container Insights dashboards showing ECS Task metrics. <br>&emsp; + Capture screenshots of AWS Budget configuration and active cost warnings set at 50%, 80%, and 100% thresholds. | 30/07/2026 | 30/07/2026 | [Observability Dashboard](file:///d:/AWS%20FCAJ/CICD-DevSecOps-using-AWS-services/README.md) |
| 6 | - **Observability & Teardown Documentation:** <br>&emsp; + Write the detailed Observability section (monitoring mechanisms, screenshots) in the core project documentation. <br>&emsp; + Formulate cloud resources cost optimization and ECS Fargate scaling down to `0` (`desired-count 0`) procedures post-project. <br>&emsp; + Review and evaluate Week 7 progress. | 31/07/2026 | 31/07/2026 | [Observability Section](file:///d:/AWS%20FCAJ/CICD-DevSecOps-using-AWS-services/tasks.md) |

### Week 7 Achievements:

* **Established Centralized CloudWatch Observability for ECS Fargate:**
  * Enabled and configured CloudWatch Container Insights on the ECS Cluster to collect CPU and Memory utilization metrics across ECS Tasks.
  * Created structured log queries and interactive dashboards on the CloudWatch Console.

* **Finalized Observability Deliverables & Cost Management:**
  * Gathered comprehensive screenshots of CloudWatch Dashboards, Log Groups, and AWS Budget threshold alerts.
  * Completed the Observability documentation section and finalized resource decommissioning plans.
