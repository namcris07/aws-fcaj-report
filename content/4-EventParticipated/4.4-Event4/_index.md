---
title: "Event 4"
date: 2026-07-25
weight: 4
chapter: false
pre: " <b> 4.4. </b> "
---

# EVENT REPORT: FCAJ X AGENTIC AI BUILD WEEK – "SHOW UP. BUILD. PITCH. WIN!"

## Purpose of the Event

- **Purpose:**
  - To create a hands-on, competitive environment for developers, data engineers, and students to build autonomous AI (Agentic AI) applications that solve real-world business problems.
  - To share cloud-native architectural solutions on AWS and practical insights from award-winning Hackathon teams of the Agentic AI Build Week.
  - To inspire innovative mental models, a lifelong learning mindset, and effective teamwork skills in the AI Agent era.

## Speakers & Team Representatives

- **Speakers & competing teams:**
  - **Joseph Marazota:** Head of Technology at AWS.
  - **Nguyen Gia Hung:** Head of Solution Architect at AWS Vietnam, Founder of First Cloud AI Journey (FCAJ).
  - **One Team (1st Place – AWS Track):** Represented by Chung and team members.
  - **Signal Scout (2nd Place – AWS Track):** Hoang Hieu, Quoc Hao, Minh Quan (Willer), Cong Minh, Duy Khiem, Tuan Luc.
  - **Plan B Team:** Long, Vi, Phat, An, Nghia.
  - **3K Team:** Nguyen Huy, Huynh An Khuong, Hoang Huynh Duc, Ngo Khoi, Dang Nguyen Phuoc Loc.
  - **Six Pillars Team:** Viet, Nguyen Van Linh, Nguyen, Minh Nhat, Phuoc Huyen.

## Key Content

### Opening Keynote: Reframing the Mental Model for the AI Agent Era (Joseph Marazota)

