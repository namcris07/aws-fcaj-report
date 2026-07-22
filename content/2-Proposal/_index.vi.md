---
title: "Bản đề xuất"
date: 2026-07-06
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# DevSecOps Factory trên AWS
## Xây dựng hệ thống CI/CD DevSecOps end-to-end cho ứng dụng Web React trên Amazon ECS Fargate

### 1. Tóm tắt điều hành
DevSecOps Factory là một hệ thống CI/CD hoàn chỉnh tích hợp bảo mật vào mọi giai đoạn của vòng đời phát triển phần mềm (Shift-Left Security). Dự án xây dựng pipeline tự động hóa từ lúc developer push code cho đến khi ứng dụng được triển khai lên môi trường staging và production trên **Amazon ECS Fargate**, sử dụng Jenkins làm CI engine, Argo CD (local k3d) cho GitOps, Amazon S3 lưu trữ báo cáo quét bảo mật, AWS Lambda tổng hợp lỗ hổng, và Amazon CloudWatch làm nền tảng giám sát tập trung. Hệ thống áp dụng chiến lược **Local-first, Cloud-after** — phát triển và kiểm thử toàn bộ trên máy cá nhân với k3d trước khi chuyển lên hạ tầng AWS thật, giúp tiết kiệm tối đa chi phí và giảm rủi ro vận hành.

Pipeline tích hợp 6 cổng kiểm tra bảo mật (Security Gates) tự động bao gồm: Secrets Scan (Gitleaks), SCA (Trivy Filesystem), SAST (SonarQube), IaC Scan (Checkov), Container Image Scan (Trivy Image) và DAST (OWASP ZAP), đảm bảo mỗi bản build đều được kiểm định và lưu báo cáo tập trung lên Amazon S3 trước khi đưa vào vận hành.

### 2. Tuyên bố vấn đề

*Vấn đề hiện tại*

Trong thực tiễn phát triển phần mềm, nhiều đội nhóm vẫn triển khai thủ công, không có quy trình kiểm tra bảo mật tự động, và thiếu khả năng quan sát (observability) sau khi ứng dụng chạy trên môi trường production. Cụ thể:
- **Triển khai thủ công:** Copy file, SSH vào server để deploy, dẫn đến sai sót con người và thiếu tính nhất quán giữa các môi trường.
- **Thiếu kiểm tra bảo mật:** Lỗ hổng bảo mật trong mã nguồn, dependency và container image chỉ được phát hiện sau khi ứng dụng đã chạy thật, gây rủi ro lớn.
- **Không có lưu trữ báo cáo tập trung:** Báo cáo quét bảo mật bị phân tán rải rác trên Jenkins agent, khó tổng hợp và theo dõi theo thời gian.
- **Chi phí hạ tầng đám mây cao:** Việc duy trì EKS Control Plane tĩnh ($72/tháng) không phù hợp với ngân sách dự án nhỏ và các đội nhóm tối ưu chi phí.
- **Trôi lệch cấu hình (Configuration Drift):** Trạng thái thực tế trên môi trường triển khai lệch khỏi cấu hình khai báo trong Git, gây ra lỗi khó tái hiện.

*Giải pháp*

Dự án xây dựng một pipeline CI/CD DevSecOps end-to-end trên AWS với các thành phần chính:
- **Jenkins** điều phối pipeline 12 bước (stage), tự động hóa từ build, 6 security scans, đóng gói Docker image, push lên Amazon ECR và đẩy báo cáo lên Amazon S3.
- **6 Security Gates** tích hợp trực tiếp vào pipeline: Gitleaks (secrets), Trivy (SCA + container scan), SonarQube (SAST), Checkov (IaC scan), OWASP ZAP (DAST).
- **Amazon S3 & AWS Lambda Aggregator:** Lưu trữ toàn bộ kết quả scan định dạng JSON/HTML trên S3 (`reports/secrets/`, `reports/sca/`, `reports/sast/`, `reports/container/`, `reports/dast/`); AWS Lambda tự động tổng hợp findings và quản lý ngưỡng rủi ro.
- **Amazon ECS Fargate:** Đóng vai trò Serverless Container Runtime cho staging và production. Không tốn phí cluster base, dễ dàng scale về 0 (`desired-count 0`) sau khi kết thúc demo để tối ưu ngân sách.
- **Argo CD & Kustomize:** Quản lý cấu hình K8s local (k3d) và luồng đồng bộ GitOps tự động cho staging, yêu cầu phê duyệt thủ công (manual approval gate) cho production.
- **Amazon CloudWatch Container Insights & Prometheus/Grafana:** Cung cấp khả năng giám sát tập trung cho logs, metrics, container health và cảnh báo chi phí.

