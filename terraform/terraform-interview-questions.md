# Terraform Interview Questions (Azure Focused)

**80+ Interview Questions from Basic to Advanced Production Level**

For experienced professionals (7+ years) working with Infrastructure as Code and Azure.

---

## 📋 Question Categories

1. **Beginner (20 questions)** - Fundamentals & basics
2. **Intermediate (25 questions)** - State, modules, Azure resources
3. **Advanced (25 questions)** - Enterprise patterns & best practices
4. **Expert (10 questions)** - Architecture & complex scenarios

---

## 🟢 Beginner Level Questions

### **Q1: What is Terraform and why use it?**

**Answer:**
Terraform is an open-source Infrastructure as Code (IaC) tool by HashiCorp that allows you to define, provision, and manage infrastructure using declarative configuration files.

**Key Benefits:**
1. **Declarative**: Define desired state, Terraform handles the how
2. **Cloud-Agnostic**: Works with 100+ providers (Azure, AWS, GCP, etc.)
3. **Version Control**: Infrastructure as code in Git
4. **Plan Before Apply**: Preview changes before execution
5. **State Management**: Tracks infrastructure state
6. **Modularity**: Reusable components
7. **Collaboration**: Team workflows with remote state

**Why Terraform over alternatives:**
- More providers than CloudFormation (Azure-only) or ARM templates
- Better community and module ecosystem
- Consistent workflow across clouds
- Immutable infrastructure approach

---

### **Q2: Explain the Terraform workflow.**

**Answer:**

**Workflow Steps:**
```
Write → Init → Plan → Apply → Manage
```

**1. Write**
```hcl
# main.tf
resource "azurerm_resource_group" "example" {
  name     = "rg-example"
  location = "East US"
}
```

**2. Initialize (`terraform init`)**
- Downloads provider plugins
- Initializes backend
- Downloads modules
```bash
terraform init
```

**3. Plan (`terraform plan`)**
- Creates execution plan
- Shows what will change
- No actual changes made
```bash
terraform plan
```

**4. Apply (`terraform apply`)**
- Executes the plan
- Creates/modifies/deletes resources
- Updates state file
```bash
terraform apply
```

**5. Manage**
- View state: `terraform show`
- List resources: `terraform state list`
- Modify: Update code and repeat plan/apply
- Destroy: `terraform destroy`

**Additional Commands:**
- `terraform fmt`: Format code
- `terraform validate`: Check syntax
- `terraform refresh`: Sync state with reality
- `terraform output`: Display outputs

---

### **Q3: What is the difference between Terraform and ARM templates?**

**Answer:**

| Aspect | Terraform | ARM Templates |
|--------|-----------|---------------|
| **Language** | HCL (human-readable) | JSON (verbose) |
| **Cloud Support** | Multi-cloud | Azure only |
| **State Management** | Explicit state file | Deployment history |
| **Execution** | Plan → Apply | Direct deployment |
| **Modularity** | Modules (reusable) | Linked templates |
| **Community** | Large ecosystem | Azure-focused |
| **Learning Curve** | Easier HCL syntax | Complex JSON |
| **Rollback** | Via version control | Azure rollback feature |

**When to use Terraform:**
- Multi-cloud strategy
- Better readability needed
- Large community modules
- Consistent workflow across clouds

**When to use ARM Templates:**
- Azure-only environment
- Deep Azure integration
- Native Azure tooling

---

### **Q4: What is a Terraform provider?**

**Answer:**
A provider is a plugin that Terraform uses to interact with cloud platforms, SaaS providers, and other APIs.

**Azure Provider Example:**
```hcl
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
  
  subscription_id = var.subscription_id
  tenant_id       = var.tenant_id
}
```

**Provider Responsibilities:**
- Authentication with service
- Resource CRUD operations
- Data source queries
- Error handling
- API versioning

**Common Providers:**
- `azurerm`: Azure Resource Manager
- `azuread`: Azure Active Directory
- `aws`: Amazon Web Services
- `google`: Google Cloud Platform
- `kubernetes`: Kubernetes
- `helm`: Helm charts
- `random`: Random values
- `time`: Time-based resources

**Multiple Provider Configurations:**
```hcl
# Default provider
provider "azurerm" {
  features {}
}

# Secondary region
provider "azurerm" {
  alias           = "westus"
  subscription_id = var.west_subscription_id
  features {}
}

# Usage
resource "azurerm_resource_group" "west" {
  provider = azurerm.westus
  name     = "rg-west"
  location = "West US"
}
```

---

### **Q5: Explain Terraform state. Why is it important?**

**Answer:**

**What is State:**
Terraform state is a file (`terraform.tfstate`) that maps your configuration to real-world resources.

**Contents:**
```json
{
  "version": 4,
  "terraform_version": "1.0.0",
  "resources": [
    {
      "type": "azurerm_resource_group",
      "name": "example",
      "instances": [{
        "attributes": {
          "id": "/subscriptions/.../resourceGroups/rg-example",
          "name": "rg-example",
          "location": "eastus"
        }
      }]
    }
  ]
}
```

**Why Important:**

1. **Mapping**: Links configuration to actual resources
   ```
   Code: azurerm_resource_group.example
   ↓
   State: Resource ID in Azure
   ```

2. **Performance**: Caches resource attributes
   - Avoids querying Azure for every plan
   - Faster operations

3. **Collaboration**: Shared source of truth
   - Multiple team members
   - Consistent view of infrastructure

4. **Dependency Tracking**: Understands resource relationships
   ```hcl
   subnet → vnet dependency
   ```

5. **Change Detection**: Knows what changed
   ```
   Plan compares: Configuration vs State vs Azure
   ```

**State Locations:**

