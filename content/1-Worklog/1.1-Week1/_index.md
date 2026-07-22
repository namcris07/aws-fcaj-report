---
title: "Week 1 Worklog"
date: 2026-06-15
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---


### Week 1 Objectives:

* Connect and get to know the members of the First Cloud AI Journey.
* Understand basic AWS services, and how to use the AWS Console & CLI.
* Understand the basic concepts of cloud computing and AWS Global Infrastructure.
### Tasks to implement this week:
| Day | Tasks | Start Date | Completion Date | Reference Materials |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | - Get familiar with FCAJ members and read internal regulations. <br> - **Study AWS Module 1 & Practice Lab 12:** <br>&emsp; + **Theory:** Cloud Computing concepts & 4 core benefits; AWS differences (Economies of Scale, Leadership Principles); Global Infrastructure (Region, AZ, Edge Location); Management tools (Console, CLI, SDK); Cost optimization & AWS Support plans. <br>&emsp; + **Practice Lab 12 (Identity Center):** Enable AWS Organizations, set up AWS IAM Identity Center, configure Users/Groups, assign permissions to member accounts using Permission Sets, implement Time-based Access Control, and verify access via User Portal & AWS CLI. | 15/06/2026 | 15/06/2026 | [Module 1](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%201/module-1.md) <br> [Lab 12](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%201/Hand-ons/Lab12.md) |
| 3   | - **Implement AWS Intermediate Workshop & VPC Basics:** <br>&emsp; + **Identity & CLI:** Set up AWS IAM Identity Center (User, Group, Permission Sets), install and configure AWS CLI v2. <br>&emsp; + **Networking Basics (Amazon VPC):** Study Amazon VPC concepts (isolated virtual network, Multi-AZ) and Subnets (Public Subnet routing to internet, Private Subnet for internal resources; understanding the 5 Reserved IPs of AWS). <br>&emsp; + **Networking Components:** Master Route Tables (Main & Custom), Internet Gateways (IGW), Elastic Network Interfaces (ENI), Elastic IPs (EIP), NAT Gateways for private subnet outbound access, and VPC Endpoints (Interface/Gateway Endpoint) for private service connections via AWS PrivateLink without public internet exposure. | 16/06/2026   | 16/06/2026      | [Module 2](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%202/module-2.md) |
| 4  | - **Security Governance & VPC Network Security:** <br>&emsp; + **Frameworks:** Study AWS CAF (Directive) and NIST CSF (Identify) frameworks to construct network security strategies. <br>&emsp; + **VPC Network Security:** Compare and analyze Security Groups (SG - Stateful at ENI level, Allow rules only) and Network ACLs (NACL - Stateless at Subnet level, supporting both Allow/Deny rules with prioritized execution). <br>&emsp; + **Traffic Monitoring:** Configure VPC Flow Logs to capture IP traffic metadata for audit and threat detection purposes. | 17/06/2026   | 17/06/2026      | [Module 2](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%202/module-2.md) |
| 5   | - **Inter-VPC Connectivity & Troubleshooting:** <br>&emsp; + **Troubleshooting:** Resolve CLI configuration errors, OIDC synchronization issues, and IAM log permissions in CloudWatch. <br>&emsp; + **Inter-VPC Connections:** Differentiate between VPC Peering (1-1 direct connection, non-transitive, no overlapping CIDRs) and Transit Gateway (TGW - central hub-and-spoke router for large-scale routing via Transit Gateway Attachments). | 18/06/2026   | 18/06/2026      | [Module 2](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%202/module-2.md) |
| 6   | - **Hybrid Networking & Elastic Load Balancing (ELB):** <br>&emsp; + **Hybrid Networks:** Study connectivity models including Site-to-Site VPN (IPsec encryption via CGW & VGW), Client VPN for remote access, and AWS Direct Connect dedicated physical lines (Dedicated/Hosted) for ultra-low latency. <br>&emsp; + **Elastic Load Balancing (ELB):** Master ELB mechanisms (Health checks, Sticky sessions, Access logs); classify and apply Application Load Balancer (ALB - Layer 7, Path/Host-based routing), Network Load Balancer (NLB - Layer 4, high performance, static IPs), and Gateway Load Balancer (GWLB - Layer 3, GENEVE port 6081 forwarding to virtual appliances). | 19/06/2026   | 19/06/2026      | [Module 2](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%202/module-2.md) |


### Week 1 Achievements:

* **Mastering the fundamentals of AWS and Cloud Computing:**
  * Understand the concept of Cloud Computing and the difference between traditional infrastructure (CapEx) and cloud (OpEx).
  * Analyze and master the 4 core benefits of the cloud: Cost Optimization, Speed & Agility, Scalability, and Global Reach.
  * Gain deep understanding of AWS Global Infrastructure, including Data Centers, Availability Zones (AZ) with fault isolation design, Regions, and Edge Locations (serving CloudFront CDN, WAF, Route 53).
  * Approach AWS operational culture through Leadership Principles (Customer Obsession, Ownership, Deliver Results) and the Economies of Scale pricing philosophy.

* **Mastering AWS management and interaction tools:**
  * Know how to use the AWS Management Console, install and configure the AWS CLI (using Access Key/Secret Access Key), and interact via the AWS SDK.
  * Understand and apply security authorization mechanisms: distinguish between Root User (always enable MFA) and IAM User applying the principle of Least Privilege.
  * Deployed centralized identity management via AWS IAM Identity Center and AWS Organizations; created Users/Groups and mapped Permission Sets to member accounts; configured Time-based Access Control for security enforcement, and accessed resources via AWS User Portal and AWS CLI.

* **Managing costs and AWS Support plans:**
  * Master cost optimization strategies (Right-sizing, Serverless) and distinguish between EC2 resource purchasing models (On-Demand, Reserved Instances, Spot Instances).
  * Know how to use AWS Budgets to monitor budgets, set threshold alerts, and use Cost Allocation Tags.
  * Distinguish the privileges and scope of AWS Support plans (Basic, Developer, Business, Enterprise).

* **Core knowledge of Amazon VPC & Networking components:**
  * Understand the concept of Virtual Private Cloud (VPC) and set up Subnets (Public Subnet routing to Internet Gateway, Private Subnet connecting via NAT Gateway).
  * Master the roles of Elastic Network Interface (ENI), Elastic IP (EIP), and Route Tables in routing traffic.
  * Understand the mechanism of VPC Endpoints, enabling private internal connections to other AWS services (S3, DynamoDB) without going through the public Internet.

* **Network security mechanisms and Inter-VPC connectivity solutions:**
  * Compare and clearly distinguish the operating mechanisms of Security Groups (Stateful, ENI level, Allow only) and Network ACLs (NACL - Stateless, Subnet level, support both Allow/Deny).
  * Learn about VPC Flow Logs for security analysis and anomaly detection.
  * Clearly understand two methods for Inter-VPC connection: VPC Peering (1-1 direct connection, non-transitive) and Transit Gateway (central hub-and-spoke router for large scale).

* **Hybrid networking solutions and Elastic Load Balancing (ELB) services:**
  * Study hybrid networking connectivity methods: Site-to-Site VPN, Client VPN, and AWS Direct Connect dedicated physical connection (combining VPN for security).
  * Understand the working mechanism of Elastic Load Balancing (ELB), target health check mechanisms, and sticky sessions.
  * Classify and master the main types of Load Balancers: Application Load Balancer (ALB - Layer 7), Network Load Balancer (NLB - Layer 4), and Gateway Load Balancer (GWLB).
