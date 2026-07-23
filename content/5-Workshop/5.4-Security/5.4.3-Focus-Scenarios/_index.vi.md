---
title : "Các tình huống chú trọng bảo mật"
date : 2026-07-06 
weight : 3
chapter : false
pre : " <b> 5.4.3. </b> "
---

# 5.4.3 Các tình huống cần đặc biệt chú trọng bảo mật

---

Trong suốt vòng đời phát triển và vận hành hệ thống DevSecOps trên nền tảng đám mây, các kịch bản rà soát và ứng phó an ninh cần đặc biệt kích hoạt trong những tình huống nhạy cảm sau đây:

---

### 1. Triển khai dịch vụ mới hoặc khởi tạo hạ tầng
- Kiểm tra kỹ lưỡng các tệp khai báo triển khai (Kubernetes manifests, Terraform scripts).
- Thiết lập giới hạn tài nguyên CPU và RAM chặt chẽ.
- Cấu hình chứng chỉ bảo mật đường truyền (TLS) và định tuyến rõ ràng các chính sách mạng.

---

### 2. Phát hành bản cập nhật và thay đổi mã nguồn
- Kích hoạt lại toàn bộ các trạm rà quét mã nguồn tĩnh (Gitleaks, SonarQube).
- Đánh giá lại các điểm tiếp nhận dữ liệu đầu vào.
- Kiểm tra nghiêm ngặt luồng xác thực và phân quyền API.

---

### 3. Tích hợp thành phần phụ thuộc hoặc ảnh nền mới
- Rà soát nguồn gốc phát hành và nhà cung cấp gói phần mềm.
- Đánh giá mức độ tin cậy của nhà cung cấp.
- Quét toàn diện các điểm yếu bảo mật (CVEs) liên quan đến thành phần mới bằng Trivy FS / Trivy Image.

---

### 4. Hệ thống giám sát phát hiện hành vi bất thường
- Khi bảng điều khiển trung tâm (CloudWatch / Grafana) ghi nhận lưu lượng mạng đột biến hoặc nỗ lực truy cập vượt quyền.
- Lập tức trích xuất nhật ký sự kiện chi tiết để điều tra nguyên nhân root cause.
- Cô lập vùng chứa (Container/Pod) bị nghi ngờ nhiễm mã độc ra khỏi mạng nội bộ.
