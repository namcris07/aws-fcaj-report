---
title: "Bản đề xuất"
date: 2026-07-06
weight: 2
chapter: false
pre: " <b> 2. </b> "
---


# DevSecOps Factory trên AWS
## Xây dựng hệ thống CI/CD DevSecOps end-to-end cho ứng dụng Web React trên Amazon EKS

### 1. Tóm tắt điều hành
DevSecOps Factory là một hệ thống CI/CD hoàn chỉnh tích hợp bảo mật vào mọi giai đoạn của vòng đời phát triển phần mềm (Shift-Left Security). Dự án xây dựng pipeline tự động hóa từ lúc developer push code cho đến khi ứng dụng được triển khai lên môi trường staging và production trên Amazon EKS, sử dụng Jenkins làm CI engine, Argo CD làm GitOps CD engine, và Amazon CloudWatch làm nền tảng giám sát. Hệ thống áp dụng chiến lược **Local-first, Cloud-after** — phát triển và kiểm thử toàn bộ trên máy cá nhân với k3d trước khi chuyển lên hạ tầng AWS thật, giúp tiết kiệm chi phí và giảm rủi ro trong quá trình phát triển.

Pipeline tích hợp 6 cổng kiểm tra bảo mật (Security Gates) tự động bao gồm: Secrets Scan, SCA (Software Composition Analysis), SAST (Static Application Security Testing), IaC Scan, Container Image Scan và DAST (Dynamic Application Security Testing), đảm bảo mỗi bản build đều được kiểm định trước khi đưa vào vận hành.

### 2. Tuyên bố vấn đề

*Vấn đề hiện tại*

Trong thực tiễn phát triển phần mềm, nhiều đội nhóm vẫn triển khai thủ công, không có quy trình kiểm tra bảo mật tự động, và thiếu khả năng quan sát (observability) sau khi ứng dụng chạy trên môi trường production. Cụ thể:
- **Triển khai thủ công:** Copy file, SSH vào server để deploy, dẫn đến sai sót con người và thiếu tính nhất quán giữa các môi trường.
- **Thiếu kiểm tra bảo mật:** Lỗ hổng bảo mật trong mã nguồn, dependency và container image chỉ được phát hiện sau khi ứng dụng đã chạy thật, gây rủi ro lớn.
- **Không có khả năng quan sát:** Khi ứng dụng gặp lỗi trên production, đội ngũ không có dashboard, log tập trung hoặc cảnh báo để phát hiện và xử lý kịp thời.
- **Trôi lệch cấu hình (Configuration Drift):** Trạng thái thực tế trên cluster Kubernetes dần lệch khỏi cấu hình khai báo trong Git, gây ra lỗi khó tái hiện.

*Giải pháp*

Dự án xây dựng một pipeline CI/CD DevSecOps end-to-end trên AWS với các thành phần chính:
- **Jenkins** điều phối pipeline 12 bước (stage), tự động hóa từ build, scan bảo mật, đóng gói Docker image đến push lên Amazon ECR.
- **6 Security Gates** tích hợp trực tiếp vào pipeline: Gitleaks (secrets), Trivy (SCA + container scan), SonarQube (SAST), Checkov (IaC scan), OWASP ZAP (DAST).
- **Argo CD** áp dụng mô hình GitOps kéo (pull-based deployment) để tự động đồng bộ manifest từ Git repository lên Amazon EKS, đảm bảo trạng thái cluster luôn khớp với cấu hình khai báo.
- **Amazon CloudWatch Container Insights** cung cấp khả năng giám sát tập trung cho logs, metrics và cảnh báo hiệu năng.

*Lợi ích và hoàn vốn đầu tư (ROI)*

- Giảm thời gian triển khai từ hàng giờ (thủ công) xuống còn vài phút (tự động).
- Phát hiện sớm lỗ hổng bảo mật ngay trong quá trình CI, trước khi code được đưa vào production.
- Ngăn chặn trôi lệch cấu hình nhờ GitOps, mọi thay đổi hạ tầng đều được kiểm soát qua Git.
- Cung cấp khả năng quan sát toàn diện để phát hiện và xử lý sự cố nhanh chóng.
- Tạo nền tảng tái sử dụng cho các dự án phần mềm tương lai trong tổ chức.

### 3. Kiến trúc giải pháp

Hệ thống được thiết kế theo mô hình kiến trúc microservices trên Kubernetes, sử dụng Kustomize để quản lý cấu hình cho nhiều môi trường (staging/production). Luồng hoạt động end-to-end như sau:

```text
React App → Docker Image → Jenkins Security Gates → Amazon ECR → Argo CD → Amazon EKS → CloudWatch
```

