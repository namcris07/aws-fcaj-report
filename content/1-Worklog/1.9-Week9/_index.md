---
title: "Week 9 Worklog"
date: 2026-08-10
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Week 9 Objectives:

* Initialize the Workshop Website repository using the Hugo framework (`FCAJ-workshop-template`), establishing custom navigation menus and a bilingual (VI/EN) layout.
* Integrate 100% of project content (Student Information, Proposal, 9-week Worklogs, 3 Technical Blogs, Events Participated, Self-evaluation & Sharing Feedback) into the site structure.
* Conduct bilingual audits (VI/EN), verify all hyperlinks and rendering components, generate the production site build (`hugo --minify`), hand over final deliverables, scale ECS Fargate services to 0 (`desired-count 0`), and complete AWS cloud resource decommissioning (ECR, S3, ALB) to eliminate recurring costs.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - **Initialize Hugo Site & Menu Navigation:** <br>&emsp; + Study Hugo project layouts and spin up local website preview via `hugo server`. <br>&emsp; + Configure global settings in `config.toml` and set weight ordering for all menu sections (Student Info, Worklog, Proposal, Blogs, Events, Self-evaluation, Sharing & Feedback). | 10/08/2026 | 10/08/2026 | [FCAJ Workshop Template](https://github.com/namcris07/aws-fcaj-learning/blob/main/Worklogs/Tu%E1%BA%A7n-09-Hugo-Website-Handover.md) |
| 3 | - **Integrate Info, Proposal & Worklog Content:** <br>&emsp; + Format and publish Student Information, Project Proposal, and the complete 9-week Worklog (Weeks 1 to 9) inside `content/`. <br>&emsp; + Embed high-resolution Serverless DevSecOps architecture diagrams and CI/CD pipeline flowcharts. | 11/08/2026 | 11/08/2026 | [Website Content Map](/1-worklog/) |
| 4 | - **Incorporate Blogs, Events & Evaluation Pages:** <br>&emsp; + Publish 3 technical Blog posts (IAM/VPC/ECR, Jenkins CI/CD & ECS Fargate, CloudWatch/Cost & Prometheus/Grafana) with live article links and publication screenshots under Blogs. <br>&emsp; + Update Events Participated section with extracurricular activity details and visual evidence. <br>&emsp; + Gather and publish team Self-evaluation & Sharing Feedback responses. | 12/08/2026 | 12/08/2026 | [Blogs & Events](/4-eventparticipated/) |
| 5 | - **Bilingual Audit & UI/Link Verification:** <br>&emsp; + Cross-audit Vietnamese and English translations across all pages to guarantee technical accuracy and natural phrasing. <br>&emsp; + Verify all internal/external hyperlinks, shortcode components, tables, and responsive mobile UI layouts. | 13/08/2026 | 13/08/2026 | [Bilingual Review](https://github.com/namcris07/aws-fcaj-report/blob/main/config.toml) |
| 6 | - **Production Build, Final Handover & AWS Teardown:** <br>&emsp; + Execute production build command (`hugo --minify`) and verify zero rendering flaws or broken links. <br>&emsp; + Package and submit final project deliverables and presentation slides to program committee. <br>&emsp; + Scale ECS Fargate services to 0 (`desired-count 0`) and perform comprehensive teardown of AWS cloud resources (delete ALB Ingress, ECR images, S3 report buckets, VPC resources) to eliminate recurring charges. | 14/08/2026 | 14/08/2026 | [Final Handover](https://github.com/loi-bui0703/CICD-DevSecOps-using-AWS-services/blob/main/tasks.md) |

### Week 9 Achievements:

* **Finalized Capstone Workshop Report Website:**
  * Successfully built the capstone project workshop website on Hugo, featuring clean typography, SEO optimization, and complete bilingual (VI/EN) support.
  * Integrated 100% of project assets: Team Info, Proposal, 9-week Worklogs, Step-by-step Workshop Labs, 3 Blog posts, Events, Self-evaluation, and Feedback.

* **Project Acceptance & Cloud Resource Decommissioning:**
  * Verified responsive UI layout, generated production build (`hugo --minify`) with zero broken links, and packaged final handover materials by August 14, 2026.
  * Executed comprehensive teardown of AWS resources (scaled ECS Fargate tasks to 0, removed ALBs, ECR repos), eliminating surprise cloud costs.
