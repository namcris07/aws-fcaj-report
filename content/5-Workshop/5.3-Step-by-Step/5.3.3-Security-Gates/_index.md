---
title : "Configure 6 Security Gates"
date : 2026-07-06 
weight : 3
chapter : false
pre : " <b> 5.3.3. </b> "
---

# 5.3.3 Configure 6 Security Gates in Jenkinsfile

---

Integrate 6 Security Gates and ECS deployment pipeline stages into `ci/Jenkinsfile`:

```groovy
pipeline {
    agent any
    environment {
        AWS_REGION      = 'ap-southeast-1'
        REGISTRY_TARGET = 'ecr'
        IMAGE_NAME      = 'devsecops/tetris'
        SECURITY_MODE   = 'enforce'
        ENABLE_ECS_DEPLOY = 'true'
        ENABLE_DAST     = 'true'
        ENABLE_S3_UPLOAD = 'true'
    }
    stages {
        stage('Checkout Source') {
            steps { checkout scm }
        }
        stage('Gate 1: Secrets Scan') {
            steps {
                sh './ci/stages/01-secrets-scan.sh'
            }
        }
        stage('Gate 2: SCA Dependency Scan') {
            steps {
                sh './ci/stages/02-sca-scan.sh'
            }
        }
        stage('Gate 3: SAST Code Analysis') {
            steps {
                sh './ci/stages/03-sast-scan.sh'
            }
        }
        stage('Gate 4: IaC Scan') {
            steps {
                sh './ci/stages/04-iac-scan.sh'
            }
        }
        stage('Docker Build') {
            steps {
                sh 'docker build -t ${ECR_REGISTRY}/${IMAGE_NAME}:${COMMIT_SHA} app/'
            }
        }
        stage('Gate 5: Container Image Scan') {
            steps {
                sh './ci/stages/05-container-scan.sh'
            }
        }
        stage('Push Image to Amazon ECR') {
            steps {
                sh 'aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}'
                sh 'docker push ${ECR_REGISTRY}/${IMAGE_NAME}:${COMMIT_SHA}'
            }
        }
        stage('Deploy to ECS Staging') {
            steps {
                sh 'aws ecs update-service --cluster devsecops-factory-cluster --service devsecops-factory-staging --force-new-deployment'
            }
        }
        stage('Gate 6: OWASP ZAP DAST Scan') {
            steps {
                sh './ci/stages/06-dast-scan.sh'
            }
        }
        stage('Normalize Reports & Upload to S3') {
            steps {
                sh 'python3 ci/stages/normalize-reports.py'
                sh 'aws s3 cp results/ s3://${S3_BUCKET}/reports/${COMMIT_SHA}/ --recursive'
            }
        }
        stage('Production Approval Gate') {
            steps {
                input message: 'Approve deployment to ECS Production?'
            }
        }
        stage('Deploy to ECS Production') {
            steps {
                sh 'aws ecs update-service --cluster devsecops-factory-cluster --service devsecops-factory-prod --force-new-deployment'
            }
        }
    }
}
```

---

### Screenshots: Jenkins Pipeline & Security Artifacts

![Jenkins Stages](/images/5-Workshop/5.3-Step-by-Step/task2-01-jenkins-stages.png)
*Figure 5.3.3a: Sequential stage execution output in Jenkins Pipeline.*

![Jenkins Build & Push ECR](/images/5-Workshop/5.3-Step-by-Step/task2-01b-jenkins-build-push-stages.png)
*Figure 5.3.3b: Docker Image Build and Amazon ECR Push stages.*

![Security Report Artifacts](/images/5-Workshop/5.3-Step-by-Step/task2-02-security-artifacts.png)
*Figure 5.3.3c: Archived Security Inspection Report Artifacts.*
