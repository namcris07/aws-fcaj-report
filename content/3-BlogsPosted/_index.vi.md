---
title: "Các bài blogs đã dịch"
date: 2026-07-06
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

# Các bài blogs đã dịch trên AWS Study Group

Dưới đây là tổng hợp và giới thiệu các bài viết kỹ thuật chuyên sâu đã được dịch và đăng tải trên cộng đồng **[AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj)**:

---

### 1. [Blog 1 - Xây dựng môi trường Jenkins serverless trên AWS Fargate](3.1-Blog1/)
**Tác giả:** Krithivasan Balasubramaniyan | **Ngày đăng:** 26/03/2021

Bài viết hướng dẫn chi tiết cách tự động hóa và triển khai một môi trường Jenkins Controller-Agent hoàn toàn Serverless trên **Amazon ECS Fargate** sử dụng **Terraform**. Giải pháp sử dụng **Amazon EFS** làm bộ lưu trữ dữ liệu liên tục (persistent storage) cho Jenkins Controller và sử dụng **Fargate Spot Capacity Providers** để khởi chạy các Jenkins Build Agents dưới dạng các container ngắn hạn (ephemeral containers), giúp tối ưu hóa chi phí vận hành và đảm bảo tính sẵn sàng cao cho quy trình CI/CD.

---

### 2. [Blog 2 - Xây dựng nhà máy phần mềm DevSecOps end-to-end dựa trên Kubernetes trên AWS](3.2-Blog2/)
**Tác giả:** Srinivas Manepalli | **Ngày đăng:** 25/06/2021

Bài viết trình bày kiến trúc tham chiếu hoàn chỉnh xây dựng nhà máy phần mềm DevSecOps (Software Factory) trên nền tảng **Amazon EKS** theo triết lý **Shift-Left Security**. Pipeline kết hợp các dịch vụ AWS Cloud-Native (*AWS CodePipeline, CodeBuild, CodeCommit, Security Hub*) với các công cụ kiểm thử bảo mật chuyên dụng (*git-secrets, Snyk/Anchore, ECR Image Scan, OWASP ZAP, Sysdig Falco*) để thực hiện rà quét toàn diện 6 tầng (Secrets, SCA, SAST, DAST, RASP) và tự động tổng hợp dữ liệu lỗ hổng về **AWS Security Hub**.

---

### 3. [Blog 3 - Giám sát và mở rộng quy mô ứng dụng Amazon ECS trên AWS Fargate bằng chỉ số Prometheus](3.3-Blog3/)
**Tác giả:** Mike George | **Ngày đăng:** 05/02/2021

Bài viết hướng dẫn cách nâng cao khả năng quan sát (Observability) cho các ứng dụng Java/Tomcat chạy trên **Amazon ECS Fargate** bằng việc kết hợp **CloudWatch Container Insights** và **Prometheus Exporter**. Bằng cách thu thập các chỉ số ứng dụng chuyên sâu (*JVM memory, connection count, garbage collection*), bài viết hướng dẫn từng bước thiết lập **CloudWatch Alarms** và cấu hình **ECS Service Auto Scaling** để tự động mở rộng quy mô (scale-out / scale-in) linh hoạt theo nhu cầu thực tế của lưu lượng truy cập.