```text
Developer push code lên GitHub
    ↓
GitHub webhook → Jenkins pipeline
    ↓
Jenkins Jenkinsfile 12 stages:
  ① Secrets scan        (Gitleaks)
  ② Build + unit tests  (npm build)
  ③ SCA                 (Trivy filesystem)
  ④ SAST                (SonarQube)
  ⑤ Docker build        (multi-stage Dockerfile)
  ⑥ Container scan      (Trivy image)
  ⑦ IaC scan            (Checkov)
  ⑧ Push image → Amazon ECR
  ⑨ Update image tag → Argo CD auto-sync staging
  ⑩ DAST                (OWASP ZAP vs staging URL)
  ⑪ Manual approval gate
  ⑫ Promote → production
    ↓
CloudWatch Container Insights (logs, metrics, alarms)
```

![Kiến trúc DevSecOps CI/CD Pipeline trên AWS](/images/2-Proposal/devsecops_pipeline_architecture.png)

*Dịch vụ AWS sử dụng*

| Dịch vụ AWS | Vai trò trong hệ thống |
|---|---|
| **Amazon EKS** | Điều phối container Kubernetes cho ứng dụng web, quản lý Node Group với instance type `t3.small`, hỗ trợ multi-namespace (staging/production). |
| **Amazon ECR** | Lưu trữ Docker image bảo mật với tính năng Scan on Push, quản lý image tag theo Git Commit SHA. |
| **Amazon CloudWatch** | Giám sát tập trung: Container Insights thu thập metrics Node/Pod, Logs Insights truy vấn log ứng dụng, Budget Alerts cảnh báo chi phí. |
| **AWS IAM** | Phân quyền hạt mịn theo nguyên tắc Least Privilege; IRSA (IAM Roles for Service Accounts) cho các workload trên EKS. |
| **AWS Load Balancer Controller** | Tự động tạo Application Load Balancer (ALB) từ cấu hình Ingress trên Kubernetes, định tuyến Layer 7. |

*Công cụ DevOps bổ trợ*

| Công cụ | Vai trò |
|---|---|
| **Jenkins** | CI engine điều phối pipeline 12 bước; Configuration as Code (CasC). |
| **Argo CD** | GitOps CD engine, pull-based deployment, auto-sync staging, manual approval production. |
| **Docker** | Đóng gói ứng dụng React thành container image multi-stage, non-root runtime (Nginx Unprivileged). |
| **Kustomize** | Quản lý cấu hình Kubernetes base/overlays cho staging và production. |
| **k3d** | Kubernetes cluster local để phát triển và kiểm thử offline trước khi lên cloud. |

*Công cụ bảo mật (Security Gates)*

| Gate | Công cụ | Mục đích |
|---|---|---|
| Secrets Scan | Gitleaks | Phát hiện khóa bí mật, token, password bị commit vào source code. |
| SCA | Trivy Filesystem | Quét lỗ hổng CVE trong dependency (npm packages). |
| SAST | SonarQube | Phân tích tĩnh mã nguồn phát hiện code injection, bugs, code smells. |
| IaC Scan | Checkov | Kiểm tra cấu hình Kubernetes manifests và Dockerfile theo chuẩn bảo mật. |
| Container Scan | Trivy Image | Quét lỗ hổng CVE trong base image và các layer của Docker image. |
| DAST | OWASP ZAP | Quét bảo mật ứng dụng web đang chạy trên staging (security headers, XSS, injection). |

*Thiết kế thành phần*

- **Ứng dụng (Application Layer):** Ứng dụng web Tetris viết bằng React, đóng gói bằng Dockerfile multi-stage (Node 16 build → Nginx Unprivileged runtime), expose port 8080 với health check endpoint `/`.
- **CI Pipeline (Jenkins):** Jenkinsfile khai báo 12 stages tuần tự, biến môi trường chuẩn hóa (`AWS_REGION`, `REGISTRY`, `IMAGE_TAG`), archive security reports dưới dạng artifacts.
- **CD GitOps (Argo CD):** Application CRD cho staging (auto-sync) và production (manual sync), theo dõi thay đổi image tag trong file `kustomization.yaml`.
- **Hạ tầng Kubernetes (EKS):** Deployment với Liveness/Readiness Probes, SecurityContext (runAsNonRoot, drop ALL capabilities, readOnlyRootFilesystem), Service và Ingress ALB.
- **Giám sát (Observability):** CloudWatch Container Insights với DaemonSet agent, Fluent Bit thu thập logs, CloudWatch Logs Insights truy vấn, AWS Budget cảnh báo chi phí.

### 4. Triển khai kỹ thuật

*Các giai đoạn triển khai*

Dự án được chia thành 9 tuần thực tập (15/06/2026 – 14/08/2026), triển khai theo 3 giai đoạn chính:

