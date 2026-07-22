---
title: "Sự kiện 2"
date: 2026-07-04
weight: 2
chapter: false
pre: " <b> 4.2. </b> "
---

# BÀI THU HOẠCH: “ENTERPRISE CLOUD ARCHITECTURES AND INDUSTRY APPLICATION” STUDY TOUR

## Mục Đích Của Sự Kiện

- **Mục đích:** Giới thiệu góc nhìn thực tế về điện toán đám mây và kiến trúc hệ thống trong môi trường doanh nghiệp; phân tích tổng quan thị trường việc làm, xu hướng công nghệ (Cloud, AI, K8s) và các khoảng cách kỹ năng (skill gaps); chia sẻ phương pháp tư duy thiết kế hệ thống dữ liệu (Data Engineering) qua mô hình thực tế; đồng thời trao đổi về kỹ năng mềm, định vị bản thân và cách thức nắm bắt cơ hội nghề nghiệp trong kỷ nguyên AI.

## Danh Sách Diễn Giả

- **Danh sách diễn giả:**
  - **Chị Quỳnh Mai:** Host/MC.
  - **Anh Nguyễn Trần Minh Duy:** Industry Liaison Officer tại Swinburne Việt Nam.
  - **Anh Nguyễn Gia Hưng:** Head of Solution Architect tại AWS Việt Nam, Founder AWS First Cloud AI Journey.
  - **Anh Bành Cẩm Vĩnh:** Data Engineer tại Renova Cloud, AWS Community Builder.
  - **Chị Như Trần:** Account Manager tại AWS Việt Nam.
  - **Anh Khang Nguyễn:** Solution Architect tại Cloud Kinetics.

## Nội Dung Nổi Bật

### Tổng quan thị trường Cloud và nhân lực kỹ thuật tại Việt Nam

- **Tốc độ tăng trưởng vượt bậc:** Doanh thu của đám mây (Cloud) tăng trưởng gần 20 lần trong 6 năm qua, trong khi quy mô phần cứng truyền thống tại các trung tâm dữ liệu giảm 10 lần. Xu hướng dịch chuyển là tất yếu với chiến lược "Cloud first" từ các doanh nghiệp lớn (tài chính, ngân hàng, bảo hiểm...).
- **Thách thức tuyển dụng đối với Early Talent:** Thị trường hiện tại yêu cầu rất cao ngay cả ở vị trí thực tập sinh (Intern). Doanh nghiệp không còn ưu tiên mô hình "cầm tay chỉ việc" cho nhân sự chưa có kỹ năng, đòi hỏi ứng viên Intern/Fresher phải nắm vững các công nghệ hiện đại như Kubernetes (K8s) hay Cloud Native.
- **Đầu tư dài hạn của AWS tại Việt Nam:** Tập trung vào 3 trụ cột chính:
  - *Local Talent:* Đội ngũ kỹ sư bản địa.
  - *Infrastructure Investment:* Xây dựng hệ thống CDN, Caching tại TP. Hồ Chí Minh và Hà Nội, phát triển Local Zone và hướng tới AWS Region quy mô lớn.
  - *Future Talent:* Xây dựng cộng đồng kỹ sư tương lai qua các dự án như First Cloud AI Journey.
- **6 nhóm ngành công nghiệp lớn ứng dụng Cloud:** Fintech & Banking, Retail, Manufacturing, Logistics, Media & Entertainment, và Healthcare. Hồ sơ ứng viên cần ánh xạ (map) năng lực kỹ thuật vào việc giải quyết bài toán đặc thù của một trong các nhóm ngành này để tăng tính cạnh tranh.
- **Thị trường tuyển dụng ẩn (Hidden Job Market):** Khoảng 90% cơ hội việc làm Cloud chất lượng không được đăng tuyển công khai (public) mà thông qua mạng lưới nội bộ hoặc giới thiệu (referral). Tham gia cộng đồng (community) là yếu tố quyết định để tăng sự hiện diện (visibility).

### Phân tích chuyên sâu về Data Engineering (Học đường vs Thực tế)