- **The Release Velocity Shift:** Previously, enterprises required a full quarter or two weeks for a software release cycle. In the AI Agent era, updates and deployments can happen within minutes — a fundamental disruption to the traditional product development lifecycle.
- **Challenging Legacy Mental Models:** Young engineers must actively challenge the "it can't be done" barriers embedded in traditional workflows. Not being burdened by legacy infrastructure thinking (mainframe/on-premise mindset) is a competitive advantage to drive organizational transformation.
- **Human in the Loop:** AI and robotics (including 1 million robots in Amazon's fulfillment centers) are merely hardware without human direction. Engineers remain the decision-makers who evaluate and approve AI Agent recommendations, ensuring safety and accuracy.

### AI-Powered Conversational Ordering – KFC Agent (One Team – 1st Place)

- **Real-World Problem Statement:** Analyzed McDonald's failed Drive-through AI ordering trial caused by hallucination and an inability to understand natural conversational context (e.g., erroneously ordering 100 pieces of chicken). This served as the practical starting point for the design challenge.
- **No App-Switching Solution:** Built an Agent integrated directly into existing messaging platforms (Zalo/WhatsApp), allowing customers to place orders within a familiar chat interface — no new app download or account registration required, minimizing user friction.
- **AWS Technical Architecture:**
  - **Amazon Bedrock Agent Core** manages session memory to retain each user's order preferences and conversational context.
  - **AWS WAF** secures the infrastructure; **Tiny Fish** (AI Web Scraping) crawls real-time menu data from KFC's official website and stores it in an AWS Database.
- **Cost Efficiency:** Operating cost of approximately **$0.006 per order** — a 75% reduction compared to traditional processing models by replacing heavyweight processing layers with Agent Core.

### Multi-Agent Competitor Intelligence Platform – Signal Scout (Signal Scout – 2nd Place)

- **Business Core:** Automatically collects scattered public market signals (financial reports, shareholder statements, tax identification data) about competitors, chains them together, and forecasts ROI projections if a business adopts similar models or strategies.
- **Multi-Agent Architecture (A2A – Agent to Agent):**
  - **Agent Management (Supervisor):** Orchestrates all Sub-agents across task workflows.
  - **Crawler Subagent:** Dynamically selects between **Apify** (for static pages) and **Tiny Fish** (for dynamic pages, bypassing public login walls).
  - **Analysis Subagent & Validation Loop:** Uses **LangFields** to score data quality. If the score is insufficient, the system automatically retries retrieval (up to 2 times to preserve token budget); only unresolved cases escalate to human review.
- **Cost Optimization & Compliance:** Migrated from third-party tools (driving costs to **$94/month**) to **Native AWS Browser/Web Tools**, reducing costs to **$35/month** while guaranteeing on-premises Data Residency compliance.

### Automated Architecture Diagramming & Infrastructure Code Generation (Plan B Team)

- **Pain Point for Solutions Architects (SA):** Customers frequently demand architecture designs and cost estimates on extremely tight deadlines — a critical bottleneck in technical consulting and pre-sales workflows.
- **AI Native App Workflow:**
  - Input requirements in natural language (Prompt/Document)
  - → AI analyzes enterprise policies and constraints
  - → Automatically generates architecture diagrams on **Draw.io**
  - → Automatically computes a **Cost Estimation** breakdown
  - → Auto-generates **IaC code (Terraform/CloudFormation)**
  - → Automatically deploys the infrastructure to AWS.
- **Output Quality Control:** Integrates official AWS open-source icon packages (AWS Icons) and applies a Validation Script to scan and block any services on the enterprise's internal Blacklist, ensuring full policy compliance.

### Real-Time Crowd Flow Monitoring System – Sheper (3K Team)

- **Real-World Problem:** Reduce congestion at airports, supermarkets, and large events by analyzing pedestrian flow through surveillance cameras in real time, enabling proactive staff deployment.
- **Technical Architecture:**
  - Streams video through **Amazon Kinesis Video Streams** into **AWS Fargate** for distributed processing.
  - Applies **YOLOv8/YOLOv11** combined with **ByteTrack** to detect individuals, assign tracking IDs, and compute Confidence Scores.
  - Allows operators to define custom monitoring zones (Edit Zone) to calculate real-time movement density.
  - Integrates **Bedrock Agent** to synthesize analysis and generate automated staff coordination recommendations.

### Anti-Money Laundering Detection Framework – Adaptive Workflow Engine (Six Pillars)

- **Industry Reality in Banking/Fintech:** 90–95% of suspicious transaction alerts are False Positives, costing analysts an average of **3 hours per case** and **$20–$25 per manual review**. This is a severe operational cost inefficiency in the financial sector.
- **3-Layer Processing Architecture:**
  - **Layer 1 – Fast Detection:** **Kinesis Data Stream** ingests data → **Lambda Feature Engineering** → **XGBoost** classifier rapidly flags anomalous transactions (low cost, filters out 90–95% of non-suspicious activity).
  - **Layer 2 – Deep Investigation (Agent Core):** **AWS Step Functions** orchestrates 3 specialized Sub-agents (**KYC Check, Money Flow Check, Sanction Check**) combined with an **OpenSearch Vector Database (RAG)** to extract legal precedents and compile structured Evidence Files.
  - **Layer 3 – Decision & Human in the Loop:** Two LLM models perform **mutual cross-validation (Double-check)** combined with **Bedrock Guardrails** to prevent hallucination; clean cases are handled automatically while only high-risk cases are **escalated** to a human-operated Dashboard for final decision.

## Comparative Analysis

Based on solutions presented at the event, the table below contrasts traditional approaches with AWS-powered Agentic AI architectures:

| Criteria | Traditional / Manual Approach | Agentic AI Architecture on AWS |
| :--- | :--- | :--- |
| **F&B Ordering (Drive-through)** | Manual staff or basic AI prone to context errors and hallucination. | Agent integrated into Zalo/WhatsApp + Bedrock Session Memory; $0.006/order, 75% cost reduction. |
| **Competitor Intelligence** | Manual research teams, slow turnaround, high cost ($94+/month). | Multi-Agent A2A (Supervisor → Crawler → Analysis + Validation Loop); reduced to $35/month with Native AWS Tools. |
| **Architecture Design & Estimation** | SA spends hours hand-drawing, calculating costs in spreadsheets. | AI Native App: Natural language → Draw.io diagram → Cost Estimation → Terraform → auto-deploy; 80%+ time savings. |
| **Crowd Monitoring** | Passive CCTV review after congestion occurs. | Kinesis Video + YOLO + ByteTrack + Bedrock Agent: real-time analysis with proactive staff recommendations. |
| **AML Detection** | 90–95% False Positive rate, 3 hrs/case, $20–$25/manual review. | 3-Layer (XGBoost → Step Functions/RAG → Double LLM + Guardrails); only genuine high-risk cases escalate. |

## Key Takeaways

### Design Thinking

- **Business-First & Pain-Point Driven Thinking:** No matter how sophisticated the technology, it cannot transcend business constraints. Products must focus on solving specific user/enterprise pain points — not building generic applications (to-do lists, basic CRUD systems) to demonstrate technical complexity.
- **Scope Management:** Under time-pressured conditions (such as a 24–48h Hackathon), clearly defining In-scope and Out-of-scope boundaries is critical. Prioritize delivering a stable, functional **MVP** over expanding features endlessly and risking system failure.
- **Cost Optimization & Cloud Sovereignty (Cost & Compliance Optimization):** Always measure infrastructure operating costs (Cloud pricing model); prioritize cloud-native solutions to reduce third-party dependency and meet internal Data Residency compliance standards.

### Technical Architecture

- **Multi-Agent Systems & A2A Architecture:** Mastered the practical application of the Orchestrator/Supervisor model coordinating specialized Sub-agents (Crawler, KYC, Analysis) to handle complex workflows without creating system bottlenecks.
- **Hallucination Prevention & Data Quality Control:** Combine dual LLM cross-validation, **Bedrock Guardrails**, iterative **Validation Loops**, and a human approval step (**Human-in-the-Loop**) for high-risk financial and legal operations.
- **Real-Time Streaming Pipelines:** Mastered the integration of **Kinesis Video/Data Streams** with AI Vision models (YOLO) and tabular ML models (XGBoost) for high-throughput processing without bottlenecks.

### Personal Development Strategy

- **Hands-On Learning through Real Pressure:** Theoretical learning must be paired with real high-pressure experiences (e.g., working continuously for 24 hours) to develop psychological resilience and on-the-spot incident response capabilities.
- **The Power of Teamwork & Low Ego:** Actively listening to teammates, suppressing individual ego, and clearly dividing responsibilities by strength (Frontend, Backend, AI, Business/Pitching) are the critical success factors that carry a project to completion.

## Applying Learnings to Work

- **Integrate Agentic Workflows into Personal Projects:** Apply the Supervisor–Sub-agent model to group assignments or e-commerce systems to automate complex task flows, replacing brittle chains of IF-ELSE logic.
- **Optimize CI/CD Pipelines & Testing Practices:** Adopt disciplined environment variable management (`.env` files) and clean Git hygiene to prevent leaking secrets or API keys to public repositories.
- **Infrastructure as Code (IaC) Practice:** Use tools like **Terraform** to manage cloud infrastructure as version-controlled source code, enabling rapid reuse and consistent deployment across Dev/Staging/Production environments.
- **Production-Ready Demo Preparation:** Aim to build a functional, visually demonstrable product (Production-ready or MVP), and always prepare contingency plans for network outages or AI token exhaustion during live presentations.

## Event Experience

- **Open & Barrier-Free Knowledge Sharing:** Speakers and competing teams openly shared their most stressful moments and real technical mistakes (accidentally pushing `.env` to GitHub, system lag due to network issues, unexpected SageMaker cost spikes during demos) — allowing attendees to absorb lessons organically and authentically.
- **Diverse Networking Opportunities:** The event connected students and engineers from multiple universities (FPT, HUFLIT, etc.) and disciplines (AI, Security, Software Engineering, Business) in a collaborative, mutually respectful setting to tackle shared challenges.
- **Inspiration to Break Personal Limits:** Witnessing complete, production-quality products built in just 24–48 hours gave attendees the confidence to sign up for future tech competitions, dismantling the psychological barrier of "I'm not good enough to join a Hackathon."

## Lessons Learned

- **Technology is the means; practical value is the goal:** No matter how sophisticated the architecture, it carries zero value if it fails to address the right user or business pain point.
- **Failure in testing is an opportunity to learn:** Confronting time pressure, infrastructure failures, and depleted AI token budgets trains engineers to adapt quickly and persist through adversity.
- **The power of collaborative growth:** *"If you want to go fast, go alone. If you want to build something great in the AI era, go together — with clear role definition and an unrelenting spirit of learning."*

## Event Photos

![event4](/images/event4.jpg)