1. **Giai đoạn 1 — Nền tảng và phân tích (Tuần 1–3, 15/06 – 05/07):** Tìm hiểu kiến thức AWS cơ bản (VPC, IAM, EC2, EKS), phân tích hệ thống hiện tại, tạo outline báo cáo và xác định phạm vi dự án. Thử nghiệm build ứng dụng React, Dockerfile và Kubernetes manifests trên local.

2. **Giai đoạn 2 — Xây dựng pipeline, Security Gates & Triển khai (Tuần 4–6, 06/07 – 26/07):** Chuẩn hóa Jenkins pipeline, tích hợp 6 security scan scripts, tạo ECR repository, push image với tag theo commit SHA, cài đặt EKS cluster, cài AWS Load Balancer Controller, deploy ứng dụng qua Argo CD, hoàn thiện luồng GitOps staging/production.

3. **Giai đoạn 3 — Monitoring, tài liệu, nghiệm thu & nộp bài (Tuần 7–9, 27/07 – 14/08):** Bật CloudWatch Container Insights, tổng hợp security findings, thiết kế slide báo cáo, dựng workshop website Hugo song ngữ, viết hướng dẫn thực hành Lab step-by-step, viết 3 blog posts, thu thập events/self-evaluation/feedback, rà soát chất lượng song ngữ, đối chiếu tiêu chí chấm điểm, nộp báo cáo cuối khóa và dọn dẹp tài nguyên AWS.

*Yêu cầu kỹ thuật*

- **Hạ tầng AWS:** Tài khoản AWS với quyền tạo EKS, ECR, IAM, CloudWatch; region `ap-southeast-1`; AWS Budget cảnh báo chi phí.
- **CI/CD:** Jenkins (Docker Compose local hoặc EC2), Docker, Argo CD cài trên EKS qua Helm.
- **Bảo mật:** Gitleaks, Trivy, SonarQube (Docker Compose), Checkov, OWASP ZAP.
- **Kubernetes:** kubectl, Kustomize, k3d (local), Helm (AWS Load Balancer Controller, Argo CD).
- **Tài liệu:** Hugo (workshop website), Markdown, Git (quản lý source code và nội dung).

*Phân công nhóm*

| Thành viên | Vai trò chính | Trọng tâm | Email |
|---|---|---|---|
| Vũ Hải An | AWS Infrastructure & Kubernetes Platform | AWS account, IAM, EKS, networking, ECR, nền tảng triển khai. | 23520035@gm.uit.edu.vn |
| Huỳnh Nhật Linh | CI/CD & GitOps | Jenkinsfile, pipeline, push image, Argo CD, promotion staging/production. | linh.huynhnhat@hcmut.edu.vn |
| Bùi Hữu Lợi | DevSecOps Security | Secrets scan, SCA, SAST, IaC scan, container scan, DAST, policy bảo mật. | loi.bui2311972@hcmut.edu.vn |
| Nguyễn Văn Hào | Application, Docker & K8s Manifests | React app, Dockerfile, health check, Kubernetes base/overlays. | 23520448@gm.uit.edu.vn |
| Nguyễn Chu Hải Nam | Observability, QA, Documentation & Demo | CloudWatch, kiểm thử, workshop website. | nam.nguyennamcris7@hcmut.edu.vn |

### 5. Lộ trình & Mốc triển khai

| Giai đoạn | Thời gian | Công việc chính | Kết quả bàn giao |
|---|---|---|---|
| Nền tảng & Phân tích | Tuần 1–3 (15/06 – 05/07) | Học AWS, phân tích hệ thống, build app/Docker local, implement secrets scan, tạo outline báo cáo. | App build được, Dockerfile chạy local, pipeline draft, dàn ý báo cáo. |
| Pipeline, Security & Triển khai | Tuần 4–6 (06/07 – 26/07) | Tạo ECR, push image, tạo EKS cluster, cài ALB Controller, hoàn thiện 6 security gates, deploy staging/production qua Argo CD. | Pipeline end-to-end, ECR có image, EKS nodes Ready, Argo CD Synced & Healthy. |
| Monitoring, Tài liệu & Nghiệm thu | Tuần 7–9 (27/07 – 14/08) | Bật CloudWatch, tổng hợp findings, thiết kế slide, dựng Hugo website, viết workshop lab, viết 3 blogs, thu thập events/evaluation, rà soát song ngữ, nộp sản phẩm cuối, cleanup AWS. | Dashboard CloudWatch, slide báo cáo, workshop website hoàn chỉnh, 3 blogs đăng, sản phẩm nộp, AWS đã dọn dẹp. |

### 6. Ước tính ngân sách

*Chi phí hạ tầng AWS (ước tính hàng tháng)*