**Local State** (default):
```
project/
├── main.tf
└── terraform.tfstate  # Local file
```

**Remote State** (recommended):
```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "tfstate"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}
```

**State Locking:**
- Prevents concurrent modifications
- Automatic with Azure Storage backend
- Critical for team environments

---

### **Q6: What are Terraform variables? Show examples.**

**Answer:**

**Types of Variables:**

**1. Input Variables** (parameters)
```hcl
# variables.tf
variable "location" {
  description = "Azure region"
  type        = string
  default     = "East US"
}

variable "vm_count" {
  description = "Number of VMs"
  type        = number
  default     = 2
  
  validation {
    condition     = var.vm_count > 0 && var.vm_count <= 10
    error_message = "VM count must be 1-10."
  }
}

variable "tags" {
  description = "Resource tags"
  type        = map(string)
  default = {
    Environment = "Dev"
  }
}

variable "subnets" {
  description = "Subnet configuration"
  type = map(object({
    address_prefix = string
  }))
}

variable "admin_password" {
  description = "Admin password"
  type        = string
  sensitive   = true
}
```

**2. Local Variables** (computed values)
```hcl
# locals.tf
locals {
  environment     = terraform.workspace
  resource_prefix = "${var.project}-${local.environment}"
  
  common_tags = {
    Environment = local.environment
    ManagedBy   = "Terraform"
    Project     = var.project
  }
  
  # Conditional
  vm_size = local.environment == "prod" ? "Standard_D4s_v3" : "Standard_B2s"
}
```

**3. Output Variables** (return values)
```hcl
# outputs.tf
output "resource_group_id" {
  description = "Resource group ID"
  value       = azurerm_resource_group.example.id
}

output "vm_private_ips" {
  description = "VM private IPs"
  value       = azurerm_network_interface.vms[*].private_ip_address
}
```

**Variable Assignment:**

**1. Default values** (in variable definition)
**2. terraform.tfvars**
```hcl
location = "West Europe"
vm_count = 5
```

**3. Environment variables**
```bash
export TF_VAR_location="West US"
```

**4. Command line**
```bash
terraform apply -var="location=Central US"
terraform apply -var-file="prod.tfvars"
```

**5. Interactive prompt** (if no value provided)

**Precedence** (highest to lowest):
1. Command line `-var`
2. `*.auto.tfvars` files
3. `terraform.tfvars`
4. Environment variables
5. Default values

---

### **Q7: What is the difference between `count` and `for_each`?**

**Answer:**

**Count:**
- Creates multiple instances using index
- Index-based (0, 1, 2, ...)
- Reorders on deletion

```hcl
variable "vm_count" {
  default = 3
}

resource "azurerm_linux_virtual_machine" "vms" {
  count               = var.vm_count
  name                = "vm-${count.index + 1}"
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location
  size                = "Standard_B2s"
  # ...
}

# Creates: vm-1, vm-2, vm-3
# If you delete vm-2, vm-3 becomes vm-2 (BAD!)
```

**For_each:**
- Creates instances using map or set
- Key-based identification
- No reordering issues

```hcl
variable "vms" {
  type = map(object({
    size     = string
    location = string
  }))
  default = {
    web1 = { size = "Standard_B2s", location = "East US" }
    web2 = { size = "Standard_B2s", location = "East US" }
    db1  = { size = "Standard_D4s_v3", location = "West US" }
  }
}

resource "azurerm_linux_virtual_machine" "vms" {
  for_each = var.vms
  
  name                = each.key
  resource_group_name = azurerm_resource_group.example.name
  location            = each.value.location
  size                = each.value.size
  # ...
}

# Creates: web1, web2, db1
# If you delete web2, web1 and db1 are unchanged (GOOD!)
```

**When to Use:**

**Use `count` when:**
- Creating identical resources
- Simple numeric iteration
- Order doesn't matter

**Use `for_each` when:**
- Resources have different configurations
- You need stable identifiers
- You might add/remove in the middle
- **Generally preferred for production**

**Accessing Values:**

**Count:**
```hcl
azurerm_linux_virtual_machine.vms[0].id
azurerm_linux_virtual_machine.vms[1].id
```

**For_each:**
```hcl
azurerm_linux_virtual_machine.vms["web1"].id
azurerm_linux_virtual_machine.vms["db1"].id
```

---

### **Q8: How do you manage secrets in Terraform?**

**Answer:**

