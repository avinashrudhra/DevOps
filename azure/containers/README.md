# Azure Containers - Complete Learning Guide

**Azure Container Registry (ACR) & Azure Container Instances (ACI)**

---

## 📚 Table of Contents

1. [Azure Container Services Overview](#overview)
2. [Azure Container Registry (ACR)](#azure-container-registry)
3. [Building and Pushing Images](#building-pushing-images)
4. [ACR Tasks (CI/CD)](#acr-tasks)
5. [Geo-Replication](#geo-replication)
6. [Security and Scanning](#security-scanning)
7. [Azure Container Instances (ACI)](#azure-container-instances)
8. [Real-World Scenarios](#real-world-scenarios)

---

## 🎯 Azure Container Services Overview

Azure provides multiple container services:

| Service | Use Case | Complexity |
|---------|----------|------------|
| **ACR** | Private Docker registry | Low |
| **ACI** | Quick container deployment | Low |
| **AKS** | Production Kubernetes | High |
| **App Service** | Web apps in containers | Medium |

---

## 📦 Azure Container Registry (ACR)

**ACR** is Azure's private Docker registry service.

### **ACR Tiers:**

| Tier | Storage | Webhooks | Geo-Replication | Use Case |
|------|---------|----------|-----------------|----------|
| **Basic** | 10 GB | 2 | ❌ | Dev/Test |
| **Standard** | 100 GB | 10 | ❌ | Small production |
| **Premium** | 500 GB | 500 | ✅ | Enterprise |

**Premium Features:**
- Geo-replication
- Content trust (image signing)
- Private endpoints
- Dedicated resource pool
- More concurrent operations

---

### **Create ACR:**

```bash
# Create Premium ACR
az acr create \
  --resource-group rg-containers \
  --name acrprodregistry001 \
  --sku Premium \
  --location eastus \
  --admin-enabled false

# Get ACR details
az acr show \
  --resource-group rg-containers \
  --name acrprodregistry001 \
  --output table

# Get login server
ACR_LOGIN_SERVER=$(az acr show \
  --resource-group rg-containers \
  --name acrprodregistry001 \
  --query loginServer \
  --output tsv)

echo "ACR Login Server: $ACR_LOGIN_SERVER"
# Output: acrprodregistry001.azurecr.io
```

**Explanation:**
- `--admin-enabled false` - Disable admin user (use Azure AD/Managed Identity instead)
- `--sku Premium` - Required for geo-replication and private endpoints

---

### **Login to ACR:**

```bash
# Method 1: Using Azure CLI (recommended)
az acr login --name acrprodregistry001

# Method 2: Using Docker directly
PASSWORD=$(az acr credential show \
  --name acrprodregistry001 \
  --query passwords[0].value \
  --output tsv)

docker login acrprodregistry001.azurecr.io \
  --username acrprodregistry001 \
  --password $PASSWORD

# Method 3: Using Service Principal
az acr login \
  --name acrprodregistry001 \
  --username <SERVICE_PRINCIPAL_ID> \
  --password <SERVICE_PRINCIPAL_PASSWORD>
```

---

## 🏗️ Building and Pushing Images

### **Method 1: Build Locally and Push:**

```bash
# Build Docker image locally
cd my-app
docker build -t myapp:v1.0 .

# Tag for ACR
docker tag myapp:v1.0 acrprodregistry001.azurecr.io/myapp:v1.0

# Push to ACR
docker push acrprodregistry001.azurecr.io/myapp:v1.0

# Verify
az acr repository list \
  --name acrprodregistry001 \
  --output table

# List tags
az acr repository show-tags \
  --name acrprodregistry001 \
  --repository myapp \
  --output table
```

---

### **Method 2: Build in ACR (Recommended):**

**ACR Build** - Build images directly in Azure without local Docker.

```bash
# Build image in ACR
az acr build \
  --registry acrprodregistry001 \
  --image myapp:v1.0 \
  --file Dockerfile \
  .

# Build with build arguments
az acr build \
  --registry acrprodregistry001 \
  --image myapp:v1.1 \
  --build-arg NODE_ENV=production \
  --build-arg API_VERSION=v2 \
  --file Dockerfile \
  .

# Build multi-platform image
az acr build \
  --registry acrprodregistry001 \
  --image myapp:v1.2 \
  --platform linux/amd64,linux/arm64 \
  --file Dockerfile \
  .
```

**Benefits:**
- No local Docker daemon required
- Faster (runs in Azure)
- Automatically pushes to registry
- Build logs stored in ACR

---

### **Sample Dockerfile:**

**Node.js Application:**
```dockerfile
# Multi-stage build
FROM node:18-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Build application
RUN npm run build

# Production image
FROM node:18-alpine

WORKDIR /app

# Copy from builder
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package.json ./

# Non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

USER nodejs

EXPOSE 3000

CMD ["node", "dist/server.js"]
```

---

## 🔄 ACR Tasks (CI/CD)

**ACR Tasks** automate image building and patching.

### **Quick Task (One-time Build):**

```bash
# Build and push on-demand
az acr build \
  --registry acrprodregistry001 \
  --image myapp:{{.Run.ID}} \
  --file Dockerfile \
  https://github.com/user/myapp.git#main
```

---

### **Automated Task (Trigger on Git Commit):**

```bash
# Create task triggered by Git commit
az acr task create \
  --registry acrprodregistry001 \
  --name build-on-commit \
  --image myapp:{{.Run.ID}} \
  --context https://github.com/user/myapp.git#main \
  --file Dockerfile \
  --git-access-token $(cat github-token.txt) \
  --commit-trigger-enabled true

# List tasks
az acr task list \
  --registry acrprodregistry001 \
  --output table

# Run task manually
az acr task run \
  --registry acrprodregistry001 \
  --name build-on-commit

# View task runs
az acr task list-runs \
  --registry acrprodregistry001 \
  --output table

# View logs
az acr task logs \
  --registry acrprodregistry001 \
  --run-id <RUN_ID>
```

---

### **Base Image Update Task:**

Automatically rebuild when base image updates:

```bash
# Create base image update task
az acr task create \
  --registry acrprodregistry001 \
  --name rebuild-on-base-update \
  --image myapp:{{.Run.ID}} \
  --context https://github.com/user/myapp.git \
  --file Dockerfile \
  --git-access-token $(cat github-token.txt) \
  --base-image-trigger-enabled true \
  --base-image-trigger-type Runtime
```

**How it works:**
1. Your image uses `FROM node:18-alpine`
2. When `node:18-alpine` updates in Docker Hub
3. ACR automatically rebuilds your image
4. Keeps your images patched

---

## 🌍 Geo-Replication

**Geo-replication** creates replicas in multiple regions (Premium SKU only).

```bash
# Enable geo-replication to West US
az acr replication create \
  --registry acrprodregistry001 \
  --location westus

# Enable geo-replication to Europe
az acr replication create \
  --registry acrprodregistry001 \
  --location westeurope

# List replications
az acr replication list \
  --registry acrprodregistry001 \
  --output table

# Delete replication
az acr replication delete \
  --registry acrprodregistry001 \
  --location westus
```

**Benefits:**
- Faster image pulls (closer to deployment region)
- High availability
- Single registry name across regions
- Automatic sync

**Use Case:**
```
AKS in East US → Pulls from acrprodregistry001.azurecr.io (East US replica)
AKS in West Europe → Pulls from acrprodregistry001.azurecr.io (West Europe replica)
```

---

## 🔒 Security and Scanning

### **Image Scanning with Azure Defender:**

```bash
# Enable Azure Defender for ACR
az security pricing create \
  --name ContainerRegistry \
  --tier Standard

# Scan specific image
az acr task create \
  --registry acrprodregistry001 \
  --name scan-image \
  --cmd 'az acr task run --registry acrprodregistry001 --name scan-image' \
  --context /dev/null \
  --assign-identity
```

---

### **Content Trust (Image Signing):**

```bash
# Enable content trust
export DOCKER_CONTENT_TRUST=1
export DOCKER_CONTENT_TRUST_SERVER=https://acrprodregistry001.azurecr.io

# Push signed image
docker push acrprodregistry001.azurecr.io/myapp:v1.0
# Prompts for passphrase, creates signing keys

# Pull only signed images
docker pull acrprodregistry001.azurecr.io/myapp:v1.0
```

---

### **Network Security:**

```bash
# Disable public access
az acr update \
  --name acrprodregistry001 \
  --public-network-enabled false

# Allow specific IP
az acr network-rule add \
  --name acrprodregistry001 \
  --ip-address 203.0.113.5

# Create private endpoint
az network private-endpoint create \
  --resource-group rg-containers \
  --name pe-acr \
  --vnet-name vnet-prod \
  --subnet subnet-containers \
  --private-connection-resource-id $(az acr show -g rg-containers -n acrprodregistry001 --query id -o tsv) \
  --group-id registry \
  --connection-name acr-connection

# Create private DNS zone
az network private-dns zone create \
  --resource-group rg-containers \
  --name privatelink.azurecr.io

# Link to VNet
az network private-dns link vnet create \
  --resource-group rg-containers \
  --zone-name privatelink.azurecr.io \
  --name acr-dns-link \
  --virtual-network vnet-prod \
  --registration-enabled false
```

---

### **Managed Identity Access:**

```bash
# Grant AKS access to ACR
AKS_IDENTITY=$(az aks show \
  --resource-group rg-aks \
  --name aks-prod-cluster \
  --query identityProfile.kubeletidentity.objectId \
  --output tsv)

az role assignment create \
  --assignee $AKS_IDENTITY \
  --role AcrPull \
  --scope $(az acr show -g rg-containers -n acrprodregistry001 --query id -o tsv)

# Grant App Service access
az webapp identity assign \
  --resource-group rg-webapps \
  --name webapp-container-001

WEBAPP_IDENTITY=$(az webapp identity show \
  --resource-group rg-webapps \
  --name webapp-container-001 \
  --query principalId \
  --output tsv)

az role assignment create \
  --assignee $WEBAPP_IDENTITY \
  --role AcrPull \
  --scope $(az acr show -g rg-containers -n acrprodregistry001 --query id -o tsv)
```

---

## 🚀 Azure Container Instances (ACI)

**ACI** provides fast container deployment without managing VMs.

### **When to Use ACI:**

**✅ Good For:**
- Batch jobs
- Build agents (GitHub Actions, Azure DevOps)
- Event-driven processing
- Dev/test environments
- Temporary workloads

**❌ Not Good For:**
- Production applications (use AKS)
- High availability requirements
- Complex networking
- Persistent storage needs

---

### **Create Container Instance:**

**Simple Container:**
```bash
# Deploy nginx container
az container create \
  --resource-group rg-containers \
  --name aci-nginx \
  --image nginx:latest \
  --cpu 1 \
  --memory 1 \
  --ip-address Public \
  --ports 80 \
  --dns-name-label aci-nginx-demo-001

# Get FQDN
az container show \
  --resource-group rg-containers \
  --name aci-nginx \
  --query ipAddress.fqdn \
  --output tsv
# Output: aci-nginx-demo-001.eastus.azurecontainer.io

# Test
curl http://aci-nginx-demo-001.eastus.azurecontainer.io
```

---

**Container from ACR:**
```bash
# Deploy from ACR with managed identity
az container create \
  --resource-group rg-containers \
  --name aci-myapp \
  --image acrprodregistry001.azurecr.io/myapp:v1.0 \
  --acr-identity $(az identity show -g rg-security -n id-app-identity --query id -o tsv) \
  --cpu 2 \
  --memory 4 \
  --ip-address Public \
  --ports 80 443 \
  --environment-variables \
    NODE_ENV=production \
    API_KEY=secret123

# Or with ACR credentials
az container create \
  --resource-group rg-containers \
  --name aci-myapp \
  --image acrprodregistry001.azurecr.io/myapp:v1.0 \
  --registry-login-server acrprodregistry001.azurecr.io \
  --registry-username acrprodregistry001 \
  --registry-password $(az acr credential show --name acrprodregistry001 --query "passwords[0].value" -o tsv) \
  --cpu 2 \
  --memory 4 \
  --ip-address Public \
  --ports 80
```

---

### **Container Groups (Multi-Container):**

**YAML Definition:**
```yaml
# aci-group.yaml
apiVersion: '2021-09-01'
location: eastus
name: myapp-group
properties:
  containers:
  - name: frontend
    properties:
      image: acrprodregistry001.azurecr.io/frontend:latest
      resources:
        requests:
          cpu: 1.0
          memoryInGb: 1.5
      ports:
      - port: 80
        protocol: TCP
      environmentVariables:
      - name: API_URL
        value: 'http://localhost:8080'
  
  - name: backend
    properties:
      image: acrprodregistry001.azurecr.io/backend:latest
      resources:
        requests:
          cpu: 1.0
          memoryInGb: 1.5
      ports:
      - port: 8080
        protocol: TCP
      environmentVariables:
      - name: DATABASE_URL
        secureValue: 'postgresql://...'
  
  osType: Linux
  ipAddress:
    type: Public
    ports:
    - protocol: TCP
      port: 80
    dnsNameLabel: myapp-demo
  imageRegistryCredentials:
  - server: acrprodregistry001.azurecr.io
    username: acrprodregistry001
    password: <password>
tags: null
type: Microsoft.ContainerInstance/containerGroups
```

**Deploy:**
```bash
az container create \
  --resource-group rg-containers \
  --file aci-group.yaml
```

---

### **Monitoring and Logs:**

```bash
# View container logs
az container logs \
  --resource-group rg-containers \
  --name aci-myapp

# Stream logs
az container attach \
  --resource-group rg-containers \
  --name aci-myapp

# Execute command in container
az container exec \
  --resource-group rg-containers \
  --name aci-myapp \
  --exec-command "/bin/bash"

# View container events
az container show \
  --resource-group rg-containers \
  --name aci-myapp \
  --query containers[0].instanceView.events

# Delete container
az container delete \
  --resource-group rg-containers \
  --name aci-myapp \
  --yes
```

---

## 🌟 Real-World Scenarios

### **Scenario 1: CI/CD Pipeline with ACR**

```
GitHub Push → ACR Task Trigger
     ↓
  Build in ACR
     ↓
  Push to ACR
     ↓
  AKS pulls new image
```

---

### **Scenario 2: Scheduled Batch Job with ACI**

```bash
# Deploy batch processing container
az container create \
  --resource-group rg-containers \
  --name batch-processor \
  --image acrprodregistry001.azurecr.io/batch-job:latest \
  --restart-policy Never \
  --cpu 4 \
  --memory 8 \
  --environment-variables BATCH_SIZE=1000

# Container runs once and stops
```

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)
