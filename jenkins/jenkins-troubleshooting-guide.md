# Jenkins Troubleshooting Guide

Comprehensive solutions to 50+ common Jenkins issues, from basic setup to complex production scenarios.

---

## Table of Contents
1. [Installation & Setup Issues](#installation--setup-issues)
2. [Authentication & Authorization](#authentication--authorization)
3. [Git Integration Issues](#git-integration-issues)
4. [Maven Build Problems](#maven-build-problems)
5. [SonarQube Integration](#sonarqube-integration)
6. [Docker Issues](#docker-issues)
7. [Kubernetes Deployment](#kubernetes-deployment)
8. [Pipeline Syntax Errors](#pipeline-syntax-errors)
9. [Performance Issues](#performance-issues)
10. [Plugin Problems](#plugin-problems)
11. [Workspace & Disk Space](#workspace--disk-space)
12. [Network & Connectivity](#network--connectivity)
13. [Security Issues](#security-issues)
14. [Distributed Builds](#distributed-builds)
15. [Production Incidents](#production-incidents)

---

## Installation & Setup Issues

### Issue 1: Cannot Access Jenkins After Installation

**Symptoms:**
- Browser shows "Connection refused"
- Port 8080 not responding
- Timeout errors

**Diagnosis:**
```bash
# Check if Jenkins is running
systemctl status jenkins  # Linux
Get-Service jenkins       # Windows

# Check port binding
netstat -tuln | grep 8080  # Linux
netstat -an | findstr 8080 # Windows

# Check Jenkins logs
tail -f /var/log/jenkins/jenkins.log  # Linux
docker logs jenkins                    # Docker
```

**Solutions:**

**1. Service Not Running:**
```bash
# Linux
systemctl start jenkins
systemctl enable jenkins

# Docker
docker start jenkins
docker ps -a  # Check status
```

**2. Port Conflict:**
```bash
# Change Jenkins port
# Edit /etc/default/jenkins
HTTP_PORT=8081

# Or Docker
docker run -p 8081:8080 jenkins/jenkins:lts
```

**3. Firewall Blocking:**
```bash
# Linux
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --reload

# Windows
netsh advfirewall firewall add rule name="Jenkins" dir=in action=allow protocol=TCP localport=8080
```

**Prevention:**
- Verify port availability before installation
- Check firewall rules
- Monitor service health

---

### Issue 2: Lost Administrator Password

**Symptoms:**
- Cannot login to Jenkins
- Forgot admin password
- Initial admin password not working

**Solutions:**

**Method 1: Reset via Config File**
```bash
# 1. Stop Jenkins
systemctl stop jenkins

# 2. Edit config to disable security
vim /var/lib/jenkins/config.xml

# Change:
<useSecurity>false</useSecurity>

# 3. Restart Jenkins
systemctl start jenkins

# 4. Access Jenkins, go to Manage Jenkins → Configure Global Security
# 5. Re-enable security and set new password
```

**Method 2: Password Reset via Script Console**
```groovy
// Access http://jenkins:8080/script
import jenkins.model.*
import hudson.security.*

def instance = Jenkins.getInstance()
def hudsonRealm = new HudsonPrivateSecurityRealm(false)
hudsonRealm.createAccount("admin", "newpassword")
instance.setSecurityRealm(hudsonRealm)
instance.save()
```

**Method 3: Docker Container**
```bash
# Get container ID
docker ps

# Access container
docker exec -it <container-id> bash

# Remove security config
rm /var/jenkins_home/config.xml

# Restart container
docker restart <container-id>
```

---

### Issue 3: Jenkins Runs Out of Memory

**Symptoms:**
```
java.lang.OutOfMemoryError: Java heap space
Jenkins is slow or unresponsive
Builds fail randomly
```

**Diagnosis:**
```bash
# Check current memory allocation
ps aux | grep jenkins

# Check memory usage
free -h
top -p $(pgrep -f jenkins)
```

**Solutions:**

**1. Increase Heap Size:**
```bash
# Linux - Edit /etc/default/jenkins
JAVA_ARGS="-Xmx4096m -Xms2048m"

# Docker
docker run -e JAVA_OPTS="-Xmx4096m" jenkins/jenkins:lts

# Restart Jenkins
systemctl restart jenkins
```

**2. Analyze Heap Dump:**
```bash
# Enable heap dump on OOM
JAVA_ARGS="-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/var/log/jenkins/"

# Analyze with Eclipse MAT or VisualVM
```

**3. Reduce Memory Usage:**
- Limit build history: Configure → Discard old builds
- Clean up unused plugins
- Use external storage for artifacts
- Implement build cleanup:

```groovy
// In Jenkinsfile
options {
    buildDiscarder(logRotator(numToKeepStr: '10'))
}
```

---

## Authentication & Authorization

### Issue 4: Users Cannot Access Jenkins

**Symptoms:**
- "Access Denied" errors
- Users locked out after security changes
- Role-based access not working

**Solutions:**

**1. Check Security Realm:**
```
Manage Jenkins → Configure Global Security → Security Realm
```

**2. Fix LDAP Connection:**
```groovy
// Test LDAP connection in Script Console
import javax.naming.*
import javax.naming.directory.*

def env = new Hashtable()
env.put(Context.INITIAL_CONTEXT_FACTORY, "com.sun.jndi.ldap.LdapCtxFactory")
env.put(Context.PROVIDER_URL, "ldap://ldap.example.com:389")
env.put(Context.SECURITY_AUTHENTICATION, "simple")
env.put(Context.SECURITY_PRINCIPAL, "cn=admin,dc=example,dc=com")
env.put(Context.SECURITY_CREDENTIALS, "password")

try {
    DirContext ctx = new InitialDirContext(env)
    println "LDAP connection successful"
    ctx.close()
} catch (Exception e) {
    println "LDAP connection failed: ${e.message}"
}
```

**3. Grant Matrix Permissions:**
```
Manage Jenkins → Configure Global Security → Authorization
Strategy: Matrix-based security

Add user/group and grant permissions:
- Overall: Read
- Job: Build, Read, Workspace
```

---

## Git Integration Issues

### Issue 5: Git Authentication Failed

**Symptoms:**
```
ERROR: Error cloning remote repo 'origin'
hudson.plugins.git.GitException: Command "git fetch" returned status code 128
Authentication failed
```

**Solutions:**

**1. HTTP/HTTPS Authentication:**
```bash
# Add credentials in Jenkins
Manage Jenkins → Manage Credentials → Add Credentials
Kind: Username with password
Username: git-username
Password: personal-access-token (not password!)
ID: git-credentials
```

**Update Jenkinsfile:**
```groovy
checkout([
    $class: 'GitSCM',
    branches: [[name: '*/main']],
    userRemoteConfigs: [[
        url: 'https://github.com/user/repo.git',
        credentialsId: 'git-credentials'
    ]]
])
```

**2. SSH Key Authentication:**
```bash
# Generate SSH key on Jenkins server
ssh-keygen -t rsa -b 4096 -f ~/.ssh/jenkins_rsa

# Add public key to GitHub/GitLab
cat ~/.ssh/jenkins_rsa.pub

# Add private key to Jenkins
Manage Credentials → Add → SSH Username with private key
ID: jenkins-ssh
Username: git
Private Key: (paste contents of jenkins_rsa)
```

**3. Host Key Verification:**
```bash
# Add known host
ssh-keyscan github.com >> ~/.ssh/known_hosts

# Or disable strict checking (not recommended for production)
git config --global core.sshCommand "ssh -o StrictHostKeyChecking=no"
```

---

### Issue 6: Git Submodules Not Checking Out

**Symptoms:**
- Submodules appear empty
- Build fails due to missing files

**Solution:**
```groovy
checkout([
    $class: 'GitSCM',
    branches: [[name: '*/main']],
    extensions: [
        [$class: 'SubmoduleOption',
         recursiveSubmodules: true,
         trackingSubmodules: false]
    ],
    userRemoteConfigs: [[
        url: 'https://github.com/user/repo.git',
        credentialsId: 'git-credentials'
    ]]
])
```

---

### Issue 7: Large Repository Clone Timeout

**Symptoms:**
```
fatal: The remote end hung up unexpectedly
fatal: early EOF
```

**Solutions:**

**1. Shallow Clone:**
```groovy
checkout([
    $class: 'GitSCM',
    branches: [[name: '*/main']],
    extensions: [
        [$class: 'CloneOption', 
         depth: 1, 
         shallow: true,
         timeout: 30]
    ],
    userRemoteConfigs: [[url: 'https://github.com/user/repo.git']]
])
```

**2. Increase Git Buffer:**
```bash
git config --global http.postBuffer 524288000  # 500MB
git config --global http.lowSpeedLimit 0
git config --global http.lowSpeedTime 999999
```

---

## Maven Build Problems

### Issue 8: Maven Build Fails - Dependencies Not Found

**Symptoms:**
```
[ERROR] Failed to execute goal on project myapp: Could not resolve dependencies
Could not find artifact com.example:library:jar:1.0.0
```

**Diagnosis:**
```bash
# Check Maven settings
cat ~/.m2/settings.xml

# Verify repository connectivity
curl -I https://repo.maven.apache.org/maven2/

# Check local cache
ls -la ~/.m2/repository/
```

**Solutions:**

**1. Clear Local Repository:**
```groovy
stage('Clean Maven Cache') {
    steps {
        sh 'rm -rf ~/.m2/repository/com/example'
        sh 'mvn dependency:purge-local-repository'
    }
}
```

**2. Configure Custom Repository:**
```xml
<!-- ~/.m2/settings.xml -->
<settings>
    <mirrors>
        <mirror>
            <id>nexus</id>
            <mirrorOf>*</mirrorOf>
            <url>http://nexus.example.com/repository/maven-public/</url>
        </mirror>
    </mirrors>
</settings>
```

**3. Use Jenkins Managed Settings:**
```groovy
stage('Build') {
    steps {
        configFileProvider([
            configFile(fileId: 'maven-settings', variable: 'MAVEN_SETTINGS')
        ]) {
            sh 'mvn -s $MAVEN_SETTINGS clean package'
        }
    }
}
```

---

### Issue 9: Maven Out of Memory During Build

**Symptoms:**
```
java.lang.OutOfMemoryError: GC overhead limit exceeded
The forked VM terminated without saying properly goodbye
```

**Solutions:**

**1. Increase Maven Memory:**
```groovy
stage('Build') {
    steps {
        sh 'MAVEN_OPTS="-Xmx2048m -XX:MaxPermSize=512m" mvn clean package'
    }
}
```

**2. Configure in pom.xml:**
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <configuration>
                <argLine>-Xmx1024m -XX:MaxPermSize=256m</argLine>
                <forkCount>1</forkCount>
                <reuseForks>false</reuseForks>
            </configuration>
        </plugin>
    </plugins>
</build>
```

---

### Issue 10: Tests Fail in Jenkins but Pass Locally

**Symptoms:**
- Tests pass on developer machine
- Same tests fail in Jenkins

**Common Causes & Solutions:**

**1. Environment Differences:**
```groovy
stage('Debug Environment') {
    steps {
        sh '''
            echo "Java Version:"
            java -version
            echo "Maven Version:"
            mvn -version
            echo "Environment Variables:"
            printenv | sort
        '''
    }
}
```

**2. Timezone Issues:**
```groovy
environment {
    TZ = 'America/New_York'
}
```

**3. Display Issues (GUI Tests):**
```groovy
stage('GUI Tests') {
    steps {
        sh '''
            export DISPLAY=:99
            Xvfb :99 -screen 0 1024x768x24 &
            mvn test
        '''
    }
}
```

**4. File Path Issues:**
```groovy
// Use relative paths, not absolute
def configFile = "${env.WORKSPACE}/config/app.properties"
```

---

## SonarQube Integration

### Issue 11: SonarQube Analysis Fails

**Symptoms:**
```
ERROR: Error during SonarQube Scanner execution
Not authorized. Please check the properties sonar.login and sonar.password
```

**Solutions:**

**1. Verify Authentication:**
```bash
# Test SonarQube connection
curl -u admin:admin http://sonarqube:9000/api/system/status
```

**2. Configure Token:**
```groovy
withSonarQubeEnv('SonarQube') {
    sh '''
        mvn sonar:sonar \
          -Dsonar.projectKey=myapp \
          -Dsonar.host.url=$SONAR_HOST_URL \
          -Dsonar.login=$SONAR_AUTH_TOKEN
    '''
}
```

**3. Check SonarQube Server Configuration:**
```
Manage Jenkins → Configure System → SonarQube servers
Name: SonarQube
Server URL: http://sonarqube:9000
Server authentication token: (must be Secret text credential)
```

---

### Issue 12: Quality Gate Timeout

**Symptoms:**
```
Timeout of 1 hour exceeded while waiting for Quality Gate status
```

**Solutions:**

**1. Check Webhook:**
```
SonarQube → Administration → Configuration → Webhooks
URL: http://jenkins:8080/sonarqube-webhook/
Secret: (optional)
```

**Test webhook:**
```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"status":"OK"}' \
  http://jenkins:8080/sonarqube-webhook/
```

**2. Increase Timeout:**
```groovy
stage('Quality Gate') {
    steps {
        timeout(time: 2, unit: 'HOURS') {
            waitForQualityGate abortPipeline: true
        }
    }
}
```

**3. Check SonarQube Compute Engine:**
```bash
# SonarQube logs
tail -f /opt/sonarqube/logs/ce.log

# Check pending tasks
curl http://sonarqube:9000/api/ce/activity?status=PENDING
```

---

## Docker Issues

### Issue 13: Permission Denied While Connecting to Docker Daemon

**Symptoms:**
```
permission denied while trying to connect to the Docker daemon socket
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock
```

**Solutions:**

**1. Add Jenkins User to Docker Group:**
```bash
# Add user
sudo usermod -aG docker jenkins

# Restart Jenkins
sudo systemctl restart jenkins

# Verify
sudo -u jenkins docker ps
```

**2. Docker Socket Permissions:**
```bash
# Temporary fix
sudo chmod 666 /var/run/docker.sock

# Permanent fix
sudo chmod 660 /var/run/docker.sock
sudo chown root:docker /var/run/docker.sock
```

**3. Use Docker-in-Docker (DinD):**
```groovy
pipeline {
    agent {
        docker {
            image 'docker:dind'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
}
```

---

### Issue 14: Docker Build Fails with "No Space Left on Device"

**Symptoms:**
```
no space left on device
docker: write /var/lib/docker/tmp/...: no space left on device
```

**Diagnosis:**
```bash
# Check disk space
df -h

# Check Docker disk usage
docker system df

# Check Docker info
docker info | grep "Data Space"
```

**Solutions:**

**1. Clean Up Docker:**
```bash
# Remove unused images
docker image prune -a --force

# Remove unused containers
docker container prune --force

# Remove unused volumes
docker volume prune --force

# Complete cleanup
docker system prune -a --volumes --force
```

**2. Automate Cleanup in Pipeline:**
```groovy
post {
    always {
        sh '''
            docker system prune -f
            docker volume prune -f
        '''
    }
}
```

**3. Configure Docker Storage Driver:**
```bash
# Edit /etc/docker/daemon.json
{
    "storage-driver": "overlay2",
    "storage-opts": [
        "overlay2.override_kernel_check=true",
        "overlay2.size=20G"
    ]
}

# Restart Docker
systemctl restart docker
```

---

### Issue 15: Docker Image Push Fails

**Symptoms:**
```
denied: requested access to the resource is denied
unauthorized: authentication required
error parsing HTTP 401 response body
```

**Solutions:**

**1. Docker Login:**
```groovy
stage('Push Image') {
    steps {
        script {
            docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-creds') {
                dockerImage.push("${env.BUILD_NUMBER}")
            }
        }
    }
}
```

**2. Manual Login Test:**
```bash
# Test credentials
echo "password" | docker login -u username --password-stdin

# Test push
docker push username/image:tag
```

**3. Check Registry Configuration:**
```groovy
// For private registry
docker.withRegistry('https://registry.example.com:5000', 'registry-creds') {
    // push operations
}
```

---

## Kubernetes Deployment

### Issue 16: kubectl Command Not Found

**Symptoms:**
```
sh: kubectl: command not found
```

**Solutions:**

**1. Install kubectl in Jenkins:**
```bash
# On Jenkins server
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

**2. Use kubectl Docker Image:**
```groovy
stage('Deploy') {
    agent {
        docker {
            image 'bitnami/kubectl:latest'
            args '-v $HOME/.kube:/root/.kube'
        }
    }
    steps {
        sh 'kubectl get pods'
    }
}
```

**3. Install as Jenkins Tool:**
```
Manage Jenkins → Global Tool Configuration → kubectl
Name: kubectl
Install automatically: yes
Version: latest
```

---

### Issue 17: Kubernetes Authentication Failed

**Symptoms:**
```
error: You must be logged in to the server (Unauthorized)
Unable to connect to the server: x509: certificate signed by unknown authority
```

**Solutions:**

**1. Configure kubeconfig:**
```groovy
withKubeConfig([
    credentialsId: 'kubeconfig',
    serverUrl: 'https://k8s.example.com:6443'
]) {
    sh 'kubectl get pods'
}
```

**2. Add Kubeconfig Credential:**
```
Manage Jenkins → Manage Credentials → Add
Kind: Secret file
File: (upload kubeconfig file)
ID: kubeconfig
```

**3. Use Service Account Token:**
```groovy
withKubeConfig([
    credentialsId: 'k8s-token',
    serverUrl: 'https://k8s.example.com:6443',
    clusterName: 'production'
]) {
    sh 'kubectl apply -f deployment.yaml'
}
```

**4. Test Connection:**
```bash
# Verify kubeconfig
kubectl cluster-info

# Test from Jenkins
cat ~/.kube/config
kubectl auth can-i get pods --all-namespaces
```

---

### Issue 18: Deployment Rollout Failed

**Symptoms:**
```
error: deployment "myapp" exceeded its progress deadline
Error from server: error when applying patch
ImagePullBackOff
```

**Diagnosis:**
```bash
# Check deployment status
kubectl get deployment myapp -n production
kubectl describe deployment myapp -n production

# Check pods
kubectl get pods -n production
kubectl describe pod myapp-xxx -n production

# Check events
kubectl get events -n production --sort-by='.lastTimestamp'

# Check logs
kubectl logs deployment/myapp -n production
```

**Solutions:**

**1. Fix ImagePullBackOff:**
```groovy
stage('Verify Image') {
    steps {
        script {
            // Verify image exists
            sh "docker pull ${IMAGE}:${TAG}"
        }
    }
}

stage('Deploy') {
    steps {
        withKubeConfig([credentialsId: 'kubeconfig']) {
            sh '''
                # Create image pull secret if private registry
                kubectl create secret docker-registry regcred \
                  --docker-server=registry.example.com \
                  --docker-username=user \
                  --docker-password=pass \
                  -n production \
                  --dry-run=client -o yaml | kubectl apply -f -
                
                # Deploy
                kubectl set image deployment/myapp \
                  myapp=${IMAGE}:${TAG} \
                  -n production
                
                # Wait with timeout
                kubectl rollout status deployment/myapp \
                  -n production \
                  --timeout=5m
            '''
        }
    }
}
```

**2. Implement Automatic Rollback:**
```groovy
stage('Deploy') {
    steps {
        script {
            try {
                sh 'kubectl rollout status deployment/myapp -n production --timeout=5m'
            } catch (Exception e) {
                echo "Deployment failed, rolling back..."
                sh 'kubectl rollout undo deployment/myapp -n production'
                error("Deployment failed and rolled back")
            }
        }
    }
}
```

---

## Pipeline Syntax Errors

### Issue 19: Script Error - Unexpected Token

**Symptoms:**
```
WorkflowScript: unexpected token @ line 5, column 10
```

**Common Mistakes & Fixes:**

**1. Missing Quotes:**
```groovy
// ❌ Wrong
sh echo Hello World

// ✅ Correct
sh 'echo Hello World'
```

**2. Wrong Bracket Type:**
```groovy
// ❌ Wrong
stage('Build') (
    steps {
        sh 'mvn clean'
    }
)

// ✅ Correct
stage('Build') {
    steps {
        sh 'mvn clean'
    }
}
```

**3. Missing script Block:**
```groovy
// ❌ Wrong
stage('Build') {
    steps {
        def version = '1.0'
        sh "echo ${version}"
    }
}

// ✅ Correct
stage('Build') {
    steps {
        script {
            def version = '1.0'
            sh "echo ${version}"
        }
    }
}
```

---

### Issue 20: Variable Scope Issues

**Symptoms:**
```
groovy.lang.MissingPropertyException: No such property: variable
```

**Solutions:**

```groovy
// ❌ Wrong - variable defined in script block not accessible outside
stage('Define') {
    steps {
        script {
            def version = '1.0'
        }
    }
}
stage('Use') {
    steps {
        echo version  // ERROR!
    }
}

// ✅ Correct - use environment or explicit declaration
pipeline {
    environment {
        VERSION = '1.0'
    }
    stages {
        stage('Use') {
            steps {
                echo env.VERSION  // Works!
            }
        }
    }
}

// ✅ Correct - declare at pipeline level
def version

pipeline {
    agent any
    stages {
        stage('Define') {
            steps {
                script {
                    version = '1.0'
                }
            }
        }
        stage('Use') {
            steps {
                echo version  // Works!
            }
        }
    }
}
```

---

## Performance Issues

### Issue 21: Slow Build Times

**Diagnosis:**
```groovy
stage('Build') {
    steps {
        script {
            def startTime = System.currentTimeMillis()
            sh 'mvn clean package'
            def duration = (System.currentTimeMillis() - startTime) / 1000
            echo "Build took ${duration} seconds"
        }
    }
}
```

**Solutions:**

**1. Implement Caching:**
```groovy
stage('Build') {
    steps {
        sh '''
            # Maven cache
            mvn -Dmaven.repo.local=$WORKSPACE/.m2 clean package
        '''
    }
}
```

**2. Parallel Execution:**
```groovy
stage('Test') {
    parallel {
        stage('Unit') {
            steps { sh 'mvn test' }
        }
        stage('Integration') {
            steps { sh 'mvn verify -DskipUnitTests' }
        }
    }
}
```

**3. Incremental Builds:**
```groovy
stage('Build') {
    when {
        changeset "src/**"
    }
    steps {
        sh 'mvn compile'
    }
}
```

---

### Issue 22: Queue Builds Up

**Symptoms:**
- Many jobs waiting in queue
- Builds delayed
- "Waiting for next available executor"

**Solutions:**

**1. Increase Executors:**
```
Manage Jenkins → Configure System → # of executors: 4
```

**2. Add Build Agents:**
```
Manage Jenkins → Manage Nodes → New Node
```

**3. Limit Concurrent Builds:**
```groovy
options {
    disableConcurrentBuilds()
}
```

**4. Throttle Builds:**
```groovy
options {
    throttleJobProperty(
        categories: ['heavy-builds'],
        throttleEnabled: true,
        throttleOption: 'category',
        maxConcurrentTotal: 2
    )
}
```

---

## Plugin Problems

### Issue 23: Plugin Installation Fails

**Symptoms:**
```
Failed to install plugin
Connection timeout
Plugin dependency resolution failed
```

**Solutions:**

**1. Update Plugin Center:**
```
Manage Jenkins → Manage Plugins → Advanced
Update Site URL: https://updates.jenkins.io/update-center.json
Submit → Check now
```

**2. Manual Installation:**
```bash
# Download .hpi file
wget https://updates.jenkins.io/download/plugins/git/4.11.0/git.hpi

# Copy to plugins directory
cp git.hpi /var/lib/jenkins/plugins/

# Restart Jenkins
systemctl restart jenkins
```

**3. Fix Dependencies:**
```bash
# Install missing dependencies first
# Check plugin page on jenkins.io for requirements
```

---

### Issue 24: Plugin Conflicts

**Symptoms:**
- Jenkins fails to start after plugin update
- Features stop working
- ClassNotFoundException

**Solutions:**

**1. Safe Mode:**
```bash
# Start Jenkins in safe mode (no plugins)
java -jar jenkins.war --httpPort=8080 --prefix=/jenkins --enable-future-java

# Or edit config
vim /etc/default/jenkins
JAVA_ARGS="-Dhudson.Main.development=true"
```

**2. Rollback Plugin:**
```
Manage Jenkins → Manage Plugins → Installed
Find plugin → Downgrade to earlier version
```

**3. Check Logs:**
```bash
tail -f /var/log/jenkins/jenkins.log
# Look for ClassNotFoundException or NoSuchMethodError
```

---

## Workspace & Disk Space

### Issue 25: Workspace Cleanup Failures

**Symptoms:**
```
ERROR: Unable to delete workspace
Directory not empty
```

**Solutions:**

**1. Force Cleanup:**
```groovy
post {
    always {
        cleanWs deleteDirs: true,
                disableDeferredWipeout: true,
                notFailBuild: true
    }
}
```

**2. Manual Cleanup:**
```bash
# On Jenkins server
cd /var/lib/jenkins/workspace
find . -name "myapp*" -type d -exec rm -rf {} +
```

**3. Scheduled Cleanup:**
```groovy
// Use Job DSL or pipeline
properties([
    buildDiscarder(logRotator(numToKeepStr: '10')),
    disableConcurrentBuilds()
])
```

---

## Production Incidents

### Issue 26: Jenkins Crash Recovery

**Emergency Response:**

**1. Assess Situation:**
```bash
# Check if Jenkins is running
systemctl status jenkins
docker ps -a

# Check last logs
tail -100 /var/log/jenkins/jenkins.log

# Check system resources
free -h
df -h
top
```

**2. Quick Recovery:**
```bash
# Try restart
systemctl restart jenkins

# If fails, check port
netstat -tuln | grep 8080
# Kill conflicting process if needed
lsof -i :8080
kill -9 <PID>

# Start Jenkins
systemctl start jenkins
```

**3. Safe Mode Recovery:**
```bash
# Move plugins temporarily
mv /var/lib/jenkins/plugins /var/lib/jenkins/plugins.bak
mkdir /var/lib/jenkins/plugins

# Start Jenkins
systemctl start jenkins

# Gradually restore plugins
```

---

**Complete DevOps Integration Troubleshooting Expertise Achieved! 🔧✅**

For more resources:
- [Learning Roadmap](jenkins-learning-roadmap.md)
- [Hands-On Exercises](jenkins-hands-on-exercises.md)
- [Interview Questions](jenkins-interview-questions.md)


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

