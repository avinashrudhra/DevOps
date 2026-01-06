# Jenkins Hands-On Exercises

Complete practical exercises to master Jenkins with full DevOps tool integration.

---

## Beginner Level Exercises

### Exercise 1: Install Jenkins and Create First Job
**Objective**: Set up Jenkins environment and create first freestyle job

**Tasks:**
1. Install Jenkins (Docker or native)
2. Complete initial setup
3. Create freestyle job
4. Run first build

<details>
<summary>Solution</summary>

```bash
# Install Jenkins with Docker
docker run -d \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  --name jenkins \
  jenkins/jenkins:lts

# Get initial password
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword

# Access http://localhost:8080
# Complete setup wizard
# Install suggested plugins
# Create admin user
```

**Create Freestyle Job:**
1. New Item → Freestyle project → Name: "hello-world"
2. Build Steps → Execute shell
```bash
echo "Hello from Jenkins!"
echo "Build Number: $BUILD_NUMBER"
echo "Job Name: $JOB_NAME"
date
```
3. Save and Build Now
4. Check Console Output

**Verification:**
- Jenkins accessible
- Job created successfully
- Build completes with SUCCESS
- Console shows output
</details>

---

### Exercise 2: Create Your First Pipeline
**Objective**: Write basic declarative pipeline

**Tasks:**
1. Create pipeline job
2. Write Jenkinsfile
3. Add multiple stages
4. Run pipeline

<details>
<summary>Solution</summary>

**Create Pipeline Job:**
1. New Item → Pipeline → Name: "first-pipeline"
2. Pipeline section → Pipeline script

**Jenkinsfile:**
```groovy
pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code...'
            }
        }
        
        stage('Build') {
            steps {
                echo 'Building application...'
                sh 'echo "Compiling source code"'
            }
        }
        
        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'echo "Running unit tests"'
            }
        }
        
        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                sh 'echo "Deployment complete"'
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

3. Save and Build Now
4. View Stage View
5. Check Blue Ocean UI (if installed)
</details>

---

### Exercise 3: Git Integration
**Objective**: Integrate Jenkins with Git repository

**Tasks:**
1. Create Git repository
2. Add Jenkinsfile to repo
3. Configure pipeline from SCM
4. Test automatic builds

<details>
<summary>Solution</summary>

**Create Repository:**
```bash
# Create local repo
mkdir myapp
cd myapp
git init

# Create Jenkinsfile
cat > Jenkinsfile <<'EOF'
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                echo "Building from Git: ${env.GIT_BRANCH}"
                echo "Commit: ${env.GIT_COMMIT}"
            }
        }
    }
}
EOF

# Commit
git add Jenkinsfile
git commit -m "Add Jenkinsfile"

# Push to GitHub (create repo first)
git remote add origin https://github.com/user/myapp.git
git push -u origin main
```

**Configure Jenkins:**
1. New Item → Pipeline → "git-pipeline"
2. Pipeline → Definition: Pipeline script from SCM
3. SCM: Git
4. Repository URL: https://github.com/user/myapp.git
5. Branch: */main
6. Script Path: Jenkinsfile
7. Save and Build

**Verification:**
- Pipeline runs successfully
- Uses Jenkinsfile from repo
- Shows Git information
</details>

---

## Intermediate Level Exercises

### Exercise 4: Complete Maven Build Pipeline
**Objective**: Build Java application with Maven

**Tasks:**
1. Create Maven project
2. Write comprehensive build pipeline
3. Run tests and generate reports
4. Archive artifacts

<details>
<summary>Solution</summary>

**Create Maven Project:**
```bash
# Create simple Maven project
mvn archetype:generate \
  -DgroupId=com.example \
  -DartifactId=myapp \
  -DarchetypeArtifactId=maven-archetype-quickstart \
  -DinteractiveMode=false

cd myapp
git init
git add .
git commit -m "Initial commit"
```

**Jenkinsfile:**
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
                checkout scm
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
    
    post {
        always {
            cleanWs()
        }
    }
}
```

**Configure Tools:**
1. Manage Jenkins → Global Tool Configuration
2. Add JDK: Name "JDK 11", Install automatically
3. Add Maven: Name "Maven 3.8", Install automatically
4. Save

