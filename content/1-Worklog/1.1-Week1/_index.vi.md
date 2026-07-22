---
title: "Worklog Tuần 1"
date: 2026-06-15
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---


### Mục tiêu tuần 1:

* Kết nối, làm quen với các thành viên trong First Cloud AI Journey.
* Hiểu dịch vụ AWS cơ bản, cách dùng console & CLI.
* Hiểu các kiến thức cơ bản về điện toán đám mây và hạ tầng toàn cầu của AWS.
### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | - Làm quen với các thành viên FCAJ và đọc quy định nội bộ. <br> - **Nghiên cứu AWS Module 1 & Thực hành Lab 12:** <br>&emsp; + **Lý thuyết:** Khái niệm Điện toán đám mây & 4 lợi ích cốt lõi; Sự khác biệt của AWS (Economies of Scale, Leadership Principles); Hạ tầng toàn cầu (Region, AZ, Edge Location); Công cụ quản lý (Console, CLI, SDK); Tối ưu chi phí & các gói AWS Support. <br>&emsp; + **Thực hành Lab 12 (Identity Center):** Kích hoạt AWS Organizations, cấu hình AWS IAM Identity Center tập trung, khởi tạo User/Group, phân quyền qua Permission Sets truy cập các tài khoản thành viên, thiết lập Time-based Access Control và kiểm tra đăng nhập qua Portal & CLI. | 15/06/2026 | 15/06/2026 | [Module 1](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%201/module-1.md) <br> [Lab 12](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%201/Hand-ons/Lab12.md) |
| 3   | - **Triển khai AWS Intermediate Workshop & VPC Basics:** <br>&emsp; + **Identity & CLI:** Thiết lập AWS IAM Identity Center (User, Group, Permission Sets), cài đặt và cấu hình AWS CLI v2. <br>&emsp; + **Mạng cơ bản (Amazon VPC):** Nghiên cứu khái niệm Amazon VPC (vùng mạng cô lập logic, Multi-AZ) và phân chia Subnet (Public Subnet định tuyến ra internet, Private Subnet kết nối nội bộ; lưu ý 5 IP Reserved của AWS). <br>&emsp; + **Thành phần mạng:** Tìm hiểu vai trò của Route Table (bảng định tuyến chính và tùy chỉnh), cổng Internet Gateway (IGW), card mạng ảo Elastic Network Interface (ENI), địa chỉ IP tĩnh Elastic IP (EIP), cổng dịch chuyển NAT Gateway hỗ trợ Private Subnet truy cập ra ngoài, và VPC Endpoint (Interface/Gateway Endpoint) kết nối nội bộ riêng tư (AWS PrivateLink) không qua Internet công cộng. | 16/06/2026   | 16/06/2026      | [Module 2](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%202/module-2.md) |
| 4  | - **Quản trị Bảo mật & Bảo mật mạng VPC:** <br>&emsp; + **Khung lý thuyết:** Nghiên cứu sâu lý thuyết khung AWS CAF (Directive) và NIST CSF (Identify) phục vụ xây dựng chiến lược an toàn thông tin. <br>&emsp; + **Bảo mật mạng VPC:** Phân tích cơ chế hoạt động của tường lửa ảo Security Group (SG - Stateful ở cấp độ ENI, chỉ hỗ trợ luật Allow) và Network ACL (NACL - Stateless ở cấp độ Subnet, hỗ trợ cả luật Allow/Deny, đọc theo thứ tự ưu tiên). <br>&emsp; + **Giám sát lưu lượng:** Cấu hình VPC Flow Logs ghi nhận nhật ký lưu lượng IP đến/đi phục vụ kiểm toán và phát hiện rủi ro bảo mật. | 17/06/2026   | 17/06/2026      | [Module 2](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%202/module-2.md) |
| 5   | - **Kết nối liên VPC & Xử lý sự cố:** <br>&emsp; + **Troubleshooting:** Tổng hợp và xử lý các lỗi cấu hình CLI, lỗi đồng bộ OIDC xác thực và phân quyền IAM cho CloudWatch Logs. <br>&emsp; + **Kết nối liên VPC:** Phân biệt giải pháp kết nối VPC Peering (kết nối trực tiếp 1-1, non-transitive, không trùng CIDR) và giải pháp bộ định tuyến trung tâm Transit Gateway (TGW - kiến trúc Hub-and-Spoke, quản lý tập trung và đơn giản hóa định tuyến quy mô lớn qua Transit Gateway Attachment). | 18/06/2026   | 18/06/2026      | [Module 2](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%202/module-2.md) |
| 6   | - **Mạng hỗn hợp Hybrid & Dịch vụ cân bằng tải (ELB):** <br>&emsp; + **Mạng Hybrid:** Nghiên cứu các phương thức kết nối On-premises lên đám mây bao gồm VPN Site-to-Site (mã hóa IPsec qua CGW & VGW), Client VPN kết nối từ xa, và đường truyền vật lý chuyên dụng AWS Direct Connect (Dedicated/Hosted) cho độ trễ cực thấp. <br>&emsp; + **Cân bằng tải Elastic Load Balancing (ELB):** Tìm hiểu cơ chế hoạt động của ELB (Health check, Sticky sessions, Access logs); phân loại và ứng dụng Application Load Balancer (ALB - Layer 7, Path/Host-based routing), Network Load Balancer (NLB - Layer 4, hiệu năng cao, IP tĩnh), và Gateway Load Balancer (GWLB - Layer 3, GENEVE port 6081 chuyển traffic qua Virtual Appliance). | 19/06/2026   | 19/06/2026      | [Module 2](https://github.com/namcris07/aws-fcaj-learning/blob/main/First%20Cloud%20AI%20Journey/Module%202/module-2.md) |


### Kết quả đạt được tuần 1:

* **Nắm vững kiến thức nền tảng về AWS và Điện toán đám mây:**
  * Hiểu rõ khái niệm Điện toán đám mây và sự khác biệt giữa hạ tầng truyền thống (CapEx) và đám mây (OpEx).
  * Phân tích và nắm vững 4 lợi ích cốt lõi của đám mây: Tối ưu chi phí, Tốc độ & Linh hoạt, Khả năng co giãn và Quy mô toàn cầu.
  * Hiểu sâu về hạ tầng toàn cầu của AWS bao gồm Data Center, Vùng sẵn sàng (AZ) với thiết kế cô lập lỗi, Region và Edge Location (phục vụ CloudFront CDN, WAF, Route 53).
  * Tiếp cận văn hóa vận hành của AWS qua các nguyên tắc Leadership Principles (Customer Obsession, Ownership, Deliver Results) và triết lý giá Lợi thế quy mô (Economies of Scale).

* **Làm chủ các công cụ quản lý và tương tác AWS:**
  * Biết cách sử dụng AWS Management Console, cài đặt và cấu hình dòng lệnh AWS CLI (sử dụng Access Key/Secret Access Key) cũng như tương tác qua AWS SDK.
  * Hiểu và áp dụng cơ chế phân quyền bảo mật: phân biệt Root User (luôn bật MFA) và IAM User áp dụng nguyên tắc đặc quyền tối thiểu (Least Privilege).
  * Triển khai quản lý định danh tập trung qua AWS IAM Identity Center và AWS Organizations; thiết lập Permission Sets (bộ quyền), tạo User/Group quản lý truy cập đa tài khoản, cấu hình kiểm soát quyền hạn có điều kiện theo ca làm việc (Time-based Access Control) và đăng nhập an toàn qua AWS CLI/Portal.

* **Quản lý chi phí và gói hỗ trợ AWS Support:**
  * Nắm vững các chiến lược tối ưu chi phí (Right-sizing, Serverless) và phân biệt các mô hình mua tài nguyên EC2 (On-Demand, Reserved Instances, Spot Instances).
  * Biết cách sử dụng AWS Budgets để giám sát ngân sách, thiết lập cảnh báo vượt ngưỡng và sử dụng Cost Allocation Tags.
  * Phân biệt đặc quyền và phạm vi của các gói hỗ trợ kỹ thuật AWS Support (Basic, Developer, Business, Enterprise).

* **Kiến thức cốt lõi về Amazon VPC & Thành phần mạng:**
  * Hiểu khái niệm Virtual Private Cloud (VPC) và thiết lập các Subnet (Public Subnet định tuyến ra Internet Gateway, Private Subnet kết nối qua NAT Gateway).
  * Nắm vững vai trò của Elastic Network Interface (ENI), Elastic IP (EIP) và Route Table trong việc định hướng đi của luồng dữ liệu.
  * Hiểu cơ chế hoạt động của VPC Endpoint giúp kết nối nội bộ riêng tư tới các dịch vụ AWS khác (S3, DynamoDB) không qua Internet công cộng.

* **Cơ chế bảo mật mạng và các giải pháp kết nối liên VPC:**
  * So sánh và phân biệt rõ cơ chế hoạt động của Security Group (Stateful, cấp ENI, chỉ Allow) và Network ACL (NACL - Stateless, cấp Subnet, hỗ trợ cả Allow/Deny).
  * Tìm hiểu tính năng ghi nhật ký lưu lượng mạng VPC Flow Logs để phục vụ phân tích bảo mật và phát hiện lưu lượng bất thường.
  * Hiểu rõ hai phương thức kết nối liên VPC: VPC Peering (kết nối trực tiếp 1-1, non-transitive) và Transit Gateway (bộ định tuyến trung tâm Hub-and-spoke cho quy mô lớn).

* **Giải pháp mạng Hybrid và dịch vụ cân bằng tải Elastic Load Balancing (ELB):**
  * Nghiên cứu các phương thức kết nối mạng hỗn hợp Hybrid: VPN Site-to-Site, Client VPN, và đường truyền vật lý riêng AWS Direct Connect (kết hợp VPN để bảo mật).
  * Hiểu rõ cơ chế hoạt động của Elastic Load Balancing (ELB), cơ chế kiểm tra sức khỏe máy chủ (Health check) và giữ cố định phiên làm việc (Sticky sessions).
  * Phân loại và nắm vững các loại Load Balancer chính: Application Load Balancer (ALB - Layer 7), Network Load Balancer (NLB - Layer 4), và Gateway Load Balancer (GWLB).