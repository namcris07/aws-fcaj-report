---
title : "Chuẩn hóa Báo cáo & Phân tích ROI Bảo mật"
date : 2026-07-06 
weight : 3
chapter : false
pre : " <b> 5.4.3. </b> "
---

# 5.4.3 Chuẩn hóa Báo cáo, ASFF & Phân tích Hiệu quả Bảo mật (ROI)

---

### 1. Chuẩn hóa Báo cáo Bảo mật (Normalize Schema)

Sáu công cụ bảo mật tạo ra 6 định dạng báo cáo JSON/XML/HTML khác nhau. Script `ci/stages/normalize-reports.py` thu gom tất cả và chuyển đổi về một schema chuẩn duy nhất:

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

### 2. Định dạng ASFF (Amazon Security Finding Format)

Script `ci/stages/generate-asff.py` tự động chuyển đổi normalized findings sang chuẩn ASFF để AWS Security Hub có thể tiếp nhận trực tiếp qua API `BatchImportFindings`:

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

### 3. Phân tích Hiệu quả Chi phí Bảo mật (Security ROI)

Toàn bộ 6 Security Gates trong hệ thống `devsecops-factory` đều sử dụng các công cụ mã nguồn mở (Open-Source Software - OSS), **không tốn bất kỳ chi phí bản quyền license nào**:

| Security Gate | Loại lỗ hổng phát hiện | Thời gian quét trung bình | Chi phí phần mềm |
|---|---|---|---|
| **Gate 1: Gitleaks** | Secrets, Hardcoded API Keys | ~30 giây | Miễn phí (OSS) |
| **Gate 2: Trivy FS** | CVEs trong npm packages | ~2 phút | Miễn phí (OSS) |
| **Gate 3: SonarQube** | Code Vulnerabilities, Bugs, Hotspots | ~3 phút | Miễn phí (Community Edition) |
| **Gate 4: Checkov** | IaC Misconfigurations (K8s, Docker) | ~1 phút | Miễn phí (OSS) |
| **Gate 5: Trivy Image** | CVEs trong Container Base Image | ~3 phút | Miễn phí (OSS) |
| **Gate 6: OWASP ZAP** | Runtime Web Vulnerabilities | ~10 phút | Miễn phí (OSS) |
| **TỔNG CỘNG** | **6 loại rủi ro bảo mật** | **~20 phút / build** | **$0 USD bản quyền** |

> **Lợi ích của tự động hóa bảo mật:** Chi phí duy nhất là thời gian máy chạy Jenkins (~20 phút/build). So sánh với chi phí khắc phục một sự cố rò rỉ dữ liệu hoặc lỗ hổng bảo mật sau khi đã phát hành trên môi trường Production (**gấp 1.000 lần** chi phí sửa lúc lập trình), đây là khoản đầu tư tự động hóa cực kỳ hiệu quả và thiết yếu đối với mọi doanh nghiệp.