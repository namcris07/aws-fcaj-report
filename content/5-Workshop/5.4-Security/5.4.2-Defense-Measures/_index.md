---
title : "Implemented Defense Controls"
date : 2026-07-06 
weight : 2
chapter : false
pre : " <b> 5.4.2. </b> "
---

# 5.4.2 Implemented Defense Controls

---

For modern cloud-native systems, relying on a single security perimeter is insufficient. The architecture applies a **Defense-in-Depth** model, embedding security controls across all architectural layers throughout the DevSecOps pipeline. Key defense measures include:

---

### 1. Application-Level Security

- **Transport Layer Encryption (HTTPS):** Enforcing SSL/TLS certificates to encrypt data in transit between clients and servers.
- **Input Validation & Sanitization:** Implementing strict input validation patterns to neutralize malicious injection payloads (SQLi, XSS).
- **Access Control & Identity Hashing:** Enforcing fine-grained authorization and hashing passwords using strong one-way hashing algorithms (bcrypt, Argon2).
- **Anti-CSRF Protection:** Injecting random CSRF tokens into sensitive web forms.

---

### 2. Infrastructure & Container Security

- **Container Image Security Audits:** Performing static base image vulnerability scans prior to deployment and forcing services to execute under non-root users.
- **Infrastructure as Code (IaC) Auditing:** Utilizing automated static analysis (Checkov) to detect misconfigurations in deployment manifests.
- **Network Segmentation & Isolation:** Enforcing restrictive Network Policies and Security Groups to restrict lateral movement.

---

### 3. DevSecOps Pipeline & Operational Controls

- **Centralized Secrets Management:** Integrating automated secrets scanning (Gitleaks) at the git repository level and retrieving runtime credentials via Secrets Manager / Parameter Store.
- **Dependency Management:** Automating Software Composition Analysis (Trivy SCA) to mitigate known CVEs.
- **Behavioral Monitoring & Centralized Logging:** Aggregating event logs (CloudWatch Logs) to enable real-time anomaly detection.
