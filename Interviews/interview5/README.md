# DevOps Interview Session 5
## Native DevOps Tools Focus (3-7 Years)

---

### Question 1: Git Cherry-Pick vs Merge

**Interviewer:** When would you use cherry-pick instead of merge?

**Candidate:** Cherry-pick takes a specific commit from one branch and applies it to another. Use it for hotfixes - pick just the fix commit to production without merging unrelated feature work. Merge brings all commits, cherry-pick is selective.

```bash
# Hotfix scenario
git checkout production
git cherry-pick abc1234  # Just the fix commit
git push origin production

# Later, ensure develop gets it too
git checkout develop
git cherry-pick abc1234
```

**Interviewer:** Any risks?

**Candidate:** Creates duplicate commits with different hashes. Can cause conflicts later when branches eventually merge. Use sparingly for urgent fixes only.

---

### Question 2: Maven Dependency Scope

**Interviewer:** Explain Maven dependency scopes and when to use each.

**Candidate:** 
- **compile** (default): Available in all classpaths, included in final artifact
- **provided**: Needed for compilation but not bundled (container provides it, like servlet-api)
- **runtime**: Not needed for compilation, only runtime (like JDBC drivers)
- **test**: Only for tests, not bundled
- **system**: Like provided but you specify JAR path (avoid this)

```xml
<dependencies>
  <!-- Compile - included in WAR -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <scope>compile</scope>
  </dependency>
  
  <!-- Provided - Tomcat already has this -->
  <dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>servlet-api</artifactId>
    <scope>provided</scope>
  </dependency>
  
  <!-- Test only -->
  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <scope>test</scope>
  </dependency>
</dependencies>
```

**Interviewer:** Why is scope important?

**Candidate:** Wrong scope bloats artifact size or causes ClassNotFoundExceptions. Provided scope prevents duplicate JARs. Test scope keeps test libraries out of production.

---

### Question 3: SonarQube Custom Rules

**Interviewer:** Your team has coding standards not covered by default SonarQube rules. How do you enforce them?

**Candidate:** Create custom rules using SonarQube plugins or XPath rules for simple cases. For Java, extend SonarJava plugin with custom checks.

```java
// Custom rule example - detect hardcoded secrets
@Rule(key = "NoHardcodedSecrets")
public class NoHardcodedSecretsRule extends BaseTreeVisitor implements JavaFileScanner {
    @Override
    public void visitLiteralExpression(LiteralTree tree) {
        String value = tree.value();
        if (value.matches(".*password.*|.*secret.*|.*apikey.*")) {
            context.reportIssue(this, tree, "Hardcoded secret detected");
        }
        super.visitLiteralExpression(tree);
    }
}
```

**Interviewer:** Easier approach?

**Candidate:** Quality Profile templates - duplicate existing profile, adjust rule severities, add community plugins. For simple regex patterns, use built-in rule customization.

---

### Question 4: Docker Volume Drivers

**Interviewer:** Beyond local volumes, what other volume drivers exist?

**Candidate:** Several for different use cases:
- **local**: Default, stores on host
- **nfs**: Mount NFS shares
- **azure-file**: Azure Files for shared storage
- **rexray**: Multi-cloud storage (EBS, Azure Disk)
- **vieux/sshfs**: Mount remote directories via SSH

```bash
# Create volume with Azure Files driver
docker volume create \
  --driver azure-file \
  --name myvolume \
  -o share=myshare \
  -o storageaccount=mystorageaccount \
  -o storageaccountkey=<key>

# Use in container
docker run -v myvolume:/data myapp
```

**Interviewer:** When would you use network volumes?

**Candidate:** Shared data across multiple containers/hosts - like uploaded files in a web farm. Local volumes isolate to single host, network volumes enable multi-host persistence.

---

### Question 5: Jenkins Shared Library Structure

**Interviewer:** You're building reusable Jenkins pipelines. Structure a shared library.

**Candidate:**
```
jenkins-shared-library/
├── vars/
│   ├── dockerBuild.groovy
│   ├── kubernetDeploy.groovy
│   └── sonarScan.groovy
├── src/
│   └── com/
│       └── company/
│           └── jenkins/
│               ├── Docker.groovy
│               └── Kubernetes.groovy
└── resources/
    └── templates/
        └── Dockerfile.template

# vars/dockerBuild.groovy
def call(Map config) {
    docker.build("${config.imageName}:${env.BUILD_ID}", config.dockerfilePath)
    docker.withRegistry(config.registry, config.credentialsId) {
        docker.image("${config.imageName}:${env.BUILD_ID}").push()
        docker.image("${config.imageName}:${env.BUILD_ID}").push('latest')
    }
}

# Usage in Jenkinsfile
@Library('jenkins-shared-library') _
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                dockerBuild(
                    imageName: 'myapp',
                    dockerfilePath: '.',
                    registry: 'https://myregistry.azurecr.io',
                    credentialsId: 'acr-credentials'
                )
            }
        }
    }
}
```

**Interviewer:** How do you version shared libraries?

**Candidate:** Tag releases in Git, reference specific versions in Jenkinsfile - `@Library('my-lib@v1.2.0')`. Allows testing new versions before updating all pipelines.

---

### Question 6: Linux SystemD Service Management

**Interviewer:** Create a systemd service for a Node.js application.

**Candidate:**
```bash
# /etc/systemd/system/myapp.service
[Unit]
Description=My Node.js Application
After=network.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/node server.js
Restart=on-failure
RestartSec=10
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=myapp
Environment=NODE_ENV=production
Environment=PORT=3000

[Install]
WantedBy=multi-user.target

# Enable and start
sudo systemctl daemon-reload
sudo systemctl enable myapp
sudo systemctl start myapp
sudo systemctl status myapp

# View logs
sudo journalctl -u myapp -f
```

