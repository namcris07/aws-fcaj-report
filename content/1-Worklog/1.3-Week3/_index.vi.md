---
title: "Worklog Tuần 3"
date: 2026-06-29
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---



### Mục tiêu tuần 3:

* Nắm vững kiến thức nền tảng về Bảo mật & Quản trị Định danh trên AWS.
* Hoàn thành các bài thực hành thực tế (Hands-on Labs) về quản lý phân quyền IAM, mã hóa dữ liệu với KMS, và giám sát tuân thủ bảo mật với Security Hub.
* Master các khái niệm và dịch vụ Cơ sở dữ liệu (RDS, Aurora, Redshift), Bộ nhớ đệm (ElastiCache), và Di trú dữ liệu (SCT, DMS) trên AWS.
* Hiểu các dịch vụ AWS Compute & Storage cơ bản (EC2, EBS, Elastic IP) và phương thức kết nối, quản lý tài nguyên.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | - **Bảo mật & Quản trị Định danh** <br>&emsp; + **Triết lý & Trách nhiệm:** Tìm hiểu ranh giới bảo mật đám mây (Shared Responsibility Model) giữa khách hàng và AWS; triết lý ưu tiên bảo mật "Job Zero". <br>&emsp; + **Kiểm soát Truy cập:** Nghiên cứu cơ chế IAM (phân quyền Least Privilege, STS Temporary Credentials) và phương thức quản trị đa tài khoản (AWS Organizations SCPs, Identity Center SSO). <br>&emsp; + **Định danh Ứng dụng:** Khảo sát giải pháp xác thực và cấp quyền người dùng thông qua Amazon Cognito (User Pools & Identity Pools). <br>&emsp; + **Bảo vệ Dữ liệu & Tuân thủ:** Tìm hiểu kỹ thuật mã hóa dữ liệu (Envelope Encryption) bằng khóa AWS KMS và giám sát rủi ro hạ tầng qua Security Hub. | 29/06/2026   | 29/06/2026      | [Module 5](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%205/module-5.md) |
| 3   | - **Hands-on Labs: AWS IAM & Security Integration** <br>&emsp; + **Quản trị Định danh (Lab 02):** Khởi tạo cấu trúc IAM User/Group, cấu hình Multi-Factor Authentication (MFA) bảo mật và thiết lập chuyển đổi vai trò (Switch Role) phân quyền động. <br>&emsp; + **Rào cản Điều kiện (Lab 44):** Cấu hình chính sách tin cậy (Trust Policy) giới hạn quyền truy cập theo dải IP nguồn và thời gian biểu hiệu lực. <br>&emsp; + **Ủy quyền Hạ tầng (Lab 48):** Tạo và gắn IAM Role (Instance Profile) cho EC2 instance, loại bỏ rủi ro lộ lọt thông tin xác thực từ việc lưu trữ khóa Access Keys cố định. | 30/06/2026   | 30/06/2026      | [Lab 02](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%205/module-5.md) <br>[Lab 44](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%205/module-5.md)<br>[Lab 48](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%205/module-5.md)|
| 4   | - **Hands-on Lab: AWS Security Compliance, Access Control & Data Encryption** <br>&emsp; + **Giám sát Tuân thủ Bảo mật (Lab 18):** Kích hoạt dịch vụ AWS Security Hub kết hợp cấu hình bộ quy chuẩn tự động để đánh giá và chấm điểm an toàn thông tin hệ thống liên tục. <br>&emsp; + **Thắt chặt Ngưỡng quyền Tối đa (Lab 30):** Triển khai tính năng nâng cao IAM Permission Boundary nhằm thiết lập rào dải kiểm soát, chặn đứng rủi ro leo thang đặc quyền (Privilege Escalation) trên các phân hệ địa lý. <br>&emsp; + **Mã hóa Dữ liệu Trạng thái Nghỉ (Lab 33):** Khởi tạo khóa gốc đối xứng CMK qua AWS KMS, áp dụng mã hóa phong bì cho Amazon S3 và thiết lập CloudTrail – Athena để kiểm toán nhật ký gọi API hệ thống. | 01/07/2026   | 01/07/2026      | [Lab 18](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%205/module-5.md)<br>[Lab 30](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%205/module-5.md) <br> [Lab 33](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%205/module-5.md)|
| 5   | - **Dịch vụ Cơ sở dữ liệu và Lưu trữ bộ nhớ đệm** <br>&emsp; + **Khái niệm cơ bản & Phân loại:** Tìm hiểu cấu trúc Database (Primary/Foreign Key, Normalization, Indexing, Partitioning, Buffer), phân biệt RDBMS vs NoSQL (Fixed vs Dynamic Schema, ACID vs BASE) và mô hình OLTP vs OLAP. <br>&emsp; + **Dịch vụ AWS Database:** Khảo sát kiến trúc và cơ chế hoạt động của Amazon RDS (Multi-AZ synchronous replication & Read Replicas asynchronous replication), Amazon Aurora (Cluster Volume dùng chung, 6 bản sao trên 3 AZs, Backtrack, Cloning) và kho dữ liệu phân tích Amazon Redshift (MPP, Columnar Storage, Spectrum). <br>&emsp; + **Bộ nhớ đệm & Di trú:** Tìm hiểu cơ chế lưu trữ bộ nhớ đệm Amazon ElastiCache (Memcached & Redis), phân định trách nhiệm Caching Logic; nghiên cứu giải pháp di trú cơ sở dữ liệu sử dụng AWS SCT và AWS DMS. | 02/07/2026   | 02/07/2026      | [Module 6](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%206/module-6.md) |
| 6   | - **Thực hành: CI/CD trên EKS với CodePipeline và GitHub** <br>&emsp; + **Tổng quan Kubernetes & EKS:** Tìm hiểu Amazon EKS để tạo, quản lý, chạy và mở rộng quy mô các ứng dụng Kubernetes trong môi trường AWS hoặc on-premise. <br>&emsp; + **Tự động hóa Pipeline:** Nghiên cứu AWS CodePipeline để tự động hóa build, test và deploy mỗi khi có sự thay đổi mã nguồn từ GitHub. <br>&emsp; + **Triển khai:** Tạo một EKS Cluster, deploy ứng dụng web lên cluster và cấu hình pipeline CI/CD để tự động deploy. | 03/07/2026   | 03/07/2026      | <https://000062.awsstudygroup.com/> |