*Lợi ích và hoàn vốn đầu tư (ROI)*

- Giảm thời gian triển khai từ hàng giờ (thủ công) xuống còn vài phút (tự động).
- Phát hiện sớm lỗ hổng bảo mật ngay trong quá trình CI (Shift-Left Security), tự động lưu vết báo cáo trên S3.
- Tối ưu hóa 85% chi phí hạ tầng AWS bằng việc chuyển đổi từ EKS sang ECS Fargate Serverless.
- Cung cấp khả năng quan sát toàn diện để phát hiện và xử lý sự cố nhanh chóng.
- Tạo nền tảng DevSecOps tiêu chuẩn có thể tái sử dụng cho các dự án phần mềm trong tổ chức.

### 3. Kiến trúc giải pháp

Hệ thống được thiết kế theo mô hình kiến trúc microservices, áp dụng chiến lược Local-first (k3d) và Cloud-after (Amazon ECS Fargate). Luồng hoạt động end-to-end như sau:

```text
React App → Jenkins Security Gates → Amazon ECR → ECS Fargate (Staging) → Scan Reports → Amazon S3 → AWS Lambda (Aggregator) → ECS Fargate (Production) → CloudWatch
```

```text
Developer push code lên GitHub
    ↓
GitHub webhook → Jenkins pipeline
    ↓
Jenkins Jenkinsfile 12 stages:
  ① Secrets scan        (Gitleaks → S3)
  ② Build + unit tests  (npm build)
  ③ SCA                 (Trivy filesystem → S3)
  ④ SAST                (SonarQube → S3)
  ⑤ Docker build        (multi-stage Dockerfile)
  ⑥ Container scan      (Trivy image → S3)
  ⑦ IaC scan            (Checkov → S3)
  ⑧ Push image → Amazon ECR
  ⑨ Deploy Staging     (ECS Fargate / Argo CD auto-sync)
  ⑩ DAST                (OWASP ZAP vs staging URL → S3)
  ⑪ Lambda Aggregator   (Tổng hợp findings từ S3)
  ⑫ Manual approval gate → Promote Production (ECS Fargate)
    ↓
CloudWatch Container Insights & Logs (logs, metrics, alarms)
```

![Kiến trúc DevSecOps CI/CD Pipeline trên AWS](/images/2-Proposal/devsecops_pipeline_architecture.png)

*Dịch vụ AWS sử dụng*

| Dịch vụ AWS | Vai trò trong hệ thống |
|---|---|
| **Amazon ECS Fargate** | Serverless Container Engine triển khai ứng dụng web React cho Staging và Production, không phí quản lý cluster, tự động scale. |
| **Amazon ECR** | Lưu trữ Docker image bảo mật với tính năng Scan on Push, quản lý image tag theo Git Commit SHA. |
| **Amazon S3** | Lưu trữ tập trung báo cáo quét bảo mật (`reports/secrets/`, `reports/sca/`, `reports/sast/`, `reports/container/`, `reports/dast/`) với SSE-S3 encryption và lifecycle 30 ngày. |
| **AWS Lambda** | Function tự động kích hoạt khi có report mới trên S3, tổng hợp dữ liệu lỗ hổng (aggregator) và phân loại mức độ rủi ro. |
| **Amazon CloudWatch** | Giám sát tập trung: Container Insights thu thập metrics ECS Task, Logs Insights truy vấn log ứng dụng, Budget Alerts cảnh báo chi phí. |
| **AWS IAM** | Phân quyền hạt mịn theo nguyên tắc Least Privilege cho Jenkins agent, ECS Task Execution Role, và Lambda execution role. |
| **Application Load Balancer (ALB)** | Định tuyến traffic Layer 7 cho các môi trường Staging và Production trên ECS Fargate. |

*Công cụ DevOps bổ trợ*

