# Terraform - Infrastructure as Code (Azure Focused)

Complete guide to mastering Terraform for Azure infrastructure automation, from beginner to production expert.

---

## 📊 What is Terraform?

**Terraform** is an open-source Infrastructure as Code (IaC) tool created by HashiCorp that enables you to define, provision, and manage infrastructure using declarative configuration files.

**Why Terraform?**
- ✅ **Cloud-Agnostic**: Works with Azure, AWS, GCP, and 100+ providers
- ✅ **Declarative**: Define desired state, Terraform handles the rest
- ✅ **Version Control**: Infrastructure as code in Git
- ✅ **Plan Before Apply**: Preview changes before execution
- ✅ **State Management**: Track infrastructure state
- ✅ **Modularity**: Reusable, shareable modules
- ✅ **Collaboration**: Team workflows with remote state
- ✅ **Immutable Infrastructure**: Replace rather than modify

**Azure-Specific Benefits:**
- Native Azure provider (AzureRM)
- Support for all Azure services
- Azure AD integration
- Azure DevOps integration
- Terraform Cloud/Enterprise support

---

## 🎯 Terraform Workflow

```
┌─────────────────────────────────────────────────┐
│                                                 │
│  1. Write → 2. Plan → 3. Apply → 4. Manage    │
│                                                 │
└─────────────────────────────────────────────────┘

┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ Write .tf    │────▶│ terraform    │────▶│ terraform    │
│ files        │     │ plan         │     │ apply        │
└──────────────┘     └──────────────┘     └──────────────┘
                             │                    │
                             ▼                    ▼
                      ┌──────────────┐     ┌──────────────┐
                      │ Review       │     │ Provision    │
                      │ Changes      │     │ Resources    │
                      └──────────────┘     └──────────────┘
```

**Commands:**
```bash
# Initialize - Download providers & modules
terraform init

# Format - Clean up code formatting
terraform fmt

# Validate - Check configuration syntax
terraform validate

# Plan - Preview infrastructure changes
terraform plan

# Apply - Execute the changes
terraform apply

# Destroy - Remove infrastructure
terraform destroy
```

---

## 🚀 Quick Start - Azure

### **Prerequisites**

```bash
# 1. Install Terraform
# Windows (Chocolatey)
choco install terraform

# macOS (Homebrew)
brew install terraform

# Linux
wget https://releases.hashicorp.com/terraform/1.7.0/terraform_1.7.0_linux_amd64.zip
unzip terraform_1.7.0_linux_amd64.zip
sudo mv terraform /usr/local/bin/

# Verify
terraform version
```

```bash
# 2. Install Azure CLI
# Windows
winget install Microsoft.AzureCLI

# macOS
brew install azure-cli

# Linux
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Login to Azure
az login

# Set subscription
az account set --subscription "YOUR_SUBSCRIPTION_ID"
```

### **First Terraform Project**

```bash
# Create project directory
mkdir azure-terraform-demo
cd azure-terraform-demo
```

```hcl
# main.tf
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

# Resource Group
resource "azurerm_resource_group" "example" {
  name     = "rg-terraform-demo"
  location = "East US"
  
  tags = {
    Environment = "Development"
    ManagedBy   = "Terraform"
  }
}

# Virtual Network
resource "azurerm_virtual_network" "example" {
  name                = "vnet-demo"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
  
  tags = {
    Environment = "Development"
  }
}

# Subnet
resource "azurerm_subnet" "example" {
  name                 = "subnet-demo"
  resource_group_name  = azurerm_resource_group.example.name
  virtual_network_name = azurerm_virtual_network.example.name
  address_prefixes     = ["10.0.1.0/24"]
}

# Output
output "resource_group_name" {
  value = azurerm_resource_group.example.name
}

output "vnet_id" {
  value = azurerm_virtual_network.example.id
}
```

```bash
# Initialize
terraform init

# Preview changes
terraform plan

# Apply changes
terraform apply

# Destroy when done
terraform destroy
```

---

## 📚 Core Concepts

### **1. Providers**

Providers are plugins that interact with cloud platforms and services.

```hcl
# Azure Provider
provider "azurerm" {
  features {}
  
  subscription_id = "00000000-0000-0000-0000-000000000000"
  tenant_id       = "00000000-0000-0000-0000-000000000000"
  
  # Optional: Skip provider registration
  skip_provider_registration = true
}

# Alternative authentication
provider "azurerm" {
  features {}
  
  # Service Principal
  client_id       = var.client_id
  client_secret   = var.client_secret
  subscription_id = var.subscription_id
  tenant_id       = var.tenant_id
}

# Multiple providers (different regions)
provider "azurerm" {
  alias = "eastus"
  features {}
}

provider "azurerm" {
  alias = "westus"
  features {}
}
```

### **2. Resources**

Resources are infrastructure components you manage.

```hcl
# Basic resource
resource "azurerm_storage_account" "example" {
  name                     = "storageacctdemo"
  resource_group_name      = azurerm_resource_group.example.name
  location                 = azurerm_resource_group.example.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
  
  tags = {
    Environment = "Development"
  }
}

# Resource with dependencies
resource "azurerm_network_interface" "example" {
  name                = "nic-vm-demo"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
  
  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.example.id
    private_ip_address_allocation = "Dynamic"
  }
  
  # Explicit dependency
  depends_on = [azurerm_subnet.example]
}
```

### **3. Variables**

Variables make configurations flexible and reusable.