| Dịch vụ | Chi phí ước tính | Ghi chú |
|---|---|---|
| Amazon EKS (Control Plane) | ~73,00 USD/tháng | 1 cluster. |
| EC2 Managed Node Group (t3.small × 2) | ~30,00 USD/tháng | 2 nodes, On-Demand pricing. |
| Amazon ECR | ~1,00 USD/tháng | Lưu trữ < 5 GB image. |
| Amazon CloudWatch | ~5,00 USD/tháng | Container Insights + Logs Insights. |
| Elastic Load Balancer (ALB) | ~20,00 USD/tháng | 1 ALB cho staging + production. |
| Data Transfer | ~2,00 USD/tháng | Outbound traffic. |
| **Tổng ước tính** | **~131 USD/tháng** | |

> **Lưu ý:** Chi phí chính đến từ EKS Control Plane và EC2 nodes. Để giảm chi phí trong quá trình phát triển, nhóm áp dụng chiến lược **Local-first** với k3d — chỉ tạo EKS cluster trên AWS khi cần demo thực tế, và cleanup ngay sau khi hoàn thành. Dự kiến tổng chi phí AWS thực tế cho cả dự án khoảng **150–250 USD** (do chỉ chạy cloud 1–2 tuần).

*Chiến lược tối ưu chi phí*

- Phát triển và kiểm thử trên k3d local (miễn phí), chỉ chuyển sang EKS khi cần.
- Sử dụng AWS Budget alerts ở mức 50%, 80%, 100% để cảnh báo sớm.
- Dọn dẹp triệt để tài nguyên (EKS, ALB, ECR, CloudWatch) ngay sau khi kết thúc demo.
- Sử dụng instance type tiết kiệm (`t3.small`) và giữ số lượng nodes tối thiểu.

### 7. Đánh giá rủi ro

*Ma trận rủi ro*

| Rủi ro | Mức ảnh hưởng | Xác suất xảy ra | Chiến lược giảm thiểu |
|---|---|---|---|
| Vượt ngân sách AWS | Cao | Trung bình | AWS Budget alerts; chiến lược Local-first; cleanup ngay sau demo; dùng t3.small instance. |
| EKS cluster lỗi khi demo | Cao | Thấp | Chuẩn bị bộ slide demo tĩnh (ảnh chụp backup); test kỹ trước ngày bảo vệ; có local k3d làm phương án dự phòng. |
| Security scan block pipeline | Trung bình | Trung bình | Giai đoạn đầu dùng `--soft-fail`; bật `--exit-code 1` khi demo để thể hiện security gate hoạt động thật. |
| Xung đột merge code giữa các thành viên | Trung bình | Trung bình | Mỗi người làm việc trên branch riêng; Pull Request phải có review; sử dụng CODEOWNERS. |
| Mất kết nối mạng khi demo live | Trung bình | Thấp | Demo offline trên k3d local; ảnh chụp backup cho từng bước pipeline. |
| Dependency cũ gây lỗi build ứng dụng | Thấp | Trung bình | Ghi lại dependency issues trong `VULNERABILITIES.md`; chấp nhận một số warning cho mục đích demo security scan. |

*Kế hoạch dự phòng*

- Nếu EKS không hoạt động: Chuyển sang demo trên k3d local cluster với local Docker registry.
- Nếu Jenkins gặp lỗi: Trình bày qua slide demo tĩnh với ảnh chụp log pipeline thành công.
- Nếu vượt ngân sách: Dừng EKS cluster ngay, quay lại local-only; sử dụng CloudFormation/Terraform để cleanup tự động.

### 8. Kết quả kỳ vọng

*Cải tiến kỹ thuật*
- Xây dựng thành công pipeline CI/CD DevSecOps end-to-end trong 9 tuần, chạy hoàn chỉnh từ code push đến deploy production.
- Tích hợp 6 cổng kiểm tra bảo mật tự động, phát hiện và báo cáo lỗ hổng trước khi code lên production.
- Triển khai GitOps với Argo CD, tự động đồng bộ staging và kiểm soát production qua manual approval.
- Giám sát toàn diện qua CloudWatch Container Insights với dashboard, log query và cảnh báo chi phí.

*Sản phẩm bàn giao*
- Workshop website song ngữ (Việt – Anh) theo chuẩn FCAJ template, có hướng dẫn thực hành Lab step-by-step.
- Source code dự án trên GitHub với cấu trúc rõ ràng, tài liệu đầy đủ.
- 3 bài blog đăng trên AWS Study Group.
- Slide báo cáo đồ án và bộ ảnh minh chứng hoạt động.
- Bảng tổng hợp security findings và đề xuất remediation.
- Checklist cleanup AWS để tránh phát sinh chi phí sau dự án.

*Giá trị dài hạn*
- Mô hình pipeline có thể tái sử dụng cho các dự án phần mềm khác trong tổ chức.
- Workshop website trở thành tài liệu tham khảo cho các khóa thực tập tiếp theo.
- Kinh nghiệm thực tế về DevSecOps, Kubernetes, GitOps và AWS services cho toàn bộ thành viên nhóm.