- **5 môn học nền tảng thiết yếu:** Cơ sở dữ liệu, Ngôn ngữ lập trình, Cấu trúc dữ liệu và giải thuật, Cơ sở dữ liệu phân tán, và API. Đây là gốc rễ luôn được kiểm tra kỹ khi phỏng vấn.
- **Mô hình DNA của một Data Platform:** Kiến trúc lõi không thay đổi qua các doanh nghiệp bao gồm các thành phần: Ingestion (Thu thập) → Processing (Xử lý) → Storage (Lưu trữ) → Data Governance (Quản trị) → Catalog (Danh mục) → Analytics (Phân tích). Công cụ kỹ thuật (tools) có thể thay đổi liên tục nhưng tư duy kiến trúc luôn giữ nguyên.
- **Bề nổi và Bề chìm:** Các dự án cá nhân/bài tập thường chỉ xử lý phần bề nổi (đọc, transform, load dữ liệu, lên dashboard). Doanh nghiệp cần 90% phần chìm bên dưới: xử lý khi hệ thống fail (vận hành bất đồng bộ), đảm bảo chất lượng dữ liệu (Data Quality), bảo mật thông tin nhạy cảm.

### Thảo Luận So Sánh

Dưới đây là bảng đối chiếu chi tiết sự khác biệt giữa môi trường học tập (Bài tập lớn/Side Project) và môi trường Production thực tế tại doanh nghiệp:

| Tiêu chí | Môi trường học tập (Bài tập lớn / Side Project) | Môi trường Production doanh nghiệp |
| :--- | :--- | :--- |
| **Tính chất dữ liệu** | Dữ liệu sạch, dung lượng nhỏ, yêu cầu rõ ràng. | Dữ liệu từ nhiều nguồn khác nhau, thiếu nhất quán, thay đổi liên tục. |
| **Yêu cầu & Nghiệp vụ** | Cố định, ít thay đổi trong suốt môn học. | Yêu cầu nghiệp vụ (business) thay đổi liên tục hàng ngày. |
| **Thời hạn (Timeline)** | Timeline thoải mái, có hạn nộp cố định. | Timeline gấp, đòi hỏi tốc độ xử lý nhanh chóng. |
| **Hậu quả thất bại** | Chỉ mất điểm số của môn học. | Ảnh hưởng trực tiếp đến vận hành kinh doanh và doanh thu của doanh nghiệp. |
| **Phạm vi xử lý** | Tập trung phần bề nổi (đọc, transform, load dữ liệu, dựng dashboard). | Tập trung 90% phần chìm (xử lý khi hệ thống fail, chạy bất đồng bộ, Data Quality, bảo mật thông tin nhạy cảm). |

### Vai trò và vị trí của AI trong quy trình làm việc

- **Mô hình kim tự tháp nhân lực đảo ngược:** Nhu cầu tuyển dụng Junior theo cách truyền thống giảm mạnh do tầng lớp trung gian này đang dần được thay thế bởi các AI Agent/AI Tool. Ngược lại, nhu cầu tuyển dụng Senior có năng lực kết hợp AI để tối ưu hóa hiệu suất lại tăng lên.
- **Nguyên lý khuếch đại (Amplify) của AI:** AI đóng vai trò như một đòn bẩy hiệu suất. Nếu nền tảng kỹ thuật tốt, AI giúp hoàn thiện nhiều dự án nhanh chóng. Nếu không nắm vững bản chất hệ thống, ứng viên dễ bị AI dẫn dắt sai lệch vì bản chất của AI là mô hình xác xuất, luôn có tỷ lệ lỗi.
- **Tư duy cốt lõi của con người mà AI không thể thay thế:** Thấu hiểu bài toán kinh doanh (Business Understanding), thiết kế kiến trúc hệ thống (Architecture Thinking), giao tiếp liên phòng ban (Communication), và ra quyết định dựa trên sự đánh đổi (Decision Making & Trade-offs).

### Kỹ năng định vị bản thân và phát triển sự nghiệp

- **Công thức thành công:**
  > **Kết quả = Capability (Năng lực) × Visibility (Sự hiện diện) × Consistency (Sự kiên trì)**

  Thiếu bất kỳ yếu tố nào trong ba yếu tố trên thì kết quả đều bằng không.
- **Chiến lược Đại dương xanh (Blue Ocean):** Thay vì cạnh tranh khốc liệt tại các thị trường tuyển dụng phổ thông đại trà (Red Ocean), ứng viên cần chủ động tìm kiếm các thị trường ngách, sẵn sàng làm những việc khó, việc ít người muốn làm để tạo dấu ấn riêng.
- **Mô hình 3 vòng tròn chọn việc:** Sự giao thoa giữa Đam mẻ (Passion), Trách nhiệm (Responsibility), và Lợi ích mang lại (Benefit - bao gồm lương, kinh nghiệm, mạng lưới mối quan hệ, kiến thức và sự phát triển cá nhân).
- **5 tiêu chí đánh giá ứng viên của nhà tuyển dụng:** Thái độ (Attitude - quan trọng nhất đối với Fresher), Trình độ (Competence), Kinh nghiệm (Experience - số năm làm việc), Trải nghiệm (Exposure - số lượng hệ thống, dự án, khách hàng đã va chạm), và Tố chất (Talent).

