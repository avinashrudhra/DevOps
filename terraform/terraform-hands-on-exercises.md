# Terraform Hands-On Exercises (Azure Focused)

**30+ Practical Labs from Basic Deployment to Production Infrastructure**

---

## 🎯 Exercise Structure

Each exercise includes:
- **Objective**: What you'll learn
- **Prerequisites**: What you need
- **Steps**: Detailed instructions
- **Validation**: How to verify success
- **Challenge**: Extend your learning

---

## 📚 Lab Environment Setup

### **Exercise 0: Setup Development Environment**

**Objective:** Prepare your environment for Terraform development

**Prerequisites:**
- Azure subscription
- Administrative access to local machine

**Steps:**

```bash
# 1. Install Terraform (Windows with Chocolatey)
choco install terraform

# Or download from https://www.terraform.io/downloads
# Extract and add to PATH

# Verify
terraform version

# 2. Install Azure CLI
# Windows
winget install Microsoft.AzureCLI

# Verify
az version

# 3. Login to Azure
az login

# 4. List subscriptions
az account list --output table

# 5. Set active subscription
az account set --subscription "YOUR_SUBSCRIPTION_NAME_OR_ID"

# 6. Show current subscription
az account show

# 7. Create service principal (optional, for automation)
az ad sp create-for-rbac --name "terraform-sp" --role="Contributor" --scopes="/subscriptions/YOUR_SUBSCRIPTION_ID"

# Output (save this!):
# {
#   "appId": "00000000-0000-0000-0000-000000000000",
#   "displayName": "terraform-sp",
#   "password": "YOUR_PASSWORD",
#   "tenant": "00000000-0000-0000-0000-000000000000"
# }

# 8. Set environment variables (optional)
# PowerShell
$env:ARM_CLIENT_ID="00000000-0000-0000-0000-000000000000"
$env:ARM_CLIENT_SECRET="YOUR_PASSWORD"
$env:ARM_SUBSCRIPTION_ID="00000000-0000-0000-0000-000000000000"
$env:ARM_TENANT_ID="00000000-0000-0000-0000-000000000000"

# Bash
export ARM_CLIENT_ID="00000000-0000-0000-0000-000000000000"
export ARM_CLIENT_SECRET="YOUR_PASSWORD"
export ARM_SUBSCRIPTION_ID="00000000-0000-0000-0000-000000000000"
export ARM_TENANT_ID="00000000-0000-0000-0000-000000000000"

# 9. Install VS Code extensions (optional)
# - HashiCorp Terraform
# - Azure Terraform
```

**Validation:**
```bash
terraform version
az account show
```

---

## 🟢 Beginner Exercises

### **Exercise 1: First Terraform Project**

**Objective:** Create your first Azure resource with Terraform

**Steps:**

```bash
# 1. Create project directory
mkdir terraform-exercise-01
cd terraform-exercise-01
```

```hcl
# 2. Create main.tf
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "example" {
  name     = "rg-terraform-lab-01"
  location = "East US"
  
  tags = {
    Environment = "Learning"
    Lab         = "01"
  }
}

output "resource_group_id" {
  value = azurerm_resource_group.example.id
}
```

```bash
# 3. Initialize
terraform init

# 4. Format
terraform fmt

# 5. Validate
terraform validate

# 6. Plan
terraform plan

# 7. Apply
terraform apply

# Type 'yes' when prompted

# 8. Show state
terraform show

# 9. List state
terraform state list

# 10. Destroy
terraform destroy

# Type 'yes' when prompted
```

**Validation:**
```bash
# Check Azure portal or CLI
az group list --output table
```

**Challenge:**
- Add more tags
- Change location to "West Europe"
- Add another resource group
- Use variables for name and location

---

### **Exercise 2: Variables and Outputs**

**Objective:** Use variables to make configurations flexible

**Steps:**

