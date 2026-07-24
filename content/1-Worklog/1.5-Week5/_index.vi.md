---
title: "Worklog Tuần 5"
date: 2026-07-13
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5:

* Khởi tạo hạ tầng Serverless Container **Amazon ECS Fargate** (thay thế EKS để tối ưu chi phí ~$72/tháng) và cấu hình 2 ECS Services (`tetris-staging` và `tetris-production`).
* Tạo **Amazon S3 Report Bucket** cấu hình mã hóa SSE-S3 & Versioning lưu kết quả scan an ninh tập trung (`reports/secrets/`, `reports/sca/`, `reports/sast/`, `reports/container/`, `reports/dast/`).
* Triển khai hàm **AWS Lambda Aggregator** tự động đọc và phân tích file JSON report khi S3 nhận được file mới.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - **Nghiên cứu Amazon ECS Fargate Architecture:** <br>&emsp; + Tìm hiểu mô hình tính toán Serverless Container Amazon ECS Fargate (chạy container không cần quản lý máy chủ EC2 hay EKS Control Plane). <br>&emsp; + Phân tích lý do lựa chọn ECS Fargate giúp tối ưu hoàn toàn chi phí cố định EKS cluster (~$72/tháng) cho đồ án sinh viên. <br>&emsp; + Nghiên cứu cơ chế phân quyền Task Execution Role và Task Role cho container. | 13/07/2026 | 13/07/2026 | [ECS Fargate Tasks](file:///d:/AWS%20FCAJ/CICD-DevSecOps-using-AWS-services/tasks.md) |
| 3 | - **Khởi tạo Amazon ECS Cluster & Services:** <br>&emsp; + Khởi tạo ECS Cluster với Fargate Capacity Provider. <br>&emsp; + Cấu hình 2 ECS Services: `tetris-staging` và `tetris-production`. <br>&emsp; + Đính kèm Application Load Balancer (ALB) định tuyến cổng HTTP 8080 của ứng dụng Web React Nginx. | 14/07/2026 | 14/07/2026 | [ECS Module](https://github.com/namcris07/aws-fcaj-learning/blob/main/Worklogs/Tu%E1%BA%A7n-05-Amazon-ECS-Fargate.md) |
| 4 | - **Tạo S3 Security Report Bucket:** <br>&emsp; + Tạo S3 bucket lưu kết quả quét an ninh tập trung từ pipeline Jenkins. <br>&emsp; + Phân chia thư mục prefix: `reports/secrets/`, `reports/sca/`, `reports/sast/`, `reports/container/`, `reports/dast/`. <br>&emsp; + Bật mã hóa SSE-S3, Versioning và Lifecycle Policy tự dọn dẹp report sau 30 ngày. | 15/07/2026 | 15/07/2026 | [S3 Reports Config](file:///d:/AWS%20FCAJ/CICD-DevSecOps-using-AWS-services/tasks.md) |
| 5 | - **Phát triển AWS Lambda Aggregator Function:** <br>&emsp; + Viết script xử lý tự động chạy trên AWS Lambda (Python runtime). <br>&emsp; + Cấu hình S3 Event Notification trigger Lambda khi có file JSON report mới upload lên bucket. <br>&emsp; + Parse dữ liệu báo cáo lỗ hổng High/Critical và đẩy log phân tích về AWS CloudWatch Logs. | 16/07/2026 | 16/07/2026 | [Lambda Security Aggregator](file:///d:/AWS%20FCAJ/CICD-DevSecOps-using-AWS-services/README.md) |
| 6 | - **Kiểm thử Triển khai & Ghi nhận Quy trình Scaling:** <br>&emsp; + Kiểm thử deploy ECS Task thành công trên môi trường staging qua ALB URL. <br>&emsp; + Ghi nhận quy trình scale ECS Fargate service về `0` (`desired-count 0`) sau khi kết thúc demo để chấm dứt hoàn toàn chi phí. <br>&emsp; + Đánh giá tổng kết tuần 5. | 17/07/2026 | 17/07/2026 | [ECS Teardown Guide](file:///d:/AWS%20FCAJ/CICD-DevSecOps-using-AWS-services/tasks.md) |

### Kết quả đạt được tuần 5:

* **Làm chủ hạ tầng Serverless Container Amazon ECS Fargate:**
  * Triển khai thành công cụm ECS Fargate Cluster và 2 dịch vụ `tetris-staging`, `tetris-production` đi kèm Application Load Balancer (ALB).
  * Giải quyết hoàn hảo bài toán tối ưu ngân sách sinh viên nhờ thay thế EKS Control Plane tốn phí cố định bằng mô hình ECS Fargate Serverless.

* **Xây dựng giải pháp lưu trữ & xử lý báo cáo an ninh tập trung:**
  * Khởi tạo S3 Security Report Bucket phân chia thư mục chuẩn hóa cho 6 lớp kiểm thử an toàn thông tin.
  * Xây dựng và kích hoạt thành công hàm AWS Lambda Aggregator tự động phân tích lỗ hổng an ninh từ S3 và xuất nhật ký kiểm toán về CloudWatch Logs.