**Bad Practices (DON'T DO):**
```hcl
# ❌ Hardcoded secrets
variable "admin_password" {
  default = "P@ssw0rd123!"
}

# ❌ Committed to Git
# terraform.tfvars with secrets
```

**Good Practices:**

**1. Mark Variables as Sensitive**
```hcl
variable "admin_password" {
  type      = string
  sensitive = true
}

output "password" {
  value     = var.admin_password
  sensitive = true  # Hidden in output
}
```

**2. Environment Variables**
```bash
export TF_VAR_admin_password="SecurePassword123!"
terraform apply
```

**3. Azure Key Vault Data Source**
```hcl
data "azurerm_key_vault" "example" {
  name                = "kv-secrets"
  resource_group_name = "rg-secrets"
}

data "azurerm_key_vault_secret" "admin_password" {
  name         = "vm-admin-password"
  key_vault_id = data.azurerm_key_vault.example.id
}

resource "azurerm_windows_virtual_machine" "example" {
  # ...
  admin_password = data.azurerm_key_vault_secret.admin_password.value
}
```

**4. CI/CD Secret Management**
```yaml
# Azure DevOps Pipeline
variables:
  - group: terraform-secrets  # Variable group with secrets

steps:
  - task: TerraformTaskV4@4
    env:
      TF_VAR_admin_password: $(admin-password)  # From variable group
```

**5. Random Password Generation**
```hcl
resource "random_password" "admin" {
  length  = 16
  special = true
}

resource "azurerm_key_vault_secret" "admin_password" {
  name         = "vm-admin-password"
  value        = random_password.admin.result
  key_vault_id = azurerm_key_vault.example.id
}

resource "azurerm_windows_virtual_machine" "example" {
  # ...
  admin_password = random_password.admin.result
}
```

**6. Encrypt State File**
```hcl
# Azure Storage encrypts at rest by default
terraform {
  backend "azurerm" {
    # State file is encrypted
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "tfstate"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}
```

**Best Practices:**
- ✅ Never commit secrets to Git
- ✅ Use `.gitignore` for `*.tfvars` with secrets
- ✅ Mark sensitive variables
- ✅ Use Key Vault or secret managers
- ✅ Rotate secrets regularly
- ✅ Encrypt state files
- ✅ Use service principals with limited scope

---

### **Q9: What is a Terraform module?**

**Answer:**

A module is a container for multiple resources that are used together. It's a way to organize and reuse Terraform configurations.

**Benefits:**
1. **Reusability**: Write once, use many times
2. **Organization**: Logical grouping of resources
3. **Encapsulation**: Hide complexity
4. **Testing**: Test modules independently
5. **Versioning**: Version modules for stability

**Module Structure:**
```
modules/
└── azure-vm/
    ├── main.tf        # Resources
    ├── variables.tf   # Input variables
    ├── outputs.tf     # Output values
    └── README.md      # Documentation
```

**Module Example:**
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
}