**Run Pipeline:**
- Push Jenkinsfile to repo
- Configure pipeline job
- Build and verify artifacts
</details>

---

### Exercise 5: SonarQube Integration
**Objective**: Add code quality scanning with quality gates

**Tasks:**
1. Set up SonarQube server
2. Configure Jenkins-SonarQube integration
3. Add analysis to pipeline
4. Implement quality gate check

<details>
<summary>Solution</summary>

**Start SonarQube:**
```bash
docker run -d \
  --name sonarqube \
  -p 9000:9000 \
  sonarqube:lts

# Access http://localhost:9000
# Login: admin/admin
# Generate token: My Account → Security → Generate Token
```

**Configure Jenkins:**
1. Install "SonarQube Scanner" plugin
2. Manage Jenkins → Configure System → SonarQube servers
   - Name: SonarQube
   - Server URL: http://sonarqube:9000
   - Server authentication token: (add as Secret text)

**Complete Pipeline:**
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
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        mvn sonar:sonar \
                          -Dsonar.projectKey=myapp \
                          -Dsonar.projectName=myapp
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
                            error "Quality gate failed: ${qg.status}"
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

**Configure Webhook:**
- SonarQube → Administration → Configuration → Webhooks
- URL: http://jenkins:8080/sonarqube-webhook/
- Save

**Test:**
- Run pipeline
- Check SonarQube dashboard
- Verify quality gate check
</details>

---

### Exercise 6: Docker Image Build
**Objective**: Build and push Docker images

**Tasks:**
1. Create Dockerfile
2. Build image in Jenkins
3. Push to Docker Hub
4. Clean up images

<details>
<summary>Solution</summary>

**Create Dockerfile:**
```dockerfile
FROM openjdk:11-jre-slim
WORKDIR /app
COPY target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Complete Pipeline:**
```groovy
pipeline {
    agent any
    
    environment {
        DOCKER_HUB = credentials('dockerhub-credentials')
        IMAGE_NAME = "username/myapp"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    
    tools {
        maven 'Maven 3.8'
    }
    
    stages {
        stage('Build Application') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials') {
                        dockerImage.push("${IMAGE_TAG}")
                        dockerImage.push("latest")
                    }
                }
            }
        }
    }
    
    post {
        always {
            sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true"
            sh "docker rmi ${IMAGE_NAME}:latest || true"
        }
    }
}
```

**Add Docker Hub Credentials:**
1. Manage Jenkins → Manage Credentials
2. Add Credentials → Username with password
3. ID: dockerhub-credentials
4. Username: your-dockerhub-username
5. Password: your-dockerhub-token

**Test:**
- Run pipeline
- Verify image on Docker Hub
- Check local images cleaned up
</details>

---

## Advanced Level Exercises

### Exercise 7: Complete CI/CD Pipeline (All Tools)
**Objective**: Implement end-to-end pipeline integrating all DevOps tools

**Tasks:**
1. Git checkout
2. Maven build
3. SonarQube scan
4. Docker build
5. Push to registry
6. Deploy to Kubernetes

<details>
<summary>Solution</summary>

**Complete Jenkinsfile:**
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
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    env.GIT_COMMIT_SHORT = sh(
                        script: "git rev-parse --short HEAD",
                        returnStdout: true
                    ).trim()
                }
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }
        
        stage('Unit Tests') {
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
        
        stage('SonarQube Analysis') {
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
        
        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${IMAGE}:${TAG}")
                }
            }
        }
        
        stage('Security Scan') {
            steps {
                sh "trivy image --severity HIGH,CRITICAL ${IMAGE}:${TAG}"
            }
        }
        
        stage('Push Image') {
            steps {
                script {
                    docker.withRegistry("https://${REGISTRY}", 'docker-creds') {
                        dockerImage.push("${TAG}")
                        dockerImage.push("latest")
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
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
        
        stage('Verify Deployment') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh "kubectl get pods -n ${K8S_NAMESPACE}"
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
        success {
            slackSend color: 'good',
                     message: "✅ Build ${BUILD_NUMBER} deployed successfully!"
        }
        failure {
            slackSend color: 'danger',
                     message: "❌ Build ${BUILD_NUMBER} failed!"
            
            script {
                if (env.STAGE_NAME == 'Deploy to Kubernetes') {
                    withKubeConfig([credentialsId: 'kubeconfig']) {
                        sh "kubectl rollout undo deployment/myapp -n ${K8S_NAMESPACE}"
                    }
                }
            }
        }
        always {
            cleanWs()
        }
    }
}
```