**Interviewer:** What's Restart=on-failure do?

**Candidate:** Auto-restarts service if it crashes. RestartSec=10 waits 10 seconds between restart attempts. Prevents rapid restart loops if app crashes immediately.

---

### Question 7: Python Script for Log Analysis

**Interviewer:** Write a Python script to parse application logs and report errors.

**Candidate:**
```python
#!/usr/bin/env python3
import re
from collections import defaultdict
from datetime import datetime

def analyze_logs(log_file):
    """Parse logs and count error types"""
    error_counts = defaultdict(int)
    error_examples = defaultdict(list)
    
    error_pattern = re.compile(r'\[ERROR\] (.*)')
    timestamp_pattern = re.compile(r'^(\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2})')
    
    with open(log_file, 'r') as f:
        for line in f:
            # Check for errors
            error_match = error_pattern.search(line)
            if error_match:
                error_msg = error_match.group(1)
                
                # Categorize error
                if 'database' in error_msg.lower():
                    category = 'Database Errors'
                elif 'timeout' in error_msg.lower():
                    category = 'Timeout Errors'
                elif 'null' in error_msg.lower():
                    category = 'Null Pointer Errors'
                else:
                    category = 'Other Errors'
                
                error_counts[category] += 1
                
                # Store example (first 3 per category)
                if len(error_examples[category]) < 3:
                    timestamp_match = timestamp_pattern.search(line)
                    timestamp = timestamp_match.group(1) if timestamp_match else 'Unknown'
                    error_examples[category].append(f"{timestamp}: {error_msg}")
    
    return error_counts, error_examples

def generate_report(error_counts, error_examples):
    """Generate formatted error report"""
    print("=" * 80)
    print("ERROR ANALYSIS REPORT")
    print("=" * 80)
    print()
    
    total_errors = sum(error_counts.values())
    print(f"Total Errors: {total_errors}")
    print()
    
    for category, count in sorted(error_counts.items(), key=lambda x: x[1], reverse=True):
        percentage = (count / total_errors) * 100
        print(f"{category}: {count} ({percentage:.1f}%)")
        print("Examples:")
        for example in error_examples[category]:
            print(f"  - {example}")
        print()

if __name__ == '__main__':
    import sys
    if len(sys.argv) != 2:
        print("Usage: python analyze_logs.py <log_file>")
        sys.exit(1)
    
    log_file = sys.argv[1]
    error_counts, error_examples = analyze_logs(log_file)
    generate_report(error_counts, error_examples)
```

**Interviewer:** How would you extend this for real-time monitoring?

**Candidate:** Use `tail -f` equivalent with `watchdog` library or read from syslog socket. Send alerts when error rate exceeds threshold using requests library to call webhook.

---

### Question 8: GitHub Actions Matrix Strategy

**Interviewer:** Test your application across multiple versions and OSes.

**Candidate:**
```yaml
name: CI Test Matrix

on: [push, pull_request]

jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [14, 16, 18, 20]
        exclude:
          - os: macos-latest
            node-version: 14  # Exclude specific combinations
      fail-fast: false  # Continue even if one fails
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run tests
      run: npm test
    
    - name: Upload coverage
      if: matrix.os == 'ubuntu-latest' && matrix.node-version == '18'
      uses: codecov/codecov-action@v3
```

**Interviewer:** This creates how many jobs?

**Candidate:** 11 jobs - 4 Node versions × 3 OSes = 12, minus 1 excluded combination = 11 parallel jobs.

---

### Question 9: Ansible Vault for Secrets

**Interviewer:** Securely manage passwords in Ansible playbooks.

**Candidate:**
```bash
# Create encrypted file
ansible-vault create secrets.yml
# Enter vault password, then edit:
db_password: SuperSecret123
api_key: abc-def-ghi-jkl

# Or encrypt existing file
ansible-vault encrypt secrets.yml

# Use in playbook
# playbook.yml
---
- name: Deploy Application
  hosts: webservers
  vars_files:
    - secrets.yml
  tasks:
    - name: Configure database
      template:
        src: app.conf.j2
        dest: /etc/app/app.conf
      # Template uses {{ db_password }}

# Run playbook
ansible-playbook playbook.yml --ask-vault-pass

# Or use password file
ansible-playbook playbook.yml --vault-password-file ~/.vault_pass
```

**Interviewer:** Can you encrypt just one variable?

**Candidate:** Yes - `ansible-vault encrypt_string 'password123' --name 'db_password'`. Embeds encrypted string directly in playbook. Rest of file stays readable.

---

### Question 10: Prometheus PromQL Queries

**Interviewer:** Write queries to monitor application health.

**Candidate:**
```promql
# Request rate (requests per second)
rate(http_requests_total[5m])

# Error rate percentage
(rate(http_requests_total{status=~"5.."}[5m]) 
  / 
 rate(http_requests_total[5m])) * 100

# 95th percentile response time
histogram_quantile(0.95, 
  rate(http_request_duration_seconds_bucket[5m]))

# Memory usage percentage
(container_memory_usage_bytes 
  / 
 container_spec_memory_limit_bytes) * 100

# CPU throttling
rate(container_cpu_cfs_throttled_seconds_total[5m])

# Pod restart count
increase(kube_pod_container_status_restarts_total[1h])

# Disk usage alert query
(node_filesystem_avail_bytes{mountpoint="/"} 
  / 
 node_filesystem_size_bytes{mountpoint="/"}) * 100 < 20
```

**Interviewer:** How do you use these in alerts?

**Candidate:** Define alerting rules referencing these queries with thresholds and duration. Prometheus evaluates continuously, fires alerts to Alertmanager when conditions met.

---

### Question 11: Grafana Dashboard Variables

