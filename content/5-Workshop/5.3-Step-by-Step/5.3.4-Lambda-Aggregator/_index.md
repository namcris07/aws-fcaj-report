---
title : "Provision Lambda Aggregator"
date : 2026-07-06 
weight : 4
chapter : false
pre : " <b> 5.3.4. </b> "
---

# 5.3.4 Provision AWS Lambda Security Report Aggregator

---

The AWS Lambda function `devsecops-factory-securityhub-importer` (Python 3.12) automatically reads normalized ASFF security reports from the Amazon S3 bucket and imports findings into **AWS Security Hub** via the `batch_import_findings` API. Managed entirely via Terraform and triggered via S3 Event Notifications.

---

### 1. Event-Driven Security Hub Integration Workflow

```text
Jenkins Stage 17: generate-asff.py
     ↓
Jenkins Stage 18: aws s3 cp securityhub-asff.json → s3://bucket/reports/asff/
     ↓
S3 Event Notification (ObjectCreated) → prefix: reports/asff/ | suffix: securityhub-asff.json
     ↓
Lambda: devsecops-factory-securityhub-importer
     ↓
SECURITYHUB.batch_import_findings() (batched at 100 findings/call)
     ↓
AWS Security Hub Dashboard
```

---

### 2. Terraform Configuration for Lambda & S3 Event Trigger

```hcl
# infrastructure/terraform/main.tf

# S3 Event Notification triggering Lambda upon ASFF report upload
resource "aws_s3_bucket_notification" "securityhub_importer" {
  bucket = aws_s3_bucket.security_reports.id

  dynamic "lambda_function" {
    for_each = var.enable_security_hub_importer ? [1] : []
    content {
      lambda_function_arn = aws_lambda_function.securityhub_importer[0].arn
      events              = ["s3:ObjectCreated:*"]
      filter_prefix       = "reports/asff/"           # Filter target prefix
      filter_suffix       = "securityhub-asff.json"   # Filter target suffix
    }
  }
  depends_on = [aws_lambda_permission.allow_s3]
}
```

> **Activation Flag:** Set `enable_security_hub_importer = true` in `terraform.tfvars` before executing `terraform apply`.

---

### 3. Lambda Security Hub Importer Code (Python 3.12)

```python
# infrastructure/lambda/securityhub_importer.py
import json, logging, os, urllib.parse, boto3

S3 = boto3.client("s3")
SECURITYHUB = boto3.client("securityhub",
    region_name=os.getenv("SECURITYHUB_REGION", "ap-southeast-1"))
ASFF_SUFFIX = os.getenv("ASFF_SUFFIX", "securityhub-asff.json")

def lambda_handler(event, context):
    imported = failed = skipped = 0

    for record in event.get("Records", []):
        bucket = record["s3"]["bucket"]["name"]
        key    = urllib.parse.unquote_plus(record["s3"]["object"]["key"])

        # Skip object if not matching ASFF suffix
        if not key.endswith(ASFF_SUFFIX):
            skipped += 1; continue

        # Fetch findings JSON from S3
        response = S3.get_object(Bucket=bucket, Key=key)
        findings = json.loads(response["Body"].read())

        # Batch import to Security Hub (max 100 findings per API call)
        for batch in [findings[i:i+100] for i in range(0, len(findings), 100)]:
            result = SECURITYHUB.batch_import_findings(Findings=batch)
            imported += len(batch) - result.get("FailedCount", 0)
            failed   += result.get("FailedCount", 0)

    return {
        "importedFindings": imported,
        "failedFindings":   failed,
        "skippedObjects":   skipped
    }
```

---

### 4. S3 Event Notification Trigger Specification

| Attribute | Value |
|---|---|
| **Event Type** | `s3:ObjectCreated:*` |
| **Filter Prefix** | `reports/asff/` |
| **Filter Suffix** | `securityhub-asff.json` |
| **Target Function** | Lambda `devsecops-factory-securityhub-importer` |
| **Runtime** | Python 3.12 |
| **Allocated Memory** | 128 MB |
| **Timeout Limit** | 60 seconds |

---

### 5. Generate ASFF from Normalized Findings (Stage 17)

Jenkins runs a Python script to convert raw scan findings into AWS Security Finding Format (ASFF) prior to S3 upload:

```bash
# Stage 17: Generate ASFF for Security Hub
mkdir -p "${SCAN_REPORT_DIR}/asff"
AWS_ACCOUNT_ID="$(aws sts get-caller-identity --query Account --output text)"

python3 ci/stages/generate-asff.py \
  --input  "${SCAN_REPORT_DIR}/normalized/findings.json" \
  --out    "${SCAN_REPORT_DIR}/asff/securityhub-asff.json" \
  --region "${AWS_REGION}" \
  --account-id "${AWS_ACCOUNT_ID}"
```

---

### Actual Screenshots: Lambda & Security Hub Integration

![AWS Lambda CloudWatch Logs](/images/5-Workshop/5.3-Step-by-Step/lambda_logs.png)
*Figure 5.3.4a: CloudWatch Logs for Lambda function `devsecops-factory-securityhub-importer` displaying successful batch finding ingestion.*

![AWS Security Hub Findings](/images/5-Workshop/5.3-Step-by-Step/securityhub_findings.png)
*Figure 5.3.4b: AWS Security Hub Findings Dashboard logging imported vulnerabilities automatically delivered by Lambda.*

![AWS Security Hub Dashboard](/images/5-Workshop/5.3-Step-by-Step/securityhub_dashboard.png)
*Figure 5.3.4c: AWS Security Hub overview aggregating findings categorized by severity level (Critical, High, Medium, Low).*
