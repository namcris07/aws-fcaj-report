---
title : "Các biện pháp bảo mật áp dụng"
date : 2026-07-06 
weight : 2
chapter : false
pre : " <b> 5.4.2. </b> "
---

# 5.4.2 Các biện pháp bảo mật áp dụng

---

Đối với các hệ thống hiện đại triển khai trên nền tảng đám mây, việc chỉ thiết lập một lớp rào chắn là không đủ. Hệ thống áp dụng mô hình **phòng thủ theo chiều sâu (Defense-in-Depth)**, tích hợp bảo mật vào từng tầng kiến trúc và xuyên suốt vòng đời phát triển phần mềm thông qua đường ống DevSecOps. Các biện pháp trọng tâm bao gồm:

---

### 1. Bảo mật tầng ứng dụng

- **Mã hóa đường truyền bằng giao thức HTTPS:** Thiết lập chứng chỉ SSL/TLS nhằm đảm bảo mọi dữ liệu trao đổi giữa trình duyệt và máy chủ đều được mã hóa toàn vẹn.
- **Kiểm duyệt và làm sạch dữ liệu đầu vào:** Xây dựng cơ chế kiểm tra khắt khe đối với mọi luồng dữ liệu trước khi đưa vào xử lý logic để vô hiệu hóa các đoạn mã độc hại (SQLi, XSS).
- **Kiểm soát truy cập và mã hóa định danh:** Áp dụng phân quyền rành mạch, mã hóa mật khẩu bằng các thuật toán băm một chiều mạnh mẽ (bcrypt, Argon2).
- **Bảo vệ biểu mẫu bằng mã thông báo duy nhất:** Tích hợp mã thông báo ngẫu nhiên (CSRF Tokens) trên các biểu mẫu quan trọng.

---

### 2. Bảo mật hạ tầng và môi trường vùng chứa

- **Rà soát an toàn môi trường vùng chứa:** Thực hiện quét tĩnh các ảnh nền (Base Images) trước khi triển khai, đồng thời buộc các dịch vụ phải chạy dưới quyền người dùng thông thường (`non-root user`).
- **Kiểm định cấu hình hạ tầng dưới dạng mã:** Áp dụng các công cụ rà quét tự động (Checkov) để phân tích tệp cấu hình triển khai nhằm ngăn chặn các thiết lập nguy hiểm.
- **Phân mảnh và kiểm soát mạng nội bộ:** Thiết lập các chính sách mạng nghiêm ngặt (Network Policies/Security Groups) để cô lập rủi ro theo chiều ngang.

---

### 3. Bảo mật trong quy trình phát triển và vận hành

- **Quản lý khóa bí mật tập trung:** Tích hợp công cụ quét tự động (Gitleaks) để chặn đứng việc rò rỉ khóa tại kho lưu trữ mã nguồn, đồng thời sử dụng Secrets Manager / Parameter Store.
- **Quản lý thành phần phụ thuộc:** Tự động hóa rà soát phụ thuộc (Trivy SCA) để loại bỏ các lỗ hổng đã bị công bố rộng rãi.
- **Giám sát hành vi và ghi nhật ký hệ thống:** Thu thập nhật ký sự kiện chi tiết (CloudWatch Logs) cho mọi hoạt động giao tiếp nhằm phát hiện hành vi bất thường.
