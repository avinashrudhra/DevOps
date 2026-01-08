# Azure Functions - Complete Learning Guide

**Serverless Computing: Build Event-Driven Applications at Scale**

---

## 📚 Table of Contents

1. [What is Azure Functions?](#what-is-azure-functions)
2. [Function App Hosting Plans](#hosting-plans)
3. [Triggers and Bindings](#triggers-and-bindings)
4. [Creating Your First Function](#creating-first-function)
5. [HTTP Triggered Functions](#http-triggered-functions)
6. [Timer Triggered Functions](#timer-triggered-functions)
7. [Queue and Blob Triggers](#queue-blob-triggers)
8. [Deployment Methods](#deployment-methods)
9. [Configuration and Settings](#configuration-settings)
10. [Scaling and Performance](#scaling-performance)
11. [Monitoring and Troubleshooting](#monitoring-troubleshooting)
12. [Security Best Practices](#security-best-practices)
13. [Real-World Scenarios](#real-world-scenarios)

---

## 🎯 What is Azure Functions?

**Azure Functions** is a serverless compute service that enables you to run event-driven code without explicitly provisioning or managing infrastructure.

### **Key Concepts:**

**Serverless Benefits:**
- **Pay-per-execution** - Only charged for actual compute time used
- **Auto-scaling** - Automatically scales based on demand
- **No infrastructure management** - Focus on code, not servers
- **Event-driven** - Responds to triggers from various sources
- **Multi-language support** - C#, JavaScript, Python, Java, PowerShell, TypeScript

**When to Use Azure Functions:**
- ✅ Processing file uploads or changes
- ✅ Scheduled tasks (cron jobs)
- ✅ HTTP-based APIs and webhooks
- ✅ Message queue processing
- ✅ Real-time data stream processing
- ✅ IoT data processing
- ✅ Connecting and orchestrating services

**When NOT to Use:**
- ❌ Long-running processes (>10 minutes for Consumption plan)
- ❌ CPU-intensive workloads requiring sustained compute
- ❌ Applications requiring full control over environment
- ❌ Complex stateful applications (without Durable Functions)

---

## 🏗️ Hosting Plans

### **1. Consumption Plan (Serverless)**

**Best for:** Variable workloads, infrequent execution, cost optimization

**Characteristics:**
- Automatic scaling (up to 200 instances)
- Timeout: 5 minutes (default), up to 10 minutes (maximum)
- Memory: 1.5 GB per instance
- Billing: Per execution + execution time
- Cold start: Yes (0-10 seconds typical)

**Pricing Example:**
```
First 1M executions: Free
After: $0.20 per million executions
Execution time: $0.000016/GB-s
```

**CLI Command:**
```bash
# Create Consumption plan Function App
az functionapp create \
  --resource-group rg-functions \
  --name func-consumption-app \
  --storage-account stfuncapp001 \
  --consumption-plan-location eastus \
  --runtime python \
  --runtime-version 3.11 \
  --functions-version 4 \
  --os-type Linux
```

---

### **2. Premium Plan (Elastic Premium)**

**Best for:** Enterprise applications, VNet integration, no cold starts

**Characteristics:**
- Pre-warmed workers (no cold start)
- Unlimited execution duration
- VNet connectivity
- More powerful instances (up to 14 GB RAM)
- Minimum 1 always-on instance

**CLI Command:**
```bash
# Create Premium plan
az functionapp plan create \
  --resource-group rg-functions \
  --name func-plan-premium \
  --location eastus \
  --sku EP1 \
  --is-linux true \
  --min-instances 1 \
  --max-burst 20

# Create Function App on Premium plan
az functionapp create \
  --resource-group rg-functions \
  --name func-premium-app \
  --plan func-plan-premium \
  --storage-account stfuncapp001 \
  --runtime node \
  --runtime-version 18 \
  --functions-version 4
```

---

### **3. Dedicated (App Service) Plan**

**Best for:** Existing App Service Plan, predictable costs

**Characteristics:**
- Runs on existing App Service Plan VMs
- Fixed cost (not consumption-based)
- No auto-scaling limits
- Full VM control

```bash
# Create App Service Plan
az appservice plan create \
  --resource-group rg-functions \
  --name asp-functions \
  --location eastus \
  --sku B1 \
  --is-linux true

# Create Function App on App Service Plan
az functionapp create \
  --resource-group rg-functions \
  --name func-dedicated-app \
  --plan asp-functions \
  --storage-account stfuncapp001 \
  --runtime dotnet \
  --functions-version 4
```

---

## 🔌 Triggers and Bindings

### **Understanding Triggers vs Bindings:**

- **Trigger** - Event that invokes the function (every function has exactly ONE trigger)
- **Input Binding** - Data read by the function
- **Output Binding** - Data written by the function

### **Available Triggers:**

| Trigger | Use Case | Example |
|---------|----------|---------|
| **HTTP** | REST APIs, webhooks | API endpoint |
| **Timer** | Scheduled tasks | Daily report generation |
| **Queue** | Async processing | Order processing |
| **Blob** | File uploads | Image resizing |
| **Event Grid** | Event-driven architecture | Resource changes |
| **Cosmos DB** | Database changes | Data synchronization |
| **Service Bus** | Enterprise messaging | Payment processing |
| **Event Hub** | IoT data streams | Telemetry processing |

---

## 🚀 Creating Your First Function

### **Prerequisites:**

```bash
# Install Azure Functions Core Tools
npm install -g azure-functions-core-tools@4 --unsafe-perm true

# Or using chocolatey (Windows)
choco install azure-functions-core-tools-4

# Verify installation
func --version
```

### **Step 1: Create Storage Account**

Every Function App requires a storage account for state management and logging.

```bash
# Create storage account
az storage account create \
  --name stfuncapp001 \
  --resource-group rg-functions \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2

# Get connection string
STORAGE_CONNECTION=$(az storage account show-connection-string \
  --resource-group rg-functions \
  --name stfuncapp001 \
  --query connectionString \
  --output tsv)
```

### **Step 2: Create Function App**

```bash
# Create Function App
az functionapp create \
  --resource-group rg-functions \
  --name func-myapp-prod \
  --storage-account stfuncapp001 \
  --consumption-plan-location eastus \
  --runtime python \
  --runtime-version 3.11 \
  --functions-version 4 \
  --os-type Linux
```

### **Step 3: Create Local Function Project**

```bash
# Create new project directory
mkdir my-function-app
cd my-function-app

# Initialize function project (Python)
func init --python

# Create HTTP trigger function
func new --name HttpTriggerExample --template "HTTP trigger"

# Test locally
func start

# Test endpoint
curl http://localhost:7071/api/HttpTriggerExample?name=Azure
```

---

## 🌐 HTTP Triggered Functions

### **Complete HTTP Function Example (Python):**

**`HttpTriggerExample/__init__.py`**

```python
import azure.functions as func
import logging
import json

def main(req: func.HttpRequest) -> func.HttpResponse:
    """
    HTTP Trigger function that processes GET and POST requests
    
    GET: Returns greeting with name from query parameter
    POST: Processes JSON body and returns processed data
    """
    logging.info('HTTP trigger function processing request.')
    
    # Get HTTP method
    method = req.method
    logging.info(f'HTTP Method: {method}')
    
    # Handle GET request
    if method == 'GET':
        name = req.params.get('name')
        if name:
            return func.HttpResponse(
                json.dumps({
                    "message": f"Hello, {name}!",
                    "method": "GET"
                }),
                mimetype="application/json",
                status_code=200
            )
        else:
            return func.HttpResponse(
                json.dumps({"error": "Please pass a name on the query string"}),
                mimetype="application/json",
                status_code=400
            )
    
    # Handle POST request
    elif method == 'POST':
        try:
            req_body = req.get_json()
            name = req_body.get('name')
            email = req_body.get('email')
            
            if not name or not email:
                return func.HttpResponse(
                    json.dumps({"error": "Missing required fields: name and email"}),
                    mimetype="application/json",
                    status_code=400
                )
            
            # Process data (example: store, send email, etc.)
            response_data = {
                "message": f"User {name} registered successfully",
                "data": {
                    "name": name,
                    "email": email,
                    "status": "registered"
                }
            }
            
            return func.HttpResponse(
                json.dumps(response_data),
                mimetype="application/json",
                status_code=201
            )
            
        except ValueError:
            return func.HttpResponse(
                json.dumps({"error": "Invalid JSON in request body"}),
                mimetype="application/json",
                status_code=400
            )
    
    # Handle other methods
    else:
        return func.HttpResponse(
            json.dumps({"error": f"Method {method} not supported"}),
            mimetype="application/json",
            status_code=405
        )
```

**`HttpTriggerExample/function.json`**

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "authLevel": "function",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "$return"
    }
  ]
}
```

**Authorization Levels:**
- `anonymous` - No key required
- `function` - Function-specific key required
- `admin` - Master key required

---

## ⏰ Timer Triggered Functions

### **CRON Expression Basics:**

```
{second} {minute} {hour} {day} {month} {day-of-week}

Examples:
0 */5 * * * *     - Every 5 minutes
0 0 * * * *       - Every hour
0 0 9 * * *       - Every day at 9 AM UTC
0 30 9 * * 1-5    - 9:30 AM UTC Mon-Fri
0 0 0 1 * *       - First day of month at midnight
```

### **Complete Timer Function Example:**

**`DailyReportGenerator/__init__.py`**

```python
import azure.functions as func
import logging
from datetime import datetime, timedelta
import json

def main(mytimer: func.TimerRequest, reportBlob: func.Out[str]) -> None:
    """
    Timer trigger function that runs daily at 9 AM UTC
    Generates a daily report and stores it in Blob Storage
    """
    utc_timestamp = datetime.utcnow().isoformat()
    
    # Check if timer is running late
    if mytimer.past_due:
        logging.warning('The timer is past due!')
    
    logging.info(f'Daily report generator started at: {utc_timestamp}')
    
    # Generate report data (example)
    report_data = {
        "report_date": datetime.utcnow().strftime('%Y-%m-%d'),
        "generated_at": utc_timestamp,
        "metrics": {
            "total_users": 1250,
            "active_users": 875,
            "new_registrations": 45,
            "revenue": 12500.50
        },
        "summary": "Daily operations report generated successfully"
    }
    
    # Write to blob storage via output binding
    reportBlob.set(json.dumps(report_data, indent=2))
    
    logging.info(f'Report generated and saved to blob storage')
```

**`DailyReportGenerator/function.json`**

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "name": "mytimer",
      "type": "timerTrigger",
      "direction": "in",
      "schedule": "0 0 9 * * *"
    },
    {
      "name": "reportBlob",
      "type": "blob",
      "direction": "out",
      "path": "reports/daily-{DateTime}.json",
      "connection": "AzureWebJobsStorage"
    }
  ]
}
```

---

## 📦 Queue and Blob Triggers

### **Queue Trigger - Order Processing Example:**

**`ProcessOrder/__init__.py`**

```python
import azure.functions as func
import logging
import json

def main(msg: func.QueueMessage, orderBlob: func.Out[str]) -> None:
    """
    Queue trigger function that processes orders
    Triggered when a message is added to the 'orders' queue
    """
    logging.info('Processing order from queue')
    
    # Get message content
    order_data = json.loads(msg.get_body().decode('utf-8'))
    order_id = order_data.get('order_id')
    customer_name = order_data.get('customer_name')
    total_amount = order_data.get('total_amount')
    
    logging.info(f'Order ID: {order_id}, Customer: {customer_name}, Amount: ${total_amount}')
    
    # Process order (validation, payment, etc.)
    processed_order = {
        "order_id": order_id,
        "customer_name": customer_name,
        "total_amount": total_amount,
        "status": "processed",
        "processed_at": func.datetime.datetime.utcnow().isoformat()
    }
    
    # Store processed order in blob
    orderBlob.set(json.dumps(processed_order, indent=2))
    
    logging.info(f'Order {order_id} processed successfully')
```

**`ProcessOrder/function.json`**

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "name": "msg",
      "type": "queueTrigger",
      "direction": "in",
      "queueName": "orders",
      "connection": "AzureWebJobsStorage"
    },
    {
      "name": "orderBlob",
      "type": "blob",
      "direction": "out",
      "path": "processed-orders/{rand-guid}.json",
      "connection": "AzureWebJobsStorage"
    }
  ]
}
```

### **Blob Trigger - Image Processing Example:**

```python
import azure.functions as func
import logging
from PIL import Image
from io import BytesIO

def main(myblob: func.InputStream, thumbnailBlob: func.Out[bytes]):
    """
    Blob trigger function that creates thumbnails of uploaded images
    Triggered when a new blob is uploaded to 'images' container
    """
    logging.info(f"Processing blob: {myblob.name}")
    logging.info(f"Blob size: {myblob.length} bytes")
    
    # Read image
    image_data = myblob.read()
    image = Image.open(BytesIO(image_data))
    
    # Create thumbnail (200x200)
    image.thumbnail((200, 200), Image.LANCZOS)
    
    # Save to output binding
    output = BytesIO()
    image.save(output, format=image.format)
    thumbnailBlob.set(output.getvalue())
    
    logging.info(f"Thumbnail created for {myblob.name}")
```

---

## 📤 Deployment Methods

### **Method 1: Deploy from Local (Azure Functions Core Tools)**

```bash
# Deploy to Azure
func azure functionapp publish func-myapp-prod

# Deploy with specific build
func azure functionapp publish func-myapp-prod --build remote

# Publish and sync triggers
func azure functionapp publish func-myapp-prod --force
```

### **Method 2: GitHub Actions CI/CD**

**`.github/workflows/deploy-functions.yml`**

```yaml
name: Deploy Azure Functions

on:
  push:
    branches: [ main ]
  workflow_dispatch:

env:
  AZURE_FUNCTIONAPP_NAME: func-myapp-prod
  AZURE_FUNCTIONAPP_PACKAGE_PATH: '.'
  PYTHON_VERSION: '3.11'

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    - name: 'Checkout Code'
      uses: actions/checkout@v4

    - name: Setup Python
      uses: actions/setup-python@v4
      with:
        python-version: ${{ env.PYTHON_VERSION }}

    - name: 'Install Dependencies'
      run: |
        pushd './${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}'
        python -m pip install --upgrade pip
        pip install -r requirements.txt --target=".python_packages/lib/site-packages"
        popd

    - name: 'Deploy to Azure Functions'
      uses: Azure/functions-action@v1
      with:
        app-name: ${{ env.AZURE_FUNCTIONAPP_NAME }}
        package: ${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}
        publish-profile: ${{ secrets.AZURE_FUNCTIONAPP_PUBLISH_PROFILE }}
        scm-do-build-during-deployment: true
        enable-oryx-build: true
```

### **Method 3: Terraform Deployment**

```hcl
resource "azurerm_storage_account" "functions" {
  name                     = "stfuncapp001"
  resource_group_name      = azurerm_resource_group.main.name
  location                 = azurerm_resource_group.main.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
}

resource "azurerm_service_plan" "functions" {
  name                = "func-plan-prod"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  os_type             = "Linux"
  sku_name            = "Y1"  # Consumption plan
}

resource "azurerm_linux_function_app" "main" {
  name                = "func-myapp-prod"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  
  storage_account_name       = azurerm_storage_account.functions.name
  storage_account_access_key = azurerm_storage_account.functions.primary_access_key
  service_plan_id            = azurerm_service_plan.functions.id
  
  site_config {
    application_stack {
      python_version = "3.11"
    }
    
    application_insights_key               = azurerm_application_insights.main.instrumentation_key
    application_insights_connection_string = azurerm_application_insights.main.connection_string
  }
  
  app_settings = {
    "FUNCTIONS_WORKER_RUNTIME"        = "python"
    "AzureWebJobsStorage"             = azurerm_storage_account.functions.primary_connection_string
    "WEBSITE_RUN_FROM_PACKAGE"        = "1"
    "ENABLE_ORYX_BUILD"               = "true"
    "SCM_DO_BUILD_DURING_DEPLOYMENT"  = "true"
  }
  
  identity {
    type = "SystemAssigned"
  }
}

resource "azurerm_application_insights" "main" {
  name                = "ai-functions"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  application_type    = "other"
}
```

---

## ⚙️ Configuration and Settings

### **Application Settings (Environment Variables)**

```bash
# Set application settings
az functionapp config appsettings set \
  --resource-group rg-functions \
  --name func-myapp-prod \
  --settings \
    "DATABASE_CONNECTION=@Microsoft.KeyVault(SecretUri=https://kv.vault.azure.net/secrets/db-conn/)" \
    "API_KEY=your-api-key" \
    "ENVIRONMENT=production"

# List all settings
az functionapp config appsettings list \
  --resource-group rg-functions \
  --name func-myapp-prod \
  --output table

# Delete setting
az functionapp config appsettings delete \
  --resource-group rg-functions \
  --name func-myapp-prod \
  --setting-names "API_KEY"
```

### **Using Key Vault References**

```bash
# Enable managed identity
az functionapp identity assign \
  --resource-group rg-functions \
  --name func-myapp-prod

# Get identity principal ID
IDENTITY_ID=$(az functionapp identity show \
  --resource-group rg-functions \
  --name func-myapp-prod \
  --query principalId \
  --output tsv)

# Grant Key Vault access
az keyvault set-policy \
  --name kv-prod-secrets \
  --object-id $IDENTITY_ID \
  --secret-permissions get list
```

---

## 📈 Scaling and Performance

### **Consumption Plan Scaling:**

**How it works:**
1. Azure monitors queue length, CPU, memory
2. Adds instances when load increases
3. Removes instances when idle
4. Maximum 200 instances
5. Each instance processes one function execution at a time

**Configuration:**

```json
{
  "functionTimeout": "00:05:00",
  "version": "2.0",
  "extensions": {
    "queues": {
      "maxPollingInterval": "00:00:02",
      "visibilityTimeout": "00:00:30",
      "batchSize": 16,
      "maxDequeueCount": 5,
      "newBatchThreshold": 8
    },
    "http": {
      "routePrefix": "api",
      "maxOutstandingRequests": 200,
      "maxConcurrentRequests": 100
    }
  }
}
```

### **Premium Plan Scaling:**

```bash
# Configure Premium plan scaling
az functionapp plan update \
  --resource-group rg-functions \
  --name func-plan-premium \
  --min-instances 2 \
  --max-burst 20
```

---

## 🔍 Monitoring and Troubleshooting

### **Enable Application Insights:**

```bash
# Create App Insights
az monitor app-insights component create \
  --resource-group rg-functions \
  --app ai-functions \
  --location eastus \
  --application-type other

# Get instrumentation key
AI_KEY=$(az monitor app-insights component show \
  --resource-group rg-functions \
  --app ai-functions \
  --query instrumentationKey \
  --output tsv)

# Configure Function App
az functionapp config appsettings set \
  --resource-group rg-functions \
  --name func-myapp-prod \
  --settings APPINSIGHTS_INSTRUMENTATIONKEY=$AI_KEY
```

### **Common Issues and Solutions:**

**1. Cold Start Issues**

**Problem:** First request takes 5-10 seconds
**Solution:**
- Use Premium plan for pre-warmed instances
- Keep functions warm with scheduled pings
- Minimize dependencies

```python
# Keep-alive function
def main(mytimer: func.TimerRequest) -> None:
    logging.info('Keep-alive ping executed')
    # Schedule: 0 */5 * * * * (every 5 minutes)
```

**2. Timeout Errors**

**Problem:** Function execution exceeds timeout
**Solution:**
- Increase timeout in `host.json`
- Break into smaller functions
- Use Durable Functions for long-running tasks

```json
{
  "version": "2.0",
  "functionTimeout": "00:10:00"
}
```

**3. Memory Issues**

**Problem:** OutOfMemoryException
**Solution:**
- Optimize code to use less memory
- Process data in batches
- Use Premium plan for more memory

---

## 🔒 Security Best Practices

**1. Use Managed Identity**
```bash
# Enable system-assigned managed identity
az functionapp identity assign \
  --resource-group rg-functions \
  --name func-myapp-prod
```

**2. Restrict Network Access**
```bash
# Add VNet integration (Premium plan)
az functionapp vnet-integration add \
  --resource-group rg-functions \
  --name func-myapp-prod \
  --vnet myVNet \
  --subnet functions-subnet
```

**3. Secure Application Settings**
- Store secrets in Key Vault
- Use Key Vault references
- Never hardcode credentials

---

## 🌟 Real-World Scenarios

### **Scenario 1: Image Resizing API**
HTTP trigger → Resize image → Store in Blob → Return URL

### **Scenario 2: Order Processing Pipeline**
Queue trigger → Validate → Process payment → Send confirmation

### **Scenario 3: Daily Report Generation**
Timer trigger → Query database → Generate report → Email to team

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)