```hcl
# variables.tf
variable "resource_group_name" {
  description = "Name of the resource group"
  type        = string
  default     = "rg-terraform-lab-02"
}

variable "location" {
  description = "Azure region"
  type        = string
  default     = "East US"
  
  validation {
    condition     = contains(["East US", "West US", "West Europe"], var.location)
    error_message = "Location must be East US, West US, or West Europe."
  }
}

variable "environment" {
  description = "Environment name"
  type        = string
}

variable "tags" {
  description = "Resource tags"
  type        = map(string)
  default = {
    ManagedBy = "Terraform"
  }
}
```

```hcl
# main.tf
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "example" {
  name     = var.resource_group_name
  location = var.location
  
  tags = merge(
    var.tags,
    {
      Environment = var.environment
      Lab         = "02"
    }
  )
}

resource "azurerm_storage_account" "example" {
  name                     = "stlab02${random_integer.suffix.result}"
  resource_group_name      = azurerm_resource_group.example.name
  location                 = azurerm_resource_group.example.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
  
  tags = azurerm_resource_group.example.tags
}

resource "random_integer" "suffix" {
  min = 10000
  max = 99999
}
```

```hcl
# outputs.tf
output "resource_group_name" {
  description = "Resource group name"
  value       = azurerm_resource_group.example.name
}

output "resource_group_id" {
  description = "Resource group ID"
  value       = azurerm_resource_group.example.id
}

output "storage_account_name" {
  description = "Storage account name"
  value       = azurerm_storage_account.example.name
}

output "storage_account_id" {
  description = "Storage account ID"
  value       = azurerm_storage_account.example.id
}
```

```hcl
# terraform.tfvars
environment = "Development"

location = "East US"

tags = {
  ManagedBy  = "Terraform"
  CostCenter = "Engineering"
  Project    = "Learning"
}
```

```bash
# Apply
terraform init
terraform apply

# Apply with specific var file
terraform apply -var-file="prod.tfvars"

# Apply with command-line variable
terraform apply -var="environment=Production"
```

**Challenge:**
- Add more variable types (list, object)
- Create multiple tfvars files (dev, staging, prod)
- Add validation rules
- Use sensitive variables

---

### **Exercise 3: Remote State with Azure Storage**

**Objective:** Configure remote state backend

**Steps:**

```bash
# 1. Create storage account for state
az group create \
  --name rg-terraform-state \
  --location eastus

az storage account create \
  --name tfstatelab03 \
  --resource-group rg-terraform-state \
  --location eastus \
  --sku Standard_LRS \
  --encryption-services blob

az storage container create \
  --name tfstate \
  --account-name tfstatelab03

# Get access key
az storage account keys list \
  --resource-group rg-terraform-state \
  --account-name tfstatelab03 \
  --query '[0].value' \
  --output tsv
```

```hcl
# 2. Create backend.tf
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "tfstatelab03"
    container_name       = "tfstate"
    key                  = "lab03.terraform.tfstate"
  }
}
```

```bash
# 3. Initialize with backend
terraform init

# If migrating from local state
terraform init -migrate-state

# 4. Apply changes
terraform apply

# 5. Verify state is remote
# Check Azure portal - Storage Account -> Containers -> tfstate
# You should see lab03.terraform.tfstate
```

**Validation:**
```bash
# Pull state
terraform state pull

# State should be in Azure, not local
ls -la
# No terraform.tfstate file should exist
```

**Challenge:**
- Enable versioning on blob
- Add state locking
- Create separate backends for different environments
- Encrypt state file

---

## 🟡 Intermediate Exercises

### **Exercise 4: Complete Network Infrastructure**

**Objective:** Deploy hub-spoke network topology

**Solution:**

