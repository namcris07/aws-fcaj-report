---
title : "Các lỗ hổng bảo mật thường gặp"
date : 2026-07-06 
weight : 1
chapter : false
pre : " <b> 5.4.1. </b> "
---

# 5.4.1 Các lỗ hổng bảo mật thường gặp

---

### 1. Lộ lọt thông tin nhạy cảm - Hardcoded Secrets

Quá trình phát triển nhanh chóng và thiếu kiểm soát thường dẫn đến việc lập trình viên vô tình đóng cứng (hardcode) các thông tin xác thực trực tiếp vào mã nguồn hoặc các tệp tin cấu hình. Các thông tin này thường bao gồm mật khẩu kết nối cơ sở dữ liệu, API Keys của các nhà cung cấp dịch vụ đám mây (AWS, GCP, Azure), khóa mã hóa (private keys), hoặc Access Token.

- **Cơ chế khai thác:** Khi mã nguồn bị đẩy lên các hệ thống quản lý phiên bản (như GitHub, GitLab, Bitbucket), kẻ tấn công sử dụng các bot rà quét tự động để lùng sục các chuỗi ký tự có định dạng giống với khóa bí mật. Quá trình này diễn ra chỉ trong vài giây.
- **Hậu quả:** Tin tặc có thể sử dụng các khóa này để truy cập trực tiếp vào cơ sở dữ liệu, đánh cắp dữ liệu khách hàng, hoặc lạm dụng tài nguyên đám mây để đào tiền ảo, gây thiệt hại lớn về mặt tài chính cho tổ chức.

---

### 2. Lỗ hổng kiểm soát truy cập - Broken Access Control

Đây là nhóm lỗ hổng luôn nằm trong top đầu của tiêu chuẩn OWASP, xảy ra khi hệ thống không xác thực và ủy quyền chặt chẽ các yêu cầu truy cập từ người dùng.

- **Leo thang đặc quyền ngang (Horizontal Privilege Escalation):** Kẻ tấn công lợi dụng lỗ hổng Tham chiếu đối tượng trực tiếp không an toàn (IDOR) để truy cập, chỉnh sửa hoặc xóa dữ liệu của một người dùng khác có cùng cấp độ (ví dụ: thay đổi tham số ID trong URL).
- **Leo thang đặc quyền dọc (Vertical Privilege Escalation):** Kẻ tấn công vượt qua các cơ chế kiểm tra quyền hạn để truy cập vào các API hoặc chức năng quản trị hệ thống, từ đó thao túng toàn bộ dữ liệu hoặc thay đổi cấu hình ứng dụng.

---

### 3. Lỗi tiêm nhiễm - Injection Flaws

Lỗi tiêm nhiễm luôn nằm trong nhóm những rủi ro bảo mật nghiêm trọng nhất. Lỗi này xảy ra khi ứng dụng tiếp nhận luồng dữ liệu đầu vào từ phía người dùng nhưng không thực hiện cơ chế xác thực chặt chẽ hoặc làm sạch dữ liệu trước khi chuyển đến các trình thông dịch.

**Nguồn gốc phát sinh:** Lỗi này chủ yếu bắt nguồn từ thói quen lập trình thiếu an toàn, khi các nhà phát triển mặc định tin tưởng dữ liệu đầu vào và sử dụng phương pháp nối chuỗi văn bản để xây dựng các câu lệnh động thay vì sử dụng các truy vấn có tham số hóa. Các luồng dữ liệu độc hại này thường xâm nhập vào hệ thống thông qua nhiều điểm tiếp xúc khác nhau như các trường nhập liệu trên biểu mẫu web, tham số trên thanh địa chỉ URL, thông tin trong tiêu đề HTTP, tập tin cookie hoặc các gói dữ liệu giao tiếp qua giao diện lập trình ứng dụng (API).

![Sơ đồ minh họa luồng tấn công khi đầu vào URL không được xác thực dẫn đến rò rỉ dữ liệu](/images/5-Workshop/5.4-Security/url_attack_diagram.png)

*Hình 5.4.1: Sơ đồ minh họa luồng tấn công khi đầu vào URL không được xác thực dẫn đến rò rỉ dữ liệu.*

Các dạng tiêm nhiễm nguy hiểm và mang tính tàn phá cao nhất đối với hệ thống web bao gồm:

- **SQL hoặc NoSQL Injection:** Kẻ tấn công chèn các đoạn mã truy vấn độc hại qua các biểu mẫu đăng nhập hoặc tham số URL. Nếu thành công, tin tặc có thể vượt qua bước xác thực, trích xuất toàn bộ bảng dữ liệu, thay đổi cấu trúc hoặc xóa hoàn toàn cơ sở dữ liệu.
- **OS Command Injection:** Xảy ra khi ứng dụng chuyển thẳng dữ liệu của người dùng vào các câu lệnh hệ thống. Tin tặc có thể lợi dụng điều này để thực thi mã từ xa trực tiếp trên máy chủ chứa ứng dụng, từ đó chiếm quyền kiểm soát hệ điều hành.
- **Server-Side Template Injection (SSTI):** Kẻ tấn công lợi dụng việc ứng dụng web kết xuất giao diện thông qua các hệ thống mẫu động một cách không an toàn. Bằng cách chèn các biểu thức toán học hoặc mã độc vào các trường dữ liệu, tin tặc có thể lừa máy chủ thực thi mã từ xa hoặc đọc các tệp tin cấu hình nhạy cảm.
- **XML External Entity Injection (XXE):** Kẻ tấn công thao túng các tệp cấu trúc dữ liệu XML do người dùng tải lên để ép trình phân tích cú pháp của máy chủ truy xuất các tệp tin hệ thống cục bộ hoặc quét toàn bộ mạng nội bộ.
- **LDAP Injection:** Xảy ra khi hệ thống sử dụng dữ liệu chưa được làm sạch để cấu trúc các câu lệnh truy vấn danh bạ máy chủ mạng nội bộ. Kẻ tấn công có thể lợi dụng lỗi này để trích xuất cấu trúc phân quyền người dùng hoặc leo thang đặc quyền.

