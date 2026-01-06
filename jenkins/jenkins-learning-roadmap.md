# Jenkins Learning Roadmap
## 12-Week Journey from Beginner to CI/CD Expert

Complete curriculum for mastering Jenkins with full DevOps tool integration.

---

## 🎯 Learning Path Overview

```
Weeks 1-2:  Jenkins Fundamentals & Installation
Weeks 3-4:  Pipeline Basics (Declarative & Scripted)
Weeks 5-6:  Tool Integration (Git, Maven, SonarQube)
Weeks 7-8:  Docker Integration & Container Builds
Weeks 9-10: Kubernetes Deployment & GitOps
Weeks 11-12: Production, Security & Advanced Topics
```

**Time Commitment**: 12-15 hours per week  
**Prerequisites**: Git, Maven, basic CI/CD understanding  
**Target**: Production-ready Jenkins expertise with complete tool integration

---

## Week 1: Jenkins Fundamentals

### Learning Objectives
- Understand CI/CD concepts and Jenkins role
- Install and configure Jenkins
- Navigate Jenkins UI
- Understand Jenkins architecture

### Topics

#### 1.1 What is Jenkins?

**Jenkins Overview:**
- Open-source automation server
- Leading CI/CD tool (50%+ market share)
- 1800+ plugins
- Extensible and customizable
- Active community support

**Why Jenkins?**
- **Automation**: Build, test, deploy automatically
- **Integration**: Works with all DevOps tools
- **Scalability**: Distributed builds
- **Flexibility**: Pipelines as code
- **Cost**: Free and open-source

**Jenkins in DevOps Pipeline:**
```
Developer Push → Git → Jenkins (Orchestrator)
                         ↓
              ┌──────────┴──────────┐
              ↓                     ↓
         Build (Maven)        Scan (SonarQube)
              ↓                     ↓
              └──────────┬──────────┘
                         ↓
                  Docker Image Build
                         ↓
                  Push to Registry
                         ↓
              Deploy to Kubernetes
```

#### 1.2 Jenkins Architecture

**Components:**
```
┌─────────────────────────────┐
│     Jenkins Master          │
│  - Web Interface            │
│  - Job Scheduling           │
│  - Build Distribution       │
│  - Plugin Management        │
└──────────┬──────────────────┘
           │
    ┌──────┴──────┐
    ↓             ↓
┌─────────┐  ┌─────────┐
│ Agent 1 │  │ Agent 2 │
│         │  │         │
│ (Linux) │  │(Windows)│
└─────────┘  └─────────┘
```

**Master (Controller):**
- Manages configuration
- Schedules builds
- Monitors agents
- Serves web UI
- Stores build records

**Agents (Nodes):**
- Execute builds
- Can be static or dynamic
- Different OS/tools per agent
- Isolated workspaces

#### 1.3 Installation

**Docker (Recommended for Learning):**
```bash
# Run Jenkins in Docker
docker run -d \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --name jenkins \
  jenkins/jenkins:lts

# Get initial admin password
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword

# Access: http://localhost:8080
```

**Linux (Ubuntu/Debian):**
```bash
# Install Java
sudo apt update
sudo apt install openjdk-11-jdk

# Add Jenkins repository
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | \
  sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | \
  sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

# Install Jenkins
sudo apt update
sudo apt install jenkins

# Start Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins

# Access: http://localhost:8080
```

**Initial Setup:**
1. Enter initial admin password
2. Install suggested plugins
3. Create admin user
4. Configure Jenkins URL

#### 1.4 Jenkins UI Navigation

**Dashboard:**
- Build queue
- Build executor status
- Recent changes
- All jobs list

**Job Configuration:**
- General settings
- Source code management
- Build triggers
- Build environment
- Build steps
- Post-build actions

**System Configuration:**
- Manage Jenkins
- Configure System
- Global Tool Configuration
- Plugin Manager
- Security settings

### Hands-On Practice
1. Install Jenkins (Docker or native)
2. Complete initial setup
3. Create first freestyle job
4. Explore Jenkins UI
5. Install additional plugins

### Resources
- Jenkins.io documentation
- Jenkins User Handbook

---

## Week 2: Jobs & Build Configuration

### Learning Objectives
- Create and configure jobs
- Understand build triggers
- Configure build environments
- Work with parameters

### Topics

#### 2.1 Job Types

**Freestyle Project:**
- GUI-based configuration
- Good for simple tasks
- Easy to set up
- Limited flexibility

**Pipeline:**
- Code-based (Jenkinsfile)
- Version controlled
- More powerful
- Recommended approach

**Multi-configuration Project:**
- Matrix builds
- Test multiple configurations
- Different platforms/versions

**Multibranch Pipeline:**
- Automatic branch detection
- Per-branch pipelines
- PR/MR integration

#### 2.2 Build Triggers

