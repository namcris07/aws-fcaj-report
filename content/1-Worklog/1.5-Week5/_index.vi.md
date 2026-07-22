---
title: "Worklog Tuần 5"
date: 2026-07-13
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5:

* Tìm hiểu về triển khai ứng dụng trên Kubernetes/EKS.
* Xây dựng bộ test case kiểm thử quy trình deploy ứng dụng (Pod status, Ingress routing).
* Thu thập tài liệu cấu hình hạ tầng EKS từ Thành viên 1.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - **Nghiên cứu EKS & ALB Controller:** <br>&emsp; + Tìm hiểu kiến trúc dịch vụ Amazon Elastic Kubernetes Service (EKS) điều phối container. <br>&emsp; + Nghiên cứu cơ chế OIDC Provider cho EKS Service Account. <br>&emsp; + Tìm hiểu cơ chế hoạt động của AWS Load Balancer Controller kết hợp với ALB (Application Load Balancer) để tự động tạo Ingress định tuyến Layer 7 cho các namespace. | 13/07/2026 | 13/07/2026 | [EKS & K8s](https://000062.awsstudygroup.com/) |
| 3 | - **Thiết kế kiểm thử deploy EKS:** <br>&emsp; + Thiết kế tài liệu test case chi tiết cho việc deploy ứng dụng web lên Kubernetes/EKS. <br>&emsp; + Lập danh sách các trường hợp kiểm thử (test scenarios) trạng thái Pod (Running, Ready, CrashLoopBackOff), tính năng tự phục hồi (Liveness/Readiness Probes), khả năng giao tiếp nội bộ qua Service, và định tuyến Ingress routing. | 14/07/2026 | 14/07/2026 | [QA Guides](https://github.com/namcris07/aws-fcaj-learning/) |
| 4 | - **Phối hợp hạ tầng EKS:** <br>&emsp; + Phối hợp với Thành viên 1 ghi nhận thông tin cấu hình EKS node group (instance type, scaling limits), VPC subnets cho cluster, và phân quyền IAM role. <br>&emsp; + Tài liệu hóa cấu hình hạ tầng EKS vào báo cáo chung để đảm bảo tài liệu phản ánh chính xác hạ tầng thật được tạo từ Terraform/AWS CLI. | 15/07/2026 | 15/07/2026 | [EKS Configuration](file:///d:/AWS%20FCAJ/CICD-DevSecOps-using-AWS-services/tasks.md) |
| 5 | - **Thu thập minh chứng deploy:** <br>&emsp; + Chụp ảnh minh chứng Ingress ALB và service hoạt động thực tế trên EKS. <br>&emsp; + Chụp lại kết quả chạy lệnh `kubectl get nodes`, `kubectl get pods -A` trên terminal. <br>&emsp; + Tổng hợp các lỗi phát sinh trong quá trình deploy các manifest staging như lệch container port, lỗi pull image hoặc thiếu annotations cho ALB Ingress. | 16/07/2026 | 16/07/2026 | [EKS Deployment](file:///d:/AWS%20FCAJ/CICD-DevSecOps-using-AWS-services/README.md) |
| 6 | - **Hoàn thiện kết quả kiểm thử:** <br>&emsp; + Hoàn thiện checklist và kết quả kiểm thử deploy ứng dụng trên môi trường EKS staging. <br>&emsp; + Kiểm tra đối chiếu port của ứng dụng và port khai báo trong Service/Ingress (8080 vs 80). <br>&emsp; + Đánh giá tổng kết tuần 5. | 17/07/2026 | 17/07/2026 | [Test Case Templates](file:///d:/AWS%20FCAJ/CICD-DevSecOps-using-AWS-services/tasks.md) |

### Kết quả đạt được tuần 5:

* **Nắm vững kiến thức vận hành Kubernetes trên EKS:**
  * Hiểu rõ cách thức hoạt động của EKS cluster, node groups và sự bổ trợ của OIDC Provider trong việc phân quyền hạt mịn (IAM Roles for Service Accounts - IRSA).
  * Nắm chắc cơ chế Ingress Controller và cách AWS Load Balancer Controller lắng nghe các cấu hình ingress trong Kubernetes để sinh Application Load Balancer thực tế trên AWS.

* **Xây dựng hệ thống kiểm thử triển khai hạ tầng:**
  * Hoàn thành bộ test case kiểm thử toàn diện quy trình triển khai ứng dụng trên EKS.
  * Phối hợp thành công cùng nhóm ghi nhận và sửa đổi các lỗi liên quan đến cổng container (container port) lệch nhau giữa Nginx serve (8080) và K8s service (80).
  * Chụp ảnh minh chứng hoạt động của EKS Nodes ở trạng thái Ready và Ingress ALB tạo thành công trên AWS Console.
