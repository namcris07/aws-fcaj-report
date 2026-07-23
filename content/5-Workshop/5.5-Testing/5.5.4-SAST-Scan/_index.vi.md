---
title : "Stage 5 – Phân tích tĩnh (SAST Scan)"
date : 2026-07-06 
weight : 4
chapter : false
pre : " <b> 5.5.4. </b> "
---

# 5.5.4 Stage 5 – Phân tích tĩnh mã nguồn (SAST Scan)

---

### 1. Công cụ và cơ chế hoạt động

Static Application Security Testing (SAST) kiểm tra logic mã nguồn tự viết mà không cần thực thi chương trình. **SonarQube** được tích hợp qua `sonar-scanner-cli`, gửi kết quả phân tích về SonarQube Server.

---

### 2. Kịch bản tiêm lỗi và kết quả

Nhóm bổ sung 4 lỗi có chủ ý vào file `app/src/component/Game/index.js`:

#### Bảng 5.5.4: Chi tiết lỗi phát hiện bởi SAST Scan (SonarQube)

| Rule ID | Mô tả lỗi | Severity | Dòng lỗi | Ảnh hưởng |
|---|---|---|---|---|
| **S1763** | Unreachable code sau lệnh return | Medium | L317+ | Maintainability |
| **S2871** | `sort()` không có compare function | High | L397 | Reliability (Bug logic) |
| **S3776** | Cognitive Complexity = 18 > 15 | High | L326 | Maintainability |
| **S3358** | Nested ternary operator (3 cấp) | Medium | L317 | Maintainability |

---

### 3. Cách khắc phục

- **S2871:** Truyền so sánh số rõ ràng `numScores.sort((a, b) => a - b)`.
- **S1763:** Xóa bỏ mã dư thừa sau lệnh return.
- **S3776:** Tách hàm phức tạp thành các hàm nhỏ hơn.
- **S3358:** Thay toán tử lồng nhau bằng cấu trúc `if-else`.
