# Terraform Learning Roadmap (Azure Focused)

**12-Week Comprehensive Learning Path from Beginner to Production Expert**

For experienced DevOps engineers (7+ years) looking to master Infrastructure as Code with Terraform and Azure.

---

## 📋 Prerequisites

- ✅ Azure fundamentals and portal navigation
- ✅ Basic networking concepts (VNets, Subnets, NSGs)
- ✅ Command-line proficiency (PowerShell/Bash)
- ✅ Version control with Git
- ✅ Understanding of cloud infrastructure
- ✅ Azure CLI installed and configured

---

## 🎯 Learning Objectives

By the end of this roadmap, you will be able to:

✅ Write production-grade Terraform configurations  
✅ Design reusable, maintainable modules  
✅ Implement Azure best practices with Terraform  
✅ Manage state in team environments  
✅ Create CI/CD pipelines for infrastructure  
✅ Implement security and compliance  
✅ Troubleshoot complex Terraform issues  
✅ Optimize costs and performance  
✅ Lead infrastructure automation projects  
✅ Design multi-environment architectures  

---

## 🗓️ 12-Week Learning Plan

---

## **Week 1-2: Terraform Fundamentals & Azure Basics**

### **Week 1: Getting Started**

**Topics:**
- Infrastructure as Code concepts
- Terraform architecture
- HCL (HashiCorp Configuration Language)
- Installation and setup
- Azure provider basics

**Hands-On:**
```bash
# Install Terraform
choco install terraform  # Windows
brew install terraform   # macOS

# Install Azure CLI
az login
az account set --subscription "YOUR_SUBSCRIPTION"

# Create first project
mkdir terraform-learning
cd terraform-learning
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
  name     = "rg-learning"
  location = "East US"
}
```

```bash
# Initialize and apply
terraform init
terraform plan
terraform apply
terraform destroy
```

**Practice Tasks:**
- [ ] Install Terraform and Azure CLI
- [ ] Create Azure service principal
- [ ] Deploy first resource group
- [ ] Understand terraform workflow
- [ ] Explore state file