**Manual:**
```
Build Now button
```

**Poll SCM:**
```groovy
// Check every 5 minutes
H/5 * * * *
```

**Webhook (Recommended):**
```
GitHub: Settings → Webhooks
GitLab: Settings → Integrations
Payload URL: http://jenkins.example.com/github-webhook/
```

**Scheduled:**
```groovy
// Daily at 2 AM
H 2 * * *

// Every 15 minutes
H/15 * * * *

// Weekdays at 9 AM
H 9 * * 1-5
```

**Build after other projects:**
```
Trigger after: upstream-project-name
```

#### 2.3 Parameters

**String Parameter:**
```groovy
parameters {
    string(name: 'VERSION', defaultValue: '1.0', description: 'Version to deploy')
}

// Use: ${params.VERSION}
```

**Choice Parameter:**
```groovy
parameters {
    choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'])
}
```

**Boolean Parameter:**
```groovy
parameters {
    booleanParam(name: 'RUN_TESTS', defaultValue: true)
}
```

### Hands-On Practice
1. Create freestyle job
2. Configure Git repository
3. Set up webhook trigger
4. Add build parameters
5. Run parameterized build

---

## Week 3: Declarative Pipelines

### Learning Objectives
- Write Declarative Pipelines
- Understand pipeline structure
- Use stages and steps
- Handle errors

### Topics

#### 3.1 Pipeline Structure

**Basic Pipeline:**
```groovy
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                echo 'Building...'
            }
        }
        
        stage('Test') {
            steps {
                echo 'Testing...'
            }
        }
        
        stage('Deploy') {
            steps {
                echo 'Deploying...'
            }
        }
    }
}
```

#### 3.2 Agents

**Any agent:**
```groovy
agent any
```

**Specific label:**
```groovy
agent {
    label 'linux'
}
```

**Docker agent:**
```groovy
agent {
    docker {
        image 'maven:3.8-jdk-11'
    }
}
```

**Per-stage agent:**
```groovy
stage('Build') {
    agent {
        docker 'maven:3.8-jdk-11'
    }
    steps {
        sh 'mvn clean package'
    }
}
```

#### 3.3 Environment Variables

```groovy
pipeline {
    agent any
    
    environment {
        APP_NAME = 'myapp'
        VERSION = '1.0'
        DOCKER_REGISTRY = 'registry.example.com'
    }
    
    stages {
        stage('Build') {
            environment {
                BUILD_ENV = 'production'
            }
            steps {
                sh "echo Building ${APP_NAME} version ${VERSION}"
            }
        }
    }
}
```

#### 3.4 Post Actions

```groovy
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
    }
    
    post {
        always {
            echo 'This runs always'
            cleanWs()
        }
        success {
            echo 'Build succeeded!'
            slackSend color: 'good', message: 'Build successful'
        }
        failure {
            echo 'Build failed!'
            mail to: 'team@example.com',
                 subject: "Build Failed: ${env.JOB_NAME}",
                 body: "Build ${env.BUILD_NUMBER} failed"
        }
        unstable {
            echo 'Build is unstable'
        }
        cleanup {
            echo 'Clean up workspace'
        }
    }
}
```

### Hands-On Practice
1. Create Jenkinsfile in repository
2. Create pipeline job from SCM
3. Add multiple stages
4. Configure post actions
5. Test failure scenarios

---

## Week 4: Pipeline Advanced Features

### Learning Objectives
- Use parallel execution
- Implement input steps
- Handle credentials
- Use when conditions

### Topics

#### 4.1 Parallel Execution

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

#### 4.2 Input Steps

```groovy
stage('Deploy to Production') {
    input {
        message "Deploy to production?"
        ok "Deploy"
        parameters {
            choice(name: 'REGION', choices: ['us-east-1', 'eu-west-1'])
        }
    }
    steps {
        echo "Deploying to ${REGION}"
    }
}
```

#### 4.3 Credentials

```groovy
pipeline {
    agent any
    
    environment {
        DOCKER_CREDS = credentials('docker-credentials')
    }
    
    stages {
        stage('Login') {
            steps {
                sh 'echo $DOCKER_CREDS_PSW | docker login -u $DOCKER_CREDS_USR --password-stdin'
            }
        }
    }
}
```

#### 4.4 When Conditions

```groovy
stage('Deploy') {
    when {
        branch 'main'
    }
    steps {
        echo 'Deploying from main branch'
    }
}

stage('Release') {
    when {
        tag 'v*'
    }
    steps {
        echo 'Creating release'
    }
}

stage('PR Check') {
    when {
        changeRequest()
    }
    steps {
        echo 'Pull request validation'
    }
}
```

### Hands-On Practice
1. Implement parallel stages
2. Add approval step
3. Use credentials in pipeline
4. Add conditional stages
5. Test different scenarios

---

## Week 5: Git Integration

