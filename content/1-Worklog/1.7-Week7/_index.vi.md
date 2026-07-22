---
title: "Worklog Tuần 7"
date: 2026-07-27
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Mục tiêu tuần 7:

* Thiết lập và rà soát giám sát logs/metrics của EKS cluster bằng CloudWatch Container Insights.
* Tổng hợp hình ảnh giám sát, các cảnh báo chi phí (AWS Budget) và viết tài liệu Observability.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - **Nghiên cứu CloudWatch Observability:** <br>&emsp; + Tìm hiểu dịch vụ CloudWatch và các tính năng giám sát Container Insights trên AWS. <br>&emsp; + Nghiên cứu cơ chế hoạt động của CloudWatch agent chạy dưới dạng DaemonSet trên EKS và Fluent Bit thu thập log container chuyển tiếp về CloudWatch Logs. | 27/07/2026 | 27/07/2026 | [CloudWatch Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights.html) |
| 3 | - **Cấu hình Container Insights:** <br>&emsp; + Phối hợp với Thành viên 1 để triển khai và kích hoạt CloudWatch Observability add-on cho EKS. <br>&emsp; + Xác nhận các Log Groups liên quan đến cluster, node, pod và log ứng dụng bắt đầu đổ về AWS CloudWatch console. | 28/07/2026 | 28/07/2026 | [Observability Config](file:///d:/AWS%20FCAJ/CICD-DevSecOps-using-AWS-services/tasks.md) |
| 4 | - **Xây dựng Log query & Dashboard:** <br>&emsp; + Thiết lập các câu truy vấn CloudWatch Logs Insights để lọc nhanh log ứng dụng web (truy vấn lỗi, thống kê mã trạng thái HTTP). <br>&emsp; + Hỗ trợ xây dựng giao diện hiển thị metric hiệu năng CPU/RAM, Network I/O của Pod/Node phục vụ giám sát vận hành. | 29/07/2026 | 29/07/2026 | [AWS Observability](https://github.com/namcris07/aws-fcaj-learning/) |
| 5 | - **Chụp ảnh minh chứng giám sát:** <br>&emsp; + Chụp lại giao diện Dashboard Container Insights hiển thị biểu đồ CPU/Memory, EKS log streams. <br>&emsp; + Ghi nhận ảnh chụp cấu hình AWS Budget và các cảnh báo chi phí thực tế ở mức 50%, 80%, 100% để bổ sung vào báo cáo. | 30/07/2026 | 30/07/2026 | [Observability Dashboard](file:///d:/AWS%20FCAJ/CICD-DevSecOps-using-AWS-services/README.md) |
| 6 | - **Hoàn thiện tài liệu Observability:** <br>&emsp; + Biên soạn nội dung chi tiết cho phần Observability (giám sát ứng dụng và tài nguyên) trong tài liệu báo cáo. <br>&emsp; + Rà soát giải pháp tối ưu chi phí hạ tầng và dọn dẹp tài nguyên sau khi kết thúc dự án. <br>&emsp; + Tổng kết tuần 7. | 31/07/2026 | 31/07/2026 | [Observability Section](file:///d:/AWS%20FCAJ/CICD-DevSecOps-using-AWS-services/tasks.md) |

### Kết quả đạt được tuần 7:

* **Xây dựng giải pháp giám sát tập trung CloudWatch:**
  * Kích hoạt và cấu hình thành công CloudWatch Container Insights trên EKS cluster để tự động thu thập chỉ số hiệu năng Node/Pod.
  * Xây dựng thành công các câu truy vấn logs và dashboard giám sát tài nguyên CPU/Memory trực quan trên CloudWatch Console.

* **Hoàn thiện minh chứng Observability & Quản lý chi phí:**
  * Thu thập đầy đủ minh chứng ảnh chụp màn hình CloudWatch Dashboard, Log group và cấu hình AWS Budget ở các hạn mức cảnh báo.
  * Hoàn tất phần tài liệu Observability phục vụ báo cáo đồ án, thiết lập kế hoạch tối ưu và dọn dẹp tài nguyên AWS sau dự án để tránh phát sinh chi phí thừa.
