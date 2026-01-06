# SonarQube Learning Roadmap
## 12-Week Journey from Beginner to Production Expert

Complete curriculum for mastering SonarQube code quality and security analysis.

---

## 🎯 Learning Path Overview

```
Weeks 1-2:  Fundamentals & Installation
Weeks 3-4:  Code Analysis & Rules
Weeks 5-6:  Quality Gates & Profiles
Weeks 7-8:  CI/CD Integration
Weeks 9-10: Security & Advanced Topics
Weeks 11-12: Production & Enterprise
```

**Time Commitment**: 10-15 hours per week  
**Prerequisites**: Git, Maven, Java basics  
**Target**: Production-ready SonarQube expertise

---

## Week 1: SonarQube Fundamentals

### Learning Objectives
- Understand SonarQube purpose and architecture
- Install SonarQube Community Edition
- Perform first code analysis
- Navigate SonarQube UI

### Topics

#### 1.1 Introduction to SonarQube
**What is SonarQube?**
- Continuous code quality inspection platform
- Static code analysis tool
- Security vulnerability detection
- Technical debt management

**Why SonarQube?**
- Early bug detection
- Security vulnerability identification
- Code maintainability improvement
- Standardized quality metrics
- Integration with DevOps pipeline

**SonarQube Architecture:**
```
┌─────────────┐
│   Scanner   │ (Analyzes code locally)
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   Server    │ (Web UI, Compute Engine, Database)
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Database   │ (PostgreSQL, Oracle, SQL Server)
└─────────────┘
```

**Key Components:**
- **SonarQube Server**: Web interface, compute engine
- **SonarQube Scanner**: Code analysis tool
- **Database**: Stores analysis results
- **Elasticsearch**: Search and indexing

#### 1.2 Installation (Development Environment)
```bash
# Prerequisites
java -version  # Java 11 or 17 required

# Download SonarQube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.3.79811.zip
unzip sonarqube-9.9.3.79811.zip
cd sonarqube-9.9.3.79811

# Start server
bin/linux-x86-64/sonar.sh start

# View logs
tail -f logs/sonar.log

# Access web interface
# http://localhost:9000
# Default: admin/admin
```

#### 1.3 First Project Analysis
```bash
# Option 1: Using Maven
mvn clean verify sonar:sonar \
  -Dsonar.projectKey=my-first-project \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=your-token

# Option 2: Using SonarScanner CLI
sonar-scanner \
  -Dsonar.projectKey=my-project \
  -Dsonar.sources=. \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=your-token
```

### Hands-On Practice
1. Install SonarQube locally
2. Create account and generate token
3. Analyze a sample Java project
4. Explore the dashboard
5. Review issues found

### Resources
- Official Docs: https://docs.sonarqube.org/latest/
- Try SonarQube: https://www.sonarqube.org/

---

## Week 2: Understanding Analysis & Metrics

### Learning Objectives
- Understand SonarQube metrics
- Learn about issue types and severity
- Configure scanner properties
- Understand analysis scope

### Topics

#### 2.1 Core Metrics

**The 7 Axes of Quality:**

**1. Bugs**
- Issues that could cause runtime errors
- Rating: A (0 bugs) to E (many bugs)
- Example: NullPointerException, ArrayIndexOutOfBounds

**2. Vulnerabilities**
- Security issues
- Rating: A (0 vulnerabilities) to E (many)
- Example: SQL injection, XSS, hardcoded credentials

**3. Code Smells**
- Maintainability issues
- Not bugs, but make code hard to maintain
- Example: Too complex methods, duplicated code

**4. Coverage**
- Test coverage percentage
- Line coverage and branch coverage
- Target: > 80%

**5. Duplications**
- Duplicated code blocks
- Measured in lines and percentage
- Target: < 3%

**6. Security Hotspots**
- Security-sensitive code requiring review
- Not vulnerabilities, but need attention
- Example: Cryptography usage, authentication

**7. Technical Debt**
- Time to fix all code smells
- Measured in days/hours
- Debt ratio: Technical Debt / Development Cost

#### 2.2 Issue Severity & Type

**Severity Levels:**
- **Blocker**: Must fix immediately
- **Critical**: Should fix ASAP
- **Major**: Should fix
- **Minor**: Nice to fix
- **Info**: Informational

**Issue Types:**
- **Bug**: Something wrong that could cause failure
- **Vulnerability**: Security issue
- **Code Smell**: Maintainability issue
- **Security Hotspot**: Security-sensitive code

