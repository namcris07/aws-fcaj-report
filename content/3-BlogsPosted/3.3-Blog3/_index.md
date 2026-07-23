---
title: "Blog 3"
date: 2021-02-05
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

# Monitor and scale your Amazon ECS on AWS Fargate application using Prometheus metrics

**by Mike George on 05 FEB 2021 in [Advanced (300)](https://aws.amazon.com/blogs/mt/category/learning-levels/advanced-300/), [Amazon CloudWatch](https://aws.amazon.com/blogs/mt/category/management-tools/amazon-cloudwatch/), [Auto Scaling](https://aws.amazon.com/blogs/mt/category/compute/auto-scaling/), [AWS Fargate](https://aws.amazon.com/blogs/mt/category/compute/aws-fargate/), [Containers](https://aws.amazon.com/blogs/mt/category/compute/containers/), [Management Tools](https://aws.amazon.com/blogs/mt/category/management-tools/), Technical How-to**

---

If you've ever run a containerized workload, you know that it can be tricky to check what's happening in your container. In this blog post, I show how you can monitor and scale your [Amazon Elastic Container Service (Amazon ECS)](https://aws.amazon.com/ecs/) on [AWS Fargate](https://aws.amazon.com/fargate/) application using Prometheus metrics. Although there is more information about Prometheus already available, it can be difficult to get started if you've come to containers through Amazon ECS on AWS Fargate. At the end of this post, you'll be able to use metrics gathered using Prometheus to perform automatic scaling actions on your Amazon ECS on AWS Fargate application.

[CloudWatch Container Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights.html) gives you visibility into what is happening inside your container. By using its performance monitoring features, available in the [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/) console, you can see the CPU, memory, network, and bytes read/written of a container. The CloudWatch Container Insights dashboard shows some of the processing that is happening inside your container. You can use this information to performance tune your application or decide when you should scale.

## The olden days of monitoring containerized applications

Many applications are not directly CPU- or memory-bound. They have scaling characteristics that are correlated with CPU or memory, but those metrics might not be the best indicator of application performance. For example, in Java applications, JVM memory usage or connection count might be a better indicator of performance. When scaling is required, you need a better indicator than just monitoring available container memory.

What do we do if we need to better monitor our containerized applications? Are we limited to using CPU and memory metrics for all our monitoring or is there a better way? Fortunately, in September 2020, the CloudWatch team announced the general availability of [Prometheus metrics from Amazon ECS, Amazon EKS, AWS Fargate, and Kubernetes clusters](https://aws.amazon.com/about-aws/whats-new/2020/09/cloudwatch-container-insights-prometheus-support-now-generally-available-amazon-ecs-eks-fargate-kubernetes/).

[Prometheus](https://prometheus.io/) is already widely used to monitor Kubernetes workloads. It works by scraping metrics from the sites it is monitoring and sends off those metrics to a database for viewing. It's common to integrate an exporter with your application, to accept requests from Prometheus and provide the data requested. Typically, an exporter is used for data that you don't have full control over (like JVM data). Exporters generally listen on a specific port and path for Prometheus requests (like `:9404/metrics/`).

## Set up a Java-based, Prometheus enabled container

The first step to using Prometheus with [Amazon ECS](https://aws.amazon.com/ecs/) on [AWS Fargate](https://aws.amazon.com/fargate/) is to set up a container that is configured to provide metrics to Prometheus. For this demo, I deploy a Java WAR file as the application.

After you've found an application to deploy, do the following:

1. If you don't already have a WAR file to deploy, download the [Tomcat sample application](https://tomcat.apache.org/tomcat-9.0-doc/appdev/sample/) from Apache. This sample is just a vanilla Java application that runs on Tomcat.
2. Install [Docker](https://docs.docker.com/get-docker/).
3. Download an exporter for Java Tomcat, which you can get from the [official Maven repo](https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/). If you want to look at the source code, it's available in the [official Prometheus GitHub repository](https://github.com/prometheus/jmx_exporter).
4. Define the configuration for this exporter. The [jmx_exporter repo](https://github.com/prometheus/jmx_exporter) has a few example configurations. For this demo, I use their yml example. Download their example and name it `config.yaml`.
5. Create a file named `setenv.sh` with the following content:

```bash
export JAVA_OPTS="-javaagent:/opt/jmx_exporter/jmx_prometheus_javaagent-0.14.0.jar=9404:/opt/jmx_exporter/config.yaml $JAVA_OPTS"
```

6. Create a new `Dockerfile` with the following content:

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

7. Now build the Docker container.
8. Open the [Amazon Elastic Container Registry (Amazon ECR)](https://aws.amazon.com/ecr/) console, and choose **Create repository**.
9. Select the repository you created, choose **View push commands**, and follow the instructions for your operating system. In this demo, I named my container `hello-prometheus`.

![Figure 1: hello-prometheus container in the Amazon ECR console](https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2021/02/05/Figure1-1024x458.png)

*Figure 1: hello-prometheus container in the Amazon ECR console*

10. After you've built your container and pushed it to Amazon ECR, under **Image URI**, choose **Copy URI**. You need this URI in the following procedure.

## Create a Prometheus-enabled AWS Fargate task

Now that you've pushed your container, create a task. A task is the core unit of work in [AWS Fargate](https://aws.amazon.com/fargate/). When you create a Fargate task definition, you define the compute and configuration associated with this unit of work.

1. In the [Amazon ECS console](https://console.aws.amazon.com/ecs/), go to the **Task Definition** tab, and then choose **Create new task definition**.
2. On the **Select compatibilities** page, select **FARGATE** as the launch type that your task should use and choose **Next step**.
3. For **Task Definition Name**, enter a name for your task definition. I use `hello-prometheus-task`.
4. For **Task Role**, choose an [AWS Identity and Access Management (IAM)](https://aws.amazon.com/iam/) role that provides permissions for containers in your task to make calls to AWS API operations on your behalf.
5. For **Task execution IAM role**, I use the `ecsTaskExecutionRole`. If you don't have this role already created, follow the steps in [Creating the task execution IAM role](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_execution_IAM_role.html).
6. Because this task doesn't need much from a memory and CPU perspective, for **Task memory (GB)**, choose **0.5 GB**. For **Task CPU (vCPU)**, choose **0.25 vCPU**.
7. To define the container used for this task, choose **Add container**.
8. Enter a name for the container. I use `hello-prometheus`.
9. For **Image**, paste in the image URI you copied after you pushed your container to Amazon ECR.
10. Under **Port mappings**, enter TCP ports `8080` and `9404` as shown in Figure 2. The application is designed to be viewed by users over port 8080. Prometheus uses port 9404 to scrape metrics. This task responds with the internet website on `:8080/sample/` and it responds to Prometheus with metrics on `:9404/metrics/`.

![Figure 2: Port mappings for the application and Prometheus](https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2021/02/05/Figure2-2-1024x956.png)

*Figure 2: Port mappings for the application and Prometheus*

11. In **Docker Labels**, add two key-value pairs, as shown in Figure 3. Prometheus uses these labels (`Java_EMF_Metrics` with a value of `true` and `ECS_PROMETHEUS_EXPORTER_PORT` with a value of `9404`) to auto-discover your tasks.

![Figure 3: Key-value pairs](https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2021/02/05/Figure3-1-1024x326.png)

*Figure 3: Key-value pairs*

12. Choose **Add** to save the container configuration.
13. At the bottom of the task page, choose **Create** to create the task.

## Deploy the task in AWS Fargate

You can deploy this [AWS Fargate](https://aws.amazon.com/fargate/) task the same way you'd deploy any other Fargate task. If you already know how to do that, skip to the next section.

To complete this step, you need a VPC and an Application Load Balancer. To make building the infrastructure easy, you can deploy this CloudFormation template, which creates an [Amazon Virtual Private Cloud (Amazon VPC)](https://aws.amazon.com/vpc/) with two public subnets, an Application Load Balancer, and a security group for the Application Load Balancer and your Fargate task.

To deploy the template:

1. Copy the CloudFormation template file listed below, and save it to your computer. I named the template `setup-networking.yml`.
2. From a command prompt, type:

```bash
aws cloudformation deploy --template-file setup-networking.yml \
--stack-name hello-prometheus
```

Running this command requires the [AWS CLI](https://aws.amazon.com/cli/) to be configured and [IAM](https://aws.amazon.com/iam/) permissions sufficient to launch a CloudFormation stack.

3. Wait for the stack to be deployed, which should take less than 5 minutes.

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

Now that the infrastructure has been built, deploy your task!

First, create an ECS cluster. In ECS, a cluster is simply a mechanism to logically group tasks or services.

1. In the [Amazon ECS console](https://console.aws.amazon.com/ecs/), choose **Clusters**, and then choose **Create Cluster**.
2. Choose **Networking only** to create an [AWS Fargate](https://aws.amazon.com/fargate/) cluster, and then choose **Next step**.
3. Enter a name for the cluster. I use `hello-prometheus-cluster`.
4. Choose **Create** to create the cluster.

Now, deploy your AWS Fargate task to the new cluster. You can deploy an AWS Fargate task by creating an Amazon ECS service. It's the mechanism for maintaining and running task definitions simultaneously in an ECS cluster.

1. On the **Clusters** page, choose **Services**, and then choose **Create**.
2. For the launch type, choose **FARGATE**.
3. From **Task definition**, choose `hello-prometheus-task`. The revision should be updated to the latest version, which is 1 (latest).
4. For **Service name**, I use `hello-prometheus-service`. For the number of tasks, I enter `2` to ensure that I'll always have two replicas of this task running, and then choose **Next step**.
5. In the **Networking** section, choose the VPC created by the CloudFormation template. For **Subnets**, choose the public subnets. For **Security group**, choose **Edit**, and then choose the appropriate security group (in my case, `hello-prometheus-public-sg`). Choose **Save**.
6. Under **Load balancing**, for **Load balancer type**, choose **Application Load Balancer**.
7. Choose **Use an existing load balancer**, and then choose your Application Load Balancer (in my case, `hello-prometheus-alb`).
8. Because the container is designed to serve public traffic over port 8080, under the container name: port dropdown list, choose `8080`, and then choose **Add to load balancer**.
9. For **Listener**, enter `80`.
10. For **Target group name**, enter `hello-prometheus-target`.
11. For **Health check path**, enter `/sample/`. (Be sure to include the trailing slash.)
12. Choose **Next step**, choose **Next step** again, and then choose **Create service**.

You have now deployed a Prometheus-enabled application on [AWS Fargate](https://aws.amazon.com/fargate/). After the containers enter a RUNNING state and pass health checks, you can view the website. To get the application's DNS name, go to the [Amazon Elastic Compute Cloud (Amazon EC2)](https://aws.amazon.com/ec2/) console, choose your load balancer, and then view its details on the **Description** tab.

![Figure 4: Description tab displays details, including DNS name](https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2021/02/05/Figure4-2-1024x656.png)

*Figure 4: Description tab displays details, including DNS name*

To view the new application, go to `/sample/` at the DNS address of your Application Load Balancer. You should see the following webpage:

![Figure 5: Tomcat "Hello, World" webpage](https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2021/02/05/Figure5-1024x330.png)

*Figure 5: Tomcat "Hello, World" webpage*

## View Prometheus metrics in CloudWatch

To this point, you've deployed a Prometheus-enabled Fargate task, but the data is not yet being ingested into [CloudWatch](https://aws.amazon.com/cloudwatch/). Remember that the Prometheus-enabled Fargate task is producing metrics that can be viewed on port 9404 from the `/metrics` path. Set up a new Fargate task that scrapes these metrics from that path and sends them to CloudWatch. This is easy to do with the [CloudWatch agent with Prometheus support](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights-Prometheus-Setup.html).

The CloudWatch agent with Prometheus needs two configuration files to work correctly. The first configuration file is the standard Prometheus configuration defined in the [Prometheus documentation](https://prometheus.io/docs/prometheus/latest/configuration/configuration/). The second configuration is for the CloudWatch agent. For more information about these files, see [Scraping additional Prometheus sources and importing those metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights-Prometheus-Setup-configure.html) in the Amazon CloudWatch User Guide.

You can use the [CloudFormation template in AWS samples for Amazon CloudWatch Container Insights](https://github.com/aws-samples/amazon-cloudwatch-container-insights) as a starting point to set up and configure the CloudWatch agent. I edited the `CWAgentConfigSSMParameter` configuration in this template to control the application metrics being ingested into CloudWatch. Copy the following CloudFormation template and save it as `install-prometheus-collector.yaml`.

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

Copy the `prometheus-install.sh` script below to a file. Edit the first four lines of the script to correspond to your environment.

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

`AWS_DEFAULT_REGION` is the AWS Region where your application is running (for example, `us-east-1`). `ECS_CLUSTER_NAME` is the name of the ECS cluster you created for this demo (in my case, `hello-prometheus-cluster`). `ECS_CLUSTER_SECURITY_GROUP` is the security group ID associated with the AWS Fargate tasks. If you used the CloudFormation template to build your VPC, you can get the name of the security group by running this command:

```bash
aws cloudformation describe-stacks --stack-name hello-prometheus \
--query "Stacks[0].Outputs[?OutputKey=='PublicSecurityGroup'].OutputValue" \
--output text
```

You can get the name of a public subnet in your VPC (the `ECS_CLUSTER_SUBNET` value) by running this command:

```bash
aws cloudformation describe-stacks --stack-name hello-prometheus \
--query "Stacks[0].Outputs[?OutputKey=='PublicSubnet1'].OutputValue" \
--output text
```

Save the file and run it from the command line. You might need to run `chmod +x` on the file to make it executable. In my case, I've named this script `prometheus-install.sh`.

```bash
./prometheus-install.sh
```

This script does the following:

- Creates a new Fargate task using the CloudWatch agent with Prometheus.
- Adds the task to a service in the ECS cluster you created.
- Sets up the appropriate [IAM](https://aws.amazon.com/iam/) roles for this task to work properly.

This script also creates two values in [AWS Systems Manager Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html):

- The value that starts with `AmazonCloudWatch-CWAgentConfig-` contains the configuration for the CloudWatch agent.
- The value that starts with `AmazonCloudWatch-PrometheusConfigName-` contains the Prometheus scraping configuration.

If you review the configurations in Parameter Store, you'll find a few references to the Docker labels that were added to the Fargate task definition. The Prometheus scraper uses those Docker labels to detect and pick up your application—or even new applications you add—and starts sending the metrics it scrapes from these applications to CloudWatch.

After the CloudFormation stack is created, you should see a new task running in your cluster. This task uses the configuration information from the two Parameter Store fields to determine which metrics to scrape from `hello-prometheus-task` and how often.

![Figure 6: Three tasks running in the Fargate cluster](https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2021/02/05/Figure6-2-1024x565.png)

*Figure 6: Three tasks running in the Fargate cluster*

After a few minutes, new metrics appear in [CloudWatch](https://aws.amazon.com/cloudwatch/) under the `ECS/ContainerInsights/Prometheus` namespace. These metrics have been gathered from your application and delivered to CloudWatch through Prometheus. These metrics are like any other metrics you'll find in the CloudWatch console.

If you look at the metrics gathered from Prometheus, you'll find that I am collecting JVM information, which gives me better observability into my Java application. I can use this information to make better decisions about the health of the application.

## Use metrics collected from Prometheus for automatic scaling

I want to scale my application based on connection count. As the number of connections to my application increase, I want to scale out. As the number of connections drop, the application should scale back in. It's simple to use a Prometheus-gathered metric for Fargate scaling actions.

First, choose the metric you want to scale on. I'm going to use the `tomcat_threadpool_connectioncount` metric, which is in [CloudWatch](https://aws.amazon.com/cloudwatch/) under `ECS/ContainerInsights/Prometheus > [ClusterName, TaskDefinitionFamily, port. protocol]`. I want to see how the number of Fargate tasks increase as the scaling rule is applied. To find the number of on-demand Fargate resources, open the CloudWatch console, and then choose **Metrics**. Under **Usage**, choose **By AWS Resource**. Select the `ResourceCount` metric associated with the Fargate Service and OnDemand Resource.

![Figure 7: tomcat_threadpool_connectioncount metric](https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2021/02/05/Figure7-1024x544.png)

*Figure 7: tomcat_threadpool_connectioncount metric*

Fargate scales the number of running tasks based on alarms that fire. Again, for this demo, the application should scale out as the number of connections to the application increases and scale in as the number of connections to the application drops. To set up these alarms, choose the alarm icon (bell) next to the `tomcat_threadpool_connectioncount` metric.

To create the alarm that can be used by Fargate to perform scale-out activities, begin by navigating to the **Alarms** page of the [CloudWatch console](https://console.aws.amazon.com/cloudwatch/):

1. On the **Alarms** page of the CloudWatch console, use all defaults. Select **Greater**. Under **than**, enter `150`. Choose **Next**.
2. Choose **Create a new topic** and enter a name for the topic. I use `hello-prometheus-scaleup`. In **Email endpoints that will receive the notification**, enter your email address. Choose **Create topic**, and then choose **Next**.
3. In the **Alarm name** field enter a name for the alarm. I use `hello-prometheus-scaleup`. Choose **Next**, and then **Create alarm**.

Now create the alarm that can be used by Fargate for scale-in activities by navigating to the **Alarms** page of the CloudWatch console:

1. On the **Alarms** page of the CloudWatch console, choose **Create alarm**.
2. Choose **Select metric**, and then choose **ECS/ContainerInsights/Prometheus > [ClusterName, TaskDefinitionFamily, port, protocol]**.
3. Select `tomcat_threadpool_connectioncount`, and then choose **Select metric**.
4. Use the defaults. Select **Lower**. Under **than**, enter `100`. Choose **Next**.
5. Choose **Create a new topic**, and then enter a topic name. I use `hello-prometheus-scaledown`. In the **Email endpoints that will receive the notification**, enter your email address.
6. Choose **Create topic**, and then choose **Next**.
7. In the **Alarm name** field enter a name for the alarm. I use `hello-prometheus-scaledown`.
8. Choose **Next**, and then choose **Create alarm**.

After a few minutes, both alarms should move from "insufficient data" and enter either an **In alarm** or an **OK** state.

![Figure 8: Alarms page in the CloudWatch console](https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2021/02/05/Figure8-1-1024x306.png)

*Figure 8: Alarms page in the CloudWatch console*

The `hello-prometheus-scaledown` alarm is in the **In alarm** state because there is no traffic on the site currently. Now configure [AWS Fargate](https://aws.amazon.com/fargate/) to scale in whenever this alarm fires, until the minimum number of tasks that should be running is reached. When enough traffic hits the site, the `hello-prometheus-scaleup` alarm should fire, which causes AWS Fargate to scale out the tasks, until the maximum number of tasks that should be running is reached.

To define the AWS Fargate scale-out policy, begin by navigating to the [Amazon ECS console](https://console.aws.amazon.com/ecs/):

1. In the Amazon ECS console, choose the AWS Fargate service that you want to scale, and then choose **Update**. My AWS Fargate service is `hello-prometheus-service`.
2. Choose **Next step**, and then choose **Next step**.
3. On the **Set Auto Scaling (optional)** page, choose **Configure Service Auto Scaling to adjust to your service's desired count**.
4. For **Minimum number of tasks**, enter `2`.
5. For **Maximum number of tasks**, enter `10`.
6. Choose **Add scaling policy**.
7. On the **Add policy** page, change **Scaling policy type** to **Step scaling**.
8. For **Policy name**, I use `hello-prometheus-scaleup-policy`.
9. Choose **Use an existing alarm**, and then choose the scale-out alarm you just created. My alarm is `hello-prometheus-scaleup`.
10. Under **Scaling action**, choose **Add**, and then enter `1`.
11. Choose **Save**.

You have now defined the policy that tells AWS Fargate how to scale out. Repeat the process to define the policy that tells AWS Fargate how to scale in.

1. Choose **Add scaling policy**.
2. On the **Add policy** page, choose **Step scaling**.
3. For **Policy name**, I use `hello-prometheus-scaledown-policy`.
4. Choose **Use an existing alarm**, and then choose the scale-in alarm you created. My alarm is `hello-prometheus-scaledown`.
5. Under **Scaling action**, choose **Remove**, and then enter `1`.
6. Choose **Save**.
7. Choose **Next step**, and then choose **Update Service**.

The cluster is now updated to auto scale based on the custom metrics gathered through Prometheus! After sending traffic to your application, you can see your scale-out and scale-in alarms in action. The blue line on the following chart is the number of connections to the application. The orange line is the number of running tasks. You can see that as traffic increased, the application scaled out, and as traffic decreased, the application slowly scaled itself back in.

![Figure 9: Fargate scaling graph](https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2021/02/05/Figure9-1-1024x685.png)

*Figure 9: Fargate scaling graph*

## Cleanup

To avoid ongoing charges to your AWS account, remove the resources you created.

1. Delete the scale-out and scale-in [CloudWatch](https://aws.amazon.com/cloudwatch/) alarms.
2. Delete all [AWS Fargate](https://aws.amazon.com/fargate/) services and running tasks in the ECS cluster.
3. Use the [AWS CloudFormation](https://aws.amazon.com/cloudformation/) console to delete the two stacks created as part of this demo. Choose each stack, choose **Delete**, and then choose **Delete stack**.

## Cost considerations

With [Amazon CloudWatch Container Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights.html), you pay only for what you use. For a good example of costs, see the [Amazon CloudWatch pricing page](https://aws.amazon.com/cloudwatch/pricing/). You'll find pricing for CloudWatch alarms on the **Alarms** tab.

[AWS Fargate](https://aws.amazon.com/fargate/) pricing is calculated based on the vCPU and memory resources used from the time you start to download your container image until the AWS Fargate task shuts down, rounded up to the nearest second. For current pricing and supported configurations, see the [AWS Fargate pricing page](https://aws.amazon.com/fargate/pricing/).

Although this demo uses a VPC created in [Amazon VPC](https://aws.amazon.com/vpc/), it does not use Amazon VPC features that incur a cost. For current pricing, see the [Amazon VPC pricing page](https://aws.amazon.com/vpc/pricing/).

You are charged for each hour or partial hour that an [Application Load Balancer](https://aws.amazon.com/elasticloadbalancing/application-load-balancer/) is running and the number of Load Balancer Capacity Units (LCU) used per hour. An LCU consists of new connections, active connections, processed bytes, and the number of rules processed by the load balancer. For current pricing and examples, see the [Elastic Load Balancing pricing page](https://aws.amazon.com/elasticloadbalancing/pricing/).

## Conclusion

In this blog post, I showed you how using Prometheus metrics makes it possible to get better insight into container workloads. I showed you how to:

- Set up and deploy a Prometheus-enabled [Amazon ECS](https://aws.amazon.com/ecs/) for [AWS Fargate](https://aws.amazon.com/fargate/) task.
- Deliver metrics from the task to [CloudWatch](https://aws.amazon.com/cloudwatch/).
- Use a custom metric gathered through Prometheus to scale out and scale in an Amazon ECS for AWS Fargate application.

Using the CloudWatch agent with Prometheus, you have a powerful new way to ingest performance metrics from workloads into [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/). Application metrics gathered through the CloudWatch agent with Prometheus provides better visibility into container performance and application health. You can use what you've learned in this blog post to gather custom metrics via Prometheus, and you can use those metrics to configure auto scaling on your [Amazon ECS](https://aws.amazon.com/ecs/) for [AWS Fargate](https://aws.amazon.com/fargate/) applications.

For more information, read about how to [set up and configure Prometheus metrics collection on Amazon ECS clusters](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights-Prometheus-Setup.html).

## About the Author

<table>
  <tr>
    <td width="130" valign="top">
      <img src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2021/08/09/gmike-profile-800x800-2.png" width="120" alt="Mike George" />
    </td>
    <td valign="top">
      <br/>
      <b>Mike George</b>
      <br/><br/>
      <font color="#555555">Mike George is a Principal Solutions Architect based out of Salt Lake City, Utah. He enjoys helping customers solve their technology problems. His interests include software engineering, security, artificial intelligence (AI), and machine learning (ML).</font>
    </td>
  </tr>
</table>