| Công cụ | Vai trò |
|---|---|
| **Jenkins** | CI engine điều phối pipeline 12 bước; Configuration as Code (CasC). |
| **Argo CD** | GitOps CD engine trên local k3d cluster, pull-based deployment, auto-sync staging. |
| **Docker** | Đóng gói ứng dụng React thành container image multi-stage, non-root runtime (Nginx Unprivileged). |
| **Kustomize** | Quản lý cấu hình base/overlays cho staging và production. |
| **k3d** | Kubernetes cluster local để phát triển và kiểm thử offline trước khi deploy cloud. |

*Công cụ bảo mật (Security Gates)*

| Gate | Công cụ | Mục đích | Nơi lưu report |
|---|---|---|---|
| Secrets Scan | Gitleaks | Phát hiện khóa bí mật, token, password bị commit vào source code. | Amazon S3 |
| SCA | Trivy Filesystem | Quét lỗ hổng CVE trong dependency (npm packages). | Amazon S3 |
| SAST | SonarQube | Phân tích tĩnh mã nguồn phát hiện code injection, bugs, code smells. | Amazon S3 |
| IaC Scan | Checkov | Kiểm tra cấu hình Kubernetes manifests, Dockerfile và Terraform theo chuẩn bảo mật. | Amazon S3 |
| Container Scan | Trivy Image | Quét lỗ hổng CVE trong base image và các layer của Docker image. | Amazon S3 |
| DAST | OWASP ZAP | Quét bảo mật ứng dụng web đang chạy trên staging (security headers, XSS, injection). | Amazon S3 |

*Thiết kế thành phần*

- **Ứng dụng (Application Layer):** Ứng dụng web Tetris viết bằng React, đóng gói bằng Dockerfile multi-stage (Node 16 build → Nginx Unprivileged runtime), expose port 8080 với health check endpoint `/`.
- **CI Pipeline (Jenkins):** Jenkinsfile khai báo 12 stages tuần tự, biến môi trường chuẩn hóa (`AWS_REGION`, `REGISTRY`, `IMAGE_TAG`, `SCAN_REPORT_DIR`), tự động đẩy báo cáo lên S3.
- **Security & Aggregation:** S3 Bucket lưu trữ dữ liệu scan tập trung; Lambda Aggregator tự động phân tích mức độ nghiêm trọng và quyết định duyệt hay chặn pipeline.
- **CD & Runtime (ECS Fargate / Argo CD):** Staging tự động cập nhật image tag mới; Production yêu cầu Manual Approval Gate trước khi kích hoạt ECS Task mới.
- **Giám sát (Observability):** CloudWatch Container Insights thu thập ECS metrics/logs, Prometheus/Grafana giám sát local, AWS Budget cảnh báo hạn mức chi phí.

### 4. Triển khai kỹ thuật

*Các giai đoạn triển khai*

Dự án được chia thành 9 tuần thực tập (15/06/2026 – 14/08/2026), triển khai theo 3 giai đoạn chính:

1. **Giai đoạn 1 — Nền tảng và phân tích (Tuần 1–3, 15/06 – 05/07):** Tìm hiểu kiến thức AWS cơ bản (VPC, IAM, EC2, ECS Fargate), phân tích hệ thống hiện tại, tạo outline báo cáo và xác định phạm vi dự án. Thử nghiệm build ứng dụng React, Dockerfile và Kubernetes/ECS manifests trên local.

2. **Giai đoạn 2 — Xây dựng pipeline, Security Gates & Triển khai (Tuần 4–6, 06/07 – 26/07):** Chuẩn hóa Jenkins pipeline, tích hợp 6 security scan scripts, tạo ECR repository, S3 report bucket và Lambda aggregator, push image với tag theo commit SHA, cài đặt ECS Fargate cluster + ALB, deploy ứng dụng qua Argo CD / ECS Services, hoàn thiện luồng GitOps staging/production.

3. **Giai đoạn 3 — Monitoring, tài liệu, nghiệm thu & nộp bài (Tuần 7–9, 27/07 – 14/08):** Bật CloudWatch Container Insights, tổng hợp security findings, thiết kế slide báo cáo, dựng workshop website Hugo song ngữ, viết hướng dẫn thực hành Lab step-by-step, viết 3 blog posts, thu thập events/self-evaluation/feedback, rà soát chất lượng song ngữ, đối chiếu tiêu chí chấm điểm, nộp báo cáo cuối khóa và thực hiện scale ECS services về 0 / dọn dẹp tài nguyên AWS.