## Những Bài Học Rút Ra

### Tư Duy Thiết Kế

- **Tư duy hướng kết quả (Outcome-oriented):** Luôn bắt đầu từ mục tiêu cuối cùng và bài toán thực tế của doanh nghiệp (Business-first approach) trước khi lựa chọn công cụ hay công nghệ.
- **Tư duy kiến trúc thay vì tư duy công cụ:** Tập trung nắm chắc nền tảng core (ví dụ mô hình DNA của dữ liệu, cấu trúc mạng networking). Công cụ (tools/services) của các hãng công nghệ chỉ là phương tiện thực thi, tư duy hệ thống mới là giá trị cốt lõi bền vững.
- **Chấp nhận sự đánh đổi (Trade-offs):** Mọi quyết định kỹ thuật hay kiến trúc đều có chi phí và hệ quả đi kèm. Không có giải pháp hoàn hảo tuyệt đối, chỉ có giải pháp phù hợp nhất với giới hạn tài chính, thời gian và công nghệ của doanh nghiệp tại thời điểm đó.

### Kiến Trúc Kỹ Thuật

- **Phương pháp tối ưu hóa thiết kế bằng Framework:** Sử dụng các bộ khung chuẩn hóa như AWS Well-Architected Framework để tự đánh giá, rà soát và tối ưu các dự án cá nhân theo các tiêu chuẩn vận hành xuất sắc, bảo mật, độ tin cậy và tối ưu chi phí.
- **Nâng cao tiêu chuẩn Production cho dự án:** Dự án không dừng lại ở mức "chạy được" (giống như bài tập trường học), mà phải tính toán đến các kịch bản thực tế: Scalable (khả năng mở rộng khi lượng truy cập tăng), Security (bảo mật lớp dữ liệu), Data Quality (kiểm soát lỗi dữ liệu nguồn), và giải pháp phục hồi khi hệ thống sụp đổ (Failover/Resilience).
- **Ứng dụng AI thông minh:** Không dùng AI để làm thay hay "outsource" tư duy lập trình (việc chép code làm bài tập mà không hiểu bản chất dẫn đến rỗng kiến thức). Sử dụng AI làm công cụ cố vấn phản biện (Review/Scan code), tìm kiếm lỗi kiến trúc, và tối ưu hóa workflow phát triển phần mềm nhằm rút ngắn thời gian hoàn thiện sản phẩm.

### Chiến Lược Hiện Đại Hóa & Phát Triển Cá Nhân

- **Học tập suốt đời (Lifelong Learning):** Nhà tuyển dụng không chỉ đánh giá năng lực hiện tại mà đánh giá tiềm năng phát triển tương lai. Hồ sơ (CV) cần thể hiện tần suất cập nhật kiến thức, chứng chỉ và các dự án cá nhân một cách liên tục, đều đặn.
- **Chủ động tăng cường Visibility:** Gỡ bỏ rào cản "hướng nội", tích cực tham gia các hoạt động cộng đồng, viết blog chia sẻ kỹ thuật, chủ động đặt câu hỏi phản biện để mạng lưới ngành ghi nhận năng lực thực tế.
- **Xây dựng tính kiên định (Resilience & Consistency):** Sẵn sàng đối mặt với áp lực sai lầm khi còn ngồi trên ghế nhà trường ("Học là trả tiền để được sai, đi làm là được trả tiền để không làm sai"). Kiên trì theo đuổi mục tiêu nghề nghiệp dài hạn, không nản lòng trước các đợt lọc hồ sơ hay phỏng vấn thất bại.

## Ứng Dụng Vào Công Việc

