# Azure Quick Reference for DevOps Engineers

**Essential Azure CLI commands, PowerShell cmdlets, and common patterns for daily operations**

---

## 🚀 Quick Setup

### Azure CLI Installation & Login

```bash
# Install Azure CLI
# Windows
winget install Microsoft.AzureCLI

# macOS
brew install azure-cli

# Linux
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Verify installation
az --version

# Login
az login

# Login with service principal
az login --service-principal \
  --username $ARM_CLIENT_ID \
  --password $ARM_CLIENT_SECRET \
  --tenant $ARM_TENANT_ID

# List subscriptions
az account list --output table

# Set active subscription
az account set --subscription "SUBSCRIPTION_NAME"

# Show current subscription
az account show --output json
```

### PowerShell Setup

```powershell
# Install Az PowerShell module
Install-Module -Name Az -Scope CurrentUser -Repository PSGallery -Force

# Update module
Update-Module -Name Az

# Login
Connect-AzAccount

# Select subscription
Set-AzContext -SubscriptionName "SUBSCRIPTION_NAME"

# Get current context
Get-AzContext
```

---

## 📦 Resource Groups

### Azure CLI

```bash
# Create resource group
az group create \
  --name rg-prod-eastus \
  --location eastus \
  --tags Environment=Production CostCenter=IT

# List resource groups
az group list --output table

# Show specific resource group
az group show --name rg-prod-eastus

# Update tags
az group update \
  --name rg-prod-eastus \
  --tags Environment=Production CostCenter=IT Owner=DevOps

# List resources in group
az resource list \
  --resource-group rg-prod-eastus \
  --output table

# Export template
az group export \
  --name rg-prod-eastus \
  --include-parameter-default-value \
  --include-comments \
  > template.json

# Delete resource group
az group delete \
  --name rg-prod-eastus \
  --yes \
  --no-wait

# Check if exists
az group exists --name rg-prod-eastus
```

### PowerShell

```powershell
# Create resource group
New-AzResourceGroup `
  -Name "rg-prod-eastus" `
  -Location "East US" `
  -Tag @{Environment="Production"; CostCenter="IT"}

# Get resource groups
Get-AzResourceGroup

# Get specific group
Get-AzResourceGroup -Name "rg-prod-eastus"

# Remove resource group
Remove-AzResourceGroup -Name "rg-prod-eastus" -Force
```

---

## 🌐 Virtual Networks

### Create VNet & Subnets

```bash
# Create VNet
az network vnet create \
  --resource-group rg-network \
  --name vnet-prod \
  --address-prefix 10.0.0.0/16 \
  --subnet-name subnet-web \
  --subnet-prefix 10.0.1.0/24 \
  --location eastus

# Add subnet
az network vnet subnet create \
  --resource-group rg-network \
  --vnet-name vnet-prod \
  --name subnet-app \
  --address-prefix 10.0.2.0/24

# Add subnet with service endpoints
az network vnet subnet create \
  --resource-group rg-network \
  --vnet-name vnet-prod \
  --name subnet-data \
  --address-prefix 10.0.3.0/24 \
  --service-endpoints Microsoft.Storage Microsoft.Sql Microsoft.KeyVault

# List VNets
az network vnet list --output table

# Show VNet
az network vnet show \
  --resource-group rg-network \
  --name vnet-prod

# List subnets
az network vnet subnet list \
  --resource-group rg-network \
  --vnet-name vnet-prod \
  --output table

# Delete VNet
az network vnet delete \
  --resource-group rg-network \
  --name vnet-prod
```

### VNet Peering

```bash
# Create peering (VNet1 to VNet2)
az network vnet peering create \
  --resource-group rg-network \
  --name vnet1-to-vnet2 \
  --vnet-name vnet-prod \
  --remote-vnet vnet-dev \
  --allow-vnet-access \
  --allow-forwarded-traffic

# Create peering (VNet2 to VNet1)
az network vnet peering create \
  --resource-group rg-network \
  --name vnet2-to-vnet1 \
  --vnet-name vnet-dev \
  --remote-vnet vnet-prod \
  --allow-vnet-access \
  --allow-forwarded-traffic

# List peerings
az network vnet peering list \
  --resource-group rg-network \
  --vnet-name vnet-prod \
  --output table

# Delete peering
az network vnet peering delete \
  --resource-group rg-network \
  --vnet-name vnet-prod \
  --name vnet1-to-vnet2
