---
title : "Stage 4 – Quét key bí mật (Secrets Scan)"
date : 2026-07-06 
weight : 2
chapter : false
pre : " <b> 5.5.2. </b> "
---

# 5.5.2 Stage 4 – Quét key bí mật mã hoá cứng (Secrets Scan — Gitleaks)

---

### 1. Công cụ và cơ chế hoạt động

Công cụ **Gitleaks** được tích hợp tại Stage 4 để quét toàn bộ source code và lịch sử commit Git, phát hiện thông tin nhạy cảm bị hardcode như: API key, mật khẩu, token xác thực, private key, connection string, hoặc bất kỳ pattern nào khớp với cơ sở dữ liệu rule của Gitleaks (hơn 150 built-in rule pattern).

Cơ chế phân biệt exit code là điểm thiết kế quan trọng: **exit code 1** có nghĩa là Gitleaks chạy thành công và phát hiện secrets, trong khi **exit code khác 0 và khác 1** là lỗi hệ thống.

![Luồng xử lý exit code của Gitleaks trong pipeline](/images/5-Workshop/5.5-Testing/gitleaks_flowchart.png)

*Hình 5.5.2a: Luồng xử lý exit code của Gitleaks trong pipeline – phân biệt lỗi bảo mật và lỗi hệ thống.*

---

### 2. Kịch bản kiểm thử và kết quả

Để kiểm thử stage này, nhóm tiêm hai secret giả vào `app/src/config.js`:

```javascript
// app/src/config.js - Cấu hình dịch vụ
const API_CONFIG = {
  // [LỖI HARDCODE] GitHub Personal Access Token
  githubToken: "ghp_XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",

  // [LỖI HARDCODE] AWS Access Key ID
  awsAccessKeyId: "AKIAIOSFODNN7EXAMPLEFAKE",
};

export default API_CONFIG;
```

Khi pipeline chạy, Gitleaks phát hiện 2 secrets và báo cáo chi tiết vị trí trong từng commit:

```json
[
  {
    "Description": "GitHub Personal Access Token",
    "StartLine": 2,
    "File": "app/src/config.js",
    "Secret": "ghp_XXXXXX...",
    "RuleID": "github-pat"
  },
  {
    "Description": "AWS Access Key ID",
    "StartLine": 3,
    "File": "app/src/config.js",
    "Secret": "AKIAIOSFODNN7EXAMPLEFAKE",
    "RuleID": "aws-access-key-id"
  }
]
```

**Kết quả:** Pipeline trả về **exit code 1 (FAIL)** và dừng ngay tại Stage 4 trong vòng **30 giây**, không cho phép tiếp tục sang Stage 5 (SCA Scan) hay Stage 8 (Build Docker Image).

---

### 3. Phân tích rủi ro và cách khắc phục

- **Rủi ro:** Khi secrets được commit vào Git, nó vẫn tồn tại vĩnh viễn trong lịch sử Git dù đã bị xóa trong commit sau.
- **Giải pháp:** Secrets phải được đọc từ biến môi trường (`process.env`) và inject an toàn tại runtime thông qua Jenkins Credentials Store hoặc AWS Secrets Manager.
- **Custom Rule & Allowlist:** File `.gitleaks.toml` cho phép cấu hình custom rules và whitelist các false positives (ví dụ demo credentials trong tài liệu).