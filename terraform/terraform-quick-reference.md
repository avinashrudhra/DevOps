# Terraform Quick Reference (Azure Focused)

**Essential commands, syntax, and Azure resource examples for daily IaC operations**

---

## 🚀 Quick Start Commands

```bash
# Initialize working directory
terraform init

# Format code
terraform fmt

# Validate configuration
terraform validate

# Create execution plan
terraform plan

# Apply changes
terraform apply

# Apply without confirmation
terraform apply -auto-approve

# Destroy infrastructure
terraform destroy

# Show current state
terraform show

# List resources in state
terraform state list

# Show specific resource
terraform state show <resource>
```

---

## 📁 Project Structure

```
terraform-project/
├── main.tf                 # Primary resources
├── variables.tf            # Input variables
├── outputs.tf              # Output values
├── providers.tf            # Provider configuration
├── backend.tf              # Backend configuration
├── terraform.tfvars        # Variable values
├── versions.tf             # Version constraints
├── locals.tf               # Local values
├── data.tf                 # Data sources
└── modules/
    ├── network/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    └── compute/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

---

## 🔧 Core Terraform Syntax

### **Provider Configuration**

```hcl
# Basic provider
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
    random = {
      source  = "hashicorp/random"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {
    resource_group {
      prevent_deletion_if_contains_resources = true
    }
    key_vault {
      purge_soft_delete_on_destroy = true
    }
  }
  
  subscription_id = var.subscription_id
  tenant_id       = var.tenant_id
}

# Multiple providers (aliases)
provider "azurerm" {
  alias           = "secondary"
  subscription_id = var.secondary_subscription_id
  features {}
}
```

### **Variables**

```hcl
# String variable
variable "location" {
  description = "Azure region"
  type        = string
  default     = "East US"
}

# Number variable
variable "vm_count" {
  description = "Number of VMs"
  type        = number
  default     = 2
  
  validation {
    condition     = var.vm_count > 0 && var.vm_count <= 10
    error_message = "VM count must be between 1 and 10."
  }
}

# Bool variable
variable "enable_monitoring" {
  description = "Enable monitoring"
  type        = bool
  default     = true
}

# List variable
variable "allowed_ips" {
  description = "Allowed IP addresses"
  type        = list(string)
  default     = ["10.0.0.0/8", "172.16.0.0/12"]
}

# Map variable
variable "tags" {
  description = "Resource tags"
  type        = map(string)
  default = {
    Environment = "Production"
    ManagedBy   = "Terraform"
  }
}

# Object variable
variable "vm_config" {
  description = "VM configuration"
  type = object({
    name     = string
    size     = string
    image_sku = string
  })
  default = {
    name     = "vm-web-01"
    size     = "Standard_B2s"
    image_sku = "2022-Datacenter"
  }
}

# Sensitive variable
variable "admin_password" {
  description = "Admin password"
  type        = string
  sensitive   = true
}
```

### **Locals**

```hcl
locals {
  # Simple values
  environment = terraform.workspace
  
  # Computed values
  resource_name_prefix = "${var.project}-${local.environment}"
  
  # Common tags
  common_tags = {
    Environment = local.environment
    ManagedBy   = "Terraform"
    Project     = var.project
    CostCenter  = var.cost_center
  }
  
  # Conditional
  vm_size = local.environment == "prod" ? "Standard_D4s_v3" : "Standard_B2s"
  
  # Map transformation
  subnet_ids = {
    for k, v in azurerm_subnet.example : k => v.id
  }
}
```

### **Outputs**

```hcl
# Simple output
output "resource_group_name" {
  description = "Resource group name"
  value       = azurerm_resource_group.example.name
}

# Sensitive output
output "database_password" {
  description = "Database admin password"
  value       = azurerm_sql_server.example.administrator_login_password
  sensitive   = true
}

# Complex output
output "vm_details" {
  description = "VM details"
  value = {
    name       = azurerm_virtual_machine.example.name
    id         = azurerm_virtual_machine.example.id
    private_ip = azurerm_network_interface.example.private_ip_address
  }
}

# Module output
output "network_info" {
  description = "Network module outputs"
  value       = module.network
}
```

### **Data Sources**

```hcl
# Current Azure client config
data "azurerm_client_config" "current" {}

