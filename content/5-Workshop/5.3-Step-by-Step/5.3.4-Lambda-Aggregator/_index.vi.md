---
title : "Khởi tạo Lambda Aggregator"
date : 2026-07-06 
weight : 4
chapter : false
pre : " <b> 5.3.4. </b> "
---

# 5.3.4 Khởi tạo AWS Lambda Security Report Aggregator

---

### 1. Tạo Lambda Function (Python 3.11)
Tạo function `devsecops-report-aggregator` để tự động phân tích báo cáo khi có sự kiện S3 `ObjectCreated`:

```python
import json
import boto3

s3_client = boto3.client('s3')

def lambda_handler(event, context):
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    response = s3_client.get_object(Bucket=bucket, Key=key)
    report_data = json.loads(response['Body'].read().decode('utf-8'))
    
    critical_vulnerabilities = 0
    if 'Vulnerabilities' in report_data:
        for item in report_data['Vulnerabilities']:
            if item.get('Severity') == 'CRITICAL':
                critical_vulnerabilities += 1
                
    result = {
        "file": key,
        "critical_count": critical_vulnerabilities,
        "status": "REJECTED" if critical_vulnerabilities > 0 else "APPROVED"
    }
    
    print(f"Report Evaluation Result: {json.dumps(result)}")
    return {"statusCode": 200, "body": json.dumps(result)}
```

---

### 2. Cấu hình S3 Event Notification Trigger
Liên kết S3 Bucket với Lambda Function để tự động chạy khi có file báo cáo mới trong đường dẫn `reports/`.
