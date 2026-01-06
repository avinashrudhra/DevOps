# SonarQube Quick Reference Guide

Essential commands, configurations, and templates for daily SonarQube operations.

---

## Table of Contents
1. [Installation & Setup](#installation--setup)
2. [Scanner Commands](#scanner-commands)
3. [Quality Gates](#quality-gates)
4. [Quality Profiles](#quality-profiles)
5. [Configuration Files](#configuration-files)
6. [REST API](#rest-api)
7. [CI/CD Integration](#cicd-integration)
8. [Troubleshooting Commands](#troubleshooting-commands)

---

## Installation & Setup

### Quick Install (Community Edition)
```bash
# Download
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.3.79811.zip
unzip sonarqube-9.9.3.79811.zip
cd sonarqube-9.9.3.79811

# Start
bin/linux-x86-64/sonar.sh start

# Stop
bin/linux-x86-64/sonar.sh stop

# Status
bin/linux-x86-64/sonar.sh status

# Logs
tail -f logs/sonar.log
```

### Docker Installation
```bash
# Run SonarQube
docker run -d --name sonarqube \
  -p 9000:9000 \
  -v sonarqube_data:/opt/sonarqube/data \
  -v sonarqube_logs:/opt/sonarqube/logs \
  -v sonarqube_extensions:/opt/sonarqube/extensions \
  sonarqube:lts

# With PostgreSQL
docker run -d --name postgres \
  -e POSTGRES_USER=sonar \
  -e POSTGRES_PASSWORD=sonar \
  -e POSTGRES_DB=sonar \
  postgres:13

docker run -d --name sonarqube \
  -p 9000:9000 \
  -e SONAR_JDBC_URL=jdbc:postgresql://postgres:5432/sonar \
  -e SONAR_JDBC_USERNAME=sonar \
  -e SONAR_JDBC_PASSWORD=sonar \
  --link postgres \
  sonarqube:lts
```

### Docker Compose
```yaml
version: '3'
services:
  sonarqube:
    image: sonarqube:lts
    ports:
      - "9000:9000"
    environment:
      - SONAR_JDBC_URL=jdbc:postgresql://db:5432/sonar
      - SONAR_JDBC_USERNAME=sonar
      - SONAR_JDBC_PASSWORD=sonar
    volumes:
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_logs:/opt/sonarqube/logs
      - sonarqube_extensions:/opt/sonarqube/extensions
    depends_on:
      - db
  
  db:
    image: postgres:13
    environment:
      - POSTGRES_USER=sonar
      - POSTGRES_PASSWORD=sonar
      - POSTGRES_DB=sonar
    volumes:
      - postgresql_data:/var/lib/postgresql/data

volumes:
  sonarqube_data:
  sonarqube_logs:
  sonarqube_extensions:
  postgresql_data:
```

---

## Scanner Commands

### Maven Scanner
```bash
# Basic analysis
mvn clean verify sonar:sonar

# With parameters
mvn sonar:sonar \
  -Dsonar.projectKey=my-project \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=your-token

# With coverage
mvn clean verify sonar:sonar \
  -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml

# Skip tests but use existing coverage
mvn sonar:sonar -DskipTests

# Debug mode
mvn sonar:sonar -X

# Offline mode
mvn sonar:sonar -o
```

### Gradle Scanner
```bash
# Basic analysis
./gradlew sonarqube

# With parameters
./gradlew sonarqube \
  -Dsonar.projectKey=my-project \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=your-token

# With coverage
./gradlew test jacocoTestReport sonarqube
```

### SonarScanner CLI
```bash
# Install
# Linux
unzip sonar-scanner-cli-4.8.0.2856-linux.zip
export PATH=$PATH:sonar-scanner-4.8.0.2856-linux/bin

# Mac
brew install sonar-scanner

# Basic scan
sonar-scanner \
  -Dsonar.projectKey=my-project \
  -Dsonar.sources=. \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=your-token

# With properties file
sonar-scanner

# Debug
sonar-scanner -X
```

### Branch Analysis (Developer Edition+)
```bash
# Main branch
mvn sonar:sonar -Dsonar.branch.name=main

# Feature branch
mvn sonar:sonar -Dsonar.branch.name=feature/my-feature

# Long-lived branch
mvn sonar:sonar \
  -Dsonar.branch.name=develop \
  -Dsonar.branch.target=main

# Short-lived branch
mvn sonar:sonar \
  -Dsonar.branch.name=feature/temp \
  -Dsonar.branch.target=main \
  -Dsonar.branch.type=short
```

### Pull Request Analysis (Developer Edition+)
```bash
# GitHub PR
mvn sonar:sonar \
  -Dsonar.pullrequest.key=123 \
  -Dsonar.pullrequest.branch=feature/new-feature \
  -Dsonar.pullrequest.base=main

# GitLab Merge Request
mvn sonar:sonar \
  -Dsonar.pullrequest.key=$CI_MERGE_REQUEST_IID \
  -Dsonar.pullrequest.branch=$CI_MERGE_REQUEST_SOURCE_BRANCH_NAME \
  -Dsonar.pullrequest.base=$CI_MERGE_REQUEST_TARGET_BRANCH_NAME
```

---

## Quality Gates

### Default Quality Gate (Sonar way)
```yaml
Conditions on New Code:
  - Coverage >= 80%
  - Duplicated Lines < 3%
  - Maintainability Rating = A
  - Reliability Rating = A
  - Security Rating = A
  - Security Hotspots Reviewed = 100%
```

### Strict Quality Gate Template
```yaml
On New Code:
  - Coverage >= 90%
  - Duplicated Lines < 2%
  - Bugs = 0
  - Vulnerabilities = 0
  - Security Hotspots Reviewed = 100%
  - Code Smells <= 5
  - Maintainability Rating = A
  - Reliability Rating = A
  - Security Rating = A
  - Technical Debt Ratio < 5%

On Overall Code:
  - Security Rating <= B
  - Reliability Rating <= B
  - Duplicated Lines < 5%
```

### Relaxed Quality Gate Template
```yaml
On New Code:
  - Coverage >= 70%
  - Duplicated Lines < 5%
  - Maintainability Rating <= B
  - Reliability Rating <= B
  - Security Rating = A
```

### Security-Focused Quality Gate
```yaml
On New Code:
  - Vulnerabilities = 0
  - Security Rating = A
  - Security Hotspots Reviewed = 100%
  - Bugs = 0
  - Reliability Rating = A

On Overall Code:
  - Security Rating = A
  - Vulnerabilities = 0
```

### Check Quality Gate Status
```bash
# Via API
curl -u token: \
  "http://localhost:9000/api/qualitygates/project_status?projectKey=my-project"

# Via Maven (check logs)
mvn sonar:sonar | grep "QUALITY GATE"
```

---

## Quality Profiles

### Activate Rules by Tag
```
Quality Profiles → [Profile] → Activate More
- Tag: security → Activate all
- Tag: owasp → Activate all
- Tag: performance → Activate selected
- Tag: cert → Activate all
- Tag: cwe → Activate all
```

### Custom Java Profile Template
```
Create Profile:
  Name: "Company Java Standard"
  Language: Java
  Parent: Sonar way
  
Modifications:
  1. Activate all security rules
  2. Set complexity threshold: 15
  3. Set method lines threshold: 50
  4. Enable performance rules
  5. Custom exclusions for generated code
```

### Export/Import Profile
```bash
# Export (UI)
Quality Profiles → [Profile] → Back Up → Download

# Import (UI)
Quality Profiles → Restore

# Via API - Export
curl -u token: \
  "http://localhost:9000/api/qualityprofiles/backup?language=java&qualityProfile=Sonar%20way" \
  > profile.xml

# Via API - Import
curl -u admin:admin -X POST \
  -F "backup=@profile.xml" \
  "http://localhost:9000/api/qualityprofiles/restore"
```

---

## Configuration Files

### sonar-project.properties
```properties
# Required settings
sonar.projectKey=my-project
sonar.projectName=My Project
sonar.projectVersion=1.0

# Comma-separated paths to directories with sources
sonar.sources=src/main/java

# Comma-separated paths to directories with tests
sonar.tests=src/test/java

# Source encoding
sonar.sourceEncoding=UTF-8

# Exclusions
sonar.exclusions=**/generated/**,**/vendor/**,**/*.min.js

# Test exclusions
sonar.test.exclusions=**/test/**/*.java

# Coverage
sonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml

# Language
sonar.language=java

# Java specific
sonar.java.binaries=target/classes
sonar.java.libraries=target/dependency/*.jar
```

### Maven pom.xml Properties
```xml
<properties>
    <!-- SonarQube -->
    <sonar.organization>my-org</sonar.organization>
    <sonar.host.url>https://sonarcloud.io</sonar.host.url>
    <sonar.projectKey>com.example:my-project</sonar.projectKey>
    
    <!-- Coverage -->
    <sonar.coverage.jacoco.xmlReportPaths>
        ${project.build.directory}/site/jacoco/jacoco.xml
    </sonar.coverage.jacoco.xmlReportPaths>
    
    <!-- Exclusions -->
    <sonar.exclusions>
        **/generated/**,
        **/dto/**,
        **/config/**
    </sonar.exclusions>
    
    <!-- Test exclusions -->
    <sonar.test.exclusions>
        **/test/**
    </sonar.test.exclusions>
    
    <!-- CPD (Copy-Paste Detection) -->
    <sonar.cpd.exclusions>
        **/model/**
    </sonar.cpd.exclusions>
</properties>
```

### Gradle build.gradle
```groovy
plugins {
    id "org.sonarqube" version "3.5.0.2730"
}

sonarqube {
    properties {
        property "sonar.projectKey", "my-project"
        property "sonar.projectName", "My Project"
        property "sonar.host.url", "http://localhost:9000"
        property "sonar.login", "your-token"
        
        // Sources
        property "sonar.sources", "src/main/java"
        property "sonar.tests", "src/test/java"
        
        // Exclusions
        property "sonar.exclusions", "**/generated/**"
        
        // Coverage
        property "sonar.coverage.jacoco.xmlReportPaths", 
                 "${buildDir}/reports/jacoco/test/jacocoTestReport.xml"
    }
}
```

### sonar.properties (Server)
```properties
# Database
sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube
sonar.jdbc.username=sonarqube
sonar.jdbc.password=mypassword

# Web Server
sonar.web.host=0.0.0.0
sonar.web.port=9000
sonar.web.context=/sonarqube

# Compute Engine
sonar.ce.workerCount=4
sonar.ce.javaOpts=-Xmx2048m -Xms1024m

# Elasticsearch
sonar.search.javaOpts=-Xmx1024m -Xms1024m

# Paths
sonar.path.data=/opt/sonarqube/data
sonar.path.temp=/opt/sonarqube/temp

# Logging
sonar.log.level=INFO

# Authentication
sonar.forceAuthentication=true

# LDAP
sonar.security.realm=LDAP
ldap.url=ldap://ldap.example.com
ldap.bindDn=cn=sonar,ou=users,dc=example,dc=com
ldap.bindPassword=secret
```

---

## REST API

### Authentication
```bash
# Generate token (UI)
My Account → Security → Generate Token

# Use token in API
curl -u YOUR_TOKEN: "http://localhost:9000/api/..."

# Or with username:password
curl -u admin:admin "http://localhost:9000/api/..."
```

### Common API Endpoints

#### Project Management
```bash
# Create project
curl -u token: -X POST \
  "http://localhost:9000/api/projects/create?name=MyProject&project=my-project"

# Delete project
curl -u token: -X POST \
  "http://localhost:9000/api/projects/delete?project=my-project"

# Search projects
curl -u token: \
  "http://localhost:9000/api/projects/search"

# Get project status
curl -u token: \
  "http://localhost:9000/api/qualitygates/project_status?projectKey=my-project"
```

#### Metrics & Measures
```bash
# Get component measures
curl -u token: \
  "http://localhost:9000/api/measures/component?component=my-project&metricKeys=bugs,vulnerabilities,code_smells,coverage"

# Get metrics list
curl -u token: \
  "http://localhost:9000/api/metrics/search"

# Get all measures for project
curl -u token: \
  "http://localhost:9000/api/measures/component?component=my-project&metricKeys=ncloc,complexity,violations"
```

#### Issues
```bash
# Search issues
curl -u token: \
  "http://localhost:9000/api/issues/search?componentKeys=my-project&resolved=false"

# Search by severity
curl -u token: \
  "http://localhost:9000/api/issues/search?componentKeys=my-project&severities=CRITICAL,BLOCKER"

# Search by type
curl -u token: \
  "http://localhost:9000/api/issues/search?componentKeys=my-project&types=BUG,VULNERABILITY"

# Assign issue
curl -u token: -X POST \
  "http://localhost:9000/api/issues/assign?issue=ISSUE_KEY&assignee=john"

# Add comment
curl -u token: -X POST \
  "http://localhost:9000/api/issues/add_comment?issue=ISSUE_KEY&text=my-comment"
```

#### Quality Gates
```bash
# List quality gates
curl -u token: \
  "http://localhost:9000/api/qualitygates/list"

# Create quality gate
curl -u token: -X POST \
  "http://localhost:9000/api/qualitygates/create?name=MyGate"

# Get project gate
curl -u token: \
  "http://localhost:9000/api/qualitygates/get_by_project?project=my-project"

# Set project gate
curl -u token: -X POST \
  "http://localhost:9000/api/qualitygates/select?gateId=1&projectKey=my-project"
```

#### Quality Profiles
```bash
# Search profiles
curl -u token: \
  "http://localhost:9000/api/qualityprofiles/search"

# Set default profile
curl -u token: -X POST \
  "http://localhost:9000/api/qualityprofiles/set_default?language=java&qualityProfile=MyProfile"

# Add project to profile
curl -u token: -X POST \
  "http://localhost:9000/api/qualityprofiles/add_project?project=my-project&language=java&qualityProfile=MyProfile"
```

#### System
```bash
# System health
curl -u token: \
  "http://localhost:9000/api/system/health"

# System status
curl -u token: \
  "http://localhost:9000/api/system/status"

# System info
curl -u token: \
  "http://localhost:9000/api/system/info"

# Database migration status
curl -u token: \
  "http://localhost:9000/api/system/db_migration_status"
```

---

## CI/CD Integration

### Jenkins Pipeline
```groovy
pipeline {
    agent any
    
    tools {
        maven 'Maven 3.8'
        jdk 'JDK 11'
    }
    
    environment {
        SCANNER_HOME = tool 'SonarQubeScanner'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/org/repo.git'
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
                branch 'main'
            }
            steps {
                sh 'mvn deploy'
            }
        }
    }
    
    post {
        always {
            junit '**/target/surefire-reports/*.xml'
        }
    }
}
```

### GitLab CI/CD
```yaml
variables:
  SONAR_USER_HOME: "${CI_PROJECT_DIR}/.sonar"
  GIT_DEPTH: "0"

stages:
  - build
  - test
  - sonarqube
  - deploy

build:
  stage: build
  image: maven:3.8-jdk-11
  script:
    - mvn clean compile
  artifacts:
    paths:
      - target/

test:
  stage: test
  image: maven:3.8-jdk-11
  script:
    - mvn test
  artifacts:
    paths:
      - target/

sonarqube-check:
  stage: sonarqube
  image: maven:3.8-jdk-11
  cache:
    key: "${CI_JOB_NAME}"
    paths:
      - .sonar/cache
  script:
    - mvn verify sonar:sonar
      -Dsonar.projectKey=$CI_PROJECT_NAME
      -Dsonar.host.url=$SONAR_HOST_URL
      -Dsonar.login=$SONAR_TOKEN
  allow_failure: false
  only:
    - merge_requests
    - main
    - develop
```

### GitHub Actions
```yaml
name: SonarQube Analysis

on:
  push:
    branches: [main, develop]
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  sonarqube:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
      with:
        fetch-depth: 0
    
    - name: Set up JDK 11
      uses: actions/setup-java@v3
      with:
        java-version: '11'
        distribution: 'temurin'
        cache: maven
    
    - name: Build and analyze
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      run: |
        mvn clean verify sonar:sonar \
          -Dsonar.projectKey=my-project \
          -Dsonar.host.url=${{ secrets.SONAR_HOST_URL }} \
          -Dsonar.login=${{ secrets.SONAR_TOKEN }}
```

### Azure DevOps
```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  MAVEN_CACHE_FOLDER: $(Pipeline.Workspace)/.m2/repository

steps:
- task: Maven@3
  inputs:
    mavenPomFile: 'pom.xml'
    goals: 'clean verify'
    options: '-DskipTests'
    jdkVersionOption: '1.11'

- task: SonarQubePrepare@5
  inputs:
    SonarQube: 'SonarQube-Connection'
    scannerMode: 'Maven'
    configMode: 'manual'
    projectKey: 'my-project'
    projectName: 'My Project'

- task: Maven@3
  inputs:
    mavenPomFile: 'pom.xml'
    goals: 'sonar:sonar'
    jdkVersionOption: '1.11'

- task: SonarQubePublish@5
  inputs:
    pollingTimeoutSec: '300'
```

---

## Troubleshooting Commands

### Check Scanner Version
```bash
mvn sonar:help -Ddetail=true
sonar-scanner --version
```

### Debug Analysis
```bash
# Maven debug
mvn sonar:sonar -X -e

# SonarScanner debug
sonar-scanner -X

# Show scanner properties
mvn sonar:sonar -Dsonar.verbose=true
```

### Clear Scanner Cache
```bash
# Maven
mvn clean
rm -rf ~/.sonar/cache

# Gradle
./gradlew clean
rm -rf ~/.sonar/cache
```

### Check Server Status
```bash
# Health check
curl http://localhost:9000/api/system/health

# Status
curl http://localhost:9000/api/system/status

# Ping
curl http://localhost:9000/api/system/ping
```

### Database Connection Test
```bash
# From server directory
./bin/linux-x86-64/sonar.sh console

# Check logs
tail -f logs/web.log
tail -f logs/ce.log
tail -f logs/es.log
```

### View Compute Engine Tasks
```bash
# Via API
curl -u token: \
  "http://localhost:9000/api/ce/activity?status=FAILED"

# Via UI
Administration → Projects → Background Tasks
```

---

## Metrics Quick Reference

| Metric | Key | Description | Target |
|--------|-----|-------------|--------|
| Bugs | `bugs` | Reliability issues | 0 |
| Vulnerabilities | `vulnerabilities` | Security issues | 0 |
| Code Smells | `code_smells` | Maintainability issues | < 100 |
| Coverage | `coverage` | Test coverage % | > 80% |
| Duplications | `duplicated_lines_density` | % duplicated lines | < 3% |
| Lines of Code | `ncloc` | Non-commented lines | N/A |
| Complexity | `complexity` | Cyclomatic complexity | Low |
| Technical Debt | `sqale_index` | Remediation time (min) | Low |
| Debt Ratio | `sqale_debt_ratio` | Debt / Dev cost | < 5% |
| Security Rating | `security_rating` | A (best) to E | A |
| Reliability Rating | `reliability_rating` | A (best) to E | A |
| Maintainability Rating | `sqale_rating` | A (best) to E | A |

---

## Useful Links

- **Official Docs**: https://docs.sonarqube.org/latest/
- **Community**: https://community.sonarsource.com/
- **API Docs**: http://localhost:9000/web_api
- **Rules**: https://rules.sonarsource.com/
- **SonarCloud**: https://sonarcloud.io/

---

**Tip**: Bookmark this page for quick access to common commands and configurations!


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

