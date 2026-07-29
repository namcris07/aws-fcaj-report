# AWS FCAJ Internship Report 🌟

<div align="center">

![AWS](https://img.shields.io/badge/AWS-Amazon%20ECS%20Fargate-orange?style=for-the-badge&logo=amazon-aws)
![DevSecOps](https://img.shields.io/badge/DevSecOps-Shift--Left-blue?style=for-the-badge&logo=shieldsdotio)
![Hugo](https://img.shields.io/badge/Hugo-v0.120%2B-ff4088?style=for-the-badge&logo=hugo)
![GitHub Pages](https://img.shields.io/badge/Deployment-GitHub%20Pages-222222?style=for-the-badge&logo=githubpages)
![Languages](https://img.shields.io/badge/Languages-Bilingual%20(EN%2FVI)-purple?style=for-the-badge)

</div>

---

Chào mừng bạn đến với repo báo cáo thực tập của nhóm dự án **DevSecOps Factory trên AWS** thuộc chương trình **AWS First Cloud AI Journey (FCAJ)**! 👋

## 📋 Giới thiệu

Website báo cáo thực tập này ghi lại toàn bộ hành trình 9 tuần thực tập (từ **15/06/2026** đến **14/08/2026**) của nhóm mình với đề tài:

> **"Xây dựng hệ thống CI/CD DevSecOps end-to-end cho ứng dụng Web React trên Amazon ECS Fargate"**

Dự án áp dụng mô hình **Shift-Left Security** và chiến lược **Local-first, Cloud-after** — phát triển và kiểm thử offline trên cụm **k3d** local trước khi triển khai lên **Amazon ECS Fargate**. Quy trình tự động hóa tích hợp 6 cổng kiểm thử an ninh (Gitleaks, Trivy SCA, SonarQube, Checkov, Trivy Image, OWASP ZAP), lưu trữ báo cáo tập trung trên **Amazon S3**, tổng hợp dữ liệu lỗ hổng bằng **AWS Lambda Aggregator** và triển khai GitOps với **Argo CD**.

---

## 🚀 Xem báo cáo trực tiếp (Live Demo)

Website báo cáo thực tập được xuất bản tự động qua GitHub Pages:

👉 **[https://namcris07.github.io/aws-fcaj-report/](https://namcris07.github.io/aws-fcaj-report/)**

---

## 🏗️ Kiến trúc & Quy trình DevSecOps (Pipeline Flow)

```mermaid
flowchart TD
    A[👨‍💻 Developer Push Code] -->|Git Webhook| B[⚙️ Jenkins CI Engine - 12 Stages]
    
    subgraph Security_Gates [🛡️ 6 Security Gates & Report Storage]
        B --> C1[1. Gitleaks - Secrets Scan]
        C1 --> C2[2. Trivy - SCA Dependency Scan]
        C2 --> C3[3. SonarQube - SAST Code Quality]
        C3 --> C4[4. Checkov - IaC Security Scan]
        C1 & C2 & C3 & C4 -->|Save Reports| S3[🪣 Amazon S3 Report Storage]
    end

    C4 -->|Build & Package| D[🐳 Multi-stage Docker Image]
    D --> C5[5. Trivy - Container Image Scan]
    C5 -->|Push Image| E[📦 Amazon ECR]

    E -->|Auto-sync / GitOps| F[🐙 Argo CD / Kustomize]
    F -->|Deploy Staging| G[🚀 Amazon ECS Fargate - Staging]

    G --> C6[6. OWASP ZAP - DAST Scan]
    C6 -->|Save DAST Report| S3
    S3 -->|Trigger Aggregator| L[⚡ AWS Lambda Findings Aggregator]

    L -->|Manual Approval Gate| H[🌐 Amazon ECS Fargate - Production]
    H --> I[📊 Amazon CloudWatch Container Insights]
```

---

## 📚 Nội dung báo cáo (8 Phần chính)

Trang báo cáo song ngữ (Anh - Việt) bao gồm 8 phần nội dung chi tiết:

| STT | Danh mục | Mô tả |
| :--- | :--- | :--- |
| 1️⃣ | **[Worklog](https://namcris07.github.io/aws-fcaj-report/1-worklog/)** | Nhật ký công việc hàng tuần của nhóm từ Tuần 1 đến Tuần 9 |
| 2️⃣ | **[Proposal](https://namcris07.github.io/aws-fcaj-report/2-proposal/)** | Ý tưởng, mục tiêu và đề xuất kiến trúc giải pháp DevSecOps Factory trên Amazon ECS Fargate |
| 3️⃣ | **[Blogs Posted](https://namcris07.github.io/aws-fcaj-report/3-blogsposted/)** | Tổng hợp các bài blog kỹ thuật chuyên sâu xuất bản trên AWS Study Group |
| 4️⃣ | **[Event Participated](https://namcris07.github.io/aws-fcaj-report/4-eventparticipated/)** | Nhật ký tham gia FCAJ Community Day, AWS Study Tour & các buổi Tech Meetup |
| 5️⃣ | **[Workshop](https://namcris07.github.io/aws-fcaj-report/5-workshop/)** | Hướng dẫn lab thực hành chi tiết từng bước (Step-by-step Lab Guides) với đầy đủ Code Snippets |
| 6️⃣ | **[Self-evaluation](https://namcris07.github.io/aws-fcaj-report/6-self-evaluation/)** | Tự đánh giá năng lực, kết quả thu hoạch và bài học kinh nghiệm |
| 7️⃣ | **[Feedback](https://namcris07.github.io/aws-fcaj-report/7-feedback/)** | Cảm nhận, chia sẻ và đóng góp ý kiến về chương trình thực tập AWS FCAJ |
| 8️⃣ | **[References](https://namcris07.github.io/aws-fcaj-report/8-references/)** | Tài liệu tham khảo, AWS Whitepapers và các liên kết hữu ích |

---

## 🛠️ Công nghệ & Công cụ sử dụng

| Lĩnh vực | Công nghệ / Công cụ |
| :--- | :--- |
| **AWS Cloud Services** | Amazon ECS Fargate *(Serverless Runtime)*, Amazon ECR, Amazon S3 *(Reports Storage)*, AWS Lambda *(Aggregator)*, Amazon CloudWatch Container Insights, AWS IAM |
| **DevOps & GitOps** | Jenkins *(12-stage CI)*, Argo CD *(GitOps)*, Docker *(Multi-stage)*, Kustomize, k3d *(Local K8s)*, ALB |
| **Security Gates (6 Cổng)** | Gitleaks *(Secrets)*, Trivy *(SCA & Container Image)*, SonarQube *(SAST)*, Checkov *(IaC)*, OWASP ZAP *(DAST)* |
| **Website & Hosting** | Hugo Static Site Generator (`hugo-theme-learn`), GitHub Pages, GitHub Actions |
| **Ngôn ngữ hỗ trợ** | Song ngữ Việt - Anh (Bilingual VN/EN) |

---

## 📁 Cấu trúc Repository

```text
fcj-workshop-template/
├── .github/              # Workflow CI/CD build & deploy GitHub Pages
├── archetypes/           # Hugo template archetypes
├── content/              # Nội dung bài viết Markdown (Song ngữ .md / .vi.md)
│   ├── 1-Worklog/        # Nhật ký công việc hàng tuần
│   ├── 2-Proposal/       # Đề xuất dự án & kiến trúc ECS Fargate
│   ├── 3-BlogsPosted/    # Bài viết kỹ thuật
│   ├── 4-EventParticipated/ # Sự kiện tham gia
│   ├── 5-Workshop/       # Hướng dẫn workshop lab
│   ├── 6-Self-evaluation/ # Tự đánh giá
│   ├── 7-Feedback/       # Góp ý & Phản hồi
│   └── 8-References/     # Tài liệu tham khảo
├── layouts/              # Custom Hugo layouts & shortcodes
├── static/               # Hình ảnh, tài nguyên tĩnh & favicon
├── themes/               # Hugo theme (hugo-theme-learn submodule)
├── config.toml           # Cấu hình website Hugo & Menu
└── README.md             # Tài liệu hướng dẫn Repository
```

---

## 💻 Hướng dẫn chạy cục bộ (Local Development)

### Yêu cầu tiên quyết
- **Hugo** (Phiên bản Extended, v0.120.0 trở lên)
- **Git**

### Các bước thực hiện

1. **Clone repository bao gồm submodules:**
   ```bash
   git clone --recursive https://github.com/namcris07/aws-fcaj-report.git
   cd aws-fcaj-report
   ```
   *Nếu đã clone repo mà chưa pull submodule:*
   ```bash
   git submodule update --init --recursive
   ```

2. **Khởi chạy Hugo Dev Server:**
   ```bash
   hugo server -D
   ```

3. **Truy cập website trên trình duyệt:**
   Mở địa chỉ: `http://localhost:1313/aws-fcaj-report/`

---

## 👥 Thành viên nhóm thực tập

| Thành viên | Vai trò chính | Email liên hệ | GitHub |
| :--- | :--- | :--- | :--- |
| **Vũ Hải An** | AWS Infrastructure & Platform *(ECS Fargate, ECR, S3, IAM)* | `23520035@gm.uit.edu.vn` | - |
| **Huỳnh Nhật Linh** | CI/CD Engine & GitOps Pipeline *(Jenkins, Argo CD)* | `linh.huynhnhat@hcmut.edu.vn` | - |
| **Bùi Hữu Lợi** | DevSecOps Security Gates *(6 Security Gates, Lambda Aggregator)* | `loi.bui2311972@hcmut.edu.vn` | - |
| **Nguyễn Văn Hào** | Application, Docker & ECS Manifests *(React, Dockerfile, Task Def)* | `23520448@gm.uit.edu.vn` | - |
| **Nguyễn Chu Hải Nam** | Observability, QA & Documentation *(CloudWatch, Hugo Site)* | `nam.nguyennamcris7@hcmut.edu.vn` | [@namcris07](https://github.com/namcris07) |

---

## 📄 License & Bản quyền

Copyright © 2026 AWS FCAJ Internship Report - **DevSecOps Factory Team**.

---

<div align="center">

_Được thực hiện với ❤️ và ☕ trong chương trình AWS First Cloud AI Journey (FCAJ) Internship_

**Happy Coding & Safe Deployment! 🚀**

</div>
