---
title : "Configure 6 Security Gates"
date : 2026-07-06 
weight : 3
chapter : false
pre : " <b> 5.3.3. </b> "
---

# 5.3.3 Configure 6 Security Gates in Jenkinsfile

---

Integrate 6 security inspection gates into `Jenkinsfile`:

```groovy
pipeline {
    agent any
    environment {
        AWS_REGION     = 'ap-southeast-1'
        ECR_REGISTRY   = '123456789012.dkr.ecr.ap-southeast-1.amazonaws.com'
        REPO_NAME      = 'devsecops-react-app'
        S3_BUCKET      = 'devsecops-reports-bucket-123456789012'
        IMAGE_TAG      = "${GIT_COMMIT}"
    }
    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }
        stage('Gate 1: Secrets Scan') {
            steps {
                sh 'gitleaks detect --source=. --report-path=reports/gitleaks.json --exit-code=0'
            }
        }
        stage('Build React App') {
            steps {
                sh 'npm install && npm run build'
            }
        }
        stage('Gate 2: SCA Scan') {
            steps {
                sh 'trivy fs --format json --output reports/trivy-sca.json .'
            }
        }
        stage('Gate 3: SAST Scan') {
            steps {
                sh 'sonar-scanner -Dsonar.projectKey=react-app -Dsonar.sources=src'
            }
        }
        stage('Docker Build') {
            steps {
                sh 'docker build -t ${ECR_REGISTRY}/${REPO_NAME}:${IMAGE_TAG} .'
            }
        }
        stage('Gate 4: Container Scan') {
            steps {
                sh 'trivy image --format json --output reports/trivy-image.json ${ECR_REGISTRY}/${REPO_NAME}:${IMAGE_TAG}'
            }
        }
        stage('Gate 5: IaC Scan') {
            steps {
                sh 'checkov -d . --output json > reports/checkov.json || true'
            }
        }
        stage('Push Image to ECR') {
            steps {
                sh 'aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}'
                sh 'docker push ${ECR_REGISTRY}/${REPO_NAME}:${IMAGE_TAG}'
            }
        }
        stage('Upload Reports to S3') {
            steps {
                sh 'aws s3 cp reports/ s3://${S3_BUCKET}/reports/${IMAGE_TAG}/ --recursive'
            }
        }
    }
}
```
