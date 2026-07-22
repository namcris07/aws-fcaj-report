---
title: "Week 9 Worklog"
date: 2026-08-10
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Week 9 Objectives:

* Initialize the Workshop Website using the Hugo framework (`FCAJ-workshop-template`), establishing navigation menus and bilingual (VI/EN) layout structures.
* Integrate all project deliverables (Student Info, Proposal, 9-week Worklog, 3 Blog posts, Events Participated, Self-Evaluation & Sharing Feedback) onto the website.
* Conduct cross-bilingual audits (VI/EN), test link integrity and visual assets, run production build (`hugo --minify`), submit final deliverables, and perform complete AWS resource teardown (EKS cluster, Load Balancer, ECR) for cost optimization.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - **Initialize Hugo Site & Menu Navigation:** <br>&emsp; + Study Hugo project layouts and spin up local website preview via `hugo server`. <br>&emsp; + Configure global settings in `config.toml` and set weight ordering for all menu sections (Student Info, Worklog, Proposal, Blogs, Events, Self-evaluation, Sharing & Feedback). | 10/08/2026 | 10/08/2026 | [FCAJ Workshop Template](file:///d:/AWS%20FCAJ/fcj-workshop-template/) |
| 3 | - **Integrate Info, Proposal & Worklog Content:** <br>&emsp; + Format and publish Student Information, Project Proposal, and the complete 9-week Worklog (Weeks 1 to 9) inside `content/`. <br>&emsp; + Embed high-resolution architecture diagrams and CI/CD pipeline flowcharts. | 11/08/2026 | 11/08/2026 | [Website Content Map](file:///d:/AWS%20FCAJ/fcj-workshop-template/content/) |
| 4 | - **Incorporate Blogs, Events & Evaluation Pages:** <br>&emsp; + Publish 3 technical Blog posts (IAM/VPC, EKS CI/CD, CloudWatch/Cost) with live article links and publication screenshots under Blogs. <br>&emsp; + Update Events Participated section with extracurricular activity details and visual evidence. <br>&emsp; + Gather and publish team Self-evaluation & Sharing Feedback responses. | 12/08/2026 | 12/08/2026 | [Blogs & Events](file:///d:/AWS%20FCAJ/fcj-workshop-template/content/4-EventParticipated/) |
| 5 | - **Bilingual Audit & UI/Link Verification:** <br>&emsp; + Cross-audit Vietnamese and English translations across all pages to guarantee technical accuracy and natural phrasing. <br>&emsp; + Verify all internal/external hyperlinks, shortcode components, tables, and responsive mobile UI layouts. | 13/08/2026 | 13/08/2026 | [Bilingual Review](file:///d:/AWS%20FCAJ/fcj-workshop-template/config.toml) |
| 6 | - **Production Build, Final Handover & AWS Teardown:** <br>&emsp; + Execute production build command (`hugo --minify`) and verify zero rendering flaws or broken links. <br>&emsp; + Package and submit final project deliverables and presentation slides to program committee. <br>&emsp; + Perform comprehensive teardown of AWS cloud resources (delete EKS cluster, ALB Ingress, ECR images, VPC resources) to stop all recurring charges. | 14/08/2026 | 14/08/2026 | [Final Handover](file:///d:/AWS%20FCAJ/CICD-DevSecOps-using-AWS-services/tasks.md) |

### Week 9 Achievements:

* **Finalized Project Workshop Website:**
  * Successfully built the project workshop website on the Hugo framework with modern styling, SEO best practices, and complete bilingual support.
  * Consolidated 100% of project assets: Team Info, Proposal, 9-week Worklogs, Step-by-step Workshop Labs, 3 Blog posts, Events, Self-evaluation, and Feedback.

* **Project Acceptance & AWS Cloud Teardown:**
  * Verified UI responsiveness and compiled clean production builds; delivered final project submission on deadline by August 14, 2026.
  * Fully deleted active AWS resources (EKS cluster, Load Balancers, ECR repos), eliminating any risk of unforeseen billing.
