---
title : "Bối cảnh & Mục tiêu dự án"
date : 2026-07-06 
weight : 1
chapter : false
pre : " <b> 5.1.1. </b> "
---

# 5.1.1 Bối cảnh, Bài toán & Mục tiêu Dự án

---

### 1. Bối cảnh
Trong các tổ chức phát triển phần mềm hiện đại, quy trình phát triển và triển khai (CI/CD) diễn ra với tốc độ ngày càng nhanh. Tuy nhiên, yếu tố kiểm tra bảo mật thường bị xem nhẹ hoặc thực hiện thủ công ở giai đoạn cuối trước khi ra mắt sản phẩm (Late-stage Security Gate). Điều này dẫn đến việc phát hiện lỗ hổng muộn, phát sinh chi phí khắc phục khổng lồ và tăng nguy cơ rò rỉ dữ liệu nghiêm trọng trên môi trường Production.

### 2. Khách hàng & Đối tượng sử dụng
- Các đội ngũ kỹ sư phần mềm (Developers), kỹ sư bảo mật (Security Engineers) và đội ngũ vận hành hạ tầng (DevOps/SRE).
- Các doanh nghiệp vừa và nhỏ (SMEs) cần xây dựng quy trình CI/CD tự động hóa bảo mật với chi phí tối ưu trên đám mây AWS.

### 3. Bài toán cần giải quyết
- **Triển khai thủ công & Thiếu nhất quán:** Thiếu quy trình đóng gói container tự động và đồng bộ giữa các môi trường.
- **Rò rỉ thông tin nhạy cảm:** Lập trình viên vô tình đóng cứng (hardcode) AWS Access Keys, API Keys hoặc mật khẩu vào kho mã nguồn Git.
- **Sử dụng thư viện & Base Image có lỗ hổng:** Tích hợp các dependency mã nguồn mở chứa CVEs chưa qua rà quét.
- **Chi phí hạ tầng quá cao:** Duy trì các cụm Kubernetes (EKS) tĩnh gây tốn kém ngân sách lớn (~$72/tháng cố định cho Control Plane).

---

### 4. Mục tiêu dự án & Sản phẩm bàn giao
- **Pipeline CI/CD 12 Stages:** Tự động hóa kiểm định qua 6 Security Gates (Gitleaks, Trivy SCA, SonarQube, Checkov, Trivy Image, OWASP ZAP).
- **Hệ thống lưu trữ & Tổng hợp Báo cáo Bảo mật:** Lưu trữ tập trung báo cáo JSON/HTML trên **Amazon S3** và phân tích tự động qua **AWS Lambda Aggregator**.
- **Môi trường Serverless Container:** Triển khai tự động lên **Amazon ECS Fargate** cho Staging và kiểm soát phê duyệt thủ công (Manual Approval) cho Production.
- **Hệ thống Giám sát Tập trung:** Quan sát log/metrics qua **Amazon CloudWatch Container Insights** và thiết lập cảnh báo ngân sách.

---

### 5. Tiêu chí đánh giá thành công
- 100% mã nguồn và container image được kiểm định tự động trước khi triển khai.
- Chặn đứng (FAIL-FAST) pipeline ngay lập tức nếu phát hiện lộ lọt Secrets hoặc CVEs mức Critical/High.
- Tối ưu chi phí hạ tầng AWS xuống dưới **$20.50 USD/tháng** (tiết kiệm > 85% so với EKS).