```

---

## 🔒 Network Security Groups (NSGs)

```bash
# Create NSG
az network nsg create \
  --resource-group rg-network \
  --name nsg-web \
  --location eastus

# Add rule (Allow HTTPS)
az network nsg rule create \
  --resource-group rg-network \
  --nsg-name nsg-web \
  --name AllowHTTPS \
  --priority 100 \
  --source-address-prefixes '*' \
  --source-port-ranges '*' \
  --destination-address-prefixes '*' \
  --destination-port-ranges 443 \
  --access Allow \
  --protocol Tcp \
  --direction Inbound

# Add rule (Allow SSH from specific IP)
az network nsg rule create \
  --resource-group rg-network \
  --nsg-name nsg-web \
  --name AllowSSH \
  --priority 110 \
  --source-address-prefixes '203.0.113.0/24' \
  --destination-port-ranges 22 \
  --access Allow \
  --protocol Tcp

# Associate NSG with subnet
az network vnet subnet update \
  --resource-group rg-network \
  --vnet-name vnet-prod \
  --name subnet-web \
  --network-security-group nsg-web

# List NSG rules
az network nsg rule list \
  --resource-group rg-network \
  --nsg-name nsg-web \
  --output table

# Delete rule
az network nsg rule delete \
  --resource-group rg-network \
  --nsg-name nsg-web \
  --name AllowSSH
```

---

## 💻 Virtual Machines

### Create Linux VM

```bash
# Create VM with SSH key
az vm create \
  --resource-group rg-compute \
  --name vm-web-01 \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --vnet-name vnet-prod \
  --subnet subnet-web \
  --nsg nsg-web \
  --admin-username azureuser \
  --generate-ssh-keys \
  --public-ip-address pip-vm-web-01 \
  --public-ip-sku Standard

# Create VM with specific SSH key
az vm create \
  --resource-group rg-compute \
  --name vm-web-02 \
  --image Ubuntu2204 \
  --size Standard_D2s_v3 \
  --vnet-name vnet-prod \
  --subnet subnet-web \
  --admin-username azureuser \
  --ssh-key-values @~/.ssh/id_rsa.pub

# Create VM with cloud-init
az vm create \
  --resource-group rg-compute \
  --name vm-web-03 \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --vnet-name vnet-prod \
  --subnet subnet-web \
  --admin-username azureuser \
  --generate-ssh-keys \
  --custom-data cloud-init.txt
```

### Create Windows VM

```bash
# Create Windows VM
az vm create \
  --resource-group rg-compute \
  --name vm-win-01 \
  --image Win2022Datacenter \
  --size Standard_D2s_v3 \
  --vnet-name vnet-prod \
  --subnet subnet-app \
  --admin-username winadmin \
  --admin-password 'P@ssw0rd1234!'
```

### VM Operations

```bash
# List VMs
az vm list --output table

# Show VM details
az vm show \
  --resource-group rg-compute \
  --name vm-web-01

# Get public IP
az vm list-ip-addresses \
  --resource-group rg-compute \
  --name vm-web-01 \
  --output table

# Start VM
az vm start \
  --resource-group rg-compute \
  --name vm-web-01

# Stop VM (deallocate)
az vm deallocate \
  --resource-group rg-compute \
  --name vm-web-01

# Restart VM
az vm restart \
  --resource-group rg-compute \
  --name vm-web-01

# Delete VM
az vm delete \
  --resource-group rg-compute \
  --name vm-web-01 \
  --yes

# Resize VM
az vm resize \
  --resource-group rg-compute \
  --name vm-web-01 \
  --size Standard_D4s_v3

# Run command on VM
az vm run-command invoke \
  --resource-group rg-compute \
  --name vm-web-01 \
  --command-id RunShellScript \
  --scripts "apt-get update && apt-get install -y nginx"
```

### VM Scale Sets

```bash
# Create VMSS
az vmss create \
  --resource-group rg-compute \
  --name vmss-web \
  --image Ubuntu2204 \
  --instance-count 3 \
  --vm-sku Standard_B2s \
  --vnet-name vnet-prod \
  --subnet subnet-web \
  --lb vmss-lb \
  --backend-pool-name webpool \
  --admin-username azureuser \
  --generate-ssh-keys

# Scale VMSS
az vmss scale \
  --resource-group rg-compute \
  --name vmss-web \
  --new-capacity 5