resource "azurerm_linux_virtual_machine" "this" {
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
}
```

```hcl
# modules/azure-vm/variables.tf
variable "vm_name" {
  description = "VM name"
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

variable "admin_username" {
  description = "Admin username"
  type        = string
  default     = "azureuser"
}

variable "ssh_public_key" {
  description = "SSH public key"
  type        = string
}
```

```hcl
# modules/azure-vm/outputs.tf
output "vm_id" {
  description = "VM ID"
  value       = azurerm_linux_virtual_machine.this.id
}

output "private_ip" {
  description = "Private IP"
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
  ssh_public_key      = file("~/.ssh/id_rsa.pub")
}

output "web_vm_ip" {
  value = module.web_vm.private_ip
}
```

**Module Sources:**

1. **Local**: `source = "./modules/azure-vm"`
2. **Git**: `source = "git::https://github.com/user/repo.git?ref=v1.0.0"`
3. **Registry**: `source = "Azure/aks/azurerm"`
4. **HTTP**: `source = "https://example.com/module.zip"`

---

### **Q10: Explain `terraform plan` vs `terraform apply`.**

**Answer:**

**terraform plan:**
- **Read-only operation**
- Creates execution plan
- Shows what will change
- No modifications made
- Safe to run anytime

```bash
terraform plan

# Output:
Terraform will perform the following actions:

  # azurerm_resource_group.example will be created
  + resource "azurerm_resource_group" "example" {
      + id       = (known after apply)
      + location = "eastus"
      + name     = "rg-example"
      + tags     = {
          + "Environment" = "Dev"
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```

**Key Features:**
- `+` = Create
- `-` = Delete
- `~` = Modify in-place
- `-/+` = Destroy and recreate
- `(known after apply)` = Value determined after creation

**Save Plan:**
```bash
terraform plan -out=tfplan
```

**terraform apply:**
- **Makes actual changes**
- Creates/modifies/deletes resources
- Updates state file
- Requires confirmation (unless `-auto-approve`)

```bash
terraform apply

# Or apply saved plan (no confirmation needed)
terraform apply tfplan
```

**Workflow:**
```bash
# 1. Write configuration
vim main.tf

# 2. Plan (review changes)
terraform plan

# 3. Apply (if plan looks good)
terraform apply

# Type 'yes' to confirm

# 4. Or auto-approve (CI/CD)
terraform apply -auto-approve
```

**Key Differences:**

| Aspect | Plan | Apply |
|--------|------|-------|
| Changes | No | Yes |
| State | No update | Updates state |
| Azure | No API calls | Creates resources |
| Confirmation | Not needed | Required |
| Idempotent | Yes | Yes |
| Safety | Always safe | Destructive |

**Best Practices:**
- Always run `plan` before `apply`
- Review plan output carefully
- Save plans in CI/CD pipelines
- Use `-auto-approve` only in automation
- Check for unexpected changes

---

## 🟡 Intermediate Level Questions

### **Q11: How do you import existing Azure resources into Terraform?**

**Answer:**

**Scenario:** You have resources created manually in Azure and want Terraform to manage them.

**Process:**

**Step 1: Write Configuration**
```hcl
# main.tf
resource "azurerm_resource_group" "existing" {
  name     = "rg-existing"
  location = "East US"
  
  tags = {
    Environment = "Production"
  }
}
```

**Step 2: Import Resource**
```bash
# Syntax: terraform import <resource_type>.<resource_name> <azure_resource_id>

terraform import azurerm_resource_group.existing \
  /subscriptions/{subscription-id}/resourceGroups/rg-existing

# Success output:
# azurerm_resource_group.existing: Importing from ID "/subscriptions/..."
# azurerm_resource_group.existing: Import prepared!
# azurerm_resource_group.existing: Import complete!
```

**Step 3: Verify**
```bash
terraform state list
# Output: azurerm_resource_group.existing

terraform state show azurerm_resource_group.existing
```

**Step 4: Align Configuration**
```bash
terraform plan

# If there are differences, update your .tf files to match
```

**More Examples:**

**Storage Account:**
```bash
terraform import azurerm_storage_account.example \
  /subscriptions/{sub-id}/resourceGroups/{rg}/providers/Microsoft.Storage/storageAccounts/{name}
```

**Virtual Network:**
```bash
terraform import azurerm_virtual_network.example \
  /subscriptions/{sub-id}/resourceGroups/{rg}/providers/Microsoft.Network/virtualNetworks/{vnet-name}
```

**Virtual Machine:**
```bash
terraform import azurerm_linux_virtual_machine.example \
  /subscriptions/{sub-id}/resourceGroups/{rg}/providers/Microsoft.Compute/virtualMachines/{vm-name}
```

**Bulk Import Script:**
```bash
#!/bin/bash
# import_resources.sh

# Resource Groups
for rg in $(az group list --query "[].name" -o tsv); do
  terraform import "azurerm_resource_group.${rg}" \
    "/subscriptions/${SUB_ID}/resourceGroups/${rg}"
done

# Storage Accounts
for st in $(az storage account list --query "[].name" -o tsv); do
  rg=$(az storage account show -n $st --query resourceGroup -o tsv)
  terraform import "azurerm_storage_account.${st}" \
    "/subscriptions/${SUB_ID}/resourceGroups/${rg}/providers/Microsoft.Storage/storageAccounts/${st}"
done
```

**Challenges:**
- Must write configuration beforehand
- Complex resources (VMs) require multiple imports
- Some attributes may not be importable
- Time-consuming for many resources

**Tools to Help:**
- `aztfexport`: Microsoft tool to export Azure resources
- `terraformer`: Third-party import tool
- `terraform-import-github-organization`: For bulk imports

---

### **Q12: What is the difference between remote state and local state?**

**Answer:**

**Local State:**

**Storage:**
```
project/
├── main.tf
├── variables.tf
└── terraform.tfstate  # Local file
```

**Pros:**
- Simple setup
- No additional infrastructure
- Fast operations

**Cons:**
- ❌ Not shareable (team issues)
- ❌ No locking (concurrent modifications)
- ❌ No versioning
- ❌ Lost if computer fails
- ❌ Secrets stored in plain text

**Remote State:**

**Configuration:**
```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "tfstatestorage123"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}
```

**Storage:**
```
Azure Storage Account
└── Container: tfstate
    └── Blob: prod.terraform.tfstate
```

**Pros:**
- ✅ Shared across team
- ✅ State locking (prevents conflicts)
- ✅ Versioning (Azure blob versions)
- ✅ Encryption at rest
- ✅ Backup and recovery
- ✅ Access control (Azure RBAC)

**Cons:**
- Requires additional setup
- Network dependency
- Potential costs

**Setup Remote State:**

```bash
# 1. Create resources
az group create --name rg-terraform-state --location eastus

az storage account create \
  --name tfstatestorage123 \
  --resource-group rg-terraform-state \
  --location eastus \
  --sku Standard_LRS \
  --encryption-services blob

az storage container create \
  --name tfstate \
  --account-name tfstatestorage123

# 2. Enable versioning
az storage account blob-service-properties update \
  --account-name tfstatestorage123 \
  --enable-versioning true

# 3. Configure Terraform
# Add backend.tf with configuration above

# 4. Initialize
terraform init

# 5. Migrate from local (if exists)
terraform init -migrate-state
```

**State Locking:**
```
User A runs: terraform apply
├─ Acquires lock on state file
├─ Makes changes
└─ Releases lock

User B runs: terraform apply (simultaneous)
└─ Blocked by lock
    └─ Error: state is locked by User A
```

**Best Practices:**
- ✅ **Always use remote state for production**
- ✅ Enable versioning
- ✅ Encrypt state
- ✅ Use separate backends per environment
- ✅ Restrict access with RBAC
- ✅ Backup state regularly

---

### **Q13: How do you handle multiple environments (dev, staging, prod)?**

**Answer:**

**Approach 1: Workspaces**

```bash
# Create workspaces
terraform workspace new dev
terraform workspace new staging
terraform workspace new prod

# Switch workspace
terraform workspace select dev

# List workspaces
terraform workspace list
```

```hcl
# Use workspace in configuration
locals {
  environment = terraform.workspace
  
  config = {
    dev = {
      vm_size  = "Standard_B2s"
      vm_count = 1
      location = "East US"
    }
    staging = {
      vm_size  = "Standard_D2s_v3"
      vm_count = 2
      location = "East US"
    }
    prod = {
      vm_size  = "Standard_D4s_v3"
      vm_count = 3
      location = "West US"
    }
  }
}

resource "azurerm_resource_group" "example" {
  name     = "rg-${local.environment}"
  location = local.config[local.environment].location
}

resource "azurerm_linux_virtual_machine" "vms" {
  count = local.config[local.environment].vm_count
  name  = "vm-${local.environment}-${count.index + 1}"
  size  = local.config[local.environment].vm_size
  # ...
}
```

**Pros:**
- Single codebase
- Easy switching
- Shared modules

**Cons:**
- Same backend (less isolation)
- Can accidentally affect wrong environment
- Limited flexibility

**Approach 2: Separate Directories**

```
terraform/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── terraform.tfvars
│   │   └── backend.tf
│   ├── staging/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── terraform.tfvars
│   │   └── backend.tf
│   └── prod/
│       ├── main.tf
│       ├── variables.tf
│       ├── terraform.tfvars
│       └── backend.tf
└── modules/
    ├── network/
    └── compute/
```

```hcl
# environments/dev/backend.tf
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "tfstate"
    container_name       = "tfstate"
    key                  = "dev.terraform.tfstate"
  }
}

# environments/dev/terraform.tfvars
environment = "dev"
location    = "East US"
vm_size     = "Standard_B2s"
vm_count    = 1
```

```hcl
# environments/dev/main.tf
module "infrastructure" {
  source = "../../modules/infrastructure"
  
