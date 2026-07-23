---
title : "Stage 3 – Quét key bí mật (Secrets Scan)"
date : 2026-07-06 
weight : 2
chapter : false
pre : " <b> 5.5.2. </b> "
---

# 5.5.2 Stage 3 – Quét key bí mật mã hoá cứng (Secrets Scan)

---

### 1. Công cụ và cơ chế hoạt động

Công cụ **Gitleaks** được tích hợp tại Stage 3 để quét toàn bộ source code và lịch sử commit Git, phát hiện thông tin nhạy cảm bị hardcode như: API key, mật khẩu, token xác thực, private key, connection string, hoặc bất kỳ pattern nào khớp với cơ sở dữ liệu rule của Gitleaks (hơn 150 built-in rule pattern).

Cơ chế phân biệt exit code là điểm thiết kế quan trọng: **exit code 1** có nghĩa là Gitleaks chạy thành công và phát hiện secrets, trong khi **exit code khác 0 và khác 1** là lỗi hệ thống.

![Luồng xử lý exit code của Gitleaks trong pipeline](/images/5-Workshop/5.5-Testing/gitleaks_flowchart.png)

*Hình 5.5.2: Luồng xử lý exit code của Gitleaks trong pipeline – phân biệt lỗi bảo mật và lỗi hệ thống.*

---

### 2. Kịch bản kiểm thử và kết quả

Để kiểm thử stage này, nhóm tiêm hai secret giả vào source code:

```javascript
// config.js - Cấu hình dịch vụ bên ngoài
const API_CONFIG = {
  // [LỖI] AWS Access Key - hardcoded
  awsAccessKeyId: "AKIAIOSFODNN7EXAMPLE",
  awsSecretAccessKey: "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",

  // [LỖI] Generic API Key pattern
  googleMapsApiKey: "AIzaSyD1234567890abcdefghijklmnopqrstuvwx",

  backendUrl: process.env.REACT_APP_API_URL || "http://localhost:3001",
};

export default API_CONFIG;
```

Khi pipeline chạy, Gitleaks phát hiện 2 secrets và báo cáo chi tiết vị trí trong từng commit:

```text
Finding: AKIAIOSFODNN7EXAMPLE
Secret: AKIAIOSFODNN7EXAMPLE
RuleID: aws-access-key
File: app/src/config.js

Finding: AIzaSyD1234567890abcdefghijklmnopqrstuvwx
Secret: AIzaSyD1234567890abcdefghijklmnopqrstuvwx
RuleID: google-api-key
File: app/src/config.js

[CRITICAL]: Hardcoded secrets detected by Gitleaks! Pipeline aborted.
```

**Kết quả:** Pipeline **FAIL (exit code 1)** và dừng ngay tại Stage 3.

---

### 3. Phân tích rủi ro và cách khắc phục

- **Rủi ro:** Khi secrets được commit vào Git, nó vẫn tồn tại vĩnh viễn trong lịch sử Git.
- **Giải pháp:** Secrets phải được đọc từ biến môi trường (`process.env.REACT_APP_API_URL`) và inject an toàn tại runtime thông qua Jenkins Credentials Store hoặc Kubernetes Secrets.