# Update VMSS instances
az vmss update-instances \
  --resource-group rg-compute \
  --name vmss-web \
  --instance-ids '*'

# List VMSS instances
az vmss list-instances \
  --resource-group rg-compute \
  --name vmss-web \
  --output table

# Delete VMSS
az vmss delete \
  --resource-group rg-compute \
  --name vmss-web
```

---

## 💾 Storage Accounts

### Create & Manage Storage

```bash
# Create storage account
az storage account create \
  --resource-group rg-storage \
  --name stproddata$(date +%s) \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2 \
  --access-tier Hot \
  --https-only true \
  --min-tls-version TLS1_2

# Get connection string
az storage account show-connection-string \
  --resource-group rg-storage \
  --name stproddata... \
  --output tsv

# Get account keys
az storage account keys list \
  --resource-group rg-storage \
  --name stproddata... \
  --output table

# Create blob container
az storage container create \
  --account-name stproddata... \
  --name data \
  --public-access off

# Upload blob
az storage blob upload \
  --account-name stproddata... \
  --container-name data \
  --name myfile.txt \
  --file ./myfile.txt

# Download blob
az storage blob download \
  --account-name stproddata... \
  --container-name data \
  --name myfile.txt \
  --file ./downloaded-myfile.txt

# List blobs
az storage blob list \
  --account-name stproddata... \
  --container-name data \
  --output table

# Delete blob
az storage blob delete \
  --account-name stproddata... \
  --container-name data \
  --name myfile.txt

# Generate SAS token
az storage blob generate-sas \
  --account-name stproddata... \
  --container-name data \
  --name myfile.txt \
  --permissions r \
  --expiry 2026-12-31T23:59:00Z \
  --output tsv
```

### File Shares

```bash
# Create file share
az storage share create \
  --account-name stproddata... \
  --name share1 \
  --quota 100

# List file shares
az storage share list \
  --account-name stproddata... \
  --output table

# Upload file
az storage file upload \
  --account-name stproddata... \
  --share-name share1 \
  --source ./localfile.txt \
  --path remotefile.txt

# List files
az storage file list \
  --account-name stproddata... \
  --share-name share1 \
  --output table
```

---

## 🗄️ Azure SQL Database

```bash
# Create SQL Server
az sql server create \
  --resource-group rg-databases \
  --name sql-prod-server \
  --location eastus \
  --admin-user sqladmin \
  --admin-password 'P@ssw0rd1234!'

# Create SQL Database
az sql db create \
  --resource-group rg-databases \
  --server sql-prod-server \
  --name sqldb-prod \
  --service-objective S1 \
  --zone-redundant false

# List databases
az sql db list \
  --resource-group rg-databases \
  --server sql-prod-server \
  --output table

# Show database
az sql db show \
  --resource-group rg-databases \
  --server sql-prod-server \
  --name sqldb-prod

# Add firewall rule
az sql server firewall-rule create \
  --resource-group rg-databases \
  --server sql-prod-server \
  --name AllowMyIP \
  --start-ip-address 203.0.113.5 \
  --end-ip-address 203.0.113.5

# Get connection string
az sql db show-connection-string \
  --client ado.net \
  --server sql-prod-server \
  --name sqldb-prod

# Create read replica
az sql db replica create \
  --resource-group rg-databases \
  --server sql-prod-server \
  --name sqldb-prod \
  --partner-server sql-prod-server-secondary \
  --partner-resource-group rg-databases-secondary
```

---

## ☸️ Azure Kubernetes Service (AKS)

### Create & Manage AKS

```bash
# Create AKS cluster
az aks create \
  --resource-group rg-aks \
  --name aks-prod-cluster \
  --kubernetes-version 1.28.0 \
  --node-count 3 \
  --node-vm-size Standard_D2s_v3 \
  --network-plugin azure \
  --network-policy calico \
  --enable-managed-identity \
  --generate-ssh-keys

# Get credentials
az aks get-credentials \
  --resource-group rg-aks \
  --name aks-prod-cluster

# List AKS clusters
az aks list --output table

# Show cluster details
az aks show \
  --resource-group rg-aks \
  --name aks-prod-cluster

# Scale cluster
az aks scale \
  --resource-group rg-aks \
  --name aks-prod-cluster \
  --node-count 5

# Add node pool
az aks nodepool add \
  --resource-group rg-aks \
  --cluster-name aks-prod-cluster \
  --name userpool \
  --node-count 3 \
  --node-vm-size Standard_D4s_v3

