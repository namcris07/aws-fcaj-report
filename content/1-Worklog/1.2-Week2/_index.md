---
title: "Week 2 Worklog"
date: 2026-06-22
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---



### Week 2 Objectives:

* Connect and get acquainted with members of the First Cloud AI Journey community.
* Master core concepts and administration of Amazon EC2 instances, EBS storage, Instance Store, AMIs, and Snapshots.
* Learn about system automation with Auto Scaling Groups (ASG) and load balancing via Elastic Load Balancer (ELB).
* Master Amazon S3 object storage management, including security controls (Bucket Policy, ACLs, Block Public Access), Versioning, and S3 Static Website Hosting integrated with Amazon CloudFront CDN.
* Deploy Shared Storage (EFS, FSx) and Hybrid Cloud storage solutions using AWS Storage Gateway (File Gateway) to bridge on-premises environments with the cloud.
* Study Disaster Recovery (DR) strategies on AWS based on RTO/RPO metrics, and manage centralized backups using AWS Backup.

### Tasks to be carried out this week:
| Day | Task                                                                                                                                                                                                   | Start Date | Completion Date | Reference Material                        |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------- | --------------- | ----------------------------------------- |
| 2   | - **AWS Compute & EC2 Administration** <br>&emsp; + **Compute Theory:** Research Amazon EC2 architecture and Nitro Hypervisor layer to optimize performance; analyze Instance Types using Intel/AMD chipsets and ARM (AWS Graviton) to save up to 40% cost; secure access via Key Pairs (SSH for Linux, RDP password decryption for Windows). <br>&emsp; + **Storage for Compute:** Differentiate between EBS (Block storage with dedicated network, 99.999% availability replicated across 3 physical nodes within an AZ, EBS Multi-Attach) and Instance Store (NVMe local high-speed but with Ephemeral risk where data is lost upon Stop); manage AMIs and Incremental Snapshots on S3. <br>&emsp; + **Hands-on Lab:** <br>&emsp;&emsp; * Launch and manage EC2 virtual instances (Linux & Windows). <br>&emsp;&emsp; * Connect securely via SSH Client and Remote Desktop (RDP). <br>&emsp;&emsp; * Create a new EBS volume, attach it to the instance, partition and mount. | 22/06/2026 | 22/06/2026      | [Module 3](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%203/module-3.md) |
| 3   | - **HA Operations, Shared Storage & Migration** <br>&emsp; + **Automation & Scaling:** Configure User Data script to automatically deploy Web Servers; query instance information using EC2 Metadata via the Link-local IP `169.254.169.254`. Configure Auto Scaling Group (ASG) for automatic scaling combined with Elastic Load Balancer (ELB) for load distribution; study pricing models (On-Demand, RI/Savings Plans, Spot Instances). <br>&emsp; + **Amazon Lightsail:** Fixed-price packaged computing, One-Click VPC Peering, and converting snapshots to EC2. <br>&emsp; + **Shared Storage (EFS & FSx):** Configure Amazon EFS (NFSv4 for Linux) connecting thousands of instances simultaneously and Amazon FSx for Windows Server (SMB/NTFS) with Data Deduplication saving 30-50% cost. <br>&emsp; + **Migration & Disaster Recovery (AWS MGN):** Study migration using AWS Application Migration Service; understand Agent replication to Staging VPC and Cut-over target instances. | 23/06/2026 | 23/06/2026      | [Module 3](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%203/module-3.md) |
| 4   | - **AWS Storage & Disaster Recovery** <br>&emsp; + **Object Storage & Lifecycle:** Study Amazon S3 (11 9s durability, >=3 AZs replication, S3 Event Trigger, Object Key prefix partition hashing); S3 Storage Classes (Standard, IA, Intelligent-Tiering, Glacier Archive); Lifecycle Management and S3 Access Points. <br>&emsp; + **Security & Advanced S3:** Configure S3 Static Website Hosting & CORS; access control (IAM Policy vs Resource Policy); establish S3 Gateway Endpoints for private internal connection via Private IP. Enable S3 Versioning (Delete Marker) to prevent Ransomware. <br>&emsp; + **Glacier Vaults & Vault Lock:** Manage Glacier Archives; 3 data retrieval options (Expedited, Standard, Bulk); configure strict Vault Lock compliance policies. <br>&emsp; + **Hybrid Storage & Snow Family:** Offline data transfer via Snow Family (Snowball, Snowball Edge, Snowmobile); implement AWS Storage Gateway (File Gateway, Volume Gateway Stored/Cached, Tape Gateway VTL). <br>&emsp; + **Disaster Recovery (DR) & AWS Backup:** Analyze RTO/RPO metrics; compare 4 DR strategies (Backup & Restore, Pilot Light, Warm Standby, Multi-site Active-Active); configure centralized AWS Backup and VM Import/Export. | 24/06/2026 | 24/06/2026      | [Module 4](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%204/module-4.md) |
| 5   | - **S3 Hands-on & Web Optimization (Lab 57):** <br>&emsp; + Create S3 Bucket, configure Object Ownership (ACLs enabled) and upload static website source code. <br>&emsp; + Enable S3 Static Website Hosting. <br>&emsp; + Disable Block Public Access and configure public read access for objects using ACLs. <br>&emsp; + Integrate Amazon CloudFront (CDN) for security and global delivery acceleration (OAI, Bucket Policy update). <br>&emsp; + Enable Bucket Versioning to prevent Ransomware data loss. | 25/06/2026 | 25/06/2026      | [Lab 57](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%204/Hand-ons/Lab57.md) |
| 6   | - **Hybrid Storage & Shared Storage Hands-on (Lab 24 & Lab 25):** <br>&emsp; + **Lab 24 (File Gateway):** Create S3 Bucket. Launch EC2 Storage Gateway to emulate an on-premises appliance. Configure Security Group inbound rules for SMB (445) and NFS (111, 2049, 20048) ports. Set up Storage Gateway and SMB File Share in the AWS Console. Mount the network drive on the client machine using CMD, create test files, and verify automatic sync to S3. <br>&emsp; + **Lab 25 (Amazon FSx for Windows):** Set up AWS Managed Microsoft AD for identity verification. Deploy Amazon FSx for Windows File Server (Single-AZ SSD 32 GiB) joined to the Active Directory domain. Launch Windows EC2 instances, join the domain, mount the shared network drive via SMB, and verify real-time data sync. | 26/06/2026 | 26/06/2026      | [Lab 24](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%204/Hand-ons/Lab24.md) <br> [Lab 25](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%204/Hand-ons/Lab25.md) |


