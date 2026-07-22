---
title: "Worklog Tuần 2"
date: 2026-06-22
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---


### Mục tiêu tuần 2:

* Kết nối, làm quen với các thành viên trong cộng đồng First Cloud AI Journey.
* Nắm vững kiến thức cốt lõi và kỹ năng quản trị máy chủ ảo Amazon EC2, đĩa lưu trữ EBS, Instance Store cùng cơ chế AMI và Snapshot.
* Tìm hiểu giải pháp tự động hóa co giãn hệ thống Auto Scaling (ASG) và điều phối tải Elastic Load Balancer (ELB).
* Làm chủ dịch vụ lưu trữ đối tượng Amazon S3, bao gồm quản trị bảo mật (Bucket Policy, ACL, Block Public Access), phiên bản Versioning, và tối ưu hóa Web tĩnh kết hợp CDN CloudFront.
* Triển khai giải pháp Shared Storage (EFS, FSx) và lưu trữ hỗn hợp Hybrid Cloud sử dụng AWS Storage Gateway (File Gateway) kết nối On-premises lên đám mây.
* Nghiên cứu các chiến lược khắc phục sự cố sau thảm họa (Disaster Recovery) trên AWS dựa theo chỉ số RTO/RPO và thiết lập quản lý sao lưu tập trung qua AWS Backup.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | - **AWS Compute & EC2 Administration** <br>&emsp; + **Compute Theory:** Nghiên cứu kiến trúc máy chủ ảo Amazon EC2 và lớp ảo hóa Nitro Hypervisor tối ưu hiệu năng; phân tích cấu hình Instance Types sử dụng chipset Intel/AMD và chip ARM (AWS Graviton) tiết kiệm 40% chi phí; cơ chế bảo mật đăng nhập Key Pairs (SSH cho Linux, giải mã RDP password cho Windows). <br>&emsp; + **Storage for Compute:** Phân biệt EBS (Block storage kết nối qua mạng riêng, độ sẵn sàng 99.999% nhân bản 3 node trong AZ, EBS Multi-Attach) và Instance Store (NVMe cục bộ tốc độ cao nhưng mang rủi ro Ephemeral dữ liệu bị xóa khi Stop máy chủ); tìm hiểu quản lý AMI và Incremental Snapshot lưu trên S3. <br>&emsp; + **Hands-on Lab:** <br>&emsp;&emsp; * Thực hành khởi tạo và quản lý máy chủ ảo EC2 (Linux & Windows). <br>&emsp;&emsp; * Kết nối SSH qua terminal và Remote Desktop (RDP) an toàn. <br>&emsp;&emsp; * Tạo đĩa EBS volume mới, gán vào instance, phân vùng và mount ổ đĩa. | 22/06/2026   | 22/06/2026      | [Module 3](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%203/module-3.md) |
| 3   | - **HA Operations, Shared Storage & Migration** <br>&emsp; + **Automation & Scaling:** Cấu hình User Data script tự động triển khai Web Server; truy vấn thông tin máy chủ bằng EC2 Metadata từ IP Link-local `169.254.169.254`. Cấu hình Auto Scaling Group (ASG) co giãn máy chủ tự động kết hợp Elastic Load Balancer (ELB) để điều phối tải; tìm hiểu các mô hình giá (On-Demand, RI/Savings Plans, Spot Instances). <br>&emsp; + **Amazon Lightsail:** Điện toán trọn gói giá cố định, tính năng One-Click VPC Peering và Convert snapshot sang EC2. <br>&emsp; + **Shared Storage (EFS & FSx):** Cấu hình Amazon EFS (NFSv4 cho Linux) kết nối đồng thời hàng ngàn instance và Amazon FSx cho Windows Server (SMB/NTFS) với cơ chế chống trùng lặp dữ liệu Data Deduplication tiết kiệm 30-50% chi phí. <br>&emsp; + **Migration & Disaster Recovery (AWS MGN):** Nghiên cứu di trú hệ thống bằng AWS Application Migration Service; cơ chế Agent replication về Staging VPC và Cut-over khởi chạy Target EC2. | 23/06/2026   | 23/06/2026      | [Module 3](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%203/module-3.md) |
| 4   | - **AWS Storage & Disaster Recovery** <br>&emsp; + **Object Storage & Lifecycle:** Nghiên cứu Amazon S3 (độ bền 11 số 9, nhân bản >=3 AZs, S3 Event Trigger, băm Object Key prefix partition); các lớp lưu trữ S3 (Standard, IA, Intelligent-Tiering, Glacier Archive); cơ chế Lifecycle Management và S3 Access Points. <br>&emsp; + **Security & Advanced S3:** Cấu hình S3 Static Website Hosting & CORS; kiểm soát quyền truy cập (IAM Policy vs Resource Policy); thiết lập S3 Gateway Endpoints kết nối mạng nội bộ riêng tư qua IP Private. Bật S3 Versioning (Delete Marker) phòng chống Ransomware. <br>&emsp; + **Glacier Vaults & Vault Lock:** Quản lý Glacier Archives; 3 tốc độ khôi phục dữ liệu (Expedited, Standard, Bulk); thiết lập chính sách Vault Lock tuân thủ pháp lý. <br>&emsp; + **Hybrid Storage & Snow Family:** Chuyển tải dữ liệu offline qua Snow Family (Snowball, Snowball Edge, Snowmobile); triển khai AWS Storage Gateway (File Gateway, Volume Gateway Stored/Cached, Tape Gateway VTL). <br>&emsp; + **Disaster Recovery (DR) & AWS Backup:** Phân tích các chỉ số RTO/RPO; so sánh 4 mô hình DR (Backup & Restore, Pilot Light, Warm Standby, Multi-site Active-Active); cấu hình AWS Backup tập trung và VM Import/Export. | 24/06/2026   | 24/06/2026      | [Module 4](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%204/module-4.md) |
| 5   | - **Thực hành S3 & Tối ưu hóa Web tĩnh (Lab 57):** <br>&emsp; + Khởi tạo S3 Bucket, cấu hình Object Ownership (ACLs enabled) và upload mã nguồn web tĩnh. <br>&emsp; + Bật tính năng Static Website Hosting trên S3. <br>&emsp; + Cấu hình gỡ Block Public Access, thiết lập quyền đọc công khai cho các object bằng ACL. <br>&emsp; + Tích hợp Amazon CloudFront (CDN) làm cổng bảo mật và tăng tốc phân phối web toàn cầu (OAI, Bucket Policy). <br>&emsp; + Bật tính năng Bucket Versioning để bảo vệ dữ liệu chống Ransomware. | 25/06/2026   | 25/06/2026      | [Lab 57](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%204/Hand-ons/Lab57.md) |
| 6   | - **Thực hành Hybrid Storage & Shared Storage (Lab 24 & Lab 25):** <br>&emsp; + **Lab 24 (File Gateway):** Khởi tạo S3 Bucket làm bộ lưu trữ gốc. Khởi chạy máy chủ EC2 Storage Gateway giả lập thiết bị appliance tại chỗ. Cấu hình Inbound Rules mở cổng SMB (445), NFS (111, 2049, 20048). Thiết lập Storage Gateway và File Share SMB trên AWS Console. Mount ổ đĩa mạng chia sẻ trực tiếp lên máy client bằng dòng lệnh CMD, tạo file test và kiểm tra đồng bộ tự động lên S3. <br>&emsp; + **Lab 25 (Amazon FSx for Windows):** Khởi tạo AWS Managed Microsoft AD làm hệ thống xác thực định danh tập trung. Triển khai Amazon FSx for Windows File Server (Single-AZ SSD 32 GiB) kết nối vào Active Directory. Khởi chạy các máy Windows EC2, join domain và mount đĩa mạng chia sẻ qua giao thức SMB, kiểm tra tính đồng bộ dữ liệu thời gian thực. | 26/06/2026   | 26/06/2026      | [Lab 24](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%204/Hand-ons/Lab24.md) <br> [Lab 25](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%204/Hand-ons/Lab25.md) |