```hcl
# variables.tf
variable "location" {
  description = "Azure region for resources"
  type        = string
  default     = "East US"
}

variable "environment" {
  description = "Environment name"
  type        = string
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "vm_size" {
  description = "VM size"
  type        = string
  default     = "Standard_B2s"
}

variable "tags" {
  description = "Resource tags"
  type        = map(string)
  default = {
    ManagedBy = "Terraform"
  }
}

# Usage in resources
resource "azurerm_resource_group" "example" {
  name     = "rg-${var.environment}"
  location = var.location
  tags     = var.tags
}
```

**Variable Files:**
```hcl
# terraform.tfvars
location    = "West Europe"
environment = "prod"
vm_size     = "Standard_D2s_v3"

tags = {
  ManagedBy   = "Terraform"
  CostCenter  = "Engineering"
  Environment = "Production"
}
```

### **4. Outputs**

Outputs expose values from your infrastructure.

```hcl
# outputs.tf
output "resource_group_id" {
  description = "Resource group ID"
  value       = azurerm_resource_group.example.id
}

output "vnet_address_space" {
  description = "Virtual network address space"
  value       = azurerm_virtual_network.example.address_space
}

output "vm_private_ip" {
  description = "VM private IP address"
  value       = azurerm_network_interface.example.private_ip_address
  sensitive   = false
}

output "connection_string" {
  description = "Database connection string"
  value       = azurerm_sql_server.example.fully_qualified_domain_name
  sensitive   = true
}
```

### **5. State**

State tracks your infrastructure and maps resources to configuration.

```hcl
# Backend configuration for remote state (Azure Storage)
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "tfstatestorageacct"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}
```

```bash
# State commands
terraform state list                    # List resources
terraform state show <resource>         # Show resource details
terraform state mv <source> <dest>      # Rename resource
terraform state rm <resource>           # Remove from state
terraform state pull                    # Download state
terraform state push                    # Upload state
```

### **6. Modules**

Modules are reusable, shareable Terraform configurations.

```hcl
# Module structure
modules/
├── azure-vm/
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf

# Using a module
module "web_vm" {
  source = "./modules/azure-vm"
  
  vm_name             = "vm-web-01"
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location
  subnet_id           = azurerm_subnet.web.id
  vm_size             = "Standard_B2s"
  
  tags = {
    Role = "WebServer"
  }
}

# Access module outputs
output "web_vm_ip" {
  value = module.web_vm.private_ip
}
```

---

## 🔧 Common Azure Patterns

### **Pattern 1: Hub-Spoke Network**

```hcl
# Hub VNet
resource "azurerm_virtual_network" "hub" {
  name                = "vnet-hub"
  address_space       = ["10.0.0.0/16"]
  location            = var.location
  resource_group_name = azurerm_resource_group.network.name
}

# Spoke VNet
resource "azurerm_virtual_network" "spoke" {
  name                = "vnet-spoke-${var.environment}"
  address_space       = ["10.1.0.0/16"]
  location            = var.location
  resource_group_name = azurerm_resource_group.network.name
}

# VNet Peering
resource "azurerm_virtual_network_peering" "hub_to_spoke" {
  name                      = "hub-to-spoke"
  resource_group_name       = azurerm_resource_group.network.name
  virtual_network_name      = azurerm_virtual_network.hub.name
  remote_virtual_network_id = azurerm_virtual_network.spoke.id
}

resource "azurerm_virtual_network_peering" "spoke_to_hub" {
  name                      = "spoke-to-hub"
  resource_group_name       = azurerm_resource_group.network.name
  virtual_network_name      = azurerm_virtual_network.spoke.name
  remote_virtual_network_id = azurerm_virtual_network.hub.id
}
```

### **Pattern 2: AKS Cluster**

```hcl
resource "azurerm_kubernetes_cluster" "example" {
  name                = "aks-${var.environment}"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
  dns_prefix          = "aks-${var.environment}"
  
  default_node_pool {
    name       = "default"
    node_count = 3
    vm_size    = "Standard_D2s_v3"
    vnet_subnet_id = azurerm_subnet.aks.id
  }
  
  identity {
    type = "SystemAssigned"
  }
  
  network_profile {
    network_plugin = "azure"
    network_policy = "calico"
  }
}
```

---

## 📚 Learning Resources

### **📖 Documentation**
1. [Learning Roadmap](terraform-learning-roadmap.md) - 12-week structured curriculum
2. [Quick Reference](terraform-quick-reference.md) - Commands & syntax
3. [Hands-On Exercises](terraform-hands-on-exercises.md) - 30+ practical labs
4. [Troubleshooting Guide](terraform-troubleshooting-guide.md) - Common issues
5. [Interview Questions](terraform-interview-questions.md) - 80+ questions

### **🔗 Official Resources**
- [Terraform Documentation](https://www.terraform.io/docs/)
- [Azure Provider Documentation](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs)
- [Terraform Registry](https://registry.terraform.io/)
- [Learn Terraform](https://learn.hashicorp.com/terraform)

---

## 🎯 Next Steps

1. **Start Learning**: Follow the [Learning Roadmap](terraform-learning-roadmap.md)
2. **Practice**: Complete [Hands-On Exercises](terraform-hands-on-exercises.md)
3. **Reference**: Use [Quick Reference](terraform-quick-reference.md) guide
4. **Troubleshoot**: Check [Troubleshooting Guide](terraform-troubleshooting-guide.md)
5. **Interview Prep**: Study [Interview Questions](terraform-interview-questions.md)

---

**Master Infrastructure as Code with Terraform & Azure! 🚀☁️**

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

