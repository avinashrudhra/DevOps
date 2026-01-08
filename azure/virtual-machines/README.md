# Azure Virtual Machines - Complete Learning Guide

**Comprehensive Tutorial: Configuration, Deployment, Scaling & Troubleshooting**

---

## 📚 What You'll Learn

This comprehensive guide covers everything about Azure Virtual Machines from basics to production deployment:

1. **VM Fundamentals** - Understanding VM concepts
2. **Configuration** - Sizing, networking, storage
3. **Deployment Methods**
   - Azure Portal
   - Azure CLI
   - Azure PowerShell
   - ARM Templates
   - Terraform
4. **VM Scale Sets (VMSS)** - Auto-scaling infrastructure
5. **Scaling Types** - Vertical & Horizontal scaling
6. **High Availability** - Availability sets, zones, load balancing
7. **Backup & Recovery** - Disaster recovery strategies
8. **Monitoring & Optimization** - Performance tuning
9. **Troubleshooting** - Common issues and solutions
10. **Best Practices** - Production-ready patterns

---

## 🎯 Prerequisites

- Azure subscription
- Azure CLI installed
- Basic Linux/Windows server knowledge
- Understanding of networking basics
- Git (for Terraform examples)

---

## 📖 Table of Contents

1. [VM Fundamentals](#vm-fundamentals)
2. [VM Configuration](#vm-configuration)
3. [Deployment with Azure Portal](#deployment-with-azure-portal)
4. [Deployment with Azure CLI](#deployment-with-azure-cli)
5. [Deployment with PowerShell](#deployment-with-powershell)
6. [Deployment with ARM Templates](#deployment-with-arm-templates)
7. [Deployment with Terraform](#deployment-with-terraform)
8. [VM Scale Sets](#vm-scale-sets)
9. [Scaling Strategies](#scaling-strategies)
10. [High Availability](#high-availability)
11. [Backup & Disaster Recovery](#backup--disaster-recovery)
12. [Monitoring & Performance](#monitoring--performance)
13. [Troubleshooting Guide](#troubleshooting-guide)
14. [Best Practices](#best-practices)

---

## 🖥️ VM Fundamentals

### **What is an Azure VM?**

An Azure Virtual Machine is an on-demand, scalable computing resource that provides the flexibility of virtualization without having to buy and maintain physical hardware.

### **VM Components:**

```
┌─────────────────────────────────────────┐
│          Azure Virtual Machine          │
│  ┌───────────────────────────────────┐  │
│  │      Operating System             │  │
│  │  (Windows/Linux)                  │  │
│  └───────────────────────────────────┘  │
│  ┌───────────────────────────────────┐  │
│  │      Compute (CPU/RAM)            │  │
│  │  Size: Standard_D2s_v3            │  │
│  └───────────────────────────────────┘  │
│  ┌───────────────────────────────────┐  │
│  │      Storage (Disks)              │  │
│  │  - OS Disk (SSD/HDD)              │  │
│  │  - Data Disks                     │  │
│  └───────────────────────────────────┘  │
│  ┌───────────────────────────────────┐  │
│  │      Networking                   │  │
│  │  - Virtual Network (VNet)         │  │
│  │  - Network Interface (NIC)        │  │
│  │  - Public/Private IP              │  │
│  │  - Network Security Group (NSG)   │  │
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

### **VM Series & Use Cases:**

| Series | vCPU | RAM | Use Case |
|--------|------|-----|----------|
| **B-Series** | 1-20 | 0.5-80 GB | Burstable, dev/test, low traffic |
| **D-Series** | 2-64 | 8-256 GB | General purpose, balanced |
| **E-Series** | 2-64 | 16-432 GB | Memory-intensive, databases |
| **F-Series** | 2-64 | 4-128 GB | Compute-intensive, batch |
| **M-Series** | 8-416 | 224-11,400 GB | Large in-memory workloads |
| **N-Series** | 6-24 | 112-448 GB | GPU, AI/ML, graphics |

---

## ⚙️ VM Configuration

### **1. Choosing VM Size**

```bash
# List all available VM sizes in region
az vm list-sizes --location eastus --output table

# List by family
az vm list-sizes --location eastus --query "[?contains(name, 'Standard_D')]" --output table

# Check size availability
az vm list-skus --location eastus --size Standard_D --all --output table
```

### **2. Storage Configuration**

**Disk Types:**

| Type | IOPS | Throughput | Use Case |
|------|------|------------|----------|
| **Standard HDD** | 500 | 60 MB/s | Non-critical, backups |
| **Standard SSD** | 500-6000 | 60-750 MB/s | Web servers, dev/test |
| **Premium SSD** | 120-20000 | 25-900 MB/s | Production, databases |
| **Ultra Disk** | 160K-400K | 2000-4000 MB/s | Intensive I/O, SAP HANA |

**Disk Configuration Example:**
```bash
# Create VM with Premium SSD
az vm create \
  --resource-group rg-prod \
  --name vm-web-01 \
  --image Ubuntu2204 \
  --size Standard_D2s_v3 \
  --storage-sku Premium_LRS \
  --os-disk-size-gb 128 \
  --data-disk-sizes-gb 512 1024
```

### **3. Networking Configuration**

```bash
# Create VM with custom networking
az vm create \
  --resource-group rg-prod \
  --name vm-web-01 \
  --image Ubuntu2204 \
  --vnet-name vnet-prod \
  --subnet subnet-web \
  --nsg nsg-web \
  --public-ip-address pip-vm-web-01 \
  --public-ip-sku Standard \
  --private-ip-address 10.0.1.10
```

---

## 🚀 Deployment Methods

### **Method 1: Azure Portal (GUI)**

**Step-by-Step:**

1. **Navigate to Azure Portal** → Virtual machines → Create

2. **Basics Configuration:**
   ```
   Subscription: Your subscription
   Resource Group: rg-prod (create new)
   VM Name: vm-web-01
   Region: East US
   Image: Ubuntu Server 22.04 LTS
   Size: Standard_D2s_v3 (2 vCPUs, 8 GB RAM)
   Authentication: SSH public key
   Username: azureuser
   ```

3. **Disks Configuration:**
   ```
   OS disk type: Premium SSD
   Encryption: (Default) Encryption at-rest
   Data disks: Add new disk → 512 GB Premium SSD
   ```

4. **Networking:**
   ```
   Virtual network: vnet-prod (create new: 10.0.0.0/16)
   Subnet: subnet-web (10.0.1.0/24)
   Public IP: (new) pip-vm-web-01
   NIC NSG: Basic
   Public inbound ports: SSH (22), HTTP (80), HTTPS (443)
   ```

5. **Management:**
   ```
   Enable auto-shutdown: Yes (7:00 PM)
   Boot diagnostics: Enable with managed storage account
   OS guest diagnostics: Enable
   ```

6. **Review + Create** → Create

---

### **Method 2: Azure CLI**

**Complete Deployment Script:**

```bash
#!/bin/bash

# Variables
RESOURCE_GROUP="rg-prod-vms"
LOCATION="eastus"
VM_NAME="vm-web-01"
VM_SIZE="Standard_D2s_v3"
IMAGE="Ubuntu2204"
ADMIN_USER="azureuser"

# Create resource group
az group create \
  --name $RESOURCE_GROUP \
  --location $LOCATION \
  --tags Environment=Production Owner=DevOps

# Create virtual network
az network vnet create \
  --resource-group $RESOURCE_GROUP \
  --name vnet-prod \
  --address-prefix 10.0.0.0/16 \
  --subnet-name subnet-web \
  --subnet-prefix 10.0.1.0/24

# Create NSG with rules
az network nsg create \
  --resource-group $RESOURCE_GROUP \
  --name nsg-web

az network nsg rule create \
  --resource-group $RESOURCE_GROUP \
  --nsg-name nsg-web \
  --name AllowSSH \
  --priority 1000 \
  --source-address-prefixes '*' \
  --destination-port-ranges 22 \
  --access Allow \
  --protocol Tcp

az network nsg rule create \
  --resource-group $RESOURCE_GROUP \
  --nsg-name nsg-web \
  --name AllowHTTP \
  --priority 1001 \
  --destination-port-ranges 80 443 \
  --access Allow \
  --protocol Tcp

# Create public IP
az network public-ip create \
  --resource-group $RESOURCE_GROUP \
  --name pip-$VM_NAME \
  --sku Standard \
  --allocation-method Static

# Create VM
az vm create \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --location $LOCATION \
  --image $IMAGE \
  --size $VM_SIZE \
  --vnet-name vnet-prod \
  --subnet subnet-web \
  --nsg nsg-web \
  --public-ip-address pip-$VM_NAME \
  --admin-username $ADMIN_USER \
  --generate-ssh-keys \
  --storage-sku Premium_LRS \
  --os-disk-size-gb 128 \
  --data-disk-sizes-gb 512 \
  --custom-data cloud-init.txt \
  --tags Application=Web Environment=Production

# Get VM details
az vm show \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --show-details \
  --output table

# Get public IP
PUBLIC_IP=$(az vm list-ip-addresses \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --query "[0].virtualMachine.network.publicIpAddresses[0].ipAddress" \
  --output tsv)

echo "VM Created Successfully!"
echo "Public IP: $PUBLIC_IP"
echo "SSH Command: ssh $ADMIN_USER@$PUBLIC_IP"
```

**Cloud-Init Configuration (cloud-init.txt):**
```yaml
#cloud-config
package_update: true
package_upgrade: true

packages:
  - nginx
  - docker.io
  - git
  - htop
  - curl

runcmd:
  - systemctl start nginx
  - systemctl enable nginx
  - systemctl start docker
  - systemctl enable docker
  - usermod -aG docker azureuser
  - echo "<h1>Azure VM - $(hostname)</h1>" > /var/www/html/index.html
```

---

### **Method 3: Azure PowerShell**

```powershell
# Variables
$resourceGroup = "rg-prod-vms"
$location = "East US"
$vmName = "vm-web-01"
$vmSize = "Standard_D2s_v3"

# Create resource group
New-AzResourceGroup `
  -Name $resourceGroup `
  -Location $location `
  -Tag @{Environment="Production"; Owner="DevOps"}

# Create VM credentials
$cred = Get-Credential -Message "Enter username and password for VM"

# Create VM
New-AzVm `
  -ResourceGroupName $resourceGroup `
  -Name $vmName `
  -Location $location `
  -Size $vmSize `
  -Image "Ubuntu2204" `
  -VirtualNetworkName "vnet-prod" `
  -SubnetName "subnet-web" `
  -SecurityGroupName "nsg-web" `
  -PublicIpAddressName "pip-$vmName" `
  -Credential $cred `
  -OpenPorts 22,80,443

# Get VM details
Get-AzVM -ResourceGroupName $resourceGroup -Name $vmName -Status

# Get public IP
$publicIp = (Get-AzPublicIpAddress `
  -ResourceGroupName $resourceGroup `
  -Name "pip-$vmName").IpAddress

Write-Host "VM Created Successfully!"
Write-Host "Public IP: $publicIp"
Write-Host "SSH Command: ssh $($cred.UserName)@$publicIp"
```

---

### **Method 4: ARM Templates**

**Template (vm-template.json):**
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "type": "string",
      "defaultValue": "vm-web-01",
      "metadata": {
        "description": "Virtual machine name"
      }
    },
    "vmSize": {
      "type": "string",
      "defaultValue": "Standard_D2s_v3",
      "metadata": {
        "description": "VM size"
      }
    },
    "adminUsername": {
      "type": "string",
      "metadata": {
        "description": "Admin username"
      }
    },
    "adminPassword": {
      "type": "securestring",
      "metadata": {
        "description": "Admin password"
      }
    }
  },
  "variables": {
    "vnetName": "vnet-prod",
    "subnetName": "subnet-web",
    "nsgName": "nsg-web",
    "publicIPName": "[concat('pip-', parameters('vmName'))]",
    "nicName": "[concat(parameters('vmName'), '-nic')]"
  },
  "resources": [
    {
      "type": "Microsoft.Network/virtualNetworks",
      "apiVersion": "2021-02-01",
      "name": "[variables('vnetName')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "addressSpace": {
          "addressPrefixes": ["10.0.0.0/16"]
        },
        "subnets": [
          {
            "name": "[variables('subnetName')]",
            "properties": {
              "addressPrefix": "10.0.1.0/24"
            }
          }
        ]
      }
    },
    {
      "type": "Microsoft.Network/networkSecurityGroups",
      "apiVersion": "2021-02-01",
      "name": "[variables('nsgName')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "securityRules": [
          {
            "name": "AllowSSH",
            "properties": {
              "priority": 1000,
              "protocol": "Tcp",
              "access": "Allow",
              "direction": "Inbound",
              "sourceAddressPrefix": "*",
              "sourcePortRange": "*",
              "destinationAddressPrefix": "*",
              "destinationPortRange": "22"
            }
          }
        ]
      }
    },
    {
      "type": "Microsoft.Network/publicIPAddresses",
      "apiVersion": "2021-02-01",
      "name": "[variables('publicIPName')]",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard"
      },
      "properties": {
        "publicIPAllocationMethod": "Static"
      }
    },
    {
      "type": "Microsoft.Network/networkInterfaces",
      "apiVersion": "2021-02-01",
      "name": "[variables('nicName')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Network/publicIPAddresses', variables('publicIPName'))]",
        "[resourceId('Microsoft.Network/virtualNetworks', variables('vnetName'))]",
        "[resourceId('Microsoft.Network/networkSecurityGroups', variables('nsgName'))]"
      ],
      "properties": {
        "ipConfigurations": [
          {
            "name": "ipconfig1",
            "properties": {
              "subnet": {
                "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('vnetName'), variables('subnetName'))]"
              },
              "privateIPAllocationMethod": "Dynamic",
              "publicIPAddress": {
                "id": "[resourceId('Microsoft.Network/publicIPAddresses', variables('publicIPName'))]"
              }
            }
          }
        ],
        "networkSecurityGroup": {
          "id": "[resourceId('Microsoft.Network/networkSecurityGroups', variables('nsgName'))]"
        }
      }
    },
    {
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2021-03-01",
      "name": "[parameters('vmName')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]"
      ],
      "properties": {
        "hardwareProfile": {
          "vmSize": "[parameters('vmSize')]"
        },
        "osProfile": {
          "computerName": "[parameters('vmName')]",
          "adminUsername": "[parameters('adminUsername')]",
          "adminPassword": "[parameters('adminPassword')]"
        },
        "storageProfile": {
          "imageReference": {
            "publisher": "Canonical",
            "offer": "0001-com-ubuntu-server-jammy",
            "sku": "22_04-lts-gen2",
            "version": "latest"
          },
          "osDisk": {
            "createOption": "FromImage",
            "managedDisk": {
              "storageAccountType": "Premium_LRS"
            },
            "diskSizeGB": 128
          }
        },
        "networkProfile": {
          "networkInterfaces": [
            {
              "id": "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]"
            }
          ]
        }
      }
    }
  ],
  "outputs": {
    "publicIP": {
      "type": "string",
      "value": "[reference(variables('publicIPName')).ipAddress]"
    }
  }
}
```

**Deploy ARM Template:**
```bash
# Deploy template
az deployment group create \
  --resource-group rg-prod-vms \
  --template-file vm-template.json \
  --parameters \
    vmName=vm-web-01 \
    vmSize=Standard_D2s_v3 \
    adminUsername=azureuser \
    adminPassword='P@ssw0rd1234!'
```

---

### **Method 5: Terraform**

**Complete Terraform Configuration (main.tf):**
```hcl
# Provider configuration
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

# Variables
variable "resource_group_name" {
  default = "rg-prod-vms"
}

variable "location" {
  default = "East US"
}

variable "vm_name" {
  default = "vm-web-01"
}

variable "vm_size" {
  default = "Standard_D2s_v3"
}

variable "admin_username" {
  default = "azureuser"
}

# Resource Group
resource "azurerm_resource_group" "main" {
  name     = var.resource_group_name
  location = var.location

  tags = {
    Environment = "Production"
    Owner       = "DevOps"
  }
}

# Virtual Network
resource "azurerm_virtual_network" "main" {
  name                = "vnet-prod"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}

# Subnet
resource "azurerm_subnet" "main" {
  name                 = "subnet-web"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.1.0/24"]
}

# Network Security Group
resource "azurerm_network_security_group" "main" {
  name                = "nsg-web"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  security_rule {
    name                       = "AllowSSH"
    priority                   = 1000
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "22"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }

  security_rule {
    name                       = "AllowHTTP"
    priority                   = 1001
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_ranges    = ["80", "443"]
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }
}

# Public IP
resource "azurerm_public_ip" "main" {
  name                = "pip-${var.vm_name}"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  allocation_method   = "Static"
  sku                 = "Standard"
}

# Network Interface
resource "azurerm_network_interface" "main" {
  name                = "${var.vm_name}-nic"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  ip_configuration {
    name                          = "ipconfig1"
    subnet_id                     = azurerm_subnet.main.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.main.id
  }
}

# Associate NSG with NIC
resource "azurerm_network_interface_security_group_association" "main" {
  network_interface_id      = azurerm_network_interface.main.id
  network_security_group_id = azurerm_network_security_group.main.id
}

# Virtual Machine
resource "azurerm_linux_virtual_machine" "main" {
  name                = var.vm_name
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  size                = var.vm_size
  admin_username      = var.admin_username

  network_interface_ids = [
    azurerm_network_interface.main.id,
  ]

  admin_ssh_key {
    username   = var.admin_username
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

  custom_data = base64encode(file("cloud-init.txt"))

  tags = {
    Application = "Web"
    Environment = "Production"
  }
}

# Data Disk
resource "azurerm_managed_disk" "data" {
  name                 = "${var.vm_name}-datadisk1"
  location             = azurerm_resource_group.main.location
  resource_group_name  = azurerm_resource_group.main.name
  storage_account_type = "Premium_LRS"
  create_option        = "Empty"
  disk_size_gb         = 512
}

# Attach Data Disk
resource "azurerm_virtual_machine_data_disk_attachment" "data" {
  managed_disk_id    = azurerm_managed_disk.data.id
  virtual_machine_id = azurerm_linux_virtual_machine.main.id
  lun                = 0
  caching            = "ReadWrite"
}

# Outputs
output "public_ip" {
  value = azurerm_public_ip.main.ip_address
}

output "ssh_command" {
  value = "ssh ${var.admin_username}@${azurerm_public_ip.main.ip_address}"
}
```

**Deploy with Terraform:**
```bash
# Initialize Terraform
terraform init

# Plan deployment
terraform plan

# Apply configuration
terraform apply -auto-approve

# Get outputs
terraform output

# Destroy resources
terraform destroy -auto-approve
```

---

## 🚀 VM Scale Sets (VMSS)

**VM Scale Sets** enable you to create and manage identical VMs that auto-scale based on demand.

### **When to Use VMSS:**

**✅ Good For:**
- Web applications with variable traffic
- Stateless applications
- Batch processing workloads
- Microservices
- Load-balanced applications

**❌ Not Good For:**
- Stateful applications requiring persistent storage
- Applications requiring specific VM customization
- Single-instance applications

---

### **Create VMSS - Azure CLI:**

```bash
# Create VM Scale Set
az vmss create \
  --resource-group rg-vmss \
  --name vmss-web-prod \
  --image Ubuntu2204 \
  --vm-sku Standard_D2s_v3 \
  --instance-count 3 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --vnet-name vnet-prod \
  --subnet subnet-web \
  --lb vmss-lb \
  --backend-pool-name vmss-backend \
  --upgrade-policy-mode Automatic \
  --zones 1 2 3

# Enable auto-scaling
az monitor autoscale create \
  --resource-group rg-vmss \
  --resource $(az vmss show --resource-group rg-vmss --name vmss-web-prod --query id --output tsv) \
  --name vmss-autoscale \
  --min-count 3 \
  --max-count 10 \
  --count 3

# Scale out when CPU > 75%
az monitor autoscale rule create \
  --resource-group rg-vmss \
  --autoscale-name vmss-autoscale \
  --condition "Percentage CPU > 75 avg 5m" \
  --scale out 2

# Scale in when CPU < 25%
az monitor autoscale rule create \
  --resource-group rg-vmss \
  --autoscale-name vmss-autoscale \
  --condition "Percentage CPU < 25 avg 5m" \
  --scale in 1

# Manual scale
az vmss scale \
  --resource-group rg-vmss \
  --name vmss-web-prod \
  --new-capacity 5

# Update VMSS (rolling upgrade)
az vmss update \
  --resource-group rg-vmss \
  --name vmss-web-prod \
  --set upgradePolicy.mode=Rolling

# List VMSS instances
az vmss list-instances \
  --resource-group rg-vmss \
  --name vmss-web-prod \
  --output table
```

---

### **VMSS with Custom Script Extension:**

```bash
# Deploy VMSS with custom script
az vmss create \
  --resource-group rg-vmss \
  --name vmss-web-prod \
  --image Ubuntu2204 \
  --vm-sku Standard_D2s_v3 \
  --instance-count 3 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --custom-data cloud-init-web.txt

# cloud-init-web.txt
cat > cloud-init-web.txt << 'EOF'
#cloud-config
package_update: true
packages:
  - nginx
  - nodejs
  - npm

write_files:
  - path: /var/www/html/index.html
    content: |
      <!DOCTYPE html>
      <html>
      <head><title>VMSS Instance</title></head>
      <body>
        <h1>VM Instance: $(hostname)</h1>
        <p>Running on Azure VMSS</p>
      </body>
      </html>

runcmd:
  - systemctl enable nginx
  - systemctl start nginx
  - echo "Instance deployed: $(date)" >> /var/log/deployment.log
EOF
```

---

### **Terraform - VMSS:**

```hcl
resource "azurerm_linux_virtual_machine_scale_set" "main" {
  name                = "vmss-web-prod"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  sku                 = "Standard_D2s_v3"
  instances           = 3
  admin_username      = "azureuser"
  
  admin_ssh_key {
    username   = "azureuser"
    public_key = file("~/.ssh/id_rsa.pub")
  }
  
  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts-gen2"
    version   = "latest"
  }
  
  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Premium_LRS"
  }
  
  network_interface {
    name    = "vmss-nic"
    primary = true
    
    ip_configuration {
      name      = "internal"
      primary   = true
      subnet_id = azurerm_subnet.web.id
      
      load_balancer_backend_address_pool_ids = [
        azurerm_lb_backend_address_pool.main.id
      ]
    }
  }
  
  custom_data = base64encode(file("cloud-init-web.txt"))
  
  zones = ["1", "2", "3"]
  
  upgrade_mode = "Rolling"
  
  rolling_upgrade_policy {
    max_batch_instance_percent              = 20
    max_unhealthy_instance_percent          = 20
    max_unhealthy_upgraded_instance_percent = 20
    pause_time_between_batches              = "PT5S"
  }
  
  health_probe_id = azurerm_lb_probe.main.id
  
  automatic_os_upgrade_policy {
    disable_automatic_rollback  = false
    enable_automatic_os_upgrade = true
  }
}

# Auto-scaling
resource "azurerm_monitor_autoscale_setting" "main" {
  name                = "vmss-autoscale"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  target_resource_id  = azurerm_linux_virtual_machine_scale_set.main.id

  profile {
    name = "AutoScale"

    capacity {
      default = 3
      minimum = 3
      maximum = 10
    }

    rule {
      metric_trigger {
        metric_name        = "Percentage CPU"
        metric_resource_id = azurerm_linux_virtual_machine_scale_set.main.id
        time_grain         = "PT1M"
        statistic          = "Average"
        time_window        = "PT5M"
        time_aggregation   = "Average"
        operator           = "GreaterThan"
        threshold          = 75
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
        metric_name        = "Percentage CPU"
        metric_resource_id = azurerm_linux_virtual_machine_scale_set.main.id
        time_grain         = "PT1M"
        statistic          = "Average"
        time_window        = "PT5M"
        time_aggregation   = "Average"
        operator           = "LessThan"
        threshold          = 25
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

## 📊 High Availability Patterns

### **1. Availability Sets**

**Protects against**: Hardware failures within a datacenter

```bash
# Create availability set
az vm availability-set create \
  --resource-group rg-compute \
  --name avset-web \
  --platform-fault-domain-count 2 \
  --platform-update-domain-count 5

# Create VMs in availability set
az vm create \
  --resource-group rg-compute \
  --name vm-web-01 \
  --availability-set avset-web \
  --image Ubuntu2204 \
  --size Standard_D2s_v3
```

**Key Concepts:**
- **Fault Domains**: Separate physical racks (Max: 3)
- **Update Domains**: Groups for planned maintenance (Max: 20)
- **SLA**: 99.95% uptime with 2+ VMs

---

### **2. Availability Zones**

**Protects against**: Entire datacenter failures

```bash
# Create VMs across zones
az vm create \
  --resource-group rg-compute \
  --name vm-web-01 \
  --zone 1 \
  --image Ubuntu2204

az vm create \
  --resource-group rg-compute \
  --name vm-web-02 \
  --zone 2 \
  --image Ubuntu2204

az vm create \
  --resource-group rg-compute \
  --name vm-web-03 \
  --zone 3 \
  --image Ubuntu2204
```

**Benefits:**
- Physically separate datacenters
- Independent power, cooling, networking
- **SLA**: 99.99% uptime
- Available in select regions

---

### **3. Load Balancer for HA:**

```bash
# Create load balancer
az network lb create \
  --resource-group rg-compute \
  --name lb-web \
  --sku Standard \
  --frontend-ip-name lb-frontend \
  --backend-pool-name lb-backend

# Create health probe
az network lb probe create \
  --resource-group rg-compute \
  --lb-name lb-web \
  --name health-probe \
  --protocol tcp \
  --port 80 \
  --interval 15 \
  --threshold 2

# Create load balancing rule
az network lb rule create \
  --resource-group rg-compute \
  --lb-name lb-web \
  --name lb-rule-http \
  --protocol tcp \
  --frontend-port 80 \
  --backend-port 80 \
  --frontend-ip-name lb-frontend \
  --backend-pool-name lb-backend \
  --probe-name health-probe

# Add VMs to backend pool
az network nic ip-config address-pool add \
  --resource-group rg-compute \
  --nic-name vm-web-01-nic \
  --ip-config-name ipconfig1 \
  --lb-name lb-web \
  --address-pool lb-backend
```

---

## 💾 Backup & Disaster Recovery

### **Azure Backup:**

```bash
# Create Recovery Services Vault
az backup vault create \
  --resource-group rg-backup \
  --name vault-backup-prod \
  --location eastus

# Enable backup on VM
az backup protection enable-for-vm \
  --resource-group rg-compute \
  --vault-name vault-backup-prod \
  --vm vm-web-01 \
  --policy-name DefaultPolicy

# Trigger on-demand backup
az backup protection backup-now \
  --resource-group rg-backup \
  --vault-name vault-backup-prod \
  --container-name vm-web-01 \
  --item-name vm-web-01 \
  --retain-until 30-12-2024

# List recovery points
az backup recoverypoint list \
  --resource-group rg-backup \
  --vault-name vault-backup-prod \
  --container-name vm-web-01 \
  --item-name vm-web-01 \
  --output table

# Restore VM
az backup restore restore-disks \
  --resource-group rg-backup \
  --vault-name vault-backup-prod \
  --container-name vm-web-01 \
  --item-name vm-web-01 \
  --rp-name <recovery-point-name> \
  --storage-account stbackupdata001
```

---

### **Azure Site Recovery (DR):**

```bash
# Enable replication to secondary region
az backup vault create \
  --resource-group rg-dr \
  --name vault-dr-prod \
  --location westus

# Configure replication (via Portal or ARM template)
# 1. Enable Site Recovery
# 2. Configure replication policy
# 3. Enable replication for VMs
# 4. Test failover
# 5. Planned/Unplanned failover when needed
```

---

## 📈 Monitoring & Performance

### **Enable Azure Monitor:**

```bash
# Enable diagnostic settings
az monitor diagnostic-settings create \
  --resource $(az vm show --resource-group rg-compute --name vm-web-01 --query id --output tsv) \
  --name vm-diagnostics \
  --workspace $(az monitor log-analytics workspace show --resource-group rg-monitoring --workspace-name law-prod --query id --output tsv) \
  --metrics '[{"category": "AllMetrics","enabled": true}]'

# View VM metrics
az monitor metrics list \
  --resource $(az vm show --resource-group rg-compute --name vm-web-01 --query id --output tsv) \
  --metric "Percentage CPU" "Available Memory Bytes" "Disk Read Bytes" \
  --start-time 2024-01-01T00:00:00Z \
  --end-time 2024-01-02T00:00:00Z \
  --interval PT1M
```

---

### **Performance Tuning Tips:**

**1. Choose Right VM Size:**
- Use **D-series** for balanced workloads
- Use **E-series** for memory-intensive (databases)
- Use **F-series** for CPU-intensive (batch processing)

**2. Optimize Disks:**
- Use **Premium SSD** for production workloads
- Enable **host caching** for OS disks (ReadWrite)
- Use **ReadOnly** caching for data disks

**3. Network Performance:**
- Enable **Accelerated Networking** for low latency
- Use proximity placement groups for multi-VM apps

```bash
# Enable accelerated networking
az network nic update \
  --resource-group rg-compute \
  --name vm-web-01-nic \
  --accelerated-networking true
```

---

## 🛠️ Comprehensive Troubleshooting

### **Issue 1: VM Won't Start**

**Symptoms**: VM stuck in "Starting" state

**Debugging:**
```bash
# Check VM status
az vm get-instance-view \
  --resource-group rg-compute \
  --name vm-web-01

# View boot diagnostics
az vm boot-diagnostics get-boot-log \
  --resource-group rg-compute \
  --name vm-web-01

# Check for Azure issues
az vm list-skus --location eastus --output table | grep Standard_D2s_v3
```

**Common Causes & Solutions:**
- **Quota exceeded**: Request quota increase
- **Disk issues**: Check disk health, detach/reattach
- **Network misconfiguration**: Verify NSG rules, subnet config

---

### **Issue 2: Cannot SSH/RDP to VM**

**Debugging:**
```bash
# Test connectivity
nslookup vm-web-01.eastus.cloudapp.azure.com
telnet <PUBLIC_IP> 22

# Check NSG rules
az network nsg rule list \
  --resource-group rg-compute \
  --nsg-name nsg-web \
  --output table

# Reset password
az vm user update \
  --resource-group rg-compute \
  --name vm-web-01 \
  --username azureuser \
  --password 'NewP@ssw0rd!'

# Run commands on VM (if SSH not working)
az vm run-command invoke \
  --resource-group rg-compute \
  --name vm-web-01 \
  --command-id RunShellScript \
  --scripts "systemctl status sshd"
```

---

### **Issue 3: High CPU Usage**

**Debugging:**
```bash
# Connect to VM
ssh azureuser@<PUBLIC_IP>

# Check CPU usage
top
htop

# Find CPU-intensive processes
ps aux --sort=-%cpu | head -10

# Check system load
uptime
w

# Review logs
journalctl -xe
dmesg | tail -50
```

**Solutions:**
- Resize to larger VM
- Optimize application code
- Enable auto-scaling (VMSS)
- Add more instances behind load balancer

---

### **Issue 4: Disk Space Full**

```bash
# Check disk usage
df -h

# Find large files
du -sh /* | sort -hr | head -10

# Clean up
sudo apt-get clean
sudo journalctl --vacuum-time=7d
sudo docker system prune -a  # If using Docker

# Expand disk
az vm deallocate --resource-group rg-compute --name vm-web-01
az disk update --resource-group rg-compute --name vm-web-01_OsDisk_1 --size-gb 256
az vm start --resource-group rg-compute --name vm-web-01

# Resize partition inside VM
sudo growpart /dev/sda 1
sudo resize2fs /dev/sda1
```

---

## ✅ Production Best Practices

### **1. Security:**
- ✅ Disable password authentication, use SSH keys only
- ✅ Use Just-In-Time (JIT) VM access
- ✅ Enable Azure Security Center recommendations
- ✅ Regular security patches with Update Management
- ✅ Use managed identities instead of service principals
- ✅ Encrypt disks with Azure Disk Encryption

```bash
# Enable JIT access
az security jit-policy create \
  --resource-group rg-compute \
  --name vm-web-01 \
  --location eastus \
  --virtual-machines "/subscriptions/{sub-id}/resourceGroups/rg-compute/providers/Microsoft.Compute/virtualMachines/vm-web-01" \
  --ports '[{"number":22,"protocol":"TCP","allowedSourceAddresses":["*"],"maxRequestAccessDuration":"PT3H"}]'
```

---

### **2. Cost Optimization:**
- ✅ Use **Reserved Instances** (1-3 year commitment, up to 72% savings)
- ✅ Use **Spot VMs** for fault-tolerant workloads (up to 90% savings)
- ✅ Auto-shutdown dev/test VMs at night
- ✅ Right-size VMs based on actual usage
- ✅ Use Azure Hybrid Benefit for Windows licenses

```bash
# Create Spot VM
az vm create \
  --resource-group rg-compute \
  --name vm-spot-01 \
  --priority Spot \
  --max-price 0.05 \
  --eviction-policy Deallocate \
  --image Ubuntu2204 \
  --size Standard_D2s_v3

# Auto-shutdown schedule
az vm auto-shutdown \
  --resource-group rg-compute \
  --name vm-web-01 \
  --time 1900 \
  --email devops@company.com
```

---

### **3. Naming Conventions:**

```
vm-{workload}-{environment}-{region}-{instance}

Examples:
- vm-web-prod-eastus-01
- vm-api-dev-westus-01
- vm-db-staging-eastus-02
```

---

### **4. Tagging Strategy:**

```bash
az vm update \
  --resource-group rg-compute \
  --name vm-web-01 \
  --set tags.Environment=Production \
       tags.Application=WebServer \
       tags.Owner=DevOps \
       tags.CostCenter=Engineering \
       tags.BackupPolicy=Daily
```

---

### **5. Monitoring & Alerting:**
- ✅ Enable Azure Monitor
- ✅ Configure alerts for CPU, memory, disk
- ✅ Enable boot diagnostics
- ✅ Use Log Analytics for centralized logging
- ✅ Set up action groups for notifications

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

