---
title: "Sự kiện 5"
date: 2026-08-01
weight: 5
chapter: false
pre: " <b> 4.5. </b> "
---

# BÀI THU HOẠCH: AWS FCAJ AGENT FORGE – DEEPDIVE  
## "DAY 1: FOUNDATIONS & AGENT SETUP"

## Mục Đích Của Sự Kiện

- **Mục đích:**
  - Cung cấp kiến thức chuyên sâu cấp độ nâng cao (Level 300) về việc xây dựng, đóng gói và triển khai hệ thống Trí tuệ Nhân tạo Tự chủ (Agentic AI Systems) trên môi trường Production của doanh nghiệp.
  - Tìm hiểu chi tiết dịch vụ Amazon Bedrock AgentCore cùng các thành phần hạ tầng cốt lõi gồm Runtime, Gateway và Identity.
  - Thực hành trực tiếp (Hands-on Lab) triển khai một AI Agent tự chủ, tích hợp các công cụ mở rộng (MCP Tools), cơ sở tri thức (Knowledge Base) và quản trị xác thực người dùng qua Amazon Cognito.

## Danh Sách Diễn Giả & Hướng Dẫn Viên

- **Diễn giả & Hướng dẫn viên:**
  - **Trần Hữu Nghĩa:** Diễn giả chính, phụ trách nội dung Lý thuyết & Kiến trúc hệ thống AgentCore.
  - **Hải Anh:** Diễn giả & Hướng dẫn viên, phụ trách phiên Thực hành (Hands-on Lab).

## Nội Dung Nổi Bật

### 1. Tổng quan về Agentic AI & Mô hình Định vị Tự chủ (Autonomous Spectrum)

- **Bản chất của Agentic AI:** Là hệ thống phần mềm tự chủ (autonomous), có khả năng lập luận (reasoning), lên kế hoạch (planning) và thực thi các chuỗi tác vụ (executing tasks) tuần tự để đạt mục tiêu.
- **Dải tự chủ (Autonomous Spectrum):** Đi từ luồng công việc xác định (Deterministic Workflow — do lập trình viên định nghĩa các bước) đến tự chủ hoàn toàn (Fully Autonomous — các Agent tự liên kết, giao tiếp và ra quyết định). Trong doanh nghiệp, mô hình lai có sự tham gia của con người ở bước quyết định (Human-in-the-loop) luôn được ưu tiên để đảm bảo an toàn.
- **Giao thức kết nối thế hệ mới:** Bên cạnh HTTP/REST API truyền thống, kỷ nguyên AI phát triển 2 giao thức chuẩn hóa mới:
  - **MCP (Model Context Protocol):** Kết nối Agent với các Tool/API bên ngoài.
  - **A2A (Agent-to-Agent Protocol):** Cho phép các Agent giao tiếp trực tiếp với nhau mà không cần lớp trung gian.
- **Bộ phần mềm Strands SDK:** AWS cung cấp bộ mã nguồn mở SDK tên là Strands, thiết kế riêng cho hệ sinh thái AWS để tạo con Agent theo dạng Factory Design Pattern chỉ với vài dòng mã lệnh.

### 2. Amazon Bedrock AgentCore Runtime & Hạ tầng Serverless

- **Kiến trúc Serverless & Mô hình chi phí:** Runtime hoạt động hoàn toàn dưới dạng Serverless, tính phí theo mức độ sử dụng thực tế (Pay-as-you-go), giúp tối ưu hóa chi phí cho doanh nghiệp.
- **Công nghệ đóng gói & Tách biệt môi trường (MicroVM Isolation):** Sử dụng công nghệ **Firecracker MicroVM** để tạo ra các máy ảo siêu nhỏ, tách biệt hoàn toàn tài nguyên phần cứng, bộ nhớ và hệ thống tệp cho từng phiên làm việc của người dùng (User Session). Điều này đảm bảo dữ liệu nội bộ không bị rò rỉ giữa các người dùng.
- **Các phương thức đóng gói triển khai (Deployment):**
  - **Khởi tạo từ mã nguồn:** Dùng Strands SDK cấu hình trực tiếp System Prompt, Model và Tools.
  - **Đóng gói Container:** Đẩy tệp Docker Image lên Amazon ECR.
  - **Lưu trữ mã nguồn Zip:** Đẩy toàn bộ tệp mã nguồn nén lên Amazon S3.
- **Chiến lược phát hành an toàn (Rollout Strategy):** Hỗ trợ gán nhãn Alias/Versioning (ARN) cho từng phiên bản Agent để triển khai thử nghiệm theo tỷ lệ phần trăm traffic (**Canary Deployment**) và khôi phục nhanh (**Rollback**) khi gặp sự cố.

### 3. Lớp quản trị an toàn & Xác thực (AgentCore Identity)