#### 2.3 Scanner Configuration

**sonar-project.properties:**
```properties
# Required settings
sonar.projectKey=my-project
sonar.projectName=My Project
sonar.projectVersion=1.0

# Source code location
sonar.sources=src/main/java
sonar.tests=src/test/java

# Exclusions
sonar.exclusions=**/generated/**,**/vendor/**

# Coverage
sonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml

# Encoding
sonar.sourceEncoding=UTF-8
```

**Maven Configuration (pom.xml):**
```xml
<properties>
    <sonar.organization>my-org</sonar.organization>
    <sonar.host.url>https://sonarcloud.io</sonar.host.url>
    <sonar.exclusions>**/generated/**</sonar.exclusions>
</properties>
```

### Hands-On Practice
1. Analyze project with intentional bugs
2. Review each metric in dashboard
3. Understand issue classifications
4. Configure scanner properties
5. Exclude test files from analysis

---

## Week 3: Rules & Quality Profiles

### Learning Objectives
- Understand SonarQube rules
- Configure quality profiles
- Activate/deactivate rules
- Create custom profiles

### Topics

#### 3.1 Understanding Rules

**What are Rules?**
- Checks performed during analysis
- Language-specific
- Categories: Bugs, Vulnerabilities, Code Smells, Security Hotspots

**Rule Repositories:**
```
Java: 600+ rules
JavaScript: 400+ rules
Python: 500+ rules
C#: 500+ rules
```

**Rule Properties:**
- **Type**: Bug, Vulnerability, Code Smell
- **Severity**: Blocker, Critical, Major, Minor, Info
- **Remediation Time**: Estimated fix time
- **Tags**: security, cert, cwe, owasp, etc.

#### 3.2 Quality Profiles

**Built-in Profiles:**
- **Sonar way**: Recommended default
- **Sonar way Recommended**: Subset of Sonar way
- Language-specific profiles

**Profile Management:**
```
Administration → Quality Profiles
- Create custom profile
- Extend existing profile
- Activate/deactivate rules
- Set as default
- Export/import profiles
```

**Creating Custom Profile:**
1. Quality Profiles → Create
2. Name: "Company Java Profile"
3. Extend: "Sonar way"
4. Activate additional rules
5. Adjust severity levels
6. Set as default

#### 3.3 Rule Customization

**Activating Rules:**
```
Quality Profiles → Your Profile → Activate More
- Search by tag (security, performance)
- Filter by type (Bug, Vulnerability)
- Activate rules
- Set severity
```

**Rule Parameters:**
```
Example: "Methods should not be too complex"
- Default threshold: 10
- Custom threshold: 15
```

### Hands-On Practice
1. Create custom quality profile
2. Activate security rules
3. Adjust complexity thresholds
4. Analyze project with custom profile
5. Export and import profile

---

## Week 4: Quality Gates

### Learning Objectives
- Understand quality gates
- Configure conditions
- Implement gate enforcement
- Create custom gates

### Topics

#### 4.1 Quality Gates Fundamentals

**What is a Quality Gate?**
- Set of conditions that must be met
- Pass/Fail decision for builds
- Enforces quality standards
- Focuses on New Code

**Default Quality Gate (Sonar way):**
```yaml
Conditions on New Code:
- Coverage >= 80%
- Duplicated Lines < 3%
- Security Rating = A
- Reliability Rating = A
- Maintainability Rating = A
```

#### 4.2 Creating Custom Quality Gates

**Navigate:** Quality Gates → Create

**Example: Strict Gate**
```yaml
On New Code:
- Coverage >= 90%
- Duplicated Lines < 2%
- Bugs = 0
- Vulnerabilities = 0
- Security Hotspots Reviewed = 100%
- Code Smells <= 10
- Security Rating = A
- Reliability Rating = A
- Maintainability Rating = A

On Overall Code:
- Security Rating <= B
- Reliability Rating <= B
```

#### 4.3 Gate Conditions

**Available Metrics:**
- Coverage on New Code
- Duplicated Lines (%)
- Security Rating
- Reliability Rating
- Maintainability Rating
- Bugs
- Vulnerabilities
- Code Smells
- Security Hotspots
- Technical Debt Ratio

**Operators:**
- is less than
- is greater than
- is equal to
- is not equal to

#### 4.4 Quality Gate Enforcement

**In CI/CD:**
```bash
# Maven
mvn sonar:sonar
# Quality gate status in logs

# Check gate status
curl -u token: \
  "https://sonarqube.example.com/api/qualitygates/project_status?projectKey=my-project"
```