# List node pools
az aks nodepool list \
  --resource-group rg-aks \
  --cluster-name aks-prod-cluster \
  --output table

# Upgrade cluster
az aks upgrade \
  --resource-group rg-aks \
  --name aks-prod-cluster \
  --kubernetes-version 1.29.0

# Start/Stop cluster
az aks stop \
  --resource-group rg-aks \
  --name aks-prod-cluster

az aks start \
  --resource-group rg-aks \
  --name aks-prod-cluster

# Delete cluster
az aks delete \
  --resource-group rg-aks \
  --name aks-prod-cluster \
  --yes
```

### AKS with ACR Integration

```bash
# Create ACR
az acr create \
  --resource-group rg-containers \
  --name acrprodregistry \
  --sku Premium

# Attach ACR to AKS
az aks update \
  --resource-group rg-aks \
  --name aks-prod-cluster \
  --attach-acr acrprodregistry

# Or during creation
az aks create \
  --resource-group rg-aks \
  --name aks-prod-cluster \
  --attach-acr acrprodregistry
```

---

## 🐳 Azure Container Registry (ACR)

```bash
# Create ACR
az acr create \
  --resource-group rg-containers \
  --name acrprodregistry \
  --sku Premium \
  --location eastus

# Login to ACR
az acr login --name acrprodregistry

# List repositories
az acr repository list \
  --name acrprodregistry \
  --output table

# Show repository tags
az acr repository show-tags \
  --name acrprodregistry \
  --repository myapp \
  --output table

# Import image
az acr import \
  --name acrprodregistry \
  --source docker.io/library/nginx:latest \
  --image nginx:latest

# Build image in ACR
az acr build \
  --registry acrprodregistry \
  --image myapp:v1 \
  --file Dockerfile \
  .

# Enable geo-replication
az acr replication create \
  --registry acrprodregistry \
  --location westus

# List replications
az acr replication list \
  --registry acrprodregistry \
  --output table

# Delete image
az acr repository delete \
  --name acrprodregistry \
  --image myapp:v1 \
  --yes
```

---

## 🔐 Azure Key Vault

```bash
# Create Key Vault
az keyvault create \
  --resource-group rg-security \
  --name kv-prod-secrets \
  --location eastus \
  --enable-soft-delete true \
  --enable-purge-protection true

# Set secret
az keyvault secret set \
  --vault-name kv-prod-secrets \
  --name db-password \
  --value 'P@ssw0rd1234!'

# Get secret
az keyvault secret show \
  --vault-name kv-prod-secrets \
  --name db-password \
  --query value \
  --output tsv

# List secrets
az keyvault secret list \
  --vault-name kv-prod-secrets \
  --output table

# Delete secret
az keyvault secret delete \
  --vault-name kv-prod-secrets \
  --name db-password

# Grant access to service principal
az keyvault set-policy \
  --vault-name kv-prod-secrets \
  --spn <SERVICE_PRINCIPAL_ID> \
  --secret-permissions get list

# Grant access to user
az keyvault set-policy \
  --vault-name kv-prod-secrets \
  --upn user@domain.com \
  --secret-permissions get list set delete
```

---

## 👤 Identity & Access Management

### Service Principals

```bash
# Create service principal
az ad sp create-for-rbac \
  --name sp-cicd-pipeline \
  --role Contributor \
  --scopes /subscriptions/{subscription-id}

# Create with specific role
az ad sp create-for-rbac \
  --name sp-aks-operator \
  --role "Azure Kubernetes Service Cluster User Role" \
  --scopes /subscriptions/{sub-id}/resourceGroups/rg-aks

# List service principals
az ad sp list --display-name sp-cicd-pipeline

# Show service principal
az ad sp show --id <APP_ID>

# Reset credentials
az ad sp credential reset \
  --id <APP_ID>

# Delete service principal
az ad sp delete --id <APP_ID>
```

### Managed Identities

```bash
# Create user-assigned managed identity
az identity create \
  --resource-group rg-identity \
  --name id-app-identity

# Assign to VM
az vm identity assign \
  --resource-group rg-compute \
  --name vm-web-01 \
  --identities id-app-identity

# Grant permissions
az role assignment create \
  --assignee <IDENTITY_PRINCIPAL_ID> \
  --role "Storage Blob Data Contributor" \
  --scope /subscriptions/{sub-id}/resourceGroups/rg-storage
