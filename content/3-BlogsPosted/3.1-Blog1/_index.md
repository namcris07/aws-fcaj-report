---
title: "Blog 1"
date: 2021-03-26
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Building a serverless Jenkins environment on AWS Fargate

**by Krithivasan Balasubramaniyan on 26 MAR 2021 in [Amazon Elastic File System (EFS)](https://aws.amazon.com/vi/blogs/devops/category/storage/amazon-elastic-file-system-efs/), [AWS Fargate](https://aws.amazon.com/vi/blogs/devops/category/compute/aws-fargate/)**

---

Jenkins is a popular open-source automation server that enables developers around the world to reliably build, test, and deploy their software. Jenkins uses a controller-agent architecture in which the controller is responsible for serving the web UI, stores the configurations and related data on disk, and delegates the jobs to the worker agents that run these jobs as their primary responsibility.

[Amazon Elastic Container Service (Amazon ECS)](https://aws.amazon.com/ecs/) is a fully managed container orchestration service, and has been a popular choice to deploy a highly available, fault tolerant, and scalable Jenkins environment. Traditionally, the [Amazon Elastic Compute Cloud (Amazon EC2)](https://aws.amazon.com/ec2/) launch type was the preferred choice, with [Amazon Elastic Block Store (Amazon EBS)](https://aws.amazon.com/ebs/) as the storage for the configurations and data. That changed with the introduction of [AWS Fargate](https://aws.amazon.com/fargate/) support for [Amazon EFS](https://aws.amazon.com/efs/).

The objective of this post is to walk you through how to set up a completely serverless Jenkins environment on [AWS Fargate](https://aws.amazon.com/fargate/) using [Terraform](https://www.terraform.io/).

## Overview of the solution

The following diagram illustrates the solution architecture.

![This diagram depicts the deployment architecture for Jenkins on AWS Fargate](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/03/24/Jenkins.jpg)

The objective is to create a fully automated deployment of highly available, production-ready Jenkins in a serverless environment on AWS. We use the following components and services:

- The Jenkins controller URL is backed by an [Application Load Balancer](https://aws.amazon.com/elasticloadbalancing/application-load-balancer/). We recommend using [AWS Certificate Manager (ACM)](https://aws.amazon.com/certificate-manager/) to provision an SSL certificate to associate with the load balancer. The SSL termination happens at the load balancer.
- We use a [VPC](https://aws.amazon.com/vpc/) with at least two public and two private subnets.
- The Jenkins controller runs as a service in [Amazon ECS](https://aws.amazon.com/ecs/) using Fargate as the launch type. We use [Amazon Elastic File System (Amazon EFS)](https://aws.amazon.com/efs/) as the persistent backing store for the Jenkins controller task. The Jenkins controller and Amazon EFS are launched in private subnets.
- Jenkins uses the [Amazon ECS Fargate plugin](https://plugins.jenkins.io/amazon-ecs/) to delegate to Amazon ECS to run the builds on Docker-based agents. Each Jenkins build is run on a dedicated Docker container that is wiped out at the end of the build. Jenkins agents discover the Jenkins controller task using [AWS Cloud Map](https://aws.amazon.com/cloud-map/) for service discovery.
- You can use the aforementioned plugin to create multiple definitions of the Amazon EC2 Container Service Cloud configuration. This allows you to use an ECS cluster with a Fargate Spot capacity provider to run Jenkins jobs that can tolerate interruptions. This provides cost optimizations to run, for example, large test suites and build jobs.
- The Jenkins controller can assume [AWS Identity and Access Management (IAM)](https://aws.amazon.com/iam/) roles based on the ARN provided with the Fargate task definition. You can define multiple ECS agent templates within the ECS plugin with different permission scopes for leveraging security best practices.
- [AWS Backup](https://aws.amazon.com/backup/) is used for assuring that you can restore all important data from the Jenkins controller in case of an accident.
- You can additionally configure user notifications using [Amazon Simple Notification Service (Amazon SNS)](https://aws.amazon.com/sns/) to alert users of any failures in the build jobs or in the environment.

## Prerequisites

For this post, we developed a Terraform module to perform the deployment. The module and deployment example scripts are available in the [GitHub repo](https://github.com/aws-samples/jenkins-ecs-agents). Refer the README for a list of all module variables. The following are required to deploy this Terraform module:

- [Terraform](https://www.terraform.io/downloads.html) 13+.
- A [VPC](https://aws.amazon.com/vpc/) with at least two public and two private subnets.
- An SSL certificate to associate with the Application Load Balancer. It's recommended to use an [ACM certificate](https://aws.amazon.com/certificate-manager/). This is not done by the main Terraform module. However, the example in the example directory uses the public ACM module to create the ACM certificate and pass it to the serverless Jenkins module. You can do it this way or explicitly pass the ARN of a certificate that you previously created or imported into ACM.
- An admin password for Jenkins must be stored in [AWS Systems Manager Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html). This parameter must be of type SecureString and have the name `jenkins-pwd`.
- Terraform must be bootstrapped. This means that a state [Amazon Simple Storage Service (Amazon S3)](https://aws.amazon.com/s3/) bucket and a state locking [Amazon DynamoDB](https://aws.amazon.com/dynamodb/) table must be initialized.

## Deployment

All the required resources and configurations are packaged as a Terraform module, which means it's not directly deployable. However, an example deployment is in the example directory. To deploy the example, complete the following steps:

1. Clone the [GitHub repo](https://github.com/aws-samples/jenkins-ecs-agents).
2. Change your working directory to the `bootstrap` directory.

   Included in this directory is sample Terraform code to bootstrap the initial Terraform state management resources. A best practice is to use a state backend such as [Amazon S3](https://aws.amazon.com/s3/) and a locking mechanism such as [DynamoDB](https://aws.amazon.com/dynamodb/) when using Terraform. For more information, see [State Storage and Locking](https://www.terraform.io/language/state/locking). Because this bootstrap code creates Terraform state management resources, special care must be taken to save the resultant Terraform state file. Be aware that this state is only saved to a local file named `terraform.tfstate`. Make sure to save this state file if you want to maintain the S3 state bucket and DynamoDB lock table using Terraform.

   Replace `my-state-bucket` and `my-lock-table` with your preferred names:

   ```bash
   terraform init
   terraform apply \
       -var="state_bucket_name=my-state-bucket" \
       -var="state_lock_table_name=my-lock-table"
   ```

3. To deploy the module, change your working directory to `example`.
4. Copy `vars.sh.example` to `vars.sh`.
5. Edit the variables in `vars.sh` as necessary, giving all details specific to your environment (VPC, subnets, [Route 53](https://aws.amazon.com/route53/) zone ID and domain name, state bucket, state locking table, and the Region in which you intend to deploy).
6. Run `deploy_example.sh`.

After you deploy all the resources, you can open a browser of your choice and enter the [Route 53](https://aws.amazon.com/route53/) alias record name. This opens the Jenkins web UI. Sign in with the user `ecsuser` and the password stored in the parameter store. All the plugins mentioned in the `plugins.txt` are also installed, and the Fargate container service cloud configurations are applied. Two clouds are created: one for Fargate and the other for the Fargate Spot cluster. The following screenshot shows the Fargate cloud.

![This picture depicts the Amazon EC2 container service cloud configuration for the Amazon Elastic Container Service (ECS) / Fargate plugin.](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/03/23/Screen-Shot-2020-11-29-at-1.37.17-PM.png)

You're presented with two preconfigured jobs.

![This picture depicts the two pre-provisioned Jenkins jobs. One of them would schedule jobs on a standard FARGATE cluster and the other would schedule jobs on a cluster with FARGATE_SPOT capacity provider.](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/03/23/Screen-Shot-2020-11-29-at-1.16.46-PM.png)

Before we dive into those jobs, let's take a look at capacity providers.

[Amazon ECS](https://aws.amazon.com/ecs/) on Fargate capacity providers enable you to use both Fargate and Fargate Spot capacity with your Amazon ECS tasks. For more information about capacity providers, see [Amazon ECS capacity providers](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cluster-capacity-providers.html). With Fargate Spot, you can run interruption tolerant Amazon ECS tasks at a discounted rate compared to the Fargate price. Fargate Spot runs tasks on spare compute capacity. When AWS needs the capacity back, your tasks are interrupted with a 2-minute warning.

As part of the module that we just deployed, two ECS Fargate clusters are provisioned: one with a `FARGATE` capacity provider and the other with a `FARGATE_SPOT` capacity provider. The Jenkins controller service is configured to run on the cluster with the `FARGATE` capacity provider.

The two ECS container service cloud configurations created earlier determine which agent the jobs run on. For example, you can configure all the workloads that can tolerate interruptions (such as build jobs and test suites) to run on the `fargate-cloud-spot`, which is backed by an ECS Fargate cluster with the `FARGATE_SPOT` capacity provider, and all the other workloads (such as Terraform apply runs) to run on the `fargate-cloud`, which is backed by an ECS Fargate cluster with the `FARGATE` capacity provider. The Fargate Spot capacity provider helps optimize costs by using the spare compute capacity in the AWS Cloud. We call the jobs scheduled to run on the Spot capacity provider non-critical tasks and those running on the Fargate capacity provider critical tasks due to the nature of those jobs. The Jenkins declarative pipeline for the critical task job looks like the following code:

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

## Test the solution

To test the solution, let's first run the **Simple Job Critical Task** by choosing **Build Now**. This creates a task in the cluster with the `FARGATE` capacity provider.

![The picture depicts that the Jenkins job run creates a task in the cluster with FARGATE capacity provider.](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/03/23/Screen-Shot-2020-11-29-at-1.55.37-PM.png)

When the task is in Running state, it registers itself as an agent and the Jenkins job runs on this agent.

![This picture depicts that the newly created ECS task registers itself as a Jenkins agent.](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/03/23/Screen-Shot-2020-11-29-at-2.00.43-PM.png)

You can see the console output in Jenkins.

![This picture depicts the Jenkins job console output.](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/03/23/Screen-Shot-2020-11-29-at-2.02.25-PM.png)

When the job is successful, the task is stopped.

![This picture depicts that the ECS task is stopped once the Jenkins job is finished.](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/03/24/Screen-Shot-2020-11-29-at-2.05.01-PM-1.png)

## Cleanup

In order to clean up all the resources that were created to build the serverless Jenkins environment, navigate to the `example` directory and run the following command:

```bash
terraform destroy -auto-approve
```

In order to clean up the DynamoDB lock table and the S3 state bucket, navigate to the `example/bootstrap` directory and run the following command:

```bash
terraform destroy -auto-approve
```

## Limitations

Because [Amazon EFS](https://aws.amazon.com/efs/) is network attached storage, in its default configuration it may not be performant enough for some Jenkins use cases. Proper research and testing is advised before using Amazon EFS for productive workloads. For more information, see [Amazon EFS performance](https://docs.aws.amazon.com/efs/latest/ug/performance.html).

If necessary, set Amazon EFS to maximum I/O mode. You can do this in the Terraform module by setting the variable `efs_performance_mode` to `maxIO`. For more information, see [What are the differences between General Purpose and Max I/O performance modes in Amazon EFS?](https://aws.amazon.com/premiumsupport/knowledge-center/linux-efs-performance-modes/)

## Conclusion

In this post, we went through how to deploy a highly available, scalable, fault tolerant, production-ready Jenkins environment in a completely serverless environment on [Fargate](https://aws.amazon.com/fargate/). We also showed how to use the Terraform module to automate the deployment of all the necessary resources and associated configurations. The post also introduced how to use `FARGATE` and `FARGATE_SPOT` capacity providers.

When we deploy each job in its own Docker container, we can control the CPU and memory requirements for each job, the base container that defines the environment, the tooling necessary for the job, and more importantly not load the controller with job runs.

## About the Authors

<table>
  <tr>
    <td width="130" valign="top">
      <img src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2021/03/16/Krithivasan-Balasubramaniyan-Author.jpg" width="120" alt="Krithivasan Balasubramaniyan" />
    </td>
    <td valign="top">
      <br/>
      <b>Krithivasan Balasubramaniyan</b>
      <br/><br/>
      <font color="#555555">Krithivasan is a Senior Consultant at AWS. He enables global enterprise customers in their digital transformation journey and helps architect cloud native solutions.</font>
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
      <font color="#555555">Jason is a Senior Cloud Architect at AWS and loves all things DevOps. In his role at AWS he guides enterprise customers through their cloud transformations.</font>
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
      <font color="#555555">Alexander is a Global DevOps Architect in AWS ProServe with more than 8 years of experience in virtualization, cloud technologies and automation.</font>
    </td>
  </tr>
</table>