**Jenkins Pipeline:**
```groovy
stage('Quality Gate') {
    timeout(time: 1, unit: 'HOURS') {
        def qg = waitForQualityGate()
        if (qg.status != 'OK') {
            error "Quality gate failed: ${qg.status}"
        }
    }
}
```

### Hands-On Practice
1. Analyze quality gate conditions
2. Create custom gate for your team
3. Assign gate to project
4. Run analysis and check gate status
5. Fail build on gate failure

---

## Week 5: Code Coverage Integration

### Learning Objectives
- Configure code coverage
- Integrate JaCoCo with Maven
- Upload coverage reports
- Analyze coverage metrics

### Topics

#### 5.1 Coverage Basics

**Types of Coverage:**
- **Line Coverage**: % of lines executed
- **Branch Coverage**: % of branches tested
- **Condition Coverage**: % of boolean sub-expressions tested

**Coverage Tools:**
- Java: JaCoCo, Cobertura
- JavaScript: Istanbul, NYC
- Python: Coverage.py
- C#: dotCover, OpenCover

#### 5.2 JaCoCo with Maven

**pom.xml Configuration:**
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.jacoco</groupId>
            <artifactId>jacoco-maven-plugin</artifactId>
            <version>0.8.10</version>
            <executions>
                <execution>
                    <goals>
                        <goal>prepare-agent</goal>
                    </goals>
                </execution>
                <execution>
                    <id>report</id>
                    <phase>test</phase>
                    <goals>
                        <goal>report</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

**Run Tests and Generate Coverage:**
```bash
mvn clean test

# Coverage report generated at:
# target/site/jacoco/jacoco.xml
```

#### 5.3 Configure SonarQube for Coverage

**sonar-project.properties:**
```properties
sonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
```

**Or in Maven:**
```bash
mvn clean verify sonar:sonar \
  -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
```

### Hands-On Practice
1. Add JaCoCo plugin to Maven project
2. Write unit tests
3. Generate coverage report
4. Upload to SonarQube
5. Review coverage in SonarQube UI

---

## Week 6: Branch Analysis & Pull Requests

### Learning Objectives
- Configure branch analysis
- Set up PR decoration
- Analyze feature branches
- Review issues on PRs

### Topics

#### 6.1 Branch Analysis (Developer Edition+)

**Why Branch Analysis?**
- Analyze each branch independently
- Compare branches
- Track quality trends per branch
- Short-lived vs long-lived branches

**Configure Branch Analysis:**
```bash
# Main branch
mvn sonar:sonar \
  -Dsonar.branch.name=main

# Feature branch
mvn sonar:sonar \
  -Dsonar.branch.name=feature/new-feature
```

#### 6.2 Pull Request Decoration

**Setup (GitHub):**
1. Install SonarQube GitHub App
2. Configure in SonarQube: Administration → Configuration → General → ALM Integrations
3. Add GitHub configuration
4. Link project to repository

**Analyze PR:**
```bash
mvn sonar:sonar \
  -Dsonar.pullrequest.key=123 \
  -Dsonar.pullrequest.branch=feature/new \
  -Dsonar.pullrequest.base=main
```

**Result:**
- Quality Gate status on PR
- Issues decorated on PR
- Coverage changes shown
- Inline comments on new issues

#### 6.3 PR Analysis in Jenkins

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

### Hands-On Practice
1. Configure branch analysis
2. Create feature branch
3. Analyze feature branch
4. Compare with main branch
5. Set up PR decoration (if available)

---

## Week 7: CI/CD Integration - Maven & Jenkins

### Learning Objectives
- Integrate SonarQube with Maven
- Configure Jenkins pipeline
- Automate analysis on commits
- Implement quality gate checks

### Topics

#### 7.1 Maven Integration

**Plugin Configuration:**
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.sonarsource.scanner.maven</groupId>
            <artifactId>sonar-maven-plugin</artifactId>
            <version>3.10.0.2594</version>
        </plugin>
    </plugins>
</build>
```

**settings.xml:**
```xml
<settings>
    <pluginGroups>
        <pluginGroup>org.sonarsource.scanner.maven</pluginGroup>
    </pluginGroups>
    <profiles>
        <profile>
            <id>sonar</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <sonar.host.url>http://localhost:9000</sonar.host.url>
            </properties>
        </profile>
    </profiles>
