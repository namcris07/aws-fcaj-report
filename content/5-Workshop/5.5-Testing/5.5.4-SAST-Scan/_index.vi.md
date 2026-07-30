---
title : "Stage 6 – Phân tích tĩnh (SAST Scan)"
date : 2026-07-06 
weight : 4
chapter : false
pre : " <b> 5.5.4. </b> "
---

# 5.5.4 Stage 6 — Phân tích tĩnh mã nguồn (SAST Scan — SonarQube)

---

### 1. Công cụ và cơ chế hoạt động

Static Application Security Testing (SAST) kiểm tra logic mã nguồn tự viết mà không cần thực thi chương trình. **SonarQube** được tích hợp qua `sonar-scanner-cli`, gửi kết quả phân tích về SonarQube Server (chạy local tại http://localhost:9000).

Kết quả scan được lưu vào `scan-reports/sonar-issues.json` và upload lên S3 `reports/sast/`.

---

### 2. Lỗ hổng được thiết kế trong mã React

File `app/src/App.js` chứa lỗi XSS được cố ý để SonarQube phát hiện:

```jsx
// BAD: dangerouslySetInnerHTML mà không sanitize input → SonarQube S5247
function GameInfo({ playerName }) {
  return (
    <div
      dangerouslySetInnerHTML={{
        __html: `Chào mừng ${playerName}!`  // [S5247] XSS risk
      }}
    />
  );
}
```

#### Bảng 5.5.4: Chi tiết lỗi phát hiện bởi SAST Scan (SonarQube)

| Rule ID | Mô tả lỗi | Severity | Loại |
|---|---|---|---|
| **S5247** | `dangerouslySetInnerHTML` XSS vulnerability | HIGH | Vulnerability |
| **S2703** | `eval()` sử dụng mà không validate input | HIGH | Security Hotspot |
| **S1763** | Unreachable code sau lệnh `return` | MEDIUM | Code Smell |
| **S3776** | Cognitive Complexity = 18 > 15 | MEDIUM | Code Smell |

**Kết quả:** SonarQube Quality Gate **FAILED** — 2 Vulnerabilities vượt ngưỡng cho phép.

---

### 3. Ảnh minh chứng thực tế: SonarQube

![SonarQube Vulnerabilities Dashboard](/images/5-Workshop/5.5-Testing/sonarqube_vulnerabilities.png)
*Hình 5.5.4a: SonarQube Dashboard hiển thị các lỗ hổng (Vulnerabilities) được phát hiện trong mã nguồn React.*

![SonarQube Quality Gate FAILED](/images/5-Workshop/5.5-Testing/sonarqube_quality_gate_fail.png)
*Hình 5.5.4b: SonarQube Quality Gate báo FAILED do vi phạm ngưỡng bảo mật — pipeline dừng tại Stage 6.*

---

### 4. Cách khắc phục

- **S5247 (XSS):** Thay `dangerouslySetInnerHTML` bằng ````textContent` hoặc dùng thư viện sanitize (DOMPurify).
- **S2703 (eval):** Loại bỏ `eval()` hoàn toàn, thay bằng ```json.parse()` hoặc cấu trúc điều kiện an toàn.
- **S1763:** Xóa mã dư thừa sau lệnh `return`.
- **S3776:** Tách hàm phức tạp thành các hàm nhỏ hơn, tối đa 15 cognitive complexity.