### Learning Objectives
- Configure Git integration
- Set up webhooks
- Implement multi-branch pipelines
- Handle PR decoration

### Topics

#### 5.1 Git Configuration

**Basic Git Checkout:**
```groovy
stage('Checkout') {
    steps {
        git branch: 'main',
            url: 'https://github.com/user/repo.git'
    }
}
```

**With Credentials:**
```groovy
stage('Checkout') {
    steps {
        git branch: 'main',
            credentialsId: 'github-credentials',
            url: 'https://github.com/user/repo.git'
    }
}
```

#### 5.2 Multibranch Pipeline

**Jenkinsfile (in repo):**
```groovy
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                echo "Building branch: ${env.BRANCH_NAME}"
            }
        }
    }
}
```

**Configuration:**
- New Item → Multibranch Pipeline
- Branch Sources → Git
- Repository URL
- Credentials
- Behaviors → Discover branches/PRs

#### 5.3 GitHub Integration

**Install Plugins:**
- GitHub Integration Plugin
- GitHub Branch Source Plugin

**Configure:**
```groovy
properties([
    githubProjectProperty(
        displayName: '',
        projectUrlStr: 'https://github.com/user/repo'
    )
])
```

**Webhook:**
- GitHub: Settings → Webhooks
- Payload URL: `http://jenkins/github-webhook/`
- Content type: application/json
- Events: Push, Pull request

### Hands-On Practice
1. Create multibranch pipeline
2. Configure GitHub webhook
3. Test branch builds
4. Test PR builds
5. Add status checks

---

## Week 6: Maven & SonarQube Integration

### Learning Objectives
- Integrate Maven builds
- Configure SonarQube scanning
- Implement quality gates
- Handle build artifacts

### Topics

#### 6.1 Maven Integration

**Complete Maven Pipeline:**
```groovy
pipeline {
    agent any
    
    tools {
        maven 'Maven 3.8'
        jdk 'JDK 11'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/user/maven-project.git'
            }
        }
        
        stage('Compile') {
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
    }
}
```

#### 6.2 SonarQube Integration

