# Azure Monitoring - Complete Learning Guide

**Azure Monitor, Log Analytics, Application Insights, Alerts & Dashboards**

---

## 📚 Table of Contents

1. [Azure Monitoring Overview](#monitoring-overview)
2. [Log Analytics Workspace](#log-analytics-workspace)
3. [Kusto Query Language (KQL)](#kusto-query-language)
4. [Application Insights](#application-insights)
5. [Metrics and Alerts](#metrics-alerts)
6. [Action Groups](#action-groups)
7. [Workbooks and Dashboards](#workbooks-dashboards)
8. [Monitoring Best Practices](#best-practices)
9. [Troubleshooting Scenarios](#troubleshooting-scenarios)

---

## 🎯 Azure Monitoring Overview

**Azure Monitor** is the unified monitoring platform for all Azure resources.

### **Components:**

```
┌─────────────────────────────────────────┐
│         Azure Monitor                    │
├─────────────────────────────────────────┤
│  Metrics  │  Logs  │  Traces  │  Events │
└─────────────────────────────────────────┘
        ↓           ↓
┌──────────┐  ┌─────────────────┐
│ Metrics  │  │ Log Analytics   │
│ Explorer │  │ Workspace       │
└──────────┘  └─────────────────┘
        ↓           ↓
┌──────────┐  ┌─────────────────┐
│ Alerts   │  │ Workbooks       │
└──────────┘  └─────────────────┘
        ↓
┌────────────────────┐
│  Action Groups     │
│  (Email, SMS, etc) │
└────────────────────┘
```

---

## 📊 Log Analytics Workspace

**Log Analytics** centralizes log data from all Azure resources.

### **Create Workspace:**

```bash
# Create Log Analytics workspace
az monitor log-analytics workspace create \
  --resource-group rg-monitoring \
  --workspace-name law-prod \
  --location eastus \
  --sku PerGB2018 \
  --retention-time 30

# Get workspace ID
WORKSPACE_ID=$(az monitor log-analytics workspace show \
  --resource-group rg-monitoring \
  --workspace-name law-prod \
  --query customerId \
  --output tsv)

# Get workspace key
WORKSPACE_KEY=$(az monitor log-analytics workspace get-shared-keys \
  --resource-group rg-monitoring \
  --workspace-name law-prod \
  --query primarySharedKey \
  --output tsv)

echo "Workspace ID: $WORKSPACE_ID"
echo "Workspace Key: $WORKSPACE_KEY"
```

---

### **Enable Diagnostics on Resources:**

**Virtual Machine:**
```bash
# Enable diagnostics on VM
az monitor diagnostic-settings create \
  --resource $(az vm show --resource-group rg-compute --name vm-web-01 --query id --output tsv) \
  --name vm-diagnostics \
  --workspace $(az monitor log-analytics workspace show --resource-group rg-monitoring --workspace-name law-prod --query id --output tsv) \
  --metrics '[{"category": "AllMetrics","enabled": true}]'
```

**Storage Account:**
```bash
# Enable diagnostics on storage account
az monitor diagnostic-settings create \
  --resource $(az storage account show --resource-group rg-storage --name stproddata001 --query id --output tsv) \
  --name storage-diagnostics \
  --workspace $(az monitor log-analytics workspace show --resource-group rg-monitoring --workspace-name law-prod --query id --output tsv) \
  --logs '[{
    "category": "StorageRead",
    "enabled": true
  },{
    "category": "StorageWrite",
    "enabled": true
  },{
    "category": "StorageDelete",
    "enabled": true
  }]' \
  --metrics '[{"category": "Transaction","enabled": true}]'
```

**App Service:**
```bash
# Enable diagnostics on web app
az monitor diagnostic-settings create \
  --resource $(az webapp show --resource-group rg-webapps --name webapp-prod-001 --query id --output tsv) \
  --name webapp-diagnostics \
  --workspace $(az monitor log-analytics workspace show --resource-group rg-monitoring --workspace-name law-prod --query id --output tsv) \
  --logs '[{
    "category": "AppServiceHTTPLogs",
    "enabled": true
  },{
    "category": "AppServiceConsoleLogs",
    "enabled": true
  },{
    "category": "AppServiceAppLogs",
    "enabled": true
  }]'
```

---

## 🔍 Kusto Query Language (KQL)

**KQL** is the query language for Log Analytics.

### **Basic Query Structure:**

```kql
TableName
| where condition
| project column1, column2
| summarize aggregation by column
| order by column desc
| take 10
```

---

### **Essential KQL Queries:**

**1. Recent Activities:**
```kql
AzureActivity
| where TimeGenerated > ago(1h)
| where OperationNameValue contains "Write"
| project TimeGenerated, Caller, OperationNameValue, ResourceGroup
| order by TimeGenerated desc
```

**2. Failed Requests:**
```kql
AppRequests
| where Success == false
| where TimeGenerated > ago(24h)
| summarize FailedRequests = count() by ResultCode, bin(TimeGenerated, 1h)
| order by TimeGenerated desc
```

**3. VM CPU Usage:**
```kql
Perf
| where ObjectName == "Processor" and CounterName == "% Processor Time"
| where Computer == "vm-web-01"
| where TimeGenerated > ago(1h)
| summarize AvgCPU = avg(CounterValue) by bin(TimeGenerated, 5m)
| render timechart
```

**4. Storage Account Requests:**
```kql
StorageBlobLogs
| where TimeGenerated > ago(1d)
| summarize RequestCount = count(), AvgDuration = avg(DurationMs) by OperationName
| order by RequestCount desc
```

**5. Web App Errors:**
```kql
AppServiceConsoleLogs
| where ResultDescription contains "error" or ResultDescription contains "exception"
| where TimeGenerated > ago(1h)
| project TimeGenerated, ResultDescription, _ResourceId
| order by TimeGenerated desc
```

**6. Security Events:**
```kql
SecurityEvent
| where EventID == 4625  // Failed login
| where TimeGenerated > ago(24h)
| summarize FailedLogins = count() by Account, Computer
| where FailedLogins > 5
| order by FailedLogins desc
```

**7. Container Insights:**
```kql
ContainerLog
| where LogEntry contains "error"
| where TimeGenerated > ago(1h)
| project TimeGenerated, LogEntry, ContainerID
| order by TimeGenerated desc
```

**8. Top Resource Consumers:**
```kql
AzureMetrics
| where TimeGenerated > ago(1h)
| where MetricName == "Percentage CPU"
| summarize AvgCPU = avg(Average) by Resource
| where AvgCPU > 80
| order by AvgCPU desc
```

---

### **Advanced KQL Techniques:**

**Join Tables:**
```kql
AppRequests
| where TimeGenerated > ago(1h)
| join kind=inner (
    AppExceptions
    | where TimeGenerated > ago(1h)
) on OperationId
| project TimeGenerated, Url, ExceptionType, ExceptionMessage
```

**Multiple Conditions:**
```kql
AzureActivity
| where TimeGenerated > ago(7d)
| where (CategoryValue == "Administrative" and OperationNameValue contains "Delete") 
    or (CategoryValue == "Security" and Level == "Critical")
| project TimeGenerated, Caller, OperationNameValue, ResourceGroup, ActivityStatusValue
```

**Percentiles:**
```kql
AppRequests
| where TimeGenerated > ago(24h)
| summarize 
    p50 = percentile(DurationMs, 50),
    p95 = percentile(DurationMs, 95),
    p99 = percentile(DurationMs, 99)
by bin(TimeGenerated, 1h)
| render timechart
```

---

## 📱 Application Insights

**Application Insights** provides APM (Application Performance Monitoring) for your applications.

### **Create Application Insights:**

```bash
# Create Application Insights
az monitor app-insights component create \
  --resource-group rg-monitoring \
  --app ai-webapp-prod \
  --location eastus \
  --application-type web \
  --workspace $(az monitor log-analytics workspace show --resource-group rg-monitoring --workspace-name law-prod --query id --output tsv)

# Get instrumentation key
AI_KEY=$(az monitor app-insights component show \
  --resource-group rg-monitoring \
  --app ai-webapp-prod \
  --query instrumentationKey \
  --output tsv)

# Get connection string
AI_CONNECTION=$(az monitor app-insights component show \
  --resource-group rg-monitoring \
  --app ai-webapp-prod \
  --query connectionString \
  --output tsv)

echo "Instrumentation Key: $AI_KEY"
echo "Connection String: $AI_CONNECTION"
```

---

### **Configure Web App:**

```bash
# Add App Insights to web app
az webapp config appsettings set \
  --resource-group rg-webapps \
  --name webapp-prod-001 \
  --settings \
    APPLICATIONINSIGHTS_CONNECTION_STRING=$AI_CONNECTION \
    APPINSIGHTS_INSTRUMENTATIONKEY=$AI_KEY \
    ApplicationInsightsAgent_EXTENSION_VERSION=~3
```

---

### **Application Insights in Code:**

**Node.js:**
```javascript
const appInsights = require('applicationinsights');

appInsights.setup(process.env.APPLICATIONINSIGHTS_CONNECTION_STRING)
  .setAutoDependencyCorrelation(true)
  .setAutoCollectRequests(true)
  .setAutoCollectPerformance(true, true)
  .setAutoCollectExceptions(true)
  .setAutoCollectDependencies(true)
  .setAutoCollectConsole(true)
  .setUseDiskRetryCaching(true)
  .setSendLiveMetrics(true)
  .start();

const client = appInsights.defaultClient;

// Track custom event
client.trackEvent({name: "UserLogin", properties: {userId: "user123"}});

// Track custom metric
client.trackMetric({name: "ActiveUsers", value: 150});

// Track trace
client.trackTrace({message: "Processing order", severity: appInsights.Contracts.SeverityLevel.Information});
```

**Python:**
```python
from applicationinsights import TelemetryClient
import os

# Initialize
tc = TelemetryClient(os.environ['APPINSIGHTS_INSTRUMENTATIONKEY'])

# Track event
tc.track_event('UserLogin', {'userId': 'user123'})

# Track metric
tc.track_metric('ActiveUsers', 150)

# Track exception
try:
    result = 10 / 0
except Exception as e:
    tc.track_exception()

# Flush data
tc.flush()
```

---

### **Application Insights Queries:**

**1. Request Performance:**
```kql
requests
| where timestamp > ago(24h)
| summarize 
    TotalRequests = count(),
    AvgDuration = avg(duration),
    P95Duration = percentile(duration, 95),
    FailureRate = countif(success == false) * 100.0 / count()
by bin(timestamp, 1h)
| render timechart
```

**2. Failed Requests:**
```kql
requests
| where success == false
| where timestamp > ago(1h)
| project timestamp, url, resultCode, duration
| order by timestamp desc
```

**3. Exceptions:**
```kql
exceptions
| where timestamp > ago(24h)
| summarize ExceptionCount = count() by type, outerMessage
| order by ExceptionCount desc
```

**4. Dependencies (API Calls):**
```kql
dependencies
| where timestamp > ago(1h)
| summarize 
    CallCount = count(),
    AvgDuration = avg(duration),
    FailureRate = countif(success == false) * 100.0 / count()
by name, type
| order by CallCount desc
```

**5. Custom Events:**
```kql
customEvents
| where name == "UserLogin"
| where timestamp > ago(24h)
| summarize LoginCount = count() by tostring(customDimensions.userId)
| order by LoginCount desc
| take 10
```

---

## 🚨 Metrics and Alerts

### **Create Metric Alert:**

```bash
# Create alert for high CPU
az monitor metrics alert create \
  --resource-group rg-monitoring \
  --name alert-high-cpu \
  --scopes $(az vm show --resource-group rg-compute --name vm-web-01 --query id --output tsv) \
  --condition "avg Percentage CPU > 80" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --action $(az monitor action-group show --resource-group rg-monitoring --name ag-devops --query id --output tsv) \
  --description "Alert when CPU exceeds 80%"

# Create alert for failed requests
az monitor metrics alert create \
  --resource-group rg-monitoring \
  --name alert-failed-requests \
  --scopes $(az monitor app-insights component show --resource-group rg-monitoring --app ai-webapp-prod --query id --output tsv) \
  --condition "count requests/failed > 10" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --action $(az monitor action-group show --resource-group rg-monitoring --name ag-devops --query id --output tsv)

# List alerts
az monitor metrics alert list \
  --resource-group rg-monitoring \
  --output table

# Disable alert
az monitor metrics alert update \
  --resource-group rg-monitoring \
  --name alert-high-cpu \
  --enabled false
```

---

### **Create Log Query Alert:**

```bash
# Create alert based on KQL query
az monitor scheduled-query create \
  --resource-group rg-monitoring \
  --name alert-failed-logins \
  --scopes $(az monitor log-analytics workspace show --resource-group rg-monitoring --workspace-name law-prod --query id --output tsv) \
  --condition "count 'Heartbeat | where TimeGenerated > ago(5m) | where Computer == \"vm-web-01\"' < 1" \
  --condition-query "Heartbeat | where TimeGenerated > ago(5m) | where Computer == 'vm-web-01' | summarize Count = count()" \
  --evaluation-frequency 5m \
  --window-size 5m \
  --action $(az monitor action-group show --resource-group rg-monitoring --name ag-devops --query id --output tsv) \
  --description "Alert when VM stops sending heartbeat"
```

---

## 📧 Action Groups

**Action Groups** define what happens when an alert fires.

```bash
# Create action group with email
az monitor action-group create \
  --resource-group rg-monitoring \
  --name ag-devops \
  --short-name DevOps

# Add email receiver
az monitor action-group update \
  --resource-group rg-monitoring \
  --name ag-devops \
  --add-action email devops-team devops@company.com

# Add SMS receiver
az monitor action-group update \
  --resource-group rg-monitoring \
  --name ag-devops \
  --add-action sms oncall +1234567890

# Add webhook
az monitor action-group update \
  --resource-group rg-monitoring \
  --name ag-devops \
  --add-action webhook slack https://hooks.slack.com/services/xxx

# Add Azure Function
az monitor action-group update \
  --resource-group rg-monitoring \
  --name ag-devops \
  --add-action azurefunction incident-handler $(az functionapp function show --resource-group rg-functions --name func-app-prod --function-name AlertHandler --query invokeUrlTemplate --output tsv)

# Add Logic App
az monitor action-group update \
  --resource-group rg-monitoring \
  --name ag-devops \
  --add-action logicapp ticketing-system $(az logic workflow show --resource-group rg-automation --name logic-create-ticket --query id --output tsv)
```

---

## 📊 Workbooks and Dashboards

### **Create Azure Dashboard:**

```bash
# Create dashboard (JSON definition)
cat > dashboard.json << 'EOF'
{
  "properties": {
    "lenses": {
      "0": {
        "order": 0,
        "parts": {
          "0": {
            "position": {
              "x": 0,
              "y": 0,
              "colSpan": 6,
              "rowSpan": 4
            },
            "metadata": {
              "inputs": [],
              "type": "Extension/HubsExtension/PartType/MarkdownPart",
              "settings": {
                "content": {
                  "settings": {
                    "content": "# Production Monitoring Dashboard\n\nReal-time monitoring for production resources"
                  }
                }
              }
            }
          }
        }
      }
    }
  },
  "name": "Production-Monitoring",
  "type": "Microsoft.Portal/dashboards",
  "location": "eastus",
  "tags": {
    "hidden-title": "Production Monitoring"
  }
}
EOF

# Deploy dashboard
az portal dashboard create \
  --resource-group rg-monitoring \
  --name Production-Monitoring \
  --input-path dashboard.json
```

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)
