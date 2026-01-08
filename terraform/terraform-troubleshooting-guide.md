# Terraform Troubleshooting Guide (Azure Focused)

**Solutions to common issues in Terraform Azure deployments**

---

## 🎯 Troubleshooting Strategy

```
1. Read the error message carefully
2. Check Terraform and provider versions
3. Verify Azure credentials
4. Review state file
5. Check Azure portal
6. Enable debug logging
7. Search documentation/community
8. Isolate the problem
```

---

## 🔥 Common Terraform Issues

### **Issue 1: Authentication Failures**

**Error Messages:**
```
Error: Error building account: Error getting authenticated object ID: Error parsing json result from the Azure CLI
Error: Error building AzureRM Client: obtain subscription() from Azure CLI
Error: Error getting AzureRM credentials: AADSTS700016: Application with identifier 'xxx' was not found
```

**Diagnosis:**
```bash
# Check Azure CLI login
az account show

# List available subscriptions
az account list --output table

# Check service principal (if using)
az ad sp show --id $ARM_CLIENT_ID
```

**Solutions:**

**Solution 1: Azure CLI Not Logged In**
```bash
# Login
az login

# Select subscription
az account set --subscription "SUBSCRIPTION_NAME_OR_ID"

# Verify
az account show
```

**Solution 2: Service Principal Issues**
```bash
# Check environment variables (PowerShell)
$env:ARM_CLIENT_ID
$env:ARM_CLIENT_SECRET
$env:ARM_SUBSCRIPTION_ID
$env:ARM_TENANT_ID

# Bash
echo $ARM_CLIENT_ID
echo $ARM_CLIENT_SECRET
echo $ARM_SUBSCRIPTION_ID
echo $ARM_TENANT_ID

# Recreate service principal
az ad sp create-for-rbac \
  --name "terraform-sp" \
  --role="Contributor" \
  --scopes="/subscriptions/YOUR_SUBSCRIPTION_ID"
```

**Solution 3: Token Expired**
```bash
# Clear Azure CLI cache
az account clear
az login
```

**Solution 4: Explicit Provider Config**
```hcl
provider "azurerm" {
  features {}
  
  subscription_id = "00000000-0000-0000-0000-000000000000"
  tenant_id       = "00000000-0000-0000-0000-000000000000"
  client_id       = "00000000-0000-0000-0000-000000000000"
  client_secret   = "your-client-secret"
}
```

---

### **Issue 2: Provider Plugin Errors**

**Error Messages:**
```
Error: Failed to query available provider packages
Error: Could not retrieve the list of available versions for provider
Error: Provider registry.terraform.io/hashicorp/azurerm: Error installing provider
```

**Solutions:**

**Solution 1: Network/Proxy Issues**
```bash
# Check internet connectivity
curl https://registry.terraform.io/

# Configure proxy (if needed)
export HTTP_PROXY="http://proxy:port"
export HTTPS_PROXY="http://proxy:port"
```

**Solution 2: Clear Plugin Cache**
```bash
# Remove .terraform directory
rm -rf .terraform

# Re-initialize
terraform init
```

**Solution 3: Specify Provider Source**
```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"  # Explicit source
      version = "~> 3.0"
    }
  }
}
```

**Solution 4: Use Plugin Cache**
```bash
# Set plugin cache directory
export TF_PLUGIN_CACHE_DIR="$HOME/.terraform.d/plugin-cache"
mkdir -p $TF_PLUGIN_CACHE_DIR

terraform init
```

---

### **Issue 3: State Lock Errors**

**Error Messages:**
```
Error: Error locking state: Error acquiring the state lock
Error: state blob is already locked
Lock Info:
  ID:        xxxxx-xxxxx-xxxxx
  Path:      xxx.terraform.tfstate
  Operation: OperationTypeApply
  Who:       user@hostname
  Version:   1.0.0
  Created:   2026-01-07 10:00:00
```

**Diagnosis:**
```bash
# Check if another terraform process is running
ps aux | grep terraform

# Check Azure Storage blob lease
az storage blob show \
  --container-name tfstate \
  --name prod.terraform.tfstate \
  --account-name tfstatestorage
```

**Solutions:**

