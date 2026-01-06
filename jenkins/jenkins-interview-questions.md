# Jenkins Interview Questions

Complete interview preparation for Jenkins professionals (7+ years experience) with focus on production environments and full DevOps tool integration.

---

## Table of Contents
1. [Core Jenkins Concepts](#core-jenkins-concepts)
2. [Pipeline as Code](#pipeline-as-code)
3. [Git Integration](#git-integration)
4. [Maven & Build Tools](#maven--build-tools)
5. [SonarQube Integration](#sonarqube-integration)
6. [Docker Integration](#docker-integration)
7. [Kubernetes Deployment](#kubernetes-deployment)
8. [Complete CI/CD Pipeline](#complete-cicd-pipeline)
9. [Security & Best Practices](#security--best-practices)
10. [Performance & Scalability](#performance--scalability)
11. [Troubleshooting & Debugging](#troubleshooting--debugging)
12. [Architecture & Design](#architecture--design)
13. [Real-World Scenarios](#real-world-scenarios)

---

## Core Jenkins Concepts

### Q1: What is Jenkins and why is it used in DevOps?
**Answer:**
Jenkins is an open-source automation server written in Java that enables continuous integration and continuous delivery (CI/CD). It's the most popular CI/CD tool used in DevOps because:

**Key Features:**
- **Automation**: Automates build, test, and deployment processes
- **Integration**: 1800+ plugins for integrating with virtually any tool
- **Distributed**: Master-slave architecture for scalability
- **Pipeline as Code**: Jenkinsfile enables version-controlled pipelines
- **Extensibility**: Plugin ecosystem and Groovy scripting

**Why Jenkins in DevOps:**
- Orchestrates entire DevOps toolchain (Git, Maven, SonarQube, Docker, Kubernetes)
- Provides visibility into build and deployment status
- Enables rapid feedback loops
- Automates repetitive tasks
- Integrates with notification systems (Slack, Email)

---

### Q2: Explain Jenkins architecture in detail
**Answer:**

**Core Components:**

1. **Jenkins Master:**
   - Central coordinator
   - Schedules builds
   - Dispatches jobs to agents
   - Monitors agent status
   - Records and presents results
   - Manages configuration

2. **Jenkins Agents (Slaves):**
   - Execute jobs dispatched by master
   - Can be on different platforms (Linux, Windows, macOS)
   - Connected via SSH, JNLP, or other protocols
   - Labeled for specific job types

3. **Executor:**
   - Slot for executing a job
   - Each agent has configured number of executors
   - Master also has executors (best practice: set to 0)

4. **Job/Project:**
   - Runnable task configured in Jenkins
   - Types: Freestyle, Pipeline, Multibranch, Organization Folder

5. **Build:**
   - Single execution of a job
   - Has number, status, console output, artifacts

6. **Workspace:**
   - Directory on agent where build executes
   - Contains checked-out source code
   - Cleaned between builds (configurable)

**Architecture Diagram:**
```
┌─────────────────────────────────────────────┐
│            Jenkins Master                    │
│  ┌────────────┐  ┌──────────────────────┐  │
│  │   Plugins  │  │   Job Configurations │  │
│  └────────────┘  └──────────────────────┘  │
│  ┌────────────────────────────────────────┐ │
│  │        Build Queue                      │ │
│  └────────────────────────────────────────┘ │
└────────┬────────────────────────┬───────────┘
         │                        │
    ┌────▼─────┐            ┌────▼─────┐
    │ Agent 1  │            │ Agent 2  │
    │ (Linux)  │            │ (Windows)│
    │ 2 Exec.  │            │ 2 Exec.  │
    └──────────┘            └──────────┘
```

---

### Q3: What are the different types of Jenkins jobs?
**Answer:**

**1. Freestyle Project:**
- Traditional job type
- GUI-based configuration
- Good for simple tasks
- Limited flexibility

**Example Use Case:**
```
Build Steps:
1. Execute shell: git clone ...
2. Invoke Maven: clean package
3. Archive artifacts: target/*.jar
```

**2. Pipeline:**
- Define workflow as code
- Jenkinsfile in repository
- Declarative or Scripted syntax
- Full control over build process

**3. Multibranch Pipeline:**
- Automatically creates pipeline for each branch
- Detects Jenkinsfile in repository
- Creates jobs for pull requests
- Perfect for GitFlow workflow

**Configuration:**
```
Branch Sources: Git
Repository URL: https://github.com/user/repo
Behaviors: Discover branches, Discover PRs
Script Path: Jenkinsfile
```

**4. Organization Folder:**
- Scans entire GitHub/GitLab organization
- Creates multibranch pipelines for each repository
- Automatic project discovery
- Enterprise-scale automation

**5. Multi-configuration Project:**
- Run same build with different configurations
- Test on multiple platforms
- Matrix builds

**6. External Job:**
- Monitor externally run jobs
- Legacy integration

**Production Recommendation:**
For production environments with 7+ years experience, **always use Pipeline or Multibranch Pipeline**. They provide:
- Version control
- Code review
- Reusability
- Testability
- Full programmatic control

---

### Q4: Explain Jenkins plugins you've used in production
**Answer:**

**Essential Plugins (Production Environment):**

**1. Pipeline Plugins:**
- **Pipeline**: Core pipeline functionality
- **Pipeline: Stage View**: Visual pipeline representation
- **Blue Ocean**: Modern UI for pipelines

**2. SCM Plugins:**
- **Git Plugin**: Git integration
- **GitHub Plugin**: GitHub-specific features
- **GitLab Plugin**: GitLab integration
- **Bitbucket Plugin**: Bitbucket integration

**3. Build Tools:**
- **Maven Integration**: Maven build tool
- **Gradle Plugin**: Gradle builds
- **NodeJS Plugin**: Node.js projects

**4. Code Quality:**
- **SonarQube Scanner**: Code quality analysis
- **JaCoCo**: Code coverage
- **Checkstyle**, **PMD**, **FindBugs**: Static analysis

**5. Containerization:**
- **Docker Pipeline**: Docker operations in pipeline
- **Kubernetes**: K8s deployment
- **Amazon ECR**: AWS container registry

**6. Credentials:**
- **Credentials Binding**: Bind credentials to variables
- **HashiCorp Vault**: External secret management

**7. Notifications:**
- **Slack Notification**: Slack integration
- **Email Extension**: Enhanced email notifications
- **Teams**: Microsoft Teams integration

**8. Quality Gates:**
- **Quality Gates**: Pipeline quality checks

**9. Security:**
- **OWASP Dependency-Check**: Security vulnerabilities
- **Role-based Authorization Strategy**: RBAC

**10. Utilities:**
- **Config File Provider**: Manage configuration files
- **Build Timeout**: Prevent hanging builds
- **Timestamper**: Add timestamps to console
- **AnsiColor**: Color console output

**Production Example:**
```groovy
// Using multiple plugins in single pipeline
@Library('shared-library') _
pipeline {
    agent {
        kubernetes {
            yaml '''
                apiVersion: v1
                kind: Pod
                spec:
                  containers:
                  - name: maven
                    image: maven:3.8-jdk-11
                  - name: docker
                    image: docker:20.10
            '''
        }
    }
    stages {
        stage('Build') {
            steps {
                // Maven Integration Plugin
                container('maven') {
                    sh 'mvn clean package'
                }
            }
        }
        stage('Quality') {
            steps {
                // SonarQube Scanner Plugin
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
        stage('Docker') {
            steps {
                // Docker Pipeline Plugin
                container('docker') {
                    script {
                        docker.build("myapp:${BUILD_NUMBER}")
                    }
                }
            }
        }
    }
    post {
        always {
            // Slack Notification Plugin
            slackSend color: 'good',
                     message: "Build ${currentBuild.result}"
        }
    }
}
```

---

## Pipeline as Code

### Q5: What's the difference between Declarative and Scripted Pipeline?
**Answer:**

**Declarative Pipeline:**

**Pros:**
- Simpler, more structured syntax
- Better for most use cases
- Easier to learn
- Built-in syntax validation
- Supports pipeline syntax generator

**Syntax:**
```groovy
pipeline {
    agent any
    
    environment {
        VERSION = '1.0'
    }
    
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
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
    }
    
    post {
        success {
            echo 'Success!'
        }
    }
}
```

**Scripted Pipeline:**

**Pros:**
- Full Groovy power
- Maximum flexibility
- Better for complex logic
- No structural limitations

**Syntax:**
```groovy
node {
    def version = '1.0'
    
    try {
        stage('Build') {
            sh 'mvn clean package'
        }
        
        stage('Test') {
            sh 'mvn test'
        }
        
        currentBuild.result = 'SUCCESS'
    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    } finally {
        junit '**/target/surefire-reports/*.xml'
    }
}
```

**Key Differences:**

| Feature | Declarative | Scripted |
|---------|-------------|----------|
| Syntax | Structured | Free-form |
| Learning Curve | Easy | Harder |
| Flexibility | Limited | Full |
| Validation | Built-in | None |
| Recommended For | Most cases | Complex logic |

**Production Recommendation:**
Use **Declarative** for 90% of pipelines. Use **Scripted** sections within Declarative when needed:

```groovy
pipeline {
    agent any
    stages {
        stage('Complex Logic') {
            steps {
                script {
                    // Scripted syntax here
                    if (env.BRANCH_NAME == 'main') {
                        // Complex logic
                    }
                }
            }
        }
    }
}
```

---

### Q6: Explain pipeline best practices in production
**Answer:**

**1. Always Use Version Control:**
```groovy
// ✅ Jenkinsfile in repository
// Not configured in Jenkins UI
```

**2. Use Shared Libraries:**
```groovy
// vars/standardPipeline.groovy
def call(Map config) {
    pipeline {
        agent any
        stages {
            stage('Build') {
                steps {
                    sh "mvn clean package -Dapp=${config.appName}"
                }
            }
        }
    }
}

// Jenkinsfile
@Library('jenkins-shared-library') _
standardPipeline(
    appName: 'myapp',
    environment: 'production'
)
```

**3. Implement Proper Error Handling:**
```groovy
pipeline {
    agent any
    stages {
        stage('Deploy') {
            steps {
                script {
                    try {
                        sh 'kubectl apply -f deployment.yaml'
                    } catch (Exception e) {
                        echo "Deployment failed: ${e.message}"
                        sh 'kubectl rollout undo deployment/myapp'
                        error("Rolled back due to failure")
                    }
                }
            }
        }
    }
}
```

**4. Use Credentials Properly:**
```groovy
// ❌ Never hardcode
def password = "mysecretpassword"

// ✅ Use credentials binding
withCredentials([
    string(credentialsId: 'db-password', variable: 'DB_PASS')
]) {
    sh 'mysql -u user -p$DB_PASS < schema.sql'
}
```

**5. Implement Timeouts:**
```groovy
options {
    timeout(time: 1, unit: 'HOURS')
}

stage('Long Operation') {
    steps {
        timeout(time: 10, unit: 'MINUTES') {
            sh './long-running-script.sh'
        }
    }
}
```

**6. Clean Up Resources:**
```groovy
post {
    always {
        // Clean workspace
        cleanWs()
        
        // Remove Docker images
        sh 'docker system prune -f'
        
        // Archive logs
        archiveArtifacts artifacts: '**/*.log',
                       allowEmptyArchive: true
    }
}
```

**7. Use Parallel Execution:**
```groovy
stage('Tests') {
    parallel {
        stage('Unit') {
            steps { sh 'mvn test' }
        }
        stage('Integration') {
            steps { sh 'mvn verify -DskipUnitTests' }
        }
        stage('Security') {
            steps { sh 'mvn dependency-check:check' }
        }
    }
}
```

**8. Implement Build Caching:**
```groovy
stage('Build') {
    steps {
        sh '''
            # Use persistent Maven cache
            mvn -Dmaven.repo.local=/cache/.m2 clean package
        '''
    }
}
```

**9. Add Notifications:**
```groovy
post {
    success {
        slackSend color: 'good',
                 channel: '#deployments',
                 message: "✅ ${env.JOB_NAME} #${env.BUILD_NUMBER} succeeded"
    }
    failure {
        slackSend color: 'danger',
                 channel: '#alerts',
                 message: "❌ ${env.JOB_NAME} #${env.BUILD_NUMBER} failed"
    }
}
```

**10. Use Declarative Directives:**
```groovy
pipeline {
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()
        timestamps()
    }
    
    triggers {
        cron('H 2 * * *')  // Nightly builds
    }
}
```

---

### Q7: How do you handle secrets in Jenkins pipelines?
**Answer:**

**1. Jenkins Credentials:**
```groovy
withCredentials([
    usernamePassword(
        credentialsId: 'docker-creds',
        usernameVariable: 'USER',
        passwordVariable: 'PASS'
    ),
    string(
        credentialsId: 'api-token',
        variable: 'API_TOKEN'
    ),
    file(
        credentialsId: 'kubeconfig',
        variable: 'KUBECONFIG'
    )
]) {
    sh 'docker login -u $USER -p $PASS'
    sh 'kubectl --kubeconfig=$KUBECONFIG get pods'
}
```

**2. HashiCorp Vault:**
```groovy
// Using Vault plugin
def secrets = [
    [path: 'secret/production/db', 
     engineVersion: 2,
     secretValues: [
        [envVar: 'DB_USER', vaultKey: 'username'],
        [envVar: 'DB_PASS', vaultKey: 'password']
    ]]
]

withVault([vaultSecrets: secrets]) {
    sh 'mysql -u $DB_USER -p$DB_PASS'
}
```

**3. Kubernetes Secrets:**
```groovy
stage('Deploy') {
    steps {
        sh '''
            kubectl create secret generic app-secrets \
              --from-literal=db-password=$DB_PASSWORD \
              -n production \
              --dry-run=client -o yaml | kubectl apply -f -
        '''
    }
}
```

**4. AWS Secrets Manager:**
```groovy
stage('Get Secrets') {
    steps {
        script {
            def secret = sh(
                script: """
                    aws secretsmanager get-secret-value \
                      --secret-id prod/db/password \
                      --query SecretString \
                      --output text
                """,
                returnStdout: true
            ).trim()
            
            env.DB_PASSWORD = secret
        }
    }
}
```

**Best Practices:**

1. **Never log secrets:**
```groovy
// ❌ Bad
sh "echo Password: ${PASSWORD}"

// ✅ Good
withCredentials([...]) {
    sh '''
        set +x  # Disable command echoing
        mysql -u user -p$PASSWORD
        set -x
    '''
}
```

2. **Mask output:**
```groovy
wrap([$class: 'MaskPasswordsBuildWrapper',
      varPasswordPairs: [[password: env.SECRET, var: 'SECRET']]]) {
    // steps
}
```

3. **Rotate regularly:**
- Implement automated rotation
- Use short-lived tokens
- Audit secret access

---

## Git Integration

### Q8: How do you implement Git workflow in Jenkins?
**Answer:**

**Production Git Integration:**

**1. Multibranch Pipeline Setup:**
```groovy
// Jenkinsfile
pipeline {
    agent any
    
    stages {
        stage('Info') {
            steps {
                script {
                    echo "Branch: ${env.BRANCH_NAME}"
                    echo "Commit: ${env.GIT_COMMIT}"
                    echo "Author: ${env.GIT_AUTHOR_NAME}"
                    
                    // Determine environment
                    env.DEPLOY_ENV = 'dev'
                    if (env.BRANCH_NAME == 'staging') {
                        env.DEPLOY_ENV = 'staging'
                    } else if (env.BRANCH_NAME == 'main') {
                        env.DEPLOY_ENV = 'production'
                    }
                }
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('Deploy Dev') {
            when {
                branch 'develop'
            }
            steps {
                echo 'Deploying to dev'
                sh './deploy.sh dev'
            }
        }
        
        stage('Deploy Staging') {
            when {
                branch 'staging'
            }
            steps {
                echo 'Deploying to staging'
                sh './deploy.sh staging'
            }
        }
        
        stage('Deploy Production') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to production?', ok: 'Deploy'
                echo 'Deploying to production'
                sh './deploy.sh production'
            }
        }
        
        stage('PR Validation') {
            when {
                changeRequest()
            }
            steps {
                echo "Validating PR #${env.CHANGE_ID}"
                sh 'mvn verify'
                
                // Post comment to PR
                script {
                    pullRequest.comment("✅ Build succeeded for ${env.GIT_COMMIT}")
                }
            }
        }
    }
}
```

**2. GitFlow Workflow:**
```
main (production) ──────●────────●────────●
                        │        │        │
staging (pre-prod) ─────┼────●───┼────●───┼
                        │    │   │    │   │
develop (dev) ──────●───●────●───●────●───●
                    │        │        │
feature/* ──────────●        │        │
                             │        │
hotfix/* ────────────────────●        │
                                      │
bugfix/* ─────────────────────────────●
```

**3. Webhook Configuration:**
```groovy
// GitHub webhook triggers
properties([
    pipelineTriggers([
        githubPush(),
        pollSCM('H/5 * * * *')  // Fallback polling
    ])
])
```

**4. Advanced Git Operations:**
```groovy
stage('Tag Release') {
    when {
        branch 'main'
    }
    steps {
        script {
            sshagent(['github-ssh']) {
                sh """
                    git config user.email "jenkins@example.com"
                    git config user.name "Jenkins"
                    git tag -a v${env.BUILD_NUMBER} \
                      -m "Release ${env.BUILD_NUMBER}"
                    git push origin v${env.BUILD_NUMBER}
                """
            }
        }
    }
}
```

---

### Q9: How do you handle merge conflicts in Jenkins?
**Answer:**

**Strategy 1: Automated Conflict Detection:**
```groovy
stage('Check for Conflicts') {
    steps {
        script {
            try {
                sh """
                    git fetch origin ${env.CHANGE_TARGET}
                    git merge --no-commit --no-ff origin/${env.CHANGE_TARGET}
                    git merge --abort
                """
            } catch (Exception e) {
                pullRequest.comment("⚠️ Merge conflicts detected. Please rebase.")
                error("Merge conflicts found")
            }
        }
    }
}
```

**Strategy 2: Automatic Rebase:**
```groovy
stage('Rebase') {
    when {
        changeRequest()
    }
    steps {
        script {
            sshagent(['github-ssh']) {
                try {
                    sh """
                        git fetch origin ${env.CHANGE_TARGET}
                        git rebase origin/${env.CHANGE_TARGET}
                    """
                } catch (Exception e) {
                    sh 'git rebase --abort'
                    error("Rebase failed - manual intervention required")
                }
            }
        }
    }
}
```

**Strategy 3: Prevention:**
```groovy
// Require up-to-date branches
when {
    expression {
        def ahead = sh(
            script: "git rev-list --count origin/${env.CHANGE_TARGET}..HEAD",
            returnStdout: true
        ).trim()
        
        def behind = sh(
            script: "git rev-list --count HEAD..origin/${env.CHANGE_TARGET}",
            returnStdout: true
        ).trim()
        
        if (behind.toInteger() > 0) {
            pullRequest.comment("⚠️ Branch is ${behind} commits behind. Please rebase.")
            return false
        }
        return true
    }
}
```

---

## Complete CI/CD Pipeline

### Q10: Design a complete CI/CD pipeline integrating all DevOps tools
**Answer:**

**Enterprise-Grade CI/CD Pipeline:**

```groovy
@Library('jenkins-shared-library') _

def deploymentInfo = [:]

pipeline {
    agent any
    
    options {
        buildDiscarder(logRotator(numToKeepStr: '30'))
        timeout(time: 1, unit: 'HOURS')
        timestamps()
        disableConcurrentBuilds()
    }
    
    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'staging', 'production'],
            description: 'Target deployment environment'
        )
        booleanParam(
            name: 'SKIP_TESTS',
            defaultValue: false,
            description: 'Skip test execution'
        )
    }
    
    environment {
        // Tool versions
        MAVEN_VERSION = '3.8.4'
        JDK_VERSION = '11'
        
        // Registry
        DOCKER_REGISTRY = 'registry.example.com'
        IMAGE_NAME = "${DOCKER_REGISTRY}/myapp"
        IMAGE_TAG = "${env.BUILD_NUMBER}-${env.GIT_COMMIT.take(8)}"
        
        // Kubernetes
        K8S_NAMESPACE = "${params.ENVIRONMENT}"
        
        // SonarQube
        SONAR_PROJECT_KEY = 'myapp'
    }
    
    tools {
        maven 'Maven 3.8'
        jdk 'JDK 11'
    }
    
    stages {
        stage('🔍 Initialize') {
            steps {
                script {
                    // Collect build information
                    deploymentInfo = [
                        buildNumber: env.BUILD_NUMBER,
                        gitCommit: env.GIT_COMMIT,
                        gitBranch: env.GIT_BRANCH,
                        gitAuthor: env.GIT_AUTHOR_NAME,
                        startTime: new Date(),
                        environment: params.ENVIRONMENT
                    ]
                    
                    echo """
                    ╔═══════════════════════════════════════╗
                    ║        BUILD INFORMATION              ║
                    ╠═══════════════════════════════════════╣
                    ║ Build: ${deploymentInfo.buildNumber}
                    ║ Branch: ${deploymentInfo.gitBranch}
                    ║ Commit: ${deploymentInfo.gitCommit.take(8)}
                    ║ Author: ${deploymentInfo.gitAuthor}
                    ║ Environment: ${deploymentInfo.environment}
                    ╚═══════════════════════════════════════╝
                    """
                }
            }
        }
        
        stage('📦 Checkout') {
            steps {
                checkout scm
                script {
                    // Verify clean state
                    sh 'git status'
                    sh 'git log -1 --pretty=format:"%h - %an: %s"'
                }
            }
        }
        
        stage('🔨 Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }
        
        stage('🧪 Unit Tests') {
            when {
                expression { !params.SKIP_TESTS }
            }
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                    jacoco execPattern: '**/target/jacoco.exec'
                }
            }
        }
        
        stage('🔍 SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh """
                        mvn sonar:sonar \
                          -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                          -Dsonar.projectName=myapp \
                          -Dsonar.branch.name=${env.BRANCH_NAME}
                    """
                }
            }
        }
        
        stage('✅ Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    script {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Quality gate failed: ${qg.status}"
                        }
                        echo "✅ Quality gate passed!"
                    }
                }
            }
        }
        
        stage('🔐 Security Scan') {
            parallel {
                stage('Dependency Check') {
                    steps {
                        sh 'mvn dependency-check:check'
                        dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                    }
                }
                
                stage('Trivy Filesystem Scan') {
                    steps {
                        sh 'trivy fs --severity HIGH,CRITICAL .'
                    }
                }
            }
        }
        
        stage('📦 Package') {
            steps {
                sh 'mvn package -DskipTests'
                archiveArtifacts artifacts: '**/target/*.jar',
                               fingerprint: true
            }
        }
        
        stage('🐳 Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                    
                    // Tag with latest
                    sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
                }
            }
        }
        
        stage('🔐 Image Security Scan') {
            steps {
                sh """
                    trivy image \
                      --severity HIGH,CRITICAL \
                      --exit-code 1 \
                      ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }
        
        stage('📤 Push Image') {
            steps {
                script {
                    docker.withRegistry("https://${DOCKER_REGISTRY}", 'docker-registry-creds') {
                        dockerImage.push("${IMAGE_TAG}")
                        dockerImage.push("latest")
                    }
                }
            }
        }
        
        stage('🚀 Deploy to Kubernetes') {
            steps {
                script {
                    withKubeConfig([credentialsId: 'kubeconfig']) {
                        sh """
                            # Update deployment
                            kubectl set image deployment/myapp \
                              myapp=${IMAGE_NAME}:${IMAGE_TAG} \
                              -n ${K8S_NAMESPACE} \
                              --record
                            
                            # Wait for rollout
                            kubectl rollout status deployment/myapp \
                              -n ${K8S_NAMESPACE} \
                              --timeout=5m
                            
                            # Verify deployment
                            kubectl get pods -n ${K8S_NAMESPACE} \
                              -l app=myapp
                        """
                    }
                }
            }
        }
        
        stage('🔍 Smoke Tests') {
            steps {
                script {
                    sleep 30  // Wait for service to stabilize
                    
                    def url = "https://myapp.${params.ENVIRONMENT}.example.com/health"
                    def response = sh(
                        script: "curl -s -o /dev/null -w '%{http_code}' ${url}",
                        returnStdout: true
                    ).trim()
                    
                    if (response != '200') {
                        error "Health check failed: HTTP ${response}"
                    }
                    echo "✅ Smoke tests passed!"
                }
            }
        }
        
        stage('📊 Performance Tests') {
            when {
                expression { params.ENVIRONMENT != 'production' }
            }
            steps {
                sh '''
                    # JMeter or Gatling tests
                    ./run-performance-tests.sh
                '''
            }
        }
    }
    
    post {
        success {
            script {
                def duration = (new Date().time - deploymentInfo.startTime.time) / 1000
                
                slackSend(
                    color: 'good',
                    channel: '#deployments',
                    message: """
                    ✅ *Deployment Successful*
                    Environment: ${params.ENVIRONMENT}
                    Build: #${env.BUILD_NUMBER}
                    Branch: ${env.BRANCH_NAME}
                    Commit: ${env.GIT_COMMIT.take(8)}
                    Author: ${env.GIT_AUTHOR_NAME}
                    Duration: ${duration}s
                    Image: ${IMAGE_NAME}:${IMAGE_TAG}
                    """
                )
                
                // Update deployment tracking
                sh """
                    curl -X POST https://api.example.com/deployments \
                      -H 'Content-Type: application/json' \
                      -d '{
                        "environment": "${params.ENVIRONMENT}",
                        "version": "${IMAGE_TAG}",
                        "status": "success",
                        "build": "${env.BUILD_NUMBER}"
                      }'
                """
            }
        }
        
        failure {
            script {
                slackSend(
                    color: 'danger',
                    channel: '#alerts',
                    message: """
                    ❌ *Deployment Failed*
                    Environment: ${params.ENVIRONMENT}
                    Build: #${env.BUILD_NUMBER}
                    Branch: ${env.BRANCH_NAME}
                    Stage: ${env.STAGE_NAME}
                    """
                )
                
                // Rollback on production failure
                if (params.ENVIRONMENT == 'production') {
                    withKubeConfig([credentialsId: 'kubeconfig']) {
                        sh "kubectl rollout undo deployment/myapp -n ${K8S_NAMESPACE}"
                    }
                }
            }
        }
        
        always {
            // Cleanup
            cleanWs()
            sh 'docker system prune -f'
            
            // Archive logs
            archiveArtifacts artifacts: '**/*.log',
                           allowEmptyArchive: true
        }
    }
}
```

**This pipeline demonstrates:**
- ✅ Complete DevOps tool integration
- ✅ Multi-stage quality gates
- ✅ Security scanning at multiple levels
- ✅ Parallel execution
- ✅ Comprehensive error handling
- ✅ Automated rollback
- ✅ Notifications and tracking
- ✅ Production-ready practices

---

## Scenario-Based Questions

### Q11: How would you handle a production deployment failure?
**Answer:**

**Incident Response Strategy:**

**1. Immediate Actions (0-5 minutes):**
```groovy
stage('Rollback Decision') {
    steps {
        script {
            try {
                // Verify deployment health
                def health = sh(
                    script: 'curl -f http://myapp/health',
                    returnStatus: true
                )
                
                if (health != 0) {
                    echo "❌ Health check failed, initiating rollback"
                    
                    // Automatic rollback
                    withKubeConfig([credentialsId: 'kubeconfig']) {
                        sh """
                            kubectl rollout undo deployment/myapp -n production
                            kubectl rollout status deployment/myapp -n production
                        """
                    }
                    
                    // Alert team
                    slackSend(
                        color: 'danger',
                        channel: '#incidents',
                        message: """
                        🚨 *PRODUCTION ROLLBACK INITIATED*
                        Build: ${env.BUILD_NUMBER}
                        Reason: Health check failure
                        Status: Rolling back to previous version
                        """
                    )
                }
            } catch (Exception e) {
                error("Deployment verification failed: ${e.message}")
            }
        }
    }
}
```

**2. Investigation (5-30 minutes):**
```bash
# Check pod status
kubectl get pods -n production
kubectl describe pod <pod-name> -n production

# Check logs
kubectl logs deployment/myapp -n production --tail=100

# Check events
kubectl get events -n production --sort-by='.lastTimestamp'

# Check resource usage
kubectl top pods -n production
```

**3. Root Cause Analysis:**
- Review Jenkins console output
- Check SonarQube for new issues
- Review Docker image changes
- Check infrastructure changes
- Review application logs

**4. Prevention:**
```groovy
// Implement canary deployment
stage('Canary Deploy') {
    steps {
        withKubeConfig([credentialsId: 'kubeconfig']) {
            sh """
                # Deploy canary version (10% traffic)
                kubectl apply -f k8s/canary-deployment.yaml
                
                # Monitor for 10 minutes
                sleep 600
                
                # Check error rate
                ERROR_RATE=\$(curl -s prometheus/api/v1/query?query='error_rate')
                
                if [ "\$ERROR_RATE" -gt "5" ]; then
                    echo "Canary failed, rolling back"
                    kubectl delete -f k8s/canary-deployment.yaml
                    exit 1
                fi
                
                # Promote canary to full deployment
                kubectl apply -f k8s/deployment.yaml
            """
        }
    }
}
```

---

**🎉 COMPLETE! You now have MASTERY of Jenkins in production environments with full DevOps integration! 🚀**

For more resources:
- [Learning Roadmap](jenkins-learning-roadmap.md)
- [Quick Reference](jenkins-quick-reference.md)
- [Hands-On Exercises](jenkins-hands-on-exercises.md)
- [Troubleshooting Guide](jenkins-troubleshooting-guide.md)

**Good luck with your interviews! 💼✨**


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

