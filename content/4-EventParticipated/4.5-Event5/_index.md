---
title: "Event 5"
date: 2026-08-01
weight: 5
chapter: false
pre: " <b> 4.5. </b> "
---

# EVENT REPORT: AWS FCAJ AGENT FORGE – DEEPDIVE  
## "DAY 1: FOUNDATIONS & AGENT SETUP"

## Purpose of the Event

- **Purpose:**
  - To deliver advanced Level 300 knowledge on building, packaging, and deploying Agentic AI Systems in enterprise production environments.
  - To explore Amazon Bedrock AgentCore in detail, covering its three core infrastructure components: Runtime, Gateway, and Identity.
  - To provide direct Hands-on Lab experience deploying an autonomous AI Agent, integrating MCP Tools, a Knowledge Base, and user authentication management via Amazon Cognito.

## Speakers & Instructors

- **Speakers & Instructors:**
  - **Trần Hữu Nghĩa:** Lead speaker, covering Theory & System Architecture of AgentCore.
  - **Hải Anh:** Speaker & Lab Instructor, leading the Hands-on Lab session.

## Key Content

### 1. Overview of Agentic AI & the Autonomous Spectrum

- **The Nature of Agentic AI:** An autonomous software system capable of reasoning, planning, and sequentially executing chains of tasks to achieve defined objectives.
- **The Autonomous Spectrum:** A continuum ranging from Deterministic Workflows (developer-defined steps) to Fully Autonomous systems (Agents that independently interconnect, communicate, and make decisions). In enterprise settings, a hybrid model incorporating Human-in-the-Loop checkpoints at critical decision points is always preferred to ensure safety and governance.
- **Next-Generation Connectivity Protocols:** Beyond traditional HTTP/REST APIs, the AI era introduces two new standardized protocols:
  - **MCP (Model Context Protocol):** Connects Agents to external Tools and APIs.
  - **A2A (Agent-to-Agent Protocol):** Enables direct inter-Agent communication without an intermediary layer.
- **Strands SDK:** AWS provides an open-source SDK named Strands, purpose-built for the AWS ecosystem, allowing engineers to instantiate Agents using a Factory Design Pattern in just a few lines of code.

### 2. Amazon Bedrock AgentCore Runtime & Serverless Infrastructure

- **Serverless Architecture & Cost Model:** Runtime operates entirely serverless, billed on a Pay-as-you-go basis for actual usage — minimizing idle infrastructure costs for enterprises.
- **Packaging & Environment Isolation Technology (MicroVM Isolation):** Leverages **Firecracker MicroVM** technology to create ultra-lightweight virtual machines that completely isolate hardware resources, memory, and filesystem for each individual User Session, guaranteeing that internal data never leaks between users.
- **Supported Deployment Methods:**
  - **Source Code Initialization:** Configure System Prompt, Model, and Tools directly using Strands SDK.
  - **Container Packaging:** Push a Docker Image to Amazon ECR.
  - **Zip Source Upload:** Package and upload all source files to Amazon S3.
- **Safe Rollout Strategy:** Supports Alias/Versioning (ARN) tagging for each Agent version, enabling **Canary Deployments** (percentage-based traffic splits) and instant **Rollbacks** to previous versions in the event of incidents.

### 3. Security & Authentication Layer (AgentCore Identity)

- **Bidirectional Security Mechanism (Inbound & Outbound):**
  - **Inbound:** Authenticates users/applications before they can access and invoke the Agent. Utilizes JSON Web Tokens (JWT) or Amazon Cognito User Pools to verify identity.
  - **Outbound:** Governs Agent permissions when calling external tools, APIs, or third-party databases.
- **Secure Token Exchange Model (Workload Access Token — WAT):** Upon receiving a JWT Token from a user, AgentCore Identity exchanges it for a new intermediate token (WAT Token) that combines the user's identity with the Agent's identity. This token is encrypted and stored in AgentCore's **Token Vault**, ensuring the original user token is never exposed to external systems.

### 4. Middleware Connectivity Layer (AgentCore Gateway)

- **Solution to the Scaling Challenge:** As a system grows to hundreds of Agents and thousands of Tools/MCP Servers, direct connections become unmanageably complex. The Gateway acts as a single Middleware layer that centrally enforces all security policies, authentication requirements, and access control rules.
- **Human-in-the-Loop & Interceptor Mechanisms:**
  - Allows Admins to directly intervene to **Approve** or **Deny** requests that exceed Agent policy limits or quotas.
  - The **Interceptor** layer automatically filters sensitive data (such as Personally Identifiable Information — PII) before returning results to users.
- **Semantic Tool Search:** Integrates vector indexing and semantic search on the Gateway, enabling Agents to automatically identify and retrieve the most relevant tool based on JSON Schema descriptions (name, function, required parameters).