**Resources:**
- [Terraform Documentation](https://www.terraform.io/docs/)
- [Azure Provider](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs)

---

### **Week 2: HCL Syntax & Basic Resources**

**Topics:**
- HCL syntax and structure
- Resource blocks
- Data sources
- Variables and outputs
- Basic Azure resources

**HCL Fundamentals:**
```hcl
# Variables
variable "location" {
  description = "Azure region"
  type        = string
  default     = "East US"
}

variable "tags" {
  description = "Resource tags"
  type        = map(string)
  default = {
    Environment = "Dev"
    ManagedBy   = "Terraform"
  }
}

# Resource with variables
resource "azurerm_storage_account" "example" {
  name                     = "staccountdemo${random_integer.suffix.result}"
  resource_group_name      = azurerm_resource_group.example.name
  location                 = var.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
  tags                     = var.tags
}

# Random suffix
resource "random_integer" "suffix" {
  min = 10000
  max = 99999
}

# Data source
data "azurerm_client_config" "current" {}

# Output
output "storage_account_name" {
  value = azurerm_storage_account.example.name
}
```

**Practice Project: Basic Infrastructure**
```hcl
# Deploy:
# - Resource Group
# - Virtual Network
# - Subnet
# - Network Security Group
# - Storage Account
```

**Practice Tasks:**
- [ ] Create variables file
- [ ] Use multiple resource types
- [ ] Implement data sources
- [ ] Add outputs
- [ ] Use terraform.tfvars

---

## **Week 3-4: State Management & Workspaces**

### **Week 3: Understanding State**

**Topics:**
- State file structure
- Local vs remote state
- State locking
- Azure Storage backend
- State commands

**Local State:**
```bash
# State commands
terraform state list
terraform state show azurerm_resource_group.example
terraform state mv azurerm_resource_group.old azurerm_resource_group.new
terraform state rm azurerm_resource_group.example
terraform state pull > state.json
```

**Remote State (Azure Storage):**
```bash
# Create storage account for state
az group create --name rg-terraform-state --location eastus

az storage account create \
  --name tfstatestorage123 \
  --resource-group rg-terraform-state \
  --location eastus \
  --sku Standard_LRS

az storage container create \
  --name tfstate \
  --account-name tfstatestorage123

az storage account keys list \
  --resource-group rg-terraform-state \
  --account-name tfstatestorage123
```

```hcl
# backend.tf
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "tfstatestorage123"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}
```

```bash
# Initialize with backend
terraform init -backend-config="access_key=YOUR_ACCESS_KEY"

# Migrate from local to remote
terraform init -migrate-state
```

**Practice Tasks:**
- [ ] Set up Azure Storage backend
- [ ] Configure state locking
- [ ] Practice state commands
- [ ] Migrate state files
- [ ] Import existing resources

---

### **Week 4: Workspaces & Environments**

**Topics:**
- Terraform workspaces
- Environment management
- Workspace strategies
- Backend per environment

**Workspaces:**
```bash
# Create workspaces
terraform workspace new dev
terraform workspace new staging
terraform workspace new prod

# List workspaces
terraform workspace list

# Switch workspace
terraform workspace select dev

# Current workspace
terraform workspace show
```

**Using Workspaces in Code:**
```hcl
locals {
  environment = terraform.workspace
  
  vm_size = {
    dev     = "Standard_B2s"
    staging = "Standard_D2s_v3"
    prod    = "Standard_D4s_v3"
  }
}

resource "azurerm_resource_group" "example" {
  name     = "rg-${local.environment}"
  location = var.location
  
  tags = {
    Environment = local.environment
  }
}

resource "azurerm_virtual_machine" "example" {
  # ...
  vm_size = local.vm_size[local.environment]
}
```

**Alternative: Separate Backends**
```hcl
# environments/dev/backend.tf
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "tfstatestorage123"
    container_name       = "tfstate"
    key                  = "dev.terraform.tfstate"
  }
}

# environments/prod/backend.tf
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "tfstatestorage123"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}
```

**Practice Tasks:**
- [ ] Create dev/staging/prod workspaces
- [ ] Deploy to multiple environments
- [ ] Implement environment-specific configs
- [ ] Test workspace isolation
- [ ] Document workspace strategy

---

## **Week 5-6: Modules & Code Organization**

### **Week 5: Creating Modules**

**Topics:**
- Module structure
- Input variables
- Output values
- Module versioning
- Azure module patterns

**Module Structure:**
```
modules/
├── azure-vm/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── README.md
├── azure-network/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── README.md
```

**Example Module: Azure VM**
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
  description = "Subnet ID for VM"
  type        = string
}

variable "admin_username" {
  description = "Admin username"
  type        = string
  default     = "azureuser"
}

variable "admin_password" {
  description = "Admin password"
  type        = string
  sensitive   = true
}

variable "tags" {
  description = "Resource tags"
  type        = map(string)
  default     = {}
}
```

```hcl
# modules/azure-vm/main.tf
resource "azurerm_network_interface" "this" {
  name                = "${var.vm_name}-nic"
  location            = var.location
  resource_group_name = var.resource_group_name
  
  ip_configuration {
    name                          = "internal"
    subnet_id                     = var.subnet_id
    private_ip_address_allocation = "Dynamic"
  }
  
  tags = var.tags
}

resource "azurerm_windows_virtual_machine" "this" {
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
    storage_account_type = "Standard_LRS"
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
  value       = azurerm_windows_virtual_machine.this.id
}

output "vm_name" {
  description = "Virtual machine name"
  value       = azurerm_windows_virtual_machine.this.name
}

output "private_ip" {
  description = "Private IP address"
  value       = azurerm_network_interface.this.private_ip_address
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
  admin_username      = "webadmin"
  admin_password      = var.vm_password
  
  tags = {
    Role        = "WebServer"
    Environment = "Production"
  }
}

