---
title : "Khởi tạo Lambda Aggregator"
date : 2026-07-06 
weight : 4
chapter : false
pre : " <b> 5.3.4. </b> "
---

# 5.3.4 Khởi tạo AWS Lambda Security Report Aggregator

---

Lambda function `devsecops-factory-securityhub-importer` (Python 3.12) tự động đọc báo cáo chuẩn hóa ASFF từ S3 Bucket và đẩy vào **AWS Security Hub** qua API `batch_import_findings`. Được quản lý bằng Terraform và kích hoạt bởi S3 Event Notification.

---

### 1. Kiến trúc Event-Driven

```text
Jenkins Stage 17: generate-asff.py
     ↓
Jenkins Stage 18: aws s3 cp securityhub-asff.json → s3://bucket/reports/asff/
     ↓
S3 Event Notification (ObjectCreated) → prefix: reports/asff/ | suffix: securityhub-asff.json
     ↓
Lambda: devsecops-factory-securityhub-importer
     ↓
SECURITYHUB.batch_import_findings() (batch 100 findings/lần)
     ↓
AWS Security Hub Dashboard
```

---

### 2. Cấu hình Terraform cho Lambda & S3 Trigger

```hcl
# infrastructure/terraform/main.tf

# S3 Event Notification kích hoạt Lambda khi có ASFF file
resource "aws_s3_bucket_notification" "securityhub_importer" {
  bucket = aws_s3_bucket.security_reports.id

  dynamic "lambda_function" {
    for_each = var.enable_security_hub_importer ? [1] : []
    content {
      lambda_function_arn = aws_lambda_function.securityhub_importer[0].arn
      events              = ["s3:ObjectCreated:*"]
      filter_prefix       = "reports/asff/"           # Chỉ theo dõi prefix này
      filter_suffix       = "securityhub-asff.json"   # Chỉ file có suffix này
    }
  }
  depends_on = [aws_lambda_permission.allow_s3]
}
```

> **Điều kiện kích hoạt Lambda:** Cần bật `enable_security_hub_importer = true` trong `terraform.tfvars` trước khi chạy `terraform apply`.

---

### 3. Lambda Handler (Python 3.12)

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

        # Bỏ qua nếu không phải file ASFF
        if not key.endswith(ASFF_SUFFIX):
            skipped += 1; continue

        # Tải findings từ S3
        response = S3.get_object(Bucket=bucket, Key=key)
        findings = json.loads(response["Body"].read())

        # Import vào Security Hub (batch 100 findings/lần)
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

### 4. Cấu hình S3 Event Notification Trigger (tóm tắt)

| Thuộc tính | Giá trị |
|---|---|
| **Event Type** | `s3:ObjectCreated:*` |
| **Filter Prefix** | `reports/asff/` |
| **Filter Suffix** | `securityhub-asff.json` |
| **Target** | Lambda `devsecops-factory-securityhub-importer` |
| **Runtime** | Python 3.12 |
| **Memory** | 128 MB |
| **Timeout** | 60 giây |

---

### 5. Tạo ASFF từ normalized findings (Stage 17)

Jenkins chạy script Python để chuyển đổi kết quả scan sang định dạng ASFF trước khi upload S3:

```bash
# Stage 17: Generate ASFF cho Security Hub
mkdir -p "${SCAN_REPORT_DIR}/asff"
AWS_ACCOUNT_ID="$(aws sts get-caller-identity --query Account --output text)"

python3 ci/stages/generate-asff.py \
  --input  "${SCAN_REPORT_DIR}/normalized/findings.json" \
  --out    "${SCAN_REPORT_DIR}/asff/securityhub-asff.json" \
  --region "${AWS_REGION}" \
  --account-id "${AWS_ACCOUNT_ID}"
```

---

### Ảnh chụp màn hình thực tế: Lambda & Security Hub

![AWS Lambda CloudWatch Logs](/images/5-Workshop/5.3-Step-by-Step/lambda_logs.png)
*Hình 5.3.4a: CloudWatch Logs của Lambda `devsecops-factory-securityhub-importer` hiển thị kết quả import findings thành công.*

![AWS Security Hub Findings](/images/5-Workshop/5.3-Step-by-Step/securityhub_findings.png)
*Hình 5.3.4b: AWS Security Hub Findings Dashboard ghi nhận danh sách lỗ hổng được import tự động từ Lambda.*

![AWS Security Hub Dashboard](/images/5-Workshop/5.3-Step-by-Step/securityhub_dashboard.png)
*Hình 5.3.4c: AWS Security Hub tổng quan Findings theo mức độ nghiêm trọng (Critical, High, Medium, Low).*