# Existing resource group
data "azurerm_resource_group" "existing" {
  name = "rg-existing"
}

# Existing virtual network
data "azurerm_virtual_network" "existing" {
  name                = "vnet-existing"
  resource_group_name = data.azurerm_resource_group.existing.name
}

# Key Vault secret
data "azurerm_key_vault_secret" "example" {
  name         = "secret-name"
  key_vault_id = data.azurerm_key_vault.example.id
}

# Usage
resource "azurerm_storage_account" "example" {
  name                = "storageacct"
  resource_group_name = data.azurerm_resource_group.existing.name
  location            = data.azurerm_resource_group.existing.location
  # ...
}
```

---

## ☁️ Azure Resource Examples

### **Resource Group**

```hcl
resource "azurerm_resource_group" "example" {
  name     = "rg-${var.environment}"
  location = var.location
  
  tags = merge(
    local.common_tags,
    {
      Name = "Main Resource Group"
    }
  )
}
```

### **Virtual Network & Subnets**

```hcl
# Virtual Network
resource "azurerm_virtual_network" "example" {
  name                = "vnet-${var.environment}"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
  
  tags = local.common_tags
}

# Subnet
resource "azurerm_subnet" "web" {
  name                 = "subnet-web"
  resource_group_name  = azurerm_resource_group.example.name
  virtual_network_name = azurerm_virtual_network.example.name
  address_prefixes     = ["10.0.1.0/24"]
  
  service_endpoints = [
    "Microsoft.Storage",
    "Microsoft.Sql",
    "Microsoft.KeyVault"
  ]
}

# Multiple subnets with for_each
resource "azurerm_subnet" "subnets" {
  for_each = var.subnets
  
  name                 = each.key
  resource_group_name  = azurerm_resource_group.example.name
  virtual_network_name = azurerm_virtual_network.example.name
  address_prefixes     = [each.value.address_prefix]
}
```

### **Network Security Group**

```hcl
resource "azurerm_network_security_group" "web" {
  name                = "nsg-web"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
  
  security_rule {
    name                       = "AllowHTTPS"
    priority                   = 100
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "443"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }
  
  security_rule {
    name                       = "AllowHTTP"
    priority                   = 110
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "80"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }
  
  tags = local.common_tags
}

# Associate with subnet
resource "azurerm_subnet_network_security_group_association" "web" {
  subnet_id                 = azurerm_subnet.web.id
  network_security_group_id = azurerm_network_security_group.web.id
}
```

### **Public IP**

```hcl
resource "azurerm_public_ip" "example" {
  name                = "pip-${var.vm_name}"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
  allocation_method   = "Static"
  sku                 = "Standard"
  zones               = ["1", "2", "3"]
  
  tags = local.common_tags
}
```

### **Virtual Machine**

```hcl
# Network Interface
resource "azurerm_network_interface" "example" {
  name                = "nic-${var.vm_name}"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
  
  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.web.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.example.id
  }
  
  tags = local.common_tags
}

# Linux VM
resource "azurerm_linux_virtual_machine" "example" {
  name                = var.vm_name
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location
  size                = "Standard_D2s_v3"
  admin_username      = "azureuser"
  
  network_interface_ids = [
    azurerm_network_interface.example.id,
  ]
  
  admin_ssh_key {
    username   = "azureuser"
    public_key = file("~/.ssh/id_rsa.pub")
  }
  
  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Premium_LRS"
    disk_size_gb         = 128
  }
  
  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts-gen2"
    version   = "latest"
  }
  
  boot_diagnostics {
    storage_account_uri = azurerm_storage_account.diag.primary_blob_endpoint
  }
  
  tags = local.common_tags
}