</settings>
```

#### 7.2 Jenkins Integration

**Install Plugins:**
- SonarQube Scanner plugin
- SonarQube Quality Gate plugin

**Configure SonarQube Server:**
```
Jenkins → Manage Jenkins → Configure System → SonarQube servers
- Name: SonarQube
- Server URL: http://sonarqube:9000
- Server authentication token
```

**Jenkinsfile:**
```groovy
pipeline {
    agent any
    
    tools {
        maven 'Maven 3.8.6'
        jdk 'JDK 11'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/org/repo.git'
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
        
        stage('Deploy') {
            when {
                expression { currentBuild.result == 'SUCCESS' }
            }
            steps {
                sh 'mvn deploy'
            }
        }
    }
}
```

### Hands-On Practice
1. Configure Maven for SonarQube
2. Install Jenkins plugins
3. Configure SonarQube server in Jenkins
4. Create Jenkins pipeline
5. Run automated analysis

---

## Week 8: Advanced Integration Patterns

### Learning Objectives
- Multi-module project analysis
- Monorepo analysis
- Docker-based analysis
- GitLab CI/CD integration

### Topics

#### 8.1 Multi-Module Maven Projects

**Parent POM:**
```xml
<properties>
    <sonar.projectKey>com.example:parent</sonar.projectKey>
    <sonar.organization>my-org</sonar.organization>
</properties>
```

**Analysis:**
```bash
# From parent directory
mvn clean verify sonar:sonar

# SonarQube automatically detects modules
```

#### 8.2 Docker-Based Analysis

**Dockerfile:**
```dockerfile
FROM maven:3.8.6-jdk-11
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean verify sonar:sonar \
  -Dsonar.host.url=${SONAR_HOST_URL} \
  -Dsonar.login=${SONAR_TOKEN}
```

**Docker Compose:**
```yaml
version: '3'
services:
  sonarqube:
    image: sonarqube:lts
    ports:
      - "9000:9000"
    environment:
      - SONAR_JDBC_URL=jdbc:postgresql://db:5432/sonar
    depends_on:
      - db
  
  db:
    image: postgres:13
    environment:
      - POSTGRES_USER=sonar
      - POSTGRES_PASSWORD=sonar
      - POSTGRES_DB=sonar
```

#### 8.3 GitLab CI/CD

**.gitlab-ci.yml:**
```yaml
sonarqube-check:
  stage: test
  image: maven:3.8.6-jdk-11
  variables:
    SONAR_USER_HOME: "${CI_PROJECT_DIR}/.sonar"
  cache:
    key: "${CI_JOB_NAME}"
    paths:
      - .sonar/cache
  script:
    - mvn verify sonar:sonar
      -Dsonar.projectKey=my-project
      -Dsonar.host.url=$SONAR_HOST_URL
      -Dsonar.login=$SONAR_TOKEN
  only:
    - merge_requests
    - main
```

### Hands-On Practice
1. Analyze multi-module project
2. Set up Docker-based analysis
3. Configure GitLab CI/CD
4. Test analysis in containers

---

## Week 9: Security Analysis

### Learning Objectives
- Understand security rules
- Review security hotspots
- Fix vulnerabilities
- Configure security standards

### Topics

#### 9.1 Security Vulnerabilities

**OWASP Top 10 Coverage:**
- A1: Injection (SQL, XXE, Command)
- A2: Broken Authentication
- A3: Sensitive Data Exposure
- A4: XML External Entities (XXE)
- A5: Broken Access Control
- A6: Security Misconfiguration
- A7: Cross-Site Scripting (XSS)
- A8: Insecure Deserialization
- A9: Using Components with Known Vulnerabilities
- A10: Insufficient Logging

**Example Vulnerabilities:**
```java
// SQL Injection
String query = "SELECT * FROM users WHERE id = " + userId;

// Hardcoded Credentials
String password = "admin123";