  environment = var.environment
  location    = var.location
  vm_size     = var.vm_size
  vm_count    = var.vm_count
}
```

**Pros:**
- Complete isolation
- Different backends
- Environment-specific configurations
- Less risk of accidents

**Cons:**
- Code duplication
- More directories to manage

**Approach 3: Terragrunt**

```
terraform/
├── terragrunt.hcl  # Root config
├── dev/
│   └── terragrunt.hcl
├── staging/
│   └── terragrunt.hcl
├── prod/
│   └── terragrunt.hcl
└── modules/
    └── infrastructure/
```

```hcl
# dev/terragrunt.hcl
include "root" {
  path = find_in_parent_folders()
}

inputs = {
  environment = "dev"
  vm_size     = "Standard_B2s"
  vm_count    = 1
}
```

**Pros:**
- DRY (Don't Repeat Yourself)
- Advanced features
- Dependency management

**Cons:**
- Additional tool
- Learning curve

**Best Practices:**
- ✅ Use separate backends per environment
- ✅ Apply principle of least privilege
- ✅ Use approval gates for prod
- ✅ Test in dev before prod
- ✅ Document environment differences

---

### **Q14: How do you manage Terraform state securely?**

**Answer:**

**1. Use Remote Backend with Encryption**
```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "tfstate"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
    # Azure Storage encrypts at rest by default
  }
}
```

**2. Enable State Locking**
```
Automatic with Azure Storage backend
```

**3. Enable Versioning**
```bash
az storage account blob-service-properties update \
  --account-name tfstate \
  --enable-versioning true

# View versions
az storage blob list \
  --container-name tfstate \
  --account-name tfstate \
  --include v \
  --output table
```

**4. Restrict Access with RBAC**
```bash
# Only specific service principals/users can access
az role assignment create \
  --assignee $PRINCIPAL_ID \
  --role "Storage Blob Data Contributor" \
  --scope "/subscriptions/${SUB_ID}/resourceGroups/rg-terraform-state/providers/Microsoft.Storage/storageAccounts/tfstate"
```

**5. Use Private Endpoints**
```hcl
resource "azurerm_private_endpoint" "tfstate" {
  name                = "pe-tfstate"
  location            = azurerm_resource_group.state.location
  resource_group_name = azurerm_resource_group.state.name
  subnet_id           = azurerm_subnet.private.id
  
  private_service_connection {
    name                           = "psc-tfstate"
    private_connection_resource_id = azurerm_storage_account.tfstate.id
    subresource_names              = ["blob"]
    is_manual_connection           = false
  }
}
```

**6. Enable Soft Delete**
```bash
az storage blob service-properties delete-policy update \
  --account-name tfstate \
  --enable true \
  --days-retained 30
```

**7. Regular Backups**
```bash
#!/bin/bash
# backup_state.sh

DATE=$(date +%Y%m%d_%H%M%S)

az storage blob download \
  --container-name tfstate \
  --name prod.terraform.tfstate \
  --account-name tfstate \
  --file "backups/prod_${DATE}.tfstate"
```

**8. Audit Logging**
```bash
# Enable diagnostic settings
az monitor diagnostic-settings create \
  --name tfstate-logs \
  --resource "/subscriptions/${SUB_ID}/resourceGroups/rg-terraform-state/providers/Microsoft.Storage/storageAccounts/tfstate" \
  --workspace $LOG_ANALYTICS_ID \
  --logs '[{"category": "StorageRead","enabled": true},{"category": "StorageWrite","enabled": true}]'
```

**9. Separate State Per Environment**
```
tfstate/
├── dev.terraform.tfstate
├── staging.terraform.tfstate
└── prod.terraform.tfstate

# Or separate storage accounts
```

**10. Sensitive Data Handling**
```hcl
# Mark outputs as sensitive
output "database_password" {
  value     = azurerm_sql_server.example.administrator_login_password
  sensitive = true  # Won't show in logs
}

# State still contains it, hence encryption is critical
```

**Security Checklist:**
- [x] Remote backend with encryption
- [x] State locking enabled
- [x] Versioning enabled
- [x] RBAC configured
- [x] Network isolation (private endpoints)
- [x] Soft delete enabled
- [x] Regular backups
- [x] Audit logging
- [x] Separate state per environment
- [x] Sensitive outputs marked

---

### **Q15: Explain dynamic blocks in Terraform.**

**Answer:**

Dynamic blocks generate nested blocks dynamically based on a collection.

**Without Dynamic Block (Repetitive):**
```hcl
resource "azurerm_network_security_group" "example" {
  name                = "nsg-example"
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
  
  security_rule {
    name                       = "AllowSSH"
    priority                   = 120
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "22"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }
  
  # And many more...
}
```

**With Dynamic Block (Clean):**
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
  default = [
    {
      name                       = "AllowHTTPS"
      priority                   = 100
      direction                  = "Inbound"
      access                     = "Allow"
      protocol                   = "Tcp"
      source_port_range          = "*"
      destination_port_range     = "443"
      source_address_prefix      = "*"
      destination_address_prefix = "*"
    },
    {
      name                       = "AllowHTTP"
      priority                   = 110
      direction                  = "Inbound"
      access                     = "Allow"
      protocol                   = "Tcp"
      source_port_range          = "*"
      destination_port_range     = "80"
      source_address_prefix      = "*"
      destination_address_prefix = "*"
    },
    {
      name                       = "AllowSSH"
      priority                   = 120
      direction                  = "Inbound"
      access                     = "Allow"
      protocol                   = "Tcp"
      source_port_range          = "*"
      destination_port_range     = "22"
      source_address_prefix      = "*"
      destination_address_prefix = "*"
    }
  ]
}

resource "azurerm_network_security_group" "example" {
  name                = "nsg-example"
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

**Key Components:**
- `dynamic "<block_type>"`: The block to generate
- `for_each`: Collection to iterate over
- `content`: The block content
- `<block_type>.value`: Current item
- `<block_type>.key`: Current key (if map)

**More Examples:**

**AKS Node Pools:**
```hcl
variable "node_pools" {
  type = map(object({
    vm_size    = string
    node_count = number
  }))
}

