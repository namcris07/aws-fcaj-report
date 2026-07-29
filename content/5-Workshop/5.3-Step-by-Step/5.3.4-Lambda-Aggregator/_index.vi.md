---
title : "Khởi tạo Lambda Aggregator"
date : 2026-07-06 
weight : 4
chapter : false
pre : " <b> 5.3.4. </b> "
---

# 5.3.4 Khởi tạo AWS Lambda Security Report Aggregator

---

### 1. Lambda Function (`devsecops-factory-securityhub-importer`)

Tạo Lambda function `devsecops-factory-securityhub-importer` (Python 3.12) để tự động đọc báo cáo chuẩn hóa ASFF từ S3 Bucket và đẩy trực tiếp vào **AWS Security Hub** qua API `batch_import_findings`:

```python
import json
import logging
import os
import urllib.parse
import boto3

LOG = logging.getLogger()
LOG.setLevel(logging.INFO)

S3 = boto3.client("s3")
SECURITYHUB = boto3.client("securityhub", region_name=os.getenv("SECURITYHUB_REGION", "ap-southeast-1"))
ASFF_SUFFIX = os.getenv("ASFF_SUFFIX", "securityhub-asff.json")

def chunks(items, size):
    return [items[i : i + size] for i in range(0, len(items), size)]

def load_findings(bucket, key):
    response = S3.get_object(Bucket=bucket, Key=key)
    payload = json.loads(response["Body"].read())
    if isinstance(payload, list):
        return payload
    if isinstance(payload, dict):
        return payload.get("findings", [])
    return []

def import_findings(findings):
    imported = 0
    failed = 0
    for batch in chunks(findings, 100):
        result = SECURITYHUB.batch_import_findings(Findings=batch)
        imported += len(batch) - int(result.get("FailedCount", 0))
        failed += int(result.get("FailedCount", 0))
    return {"imported": imported, "failed": failed}

def lambda_handler(event, context):
    imported = 0
    for record in event.get("Records", []):
        bucket = record["s3"]["bucket"]["name"]
        key = urllib.parse.unquote_plus(record["s3"]["object"]["key"])
        if not key.endswith(ASFF_SUFFIX):
            continue
        findings = load_findings(bucket, key)
        result = import_findings(findings)
        imported += result["imported"]
    return {"importedFindings": imported}
```

---

### 2. Cấu hình S3 Event Notification Trigger

Cấu hình trigger S3 Notification trên bucket `devsecops-reports-*` khi có sự kiện `s3:ObjectCreated:*` tại thư mục `reports/asff/` với hậu tố `securityhub-asff.json`:

- **Event Type:** `s3:ObjectCreated:*`
- **Filter Prefix:** `reports/asff/`
- **Filter Suffix:** `securityhub-asff.json`
- **Target:** Lambda `devsecops-factory-securityhub-importer`

---

### Ảnh chụp màn hình thực tế: AWS Lambda Function & Security Hub Findings

![AWS Lambda Function](/images/5-Workshop/5.3-Step-by-Step/aws_lambda_function.png)
*Hình 5.3.4a: Màn hình quản lý AWS Lambda Function (`devsecops-securityhub-importer`) được liên kết với S3 Event Notification trigger.*

![AWS Security Hub Findings](/images/5-Workshop/5.3-Step-by-Step/aws_security_hub_findings.png)
*Hình 5.3.4b: Giao diện bảng điều khiển AWS Security Hub CSPM Findings ghi nhận danh sách lỗ hổng bảo mật tự động.*