**Solution 1: Wait for Lock Release**
```bash
# Wait for other operation to complete
# Check with team members

# Monitor lock status
az storage blob show \
  --container-name tfstate \
  --name prod.terraform.tfstate \
  --account-name tfstatestorage \
  --query properties.lease.status
```

**Solution 2: Force Unlock (Use with Caution)**
```bash
# Get lock ID from error message
terraform force-unlock LOCK_ID

# Confirm with 'yes'
```

**Solution 3: Break Blob Lease**
```bash
# DANGEROUS: Only if you're sure no other operation is running
az storage blob lease break \
  --container-name tfstate \
  --blob-name prod.terraform.tfstate \
  --account-name tfstatestorage
```

**Solution 4: Prevent Future Locks**
```hcl
# Disable locking (NOT RECOMMENDED for production)
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "tfstatestorage"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
    # Disable locking (use only for debugging)
    # use_azuread_auth = false
  }
}
```

---

### **Issue 4: Resource Already Exists**

**Error Messages:**
```
Error: A resource with the ID "/subscriptions/.../resourceGroups/rg-example" already exists
Error: storage account "staccount" already exists
```

**Diagnosis:**
```bash
# Check if resource exists in Azure
az group show --name rg-example
az storage account show --name staccount

# Check state file
terraform state list
terraform state show azurerm_resource_group.example
```

**Solutions:**

**Solution 1: Import Existing Resource**
```bash
# Import resource group
terraform import azurerm_resource_group.example \
  /subscriptions/{subscription-id}/resourceGroups/{resource-group-name}

# Import storage account
terraform import azurerm_storage_account.example \
  /subscriptions/{subscription-id}/resourceGroups/{rg}/providers/Microsoft.Storage/storageAccounts/{account}

# Verify import
terraform plan
```

**Solution 2: Remove from Azure (if safe)**
```bash
# Delete resource in Azure
az group delete --name rg-example

# Or remove from state
terraform state rm azurerm_resource_group.example
```

**Solution 3: Change Resource Name**
```hcl
# Use unique naming
resource "azurerm_storage_account" "example" {
  name = "st${var.environment}${random_integer.suffix.result}"
  # ...
}

resource "random_integer" "suffix" {
  min = 10000
  max = 99999
}
```

---

### **Issue 5: Resource Depends On Deleted Resource**

**Error Messages:**
```
Error: resource group has been deleted
Error: subnet not found
Error: Error making Read request on resource
```

**Diagnosis:**
```bash
# Check state
terraform state list
terraform show

# Verify resources in Azure
az group list
az network vnet list
```

**Solutions:**

**Solution 1: Refresh State**
```bash
# Refresh state to sync with Azure
terraform refresh

# Or
terraform plan -refresh-only
terraform apply -refresh-only
```

**Solution 2: Remove from State**
```bash
# Remove missing resources
terraform state rm azurerm_subnet.example

# Re-create
terraform plan
terraform apply
```

**Solution 3: Taint and Recreate**
```bash
# Mark for recreation
terraform taint azurerm_resource_group.example

# Apply
terraform apply
```

---

### **Issue 6: Naming Constraint Violations**

**Error Messages:**
```
Error: storage account name must be between 3 and 24 characters
Error: storage account name must consist of lowercase letters and numbers only
Error: resource name must start with a letter or number
```

**Solutions:**

```hcl
# Storage Account - 3-24 chars, lowercase, numbers only
resource "azurerm_storage_account" "example" {
  name = lower(replace("st${var.environment}${var.project}", "-", ""))
  # ...
  
  # Or use validation
}

# Resource Group - 1-90 chars
resource "azurerm_resource_group" "example" {
  name = substr("rg-${var.long_project_name}", 0, 90)
  # ...
}

# Validate variable
variable "storage_account_name" {
  type = string
  
  validation {
    condition     = can(regex("^[a-z0-9]{3,24}$", var.storage_account_name))
    error_message = "Storage account name must be 3-24 characters, lowercase letters and numbers only."
  }
}
```

---

### **Issue 7: Quota/Limit Exceeded**

**Error Messages:**
```
Error: Operation results in exceeding quota limits of Core
Error: Maximum number of resources of type 'Virtual Machines' has been reached
```

**Diagnosis:**
```bash
# Check quotas
az vm list-usage --location eastus --output table

# Check subscription limits
az network vnet list-usage \
  --resource-group rg-network \
  --name vnet-example \
  --output table
```

