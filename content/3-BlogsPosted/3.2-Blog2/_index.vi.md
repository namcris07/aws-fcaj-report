---
title: "Blog 2"
date: 2021-06-25
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# Xây dựng nhà máy phần mềm DevSecOps end-to-end dựa trên Kubernetes trên AWS

**bởi Srinivas Manepalli vào ngày 25 THÁNG 6 2021 trong [Architecture](https://aws.amazon.com/blogs/devops/category/architecture/), [AWS CodeBuild](https://aws.amazon.com/blogs/devops/category/developer-tools/aws-codebuild/), [AWS CodeDeploy](https://aws.amazon.com/blogs/devops/category/developer-tools/aws-codedeploy/), [AWS CodePipeline](https://aws.amazon.com/blogs/devops/category/developer-tools/aws-codepipeline/), [AWS Security Hub](https://aws.amazon.com/blogs/devops/category/security-identity-compliance/aws-security-hub/), [Compliance](https://aws.amazon.com/blogs/devops/category/compliance/), [DevOps](https://aws.amazon.com/blogs/devops/category/devops/), Public Sector, Security**

---

Việc triển khai nhà máy phần mềm DevSecOps có thể khác biệt đáng kể tùy thuộc vào ứng dụng, hạ tầng, kiến trúc, cũng như các dịch vụ và công cụ được sử dụng. Trong một bài viết trước, tôi đã cung cấp một pipeline DevSecOps end-to-end cho ứng dụng web 3 lớp được triển khai bằng [AWS Elastic Beanstalk](https://aws.amazon.com/elasticbeanstalk/). Pipeline đó sử dụng các dịch vụ cloud-native cùng với một số công cụ bảo mật mã nguồn mở. Giải pháp này tương tự, nhưng thay vào đó sử dụng phương pháp dựa trên container với các giai đoạn phân tích bảo mật bổ sung. Giải pháp định nghĩa một nhà máy phần mềm bằng cách sử dụng Kubernetes cùng với các dịch vụ AWS Cloud-native cần thiết và các công cụ mã nguồn mở của bên thứ ba. Mã nguồn được cung cấp trong [GitHub repo](https://github.com/aws-samples/aws-devsecops-workshop) để xây dựng nhà máy phần mềm DevSecOps này, bao gồm cả mã tích hợp cho các công cụ quét của bên thứ ba.

DevOps là sự kết hợp giữa các triết lý văn hóa, thực hành và công cụ nhằm kết hợp việc phát triển phần mềm với vận hành công nghệ thông tin. Các thực hành kết hợp này cho phép các công ty cung cấp các tính năng ứng dụng mới và dịch vụ cải tiến đến khách hàng với tốc độ cao hơn. DevSecOps tiến thêm một bước bằng cách tích hợp và tự động hóa việc thực thi các kiểm soát bảo mật phòng ngừa (preventive), phát hiện (detective) và phản ứng (responsive) vào trong pipeline.

Trong một nhà máy DevSecOps, bảo mật cần được giải quyết từ hai khía cạnh: bảo mật CỦA nhà máy phần mềm (security of the software factory), và bảo mật TRONG nhà máy phần mềm (security in the software factory). Trong kiến trúc này, chúng tôi sử dụng các dịch vụ AWS để giải quyết bảo mật của nhà máy phần mềm, và sử dụng các công cụ của bên thứ ba cùng với các dịch vụ AWS để giải quyết bảo mật trong nhà máy phần mềm. Kiến trúc tham chiếu AWS DevSecOps này bao gồm các thực hành DevSecOps và các giai đoạn quét lỗ hổng bảo mật bao gồm phân tích bí mật (secret analysis), SCA (Software Composite Analysis - Phân tích thành phần phần mềm), SAST (Static Application Security Testing - Kiểm thử bảo mật ứng dụng tĩnh), DAST (Dynamic Application Security Testing - Kiểm thử bảo mật ứng dụng động), RASP (Runtime Application Self Protection - Tự bảo vệ ứng dụng khi thực thi), và tổng hợp các phát hiện lỗ hổng vào một giao diện quản lý tập trung duy nhất (single pane of glass).

Trọng tâm của bài viết này là quét lỗ hổng bảo mật ứng dụng. Việc quét lỗ hổng của hạ tầng bên dưới như cụm [Amazon Elastic Kubernetes Service (Amazon EKS)](https://aws.amazon.com/eks/) và mạng nằm ngoài phạm vi của bài viết này. Để biết thông tin về lập kế hoạch bảo mật cấp hạ tầng, hãy tham khảo [Amazon GuardDuty](https://aws.amazon.com/guardduty/), [Amazon Inspector](https://aws.amazon.com/inspector/), và [AWS Shield](https://aws.amazon.com/shield/).

Bạn có thể triển khai pipeline này trong Region AWS GovCloud (US) hoặc các Region AWS tiêu chuẩn. Tất cả các dịch vụ AWS được liệt kê đều đạt ủy quyền cho FedRamp High và DoD SRG IL4/IL5.

## Bảo mật và tuân thủ (Security and compliance)

Việc thực hiện triệt để bảo mật và tuân thủ trong khu vực công (public sector) và các khối lượng công việc được quản lý nghiêm ngặt khác là rất quan trọng để đạt được ATO (Authority to Operate) và duy trì liên tục ATO (c-ATO). DevSecOps dịch chuyển bảo mật về phía trước (shift left) trong quy trình, tích hợp bảo mật vào từng giai đoạn của nhà máy phần mềm, điều này có thể làm cho ATO trở thành một quy trình liên tục và nhanh chóng hơn. Với DevSecOps, một tổ chức có thể phân phối các thay đổi ứng dụng an toàn và tuân thủ nhanh chóng trong khi vận hành nhất quán nhờ tự động hóa.

Bảo mật và tuân thủ là trách nhiệm chia sẻ giữa AWS và khách hàng. Tùy thuộc vào yêu cầu tuân thủ (như FedRamp hoặc DoD SRG), nhà máy phần mềm DevSecOps cần triển khai các kiểm soát bảo mật nhất định. AWS cung cấp các công cụ và dịch vụ để triển khai hầu hết các kiểm soát này. Ví dụ, để giải quyết các nhóm kiểm soát bảo mật NIST 800-53 như kiểm soát truy cập (access control), bạn có thể sử dụng các role [AWS Identity Access and Management (IAM)](https://aws.amazon.com/iam/) và các policy cho bucket [Amazon Simple Storage Service (Amazon S3)](https://aws.amazon.com/s3/). Để giải quyết việc kiểm toán và trách nhiệm giải trình (auditing and accountability), bạn có thể sử dụng [AWS CloudTrail](https://aws.amazon.com/cloudtrail/) và [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/). Để giải quyết quản lý cấu hình, bạn có thể sử dụng các rule [AWS Config](https://aws.amazon.com/config/) và [AWS Systems Manager](https://aws.amazon.com/systems-manager/). Tương tự, để giải quyết đánh giá rủi ro, bạn có thể sử dụng các công cụ quét lỗ hổng từ AWS.

Bảng sau đây là ánh xạ cấp cao của các nhóm kiểm soát bảo mật NIST 800-53 và các dịch vụ AWS được sử dụng trong kiến trúc tham chiếu DevSecOps này. Danh sách này chỉ bao gồm các dịch vụ được định nghĩa trong template [AWS CloudFormation](https://aws.amazon.com/cloudformation/), vốn cung cấp pipeline dưới dạng mã (pipeline as code) trong giải pháp này. Bạn có thể sử dụng các dịch vụ và công cụ AWS bổ sung hoặc các dịch vụ và công cụ đặc thù theo môi trường khác để giải quyết các nhóm kiểm soát bảo mật này và các nhóm kiểm soát còn lại ở mức độ chi tiết hơn.

| # | Nhóm kiểm soát bảo mật NIST 800-53 – Rev 5 | Dịch vụ AWS được sử dụng (Trong Pipeline DevSecOps này) |
|---|---|---|
| 1 | AC – Access Control (Kiểm soát truy cập) | Sử dụng AWS IAM, Amazon S3, và Amazon CloudWatch. `AWS::IAM::ManagedPolicy` `AWS::IAM::Role` `AWS::S3::BucketPolicy` `AWS::CloudWatch::Alarm` |
| 2 | AU – Audit and Accountability (Kiểm toán và Trách nhiệm giải trình) | Sử dụng AWS CloudTrail, Amazon S3, Amazon SNS, và Amazon CloudWatch. `AWS::CloudTrail::Trail` `AWS::Events::Rule` `AWS::CloudWatch::LogGroup` `AWS::CloudWatch::Alarm` `AWS::SNS::Topic` |
| 3 | CM – Configuration Management (Quản lý cấu hình) | Sử dụng AWS Systems Manager, Amazon S3, và AWS Config. `AWS::SSM::Parameter` `AWS::S3::Bucket` `AWS::Config::ConfigRule` |
| 4 | CP – Contingency Planning (Lập kế hoạch dự phòng) | Sử dụng AWS CodeCommit và Amazon S3. `AWS::CodeCommit::Repository` `AWS::S3::Bucket` |
| 5 | IA – Identification and Authentication (Định danh và Xác thực) | Sử dụng AWS IAM. `AWS:IAM:User` `AWS::IAM::Role` |
| 6 | RA – Risk Assessment (Đánh giá rủi ro) | Sử dụng AWS Config, AWS CloudTrail, AWS Security Hub, và các công cụ quét của bên thứ ba. `AWS::Config::ConfigRule` `AWS::CloudTrail::Trail` `AWS::SecurityHub::Hub` Các công cụ quét lỗ hổng |
| 7 | CA – Assessment, Authorization, and Monitoring (Đánh giá, Cấp phép và Giám sát) | Sử dụng AWS CloudTrail, Amazon CloudWatch, và AWS Config. `AWS::CloudTrail::Trail` `AWS::CloudWatch::LogGroup` `AWS::CloudWatch::Alarm` `AWS::Config::ConfigRule` |
| 8 | SC – System and Communications Protection (Bảo vệ Hệ thống và Truyền thông) | Sử dụng AWS KMS và AWS Systems Manager. `AWS::KMS::Key` `AWS::SSM::Parameter` Giao tiếp SSL/TLS |
| 9 | SI – System and Information Integrity (Tính toàn vẹn Hệ thống và Thông tin) | Sử dụng AWS Security Hub, và các công cụ quét của bên thứ ba. `AWS::SecurityHub::Hub` Các công cụ quét lỗ hổng |
| 10 | AT – Awareness and Training | N/A |
| 11 | SA – System and Services Acquisition | N/A |
| 12 | IR – Incident Response (Phản ứng sự cố) | Chưa triển khai, nhưng các dịch vụ như [AWS Lambda](https://aws.amazon.com/lambda/) và [Amazon CloudWatch Events](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/WhatIsCloudWatchEvents.html) có thể được sử dụng. |
| 13–20 | MA, MP, PS, PE, PL, PM, PT, SR | N/A |

## Các dịch vụ và công cụ (Services and tools)

Trong phần này, chúng tôi thảo luận về các dịch vụ AWS khác nhau và các công cụ của bên thứ ba được sử dụng trong giải pháp này.

### Các dịch vụ CI/CD

Đối với tích hợp liên tục và giao hàng liên tục (CI/CD) trong kiến trúc tham chiếu này, chúng tôi sử dụng các dịch vụ AWS sau:

- [**AWS CodeBuild**](https://aws.amazon.com/codebuild/) – Một dịch vụ tích hợp liên tục được quản lý toàn phần giúp biên dịch mã nguồn, chạy kiểm thử và tạo ra các gói phần mềm sẵn sàng triển khai.
- [**AWS CodeCommit**](https://aws.amazon.com/codecommit/) – Dịch vụ quản lý mã nguồn được quản lý toàn phần, lưu trữ các kho chứa dựa trên Git an toàn.
- [**AWS CodeDeploy**](https://aws.amazon.com/codedeploy/) – Dịch vụ triển khai được quản lý toàn phần giúp tự động hóa việc triển khai phần mềm lên nhiều dịch vụ tính toán khác nhau như [Amazon Elastic Compute Cloud (Amazon EC2)](https://aws.amazon.com/ec2/), [AWS Fargate](https://aws.amazon.com/fargate/), [AWS Lambda](https://aws.amazon.com/lambda/), và các máy chủ on-premises của bạn.
- [**AWS CodePipeline**](https://aws.amazon.com/codepipeline/) – Dịch vụ giao hàng liên tục được quản lý toàn phần giúp tự động hóa các pipeline phát hành để cập nhật ứng dụng và hạ tầng nhanh chóng, đáng tin cậy.
- [**AWS Lambda**](https://aws.amazon.com/lambda/) – Dịch vụ cho phép bạn chạy mã mà không cần cấp phát hoặc quản lý máy chủ. Bạn chỉ trả tiền cho thời gian tính toán mà bạn tiêu thụ.
- [**Amazon Simple Notification Service (Amazon SNS)**](https://aws.amazon.com/sns/) – Dịch vụ nhắn tin được quản lý toàn phần cho cả giao tiếp ứng dụng với ứng dụng (A2A) và ứng dụng với người dùng (A2P).
- [**Amazon S3**](https://aws.amazon.com/s3/) – Bộ lưu trữ internet. Bạn có thể sử dụng Amazon S3 để lưu trữ và truy xuất bất kỳ lượng dữ liệu nào vào bất kỳ lúc nào, từ bất kỳ đâu trên web.
- [**AWS Systems Manager Parameter Store**](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html) – Cung cấp bộ lưu trữ an toàn, phân cấp để quản lý dữ liệu cấu hình và quản lý bí mật.

### Các công cụ kiểm thử liên tục (Continuous testing tools)

Sau đây là các công cụ quét mã nguồn mở được tích hợp vào pipeline cho mục đích của bài viết này, nhưng bạn có thể tích hợp các công cụ khác đáp ứng yêu cầu cụ thể của mình. Bạn có thể sử dụng công cụ đánh giá mã tĩnh [Amazon CodeGuru](https://aws.amazon.com/codeguru/) để phân tích tĩnh, nhưng tại thời điểm viết bài này, nó chưa có sẵn trong AWS GovCloud và hiện tại hỗ trợ Java và Python.

- **Anchore (SCA và SAST)** – Anchore Engine là một hệ thống phần mềm mã nguồn mở cung cấp dịch vụ tập trung để phân tích container image, quét các lỗ hổng bảo mật và thực thi các chính sách triển khai.
- [**Amazon Elastic Container Registry image scanning**](https://docs.aws.amazon.com/AmazonECR/latest/userguide/image-scanning.html) – Việc quét image của Amazon ECR giúp xác định các lỗ hổng phần mềm trong container image của bạn. Amazon ECR sử dụng cơ sở dữ liệu Common Vulnerabilities and Exposures (CVEs) từ dự án mã nguồn mở Clair và cung cấp danh sách các phát hiện quét.
- **Git-Secrets (Secrets Scanning)** – Ngăn chặn việc commit thông tin nhạy cảm vào các kho chứa Git. Đây là một công cụ mã nguồn mở từ AWS Labs.
- **OWASP ZAP (DAST)** – Giúp bạn tự động tìm các lỗ hổng bảo mật trong các ứng dụng web khi bạn đang phát triển và kiểm thử ứng dụng.
- **Snyk (SCA và SAST)** – Snyk là một nền tảng bảo mật mã nguồn mở được thiết kế để giúp các doanh nghiệp phát triển phần mềm nâng cao bảo mật cho nhà phát triển.
- **Sysdig Falco (RASP)** – Falco là một dự án bảo mật runtime cloud-native mã nguồn mở giúp phát hiện hành vi ứng dụng bất thường và cảnh báo về các mối đe dọa lúc thực thi. Đây là dự án bảo mật runtime đầu tiên gia nhập CNCF dưới dạng dự án cấp ấp ủ (incubation-level).

Bạn có thể tích hợp các giai đoạn bảo mật bổ sung như IAST (Interactive Application Security Testing - Kiểm thử bảo mật ứng dụng tương tác) vào pipeline để có được thông tin chuyên sâu về mã trong khi ứng dụng đang chạy. Bạn có thể sử dụng các công cụ đối tác của AWS như Contrast Security, Synopsys, và WhiteSource để tích hợp quét IAST vào pipeline. Các công cụ quét mã độc và công cụ ký số cho image cũng có thể được tích hợp vào pipeline để tăng cường bảo mật.

### Dịch vụ ghi log và giám sát liên tục

Các dịch vụ AWS sau đây được sử dụng để ghi log và giám sát liên tục trong kiến trúc tham chiếu này:

- [**Amazon CloudWatch Events**](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/WhatIsCloudWatchEvents.html) – Cung cấp luồng sự kiện hệ thống gần như thời gian thực mô tả những thay đổi trong tài nguyên AWS.
- [**Amazon CloudWatch Logs**](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html) – Cho phép bạn giám sát, lưu trữ và truy cập các file log từ các instance EC2, CloudTrail, Amazon Route 53 và các nguồn khác.

### Dịch vụ kiểm toán và quản trị (Auditing and governance services)

Các dịch vụ kiểm toán và quản trị của AWS sau đây được sử dụng trong kiến trúc tham chiếu này:

- [**AWS CloudTrail**](https://aws.amazon.com/cloudtrail/) – Cho phép quản trị, tuân thủ, kiểm toán vận hành và kiểm toán rủi ro cho tài khoản AWS của bạn.
- [**AWS Config**](https://aws.amazon.com/config/) – Cho phép bạn đánh giá, kiểm toán và phân tích các cấu hình tài nguyên AWS của bạn.
- [**AWS Identity and Access Management**](https://aws.amazon.com/iam/) – Cho phép bạn quản lý quyền truy cập vào các dịch vụ và tài nguyên AWS một cách an toàn. Với IAM, bạn có thể tạo và quản lý người dùng và nhóm AWS, cũng như sử dụng các quyền để cho phép và từ chối truy cập vào tài nguyên AWS.

### Dịch vụ vận hành (Operations services)

Sau đây là các dịch vụ vận hành của AWS được sử dụng trong kiến trúc tham chiếu này:

- [**AWS CloudFormation**](https://aws.amazon.com/cloudformation/) – Cung cấp cho bạn một cách dễ dàng để mô hình hóa một tập hợp các tài nguyên AWS và bên thứ ba có liên quan, cấp phát chúng nhanh chóng và nhất quán, đồng thời quản lý chúng trong suốt vòng đời bằng cách xem hạ tầng như mã (infrastructure as code).
- [**Amazon ECR**](https://aws.amazon.com/ecr/) – Một container registry được quản lý toàn phần giúp dễ dàng lưu trữ, quản lý, chia sẻ và triển khai container image và artifact ở bất kỳ đâu.
- [**Amazon EKS**](https://aws.amazon.com/eks/) – Một dịch vụ được quản lý mà bạn có thể sử dụng để chạy Kubernetes trên AWS mà không cần cài đặt, vận hành và bảo trì control plane hoặc các node Kubernetes của riêng mình. Amazon EKS chạy các phiên bản phần mềm Kubernetes mã nguồn mở cập nhật nhất, vì vậy bạn có thể sử dụng tất cả các plugin và công cụ hiện có từ cộng đồng Kubernetes.
- [**AWS Security Hub**](https://aws.amazon.com/security-hub/) – Cung cấp cho bạn cái nhìn toàn diện về các cảnh báo bảo mật và trạng thái bảo mật trên các tài khoản AWS của bạn. Bài viết này sử dụng Security Hub để tổng hợp tất cả các phát hiện lỗ hổng thành một giao diện quản lý tập trung duy nhất (single pane of glass).
- [**AWS Systems Manager Parameter Store**](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html) – Cung cấp bộ lưu trữ an toàn, phân cấp để quản lý dữ liệu cấu hình và bí mật. Bạn có thể lưu trữ dữ liệu như mật khẩu, chuỗi kết nối cơ sở dữ liệu, ID Amazon Machine Image (AMI) và mã giấy phép dưới dạng các giá trị tham số.

## Kiến trúc Pipeline (Pipeline architecture)

Sơ đồ sau đây hiển thị kiến trúc của giải pháp. Chúng tôi sử dụng [AWS CloudFormation](https://aws.amazon.com/cloudformation/) để mô tả pipeline dưới dạng mã.

![Kiến trúc Pipeline DevSecOps cho Kubernetes](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/06/25/containers-pipeline-architecture-blog-v2-1.png)

*Kiến trúc Pipeline DevSecOps cho Kubernetes*

Các bước chính như sau:

1. Khi người dùng commit mã nguồn vào kho chứa [CodeCommit](https://aws.amazon.com/codecommit/), một sự kiện [CloudWatch](https://aws.amazon.com/cloudwatch/) được tạo ra, kích hoạt [CodePipeline](https://aws.amazon.com/codepipeline/) điều phối các sự kiện.
2. [CodeBuild](https://aws.amazon.com/codebuild/) đóng gói bản build và tải các artifact lên một bucket [S3](https://aws.amazon.com/s3/).
3. CodeBuild quét mã nguồn bằng **git-secrets**. Nếu có bất kỳ thông tin nhạy cảm nào trong mã nguồn như AWS access key hoặc secret key, CodeBuild sẽ làm thất bại bản build.
4. CodeBuild tạo container image và thực hiện SCA và SAST bằng cách quét image với **Snyk** hoặc **Anchore**. Trong template CloudFormation được cung cấp, bạn có thể chọn một trong các công cụ này trong quá trình triển khai. Lưu ý, CodeBuild hoàn toàn hỗ trợ phương pháp "tự mang công cụ của bạn" (bring your own tool).
   - **(4a)** Nếu có bất kỳ lỗ hổng nào, CodeBuild sẽ gọi hàm [Lambda](https://aws.amazon.com/lambda/). Hàm này phân tích kết quả thành Định dạng Phát hiện Bảo mật AWS (ASFF) và gửi chúng đến [Security Hub](https://aws.amazon.com/security-hub/). Security Hub giúp tổng hợp và xem tất cả các phát hiện lỗ hổng ở một nơi như một giao diện tập trung duy nhất. Hàm Lambda cũng tải kết quả quét lên một bucket S3.
   - **(4b)** Nếu không có lỗ hổng nào, CodeBuild đẩy container image lên [Amazon ECR](https://aws.amazon.com/ecr/) và kích hoạt một đợt quét khác bằng tính năng quét tích hợp của Amazon ECR.
5. CodeBuild lấy kết quả quét.
   - **(5a)** Nếu có bất kỳ lỗ hổng nào, CodeBuild lại gọi hàm Lambda và gửi các phát hiện đến Security Hub. Hàm Lambda cũng tải kết quả quét lên một bucket S3.
   - **(5b)** Nếu không có lỗ hổng nào, CodeBuild triển khai container image lên môi trường staging trên [Amazon EKS](https://aws.amazon.com/eks/).
6. Sau khi triển khai thành công, CodeBuild kích hoạt đợt quét DAST với công cụ **OWASP ZAP** (một lần nữa, điều này hoàn toàn hỗ trợ phương pháp "tự mang công cụ của bạn").
   - **(6a)** Nếu có bất kỳ lỗ hổng nào, CodeBuild gọi hàm Lambda, hàm này phân tích kết quả thành ASFF và gửi đến Security Hub. Hàm này cũng tải kết quả quét lên bucket S3 (tương tự bước 4a).
7. Nếu không có lỗ hổng nào, giai đoạn phê duyệt được kích hoạt và một email được gửi đến người phê duyệt qua [Amazon SNS](https://aws.amazon.com/sns/) để thực hiện hành động.
8. Sau khi được phê duyệt, CodeBuild triển khai mã nguồn lên môi trường Amazon EKS production.
9. Trong suốt quá trình pipeline chạy, [CloudWatch Events](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/WhatIsCloudWatchEvents.html) bắt các thay đổi trạng thái build và gửi thông báo email đến những người dùng đã đăng ký qua Amazon SNS.
10. [CloudTrail](https://aws.amazon.com/cloudtrail/) theo dõi các lệnh gọi API và gửi thông báo về các sự kiện quan trọng trên pipeline và các dự án CodeBuild, chẳng hạn như `UpdatePipeline`, `DeletePipeline`, `CreateProject`, và `DeleteProject`, cho mục đích kiểm toán.
11. [AWS Config](https://aws.amazon.com/config/) theo dõi tất cả các thay đổi cấu hình của dịch vụ AWS. Các rule AWS Config sau đây được thêm vào pipeline này như các thực hành bảo mật tốt nhất:
    - `CODEBUILD_PROJECT_ENVVAR_AWSCRED_CHECK` – Kiểm tra xem dự án có chứa các biến môi trường `AWS_ACCESS_KEY_ID` và `AWS_SECRET_ACCESS_KEY` hay không. Rule sẽ ghi nhận NON_COMPLIANT khi các biến môi trường của dự án chứa thông tin xác thực văn bản thuần. Rule này đảm bảo thông tin nhạy cảm không được lưu trữ trong biến môi trường của dự án CodeBuild.
    - `CLOUD_TRAIL_LOG_FILE_VALIDATION_ENABLED` – Kiểm tra xem CloudTrail có tạo file digest được ký cùng với log hay không. AWS khuyến nghị nên bật tính năng xác thực file trên tất cả các trail. Rule sẽ ghi nhận noncompliant nếu xác thực chưa được bật. Rule này đảm bảo các tài nguyên pipeline như dự án CodeBuild không bị thay đổi để bỏ qua các bước kiểm tra lỗ hổng quan trọng.

Bảo mật CỦA pipeline được triển khai bằng cách sử dụng các role [IAM](https://aws.amazon.com/iam/) và policy cho S3 bucket để hạn chế quyền truy cập vào tài nguyên pipeline. Dữ liệu pipeline khi nghỉ (at rest) và khi truyền tải (in transit) được bảo vệ bằng mã hóa và truyền tải an toàn SSL. Chúng tôi sử dụng [Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html) để lưu trữ thông tin nhạy cảm như API token và mật khẩu. Để hoàn toàn tuân thủ các khung như FedRAMP, có thể cần các yếu tố khác như MFA.

Bảo mật TRONG pipeline được triển khai bằng cách thực hiện các bước kiểm tra bảo mật Secret Analysis, SCA, SAST, DAST và RASP. Các dịch vụ AWS áp dụng cung cấp mã hóa khi nghỉ và khi truyền tải theo mặc định. Bạn có thể bật thêm các kiểm soát bổ sung trên đó bất cứ khi nào cần.

Trong phần tiếp theo, tôi sẽ giải thích cách triển khai và chạy template CloudFormation của pipeline được sử dụng cho ví dụ này. Theo thực hành tốt nhất, chúng tôi khuyên bạn nên sử dụng các công cụ linting như `cfn-nag` và `cfn-guard` để quét các template CloudFormation nhằm tìm lỗ hổng bảo mật. Tham khảo các liên kết dịch vụ được cung cấp để tìm hiểu thêm về từng dịch vụ trong pipeline.

## Điều kiện tiên quyết (Prerequisites)

Trước khi bắt đầu, hãy đảm bảo bạn có các điều kiện tiên quyết sau:

- Môi trường [cụm EKS](https://aws.amazon.com/eks/) đã triển khai ứng dụng của bạn. Trong bài viết này, chúng tôi sử dụng PHP WordPress làm ứng dụng mẫu, nhưng bạn có thể sử dụng bất kỳ ứng dụng nào khác.
- [Sysdig Falco](https://falco.org/) đã được cài đặt trên cụm EKS. Sysdig Falco bắt các sự kiện trên cụm EKS và gửi các sự kiện đó đến [CloudWatch](https://aws.amazon.com/cloudwatch/) bằng [AWS FireLens](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/using_firelens.html). Để biết hướng dẫn triển khai, xem [Implementing Runtime security in Amazon EKS using CNCF Falco](https://aws.amazon.com/blogs/containers/implementing-runtime-security-in-amazon-eks-using-cncf-falco/). Bước này chỉ bắt buộc nếu bạn cần triển khai RASP trong nhà máy phần mềm.
- Một kho chứa [CodeCommit](https://aws.amazon.com/codecommit/) với mã ứng dụng của bạn và một Dockerfile. Để biết thêm thông tin, xem [Create an AWS CodeCommit repository](https://docs.aws.amazon.com/codecommit/latest/userguide/how-to-create-repository.html).
- Một kho chứa [Amazon ECR](https://aws.amazon.com/ecr/) để lưu trữ container image và quét lỗ hổng. Bật tính năng quét lỗ hổng khi đẩy image (image push) trong Amazon ECR.
- Các file `buildspec-*.yml` được cung cấp cho git-secrets, Anchore, Snyk, Amazon ECR, OWASP ZAP, và các file `.yml` triển khai Kubernetes của bạn được tải lên thư mục gốc của kho chứa mã ứng dụng.
- Một API key của Snyk nếu bạn sử dụng Snyk làm công cụ SAST.
- Hàm Lambda đã được tải lên một bucket [S3](https://aws.amazon.com/s3/). Chúng tôi sử dụng hàm này để phân tích các báo cáo quét và gửi kết quả đến [Security Hub](https://aws.amazon.com/security-hub/).
- Một URL OWASP ZAP và API key được tạo cho việc quét web động.
- Một URL web ứng dụng để chạy kiểm thử DAST.
- Một địa chỉ email để nhận thông báo phê duyệt triển khai, thông báo thay đổi pipeline, và các sự kiện CloudTrail.
- Các dịch vụ [AWS Config](https://aws.amazon.com/config/) và [Security Hub](https://aws.amazon.com/security-hub/) đã được bật. Để biết hướng dẫn, xem [Managing the Configuration Recorder](https://docs.aws.amazon.com/config/latest/developerguide/stop-start-recorder.html) và [Enabling Security Hub manually](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-enable.html).

## Triển khai pipeline (Deploying the pipeline)

Để triển khai pipeline, hoàn thành các bước sau:

1. Tải xuống template [CloudFormation](https://aws.amazon.com/cloudformation/) và mã pipeline từ [GitHub repo](https://github.com/aws-samples/aws-devsecops-workshop).
2. Đăng nhập vào tài khoản AWS của bạn nếu chưa đăng nhập.
3. Trên [CloudFormation console](https://console.aws.amazon.com/cloudformation/), chọn **Create Stack**.
4. Chọn template pipeline CloudFormation.
5. Chọn **Next**.
6. Dưới phần **Code**, cung cấp các thông tin sau:
   - Chi tiết mã nguồn, như tên kho chứa và branch để kích hoạt pipeline.
   - Tên kho chứa container image trên Amazon ECR.
7. Dưới phần **SAST**, cung cấp các thông tin sau:
   - Chọn công cụ SAST (Anchore hoặc Snyk) để phân tích mã.
   - Nếu chọn Snyk, cung cấp API key cho Snyk.
8. Dưới phần **DAST**, chọn công cụ DAST (OWASP ZAP) cho kiểm thử động và nhập API token, URL công cụ DAST, và URL ứng dụng để chạy đợt quét.
9. Dưới phần **Lambda functions**, nhập tên bucket S3 chứa hàm Lambda, tên file, và tên handler.
10. Đối với **STG EKS cluster**, nhập tên cụm EKS staging.
11. Đối với **PRD EKS cluster**, nhập tên cụm EKS production mà pipeline này sẽ triển khai container image tới.
12. Dưới phần **General**, nhập các địa chỉ email để nhận thông báo về phê duyệt và thay đổi trạng thái pipeline.
13. Chọn **Next**.
14. Hoàn thành stack.
15. Sau khi pipeline được triển khai, xác nhận đăng ký bằng cách chọn liên kết được cung cấp trong email để nhận thông báo.

![Các tham số CloudFormation của Pipeline](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/06/21/Pipeline-CF-Parameters-1.jpg)

*Các tham số CloudFormation của Pipeline*

> **Lưu ý:** Template CloudFormation được cung cấp trong bài viết này được định dạng cho AWS GovCloud. Nếu bạn đang thiết lập trong một Region tiêu chuẩn, bạn phải điều chỉnh tên partition trong template CloudFormation. Ví dụ: thay đổi giá trị ARN từ `arn:aws-us-gov` thành `arn:aws`.

## Chạy pipeline (Running the pipeline)

Để kích hoạt pipeline, hãy commit các thay đổi vào các file trong kho chứa ứng dụng của bạn. Thao tác đó tạo ra một sự kiện [CloudWatch](https://aws.amazon.com/cloudwatch/) và kích hoạt pipeline. [CodeBuild](https://aws.amazon.com/codebuild/) quét mã nguồn và nếu có bất kỳ lỗ hổng nào, nó sẽ gọi hàm [Lambda](https://aws.amazon.com/lambda/) để phân tích và gửi kết quả đến [Security Hub](https://aws.amazon.com/security-hub/).

Khi gửi thông tin phát hiện lỗ hổng đến Security Hub, chúng ta cần cung cấp mức độ nghiêm trọng (severity level) của lỗ hổng. Dựa trên giá trị mức độ nghiêm trọng được cung cấp, Security Hub gán nhãn như sau. Hãy điều chỉnh các mức độ nghiêm trọng trong mã của bạn dựa trên yêu cầu của tổ chức.

| Giá trị | Nhãn (Label) |
|-------|-------|
| 0 | INFORMATIONAL |
| 1–39 | LOW |
| 40–69 | MEDIUM |
| 70–89 | HIGH |
| 90–100 | CRITICAL |

Ảnh chụp màn hình sau đây hiển thị sự tiến triển của pipeline của bạn.

![Pipeline CI/CD DevSecOps Kubernetes](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/06/18/complete-pipeline.png)

*Pipeline CI/CD DevSecOps Kubernetes*

## Quét phân tích bí mật (Secrets analysis scanning)

Trong kiến trúc này, sau khi pipeline được khởi tạo, [CodeBuild](https://aws.amazon.com/codebuild/) kích hoạt giai đoạn Secret Analysis bằng cách sử dụng **git-secrets** và file `buildspec-gitsecrets.yml`. Git-Secrets tìm kiếm bất kỳ thông tin nhạy cảm nào như AWS access key và secret access key. Git-Secrets cho phép bạn thêm các chuỗi tùy chỉnh để tìm kiếm trong quá trình phân tích. CodeBuild sử dụng file `buildspec-gitsecrets.yml` được cung cấp trong giai đoạn build.

## Quét SCA và SAST (SCA and SAST scanning)

Trong kiến trúc này, [CodeBuild](https://aws.amazon.com/codebuild/) kích hoạt đợt quét SCA và SAST bằng cách sử dụng **Anchore**, **Snyk**, và **Amazon ECR**. Trong giải pháp này, chúng tôi sử dụng các phiên bản mã nguồn mở của Anchore và Snyk. [Amazon ECR](https://aws.amazon.com/ecr/) sử dụng Clair mã nguồn mở bên dưới, đi kèm với Amazon ECR mà không tốn thêm chi phí. Như đã đề cập trước đó, bạn có thể chọn Anchore hoặc Snyk để thực hiện quét image ban đầu.

### Quét với Anchore

Nếu bạn chọn Anchore làm công cụ SAST trong quá trình triển khai, giai đoạn build sử dụng file `buildspec-anchore.yml` để quét container image. Nếu có bất kỳ lỗ hổng nào, nó sẽ làm thất bại bản build và kích hoạt hàm [Lambda](https://aws.amazon.com/lambda/) để gửi các phát hiện đó đến [Security Hub](https://aws.amazon.com/security-hub/). Nếu không có lỗ hổng nào, nó sẽ chuyển sang giai đoạn tiếp theo.

![Đoạn mã Lambda cho Anchore](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/06/18/Anchore-lambda-codesnippet-1024x693.png)

*Đoạn mã Lambda cho Anchore*

### Quét với Snyk

Nếu bạn chọn Snyk làm công cụ SAST trong quá trình triển khai, giai đoạn build sử dụng file `buildspec-snyk.yml` để quét container image. Nếu có bất kỳ lỗ hổng nào, nó sẽ làm thất bại bản build và kích hoạt hàm [Lambda](https://aws.amazon.com/lambda/) để gửi các phát hiện đó đến [Security Hub](https://aws.amazon.com/security-hub/). Nếu không có lỗ hổng nào, nó sẽ chuyển sang giai đoạn tiếp theo.

![Đoạn mã Lambda cho Snyk](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/06/18/Snyk-lambda-codesnippet-1024x729.png)

*Đoạn mã Lambda cho Snyk*

### Quét với Amazon ECR

Nếu không có lỗ hổng nào từ đợt quét Anchore hoặc Snyk, image sẽ được đẩy lên [Amazon ECR](https://aws.amazon.com/ecr/), và đợt quét Amazon ECR được kích hoạt tự động. Amazon ECR liệt kê các phát hiện lỗ hổng trên Amazon ECR console. Để cung cấp một góc nhìn giao diện tập trung duy nhất cho tất cả các phát hiện lỗ hổng và để quản trị dễ dàng, chúng tôi lấy các phát hiện đó và gửi chúng đến [Security Hub](https://aws.amazon.com/security-hub/). Nếu không có lỗ hổng nào, image sẽ được triển khai lên cụm EKS staging và giai đoạn tiếp theo (quét DAST) được kích hoạt.

![Đoạn mã Lambda cho ECR](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/06/18/ECR-lambda-codesnippet-1024x661.png)

*Đoạn mã Lambda cho ECR*

## Quét DAST với OWASP ZAP

Trong kiến trúc này, [CodeBuild](https://aws.amazon.com/codebuild/) kích hoạt đợt quét DAST bằng cách sử dụng công cụ DAST **OWASP ZAP**.

Sau khi triển khai thành công, CodeBuild khởi tạo đợt quét DAST. Khi đợt quét hoàn tất, nếu có bất kỳ lỗ hổng nào, nó sẽ gọi hàm [Lambda](https://aws.amazon.com/lambda/), tương tự như phân tích SAST. Hàm phân tích và gửi kết quả đến [Security Hub](https://aws.amazon.com/security-hub/). Dưới đây là đoạn mã của hàm Lambda.

![Đoạn mã Lambda cho ZAP](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/06/25/Zap-lambda-codesnippet-1-1024x500.png)

*Đoạn mã Lambda cho ZAP*

Ảnh chụp màn hình sau đây hiển thị kết quả trong Security Hub. Phần được làm nổi bật cho thấy các phát hiện lỗ hổng từ các giai đoạn quét khác nhau.

![Các phát hiện lỗ hổng trong Security Hub](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/06/18/SecurityHub-Screenshot.png)

*Các phát hiện lỗ hổng trong Security Hub*

Chúng ta có thể đi chi tiết vào từng ID tài nguyên để lấy danh sách các phát hiện lỗ hổng. Ví dụ, nếu đi chi tiết vào ID tài nguyên của `SASTBuildProject*`, chúng ta có thể xem xét tất cả các phát hiện từ ID tài nguyên đó.

![Lỗ hổng SAST trong Security Hub](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/06/18/SecurityHub-Anchore-Screenshot.png)

*Lỗ hổng SAST trong Security Hub*

Nếu không có lỗ hổng nào trong đợt quét DAST, pipeline sẽ chuyển sang giai đoạn phê duyệt thủ công và một email sẽ được gửi đến người phê duyệt. Người phê duyệt có thể xem xét và phê duyệt hoặc từ chối triển khai. Nếu được phê duyệt, pipeline chuyển sang giai đoạn tiếp theo và triển khai ứng dụng lên cụm EKS production.

Tập hợp các phát hiện lỗ hổng trong [Security Hub](https://aws.amazon.com/security-hub/) cung cấp cơ hội để tự động hóa việc khắc phục. Ví dụ, dựa trên phát hiện lỗ hổng, bạn có thể kích hoạt một hàm [Lambda](https://aws.amazon.com/lambda/) để thực hiện hành động khắc phục cần thiết. Điều này cũng làm giảm gánh nặng cho các đội ngũ vận hành và bảo mật vì hiện tại họ có thể giải quyết các lỗ hổng từ một giao diện quản lý tập trung duy nhất thay vì phải đăng nhập vào nhiều dashboard công cụ khác nhau.

Cùng với Security Hub, bạn có thể gửi các phát hiện lỗ hổng đến các hệ thống theo dõi sự cố của mình như JIRA, [Systems Manager SysOps](https://aws.amazon.com/systems-manager/), hoặc có thể tự động tạo một ticket quản lý sự cố. Điều này nằm ngoài phạm vi của bài viết này, nhưng là một trong những khả năng bạn có thể cân nhắc khi triển khai các nhà máy phần mềm DevSecOps.

## Quét RASP (RASP scanning)

Sysdig Falco là một công cụ bảo mật runtime mã nguồn mở. Dựa trên các rule được cấu hình, Falco có thể phát hiện hoạt động đáng nghi và cảnh báo về bất kỳ hành vi nào liên quan đến việc thực hiện các lệnh gọi hệ thống Linux (Linux system calls). Bạn có thể sử dụng các rule của Falco để giải quyết các kiểm soát bảo mật như NIST SP 800-53. Các agent Falco trên mỗi node EKS liên tục quét các container đang chạy trong pod và gửi các sự kiện dưới dạng STDOUT. Các sự kiện này sau đó có thể được gửi đến [CloudWatch](https://aws.amazon.com/cloudwatch/) hoặc bất kỳ trình tổng hợp log bên thứ ba nào để gửi cảnh báo và phản ứng. Để biết thêm thông tin, xem [Implementing Runtime security in Amazon EKS using CNCF Falco](https://aws.amazon.com/blogs/containers/implementing-runtime-security-in-amazon-eks-using-cncf-falco/). Bạn cũng có thể sử dụng [Lambda](https://aws.amazon.com/lambda/) để kích hoạt và tự động khắc phục một số sự kiện bảo mật nhất định.

Ảnh chụp màn hình sau đây hiển thị các sự kiện Falco trên console [CloudWatch](https://aws.amazon.com/cloudwatch/). Đoạn văn bản được làm nổi bật mô tả sự kiện Falco đã được kích hoạt dựa trên các rule Falco mặc định trên cụm EKS. Bạn có thể thêm các rule tùy chỉnh bổ sung để đáp ứng các yêu cầu kiểm soát bảo mật của mình. Bạn cũng có thể kích hoạt các hành động phản ứng từ các sự kiện CloudWatch này bằng cách sử dụng các dịch vụ như Lambda.

![Cảnh báo Falco trong CloudWatch](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/06/25/falco-alerts-1024x387.png)

*Cảnh báo Falco trong CloudWatch*

## Dọn dẹp (Cleanup)

Phần này cung cấp hướng dẫn dọn dẹp thiết lập pipeline DevSecOps:

1. Xóa [cụm EKS](https://aws.amazon.com/eks/).
2. Xóa [bucket S3](https://aws.amazon.com/s3/).
3. Xóa kho chứa [CodeCommit](https://aws.amazon.com/codecommit/).
4. Xóa kho chứa [Amazon ECR](https://aws.amazon.com/ecr/).
5. Tắt [Security Hub](https://aws.amazon.com/security-hub/).
6. Tắt [AWS Config](https://aws.amazon.com/config/).
7. Xóa stack [CloudFormation](https://aws.amazon.com/cloudformation/) của pipeline.

## Kết luận (Conclusion)

Trong bài viết này, tôi đã trình bày một nhà máy phần mềm DevSecOps dựa trên Kubernetes end-to-end trên AWS với kiểm thử liên tục, ghi log và giám sát liên tục, kiểm toán và quản trị, cùng với vận hành. Tôi đã minh họa cách tích hợp các công cụ quét mã nguồn mở khác nhau, chẳng hạn như Git-Secrets, Anchore, Snyk, OWASP ZAP, và Sysdig Falco cho phân tích Secret Analysis, SCA, SAST, DAST, và RASP tương ứng. Để giảm chi phí quản lý vận hành, tôi đã giải thích cách tổng hợp và quản lý các phát hiện lỗ hổng trong [Security Hub](https://aws.amazon.com/security-hub/) dưới dạng một giao diện tập trung duy nhất. Bài viết này cũng thảo luận về cách triển khai bảo mật CỦA pipeline và TRONG pipeline bằng cách sử dụng các dịch vụ AWS Cloud-native. Cuối cùng, tôi đã cung cấp nhà máy phần mềm DevSecOps dưới dạng mã bằng cách sử dụng [AWS CloudFormation](https://aws.amazon.com/cloudformation/).

Để bắt đầu với DevSecOps trên AWS, xem [AWS DevOps](https://aws.amazon.com/devops/) và [DevOps blog](https://aws.amazon.com/blogs/devops/).

## Về tác giả (About the Author)

<table>
  <tr>
    <td width="130" valign="top">
      <img src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/01/21/IMG_2614-150x150.jpg" width="120" alt="Srinivas Manepalli" />
    </td>
    <td valign="top">
      <br/>
      <b>Srinivas Manepalli</b>
      <br/><br/>
      <font color="#555555">Srinivas Manepalli là DevSecOps Solutions Architect trong đội ngũ U.S. Fed SI SA tại Amazon Web Services (AWS). Ông đam mê hỗ trợ khách hàng, xây dựng và thiết kế các hệ thống phần mềm DevSecOps và hệ thống phần mềm có tính sẵn sàng cao. Ngoài công việc, ông thích dành thời gian cho gia đình, thiên nhiên và món ăn ngon.</font>
    </td>
  </tr>
</table>