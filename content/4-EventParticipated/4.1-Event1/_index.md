---
title: "Event 1"
date: 2026-06-27
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---



# SUMMARY REPORT: FCAJ COMMUNITY DAY – JUNE 2026

## Event Objectives

- **Objective:** Connect the Cloud and AI engineering community; share practical corporate perspectives on Cloud career roadmaps, Voice AI architectures, infrastructure automation solutions (DevOps Agents), and secure, cost-optimized Generative AI applications on AWS.

## Speakers
- **Speakers:**
  - **Mr. Steve Tran:** Founder of Cloud Thinker, former Solution Architect at AWS Vietnam.
  - **Mr. Hieu Nghi (Renova Cloud), Mr. Kiet (AWS Student User Group Leader) & Mr. Trung (Founder & CEO of R AI):** Voice AI research and deployment experts.
  - **Ms. Bao & Mr. Nguyen (Cloud Engineers from Cloud Kinetic):** Cloud infrastructure and DevOps AI Agents experts.
  - **Mr. Truong (Wynn) & Ms. Minh Anh (Noventis):** AI solutions and HR (HR Solution) experts.
  - **Mr. Toan Nguyen (AWS Security Builder):** Cloud security expert.

## Key Highlights

### Career Roadmap and the Complexity Problem in Cloud Operations
- **Job Market Shift:** Recruitment trends are shifting dynamically. Businesses are moving from mass hiring to focusing on Senior personnel or engineers who can optimize AI applications to accelerate development and deployment.
- **Technical Debt & Complexity:** As enterprises scale and adopt microservices architectures, system complexity and "technical debts" rise. Systems require highly skilled operations teams to make quick decisions in production-critical environments to minimize financial losses from downtime.

### Voice AI Architecture Classification and Language Challenges
- **System Architecture:** The speakers analyzed two main Voice AI models:
  - **Speech-to-Speech (S2S):** A model that receives audio directly, processes it, and responds with audio (currently optimized mainly for English).
  - **Three-Component Model (STT -> LLM -> TTS):** Receives audio input $\rightarrow$ Converts it to text (Speech-to-Text) $\rightarrow$ Feeds text data into a Large Language Model (LLM) to process context and generate text response $\rightarrow$ Converts the response text to speech (Text-to-Speech) to return to the user.
- **Vietnamese Language Processing (Low-resource language):** Vietnamese lacks large datasets. For real-world deployment at major banks (such as VPBank, VIP), the system must use the three-component model to handle the context of the Vietnamese language effectively via prompting.
- **Advanced Techniques:** The system needs to train auxiliary models to perform:
  - **Gender Detection:** To enable natural "anh/chị" (brother/sister) address forms in Vietnamese.
  - **Natural Interruption Mechanism & Paused Context Understanding:** Detecting when a customer pauses to think (e.g., reading a phone number) so that the Boss does not awkwardly interrupt the customer.
  - **Tool Calling:** Allowing the AI to not only respond but also execute highly secure business tasks directly, such as blocking a bank card via API after Citizen ID (CCCD) verification.

### Working Mechanism of DevOps AI Agent
- **Traditional Challenges:** When system incidents occur, DevOps engineers face fragmented telemetry data across multiple sources (CloudWatch, CloudTrail, Grafana) and context loss, which increases troubleshooting time.
- **Agent Space Architecture:** The DevOps Agent operates based on a logical container called Agent Space. Administrators grant permissions for the Agent to learn the system structure based on resource tags. From there, the Agent automatically builds a system architecture map (Topology) containing hundreds of relationships among ECS, IAM, Lambda, and Network components to serve as an inference database.
- **4-Step Incident Response Automation Process:**
  1. **Triage & Extraction:** Automatically triggered by events (from CloudWatch Alerts or direct user chat), quickly aggregating related logs and traces.
  2. **Investigation:** Formulating technical hypotheses based on the Topology and verifying them with raw log evidence to determine the root cause (Root Cause Analysis).
  3. **Mitigation Proposal:** Generating a detailed mitigation plan (including Prepare, Validate, Execute, and Post-validate phases) without executing it autonomously to ensure absolute system safety (Human-in-the-loop).
  4. **System Improvement:** Analyzing historical incident logs to propose long-term optimization plans, minimizing the recurrence of errors.

### Application of Amazon Q (GenAI) in HR Management
- **Solving the Data Silo Problem:** Amazon Q acts as a cross-platform enterprise AI assistant. Not restricted to a single ecosystem, Amazon Q supports connections through built-in Action Connectors to Microsoft OneDrive, SharePoint, Google Drive, Gmail, or dedicated management systems like Jira, Salesforce, and GitHub.
- **Token Optimization:** Thanks to context storage within an internal memory graph (Memory Graph), Amazon Q handles long conversation threads or internal document queries (policies, CVs, templates) without consuming massive amounts of tokens compared to other public LLM tools.