**Solutions:**

**Solution 1: Request Quota Increase**
```bash
# Azure Portal:
# Subscriptions → Usage + quotas → Request increase
```

**Solution 2: Use Different Region**
```hcl
variable "regions" {
  default = ["East US", "West US", "Central US"]
}

# Try different regions
resource "azurerm_resource_group" "example" {
  name     = "rg-example"
  location = var.regions[0]  # If fails, try regions[1]
}
```

**Solution 3: Different VM Size**
```hcl
variable "vm_size_options" {
  default = ["Standard_D4s_v3", "Standard_D2s_v3", "Standard_B2s"]
}

# Fallback to smaller size
```

---

### **Issue 8: API Throttling**

**Error Messages:**
```
Error: Error waiting for provisioning: Code="Throttled"
Error: Too many requests. Retry after some time.
```

**Solutions:**

**Solution 1: Add Delays**
```hcl
# Use time_sleep resource
resource "time_sleep" "wait_30_seconds" {
  depends_on = [azurerm_resource_group.example]
  
  create_duration = "30s"
}

resource "azurerm_virtual_network" "example" {
  depends_on = [time_sleep.wait_30_seconds]
  # ...
}
```

**Solution 2: Reduce Parallelism**
```bash
# Limit concurrent operations
terraform apply -parallelism=1
```

**Solution 3: Retry Logic in Provider**
```hcl
provider "azurerm" {
  features {}
  
  # Provider automatically retries throttled requests
  # Adjust timeout if needed
}
```

---

## 🔧 Azure-Specific Issues

### **Issue 9: Virtual Network Peering Fails**

**Error Messages:**
```
Error: remote virtual network is already peered
Error: cannot peer with itself
```

**Solutions:**

```hcl
# Ensure bidirectional peering
resource "azurerm_virtual_network_peering" "hub_to_spoke" {
  name                      = "hub-to-spoke"
  resource_group_name       = azurerm_resource_group.hub.name
  virtual_network_name      = azurerm_virtual_network.hub.name
  remote_virtual_network_id = azurerm_virtual_network.spoke.id
  allow_forwarded_traffic   = true
}

resource "azurerm_virtual_network_peering" "spoke_to_hub" {
  name                      = "spoke-to-hub"
  resource_group_name       = azurerm_resource_group.spoke.name
  virtual_network_name      = azurerm_virtual_network.spoke.name
  remote_virtual_network_id = azurerm_virtual_network.hub.id
  allow_forwarded_traffic   = true
  
  # Explicit dependency
  depends_on = [azurerm_virtual_network_peering.hub_to_spoke]
}
```

---

### **Issue 10: AKS Creation Fails**

**Error Messages:**
```
Error: Code="InvalidTemplateDeployment" Message="The template deployment failed"
Error: Service principal doesn't have permissions
```

**Solutions:**

**Solution 1: Use System Assigned Identity**
```hcl
resource "azurerm_kubernetes_cluster" "example" {
  # ...
  
  identity {
    type = "SystemAssigned"  # Recommended
  }
  
  # Instead of:
  # service_principal {
  #   client_id     = "xxx"
  #   client_secret = "xxx"
  # }
}
```

**Solution 2: Grant Required Permissions**
```bash
# Network Contributor on subnet
az role assignment create \
  --assignee $PRINCIPAL_ID \
  --role "Network Contributor" \
  --scope $SUBNET_ID

# AcrPull on ACR
az role assignment create \
  --assignee $PRINCIPAL_ID \
  --role "AcrPull" \
  --scope $ACR_ID
```

---

### **Issue 11: Private Endpoint Connection Issues**

**Error Messages:**
```
Error: private endpoint connection must be approved
Error: private DNS zone not configured
```

**Solutions:**

