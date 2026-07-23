---
title: "Blog 1"
date: 2021-03-26
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Xây dựng môi trường Jenkins serverless trên AWS Fargate

**bởi Krithivasan Balasubramaniyan vào ngày 26 THÁNG 3 2021 trong [Amazon Elastic File System (EFS)](https://aws.amazon.com/vi/blogs/devops/category/storage/amazon-elastic-file-system-efs/), [AWS Fargate](https://aws.amazon.com/vi/blogs/devops/category/compute/aws-fargate/)**

---

Jenkins là một máy chủ tự động hóa mã nguồn mở phổ biến, cho phép các nhà phát triển trên toàn thế giới xây dựng, kiểm thử và triển khai phần mềm một cách đáng tin cậy. Jenkins sử dụng kiến trúc controller-agent, trong đó controller chịu trách nhiệm phục vụ giao diện web, lưu trữ các cấu hình và dữ liệu liên quan trên đĩa, đồng thời phân phối các công việc đến các agent thực thi — đây là nhiệm vụ chính của các agent.

[Amazon Elastic Container Service (Amazon ECS)](https://aws.amazon.com/ecs/) là dịch vụ điều phối container được quản lý toàn phần và là lựa chọn phổ biến để triển khai môi trường Jenkins có tính sẵn sàng cao, khả năng chịu lỗi và khả năng mở rộng. Theo cách truyền thống, kiểu khởi chạy [Amazon Elastic Compute Cloud (Amazon EC2)](https://aws.amazon.com/ec2/) là lựa chọn ưa tiên, kết hợp với [Amazon Elastic Block Store (Amazon EBS)](https://aws.amazon.com/ebs/) làm bộ nhớ lưu trữ cấu hình và dữ liệu. Điều này đã thay đổi khi [AWS Fargate](https://aws.amazon.com/fargate/) hỗ trợ [Amazon EFS](https://aws.amazon.com/efs/).

Mục tiêu của bài viết này là hướng dẫn bạn cách thiết lập một môi trường Jenkins hoàn toàn serverless trên [AWS Fargate](https://aws.amazon.com/fargate/) bằng cách sử dụng [Terraform](https://www.terraform.io/).

## Tổng quan về giải pháp

Sơ đồ dưới đây minh họa kiến trúc của giải pháp.

![Sơ đồ này mô tả kiến trúc triển khai Jenkins trên AWS Fargate](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/03/24/Jenkins.jpg)

Mục tiêu là tạo ra một triển khai hoàn toàn tự động, có tính sẵn sàng cao, sẵn sàng cho môi trường sản xuất của Jenkins trong một môi trường serverless trên AWS. Chúng ta sử dụng các thành phần và dịch vụ sau:

- URL của Jenkins controller được hỗ trợ bởi [Application Load Balancer](https://aws.amazon.com/elasticloadbalancing/application-load-balancer/). Khuyến nghị sử dụng [AWS Certificate Manager (ACM)](https://aws.amazon.com/certificate-manager/) để cấp phát chứng chỉ SSL liên kết với load balancer. Việc chấm dứt SSL diễn ra tại load balancer.
- Chúng ta sử dụng một [VPC](https://aws.amazon.com/vpc/) với ít nhất hai public subnet và hai private subnet.
- Jenkins controller chạy như một service trong [Amazon ECS](https://aws.amazon.com/ecs/) sử dụng Fargate làm kiểu khởi chạy. Chúng ta sử dụng [Amazon Elastic File System (Amazon EFS)](https://aws.amazon.com/efs/) làm bộ lưu trữ liên tục cho task Jenkins controller. Jenkins controller và Amazon EFS được khởi chạy trong các private subnet.
- Jenkins sử dụng [plugin Amazon ECS Fargate](https://plugins.jenkins.io/amazon-ecs/) để ủy quyền cho Amazon ECS chạy các bản build trên các agent dựa trên Docker. Mỗi bản build Jenkins chạy trên một Docker container riêng biệt, container này sẽ bị xóa sạch sau khi build hoàn thành. Các Jenkins agent khám phá Jenkins controller task thông qua [AWS Cloud Map](https://aws.amazon.com/cloud-map/) để tìm kiếm dịch vụ.
- Bạn có thể sử dụng plugin nêu trên để tạo nhiều định nghĩa cấu hình Amazon EC2 Container Service Cloud. Điều này cho phép bạn sử dụng ECS cluster với capacity provider Fargate Spot để chạy các Jenkins job có thể chịu đựng sự gián đoạn. Điều này giúp tối ưu hóa chi phí cho các trường hợp như chạy bộ kiểm thử lớn và các job build.
- Jenkins controller có thể đảm nhận các vai trò [AWS Identity and Access Management (IAM)](https://aws.amazon.com/iam/) dựa trên ARN được cung cấp cùng với định nghĩa task Fargate. Bạn có thể định nghĩa nhiều template ECS agent trong plugin ECS với các phạm vi quyền khác nhau để áp dụng các thực hành bảo mật tốt nhất.
- [AWS Backup](https://aws.amazon.com/backup/) được sử dụng để đảm bảo bạn có thể khôi phục tất cả dữ liệu quan trọng từ Jenkins controller trong trường hợp xảy ra sự cố.
- Ngoài ra, bạn có thể cấu hình thông báo cho người dùng thông qua [Amazon Simple Notification Service (Amazon SNS)](https://aws.amazon.com/sns/) để cảnh báo người dùng về bất kỳ lỗi nào trong các job build hoặc trong môi trường.

## Điều kiện tiên quyết

Trong bài viết này, chúng tôi đã phát triển một Terraform module để thực hiện việc triển khai. Module và các script ví dụ triển khai có sẵn trong [GitHub repo](https://github.com/aws-samples/jenkins-ecs-agents). Tham khảo README để biết danh sách tất cả các biến của module. Sau đây là những yêu cầu cần có để triển khai Terraform module này:

- [Terraform](https://www.terraform.io/downloads.html) phiên bản 13 trở lên.
- Một [VPC](https://aws.amazon.com/vpc/) với ít nhất hai public subnet và hai private subnet.
- Một chứng chỉ SSL để liên kết với Application Load Balancer. Khuyến nghị sử dụng [chứng chỉ ACM](https://aws.amazon.com/certificate-manager/). Việc này không được thực hiện bởi Terraform module chính. Tuy nhiên, ví dụ trong thư mục example sử dụng module ACM công khai để tạo chứng chỉ ACM và truyền vào module serverless Jenkins. Bạn có thể thực hiện theo cách này hoặc trực tiếp truyền ARN của một chứng chỉ bạn đã tạo hoặc nhập vào ACM trước đó.
- Mật khẩu admin cho Jenkins phải được lưu trữ trong [AWS Systems Manager Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html). Tham số này phải có kiểu SecureString và có tên là `jenkins-pwd`.
- Terraform phải được khởi tạo (bootstrap). Điều này có nghĩa là một bucket [Amazon Simple Storage Service (Amazon S3)](https://aws.amazon.com/s3/) để lưu trữ state và một bảng [Amazon DynamoDB](https://aws.amazon.com/dynamodb/) để khóa state phải được khởi tạo trước.

## Triển khai

Tất cả các tài nguyên và cấu hình cần thiết đều được đóng gói thành một Terraform module, nghĩa là module này không thể triển khai trực tiếp. Tuy nhiên, một ví dụ triển khai đã có trong thư mục example. Để triển khai ví dụ, hãy thực hiện các bước sau:

1. Clone [GitHub repo](https://github.com/aws-samples/jenkins-ecs-agents).
2. Chuyển thư mục làm việc sang thư mục `bootstrap`.

   Trong thư mục này có chứa code Terraform mẫu để khởi tạo các tài nguyên quản lý state Terraform ban đầu. Một thực hành tốt nhất là sử dụng một state backend như [Amazon S3](https://aws.amazon.com/s3/) và cơ chế khóa như [DynamoDB](https://aws.amazon.com/dynamodb/) khi sử dụng Terraform. Để biết thêm thông tin, xem [State Storage and Locking](https://www.terraform.io/language/state/locking). Vì code bootstrap này tạo ra các tài nguyên quản lý state Terraform, cần đặc biệt cẩn thận để lưu file state Terraform kết quả. Lưu ý rằng state này chỉ được lưu vào một file cục bộ có tên `terraform.tfstate`. Hãy chắc chắn lưu file state này nếu bạn muốn duy trì S3 state bucket và DynamoDB lock table bằng Terraform.

   Thay `my-state-bucket` và `my-lock-table` bằng tên bạn muốn:

   ```bash
   terraform init
   terraform apply \
       -var="state_bucket_name=my-state-bucket" \
       -var="state_lock_table_name=my-lock-table"
   ```

3. Để triển khai module, chuyển thư mục làm việc sang `example`.
4. Sao chép `vars.sh.example` thành `vars.sh`.
5. Chỉnh sửa các biến trong `vars.sh` theo môi trường của bạn (VPC, các subnet, ID vùng [Route 53](https://aws.amazon.com/route53/) và tên miền, state bucket, bảng state locking, và Region mà bạn muốn triển khai).
6. Chạy `deploy_example.sh`.

Sau khi triển khai tất cả các tài nguyên, bạn có thể mở trình duyệt và nhập tên bản ghi alias [Route 53](https://aws.amazon.com/route53/). Thao tác này sẽ mở giao diện web Jenkins. Đăng nhập bằng người dùng `ecsuser` và mật khẩu được lưu trong parameter store. Tất cả các plugin được đề cập trong `plugins.txt` cũng được cài đặt, và các cấu hình Fargate container service cloud được áp dụng. Hai cloud được tạo ra: một cho Fargate và một cho cluster Fargate Spot. Ảnh chụp màn hình sau đây hiển thị Fargate cloud.

![Ảnh này mô tả cấu hình Amazon EC2 container service cloud cho plugin Amazon Elastic Container Service (ECS) / Fargate.](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/03/23/Screen-Shot-2020-11-29-at-1.37.17-PM.png)

Bạn sẽ thấy hai job đã được cấu hình sẵn.

![Ảnh này mô tả hai Jenkins job được cấu hình sẵn. Một job sẽ lên lịch chạy trên cluster FARGATE tiêu chuẩn và job kia sẽ lên lịch chạy trên cluster với capacity provider FARGATE_SPOT.](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/03/23/Screen-Shot-2020-11-29-at-1.16.46-PM.png)

Trước khi đi sâu vào các job đó, hãy cùng tìm hiểu về capacity provider.

[Amazon ECS](https://aws.amazon.com/ecs/) trên Fargate capacity provider cho phép bạn sử dụng cả dung lượng Fargate và Fargate Spot với các task Amazon ECS của mình. Để biết thêm thông tin về capacity provider, xem [Amazon ECS capacity providers](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cluster-capacity-providers.html). Với Fargate Spot, bạn có thể chạy các Amazon ECS task có khả năng chịu gián đoạn với mức giá được chiết khấu so với giá Fargate thông thường. Fargate Spot chạy các task trên dung lượng tính toán dự phòng. Khi AWS cần lấy lại dung lượng đó, các task của bạn sẽ bị gián đoạn với cảnh báo trước 2 phút.

Là một phần của module vừa triển khai, hai ECS Fargate cluster được cung cấp: một cluster với capacity provider `FARGATE` và cluster còn lại với capacity provider `FARGATE_SPOT`. Dịch vụ Jenkins controller được cấu hình để chạy trên cluster với capacity provider `FARGATE`.

Hai cấu hình ECS container service cloud được tạo ra trước đó quyết định agent nào sẽ chạy các job. Ví dụ, bạn có thể cấu hình tất cả các workload có thể chịu gián đoạn (như các job build và bộ kiểm thử) để chạy trên `fargate-cloud-spot` — được hỗ trợ bởi ECS Fargate cluster với capacity provider `FARGATE_SPOT`, và tất cả các workload còn lại (như các lần chạy Terraform apply) để chạy trên `fargate-cloud` — được hỗ trợ bởi ECS Fargate cluster với capacity provider `FARGATE`. Capacity provider Fargate Spot giúp tối ưu hóa chi phí bằng cách sử dụng dung lượng tính toán dự phòng trong AWS Cloud. Chúng ta gọi các job được lên lịch chạy trên Spot capacity provider là các task không quan trọng, và các task chạy trên Fargate capacity provider là các task quan trọng, dựa trên bản chất của những job đó. Pipeline khai báo Jenkins cho job task quan trọng trông như sau:

```groovy
pipeline {
    agent {
        ecs {
            inheritFrom 'build-example'
        }
    }
    stages {
      stage('Test') {
          steps {
              script {
                  sh "echo this was executed on non spot instance"
              }
              sh 'sleep 120'
              sh 'echo sleep is done'
          }
      }
    }
}
```

## Kiểm thử giải pháp

Để kiểm thử giải pháp, trước tiên hãy chạy **Simple Job Critical Task** bằng cách chọn **Build Now**. Thao tác này tạo ra một task trong cluster với capacity provider `FARGATE`.

![Ảnh minh họa rằng lần chạy Jenkins job tạo ra một task trong cluster với capacity provider FARGATE.](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/03/23/Screen-Shot-2020-11-29-at-1.55.37-PM.png)

Khi task ở trạng thái Running, nó tự đăng ký thành một agent và Jenkins job chạy trên agent này.

![Ảnh minh họa rằng ECS task mới được tạo tự đăng ký thành một Jenkins agent.](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/03/23/Screen-Shot-2020-11-29-at-2.00.43-PM.png)

Bạn có thể xem đầu ra console trong Jenkins.

![Ảnh minh họa đầu ra console của Jenkins job.](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/03/23/Screen-Shot-2020-11-29-at-2.02.25-PM.png)

Khi job hoàn thành thành công, task sẽ bị dừng lại.

![Ảnh minh họa rằng ECS task bị dừng sau khi Jenkins job hoàn thành.](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/03/24/Screen-Shot-2020-11-29-at-2.05.01-PM-1.png)

## Dọn dẹp tài nguyên

Để dọn dẹp tất cả các tài nguyên đã được tạo ra để xây dựng môi trường Jenkins serverless, điều hướng đến thư mục `example` và chạy lệnh sau:

```bash
terraform destroy -auto-approve
```

Để dọn dẹp bảng DynamoDB lock và S3 state bucket, điều hướng đến thư mục `example/bootstrap` và chạy lệnh sau:

```bash
terraform destroy -auto-approve
```

## Hạn chế

Vì [Amazon EFS](https://aws.amazon.com/efs/) là bộ nhớ lưu trữ gắn qua mạng, trong cấu hình mặc định của nó có thể không đủ hiệu năng cho một số trường hợp sử dụng Jenkins. Khuyến nghị nên nghiên cứu và kiểm thử kỹ lưỡng trước khi sử dụng Amazon EFS cho các workload trong môi trường sản xuất. Để biết thêm thông tin, xem [Amazon EFS performance](https://docs.aws.amazon.com/efs/latest/ug/performance.html).

Nếu cần thiết, hãy cài đặt Amazon EFS ở chế độ maximum I/O. Bạn có thể thực hiện điều này trong Terraform module bằng cách đặt biến `efs_performance_mode` thành `maxIO`. Để biết thêm thông tin, xem [What are the differences between General Purpose and Max I/O performance modes in Amazon EFS?](https://aws.amazon.com/premiumsupport/knowledge-center/linux-efs-performance-modes/)

## Kết luận

Trong bài viết này, chúng ta đã tìm hiểu cách triển khai một môi trường Jenkins có tính sẵn sàng cao, khả năng mở rộng, chịu lỗi tốt và sẵn sàng cho môi trường sản xuất trong một môi trường hoàn toàn serverless trên [Fargate](https://aws.amazon.com/fargate/). Chúng ta cũng đã trình bày cách sử dụng Terraform module để tự động hóa việc triển khai tất cả các tài nguyên cần thiết và các cấu hình liên quan. Bài viết cũng giới thiệu cách sử dụng capacity provider `FARGATE` và `FARGATE_SPOT`.

Khi chúng ta triển khai từng job trong Docker container riêng của nó, chúng ta có thể kiểm soát các yêu cầu CPU và bộ nhớ cho từng job, container cơ sở xác định môi trường, các công cụ cần thiết cho job, và quan trọng hơn là không làm quá tải controller với các lần chạy job.

## Về các tác giả

<table>
  <tr>
    <td width="130" valign="top">
      <img src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2021/03/16/Krithivasan-Balasubramaniyan-Author.jpg" width="120" alt="Krithivasan Balasubramaniyan" />
    </td>
    <td valign="top">
      <br/>
      <b>Krithivasan Balasubramaniyan</b>
      <br/><br/>
      <font color="#555555">Krithivasan là Senior Consultant tại AWS. Ông hỗ trợ các khách hàng doanh nghiệp toàn cầu trong hành trình chuyển đổi số và giúp thiết kế các giải pháp cloud-native.</font>
    </td>
  </tr>
</table>

<table>
  <tr>
    <td width="130" valign="top">
      <img src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/03/24/aws_badge_photo.jpeg" width="120" alt="Jason Steenblik" />
    </td>
    <td valign="top">
      <br/>
      <b>Jason Steenblik</b>
      <br/><br/>
      <font color="#555555">Jason là Senior Cloud Architect tại AWS và rất đam mê mọi thứ về DevOps. Trong vai trò của mình tại AWS, ông hướng dẫn các khách hàng doanh nghiệp trong quá trình chuyển đổi lên cloud.</font>
    </td>
  </tr>
</table>

<table>
  <tr>
    <td width="130" valign="top">
      <img src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/03/23/pogorie-high-res-current-photo.jpg" width="120" alt="Alexander Pogorielov" />
    </td>
    <td valign="top">
      <br/>
      <b>Alexander Pogorielov</b>
      <br/><br/>
      <font color="#555555">Alexander là Global DevOps Architect trong AWS ProServe với hơn 8 năm kinh nghiệm trong lĩnh vực ảo hóa, công nghệ cloud và tự động hóa.</font>
    </td>
  </tr>
</table>