- **Cơ chế Bảo mật Hai chiều (Inbound & Outbound Security):**
  - **Inbound (Đầu vào):** Xác thực người dùng/ứng dụng trước khi truy cập và gọi Agent. Sử dụng JSON Web Token (JWT) hoặc Amazon Cognito User Pool để xác minh danh tính.
  - **Outbound (Đầu ra):** Quản lý quyền hạn của Agent khi gọi đến các công cụ, API hoặc cơ sở dữ liệu bên thứ ba.
- **Mô hình Chuyển đổi Token An toàn (Workload Access Token — WAT):** Khi nhận JWT Token từ người dùng, AgentCore Identity chuyển đổi sang một token trung gian mới (WAT Token) kết hợp giữa danh tính người dùng và danh tính của Agent. Token này được lưu trữ mã hóa trong kho **Token Vault** của AgentCore, tránh việc lộ token gốc của người dùng ra các hệ thống bên ngoài.

### 4. Cổng kết nối trung gian (AgentCore Gateway)

- **Giải pháp cho bài toán mở rộng (Scaling Challenge):** Khi hệ thống có hàng trăm Agent và hàng nghìn công cụ (Tools/MCP Servers), việc kết nối trực tiếp tạo ra sự phức tạp cồng kềnh. Gateway đóng vai trò là một lớp trung gian (Middleware) duy nhất quản lý toàn bộ chính sách bảo mật, xác thực và kiểm soát quyền truy cập.
- **Cơ chế Human-in-the-Loop & Interceptor:**
  - Cho phép người quản trị (Admin) can thiệp trực tiếp để phê duyệt (Approve) hoặc từ chối (Deny) các yêu cầu vượt quá hạn mức hoặc chính sách của Agent.
  - Lớp chặn **Interceptor** tự động lọc các dữ liệu nhạy cảm (như thông tin định danh cá nhân — PII) trước khi trả kết quả về cho người dùng.
- **Tìm kiếm công cụ ngữ nghĩa (Semantic Tool Search):** Tích hợp chỉ mục (Indexing) và tìm kiếm vector trên Gateway để Agent tự động lọc và trích xuất đúng công cụ phù hợp nhất dựa trên mô tả dạng tệp JSON Schema (tên, chức năng, tham số yêu cầu).

## Thảo Luận So Sánh

Dựa trên nội dung được trình bày tại sự kiện, dưới đây là bảng đối chiếu giữa mô hình triển khai truyền thống và kiến trúc Amazon Bedrock AgentCore:

| Tiêu chí | Mô hình Truyền thống | Kiến trúc Amazon Bedrock AgentCore |
| :--- | :--- | :--- |
| **Môi trường chạy Agent** | Máy chủ cố định hoặc container tự quản lý; cần cấu hình thủ công, chi phí luôn phát sinh dù không có tải. | Serverless hoàn toàn (Firecracker MicroVM); tự động co giãn, tính phí theo lượng dùng thực tế (Pay-as-you-go). |
| **Tách biệt dữ liệu người dùng** | Khó đảm bảo tách biệt triệt để; rủi ro rò rỉ dữ liệu giữa các phiên. | MicroVM Isolation tạo môi trường tách biệt hoàn toàn cho từng User Session — không có rò rỉ chéo. |
| **Triển khai phiên bản mới** | Cập nhật toàn bộ (Big-bang deploy); dừng dịch vụ, rủi ro cao khi rollback. | Alias/Versioning (ARN) + Canary Deployment theo tỷ lệ traffic; rollback tức thì chỉ bằng cập nhật Alias. |
| **Xác thực & Bảo mật** | Tự triển khai xác thực; token người dùng truyền trực tiếp đến hệ thống ngoài — rủi ro lộ thông tin. | AgentCore Identity: JWT → WAT Token; Token Vault mã hóa — token gốc không bao giờ rời khỏi hệ thống kiểm soát. |
| **Kết nối & Quản lý Tools** | Kết nối trực tiếp Agent↔Tool; khi mở rộng lên hàng trăm tools trở nên cồng kềnh và khó bảo mật. | AgentCore Gateway: lớp Middleware trung tâm; Semantic Tool Search tự động khớp tool đúng theo ngữ nghĩa. |
| **Lọc dữ liệu nhạy cảm** | Phải tự viết logic lọc PII và dữ liệu nhạy cảm ở từng điểm tích hợp. | Interceptor tích hợp sẵn trong Gateway; tự động phát hiện và lọc PII trước khi trả kết quả về người dùng. |

## Những Gì Học Được

### Tư Duy Thiết Kế