```hcl
# Private Endpoint
resource "azurerm_private_endpoint" "example" {
  name                = "pe-sqlserver"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
  subnet_id           = azurerm_subnet.private_endpoints.id
  
  private_service_connection {
    name                           = "psc-sqlserver"
    private_connection_resource_id = azurerm_mssql_server.example.id
    subresource_names              = ["sqlServer"]
    is_manual_connection           = false  # Auto-approve
  }
  
  private_dns_zone_group {
    name                 = "pdns-group"
    private_dns_zone_ids = [azurerm_private_dns_zone.sql.id]
  }
}

# Private DNS Zone
resource "azurerm_private_dns_zone" "sql" {
  name                = "privatelink.database.windows.net"
  resource_group_name = azurerm_resource_group.example.name
}

# Link to VNet
resource "azurerm_private_dns_zone_virtual_network_link" "sql" {
  name                  = "pdns-link"
  resource_group_name   = azurerm_resource_group.example.name
  private_dns_zone_name = azurerm_private_dns_zone.sql.name
  virtual_network_id    = azurerm_virtual_network.example.id
}
```

---

## 🐛 Debugging Techniques

### **Enable Debug Logging**

```bash
# Trace level (most verbose)
TF_LOG=TRACE terraform apply

# Different log levels
TF_LOG=DEBUG terraform plan
TF_LOG=INFO terraform apply
TF_LOG=WARN terraform apply
TF_LOG=ERROR terraform apply

# Log to file
TF_LOG=TRACE TF_LOG_PATH=./terraform.log terraform apply

# Azure CLI debug
az --debug group list
```

### **Inspect State File**

```bash
# Show entire state
terraform show

# Show specific resource
terraform state show azurerm_resource_group.example

# List all resources
terraform state list

# Pull state (if remote)
terraform state pull > state.json

# Inspect JSON
cat state.json | jq '.resources[] | select(.type=="azurerm_resource_group")'
```

### **Validate Configuration**

```bash
# Format code
terraform fmt -recursive

# Validate syntax
terraform validate

# Validate and check variables
terraform validate -json

# Plan with detailed output
terraform plan -out=tfplan

# Show plan in JSON
terraform show -json tfplan | jq '.'
```

### **Test in Isolation**

```bash
# Target specific resource
terraform apply -target=azurerm_resource_group.example

# Destroy specific resource
terraform destroy -target=azurerm_resource_group.example

# Refresh specific resource
terraform refresh -target=azurerm_resource_group.example
```

---

## 📊 Performance Issues

### **Slow Plan/Apply**

**Causes:**
- Large state file
- Many resources
- Slow provider API
- Network latency

**Solutions:**

```bash
# Use parallelism
terraform apply -parallelism=10  # Default is 10

# Target specific resources
terraform apply -target=module.network

# Use -refresh=false if state is current
terraform plan -refresh=false

# Break into smaller configurations
# Use modules and separate state files
```

### **Optimize Provider Configuration**

```hcl
provider "azurerm" {
  features {}
  
  # Skip provider registration (faster)
  skip_provider_registration = true
  
  # Disable correlation request ID
  disable_correlation_request_id = true
}
```

---

## 🎯 Best Practices to Avoid Issues

1. **Version Pinning**
```hcl
terraform {
  required_version = "~> 1.0"
  
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"  # Not >= or latest
    }
  }
}
```

2. **Use Remote State**
```hcl
terraform {
  backend "azurerm" {
    # ... configuration
  }
}
```

3. **Validate Before Apply**
```bash
terraform fmt -check
terraform validate
terraform plan
# Review plan carefully!
terraform apply
```

4. **Use Lifecycle Rules**
```hcl
resource "azurerm_resource_group" "example" {
  # ...
  
  lifecycle {
    prevent_destroy = true
    ignore_changes  = [tags]
  }
}
```

5. **Implement Naming Conventions**
```hcl
locals {
  resource_prefix = "${var.project}-${var.environment}"
}

resource "azurerm_resource_group" "example" {
  name = "${local.resource_prefix}-rg"
  # ...
}
```

---

## 🔍 Common Error Messages Quick Reference

| Error | Cause | Solution |
|-------|-------|----------|
| `Error building account` | Auth issue | `az login` |
| `Error locking state` | Concurrent operation | Wait or `force-unlock` |
| `already exists` | Resource conflict | Import or rename |
| `quota exceeded` | Subscription limit | Request increase |
| `Throttled` | Too many requests | Reduce parallelism |
| `subnet not found` | State mismatch | `terraform refresh` |
| `validation error` | Invalid config | Check constraints |
| `cycle detected` | Circular dependency | Fix dependencies |

---

**Happy troubleshooting! 🔧🚀**

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