**Prerequisites:**
- All tools configured (Git, Maven, SonarQube, Docker, Kubernetes)
- All credentials added
- Kubernetes cluster accessible

**Test Complete Pipeline:**
1. Push code change
2. Pipeline triggers automatically
3. All stages execute
4. Application deployed to Kubernetes
5. Notification sent
</details>

---

### Exercise 8: Multi-Branch Pipeline
**Objective**: Set up automated branch and PR builds

**Tasks:**
1. Configure multibranch pipeline
2. Test multiple branches
3. Configure PR builds
4. Add branch-specific logic

<details>
<summary>Solution</summary>

**Jenkinsfile (in repository):**
```groovy
pipeline {
    agent any
    
    tools {
        maven 'Maven 3.8'
    }
    
    stages {
        stage('Info') {
            steps {
                echo "Branch: ${env.BRANCH_NAME}"
                echo "Change ID: ${env.CHANGE_ID}"
                echo "Change Target: ${env.CHANGE_TARGET}"
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('Deploy to Dev') {
            when {
                branch 'develop'
            }
            steps {
                echo 'Deploying to dev environment'
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'staging'
            }
            steps {
                echo 'Deploying to staging environment'
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to production?'
                echo 'Deploying to production'
            }
        }
        
        stage('PR Checks') {
            when {
                changeRequest()
            }
            steps {
                echo 'Running PR validation'
                sh 'mvn verify'
            }
        }
    }
}
```

**Configure Multibranch Pipeline:**
1. New Item → Multibranch Pipeline → "myapp-multibranch"
2. Branch Sources → Add source → Git
3. Project Repository: https://github.com/user/repo.git
4. Credentials: (add if private)
5. Behaviors:
   - Discover branches
   - Discover pull requests from origin
6. Build Configuration: by Jenkinsfile
7. Save

**Test:**
```bash
# Create feature branch
git checkout -b feature/new-feature
git push origin feature/new-feature

# Create pull request on GitHub
# Jenkins automatically creates job and builds

# Merge to main
# Jenkins builds and prompts for production deployment
```
</details>

---

### Exercise 9: Parameterized Builds
**Objective**: Create flexible pipeline with parameters

**Tasks:**
1. Add build parameters
2. Use parameters in pipeline
3. Test different configurations

<details>
<summary>Solution</summary>

```groovy
pipeline {
    agent any
    
    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'staging', 'prod'],
            description: 'Target environment'
        )
        
        string(
            name: 'VERSION',
            defaultValue: '1.0.0',
            description: 'Version to deploy'
        )
        
        booleanParam(
            name: 'RUN_TESTS',
            defaultValue: true,
            description: 'Run tests?'
        )
        
        booleanParam(
            name: 'DEPLOY',
            defaultValue: false,
            description: 'Deploy after build?'
        )
        
        text(
            name: 'NOTES',
            defaultValue: '',
            description: 'Release notes'
        )
    }
    
    stages {
        stage('Info') {
            steps {
                echo "Environment: ${params.ENVIRONMENT}"
                echo "Version: ${params.VERSION}"
                echo "Run Tests: ${params.RUN_TESTS}"
                echo "Deploy: ${params.DEPLOY}"
                echo "Notes: ${params.NOTES}"
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn clean package -Dversion=${params.VERSION}"
            }
        }
        
        stage('Test') {
            when {
                expression { params.RUN_TESTS }
            }
            steps {
                sh 'mvn test'
            }
        }
        
        stage('Deploy') {
            when {
                expression { params.DEPLOY }
            }
            steps {
                script {
                    def namespace = params.ENVIRONMENT
                    echo "Deploying version ${params.VERSION} to ${namespace}"
                    // Deploy logic here
                }
            }
        }
    }
}
```

**Test:**
- First build creates parameters
- Build with Parameters
- Try different combinations
- Verify conditional stages
</details>