resource "azurerm_kubernetes_cluster" "example" {
  # ... other config ...
  
  dynamic "agent_pool_profile" {
    for_each = var.node_pools
    
    content {
      name       = agent_pool_profile.key
      vm_size    = agent_pool_profile.value.vm_size
      node_count = agent_pool_profile.value.node_count
    }
  }
}
```

**Nested Dynamic Blocks:**
```hcl
dynamic "ip_configuration" {
  for_each = var.ip_configurations
  
  content {
    name                          = ip_configuration.value.name
    subnet_id                     = ip_configuration.value.subnet_id
    private_ip_address_allocation = ip_configuration.value.allocation
    
    dynamic "public_ip_address" {
      for_each = ip_configuration.value.create_public_ip ? [1] : []
      
      content {
        public_ip_address_id = azurerm_public_ip.example[ip_configuration.key].id
      }
    }
  }
}
```

**When to Use:**
- Repeated nested blocks
- Configuration-driven infrastructure
- Variable number of items
- DRY principle

**When NOT to Use:**
- Overcomplicates simple scenarios
- Makes code less readable
- Use regular blocks if count < 3

---

## 🔴 Advanced Level Questions

### **Q16: Design a multi-region, highly available Azure infrastructure with Terraform.**

**Answer:**

**Architecture:**
```
┌──────────────────────────────────────────┐
│         Azure Traffic Manager            │
│         (Global Load Balancer)           │
└──────────┬─────────────────┬─────────────┘
           │                 │
    ┌──────▼──────┐   ┌─────▼──────┐
    │  Region 1   │   │  Region 2  │
    │  (Primary)  │   │ (Secondary)│
    └─────────────┘   └────────────┘
```

**Implementation:**

```hcl
# main.tf
locals {
  regions = {
    primary = {
      location = "East US"
      name     = "primary"
    }
    secondary = {
      location = "West US"
      name     = "secondary"
    }
  }
}

# Resource Groups per region
resource "azurerm_resource_group" "regions" {
  for_each = local.regions
  
  name     = "rg-${each.value.name}"
  location = each.value.location
  
  tags = {
    Region = each.value.name
  }
}

# Hub-Spoke Network per region
module "network" {
  for_each = local.regions
  source   = "./modules/network"
  
  environment         = var.environment
  region_name         = each.value.name
  location            = each.value.location
  resource_group_name = azurerm_resource_group.regions[each.key].name
  
  hub_address_space   = cidrsubnet("10.0.0.0/8", 8, index(keys(local.regions), each.key))
  spoke_address_space = cidrsubnet("10.0.0.0/8", 8, index(keys(local.regions), each.key) + 100)
}

# VNet Peering between regions
resource "azurerm_virtual_network_peering" "primary_to_secondary" {
  name                      = "primary-to-secondary"
  resource_group_name       = azurerm_resource_group.regions["primary"].name
  virtual_network_name      = module.network["primary"].hub_vnet_name
  remote_virtual_network_id = module.network["secondary"].hub_vnet_id
  
  allow_forwarded_traffic = true
}

resource "azurerm_virtual_network_peering" "secondary_to_primary" {
  name                      = "secondary-to-primary"
  resource_group_name       = azurerm_resource_group.regions["secondary"].name
  virtual_network_name      = module.network["secondary"].hub_vnet_name
  remote_virtual_network_id = module.network["primary"].hub_vnet_id
  
  allow_forwarded_traffic = true
}

# AKS per region
module "aks" {
  for_each = local.regions
  source   = "./modules/aks"
  
  cluster_name        = "aks-${each.value.name}"
  location            = each.value.location
  resource_group_name = azurerm_resource_group.regions[each.key].name
  subnet_id           = module.network[each.key].aks_subnet_id
  kubernetes_version  = "1.28.0"
  
  node_pools = {
    system = {
      vm_size    = "Standard_D2s_v3"
      node_count = 3
      zones      = ["1", "2", "3"]
    }
    user = {
      vm_size    = "Standard_D4s_v3"
      min_count  = 3
      max_count  = 10
      zones      = ["1", "2", "3"]
    }
  }
}

# Azure SQL with Geo-Replication
module "sql" {
  source = "./modules/sql"
  
  primary_location   = local.regions.primary.location
  secondary_location = local.regions.secondary.location
  primary_rg_name    = azurerm_resource_group.regions["primary"].name
  secondary_rg_name  = azurerm_resource_group.regions["secondary"].name
  
  enable_failover_group = true
}

# Traffic Manager
resource "azurerm_traffic_manager_profile" "global" {
  name                   = "tm-global"
  resource_group_name    = azurerm_resource_group.regions["primary"].name
  traffic_routing_method = "Priority"  # Or "Performance", "Geographic"
  
  dns_config {
    relative_name = "myapp-global"
    ttl           = 30
  }
  
  monitor_config {
    protocol                     = "HTTPS"
    port                         = 443
    path                         = "/health"
    interval_in_seconds          = 30
    timeout_in_seconds           = 10
    tolerated_number_of_failures = 3
  }
}