**Interviewer:** Create interactive dashboards with user-selectable filters.

**Candidate:**
```
Dashboard Settings → Variables → Add variable

Name: namespace
Label: Namespace
Type: Query
Data source: Prometheus
Query: label_values(kube_pod_info, namespace)
Multi-value: true
Include All option: true

# Use in panel queries:
rate(http_requests_total{namespace=~"$namespace"}[5m])

# Users can now select namespaces from dropdown
# Dashboard updates automatically
```

**Interviewer:** Other variable types?

**Candidate:** Custom (hardcoded list), Interval (time ranges), Text box (free input), Constant (hidden values), Data source (switch between Prometheus instances), Query (from data source). Variables enable reusable, interactive dashboards.

---

### Question 12: Helm Chart Dependencies

**Interviewer:** Your application needs MySQL. Include it as Helm dependency.

**Candidate:**
```yaml
# Chart.yaml
apiVersion: v2
name: myapp
version: 1.0.0
dependencies:
- name: mysql
  version: 9.3.0
  repository: https://charts.bitnami.com/bitnami
  condition: mysql.enabled  # Optional conditional

# values.yaml
mysql:
  enabled: true
  auth:
    rootPassword: changeme
    database: myapp_db
  primary:
    persistence:
      size: 20Gi

# Update dependencies
helm dependency update

# This downloads mysql chart to charts/ directory
# helm install deploys both charts together
```

**Interviewer:** What if you want external MySQL instead?

**Candidate:** Set `mysql.enabled: false` in production values, provide external connection details. Internal MySQL for dev, external managed MySQL for production - same chart.

---

### Question 13: ArgoCD App of Apps Pattern

**Interviewer:** Manage multiple applications declaratively with ArgoCD.

**Candidate:**
```yaml
# apps/root-app.yaml - The "app of apps"
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: root-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/org/gitops
    targetRevision: HEAD
    path: apps  # Points to directory with other Application manifests
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated:
      prune: true
      selfHeal: true

# apps/frontend-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: frontend
spec:
  project: default
  source:
    repoURL: https://github.com/org/gitops
    path: manifests/frontend
  destination:
    namespace: production

# apps/backend-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: backend
spec:
  # ... similar structure
```

**Interviewer:** Benefits?

**Candidate:** Single source of truth - one Application controls all others. Add new app by committing YAML. Ensures all apps exist, automatically creates missing ones. GitOps for your GitOps apps.

---

### Question 14: Docker BuildX Bake

**Interviewer:** Build multiple related images efficiently.

**Candidate:**
```hcl
# docker-bake.hcl
group "default" {
  targets = ["frontend", "backend", "worker"]
}

target "frontend" {
  context = "./frontend"
  dockerfile = "Dockerfile"
  tags = ["myregistry.azurecr.io/frontend:latest"]
  platforms = ["linux/amd64", "linux/arm64"]
}

target "backend" {
  context = "./backend"
  dockerfile = "Dockerfile"
  tags = ["myregistry.azurecr.io/backend:latest"]
  platforms = ["linux/amd64", "linux/arm64"]
}

target "worker" {
  context = "./worker"
  dockerfile = "Dockerfile"
  tags = ["myregistry.azurecr.io/worker:latest"]
  args = {
    NODE_VERSION = "18"
  }
}

# Build all at once
docker buildx bake

# Build specific target
docker buildx bake backend

# Override values
docker buildx bake --set *.tags=myregistry.azurecr.io/*:v1.2.0
```

**Interviewer:** Why use this instead of shell script?

**Candidate:** Parallel builds, better caching across images, consistent configuration, platform specification in one place. Cleaner than bash scripts with docker build loops.

---

### Question 15: Maven Profiles for Environment-Specific Builds

**Interviewer:** Build application with different configurations for dev/staging/prod.

**Candidate:**
```xml
<profiles>
  <!-- Development profile -->
  <profile>
    <id>dev</id>
    <activation>
      <activeByDefault>true</activeByDefault>
    </activation>
    <properties>
      <env>dev</env>
      <db.url>jdbc:postgresql://localhost:5432/myapp_dev</db.url>
      <log.level>DEBUG</log.level>
    </properties>
  </profile>
  
  <!-- Production profile -->
  <profile>
    <id>prod</id>
    <properties>
      <env>prod</env>
      <db.url>jdbc:postgresql://prod-db.company.com:5432/myapp</db.url>
      <log.level>WARN</log.level>
    </properties>
    <build>
      <plugins>
        <!-- Additional optimizations for prod -->
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-compiler-plugin</artifactId>
          <configuration>
            <debug>false</debug>
            <optimize>true</optimize>
          </configuration>
        </plugin>
      </plugins>
    </build>
  </profile>
</profiles>

<!-- Use in resources filtering -->
<build>
  <resources>
    <resource>
      <directory>src/main/resources</directory>
      <filtering>true</filtering>
    </resource>
  </resources>
</build>

# application.properties
database.url=${db.url}
logging.level=${log.level}

# Build with profile
mvn clean package -Pprod
```

---

### Question 16: SonarQube Project Branches

**Interviewer:** Analyze feature branches separately from main.

**Candidate:** SonarQube Developer Edition supports branch analysis. Each branch gets its own quality gate and metrics.

```groovy
// Jenkinsfile
stage('SonarQube Scan') {
    steps {
        script {
            def sonarScanner = tool 'SonarScanner'
            withSonarQubeEnv('SonarQube') {
                sh """
                    ${sonarScanner}/bin/sonar-scanner \
                      -Dsonar.projectKey=myapp \
                      -Dsonar.branch.name=${env.BRANCH_NAME} \
                      -Dsonar.sources=src
                """
            }
        }
    }
}

// For PRs, add:
// -Dsonar.pullrequest.key=${env.CHANGE_ID}
// -Dsonar.pullrequest.branch=${env.CHANGE_BRANCH}
// -Dsonar.pullrequest.base=${env.CHANGE_TARGET}
```