output "web_vm_ip" {
  value = module.web_vm.private_ip
}
```

**Practice Tasks:**
- [ ] Create 3+ reusable modules
- [ ] Version modules with Git tags
- [ ] Document module usage
- [ ] Test modules in isolation
- [ ] Publish to private registry

---

### **Week 6: Advanced Module Patterns**

**Topics:**
- Module composition
- For_each and count in modules
- Dynamic blocks
- Module sources
- Registry usage

**Module Composition:**
```hcl
# High-level module using other modules
module "app_infrastructure" {
  source = "./modules/app-infrastructure"
  
  environment = "prod"
  location    = "East US"
}

# modules/app-infrastructure/main.tf
module "network" {
  source = "../azure-network"
  
  vnet_name           = "vnet-${var.environment}"
  resource_group_name = azurerm_resource_group.this.name
  location            = var.location
  address_space       = ["10.0.0.0/16"]
}

module "aks" {
  source = "../azure-aks"
  
  cluster_name        = "aks-${var.environment}"
  resource_group_name = azurerm_resource_group.this.name
  location            = var.location
  subnet_id           = module.network.subnet_ids["aks"]
}

module "database" {
  source = "../azure-sql"
  
  server_name         = "sql-${var.environment}"
  resource_group_name = azurerm_resource_group.this.name
  location            = var.location
  subnet_id           = module.network.subnet_ids["data"]
}
```

**For_each with Modules:**
```hcl
# Create multiple VMs
variable "vms" {
  type = map(object({
    size      = string
    subnet_id = string
  }))
  default = {
    web1 = {
      size      = "Standard_B2s"
      subnet_id = "subnet-id-1"
    }
    web2 = {
      size      = "Standard_B2s"
      subnet_id = "subnet-id-1"
    }
    db1 = {
      size      = "Standard_D4s_v3"
      subnet_id = "subnet-id-2"
    }
  }
}

module "vms" {
  for_each = var.vms
  
  source = "./modules/azure-vm"
  
  vm_name             = each.key
  vm_size             = each.value.size
  subnet_id           = each.value.subnet_id
  resource_group_name = azurerm_resource_group.example.name
  location            = var.location
}

output "vm_ips" {
  value = {
    for k, v in module.vms : k => v.private_ip
  }
}
```

**Practice Tasks:**
- [ ] Create composite modules
- [ ] Use for_each with modules
- [ ] Implement dynamic blocks
- [ ] Use registry modules
- [ ] Build module library

---

## **Week 7-8: Azure-Specific Advanced Patterns**

### **Week 7: Networking & Security**

**Topics:**
- Hub-spoke topology
- VNet peering
- Network Security Groups
- Azure Firewall
- Application Gateway
- Private endpoints

**Hub-Spoke Network:**
```hcl
# Hub VNet
resource "azurerm_virtual_network" "hub" {
  name                = "vnet-hub"
  address_space       = ["10.0.0.0/16"]
  location            = var.location
  resource_group_name = azurerm_resource_group.network.name
}

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
  location            = var.location
  resource_group_name = azurerm_resource_group.network.name
}

# VNet Peering
resource "azurerm_virtual_network_peering" "hub_to_spoke" {
  for_each = var.spokes
  
  name                      = "hub-to-${each.key}"
  resource_group_name       = azurerm_resource_group.network.name
  virtual_network_name      = azurerm_virtual_network.hub.name
  remote_virtual_network_id = azurerm_virtual_network.spokes[each.key].id
  allow_forwarded_traffic   = true
  allow_gateway_transit     = true
}

resource "azurerm_virtual_network_peering" "spoke_to_hub" {
  for_each = var.spokes
  
  name                      = "${each.key}-to-hub"
  resource_group_name       = azurerm_resource_group.network.name
  virtual_network_name      = azurerm_virtual_network.spokes[each.key].name
  remote_virtual_network_id = azurerm_virtual_network.hub.id
  allow_forwarded_traffic   = true
  use_remote_gateways       = false
}

# Azure Firewall
resource "azurerm_public_ip" "firewall" {
  name                = "pip-firewall"
  location            = var.location
  resource_group_name = azurerm_resource_group.network.name
  allocation_method   = "Static"
  sku                 = "Standard"
}

