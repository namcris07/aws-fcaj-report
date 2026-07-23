---
title: "Blog 2"
date: 2021-06-25
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# Building an end-to-end Kubernetes-based DevSecOps software factory on AWS

**by Srinivas Manepalli on 25 JUN 2021 in [Architecture](https://aws.amazon.com/blogs/devops/category/architecture/), [AWS CodeBuild](https://aws.amazon.com/blogs/devops/category/developer-tools/aws-codebuild/), [AWS CodeDeploy](https://aws.amazon.com/blogs/devops/category/developer-tools/aws-codedeploy/), [AWS CodePipeline](https://aws.amazon.com/blogs/devops/category/developer-tools/aws-codepipeline/), [AWS Security Hub](https://aws.amazon.com/blogs/devops/category/security-identity-compliance/aws-security-hub/), [Compliance](https://aws.amazon.com/blogs/devops/category/compliance/), [DevOps](https://aws.amazon.com/blogs/devops/category/devops/), Public Sector, Security**

---

DevSecOps software factory implementation can significantly vary depending on the application, infrastructure, architecture, and the services and tools used. In a previous post, I provided an end-to-end DevSecOps pipeline for a three-tier web application deployed with [AWS Elastic Beanstalk](https://aws.amazon.com/elasticbeanstalk/). The pipeline used cloud-native services along with a few open-source security tools. This solution is similar, but instead uses a containers-based approach with additional security analysis stages. It defines a software factory using Kubernetes along with necessary AWS Cloud-native services and open-source third-party tools. Code is provided in the [GitHub repo](https://github.com/aws-samples/aws-devsecops-workshop) to build this DevSecOps software factory, including the integration code for third-party scanning tools.

DevOps is a combination of cultural philosophies, practices, and tools that combine software development with information technology operations. These combined practices enable companies to deliver new application features and improved services to customers at a higher velocity. DevSecOps takes this a step further by integrating and automating the enforcement of preventive, detective, and responsive security controls into the pipeline.

In a DevSecOps factory, security needs to be addressed from two aspects: security of the software factory, and security in the software factory. In this architecture, we use AWS services to address the security of the software factory, and use third-party tools along with AWS services to address the security in the software factory. This AWS DevSecOps reference architecture covers DevSecOps practices and security vulnerability scanning stages including secret analysis, SCA (Software Composite Analysis), SAST (Static Application Security Testing), DAST (Dynamic Application Security Testing), RASP (Runtime Application Self Protection), and aggregation of vulnerability findings into a single pane of glass.

The focus of this post is on application vulnerability scanning. Vulnerability scanning of underlying infrastructure such as the [Amazon Elastic Kubernetes Service (Amazon EKS)](https://aws.amazon.com/eks/) cluster and network is outside the scope of this post. For information about infrastructure-level security planning, refer to [Amazon GuardDuty](https://aws.amazon.com/guardduty/), [Amazon Inspector](https://aws.amazon.com/inspector/), and [AWS Shield](https://aws.amazon.com/shield/).

You can deploy this pipeline in either the AWS GovCloud (US) Region or standard AWS Regions. All listed AWS services are authorized for FedRamp High and DoD SRG IL4/IL5.

## Security and compliance

Thoroughly implementing security and compliance in the public sector and other highly regulated workloads is very important for achieving an ATO (Authority to Operate) and continuously maintain an ATO (c-ATO). DevSecOps shifts security left in the process, integrating it at each stage of the software factory, which can make ATO a continuous and faster process. With DevSecOps, an organization can deliver secure and compliant application changes rapidly while running operations consistently with automation.

Security and compliance are shared responsibilities between AWS and the customer. Depending on the compliance requirements (such as FedRamp or DoD SRG), a DevSecOps software factory needs to implement certain security controls. AWS provides tools and services to implement most of these controls. For example, to address NIST 800-53 security controls families such as access control, you can use [AWS Identity Access and Management (IAM)](https://aws.amazon.com/iam/) roles and [Amazon Simple Storage Service (Amazon S3)](https://aws.amazon.com/s3/) bucket policies. To address auditing and accountability, you can use [AWS CloudTrail](https://aws.amazon.com/cloudtrail/) and [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/). To address configuration management, you can use [AWS Config](https://aws.amazon.com/config/) rules and [AWS Systems Manager](https://aws.amazon.com/systems-manager/). Similarly, to address risk assessment, you can use vulnerability scanning tools from AWS.

The following table is the high-level mapping of the NIST 800-53 security control families and AWS services that are used in this DevSecOps reference architecture. This list only includes the services that are defined in the [AWS CloudFormation](https://aws.amazon.com/cloudformation/) template, which provides pipeline as code in this solution. You can use additional AWS services and tools or other environmental specific services and tools to address these and the remaining security control families on a more granular level.

| # | NIST 800-53 Security Control Family – Rev 5 | AWS Services Used (In this DevSecOps Pipeline) |
|---|---|---|
| 1 | AC – Access Control | AWS IAM, Amazon S3, and Amazon CloudWatch are used. `AWS::IAM::ManagedPolicy` `AWS::IAM::Role` `AWS::S3::BucketPolicy` `AWS::CloudWatch::Alarm` |
| 2 | AU – Audit and Accountability | AWS CloudTrail, Amazon S3, Amazon SNS, and Amazon CloudWatch are used. `AWS::CloudTrail::Trail` `AWS::Events::Rule` `AWS::CloudWatch::LogGroup` `AWS::CloudWatch::Alarm` `AWS::SNS::Topic` |
| 3 | CM – Configuration Management | AWS Systems Manager, Amazon S3, and AWS Config are used. `AWS::SSM::Parameter` `AWS::S3::Bucket` `AWS::Config::ConfigRule` |
| 4 | CP – Contingency Planning | AWS CodeCommit and Amazon S3 are used. `AWS::CodeCommit::Repository` `AWS::S3::Bucket` |
| 5 | IA – Identification and Authentication | AWS IAM is used. `AWS:IAM:User` `AWS::IAM::Role` |
| 6 | RA – Risk Assessment | AWS Config, AWS CloudTrail, AWS Security Hub, and third party scanning tools are used. `AWS::Config::ConfigRule` `AWS::CloudTrail::Trail` `AWS::SecurityHub::Hub` Vulnerability Scanning Tools |
| 7 | CA – Assessment, Authorization, and Monitoring | AWS CloudTrail, Amazon CloudWatch, and AWS Config are used. `AWS::CloudTrail::Trail` `AWS::CloudWatch::LogGroup` `AWS::CloudWatch::Alarm` `AWS::Config::ConfigRule` |
| 8 | SC – System and Communications Protection | AWS KMS and AWS Systems Manager are used. `AWS::KMS::Key` `AWS::SSM::Parameter` SSL/TLS communication |
| 9 | SI – System and Information Integrity | AWS Security Hub, and third party scanning tools are used. `AWS::SecurityHub::Hub` Vulnerability Scanning Tools |
| 10 | AT – Awareness and Training | N/A |
| 11 | SA – System and Services Acquisition | N/A |
| 12 | IR – Incident Response | Not implemented, but services like [AWS Lambda](https://aws.amazon.com/lambda/) and [Amazon CloudWatch Events](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/WhatIsCloudWatchEvents.html) can be used. |
| 13–20 | MA, MP, PS, PE, PL, PM, PT, SR | N/A |

## Services and tools

In this section, we discuss the various AWS services and third-party tools used in this solution.

### CI/CD services

For continuous integration and continuous delivery (CI/CD) in this reference architecture, we use the following AWS services:

- [**AWS CodeBuild**](https://aws.amazon.com/codebuild/) – A fully managed continuous integration service that compiles source code, runs tests, and produces software packages that are ready to deploy.
- [**AWS CodeCommit**](https://aws.amazon.com/codecommit/) – A fully managed source control service that hosts secure Git-based repositories.
- [**AWS CodeDeploy**](https://aws.amazon.com/codedeploy/) – A fully managed deployment service that automates software deployments to a variety of compute services such as [Amazon Elastic Compute Cloud (Amazon EC2)](https://aws.amazon.com/ec2/), [AWS Fargate](https://aws.amazon.com/fargate/), [AWS Lambda](https://aws.amazon.com/lambda/), and your on-premises servers.
- [**AWS CodePipeline**](https://aws.amazon.com/codepipeline/) – A fully managed continuous delivery service that helps you automate your release pipelines for fast and reliable application and infrastructure updates.
- [**AWS Lambda**](https://aws.amazon.com/lambda/) – A service that lets you run code without provisioning or managing servers. You pay only for the compute time you consume.
- [**Amazon Simple Notification Service (Amazon SNS)**](https://aws.amazon.com/sns/) – A fully managed messaging service for both application-to-application (A2A) and application-to-person (A2P) communication.
- [**Amazon S3**](https://aws.amazon.com/s3/) – Storage for the internet. You can use Amazon S3 to store and retrieve any amount of data at any time, from anywhere on the web.
- [**AWS Systems Manager Parameter Store**](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html) – Provides secure, hierarchical storage for configuration data management and secrets management.

### Continuous testing tools

The following are open-source scanning tools that are integrated in the pipeline for the purpose of this post, but you could integrate other tools that meet your specific requirements. You can use the static code review tool [Amazon CodeGuru](https://aws.amazon.com/codeguru/) for static analysis, but at the time of this writing, it's not yet available in AWS GovCloud and currently supports Java and Python.

- **Anchore (SCA and SAST)** – Anchore Engine is an open-source software system that provides a centralized service for analyzing container images, scanning for security vulnerabilities, and enforcing deployment policies.
- [**Amazon Elastic Container Registry image scanning**](https://docs.aws.amazon.com/AmazonECR/latest/userguide/image-scanning.html) – Amazon ECR image scanning helps in identifying software vulnerabilities in your container images. Amazon ECR uses the Common Vulnerabilities and Exposures (CVEs) database from the open-source Clair project and provides a list of scan findings.
- **Git-Secrets (Secrets Scanning)** – Prevents you from committing sensitive information to Git repositories. It is an open-source tool from AWS Labs.
- **OWASP ZAP (DAST)** – Helps you automatically find security vulnerabilities in your web applications while you're developing and testing your applications.
- **Snyk (SCA and SAST)** – Snyk is an open-source security platform designed to help software-driven businesses enhance developer security.
- **Sysdig Falco (RASP)** – Falco is an open source cloud-native runtime security project that detects unexpected application behavior and alerts on threats at runtime. It is the first runtime security project to join CNCF as an incubation-level project.

You can integrate additional security stages like IAST (Interactive Application Security Testing) into the pipeline to get code insights while the application is running. You can use AWS partner tools like Contrast Security, Synopsys, and WhiteSource to integrate IAST scanning into the pipeline. Malware scanning tools, and image signing tools can also be integrated into the pipeline for additional security.

### Continuous logging and monitoring services

The following are AWS services for continuous logging and monitoring used in this reference architecture:

- [**Amazon CloudWatch Events**](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/WhatIsCloudWatchEvents.html) – Delivers a near-real-time stream of system events that describe changes in AWS resources.
- [**Amazon CloudWatch Logs**](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html) – Allows you to monitor, store, and access your log files from EC2 instances, CloudTrail, Amazon Route 53, and other sources.

### Auditing and governance services

The following are AWS auditing and governance services used in this reference architecture:

- [**AWS CloudTrail**](https://aws.amazon.com/cloudtrail/) – Enables governance, compliance, operational auditing, and risk auditing of your AWS account.
- [**AWS Config**](https://aws.amazon.com/config/) – Allows you to assess, audit, and evaluate the configurations of your AWS resources.
- [**AWS Identity and Access Management**](https://aws.amazon.com/iam/) – Enables you to manage access to AWS services and resources securely. With IAM, you can create and manage AWS users and groups, and use permissions to allow and deny their access to AWS resources.

### Operations services

The following are the AWS operations services used in this reference architecture:

- [**AWS CloudFormation**](https://aws.amazon.com/cloudformation/) – Gives you an easy way to model a collection of related AWS and third-party resources, provision them quickly and consistently, and manage them throughout their lifecycles, by treating infrastructure as code.
- [**Amazon ECR**](https://aws.amazon.com/ecr/) – A fully managed container registry that makes it easy to store, manage, share, and deploy your container images and artifacts anywhere.
- [**Amazon EKS**](https://aws.amazon.com/eks/) – A managed service that you can use to run Kubernetes on AWS without needing to install, operate, and maintain your own Kubernetes control plane or nodes. Amazon EKS runs up-to-date versions of the open-source Kubernetes software, so you can use all of the existing plugins and tooling from the Kubernetes community.
- [**AWS Security Hub**](https://aws.amazon.com/security-hub/) – Gives you a comprehensive view of your security alerts and security posture across your AWS accounts. This post uses Security Hub to aggregate all the vulnerability findings as a single pane of glass.
- [**AWS Systems Manager Parameter Store**](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html) – Provides secure, hierarchical storage for configuration data management and secrets management. You can store data such as passwords, database strings, Amazon Machine Image (AMI) IDs, and license codes as parameter values.

## Pipeline architecture

The following diagram shows the architecture of the solution. We use [AWS CloudFormation](https://aws.amazon.com/cloudformation/) to describe the pipeline as code.

![Kubernetes DevSecOps Pipeline Architecture](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/06/25/containers-pipeline-architecture-blog-v2-1.png)

*Kubernetes DevSecOps Pipeline Architecture*

The main steps are as follows:

1. When a user commits the code to [CodeCommit](https://aws.amazon.com/codecommit/) repository, a [CloudWatch](https://aws.amazon.com/cloudwatch/) event is generated, which triggers [CodePipeline](https://aws.amazon.com/codepipeline/) to orchestrate the events.
2. [CodeBuild](https://aws.amazon.com/codebuild/) packages the build and uploads the artifacts to an [S3](https://aws.amazon.com/s3/) bucket.
3. CodeBuild scans the code with **git-secrets**. If there is any sensitive information in the code such as AWS access keys or secrets keys, CodeBuild fails the build.
4. CodeBuild creates the container image and perform SCA and SAST by scanning the image with **Snyk** or **Anchore**. In the provided CloudFormation template, you can pick one of these tools during the deployment. Please note, CodeBuild is fully enabled for a "bring your own tool" approach.
   - **(4a)** If there are any vulnerabilities, CodeBuild invokes the [Lambda](https://aws.amazon.com/lambda/) function. The function parses the results into AWS Security Finding Format (ASFF) and posts them to [Security Hub](https://aws.amazon.com/security-hub/). Security Hub helps aggregate and view all the vulnerability findings in one place as a single pane of glass. The Lambda function also uploads the scanning results to an S3 bucket.
   - **(4b)** If there are no vulnerabilities, CodeBuild pushes the container image to [Amazon ECR](https://aws.amazon.com/ecr/) and triggers another scan using built-in Amazon ECR scanning.
5. CodeBuild retrieves the scanning results.
   - **(5a)** If there are any vulnerabilities, CodeBuild invokes the Lambda function again and posts the findings to Security Hub. The Lambda function also uploads the scan results to an S3 bucket.
   - **(5b)** If there are no vulnerabilities, CodeBuild deploys the container image to an [Amazon EKS](https://aws.amazon.com/eks/) staging environment.
6. After the deployment succeeds, CodeBuild triggers the DAST scanning with the **OWASP ZAP** tool (again, this is fully enabled for a "bring your own tool" approach).
   - **(6a)** If there are any vulnerabilities, CodeBuild invokes the Lambda function, which parses the results into ASFF and posts it to Security Hub. The function also uploads the scan results to an S3 bucket (similar to step 4a).
7. If there are no vulnerabilities, the approval stage is triggered, and an email is sent to the approver for action via [Amazon SNS](https://aws.amazon.com/sns/).
8. After approval, CodeBuild deploys the code to the production Amazon EKS environment.
9. During the pipeline run, [CloudWatch Events](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/WhatIsCloudWatchEvents.html) captures the build state changes and sends email notifications to subscribed users through Amazon SNS.
10. [CloudTrail](https://aws.amazon.com/cloudtrail/) tracks the API calls and sends notifications on critical events on the pipeline and CodeBuild projects, such as `UpdatePipeline`, `DeletePipeline`, `CreateProject`, and `DeleteProject`, for auditing purposes.
11. [AWS Config](https://aws.amazon.com/config/) tracks all the configuration changes of AWS services. The following AWS Config rules are added in this pipeline as security best practices:
    - `CODEBUILD_PROJECT_ENVVAR_AWSCRED_CHECK` – Checks whether the project contains environment variables `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`. The rule is NON_COMPLIANT when the project environment variables contain plaintext credentials. This rule ensures that sensitive information isn't stored in the CodeBuild project environment variables.
    - `CLOUD_TRAIL_LOG_FILE_VALIDATION_ENABLED` – Checks whether CloudTrail creates a signed digest file with logs. AWS recommends that the file validation be enabled on all trails. The rule is noncompliant if the validation is not enabled. This rule ensures that pipeline resources such as the CodeBuild project aren't altered to bypass critical vulnerability checks.

Security of the pipeline is implemented using [IAM](https://aws.amazon.com/iam/) roles and S3 bucket policies to restrict access to pipeline resources. Pipeline data at rest and in transit is protected using encryption and SSL secure transport. We use [Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html) to store sensitive information such as API tokens and passwords. To be fully compliant with frameworks such as FedRAMP, other things may be required, such as MFA.

Security in the pipeline is implemented by performing the Secret Analysis, SCA, SAST, DAST, and RASP security checks. Applicable AWS services provide encryption at rest and in transit by default. You can enable additional controls on top of these wherever required.

In the next section, I explain how to deploy and run the pipeline CloudFormation template used for this example. As a best practice, we recommend using linting tools like `cfn-nag` and `cfn-guard` to scan CloudFormation templates for security vulnerabilities. Refer to the provided service links to learn more about each of the services in the pipeline.

## Prerequisites

Before getting started, make sure you have the following prerequisites:

- An [EKS cluster](https://aws.amazon.com/eks/) environment with your application deployed. In this post, we use PHP WordPress as a sample application, but you can use any other application.
- [Sysdig Falco](https://falco.org/) installed on an EKS cluster. Sysdig Falco captures events on the EKS cluster and sends those events to [CloudWatch](https://aws.amazon.com/cloudwatch/) using [AWS FireLens](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/using_firelens.html). For implementation instructions, see [Implementing Runtime security in Amazon EKS using CNCF Falco](https://aws.amazon.com/blogs/containers/implementing-runtime-security-in-amazon-eks-using-cncf-falco/). This step is required only if you need to implement RASP in the software factory.
- A [CodeCommit](https://aws.amazon.com/codecommit/) repo with your application code and a Dockerfile. For more information, see [Create an AWS CodeCommit repository](https://docs.aws.amazon.com/codecommit/latest/userguide/how-to-create-repository.html).
- An [Amazon ECR](https://aws.amazon.com/ecr/) repo to store container images and scan for vulnerabilities. Enable vulnerability scanning on image push in Amazon ECR.
- The provided `buildspec-*.yml` files for git-secrets, Anchore, Snyk, Amazon ECR, OWASP ZAP, and your Kubernetes deployment `.yml` files uploaded to the root of the application code repository.
- A Snyk API key if you use Snyk as a SAST tool.
- The Lambda function uploaded to an [S3](https://aws.amazon.com/s3/) bucket. We use this function to parse the scan reports and post the results to [Security Hub](https://aws.amazon.com/security-hub/).
- An OWASP ZAP URL and generated API key for dynamic web scanning.
- An application web URL to run the DAST testing.
- An email address to receive approval notifications for deployment, pipeline change notifications, and CloudTrail events.
- [AWS Config](https://aws.amazon.com/config/) and [Security Hub](https://aws.amazon.com/security-hub/) services enabled. For instructions, see [Managing the Configuration Recorder](https://docs.aws.amazon.com/config/latest/developerguide/stop-start-recorder.html) and [Enabling Security Hub manually](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-enable.html), respectively.

## Deploying the pipeline

To deploy the pipeline, complete the following steps:

1. Download the [CloudFormation](https://aws.amazon.com/cloudformation/) template and pipeline code from the [GitHub repo](https://github.com/aws-samples/aws-devsecops-workshop).
2. Sign in to your AWS account if you have not done so already.
3. On the [CloudFormation console](https://console.aws.amazon.com/cloudformation/), choose **Create Stack**.
4. Choose the CloudFormation pipeline template.
5. Choose **Next**.
6. Under **Code**, provide the following information:
   - Code details, such as repository name and the branch to trigger the pipeline.
   - The Amazon ECR container image repository name.
7. Under **SAST**, provide the following information:
   - Choose the SAST tool (Anchore or Snyk) for code analysis.
   - If you select Snyk, provide an API key for Snyk.
8. Under **DAST**, choose the DAST tool (OWASP ZAP) for dynamic testing and enter the API token, DAST tool URL, and the application URL to run the scan.
9. Under **Lambda functions**, enter the Lambda function S3 bucket name, filename, and the handler name.
10. For **STG EKS cluster**, enter the staging EKS cluster name.
11. For **PRD EKS cluster**, enter the production EKS cluster name to which this pipeline deploys the container image.
12. Under **General**, enter the email addresses to receive notifications for approvals and pipeline status changes.
13. Choose **Next**.
14. Complete the stack.
15. After the pipeline is deployed, confirm the subscription by choosing the provided link in the email to receive notifications.

![Pipeline CloudFormation Parameters](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/06/21/Pipeline-CF-Parameters-1.jpg)

*Pipeline CloudFormation Parameters*

> **Note:** The provided CloudFormation template in this post is formatted for AWS GovCloud. If you're setting this up in a standard Region, you have to adjust the partition name in the CloudFormation template. For example, change ARN values from `arn:aws-us-gov` to `arn:aws`.

## Running the pipeline

To trigger the pipeline, commit changes to your application repository files. That generates a [CloudWatch](https://aws.amazon.com/cloudwatch/) event and triggers the pipeline. [CodeBuild](https://aws.amazon.com/codebuild/) scans the code and if there are any vulnerabilities, it invokes the [Lambda](https://aws.amazon.com/lambda/) function to parse and post the results to [Security Hub](https://aws.amazon.com/security-hub/).

When posting the vulnerability finding information to Security Hub, we need to provide a vulnerability severity level. Based on the provided severity value, Security Hub assigns the label as follows. Adjust the severity levels in your code based on your organization's requirements.

| Value | Label |
|-------|-------|
| 0 | INFORMATIONAL |
| 1–39 | LOW |
| 40–69 | MEDIUM |
| 70–89 | HIGH |
| 90–100 | CRITICAL |

The following screenshot shows the progression of your pipeline.

![DevSecOps Kubernetes CI/CD Pipeline](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/06/18/complete-pipeline.png)

*DevSecOps Kubernetes CI/CD Pipeline*

## Secrets analysis scanning

In this architecture, after the pipeline is initiated, [CodeBuild](https://aws.amazon.com/codebuild/) triggers the Secret Analysis stage using **git-secrets** and the `buildspec-gitsecrets.yml` file. Git-Secrets looks for any sensitive information such as AWS access keys and secret access keys. Git-Secrets allows you to add custom strings to look for in your analysis. CodeBuild uses the provided `buildspec-gitsecrets.yml` file during the build stage.

## SCA and SAST scanning

In this architecture, [CodeBuild](https://aws.amazon.com/codebuild/) triggers the SCA and SAST scanning using **Anchore**, **Snyk**, and **Amazon ECR**. In this solution, we use the open-source versions of Anchore and Snyk. [Amazon ECR](https://aws.amazon.com/ecr/) uses open-source Clair under the hood, which comes with Amazon ECR for no additional cost. As mentioned earlier, you can choose Anchore or Snyk to do the initial image scanning.

### Scanning with Anchore

If you choose Anchore as a SAST tool during the deployment, the build stage uses the `buildspec-anchore.yml` file to scan the container image. If there are any vulnerabilities, it fails the build and triggers the [Lambda](https://aws.amazon.com/lambda/) function to post those findings to [Security Hub](https://aws.amazon.com/security-hub/). If there are no vulnerabilities, it proceeds to next stage.

![Anchore Lambda Code Snippet](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/06/18/Anchore-lambda-codesnippet-1024x693.png)

*Anchore Lambda Code Snippet*

### Scanning with Snyk

If you choose Snyk as a SAST tool during the deployment, the build stage uses the `buildspec-snyk.yml` file to scan the container image. If there are any vulnerabilities, it fails the build and triggers the [Lambda](https://aws.amazon.com/lambda/) function to post those findings to [Security Hub](https://aws.amazon.com/security-hub/). If there are no vulnerabilities, it proceeds to next stage.

![Snyk Lambda Code Snippet](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/06/18/Snyk-lambda-codesnippet-1024x729.png)

*Snyk Lambda Code Snippet*

### Scanning with Amazon ECR

If there are no vulnerabilities from Anchore or Snyk scanning, the image is pushed to [Amazon ECR](https://aws.amazon.com/ecr/), and the Amazon ECR scan is triggered automatically. Amazon ECR lists the vulnerability findings on the Amazon ECR console. To provide a single pane of glass view of all the vulnerability findings and for easy administration, we retrieve those findings and post them to [Security Hub](https://aws.amazon.com/security-hub/). If there are no vulnerabilities, the image is deployed to the EKS staging cluster and next stage (DAST scanning) is triggered.

![ECR Lambda Code Snippet](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/06/18/ECR-lambda-codesnippet-1024x661.png)

*ECR Lambda Code Snippet*

## DAST scanning with OWASP ZAP

In this architecture, [CodeBuild](https://aws.amazon.com/codebuild/) triggers DAST scanning using the DAST tool **OWASP ZAP**.

After deployment is successful, CodeBuild initiates the DAST scanning. When scanning is complete, if there are any vulnerabilities, it invokes the [Lambda](https://aws.amazon.com/lambda/) function, similar to SAST analysis. The function parses and posts the results to [Security Hub](https://aws.amazon.com/security-hub/). The following is the code snippet of the Lambda function.

![Zap Lambda Code Snippet](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/06/25/Zap-lambda-codesnippet-1-1024x500.png)

*Zap Lambda Code Snippet*

The following screenshot shows the results in Security Hub. The highlighted section shows the vulnerability findings from various scanning stages.

![Vulnerability Findings in Security Hub](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/06/18/SecurityHub-Screenshot.png)

*Vulnerability Findings in Security Hub*

We can drill down to individual resource IDs to get the list of vulnerability findings. For example, if we drill down to the resource ID of `SASTBuildProject*`, we can review all the findings from that resource ID.

![SAST Vulnerabilities in Security Hub](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/06/18/SecurityHub-Anchore-Screenshot.png)

*SAST Vulnerabilities in Security Hub*

If there are no vulnerabilities in the DAST scan, the pipeline proceeds to the manual approval stage and an email is sent to the approver. The approver can review and approve or reject the deployment. If approved, the pipeline moves to next stage and deploys the application to the production EKS cluster.

Aggregation of vulnerability findings in [Security Hub](https://aws.amazon.com/security-hub/) provides opportunities to automate the remediation. For example, based on the vulnerability finding, you can trigger a [Lambda](https://aws.amazon.com/lambda/) function to take the needed remediation action. This also reduces the burden on operations and security teams because they can now address the vulnerabilities from a single pane of glass instead of logging into multiple tool dashboards.

Along with Security Hub, you can send vulnerability findings to your issue tracking systems such as JIRA, [Systems Manager SysOps](https://aws.amazon.com/systems-manager/), or can automatically create an incident management ticket. This is outside the scope of this post, but is one of the possibilities you can consider when implementing DevSecOps software factories.

## RASP scanning

Sysdig Falco is an open-source runtime security tool. Based on the configured rules, Falco can detect suspicious activity and alert on any behavior that involves making Linux system calls. You can use Falco rules to address security controls like NIST SP 800-53. Falco agents on each EKS node continuously scan the containers running in pods and send the events as STDOUT. These events can be then sent to [CloudWatch](https://aws.amazon.com/cloudwatch/) or any third-party log aggregator to send alerts and respond. For more information, see [Implementing Runtime security in Amazon EKS using CNCF Falco](https://aws.amazon.com/blogs/containers/implementing-runtime-security-in-amazon-eks-using-cncf-falco/). You can also use [Lambda](https://aws.amazon.com/lambda/) to trigger and automatically remediate certain security events.

The following screenshot shows Falco events on the [CloudWatch](https://aws.amazon.com/cloudwatch/) console. The highlighted text describes the Falco event that was triggered based on the default Falco rules on the EKS cluster. You can add additional custom rules to meet your security control requirements. You can also trigger responsive actions from these CloudWatch events using services like Lambda.

![Falco alerts in CloudWatch](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/06/25/falco-alerts-1024x387.png)

*Falco alerts in CloudWatch*

## Cleanup

This section provides instructions to clean up the DevSecOps pipeline setup:

1. Delete the [EKS cluster](https://aws.amazon.com/eks/).
2. Delete the [S3 bucket](https://aws.amazon.com/s3/).
3. Delete the [CodeCommit](https://aws.amazon.com/codecommit/) repo.
4. Delete the [Amazon ECR](https://aws.amazon.com/ecr/) repo.
5. Disable [Security Hub](https://aws.amazon.com/security-hub/).
6. Disable [AWS Config](https://aws.amazon.com/config/).
7. Delete the pipeline [CloudFormation](https://aws.amazon.com/cloudformation/) stack.

## Conclusion

In this post, I presented an end-to-end Kubernetes-based DevSecOps software factory on AWS with continuous testing, continuous logging and monitoring, auditing and governance, and operations. I demonstrated how to integrate various open-source scanning tools, such as Git-Secrets, Anchore, Snyk, OWASP ZAP, and Sysdig Falco for Secret Analysis, SCA, SAST, DAST, and RASP analysis, respectively. To reduce operations overhead, I explained how to aggregate and manage vulnerability findings in [Security Hub](https://aws.amazon.com/security-hub/) as a single pane of glass. This post also talked about how to implement security of the pipeline and in the pipeline using AWS Cloud-native services. Finally, I provided the DevSecOps software factory as code using [AWS CloudFormation](https://aws.amazon.com/cloudformation/).

To get started with DevSecOps on AWS, see [AWS DevOps](https://aws.amazon.com/devops/) and the [DevOps blog](https://aws.amazon.com/blogs/devops/).

## About the Author

<table>
  <tr>
    <td width="130" valign="top">
      <img src="https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2021/01/21/IMG_2614-150x150.jpg" width="120" alt="Srinivas Manepalli" />
    </td>
    <td valign="top">
      <br/>
      <b>Srinivas Manepalli</b>
      <br/><br/>
      <font color="#555555">Srinivas Manepalli is a DevSecOps Solutions Architect in the U.S. Fed SI SA team at Amazon Web Services (AWS). He is passionate about helping customers, building and architecting DevSecOps and highly available software systems. Outside of work, he enjoys spending time with family, nature and good food.</font>
    </td>
  </tr>
</table>