# Windows VM
resource "azurerm_windows_virtual_machine" "example" {
  name                = var.vm_name
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location
  size                = "Standard_D2s_v3"
  admin_username      = "adminuser"
  admin_password      = var.admin_password
  
  network_interface_ids = [
    azurerm_network_interface.example.id,
  ]
  
  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Premium_LRS"
  }
  
  source_image_reference {
    publisher = "MicrosoftWindowsServer"
    offer     = "WindowsServer"
    sku       = "2022-Datacenter"
    version   = "latest"
  }
  
  tags = local.common_tags
}
```

### **Storage Account**

```hcl
resource "azurerm_storage_account" "example" {
  name                     = "st${var.environment}${random_integer.suffix.result}"
  resource_group_name      = azurerm_resource_group.example.name
  location                 = azurerm_resource_group.example.location
  account_tier             = "Standard"
  account_replication_type = "GRS"
  account_kind             = "StorageV2"
  
  enable_https_traffic_only = true
  min_tls_version           = "TLS1_2"
  
  blob_properties {
    versioning_enabled = true
    delete_retention_policy {
      days = 30
    }
    container_delete_retention_policy {
      days = 30
    }
  }
  
  network_rules {
    default_action             = "Deny"
    virtual_network_subnet_ids = [azurerm_subnet.web.id]
    ip_rules                   = var.allowed_ips
    bypass                     = ["AzureServices"]
  }
  
  tags = local.common_tags
}

# Storage Container
resource "azurerm_storage_container" "example" {
  name                  = "data"
  storage_account_name  = azurerm_storage_account.example.name
  container_access_type = "private"
}
```

### **Azure SQL**

```hcl
# SQL Server
resource "azurerm_mssql_server" "example" {
  name                         = "sql-${var.environment}-${random_integer.suffix.result}"
  resource_group_name          = azurerm_resource_group.example.name
  location                     = azurerm_resource_group.example.location
  version                      = "12.0"
  administrator_login          = "sqladmin"
  administrator_login_password = var.sql_admin_password
  minimum_tls_version          = "1.2"
  
  azuread_administrator {
    login_username = "AzureAD Admin"
    object_id      = data.azurerm_client_config.current.object_id
  }
  
  tags = local.common_tags
}

# SQL Database
resource "azurerm_mssql_database" "example" {
  name           = "sqldb-${var.environment}"
  server_id      = azurerm_mssql_server.example.id
  collation      = "SQL_Latin1_General_CP1_CI_AS"
  license_type   = "LicenseIncluded"
  max_size_gb    = 250
  sku_name       = "S1"
  zone_redundant = false
  
  short_term_retention_policy {
    retention_days = 7
  }
  
  long_term_retention_policy {
    weekly_retention  = "P1W"
    monthly_retention = "P1M"
    yearly_retention  = "P1Y"
    week_of_year      = 1
  }
  
  tags = local.common_tags
}

# Firewall Rule
resource "azurerm_mssql_firewall_rule" "example" {
  name             = "AllowAzureServices"
  server_id        = azurerm_mssql_server.example.id
  start_ip_address = "0.0.0.0"
  end_ip_address   = "0.0.0.0"
}
```

### **Azure Kubernetes Service (AKS)**

```hcl
resource "azurerm_kubernetes_cluster" "example" {
  name                = "aks-${var.environment}"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
  dns_prefix          = "aks-${var.environment}"
  kubernetes_version  = "1.28.0"
  
  default_node_pool {
    name                = "system"
    node_count          = 3
    vm_size             = "Standard_D2s_v3"
    vnet_subnet_id      = azurerm_subnet.aks.id
    type                = "VirtualMachineScaleSets"
    enable_auto_scaling = true
    min_count           = 3
    max_count           = 10
    os_disk_size_gb     = 128
    
    upgrade_settings {
      max_surge = "33%"
    }
  }
  
  identity {
    type = "SystemAssigned"
  }
  
  network_profile {
    network_plugin     = "azure"
    network_policy     = "calico"
    load_balancer_sku  = "standard"
    service_cidr       = "172.16.0.0/16"
    dns_service_ip     = "172.16.0.10"
  }
  
  azure_active_directory_role_based_access_control {
    managed            = true
    azure_rbac_enabled = true
  }
  
  oms_agent {
    log_analytics_workspace_id = azurerm_log_analytics_workspace.example.id
  }
  
  tags = local.common_tags
}

