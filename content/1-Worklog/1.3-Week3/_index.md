---
title: "Week 3 Worklog"
date: 2026-06-29
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---



### Week 3 Objectives:

* Master foundational knowledge of Security & Identity Governance on AWS.
* Complete practical hands-on labs on IAM authorization management, data encryption with KMS, and security compliance monitoring with Security Hub.
* Master the concepts of AWS Database services (RDS, Aurora, Redshift), In-memory Caching (ElastiCache), and Database Migration tools (SCT, DMS).
* Understand basic AWS Compute & Storage services (EC2, EBS, Elastic IP) and connection/resource management methods.

### Tasks to be carried out this week:
| Day | Task                                                                                                                                                                                                   | Start Date | Completion Date | Reference Material                        |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------- | --------------- | ----------------------------------------- |
| 2   | - **Security & Identity Governance** <br>&emsp; + **Responsibility & Philosophy:** Learn the cloud security boundaries (Shared Responsibility Model) between the customer and AWS; the security-first "Job Zero" philosophy. <br>&emsp; + **Access Control:** Study IAM mechanisms (Least Privilege permissions, STS Temporary Credentials) and multi-account administration (AWS Organizations SCPs, Identity Center SSO). <br>&emsp; + **App Authentication:** Survey user authentication and authorization solutions using Amazon Cognito (User Pools & Identity Pools). <br>&emsp; + **Data Protection & Compliance:** Explore data encryption techniques (Envelope Encryption) via AWS KMS and monitor infrastructure risks using Security Hub. | 29/06/2026 | 29/06/2026      | [Module 5](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%205/module-5.md) |
| 3   | - **Hands-on Labs: AWS IAM & Security Integration** <br>&emsp; + **Identity Administration (Lab 02):** Create IAM User/Group structures, configure Multi-Factor Authentication (MFA), and establish dynamic role switching (Switch Role). <br>&emsp; + **Condition Policies (Lab 44):** Configure Trust Policies restricting access based on source IP ranges and scheduling execution timeframes. <br>&emsp; + **Infrastructure Authorization (Lab 48):** Create and attach an IAM Role (Instance Profile) to an EC2 instance, removing credential exposure risks from hardcoded Access Keys. | 30/06/2026 | 30/06/2026      | [Module 5](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%205/module-5.md) |
| 4   | - **Hands-on Lab: AWS Security Compliance, Access Control & Data Encryption** <br>&emsp; + **Security Compliance Monitoring (Lab 18):** Enable AWS Security Hub combined with automatic compliance standard configuration to continuously evaluate and score system security. <br>&emsp; + **Enforcing Maximum Privilege Boundary (Lab 30):** Deploy the advanced IAM Permission Boundary feature to establish a control barrier, preventing privilege escalation risks. <br>&emsp; + **Data Encryption at Rest (Lab 33):** Initialize a Customer Managed Key (CMK) symmetric master key via AWS KMS, apply envelope encryption for Amazon S3, and configure CloudTrail – Athena to audit system API call logs. | 01/07/2026 | 01/07/2026      | [Module 5](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%205/module-5.md) |
| 5   | - **Database Services & Caching** <br>&emsp; + **Fundamentals & Classification:** Learn Database concepts (Primary/Foreign Key, Normalization, Indexing, Partitioning, Buffer). Distinguish RDBMS vs NoSQL (Fixed vs Dynamic Schema, ACID vs BASE) and OLTP vs OLAP models. <br>&emsp; + **AWS Database Services:** Explore the architecture and mechanisms of Amazon RDS (Multi-AZ synchronous replication & Read Replicas asynchronous replication), Amazon Aurora (shared Cluster Volume, 6-way replication, Backtrack, Cloning), and Amazon Redshift (MPP, Columnar Storage, Spectrum). <br>&emsp; + **Caching & Migration:** Study Amazon ElastiCache (Memcached & Redis) and developer Caching Logic; explore database migration using AWS SCT and AWS DMS. | 02/07/2026 | 02/07/2026      | [Module 6](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%206/module-6.md) |
| 6   | - **Practice: CI/CD on EKS with CodePipeline and GitHub** <br>&emsp; + **Kubernetes & EKS Overview:** Learn Amazon EKS to create, manage, run, and scale Kubernetes applications on AWS or on-premises. <br>&emsp; + **Pipeline Automation:** Study AWS CodePipeline to automate the build, test, and deployment phases whenever code changes are pushed to GitHub. <br>&emsp; + **Deployment:** Create an EKS Cluster, deploy a web application to the cluster, and configure a CI/CD pipeline for automated deployments. | 03/07/2026 | 03/07/2026      | <https://000062.awsstudygroup.com/> |


### Week 3 Achievements:

* **Security & Identity Governance**:
  * Mastered the Shared Responsibility Model and AWS's security-first "Job Zero" philosophy.
  * Gained a deep understanding of IAM mechanisms (Least Privilege permissions, STS temporary credentials) and multi-account administration (AWS Organizations SCPs, Identity Center SSO).
  * Surveyed user authentication and authorization solutions using Amazon Cognito (User Pools & Identity Pools).
* **Hands-on Labs: IAM & Security Integration**:
  * **Lab 02**: Created IAM User/Group structures, configured Multi-Factor Authentication (MFA), and established dynamic role switching (Switch Role) between environments.
  * **Lab 44**: Configured Trust Policies with conditional restrictions based on source IP ranges and scheduling execution timeframes.
  * **Lab 48**: Created and attached an IAM Role (Instance Profile) to an EC2 instance, removing credential exposure risks from hardcoded Access Keys.
* **Hands-on Labs: Security Compliance & Encryption**:
  * **Lab 18**: Enabled AWS Security Hub combined with automatic compliance standard configuration to continuously evaluate and score system security.
  * **Lab 30**: Deployed the advanced IAM Permission Boundary feature to establish a control barrier, preventing privilege escalation risks.
  * **Lab 33**: Initialized a Customer Managed Key (CMK) symmetric master key via AWS KMS, applied envelope encryption for Amazon S3, and configured CloudTrail – Athena to audit system API call logs.
* **Database Services & Caching**:
  * Gained a clear understanding of database design concepts (Primary/Foreign Key, Normalization, Indexing, Partitioning, Database Buffer).
  * Distinguished between RDBMS (Fixed Schema, ACID) and NoSQL (Dynamic Schema, BASE) architectures, as well as OLTP (transactional) and OLAP (analytical) processing models.
  * Mastered the design and features of **Amazon RDS** (Multi-AZ synchronous replication and Read Replicas asynchronous replication) and **Amazon Aurora** (shared Cluster Volume, 6-way replication across 3 AZs, Backtrack, Cloning, Global DB).
  * Explored massive scale data warehousing with **Amazon Redshift** (MPP architecture, Columnar Storage, Redshift Spectrum, Data Sharing).
  * Understood the role of **Amazon ElastiCache** (Memcached/Redis) in reducing database load and the developer's responsibility for cache invalidation and logic.
  * Learnt cloud migration workflows utilizing **AWS SCT** (Schema Conversion Tool) and **AWS DMS** (Database Migration Service).
* **CI/CD on EKS with CodePipeline and GitHub**:
  * Gained foundational knowledge of Amazon Elastic Kubernetes Service (Amazon EKS) and container orchestration management for Kubernetes applications.
  * Understood the operational mechanisms of AWS CodePipeline in automating deployment pipelines to securely and stably update applications or infrastructure based on GitHub changes.
  * Practiced creating a simple EKS Cluster, deploying a web application, and configuring a CI/CD pipeline to automate production deployments.
