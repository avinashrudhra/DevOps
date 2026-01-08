# Azure Web Apps (App Service) - Complete Learning Guide

**Platform-as-a-Service Web Hosting: Deploy, Scale & Monitor Web Applications**

---

## 📚 Table of Contents

1. [What is Azure App Service?](#what-is-app-service)
2. [App Service Plans](#app-service-plans)
3. [Creating Your First Web App](#creating-first-web-app)
4. [Deployment Methods](#deployment-methods)
5. [Deployment Slots (Blue-Green)](#deployment-slots)
6. [Custom Domains and SSL](#custom-domains-ssl)
7. [Auto-Scaling](#auto-scaling)
8. [Configuration Management](#configuration-management)
9. [Monitoring and Logging](#monitoring-logging)
10. [Troubleshooting](#troubleshooting)
11. [Security Best Practices](#security-best-practices)
12. [Real-World Scenarios](#real-world-scenarios)

---

## 🎯 What is Azure App Service?

**Azure App Service** is a fully managed platform (PaaS) for hosting web applications, REST APIs, and mobile backends without managing infrastructure.

### **Key Capabilities:**

**✅ Supported Application Types:**
- Web Applications (HTML, CSS, JavaScript)
- REST APIs
- Single Page Applications (React, Angular, Vue)
- Mobile backends
- Background jobs

**✅ Supported Languages/Frameworks:**
- .NET / .NET Core
- Node.js / Express
- Python / Flask / Django
- Java / Spring Boot
- PHP
- Ruby
- Docker containers

**✅ Key Features:**
- Built-in auto-scaling
- CI/CD integration (GitHub, Azure DevOps, Bitbucket)
- Deployment slots for staging
- Custom domains and free SSL certificates
- Built-in authentication (Azure AD, Google, Facebook, Twitter)
- Global distribution with Traffic Manager
- Hybrid connectivity with VNet integration

### **When to Use App Service:**

**✅ Good For:**
- Modern web applications
- REST APIs
- Microservices
- SaaS applications
- Applications requiring rapid deployment

**❌ Not Ideal For:**
- Windows GUI applications
- Legacy applications requiring specific OS configurations
- Applications requiring root access
- Extremely high-performance computing

---

## 🏗️ App Service Plans

An **App Service Plan** defines the compute resources for your web apps.

### **Pricing Tiers:**

| Tier | Use Case | Features | Price Range |
|------|----------|----------|-------------|
| **Free (F1)** | Development/Testing | Shared compute, 60 min/day, 1 GB disk | Free |
| **Shared (D1)** | Development | Shared compute, custom domains | $9.49/month |
| **Basic (B1-B3)** | Low traffic production | Dedicated compute, manual scaling | $54-$217/month |
| **Standard (S1-S3)** | Production | Auto-scaling, slots, traffic manager | $69-$275/month |
| **Premium (P1v3-P3v3)** | Enterprise | Enhanced performance, more slots | $124-$496/month |
| **Isolated (I1v2-I3v2)** | High security | Private network isolation | $620-$2,480/month |

### **Compute Resources by Tier:**

**Basic B1:**
- 1 vCPU
- 1.75 GB RAM
- 10 GB storage
- Up to 3 instances (manual scale)

**Standard S1:**
- 1 vCPU
- 1.75 GB RAM
- 50 GB storage
- Up to 10 instances (auto-scale)
- 5 deployment slots

**Premium P1v3:**
- 2 vCPU
- 8 GB RAM
- 250 GB storage
- Up to 30 instances (auto-scale)
- 20 deployment slots

---

## 🚀 Creating Your First Web App

### **Step 1: Create Resource Group**

```bash
# Create resource group
az group create \
  --name rg-webapps \
  --location eastus

# Verify
az group show --name rg-webapps --output table
```

### **Step 2: Create App Service Plan**

```bash
# Create Standard tier plan
az appservice plan create \
  --resource-group rg-webapps \
  --name plan-prod \
  --location eastus \
  --sku S1 \
  --is-linux

# View plan details
az appservice plan show \
  --resource-group rg-webapps \
  --name plan-prod \
  --output table
```

**Explanation:**
- `--sku S1` - Standard tier (supports auto-scaling and deployment slots)
- `--is-linux` - Linux-based plan (cheaper than Windows)
- Linux plans support containers natively

### **Step 3: Create Web App**

**Node.js Web App:**
```bash
# Create Node.js 18 web app
az webapp create \
  --resource-group rg-webapps \
  --plan plan-prod \
  --name webapp-nodejs-prod-001 \
  --runtime "NODE|18-lts"

# Get default hostname
az webapp show \
  --resource-group rg-webapps \
  --name webapp-nodejs-prod-001 \
  --query defaultHostName \
  --output tsv
# Output: webapp-nodejs-prod-001.azurewebsites.net
```

**Python Web App:**
```bash
# Create Python 3.11 web app
az webapp create \
  --resource-group rg-webapps \
  --plan plan-prod \
  --name webapp-python-prod-001 \
  --runtime "PYTHON|3.11"
```

**Available Runtimes:**
```bash
# List all available runtimes
az webapp list-runtimes --linux --output table

# Common runtimes:
# NODE|18-lts, NODE|20-lts
# PYTHON|3.11, PYTHON|3.12
# DOTNETCORE|8.0
# JAVA|17-java17
# PHP|8.2
```

---

## 📤 Deployment Methods

### **Method 1: Local Git Deployment**

**Step-by-Step:**

```bash
# 1. Enable local git deployment
az webapp deployment source config-local-git \
  --resource-group rg-webapps \
  --name webapp-nodejs-prod-001

# 2. Create deployment user (one-time setup)
az webapp deployment user set \
  --user-name mydeployuser \
  --password 'P@ssw0rd1234!'

# 3. Get git remote URL
GIT_URL=$(az webapp deployment list-publishing-credentials \
  --resource-group rg-webapps \
  --name webapp-nodejs-prod-001 \
  --query scmUri \
  --output tsv)

echo $GIT_URL
# Output: https://mydeployuser@webapp-nodejs-prod-001.scm.azurewebsites.net/webapp-nodejs-prod-001.git

# 4. Add remote and deploy
cd my-nodejs-app
git remote add azure $GIT_URL
git push azure main

# Monitor deployment
az webapp log tail --resource-group rg-webapps --name webapp-nodejs-prod-001
```

---

### **Method 2: GitHub Actions (Recommended for Production)**

**Complete Workflow:**

**`.github/workflows/deploy-webapp.yml`**

```yaml
name: Deploy Node.js to Azure Web App

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:

env:
  AZURE_WEBAPP_NAME: webapp-nodejs-prod-001
  AZURE_WEBAPP_PACKAGE_PATH: '.'
  NODE_VERSION: '18.x'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: 'Checkout Code'
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: 'Install Dependencies'
        run: npm ci

      - name: 'Run Tests'
        run: npm test

      - name: 'Build Application'
        run: npm run build --if-present

      - name: 'Upload Artifact'
        uses: actions/upload-artifact@v3
        with:
          name: node-app
          path: .

  deploy:
    runs-on: ubuntu-latest
    needs: build
    environment:
      name: 'Production'
      url: ${{ steps.deploy.outputs.webapp-url }}

    steps:
      - name: 'Download Artifact'
        uses: actions/download-artifact@v3
        with:
          name: node-app

      - name: 'Deploy to Azure Web App'
        id: deploy
        uses: azure/webapps-deploy@v2
        with:
          app-name: ${{ env.AZURE_WEBAPP_NAME }}
          publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
          package: ${{ env.AZURE_WEBAPP_PACKAGE_PATH }}
```

**Get Publish Profile:**
```bash
# Download publish profile
az webapp deployment list-publishing-profiles \
  --resource-group rg-webapps \
  --name webapp-nodejs-prod-001 \
  --xml > publish-profile.xml

# Add to GitHub Secrets as AZURE_WEBAPP_PUBLISH_PROFILE
```

---

### **Method 3: Docker Container Deployment**

**Single Container:**

```bash
# Create Web App for Containers
az webapp create \
  --resource-group rg-webapps \
  --plan plan-prod \
  --name webapp-container-001 \
  --deployment-container-image-name nginx:latest

# Configure with Azure Container Registry
az webapp config container set \
  --resource-group rg-webapps \
  --name webapp-container-001 \
  --docker-custom-image-name acrprod.azurecr.io/myapp:v1.0 \
  --docker-registry-server-url https://acrprod.azurecr.io \
  --docker-registry-server-user acrprod \
  --docker-registry-server-password $(az acr credential show --name acrprod --query "passwords[0].value" -o tsv)

# Enable continuous deployment (webhook)
az webapp deployment container config \
  --resource-group rg-webapps \
  --name webapp-container-001 \
  --enable-cd true

# Get webhook URL
az webapp deployment container show-cd-url \
  --resource-group rg-webapps \
  --name webapp-container-001
```

---

### **Method 4: ZIP Deploy (Quick Deployment)**

```bash
# Create deployment package
cd my-app
zip -r app.zip .

# Deploy ZIP
az webapp deployment source config-zip \
  --resource-group rg-webapps \
  --name webapp-nodejs-prod-001 \
  --src app.zip

# Deploy with run from package
az webapp config appsettings set \
  --resource-group rg-webapps \
  --name webapp-nodejs-prod-001 \
  --settings WEBSITE_RUN_FROM_PACKAGE="1"
```

---

## 🎯 Deployment Slots (Blue-Green Deployment)

**Deployment slots** enable zero-downtime deployments by deploying to a staging environment first, then swapping to production.

### **Understanding Slots:**

- Each Standard/Premium plan gets 5-20 slots
- Slots are live apps with their own hostnames
- Slots can have different configurations
- Settings can be "slot-specific" or "swapped"

### **Create and Use Slots:**

```bash
# Create staging slot
az webapp deployment slot create \
  --resource-group rg-webapps \
  --name webapp-nodejs-prod-001 \
  --slot staging

# Staging URL: webapp-nodejs-prod-001-staging.azurewebsites.net

# Deploy to staging via git
az webapp deployment source config-local-git \
  --resource-group rg-webapps \
  --name webapp-nodejs-prod-001 \
  --slot staging

# Get staging git URL
STAGING_GIT=$(az webapp deployment list-publishing-credentials \
  --resource-group rg-webapps \
  --name webapp-nodejs-prod-001 \
  --slot staging \
  --query scmUri \
  --output tsv)

# Deploy to staging
git remote add staging $STAGING_GIT
git push staging main

# Test staging: https://webapp-nodejs-prod-001-staging.azurewebsites.net

# Swap staging to production (zero downtime)
az webapp deployment slot swap \
  --resource-group rg-webapps \
  --name webapp-nodejs-prod-001 \
  --slot staging \
  --target-slot production

# If issues occur, swap back
az webapp deployment slot swap \
  --resource-group rg-webapps \
  --name webapp-nodejs-prod-001 \
  --slot production \
  --target-slot staging
```

### **Slot-Specific Settings:**

```bash
# Mark settings as slot-specific (won't swap)
az webapp config appsettings set \
  --resource-group rg-webapps \
  --name webapp-nodejs-prod-001 \
  --slot staging \
  --settings DATABASE_CONNECTION="staging-db-connection" \
  --slot-settings DATABASE_CONNECTION

# Production will keep its own DATABASE_CONNECTION after swap
```

---

## 🌐 Custom Domains and SSL

### **Add Custom Domain:**

```bash
# 1. Get verification ID
az webapp show \
  --resource-group rg-webapps \
  --name webapp-nodejs-prod-001 \
  --query customDomainVerificationId \
  --output tsv

# 2. Add DNS records at your domain registrar:
# Type: CNAME
# Host: www
# Value: webapp-nodejs-prod-001.azurewebsites.net
#
# Type: TXT
# Host: asuid.www
# Value: <customDomainVerificationId>

# 3. Add custom domain to web app
az webapp config hostname add \
  --resource-group rg-webapps \
  --webapp-name webapp-nodejs-prod-001 \
  --hostname www.myapp.com

# 4. Bind free managed certificate (auto-renews)
az webapp config ssl bind \
  --resource-group rg-webapps \
  --name webapp-nodejs-prod-001 \
  --certificate-thumbprint auto \
  --ssl-type SNI

# 5. Enforce HTTPS
az webapp update \
  --resource-group rg-webapps \
  --name webapp-nodejs-prod-001 \
  --https-only true
```

---

## 📈 Auto-Scaling

### **Rule-Based Auto-Scaling:**

```bash
# Create auto-scale settings
az monitor autoscale create \
  --resource-group rg-webapps \
  --resource $(az appservice plan show --resource-group rg-webapps --name plan-prod --query id --output tsv) \
  --name autoscale-prod \
  --min-count 2 \
  --max-count 10 \
  --count 2

# Scale out when CPU > 70%
az monitor autoscale rule create \
  --resource-group rg-webapps \
  --autoscale-name autoscale-prod \
  --condition "Percentage CPU > 70 avg 5m" \
  --scale out 2

# Scale in when CPU < 30%
az monitor autoscale rule create \
  --resource-group rg-webapps \
  --autoscale-name autoscale-prod \
  --condition "Percentage CPU < 30 avg 5m" \
  --scale in 1

# View current scale settings
az monitor autoscale show \
  --resource-group rg-webapps \
  --name autoscale-prod \
  --output table
```

### **Terraform Auto-Scale Configuration:**

```hcl
resource "azurerm_monitor_autoscale_setting" "main" {
  name                = "autoscale-prod"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  target_resource_id  = azurerm_service_plan.main.id

  profile {
    name = "AutoScale"

    capacity {
      default = 2
      minimum = 2
      maximum = 10
    }

    rule {
      metric_trigger {
        metric_name        = "CpuPercentage"
        metric_resource_id = azurerm_service_plan.main.id
        time_grain         = "PT1M"
        statistic          = "Average"
        time_window        = "PT5M"
        time_aggregation   = "Average"
        operator           = "GreaterThan"
        threshold          = 70
      }

      scale_action {
        direction = "Increase"
        type      = "ChangeCount"
        value     = "2"
        cooldown  = "PT5M"
      }
    }

    rule {
      metric_trigger {
        metric_name        = "CpuPercentage"
        metric_resource_id = azurerm_service_plan.main.id
        time_grain         = "PT1M"
        statistic          = "Average"
        time_window        = "PT5M"
        time_aggregation   = "Average"
        operator           = "LessThan"
        threshold          = 30
      }

      scale_action {
        direction = "Decrease"
        type      = "ChangeCount"
        value     = "1"
        cooldown  = "PT5M"
      }
    }
  }
}
```

---

## ⚙️ Configuration Management

### **Application Settings (Environment Variables):**

```bash
# Set multiple settings
az webapp config appsettings set \
  --resource-group rg-webapps \
  --name webapp-nodejs-prod-001 \
  --settings \
    NODE_ENV=production \
    API_KEY=@Microsoft.KeyVault(SecretUri=https://kv-prod.vault.azure.net/secrets/api-key/) \
    DATABASE_HOST=sql-server.database.windows.net \
    REDIS_HOST=redis-cache.redis.cache.windows.net

# List all settings
az webapp config appsettings list \
  --resource-group rg-webapps \
  --name webapp-nodejs-prod-001 \
  --output table

# Delete setting
az webapp config appsettings delete \
  --resource-group rg-webapps \
  --name webapp-nodejs-prod-001 \
  --setting-names API_KEY
```

**Accessing in Node.js:**
```javascript
const dbHost = process.env.DATABASE_HOST;
const apiKey = process.env.API_KEY;
```

---

## 📊 Monitoring and Logging

### **Enable Application Insights:**

```bash
# Create Application Insights
az monitor app-insights component create \
  --resource-group rg-webapps \
  --app ai-webapp \
  --location eastus \
  --application-type web

# Get instrumentation key
AI_KEY=$(az monitor app-insights component show \
  --resource-group rg-webapps \
  --app ai-webapp \
  --query instrumentationKey \
  --output tsv)

# Configure web app
az webapp config appsettings set \
  --resource-group rg-webapps \
  --name webapp-nodejs-prod-001 \
  --settings APPINSIGHTS_INSTRUMENTATIONKEY=$AI_KEY
```

### **Enable Detailed Logging:**

```bash
# Enable application logging
az webapp log config \
  --resource-group rg-webapps \
  --name webapp-nodejs-prod-001 \
  --application-logging azureblobstorage \
  --level verbose \
  --web-server-logging filesystem \
  --docker-container-logging filesystem

# Stream live logs
az webapp log tail \
  --resource-group rg-webapps \
  --name webapp-nodejs-prod-001

# Download logs
az webapp log download \
  --resource-group rg-webapps \
  --name webapp-nodejs-prod-001 \
  --log-file webapp-logs.zip
```

---

## 🛠️ Troubleshooting

### **Common Issues:**

**1. Application Won't Start**

**Symptoms:** HTTP 500 errors, Application Error page

**Debugging Steps:**
```bash
# Check application logs
az webapp log tail --resource-group rg-webapps --name webapp-nodejs-prod-001

# SSH into container (Linux)
az webapp ssh --resource-group rg-webapps --name webapp-nodejs-prod-001

# Check process status
ps aux | grep node

# Check environment variables
env | grep NODE
```

**Common Causes:**
- Missing dependencies (check package.json)
- Incorrect startup command
- Port binding issues (must listen on process.env.PORT)

**Fix - Set Startup Command:**
```bash
az webapp config set \
  --resource-group rg-webapps \
  --name webapp-nodejs-prod-001 \
  --startup-file "node server.js"
```

**2. Slow Performance**

**Diagnosis:**
```bash
# Check metrics
az monitor metrics list \
  --resource $(az webapp show --resource-group rg-webapps --name webapp-nodejs-prod-001 --query id -o tsv) \
  --metric "CpuPercentage" "MemoryPercentage" "HttpResponseTime"
```

**Solutions:**
- Enable auto-scaling
- Upgrade to higher tier (more CPU/RAM)
- Optimize application code
- Enable caching (Redis, CDN)

---

## 🔒 Security Best Practices

**1. Use Managed Identity**
```bash
az webapp identity assign \
  --resource-group rg-webapps \
  --name webapp-nodejs-prod-001
```

**2. Restrict Access with VNet Integration**
```bash
az webapp vnet-integration add \
  --resource-group rg-webapps \
  --name webapp-nodejs-prod-001 \
  --vnet myVNet \
  --subnet webapp-subnet
```

**3. IP Restrictions**
```bash
az webapp config access-restriction add \
  --resource-group rg-webapps \
  --name webapp-nodejs-prod-001 \
  --rule-name AllowOfficeIP \
  --action Allow \
  --ip-address 203.0.113.0/24 \
  --priority 100
```

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)