### Kết quả đạt được tuần 2:

* **Quản trị Điện toán & Tự động hóa (EC2 & Auto Scaling):**
  * Đã tạo và quản lý máy chủ ảo EC2 (Linux/Windows), cấu hình tường lửa (Security Group) và kết nối SSH/RDP an toàn.
  * Cấu hình thành công bộ lưu trữ EBS (tạo volume mới, gán vào instance, phân vùng và mount ổ đĩa dữ liệu).
  * Sử dụng User Data script để tự động hóa cấu hình Web Server, truy vấn EC2 Metadata qua link-local IP.
  * Thiết lập và kiểm tra hoạt động của Auto Scaling Group (ASG) kết hợp Elastic Load Balancer (ELB) để tự co giãn máy chủ ứng dụng theo tải thực tế.
* **Lưu trữ Đối tượng & Phân phối Toàn cầu (S3 & CloudFront):**
  * Khởi tạo S3 Bucket, kiểm soát quyền truy cập chặt chẽ (gỡ Block Public Access, cấu hình Object Ownership ACLs, nạp Bucket Policy).
  * Kích hoạt Static Website Hosting trên S3, bật Bucket Versioning chống Ransomware.
  * Tạo CloudFront Distribution, cấu hình Legacy access identities (OAI) kết nối ngầm an toàn tới S3, phân phối và tăng tốc web tĩnh trên mạng lưới CDN toàn cầu.
* **Lưu trữ Chia sẻ & Lưu trữ Hỗn hợp (FSx, EFS & Storage Gateway):**
  * Triển khai giải pháp Hybrid Storage: Cấu hình AWS Storage Gateway (File Gateway) trên EC2 giả lập appliance local, tạo File Share SMB kết nối tới S3, mount thành công đĩa mạng `Z:` trên client và kiểm tra đồng bộ tệp tin thời gian thực.
  * Triển khai Shared Storage: Thiết lập AWS Managed Microsoft AD, cấu hình hệ thống tệp tin Amazon FSx for Windows File Server (SMB) và mount đĩa mạng dùng chung đồng thời cho các máy EC2 thuộc Domain.
* **Quản trị Bảo mật & Sao lưu:**
  * Hiểu rõ 4 chiến lược Disaster Recovery (Backup & Restore, Pilot Light, Warm Standby, Multi-site Active-Active) theo yêu cầu RTO/RPO của doanh nghiệp.
  * Biết cách lên lịch trình sao lưu tự động và quản lý chu kỳ sống dữ liệu (Retention policy) qua AWS Backup.


