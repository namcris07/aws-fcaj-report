---
title : "Common Security Vulnerabilities"
date : 2026-07-06 
weight : 1
chapter : false
pre : " <b> 5.4.1. </b> "
---

# 5.4.1 Common Security Vulnerabilities

---

### 1. Sensitive Information Leakage - Hardcoded Secrets

Rapid development cycles without automated security guardrails frequently lead developers to accidentally embed hardcoded credentials directly into source code repositories or configuration files. These secrets typically include database connection passwords, cloud service provider API keys (AWS, GCP, Azure), private encryption keys, or access tokens.

- **Exploitation Mechanism:** Once source code is pushed to version control systems (such as GitHub, GitLab, or Bitbucket), malicious actors employ automated scanning bots to scrape repositories for secret patterns in real time.
- **Consequences:** Attackers use exposed keys to gain unauthorized database access, steal customer data, or abuse cloud compute resources for cryptomining, leading to severe financial losses.

---

### 2. Broken Access Control

Broken Access Control consistently ranks among the top OWASP vulnerabilities, occurring when applications fail to properly enforce authentication and authorization policies.

- **Horizontal Privilege Escalation:** Attackers exploit Insecure Direct Object References (IDOR) to access, modify, or delete data belonging to peers at the same authorization tier (e.g., tampering with user IDs in URL parameters).
- **Vertical Privilege Escalation:** Attackers bypass authorization checks to access administrative APIs or control panels, manipulating system data or application settings.

---

### 3. Injection Flaws

Injection flaws represent some of the most destructive web application vulnerabilities, occurring when untrusted user input is passed directly to command interpreters without proper validation or sanitization.

![URL Attack Diagram](/images/5-Workshop/5.4-Security/url_attack_diagram.png)

*Figure 5.4.1: Attack flow diagram illustrating unvalidated URL input leading to data exfiltration.*

Key injection types include:
- **SQL / NoSQL Injection:** Attackers inject malicious query parameters into login forms or URL strings, bypassing authentication, extracting database contents, or dropping tables.
- **OS Command Injection:** Occurs when applications pass raw user input to underlying operating system shells, enabling Remote Code Execution (RCE).
- **Server-Side Template Injection (SSTI):** Attackers exploit insecure template engines by embedding expressions that execute server-side code or read local configuration files.
- **XML External Entity (XXE) Injection:** Attackers manipulate uploaded XML files to force server-side XML parsers to disclose local file system assets or perform internal port scanning.
- **LDAP Injection:** Attackers tamper with directory queries to extract user permission structures or escalate privileges.

---

### 4. Vulnerable Dependencies

Modern applications rely on thousands of open-source third-party dependencies managed via `npm`, `pip`, or `maven`. Key risks include:

- **Supply Chain Risks:** Vulnerabilities in widespread third-party libraries (e.g., Log4Shell) expose every application consuming the package.
- **Transitive Dependencies:** Direct dependencies drag in complex trees of nested sub-dependencies, making manual vulnerability audits impossible.
- **Typosquatting & Dependency Confusion:** Attackers publish malicious packages with names visually similar to legitimate libraries, executing payload scripts during build execution.

---

### 5. Security Misconfigurations

Misconfigurations occur across web servers, application frameworks, databases, and cloud environments:

- Leaving default vendor accounts and credentials active.
- Exposing internal services (e.g., admin dashboards, Prometheus metrics endpoints) to the public internet without authentication.
- Returning overly verbose stack traces on client interfaces, disclosing software versions and database schemas.

---

### 6. Container Image Vulnerabilities

Container technologies (such as Docker) are foundational to cloud-native architectures but introduce specific risks:

- **Large Attack Surfaces:** Utilizing full-OS base images (such as Ubuntu or Debian) bloats images with unused OS utilities and unpatched CVEs.
- **Root Privileges:** Containers running processes as `root` (UID 0) enable container breakout attacks if the application is compromised.

---

### 7. IaC & Kubernetes Misconfigurations

Infrastructure as Code (IaC) files defined in YAML require strict auditing to prevent cluster-level security exposures:

- **Overpermissive RBAC:** Granting excessive API permissions to ServiceAccounts allows a compromised Pod to control cluster resources.
- **Missing Network Policies:** Unrestricted pod-to-pod communications allow lateral movement across worker nodes.
- **Missing Resource Limits:** Failing to enforce CPU/Memory limits exposes nodes to Denial of Service (DoS) resource exhaustion.