# Additional Node Pool
resource "azurerm_kubernetes_cluster_node_pool" "user" {
  name                  = "user"
  kubernetes_cluster_id = azurerm_kubernetes_cluster.example.id
  vm_size               = "Standard_D4s_v3"
  enable_auto_scaling   = true
  min_count             = 2
  max_count             = 10
  vnet_subnet_id        = azurerm_subnet.aks.id
  
  node_labels = {
    workload = "user-apps"
  }
  
  tags = local.common_tags
}
```

### **Key Vault**

```hcl
resource "azurerm_key_vault" "example" {
  name                       = "kv-${var.environment}-${random_integer.suffix.result}"
  location                   = azurerm_resource_group.example.location
  resource_group_name        = azurerm_resource_group.example.name
  tenant_id                  = data.azurerm_client_config.current.tenant_id
  sku_name                   = "standard"
  soft_delete_retention_days = 90
  purge_protection_enabled   = true
  
  enabled_for_disk_encryption     = true
  enabled_for_deployment          = true
  enabled_for_template_deployment = true
  
  network_acls {
    default_action             = "Deny"
    bypass                     = "AzureServices"
    virtual_network_subnet_ids = [azurerm_subnet.web.id]
    ip_rules                   = var.allowed_ips
  }
  
  tags = local.common_tags
}

# Access Policy
resource "azurerm_key_vault_access_policy" "example" {
  key_vault_id = azurerm_key_vault.example.id
  tenant_id    = data.azurerm_client_config.current.tenant_id
  object_id    = data.azurerm_client_config.current.object_id
  
  secret_permissions = [
    "Get", "List", "Set", "Delete", "Purge", "Recover"
  ]
  
  key_permissions = [
    "Get", "List", "Create", "Delete", "Update"
  ]
  
  certificate_permissions = [
    "Get", "List", "Create", "Delete", "Update"
  ]
}

# Secret
resource "azurerm_key_vault_secret" "example" {
  name         = "db-connection-string"
  value        = "Server=...;Database=...;User Id=...;Password=..."
  key_vault_id = azurerm_key_vault.example.id
  
  depends_on = [azurerm_key_vault_access_policy.example]
}
```

---

## 🔄 Advanced Patterns

### **For_each**

```hcl
# Multiple resource groups
variable "resource_groups" {
  type = map(object({
    location = string
  }))
  default = {
    "rg-network" = { location = "East US" }
    "rg-compute" = { location = "East US" }
    "rg-data"    = { location = "West US" }
  }
}

resource "azurerm_resource_group" "rgs" {
  for_each = var.resource_groups
  
  name     = each.key
  location = each.value.location
  
  tags = local.common_tags
}

# Access specific resource
output "network_rg_id" {
  value = azurerm_resource_group.rgs["rg-network"].id
}
```

### **Count**

```hcl
# Multiple VMs
variable "vm_count" {
  default = 3
}

resource "azurerm_linux_virtual_machine" "vms" {
  count               = var.vm_count
  name                = "vm-web-${count.index + 1}"
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location
  size                = "Standard_B2s"
  # ...
}

# Access specific VM
output "first_vm_id" {
  value = azurerm_linux_virtual_machine.vms[0].id
}
```

### **Dynamic Blocks**

```hcl
variable "security_rules" {
  type = list(object({
    name                       = string
    priority                   = number
    direction                  = string
    access                     = string
    protocol                   = string
    source_port_range          = string
    destination_port_range     = string
    source_address_prefix      = string
    destination_address_prefix = string
  }))
}

resource "azurerm_network_security_group" "example" {
  name                = "nsg-dynamic"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
  
  dynamic "security_rule" {
    for_each = var.security_rules
    
    content {
      name                       = security_rule.value.name
      priority                   = security_rule.value.priority
      direction                  = security_rule.value.direction
      access                     = security_rule.value.access
      protocol                   = security_rule.value.protocol
      source_port_range          = security_rule.value.source_port_range
      destination_port_range     = security_rule.value.destination_port_range
      source_address_prefix      = security_rule.value.source_address_prefix
      destination_address_prefix = security_rule.value.destination_address_prefix
    }
  }
}
```

### **Conditional Resources**

```hcl
# Create public IP only if needed
variable "create_public_ip" {
  type    = bool
  default = true
}

resource "azurerm_public_ip" "example" {
  count               = var.create_public_ip ? 1 : 0
  name                = "pip-example"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
  allocation_method   = "Static"
}

resource "azurerm_network_interface" "example" {
  name                = "nic-example"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
  
  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.example.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = var.create_public_ip ? azurerm_public_ip.example[0].id : null
  }
}
```

---

## 🛠️ State Management Commands

```bash
# List all resources
terraform state list

