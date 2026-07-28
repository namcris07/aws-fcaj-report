---
title: "Sự kiện 4"
date: 2026-07-25
weight: 4
chapter: false
pre: " <b> 4.4. </b> "
---

# BÀI THU HOẠCH: FCAJ X AGENTIC AI BUILD WEEK – "SHOW UP. BUILD. PITCH. WIN!"

## Mục Đích Của Sự Kiện

- **Mục đích:**
  - Tạo môi trường thực chiến cho các lập trình viên, kỹ sư dữ liệu và sinh viên tham gia xây dựng các ứng dụng trí tuệ nhân tạo tự chủ (Agentic AI) giải quyết bài toán thực tế của doanh nghiệp.
  - Chia sẻ giải pháp kiến trúc cloud-native trên nền tảng AWS và các kinh nghiệm thực tế từ các đội thi đạt giải tại cuộc thi Hackathon Agentic AI Build Week (được đồng tổ chức cùng GenAI Fund).
  - Truyền cảm hứng về tư duy đổi mới (innovative mental model), tinh thần học tập suốt đời (lifelong learning) và kỹ năng làm việc nhóm trong kỷ nguyên AI Agent.

## Danh Sách Diễn Giả & Đại Diện Các Đội Thi

- **Danh sách diễn giả & đội thi:**
  - **Joseph Marazota:** Head of Technology tại AWS.
  - **Nguyễn Gia Hưng:** Head of Solution Architect tại AWS Việt Nam, Founder FCAJ.
  - **One Team (Đội đạt giải Nhất AWS Track):** Đại diện bởi anh Chung và các thành viên.
  - **Signal Scout (Đội đạt giải Nhì AWS Track):** Hoàng Hiếu, Triệu Quốc Hào, Minh Quân (Willer), Nguyễn Công Minh, Nguyễn Văn Duy Khiêm, Lê Tấn Lực.
  - **Plan V Team (Solution Architect Professional Native App):** Huỳnh Hoàng Long, Trần Đại Vĩ, Phạm Tiến Thuận Phát, Nguyễn An, Lê Minh Nghĩa.
  - **3KA Team (S.H.E.P.H.E.R.D.):** Huỳnh An Khương, Nguyễn Quốc Huy, Ngô Quang Khôi, Hoàng Lê Thành Đức, Đặng Nguyễn Phước Lộc, Đặng Trường Hưng.
  - **Six Pillars Team (Adaptive Workflow Engine):** Anh Việt, Anh Nguyễn Văn Linh, Nguyên, Minh Nhật, Phước Huyền.

## Nội Dung Nổi Bật

### 1. Khai mạc & Định hướng tư duy trong kỷ nguyên AI Agent (Joseph Marazota)

- **Sự dịch chuyển tốc độ phát hành (Release Velocity):** Trước đây doanh nghiệp mất 1 quý hoặc 2 tuần cho một đợt phát hành phần mềm, trong khi kỷ nguyên AI Agent cho phép thực hiện cập nhật/phát hành theo từng phút.
- **Thách thức tư duy cũ (Legacy Mental Models):** Người trẻ cần chủ động phản biện (challenge) những rào cản "không thể làm được" từ các quy trình truyền thống; tận dụng lợi thế không bị vướng bận bởi tư duy hạ tầng cũ (mainframe/legacy systems) để dẫn dắt sự thay đổi.
- **Con người trong vòng lặp đổi mới (Human in the Loop):** AI và Robot (như 1 triệu robot tại kho vận Amazon) chỉ là phần cứng nếu thiếu sự định hướng của con người. Kỹ sư đóng vai trò ra quyết định dựa trên các đề xuất từ AI Agent.

### 2. Giải pháp AI-Powered Conversational Ordering – KFC Agent (One Team – Giải Nhất)

- **Bài toán thực tế:** Phân tích thất bại của McDonald's khi thử nghiệm AI đặt hàng qua Drive-through do hiện tượng AI "ảo giác" (hallucination) và không hiểu ngữ cảnh hội thoại tự nhiên (như việc đặt nhầm 100 miếng gà).
- **Giải pháp không chuyển ứng dụng (No App-Switching):** Xây dựng Agent tích hợp trực tiếp trên ứng dụng nhắn tin sẵn có (Zalo/WhatsApp) để người dùng đặt hàng trực tiếp ngay trong đoạn chat mà không cần tải ứng dụng mới hay đăng ký tài khoản.
- **Kiến trúc Kỹ thuật AWS:**
  - Sử dụng **Amazon Bedrock Agent Core** đóng vai trò quản lý bộ nhớ (session memory) giúp ghi nhớ thói quen đặt hàng của từng người dùng.
  - Tích hợp **AWS WAF** bảo vệ hạ tầng, **Tiny Fish** (AI Web Scraping) để cào dữ liệu thực đơn từ website chính thức của KFC và lưu trữ trên AWS Database.