- **Rà soát lại toàn bộ Side Projects hiện tại:** Sử dụng các công cụ AI và tài liệu kỹ thuật chính thống (Official Documentation) để quét, đánh giá cấu trúc code và kiến trúc các dự án e-commerce hoặc sudoku đang xây dựng, bổ sung phần xử lý lỗi (exception handling) và tối ưu truy vấn dữ liệu.
- **Chuẩn hóa hồ sơ năng lực (Profile) cá nhân:** Xây dựng blog kỹ thuật cá nhân để tài liệu hóa (document) quá trình tư duy, các bài học kinh nghiệm từ lỗi lập trình; cập nhật các mã nguồn dự án lên nền tảng public (GitHub) một cách chỉn chu để vượt qua các vòng quét CV tự động bằng AI của doanh nghiệp.
- **Thực hành kỹ năng giao tiếp và Small Talk:** Áp dụng tư duy thấu hiểu đối phương (sếp/giảng viên/đối tác) trong các tình huống thực tế. Tập luyện chủ động kết nối, đặt các câu hỏi mở, câu hỏi đào sâu bản chất (Why) trong các buổi làm việc nhóm bài tập lớn (BTL) hoặc khi tương tác với các giảng viên trong trường để nâng cao năng lực truyền tải thông tin.
- **Định hướng học tập bám sát Industry:** Khi phát triển bất kỳ hệ thống hay ứng dụng nào (chẳng hạn hệ thống quản lý nhà máy sấy DryTech hay e-commerce), cần nghiên cứu kỹ bài toán nghiệp vụ, luồng quy trình vận hành (workflow) và các điểm nghẽn của ngành công nghiệp tương ứng để thiết kế tính năng giải quyết chính xác bài toán đó, thay vì chỉ xây dựng các tính năng chung chung.

## Trải nghiệm trong event

- **Trải nghiệm thực tế:** Tham gia buổi Study Tour chia sẻ về kiến trúc đám mây và ứng dụng thực tế trong doanh nghiệp mang lại một góc nhìn thực tiễn, sâu sắc và phá vỡ nhiều định kiến cũ về ngành công nghiệp phần mềm. Một số trải nghiệm nổi bật bao gồm:
  - *Tiếp cận tri thức thực chiến từ chuyên gia đầu ngành:* Được lắng nghe các bài học xương máu trong quá trình dịch chuyển hệ thống và xử lý dữ liệu qua các thời kỳ công nghệ từ các kỹ sư, quản lý cấp cao trực tiếp làm việc tại AWS, Renova Cloud và Cloud Kinetics. Đồng thời hiểu được áp lực và yêu cầu thực tế khốc liệt của thị trường việc làm hiện tại thông qua các case study tuyển dụng thực tế.
  - *Sự thay đổi lớn trong tư duy học tập và làm việc:* Trải nghiệm các phiên thảo luận cởi mở giúp nhận ra tầm quan trọng của việc học đi đôi với hiểu bản chất, thay vì học đối phó để qua môn. Việc phân tích lộ trình chuyển đổi từ một kỹ sư dữ liệu (Data Engineer) sang mảng học máy (Machine Learning) của diễn giả giúp mở rộng thế giới quan về khả năng thích ứng và tư duy mở (Growth Mindset) trong môi trường công nghệ biến động liên tục.
  - *Bài học về kỹ năng kết nối và quản trị nỗi sợ:* Phiên chia sẻ của Account Manager AWS về việc gỡ bỏ áp lực đồng trang lứa và đối diện với nỗi sợ (sợ sai, sợ nói trước đám đông, sợ làm cha mẹ thất vọng) bằng cách chuyển hóa thành hành động cụ thể mang lại giá trị tinh thần lớn. Nhận thức rõ ràng về việc xây dựng mạng lưới quan hệ (Networking) chất lượng thông qua các hoạt động tương tác trực tiếp, small talk ngắn trong thang máy hay môi trường công sở, từ đó mở khóa các cơ hội trong "thị trường việc làm ẩn".
- **Bài học rút ra:**
  - **Năng lực kết hợp Sự hiện diện & Kiên trì:** Năng lực chuyên môn vững chắc (Capability) chỉ phát huy tối đa giá trị khi được đặt trong một lộ trình có sự hiện diện rõ ràng (Visibility) và nỗ lực kiên trì đều đặn (Consistency).
  - **Làm chủ AI:** AI không trực tiếp cướp đi công việc của kỹ sư, nhưng những nhân sự biết làm chủ AI, hiểu sâu bản chất hệ thống và có năng lực tư duy kiến trúc/kinh doanh sẽ thay thế những người làm việc rập khuôn.
  - **Thái độ và Sự gieo mầm cơ hội:** Cơ hội nghề nghiệp chất lượng cao không tự nhiên đến qua các trang tuyển dụng đại trà mà được gieo mầm từ thái độ làm việc trách nhiệm, sự kiên định vượt qua sai lầm, và khả năng xây dựng lòng tin trong các mối quan hệ chuyên môn.

## Một số hình ảnh khi tham gia sự kiện

![event2](/images/event2.jpg)