---

### 4. Lỗ hổng từ các thành phần phụ thuộc - Vulnerable Dependencies

Ứng dụng hiện đại thường tích hợp hàng ngàn thư viện mã nguồn mở thông qua các trình quản lý gói như `npm`, `pip` hay `maven`. Dù giúp tăng tốc độ phát triển, điều này cũng mở rộng đáng kể bề mặt tấn công với các rủi ro cốt lõi sau:

- **Rủi ro chuỗi cung ứng (Supply Chain Risk):** Khi một thư viện bên thứ ba chứa lỗ hổng bảo mật điển hình như Log4Shell, mọi ứng dụng sử dụng nó đều tự động bị ảnh hưởng. Kẻ tấn công chỉ cần khai thác điểm yếu từ các gói phần mềm này thay vì nhắm trực tiếp vào hệ thống đích.
- **Phụ thuộc gián tiếp (Transitive Dependencies):** Các thư viện trực tiếp thường gọi đến hàng chục thư viện con khác tạo thành một cây phụ thuộc phức tạp. Lỗ hổng ẩn nấp sâu bên trong các tầng này cực kỳ khó phát hiện nếu chỉ rà soát bằng phương pháp thủ công.
- **Giả mạo gói tin (Typosquatting & Dependency Confusion):** Kẻ tấn công tạo ra các thư viện mã độc có tên gần giống thư viện phổ biến hoặc đánh lừa hệ thống tải nhầm gói tin giả mạo. Mã độc sẽ lập tức được thực thi ngay trong quá trình xây dựng mã nguồn hoặc trên máy chủ vận hành.

---

### 5. Lỗi cấu hình bảo mật - Security Misconfigurations

Lỗi cấu hình có thể xuất hiện ở bất kỳ tầng nào của hệ thống: từ máy chủ web, ứng dụng, hệ quản trị cơ sở dữ liệu cho đến môi trường đám mây. Các sai lệch cấu hình thường gặp bao gồm:

- Không vô hiệu hóa các tài khoản và mật khẩu mặc định của nhà cung cấp.
- Bỏ ngỏ các dịch vụ nội bộ (như dashboard quản trị, cổng metrics Prometheus) ra ngoài Internet mà không yêu cầu xác thực.
- Trả về các thông báo lỗi quá chi tiết (Stack Trace) trên giao diện người dùng, làm rò rỉ thông tin về phiên bản phần mềm, cấu trúc thư mục hoặc tên cơ sở dữ liệu.

---

### 6. Rủi ro từ môi trường Container - Container Image Vulnerabilities

Công nghệ Container (tiêu biểu là Docker) đóng vai trò cốt lõi trong kiến trúc Cloud-native, giúp đóng gói và triển khai ứng dụng nhất quán. Tuy nhiên, chúng cũng mang đến các rủi ro đặc thù:

- **Bề mặt tấn công rộng (Large Attack Surface):** Việc sử dụng các Base Image đầy đủ (như Ubuntu, Debian) chứa rất nhiều công cụ và thư viện hệ điều hành dư thừa, vô tình đưa các lỗ hổng hệ điều hành (OS CVEs) vào bên trong Container.
- **Vấn đề đặc quyền (Root Privileges):** Rất nhiều Image mặc định chạy tiến trình dưới quyền root. Nếu tin tặc tìm được cách xâm nhập vào ứng dụng bên trong, chúng sẽ có đặc quyền cao nhất trong Container đó, tạo tiền đề để tấn công thoát vùng (Breakout).

---

### 7. Cấu hình sai hạ tầng và Kubernetes - IaC Misconfigurations

Trong mô hình DevSecOps, hạ tầng hệ thống được định nghĩa hoàn toàn bằng mã (Infrastructure as Code - IaC). Các tệp tin cấu hình YAML nếu không được kiểm soát nghiêm ngặt sẽ dẫn đến các lỗ hổng nghiêm trọng ở mức kiến trúc cụm (cluster):

- **Cấu hình quyền lỏng lẻo (Overpermissive RBAC):** Cấp quá nhiều quyền không cần thiết cho các ServiceAccount, cho phép một Pod bị xâm nhập có thể kiểm soát các tài nguyên khác trong cụm.
- **Thiếu chính sách mạng (Network Policies):** Mặc định, mọi Pod trong Kubernetes đều có thể giao tiếp hai chiều với nhau. Nếu không giới hạn kết nối, một Pod nhiễm mã độc có thể quét và tấn công lây lan theo chiều ngang (lateral movement).
- **Thiếu giới hạn tài nguyên (Resource Limits):** Không thiết lập hạn mức CPU và bộ nhớ cho Pod, tạo điều kiện cho các cuộc tấn công từ chối dịch vụ (DoS) làm cạn kiệt tài nguyên của toàn bộ Worker Node.