- **Hiệu quả chi phí:** Chi phí vận hành chỉ khoảng **$0.006 / đơn hàng**, tiết kiệm 75% chi phí hạ tầng so với mô hình xử lý truyền thống nhờ thay thế các lớp xử lý cồng kềnh bằng Agent Core.

### 3. Hệ thống phân tích đối thủ cạnh tranh Multi-Agent – Signal Scout (Signal Scout – Giải Nhì)

- **Bản chất nghiệp vụ:** Thu thập các tín hiệu thông tin rời rạt công khai trên thị trường (báo cáo tài chính, phát biểu cổ đông, mã số thuế) của đối thủ cạnh tranh, tự động xâu chuỗi và dự báo chỉ số ROI nếu doanh nghiệp áp dụng mô hình/chiến lược tương tự.
- **Kiến trúc Multi-Agent (A2A – Agent to Agent):**
  - **Agent Management (Supervisor):** Điều phối các Sub-agent bên dưới.
  - **Crawler Subagent:** Lựa chọn linh hoạt giữa **Apify** (cho trang web tĩnh) và **Tiny Fish** (cho trang web động, bypass qua login wall công khai).
  - **Analysis Subagent & Validation Loop:** Sử dụng **Langfuse** để chấm điểm chất lượng dữ liệu. Nếu điểm thấp, hệ thống tự động yêu cầu retrieve lại (tối đa 2 lần để tiết kiệm token); nếu vẫn không đạt mới đẩy sang luồng human review.
- **Tối ưu hóa chi phí & Tuân thủ (Compliance):** Chuyển đổi từ các công cụ bên thứ ba (đẩy chi phí lên **$94/tháng**) sang sử dụng các **Native AWS Browser/Web Tools** để giảm chi phí xuống **$35/tháng** và đảm bảo lưu trữ dữ liệu tại chỗ (Data Residency).

### 4. Ứng dụng SA Professional AI Native App (Plan V Team)

- **Nỗi đau ngành Solutions Architect (SA):** Yêu cầu thiết kế kiến trúc, trích xuất yêu cầu và báo giá hệ thống gấp trong thời gian ngắn từ khách hàng.
- **Quy trình xử lý của AI Native App:** Nhập yêu cầu bằng ngôn ngữ tự nhiên (Prompt/Document) $\rightarrow$ AI phân tích chính sách/tài liệu doanh nghiệp $\rightarrow$ Tự động vẽ diagram trên **Draw.io** (sử dụng chuẩn bộ icon AWS chính thức) $\rightarrow$ Tự động tính toán bảng giá (Cost Estimation cho region ap-southeast-1) $\rightarrow$ Gen mã IaC (**Terraform/CloudFormation**) $\rightarrow$ Tự động deploy hạ tầng lên AWS.
- **Kiểm soát chất lượng đầu ra (Output Validation):** Sử dụng các gói mã nguồn mở chính thức của AWS (AWS Icons) và áp dụng cơ chế Validation Script để kiểm tra/chặn các dịch vụ nằm trong danh sách cấm (Blacklist) của doanh nghiệp.

### 5. Hệ thống S.H.E.P.H.E.R.D. – Smart Human-flow Evaluation, Prediction, Hazard Detection, Response, and Dispatch (3KA Team)

- **Ý tưởng:** Giảm ùn tắc tại sân bay, siêu thị hoặc sự kiện bằng cách tự động theo dõi, đo lường mật độ đám đông và dự báo nguy cơ ùn tắc qua camera giám sát theo thời gian thực.
- **Kiến trúc Kỹ thuật:**
  - Truyền luồng video qua **Amazon Kinesis Video Streams** vào **AWS Fargate**.
  - Áp dụng **YOLOv8/YOLOv11** kết hợp **ByteTrack** để nhận diện người, gán ID theo dõi và xác định độ tự tin (Confidence Score).
  - Cho phép người vận hành khoanh vùng (Edit Zone) để tính toán mật độ di chuyển theo thời gian thực. Kết hợp **Bedrock Agent** (Autonomous Monitor & Operator Copilot) để phân tích và đưa ra đề xuất điều phối nhân sự.

### 6. Khung quy trình phát hiện phòng chống rửa tiền – Adaptive Workflow Engine (Six Pillars)