**Interviewer:** Community Edition doesn't support branches?

**Candidate:** Correct - only analyzes main branch. Feature branches overwrite each other. Need Developer Edition or higher for true branch analysis. Workaround: Create separate projects per branch (messy).

---

### Question 17: Git Reflog Recovery

**Interviewer:** You accidentally did `git reset --hard` and lost commits. Can you recover?

**Candidate:**
```bash
# View reflog - shows all HEAD movements
git reflog

# Output:
# abc1234 HEAD@{0}: reset: moving to HEAD~3
# def5678 HEAD@{1}: commit: Important feature
# ghi9012 HEAD@{2}: commit: Bug fix

# Recover lost commit
git checkout def5678
# Or create branch
git branch recovery-branch def5678

# If you know commit message
git log --all --grep="Important feature"
```

**Interviewer:** How long does reflog keep history?

**Candidate:** Default 90 days for reachable commits, 30 days for unreachable. After that, commits are garbage collected. Reflog is local - doesn't push to remote.

---

### Question 18: Terraform Data Sources vs Resources

**Interviewer:** Explain the difference and when to use each.

**Candidate:** **Resources** create/modify infrastructure. **Data sources** query existing infrastructure without modifying.

```hcl
# Data source - read existing VNet
data "azurerm_virtual_network" "existing" {
  name                = "existing-vnet"
  resource_group_name = "networking-rg"
}

# Use existing VNet's info
resource "azurerm_subnet" "new" {
  name                 = "new-subnet"
  resource_group_name  = data.azurerm_virtual_network.existing.resource_group_name
  virtual_network_name = data.azurerm_virtual_network.existing.name
  address_prefixes     = ["10.0.3.0/24"]
}

# Query Azure to get info
data "azurerm_client_config" "current" {}

output "tenant_id" {
  value = data.azurerm_client_config.current.tenant_id
}
```

**Interviewer:** Why not just import as resource?

**Candidate:** Data source is read-only - won't accidentally modify or delete. Use for infrastructure managed elsewhere that you need to reference. Import is for taking over management.

---

### Question 19: Jenkins Credentials Plugin Best Practices

**Interviewer:** You have multiple types of credentials in Jenkins. How do you organize them?

**Candidate:** Use folders with credential scope, consistent naming conventions, and credential domains.

```
Structure:
├── Global credentials
│   └── (Shared by all jobs)
├── Folder: Production
│   └── Credentials:
│       ├── prod-acr-credentials
│       ├── prod-k8s-token
│       └── prod-db-password
└── Folder: Development
    └── Credentials:
        ├── dev-acr-credentials
        └── dev-k8s-token

Naming: <env>-<service>-<type>
Examples:
- prod-azure-sp (service principal)
- dev-github-token (PAT)
- staging-docker-registry (username/password)
```

**Interviewer:** How do you rotate credentials?

**Candidate:** Update credential in Jenkins UI, no pipeline changes needed since they reference by ID. For secrets in Kubernetes, use external secrets operator to rotate without Jenkins involvement.

---

### Question 20: Ansible Dynamic Inventory

**Interviewer:** Your Azure VMs scale dynamically. How does Ansible discover them?

**Candidate:**
```yaml
# azure_rm.yml - Azure dynamic inventory plugin
plugin: azure_rm
auth_source: auto
include_vm_resource_groups:
  - production-rg
  - staging-rg
keyed_groups:
  - key: tags.environment
    prefix: env
  - key: tags.role
    prefix: role

# Use dynamic inventory
ansible-playbook -i azure_rm.yml playbook.yml

# Automatically creates groups:
# - env_production
# - env_staging
# - role_webserver
# - role_database

# Target specific group
ansible-playbook -i azure_rm.yml playbook.yml --limit env_production
```

**Interviewer:** Other cloud providers?

**Candidate:** Similar plugins for AWS (ec2), GCP (gcp_compute), and others. All discover instances based on tags/labels, create inventory groups automatically.

---

### Question 21: Docker Compose Override Files

**Interviewer:** Share base compose file across team but allow local customizations.

**Candidate:**
```yaml
# docker-compose.yml - Base (committed to Git)
version: '3.8'
services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: development

  database:
    image: postgres:14
    environment:
      POSTGRES_DB: myapp

# docker-compose.override.yml - Local customizations (gitignored)
version: '3.8'
services:
  web:
    volumes:
      - ./:/app  # Mount local code for hot reload
    ports:
      - "3001:3000"  # Different port if 3000 taken
  
  database:
    ports:
      - "5432:5432"  # Expose for local DB tools

# Docker Compose automatically merges both files
docker-compose up

# Or specify explicit override
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up
```

---

### Question 22: Maven Plugin Configuration

**Interviewer:** Configure Maven Surefire plugin to run tests in parallel.

**Candidate:**
```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-surefire-plugin</artifactId>
      <version>3.0.0</version>
      <configuration>
        <!-- Run test classes in parallel -->
        <parallel>classes</parallel>
        <threadCount>4</threadCount>
        <perCoreThreadCount>true</perCoreThreadCount>
        
        <!-- Fork JVM for each test class for isolation -->
        <forkCount>2</forkCount>
        <reuseForks>true</reuseForks>
        
        <!-- Include/exclude patterns -->
        <includes>
          <include>**/*Test.java</include>
          <include>**/*Tests.java</include>
        </includes>
        <excludes>
          <exclude>**/*IntegrationTest.java</exclude>
        </excludes>
        
        <!-- Fail fast or continue -->
        <skipAfterFailureCount>0</skipAfterFailureCount>  <!-- Run all -->
      </configuration>
    </plugin>
  </plugins>
</build>
```

