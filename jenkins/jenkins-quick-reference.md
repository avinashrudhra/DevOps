# Jenkins Quick Reference Guide

Essential Jenkins commands, pipeline syntax, and integration patterns for daily operations.

---

## Table of Contents
1. [Installation & Setup](#installation--setup)
2. [Pipeline Syntax](#pipeline-syntax)
3. [Git Integration](#git-integration)
4. [Maven Integration](#maven-integration)
5. [SonarQube Integration](#sonarqube-integration)
6. [Docker Integration](#docker-integration)
7. [Kubernetes Integration](#kubernetes-integration)
8. [Common Patterns](#common-patterns)

---

## Installation & Setup

### Docker Installation
```bash
# Run Jenkins in Docker
docker run -d \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --name jenkins \
  jenkins/jenkins:lts

# Get initial password
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword

# Access: http://localhost:8080
```

### Linux Installation
```bash
# Add Jenkins repository
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | \
  sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | \
  sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

# Install
sudo apt update
sudo apt install jenkins

# Start
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

---

## Pipeline Syntax

### Basic Declarative Pipeline
```groovy
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                echo 'Building...'
                sh 'mvn clean compile'
            }
        }
        
        stage('Test') {
            steps {
                echo 'Testing...'
                sh 'mvn test'
            }
        }
        
        stage('Deploy') {
            steps {
                echo 'Deploying...'
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
```

### Complete Pipeline Template
```groovy
pipeline {
    agent any
    
    environment {
        APP_NAME = 'myapp'
        VERSION = '1.0'
        DOCKER_REGISTRY = 'registry.example.com'
    }
    
    tools {
        maven 'Maven 3.8'
        jdk 'JDK 11'
    }
    
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 1, unit: 'HOURS')
        timestamps()
    }
    
    triggers {
        pollSCM('H/5 * * * *')
    }
    
    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'])
        booleanParam(name: 'RUN_TESTS', defaultValue: true)
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }
        
        stage('Test') {
            when {
                expression { params.RUN_TESTS }
            }
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            echo 'Build succeeded!'
        }
        failure {
            mail to: 'team@example.com',
                 subject: "Build Failed: ${env.JOB_NAME}",
                 body: "Build ${env.BUILD_NUMBER} failed"
        }
    }
}
```

### Agent Types
```groovy
// Any available agent
agent any

// Specific label
agent {
    label 'linux'
}

// Docker agent
agent {
    docker {
        image 'maven:3.8-jdk-11'
        args '-v $HOME/.m2:/root/.m2'
    }
}

// Kubernetes pod
agent {
    kubernetes {
        yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: maven
    image: maven:3.8-jdk-11
'''
    }
}

// None (run on master)
agent none
```

### Parallel Execution
```groovy
stage('Tests') {
    parallel {
        stage('Unit Tests') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Integration Tests') {
            steps {
                sh 'mvn verify'
            }
        }
        stage('Security Scan') {
            steps {
                sh 'snyk test'
            }
        }
    }
}
```

### When Conditions
```groovy
// Branch condition
when {
    branch 'main'
}

// Environment condition
when {
    environment name: 'DEPLOY', value: 'true'
}

// Expression condition
when {
    expression { env.BRANCH_NAME ==~ /release-.*/ }
}

// Change request (PR)
when {
    changeRequest()
}

// Tag condition
when {
    tag 'v*'
}

// Multiple conditions (AND)
when {
    allOf {
        branch 'main'
        environment name: 'DEPLOY', value: 'true'
    }
}

// Multiple conditions (OR)
when {
    anyOf {
        branch 'main'
        branch 'develop'
    }
}
```

---

## Git Integration

### Basic Git Checkout
```groovy
stage('Checkout') {
    steps {
        git branch: 'main',
            url: 'https://github.com/user/repo.git'
    }
}
```

### Git with Credentials
```groovy
stage('Checkout') {
    steps {
        git branch: 'main',
            credentialsId: 'github-credentials',
            url: 'https://github.com/user/repo.git'
    }
}
```

### Advanced Git Checkout
```groovy
stage('Checkout') {
    steps {
        checkout([
            $class: 'GitSCM',
            branches: [[name: '*/main']],
            extensions: [
                [$class: 'CloneOption', depth: 1, shallow: true],
                [$class: 'CleanBeforeCheckout']
            ],
            userRemoteConfigs: [[
                credentialsId: 'github-credentials',
                url: 'https://github.com/user/repo.git'
            ]]
        ])
    }
}
```

### GitHub Webhook Configuration
```
GitHub Repository Settings:
→ Webhooks
→ Add webhook
→ Payload URL: http://jenkins.example.com/github-webhook/
→ Content type: application/json
→ Events: Push, Pull request
```

### Multibranch Pipeline
```groovy
// Jenkinsfile in repository
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                echo "Building branch: ${env.BRANCH_NAME}"
                echo "Change: ${env.CHANGE_ID}"
                sh 'mvn clean package'
            }
        }
    }
}
```

---

## Maven Integration

### Maven Pipeline
```groovy
pipeline {
    agent any
    
    tools {
        maven 'Maven 3.8'
        jdk 'JDK 11'
    }
    
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                    jacoco()
                }
            }
        }
        
        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
            }
            post {
                success {
                    archiveArtifacts artifacts: '**/target/*.jar',
                                   fingerprint: true
                }
            }
        }
        
        stage('Deploy') {
            steps {
                sh 'mvn deploy -DskipTests'
            }
        }
    }
}
```

### Maven with Docker Agent
```groovy
pipeline {
    agent {
        docker {
            image 'maven:3.8-jdk-11'
            args '-v $HOME/.m2:/root/.m2'
        }
    }
    
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
    }
}
```

---

## SonarQube Integration

### Complete SonarQube Pipeline
```groovy
pipeline {
    agent any
    
    tools {
        maven 'Maven 3.8'
        jdk 'JDK 11'
    }
    
    environment {
        SONAR_HOST = 'http://sonarqube:9000'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build & Test') {
            steps {
                sh 'mvn clean verify'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        mvn sonar:sonar \
                          -Dsonar.projectKey=${JOB_NAME} \
                          -Dsonar.projectName="${JOB_NAME}" \
                          -Dsonar.host.url=${SONAR_HOST}
                    '''
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    script {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }
        
        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }
    }
}
```

### SonarQube for Pull Requests
```groovy
stage('PR Analysis') {
    when {
        changeRequest()
    }
    steps {
        withSonarQubeEnv('SonarQube') {
            sh """
                mvn sonar:sonar \
                  -Dsonar.pullrequest.key=${env.CHANGE_ID} \
                  -Dsonar.pullrequest.branch=${env.CHANGE_BRANCH} \
                  -Dsonar.pullrequest.base=${env.CHANGE_TARGET}
            """
        }
    }
}
```

---

## Docker Integration

### Build Docker Image
```groovy
stage('Build Docker Image') {
    steps {
        script {
            dockerImage = docker.build("myapp:${BUILD_NUMBER}")
        }
    }
}
```

### Complete Docker Pipeline
```groovy
pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'registry.example.com'
        DOCKER_IMAGE = "${DOCKER_REGISTRY}/myapp"
        DOCKER_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build Application') {
            agent {
                docker {
                    image 'maven:3.8-jdk-11'
                    args '-v maven-repo:/root/.m2'
                }
            }
            steps {
                sh 'mvn clean package -DskipTests'
                stash includes: 'target/*.jar', name: 'app'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                unstash 'app'
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }
        
        stage('Scan Image') {
            steps {
                sh "trivy image ${DOCKER_IMAGE}:${DOCKER_TAG}"
            }
        }
        
        stage('Push Image') {
            steps {
                script {
                    docker.withRegistry("https://${DOCKER_REGISTRY}", 'docker-credentials') {
                        dockerImage.push("${DOCKER_TAG}")
                        dockerImage.push("latest")
                    }
                }
            }
        }
    }
    
    post {
        always {
            sh "docker rmi ${DOCKER_IMAGE}:${DOCKER_TAG} || true"
        }
    }
}
```

### Docker Compose in Pipeline
```groovy
stage('Integration Test') {
    steps {
        sh '''
            docker-compose -f docker-compose.test.yml up -d
            sleep 30
            curl http://localhost:8080/health
            docker-compose -f docker-compose.test.yml down
        '''
    }
}
```

---

## Kubernetes Integration

### Basic Kubernetes Deployment
```groovy
stage('Deploy to Kubernetes') {
    steps {
        withKubeConfig([credentialsId: 'kubeconfig']) {
            sh '''
                kubectl set image deployment/myapp \
                  myapp=${DOCKER_IMAGE}:${BUILD_NUMBER} \
                  -n production
                
                kubectl rollout status deployment/myapp -n production
            '''
        }
    }
}
```

### Complete Kubernetes Pipeline
```groovy
pipeline {
    agent any
    
    environment {
        REGISTRY = 'registry.example.com'
        IMAGE = "${REGISTRY}/myapp"
        TAG = "${BUILD_NUMBER}"
        K8S_NAMESPACE = 'production'
    }
    
    stages {
        stage('Build & Push') {
            steps {
                script {
                    docker.build("${IMAGE}:${TAG}").push()
                }
            }
        }
        
        stage('Update Manifests') {
            steps {
                sh """
                    sed -i 's|IMAGE_TAG|${TAG}|g' k8s/deployment.yaml
                """
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh '''
                        kubectl apply -f k8s/ -n ${K8S_NAMESPACE}
                        kubectl rollout status deployment/myapp -n ${K8S_NAMESPACE}
                    '''
                }
            }
        }
        
        stage('Verify') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh '''
                        kubectl get pods -n ${K8S_NAMESPACE}
                        kubectl get svc -n ${K8S_NAMESPACE}
                    '''
                }
            }
        }
        
        stage('Smoke Test') {
            steps {
                sh '''
                    sleep 30
                    curl -f http://myapp.example.com/health || exit 1
                '''
            }
        }
    }
    
    post {
        failure {
            withKubeConfig([credentialsId: 'kubeconfig']) {
                sh '''
                    kubectl rollout undo deployment/myapp -n ${K8S_NAMESPACE}
                '''
            }
        }
    }
}
```

### Helm Deployment
```groovy
stage('Deploy with Helm') {
    steps {
        withKubeConfig([credentialsId: 'kubeconfig']) {
            sh '''
                helm upgrade --install myapp ./helm/myapp \
                  --namespace production \
                  --set image.tag=${BUILD_NUMBER} \
                  --set image.repository=${DOCKER_IMAGE} \
                  --wait \
                  --timeout 5m
            '''
        }
    }
}
```

---

## Common Patterns

### Complete CI/CD Pipeline (All Tools)
```groovy
pipeline {
    agent any
    
    environment {
        REGISTRY = 'registry.example.com'
        IMAGE = "${REGISTRY}/myapp"
        TAG = "${BUILD_NUMBER}"
        SONAR_HOST = 'http://sonarqube:9000'
        K8S_NAMESPACE = 'production'
    }
    
    tools {
        maven 'Maven 3.8'
        jdk 'JDK 11'
    }
    
    stages {
        // 1. Checkout from Git
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        // 2. Build with Maven
        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }
        
        // 3. Run Tests
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                    jacoco()
                }
            }
        }
        
        // 4. SonarQube Analysis
        stage('Code Quality') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
        
        // 5. Quality Gate
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        // 6. Package Application
        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }
        
        // 7. Build Docker Image
        stage('Docker Build') {
            steps {
                script {
                    docker.build("${IMAGE}:${TAG}")
                }
            }
        }
        
        // 8. Security Scan
        stage('Security Scan') {
            steps {
                sh "trivy image ${IMAGE}:${TAG}"
            }
        }
        
        // 9. Push to Registry
        stage('Push Image') {
            steps {
                script {
                    docker.withRegistry("https://${REGISTRY}", 'docker-creds') {
                        docker.image("${IMAGE}:${TAG}").push()
                        docker.image("${IMAGE}:${TAG}").push('latest')
                    }
                }
            }
        }
        
        // 10. Deploy to Kubernetes
        stage('Deploy to K8s') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh """
                        kubectl set image deployment/myapp \
                          myapp=${IMAGE}:${TAG} \
                          -n ${K8S_NAMESPACE}
                        kubectl rollout status deployment/myapp -n ${K8S_NAMESPACE}
                    """
                }
            }
        }
        
        // 11. Smoke Tests
        stage('Smoke Test') {
            steps {
                sh '''
                    sleep 30
                    curl -f http://myapp.example.com/health || exit 1
                '''
            }
        }
    }
    
    post {
        success {
            slackSend color: 'good',
                     message: "Build ${BUILD_NUMBER} deployed successfully!"
        }
        failure {
            slackSend color: 'danger',
                     message: "Build ${BUILD_NUMBER} failed!"
            
            withKubeConfig([credentialsId: 'kubeconfig']) {
                sh "kubectl rollout undo deployment/myapp -n ${K8S_NAMESPACE}"
            }
        }
        always {
            cleanWs()
        }
    }
}
```

### Multi-Environment Deployment
```groovy
pipeline {
    agent any
    
    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'])
    }
    
    stages {
        stage('Deploy') {
            steps {
                script {
                    def namespace = params.ENVIRONMENT
                    def replicas = params.ENVIRONMENT == 'prod' ? 3 : 1
                    
                    withKubeConfig([credentialsId: "kubeconfig-${params.ENVIRONMENT}"]) {
                        sh """
                            kubectl set image deployment/myapp \
                              myapp=${IMAGE}:${TAG} \
                              -n ${namespace}
                            kubectl scale deployment/myapp --replicas=${replicas} -n ${namespace}
                            kubectl rollout status deployment/myapp -n ${namespace}
                        """
                    }
                }
            }
        }
    }
}
```

### Blue-Green Deployment
```groovy
stage('Blue-Green Deploy') {
    steps {
        withKubeConfig([credentialsId: 'kubeconfig']) {
            sh '''
                # Deploy green
                kubectl apply -f k8s/deployment-green.yaml
                kubectl rollout status deployment/myapp-green
                
                # Run tests
                curl -f http://myapp-green.example.com/health
                
                # Switch traffic
                kubectl patch service myapp -p '{"spec":{"selector":{"version":"green"}}}'
                
                # Delete blue
                kubectl delete deployment myapp-blue
            '''
        }
    }
}
```

### Canary Deployment
```groovy
stage('Canary Deploy') {
    steps {
        withKubeConfig([credentialsId: 'kubeconfig']) {
            sh '''
                # Deploy canary (10%)
                kubectl apply -f k8s/deployment-canary.yaml
                kubectl scale deployment/myapp-canary --replicas=1
                
                # Monitor for 10 minutes
                sleep 600
                
                # Check metrics
                # If OK, proceed with full rollout
                kubectl scale deployment/myapp-canary --replicas=10
                kubectl scale deployment/myapp-stable --replicas=0
            '''
        }
    }
}
```

---

## Credentials Management

### Using Credentials
```groovy
// String credential
withCredentials([string(credentialsId: 'api-key', variable: 'API_KEY')]) {
    sh 'curl -H "Authorization: ${API_KEY}" https://api.example.com'
}