```hcl
# main.tf
# Resource Group
resource "azurerm_resource_group" "network" {
  name     = "rg-network-lab-04"
  location = var.location
  tags     = local.common_tags
}

# Hub VNet
resource "azurerm_virtual_network" "hub" {
  name                = "vnet-hub"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.network.location
  resource_group_name = azurerm_resource_group.network.name
  tags                = local.common_tags
}

# Hub Subnets
resource "azurerm_subnet" "firewall" {
  name                 = "AzureFirewallSubnet"
  resource_group_name  = azurerm_resource_group.network.name
  virtual_network_name = azurerm_virtual_network.hub.name
  address_prefixes     = ["10.0.1.0/26"]
}

resource "azurerm_subnet" "gateway" {
  name                 = "GatewaySubnet"
  resource_group_name  = azurerm_resource_group.network.name
  virtual_network_name = azurerm_virtual_network.hub.name
  address_prefixes     = ["10.0.2.0/27"]
}

# Spoke VNets
resource "azurerm_virtual_network" "spokes" {
  for_each = var.spokes
  
  name                = "vnet-${each.key}"
  address_space       = [each.value.address_space]
  location            = azurerm_resource_group.network.location
  resource_group_name = azurerm_resource_group.network.name
  tags                = local.common_tags
}

# Spoke Subnets
resource "azurerm_subnet" "spoke_subnets" {
  for_each = local.spoke_subnets
  
  name                 = each.value.subnet_name
  resource_group_name  = azurerm_resource_group.network.name
  virtual_network_name = azurerm_virtual_network.spokes[each.value.spoke_name].name
  address_prefixes     = [each.value.address_prefix]
}

# VNet Peering: Hub to Spoke
resource "azurerm_virtual_network_peering" "hub_to_spoke" {
  for_each = var.spokes
  
  name                      = "hub-to-${each.key}"
  resource_group_name       = azurerm_resource_group.network.name
  virtual_network_name      = azurerm_virtual_network.hub.name
  remote_virtual_network_id = azurerm_virtual_network.spokes[each.key].id
  
  allow_forwarded_traffic   = true
  allow_gateway_transit     = true
  use_remote_gateways       = false
}

# VNet Peering: Spoke to Hub
resource "azurerm_virtual_network_peering" "spoke_to_hub" {
  for_each = var.spokes
  
  name                      = "${each.key}-to-hub"
  resource_group_name       = azurerm_resource_group.network.name
  virtual_network_name      = azurerm_virtual_network.spokes[each.key].name
  remote_virtual_network_id = azurerm_virtual_network.hub.id
  
  allow_forwarded_traffic   = true
  allow_gateway_transit     = false
  use_remote_gateways       = false
}

# Network Security Groups
resource "azurerm_network_security_group" "web" {
  name                = "nsg-web"
  location            = azurerm_resource_group.network.location
  resource_group_name = azurerm_resource_group.network.name
  
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
```

```hcl
# variables.tf
variable "location" {
  description = "Azure region"
  type        = string
  default     = "East US"
}

variable "spokes" {
  description = "Spoke VNets configuration"
  type = map(object({
    address_space = string
    subnets = map(object({
      address_prefix = string
    }))
  }))
  default = {
    prod = {
      address_space = "10.1.0.0/16"
      subnets = {
        web  = { address_prefix = "10.1.1.0/24" }
        app  = { address_prefix = "10.1.2.0/24" }
        data = { address_prefix = "10.1.3.0/24" }
      }
    }
    dev = {
      address_space = "10.2.0.0/16"
      subnets = {
        web  = { address_prefix = "10.2.1.0/24" }
        app  = { address_prefix = "10.2.2.0/24" }
        data = { address_prefix = "10.2.3.0/24" }
      }
    }
  }
}
```

```hcl
# locals.tf
locals {
  common_tags = {
    ManagedBy   = "Terraform"
    Environment = "Lab"
    Lab         = "04"
  }
  
  # Flatten spoke subnets for iteration
  spoke_subnets = merge([
    for spoke_key, spoke in var.spokes : {
      for subnet_key, subnet in spoke.subnets :
      "${spoke_key}-${subnet_key}" => {
        spoke_name     = spoke_key
        subnet_name    = subnet_key
        address_prefix = subnet.address_prefix
      }
    }
  ]...)
}
```

```hcl
# outputs.tf
output "hub_vnet_id" {
  value = azurerm_virtual_network.hub.id
}

output "spoke_vnet_ids" {
  value = {
    for k, v in azurerm_virtual_network.spokes : k => v.id
  }
}

output "spoke_subnet_ids" {
  value = {
    for k, v in azurerm_subnet.spoke_subnets : k => v.id
  }
}
```