- **Thực trạng ngành Ngân hàng/Fintech:** 90–95% cảnh báo giao dịch đáng ngờ là cảnh báo sai (False Positive), khiến chuyên viên phân tích mất trung bình **3 giờ/case** và chi phí **$20–$25** cho mỗi lần rà soát thủ công.
- **Kiến trúc xử lý 3 tầng (3-Layer Architecture):**
  - **Layer 1 (Fast Detection):** Dùng **Kinesis Data Stream** tiếp nhận dữ liệu $\rightarrow$ **Lambda Feature Engineering** $\rightarrow$ Model **XGBoost** phân loại nhanh giao dịch bất thường (rẻ, lọc 90–95% giao dịch).
  - **Layer 2 (Deep Investigation – Agent Core):** Dùng **AWS Step Functions** điều phối 3 Sub-agent (**KYC Check, Money Flow Check, Sanction Check**) kết hợp **OpenSearch Vector Database (RAG)** để trích xuất căn cứ pháp lý và tổng hợp thành tệp bằng chứng (Evidence File).
  - **Layer 3 (Decision & Human in the Loop):** Sử dụng **2 mô hình LLM kiểm tra chéo lẫn nhau** (Double-check) kết hợp **Bedrock Guardrails** để chống hiện tượng ảo giác, tự động xử lý các case an toàn và chỉ đẩy các trường hợp nghi ngờ (Escalate) lên Dashboard cho chuyên viên ra quyết định.

## Thảo Luận So Sánh

Dựa trên các giải pháp được trình bày tại sự kiện, dưới đây là bảng đối chiếu chi tiết giữa mô hình truyền thống và kiến trúc Agentic AI trên AWS:

| Tiêu chí | Mô hình Truyền thống / Thủ công | Kiến trúc Agentic AI trên AWS |
| :--- | :--- | :--- |
| **Đặt hàng F&B (Drive-through)** | Nhân viên thủ công hoặc AI cơ bản hay gặp lỗi ngữ cảnh, hallucination. | Agent tích hợp trực tiếp Zalo/WhatsApp + Bedrock Session Memory; chi phí $0.006/đơn, tiết kiệm 75%. |
| **Phân tích đối thủ cạnh tranh** | Đội ngũ nghiên cứu thủ công, tốc độ chậm, chi phí cao ($94+/tháng). | Multi-Agent A2A (Supervisor $\rightarrow$ Crawler $\rightarrow$ Analysis) với Validation Loop + Langfuse; giảm còn $35/tháng với Native AWS Tools. |
| **Thiết kế kiến trúc & báo giá** | SA mất nhiều giờ vẽ tay, tính toán bằng bảng tính (spreadsheet). | AI Native App (Plan V Team): tự động vẽ Draw.io $\rightarrow$ Cost Estimation $\rightarrow$ gen Terraform $\rightarrow$ deploy; tiết kiệm thời gian đáng kể. |
| **Giám sát đám đông** | Camera quan sát thụ động, con người xem lại sau khi ùn tắc xảy ra. | Kinesis Video + YOLO + ByteTrack + Bedrock Agent (S.H.E.P.H.E.R.D.): phân tích theo thời gian thực, đề xuất điều phối chủ động. |
| **Phát hiện rửa tiền (AML)** | 90–95% False Positive, tốn 3 giờ/case, $20–$25/lần rà soát thủ công. | 3-Layer (XGBoost $\rightarrow$ Step Functions/RAG $\rightarrow$ Double LLM + Guardrails); chỉ escalate case nghi ngờ thực sự. |

## Những Gì Học Được

### Tư Duy Thiết Kế

- **Tư duy hướng nghiệp vụ (Business-first & Pain-point Driven):** Công nghệ dù phức tạp đến đâu cũng không thể vượt qua giới hạn nghiệp vụ. Sản phẩm cần tập trung giải quyết nỗi đau cụ thể của người dùng/doanh nghiệp thay vì chỉ làm các ứng dụng đại trà (To-do list, CRUD basic).
- **Kiểm soát phạm vi dự án (Scope Management):** Trong điều kiện áp lực thời gian (như 24–48h thi Hackathon), cần xác định rõ In-scope và Out-of-scope, ưu tiên phát triển **Sản phẩm khả thi tối thiểu (MVP)** hoạt động ổn định thay vì mở rộng tính năng vô bờ bến dẫn đến lỗi hệ thống.
- **Thiết kế tối ưu chi phí & Tự chủ (Cost & Compliance Optimization):** Luôn đo lường chi phí vận hành hạ tầng (Cloud pricing model); ưu tiên các giải pháp cloud-native để giảm phụ thuộc vào dịch vụ bên thứ ba và đáp ứng tiêu chuẩn lưu trữ dữ liệu nội bộ.

### Kiến Trúc Kỹ Thuật

