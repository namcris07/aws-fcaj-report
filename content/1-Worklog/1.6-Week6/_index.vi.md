---
title: "Worklog Tuần 6"
date: 2026-07-20
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Mục tiêu tuần 6:

* Tìm hiểu nguyên lý hoạt động của GitOps và Argo CD.
* Xây dựng quy trình kiểm thử GitOps auto-sync và manual approval gate.
* Chụp ảnh minh chứng Argo CD ứng dụng staging/production.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - **Nghiên cứu GitOps & Argo CD:** <br>&emsp; + Tìm hiểu nguyên lý hoạt động của GitOps trên Kubernetes theo mô hình kéo (Pull-based deployment). <br>&emsp; + Nghiên cứu cấu trúc khai báo Application CRD (Custom Resource Definition) của Argo CD để quản lý các môi trường. <br>&emsp; + So sánh sự khác nhau của các chiến lược đồng bộ (Manual Sync vs Automated Sync). | 20/07/2026 | 20/07/2026 | [GitOps Module](https://github.com/namcris07/aws-fcaj-learning/blob/main/Worklogs/Tu%E1%BA%A7n-06-GitOps-ArgoCD.md) |
| 3 | - **Thiết kế kiểm thử Staging Auto-Sync:** <br>&emsp; + Thiết kế test case kiểm thử quy trình tự động đồng bộ (Auto-sync) từ Git repo lên EKS staging namespace khi Jenkins đẩy thẻ image tag mới lên Git. <br>&emsp; + Xác định các tiêu chí trạng thái Synced (khớp manifest) và Healthy (ứng dụng chạy ổn định). | 21/07/2026 | 21/07/2026 | [Argo CD Documentation](https://argoproj.github.io/cd/) |
| 4 | - **Quy trình Manual Approval Gate:** <br>&emsp; + Soạn thảo tài liệu quy trình kiểm soát và phê duyệt thủ công trong pipeline Jenkins trước khi tiến hành cập nhật image tag cho môi trường Production. <br>&emsp; + Thiết lập kịch bản kiểm tra phê duyệt an toàn nhằm đảm bảo không có mã nguồn chưa qua kiểm định lọt vào production. | 22/07/2026 | 22/07/2026 | [Jenkins Approval](https://github.com/loi-bui0703/CICD-DevSecOps-using-AWS-services/blob/main/tasks.md) |
| 5 | - **Chụp minh chứng GitOps:** <br>&emsp; + Tiến hành chụp ảnh minh chứng trạng thái Argo CD app trên cả hai môi trường Staging và Production. <br>&emsp; + Ghi nhận logs hoạt động đồng bộ của Argo CD server; ghi nhận cách Jenkins ghi đè file `kustomization.yaml` bằng lệnh git commit trong pipeline. | 23/07/2026 | 23/07/2026 | [Argo CD UI](https://github.com/loi-bui0703/CICD-DevSecOps-using-AWS-services/blob/main/README.md) |
| 6 | - **Tổng hợp kiểm thử & Đánh giá tuần:** <br>&emsp; + Tổng hợp danh mục test case kiểm tra GitOps hoàn chỉnh. <br>&emsp; + Rà soát tính bảo mật của Git credentials (SSH Key/Personal Access Token) sử dụng trong Jenkins. <br>&emsp; + Đánh giá và tổng kết tiến độ tuần 6. | 24/07/2026 | 24/07/2026 | [Test Checklist](https://github.com/loi-bui0703/CICD-DevSecOps-using-AWS-services/blob/main/tasks.md) |

### Kết quả đạt được tuần 6:

* **Làm chủ quy trình phân phối liên tục GitOps:**
  * Hiểu sâu về mô hình GitOps kéo (pull-based deployment) và cách Argo CD đồng bộ file manifest từ Git repo lên EKS cluster.
  * Phân tích được các lợi thế của GitOps so với CD truyền thống trong việc ngăn chặn trôi lệch cấu hình (Configuration Drift).

* **Xây dựng quy trình kiểm soát chất lượng & an toàn:**
  * Thiết kế và xác thực thành công các test case cho luồng Auto-Sync Staging và Manual Approval Gate Production.
  * Chụp ảnh minh chứng giao diện Argo CD hiển thị trạng thái Synced và Healthy của cả hai môi trường, sẵn sàng tích hợp vào báo cáo.
  * Đảm bảo tính bảo mật cho credentials của Git không bị lộ lọt trong code Jenkinsfile.