- **Tư duy thiết kế hệ thống Production-Ready:** Một hệ thống Agent thực tế không chỉ dừng ở bước Proof-of-Concept (POC) trên Jupyter Notebook, mà phải đáp ứng đầy đủ các tiêu chuẩn doanh nghiệp về khả năng mở rộng (Scalability), an toàn bảo mật (Security), quan sát hệ thống (Observability) và tối ưu chi phí (Cost Efficiency).
- **Kiến trúc phân lớp trung gian (Middleware Pattern):** Tách biệt rõ vai trò giữa hạ tầng chạy code (Runtime), hạ tầng xác thực (Identity) và hạ tầng điều phối công cụ (Gateway) để dễ dàng bảo trì và mở rộng hệ thống mà không tạo sự phụ thuộc chặt chẽ (Tight Coupling) giữa các thành phần.

### Kiến Trúc Kỹ Thuật

- **Triển khai hạ tầng mạng an toàn (Hybrid/Private Connectivity):** Sử dụng **AWS PrivateLink** và **NAT Gateway** để kết nối an toàn giữa AgentCore với hệ thống ứng dụng nội bộ (On-Premise) hoặc các hạ tầng VPC riêng biệt mà không cần mở kết nối Public Internet.
- **Xử lý phản hồi thời gian thực (Bidirectional Streaming):** Thiết lập luồng truyền tải dữ liệu hai chiều (Text/Voice/Vision) giúp Agent phản hồi câu trả lời liên tục theo dạng từng cụm từ (Streaming), giảm tối đa thời gian chờ đợi (Latency) của người dùng.

### Chiến Lược Phát Triển Cá Nhân

- **Tiếp cận tài liệu tiêu chuẩn L300:** Rèn luyện khả năng tiếp thu lượng tri thức chuyên sâu từ các chương trình đào tạo chính thức của AWS dành cho kỹ sư cấp cao, đòi hỏi nền tảng vững và tư duy hệ thống (Systems Thinking) để hiểu đúng bản chất.
- **Lựa chọn mô hình AI tối ưu:** Học cách đánh giá và thử nghiệm đa dạng các dòng mô hình (Anthropic Claude, Amazon Nova) để chọn đúng mô hình phù hợp với yêu cầu bài toán (nhanh, rẻ hay cần tư duy logic phức tạp).

## Ứng Dụng Vào Công Việc

- **Khởi tạo và đóng gói Agent bằng Strands SDK:** Thực hành viết mã nguồn cấu hình Agent, tích hợp các công cụ tra cứu tự động và đóng gói thành tệp Docker Container để sẵn sàng tải lên Amazon ECR.
- **Cấu hình lớp xác thực Amazon Cognito:** Xây dựng luồng đăng nhập/đăng ký người dùng chuẩn hóa, tích hợp phát hành token JWT để kiểm soát quyền truy cập tới AgentCore Runtime.
- **Thực hành thiết kế JSON Schema cho MCP Tools:** Chuẩn hóa các tệp mô tả công cụ (mô tả rõ ràng tên, chức năng và tham số) giúp Agent dễ dàng nhận diện và thực thi lệnh chính xác thông qua cơ chế Semantic Tool Search trên Gateway.

## Trải nghiệm trong event

- **Khối lượng tri thức cô đọng và thực chiến:** Sự kiện truyền tải lượng kiến thức nền tảng đồ sộ nhưng được cô đọng bài bản, kết hợp giữa bài giảng lý thuyết chuyên sâu và phần thực hành lab củng cố ngay tại chỗ, giúp người tham dự hiểu từ nguyên lý đến cách triển khai cụ thể.
- **Trải nghiệm giao diện console trực quan:** Việc được trực tiếp thao tác trên giao diện Amazon Bedrock AgentCore giúp hình dung rõ ràng toàn bộ luồng cấu hình từ Runtime, Gateway cho đến kiểm tra các tệp log vận hành — điều mà đọc tài liệu đơn thuần không thể cung cấp.

## Bài học rút ra

- **Một Agent hoạt động tốt trên môi trường thử nghiệm chưa chắc đã sẵn sàng cho Production;** sự khác biệt then chốt nằm ở lớp quản trị bảo mật (Identity) — xác thực ai gọi Agent và Agent được phép gọi đi đâu — cùng khả năng kiểm soát luồng dữ liệu (Gateway) để lọc thông tin nhạy cảm và áp dụng chính sách doanh nghiệp một cách nhất quán.
- **Việc ứng dụng các chuẩn giao thức mở (MCP, A2A)** giúp các Agent có khả năng linh hoạt kết nối với mọi hệ thống công cụ mà không bị phụ thuộc vào một nhà cung cấp duy nhất (Vendor Lock-in), mở ra khả năng tích hợp với hệ sinh thái công cụ rộng lớn hơn trong tương lai.

## Một số hình ảnh khi tham gia sự kiện

![event5](/images/event5.jpg)