**Complete SonarQube Pipeline:**
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
                git 'https://github.com/user/repo.git'
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
    
    post {
        success {
            echo 'Build passed quality gate!'
        }
        failure {
            echo 'Build failed quality gate!'
        }
    }
}
```

### Hands-On Practice
1. Configure Maven in Jenkins
2. Create Maven build pipeline
3. Configure SonarQube server
4. Add quality gate check
5. Test quality gate failure

---

## Week 7: Docker Integration

### Learning Objectives
- Build Docker images in Jenkins
- Use Docker agents
- Push images to registry
- Implement multi-stage builds

### Topics

#### 7.1 Docker Plugin Configuration

**Install Plugin:**
- Docker Pipeline Plugin

**Configure Docker:**
- Manage Jenkins → Configure System
- Add Docker installation

#### 7.2 Building Docker Images

**Basic Docker Build:**
```groovy
stage('Build Docker Image') {
    steps {
        script {
            docker.build("myapp:${BUILD_NUMBER}")
        }
    }
}
```

**Complete Docker Pipeline:**
```groovy
pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'registry.example.com'
        DOCKER_IMAGE = "${DOCKER_REGISTRY}/myapp"
        DOCKER_CREDS = credentials('docker-credentials')
    }
    
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/user/repo.git'
            }
        }
        
        stage('Build Application') {
            agent {
                docker {
                    image 'maven:3.8-jdk-11'
                }
            }
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE}:${BUILD_NUMBER}")
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry("https://${DOCKER_REGISTRY}", 'docker-credentials') {
                        dockerImage.push("${BUILD_NUMBER}")
                        dockerImage.push("latest")
                    }
                }
            }
        }
    }
    
    post {
        always {
            sh 'docker rmi ${DOCKER_IMAGE}:${BUILD_NUMBER} || true'
        }
    }
}
```

### Hands-On Practice
1. Install Docker plugin
2. Create Dockerfile
3. Build image in pipeline
4. Push to Docker Hub
5. Clean up images

---

## Week 8: Advanced Docker & Containerization

### Learning Objectives
- Use Docker agents
- Implement Docker Compose
- Multi-stage Docker builds
- Security scanning

### Topics

#### 8.1 Docker Agents

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

#### 8.2 Complete CI/CD with Docker

```groovy
pipeline {
    agent any
    
    environment {
        REGISTRY = 'registry.example.com'
        IMAGE = "${REGISTRY}/myapp"
        TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build App') {
            agent {
                docker {
                    image 'maven:3.8-jdk-11'
                    args '-v maven-repo:/root/.m2'
                }
            }
            steps {
                sh 'mvn clean package'
                stash includes: 'target/*.jar', name: 'app'
            }
        }
        
        stage('Test') {
            agent {
                docker 'maven:3.8-jdk-11'
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
        
        stage('SonarQube') {
            agent {
                docker 'maven:3.8-jdk-11'
            }
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage('Build Image') {
            steps {
                unstash 'app'
                script {
                    docker.build("${IMAGE}:${TAG}")
                }
            }
        }
        
        stage('Scan Image') {
            steps {
                sh "trivy image ${IMAGE}:${TAG}"
            }
        }
        
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
    }
}
```

### Hands-On Practice
1. Use Docker as build agent
2. Implement complete Docker pipeline
3. Add image scanning
4. Push to private registry
5. Optimize build performance

---

## Week 9: Kubernetes Deployment

### Learning Objectives
- Deploy to Kubernetes from Jenkins
- Implement rolling updates
- Use Helm charts
- Configure kubectl

### Topics

#### 9.1 Kubernetes Plugin Setup

**Install:**
- Kubernetes Plugin
- Kubernetes CLI Plugin

**Configure:**
- Add kubeconfig credentials
- Configure Kubernetes cloud

#### 9.2 Basic Kubernetes Deployment

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

#### 9.3 Complete K8s Pipeline

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
        stage('Build & Push Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE}:${TAG}").push()
                }
            }
        }
        
        stage('Update K8s Manifests') {
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
        
        stage('Verify Deployment') {
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
                    echo "Rolling back deployment"
                    kubectl rollout undo deployment/myapp -n ${K8S_NAMESPACE}
                '''
            }
        }
    }
}
```

### Hands-On Practice
1. Configure kubeconfig in Jenkins
2. Create Kubernetes manifests
3. Deploy application to K8s
4. Implement rollback strategy
5. Add smoke tests

---

## Week 10: Helm & GitOps

### Learning Objectives
- Use Helm in Jenkins
- Implement GitOps patterns
- Manage multiple environments
- Automated deployments

### Topics

#### 10.1 Helm Deployment

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

#### 10.2 Multi-Environment Deployment

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
                            helm upgrade --install myapp ./helm/myapp \
                              --namespace ${namespace} \
                              --set image.tag=${BUILD_NUMBER} \
                              --set replicaCount=${replicas} \
                              --values helm/values-${params.ENVIRONMENT}.yaml
                        """
                    }
                }
            }
        }
    }
}
```

### Hands-On Practice
1. Install Helm
2. Create Helm chart
3. Deploy with Helm
4. Implement multi-environment
5. Add environment-specific values

---

## Week 11: Production Best Practices

### Learning Objectives
- Implement security best practices
- Set up distributed builds
- Configure backup and recovery
- Monitor Jenkins

### Topics

#### 11.1 Security

**Authentication:**
- Enable security
- Configure user database
- LDAP/AD integration
- OAuth (GitHub, Google)

**Authorization:**
- Matrix-based security
- Project-based security
- Role-based access

**Credentials:**
- Use credential store
- Rotate credentials
- Encrypt secrets

#### 11.2 Distributed Builds

**Configure Agents:**
```groovy
pipeline {
    agent {
        label 'linux && docker'
    }
    
    stages {
        stage('Build') {
            steps {
                sh 'echo Building on agent'
            }
        }
    }
}
```

#### 11.3 Backup Strategy

```bash
# Backup Jenkins home
tar -czf jenkins-backup-$(date +%Y%m%d).tar.gz \
  /var/lib/jenkins/

# Backup to S3
aws s3 cp jenkins-backup.tar.gz s3://backup-bucket/
```

### Hands-On Practice
1. Configure security
2. Set up agents
3. Implement backup
4. Test disaster recovery
5. Monitor Jenkins

---

## Week 12: Advanced Topics

### Learning Objectives
- Create shared libraries
- Implement advanced patterns
- Optimize performance
- Troubleshoot issues

### Topics

#### 12.1 Shared Libraries

**Library Structure:**
```
jenkins-shared-library/
├── vars/
│   └── buildAndDeploy.groovy
└── src/
    └── org/
        └── example/
            └── Utils.groovy
```

**Usage:**
```groovy
@Library('jenkins-shared-library') _

pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                buildAndDeploy()
            }
        }
    }
}
```

#### 12.2 Performance Optimization

- Use lightweight agents
- Parallel execution
- Caching strategies
- Clean old builds
- Optimize plugins

### Hands-On Practice
1. Create shared library
2. Implement advanced pipelines
3. Optimize build performance
4. Troubleshoot common issues
5. Document patterns

---

## Summary

After 12 weeks, you will have mastered:
✅ Jenkins installation and configuration  
✅ Declarative and scripted pipelines  
✅ Complete DevOps tool integration  
✅ Docker and Kubernetes deployment  
✅ Production best practices  
✅ Security and optimization  

**You now have COMPLETE DevOps Pipeline Orchestration skills! 🚀**


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

