---
title: "Event 3"
date: 2026-07-22
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

# SUMMARY REPORT: AWS TECH MEETUP & COMMUNITY KNOWLEDGE SHARING

## Event Objectives

- **Objectives:**
  - Present in-depth topics on infrastructure, system operations, automated monitoring, and web application security on the AWS platform.
  - Guide the roadmap to conquer the international AWS Certified Cloud Practitioner (CLF-C02) certificate for technology personnel who are just starting out.
  - Introduce trends in applying Generative AI and AI Agents in automated security testing workflows (DevSecOps/Pentesting).
  - Create a forum to exchange hands-on experiences between infrastructure engineers, security experts, and members of the AWS First Cloud AI Journey community.

## Speakers

- **Speakers:**
  - **Nguyen Huynh Son:** Infrastructure Support Engineer at Endava, Ex-Infrastructure Reliability Engineer at SPS, Member of AWS Student Builder Group HUFLIT.
  - **Ngo Le Tan Huy:** Speaker sharing the AWS Cloud Practitioner certification roadmap.
  - **Nguyen Tuan Thinh:** DevOps/DevSecOps/Cloud Engineer at Styl Solutions, Member of First Cloud AI Journey.

## Key Highlights

### Topic 1: SLA and Monitoring – From SLA to Monitoring What Really Matters (Speaker: Nguyen Huynh Son)

- **SLA and Risk Management Essence:**
  - **SLA (Service Level Agreement):** The official commitment on service levels between the provider and customers.
  - **Monitoring:** Part of the risk management process to detect potential risks early before the SLA is breached.
  - **Risk management loop:** Identify risk → Monitor signals (via metrics, logs, alarms) → Respond (via SNS, SOP) → Improve (Review and optimize).
- **"Healthy Infrastructure ≠ Healthy User Experience" Reality:**
  - **Monitoring Pyramid:** Hierarchical classification from bottom to top: Cloud Provider (EC2, RDS, ALB) → Infrastructure (CPU, Memory, Disk) → Application (Latency, Errors) → Business (Login success, Orders) → Customer Experience (Ability to log in, pay).
  - **Real-world risk:** The system may report a "Green" status (CPU 18%, ALB Target 2/2, Healthcheck `/health` returns `200 OK`), but real users experience login failures (Login Failure) because the RDS database connection failed. Monitoring pure infrastructure at the lower layer does not guarantee the actual user experience.
- **Automated Alerting Flow:**
  - Set up custom metrics for user actions (such as `LoginFailure`).
  - Trigger a CloudWatch Alarm when thresholds are exceeded.
  - Send notifications via SNS Topic to the technical team's Email/Slack before receiving customer complaints.
- **System Operation Philosophy:**
  - Keep in mind the quote from Dr. Werner Vogels (CTO Amazon): *"Everything fails all the time, so plan for failure and nothing fails"*. AWS ensures cloud infrastructure, but the engineer is fully responsible for the user experience.

### Topic 2: Inside The Exam – AWS Certified Cloud Practitioner CLF-C02 (Speaker: Ngo Le Tan Huy)

- **Overview of CLF-C02 Exam Structure:**
  - **Structure:** 65 multiple-choice questions in 90 minutes (additional 30 minutes for non-native English speakers).
  - **Passing score:** 700/1000, certification validity: 3 years.
  - **Weight of 4 Knowledge Domains:**
    - Domain 1: Cloud Concepts (24%)
    - Domain 2: Security and Compliance (30%)
    - Domain 3: Cloud Technology and Services (34%)
    - Domain 4: Billing, Pricing, and Support (12%)
- **Shared Responsibility Model:**
  - **AWS is responsible for:** "Security OF the Cloud" (Physical infrastructure, hardware, data centers).
  - **Customer is responsible for:** "Security IN the Cloud" (Data, IAM, firewall/Security Group configurations, OS patches).
- **Preparation Strategy & Exam Tips & Tricks:**
  - **Keyword Mapping Mindset:** Connect each AWS service with 1-2 practical keywords (e.g., "Decouple/Microservices" → choose SQS; "Data monetization" → choose Business Perspective in AWS CAF).
  - **Review Mistakes:** In mock tests, focus on analyzing why option A is correct and why B, C, D are wrong to avoid exam traps.
  - **Elimination & Simplification:** Eliminate fabricated or irrelevant services to increase the correct answer rate to 50%. Choose the simplest solution, avoiding overcomplicating things since Cloud Practitioner focuses on high-level concepts.

### Topic 3: Securing Your Web Apps With AWS Security Agent (Speaker: Nguyen Tuan Thinh)

- **Security Bottleneck in Traditional Web App Security:**
  - Manual penetration testing (Pentest) takes weeks, is expensive ($5k - $20k per assessment), and has inconsistent results depending on the expert's skill and mood.
- **AWS Security Agent Solution (Frontier Agent):**
  - Powered by Amazon Bedrock, capable of autonomous planning and execution of security tasks without human manual intervention.
  - **Significant difference from normal LLM chatbots:** Not just limited to theoretical alerts, it automatically performs real-world exploit behaviors (Verifiable Findings) to prove vulnerabilities.