```

### RBAC

```bash
# Assign role to user
az role assignment create \
  --assignee user@domain.com \
  --role "Virtual Machine Contributor" \
  --resource-group rg-compute

# Assign role to group
az role assignment create \
  --assignee <GROUP_OBJECT_ID> \
  --role "Network Contributor" \
  --scope /subscriptions/{subscription-id}

# List role assignments
az role assignment list \
  --resource-group rg-compute \
  --output table

# Remove role assignment
az role assignment delete \
  --assignee user@domain.com \
  --role "Virtual Machine Contributor" \
  --resource-group rg-compute

# List all roles
az role definition list --output table

# Show role definition
az role definition list \
  --name "Virtual Machine Contributor"
```

---

## 📊 Monitoring & Alerts

### Log Analytics

```bash
# Create Log Analytics workspace
az monitor log-analytics workspace create \
  --resource-group rg-monitoring \
  --workspace-name law-prod-monitoring \
  --location eastus

# Enable diagnostics
az monitor diagnostic-settings create \
  --resource <RESOURCE_ID> \
  --name diag-settings \
  --workspace <WORKSPACE_ID> \
  --logs '[{"category": "Administrative","enabled": true}]' \
  --metrics '[{"category": "AllMetrics","enabled": true}]'

# Query logs
az monitor log-analytics query \
  --workspace <WORKSPACE_ID> \
  --analytics-query "AzureActivity | where TimeGenerated > ago(1h) | project TimeGenerated, Caller, OperationName" \
  --output table
```

### Alerts

```bash
# Create action group
az monitor action-group create \
  --resource-group rg-monitoring \
  --name ag-devops-team \
  --short-name DevOps \
  --email-receiver name=Team email=devops@company.com

# Create metric alert
az monitor metrics alert create \
  --resource-group rg-monitoring \
  --name alert-high-cpu \
  --scopes <VM_ID> \
  --condition "avg Percentage CPU > 80" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --action ag-devops-team

# List alerts
az monitor metrics alert list \
  --resource-group rg-monitoring \
  --output table

# Delete alert
az monitor metrics alert delete \
  --resource-group rg-monitoring \
  --name alert-high-cpu
```

---

## 💰 Cost Management

```bash
# Create budget
az consumption budget create \
  --budget-name monthly-budget \
  --amount 10000 \
  --time-grain Monthly \
  --start-date 2026-01-01 \
  --end-date 2026-12-31 \
  --resource-group rg-prod

# Show cost
az consumption usage list \
  --start-date 2026-01-01 \
  --end-date 2026-01-31

# Get cost by resource group
az consumption usage list \
  --start-date 2026-01-01 \
  --end-date 2026-01-31 \
  | jq '[.[] | {name: .instanceName, cost: .pretaxCost}]'
```

---

## 🛠️ Common Patterns

### Resource Tagging

```bash
# Tag resource group
az group update \
  --name rg-prod \
  --tags Environment=Production CostCenter=IT Owner=DevOps Project=WebApp

# Tag resource
az resource tag \
  --resource-group rg-prod \
  --name vm-web-01 \
  --resource-type "Microsoft.Compute/virtualMachines" \
  --tags Environment=Production Tier=Web

# Query resources by tag
az resource list \
  --tag Environment=Production \
  --output table
```

### Bulk Operations

```bash
# Delete all resources in resource group
az resource list \
  --resource-group rg-old \
  --query "[].id" \
  --output tsv \
  | xargs -I {} az resource delete --ids {}

# Stop all VMs in resource group
az vm list \
  --resource-group rg-dev \
  --query "[].id" \
  --output tsv \
  | xargs -I {} az vm deallocate --ids {}

# Start all VMs in resource group
az vm list \
  --resource-group rg-dev \
  --query "[].id" \
  --output tsv \
  | xargs -I {} az vm start --ids {}
```

---

## 📝 Output Formatting

```bash
# Table output
az vm list --output table

# JSON output (default)
az vm list --output json

# YAML output
az vm list --output yaml

# TSV output
az vm list --output tsv

# JMESPath query
az vm list --query "[?powerState=='VM running'].{name:name, resourceGroup:resourceGroup}" --output table

# Get specific field
az vm show \
  --resource-group rg-compute \
  --name vm-web-01 \
  --query "osProfile.computerName" \
  --output tsv
```

---

**Save this reference for quick Azure operations! ☁️🚀**

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