resource "azurerm_firewall" "example" {
  name                = "fw-hub"
  location            = var.location
  resource_group_name = azurerm_resource_group.network.name
  sku_name            = "AZFW_VNet"
  sku_tier            = "Standard"
  
  ip_configuration {
    name                 = "configuration"
    subnet_id            = azurerm_subnet.firewall.id
    public_ip_address_id = azurerm_public_ip.firewall.id
  }
}
```

**Network Security:**
```hcl
resource "azurerm_network_security_group" "web" {
  name                = "nsg-web"
  location            = var.location
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
  
  security_rule {
    name                       = "DenyAllInbound"
    priority                   = 4096
    direction                  = "Inbound"
    access                     = "Deny"
    protocol                   = "*"
    source_port_range          = "*"
    destination_port_range     = "*"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }
}

resource "azurerm_subnet_network_security_group_association" "web" {
  subnet_id                 = azurerm_subnet.web.id
  network_security_group_id = azurerm_network_security_group.web.id
}
```

**Practice Tasks:**
- [ ] Implement hub-spoke topology
- [ ] Configure Azure Firewall
- [ ] Set up NSG rules
- [ ] Create private endpoints
- [ ] Deploy Application Gateway

---

### **Week 8: AKS & Container Services**

**Topics:**
- Azure Kubernetes Service
- Container Registry
- AKS networking
- Identity management
- Monitoring integration

**Complete AKS Deployment:**
```hcl
# ACR
resource "azurerm_container_registry" "example" {
  name                = "acr${var.environment}${random_integer.suffix.result}"
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location
  sku                 = "Premium"
  admin_enabled       = false
}

# AKS
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
    docker_bridge_cidr = "172.17.0.1/16"
  }
  
  azure_active_directory_role_based_access_control {
    managed            = true
    azure_rbac_enabled = true
  }
  
  oms_agent {
    log_analytics_workspace_id = azurerm_log_analytics_workspace.example.id
  }
  
  key_vault_secrets_provider {
    secret_rotation_enabled = true
  }
}

# Additional node pool
resource "azurerm_kubernetes_cluster_node_pool" "user" {
  name                  = "user"
  kubernetes_cluster_id = azurerm_kubernetes_cluster.example.id
  vm_size               = "Standard_D4s_v3"
  node_count            = 2
  vnet_subnet_id        = azurerm_subnet.aks.id
  enable_auto_scaling   = true
  min_count             = 2
  max_count             = 10
  
  node_labels = {
    workload = "user-apps"
  }
}

# ACR role assignment
resource "azurerm_role_assignment" "aks_acr" {
  principal_id                     = azurerm_kubernetes_cluster.example.kubelet_identity[0].object_id
  role_definition_name             = "AcrPull"
  scope                            = azurerm_container_registry.example.id
  skip_service_principal_aad_check = true
}