**Interviewer:** How much faster is parallel execution?

**Candidate:** Depends on test suite, typically 50-70% time reduction with 4 threads. Diminishing returns beyond core count. Must ensure tests are thread-safe and don't share state.

---

### Question 23: GitHub Actions Reusable Workflows

**Interviewer:** Multiple repositories need the same build workflow. How do you share it?

**Candidate:**
```yaml
# .github/workflows/reusable-build.yml in shared-workflows repo
name: Reusable Build Workflow

on:
  workflow_call:
    inputs:
      node-version:
        required: true
        type: string
      runs-on:
        required: false
        type: string
        default: 'ubuntu-latest'
    secrets:
      npm-token:
        required: true
    outputs:
      artifact-name:
        value: ${{ jobs.build.outputs.artifact }}

jobs:
  build:
    runs-on: ${{ inputs.runs-on }}
    outputs:
      artifact: ${{ steps.upload.outputs.artifact-id }}
    steps:
    - uses: actions/checkout@v3
    - uses: actions/setup-node@v3
      with:
        node-version: ${{ inputs.node-version }}
    - run: npm ci
      env:
        NPM_TOKEN: ${{ secrets.npm-token }}
    - run: npm run build
    - uses: actions/upload-artifact@v3
      id: upload
      with:
        name: build-artifact
        path: dist/

---
# Consumer repo's workflow
name: CI
on: [push]

jobs:
  build:
    uses: org/shared-workflows/.github/workflows/reusable-build.yml@main
    with:
      node-version: '18'
      runs-on: 'ubuntu-latest'
    secrets:
      npm-token: ${{ secrets.NPM_TOKEN }}
```

**Interviewer:** Can you version these?

**Candidate:** Yes - reference specific tag or commit: `uses: org/shared-workflows/.github/workflows/reusable-build.yml@v1.2.0`. Test new versions without affecting all repos.

---

### Question 24: Linux Find Command Advanced Usage

**Interviewer:** Find all log files modified in last 24 hours larger than 100MB and compress them.

**Candidate:**
```bash
# Find and compress
find /var/log \
  -name "*.log" \
  -type f \
  -mtime -1 \
  -size +100M \
  -exec gzip {} \;

# Find with details
find /var/log \
  -name "*.log" \
  -type f \
  -mtime -1 \
  -size +100M \
  -exec ls -lh {} \;

# Find and move to archive
find /var/log \
  -name "*.log" \
  -type f \
  -mtime +30 \
  -exec mv {} /archive/ \;

# Find and delete empty files
find /tmp -type f -empty -delete

# Find and change ownership
find /app \
  -type f \
  -user root \
  -exec chown appuser:appuser {} \;

# Complex: Find and process with script
find /data \
  -name "*.csv" \
  -mtime -7 \
  -size +50M \
  -exec sh -c 'echo "Processing: $1"; python process.py "$1"' _ {} \;
```

---

### Question 25: Python Script for Kubernetes Resource Report

**Interviewer:** Generate report of pod resource usage vs limits.

**Candidate:**
```python
#!/usr/bin/env python3
from kubernetes import client, config
from tabulate import tabulate

def get_pod_resources(namespace='default'):
    """Get pod resource usage and limits"""
    config.load_kube_config()
    v1 = client.CoreV1Api()
    
    pods = v1.list_namespaced_pod(namespace)
    metrics_api = client.CustomObjectsApi()
    
    try:
        pod_metrics = metrics_api.list_namespaced_custom_object(
            "metrics.k8s.io", "v1beta1", namespace, "pods"
        )
    except:
        print("Metrics server not available")
        return
    
    # Build metrics dict
    metrics_dict = {}
    for item in pod_metrics['items']:
        pod_name = item['metadata']['name']
        for container in item['containers']:
            key = f"{pod_name}/{container['name']}"
            metrics_dict[key] = {
                'cpu': container['usage']['cpu'],
                'memory': container['usage']['memory']
            }
    
    # Collect data
    data = []
    for pod in pods.items:
        for container in pod.spec.containers:
            key = f"{pod.metadata.name}/{container.name}"
            
            # Get requests/limits
            requests = container.resources.requests or {}
            limits = container.resources.limits or {}
            
            # Get current usage
            usage = metrics_dict.get(key, {})
            
            data.append([
                pod.metadata.name,
                container.name,
                usage.get('cpu', 'N/A'),
                requests.get('cpu', 'None'),
                limits.get('cpu', 'None'),
                usage.get('memory', 'N/A'),
                requests.get('memory', 'None'),
                limits.get('memory', 'None')
            ])
    
    headers = ['Pod', 'Container', 'CPU Usage', 'CPU Request', 'CPU Limit',
               'Mem Usage', 'Mem Request', 'Mem Limit']
    print(tabulate(data, headers=headers, tablefmt='grid'))

if __name__ == '__main__':
    import sys
    namespace = sys.argv[1] if len(sys.argv) > 1 else 'default'
    get_pod_resources(namespace)
```

---

### Question 26: Prometheus Alertmanager Configuration

**Interviewer:** Route alerts based on severity and team ownership.