- **3-Phase Comprehensive Security (Full Lifecycle):**
  - **Design Review:** Analyze architectural documents (Markdown) or Infrastructure as Code (Terraform) before coding, cross-referencing with Managed Packs (PCI DSS, NIST CSF, AWS Well-Architected).
  - **Code Review:** Integrate directly into GitHub/GitLab Pull Requests, automatically commenting on each line of erroneous code and suggesting automated code fixes (Auto-PR Fixes).
  - **Automated Pentesting:** Act as an agent proactively attacking running applications with multi-step exploit chains (e.g., IDOR → XSS) and exporting detailed attack diagrams.
- **Cost & Technical Limits:**
  - **Cost comparison:** Pay-as-you-go pricing ($50/Task-Hour) helps reduce security testing costs from $10,000 (traditional pentest teams) to $1,500 - $2,500 per actual project.
  - **Practical limits:** The agent will be hindered by rigid authentication mechanisms like MFA, Biometrics, mTLS, and struggles to detect complex business-logic fraud.

## Comparative Discussion

Based on data and discussion panels at the event, below is a detailed comparative table of operational and security efficiency between traditional and modern solutions:

| Criteria | Traditional / Manual Model | Cloud-based & AI Solutions |
| :--- | :--- | :--- |
| **System Monitoring** | Focuses on infrastructure (CPU/RAM/Disk). System reports "Green" but users still get errors (RDS connection fail). | User-centric. Tracks Custom Metrics of key business flows (Login success, Orders) via CloudWatch. |
| **Incident Response & Alerting** | Delayed detection upon receiving complaints from customers. | Automated alerting via CloudWatch Alarm & SNS Topic sent directly to Slack/Email before customers notice. |
| **Application Security (Pentest)** | Manual (takes weeks), expensive ($5k - $20k/run), heavily depends on expert skills. | Automated via AWS Security Agent, detects and proposes automated code fixes (Auto-PR Fixes) in GitHub/GitLab PRs. |
| **Security Testing Costs** | Very high (~ $10,000/project) and difficult to perform regularly. | Optimized (only $1,500 - $2,500/project) thanks to the Pay-as-you-go pricing model. |
| **Exam Strategy** | Rote learning, purely theoretical, easily confuses services. | Keyword Mapping (associate service with keyword), review mistakes during mock tests. |

## Lessons Learned

### Design Mindset

- **User-centric Monitoring:** Do not evaluate system health by server utilization (CPU/RAM), but measure it by the success rate of core business flows (User Journey).
- **Shift-Left Security:** Integrate automated security testing from the architecture design phase (Design Review) and coding phase (Code Security Review), instead of waiting until the application is deployed to Production.
- **Simplification in Design:** Adhere to the core overview mindset of the AWS Well-Architected Framework, prioritizing simple infrastructure solutions while ensuring reliability and cost governance.

### Technical Architecture

- **Full-stack Observability:** Build a measurement system combining CloudWatch, VPC Flow Logs, ALB Metrics, and Custom Business Metrics to detect early failures in RDS connections or dependent services.
- **Automated Incident Response Workflow:** Connect CloudWatch Alarm with SNS Topic to transmit alerts to team messaging platforms (Slack/Email), supporting Ops teams to respond quickly before SLA commitments are affected.
- **CI/CD Integration of AI Agent:** Integrate AWS Security Agent into CI/CD pipelines (GitHub/GitLab) to scan source code automatically, leveraging automated fixes (Auto-PR Fixes) to free up developers' time.

### Modernization & Personal Growth Strategy

- **Optimizing Security Operational Cost:** Flexibly combine AWS Free Tier with trial packages of AI Security Agents to practice skills, helping businesses cut down on expensive security testing.
- **Tactics to Conquer International Certifications:** Use the Keyword Mapping method and analyze incorrect answers to grasp the nature of cloud services, building a systematic learning path.

## Work Application

- **Improve Monitoring for Current Projects:** Add custom metrics to e-commerce and full-stack project codebases (such as tracking order/login success rates) instead of just monitoring server metrics.
- **Apply "Security IN the Cloud" Mindset:** Review Security Groups and IAM Policies based on the Principle of Least Privilege for active AWS resources.
- **Automated Testing with GitHub Actions:** Integrate automated source code scanners into group project Pull Request workflows to detect secret leaks or programming vulnerabilities early.
- **AWS Certification Prep:** Practice with mock tests combined with reading AWS Skill Builder materials to prepare for the AWS Certified Cloud Practitioner exam.

## Event Experience

- **Access to Practical Knowledge and Diverse Perspectives:** The event provided a comprehensive view from Site Reliability Engineering, certification strategies, to advanced AI-driven DevSecOps trends.
- **Intuitive Live Demos:** The live demo scenario of a system reporting "Green" while users cannot log in due to database connection issues deeply emphasized the importance of user-experience monitoring.
- **Learning from the Community:** The open networking environment between experts and members of HUFLIT AWS Student Builder Group/First Cloud AI Journey expanded connections and shared real-world experiences.

## Key Takeaways

- A healthy server infrastructure does not mean users are having a good experience. Monitoring must focus on user behavior and business value.
- Using autonomous AI agents significantly optimizes security operational time and cost, but humans still play the decisive role in business logic control and risk management.
- The core of mastering cloud technology lies in understanding the Shared Responsibility Model and always preparing backup plans for system failures.

## Some Event Photos

![event3](/images/event3.jpg)