// XSS
response.getWriter().write(request.getParameter("name"));
```

#### 9.2 Security Hotspots

**What are Hotspots?**
- Security-sensitive code requiring review
- Not vulnerabilities, but need attention
- Developer must review and mark as safe/fixed

**Common Hotspots:**
- Cryptography usage
- Authentication mechanisms
- Database queries
- File operations
- Network connections

**Reviewing Hotspots:**
1. Open Security Hotspot
2. Review code context
3. Determine if secure
4. Mark as: Safe, Fixed, or Won't Fix
5. Add comment explaining decision

#### 9.3 Security Standards

**Supported Standards:**
- OWASP Top 10
- CWE (Common Weakness Enumeration)
- SANS Top 25
- CERT
- MISRA

**Configure Security Focus:**
```
Quality Profiles → Rules → Tag: security
Activate all security rules
Set severity to Blocker/Critical
```

### Hands-On Practice
1. Activate security rules
2. Analyze project for vulnerabilities
3. Review security hotspots
4. Fix identified vulnerabilities
5. Verify fixes with re-analysis

---

## Week 10: Advanced Topics & Custom Rules

### Learning Objectives
- Configure advanced settings
- Optimize analysis performance
- Create custom rules
- Use SonarQube API

### Topics

#### 10.1 Performance Optimization

**Scanner Performance:**
```properties
# Parallel processing
sonar.scanner.threads=4

# Memory settings
SONAR_SCANNER_OPTS="-Xmx2048m"

# Incremental analysis (Enterprise+)
sonar.incrementalAnalysis=true
```

**Server Performance:**
```properties
# sonar.properties
sonar.ce.workerCount=4
sonar.web.javaOpts=-Xmx2048m -Xms1024m
sonar.search.javaOpts=-Xmx1024m -Xms1024m
```

#### 10.2 Custom Rules (Advanced)

**Java Custom Rule (Plugin Development):**
```java
@Rule(key = "CustomRule")
public class CustomRule extends BaseTreeVisitor implements JavaFileScanner {
    @Override
    public void visitMethod(MethodTree tree) {
        if (tree.simpleName().name().startsWith("get")) {
            // Check rule logic
            context.reportIssue(this, tree, "Custom rule message");
        }
        super.visitMethod(tree);
    }
}
```

#### 10.3 SonarQube API

**Common API Calls:**
```bash
# Get project status
curl -u token: \
  "http://localhost:9000/api/qualitygates/project_status?projectKey=my-project"

# Get metrics
curl -u token: \
  "http://localhost:9000/api/measures/component?component=my-project&metricKeys=bugs,vulnerabilities,code_smells"

# Create project
curl -u token: -X POST \
  "http://localhost:9000/api/projects/create?name=MyProject&project=my-project"

# Get issues
curl -u token: \
  "http://localhost:9000/api/issues/search?componentKeys=my-project&resolved=false"
```

### Hands-On Practice
1. Optimize scanner configuration
2. Tune server performance
3. Use SonarQube API
4. Export metrics programmatically
5. Automate project creation

---

## Week 11: Production Deployment

### Learning Objectives
- Deploy SonarQube for production
- Configure PostgreSQL database
- Set up high availability
- Implement backup strategy

### Topics

#### 11.1 Production Installation

**System Requirements:**
- 4GB RAM minimum (8GB+ recommended)
- 2 CPUs (4+ recommended)
- PostgreSQL 12-14 (recommended database)
- Elasticsearch (embedded)

**PostgreSQL Setup:**
```sql
CREATE DATABASE sonarqube;
CREATE USER sonarqube WITH ENCRYPTED PASSWORD 'mypassword';
GRANT ALL PRIVILEGES ON DATABASE sonarqube TO sonarqube;
ALTER DATABASE sonarqube OWNER TO sonarqube;
```

**sonar.properties:**
```properties
# Database
sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube
sonar.jdbc.username=sonarqube
sonar.jdbc.password=mypassword

# Web server
sonar.web.host=0.0.0.0
sonar.web.port=9000
sonar.web.context=/sonarqube

# Compute engine
sonar.ce.workerCount=4

# Elasticsearch
sonar.search.javaOpts=-Xmx1024m -Xms1024m
```

#### 11.2 Kubernetes Deployment

**sonarqube-deployment.yaml:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sonarqube
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sonarqube
  template:
    metadata:
      labels:
        app: sonarqube
    spec:
      containers:
      - name: sonarqube
        image: sonarqube:lts
        ports:
        - containerPort: 9000
        env:
        - name: SONAR_JDBC_URL
          value: "jdbc:postgresql://postgres:5432/sonar"
        - name: SONAR_JDBC_USERNAME
          valueFrom:
            secretKeyRef:
              name: sonar-secret
              key: username
        - name: SONAR_JDBC_PASSWORD
          valueFrom:
            secretKeyRef:
              name: sonar-secret
              key: password
        resources:
          requests:
            memory: "2Gi"
            cpu: "1000m"
          limits:
            memory: "4Gi"
            cpu: "2000m"
        volumeMounts:
        - name: sonar-data
          mountPath: /opt/sonarqube/data
      volumes:
      - name: sonar-data
        persistentVolumeClaim:
          claimName: sonar-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: sonarqube
spec:
  type: LoadBalancer
  ports:
  - port: 9000
    targetPort: 9000
  selector:
    app: sonarqube
```