## Comparative Analysis

Based on the session content, the table below contrasts traditional deployment models with Amazon Bedrock AgentCore architecture:

| Criteria | Traditional Approach | Amazon Bedrock AgentCore Architecture |
| :--- | :--- | :--- |
| **Agent Runtime Environment** | Fixed servers or self-managed containers; manual configuration required; costs incurred even with zero load. | Fully Serverless (Firecracker MicroVM); auto-scales on demand; Pay-as-you-go billing for actual usage only. |
| **User Data Isolation** | Difficult to guarantee complete isolation; risk of data leakage across sessions. | MicroVM Isolation creates a fully isolated environment per User Session — zero cross-session leakage. |
| **New Version Deployment** | Big-bang deploys; service downtime required; high-risk rollbacks. | Alias/Versioning (ARN) + Canary Deployment via traffic splitting; instant rollback by simply updating an Alias pointer. |
| **Authentication & Security** | Self-implemented authentication; user tokens passed directly to external systems — risk of credential exposure. | AgentCore Identity: JWT → WAT Token exchange; Token Vault encryption — original tokens never leave the controlled environment. |
| **Tool Connectivity & Management** | Direct Agent↔Tool connections; scaling to hundreds of tools becomes cumbersome and difficult to secure. | AgentCore Gateway: centralized Middleware layer; Semantic Tool Search automatically matches the correct tool by semantic intent. |
| **Sensitive Data Filtering** | Custom PII filtering logic required at every integration point. | Built-in Interceptor in Gateway; automatically detects and filters PII before returning results to users. |

## Key Takeaways

### Design Thinking

- **Production-Ready System Design Mindset:** A real-world Agent system cannot stop at the Proof-of-Concept (POC) stage on a Jupyter Notebook. It must fully meet enterprise standards for Scalability, Security, Observability, and Cost Efficiency before it is fit for production deployment.
- **Middleware Architecture Pattern:** Clearly separating the roles of the code execution layer (Runtime), the authentication layer (Identity), and the tool orchestration layer (Gateway) enables independent maintenance, scaling, and replacement of each component without creating Tight Coupling between them.

### Technical Architecture

- **Secure Hybrid/Private Network Connectivity:** Using **AWS PrivateLink** and **NAT Gateway** to establish secure connections between AgentCore and internal on-premises systems or isolated VPC infrastructures — without exposing any traffic to the public internet.
- **Bidirectional Streaming for Real-Time Responses:** Establishing bidirectional data streaming channels (Text/Voice/Vision) allows Agents to deliver answers progressively in chunks (Streaming), dramatically reducing end-user perceived latency.

### Personal Development Strategy

- **Engaging with L300-Level Standards:** Building the capacity to absorb dense, expert-level knowledge from official AWS advanced training programs requires strong foundational knowledge and Systems Thinking to correctly internalize the underlying principles.
- **Selecting the Optimal AI Model:** Learning to evaluate and benchmark diverse model families (Anthropic Claude, Amazon Nova) to select the right model for each use case — optimizing for speed, cost, or complex reasoning capability as the task demands.

## Applying Learnings to Work

- **Packaging Agents with Strands SDK:** Practiced writing Agent configuration code, integrating automated lookup tools, and packaging the complete solution as a Docker Container ready for upload to Amazon ECR.
- **Configuring Amazon Cognito Authentication:** Building standardized user sign-in/sign-up flows and integrating JWT token issuance to control access permissions to the AgentCore Runtime.
- **Designing JSON Schemas for MCP Tools:** Standardizing tool description files with clear names, functional descriptions, and parameter definitions so Agents can accurately identify and invoke the correct tool via the Gateway's Semantic Tool Search mechanism.

## Event Experience

- **Condensed, Battle-Tested Knowledge Delivery:** The event transmitted a large volume of foundational knowledge in a structured and compact format, pairing deep theoretical lectures with immediate hands-on lab reinforcement — giving attendees a clear path from principles to concrete implementation steps.
- **Direct Console Interaction Experience:** Hands-on interaction with the Amazon Bedrock AgentCore console provided a vivid mental model of the complete configuration flow — from Runtime and Gateway setup through to operational log inspection — something that reading documentation alone cannot replicate.

## Lessons Learned

- **An Agent that performs well in a test environment is not necessarily Production-ready.** The critical differentiator lies in the security governance layer (Identity) — controlling who can invoke the Agent and what external resources the Agent is permitted to access — combined with data flow control (Gateway) to filter sensitive information and enforce enterprise policies consistently.
- **Adopting open standard protocols (MCP, A2A)** gives Agents the flexibility to connect with any tool ecosystem without being locked into a single vendor (Vendor Lock-in), opening the door to broader integrations with the growing global ecosystem of AI tools in the future.

## Event Photos

![event5](/images/event5.jpg)
