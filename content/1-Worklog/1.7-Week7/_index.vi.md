---
title: "Worklog Tuần 7"
date: 2026-07-27
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Mục tiêu tuần 7:

* Thiết lập và rà soát giám sát logs/metrics của cụm **Amazon ECS Fargate** bằng **CloudWatch Container Insights**.
* Tổng hợp hình ảnh giám sát, các cảnh báo chi phí (AWS Budget) và viết tài liệu Observability.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - **Nghiên cứu CloudWatch Observability cho ECS:** <br>&emsp; + Tìm hiểu dịch vụ CloudWatch và tính năng giám sát Container Insights cho Amazon ECS Fargate. <br>&emsp; + Nghiên cứu cơ chế tự động đẩy log ứng dụng từ ECS Fargate qua driver `awslogs` đổ trực tiếp về CloudWatch Log Groups. | 27/07/2026 | 27/07/2026 | [CloudWatch Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights.html) |
| 3 | - **Cấu hình Container Insights cho ECS Cluster:** <br>&emsp; + Phối hợp với Thành viên 1 chạy lệnh `aws ecs update-cluster-settings` kích hoạt Container Insights cho ECS Cluster `devsecops-factory`. <br>&emsp; + Xác nhận các Log Groups liên quan đến ECS task, service và application container bắt đầu đổ về AWS CloudWatch console. | 28/07/2026 | 28/07/2026 | [Observability Config](https://github.com/loi-bui0703/CICD-DevSecOps-using-AWS-services/blob/main/tasks.md) |
| 4 | - **Xây dựng Log query & Dashboard:** <br>&emsp; + Thiết lập các câu truy vấn CloudWatch Logs Insights để lọc nhanh log ứng dụng web React (truy vấn lỗi, thống kê mã trạng thái HTTP). <br>&emsp; + Hỗ trợ xây dựng giao diện hiển thị metric hiệu năng CPU/RAM, Network I/O của ECS Task phục vụ giám sát vận hành. | 29/07/2026 | 29/07/2026 | [Observability Module](https://github.com/namcris07/aws-fcaj-learning/blob/main/Worklogs/Tu%E1%BA%A7n-07-CloudWatch-Observability.md) |
| 5 | - **Chụp ảnh minh chứng giám sát:** <br>&emsp; + Chụp lại giao diện Dashboard Container Insights hiển thị biểu đồ CPU/Memory, ECS log streams. <br>&emsp; + Ghi nhận ảnh chụp cấu hình AWS Budget và các cảnh báo chi phí thực tế ở mức 50%, 80%, 100% để bổ sung vào báo cáo. | 30/07/2026 | 30/07/2026 | [Observability Dashboard](https://github.com/loi-bui0703/CICD-DevSecOps-using-AWS-services/blob/main/README.md) |
| 6 | - **Hoàn thiện tài liệu Observability & Teardown Plan:** <br>&emsp; + Biên soạn nội dung chi tiết cho phần Observability (giám sát ứng dụng và tài nguyên) trong tài liệu báo cáo. <br>&emsp; + Rà soát giải pháp tối ưu chi phí hạ tầng và quy trình scale ECS Fargate về `0` (`desired-count 0`) sau khi kết thúc dự án. <br>&emsp; + Tổng kết tuần 7. | 31/07/2026 | 31/07/2026 | [Observability Section](https://github.com/loi-bui0703/CICD-DevSecOps-using-AWS-services/blob/main/tasks.md) |

### Kết quả đạt được tuần 7:

* **Xây dựng giải pháp giám sát tập trung CloudWatch cho ECS Fargate:**
  * Kích hoạt và cấu hình thành công CloudWatch Container Insights trên ECS Cluster để tự động thu thập chỉ số hiệu năng CPU/Memory của các ECS Tasks.
  * Xây dựng thành công các câu truy vấn logs và dashboard giám sát tài nguyên trực quan trên CloudWatch Console.

* **Hoàn thiện minh chứng Observability & Quản lý chi phí:**
  * Thu thập đầy đủ minh chứng ảnh chụp màn hình CloudWatch Dashboard, Log group và cấu hình AWS Budget ở các hạn mức cảnh báo.
  * Hoàn tất phần tài liệu Observability phục vụ báo cáo đồ án, thiết lập kế hoạch tối ưu và dọn dẹp tài nguyên AWS sau dự án để tránh phát sinh chi phí thừa.