#### 11.3 Backup & Restore

**Backup Strategy:**
```bash
# Database backup
pg_dump -U sonarqube sonarqube > sonarqube_backup.sql

# Data directory backup
tar -czf sonarqube_data.tar.gz /opt/sonarqube/data

# Configuration backup
cp -r /opt/sonarqube/conf /backup/conf
```

**Restore:**
```bash
# Restore database
psql -U sonarqube sonarqube < sonarqube_backup.sql

# Restore data
tar -xzf sonarqube_data.tar.gz -C /opt/sonarqube/
```

### Hands-On Practice
1. Install PostgreSQL
2. Configure production SonarQube
3. Deploy to Kubernetes
4. Implement backup strategy
5. Test disaster recovery

---

## Week 12: Enterprise Governance & Best Practices

### Learning Objectives
- Implement organization-wide standards
- Configure authentication & authorization
- Monitor and maintain SonarQube
- Establish governance policies

### Topics

#### 12.1 Authentication & Authorization

**LDAP Integration:**
```properties
# sonar.properties
sonar.security.realm=LDAP
ldap.url=ldap://ldap.example.com
ldap.bindDn=cn=sonar,ou=users,dc=example,dc=com
ldap.bindPassword=secret
ldap.user.baseDn=ou=users,dc=example,dc=com
ldap.user.request=(&(objectClass=user)(sAMAccountName={login}))
```

**SAML Integration:**
```properties
sonar.auth.saml.enabled=true
sonar.auth.saml.applicationId=sonarqube
sonar.auth.saml.providerName=SAML
sonar.auth.saml.providerId=https://idp.example.com
sonar.auth.saml.loginUrl=https://idp.example.com/sso
```

**Permission Management:**
- Global Permissions (Admin, Quality Profile Admin)
- Project Permissions (Browse, Execute Analysis, Admin)
- Groups and Users

#### 12.2 Governance Policies

**Organization Standards:**
1. **Quality Gate**:
   - Single organization-wide gate
   - Exception process for deviations
   - Regular review and updates

2. **Quality Profiles**:
   - Language-specific profiles
   - Inheritance hierarchy
   - Version control in Git

3. **Project Organization**:
   - Naming conventions
   - Tagging strategy
   - Portfolio structure

#### 12.3 Monitoring & Maintenance

**Health Monitoring:**
```bash
# System health
curl http://localhost:9000/api/system/health

# System status
curl http://localhost:9000/api/system/status

# Database stats
curl -u admin:admin http://localhost:9000/api/system/db_migration_status
```

**Regular Maintenance:**
- Weekly: Review quality gate failures
- Monthly: Update quality profiles
- Quarterly: Review security vulnerabilities
- Yearly: Upgrade SonarQube version

**Housekeeping:**
```sql
-- Clean old snapshots (keep last 30 days)
-- Administration → Configuration → General → Housekeeping
Days before deleting closed issues: 30
Days before deleting inactive branches: 30
```

### Hands-On Practice
1. Configure LDAP/SAML
2. Set up permission groups
3. Create governance policies
4. Monitor system health
5. Perform maintenance tasks

---

## Certification & Next Steps

### SonarSource Certifications
While SonarSource doesn't have formal certifications, demonstrate expertise through:
- GitHub projects with quality analysis
- Blog posts about SonarQube implementation
- Contributions to SonarQube community
- Case studies from production deployments

### Continue Learning
1. **Jenkins** - Next in pipeline
2. **Docker** - Containerization
3. **Kubernetes** - Orchestration
4. Advanced DevSecOps practices

### Community Engagement
- SonarSource Community Forum
- Stack Overflow (sonarqube tag)
- GitHub Issues
- Contributing to SonarQube plugins

---

## Summary

After 12 weeks, you will have mastered:
✅ SonarQube installation and configuration  
✅ Code quality and security analysis  
✅ Quality gates and profiles management  
✅ CI/CD integration  
✅ Security vulnerability detection  
✅ Production deployment and maintenance  
✅ Enterprise governance and best practices

**Next Package**: [Jenkins](../jenkins/) for complete CI/CD automation!

**Happy Learning! 🚀**


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

