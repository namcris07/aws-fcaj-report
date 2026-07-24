---
title: "Worklog Tuần 9"
date: 2026-08-10
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Mục tiêu tuần 9:

* Khởi tạo dự án Workshop Website trên Hugo framework (`FCAJ-workshop-template`), thiết lập menu, thanh điều hướng và giao diện song ngữ (Việt - Anh).
* Đưa toàn bộ tài liệu dự án (Student Info, Proposal, Worklog 9 tuần, 3 bài Blogs, Events Participated, Self-Evaluation & Sharing Feedback) lên website.
* Rà soát song ngữ chéo (VI/EN), kiểm thử toàn bộ liên kết/hình ảnh hiển thị, build sản phẩm production (`hugo --minify`), bàn giao báo cáo hoàn chỉnh, scale ECS Fargate về 0 (`desired-count 0`) và dọn dẹp tài nguyên trên AWS (ECR, S3, ALB) để tối ưu chi phí.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - **Khởi tạo Hugo Website & Menu Navigation:** <br>&emsp; + Nghiên cứu cấu trúc Hugo và khởi dựng codebase website local bằng lệnh `hugo server`. <br>&emsp; + Cấu hình `config.toml` và thiết lập thứ tự trọng số (weight) cho thanh điều hướng menu (Student Info, Worklog, Proposal, Blogs, Events, Self-evaluation, Sharing & Feedback). | 10/08/2026 | 10/08/2026 | [Handover Module](https://github.com/namcris07/aws-fcaj-learning/blob/main/Worklogs/Tu%E1%BA%A7n-09-Hugo-Website-Handover.md) |
| 3 | - **Đưa nội dung Info, Proposal & Worklog lên Site:** <br>&emsp; + Soạn thảo và đưa thông tin sinh viên (Student Information), đề xuất dự án (Proposal) và toàn bộ nhật ký công việc 9 tuần (Worklog Tuần 1 - 9) lên hệ thống nội dung `content/`. <br>&emsp; + Nhúng các sơ đồ kiến trúc hệ thống Serverless DevSecOps và sơ đồ luồng CI/CD dưới dạng hình ảnh chất lượng cao. | 11/08/2026 | 11/08/2026 | [Website Content Map](/1-worklog/) |
| 4 | - **Tích hợp Blogs, Events & Evaluation Pages:** <br>&emsp; + Đưa 3 bài Blog kỹ thuật (IAM/VPC/ECR, Jenkins CI/CD & ECS Fargate, CloudWatch/Cost & Prometheus/Grafana) kèm link/screenshot bài đăng thực tế lên mục Blogs. <br>&emsp; + Cập nhật thông tin các sự kiện ngoại khóa (Events Participated) cùng ảnh minh chứng. <br>&emsp; + Thu thập và cập nhật các trang Tự đánh giá (Self-evaluation) & Cảm nhận phản hồi (Sharing & Feedback) của cả nhóm. | 12/08/2026 | 12/08/2026 | [Blogs & Events](/4-eventparticipated/) |
| 5 | - **Rà soát Song ngữ & Kiểm thử UI/Links:** <br>&emsp; + Thực hiện so khớp chéo bản dịch tiếng Việt và tiếng Anh trên tất cả các trang, đảm bảo thuật ngữ kỹ thuật chính xác và chuẩn nghĩa. <br>&emsp; + Rà soát kiểm tra toàn bộ hyperlinks, shortcodes, bảng biểu và responsive UI trên các kích thước màn hình khác nhau. | 13/08/2026 | 13/08/2026 | [Bilingual Review](https://github.com/namcris07/aws-fcaj-report/blob/main/config.toml) |
| 6 | - **Build Production, Nộp Báo cáo & Dọn dẹp AWS:** <br>&emsp; + Chạy lệnh build Hugo production (`hugo --minify`), kiểm tra không còn lỗi render hay broken link. <br>&emsp; + Đóng gói và bàn giao báo cáo cuối khóa cùng slide thuyết trình cho ban tổ chức. <br>&emsp; + Thực hiện scale ECS Fargate services về 0 (`desired-count 0`) và dọn dẹp triệt để các tài nguyên AWS (ALB, ECR images, S3 buckets, VPC resources) để chấm dứt phát sinh chi phí. | 14/08/2026 | 14/08/2026 | [Final Handover](https://github.com/loi-bui0703/CICD-DevSecOps-using-AWS-services/blob/main/tasks.md) |

### Kết quả đạt được tuần 9:

* **Hoàn thiện Workshop Website Báo cáo Đồ án:**
  * Xây dựng thành công trang web workshop báo cáo đồ án trên Hugo framework đẹp mắt, chuẩn SEO, hỗ trợ song ngữ Việt - Anh hoàn chỉnh.
  * Tích hợp 100% tài liệu đồ án: Thông tin nhóm, Đề xuất dự án, Worklog 9 tuần, Workshop Labs step-by-step, 3 bài Blogs, Events ngoại khóa, Tự đánh giá và Phản hồi.

* **Nghiệm thu Sản phẩm & Dọn dẹp Tài nguyên Đám mây:**
  * Kiểm thử giao diện mượt mà, build thành công gói production không lỗi; hoàn tất đóng gói bàn giao đồ án đúng thời hạn 14/08/2026.
  * Thực hiện dọn dẹp sạch sẽ toàn bộ tài nguyên chạy trên AWS (scale ECS Fargate về 0, xóa Load Balancers, ECR repos), ngăn ngừa hoàn toàn rủi ro phát sinh chi phí bất ngờ.