### Kết quả đạt được tuần 3:

* **Bảo mật & Quản trị Định danh (Security & Identity Governance)**:
  * Nắm vững mô hình trách nhiệm chia sẻ (Shared Responsibility Model) và triết lý bảo mật "Job Zero" của AWS.
  * Hiểu sâu về cơ chế IAM (Least Privilege, STS temporary credentials) và quản trị đa tài khoản (Organizations SCPs, Identity Center SSO).
  * Khảo sát các giải pháp xác thực với Amazon Cognito (User Pools & Identity Pools).
* **Thực hành Bảo mật IAM & Tích hợp (Hands-on Labs: IAM & Security)**:
  * **Lab 02**: Cấu hình cấu trúc IAM User/Group, kích hoạt MFA bảo mật và thiết lập Switch Role giữa các môi trường.
  * **Lab 44**: Xây dựng Trust Policy ràng buộc điều kiện truy cập theo dải IP (Source IP) và khung giờ hiệu lực.
  * **Lab 48**: Thiết lập IAM Role (Instance Profile) cho EC2, hạn chế tối đa việc sử dụng Access Key cố định.
* **Thực hành Tuân thủ Bảo mật & Mã hóa dữ liệu (Hands-on Labs: Security Compliance & Encryption)**:
  * **Lab 18**: Kích hoạt AWS Security Hub và cấu hình bộ tiêu chuẩn bảo mật (Security Standards) để tự động đánh giá, chấm điểm tuân thủ hệ thống liên tục.
  * **Lab 30**: Triển khai IAM Permission Boundary để kiểm soát giới hạn quyền tối đa cho các IAM Role/User, triệt tiêu nguy cơ leo thang đặc quyền (Privilege Escalation).
  * **Lab 33**: Khởi tạo khóa KMS CMK đối xứng, áp dụng mã hóa phong bì (Envelope Encryption) cho S3, đồng thời cấu hình CloudTrail tích hợp Athena để truy vấn nhật ký API một cách toàn diện.
* **Cơ sở dữ liệu & Lưu trữ bộ nhớ đệm (Database Services & Caching)**:
  * Nắm vững các khái niệm thiết kế database cơ bản (Primary/Foreign Key, Normalization, Indexing, Partitioning, Buffer).
  * Phân biệt rõ nét giữa hệ quản trị RDBMS (Fixed Schema, ACID) và NoSQL (Dynamic Schema, BASE) cùng mô hình OLTP (xử lý giao dịch trực tuyến) và OLAP (kho dữ liệu phân tích lịch sử).
  * Hiểu sâu kiến trúc dịch vụ Amazon RDS (Multi-AZ đồng bộ, Read Replicas bất đồng bộ) và Amazon Aurora (Cluster Volume chia sẻ tầng lưu trữ, nhân bản 6 data copies trên 3 AZs, các tính năng Backtrack, Cloning, Global DB).
  * Khảo sát dịch vụ kho dữ liệu quy mô lớn Amazon Redshift (kiến trúc song song MPP, lưu trữ dạng cột Columnar Storage, truy vấn S3 trực tiếp qua Redshift Spectrum và Data Sharing).
  * Hiểu rõ vai trò của Amazon ElastiCache (Memcached/Redis) đối với hiệu năng ứng dụng và trách nhiệm của lập trình viên trong việc quản lý Caching Logic & Cache Invalidation.
  * Tìm hiểu cách thức thực hiện di trú database lên đám mây AWS sử dụng AWS SCT và AWS DMS.
* **CI/CD trên EKS với CodePipeline và GitHub**:
  * Nắm vững kiến thức cơ bản về Amazon Elastic Kubernetes Service (Amazon EKS) và cách quản lý các container orchestration cho ứng dụng Kubernetes.
  * Hiểu cơ chế hoạt động của AWS CodePipeline trong việc tự động hóa các deployment pipeline, giúp cập nhật ứng dụng hoặc hạ tầng ổn định khi có thay đổi mã từ GitHub.
  * Thực hành tạo một EKS Cluster đơn giản, triển khai trang web và cấu hình pipeline CI/CD tự động deploy lên môi trường production.