**Candidate:**
```yaml
# alertmanager.yml
global:
  slack_api_url: 'https://hooks.slack.com/services/xxx'
  pagerduty_url: 'https://events.pagerduty.com/v2/enqueue'

route:
  receiver: 'default'
  group_by: ['alertname', 'cluster', 'service']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  
  routes:
  # Critical alerts to PagerDuty
  - match:
      severity: critical
    receiver: pagerduty-critical
    continue: true  # Also send to Slack
  
  # Team-specific routing
  - match:
      team: frontend
    receiver: slack-frontend-team
  
  - match:
      team: backend
    receiver: slack-backend-team
  
  # Database alerts
  - match_re:
      service: .*database.*
    receiver: slack-database-team
    group_wait: 10s  # Faster grouping for DB

receivers:
- name: 'default'
  slack_configs:
  - channel: '#alerts'
    title: 'Alert: {{ .GroupLabels.alertname }}'
    text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'

- name: 'pagerduty-critical'
  pagerduty_configs:
  - service_key: 'your-service-key'
    description: '{{ .GroupLabels.alertname }}'

- name: 'slack-frontend-team'
  slack_configs:
  - channel: '#frontend-alerts'

- name: 'slack-backend-team'
  slack_configs:
  - channel: '#backend-alerts'

inhibit_rules:
# Inhibit warning if critical is firing
- source_match:
    severity: 'critical'
  target_match:
    severity: 'warning'
  equal: ['alertname', 'cluster', 'service']
```

---

### Question 27: Helm Chart Testing

**Interviewer:** How do you test Helm charts before deploying?

**Candidate:**
```bash
# Lint chart for errors
helm lint ./mychart

# Template and review output
helm template mychart ./mychart --values values-dev.yaml

# Dry-run install
helm install mychart ./mychart \
  --dry-run \
  --debug \
  --values values-dev.yaml

# Install to test namespace
helm install mychart ./mychart \
  --namespace test \
  --create-namespace \
  --values values-dev.yaml

# Run chart tests
helm test mychart -n test

# Chart tests defined as:
# templates/tests/test-connection.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ .Release.Name }}-test-connection"
  annotations:
    "helm.sh/hook": test
spec:
  containers:
  - name: wget
    image: busybox
    command: ['wget']
    args: ['{{ .Release.Name }}:{{ .Values.service.port }}']
  restartPolicy: Never
```

---

### Question 28: ArgoCD Sync Waves

**Interviewer:** Ensure database deploys before application during ArgoCD sync.

**Candidate:**
```yaml
# Database StatefulSet - deploys first
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
  annotations:
    argocd.argoproj.io/sync-wave: "0"  # Wave 0
spec:
  # ... postgres config

---
# Database service
apiVersion: v1
kind: Service
metadata:
  name: postgres
  annotations:
    argocd.argoproj.io/sync-wave: "0"
spec:
  # ... service config

---
# Migration job - runs after database ready
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
  annotations:
    argocd.argoproj.io/sync-wave: "1"  # Wave 1
    argocd.argoproj.io/hook: Sync
spec:
  # ... migration job

---
# Application deployment - after migration
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  annotations:
    argocd.argoproj.io/sync-wave: "2"  # Wave 2
spec:
  # ... backend config
```

**Interviewer:** What if a wave fails?

**Candidate:** ArgoCD stops syncing. Wave 2 won't deploy if wave 1 fails. Fix the issue, resync. Ensures dependencies are respected.

---

### Question 29: Maven BOM (Bill of Materials)

**Interviewer:** Manage dependency versions centrally across multiple projects.

**Candidate:**
```xml
<!-- company-bom/pom.xml - BOM project -->
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.company</groupId>
  <artifactId>company-bom</artifactId>
  <version>1.0.0</version>
  <packaging>pom</packaging>
  
  <properties>
    <spring.boot.version>3.1.0</spring.boot.version>
    <jackson.version>2.15.0</jackson.version>
  </properties>
  
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-dependencies</artifactId>
        <version>${spring.boot.version}</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
      <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>${jackson.version}</version>
      </dependency>
    </dependencies>
  </dependencyManagement>
</project>

<!-- Consumer project -->
<project>
  <dependencyManagement>
    <dependencies>
      <!-- Import BOM -->
      <dependency>
        <groupId>com.company</groupId>
        <artifactId>company-bom</artifactId>
        <version>1.0.0</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>
  
  <dependencies>
    <!-- No version needed - comes from BOM -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
    </dependency>
  </dependencies>
</project>
```

**Interviewer:** Benefits?

**Candidate:** Single version update in BOM updates all projects. Prevents version conflicts across microservices. Tested compatible version combinations.

---

### Question 30: SonarQube Quality Profiles

**Interviewer:** Customize rules for your organization's standards.

**Candidate:** Create custom quality profile based on built-in profile, activate/deactivate rules, adjust severities.

```
SonarQube → Quality Profiles → Create
Base on: Sonar way (Java)
Name: Company Java Standards

Modifications:
- Activate rule: No hardcoded IP addresses (Major)
- Deactivate: Maximum line length (too strict for us)
- Change severity: Commented code → Major (we want this fixed)
- Add parameters: Cyclomatic complexity → Max: 15 (from 10)

Set as default for all new projects
Or assign to specific projects
```

**Interviewer:** Can you export/import profiles?

**Candidate:** Yes - export as XML, commit to Git, import in other SonarQube instances. Ensures consistent quality across dev/staging/prod SonarQube servers.

---

### Question 31: Git Worktree for Parallel Development

**Interviewer:** You need to work on two features simultaneously without stashing.

**Candidate:**
```bash
# Main repo in ~/project
cd ~/project

# Add worktree for feature-a
git worktree add ~/project-feature-a feature/feature-a

# Add worktree for hotfix
git worktree add ~/project-hotfix hotfix/critical-bug

# Now you have three working directories:
# ~/project           (main branch)
# ~/project-feature-a (feature/feature-a branch)
# ~/project-hotfix    (hotfix/critical-bug branch)

# Work in each independently
cd ~/project-feature-a
npm install && npm start  # Terminal 1

cd ~/project-hotfix
npm install && npm start  # Terminal 2

# List worktrees
git worktree list

# Remove worktree when done
git worktree remove ~/project-feature-a
```

---

### Question 32: Docker Healthcheck

**Interviewer:** Add healthchecks to Docker containers.