# Traffic Manager Endpoints
resource "azurerm_traffic_manager_azure_endpoint" "regions" {
  for_each = local.regions
  
  name               = "endpoint-${each.value.name}"
  profile_id         = azurerm_traffic_manager_profile.global.id
  priority           = index(keys(local.regions), each.key) + 1
  target_resource_id = module.app_gateway[each.key].public_ip_id
}

# Azure Front Door (Alternative)
resource "azurerm_cdn_frontdoor_profile" "global" {
  name                = "fd-global"
  resource_group_name = azurerm_resource_group.regions["primary"].name
  sku_name            = "Premium_AzureFrontDoor"
}

# Disaster Recovery
resource "azurerm_recovery_services_vault" "regions" {
  for_each = local.regions
  
  name                = "rsv-${each.value.name}"
  location            = each.value.location
  resource_group_name = azurerm_resource_group.regions[each.key].name
  sku                 = "Standard"
}
```

**Key HA Features:**
1. Multi-region deployment
2. Availability zones (3 zones per region)
3. Load balancing (Traffic Manager / Front Door)
4. Auto-scaling (AKS)
5. Database geo-replication
6. Backup and recovery
7. Health monitoring
8. Automatic failover

---

### **Q17: How do you implement CI/CD for Terraform in Azure DevOps?**

**Answer:** (See complete pipeline in [terraform-learning-roadmap.md](terraform-learning-roadmap.md) Week 9)

**Key Components:**

1. **Stages**: Validate → Plan → Apply
2. **Approval Gates**: Manual approval before production
3. **Service Connection**: Azure authentication
4. **Variable Groups**: Sensitive values
5. **Artifacts**: Store plan files
6. **Testing**: Automated validation

---

### **Q18: How would you migrate from monolithic Terraform to modules?**

**Answer:**

**Current State (Monolithic):**
```
terraform/
├── main.tf                # 5000 lines
├── variables.tf           # 500 lines
├── outputs.tf             # 200 lines
└── terraform.tfstate
```

**Target State (Modular):**
```
terraform/
├── main.tf                # 100 lines (calls modules)
├── variables.tf
├── outputs.tf
└── modules/
    ├── network/
    ├── compute/
    ├── database/
    └── monitoring/
```

**Migration Strategy:**

**Step 1: Backup**
```bash
terraform state pull > state-backup-$(date +%Y%m%d).json
cp -r . ../terraform-backup
```

**Step 2: Identify Module Boundaries**
```
Network resources   → network module
VMs and scale sets → compute module
Databases          → database module
Monitoring         → monitoring module
```

**Step 3: Create Module Structure**
```bash
mkdir -p modules/{network,compute,database,monitoring}
```

**Step 4: Move Resources to Modules (One at a Time)**

```hcl
# Step 4a: Extract network resources to module
# modules/network/main.tf
resource "azurerm_virtual_network" "this" {
  name                = var.vnet_name
  address_space       = var.address_space
  location            = var.location
  resource_group_name = var.resource_group_name
}
# ... other network resources
```

```hcl
# Step 4b: Update main.tf to use module
module "network" {
  source = "./modules/network"
  
  vnet_name           = "vnet-prod"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
}
```

**Step 5: Move State**
```bash
# Move resources into module
terraform state mv \
  azurerm_virtual_network.main \
  module.network.azurerm_virtual_network.this

terraform state mv \
  azurerm_subnet.web \
  module.network.azurerm_subnet.subnets["web"]

# Verify
terraform plan  # Should show no changes
```

**Step 6: Test**
```bash
terraform plan  # No changes expected
terraform apply -auto-approve
```

**Step 7: Repeat for Other Modules**

**Step 8: Clean Up**
```bash
# Remove old monolithic files
# Keep only module calls in main.tf
```

**Challenges:**
- State movement is error-prone
- Can't easily rollback mid-migration
- Downtime risk if mistakes made
- Team coordination needed

**Best Practices:**
- Migrate one module at a time
- Test thoroughly after each migration
- Use feature branches
- Communicate with team
- Consider blue-green approach

---

## ⚫ Expert Level Questions

### **Q19: You have 10,000 Azure resources managed by Terraform. Performance is very slow. How do you optimize?**

**Answer:**

**Problem Diagnosis:**
```bash
# Enable timing
TF_LOG=INFO terraform plan 2>&1 | grep "time="

# Identify slow resources
terraform plan | grep "Refreshing state"
```

**Optimization Strategies:**

**1. State Management**
```bash
# Disable refresh if state is current
terraform plan -refresh=false
terraform apply -refresh=false

# Targeted operations
terraform apply -target=module.network
```

**2. Parallelism**
```bash
# Increase parallel operations (default 10)
terraform apply -parallelism=20

# Be cautious - Azure has rate limits
```

**3. Split State Files**
```
terraform/
├── networking/
│   └── terraform.tfstate (2000 resources)
├── compute/
│   └── terraform.tfstate (3000 resources)
├── databases/
│   └── terraform.tfstate (2000 resources)
└── monitoring/
    └── terraform.tfstate (3000 resources)
```

**4. Use Data Sources Instead of Resources**
```hcl
# Bad - Managed in state
resource "azurerm_resource_group" "shared" {
  name     = "rg-shared"
  location = "East US"
}

# Good - Not in state
data "azurerm_resource_group" "shared" {
  name = "rg-shared"
}
```

**5. Lazy Loading with Modules**
```hcl
# Only instantiate needed modules
module "network" {
  count  = var.deploy_network ? 1 : 0
  source = "./modules/network"
  # ...
}
```

**6. Terraform Cloud/Enterprise**
- Remote execution
- Parallel runs
- Better caching
- Sentinel policies

**7. Provider Optimization**
```hcl
provider "azurerm" {
  features {}
  
  # Skip provider registration
  skip_provider_registration = true
  
  # Disable correlation ID
  disable_correlation_request_id = true
}
```

**8. Use `depends_on` Sparingly**
```hcl
# Implicit dependencies are faster
resource "azurerm_subnet" "example" {
  virtual_network_name = azurerm_virtual_network.example.name  # Implicit
  # ...
}