---

### Exercise 10: Parallel Execution
**Objective**: Speed up builds with parallel stages

**Tasks:**
1. Identify parallelizable stages
2. Implement parallel execution
3. Measure performance improvement

<details>
<summary>Solution</summary>

```groovy
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }
        
        stage('Parallel Tests') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'mvn test'
                    }
                    post {
                        always {
                            junit '**/target/surefire-reports/*.xml'
                        }
                    }
                }
                
                stage('Integration Tests') {
                    steps {
                        sh 'mvn verify -DskipUnitTests'
                    }
                }
                
                stage('Security Scan') {
                    steps {
                        sh 'mvn dependency-check:check'
                    }
                }
                
                stage('Code Quality') {
                    steps {
                        withSonarQubeEnv('SonarQube') {
                            sh 'mvn sonar:sonar'
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

**Measure Performance:**
```groovy
stage('Measure') {
    steps {
        script {
            def startTime = System.currentTimeMillis()
            // ... build steps
            def endTime = System.currentTimeMillis()
            def duration = (endTime - startTime) / 1000
            echo "Build took ${duration} seconds"
        }
    }
}
```
</details>

---

## Production Exercises

### Exercise 11: Blue-Green Deployment
**Objective**: Implement zero-downtime deployment

<details>
<summary>Solution</summary>

```groovy
stage('Blue-Green Deploy') {
    steps {
        withKubeConfig([credentialsId: 'kubeconfig']) {
            script {
                // Deploy green version
                sh """
                    kubectl apply -f k8s/deployment-green.yaml
                    kubectl rollout status deployment/myapp-green -n production
                """
                
                // Run smoke tests on green
                sh 'curl -f http://myapp-green.example.com/health'
                
                // Switch traffic to green
                input message: 'Switch traffic to green?'
                
                sh """
                    kubectl patch service myapp -n production \
                      -p '{"spec":{"selector":{"version":"green"}}}'
                """
                
                // Wait and monitor
                sleep 60
                
                // Remove blue version
                sh 'kubectl delete deployment myapp-blue -n production'
            }
        }
    }
}
```
</details>

---

### Exercise 12: Shared Library
**Objective**: Create reusable pipeline components

<details>
<summary>Solution</summary>

**Library Structure:**
```
jenkins-shared-library/
├── vars/
│   ├── standardPipeline.groovy
│   └── deployToK8s.groovy
└── src/
    └── org/
        └── example/
            └── Utils.groovy
```

**vars/standardPipeline.groovy:**
```groovy
def call(Map config) {
    pipeline {
        agent any
        
        tools {
            maven 'Maven 3.8'
        }
        
        stages {
            stage('Build') {
                steps {
                    sh "mvn clean package -DappName=${config.appName}"
                }
            }
            
            stage('Test') {
                steps {
                    sh 'mvn test'
                }
            }
            
            stage('Deploy') {
                steps {
                    deployToK8s(
                        namespace: config.namespace,
                        image: config.image
                    )
                }
            }
        }
    }
}
```

**Usage in Jenkinsfile:**
```groovy
@Library('jenkins-shared-library') _

standardPipeline(
    appName: 'myapp',
    namespace: 'production',
    image: 'myapp:latest'
)
```
</details>

---

## Tips for Practice

1. **Start Simple**: Master basics before advanced topics
2. **Use Version Control**: Always store Jenkinsfiles in Git
3. **Test Locally**: Use Docker for local Jenkins testing
4. **Read Logs**: Console output is your friend
5. **Incremental Changes**: Test one change at a time
6. **Use Blue Ocean**: Better visualization
7. **Monitor Performance**: Track build times
8. **Security First**: Use credentials management
9. **Clean Up**: Remove old builds and artifacts
10. **Document**: Comment complex pipeline logic

---

## Next Steps

1. Complete all exercises in order
2. Practice with real applications
3. Study [Learning Roadmap](jenkins-learning-roadmap.md)
4. Review [Troubleshooting Guide](jenkins-troubleshooting-guide.md)
5. Prepare with [Interview Questions](jenkins-interview-questions.md)
6. Implement in production

**You now have complete CI/CD expertise! 🔧🚀**


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