**Candidate:**
```dockerfile
FROM node:18-alpine

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .

# Add healthcheck
HEALTHCHECK --interval=30s \
            --timeout=3s \
            --start-period=40s \
            --retries=3 \
  CMD node healthcheck.js || exit 1

EXPOSE 3000
CMD ["node", "server.js"]

# healthcheck.js
const http = require('http');
const options = {
  host: 'localhost',
  port: 3000,
  path: '/health',
  timeout: 2000
};

http.get(options, (res) => {
  process.exit(res.statusCode === 200 ? 0 : 1);
}).on('error', () => {
  process.exit(1);
});
```

**Interviewer:** How does Docker use this?

**Candidate:** Container status shows "healthy" or "unhealthy". Docker Compose can wait for dependencies to be healthy. In orchestrators, unhealthy containers get replaced automatically.

---

### Question 33: Terraform Module Versioning

**Interviewer:** Manage Terraform module versions across environments.

**Candidate:**
```hcl
# Development - use latest
module "network" {
  source = "git::https://github.com/company/terraform-modules.git//network?ref=main"
  # ...
}

# Staging - use specific version for testing
module "network" {
  source = "git::https://github.com/company/terraform-modules.git//network?ref=v2.1.0"
  # ...
}

# Production - use tested stable version
module "network" {
  source = "git::https://github.com/company/terraform-modules.git//network?ref=v2.0.5"
  # ...
}

# Or use Terraform Registry
module "network" {
  source  = "company/network/azurerm"
  version = "~> 2.0"  # Any 2.x version
  # ...
}
```

**Interviewer:** Version constraint syntax?

**Candidate:** `~>` allows rightmost digit to increment. `~> 2.0` means `>= 2.0` and `< 3.0`. Use pessimistic constraints to get bug fixes but not breaking changes.

---

### Question 34: Jenkins Pipeline Notification

**Interviewer:** Send customized notifications for pipeline status.

**Candidate:**
```groovy
def notifySlack(String buildStatus = 'STARTED') {
    def color = 'warning'
    def message = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
    
    if (buildStatus == 'SUCCESS') {
        color = 'good'
    } else if (buildStatus == 'FAILURE') {
        color = 'danger'
    }
    
    slackSend(
        color: color,
        message: message,
        channel: '#ci-cd',
        tokenCredentialId: 'slack-token'
    )
}

pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                notifySlack('STARTED')
                sh 'npm run build'
            }
        }
    }
    
    post {
        success {
            notifySlack('SUCCESS')
        }
        failure {
            notifySlack('FAILURE')
            emailext(
                subject: "Build Failed: ${env.JOB_NAME}",
                body: """Build ${env.BUILD_NUMBER} failed.
                         Check console output at ${env.BUILD_URL}""",
                to: '${DEFAULT_RECIPIENTS}'
            )
        }
    }
}
```

---

### Question 35: Python Slack Notification Script

**Interviewer:** Send deployment notifications to Slack from CI/CD.

**Candidate:**
```python
#!/usr/bin/env python3
import requests
import sys
import os

def send_slack_notification(webhook_url, status, environment, version):
    """Send deployment notification to Slack"""
    
    color = {
        'success': '#36a64f',
        'failure': '#ff0000',
        'warning': '#ffcc00'
    }.get(status.lower(), '#808080')
    
    emoji = {
        'success': ':white_check_mark:',
        'failure': ':x:',
        'warning': ':warning:'
    }.get(status.lower(), ':information_source:')
    
    payload = {
        'attachments': [{
            'color': color,
            'title': f'{emoji} Deployment {status.title()}',
            'fields': [
                {'title': 'Environment', 'value': environment, 'short': True},
                {'title': 'Version', 'value': version, 'short': True},
                {'title': 'Deployed By', 'value': os.getenv('USER', 'CI/CD'), 'short': True},
                {'title': 'Build', 'value': os.getenv('BUILD_NUMBER', 'N/A'), 'short': True}
            ],
            'footer': 'Deployment System',
            'ts': int(__import__('time').time())
        }]
    }
    
    response = requests.post(webhook_url, json=payload)
    return response.status_code == 200

if __name__ == '__main__':
    if len(sys.argv) != 4:
        print("Usage: notify_slack.py <status> <environment> <version>")
        sys.exit(1)
    
    webhook_url = os.getenv('SLACK_WEBHOOK_URL')
    if not webhook_url:
        print("SLACK_WEBHOOK_URL not set")
        sys.exit(1)
    
    status, environment, version = sys.argv[1:4]
    
    if send_slack_notification(webhook_url, status, environment, version):
        print("Notification sent successfully")
    else:
        print("Failed to send notification")
        sys.exit(1)

# Usage in CI/CD:
# python notify_slack.py success production v1.2.3
```

---

### Question 36: Ansible Handlers

**Interviewer:** Restart service only when configuration changes.

**Candidate:**
```yaml
---
- name: Configure Nginx
  hosts: webservers
  become: yes
  
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
    
    - name: Copy main config
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Reload Nginx  # Triggers handler only if file changed
    
    - name: Copy site config
      template:
        src: site.conf.j2
        dest: /etc/nginx/sites-available/mysite
      notify: Reload Nginx
    
    - name: Enable site
      file:
        src: /etc/nginx/sites-available/mysite
        dest: /etc/nginx/sites-enabled/mysite
        state: link
      notify: Reload Nginx
    
    - name: Ensure Nginx is running
      service:
        name: nginx
        state: started
        enabled: yes
  
  handlers:
    - name: Reload Nginx
      service:
        name: nginx
        state: reloaded
    
    # Different handler for restart
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```

**Interviewer:** When do handlers run?

**Candidate:** End of play, after all tasks. Multiple tasks can notify same handler, but it runs only once. If task doesn't change anything (idempotent), handler doesn't trigger.