*Yêu cầu kỹ thuật*

- **Hạ tầng AWS:** Tài khoản AWS với quyền tạo ECS Fargate, ECR, S3, Lambda, IAM, CloudWatch; region `ap-southeast-1`; AWS Budget cảnh báo chi phí.
- **CI/CD:** Jenkins (Docker Compose local hoặc EC2), Docker, Argo CD (local k3d).
- **Bảo mật:** Gitleaks, Trivy, SonarQube, Checkov, OWASP ZAP, AWS Lambda Aggregator.
- **Container & Orchestration:** Docker, Kustomize, k3d (local), Amazon ECS Fargate.
- **Tài liệu:** Hugo (workshop website), Markdown, Git (quản lý source code và nội dung).

*Phân công nhóm*

| Thành viên | Vai trò chính | Trọng tâm | Email |
|---|---|---|---|
| Vũ Hải An | AWS Infrastructure & Platform | AWS account, IAM, ECR, ECS Fargate, S3 (reports), networking. | 23520035@gm.uit.edu.vn |
| Huỳnh Nhật Linh | CI/CD & GitOps | Jenkinsfile, pipeline, push image ECR, deploy ECS Fargate, Argo CD, promotion. | linh.huynhnhat@hcmut.edu.vn |
| Bùi Hữu Lợi | DevSecOps Security | Secrets scan, SCA, SAST, IaC scan, container scan, DAST, S3 reports, Lambda aggregator. | loi.bui2311972@hcmut.edu.vn |
| Nguyễn Văn Hào | Application, Docker, K8s & ECS Task | React app, Dockerfile, health check, K8s base/overlays, ECS Task Definition. | 23520448@gm.uit.edu.vn |
| Nguyễn Chu Hải Nam | Observability, QA, Documentation & Demo | CloudWatch, Prometheus/Grafana, kiểm thử, workshop website, slide demo. | nam.nguyennamcris7@hcmut.edu.vn |

### 5. Lộ trình & Mốc triển khai

| Giai đoạn | Thời gian | Công việc chính | Kết quả bàn giao |
|---|---|---|---|
| Nền tảng & Phân tích | Tuần 1–3 (15/06 – 05/07) | Học AWS, phân tích hệ thống, build app/Docker local, implement secrets scan, tạo outline báo cáo. | App build được, Dockerfile chạy local, pipeline draft, dàn ý báo cáo. |
| Pipeline, Security & Triển khai | Tuần 4–6 (06/07 – 26/07) | Tạo ECR & S3 bucket, push image, cấu hình ECS Fargate & ALB, cài Lambda aggregator, hoàn thiện 6 security gates, deploy staging/production qua ECS/Argo CD. | Pipeline end-to-end, ECR có image, S3 lưu reports, ECS Fargate running, Argo CD Synced & Healthy. |
| Monitoring, Tài liệu & Nghiệm thu | Tuần 7–9 (27/07 – 14/08) | Bật CloudWatch, tổng hợp findings, thiết kế slide, dựng Hugo website, viết workshop lab, viết 3 blogs, thu thập events/evaluation, rà soát song ngữ, nộp sản phẩm cuối, scale ECS về 0 & cleanup AWS. | Dashboard CloudWatch, slide báo cáo, workshop website hoàn chỉnh, 3 blogs đăng, sản phẩm nộp, tài nguyên AWS đã dọn dẹp/scale 0. |

### 6. Ước tính ngân sách

*Chi phí hạ tầng AWS (ước tính hàng tháng)*

| Dịch vụ | Chi phí ước tính | Ghi chú |
|---|---|---|
| Amazon ECS Fargate | ~$5,00 USD/tháng | Trả theo thời gian task chạy demo; scale về `desired-count 0` khi idle. |
| Amazon ECR | ~$1,00 USD/tháng | Lưu trữ < 5 GB image. |
| Amazon S3 (Security Reports) | ~$0,50 USD/tháng | Lưu trữ JSON/HTML reports với lifecycle 30 ngày. |
| AWS Lambda (Aggregator) | ~$0,00 USD/tháng | Nằm trong AWS Free Tier (1M request/tháng). |
| Amazon CloudWatch | ~$3,00 USD/tháng | Container Insights + Logs Insights. |
| Elastic Load Balancer (ALB) | ~$10,00 USD/tháng | 1 ALB dùng chung Staging & Production. |
| Data Transfer | ~$1,00 USD/tháng | Outbound traffic. |
| **Tổng ước tính** | **~$20,50 USD/tháng** | **Tiết kiệm > 85% so với EKS (~$131/tháng)** |