# Log Analytics
resource "azurerm_log_analytics_workspace" "example" {
  name                = "log-aks-${var.environment}"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
  sku                 = "PerGB2018"
  retention_in_days   = 30
}
```

**Practice Tasks:**
- [ ] Deploy production-ready AKS
- [ ] Configure ACR integration
- [ ] Set up Azure AD RBAC
- [ ] Enable monitoring
- [ ] Configure autoscaling

---

## **Week 9-10: CI/CD & Best Practices**

### **Week 9: Azure DevOps Integration**

**Topics:**
- Terraform in Azure Pipelines
- Service connections
- Variable groups
- Approval gates
- Pipeline best practices

**Azure Pipeline:**
```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main
  paths:
    include:
      - terraform/**

variables:
  - group: terraform-variables
  - name: workingDirectory
    value: '$(System.DefaultWorkingDirectory)/terraform'

stages:
  - stage: Validate
    jobs:
      - job: Validate
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          - task: TerraformInstaller@0
            inputs:
              terraformVersion: '1.7.0'
          
          - task: TerraformTaskV4@4
            displayName: 'Terraform Init'
            inputs:
              provider: 'azurerm'
              command: 'init'
              workingDirectory: '$(workingDirectory)'
              backendServiceArm: 'Azure-Service-Connection'
              backendAzureRmResourceGroupName: 'rg-terraform-state'
              backendAzureRmStorageAccountName: 'tfstatestorage'
              backendAzureRmContainerName: 'tfstate'
              backendAzureRmKey: 'prod.terraform.tfstate'
          
          - task: TerraformTaskV4@4
            displayName: 'Terraform Validate'
            inputs:
              provider: 'azurerm'
              command: 'validate'
              workingDirectory: '$(workingDirectory)'
          
          - task: TerraformTaskV4@4
            displayName: 'Terraform Format Check'
            inputs:
              provider: 'azurerm'
              command: 'custom'
              customCommand: 'fmt'
              commandOptions: '-check -recursive'
              workingDirectory: '$(workingDirectory)'

  - stage: Plan
    dependsOn: Validate
    jobs:
      - job: Plan
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          - task: TerraformInstaller@0
            inputs:
              terraformVersion: '1.7.0'
          
          - task: TerraformTaskV4@4
            displayName: 'Terraform Init'
            inputs:
              provider: 'azurerm'
              command: 'init'
              workingDirectory: '$(workingDirectory)'
              backendServiceArm: 'Azure-Service-Connection'
              backendAzureRmResourceGroupName: 'rg-terraform-state'
              backendAzureRmStorageAccountName: 'tfstatestorage'
              backendAzureRmContainerName: 'tfstate'
              backendAzureRmKey: 'prod.terraform.tfstate'
          
          - task: TerraformTaskV4@4
            displayName: 'Terraform Plan'
            inputs:
              provider: 'azurerm'
              command: 'plan'
              workingDirectory: '$(workingDirectory)'
              environmentServiceNameAzureRM: 'Azure-Service-Connection'
              commandOptions: '-out=tfplan'
          
          - task: PublishPipelineArtifact@1
            inputs:
              targetPath: '$(workingDirectory)/tfplan'
              artifact: 'tfplan'

  - stage: Apply
    dependsOn: Plan
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
    jobs:
      - deployment: Apply
        pool:
          vmImage: 'ubuntu-latest'
        environment: 'Production'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: DownloadPipelineArtifact@2
                  inputs:
                    artifactName: 'tfplan'
                    downloadPath: '$(workingDirectory)'
                
                - task: TerraformInstaller@0
                  inputs:
                    terraformVersion: '1.7.0'
                
                - task: TerraformTaskV4@4
                  displayName: 'Terraform Init'
                  inputs:
                    provider: 'azurerm'
                    command: 'init'
                    workingDirectory: '$(workingDirectory)'
                    backendServiceArm: 'Azure-Service-Connection'
                    backendAzureRmResourceGroupName: 'rg-terraform-state'
                    backendAzureRmStorageAccountName: 'tfstatestorage'
                    backendAzureRmContainerName: 'tfstate'
                    backendAzureRmKey: 'prod.terraform.tfstate'
                
                - task: TerraformTaskV4@4
                  displayName: 'Terraform Apply'
                  inputs:
                    provider: 'azurerm'
                    command: 'apply'
                    workingDirectory: '$(workingDirectory)'
                    environmentServiceNameAzureRM: 'Azure-Service-Connection'
                    commandOptions: 'tfplan'
```

**Practice Tasks:**
- [ ] Set up Azure DevOps pipeline
- [ ] Configure service connections
- [ ] Implement approval gates
- [ ] Add automated testing
- [ ] Set up notifications

---

### **Week 10: Testing & Validation**

**Topics:**
- Terraform testing frameworks
- Terratest
- Policy as Code (Azure Policy)
- Compliance checking
- Security scanning

**Terratest Example:**
```go
// test/terraform_azure_test.go
package test

import (
	"testing"
	"github.com/gruntwork-io/terratest/modules/terraform"
	"github.com/stretchr/testify/assert"
)

func TestTerraformAzureResourceGroup(t *testing.T) {
	t.Parallel()
	
	terraformOptions := terraform.WithDefaultRetryableErrors(t, &terraform.Options{
		TerraformDir: "../",
		Vars: map[string]interface{}{
			"resource_group_name": "rg-test",
			"location":            "East US",
		},
	})
	
	defer terraform.Destroy(t, terraformOptions)
	terraform.InitAndApply(t, terraformOptions)
	
	resourceGroupName := terraform.Output(t, terraformOptions, "resource_group_name")
	assert.Equal(t, "rg-test", resourceGroupName)
}
```

**Azure Policy:**
```hcl
# policy.tf
resource "azurerm_resource_group_policy_assignment" "require_tags" {
  name                 = "require-tags"
  resource_group_id    = azurerm_resource_group.example.id
  policy_definition_id = "/providers/Microsoft.Authorization/policyDefinitions/96670d01-0a4d-4649-9c89-2d3abc0a5025"
  
  parameters = jsonencode({
    tagNames = {
      value = ["Environment", "CostCenter"]
    }
  })
}
```

**Practice Tasks:**
- [ ] Write Terratest tests
- [ ] Implement Azure Policies
- [ ] Set up tflint
- [ ] Use Checkov for security
- [ ] Automate compliance checks

---

## **Week 11-12: Production & Enterprise Patterns**

### **Week 11: Enterprise Architecture**

**Topics:**
- Multi-subscription strategy
- Landing zones
- Governance
- Cost management
- Disaster recovery

**Multi-Subscription Pattern:**
```hcl
# providers.tf
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
      configuration_aliases = [
        azurerm.hub,
        azurerm.prod,
        azurerm.dev,
      ]
    }
  }
}

provider "azurerm" {
  alias           = "hub"
  subscription_id = var.hub_subscription_id
  features {}
}

provider "azurerm" {
  alias           = "prod"
  subscription_id = var.prod_subscription_id
  features {}
}

provider "azurerm" {
  alias           = "dev"
  subscription_id = var.dev_subscription_id
  features {}
}

# Usage
resource "azurerm_resource_group" "hub" {
  provider = azurerm.hub
  name     = "rg-hub"
  location = "East US"
}

resource "azurerm_resource_group" "prod" {
  provider = azurerm.prod
  name     = "rg-prod"
  location = "East US"
}
```

**Practice Tasks:**
- [ ] Design landing zone
- [ ] Implement governance
- [ ] Set up cost tracking
- [ ] Create DR strategy
- [ ] Document architecture

---

### **Week 12: Final Project**

**Objective:** Deploy complete production infrastructure

**Requirements:**
1. Multi-region architecture
2. Hub-spoke network topology
3. AKS cluster with monitoring
4. Azure SQL with private endpoint
5. Application Gateway
6. Key Vault integration
7. CI/CD pipeline
8. Disaster recovery
9. Cost optimization
10. Complete documentation

**Deliverables:**
- [ ] Working Terraform code
- [ ] CI/CD pipeline
- [ ] Architecture diagram
- [ ] Documentation
- [ ] Cost analysis
- [ ] Security assessment

---

## 📚 Additional Resources

### **Books:**
- "Terraform: Up & Running" by Yevgeniy Brikman
- "Infrastructure as Code" by Kief Morris

### **Certifications:**
- HashiCorp Certified: Terraform Associate
- Azure Administrator Associate (AZ-104)
- Azure Solutions Architect Expert (AZ-305)

### **Communities:**
- HashiCorp Discuss Forum
- Terraform Azure Provider GitHub
- Azure Terraform Community

---

## 🎯 Validation & Next Steps

After completing this roadmap:

**Skills Validation:**
- [ ] Can write production Terraform code
- [ ] Understand state management
- [ ] Can create reusable modules
- [ ] Implement CI/CD pipelines
- [ ] Follow Azure best practices
- [ ] Troubleshoot complex issues
- [ ] Lead infrastructure projects

**Next Steps:**
1. Get HashiCorp Terraform Associate certification
2. Contribute to open-source Terraform modules
3. Mentor junior team members
4. Build company Terraform library
5. Explore Terraform Cloud/Enterprise
6. Learn advanced patterns (Terragrunt, etc.)

---

**Master Infrastructure as Code! Start your journey today! 🚀☁️**

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