# Show resource details
terraform state show azurerm_resource_group.example

# Move resource (rename)
terraform state mv azurerm_resource_group.old azurerm_resource_group.new

# Remove from state (keep in Azure)
terraform state rm azurerm_resource_group.example

# Pull state
terraform state pull > state.json

# Import existing resource
terraform import azurerm_resource_group.example /subscriptions/{subscription-id}/resourceGroups/{resource-group-name}

# Replace provider
terraform state replace-provider hashicorp/azurerm registry.terraform.io/hashicorp/azurerm
```

---

## 📦 Module Usage

```hcl
# Local module
module "network" {
  source = "./modules/network"
  
  vnet_name           = "vnet-prod"
  resource_group_name = azurerm_resource_group.example.name
  location            = var.location
  address_space       = ["10.0.0.0/16"]
  
  subnets = {
    web  = { address_prefix = "10.0.1.0/24" }
    app  = { address_prefix = "10.0.2.0/24" }
    data = { address_prefix = "10.0.3.0/24" }
  }
  
  tags = local.common_tags
}

# Registry module
module "aks" {
  source  = "Azure/aks/azurerm"
  version = "6.0.0"
  
  cluster_name        = "aks-prod"
  resource_group_name = azurerm_resource_group.example.name
  location            = var.location
  # ...
}

# Git module
module "custom" {
  source = "git::https://github.com/example/terraform-azure-module.git?ref=v1.0.0"
  
  # variables...
}
```

---

## 🔍 Functions Reference

```hcl
# String functions
upper("hello")                        # HELLO
lower("HELLO")                        # hello
title("hello world")                  # Hello World
replace("hello world", "world", "tf") # hello tf
join("-", ["rg", "prod", "eastus"])   # rg-prod-eastus
split("-", "rg-prod-eastus")          # ["rg", "prod", "eastus"]
format("vm-%s-%02d", "web", 1)        # vm-web-01
substr("hello", 0, 2)                 # he

# Collection functions
length([1, 2, 3])                     # 3
merge({a=1}, {b=2})                   # {a=1, b=2}
lookup({a=1, b=2}, "a", "default")    # 1
keys({a=1, b=2})                      # ["a", "b"]
values({a=1, b=2})                    # [1, 2]
contains(["a", "b"], "a")             # true
concat(["a"], ["b"])                  # ["a", "b"]
flatten([[1, 2], [3, 4]])             # [1, 2, 3, 4]
distinct([1, 2, 2, 3])                # [1, 2, 3]

# Numeric functions
min(1, 2, 3)                          # 1
max(1, 2, 3)                          # 3
ceil(5.1)                             # 6
floor(5.9)                            # 5

# Date/Time functions
timestamp()                           # 2026-01-07T10:00:00Z
formatdate("DD MMM YYYY", timestamp()) # 07 Jan 2026

# Filesystem functions
file("path/to/file.txt")              # Read file
fileexists("path/to/file.txt")        # true/false
basename("/path/to/file.txt")         # file.txt
dirname("/path/to/file.txt")          # /path/to

# Encoding functions
base64encode("hello")                 # aGVsbG8=
base64decode("aGVsbG8=")              # hello
jsonencode({a=1})                     # {"a":1}
jsondecode('{"a":1}')                 # {a=1}

# IP Network functions
cidrsubnet("10.0.0.0/16", 8, 1)      # 10.0.1.0/24
cidrhost("10.0.1.0/24", 5)           # 10.0.1.5
cidrnetmask("10.0.1.0/24")           # 255.255.255.0
```

---

## 🎯 Best Practices

```hcl
# Use consistent naming
resource "azurerm_resource_group" "example" {
  name     = "${var.prefix}-${var.environment}-rg"
  location = var.location
}

# Always tag resources
tags = merge(
  local.common_tags,
  {
    Name = "Specific Name"
  }
)

# Use remote state
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "tfstate"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}

# Lock versions
terraform {
  required_version = ">= 1.0, < 2.0"
  
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

# Use modules
# Avoid duplicating code
# Create reusable components

# Secure sensitive data
variable "password" {
  sensitive = true
}

# Use data sources
data "azurerm_resource_group" "existing" {
  name = "rg-existing"
}
```

---

**Keep this reference handy for daily Terraform operations! 🚀**

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

