---
title: "Worklog Tuần 8"
date: 2026-08-03
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Mục tiêu tuần 8:

* Tập hợp toàn bộ kết quả kỹ thuật từ các phân hệ (VPC/ECS Fargate foundation, Jenkins CI/CD pipeline, Security gates, S3 reports & Lambda aggregator, Argo CD GitOps, Observability).
* Thiết kế slide thuyết trình báo cáo đồ án với bố cục chuẩn, sơ đồ kiến trúc hệ thống Serverless DevSecOps và 6 Security Gates trong pipeline.
* Soạn thảo cẩm nang hướng dẫn thực hành (Workshop Lab) chi tiết từng bước (Step-by-step), tích hợp code snippets mẫu (Dockerfile, Jenkinsfile, ECS Task Definition, Kustomization), hình ảnh minh chứng thực tế và kịch bản demo backup.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - **Thu thập & Đối chiếu số liệu kỹ thuật:** <br>&emsp; + Tập hợp báo cáo kết quả thực hiện chi tiết từ các thành viên (AWS ECS Fargate Infra, Jenkins CI/CD, Gitleaks/Trivy/SonarQube/OWASP security scans, S3 reports, Lambda aggregator). <br>&emsp; + Đối chiếu sự thống nhất giữa hạ tầng thực tế và sơ đồ logic; xác định các điểm sáng kỹ thuật (6 Security Gates, S3 Centralized Reports, ECS Fargate Serverless, GitOps Sync) cần nhấn mạnh trong bài báo cáo. | 03/08/2026 | 03/08/2026 | [Project Tasks](file:///d:/AWS%20FCAJ/CICD-DevSecOps-using-AWS-services/tasks.md) |
| 3 | - **Thiết kế Slide thuyết trình đồ án:** <br>&emsp; + Thiết kế bộ slide thuyết trình cho buổi bảo vệ đồ án, thống nhất bố cục và flow trình bày chung của nhóm. <br>&emsp; + Vẽ lại sơ đồ kiến trúc hệ thống tổng thể trên ECS Fargate và sơ đồ chi tiết 6 Security Gates tích hợp trong pipeline Jenkins. <br>&emsp; + Tùy biến slide với màu sắc chuyên nghiệp, hiện đại đồng nhất với giao diện website. | 04/08/2026 | 04/08/2026 | [Architecture & Pipeline](file:///d:/AWS%20FCAJ/CICD-DevSecOps-using-AWS-services/README.md) |
| 4 | - **Soạn thảo Workshop Lab Step-by-Step:** <br>&emsp; + Soạn thảo hướng dẫn thực hành chi tiết từng bước cho các bài Lab (Khởi tạo VPC & ECS Fargate cluster, Tạo S3 Report Bucket, Cài đặt Jenkins, Cấu hình Webhook, Deploy ứng dụng với ECS/Argo CD). <br>&emsp; + Đảm bảo văn bản hướng dẫn mạch lạc, có tính khả thi cao giúp người đọc dễ dàng tự thực hành tái lập (replicability). | 05/08/2026 | 05/08/2026 | [Workshop Structure](file:///d:/AWS%20FCAJ/fcj-workshop-template/content/5-Workshop/) |
| 5 | - **Tích hợp Code Snippets & Images cho Lab:** <br>&emsp; + Tích hợp các đoạn mã nguồn cấu hình mẫu thực tế (Jenkinsfile, Dockerfile, task-definition.json, kustomization.yaml, Helm values) vào bài hướng dẫn Lab với định dạng code block highlight. <br>&emsp; + Nhúng hình ảnh thực tế từ giao diện AWS Console, Jenkins, Argo CD, CloudWatch và bổ sung mục Kết quả mong đợi (Expected Results) ở cuối mỗi bước. | 06/08/2026 | 06/08/2026 | [Technical Codebase](file:///d:/AWS%20FCAJ/CICD-DevSecOps-using-AWS-services/) |
| 6 | - **Hoàn thiện Demo Script & Review:** <br>&emsp; + Xây dựng kịch bản thuyết trình và slide demo tĩnh (ảnh chụp từng bước pipeline thành công) làm phương án backup phòng ngừa sự cố khi demo live. <br>&emsp; + Phân chia vai trò thuyết trình cho từng thành viên trong buổi bảo vệ; rà soát lỗi chính tả và tổng kết tiến độ tuần 8. | 07/08/2026 | 07/08/2026 | [Project Roadmap](file:///d:/AWS%20FCAJ/CICD-DevSecOps-using-AWS-services/tasks.md) |

### Kết quả đạt được tuần 8:

* **Hoàn thiện Bộ Slide thuyết trình và Kịch bản trình bày:**
  * Thu thập đầy đủ số liệu, nhật ký quét bảo mật và hình ảnh thực tế từ các phân hệ infra, CI/CD, GitOps và Observability.
  * Hoàn thành slide thuyết trình đồ án với giao diện thẩm mỹ cao, sơ đồ kiến trúc mạch lạc và kịch bản demo tĩnh backup an toàn.

* **Biên soạn Cẩm nang Workshop Lab chi tiết từng bước:**
  * Hoàn thành biên soạn cẩm nang Workshop Lab từng bước từ cơ bản đến nâng cao, tích hợp đầy đủ các đoạn code snippets mẫu thực tế.
  * Nhúng hình ảnh giao diện minh chứng và phần kết quả mong đợi giúp người thực hành dễ dàng đối chiếu và tái lập thành công.
