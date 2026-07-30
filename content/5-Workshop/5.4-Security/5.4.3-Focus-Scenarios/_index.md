---
title : "Security Report Schema, ASFF & Security ROI Analysis"
date : 2026-07-06 
weight : 3
chapter : false
pre : " <b> 5.4.3. </b> "
---

# 5.4.3 Security Report Normalization, ASFF Schema & Security ROI Analysis

---

### 1. Security Report Normalization (Normalized Schema)

The six security scanners emit six different report output formats (JSON/XML/HTML). The `ci/stages/normalize-reports.py` script parses, aggregates, and transforms all raw scanner outputs into a unified schema:

```json
{
  "id": "gitleaks-001",
  "tool": "gitleaks",
  "severity": "HIGH",
  "title": "AWS Access Key detected",
  "description": "Hardcoded AWS_SECRET_ACCESS_KEY in config.js",
  "location": { "file": "app/src/config.js", "line": 3 },
  "metadata": {
    "commit": "abc123def456",
    "build": "42",
    "app": "tetris",
    "env": "staging"
  }
}
```

---

### 2. ASFF (AWS Security Finding Format) Generation

The `ci/stages/generate-asff.py` script automatically converts normalized security findings into standard ASFF schema, allowing AWS Security Hub to ingest findings directly via the `BatchImportFindings` API call:

```json
{
  "SchemaVersion": "2018-10-08",
  "Id": "tetris/gitleaks/abc123def456/gitleaks-001",
  "ProductArn": "arn:aws:securityhub:ap-southeast-1:585572506644:product/585572506644/default",
  "GeneratorId": "gitleaks-v8",
  "AwsAccountId": "585572506644",
  "Types": ["Software and Configuration Checks/Vulnerabilities/CVE"],
  "CreatedAt": "2026-07-29T10:00:00Z",
  "Severity": { "Label": "HIGH" },
  "Title": "Hardcoded AWS Access Key detected",
  "Resources": [{ "Type": "AwsEcsContainer", "Id": "tetris" }]
}
```

---

### 3. Security Cost Efficiency Analysis (Security ROI)

All 6 Security Gates across the `devsecops-factory` framework are built strictly using Open-Source Software (OSS), incurring **zero licensing overhead costs**:

| Security Gate | Target Risk Category | Average Scan Duration | Software License Cost |
|---|---|---|---|
| **Gate 1: Gitleaks** | Secrets & Hardcoded API Keys | ~30 seconds | Free (OSS) |
| **Gate 2: Trivy FS** | Dependency package CVEs | ~2 minutes | Free (OSS) |
| **Gate 3: SonarQube** | Code Vulnerabilities, Bugs, Hotspots | ~3 minutes | Free (Community Edition) |
| **Gate 4: Checkov** | IaC Misconfigurations (K8s, Docker) | ~1 minute | Free (OSS) |
| **Gate 5: Trivy Image** | Container Base Image CVEs | ~3 minutes | Free (OSS) |
| **Gate 6: OWASP ZAP** | Web Application Runtime Vulnerabilities | ~10 minutes | Free (OSS) |
| **TOTAL** | **6 Vulnerability Vectors Covered** | **~20 minutes / build** | **$0 USD Licensing** |

> **Automation ROI:** The sole operational expense is worker node compute time (~20 min/build). Remediating a breach or security flaw post-production release is estimated to cost **1,000 times more** than fixing it during initial development, making automated Shift-Left security pipelines a highly cost-effective investment.