// Username/Password
withCredentials([usernamePassword(
    credentialsId: 'docker-creds',
    usernameVariable: 'USER',
    passwordVariable: 'PASS'
)]) {
    sh 'echo $PASS | docker login -u $USER --password-stdin'
}

// SSH Key
withCredentials([sshUserPrivateKey(
    credentialsId: 'ssh-key',
    keyFileVariable: 'SSH_KEY'
)]) {
    sh 'ssh -i $SSH_KEY user@server.com'
}

// File
withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
    sh 'kubectl get pods'
}
```

---

## Notifications

### Email
```groovy
post {
    failure {
        mail to: 'team@example.com',
             subject: "Build Failed: ${env.JOB_NAME} ${env.BUILD_NUMBER}",
             body: "Build failed. Check: ${env.BUILD_URL}"
    }
}
```

### Slack
```groovy
post {
    success {
        slackSend channel: '#builds',
                 color: 'good',
                 message: "Build ${BUILD_NUMBER} succeeded!"
    }
    failure {
        slackSend channel: '#builds',
                 color: 'danger',
                 message: "Build ${BUILD_NUMBER} failed!"
    }
}
```

---

## Useful Commands

### Jenkins CLI
```bash
# Download CLI
wget http://jenkins:8080/jnlpJars/jenkins-cli.jar

# Build job
java -jar jenkins-cli.jar -s http://jenkins:8080 \
  -auth user:token build job-name

# List jobs
java -jar jenkins-cli.jar -s http://jenkins:8080 \
  -auth user:token list-jobs
```

### Groovy Console Scripts
```groovy
// Get all jobs
Jenkins.instance.getAllItems(Job.class).each { job ->
    println job.name
}

// Restart Jenkins
Jenkins.instance.restart()

// Clear queue
Jenkins.instance.queue.clear()
```

---

**Tip**: Store this reference for quick access to pipeline patterns and integrations!


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

