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
        AWS_REGION        = 'ap-southeast-1'
        REGISTRY_TARGET   = 'ecr'
        IMAGE_NAME        = 'devsecops/tetris'
        SECURITY_MODE     = 'enforce'
        ENABLE_ECS_DEPLOY = 'true'
        ENABLE_DAST       = 'true'
        ENABLE_S3_UPLOAD  = 'true'
    }
    stages {
        stage('01. Environment Preflight & Checkout') { steps { checkout scm } }
        stage('02. Static Validation')                { steps { sh './scripts/validate.sh' } }
        stage('03. Gate 1: Secrets Scan')             { steps { sh './ci/stages/secrets-scan.sh' } }
        stage('04. Gate 2: SCA Dependency Scan')      { steps { sh './ci/stages/sca-scan.sh' } }
        stage('05. Gate 3: SAST Code Analysis')       { steps { sh './ci/stages/sast-scan.sh' } }
        stage('06. Gate 4: IaC Scan')                 { steps { sh './ci/stages/iac-scan.sh' } }
        stage('07. Application Build')                { steps { sh 'cd app && npm install && npm run build' } }
        stage('08. Docker Image Build')               { steps { sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -f app/Dockerfile app' } }
        stage('09. Gate 5: Container Image Scan')     { steps { sh './ci/stages/container-scan.sh' } }
        stage('10. Push Image to Amazon ECR')         { steps { sh 'aws ecr get-login-password | docker login && docker push ${ECR_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}' } }
        stage('11. Mirror to Local Registry')         { steps { sh 'docker tag ${IMAGE_NAME}:${IMAGE_TAG} localhost:5001/${IMAGE_NAME}:${IMAGE_TAG} && docker push localhost:5001/${IMAGE_NAME}:${IMAGE_TAG}' } }
        stage('12. Deploy Local GitOps Staging')      { steps { sh 'kustomize edit set image ... && git commit && git push' } }
        stage('13. Deploy AWS ECS Fargate Staging')   { steps { sh 'aws ecs update-service --cluster devsecops-factory-cluster --service tetris-staging --force-new-deployment' } }
        stage('14. Gate 6: OWASP ZAP DAST Scan')      { steps { sh './ci/stages/dast-scan.sh' } }
        stage('15. Normalize Reports & ASFF Conversion') { steps { sh 'python3 ci/stages/normalize-reports.py' } }
        stage('16. Upload Reports to Amazon S3')      { steps { sh 'aws s3 cp scan-reports/ s3://${S3_BUCKET}/reports/ --recursive' } }
        stage('17. AWS Lambda Security Hub Ingest')   { steps { sh 'aws lambda invoke --function-name devsecops-factory-securityhub-importer response.json' } }
        stage('18. Observability Verification')       { steps { sh 'curl -I http://tetris-staging.localhost' } }
        stage('19. Production Approval Gate')         { steps { input message: 'Approve deployment to ECS Production?' } }
        stage('20. Deploy Local GitOps Production')   { steps { sh 'kustomize edit set image ... && git commit && git push' } }
        stage('21. Deploy AWS ECS Fargate Production'){ steps { sh 'aws ecs update-service --cluster devsecops-factory-cluster --service tetris-production --force-new-deployment' } }
        stage('22. Summary & Evidence Archiving')     { steps { sh 'archiveArtifacts artifacts: "scan-reports/**"' } }
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
