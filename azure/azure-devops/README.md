# Azure DevOps - Complete Tutorial Guide

**Comprehensive Guide to Azure DevOps for CI/CD, Source Control, and Project Management**

---

## 📚 Table of Contents

1. [What is Azure DevOps?](#what-is-azure-devops)
2. [Azure DevOps Services](#services)
3. [Setup & Configuration](#setup-configuration)
4. [Service Principals](#service-principals)
5. [Azure CLI & Tools](#azure-cli-tools)
6. [Agent Pools](#agent-pools)
7. [Azure Repos](#azure-repos)
8. [Azure Pipelines](#azure-pipelines)
9. [Pipeline Triggers](#pipeline-triggers)
10. [Azure Boards](#azure-boards)
11. [Azure Artifacts](#azure-artifacts)
12. [Service Connections](#service-connections)
13. [Variable Groups & Secrets](#variables-secrets)
14. [Environments](#environments)
15. [Permissions & Security](#permissions-security)
16. [Extensions & Marketplace](#extensions-marketplace)
17. [Personal Access Tokens](#personal-access-tokens)
18. [Integration with AKS](#aks-integration)
19. [Best Practices](#best-practices)
20. [Troubleshooting](#troubleshooting)

---

## 🎯 What is Azure DevOps?

**Azure DevOps** is a comprehensive suite of development tools for software teams to plan work, collaborate on code development, and build and deploy applications.

### **Why Azure DevOps?**

```
Complete DevOps Lifecycle Platform:

Plan → Code → Build → Test → Release → Monitor

Azure DevOps provides:
✓ Source control (Azure Repos)
✓ CI/CD pipelines (Azure Pipelines)
✓ Work tracking (Azure Boards)
✓ Package management (Azure Artifacts)
✓ Test management (Azure Test Plans)
✓ Integration with Azure services
✓ Integration with third-party tools
```

---

### **Azure DevOps vs GitHub:**

```
Azure DevOps:
✓ Complete ALM solution
✓ Built-in boards and tracking
✓ Enterprise features
✓ Tight Azure integration
✓ Better for enterprise teams
✓ More granular permissions

GitHub:
✓ Largest developer community
✓ Open source friendly
✓ GitHub Actions (CI/CD)
✓ Better for open source
✓ Simpler interface
✓ More integrations

Both owned by Microsoft!
Can use together (GitHub + Azure Pipelines)
```

---

## 🔧 Azure DevOps Services

Azure DevOps consists of 5 main services.

### **Service Overview:**

```
┌─────────────────────────────────────────┐
│  Azure DevOps Organization              │
│  └── Project                            │
│      ├── Azure Repos (Git/TFVC)        │
│      ├── Azure Pipelines (CI/CD)       │
│      ├── Azure Boards (Agile)          │
│      ├── Azure Test Plans              │
│      └── Azure Artifacts (Packages)    │
└─────────────────────────────────────────┘
```

---

### **Creating Organization & Project:**

**Step 1: Create Organization**
```
1. Go to: https://dev.azure.com
2. Sign in with Microsoft account
3. Click "Create new organization"
4. Name: mycompany-devops
5. Region: Select closest
```

**Step 2: Create Project**
```
1. Click "+ Create Project"
2. Project name: MyApplication
3. Visibility: Private (or Public)
4. Version control: Git
5. Work item process: Agile
6. Click "Create"
```

---

## 🛠️ Setup & Configuration

Complete setup for Azure DevOps.

### **Prerequisites:**

```
Required:
✓ Azure subscription (for deployments)
✓ Microsoft account or Azure AD account
✓ Git installed locally
✓ Azure CLI installed

Optional but recommended:
✓ Azure DevOps CLI extension
✓ Visual Studio Code
✓ Docker (for containerized builds)
✓ kubectl (for Kubernetes deployments)
```

---

### **Initial Setup Steps:**

**1. Azure Subscription Setup:**
```bash
# Login to Azure
az login

# List subscriptions
az account list --output table

# Set default subscription
az account set --subscription "My Subscription"

# Verify
az account show
```

**2. Register Resource Providers:**
```bash
# Required for Azure DevOps integration
az provider register --namespace Microsoft.ContainerRegistry
az provider register --namespace Microsoft.ContainerService
az provider register --namespace Microsoft.KeyVault

# Verify registration
az provider show --namespace Microsoft.ContainerRegistry --query "registrationState"
```

**3. Create Resource Group (for Azure resources):**
```bash
az group create \
  --name myapp-resources \
  --location eastus
```

---

## 🔐 Service Principals

Service Principals are identities used for authentication between Azure DevOps and Azure services.

### **What is a Service Principal?**

```
Service Principal = Application Identity in Azure AD

Like a service account but for applications:
- Has its own credentials
- Can be granted specific permissions
- Used for automation (CI/CD)
- No human user involved

Use cases in Azure DevOps:
✓ Deploy to Azure services
✓ Access Azure Key Vault
✓ Manage Azure resources
✓ Pull/push to Azure Container Registry
```

---

### **Creating Service Principal:**

**Method 1: Azure Portal (Manual)**

```
1. Azure Portal → Azure Active Directory
2. App registrations → New registration
3. Name: azure-devops-sp
4. Supported account types: Single tenant
5. Click "Register"

6. Copy these values:
   - Application (client) ID
   - Directory (tenant) ID

7. Certificates & secrets → New client secret
8. Description: azure-devops-key
9. Expires: 24 months
10. Copy secret VALUE (only shown once!)

11. Grant permissions:
    - Go to Subscription → Access control (IAM)
    - Add role assignment
    - Role: Contributor (or specific role)
    - Assign access to: Service Principal
    - Select: azure-devops-sp
    - Save
```

**Method 2: Azure CLI (Automated)**

```bash
# Create service principal with Contributor role
az ad sp create-for-rbac \
  --name azure-devops-sp \
  --role Contributor \
  --scopes /subscriptions/{subscription-id}

# Output:
{
  "appId": "12345678-1234-1234-1234-123456789abc",
  "displayName": "azure-devops-sp",
  "password": "super-secret-password",
  "tenant": "87654321-4321-4321-4321-abcdef123456"
}

# Save these values securely!
```

**Method 3: With Specific Resource Group Scope:**

```bash
# Limit permissions to specific resource group
az ad sp create-for-rbac \
  --name azure-devops-sp-dev \
  --role Contributor \
  --scopes /subscriptions/{subscription-id}/resourceGroups/myapp-dev

# For production (separate SP)
az ad sp create-for-rbac \
  --name azure-devops-sp-prod \
  --role Contributor \
  --scopes /subscriptions/{subscription-id}/resourceGroups/myapp-prod
```

---

### **Service Principal Permissions:**

**Common Roles:**

```
Reader:
- View resources
- Cannot make changes
- Use for: Monitoring, reporting

Contributor:
- Full access to resources
- Cannot manage access/permissions
- Use for: Deployments, resource management

Owner:
- Full access including access management
- Use for: Rare cases, usually too permissive

Custom Roles:
- Specific permissions only
- Use for: Principle of least privilege
```

**Best Practice - Least Privilege:**

```bash
# Create custom role for AKS deployment only
az role definition create --role-definition '{
  "Name": "AKS Deployer",
  "Description": "Can deploy to AKS only",
  "Actions": [
    "Microsoft.ContainerService/managedClusters/read",
    "Microsoft.ContainerService/managedClusters/write"
  ],
  "AssignableScopes": [
    "/subscriptions/{subscription-id}/resourceGroups/myapp-prod"
  ]
}'

# Assign custom role to service principal
az role assignment create \
  --assignee {app-id} \
  --role "AKS Deployer" \
  --scope /subscriptions/{subscription-id}/resourceGroups/myapp-prod
```

---

### **Using Service Principal in Azure DevOps:**

**Create Service Connection with Service Principal:**

```
1. Project Settings → Service connections
2. New service connection → Azure Resource Manager
3. Authentication method: Service principal (manual)
4. Subscription ID: {your-subscription-id}
5. Subscription Name: My Subscription
6. Service Principal ID: {appId}
7. Service Principal Key: {password}
8. Tenant ID: {tenant}
9. Service connection name: azure-prod-sp
10. Verify and save

Test connection:
- Click "Verify"
- Should show: "Verification succeeded"
```

---

### **Managing Service Principals:**

```bash
# List service principals
az ad sp list --display-name azure-devops-sp

# View service principal details
az ad sp show --id {appId}

# List role assignments
az role assignment list --assignee {appId} --output table

# Add role to service principal
az role assignment create \
  --assignee {appId} \
  --role "AcrPush" \
  --scope /subscriptions/{subscription-id}/resourceGroups/myapp/providers/Microsoft.ContainerRegistry/registries/myacr

# Delete role assignment
az role assignment delete \
  --assignee {appId} \
  --role Contributor \
  --scope /subscriptions/{subscription-id}

# Reset credentials (rotate secret)
az ad sp credential reset --id {appId}

# Delete service principal
az ad sp delete --id {appId}
```

---

### **Security Best Practices:**

```
✓ Use separate service principals for each environment
  - dev-sp (dev resources only)
  - staging-sp (staging resources only)
  - prod-sp (prod resources only)

✓ Rotate credentials regularly (every 90 days)
✓ Use shortest expiration time needed
✓ Grant minimum permissions required
✓ Monitor service principal activity
✓ Use Azure Key Vault to store credentials
✗ Never hardcode credentials in code
✗ Never share service principals across teams
```

---

## 💻 Azure CLI & Tools

Essential tools for Azure DevOps.

### **Azure CLI Installation:**

**Windows:**
```powershell
# Using MSI installer
Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi
Start-Process msiexec.exe -ArgumentList '/I AzureCLI.msi /quiet'

# Or using winget
winget install -e --id Microsoft.AzureCLI
```

**macOS:**
```bash
brew update && brew install azure-cli
```

**Linux:**
```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

**Verify installation:**
```bash
az --version

# Output:
azure-cli                         2.55.0
...
```

---

### **Azure DevOps CLI Extension:**

```bash
# Install Azure DevOps extension
az extension add --name azure-devops

# Verify
az extension list --output table

# Update extension
az extension update --name azure-devops

# Configure defaults
az devops configure --defaults \
  organization=https://dev.azure.com/myorg \
  project=MyProject
```

---

### **Azure DevOps CLI Examples:**

```bash
# Login
az login

# Set organization and project defaults
az devops configure --defaults \
  organization=https://dev.azure.com/myorg \
  project=MyApp

# Create repository
az repos create --name myrepo

# List repositories
az repos list --output table

# Create pipeline
az pipelines create \
  --name my-pipeline \
  --description 'CI/CD pipeline' \
  --repository myrepo \
  --branch main \
  --yml-path azure-pipelines.yml

# Run pipeline
az pipelines run --name my-pipeline

# List pipeline runs
az pipelines runs list --pipeline-name my-pipeline --output table

# Create work item
az boards work-item create \
  --title "Add login feature" \
  --type "User Story" \
  --assigned-to "user@example.com"

# List work items
az boards query --wiql "SELECT [System.Id], [System.Title] FROM WorkItems WHERE [System.State] = 'Active'"
```

---

### **Git Configuration for Azure Repos:**

```bash
# Configure Git user
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Configure credential helper (Windows)
git config --global credential.helper manager

# Configure credential helper (macOS)
git config --global credential.helper osxkeychain

# Configure credential helper (Linux)
git config --global credential.helper store

# Clone Azure Repos repository
git clone https://dev.azure.com/myorg/MyProject/_git/myrepo

# Or with PAT:
git clone https://{PAT}@dev.azure.com/myorg/MyProject/_git/myrepo
```

---

### **Visual Studio Code Extensions:**

```
Recommended VS Code Extensions:

1. Azure Pipelines
   - Syntax highlighting for YAML pipelines
   - IntelliSense for tasks
   - Pipeline validation

2. Azure Repos
   - Pull request creation/review
   - Work item integration

3. Azure DevOps Extension
   - Work items in VS Code
   - Pipeline management

4. YAML
   - YAML language support
   - Schema validation

5. GitLens
   - Git blame annotations
   - Commit history

Install via VS Code:
Ctrl+Shift+X → Search → Install
```

---

## 🖥️ Agent Pools

Agents run your build and deployment jobs.

### **What are Agent Pools?**

```
Agent = Machine that executes pipeline jobs

Types:
1. Microsoft-hosted agents (Cloud)
   - Managed by Microsoft
   - Fresh VM for each job
   - Pre-installed software
   - Free tier: 1800 minutes/month

2. Self-hosted agents (Your infrastructure)
   - You manage
   - Persistent machines
   - Custom software
   - Unlimited minutes
```

---

### **Microsoft-Hosted Agents:**

**Available Images:**

```yaml
# Ubuntu (Latest)
pool:
  vmImage: 'ubuntu-latest'  # Most common

# Windows
pool:
  vmImage: 'windows-latest'

# macOS (for iOS builds)
pool:
  vmImage: 'macos-latest'

# Specific versions
pool:
  vmImage: 'ubuntu-22.04'
  
pool:
  vmImage: 'windows-2022'
```

**Pre-installed Software:**

```
Ubuntu agents include:
- Docker
- Git
- Node.js (multiple versions)
- Python (multiple versions)
- .NET SDK
- Azure CLI
- kubectl
- Helm
- Terraform
- And much more...

Full list:
https://github.com/actions/runner-images
```

---

### **Self-Hosted Agents:**

**Why Self-Hosted?**

```
Use self-hosted when:
✓ Need specific software/tools
✓ Access to internal resources
✓ Corporate firewall restrictions
✓ Unlimited build minutes needed
✓ Faster builds (persistent cache)
✓ GPU requirements
✓ Specific OS/hardware needed
```

---

### **Setting Up Self-Hosted Agent (Linux):**

```bash
# 1. Create agent pool in Azure DevOps
# Organization Settings → Agent pools → Add pool
# Name: Self-Hosted-Linux
# Grant access to all pipelines

# 2. Download agent on your machine
cd ~
mkdir myagent && cd myagent
wget https://vstsagentpackage.azureedge.net/agent/3.227.2/vsts-agent-linux-x64-3.227.2.tar.gz
tar zxvf vsts-agent-linux-x64-3.227.2.tar.gz

# 3. Configure agent
./config.sh

# Prompts:
# Server URL: https://dev.azure.com/myorg
# Authentication: PAT (Personal Access Token)
# Agent pool: Self-Hosted-Linux
# Agent name: my-linux-agent
# Work folder: _work

# 4. Install dependencies
sudo ./bin/installdependencies.sh

# 5. Run agent
./run.sh

# Or install as service (recommended)
sudo ./svc.sh install
sudo ./svc.sh start

# Verify agent is online
# Azure DevOps → Project Settings → Agent pools → Self-Hosted-Linux
# Should see agent with green status
```

---

### **Setting Up Self-Hosted Agent (Windows):**

```powershell
# 1. Download agent
cd C:\
mkdir agent ; cd agent
Invoke-WebRequest -Uri https://vstsagentpackage.azureedge.net/agent/3.227.2/vsts-agent-win-x64-3.227.2.zip -OutFile agent.zip
Expand-Archive -Path agent.zip -DestinationPath .

# 2. Configure
.\config.cmd

# 3. Install as Windows service
.\config.cmd --unattended \
  --url https://dev.azure.com/myorg \
  --auth PAT \
  --token {YOUR_PAT} \
  --pool Self-Hosted-Windows \
  --agent my-windows-agent \
  --runAsService
```

---

### **Using Agent Pools in Pipeline:**

```yaml
# Use Microsoft-hosted agent
pool:
  vmImage: 'ubuntu-latest'

# Use self-hosted pool
pool:
  name: 'Self-Hosted-Linux'

# Use specific agent by name
pool:
  name: 'Self-Hosted-Linux'
  demands:
  - agent.name -equals my-linux-agent

# Use agent with specific capabilities
pool:
  name: 'Self-Hosted-Linux'
  demands:
  - docker
  - kubectl
```

---

### **Agent Capabilities:**

```bash
# View agent capabilities
# Azure DevOps → Agent pools → Self-Hosted-Linux → Agents → my-linux-agent → Capabilities

# System capabilities (automatic):
- Agent.OS = Linux
- Agent.OSArchitecture = X64
- docker = true
- kubectl = true

# User capabilities (manual):
Add custom capabilities for specific software/tools

# Set capability via config
./config.sh --addCapability "customTool:true"
```

---

### **Agent Maintenance:**

```bash
# Update agent
cd ~/myagent
./config.sh remove  # Unregister
# Download new version
# Run ./config.sh again

# View agent logs
tail -f ~/myagent/_diag/Agent_*.log

# Stop agent service (Linux)
sudo ./svc.sh stop

# Start agent service
sudo ./svc.sh start

# Uninstall agent
./config.sh remove
sudo ./svc.sh uninstall
```

---

## 📦 Azure Repos

Source control management with Git or TFVC.

### **What is Azure Repos?**

```
Azure Repos = Git Hosting + Advanced Features

Features:
✓ Unlimited private Git repositories
✓ Pull requests with code reviews
✓ Branch policies
✓ Integration with Azure Pipelines
✓ Web-based code editor
✓ File annotations and history
✓ Branch comparison
```

---

### **Creating Repository:**

```bash
# Via Azure DevOps UI:
1. Go to Repos → Files
2. Click dropdown → "New repository"
3. Name: myapp
4. Add README: ✓
5. Add .gitignore: Node, Python, etc.
6. Click "Create"

# Clone to local:
git clone https://dev.azure.com/myorg/MyProject/_git/myapp
cd myapp

# Make changes
echo "# My App" >> README.md
git add .
git commit -m "Update README"
git push origin main
```

---

### **Branch Policies:**

**Why Branch Policies?**
```
Protect main/production branches
Enforce code quality
Require reviews
Prevent direct pushes

Result:
✓ All changes via pull requests
✓ Code reviewed before merge
✓ Build validation before merge
✓ Work item linking
```

**Setting Up Branch Policies:**
```
1. Repos → Branches
2. Find "main" branch
3. Click "..." → Branch policies
4. Enable:
   ✓ Require minimum number of reviewers: 2
   ✓ Check for linked work items: Required
   ✓ Build validation: Required
   ✓ Limit merge types: Squash merge only
   ✓ Automatically include reviewers: Add team
```

---

### **Pull Request Workflow:**

```
Developer creates feature branch:
$ git checkout -b feature/add-login
$ # Make changes
$ git add .
$ git commit -m "Add login functionality"
$ git push origin feature/add-login

Create Pull Request:
1. Azure DevOps → Repos → Pull Requests
2. Click "New pull request"
3. Source: feature/add-login
4. Target: main
5. Title: "Add login functionality"
6. Description: Explain changes
7. Reviewers: Add team members
8. Work items: Link user story
9. Click "Create"

Review Process:
- Reviewers review code
- Add comments
- Request changes
- Build validation runs
- All checks pass → Approve
- Complete pull request → Merge to main

Automated:
- Branch policy enforces reviews
- Build must pass
- Work item automatically linked
```

---

## 🔄 Azure Pipelines

CI/CD automation platform.

### **What is Azure Pipelines?**

```
Azure Pipelines = CI/CD Service

Supports:
✓ Any language (Node.js, Python, Java, .NET, Go, etc.)
✓ Any platform (Linux, Windows, macOS)
✓ Any cloud (Azure, AWS, GCP)
✓ Containers and Kubernetes
✓ YAML pipelines (code as config)
✓ Classic editor (GUI)
✓ Free tier: 1800 minutes/month
```

---

### **Pipeline Concepts:**

```
Pipeline Structure:

Pipeline
  └── Stages (Dev, QA, Prod)
      └── Jobs (Build, Test, Deploy)
          └── Steps (Tasks)

Example:
Pipeline: MyApp-CI-CD
  Stage: Build
    Job: Build-Job
      Steps:
        - Checkout code
        - Install dependencies
        - Run tests
        - Build artifact
  Stage: Deploy-Dev
    Job: Deploy-Job
      Steps:
        - Download artifact
        - Deploy to Dev AKS
  Stage: Deploy-Prod
    Job: Deploy-Job
      Steps:
        - Download artifact
        - Deploy to Prod AKS
```

---

### **Creating Pipeline - YAML:**

**Simple Node.js CI Pipeline:**

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
    - main
    - develop
  paths:
    exclude:
    - docs/*
    - README.md

pool:
  vmImage: 'ubuntu-latest'

variables:
  nodeVersion: '18.x'

stages:
- stage: Build
  displayName: 'Build and Test'
  jobs:
  - job: BuildJob
    displayName: 'Build Application'
    steps:
    # Install Node.js
    - task: NodeTool@0
      displayName: 'Install Node.js'
      inputs:
        versionSpec: $(nodeVersion)
    
    # Install dependencies
    - script: |
        npm ci
      displayName: 'npm install'
    
    # Run linting
    - script: |
        npm run lint
      displayName: 'Run ESLint'
    
    # Run unit tests
    - script: |
        npm test -- --coverage
      displayName: 'Run tests with coverage'
    
    # Publish test results
    - task: PublishTestResults@2
      displayName: 'Publish test results'
      inputs:
        testResultsFormat: 'JUnit'
        testResultsFiles: '**/test-results.xml'
        mergeTestResults: true
    
    # Publish code coverage
    - task: PublishCodeCoverageResults@1
      displayName: 'Publish code coverage'
      inputs:
        codeCoverageTool: 'Cobertura'
        summaryFileLocation: '$(System.DefaultWorkingDirectory)/coverage/cobertura-coverage.xml'
    
    # Build application
    - script: |
        npm run build
      displayName: 'Build application'
    
    # Publish build artifact
    - task: PublishBuildArtifacts@1
      displayName: 'Publish artifact'
      inputs:
        PathtoPublish: '$(Build.ArtifactStagingDirectory)'
        ArtifactName: 'drop'
        publishLocation: 'Container'
```

---

### **Docker Build & Push Pipeline:**

```yaml
# azure-pipelines-docker.yml
trigger:
  branches:
    include:
    - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  dockerRegistryServiceConnection: 'myacr-connection'
  imageRepository: 'myapp'
  containerRegistry: 'myregistry.azurecr.io'
  dockerfilePath: '$(Build.SourcesDirectory)/Dockerfile'
  tag: '$(Build.BuildId)'

stages:
- stage: Build
  displayName: 'Build and Push Docker Image'
  jobs:
  - job: Docker
    displayName: 'Build Docker Image'
    steps:
    # Docker build and push
    - task: Docker@2
      displayName: 'Build and push image'
      inputs:
        command: buildAndPush
        repository: $(imageRepository)
        dockerfile: $(dockerfilePath)
        containerRegistry: $(dockerRegistryServiceConnection)
        tags: |
          $(tag)
          latest
    
    # Scan image for vulnerabilities
    - task: AzureContainerScan@0
      displayName: 'Scan image for vulnerabilities'
      inputs:
        image: '$(containerRegistry)/$(imageRepository):$(tag)'
    
    # Update Kubernetes manifests with new tag
    - script: |
        sed -i 's|IMAGE_TAG|$(tag)|g' k8s/deployment.yaml
      displayName: 'Update K8s manifest with image tag'
    
    # Publish K8s manifests
    - task: PublishBuildArtifacts@1
      displayName: 'Publish K8s manifests'
      inputs:
        PathtoPublish: 'k8s/'
        ArtifactName: 'manifests'
```

---

### **Multi-Stage CD Pipeline:**

```yaml
# azure-pipelines-cd.yml
trigger: none  # CD triggered by CI completion

resources:
  pipelines:
  - pipeline: ci-pipeline
    source: 'MyApp-CI'
    trigger:
      branches:
        include:
        - main

pool:
  vmImage: 'ubuntu-latest'

stages:
# Deploy to Development
- stage: Deploy_Dev
  displayName: 'Deploy to Development'
  jobs:
  - deployment: DeployDev
    displayName: 'Deploy to Dev AKS'
    environment: 'development'
    strategy:
      runOnce:
        deploy:
          steps:
          - download: ci-pipeline
            artifact: manifests
          
          - task: KubernetesManifest@0
            displayName: 'Deploy to AKS'
            inputs:
              action: 'deploy'
              kubernetesServiceConnection: 'aks-dev-connection'
              namespace: 'development'
              manifests: |
                $(Pipeline.Workspace)/ci-pipeline/manifests/deployment.yaml
                $(Pipeline.Workspace)/ci-pipeline/manifests/service.yaml

# Deploy to Staging
- stage: Deploy_Staging
  displayName: 'Deploy to Staging'
  dependsOn: Deploy_Dev
  condition: succeeded()
  jobs:
  - deployment: DeployStaging
    displayName: 'Deploy to Staging AKS'
    environment: 'staging'
    strategy:
      runOnce:
        deploy:
          steps:
          - download: ci-pipeline
            artifact: manifests
          
          - task: KubernetesManifest@0
            displayName: 'Deploy to AKS'
            inputs:
              action: 'deploy'
              kubernetesServiceConnection: 'aks-staging-connection'
              namespace: 'staging'
              manifests: |
                $(Pipeline.Workspace)/ci-pipeline/manifests/deployment.yaml
                $(Pipeline.Workspace)/ci-pipeline/manifests/service.yaml

# Deploy to Production
- stage: Deploy_Production
  displayName: 'Deploy to Production'
  dependsOn: Deploy_Staging
  condition: succeeded()
  jobs:
  - deployment: DeployProduction
    displayName: 'Deploy to Prod AKS'
    environment: 'production'  # Requires manual approval
    strategy:
      runOnce:
        deploy:
          steps:
          - download: ci-pipeline
            artifact: manifests
          
          - task: KubernetesManifest@0
            displayName: 'Deploy to AKS'
            inputs:
              action: 'deploy'
              kubernetesServiceConnection: 'aks-prod-connection'
              namespace: 'production'
              manifests: |
                $(Pipeline.Workspace)/ci-pipeline/manifests/deployment.yaml
                $(Pipeline.Workspace)/ci-pipeline/manifests/service.yaml
          
          # Run smoke tests
          - script: |
              curl -f https://myapp.production.com/health || exit 1
            displayName: 'Smoke test'
```

---

## ⏰ Pipeline Triggers

Control when pipelines run automatically.

### **CI Triggers (Continuous Integration):**

**Basic Trigger:**
```yaml
# Trigger on any commit to main
trigger:
  branches:
    include:
    - main
```

**Multiple Branches:**
```yaml
trigger:
  branches:
    include:
    - main
    - develop
    - release/*
    exclude:
    - feature/*
```

**Path Filters:**
```yaml
# Only trigger if specific paths change
trigger:
  branches:
    include:
    - main
  paths:
    include:
    - src/*
    - tests/*
    exclude:
    - docs/*
    - README.md
    - '**/*.md'
```

**Batch Changes:**
```yaml
trigger:
  batch: true  # Wait for pipeline to finish before starting new
  branches:
    include:
    - main
    
# Without batch: Multiple commits = multiple pipeline runs
# With batch: Multiple commits = single pipeline run with all changes
```

---

### **PR Triggers (Pull Request Validation):**

```yaml
# Trigger on PR to main
pr:
  branches:
    include:
    - main
  paths:
    include:
    - src/*
    exclude:
    - docs/*

# Or disable PR triggers
pr: none
```

**PR Validation Example:**
```yaml
# azure-pipelines-pr.yml
trigger: none  # No CI trigger
pr:
  branches:
    include:
    - main
  drafts: false  # Don't run on draft PRs

stages:
- stage: Validate
  jobs:
  - job: PRValidation
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - script: npm run lint
      displayName: 'Lint code'
    
    - script: npm test
      displayName: 'Run tests'
    
    - script: npm run security-scan
      displayName: 'Security scan'
```

---

### **Scheduled Triggers:**

```yaml
# Run nightly builds
schedules:
- cron: "0 0 * * *"  # Midnight daily
  displayName: Nightly build
  branches:
    include:
    - main
  always: true  # Run even if no changes

# Multiple schedules
schedules:
- cron: "0 0 * * *"  # Midnight
  displayName: Nightly full build
  branches:
    include:
    - main

- cron: "0 */6 * * *"  # Every 6 hours
  displayName: Integration tests
  branches:
    include:
    - develop
  always: false  # Only if changes
```

**Cron Format:**
```
┌─────── minute (0 - 59)
│ ┌────── hour (0 - 23)
│ │ ┌───── day of month (1 - 31)
│ │ │ ┌──── month (1 - 12)
│ │ │ │ ┌─── day of week (0 - 6) (Sunday=0)
│ │ │ │ │
* * * * *

Examples:
0 0 * * *     # Daily at midnight
0 9 * * 1-5   # Weekdays at 9 AM
0 0 1 * *     # First day of month
*/15 * * * *  # Every 15 minutes
```

---

### **Pipeline Resource Triggers:**

```yaml
# Trigger when another pipeline completes
resources:
  pipelines:
  - pipeline: upstream-ci  # Identifier
    source: MyApp-CI       # Pipeline name
    trigger:
      branches:
        include:
        - main

# This CD pipeline runs after CI completes
stages:
- stage: Deploy
  jobs:
  - job: DeployJob
    steps:
    - download: upstream-ci
      artifact: drop
```

---

### **Manual Triggers (No Auto-Trigger):**

```yaml
# No automatic triggers - manual only
trigger: none
pr: none
schedules: []

# Run manually from Azure DevOps UI
# Or via API/CLI
```

---

### **Trigger Best Practices:**

```
✓ Use PR triggers for validation
✓ Use path filters to avoid unnecessary runs
✓ Batch commits in high-traffic branches
✓ Schedule expensive jobs off-hours
✓ Use separate pipelines for CI and CD
✗ Don't trigger on documentation changes
✗ Don't run full pipeline for every PR
```

---

## 📊 Azure Boards

Agile project management and work tracking.

### **What is Azure Boards?**

```
Azure Boards = Work Item Tracking

Track work with:
✓ Epics (Large features)
✓ Features (Medium features)
✓ User Stories (User requirements)
✓ Tasks (Actionable work)
✓ Bugs (Issues to fix)

Agile process:
- Sprint planning
- Kanban board
- Backlogs
- Queries
- Dashboards
```

---

### **Work Item Hierarchy:**

```
Epic: Customer Portal Redesign
  ├── Feature: User Authentication
  │   ├── User Story: User can login with email
  │   │   ├── Task: Design login form
  │   │   ├── Task: Implement backend API
  │   │   ├── Task: Write unit tests
  │   │   └── Task: Update documentation
  │   │
  │   └── User Story: User can reset password
  │       ├── Task: Design reset flow
  │       └── Task: Implement email service
  │
  └── Feature: Dashboard UI
      └── User Story: Display user statistics
          └── Task: Create dashboard component
```

---

### **Creating Work Items:**

```
Via UI:
1. Boards → Work Items
2. Click "+ New Work Item"
3. Type: User Story
4. Title: "User can view order history"
5. Assigned to: Team member
6. Area: MyApp\Frontend
7. Iteration: Sprint 10
8. Description: Acceptance criteria
9. Save

Linking to Code:
In commit message:
git commit -m "Add order history view #1234"
# #1234 = Work item ID

In Pull Request:
Link work item when creating PR
Automatically closed when PR merged
```

---

## 📦 Azure Artifacts

Package management for dependencies and build artifacts.

### **What is Azure Artifacts?**

```
Azure Artifacts = Package Repository

Supports:
✓ npm packages (Node.js)
✓ NuGet packages (.NET)
✓ Maven packages (Java)
✓ Python packages (PyPI)
✓ Universal Packages

Use cases:
- Host internal packages
- Cache public packages
- Share libraries across projects
- Version management
```

---

### **Creating Feed:**

```
1. Artifacts → Create Feed
2. Name: mycompany-npm
3. Visibility: Organization (or Private/Public)
4. Upstream sources: Include npmjs.com
5. Create

Configure npm to use feed:
$ npm config set registry https://pkgs.dev.azure.com/myorg/_packaging/mycompany-npm/npm/registry/

Authenticate:
$ npx vsts-npm-auth -config .npmrc

Publish package:
$ npm publish
```

---

## 🔌 Service Connections

Connect Azure Pipelines to external services.

### **What are Service Connections?**

```
Service Connections = Credentials for External Services

Types:
- Azure Resource Manager (Azure subscription)
- Docker Registry (ACR, Docker Hub)
- Kubernetes (AKS, EKS, GKE)
- GitHub / Bitbucket
- SSH
- Generic (custom endpoints)
```

---

### **Creating Service Connection:**

**Azure Resource Manager (For Azure deployments):**
```
1. Project Settings → Service connections
2. Click "+ New service connection"
3. Type: Azure Resource Manager
4. Authentication: Service Principal (automatic)
5. Subscription: Select subscription
6. Resource group: (Optional) Limit scope
7. Service connection name: azure-prod-connection
8. Grant access to all pipelines: ✓
9. Save

Use in pipeline:
- task: AzureWebApp@1
  inputs:
    azureSubscription: 'azure-prod-connection'
    appName: 'myapp'
```

**Docker Registry (For ACR):**
```
1. New service connection
2. Type: Docker Registry
3. Registry type: Azure Container Registry
4. Subscription: Select subscription
5. Registry: Select ACR
6. Service connection name: myacr-connection
7. Save

Use in pipeline:
- task: Docker@2
  inputs:
    containerRegistry: 'myacr-connection'
    command: 'buildAndPush'
```

**Kubernetes (For AKS):**
```
1. New service connection
2. Type: Kubernetes
3. Authentication: Azure Subscription
4. Subscription: Select subscription
5. Cluster: Select AKS cluster
6. Namespace: default (or specific)
7. Service connection name: aks-prod-connection
8. Save

Use in pipeline:
- task: KubernetesManifest@0
  inputs:
    kubernetesServiceConnection: 'aks-prod-connection'
    action: 'deploy'
```

---

## 🔐 Variable Groups & Secrets

Manage configuration and secrets across pipelines.

### **What are Variable Groups?**

```
Variable Groups = Shared Variables

Benefits:
✓ Centralized configuration
✓ Reusable across pipelines
✓ Secure secrets (encrypted)
✓ Link to Azure Key Vault
✓ Environment-specific values
```

---

### **Creating Variable Group:**

```
1. Pipelines → Library
2. Click "+ Variable group"
3. Name: production-config
4. Variables:
   - DATABASE_HOST = prod-db.database.azure.com
   - DATABASE_NAME = myapp_prod
   - API_URL = https://api.production.com
   - DATABASE_PASSWORD = ******* (Secret!)
5. For secrets:
   - Click lock icon
   - Value hidden in logs
6. Save

Link to Azure Key Vault:
1. Toggle "Link secrets from Azure Key Vault"
2. Azure subscription: Select
3. Key vault: Select vault
4. Authorize
5. Add secrets from vault

Use in pipeline:
variables:
- group: production-config

steps:
- script: echo $(DATABASE_HOST)
  # Secrets automatically masked in logs
```

---

## 🌍 Environments

Manage deployment targets and approvals.

### **What are Environments?**

```
Environments = Deployment Targets

Features:
✓ Deployment history
✓ Manual approvals
✓ Branch policies
✓ Security checks
✓ Kubernetes namespaces
✓ Virtual machines
```

---

### **Creating Environment:**

```
1. Pipelines → Environments
2. Click "+ New environment"
3. Name: production
4. Resource: Kubernetes (or VM, None)
5. For Kubernetes:
   - Service connection: aks-prod-connection
   - Namespace: production
6. Create

Configure Approvals:
1. Click environment → "..." → Approvals and checks
2. Add "Approvals"
3. Approvers: Add team members
4. Minimum number: 2
5. Timeout: 30 days
6. Instructions: "Review changes before prod deployment"
7. Create

Result:
- Pipeline pauses at production stage
- Sends email to approvers
- Requires 2 approvals
- Then proceeds with deployment
```

---

## 🔒 Permissions & Security

Manage access control in Azure DevOps.

### **Permission Levels:**

```
Organization Level:
- Organization Owner
- Organization Administrator
- Project Collection Administrator

Project Level:
- Project Administrator
- Contributor
- Reader

Repository Level:
- Repository Administrator
- Contributor
- Reader

Pipeline Level:
- Pipeline Administrator
- Build Administrator
- Release Administrator
```

---

### **Adding Users to Organization:**

```
1. Organization Settings → Users
2. Click "+ Add users"
3. Enter email addresses (comma-separated)
4. Access level:
   - Basic: Full access (up to 5 free users)
   - Stakeholder: Limited access (free, unlimited)
5. Add to projects: Select projects
6. Azure DevOps Groups: Project Contributors
7. Send invite email: ✓
8. Add
```

---

### **Project-Level Permissions:**

```
1. Project Settings → Permissions
2. Select group (Contributors, Readers, etc.)
3. Members tab → Add user/group
4. Permissions tab → Set specific permissions

Common Permissions:
- View project-level information
- Edit project-level information
- Delete project
- Create new projects
- Manage build resources
- Administer build permissions
```

---

### **Repository Permissions:**

```
1. Repos → Select repo → Settings → Security
2. Add user/group
3. Grant permissions:
   - Read
   - Contribute
   - Create branch
   - Force push
   - Manage permissions
   - Remove others' locks
   - Bypass policies

Branch-specific permissions:
1. Repos → Branches → Select branch → "..." → Branch security
2. Set permissions per user/group
```

---

### **Pipeline Permissions:**

```
1. Pipelines → Select pipeline → "..." → Security
2. Add user/group
3. Grant permissions:
   - View builds
   - Edit build pipeline
   - Queue builds
   - Delete builds
   - Administer build permissions
   - Manage build queue

Best Practice:
- Developers: View + Queue
- Build engineers: View + Edit + Queue
- Admins: All permissions
```

---

### **Security Groups:**

**Built-in Groups:**

```
[Project]\Project Administrators
- Full project control
- Manage permissions
- Configure project settings

[Project]\Contributors
- Create work items
- Commit code
- Create branches
- Queue builds

[Project]\Readers
- View-only access
- Cannot make changes
- Good for stakeholders

[Project]\Build Administrators
- Manage build pipelines
- Configure agent pools
- Manage service connections
```

**Creating Custom Security Group:**

```
1. Project Settings → Permissions
2. New group
3. Name: Release Approvers
4. Description: Users who approve production releases
5. Members: Add users
6. Permissions: Grant specific permissions
7. Save

Use in approvals:
Environments → production → Approvals and checks
Add approvers → Select "Release Approvers" group
```

---

### **Service Connection Security:**

```
Service connections contain sensitive credentials

Best practices:
1. Project Settings → Service connections → Select connection
2. Security tab
3. Grant minimum permissions:
   - Creator: Full access
   - Contributors: Use only (not manage)
   - Specific pipelines: Explicitly grant access

4. Enable "Grant access to all pipelines": ✗ (Disable!)
5. Explicitly authorize per pipeline
```

---

### **Secure Files:**

```
Store sensitive files (certificates, config files)

1. Pipelines → Library → Secure files
2. Upload file (e.g., signing certificate, kubeconfig)
3. Set permissions
4. Authorize for specific pipelines only

Use in pipeline:
- task: DownloadSecureFile@1
  name: sslCertificate
  inputs:
    secureFile: 'my-cert.pfx'

- script: |
    echo "Using cert: $(sslCertificate.secureFilePath)"
```

---

### **Audit Logs:**

```
Track changes and access:

1. Organization Settings → Auditing
2. Enable auditing
3. Stream to Log Analytics (optional)
4. View audit events:
   - User added/removed
   - Permission changes
   - Pipeline runs
   - Resource access
   - Configuration changes

Query logs:
- Filter by user, date, action
- Export to CSV
- Set up alerts
```

---

## 🔌 Extensions & Marketplace

Extend Azure DevOps functionality.

### **What are Extensions?**

```
Extensions = Add-ons for Azure DevOps

Types:
- Pipeline tasks (custom build/deploy tasks)
- Dashboard widgets
- Service hooks
- Repository extensions
- Board extensions

Sources:
1. Microsoft extensions (official)
2. Verified publishers
3. Community extensions
```

---

### **Installing Extensions:**

```
1. Organization Settings → Extensions
2. Browse marketplace
3. Search for extension
4. Click extension
5. Select organization
6. Install

Or:
1. Visit: https://marketplace.visualstudio.com/azuredevops
2. Search extension
3. Click "Get it free"
4. Select organization
5. Install
```

---

### **Popular Extensions:**

**1. SonarQube/SonarCloud (Code Quality):**
```yaml
# After installing extension
- task: SonarCloudPrepare@1
  inputs:
    SonarCloud: 'SonarCloud-connection'
    organization: 'myorg'
    scannerMode: 'CLI'
    projectKey: 'myproject'

- script: npm test

- task: SonarCloudAnalyze@1

- task: SonarCloudPublish@1
  inputs:
    pollingTimeoutSec: '300'
```

**2. WhiteSource Bolt (Security Scanning):**
```yaml
- task: WhiteSource@21
  inputs:
    cwd: '$(System.DefaultWorkingDirectory)'
```

**3. Terraform (Infrastructure as Code):**
```yaml
- task: TerraformInstaller@0
  inputs:
    terraformVersion: '1.5.0'

- task: TerraformTaskV4@4
  inputs:
    provider: 'azurerm'
    command: 'init'

- task: TerraformTaskV4@4
  inputs:
    command: 'apply'
    environmentServiceNameAzureRM: 'azure-connection'
```

**4. Slack Notifications:**
```yaml
- task: SlackNotification@1
  inputs:
    SlackApiToken: '$(SlackToken)'
    Channel: '#deployments'
    Message: 'Deployment to production completed!'
```

**5. Replace Tokens (Variable Substitution):**
```yaml
- task: replacetokens@5
  inputs:
    targetFiles: '**/*.config'
    encoding: 'auto'
    tokenPattern: 'default'
    writeBOM: true
```

---

### **Managing Extensions:**

```
Organization Settings → Extensions

Installed:
- View installed extensions
- Update extensions
- Disable/Enable
- Uninstall

Shared:
- Install for all projects
- Restrict to specific projects

Requested:
- Approve/reject extension requests
```

---

### **Creating Custom Extensions:**

```bash
# Install tfx-cli
npm install -g tfx-cli

# Create extension structure
mkdir my-extension
cd my-extension

# Create manifest (vss-extension.json)
{
  "manifestVersion": 1,
  "id": "my-custom-task",
  "name": "My Custom Task",
  "version": "1.0.0",
  "publisher": "mycompany",
  "targets": [
    {"id": "Microsoft.VisualStudio.Services"}
  ],
  "contributions": [
    {
      "id": "my-task",
      "type": "ms.vss-distributed-task.task",
      "targets": ["ms.vss-distributed-task.tasks"],
      "properties": {
        "name": "MyTask"
      }
    }
  ]
}

# Package extension
tfx extension create --manifest-globs vss-extension.json

# Publish to marketplace
tfx extension publish --manifest-globs vss-extension.json --share-with myorg
```

---

## 🔑 Personal Access Tokens (PATs)

Authenticate without passwords.

### **What is a PAT?**

```
PAT = Personal Access Token

Like a password but:
✓ Scoped to specific permissions
✓ Can be revoked without changing password
✓ Time-limited (expiration)
✓ Used for automation

Use cases:
- Git authentication
- Azure DevOps CLI
- REST API calls
- Third-party tool integration
```

---

### **Creating PAT:**

```
1. Azure DevOps → User settings (top right)
2. Personal access tokens
3. "+ New Token"
4. Name: Git Access
5. Organization: Select org (or All accessible)
6. Expiration: 90 days (max 1 year)
7. Scopes:
   Custom defined:
   - Code: Read & write
   - Build: Read & execute
   - Release: Read, write, & execute
   - Or: Full access (use sparingly!)
8. Create
9. Copy token (only shown once!)
10. Store securely (password manager)
```

---

### **Scopes Explained:**

```
Code (Repos):
- Read: Clone, pull
- Write: Push, create PRs

Build (Pipelines):
- Read: View pipelines
- Execute: Run pipelines

Release:
- Read: View releases
- Write: Manage releases
- Execute: Deploy releases

Work Items:
- Read: View work items
- Write: Create/edit work items

Project and Team:
- Read: View project settings
- Write: Manage project settings

Full Access:
- All permissions (use only when necessary)
```

---

### **Using PAT:**

**Git Authentication:**
```bash
# Method 1: Clone with PAT in URL
git clone https://{PAT}@dev.azure.com/myorg/MyProject/_git/myrepo

# Method 2: Configure credential
git config --global credential.helper store
git clone https://dev.azure.com/myorg/MyProject/_git/myrepo
# Username: {PAT}
# Password: {PAT}

# Credentials stored in ~/.git-credentials
```

**Azure DevOps CLI:**
```bash
# Set PAT as environment variable
export AZURE_DEVOPS_EXT_PAT={YOUR_PAT}

# Or login with PAT
echo {YOUR_PAT} | az devops login --organization https://dev.azure.com/myorg
```

**REST API:**
```bash
# Use PAT in Authorization header
curl -u :{PAT} \
  https://dev.azure.com/myorg/_apis/projects?api-version=7.0

# Or base64 encode PAT
AUTH=$(echo -n :{PAT} | base64)
curl -H "Authorization: Basic $AUTH" \
  https://dev.azure.com/myorg/_apis/projects?api-version=7.0
```

**PowerShell:**
```powershell
$PAT = "your-pat-token"
$Token = [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes(":$($PAT)"))
$Header = @{Authorization = "Basic $Token"}

Invoke-RestMethod -Uri "https://dev.azure.com/myorg/_apis/projects?api-version=7.0" -Headers $Header
```

---

### **Managing PATs:**

```
View/Revoke PATs:
1. User settings → Personal access tokens
2. View all tokens
3. Revoke token:
   - Click "..." → Revoke
   - Confirm
4. Regenerate token:
   - Click token name
   - Regenerate
   - Copy new token

Best practices:
✓ Use short expiration (90 days)
✓ Create specific PATs for different uses
✓ Revoke immediately if compromised
✓ Never commit PATs to code
✓ Store in password manager or Key Vault
✓ Rotate regularly
✗ Don't share PATs
✗ Don't use full access scope
```

---

## ☸️ Integration with AKS

Deploy to Azure Kubernetes Service.

### **Complete AKS Deployment Pipeline:**

```yaml
# azure-pipelines-aks.yml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

variables:
  dockerRegistryServiceConnection: 'myacr'
  imageRepository: 'myapp'
  containerRegistry: 'myregistry.azurecr.io'
  tag: '$(Build.BuildId)'
  
stages:
# Build Stage
- stage: Build
  displayName: 'Build and Push Image'
  jobs:
  - job: Build
    steps:
    - task: Docker@2
      displayName: 'Build and push image to ACR'
      inputs:
        command: buildAndPush
        repository: $(imageRepository)
        dockerfile: '**/Dockerfile'
        containerRegistry: $(dockerRegistryServiceConnection)
        tags: |
          $(tag)
          latest
    
    - script: |
        echo "$(containerRegistry)/$(imageRepository):$(tag)" > $(Build.ArtifactStagingDirectory)/image-tag.txt
      displayName: 'Save image tag'
    
    - publish: k8s/
      artifact: k8s-manifests
      displayName: 'Publish K8s manifests'
    
    - publish: $(Build.ArtifactStagingDirectory)/image-tag.txt
      artifact: image-tag
      displayName: 'Publish image tag'

# Deploy to Development
- stage: DeployDev
  displayName: 'Deploy to Development'
  dependsOn: Build
  jobs:
  - deployment: DeployDev
    environment: 'development'
    strategy:
      runOnce:
        deploy:
          steps:
          - download: current
            artifact: k8s-manifests
          - download: current
            artifact: image-tag
          
          - script: |
              IMAGE_TAG=$(cat $(Pipeline.Workspace)/image-tag/image-tag.txt)
              sed -i "s|IMAGE_PLACEHOLDER|$IMAGE_TAG|g" $(Pipeline.Workspace)/k8s-manifests/*.yaml
            displayName: 'Update manifest with image tag'
          
          - task: KubernetesManifest@0
            displayName: 'Create/Update namespace'
            inputs:
              action: 'deploy'
              kubernetesServiceConnection: 'aks-dev'
              namespace: 'development'
              manifests: '$(Pipeline.Workspace)/k8s-manifests/namespace.yaml'
          
          - task: KubernetesManifest@0
            displayName: 'Deploy to AKS'
            inputs:
              action: 'deploy'
              kubernetesServiceConnection: 'aks-dev'
              namespace: 'development'
              manifests: |
                $(Pipeline.Workspace)/k8s-manifests/deployment.yaml
                $(Pipeline.Workspace)/k8s-manifests/service.yaml
                $(Pipeline.Workspace)/k8s-manifests/ingress.yaml

# Deploy to Production
- stage: DeployProd
  displayName: 'Deploy to Production'
  dependsOn: DeployDev
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  jobs:
  - deployment: DeployProd
    environment: 'production'  # Requires approval
    strategy:
      runOnce:
        deploy:
          steps:
          - download: current
            artifact: k8s-manifests
          - download: current
            artifact: image-tag
          
          - script: |
              IMAGE_TAG=$(cat $(Pipeline.Workspace)/image-tag/image-tag.txt)
              sed -i "s|IMAGE_PLACEHOLDER|$IMAGE_TAG|g" $(Pipeline.Workspace)/k8s-manifests/*.yaml
            displayName: 'Update manifest with image tag'
          
          - task: KubernetesManifest@0
            displayName: 'Deploy to Production AKS'
            inputs:
              action: 'deploy'
              kubernetesServiceConnection: 'aks-prod'
              namespace: 'production'
              manifests: |
                $(Pipeline.Workspace)/k8s-manifests/deployment.yaml
                $(Pipeline.Workspace)/k8s-manifests/service.yaml
                $(Pipeline.Workspace)/k8s-manifests/ingress.yaml
          
          - script: |
              sleep 30
              curl -f https://myapp.production.com/health || exit 1
            displayName: 'Health check'
```

---

## ✅ Best Practices

### **1. YAML Pipelines Over Classic:**

```
✓ Use YAML pipelines (infrastructure as code)
✗ Avoid classic editor (GUI)

Benefits:
- Version controlled
- Code review via PR
- Reusable templates
- Better for CI/CD
```

---

### **2. Use Templates:**

```yaml
# templates/build-template.yml
parameters:
- name: nodeVersion
  default: '18.x'

steps:
- task: NodeTool@0
  inputs:
    versionSpec: ${{ parameters.nodeVersion }}

- script: npm ci
  displayName: 'Install dependencies'

- script: npm test
  displayName: 'Run tests'

# azure-pipelines.yml
stages:
- stage: Build
  jobs:
  - job: BuildJob
    steps:
    - template: templates/build-template.yml
      parameters:
        nodeVersion: '18.x'
```

---

### **3. Secure Secrets:**

```
✓ Use Azure Key Vault for secrets
✓ Variable groups with secret variables
✗ Never hardcode secrets in YAML

Example:
variables:
- group: keyvault-secrets  # Linked to Key Vault

steps:
- script: |
    echo "Deploying with credentials..."
    # $(DB_PASSWORD) automatically fetched from Key Vault
```

---

### **4. Branch Protection:**

```
main/master branch:
✓ Require pull requests
✓ Require 2+ reviewers
✓ Require build validation
✓ No direct commits
✓ Squash merge only
```

---

### **5. Use Environments for CD:**

```
Environments provide:
✓ Deployment history
✓ Manual approvals
✓ Security checks
✓ Deployment gates

production environment:
- Requires 2 approvals
- Business hours only
- Automated testing gates
```

---

## 🛠️ Troubleshooting

### **Issue 1: Pipeline Fails to Authenticate:**

```
Error: Failed to authenticate to Azure

Solutions:
1. Check service connection:
   - Project Settings → Service connections
   - Verify connection is valid
   - Re-authorize if needed

2. Grant permissions:
   - Azure subscription → Access control (IAM)
   - Add service principal as Contributor

3. Update service connection:
   - Delete old connection
   - Create new with proper permissions
```

---

### **Issue 2: Docker Push Fails:**

```
Error: unauthorized: authentication required

Solutions:
1. Verify ACR service connection
2. Check ACR permissions (AcrPush role)
3. Login to ACR manually:
   az acr login --name myregistry
4. Update service connection credentials
```

---

### **Issue 3: AKS Deployment Fails:**

```
Error: Unable to connect to cluster

Solutions:
1. Verify service connection to AKS
2. Check kubeconfig:
   az aks get-credentials --resource-group myRG --name myAKS
3. Verify namespace exists
4. Check RBAC permissions
5. Validate manifests:
   kubectl apply --dry-run=client -f deployment.yaml
```

---

### **Issue 4: Variable Not Resolved:**

```
Error: $(MY_VARIABLE) not found

Solutions:
1. Check variable group linked:
   variables:
   - group: my-variables

2. Verify variable name (case-sensitive)
3. For runtime variables:
   - Use task output variables
   - Set with ##vso[task.setvariable]
```

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

