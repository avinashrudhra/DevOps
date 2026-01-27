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
│                          │                                      │
│  ┌───────────────────────┴──────────────────┐                 │
│  │ admin-vm-subnet (10.10.1.0/24)           │                 │
│  │                                           │                 │
│  │   ┌────────────────────────────┐         │                 │
│  │   │  kv-admin-vm               │         │                 │
│  │   │  - No Public IP            │◄────────┼────────┐        │
│  │   │  - Managed Identity        │         │        │        │
│  │   │  - Azure CLI               │         │        │        │
│  │   └────────────┬───────────────┘         │        │        │
│  └────────────────┼────────────────────────┬┘        │        │
│                   │                        │          │        │
│                   │ Private Connection     │          │        │
│                   ↓                        │     ┌────┴─────┐  │
│  ┌────────────────────────────────────────┼─┐   │ Bastion  │  │
│  │ kv-private-endpoint-subnet             │ │   │ Subnet   │  │
│  │ (10.10.2.0/24)                         │ │   │ (10.10.  │  │
│  │                                        │ │   │  3.0/26) │  │
│  │   ┌────────────────────────────┐      │ │   └──────────┘  │
│  │   │  Private Endpoint          │      │ │         ▲        │
│  │   │  10.10.2.x                 │      │ │         │        │
│  │   └────────────┬───────────────┘      │ │         │        │
│  └────────────────┼────────────────────────┘         │        │
│                   │                          Browser │        │
│                   ↓                          Access  │        │
│         ┌──────────────────────┐                     │        │
│         │  Azure Key Vault     │◄────────────────────┘        │
│         │  (Private Only)      │                              │
│         │  ✅ Private Endpoint │                              │
│         │  ❌ Public Disabled  │                              │
│         └──────────────────────┘                              │
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
| Azure Bastion | Basic tier | ~$140 |
| Private Endpoint | 1 endpoint | ~$7 |
| Private DNS Zone | 1 zone | ~$0.50 |
| Virtual Network | - | Free |
| **TOTAL** | | **~$178/month** |

### 💡 Cost Saving Tips
- **Stop VM when not in use**: Saves ~$30/month
- **Delete Bastion when not needed**: Saves ~$140/month (can recreate when needed)
- **Use shared VM**: Multiple users can use same VM

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
2. **Delete** the default subnet if it exists

#### 2.3 Create Subnet 1: Admin VM Subnet

1. Click **+ Add subnet**
2. **Subnet name**: `admin-vm-subnet`
3. **Subnet address range**: `10.10.1.0/24`
   - Provides 256 IP addresses
4. Click **Add**

#### 2.4 Create Subnet 2: Private Endpoint Subnet

1. Click **+ Add subnet**
2. **Subnet name**: `kv-private-endpoint-subnet`
3. **Subnet address range**: `10.10.2.0/24`
4. Click **Add**

#### 2.5 Create Subnet 3: Azure Bastion Subnet

1. Click **+ Add subnet**
2. **Subnet name**: `AzureBastionSubnet` ⚠️ **EXACT NAME REQUIRED**
3. **Subnet address range**: `10.10.3.0/26`
   - Minimum /26 (64 IPs) required for Bastion
4. Click **Add**

#### 2.6 Finalize VNet Creation

1. Click **Review + Create**
2. Click **Create**
3. Wait for deployment (~30 seconds)

✅ **Verification**: Go to Virtual Network → Subnets → Should see 3 subnets

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

✅ **Verification**: Key Vault is created but shows "Access Denied" in portal (this is expected and correct!)

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

---

### STEP 8: Create Azure Bastion

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

---

### STEP 9: Connect to Admin VM via Azure Bastion

Now you can securely connect to the VM through your web browser.

#### 9.1 Initiate Bastion Connection
1. Go to **Virtual Machines** → `kv-admin-vm`
2. Click **Connect** button (top menu)
3. From the dropdown, select **Connect via Bastion** (not "Bastion" directly, use the dropdown)
4. Alternatively, click the **Bastion** tab from the available connection methods

#### 9.2 Enter Credentials
- **Authentication Type**: **Password**
- **Username**: `azureadmin` (the username you created)
- **Password**: Your VM password
- Click **Connect**

#### 9.3 Browser Opens VM Session
- **For Ubuntu**: A terminal window opens in a new browser tab
- **For Windows**: An RDP session opens in a new browser tab

✅ **You are now securely connected to the VM!**

---

### STEP 10: Install Azure CLI in VM (Ubuntu Only)

If you chose Ubuntu, you need to install Azure CLI. For Windows VMs, Azure CLI is typically pre-installed.

**Inside the Bastion terminal session**, run the following commands:

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

Open a **new browser tab** (on your local machine, NOT inside the VM):

1. Go to **Azure Portal** → **Key Vault** → `demokeyvault123`
2. Try to access **Secrets** from the left menu
3. You should see an **error message** or **"Access Denied"**

✅ **This confirms public access is completely blocked!**

**Important**: Only the Admin VM inside the VNet can access the Key Vault through the private endpoint.

---

## 🔧 Troubleshooting

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

---

## ✨ Congratulations!

You now have a **production-ready, secure Azure Key Vault setup** with:
- ❌ No public internet access
- ✅ Private network access only
- ✅ Managed identity authentication
- ✅ Secure secret management

Your secrets are now protected with enterprise-grade security! 🎉

---

**Last Updated**: January 27, 2026  
**Version**: 1.0  
**Guide Type**: Azure Portal (Manual Setup)