---

### Question 37: Linux Systemctl Timer (Cron Alternative)

**Interviewer:** Schedule tasks using systemd instead of cron.

**Candidate:**
```bash
# /etc/systemd/system/backup.service
[Unit]
Description=Backup Service
[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh
User=backup
StandardOutput=journal

# /etc/systemd/system/backup.timer
[Unit]
Description=Backup Timer
[Timer]
OnCalendar=daily
OnCalendar=02:00
Persistent=true
[Install]
WantedBy=timers.target

# Enable and start timer
sudo systemctl daemon-reload
sudo systemctl enable backup.timer
sudo systemctl start backup.timer

# Check timer status
sudo systemctl list-timers
sudo systemctl status backup.timer

# View logs
sudo journalctl -u backup.service
```

**Interviewer:** Advantages over cron?

**Candidate:** Better logging with journald, dependency management, can trigger on events beyond time, service and timer separation is cleaner, easier to see status with systemctl.

---

### Question 38: Grafana Alerting

**Interviewer:** Configure Grafana alerts for pod memory usage.

**Candidate:**
```
Dashboard Panel → Alert tab → Create Alert

Alert Rule:
Name: High Memory Usage
Evaluate every: 1m
For: 5m

Conditions:
WHEN avg() OF query(A, 5m, now) IS ABOVE 85

Query A:
(container_memory_usage_bytes{namespace="production"} 
  / 
 container_spec_memory_limit_bytes{namespace="production"}) * 100

Notifications:
Send to: Slack, Email, PagerDuty
Message: Pod {{ $labels.pod }} memory usage is {{ $value }}%

State: OK | Pending | Alerting | No Data

# Notification channels in Configuration
Name: Slack - Production Alerts
Type: Slack
Webhook URL: https://hooks.slack.com/services/xxx
Username: Grafana Alerts
Channel: #production-alerts
```

---

### Question 39: GitHub Actions Composite Actions

**Interviewer:** Create reusable action for common setup steps.

**Candidate:**
```yaml
# .github/actions/setup-node-app/action.yml
name: 'Setup Node App'
description: 'Setup Node.js and install dependencies with caching'

inputs:
  node-version:
    description: 'Node.js version'
    required: true
  cache-dependency-path:
    description: 'Path to package-lock.json'
    required: false
    default: '**/package-lock.json'

runs:
  using: "composite"
  steps:
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: ${{ inputs.node-version }}
        cache: 'npm'
        cache-dependency-path: ${{ inputs.cache-dependency-path }}
    
    - name: Install dependencies
      shell: bash
      run: npm ci
    
    - name: Cache node_modules
      uses: actions/cache@v3
      with:
        path: node_modules
        key: ${{ runner.os }}-node-${{ inputs.node-version }}-${{ hashFiles('**/package-lock.json') }}

---
# Usage in workflow
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup
      uses: ./.github/actions/setup-node-app
      with:
        node-version: '18'
    
    - run: npm test
    - run: npm run build
```

---

### Question 40: Docker Registry with Authentication

**Interviewer:** Run private Docker registry with basic auth.

**Candidate:**
```bash
# Create password file
mkdir -p registry/auth
docker run --rm \
  --entrypoint htpasswd \
  httpd:2 -Bbn testuser testpassword > registry/auth/htpasswd

# Create registry with TLS
mkdir -p registry/certs
# Copy SSL cert and key to registry/certs/

# Run registry
docker run -d \
  -p 5000:5000 \
  --name registry \
  --restart=always \
  -v $(pwd)/registry/auth:/auth \
  -v $(pwd)/registry/certs:/certs \
  -v registry-data:/var/lib/registry \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e "REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd" \
  -e "REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt" \
  -e "REGISTRY_HTTP_TLS_KEY=/certs/domain.key" \
  registry:2

# Login and use
docker login myregistry.company.com:5000
docker tag myapp:1.0 myregistry.company.com:5000/myapp:1.0
docker push myregistry.company.com:5000/myapp:1.0
```

---

### Question 41-50: Quick Fire Round

**Q41:** Maven skip tests during package?
**A:** `mvn package -DskipTests` or `-Dmaven.test.skip=true`

**Q42:** Git undo last commit but keep changes?
**A:** `git reset --soft HEAD~1`

**Q43:** SonarQube minimum coverage for quality gate?
**A:** Typically 80% coverage on new code, 70% overall. Configured per project in Quality Gates.

**Q44:** Docker remove all stopped containers?
**A:** `docker container prune` or `docker rm $(docker ps -aq -f status=exited)`

**Q45:** Terraform validate syntax?
**A:** `terraform validate` checks syntax, `terraform plan` checks if changes are valid

**Q46:** Ansible check mode (dry run)?
**A:** `ansible-playbook playbook.yml --check` simulates without making changes

**Q47:** Linux check disk inode usage?
**A:** `df -i` shows inode usage per filesystem

**Q48:** Prometheus metric types?
**A:** Counter (always increasing), Gauge (can go up/down), Histogram (distributions), Summary (quantiles)

**Q49:** Git show changes in last commit?
**A:** `git show HEAD` or `git diff HEAD~1`

**Q50:** Jenkins trigger build remotely?
**A:** Enable "Trigger builds remotely" in job config, use token: `curl http://jenkins/job/myjob/build?token=TOKEN`

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

---

**✅ Interview Session 5 Complete - Focus on Native DevOps Tools!**

**Tools Covered:** Git/GitHub, Maven, SonarQube, Docker, Jenkins, Ansible, Linux, Python, Terraform, Helm, ArgoCD, Prometheus/Grafana, GitHub Actions

**Total Questions: 50/50** | **Format:** Tool-Specific Deep Dive

