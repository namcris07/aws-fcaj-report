---
title: "Worklog Tuần 4"
date: 2026-07-06
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4:

* Nghiên cứu và tìm hiểu kiến thức về lưu trữ Docker Image trên Amazon ECR.
* Phối hợp xây dựng tài liệu quy trình phân phối ảnh (image delivery flow) và quản lý tag cho dự án.
* Thực hiện chụp ảnh minh chứng ECR phục vụ báo cáo.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - **Nghiên cứu Amazon ECR & Authentication:** <br>&emsp; + Tìm hiểu dịch vụ lưu trữ container image Amazon ECR (Elastic Container Registry) trên AWS. <br>&emsp; + Phân biệt Public và Private Repositories; tìm hiểu cơ chế bảo mật quét ảnh tự động khi push (Scan on Push). <br>&emsp; + Nghiên cứu phương pháp xác thực bảo mật IAM qua lệnh `aws ecr get-login-password` thông qua AWS CLI để đăng nhập docker client và push image an toàn. | 06/07/2026 | 06/07/2026 | [ECR Module](https://github.com/namcris07/aws-fcaj-learning/blob/main/Worklogs/Tu%E1%BA%A7n-04-Amazon-ECR.md) |
| 3 | - **Phối hợp quy trình đóng gói & lưu trữ:** <br>&emsp; + Hỗ trợ các thành viên thống nhất cấu trúc thư mục Dockerfile và Docker registry. <br>&emsp; + Phối hợp cùng Thành viên 1 và Thành viên 4 để vẽ sơ đồ luồng đẩy ảnh tự động (Image Delivery Flow) đi từ Jenkins CI, qua khâu quét bảo mật Trivy, đến Amazon ECR. <br>&emsp; + Xác định các biến môi trường cần dùng như `REGISTRY`, `REPO_NAME`, `IMAGE_NAME`, `IMAGE_TAG`. | 07/07/2026 | 07/07/2026 | [ECR Module](https://github.com/namcris07/aws-fcaj-learning/blob/main/Worklogs/Tu%E1%BA%A7n-04-Amazon-ECR.md) |
| 4 | - **Tài liệu hóa quy trình CI/CD ECR:** <br>&emsp; + Ghi nhận thông tin cấu hình ECR, thiết lập hướng dẫn đẩy Docker image với thẻ (tag) động theo định dạng Git Commit SHA để tránh ghi đè các bản build cũ. <br>&emsp; + Viết tài liệu giải thích chi tiết các tham số cấu hình trong tệp tin `ci/Jenkinsfile` và cách tích hợp ECR Credential vào Jenkins Credentials Provider. | 08/07/2026 | 08/07/2026 | [ECR Module](https://github.com/namcris07/aws-fcaj-learning/blob/main/Worklogs/Tu%E1%BA%A7n-04-Amazon-ECR.md) |
| 5 | - **Cập nhật báo cáo & Chụp minh chứng:** <br>&emsp; + Thực hiện chụp ảnh minh chứng hoạt động của ECR repository trên AWS Console. <br>&emsp; + Chụp lại các thông số quét lỗ hổng bảo mật của ECR (Scan Findings). <br>&emsp; + Cập nhật nội dung chi tiết cho phần "AWS Image Registry" vào hệ thống tài liệu chung của nhóm; cập nhật cấu trúc thư mục dự án liên quan đến phần `ci/` và ECR. | 09/07/2026 | 09/07/2026 | [ECR Module](https://github.com/namcris07/aws-fcaj-learning/blob/main/Worklogs/Tu%E1%BA%A7n-04-Amazon-ECR.md) |
| 6 | - **Thiết lập checklist ECR & Review:** <br>&emsp; + Lập danh sách test case kiểm tra tính sẵn sàng của ECR cho nhóm (khả năng login, pull, push từ local và từ Jenkins agent). <br>&emsp; + Thực hiện review và đánh giá tiến độ hoàn thành tuần 4, chuẩn bị các bước chuyển giao ảnh sang khâu deploy Kubernetes. | 10/07/2026 | 10/07/2026 | [ECR Module](https://github.com/namcris07/aws-fcaj-learning/blob/main/Worklogs/Tu%E1%BA%A7n-04-Amazon-ECR.md) |

### Kết quả đạt được tuần 4:

* **Làm chủ dịch vụ Amazon ECR và cơ chế phân quyền:**
  * Hiểu sâu về cơ chế hoạt động, phân quyền truy cập (IAM Policies, ECR Repository Policies) và bảo mật lưu trữ trên Amazon ECR.
  * Phân biệt rõ sự khác nhau giữa việc xác thực IAM Identity Center SSO và việc tạo token tạm thời qua AWS CLI phục vụ Docker login daemon.
  
* **Hoàn thiện tài liệu và quy trình tự động hóa đẩy ảnh:**
  * Phối hợp cùng nhóm thiết kế thành công tài liệu về sơ đồ luồng đẩy ảnh tự động từ Jenkins CI sang Amazon ECR.
  * Thống nhất được các tham số môi trường và đường dẫn ảnh trong `Jenkinsfile` giúp ngăn chặn rủi ro ghi đè image trong ECR.
  * Hoàn thành phần tài liệu hướng dẫn và chụp ảnh minh chứng ECR Repository hoạt động thực tế với các image tag theo Commit SHA của git.