### Week 2 Achievements:

* **Compute & Automation Administration (EC2 & Auto Scaling):**
  * Launched and managed EC2 instances (Linux/Windows), configured Security Groups, and connected securely via SSH/RDP.
  * Attached, partitioned, and mounted Amazon EBS volumes to instances for dedicated data storage.
  * Used User Data scripts to automate web server deployment and queried EC2 Metadata via the link-local IP.
  * Configured Auto Scaling Groups (ASG) and Elastic Load Balancers (ELB) to automatically scale applications based on load.
* **Object Storage & Global Distribution (S3 & CloudFront):**
  * Created S3 Buckets, configured object permissions (Block Public Access settings, Object Ownership ACLs, Bucket Policies).
  * Enabled S3 Static Website Hosting and Bucket Versioning to secure data against Ransomware.
  * Created CloudFront Distributions, configured Legacy Access Identities (OAI) for secure S3 connectivity, and distributed static websites globally via CDN.
* **Shared Storage & Hybrid Cloud (FSx, EFS & Storage Gateway):**
  * Implemented Hybrid Storage: Setup AWS Storage Gateway (File Gateway) via an EC2 appliance emulator, configured SMB File Share, and mounted network drive `Z:` to verify real-time sync to S3.
  * Deployed enterprise Shared Storage: Provisioned AWS Managed Microsoft AD, configured Amazon FSx for Windows File Server (SMB), and mapped shared drives across domain-joined EC2 instances.
* **Security, Backup & Disaster Recovery:**
  * Understood the 4 DR strategies (Backup & Restore, Pilot Light, Warm Standby, Multi-site Active-Active) and mapped them to RTO/RPO targets.
  * Created automated backup plans and set retention policies using AWS Backup.