- **Mô hình kiến trúc Agent đa tầng (Multi-Agent Systems & A2A):** Cách thức áp dụng mô hình Orchestrator/Supervisor điều phối các Sub-agent chuyên biệt (Crawler, KYC, Analysis) để xử lý luồng công việc phức tạp.
- **Cơ chế chống ảo giác và Kiểm soát chất lượng dữ liệu:** Kết hợp 2 LLM chéo, sử dụng Bedrock Guardrails, áp dụng vòng lặp kiểm tra điểm (Validation Loop) và giữ lại con người ở bước cuối cùng (Human-in-the-loop) đối với các nghiệp vụ tài chính/pháp lý rủi ro cao.
- **Xử lý chuỗi dữ liệu thời gian thực (Real-time Streaming Pipeline):** Kết hợp các dịch vụ Kinesis Video/Data Streams với mô hình AI Vision (YOLO) và mô hình học máy bảng (XGBoost) để xử lý lượng truy vấn lớn không gây tắc nghẽn.

### Chiến Lược Phát Triển Cá Nhân

- **Học qua thực chiến (Hands-on Experience):** Việc học lý thuyết cần đi đôi với trải nghiệm áp lực thực tế (như làm việc liên tục 24h) để nâng cao khả năng quản trị tâm lý và giải quyết sự cố.
- **Tầm quan trọng của tinh thần đồng đội (Teamwork & Low Ego):** Lắng nghe ý kiến đồng đội, hạ gạt cái tôi cá nhân, phân chia vai trò rõ ràng theo đúng thế mạnh (Frontend, Backend, AI, Business/Pitching) là chìa khóa đưa dự án đến thành công.

## Ứng Dụng Vào Công Việc

- **Tích hợp Agentic Workflow vào dự án cá nhân/đồ án:** Áp dụng mô hình Supervisor – Sub-agent cho các bài tập lớn quản lý hoặc hệ thống thương mại điện tử để xử lý các tác vụ tự động hóa thay cho các hàm IF-ELSE truyền thống.
- **Tối ưu hóa quy trình kiểm thử và CI/CD:** Áp dụng tư duy quản lý biến môi trường (`.env`) và quản lý Git chỉn chu, tránh để lộ bí mật/API keys lên các kho lưu trữ mã nguồn.
- **Thực hành thiết kế hạ tầng bằng IaC:** Sử dụng các công cụ như **Terraform** để quản lý hạ tầng đám mây dưới dạng mã nguồn, giúp tái sử dụng và triển khai nhanh chóng.
- **Chuẩn bị kỹ lưỡng cho kịch bản Demo:** Đặt mục tiêu tạo sản phẩm có thể chạy thực tế (Production-ready hoặc MVP trực quan), chuẩn bị các phương án dự phòng khi đường truyền mạng hoặc token AI gặp sự cố.

## Trải nghiệm trong event

- **Môi trường workshop x Hackathon chia sẻ kết quả:** Mang lại không khí sôi nổi, cởi mở và giàu tính thực chiến.
- **Không khí trao đổi không rào cản:** Diễn giả và các đội thi thoải mái chia sẻ cả những khoảnh khắc làm việc căng thẳng, những sai lầm kỹ thuật thực tế (lỡ push file `.env` lên GitHub, hệ thống lag do mạng, chi phí SageMaker tăng cao do demo) giúp các bạn trẻ nhận ra bài học một cách tự nhiên.
- **Sự kết nối đa dạng (Networking):** Sự kiện kết nối sinh viên và kỹ sư đến từ nhiều trường đại học (FPT, HUFLIT...), nhiều ngành nghề (AI, Security, Software, Business) cùng ngồi lại giải quyết bài toán chung.
- **Truyền cảm hứng vượt qua giới hạn:** Nhìn thấy các sản phẩm hoàn chỉnh được xây dựng chỉ trong vòng 24–48 giờ tiếp thêm tự tin cho người tham dự đăng ký các cuộc thi công nghệ tương lai.

## Bài học rút ra

- **Công nghệ chỉ là phương tiện, giá trị thực tiễn mới là mục tiêu:** Một kiến trúc dù phức tạp đến đâu cũng không có giá trị nếu không giải quyết đúng điểm đau (pain-point) của người dùng hoặc doanh nghiệp.
- **Thất bại trong kiểm thử là cơ hội học hỏi:** Việc đối mặt với áp lực thời gian, lỗi hạ tầng hay cạn kiệt tài nguyên token dạy cho kỹ sư khả năng ứng ứng biến linh hoạt và tính kiên định.
- **Sức mạnh của sự hợp lực (Collaborative Growth):** *"Muốn đi nhanh hãy đi một mình, muốn xây dựng sản phẩm lớn trong kỷ nguyên AI hãy đi cùng đồng đội với sự phân vai rõ ràng và tinh thần học hỏi không ngừng."*

## Một số hình ảnh khi tham gia sự kiện

![event4](/images/event4.jpg)