**Challenge:**
- Add Azure Firewall
- Configure route tables
- Add VPN Gateway
- Implement service endpoints

---

### **Exercise 5: Reusable Module - Azure VM**

**Objective:** Create a reusable VM module

**Module Structure:**
```
modules/
└── azure-vm/
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    └── README.md
```

```hcl
# modules/azure-vm/variables.tf
variable "vm_name" {
  description = "Name of the virtual machine"
  type        = string
}

variable "resource_group_name" {
  description = "Resource group name"
  type        = string
}

variable "location" {
  description = "Azure region"
  type        = string
}

variable "vm_size" {
  description = "VM size"
  type        = string
  default     = "Standard_B2s"
}

variable "subnet_id" {
  description = "Subnet ID"
  type        = string
}

variable "os_type" {
  description = "OS type (Windows or Linux)"
  type        = string
  default     = "Linux"
  
  validation {
    condition     = contains(["Windows", "Linux"], var.os_type)
    error_message = "OS type must be Windows or Linux."
  }
}

variable "admin_username" {
  description = "Admin username"
  type        = string
  default     = "azureuser"
}

variable "admin_password" {
  description = "Admin password (required for Windows)"
  type        = string
  sensitive   = true
  default     = null
}

variable "ssh_public_key" {
  description = "SSH public key (required for Linux)"
  type        = string
  default     = null
}

variable "create_public_ip" {
  description = "Create public IP"
  type        = bool
  default     = false
}

variable "tags" {
  description = "Resource tags"
  type        = map(string)
  default     = {}
}
```

```hcl
# modules/azure-vm/main.tf
# Public IP (conditional)
resource "azurerm_public_ip" "this" {
  count               = var.create_public_ip ? 1 : 0
  name                = "pip-${var.vm_name}"
  location            = var.location
  resource_group_name = var.resource_group_name
  allocation_method   = "Static"
  sku                 = "Standard"
  tags                = var.tags
}

# Network Interface
resource "azurerm_network_interface" "this" {
  name                = "nic-${var.vm_name}"
  location            = var.location
  resource_group_name = var.resource_group_name
  
  ip_configuration {
    name                          = "internal"
    subnet_id                     = var.subnet_id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = var.create_public_ip ? azurerm_public_ip.this[0].id : null
  }
  
  tags = var.tags
}

# Linux VM
resource "azurerm_linux_virtual_machine" "this" {
  count               = var.os_type == "Linux" ? 1 : 0
  name                = var.vm_name
  resource_group_name = var.resource_group_name
  location            = var.location
  size                = var.vm_size
  admin_username      = var.admin_username
  
  network_interface_ids = [
    azurerm_network_interface.this.id,
  ]
  
  admin_ssh_key {
    username   = var.admin_username
    public_key = var.ssh_public_key
  }
  
  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Premium_LRS"
  }
  
  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts-gen2"
    version   = "latest"
  }
  
  tags = var.tags
}

# Windows VM
resource "azurerm_windows_virtual_machine" "this" {
  count               = var.os_type == "Windows" ? 1 : 0
  name                = var.vm_name
  resource_group_name = var.resource_group_name
  location            = var.location
  size                = var.vm_size
  admin_username      = var.admin_username
  admin_password      = var.admin_password
  
  network_interface_ids = [
    azurerm_network_interface.this.id,
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
  
  tags = var.tags
}
```

```hcl
# modules/azure-vm/outputs.tf
output "vm_id" {
  description = "Virtual machine ID"
  value       = var.os_type == "Linux" ? azurerm_linux_virtual_machine.this[0].id : azurerm_windows_virtual_machine.this[0].id
}

output "vm_name" {
  description = "Virtual machine name"
  value       = var.vm_name
}

output "private_ip" {
  description = "Private IP address"
  value       = azurerm_network_interface.this.private_ip_address
}

output "public_ip" {
  description = "Public IP address"
  value       = var.create_public_ip ? azurerm_public_ip.this[0].ip_address : null
}
```