# Avoid unless necessary
# depends_on = [azurerm_virtual_network.example]
```

**9. Optimize Queries**
```hcl
# Bad - Multiple data sources
data "azurerm_resource_group" "rg1" { name = "rg-1" }
data "azurerm_resource_group" "rg2" { name = "rg-2" }
data "azurerm_resource_group" "rg3" { name = "rg-3" }

# Good - Single query with for_each
data "azurerm_resource_group" "rgs" {
  for_each = toset(["rg-1", "rg-2", "rg-3"])
  name     = each.key
}
```

**10. Consider Alternative Tools**
- **Terragrunt**: Better dependency management
- **Pulumi**: Programming languages (faster for complex logic)
- **CDK for Terraform**: TypeScript/Python

**Benchmark:**
```bash
time terraform plan
# Before: 15 minutes
# After optimization: 2 minutes
```

---

### **Q20: Design a Terraform architecture for a large enterprise with 100+ Azure subscriptions and 50+ teams.**

**Answer:**

**Architecture:**
```
Enterprise Terraform Architecture
│
├─ 📁 terraform-platform/
│   ├─ Landing Zones (Subscription Factory)
│   ├─ Core Infrastructure (Hub Networks)
│   ├─ Shared Services (DNS, Monitoring)
│   └─ Governance (Policies, RBAC)
│
├─ 📁 terraform-modules/ (Private Registry)
│   ├─ azure-network/
│   ├─ azure-aks/
│   ├─ azure-vm/
│   ├─ azure-sql/
│   └─ ... (50+ modules)
│
└─ 📁 team-repositories/ (50+ teams)
    ├─ team-web/
    ├─ team-api/
    ├─ team-data/
    └─ ...
```

**Implementation:**

**1. Module Registry**
```hcl
# Private module registry (Azure DevOps Artifacts or Terraform Cloud)
module "network" {
  source  = "company.registry.io/azure/network"
  version = "2.1.0"
  
  # standardized inputs
}
```

**2. Landing Zone Pattern**
```hcl
# Landing zone creates subscription + base infrastructure
module "landing_zone" {
  source = "./modules/landing-zone"
  
  subscription_name = "sub-team-web-prod"
  team_name         = "web"
  environment       = "prod"
  cost_center       = "12345"
  
  # Automatically creates:
  # - Subscription
  # - Resource groups
  # - Hub-spoke network
  # - Monitoring
  # - Security baseline
}
```

**3. GitOps Workflow**
```
Team Developer
    ↓
  PR to main
    ↓
  Review & Approve
    ↓
  Azure Pipeline
    ↓
  terraform plan (comment on PR)
    ↓
  Merge to main
    ↓
  terraform apply
    ↓
  Azure Resources
```

**4. State Management**
```
State per team per environment:
- team-web-dev.tfstate
- team-web-prod.tfstate
- team-api-dev.tfstate
- team-api-prod.tfstate

Separate storage accounts per team
```

**5. RBAC Model**
```
Platform Team:
- Can modify platform infrastructure
- Can approve team requests
- Full access to all subscriptions

Team Developers:
- Can only modify their team's infrastructure
- Can't modify platform resources
- Subscription-level access

Security Team:
- Read-only access
- Can enforce policies
- Audit capabilities
```

**6. Policy as Code**
```hcl
# Enforce standards via Azure Policy
resource "azurerm_policy_assignment" "naming" {
  name                 = "enforce-naming-convention"
  scope                = data.azurerm_management_group.root.id
  policy_definition_id = azurerm_policy_definition.naming.id
  
  parameters = jsonencode({
    prefix = {
      value = var.company_prefix
    }
  })
}
```

**7. Cost Management**
```hcl
# Automatic tagging
locals {
  required_tags = {
    Team        = var.team_name
    Environment = var.environment
    CostCenter  = var.cost_center
    ManagedBy   = "Terraform"
  }
}

# Budget alerts
resource "azurerm_consumption_budget_subscription" "team" {
  name            = "budget-${var.team_name}-${var.environment}"
  subscription_id = data.azurerm_subscription.current.id
  
  amount     = var.monthly_budget
  time_grain = "Monthly"
  
  notification {
    enabled   = true
    threshold = 80
    operator  = "GreaterThan"
    
    contact_emails = var.team_emails
  }
}
```

**8. Self-Service Portal**
```
Web Portal (Azure Static Web App)
    ↓
Request New Subscription
    ↓
Approval Workflow (Azure Logic Apps)
    ↓
Trigger Terraform (Azure Pipeline)
    ↓
Provision Landing Zone
    ↓
Notify Team (Email)
```

**9. Disaster Recovery**
```
- State versioning enabled
- Daily backups of state files
- Geo-replicated storage
- Documented recovery procedures
- Tested quarterly
```

**10. Monitoring & Observability**
```
- Terraform Cloud Sentinel policies
- Azure Policy compliance
- Cost tracking per team
- Resource inventory
- Change tracking
- Audit logs
```

**Key Success Factors:**
1. Standardization (modules, naming, tagging)
2. Automation (CI/CD pipelines)
3. Self-service (reduce platform team bottleneck)
4. Governance (policies, budgets)
5. Security (RBAC, encryption, audit)
6. Documentation (runbooks, architecture diagrams)
7. Training (team enablement)
8. Continuous improvement

---

**Congratulations! You've mastered 80+ Terraform interview questions! 🎉🚀☁️**

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