> **Lưu ý:** Bằng cách chuyển đổi sang **Amazon ECS Fargate Serverless**, dự án không chịu khoản phí cố định $73/tháng cho EKS Control Plane. Tổng chi phí AWS thực tế cho toàn bộ đợt thực tập dự kiến chỉ từ **$20 – $35 USD**.

*Chiến lược tối ưu chi phí*

- Sử dụng ECS Fargate Serverless, tự động scale service count về 0 ngay sau khi kết thúc các buổi test/demo.
- Phát triển và kiểm thử offline trên k3d local (miễn phí).
- Cấu hình S3 Lifecycle Policy tự động dọn dẹp reports sau 30 ngày.
- Đặt cảnh báo AWS Budget alerts ở các mức 50%, 80%, 100%.

### 7. Đánh giá rủi ro

*Ma trận rủi ro*

| Rủi ro | Mức ảnh hưởng | Xác suất xảy ra | Chiến lược giảm thiểu |
|---|---|---|---|
| Vượt ngân sách AWS | Thấp | Thấp | Dùng ECS Fargate scale về 0; AWS Budget alerts; S3 lifecycle policy. |
| ECS Fargate task lỗi khi demo | Cao | Thấp | Chuẩn bị bộ slide demo tĩnh (ảnh chụp backup); test kỹ ALB health check trước ngày bảo vệ. |
| Security scan block pipeline | Trung bình | Trung bình | Giai đoạn đầu dùng `--soft-fail`; bật `--exit-code 1` khi demo để thể hiện security gate hoạt động thật. |
| Xung đột merge code giữa các thành viên | Trung bình | Trung bình | Mỗi người làm việc trên branch riêng (`aws-infra`, `cicd-gitops`, `security`, `app-k8s`, `observability-docs`); Pull Request bắt buộc có review. |
| Mất kết nối mạng khi demo live | Trung bình | Thấp | Demo offline trên k3d local; ảnh chụp backup cho từng bước pipeline. |

*Kế hoạch dự phòng*

- Nếu ECS Fargate gặp sự cố: Chuyển sang demo trên local k3d cluster với local Docker registry.
- Nếu Jenkins gặp lỗi: Trình bày qua slide demo tĩnh với ảnh chụp log pipeline thành công.
- Ngay sau khi demo xong: Chạy lệnh `aws ecs update-service --desired-count 0` để tạm dừng container và ngắt chi phí.

### 8. Kết quả kỳ vọng

*Cải tiến kỹ thuật*
- Xây dựng thành công pipeline CI/CD DevSecOps end-to-end trong 9 tuần trên Amazon ECS Fargate & S3/Lambda.
- Tích hợp 6 cổng kiểm tra bảo mật tự động, lưu báo cáo tập trung lên S3 và tổng hợp thông qua Lambda Aggregator.
- Triển khai GitOps staging tự động và kiểm soát production qua manual approval.
- Giám sát toàn diện qua CloudWatch Container Insights và Prometheus/Grafana với chi phí tối ưu (~$20/tháng).

*Sản phẩm bàn giao*
- Workshop website song ngữ (Việt – Anh) theo chuẩn FCAJ template, có hướng dẫn thực hành Lab step-by-step.
- Source code dự án trên GitHub với cấu trúc branch chuẩn và tài liệu `tasks.md` chi tiết.
- 3 bài blog đăng trên AWS Study Group.
- Slide báo cáo đồ án và bộ ảnh minh chứng hoạt động.
- Bảng tổng hợp security findings và đề xuất remediation.
- Kịch bản dọn dẹp & scale to 0 tài nguyên AWS.

*Giá trị dài hạn*
- Mô hình kiến trúc Serverless DevSecOps trên ECS Fargate có tính khả thi cao và tối ưu chi phí cho các doanh nghiệp vừa và nhỏ.
- Workshop website trở thành tài liệu tham khảo chất lượng cho các khóa thực tập tiếp theo.
- Trải nghiệm thực tế phong phú về DevSecOps, ECS Fargate, S3/Lambda, GitOps và CloudWatch cho toàn bộ thành viên nhóm.