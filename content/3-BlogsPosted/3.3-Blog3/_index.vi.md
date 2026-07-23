---
title: "Blog 3"
date: 2021-02-05
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

# Giám sát và mở rộng quy mô ứng dụng Amazon ECS trên AWS Fargate của bạn bằng các chỉ số Prometheus

**bởi Mike George vào ngày 05 THÁNG 2 2021 trong [Advanced (300)](https://aws.amazon.com/blogs/mt/category/learning-levels/advanced-300/), [Amazon CloudWatch](https://aws.amazon.com/blogs/mt/category/management-tools/amazon-cloudwatch/), [Auto Scaling](https://aws.amazon.com/blogs/mt/category/compute/auto-scaling/), [AWS Fargate](https://aws.amazon.com/blogs/mt/category/compute/aws-fargate/), [Containers](https://aws.amazon.com/blogs/mt/category/compute/containers/), [Management Tools](https://aws.amazon.com/blogs/mt/category/management-tools/), Technical How-to**

---

Nếu bạn đã từng chạy một workload trong container, bạn sẽ biết rằng việc kiểm tra những gì đang diễn ra bên trong container có thể rất phức tạp. Trong bài viết này, tôi sẽ hướng dẫn cách bạn có thể giám sát và mở rộng quy mô (scale) ứng dụng [Amazon Elastic Container Service (Amazon ECS)](https://aws.amazon.com/ecs/) trên [AWS Fargate](https://aws.amazon.com/fargate/) bằng cách sử dụng các chỉ số (metric) Prometheus. Mặc dù đã có nhiều thông tin về Prometheus sẵn có, nhưng việc bắt đầu có thể gặp khó khăn nếu bạn tiếp cận container thông qua Amazon ECS trên AWS Fargate. Cuối bài viết này, bạn sẽ có thể sử dụng các metric thu thập được bằng Prometheus để thực hiện các hành động tự động mở rộng quy mô cho ứng dụng Amazon ECS trên AWS Fargate của mình.

[CloudWatch Container Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights.html) cung cấp cho bạn khả năng quan sát những gì đang xảy ra bên trong container. Bằng cách sử dụng các tính năng giám sát hiệu năng có sẵn trong console [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/), bạn có thể xem các thông số CPU, bộ nhớ, mạng, và số byte đọc/ghi của container. Dashboard của CloudWatch Container Insights hiển thị một số quy trình xử lý đang diễn ra bên trong container. Bạn có thể sử dụng thông tin này để tinh chỉnh hiệu năng ứng dụng hoặc quyết định thời điểm cần mở rộng quy mô.

## Giám sát ứng dụng container trong quá khứ

Nhiều ứng dụng không trực tiếp phụ thuộc vào CPU hoặc bộ nhớ (CPU- or memory-bound). Chúng có các đặc tính mở rộng quy mô tương quan với CPU hoặc bộ nhớ, nhưng các metric đó có thể không phải là chỉ số tốt nhất đại diện cho hiệu năng ứng dụng. Ví dụ, trong các ứng dụng Java, mức sử dụng bộ nhớ JVM hoặc số lượng kết nối có thể là chỉ số tốt hơn về hiệu năng. Khi cần mở rộng quy mô, bạn cần một chỉ số tốt hơn thay vì chỉ giám sát dung lượng bộ nhớ container khả dụng.

Chúng ta làm gì nếu cần giám sát tốt hơn các ứng dụng container hóa của mình? Chúng ta có bị giới hạn ở việc chỉ sử dụng metric CPU và bộ nhớ cho tất cả hoạt động giám sát hay có cách nào tốt hơn không? May mắn thay, vào tháng 9 năm 2020, đội ngũ CloudWatch đã công bố khả năng hỗ trợ chính thức cho [các metric Prometheus từ Amazon ECS, Amazon EKS, AWS Fargate, và các cụm Kubernetes](https://aws.amazon.com/about-aws/whats-new/2020/09/cloudwatch-container-insights-prometheus-support-now-generally-available-amazon-ecs-eks-fargate-kubernetes/).

[Prometheus](https://prometheus.io/) đã được sử dụng rộng rãi để giám sát các workload Kubernetes. Nó hoạt động bằng cách thu thập (scrape) metric từ các trang/điểm cuối mà nó giám sát và gửi các metric đó đến cơ sở dữ liệu để xem. Việc tích hợp một exporter với ứng dụng của bạn để chấp nhận các yêu cầu từ Prometheus và cung cấp dữ liệu được yêu cầu là rất phổ biến. Thông thường, exporter được sử dụng cho dữ liệu mà bạn không có toàn quyền kiểm soát (như dữ liệu JVM). Các exporter thường lắng nghe trên một cổng và đường dẫn cụ thể cho các yêu cầu từ Prometheus (như `:9404/metrics/`).

## Thiết lập một container chạy Java có hỗ trợ Prometheus

Bước đầu tiên để sử dụng Prometheus với [Amazon ECS](https://aws.amazon.com/ecs/) trên [AWS Fargate](https://aws.amazon.com/fargate/) là thiết lập một container được cấu hình để cung cấp các metric cho Prometheus. Đối với bài hướng dẫn này, tôi triển khai một file WAR Java làm ứng dụng.

Sau khi bạn đã tìm được ứng dụng để triển khai, hãy làm theo các bước sau:

1. Nếu bạn chưa có file WAR để triển khai, hãy tải xuống [ứng dụng Tomcat mẫu](https://tomcat.apache.org/tomcat-9.0-doc/appdev/sample/) từ Apache. Mẫu này chỉ là một ứng dụng Java thuần chạy trên Tomcat.
2. Cài đặt [Docker](https://docs.docker.com/get-docker/).
3. Tải xuống exporter cho Java Tomcat từ [kho lưu trữ Maven chính thức](https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/). Nếu bạn muốn xem mã nguồn, nó có sẵn trong [kho lưu trữ GitHub chính thức của Prometheus](https://github.com/prometheus/jmx_exporter).
4. Định nghĩa cấu hình cho exporter này. Kho lưu trữ [jmx_exporter](https://github.com/prometheus/jmx_exporter) có sẵn một số cấu hình mẫu. Trong bài demo này, tôi sử dụng ví dụ yml của họ. Tải xuống file ví dụ và đặt tên là `config.yaml`.
5. Tạo một file tên là `setenv.sh` với nội dung sau:

```bash
export JAVA_OPTS="-javaagent:/opt/jmx_exporter/jmx_prometheus_javaagent-0.14.0.jar=9404:/opt/jmx_exporter/config.yaml $JAVA_OPTS"
```

6. Tạo một file `Dockerfile` mới với nội dung sau:

```dockerfile
# From Tomcat 9.0 JDK8 OpenJDK
FROM tomcat:9.0-jdk8-openjdk

RUN mkdir -p /opt/jmx_exporter

COPY ./jmx_prometheus_javaagent-0.14.0.jar /opt/jmx_exporter
COPY ./config.yaml /opt/jmx_exporter
COPY ./setenv.sh /usr/local/tomcat/bin
COPY sample.war /usr/local/tomcat/webapps/

RUN chmod o+x /usr/local/tomcat/bin/setenv.sh

ENTRYPOINT ["catalina.sh", "run"]
```

7. Bây giờ hãy build Docker container.
8. Mở console của [Amazon Elastic Container Registry (Amazon ECR)](https://aws.amazon.com/ecr/), và chọn **Create repository**.
9. Chọn repository bạn vừa tạo, chọn **View push commands**, và làm theo hướng dẫn cho hệ điều hành của bạn. Trong demo này, tôi đặt tên container là `hello-prometheus`.

![Hình 1: container hello-prometheus trong Amazon ECR console](https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2021/02/05/Figure1-1024x458.png)

*Hình 1: container hello-prometheus trong Amazon ECR console*

10. Sau khi bạn đã build container và đẩy nó lên Amazon ECR, tại mục **Image URI**, chọn **Copy URI**. Bạn sẽ cần URI này trong quy trình tiếp theo.

## Tạo một AWS Fargate task có hỗ trợ Prometheus

Bây giờ bạn đã đẩy container của mình lên, hãy tạo một task. Task là đơn vị công việc cốt lõi trong [AWS Fargate](https://aws.amazon.com/fargate/). Khi bạn tạo một định nghĩa task Fargate (task definition), bạn định nghĩa các tài nguyên tính toán và cấu hình gắn liền với đơn vị công việc này.

1. Trong [Amazon ECS console](https://console.aws.amazon.com/ecs/), chuyển đến tab **Task Definition**, sau đó chọn **Create new task definition**.
2. Trên trang **Select compatibilities**, chọn **FARGATE** làm kiểu khởi chạy cho task và chọn **Next step**.
3. Tại mục **Task Definition Name**, nhập tên cho task definition của bạn. Tôi sử dụng `hello-prometheus-task`.
4. Tại mục **Task Role**, chọn một role [AWS Identity and Access Management (IAM)](https://aws.amazon.com/iam/) cung cấp quyền cho các container trong task của bạn thực hiện các lời gọi AWS API thay mặt bạn.
5. Tại mục **Task execution IAM role**, tôi sử dụng `ecsTaskExecutionRole`. Nếu bạn chưa tạo role này, hãy làm theo các bước trong [Creating the task execution IAM role](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_execution_IAM_role.html).
6. Vì task này không đòi hỏi nhiều về bộ nhớ và CPU, tại mục **Task memory (GB)**, chọn **0.5 GB**. Tại mục **Task CPU (vCPU)**, chọn **0.25 vCPU**.
7. Để định nghĩa container được sử dụng cho task này, chọn **Add container**.
8. Nhập tên cho container. Tôi sử dụng `hello-prometheus`.
9. Tại mục **Image**, dán URI image mà bạn đã sao chép sau khi đẩy container lên Amazon ECR.
10. Tại mục **Port mappings**, nhập các cổng TCP `8080` và `9404` như hiển thị ở Hình 2. Ứng dụng được thiết kế để người dùng truy cập qua cổng 8080. Prometheus sử dụng cổng 9404 để thu thập (scrape) metric. Task này phản hồi trang web trên `:8080/sample/` và phản hồi metric cho Prometheus trên `:9404/metrics/`.

![Hình 2: Ánh xạ cổng cho ứng dụng và Prometheus](https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2021/02/05/Figure2-2-1024x956.png)

*Hình 2: Ánh xạ cổng cho ứng dụng và Prometheus*

11. Trong mục **Docker Labels**, thêm hai cặp key-value như hiển thị ở Hình 3. Prometheus sử dụng các label này (`Java_EMF_Metrics` với giá trị `true` và `ECS_PROMETHEUS_EXPORTER_PORT` với giá trị `9404`) để tự động phát hiện các task của bạn.

![Hình 3: Các cặp Key-value](https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2021/02/05/Figure3-1-1024x326.png)

*Hình 3: Các cặp Key-value*

12. Chọn **Add** để lưu cấu hình container.
13. Ở cuối trang task, chọn **Create** để tạo task.

## Triển khai task trên AWS Fargate

Bạn có thể triển khai task [AWS Fargate](https://aws.amazon.com/fargate/) này tương tự như bất kỳ task Fargate nào khác. Nếu bạn đã biết cách làm, hãy chuyển sang phần tiếp theo.

Để hoàn thành bước này, bạn cần một VPC và một Application Load Balancer. Để việc xây dựng hạ tầng dễ dàng hơn, bạn có thể triển khai template CloudFormation này, template sẽ tạo một [Amazon Virtual Private Cloud (Amazon VPC)](https://aws.amazon.com/vpc/) với hai public subnet, một Application Load Balancer, và một security group cho Application Load Balancer và task Fargate của bạn.

Để triển khai template:

1. Sao chép file template CloudFormation bên dưới và lưu vào máy tính của bạn. Tôi đặt tên template là `setup-networking.yml`.
2. Từ cửa sổ lệnh (command prompt), gõ:

```bash
aws cloudformation deploy --template-file setup-networking.yml \
--stack-name hello-prometheus
```

Chạy lệnh này yêu cầu [AWS CLI](https://aws.amazon.com/cli/) phải được cấu hình và có đủ quyền [IAM](https://aws.amazon.com/iam/) để thực thi stack CloudFormation.

3. Chờ stack được triển khai, quá trình này mất chưa đến 5 phút.

**setup-networking.yml**

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: This template sets up a simple VPC with 2 public subnets, ingress on tcp port 80, and an Application Load Balancer

Outputs:
  VPCId:
    Value: !Ref VPC
  PublicSubnet1:
    Value: !Ref Subnet1
  PublicSubnet2:
    Value: !Ref Subnet2
  PublicSecurityGroup:
    Value: !Ref PublicSecurityGroup

Resources:
  VPC:
    Type: "AWS::EC2::VPC"
    Properties:
      CidrBlock: "192.168.0.0/16"
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
      - Key: Name
        Value: !Ref "AWS::StackName"
  IG:
    Type: "AWS::EC2::InternetGateway"
    Properties:
      Tags:
      - Key: Name
        Value: !Ref "AWS::StackName"
  IGAttachment:
    Type: "AWS::EC2::VPCGatewayAttachment"
    Properties:
      InternetGatewayId: !Ref IG
      VpcId: !Ref VPC
  IGVPCRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
      - Key: Name
        Value: !Ref "AWS::StackName"
  IGVPCRoute:
    Type: AWS::EC2::Route
    Properties:
      DestinationCidrBlock: "0.0.0.0/0"
      RouteTableId: !Ref IGVPCRouteTable
      GatewayId: !Ref IG
  Subnet1ToRouteTable:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref IGVPCRouteTable
      SubnetId: !Ref Subnet1
  Subnet2ToRouteTable:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref IGVPCRouteTable
      SubnetId: !Ref Subnet2
  Subnet1:
    Type: AWS::EC2::Subnet
    Properties:
      CidrBlock: "192.168.1.0/26"
      AvailabilityZone: !Select [0, !GetAZs '' ]
      MapPublicIpOnLaunch: true
      Tags:
      - Key: Name
        Value: !Ref "AWS::StackName"
      - Key: SubnetType
        Value: Public Subnet
      VpcId: !Ref VPC
  Subnet2:
    Type: AWS::EC2::Subnet
    Properties:
      CidrBlock: "192.168.2.0/26"
      AvailabilityZone: !Select [1, !GetAZs '' ]
      MapPublicIpOnLaunch: true
      Tags:
      - Key: Name
        Value: !Ref "AWS::StackName"
      - Key: SubnetType
        Value: Public Subnet
      VpcId: !Ref VPC
  PublicSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      VpcId: !Ref VPC
      GroupName: !Sub "${AWS::StackName}-public-sg"
      GroupDescription: Public security group
      SecurityGroupIngress:
        - IpProtocol: tcp
          ToPort: 80
          FromPort: 80
          CidrIp: "0.0.0.0/0"
      Tags:
      - Key: Name
        Value: !Ref "AWS::StackName"
  PublicSecurityGroupIngress:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !Ref PublicSecurityGroup
      IpProtocol: tcp
      FromPort: 0
      ToPort: 65535
      SourceSecurityGroupId: !Ref PublicSecurityGroup
  LoadBalancer:
    Type : "AWS::ElasticLoadBalancingV2::LoadBalancer"
    Properties :
      Name : !Sub "${AWS::StackName}-alb"
      SecurityGroups :
      - !Ref PublicSecurityGroup
      Subnets :
      - !Ref Subnet1
      - !Ref Subnet2
      Tags:
      - Key: Name
        Value: !Ref "AWS::StackName"
```

Bây giờ hạ tầng đã được xây dựng, hãy triển khai task của bạn!

Đầu tiên, tạo một cụm ECS cluster. Trong ECS, cluster chỉ đơn giản là một cơ chế để nhóm các task hoặc service một cách logic.

1. Trong [Amazon ECS console](https://console.aws.amazon.com/ecs/), chọn **Clusters**, và chọn **Create Cluster**.
2. Chọn **Networking only** để tạo cụm [AWS Fargate](https://aws.amazon.com/fargate/), sau đó chọn **Next step**.
3. Nhập tên cho cluster. Tôi sử dụng `hello-prometheus-cluster`.
4. Chọn **Create** để tạo cluster.

Bây giờ, hãy triển khai task AWS Fargate của bạn vào cluster mới. Bạn có thể triển khai task AWS Fargate bằng cách tạo một dịch vụ Amazon ECS service. Đó là cơ chế để duy trì và chạy đồng thời các task definition trong cụm ECS cluster.

1. Trên trang **Clusters**, chọn **Services**, sau đó chọn **Create**.
2. Tại mục launch type, chọn **FARGATE**.
3. Từ **Task definition**, chọn `hello-prometheus-task`. Phiên bản (revision) nên được cập nhật lên bản mới nhất, là 1 (latest).
4. Tại mục **Service name**, tôi sử dụng `hello-prometheus-service`. Đối với số lượng task (number of tasks), tôi nhập `2` để đảm bảo luôn có 2 bản sao của task này chạy, sau đó chọn **Next step**.
5. Trong phần **Networking**, chọn VPC đã được tạo bởi template CloudFormation. Tại mục **Subnets**, chọn các public subnet. Tại mục **Security group**, chọn **Edit**, sau đó chọn security group phù hợp (trường hợp của tôi là `hello-prometheus-public-sg`). Chọn **Save**.
6. Dưới phần **Load balancing**, tại mục **Load balancer type**, chọn **Application Load Balancer**.
7. Chọn **Use an existing load balancer**, sau đó chọn Application Load Balancer của bạn (trường hợp của tôi là `hello-prometheus-alb`).
8. Vì container được thiết kế để phục vụ lưu lượng truy cập công khai qua cổng 8080, dưới danh sách dropdown container name: port, chọn `8080`, sau đó chọn **Add to load balancer**.
9. Tại mục **Listener**, nhập `80`.
10. Tại mục **Target group name**, nhập `hello-prometheus-target`.
11. Tại mục **Health check path**, nhập `/sample/`. (Hãy nhớ bao gồm dấu gạch chéo ở cuối.)
12. Chọn **Next step**, chọn **Next step** một lần nữa, sau đó chọn **Create service**.

Bây giờ bạn đã triển khai một ứng dụng có hỗ trợ Prometheus trên [AWS Fargate](https://aws.amazon.com/fargate/). Sau khi các container chuyển sang trạng thái RUNNING và vượt qua kiểm tra sức khỏe (health checks), bạn có thể xem trang web. Để lấy tên DNS của ứng dụng, hãy đến console [Amazon Elastic Compute Cloud (Amazon EC2)](https://aws.amazon.com/ec2/), chọn load balancer của bạn và xem thông tin chi tiết trên tab **Description**.

![Hình 4: Tab Description hiển thị các chi tiết, bao gồm tên DNS](https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2021/02/05/Figure4-2-1024x656.png)

*Hình 4: Tab Description hiển thị các chi tiết, bao gồm tên DNS*

Để xem ứng dụng mới, hãy truy cập đường dẫn `/sample/` tại địa chỉ DNS của Application Load Balancer của bạn. Bạn sẽ thấy trang web sau:

![Hình 5: Trang web "Hello, World" trên Tomcat](https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2021/02/05/Figure5-1024x330.png)

*Hình 5: Trang web "Hello, World" trên Tomcat*

## Xem chỉ số Prometheus trong CloudWatch

Đến thời điểm này, bạn đã triển khai một task Fargate có hỗ trợ Prometheus, nhưng dữ liệu vẫn chưa được đẩy vào [CloudWatch](https://aws.amazon.com/cloudwatch/). Hãy nhớ rằng task Fargate hỗ trợ Prometheus đang tạo ra các metric có thể xem trên cổng 9404 từ đường dẫn `/metrics`. Hãy thiết lập một task Fargate mới để thu thập (scrape) các metric này từ đường dẫn đó và gửi chúng đến CloudWatch. Điều này rất dễ thực hiện với [CloudWatch agent có hỗ trợ Prometheus](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights-Prometheus-Setup.html).

CloudWatch agent với Prometheus cần hai file cấu hình để hoạt động chính xác. File cấu hình đầu tiên là cấu hình Prometheus chuẩn được định nghĩa trong [tài liệu Prometheus](https://prometheus.io/docs/prometheus/latest/configuration/configuration/). File cấu hình thứ hai dành cho CloudWatch agent. Để biết thêm thông tin về các file này, xem [Scraping additional Prometheus sources and importing those metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights-Prometheus-Setup-configure.html) trong Hướng dẫn sử dụng Amazon CloudWatch.

Bạn có thể sử dụng [template CloudFormation trong mẫu của AWS dành cho Amazon CloudWatch Container Insights](https://github.com/aws-samples/amazon-cloudwatch-container-insights) làm điểm khởi đầu để thiết lập và cấu hình CloudWatch agent. Tôi đã chỉnh sửa cấu hình `CWAgentConfigSSMParameter` trong template này để kiểm soát các metric ứng dụng được nạp vào CloudWatch. Sao chép template CloudFormation sau đây và lưu thành `install-prometheus-collector.yaml`.

**install-prometheus-collector.yaml**

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: This template sets up the configuration and tasks necessary to collect Prometheus metrics via the CloudWatch agent for Prometheus 

Parameters:
  ECSClusterName:
    Type: String
    Description: Enter the name of your ECS cluster from which you want to collect Prometheus metrics
  CreateIAMRoles:
    Type: String
    AllowedValues:
      - 'True'
      - 'False'
    Description: Whether to create new IAM roles or using existing IAM roles for the ECS tasks
    ConstraintDescription: must specified, either True or False
  ECSLaunchType:
    Type: String
    AllowedValues:
      - 'EC2'
      - 'FARGATE'
    Description: ECS launch type for the ECS cluster
    ConstraintDescription: must specified, either EC2 or FARGATE
  TaskRoleName:
    Type: String
    Description: Enter the CloudWatch agent ECS task role name
  ExecutionRoleName:
    Type: String
    Description: Enter the CloudWatch agent ECS execution role name
  SecurityGroupID:
    Type: String
    Description: Enter the security group ID for running the CloudWatch agent ECS task
  SubnetID:
    Type: String
    Description: Enter the subnet ID for running the CloudWatch agent ECS task
Conditions:
  CreateRoles:
    !Equals [!Ref CreateIAMRoles, 'True']
  AssignPublicIp:
    !Equals [!Ref ECSLaunchType, 'FARGATE']
Resources:
  PrometheusConfigSSMParameter:
    Type: AWS::SSM::Parameter
    Properties:
      Name: !Sub 'AmazonCloudWatch-PrometheusConfigName-${ECSClusterName}-${ECSLaunchType}-awsvpc'
      Type: String
      Tier: Standard
      Description: !Sub 'Prometheus Scraping SSM Parameter for ECS Cluster: ${ECSClusterName}'
      Value: |-
        global:
          scrape_interval: 1m
          scrape_timeout: 10s
        scrape_configs:
          - job_name: cwagent-ecs-file-sd-config
            sample_limit: 10000
            file_sd_configs:
              - files: [ "/tmp/cwagent_ecs_auto_sd.yaml" ]
  CWAgentConfigSSMParameter:
    Type: AWS::SSM::Parameter
    Properties:
      Name: !Sub 'AmazonCloudWatch-CWAgentConfig-${ECSClusterName}-${ECSLaunchType}-awsvpc'
      Type: String
      Tier: Intelligent-Tiering
      Description: !Sub 'CWAgent SSM Parameter with App Mesh and Java EMF Definition for ECS Cluster: ${ECSClusterName}'
      Value: |-
        {
          "agent": {
            "debug": true
          },
          "logs": {
            "metrics_collected": {
              "prometheus": {
                "prometheus_config_path": "env:PROMETHEUS_CONFIG_CONTENT",
                "ecs_service_discovery": {
                  "sd_frequency": "1m",
                  "sd_result_file": "/tmp/cwagent_ecs_auto_sd.yaml",
                  "docker_label": {
                    "sd_port_label": "ECS_PROMETHEUS_EXPORTER_PORT",
                    "sd_job_name_label": "ECS_PROMETHEUS_JOB_NAME"
                  },
                  "task_definition_list": [
                    {
                      "sd_job_name": "bugbash-workload-java-ec2-awsvpc-task-def-sd",
                      "sd_metrics_ports": "9404;9406",
                      "sd_task_definition_arn_pattern": ".*:task-definition/hello-prometheus*"
                    }
                  ]
                },
                "emf_processor": {
                  "metric_declaration_dedup": true,
                  "metric_declaration": [
                    {
                      "source_labels": ["Java_EMF_Metrics"],
                      "label_matcher": "^true$",
                      "dimensions": [["ClusterName","TaskDefinitionFamily"]],
                      "metric_selectors": [
                        "^jvm_threads_(current|daemon)$",
                        "^jvm_classes_loaded$",
                        "^java_lang_operatingsystem_(freephysicalmemorysize|totalphysicalmemorysize|freeswapspacesize|totalswapspacesize|systemcpuload|processcpuload|availableprocessors|openfiledescriptorcount)$",
                        "^catalina_manager_(rejectedsessions|activesessions)$",
                        "^jvm_gc_collection_seconds_(count|sum)$",
                        "^catalina_globalrequestprocessor_(bytesreceived|bytessent|requestcount|errorcount|processingtime)$"
                      ]
                    },
                    {
                      "source_labels": ["Java_EMF_Metrics"],
                      "label_matcher": "^true$",
                      "dimensions": [["ClusterName","TaskDefinitionFamily","pool"]],
                      "metric_selectors": [
                        "^jvm_memory_pool_bytes_used$"
                      ]
                    },
                    {
                      "source_labels": ["Java_EMF_Metrics"],
                      "label_matcher": "^true$",
                      "dimensions": [["ClusterName","TaskDefinitionFamily","port","protocol"]],
                      "metric_selectors": [
                        "^tomcat_threadpool_connectioncount$"
                      ]
                    }
                  ]
                }
              }
            },
            "force_flush_interval": 5
          }
        }
  CWAgentECSExecutionRole:
    Type: AWS::IAM::Role
    Condition: CreateRoles
    Properties:
      RoleName: !Sub ${ExecutionRoleName}-${AWS::Region}
      Description: Allows ECS container agent to make calls to the Amazon ECS API on your behalf.
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: ecs-tasks.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy
        - arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy
      Policies:
        - PolicyName: ECSSSMInlinePolicy
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - ssm:GetParameters
                Resource: 
                  !Sub 'arn:aws:ssm:${AWS::Region}:${AWS::AccountId}:parameter/AmazonCloudWatch-*'
  CWAgentECSTaskRole:
    Type: AWS::IAM::Role
    Condition: CreateRoles
    DependsOn: CWAgentECSExecutionRole
    Properties:
      RoleName: !Sub ${TaskRoleName}-${AWS::Region}
      Description: Allows ECS tasks to call AWS services on your behalf.
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: ecs-tasks.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy
      Policies:
        - PolicyName: ECSServiceDiscoveryInlinePolicy
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - ecs:DescribeTasks
                  - ecs:ListTasks
                  - ecs:DescribeContainerInstances
                Resource: "*"
                Condition:
                  ArnEquals:
                    ecs:cluster:
                      !Sub 'arn:${AWS::Partition}:ecs:${AWS::Region}:${AWS::AccountId}:cluster/${ECSClusterName}'
              - Effect: Allow
                Action:
                  - ec2:DescribeInstances
                  - ecs:DescribeTaskDefinition
                Resource: "*"
  ECSCWAgentTaskDefinition:
    Type: 'AWS::ECS::TaskDefinition'
    DependsOn:
      - PrometheusConfigSSMParameter
      - CWAgentConfigSSMParameter
    Properties:
      Family: !Sub 'cwagent-prometheus-${ECSClusterName}-${ECSLaunchType}-awsvpc'
      TaskRoleArn: !If [CreateRoles, !GetAtt CWAgentECSTaskRole.Arn, !Sub 'arn:${AWS::Partition}:iam::${AWS::AccountId}:role/${TaskRoleName}']
      ExecutionRoleArn: !If [CreateRoles, !GetAtt CWAgentECSExecutionRole.Arn, !Sub 'arn:${AWS::Partition}:iam::${AWS::AccountId}:role/${ExecutionRoleName}']
      NetworkMode: awsvpc
      ContainerDefinitions:
        - Name: cloudwatch-agent-prometheus
          Image: amazon/cloudwatch-agent:1.248913.0-prometheus
          Essential: true
          MountPoints: []
          PortMappings: []
          Environment: []
          Secrets:
            - Name: PROMETHEUS_CONFIG_CONTENT
              ValueFrom: !Sub 'AmazonCloudWatch-PrometheusConfigName-${ECSClusterName}-${ECSLaunchType}-awsvpc'
            - Name: CW_CONFIG_CONTENT
              ValueFrom: !Sub 'AmazonCloudWatch-CWAgentConfig-${ECSClusterName}-${ECSLaunchType}-awsvpc'
          LogConfiguration:
            LogDriver: awslogs
            Options:
              awslogs-create-group: 'True'
              awslogs-group: "/ecs/ecs-cwagent-prometheus"
              awslogs-region: !Ref AWS::Region
              awslogs-stream-prefix: !Sub 'ecs-${ECSLaunchType}-awsvpc'
      RequiresCompatibilities:
        - !Ref ECSLaunchType
      Cpu: '512'
      Memory: '1024'
  ECSCWAgentService:
    Type: AWS::ECS::Service
    Properties:
      Cluster: !Ref ECSClusterName
      DesiredCount: 1
      LaunchType: !Ref ECSLaunchType
      SchedulingStrategy: REPLICA
      ServiceName: !Sub 'cwagent-prometheus-replica-service-${ECSLaunchType}-awsvpc'
      TaskDefinition: !Ref ECSCWAgentTaskDefinition
      NetworkConfiguration:
        AwsvpcConfiguration:
          AssignPublicIp: !If [AssignPublicIp, ENABLED, DISABLED]
          SecurityGroups:
            - !Ref SecurityGroupID
          Subnets:
            - !Ref SubnetID
```

Sao chép script `prometheus-install.sh` bên dưới vào một file. Chỉnh sửa 4 dòng đầu tiên của script để phù hợp với môi trường của bạn.

**prometheus-install.sh**

```bash
export AWS_DEFAULT_REGION=us-east-1
export ECS_CLUSTER_NAME=hello-prometheus-cluster
export ECS_CLUSTER_SECURITY_GROUP=sg-0000000000000
export ECS_CLUSTER_SUBNET=subnet-00000000000000000
export ECS_LAUNCH_TYPE=FARGATE
export CREATE_IAM_ROLES=True

aws cloudformation deploy --stack-name CWAgent-Prometheus-ECS-${ECS_CLUSTER_NAME}-${ECS_LAUNCH_TYPE}-awsvpc \
    --template-file install-prometheus-collector.yaml \
    --parameter-overrides ECSClusterName=${ECS_CLUSTER_NAME} \
                CreateIAMRoles=${CREATE_IAM_ROLES} \
                ECSLaunchType=${ECS_LAUNCH_TYPE} \
                SecurityGroupID=${ECS_CLUSTER_SECURITY_GROUP} \
                SubnetID=${ECS_CLUSTER_SUBNET} \
                TaskRoleName=Prometheus-TaskRole-${ECS_CLUSTER_NAME} \
                ExecutionRoleName=Prometheus-ExecutionRole-${ECS_CLUSTER_NAME} \
    --capabilities CAPABILITY_NAMED_IAM \
    --region ${AWS_DEFAULT_REGION} 
```

`AWS_DEFAULT_REGION` là Region AWS nơi ứng dụng của bạn đang chạy (ví dụ `us-east-1`). `ECS_CLUSTER_NAME` là tên của cụm ECS cluster bạn đã tạo cho bài demo này (trường hợp của tôi là `hello-prometheus-cluster`). `ECS_CLUSTER_SECURITY_GROUP` là ID của security group gắn với các task AWS Fargate. Nếu bạn đã sử dụng template CloudFormation để xây dựng VPC, bạn có thể lấy tên của security group bằng cách chạy lệnh:

```bash
aws cloudformation describe-stacks --stack-name hello-prometheus \
--query "Stacks[0].Outputs[?OutputKey=='PublicSecurityGroup'].OutputValue" \
--output text
```

Bạn có thể lấy tên của một public subnet trong VPC của bạn (giá trị `ECS_CLUSTER_SUBNET`) bằng cách chạy lệnh:

```bash
aws cloudformation describe-stacks --stack-name hello-prometheus \
--query "Stacks[0].Outputs[?OutputKey=='PublicSubnet1'].OutputValue" \
--output text
```

Lưu file và chạy từ cửa sổ lệnh. Bạn có thể cần chạy `chmod +x` trên file để cấp quyền thực thi. Trong trường hợp của tôi, tôi đã đặt tên cho script này là `prometheus-install.sh`.

```bash
./prometheus-install.sh
```

Script này thực hiện các công việc sau:

- Tạo một task Fargate mới sử dụng CloudWatch agent với Prometheus.
- Thêm task vào một service trong cụm ECS cluster bạn đã tạo.
- Thiết lập các role [IAM](https://aws.amazon.com/iam/) thích hợp cho task này hoạt động chính xác.

Script này cũng tạo hai giá trị trong [AWS Systems Manager Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html):

- Giá trị bắt đầu bằng `AmazonCloudWatch-CWAgentConfig-` chứa cấu hình cho CloudWatch agent.
- Giá trị bắt đầu bằng `AmazonCloudWatch-PrometheusConfigName-` chứa cấu hình thu thập (scraping) của Prometheus.

Nếu bạn xem lại các cấu hình trong Parameter Store, bạn sẽ thấy một vài tham chiếu đến các Docker label đã được thêm vào định nghĩa task Fargate. Bộ thu thập Prometheus sử dụng các Docker label đó để phát hiện và nhận diện ứng dụng của bạn — hoặc ngay cả các ứng dụng mới bạn thêm vào — và bắt đầu gửi các metric thu thập được từ các ứng dụng này đến CloudWatch.

Sau khi stack CloudFormation được tạo, bạn sẽ thấy một task mới đang chạy trong cluster của mình. Task này sử dụng thông tin cấu hình từ hai trường trong Parameter Store để xác định metric nào cần thu thập từ `hello-prometheus-task` và tần suất thu thập.

![Hình 6: Ba task đang chạy trong cụm Fargate cluster](https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2021/02/05/Figure6-2-1024x565.png)

*Hình 6: Ba task đang chạy trong cụm Fargate cluster*

Sau một vài phút, các metric mới sẽ xuất hiện trong [CloudWatch](https://aws.amazon.com/cloudwatch/) dưới namespace `ECS/ContainerInsights/Prometheus`. Các metric này được thu thập từ ứng dụng của bạn và gửi đến CloudWatch thông qua Prometheus. Những metric này hoàn toàn giống với bất kỳ metric nào khác bạn tìm thấy trong console CloudWatch.

Nếu xem xét các metric được thu thập từ Prometheus, bạn sẽ thấy tôi đang thu thập thông tin JVM, giúp tôi có khả năng quan sát (observability) tốt hơn đối với ứng dụng Java của mình. Tôi có thể sử dụng thông tin này để đưa ra các quyết định tốt hơn về sức khỏe của ứng dụng.

## Sử dụng chỉ số thu thập từ Prometheus để tự động mở rộng quy mô

Tôi muốn mở rộng quy mô ứng dụng của mình dựa trên số lượng kết nối. Khi số lượng kết nối đến ứng dụng tăng lên, tôi muốn mở rộng ra (scale out). Khi số lượng kết nối giảm xuống, ứng dụng sẽ thu hẹp lại (scale in). Việc sử dụng metric thu thập từ Prometheus cho các hành động mở rộng quy mô trên Fargate rất đơn giản.

Đầu tiên, chọn metric bạn muốn dựa vào đó để mở rộng quy mô. Tôi sẽ sử dụng metric `tomcat_threadpool_connectioncount`, nằm trong [CloudWatch](https://aws.amazon.com/cloudwatch/) tại đường dẫn `ECS/ContainerInsights/Prometheus > [ClusterName, TaskDefinitionFamily, port. protocol]`. Tôi muốn xem số lượng task Fargate tăng lên như thế nào khi quy tắc mở rộng quy mô được áp dụng. Để tìm số lượng tài nguyên Fargate on-demand, hãy mở console CloudWatch và chọn **Metrics**. Tại mục **Usage**, chọn **By AWS Resource**. Chọn metric `ResourceCount` liên kết với Fargate Service và OnDemand Resource.

![Hình 7: Chỉ số tomcat_threadpool_connectioncount](https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2021/02/05/Figure7-1024x544.png)

*Hình 7: Chỉ số tomcat_threadpool_connectioncount*

Fargate mở rộng số lượng task đang chạy dựa trên các cảnh báo (alarm) được kích hoạt. Một lần nữa, đối với demo này, ứng dụng sẽ mở rộng ra khi số lượng kết nối tăng lên và thu hẹp vào khi số lượng kết nối giảm xuống. Để thiết lập các cảnh báo này, chọn biểu tượng cảnh báo (hình cái chuông) bên cạnh metric `tomcat_threadpool_connectioncount`.

Để tạo cảnh báo có thể được Fargate sử dụng cho các hoạt động mở rộng ra (scale-out), hãy bắt đầu bằng cách truy cập trang **Alarms** của [CloudWatch console](https://console.aws.amazon.com/cloudwatch/):

1. Trên trang **Alarms** của CloudWatch console, giữ tất cả giá trị mặc định. Chọn **Greater**. Tại mục **than**, nhập `150`. Chọn **Next**.
2. Chọn **Create a new topic** và nhập tên cho topic. Tôi sử dụng `hello-prometheus-scaleup`. Tại mục **Email endpoints that will receive the notification**, nhập địa chỉ email của bạn. Chọn **Create topic**, và chọn **Next**.
3. Tại trường **Alarm name**, nhập tên cho cảnh báo. Tôi sử dụng `hello-prometheus-scaleup`. Chọn **Next**, và chọn **Create alarm**.

Bây giờ hãy tạo cảnh báo có thể được Fargate sử dụng cho các hoạt động thu hẹp vào (scale-in) bằng cách chuyển đến trang **Alarms** của CloudWatch console:

1. Trên trang **Alarms** của CloudWatch console, chọn **Create alarm**.
2. Chọn **Select metric**, và chọn **ECS/ContainerInsights/Prometheus > [ClusterName, TaskDefinitionFamily, port, protocol]**.
3. Chọn `tomcat_threadpool_connectioncount`, và chọn **Select metric**.
4. Sử dụng các giá trị mặc định. Chọn **Lower**. Tại mục **than**, nhập `100`. Chọn **Next**.
5. Chọn **Create a new topic**, và nhập tên topic. Tôi sử dụng `hello-prometheus-scaledown`. Tại mục **Email endpoints that will receive the notification**, nhập địa chỉ email của bạn.
6. Chọn **Create topic**, và chọn **Next**.
7. Tại trường **Alarm name**, nhập tên cho cảnh báo. Tôi sử dụng `hello-prometheus-scaledown`.
8. Chọn **Next**, và chọn **Create alarm**.

Sau một vài phút, cả hai cảnh báo sẽ chuyển từ trạng thái "insufficient data" sang trạng thái **In alarm** hoặc **OK**.

![Hình 8: Trang Alarms trong CloudWatch console](https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2021/02/05/Figure8-1-1024x306.png)

*Hình 8: Trang Alarms trong CloudWatch console*

Cảnh báo `hello-prometheus-scaledown` đang ở trạng thái **In alarm** vì hiện tại không có lưu lượng truy cập trên trang web. Bây giờ hãy cấu hình [AWS Fargate](https://aws.amazon.com/fargate/) để thu hẹp quy mô bất cứ khi nào cảnh báo này kích hoạt, cho đến khi đạt số lượng task tối thiểu cần chạy. Khi có đủ lưu lượng truy cập vào trang web, cảnh báo `hello-prometheus-scaleup` sẽ kích hoạt, khiến AWS Fargate mở rộng quy mô các task, cho đến khi đạt số lượng task tối đa cần chạy.

Để định nghĩa policy mở rộng ra (scale-out) cho AWS Fargate, hãy bắt đầu bằng cách chuyển đến [Amazon ECS console](https://console.aws.amazon.com/ecs/):

1. Trong Amazon ECS console, chọn service AWS Fargate bạn muốn mở rộng quy mô, và chọn **Update**. Service AWS Fargate của tôi là `hello-prometheus-service`.
2. Chọn **Next step**, và chọn **Next step**.
3. Trên trang **Set Auto Scaling (optional)**, chọn **Configure Service Auto Scaling to adjust to your service's desired count**.
4. Tại mục **Minimum number of tasks**, nhập `2`.
5. Tại mục **Maximum number of tasks**, nhập `10`.
6. Chọn **Add scaling policy**.
7. Trên trang **Add policy**, đổi **Scaling policy type** thành **Step scaling**.
8. Tại mục **Policy name**, tôi sử dụng `hello-prometheus-scaleup-policy`.
9. Chọn **Use an existing alarm**, và chọn cảnh báo scale-out bạn vừa tạo. Cảnh báo của tôi là `hello-prometheus-scaleup`.
10. Dưới phần **Scaling action**, chọn **Add**, và nhập `1`.
11. Chọn **Save**.

Bây giờ bạn đã định nghĩa policy hướng dẫn AWS Fargate cách mở rộng ra. Lặp lại quy trình để định nghĩa policy hướng dẫn AWS Fargate cách thu hẹp vào.

1. Chọn **Add scaling policy**.
2. Trên trang **Add policy**, chọn **Step scaling**.
3. Tại mục **Policy name**, tôi sử dụng `hello-prometheus-scaledown-policy`.
4. Chọn **Use an existing alarm**, và chọn cảnh báo scale-in bạn đã tạo. Cảnh báo của tôi là `hello-prometheus-scaledown`.
5. Dưới phần **Scaling action**, chọn **Remove**, và nhập `1`.
6. Chọn **Save**.
7. Chọn **Next step**, và chọn **Update Service**.

Cụm cluster hiện đã được cập nhật để tự động mở rộng quy mô dựa trên các metric tùy chỉnh thu thập qua Prometheus! Sau khi gửi lưu lượng truy cập đến ứng dụng của bạn, bạn có thể xem các cảnh báo scale-out và scale-in hoạt động. Đường màu xanh dương trên biểu đồ bên dưới đại diện cho số lượng kết nối đến ứng dụng. Đường màu cam đại diện cho số lượng task đang chạy. Bạn có thể thấy khi lưu lượng tăng lên, ứng dụng đã mở rộng ra, và khi lưu lượng giảm xuống, ứng dụng đã từ từ thu hẹp quy mô trở lại.

![Hình 9: Biểu đồ mở rộng quy mô của Fargate](https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2021/02/05/Figure9-1-1024x685.png)

*Hình 9: Biểu đồ mở rộng quy mô của Fargate*

## Dọn dẹp (Cleanup)

Để tránh phát sinh chi phí liên tục cho tài khoản AWS của bạn, hãy xóa các tài nguyên bạn đã tạo.

1. Xóa các cảnh báo scale-out và scale-in trên [CloudWatch](https://aws.amazon.com/cloudwatch/).
2. Xóa tất cả dịch vụ [AWS Fargate](https://aws.amazon.com/fargate/) và các task đang chạy trong cụm ECS cluster.
3. Sử dụng console [AWS CloudFormation](https://aws.amazon.com/cloudformation/) để xóa hai stack đã tạo như một phần của bài demo này. Chọn từng stack, chọn **Delete**, và chọn **Delete stack**.

## Xem xét chi phí (Cost considerations)

Với [Amazon CloudWatch Container Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights.html), bạn chỉ trả tiền cho những gì bạn sử dụng. Để xem ví dụ rõ ràng về chi phí, hãy truy cập [trang giá Amazon CloudWatch](https://aws.amazon.com/cloudwatch/pricing/). Bạn sẽ thấy giá cho các cảnh báo CloudWatch trên tab **Alarms**.

Giá của [AWS Fargate](https://aws.amazon.com/fargate/) được tính dựa trên tài nguyên vCPU và bộ nhớ được sử dụng từ thời điểm bạn bắt đầu tải xuống container image cho đến khi task AWS Fargate dừng lại, làm tròn lên giây gần nhất. Để xem giá hiện tại và các cấu hình được hỗ trợ, hãy truy cập [trang giá AWS Fargate](https://aws.amazon.com/fargate/pricing/).

Mặc dù demo này sử dụng một VPC được tạo trong [Amazon VPC](https://aws.amazon.com/vpc/), nó không sử dụng các tính năng của Amazon VPC tính phí. Để xem giá hiện tại, hãy truy cập [trang giá Amazon VPC](https://aws.amazon.com/vpc/pricing/).

Bạn sẽ bị tính phí cho mỗi giờ hoặc một phần của giờ mà một [Application Load Balancer](https://aws.amazon.com/elasticloadbalancing/application-load-balancer/) chạy và số lượng Load Balancer Capacity Units (LCU) được sử dụng mỗi giờ. Một LCU bao gồm các kết nối mới, kết nối đang hoạt động, số byte đã xử lý, và số lượng quy tắc được xử lý bởi load balancer. Để xem giá hiện tại và các ví dụ, hãy truy cập [trang giá Elastic Load Balancing](https://aws.amazon.com/elasticloadbalancing/pricing/).

## Kết luận (Conclusion)

Trong bài viết này, tôi đã cho bạn thấy cách sử dụng các metric Prometheus giúp có được những thông tin chuyên sâu tốt hơn về các workload container. Tôi đã hướng dẫn bạn cách:

- Thiết lập và triển khai một task [Amazon ECS](https://aws.amazon.com/ecs/) trên [AWS Fargate](https://aws.amazon.com/fargate/) có hỗ trợ Prometheus.
- Gửi các metric từ task đến [CloudWatch](https://aws.amazon.com/cloudwatch/).
- Sử dụng metric tùy chỉnh thu thập qua Prometheus để mở rộng ra (scale out) và thu hẹp vào (scale in) một ứng dụng Amazon ECS trên AWS Fargate.

Sử dụng CloudWatch agent với Prometheus, bạn có một phương pháp mới mạnh mẽ để đưa các metric hiệu năng từ các workload vào [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/). Các metric ứng dụng thu thập được qua CloudWatch agent với Prometheus cung cấp khả năng quan sát tốt hơn về hiệu năng container và sức khỏe ứng dụng. Bạn có thể sử dụng những gì đã học trong bài viết này để thu thập các metric tùy chỉnh qua Prometheus, và sử dụng các metric đó để cấu hình tự động mở rộng quy mô (auto scaling) trên các ứng dụng [Amazon ECS](https://aws.amazon.com/ecs/) trên [AWS Fargate](https://aws.amazon.com/fargate/) của bạn.

Để biết thêm thông tin, hãy đọc về cách [thiết lập và cấu hình thu thập chỉ số Prometheus trên các cụm Amazon ECS cluster](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights-Prometheus-Setup.html).

## Về tác giả (About the Author)

<table>
  <tr>
    <td width="130" valign="top">
      <img src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2021/08/09/gmike-profile-800x800-2.png" width="120" alt="Mike George" />
    </td>
    <td valign="top">
      <br/>
      <b>Mike George</b>
      <br/><br/>
      <font color="#555555">Mike George là Principal Solutions Architect làm việc tại Salt Lake City, Utah. Ông thích giúp đỡ khách hàng giải quyết các vấn đề công nghệ của họ. Các mối quan tâm của ông bao gồm kỹ thuật phần mềm, bảo mật, trí tuệ nhân tạo (AI), và học máy (ML).</font>
    </td>
  </tr>
</table>