### Advanced Security Architecture for AI via Private VPC Connection
- **Security Risks:** When integrating enterprise AI applications (like Amazon Q) with third-party MCP (Model Context Protocol) servers (Zalo, WhatsApp, internal CRM) over the internet, the system is vulnerable to Distributed Denial of Service (DDoS) or MIddle-man/Man-in-the-middle attacks, potentially causing data leaks.
- **Secure Internal Connection Solutions (Zero Trust):**
  - Place the MCP Server entirely within a Private Subnet.
  - Use VPC Connection to create an Interface Endpoint, allowing Amazon Q to access the VPC internally without routing through the public internet.
  - Use Route 53 Resolver to resolve internal domains for the MCP Server, ensuring that the server's IP address is never exposed to the public cloud.
  - Store sensitive credentials and API connection details (such as Zalo Tokens) centrally in AWS Secrets Manager.

### Demos

#### Demo 1: Voice Agent for Apple Product Queries
- **Tools:** Amazon Bedrock, Bedrock Agent, Knowledge Base.
- **Content:** An engineer makes a direct voice call to the Voice Agent to query specifications of the MacBook Pro line. The Agent automatically searches the specialized database (Knowledge Base) for M4/M4 Pro/M4 Max chips and responds in real-time via voice (streaming voice-to-text to the LLM and vice versa).

#### Demo 2: Investigating a Mock DDoS Attack with DevOps Agent
- **Tools:** ECS, Application Load Balancer (ALB), DevOps Agent.
- **Content:** A simulated attack scenario was executed using a script that created 10 dummy ECS tasks to flood ALB of an E-commerce application with 1000 requests/second, spiking application latency to 12 seconds. An engineer triggered the investigation flow on the DevOps Agent's chat interface in Vietnamese. The Agent automatically analyzed and returned a Root Cause Analysis pointing out exactly 10 ECS tasks causing the traffic congestion. The Agent provided the clean-up commands; the engineer only needed to copy-paste them into the terminal to stop those 10 ECS tasks, restoring the application to its normal state.

#### Demo 3: CV Screening and Automated Recruitment Workflow with Amazon Q
- **Tools:** Amazon Q Desktop, MCP Server, no-code App Builder.
- **Content:** The speaker uploaded a Markdown configuration file to train the HR Talent Review Assistant capability for Amazon Q. Then, they loaded data including 1 JD (Junior Cloud Engineer) and 6 CVs in different formats (PDF, scanned files). Amazon Q automatically ran OCR to extract data, evaluated the compatibility score based on pre-allocated weights (Technical 40%, Problem Solving 25%, Communication 15%), classified the applicants (Strong/Good/Low), and automatically exported a visual HTML dashboard report without writing any complex system code.

### Comparative Discussion

Based on data analyzed from the event discussion panels, below is a detailed comparative table of operational efficiency:

| Criteria | Traditional Model (Manual / On-premises) | Cloud-based AI Solutions (Agents / Cloud) |
| :--- | :--- | :--- |
| **Monitoring and Operations** | Manual, fragmented data. Must access manually (SSH, check logs in separate dashboards). | Fully automated collection flow. Centralized management via resource relationship Topology mapping. |
| **Initial Setup Cost** | High hardware infrastructure investment cost. Difficult to optimize when resources are under- or over-provisioned. | Pay-as-you-go. DevOps Agent charges flat rate based on seconds of operation (~0.083 USD/second). |
| **Mean Time to Resolution (MTTR)** | Entirely dependent on Senior engineer capability. Incident resolution time ranges from hours, weeks, to months. | Investigation time is measured in minutes due to parallel processing. Reduces MTTR by 75% - 77% (actual MTTR reduced from 2 hours to 28 minutes). |
| **System Integration Architecture** | Fragmented systems (Data Silos), hard to exchange information between third-party applications. | Secure and synchronous connection via MCP (Model Context Protocol) and private network architectures (Private Link, VPC Endpoint). |


## Event Experience

- **Practical Experience:** The event provided a deep, visual experience through real-world live demos (ranging from recovering a crashed e-commerce application to automating document scanning workflows for HR). It clearly demonstrated the power of Cloud and AI in solving daily business problems.
- **Technical and Operational Lessons Learned:**
  - **"Execution" Mindset:** Every solution, whether a Proof of Concept (PoC), Minimum Viable Product (MVP), or production environment, must be actively implemented and configured on actual systems rather than remaining purely theoretical.
  - **AI as a Supporting Assistant, Not a Human Replacement:** In critical infrastructure fields like Cloud Infrastructure or Cybersecurity, AI acts as an amplifier for Senior engineers, helping them automate repetitive admin tasks to focus on strategic decision-making. Systems always require a human approval mechanism (Human-in-the-loop) to control risks of hallucination or unauthorized production data manipulations.
  - **Importance of Observability:** The AI Agent can only provide accurate inferences and analyses if the enterprise has a clean data management infrastructure, recording full deployment history and configuring comprehensive logs, metrics, and alarms.
  - **Cybersecurity is Always Top Priority:** When implementing GenAI solutions in business operations, setting up secure private connections (like Private VPC Connections, internal Route 53 Resolvers) is mandatory to protect the integrity of internal data assets.

## Some Event Photos

![event1](/images/event1.jpg)