**Using the Module:**
```hcl
# main.tf
module "web_vm" {
  source = "./modules/azure-vm"
  
  vm_name             = "vm-web-01"
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location
  vm_size             = "Standard_D2s_v3"
  subnet_id           = azurerm_subnet.web.id
  os_type             = "Linux"
  admin_username      = "webadmin"
  ssh_public_key      = file("~/.ssh/id_rsa.pub")
  create_public_ip    = true
  
  tags = {
    Role = "WebServer"
  }
}

module "db_vm" {
  source = "./modules/azure-vm"
  
  vm_name             = "vm-db-01"
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location
  vm_size             = "Standard_D4s_v3"
  subnet_id           = azurerm_subnet.data.id
  os_type             = "Windows"
  admin_username      = "dbadmin"
  admin_password      = var.db_admin_password
  create_public_ip    = false
  
  tags = {
    Role = "Database"
  }
}

output "web_vm_ip" {
  value = module.web_vm.public_ip
}

output "db_vm_private_ip" {
  value = module.db_vm.private_ip
}
```

**Challenge:**
- Add data disk support
- Add custom script extension
- Version the module with Git tags
- Publish to private registry

---

## 🔴 Advanced Exercises

### **Exercise 6: Production AKS Cluster**

**Objective:** Deploy production-ready AKS with all features

**Components:**
- AKS cluster with multiple node pools
- Azure Container Registry
- Azure AD integration
- Log Analytics monitoring
- Key Vault integration
- Network policies

**Implementation:** (See [terraform-learning-roadmap.md](terraform-learning-roadmap.md) Week 8 for complete code)

**Challenge:**
- Add Azure Application Gateway ingress
- Configure Pod Identity
- Set up GitOps with Flux
- Add Azure Policy for Kubernetes

---

### **Exercise 7: Multi-Environment with Workspaces**

**Objective:** Manage multiple environments

**Setup:**
```bash
# Create workspaces
terraform workspace new dev
terraform workspace new staging
terraform workspace new prod

# List workspaces
terraform workspace list

# Select workspace
terraform workspace select dev
```

```hcl
# Use workspace in configuration
locals {
  environment = terraform.workspace
  
  vm_size = {
    dev     = "Standard_B2s"
    staging = "Standard_D2s_v3"
    prod    = "Standard_D4s_v3"
  }
  
  vm_count = {
    dev     = 1
    staging = 2
    prod    = 3
  }
}

resource "azurerm_resource_group" "example" {
  name     = "rg-${local.environment}"
  location = var.location
  
  tags = {
    Environment = local.environment
  }
}

resource "azurerm_linux_virtual_machine" "vms" {
  count               = local.vm_count[local.environment]
  name                = "vm-${local.environment}-${count.index + 1}"
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location
  size                = local.vm_size[local.environment]
  # ...
}
```

**Challenge:**
- Deploy to all three environments
- Compare resource costs
- Implement environment-specific configurations
- Create promotion workflow

---

### **Exercise 8: CI/CD with Azure DevOps**

**Objective:** Automate Terraform deployments

**Setup:** (See [terraform-learning-roadmap.md](terraform-learning-roadmap.md) Week 9 for complete pipeline)

**Tasks:**
- Create Azure DevOps project
- Set up service connection
- Create pipeline with stages
- Add approval gates
- Implement automated tests

---

## 🎯 Final Project: Complete Production Infrastructure

**Objective:** Deploy end-to-end production infrastructure

**Requirements:**
1. Multi-region hub-spoke network
2. AKS cluster with monitoring
3. Azure SQL with private endpoint
4. Application Gateway
5. Key Vault with secrets
6. Storage accounts with lifecycle policies
7. Log Analytics workspace
8. CI/CD pipeline
9. Cost management tags
10. Complete documentation

**Duration:** 2-3 days

**Deliverables:**
- [ ] Working Terraform code
- [ ] Modular structure
- [ ] Remote state
- [ ] CI/CD pipeline
- [ ] Architecture diagram
- [ ] Cost estimate
- [ ] Security assessment
- [ ] README documentation

---

**Practice makes perfect! Complete all exercises for mastery! 🚀☁️**

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

