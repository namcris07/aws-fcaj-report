---
title: "Translated Technical Blogs"
date: 2026-07-06
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

# Translated Technical Blogs on AWS Study Group

Below is the summary and introduction of technical blogs translated and published on the **[AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj)** community:

---

### 1. [Blog 1 - Building a Serverless Jenkins Environment on AWS Fargate](3.1-Blog1/)
**Author:** Krithivasan Balasubramaniyan | **Date:** March 26, 2021

This article provides a step-by-step guide to provisioning a fully serverless Jenkins Controller-Agent environment on **Amazon ECS Fargate** using **Terraform**. The architecture leverages **Amazon EFS** as persistent storage for the Jenkins Controller configuration and utilizes **Fargate Spot Capacity Providers** to launch ephemeral Jenkins Build Agents on demand, drastically reducing infrastructure costs while ensuring high availability.

---

### 2. [Blog 2 - Building an End-to-End Kubernetes-based DevSecOps Software Factory on AWS](3.2-Blog2/)
**Author:** Srinivas Manepalli | **Date:** June 25, 2021

This article presents a comprehensive reference architecture for an end-to-end DevSecOps software factory on **Amazon EKS** enforcing the **Shift-Left Security** principle. The CI/CD pipeline integrates AWS Cloud-Native services (*AWS CodePipeline, CodeBuild, CodeCommit, Security Hub*) alongside open-source security tools (*git-secrets, Snyk/Anchore, ECR Image Scan, OWASP ZAP, Sysdig Falco*) across 6 security scan layers (Secrets, SCA, SAST, DAST, RASP), aggregating findings into **AWS Security Hub**.

---

### 3. [Blog 3 - Monitoring and Auto Scaling Amazon ECS Applications on AWS Fargate Using Prometheus Metrics](3.3-Blog3/)
**Author:** Mike George | **Date:** February 05, 2021

This article demonstrates how to enhance application observability for containerized Java/Tomcat applications running on **Amazon ECS Fargate** using **CloudWatch Container Insights** and **Prometheus Exporters**. By collecting deep application-level metrics (*JVM memory, active connection counts, garbage collection*), it walks through configuring **CloudWatch Alarms** and **ECS Service Auto Scaling** to automatically scale tasks in and out dynamically based on real-time traffic demand.