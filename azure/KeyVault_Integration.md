# Azure Key Vault with Private Access - Complete Setup Guide

## 📋 Table of Contents
- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Cost Estimate](#cost-estimate)
- [Setup Steps](#setup-steps)
- [Testing Key Vault Access](#testing-key-vault-access)
- [Troubleshooting](#troubleshooting)
- [Cleanup](#cleanup)
- [Security Best Practices](#security-best-practices)
- [Frequently Asked Questions (FAQ)](#frequently-asked-questions-faq)
- [Additional Resources](#additional-resources)

---

## 🎯 Overview

This guide provides a complete walkthrough to create an **Azure Key Vault with public network access disabled**. Secrets can only be accessed and updated from a private Admin VM within an Azure Virtual Network.

### Key Features
✅ **Zero Public Access** - Key Vault unreachable from internet  
✅ **Private Endpoint** - Secure in-VNet access only  
✅ **Managed Identity** - No credentials stored on VM  
✅ **Azure Bastion** - Secure browser-based VM access  
✅ **RBAC Permissions** - Principle of least privilege  
✅ **Enterprise-Grade Security** - Production-ready setup  

---

## 🏗️ Architecture

```
┌────────────────────────────────────────────────────────────────┐
│  Internet (Public Access BLOCKED ❌)                           │
└────────────────────────────────────────────────────────────────┘
                           │
                           ✗ Cannot reach Key Vault
                           │
┌──────────────────────────┼─────────────────────────────────────┐
│ Azure Virtual Network (10.10.0.0/16)                           │
│                          │                                     │
│  ┌───────────────────────┴──────────────────┐                  │
│  │ admin-vm-subnet (10.10.1.0/24)           │                  │
│  │                                          │                  │
│  │   ┌────────────────────────────┐         │                  │
│  │   │  kv-admin-vm               │         │                  │
│  │   │  - No Public IP            │◄────────┼────────┐         │
│  │   │  - Managed Identity        │         │        │         │
│  │   │  - Azure CLI               │         │        │         │
│  │   └────────────┬───────────────┘         │        │         │
│  └────────────────┼────────────────────────┬┘        │         │
│                   │                        │         │         │
│                   │ Private Connection     │         │         │
│                   ↓                        │    ┌────┴─────┐   │
│  ┌────────────────────────────────────────┼─┐   │ Bastion  │   │
│  │ kv-private-endpoint-subnet             │ │   │ Subnet   │   │
│  │ (10.10.2.0/24)                         │ │   │ (10.10.  │   │
│  │                                        │ │   │  3.0/26) │   │
│  │   ┌────────────────────────────┐      │ │   └──────────┘    │
│  │   │  Private Endpoint          │      │ │         ▲         │
│  │   │  10.10.2.x                 │      │ │         │         │
│  │   └────────────┬───────────────┘      │ │         │         │
│  └────────────────┼────────────────────────┘         │         │
│                   │                          Browser │         │
│                   ↓                          Access  │         │
│         ┌──────────────────────┐                     │         │
│         │  Azure Key Vault     │◄────────────────────┘         │
│         │  (Private Only)      │                               │
│         │  ✅ Private Endpoint│                                │
│         │  ❌ Public Disabled │                                │
│         └──────────────────────┘                               │
└────────────────────────────────────────────────────────────────┘
```

---

## 📦 Prerequisites

### Required
- **Azure Subscription** with active access
- **Owner or Contributor** role on subscription
- **Azure Portal Access**: https://portal.azure.com
- **Modern Web Browser**: Chrome, Edge, or Firefox

### Knowledge Requirements
- Basic understanding of Azure resources
- Familiarity with virtual networks
- Basic command-line experience

---

## 💰 Cost Estimate

### Monthly Costs (Approximate USD)

| Resource | Configuration | Monthly Cost |
|----------|--------------|--------------|
| Azure Key Vault | Standard tier | ~$0.50 |
| Admin VM | Standard_B2s (2 vCPU, 4GB RAM) | ~$30 |
| Azure Bastion (Optional) | Basic tier | ~$140 |
| Public IP (If no Bastion) | Standard | ~$5 |
| Private Endpoint | 1 endpoint | ~$7 |
| Private DNS Zone | 1 zone | ~$0.50 |
| Virtual Network | - | Free |
| **TOTAL (with Bastion)** | | **~$178/month** |
| **TOTAL (without Bastion)** | | **~$43/month** ✅ |

### 💡 Cost Saving Tips
- **Skip Bastion for testing**: Saves ~$135/month (use direct RDP/SSH instead)
- **Stop VM when not in use**: Saves ~$30/month
- **Delete Bastion when not needed**: Saves ~$140/month (can recreate when needed)
- **Use shared VM**: Multiple users can use same VM

### 💰 Two Deployment Options

**Option A: Full Production Setup** (~$178/month)
- Includes Azure Bastion for maximum security
- No public IPs on VMs
- Best for production environments

**Option B: Testing/Development Setup** (~$43/month) ✅ **Recommended for Learning**
- Skip Bastion, use direct RDP/SSH
- Adds public IP to VM temporarily
- Perfect for testing and learning

---

## 🚀 Setup Steps

### STEP 1: Create Resource Group

A Resource Group is a logical container for all Azure resources.

1. Open **Azure Portal** → **Resource groups** → **+ Create**
2. **Resource Group Name**: `rg-kv-private-demo`
3. **Region**: Choose your preferred region (example: **East US** or **Central India**)
   - ⚠️ **Important**: Use the SAME region for ALL resources
4. Click **Review + Create** → **Create**
5. Wait for deployment (~10 seconds)

✅ **Verification**: Resource group appears in the list

---

### STEP 2: Create Virtual Network and Subnets

The Virtual Network provides private network isolation for your resources.

#### 2.1 Create Virtual Network

1. Click **Create a resource** → Search **"Virtual Network"** → **Create**
2. **Basics Tab**:
   - **Subscription**: Your subscription
   - **Resource Group**: `rg-kv-private-demo`
   - **Virtual Network Name**: `demokeyvaultVNet`
   - **Region**: Same as resource group
3. Click **Next: IP Addresses**

#### 2.2 Configure IP Address Space

1. **IPv4 address space**: `10.10.0.0/16`
   - This provides 65,536 IP addresses
   - 💡 **Alternative**: You can also use `10.0.0.0/16`, `172.16.0.0/16`, or `192.168.0.0/16`
   - If using `10.0.0.0/16`, adjust subnet addresses to `10.0.1.0`, `10.0.2.0`, `10.0.3.0` accordingly
2. **Delete** the default subnet if it exists (usually named "default")

**📌 Important Notes About Subnets**:
- **Subnet purpose**: Always select **"Default"** for all subnets in this guide
- **Subnet delegation**: Not required - leave as "None"
- **Service endpoints**: Optional - can leave empty for this setup
- The Azure Portal will auto-calculate the subnet address range based on your starting address and size

#### 2.3 Create Subnet 1: Admin VM Subnet

1. Click **+ Add subnet**
2. Fill in the following fields:

| Field | Value to Enter |
|-------|----------------|
| **Subnet purpose** | **Default** (keep as is) |
| **Name** | `admin-vm-subnet` |
| **Starting address** | `10.10.1.0` |
| **Size** | `/24 (256 addresses)` |
| **Subnet address range** | Auto-populates: `10.10.1.0 - 10.10.1.255` |
| **IPv6** | Leave unchecked |

3. Scroll down - you can leave **Private subnet** settings as default
4. Click **Add**

✅ **Verification**: Subnet appears in the list with address range 10.10.1.0/24

#### 2.4 Create Subnet 2: Private Endpoint Subnet

1. Click **+ Add subnet**
2. Fill in the following fields:

| Field | Value to Enter |
|-------|----------------|
| **Subnet purpose** | **Default** (keep as is) |
| **Name** | `kv-private-endpoint-subnet` |
| **Starting address** | `10.10.2.0` |
| **Size** | `/24 (256 addresses)` |
| **Subnet address range** | Auto-populates: `10.10.2.0 - 10.10.2.255` |
| **IPv6** | Leave unchecked |

3. Leave **Private subnet** settings as default
4. Click **Add**

✅ **Verification**: Subnet appears in the list with address range 10.10.2.0/24

#### 2.5 Create Subnet 3: Azure Bastion Subnet

1. Click **+ Add subnet**
2. Fill in the following fields:

| Field | Value to Enter |
|-------|----------------|
| **Subnet purpose** | **Default** (keep as is) |
| **Name** | `AzureBastionSubnet` ⚠️ **EXACT NAME - Case Sensitive!** |
| **Starting address** | `10.10.3.0` |
| **Size** | `/26 (64 addresses)` ⚠️ **Must be /26 or larger** |
| **Subnet address range** | Auto-populates: `10.10.3.0 - 10.10.3.63` |
| **IPv6** | Leave unchecked |

3. Leave **Private subnet** settings as default
4. Click **Add**

✅ **Verification**: Subnet appears with address range 10.10.3.0/26

**⚠️ Important Notes**:
- Bastion subnet name **must** be exactly `AzureBastionSubnet` (case-sensitive)
- Bastion subnet **must** be at least /26 (64 IP addresses)
- All other subnets use **"Default"** for subnet purpose (no delegation needed)

#### 2.6 Finalize VNet Creation

1. Click **Review + Create**
2. Click **Create**
3. Wait for deployment (~30 seconds)

✅ **Verification**: Go to Virtual Network → Subnets → Should see 3 subnets

#### 2.7 Final Subnet Configuration Summary

After completing all steps, your VNet should have this configuration:

| Subnet Name | Address Range | Size | Purpose |
|-------------|---------------|------|---------|
| admin-vm-subnet | 10.10.1.0/24 | 256 IPs | Admin VM hosting |
| kv-private-endpoint-subnet | 10.10.2.0/24 | 256 IPs | Private endpoint for Key Vault |
| AzureBastionSubnet | 10.10.3.0/26 | 64 IPs | Azure Bastion service |

**Visual Layout**:
```
VNet: demokeyvaultVNet (10.10.0.0/16)
├── admin-vm-subnet            → 10.10.1.0/24  (256 IPs)
├── kv-private-endpoint-subnet → 10.10.2.0/24  (256 IPs)
└── AzureBastionSubnet         → 10.10.3.0/26  (64 IPs)
```

---

### STEP 3: Create Azure Key Vault

This Key Vault will store secrets securely with public access completely disabled.

1. Click **Create a resource** → Search **"Key Vault"** → **Create**

#### 3.1 Basics Tab
- **Subscription**: Your subscription
- **Resource Group**: `rg-kv-private-demo`
- **Key Vault Name**: `demokeyvault123` (must be globally unique)
  - 💡 **Tip**: Add random numbers if the name is taken (e.g., `demokeyvault789`)
- **Region**: Same as VNet
- **Pricing Tier**: **Standard**
- **Days to retain deleted vaults**: 90 (default)

#### 3.2 Access Configuration Tab
- **Permission model**: Select **Azure role-based access control (RBAC)**
  - ⚠️ **Important**: NOT "Vault access policy"

#### 3.3 Networking Tab
- **Connectivity method**: Select **Disable public access**
  - ⚠️ This makes Key Vault unreachable from the internet
  - ⚠️ You won't be able to access it from Azure Portal until the Private Endpoint is configured

#### 3.4 Review and Create
1. Click **Review + Create**
2. Click **Create**
3. Wait for deployment (~1 minute)

✅ **Verification**: Key Vault is created successfully

#### 3.5 Understanding the "Access Denied" Message (This is Normal!)

After creating the Key Vault, if you try to view **Keys**, **Secrets**, or **Certificates**:

**You will see**:
- 🟠 Orange warning: *"Public network access is disabled and request is not from a trusted service nor via an approved private link."*
- ❌ Message: *"You are unauthorized to view these contents."*

**This is EXPECTED and CORRECT! ✅**

This proves your security setup is working:
- ✅ Public access is **successfully blocked**
- ✅ Azure Portal (internet) **cannot access** the Key Vault
- ✅ Access is **only possible** from inside the VNet via Private Endpoint

**Don't worry!** After completing Steps 4-7 (Private Endpoint + VM setup), you'll be able to access secrets from the Admin VM.

**Screenshot Example**:
```
┌──────────────────────────────────────────────────────────────┐
│ demokeyvault123 | Keys                                       │
├──────────────────────────────────────────────────────────────┤
│ 🟠 Public network access is disabled and request is not     │
│    from a trusted service nor via an approved private link.  │
│                                                              │
│ Name              Status              Expiration date        │
│ You are unauthorized to view these contents.                │
└──────────────────────────────────────────────────────────────┘
```

**Important**: Do NOT try to fix this by enabling public access! This security restriction is intentional and exactly what we want.

#### 3.6 Common Question: Can I Access Key Vault via Azure Portal from Inside the VM?

**Question**: "If I open https://portal.azure.com from inside the VM, can I access the Key Vault secrets?"

**Answer**: ❌ **No, you still cannot access Key Vault through the Azure Portal, even from inside the VM.**

**Why?**

| Access Method | Where Requests Originate | Result |
|--------------|-------------------------|--------|
| 🌐 **Azure Portal** (from anywhere) | Microsoft's portal servers (public internet) | ❌ Blocked |
| 💻 **Azure CLI in VM** | VM itself (via private endpoint) | ✅ Allowed |
| ⚡ **PowerShell in VM** | VM itself (via private endpoint) | ✅ Allowed |
| 🔧 **REST API in VM** | VM itself (via private endpoint) | ✅ Allowed |

**Explanation**:
- When you use the **Azure Portal web interface**, Microsoft's portal servers make API calls **on your behalf**
- These portal servers are on the **public internet**, not in your VNet
- Your Key Vault rejects requests from the public internet
- **It doesn't matter where you open the browser** - the requests still come from Microsoft's servers

**Visual Flow**:
```
❌ Browser in VM → portal.azure.com → Microsoft Portal Servers → Key Vault
                                      (Public Internet)         BLOCKED

✅ Azure CLI in VM → Private Endpoint → Key Vault
                     (Private Network)   ALLOWED
```

**Bottom Line**: 
- ✅ **Do use**: Azure CLI, PowerShell, or SDKs inside the VM
- ❌ **Cannot use**: Azure Portal web interface (from anywhere)

---

### STEP 4: Create Private Endpoint for Key Vault

A Private Endpoint assigns a private IP address to the Key Vault inside your VNet, enabling secure access.

#### 4.1 Navigate to Private Endpoint Settings
1. Go to **Key Vault** → `demokeyvault123` (your Key Vault name)
2. Left menu → **Networking**
3. Tab → **Private endpoint connections**
4. Click **+ Create** or **+ Private endpoint**

#### 4.2 Basics Tab
- **Subscription**: Your subscription
- **Resource Group**: `rg-kv-private-demo`
- **Name**: `kv-private-endpoint`
- **Network Interface Name**: Leave default or use `kv-pe-nic`
- **Region**: Same as VNet

#### 4.3 Resource Tab
- **Connection method**: Connect to an Azure resource in my directory
- **Subscription**: Your subscription
- **Resource type**: `Microsoft.KeyVault/vaults`
- **Resource**: `demokeyvault123` (your Key Vault)
- **Target sub-resource**: **vault** (should be auto-selected)

#### 4.4 Virtual Network Tab
- **Virtual Network**: `demokeyvaultVNet`
- **Subnet**: `kv-private-endpoint-subnet`
- **Private IP configuration**: **Dynamically allocate IP address**
- **Application security group**: None

#### 4.5 DNS Tab
- **Integrate with private DNS zone**: **Yes** (checked)
- **Subscription**: Your subscription
- **Resource Group**: `rg-kv-private-demo`
- **Private DNS Zone**: 
  - Option will show: `privatelink.vaultcore.azure.net`
  - If "Create new" appears, the zone will be created automatically
  - Click **OK** if prompted

#### 4.6 Review and Create
1. Click **Review + Create**
2. Click **Create**
3. Wait for deployment (~2-3 minutes)

✅ **Verification**: 
   - Go to Private Endpoint → **DNS Configuration**
   - Should show a private IP address (e.g., 10.10.2.4 or 10.10.2.5)

---

### STEP 5: Create Admin VM (Jump Host)

This VM acts as the secure entry point for managing Key Vault secrets.

1. Click **Create a resource** → **Virtual Machine** → **Create**

#### 5.1 Basics Tab
- **Subscription**: Your subscription
- **Resource Group**: `rg-kv-private-demo`
- **Virtual Machine Name**: `kv-admin-vm`
- **Region**: Same as VNet
- **Availability options**: No infrastructure redundancy required
- **Security type**: Standard
- **Image**: 
  - **Ubuntu Server 22.04 LTS - x64 Gen2** (recommended)
  - Alternative: Windows Server 2022 Datacenter (if you prefer GUI)
- **Size**: **Standard_B2s** (2 vCPUs, 4 GB RAM)
  - Click "See all sizes" if not visible

#### 5.2 Administrator Account
**For Ubuntu**:
- **Authentication type**: **Password**
- **Username**: `azureadmin`
- **Password**: Create a strong password (e.g., `SecureP@ssw0rd123!`)
- **Confirm password**: Re-enter the password

**For Windows**:
- **Username**: `azureadmin`
- **Password**: Create a strong password
- **Confirm password**: Re-enter the password

#### 5.3 Inbound Port Rules
- **Public inbound ports**: **None** ⚠️ **CRITICAL**
  - We'll use Azure Bastion for secure access

#### 5.4 Disks Tab
- **OS disk type**: **Standard SSD** (default is fine)
- No other changes needed

#### 5.5 Networking Tab
- **Virtual Network**: `demokeyvaultVNet`
- **Subnet**: `admin-vm-subnet`
- **Public IP**: **None** ⚠️ **CRITICAL**
  - If it shows "(new)", change the dropdown to **None**
- **NIC network security group**: Basic
- **Public inbound ports**: None
- **Delete NIC when VM is deleted**: Checked (optional, for easier cleanup)

#### 5.6 Management Tab
- Leave all defaults
- **Boot diagnostics**: Enable with managed storage account (default)

#### 5.7 Review and Create
1. Click **Review + Create**
2. Review the configuration and costs
3. Click **Create**
4. Wait for deployment (~3-5 minutes)

✅ **Verification**: VM shows "Running" status, and has NO public IP address assigned

---

### STEP 6: Enable Managed Identity on VM

Managed Identity allows the VM to authenticate to Azure services without storing passwords or keys.

1. Go to **Virtual Machines** → `kv-admin-vm`
2. Left menu → **Identity**
3. **System assigned** tab
4. **Status**: Toggle to **On**
5. Click **Save**
6. Click **Yes** to confirm
7. Wait ~10 seconds for the operation to complete

✅ **Verification**: Status shows "On" with an **Object (principal) ID** displayed

📝 **Note**: You can copy the Object (principal) ID for reference, but it's not required for the next steps.

---

### STEP 7: Grant VM Access to Key Vault

This step grants the VM permission to read, write, and manage secrets in the Key Vault.

#### 7.1 Navigate to Key Vault IAM
1. Go to **Key Vault** → `demokeyvault123` (your Key Vault)
2. Left menu → **Access control (IAM)**
3. Click **+ Add** → **Add role assignment**

#### 7.2 Select Role
1. **Role** tab
2. In the search box, type: `Key Vault Secrets Officer`
3. Select **Key Vault Secrets Officer**
   - This role allows: Create, read, update, and delete secrets
4. Click **Next**

#### 7.3 Assign to Managed Identity
1. **Members** tab
2. **Assign access to**: **Managed identity**
3. Click **+ Select members**
4. **Managed identity** dropdown: Select **Virtual Machine**
5. In the list, search and select: `kv-admin-vm`
6. Click **Select**
7. Click **Next**

#### 7.4 Review and Assign
1. **Review + assign** tab
2. Review the configuration
3. Click **Review + assign**
4. Wait ~30 seconds for the assignment to complete

✅ **Verification**: 
   - Go to **Access control (IAM)** → **Role assignments** tab
   - Search for "Key Vault Secrets Officer"
   - You should see `kv-admin-vm` listed

**🎯 Next: Choose Your Connection Method**
- **Testing/Development**: Jump to Step 8B (Direct RDP/SSH) ✅ Faster & Cheaper
- **Production**: Continue to Step 8A (Azure Bastion) 🔒 More Secure

---

### STEP 8: Connect to Admin VM

You have **two options** to connect to your Admin VM:

| Feature | Option A: Azure Bastion | Option B: Direct RDP/SSH |
|---------|------------------------|--------------------------|
| **Cost** | ~$140/month | Free (Public IP: ~$5/month) |
| **Public IP Required** | ❌ No | ✅ Yes |
| **Security** | 🔒 High (no internet exposure) | ⚠️ Medium (port exposed) |
| **Setup Time** | 5-10 minutes | 2-3 minutes |
| **Best For** | Production | Testing/Development |
| **Connection Method** | Browser-based | RDP/SSH client |

**👉 Recommendation**: 
- **Testing**: Use Option B (Direct RDP/SSH) to save $140/month ✅
- **Production**: Use Option A (Azure Bastion) for better security

---

### STEP 8A: Create Azure Bastion (Production Method)

⚠️ **Skip this if you're testing** - Jump to Step 8B for direct RDP access instead.

Azure Bastion provides secure, browser-based RDP/SSH access to VMs without exposing them to the internet.

1. Click **Create a resource** → Search **"Bastion"** → **Create**

#### 8.1 Basics Tab
- **Subscription**: Your subscription
- **Resource Group**: `rg-kv-private-demo`
- **Name**: `demo-bastion`
- **Region**: Same as VNet
- **Tier**: **Basic** (~$140/month)
  - Standard tier offers more features but costs more
- **Instance count**: 2 (default)

#### 8.2 Virtual Networks Tab
- **Virtual Network**: `demokeyvaultVNet`
- **Subnet**: `AzureBastionSubnet` (should be auto-selected)
- **Public IP address**: **Create new**
  - **Public IP address name**: `bastion-pip`
  - **Public IP address SKU**: Standard (required for Bastion)
  - **Assignment**: Static

#### 8.3 Review and Create
1. Click **Review + Create**
2. Review the configuration
3. Click **Create**
4. ⏳ **Wait 5-10 minutes** (Bastion deployment takes longer than other resources)

✅ **Verification**: Bastion resource shows "Running" or "Succeeded" status

After Bastion is created, proceed to **Step 9A** below.

---

### STEP 8B: Direct RDP/SSH Access (Testing Method - No Bastion Required)

💡 **Use this method for testing to save ~$140/month on Bastion costs.**

⚠️ **Security Note**: This method adds a public IP and opens RDP/SSH port. Use only for testing. Remove public access for production.

#### 8B.1 Add Public IP to VM

1. Go to **Virtual Machines** → `kv-admin-vm` → **Stop** the VM first
2. Wait for VM to stop completely
3. Go to **Networking** → **Network settings**
4. Click on the **Network Interface** name (e.g., `kv-admin-vm123`)
5. Left menu → **IP configurations**
6. Click on **ipconfig1** (or similar)
7. **Public IP address**: Change from "Disabled" to **Associate**
8. Click **Create new** for Public IP
   - **Name**: `kv-admin-vm-pip`
   - **SKU**: Standard or Basic
   - Click **OK**
9. Click **Save**
10. Go back to VM → Click **Start**

✅ **Verification**: VM Overview page shows a Public IP address

#### 8B.2 Add Inbound Security Rule for RDP/SSH

1. Go to **Virtual Machines** → `kv-admin-vm`
2. Left menu → **Networking** → **Network settings**
3. Click **Network security group** name (e.g., `kv-admin-vm-nsg`)
4. Left menu → **Inbound security rules**
5. Click **+ Add**

**For Windows VM (RDP)**:
- **Source**: My IP address (or IP addresses for testing)
- **Source port ranges**: *
- **Destination**: Any
- **Service**: **RDP**
- **Destination port ranges**: 3389 (auto-fills)
- **Protocol**: TCP
- **Action**: Allow
- **Priority**: 1000 (or any available number)
- **Name**: `Allow-RDP-Testing`
- Click **Add**

**For Ubuntu VM (SSH)**:
- **Source**: My IP address
- **Source port ranges**: *
- **Destination**: Any
- **Service**: **SSH**
- **Destination port ranges**: 22 (auto-fills)
- **Protocol**: TCP
- **Action**: Allow
- **Priority**: 1000
- **Name**: `Allow-SSH-Testing`
- Click **Add**

✅ **Verification**: New inbound rule appears with Priority 1000 and Action "Allow"

#### 8B.3 Connect to VM

**For Windows VM (RDP)**:
1. Go to VM → Overview → Copy the **Public IP address**
2. On your local machine, open **Remote Desktop Connection**
   - Windows: Press `Win + R`, type `mstsc`, press Enter
   - Mac: Download Microsoft Remote Desktop from App Store
3. Enter the **Public IP address**
4. Click **Connect**
5. Enter credentials:
   - Username: `azureadmin`
   - Password: Your VM password
6. Accept the certificate warning
7. You're now connected to Windows Server!

**For Ubuntu VM (SSH)**:
```bash
# From your local terminal
ssh azureadmin@<PUBLIC-IP-ADDRESS>

# Example:
# ssh azureadmin@20.123.45.67

# Enter your VM password when prompted
```

✅ **You are now connected to the VM!** Proceed to Step 10 (Install Azure CLI).

**🧹 Security Reminder**: After testing, you should:
- Remove the inbound security rule (Allow-RDP-Testing or Allow-SSH-Testing)
- Disassociate and delete the public IP address
- Or delete the entire VM and recreate without public IP for production

---

### STEP 9A: Connect to Admin VM via Azure Bastion (If Using Bastion)

⚠️ **Skip this if you used Step 8B (Direct RDP/SSH)** - You're already connected!

Now you can securely connect to the VM through your web browser.

#### 9.1 Initiate Bastion Connection
1. Go to **Virtual Machines** → `kv-admin-vm`
2. Click **Connect** button (top menu)
3. From the dropdown, select **Connect via Bastion** (not "Bastion" directly, use the dropdown)
4. Alternatively, click the **Bastion** tab from the available connection methods

#### 9A.2 Enter Credentials
- **Authentication Type**: **Password**
- **Username**: `azureadmin` (the username you created)
- **Password**: Your VM password
- Click **Connect**

#### 9A.3 Browser Opens VM Session
- **For Ubuntu**: A terminal window opens in a new browser tab
- **For Windows**: An RDP session opens in a new browser tab

✅ **You are now securely connected to the VM!**

---

### STEP 10: Install Azure CLI in VM (Ubuntu Only)

If you chose Ubuntu, you need to install Azure CLI. For Windows VMs, Azure CLI is typically pre-installed.

**Inside your VM session** (either Bastion, RDP, or SSH), run the following commands:

```bash
# Update package list
sudo apt-get update

# Install curl if not present
sudo apt-get install -y curl

# Install Azure CLI
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Verify installation
az --version
```

**Expected output**: Azure CLI version information (e.g., azure-cli 2.xx.x)

**For Windows VM**: 
- Open PowerShell inside the Bastion RDP session
- Test if Azure CLI is installed: `az --version`
- If not installed, download from: https://aka.ms/installazurecliwindows

---

### STEP 11: Login with Managed Identity

Inside the VM terminal (Ubuntu) or PowerShell (Windows):

```bash
# Login using Managed Identity (no password needed!)
az login --identity
```

**Expected output**:
```json
[
  {
    "environmentName": "AzureCloud",
    "id": "xxxx-subscription-id-xxxx",
    "name": "Your Subscription Name",
    "state": "Enabled",
    "user": {
      "assignedIdentity": {
        "systemAssigned": true
      },
      "type": "servicePrincipal"
    }
  }
]
```

```bash
# Verify you're logged in
az account show
```

✅ **Success**: You're now authenticated using Managed Identity with no credentials stored!

---

### STEP 12: Create and Manage Secrets

Now you can create, read, update, and delete secrets in the Key Vault.

```bash
# Set your Key Vault name as a variable (replace with your actual name)
KV_NAME="demokeyvault123"

# Create your first secret
az keyvault secret set \
  --vault-name $KV_NAME \
  --name "DemoSecret" \
  --value "MySecretValue"
```

**Expected output**: JSON showing the secret details, including:
- `id`: The secret identifier
- `value`: Your secret value
- `attributes`: Enabled status and timestamps

```bash
# Create more secrets
az keyvault secret set \
  --vault-name $KV_NAME \
  --name "DatabasePassword" \
  --value "P@ssw0rd123"

az keyvault secret set \
  --vault-name $KV_NAME \
  --name "ApiKey" \
  --value "demo-api-key-abc123"

az keyvault secret set \
  --vault-name $KV_NAME \
  --name "ConnectionString" \
  --value "Server=myserver;Database=mydb;User=admin;Password=pass123"
```

✅ **Success**: You've created multiple secrets in the Key Vault!

---

## 🧪 Testing Key Vault Access

### Test 1: List All Secrets

```bash
# List all secrets in table format
az keyvault secret list \
  --vault-name $KV_NAME \
  --output table
```

**Expected output**: A table showing all your secrets with their IDs and status.

### Test 2: Retrieve a Secret Value

```bash
# Get specific secret value
az keyvault secret show \
  --vault-name $KV_NAME \
  --name "DemoSecret" \
  --query value \
  --output tsv
```

**Expected output**: `MySecretValue`

### Test 3: Update an Existing Secret

```bash
# Update a secret with a new value
az keyvault secret set \
  --vault-name $KV_NAME \
  --name "DemoSecret" \
  --value "UpdatedSecretValue"

# Verify the update
az keyvault secret show \
  --vault-name $KV_NAME \
  --name "DemoSecret" \
  --query value \
  --output tsv
```

**Expected output**: `UpdatedSecretValue`

### Test 4: Delete a Secret

```bash
# Delete a secret
az keyvault secret delete \
  --vault-name $KV_NAME \
  --name "ApiKey"

# List secrets to verify deletion
az keyvault secret list \
  --vault-name $KV_NAME \
  --output table
```

**Expected result**: The "ApiKey" secret is no longer in the list.

### Test 5: Verify Public Access is Blocked

This test confirms the security is working by attempting to access from the public internet (Azure Portal).

**From your local machine** (NOT inside the VM):

1. Open **Azure Portal** → **Key Vault** → `demokeyvault123`
2. Click on **Keys**, **Secrets**, or **Certificates** from the left menu

**What you SHOULD see** (this is CORRECT ✅):
- 🟠 **Orange warning banner**: 
  ```
  Public network access is disabled and request is not from 
  a trusted service nor via an approved private link.
  ```
- ❌ **Error message**: 
  ```
  You are unauthorized to view these contents.
  ```

**What this means**:
- ✅ **Security is working perfectly!**
- ✅ Public internet access is **blocked**
- ✅ Azure Portal **cannot access** the Key Vault
- ✅ Only the Admin VM can access via Private Endpoint

**Comparison**:

| Access Method | Result | Status |
|--------------|--------|--------|
| 🌐 **Azure Portal (Internet)** | ❌ Access Denied | ✅ Correct - blocked |
| 💻 **Admin VM (Private Endpoint)** | ✅ Full Access | ✅ Correct - allowed |

**Important**: 
- Do NOT try to "fix" this error by enabling public access
- This security restriction is **intentional and required**
- Access should ONLY work from inside the Admin VM using CLI/PowerShell/API

### Test 6: Understanding Portal Access from Inside VM

**Common Question**: "What if I open portal.azure.com from INSIDE the VM?"

**Answer**: ❌ You will still see "Access Denied" - and that's correct!

**Test it yourself**:
1. Inside your VM (RDP or SSH session)
2. Open a web browser (if Windows VM) or use curl
3. Navigate to https://portal.azure.com
4. Try to view Key Vault secrets

**Result**: ❌ Still shows "unauthorized to view these contents"

**Why?**
```
Your browser in VM → portal.azure.com → Microsoft's portal servers → Key Vault
                                        (Public Internet)            ❌ BLOCKED

The requests come from Microsoft's portal backend servers,
not directly from your VM!
```

**The ONLY way to access Key Vault**:
```bash
# ✅ This works - Direct from VM
az keyvault secret show --vault-name demokeyvault123 --name "DemoSecret"

# ❌ This doesn't work - Via portal UI (even from inside VM)
Opening portal.azure.com and clicking on secrets
```

**Key Takeaway**: 
- Azure Portal UI = Microsoft's servers making requests = ❌ Blocked
- Azure CLI/PowerShell = Your VM making requests = ✅ Allowed

---

## 🔧 Troubleshooting

### Issue 0: "You are unauthorized to view these contents" in Azure Portal

**Symptoms**: When viewing Key Vault in Azure Portal, you see:
- 🟠 Orange warning: "Public network access is disabled and request is not from a trusted service..."
- ❌ "You are unauthorized to view these contents"

**This is NOT an error - this is CORRECT behavior! ✅**

**Why this happens**:
- Your Key Vault has public access disabled (Step 3)
- Azure Portal is accessed from the public internet
- The Key Vault correctly rejects the connection

**Solution**: **Do nothing!** This is expected and proves your security is working.

**To access Key Vault**:
1. ✅ Connect to Admin VM via Bastion (Step 9)
2. ✅ Use Azure CLI inside the VM (Steps 10-12)
3. ✅ Access secrets through private endpoint

**Do NOT**:
- ❌ Enable public network access on Key Vault
- ❌ Change firewall settings
- ❌ Remove private endpoint

**Visual Guide**:
```
❌ Azure Portal (Internet) → Key Vault = BLOCKED (correct!)
✅ Admin VM (Private Network) → Key Vault = ALLOWED (correct!)
```

---

### Issue 1: Cannot Access Key Vault from VM

**Symptoms**: Running `az keyvault secret list` returns "Forbidden" or access denied error

**Solutions**:

```bash
# Step 1: Verify Managed Identity is enabled
az vm show \
  --name kv-admin-vm \
  --resource-group rg-kv-private-demo \
  --query identity
```

Should show: `"type": "SystemAssigned"`

```bash
# Step 2: Check if you're logged in with Managed Identity
az account show
```

Should show: `"type": "servicePrincipal"`

```bash
# Step 3: Verify DNS resolution (inside VM)
nslookup demokeyvault123.vault.azure.net
```

Should resolve to **10.10.2.x** (private IP), NOT a public IP address.

**If resolving to public IP**:
- Wait 5-10 minutes for DNS propagation
- Restart the VM
- Check that Private DNS Zone is linked to VNet

```bash
# Step 4: Wait for RBAC permissions to propagate
# RBAC assignments can take 2-5 minutes to take effect
# Try the command again after waiting
```

---

### Issue 2: Cannot Connect to VM via Bastion

**Symptoms**: Bastion connection fails, times out, or button is grayed out

**Solutions**:

1. **Check Bastion Status**:
   - Go to Azure Portal → Bastion resource
   - Ensure status is "Succeeded" or "Running"
   - If still deploying, wait for completion

2. **Check VM Status**:
   - Go to Virtual Machines → `kv-admin-vm`
   - Ensure status is "Running"
   - If stopped, click "Start"

3. **Check Subnet Configuration**:
   - Go to Virtual Network → Subnets
   - Verify `AzureBastionSubnet` exists with /26 or larger prefix
   - Verify subnet name is EXACTLY `AzureBastionSubnet`

4. **Browser Issues**:
   - Allow pop-ups from Azure Portal
   - Try a different browser (Chrome or Edge recommended)
   - Clear browser cache

5. **Refresh Connection**:
   - Go to VM → Overview
   - Click **Connect** → **Bastion**
   - Re-enter credentials

---

### Issue 3: Key Vault Name Already Exists

**Symptoms**: "The vault name is already in use" or similar error during Key Vault creation

**Solution**: Key Vault names must be globally unique across all Azure tenants.

Try these naming patterns:
- `demokeyvault<random-number>` (e.g., `demokeyvault7392`)
- `kv-<company>-<env>-<number>` (e.g., `kv-contoso-dev-001`)
- `kv-<your-initials>-<date>` (e.g., `kv-jd-20260127`)

---

### Issue 4: DNS Not Resolving to Private IP

**Symptoms**: Inside VM, Key Vault resolves to public IP instead of 10.10.2.x

**Solutions**:

```bash
# Inside VM, test DNS resolution
nslookup demokeyvault123.vault.azure.net
```

**If showing public IP**:

1. **Wait for DNS propagation** (can take 5-10 minutes)

2. **Check Private DNS Zone**:
   - Go to Azure Portal → Private DNS zones
   - Verify `privatelink.vaultcore.azure.net` exists
   - Check Virtual network links → Should show `demokeyvaultVNet` as "Completed"

3. **Check DNS Record**:
   - Go to Private DNS zone → Record sets
   - Verify an A record exists for your Key Vault name
   - Should point to private IP (e.g., 10.10.2.4)

4. **Restart VM**:
   ```bash
   # From Azure Portal
   # VM → Restart
   ```

5. **Flush DNS cache inside VM** (Ubuntu):
   ```bash
   sudo systemd-resolve --flush-caches
   sudo systemctl restart systemd-resolved
   ```

---

### Issue 5: Managed Identity Login Fails

**Symptoms**: `az login --identity` returns error or timeout

**Solutions**:

1. **Wait for Identity Propagation**:
   - Wait 2-3 minutes after enabling Managed Identity
   - Try login command again

2. **Verify Instance Metadata Service** (inside VM):
   ```bash
   curl -H Metadata:true "http://169.254.169.254/metadata/identity/info?api-version=2021-02-01"
   ```
   
   Should return identity information. If timeout or error:
   - Check VM has network connectivity
   - Verify no custom firewall rules blocking 169.254.169.254

3. **Try login with explicit parameters**:
   ```bash
   az login --identity --allow-no-subscriptions
   ```

4. **Restart the VM**:
   - From Azure Portal: VM → Restart
   - Wait for VM to come back online
   - Try login again

---

### Issue 6: "Permission Denied" When Creating Secrets

**Symptoms**: "The user, group or application does not have secrets set permission"

**Solutions**:

1. **Verify RBAC Role Assignment**:
   - Go to Key Vault → Access control (IAM)
   - Click "Role assignments"
   - Search for `kv-admin-vm`
   - Verify "Key Vault Secrets Officer" role is assigned

2. **Wait for Permission Propagation**:
   - RBAC changes can take 2-5 minutes to take effect
   - Wait and retry

3. **Check Permission Model**:
   - Go to Key Vault → Access policies (or Access configuration)
   - Verify it shows "Azure role-based access control"
   - If it shows "Vault access policy", you need to recreate the Key Vault with RBAC enabled

4. **Re-assign the Role**:
   - Go to Key Vault → Access control (IAM)
   - Remove existing role assignment
   - Wait 1 minute
   - Add role assignment again (follow Step 7)

---

## 🧹 Cleanup

### Option 1: Delete Everything (Recommended)

This deletes the entire resource group and all resources inside it.

**Azure Portal**:
1. Go to **Resource Groups**
2. Select `rg-kv-private-demo`
3. Click **Delete resource group**
4. Type the resource group name to confirm: `rg-kv-private-demo`
5. Click **Delete**
6. Wait 5-10 minutes for deletion to complete

**Azure CLI**:
```bash
# Delete resource group and all resources
az group delete \
  --name rg-kv-private-demo \
  --yes \
  --no-wait

# Check deletion status (optional)
az group show --name rg-kv-private-demo
# Expected after a few minutes: ResourceGroupNotFound
```

---

### Option 2: Stop VM to Save Costs (Keep Setup)

If you want to keep the setup but save costs when not using it:

**Stop (Deallocate) the VM**:
```bash
# Via Azure Portal
# Go to VM → Stop

# Or via Azure CLI
az vm deallocate \
  --name kv-admin-vm \
  --resource-group rg-kv-private-demo

# This saves ~$30/month (VM costs)
```

**Start VM when needed**:
```bash
# Via Azure Portal
# Go to VM → Start

# Or via Azure CLI
az vm start \
  --name kv-admin-vm \
  --resource-group rg-kv-private-demo
```

---

### Option 3: Delete Only Bastion (Major Cost Savings)

Bastion is the most expensive component (~$140/month). You can delete it when not needed and recreate later.

**Delete Bastion**:
```bash
# Via Azure Portal
# Go to Bastion resource → Delete

# Or via Azure CLI
az network bastion delete \
  --name demo-bastion \
  --resource-group rg-kv-private-demo

# This saves ~$140/month
```

**Note**: To access the VM without Bastion, you would need to temporarily add a public IP or use other access methods.

---

## 🔒 Security Best Practices

### 1. Access Control
✅ Use Managed Identity instead of storing credentials  
✅ Grant minimum required permissions (Secrets Officer, not Administrator)  
✅ Regularly review who has access to Key Vault  
✅ Remove access when employees leave  

### 2. Network Security
✅ Keep public access disabled on Key Vault  
✅ Never add public IP to Admin VM  
✅ Use Azure Bastion for secure access (not direct RDP/SSH)  
✅ Consider additional NSG rules to restrict traffic  

### 3. Secret Management
✅ Use descriptive naming conventions (e.g., `Prod-DB-Password`, `Dev-ApiKey`)  
✅ Add tags to secrets for organization (environment, owner, expiry)  
✅ Rotate secrets regularly (30/60/90 days)  
✅ Enable soft-delete and purge protection on Key Vault  

### 4. Monitoring & Auditing
✅ Enable diagnostic settings to log all Key Vault operations  
✅ Send logs to Log Analytics or Storage Account  
✅ Set up alerts for unusual access patterns  
✅ Review audit logs monthly  

### 5. Backup
✅ Periodically backup critical secrets  
✅ Store backups in a separate, secure location  
✅ Test restore procedures  

### 6. Compliance
✅ Document all setup steps (this README helps!)  
✅ Maintain change logs for secret updates  
✅ Follow your organization's security policies  
✅ Conduct regular security reviews  

---

## ❓ Frequently Asked Questions (FAQ)

### Q1: Why can't I see Key Vault secrets in Azure Portal?

**A**: This is **intentional and correct**! Your Key Vault has public access disabled. The Azure Portal runs on Microsoft's servers (public internet), so it cannot access your Key Vault. Use Azure CLI/PowerShell from inside the VM instead.

---

### Q2: If I open portal.azure.com from INSIDE the VM, can I access secrets?

**A**: ❌ **No!** Even if you open the Azure Portal from inside the VM:
- The portal still makes API calls from **Microsoft's backend servers** (not from your VM)
- Those servers are on the public internet
- Your Key Vault blocks public internet access

**Solution**: Use Azure CLI inside the VM:
```bash
az keyvault secret show --vault-name demokeyvault123 --name "SecretName"
```

---

### Q3: How do I know my security is working?

**A**: If you see these, your setup is **working perfectly**:
- ✅ "Access Denied" or "Unauthorized" in Azure Portal
- ✅ Azure CLI works from inside the VM
- ✅ Private endpoint shows "Approved" status

---

### Q4: Can I skip Azure Bastion to save money?

**A**: Yes! For testing/development:
- ✅ Skip Bastion (saves ~$140/month)
- ✅ Add public IP + RDP/SSH access to VM
- ✅ Still maintains Key Vault security (public access disabled)
- ⚠️ For production, use Bastion for better security

---

### Q5: What's the difference between Azure Portal and Azure CLI access?

**A**: 

| Method | Where Requests Come From | Result |
|--------|-------------------------|--------|
| **Azure Portal UI** | Microsoft's servers (public internet) | ❌ Blocked |
| **Azure CLI in VM** | Your VM (private endpoint) | ✅ Allowed |
| **PowerShell in VM** | Your VM (private endpoint) | ✅ Allowed |

**Bottom line**: Portal UI = blocked everywhere. CLI/PowerShell from VM = allowed.

---

### Q6: My DNS resolves Key Vault to public IP, not private IP. What's wrong?

**A**: Inside the VM, run:
```bash
nslookup demokeyvault123.vault.azure.net
```

- ✅ **Correct**: Should resolve to `10.10.2.x` (private IP)
- ❌ **Problem**: Resolves to public IP

**Fix**:
1. Wait 5-10 minutes for DNS propagation
2. Verify Private DNS Zone is linked to VNet
3. Restart the VM
4. Check Private DNS Zone has an A record for your Key Vault

---

### Q7: Can I access Key Vault from other VMs in the same VNet?

**A**: Yes, if:
1. The VM is in the same VNet (or peered VNet)
2. The VM has Managed Identity enabled
3. The VM's Managed Identity has RBAC permission on Key Vault
4. The VM can resolve the private DNS name

---

### Q8: What happens if I enable public access temporarily?

**A**: ⚠️ **Not recommended!** This defeats the entire purpose of the private setup:
- Anyone with proper authentication can access from internet
- Removes the network isolation security layer
- Exposes Key Vault to potential attacks

**Better alternative**: Use the VM as a jump host for all access.

---

## 📚 Additional Resources

### Microsoft Documentation
- [Azure Key Vault Overview](https://docs.microsoft.com/azure/key-vault/general/overview)
- [Private Endpoints for Key Vault](https://docs.microsoft.com/azure/key-vault/general/private-link-service)
- [Managed Identities](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/)
- [Azure Bastion Documentation](https://docs.microsoft.com/azure/bastion/)

### Azure CLI Reference
- [Key Vault Commands](https://docs.microsoft.com/cli/azure/keyvault)
- [Secret Management Commands](https://docs.microsoft.com/cli/azure/keyvault/secret)

### Security Resources
- [Azure Security Baseline for Key Vault](https://docs.microsoft.com/security/benchmark/azure/baselines/key-vault-security-baseline)
- [Azure Key Vault Best Practices](https://docs.microsoft.com/azure/key-vault/general/best-practices)

---

## ✅ Quick Command Reference

### Login Commands
```bash
# Login with Managed Identity
az login --identity

# Verify logged in account
az account show
```

### Secret Management Commands
```bash
# Set your Key Vault name
KV_NAME="demokeyvault123"

# Create/Update a secret
az keyvault secret set --vault-name $KV_NAME --name "SecretName" --value "SecretValue"

# Get secret value
az keyvault secret show --vault-name $KV_NAME --name "SecretName" --query value -o tsv

# List all secrets
az keyvault secret list --vault-name $KV_NAME --output table

# Delete a secret
az keyvault secret delete --vault-name $KV_NAME --name "SecretName"
```

---

## 📊 Final Architecture Summary

```
✅ Key Vault: Public access disabled
✅ Private Endpoint: 10.10.2.x IP address
✅ Admin VM: No public IP, managed identity enabled
✅ Azure Bastion: Secure browser-based access
✅ RBAC: Key Vault Secrets Officer role assigned
✅ DNS: Private DNS zone for name resolution
✅ Network: Isolated within Azure VNet
```

### 🔐 Security Verification

**Expected Behavior**:
- 🌐 **Azure Portal Access**: ❌ Blocked - Shows "unauthorized to view these contents" ✅ Correct!
- 💻 **Admin VM Access**: ✅ Allowed - Can create/read/update secrets ✅ Correct!

If you see "Access Denied" in the Azure Portal, **congratulations** - your security is working perfectly!

---

## ✨ Congratulations!

You now have a **production-ready, secure Azure Key Vault setup** with:
- ❌ No public internet access
- ✅ Private network access only
- ✅ Managed identity authentication
- ✅ Secure secret management

**Remember**: The "unauthorized" message in Azure Portal is **not an error** - it's proof your security is working! 

Your secrets are now protected with enterprise-grade security! 🎉

---

Good luck with your practice! 🚀

## 📧 Contact Information

**Prepared by**: Manohar Gulme  
**Email**: manohar.gulme@outlook.com  
**Phone**: +91 8919161280  
**LinkedIn**: [linkedin.com/in/manohar-gulme](https://linkedin.com/in/manohar-gulme)


