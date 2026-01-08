# Azure Security - Complete Learning Guide

**Key Vault, Managed Identities, RBAC & Security Best Practices**

---

## 📚 Table of Contents

1. [Azure Security Overview](#azure-security-overview)
2. [Azure Key Vault](#azure-key-vault)
3. [Managed Identities](#managed-identities)
4. [Role-Based Access Control (RBAC)](#rbac)
5. [Service Principals](#service-principals)
6. [Network Security](#network-security)
7. [Azure Security Center/Defender](#security-center)
8. [Compliance and Governance](#compliance-governance)
9. [Real-World Security Patterns](#security-patterns)

---

## 🎯 Azure Security Overview

### **Azure Security Layers:**

```
┌─────────────────────────────────────┐
│  Physical Security (Datacenters)    │
└─────────────────────────────────────┘
           ↓
┌─────────────────────────────────────┐
│  Infrastructure Security (Network)  │
│  - NSGs, Firewalls, DDoS            │
└─────────────────────────────────────┘
           ↓
┌─────────────────────────────────────┐
│  Identity & Access (Azure AD)       │
│  - RBAC, MFA, Conditional Access    │
└─────────────────────────────────────┘
           ↓
┌─────────────────────────────────────┐
│  Application Security               │
│  - Key Vault, App Gateway WAF       │
└─────────────────────────────────────┘
           ↓
┌─────────────────────────────────────┐
│  Data Security                      │
│  - Encryption at Rest/Transit       │
└─────────────────────────────────────┘
```

---

## 🔐 Understanding Azure Key Vault

**Azure Key Vault** is a centralized cloud service for securely storing and accessing secrets, encryption keys, and certificates.

### **What Problem Does Key Vault Solve?**

**❌ The Old Way (Insecure):**
```python
# DON'T DO THIS! ❌
database_password = "MyPassword123!"  # Hardcoded in code
api_key = "sk-1234567890abcdef"       # Committed to Git
connection_string = "Server=..."      # Visible in config files
```

**Problems:**
- Secrets in source code (visible in Git history)
- Secrets in config files (exposed if file is leaked)
- Difficult to rotate credentials
- No audit trail of who accessed what
- Shared secrets (multiple apps use same password)

**✅ The Key Vault Way (Secure):**
```python
# DO THIS! ✅
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient

credential = DefaultAzureCredential()  # Uses managed identity
client = SecretClient(vault_url="https://kv-prod.vault.azure.net", credential=credential)

database_password = client.get_secret("db-password").value  # Retrieved securely
api_key = client.get_secret("api-key").value
```

**Benefits:**
- ✅ No secrets in code or config
- ✅ Centralized secret management
- ✅ Easy credential rotation
- ✅ Full audit trail
- ✅ Access control per secret
- ✅ Secrets versioned automatically

---

### **Key Vault Components:**

```
┌─────────────────────────────────────────┐
│         Azure Key Vault                  │
├─────────────────────────────────────────┤
│  🔑 Secrets                             │
│  - Database passwords                   │
│  - API keys                             │
│  - Connection strings                   │
│  - Tokens                               │
├─────────────────────────────────────────┤
│  🔐 Keys (Encryption Keys)              │
│  - Disk encryption                      │
│  - Database TDE keys                    │
│  - Application encryption               │
│  - Signing keys                         │
├─────────────────────────────────────────┤
│  📜 Certificates                        │
│  - SSL/TLS certificates                 │
│  - Code signing certificates            │
│  - Auto-renewal support                 │
└─────────────────────────────────────────┘
```

---

### **Key Vault Types (What to Store):**

| Type | What It Stores | Example | Use Case |
|------|----------------|---------|----------|
| **Secrets** | Passwords, tokens, strings | `"MyP@ssw0rd123"` | Database passwords, API keys |
| **Keys** | Encryption keys (RSA/EC) | RSA 2048-bit key | Disk encryption, data encryption |
| **Certificates** | X.509 certificates | SSL certificate | HTTPS, code signing |

---

### **Real-World Scenarios:**

**Scenario 1: Web Application Using Database**

**❌ Without Key Vault:**
```
appsettings.json:
{
  "ConnectionStrings": {
    "Database": "Server=sql.database.windows.net;Password=MyPassword123!"
  }
}
```
☹️ Password visible to anyone with access to config file!

**✅ With Key Vault:**
```
appsettings.json:
{
  "ConnectionStrings": {
    "Database": "@Microsoft.KeyVault(SecretUri=https://kv-prod.vault.azure.net/secrets/db-connection/)"
  }
}
```
😊 App Service retrieves secret automatically using managed identity!

---

**Scenario 2: Credential Rotation**

**❌ Without Key Vault:**
```
1. Developer creates new password
2. Update password in database
3. Update appsettings.json in code
4. Commit and push to Git
5. Deploy all applications
6. Restart all services

Time: 2-3 hours, High risk of downtime
```

**✅ With Key Vault:**
```
1. Update secret in Key Vault
2. Applications automatically pick up new value

Time: 2 minutes, No deployment needed, Zero downtime
```

---

**Scenario 3: Multi-Environment Management**

```
Development Environment:
  Key Vault: kv-dev
    Secret: api-key = "dev-key-123"

Production Environment:
  Key Vault: kv-prod
    Secret: api-key = "prod-key-xyz"

Same code, different Key Vaults!
Apps automatically connect to correct environment.
```

---

### **Key Vault Security Features:**

**1. Soft Delete (Safety Net):**
```
Delete secret → Retained for 7-90 days → Can be recovered
If not recovered → Permanently deleted after retention period
```

**2. Purge Protection (Extra Safety):**
```
Even if deleted, cannot be permanently deleted during retention period
Prevents accidental or malicious permanent deletion
```

**3. Access Policies vs RBAC:**

| Feature | Access Policies | Azure RBAC |
|---------|----------------|------------|
| **Scope** | Key Vault level | Vault + individual secrets |
| **Granularity** | Permissions for all secrets | Per-secret permissions |
| **Management** | Key Vault specific | Unified with Azure |
| **Recommended** | Legacy | ✅ Modern approach |

---

### **When to Use Key Vault:**

**✅ Perfect For:**
- Storing database passwords
- API keys and tokens
- Connection strings
- Encryption keys
- SSL/TLS certificates
- Service credentials
- Any sensitive configuration

**❌ Not Meant For:**
- Non-sensitive configuration (use App Configuration)
- Large files (use Blob Storage)
- Structured data (use Database)
- High-frequency access (has rate limits)

---

## 🔐 Azure Key Vault - Implementation

Now let's create and use Key Vault!

### **Create Key Vault:**

```bash
# Create Key Vault
az keyvault create \
  --resource-group rg-security \
  --name kv-prod-secrets-001 \
  --location eastus \
  --enable-soft-delete true \
  --soft-delete-retention-days 90 \
  --enable-purge-protection true \
  --enabled-for-deployment true \
  --enabled-for-disk-encryption true \
  --enabled-for-template-deployment true

# Enable diagnostic logging
az monitor diagnostic-settings create \
  --name kv-diagnostics \
  --resource $(az keyvault show -n kv-prod-secrets-001 -g rg-security --query id -o tsv) \
  --logs '[{"category": "AuditEvent","enabled": true}]' \
  --workspace <LOG_ANALYTICS_WORKSPACE_ID>
```

**Key Vault Properties Explained:**
- `--enable-soft-delete` - Deleted items retained for 90 days (prevents accidental deletion)
- `--enable-purge-protection` - Cannot permanently delete during retention period
- `--enabled-for-deployment` - VMs can retrieve certificates during deployment
- `--enabled-for-disk-encryption` - Azure Disk Encryption can access keys
- `--enabled-for-template-deployment` - ARM templates can retrieve secrets

---

### **Managing Secrets:**

```bash
# Add secret
az keyvault secret set \
  --vault-name kv-prod-secrets-001 \
  --name database-connection-string \
  --value "Server=sql-prod.database.windows.net;Database=appdb;..."

# Add secret with expiration
az keyvault secret set \
  --vault-name kv-prod-secrets-001 \
  --name api-key \
  --value "sk-abc123xyz" \
  --expires $(date -u -d "90 days" '+%Y-%m-%dT%H:%M:%SZ')

# Get secret value
az keyvault secret show \
  --vault-name kv-prod-secrets-001 \
  --name database-connection-string \
  --query value \
  --output tsv

# List all secrets
az keyvault secret list \
  --vault-name kv-prod-secrets-001 \
  --output table

# List secret versions
az keyvault secret list-versions \
  --vault-name kv-prod-secrets-001 \
  --name database-connection-string \
  --output table

# Delete secret (soft delete)
az keyvault secret delete \
  --vault-name kv-prod-secrets-001 \
  --name api-key

# Recover deleted secret
az keyvault secret recover \
  --vault-name kv-prod-secrets-001 \
  --name api-key

# Permanently delete (if purge protection disabled)
az keyvault secret purge \
  --vault-name kv-prod-secrets-001 \
  --name api-key
```

---

### **Managing Keys (Encryption Keys):**

```bash
# Create RSA key
az keyvault key create \
  --vault-name kv-prod-secrets-001 \
  --name encryption-key \
  --kty RSA \
  --size 2048 \
  --ops encrypt decrypt wrapKey unwrapKey

# Create key with rotation policy
az keyvault key rotation-policy update \
  --vault-name kv-prod-secrets-001 \
  --name encryption-key \
  --value '{
    "lifetimeActions": [
      {
        "trigger": {
          "timeAfterCreate": "P90D"
        },
        "action": {
          "type": "Rotate"
        }
      }
    ],
    "attributes": {
      "expiryTime": "P1Y"
    }
  }'

# List keys
az keyvault key list \
  --vault-name kv-prod-secrets-001 \
  --output table

# Get public key
az keyvault key show \
  --vault-name kv-prod-secrets-001 \
  --name encryption-key \
  --query key.n
```

---

### **Managing Certificates:**

```bash
# Create self-signed certificate
az keyvault certificate create \
  --vault-name kv-prod-secrets-001 \
  --name ssl-certificate \
  --policy '{
    "issuerParameters": {
      "name": "Self"
    },
    "keyProperties": {
      "exportable": true,
      "keySize": 2048,
      "keyType": "RSA",
      "reuseKey": false
    },
    "secretProperties": {
      "contentType": "application/x-pkcs12"
    },
    "x509CertificateProperties": {
      "subject": "CN=myapp.com",
      "subjectAlternativeNames": {
        "dnsNames": ["myapp.com", "www.myapp.com"]
      },
      "validityInMonths": 12
    }
  }'

# Import existing certificate
az keyvault certificate import \
  --vault-name kv-prod-secrets-001 \
  --name imported-cert \
  --file certificate.pfx \
  --password 'cert-password'

# Download certificate
az keyvault certificate download \
  --vault-name kv-prod-secrets-001 \
  --name ssl-certificate \
  --file certificate.pem
```

---

### **Access Policies:**

```bash
# Grant access to user
az keyvault set-policy \
  --name kv-prod-secrets-001 \
  --upn user@domain.com \
  --secret-permissions get list set delete \
  --key-permissions get list create delete \
  --certificate-permissions get list

# Grant access to managed identity
az keyvault set-policy \
  --name kv-prod-secrets-001 \
  --object-id <MANAGED_IDENTITY_OBJECT_ID> \
  --secret-permissions get list

# Grant access to service principal
az keyvault set-policy \
  --name kv-prod-secrets-001 \
  --spn <SERVICE_PRINCIPAL_ID> \
  --secret-permissions get

# Remove access
az keyvault delete-policy \
  --name kv-prod-secrets-001 \
  --object-id <OBJECT_ID>

# List access policies
az keyvault show \
  --name kv-prod-secrets-001 \
  --query properties.accessPolicies
```

---

### **Using Key Vault in Applications:**

**App Service Integration:**
```bash
# Reference secret in app settings
az webapp config appsettings set \
  --resource-group rg-webapps \
  --name webapp-prod-001 \
  --settings \
    DATABASE_CONNECTION=@Microsoft.KeyVault(SecretUri=https://kv-prod-secrets-001.vault.azure.net/secrets/database-connection-string/) \
    API_KEY=@Microsoft.KeyVault(VaultName=kv-prod-secrets-001;SecretName=api-key)
```

**Python Code Example:**
```python
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient

# Using Managed Identity (recommended)
credential = DefaultAzureCredential()
secret_client = SecretClient(
    vault_url="https://kv-prod-secrets-001.vault.azure.net",
    credential=credential
)

# Get secret
db_connection = secret_client.get_secret("database-connection-string")
print(f"Secret value: {db_connection.value}")
```

---

## 🆔 Understanding Managed Identities

**Managed Identities** solve the "how does my application authenticate to Azure services?" problem without using passwords or keys.

### **The Authentication Problem:**

**❌ Traditional Way (Insecure):**
```python
# How does this app authenticate to Azure SQL?
from azure.storage.blob import BlobServiceClient

# Option 1: Hardcoded credentials (TERRIBLE!)
connection_string = "DefaultEndpointsProtocol=https;AccountName=mystorageacct;AccountKey=abc123..."
client = BlobServiceClient.from_connection_string(connection_string)

# Problems:
# - Credentials in code
# - Need to rotate keys manually
# - If code is leaked, storage is compromised
# - Each app needs its own credentials
```

**✅ Managed Identity Way (Secure & Simple):**
```python
from azure.identity import DefaultAzureCredential
from azure.storage.blob import BlobServiceClient

# No credentials needed! Azure handles authentication automatically
credential = DefaultAzureCredential()  # Automatically finds managed identity
client = BlobServiceClient(
    account_url="https://mystorageacct.blob.core.windows.net",
    credential=credential
)

# Benefits:
# ✅ No secrets in code
# ✅ No credentials to manage
# ✅ Automatic key rotation
# ✅ Azure handles everything!
```

---

### **What is a Managed Identity?**

A **Managed Identity** is an automatically managed identity in Azure AD that your application uses to authenticate to Azure services.

**Think of it as:**
```
Your VM/App Service = Gets an automatic Azure AD identity
                    = No username/password needed
                    = Can access other Azure services
```

**How It Works:**

```
┌──────────────────┐
│  Your VM or App  │
│  with Managed    │  1. "I need to access storage"
│  Identity        │────────────────────┐
└──────────────────┘                    │
                                        ↓
                             ┌──────────────────────┐
                             │  Azure AD            │
                             │  "Here's your token" │
                             └──────────┬───────────┘
                                        │ 2. Token
┌──────────────────┐                   ↓
│  Storage Account │ ←──────── 3. Access with token
│  "Access granted"│
└──────────────────┘
```

---

### **Types of Managed Identities:**

#### **1. System-Assigned Managed Identity**

**Characteristics:**
- ✅ Automatically created with the resource
- ✅ Lifecycle tied to resource (deleted when resource is deleted)
- ✅ One identity per resource
- ✅ Cannot be shared across resources

**Use When:**
- Single VM needs to access storage
- App Service needs to access Key Vault
- One-to-one relationship

**Visual:**
```
VM-Web-01 → Has Identity A → Can access Storage
VM-Web-02 → Has Identity B → Can access different Storage
(Each VM has its own identity)
```

---

#### **2. User-Assigned Managed Identity**

**Characteristics:**
- ✅ Created as a standalone Azure resource
- ✅ Independent lifecycle (persists after resource deletion)
- ✅ Can be assigned to multiple resources
- ✅ Shared identity across resources

**Use When:**
- Multiple VMs need same access
- Identity needs to persist beyond resource lifecycle
- Many-to-one relationship

**Visual:**
```
Identity-AppAccess (User-Assigned)
    ↓
    ├── VM-Web-01 → Uses Identity-AppAccess
    ├── VM-Web-02 → Uses Identity-AppAccess
    └── App-Service-01 → Uses Identity-AppAccess
(All resources share the same identity)
```

---

### **System-Assigned vs User-Assigned:**

| Feature | System-Assigned | User-Assigned |
|---------|----------------|---------------|
| **Creation** | Automatic with resource | Manual standalone resource |
| **Lifecycle** | Tied to parent resource | Independent |
| **Sharing** | No (one per resource) | Yes (many resources) |
| **Management** | Easier | More flexible |
| **Use Case** | Simple scenarios | Complex multi-resource |
| **Cost** | Free | Free |

---

### **Real-World Scenarios:**

**Scenario 1: VM Accessing Key Vault**

**❌ Without Managed Identity:**
```bash
# Create service principal
az ad sp create-for-rbac --name "vm-app-sp"
# Output: appId, password

# Store credentials somewhere (insecure!)
# Configure app with these credentials
# Manually rotate every 90 days
```

**✅ With System-Assigned Managed Identity:**
```bash
# Enable managed identity on VM
az vm identity assign --resource-group rg-compute --name vm-web-01

# Grant Key Vault access
az keyvault set-policy \
  --name kv-prod \
  --object-id <VM_IDENTITY_ID> \
  --secret-permissions get list

# Done! VM can now access Key Vault automatically
# No credentials needed in code or config
```

---

**Scenario 2: Multiple App Services Accessing Same Resources**

**Problem:** 3 App Services need to access the same Storage Account and SQL Database

**✅ Solution: User-Assigned Managed Identity**
```bash
# 1. Create one user-assigned identity
az identity create \
  --resource-group rg-security \
  --name id-app-access

# 2. Assign to all 3 App Services
az webapp identity assign \
  --resource-group rg-apps \
  --name app-service-1 \
  --identities id-app-access

az webapp identity assign \
  --resource-group rg-apps \
  --name app-service-2 \
  --identities id-app-access

az webapp identity assign \
  --resource-group rg-apps \
  --name app-service-3 \
  --identities id-app-access

# 3. Grant permissions once to the identity
az role assignment create \
  --assignee <IDENTITY_PRINCIPAL_ID> \
  --role "Storage Blob Data Contributor" \
  --scope /subscriptions/{sub-id}/resourceGroups/rg-storage

# All 3 App Services can now access storage!
```

**Benefits:**
- Single point of permission management
- Add new app? Just assign the same identity
- Change permissions? Update identity once, affects all apps

---

**Scenario 3: App Service Accessing SQL Database**

**❌ Old Way:**
```csharp
// Connection string with password
"Server=tcp:sql-server.database.windows.net;Database=mydb;User ID=admin;Password=P@ssw0rd123;"
```

**✅ Managed Identity Way:**
```csharp
// Connection string without password!
"Server=tcp:sql-server.database.windows.net;Database=mydb;Authentication=Active Directory Managed Identity;"

// Azure handles authentication automatically
```

```sql
-- In SQL Database, create user for managed identity
CREATE USER [app-service-prod] FROM EXTERNAL PROVIDER;
ALTER ROLE db_datareader ADD MEMBER [app-service-prod];
ALTER ROLE db_datawriter ADD MEMBER [app-service-prod];
```

---

### **When to Use Managed Identities:**

**✅ Perfect For:**
- VM accessing Azure Storage
- App Service accessing Key Vault
- Function App accessing Cosmos DB
- Container Instances accessing ACR
- Any Azure service authenticating to another Azure service

**❌ Not Suitable For:**
- On-premises applications (use service principal)
- Third-party cloud services
- Non-Azure authentication scenarios
- When you need explicit credentials for external use

---

### **Benefits of Managed Identities:**

1. **Security:**
   - No credentials in code or config files
   - No secrets to rotate
   - Reduces attack surface

2. **Simplicity:**
   - No credential management
   - Automatic token refresh
   - Easy to implement

3. **Cost:**
   - Completely free
   - No additional charges

4. **Compliance:**
   - Audit trail in Azure AD
   - No credential sprawl
   - Easier to meet compliance requirements

---

## 🆔 Managed Identities - Implementation

Now let's enable and use Managed Identities!

### **System-Assigned Managed Identity:**

```bash
# Enable on VM
az vm identity assign \
  --resource-group rg-compute \
  --name vm-web-01

# Get identity details
IDENTITY=$(az vm identity show \
  --resource-group rg-compute \
  --name vm-web-01 \
  --query principalId \
  --output tsv)

echo "Principal ID: $IDENTITY"

# Grant permissions to storage
az role assignment create \
  --assignee $IDENTITY \
  --role "Storage Blob Data Contributor" \
  --scope /subscriptions/{sub-id}/resourceGroups/rg-storage/providers/Microsoft.Storage/storageAccounts/stproddata001

# Enable on App Service
az webapp identity assign \
  --resource-group rg-webapps \
  --name webapp-prod-001

# Enable on Function App
az functionapp identity assign \
  --resource-group rg-functions \
  --name func-app-prod
```

---

### **User-Assigned Managed Identity:**

```bash
# Create user-assigned identity
az identity create \
  --resource-group rg-security \
  --name id-app-identity

# Get identity details
IDENTITY_ID=$(az identity show \
  --resource-group rg-security \
  --name id-app-identity \
  --query id \
  --output tsv)

PRINCIPAL_ID=$(az identity show \
  --resource-group rg-security \
  --name id-app-identity \
  --query principalId \
  --output tsv)

# Assign to VM
az vm identity assign \
  --resource-group rg-compute \
  --name vm-web-01 \
  --identities $IDENTITY_ID

# Assign to multiple App Services
az webapp identity assign \
  --resource-group rg-webapps \
  --name webapp-prod-001 \
  --identities $IDENTITY_ID

az webapp identity assign \
  --resource-group rg-webapps \
  --name webapp-prod-002 \
  --identities $IDENTITY_ID

# Grant permissions
az role assignment create \
  --assignee $PRINCIPAL_ID \
  --role "Contributor" \
  --scope /subscriptions/{sub-id}/resourceGroups/rg-apps
```

---

### **Using Managed Identity in Code:**

**Python Example:**
```python
from azure.identity import ManagedIdentityCredential
from azure.storage.blob import BlobServiceClient

# Use system-assigned identity
credential = ManagedIdentityCredential()

# Connect to storage
blob_service_client = BlobServiceClient(
    account_url="https://stproddata001.blob.core.windows.net",
    credential=credential
)

# List containers
containers = blob_service_client.list_containers()
for container in containers:
    print(container.name)
```

**Node.js Example:**
```javascript
const { DefaultAzureCredential } = require("@azure/identity");
const { BlobServiceClient } = require("@azure/storage-blob");

// Use managed identity
const credential = new DefaultAzureCredential();

// Connect to storage
const blobServiceClient = new BlobServiceClient(
  "https://stproddata001.blob.core.windows.net",
  credential
);

// List containers
const containers = blobServiceClient.listContainers();
for await (const container of containers) {
  console.log(container.name);
}
```

---

## 👥 Understanding Role-Based Access Control (RBAC)

**RBAC** is Azure's authorization system that controls **who** can access **what** resources and **what** they can do with them.

### **The Access Control Problem:**

**❌ Without RBAC:**
```
Everyone is Admin
    ↓
Everyone can delete resources
    ↓
Accidental deletions
    ↓
Production downtime!
```

**✅ With RBAC:**
```
Developers → Can deploy apps, view logs
Security Team → Can manage access, view security
Auditors → Can only view resources
Admins → Full control

Each person has only the permissions they need!
```

---

### **RBAC Fundamentals:**

**The RBAC Formula:**
```
WHO (Security Principal) + WHAT (Role) + WHERE (Scope) = Permission
```

**Example:**
```
John (WHO) + Contributor (WHAT) + Production Resource Group (WHERE)
= John can manage all resources in Production, but can't grant access to others
```

---

### **RBAC Components:**

```
┌─────────────────────────────────────────────────────┐
│                RBAC Assignment                       │
├─────────────────────────────────────────────────────┤
│  WHO (Security Principal)                           │
│  - User: john@company.com                          │
│  - Group: DevOps-Team                              │
│  - Service Principal: App-Registration             │
│  - Managed Identity: vm-web-01                     │
├─────────────────────────────────────────────────────┤
│  WHAT (Role Definition)                             │
│  - Owner (full access + grant access)              │
│  - Contributor (full access, no grant)             │
│  - Reader (view only)                              │
│  - Custom roles...                                 │
├─────────────────────────────────────────────────────┤
│  WHERE (Scope)                                      │
│  - Management Group (all subscriptions)            │
│  - Subscription (entire subscription)              │
│  - Resource Group (group of resources)             │
│  - Resource (single resource)                      │
└─────────────────────────────────────────────────────┘
```

---

### **Understanding Scopes (Hierarchy):**

```
Management Group
    ↓ (Permissions inherit down)
Subscription
    ↓
Resource Group
    ↓
Resource

Example:
If you have Contributor at Subscription level
    → You're Contributor for all Resource Groups
    → You're Contributor for all Resources
```

**Scope Examples:**

| Scope Level | What It Includes | Example |
|-------------|------------------|---------|
| **Management Group** | Multiple subscriptions | All company subscriptions |
| **Subscription** | All resources in subscription | Production subscription |
| **Resource Group** | All resources in RG | rg-web-apps |
| **Resource** | Single resource | vm-web-01 |

---

### **Built-in Roles Explained:**

#### **1. Owner (Highest Privilege)**
```
Can:
✅ Create, read, update, delete resources
✅ Grant access to others (assign roles)
✅ Manage everything

Use for:
- Subscription administrators
- Platform team leads
```

#### **2. Contributor**
```
Can:
✅ Create, read, update, delete resources
❌ Cannot grant access to others

Use for:
- Developers (can deploy but not change permissions)
- DevOps engineers
- Most operational tasks
```

#### **3. Reader**
```
Can:
✅ View resources
❌ Cannot make any changes

Use for:
- Auditors
- Viewers
- Monitoring tools (read-only)
- Interns/trainees
```

#### **4. User Access Administrator**
```
Can:
✅ Manage user access (assign roles)
❌ Cannot manage resources

Use for:
- IAM/Security team (manage access but not resources)
- Least privilege for access management
```

---

### **Service-Specific Roles:**

| Role | Scope | Can Do | Cannot Do |
|------|-------|--------|-----------|
| **Virtual Machine Contributor** | VMs only | Start, stop, create VMs | Delete VMs, manage networking |
| **Storage Blob Data Contributor** | Storage | Read/write blobs | Manage storage account |
| **Key Vault Secrets User** | Key Vault | Read secrets | Create/delete secrets |
| **SQL DB Contributor** | SQL Database | Manage databases | Read data in tables |
| **Network Contributor** | Networking | Manage VNets, NSGs | Delete subscriptions |

---

### **Real-World RBAC Scenarios:**

**Scenario 1: New Developer Joins Team**

**Problem:** Developer needs to deploy apps to Dev environment, but shouldn't access Production

**Solution:**
```bash
# Give Contributor access to Dev resource group only
az role assignment create \
  --assignee developer@company.com \
  --role "Contributor" \
  --resource-group rg-dev

# Give Reader access to Production (view only)
az role assignment create \
  --assignee developer@company.com \
  --role "Reader" \
  --resource-group rg-prod

Result:
✅ Can deploy to Dev
✅ Can view Prod (for debugging)
❌ Cannot change Prod
```

---

**Scenario 2: Application Needs Storage Access**

**Problem:** App Service needs to read/write blobs, but shouldn't manage storage account

**Solution:**
```bash
# Enable managed identity on App Service
az webapp identity assign --resource-group rg-apps --name app-web-01

# Get identity ID
IDENTITY_ID=$(az webapp identity show \
  --resource-group rg-apps \
  --name app-web-01 \
  --query principalId \
  --output tsv)

# Grant storage access (data plane, not management plane)
az role assignment create \
  --assignee $IDENTITY_ID \
  --role "Storage Blob Data Contributor" \
  --scope /subscriptions/{sub-id}/resourceGroups/rg-storage/providers/Microsoft.Storage/storageAccounts/stproddata001

Result:
✅ App can read/write blobs
❌ App cannot delete storage account
❌ App cannot change storage configuration
```

---

**Scenario 3: Auditor Needs Read Access**

**Problem:** External auditor needs to view all resources but make no changes

**Solution:**
```bash
# Assign Reader role at subscription level
az role assignment create \
  --assignee auditor@external.com \
  --role "Reader" \
  --scope /subscriptions/{subscription-id}

Result:
✅ Can view all resources in subscription
✅ Can view configurations
✅ Can view metrics and logs
❌ Cannot create, modify, or delete anything
```

---

**Scenario 4: DevOps Team Needs VM Access**

**Problem:** DevOps team needs to start/stop VMs for maintenance, but shouldn't delete them

**Solution:**
```bash
# Create custom role with specific permissions
cat > vm-operator-role.json << 'EOF'
{
  "Name": "VM Operator",
  "Description": "Can start, stop, restart VMs but not delete",
  "Actions": [
    "Microsoft.Compute/virtualMachines/read",
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/powerOff/action",
    "Microsoft.Compute/virtualMachines/restart/action"
  ],
  "NotActions": [
    "Microsoft.Compute/virtualMachines/delete"
  ],
  "AssignableScopes": [
    "/subscriptions/{subscription-id}/resourceGroups/rg-compute"
  ]
}
EOF

# Create custom role
az role definition create --role-definition vm-operator-role.json

# Assign to DevOps team
az role assignment create \
  --assignee devops-team@company.com \
  --role "VM Operator" \
  --resource-group rg-compute

Result:
✅ Can start/stop/restart VMs
✅ Can view VM configurations
❌ Cannot delete VMs
❌ Cannot create new VMs
```

---

### **RBAC Best Practices:**

**1. Principle of Least Privilege:**
```
❌ Don't: Give everyone Owner role
✅ Do: Give minimum permissions needed for job
```

**2. Use Groups Instead of Individual Users:**
```
❌ Don't: Assign roles to 10 developers individually
✅ Do: Create "Developers" group, assign role to group, add users to group
```

**3. Use Built-in Roles When Possible:**
```
✅ Built-in: Well-tested, maintained by Microsoft
❌ Custom: Need to maintain yourself
```

**4. Assign Roles at Appropriate Scope:**
```
❌ Don't: Give Subscription Owner to everyone
✅ Do: Give Resource Group Contributor to specific teams
```

**5. Regular Access Reviews:**
```
✅ Review role assignments quarterly
✅ Remove access for users who left
✅ Adjust roles as responsibilities change
```

---

### **Common RBAC Mistakes:**

| Mistake | Impact | Solution |
|---------|--------|----------|
| Everyone has Owner | Security risk | Use Contributor + dedicated admins |
| Too many custom roles | Hard to maintain | Use built-in roles |
| No access reviews | Orphaned permissions | Quarterly reviews |
| Direct user assignments | Hard to manage | Use Azure AD groups |
| Wrong scope level | Too broad or too narrow | Match scope to need |

---

### **RBAC vs Azure AD Roles:**

| Feature | Azure RBAC | Azure AD Roles |
|---------|------------|----------------|
| **Controls** | Azure resources | Azure AD itself |
| **Scope** | Subscriptions, resource groups | Azure AD tenant |
| **Example Roles** | Owner, Contributor | Global Admin, User Admin |
| **Use For** | Managing VMs, storage, networks | Managing users, groups |

**Key Difference:**
- **Azure RBAC**: "Who can manage this VM?"
- **Azure AD Roles**: "Who can create users in Azure AD?"

---

## 👥 Role-Based Access Control (RBAC) - Implementation

Now let's implement RBAC!

### **Assign Roles:**

```bash
# Assign to user at subscription level
az role assignment create \
  --assignee user@domain.com \
  --role "Contributor" \
  --scope /subscriptions/{subscription-id}

# Assign to user at resource group level
az role assignment create \
  --assignee user@domain.com \
  --role "Virtual Machine Contributor" \
  --resource-group rg-compute

# Assign to user at resource level
az role assignment create \
  --assignee user@domain.com \
  --role "Storage Blob Data Contributor" \
  --scope /subscriptions/{sub-id}/resourceGroups/rg-storage/providers/Microsoft.Storage/storageAccounts/stproddata001

# Assign to managed identity
az role assignment create \
  --assignee <MANAGED_IDENTITY_PRINCIPAL_ID> \
  --role "Key Vault Secrets User" \
  --scope /subscriptions/{sub-id}/resourceGroups/rg-security/providers/Microsoft.KeyVault/vaults/kv-prod-secrets-001

# Assign to Azure AD group
az role assignment create \
  --assignee-object-id <GROUP_OBJECT_ID> \
  --assignee-principal-type Group \
  --role "Reader" \
  --resource-group rg-prod

# List role assignments
az role assignment list \
  --assignee user@domain.com \
  --output table

# Remove role assignment
az role assignment delete \
  --assignee user@domain.com \
  --role "Contributor" \
  --resource-group rg-compute
```

---

### **Custom Roles:**

```bash
# Create custom role definition
cat > vm-operator-role.json << EOF
{
  "Name": "Virtual Machine Operator",
  "Description": "Can start, stop, and restart virtual machines",
  "Actions": [
    "Microsoft.Compute/virtualMachines/read",
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/powerOff/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Network/networkInterfaces/read"
  ],
  "NotActions": [],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": [
    "/subscriptions/{subscription-id}"
  ]
}
EOF

# Create role
az role definition create --role-definition vm-operator-role.json

# Assign custom role
az role assignment create \
  --assignee user@domain.com \
  --role "Virtual Machine Operator" \
  --resource-group rg-compute

# Update custom role
az role definition update --role-definition vm-operator-role.json

# Delete custom role
az role definition delete --name "Virtual Machine Operator"
```

---

## 🔑 Service Principals

**Service Principals** are application identities for automated tools and CI/CD pipelines.

### **Create Service Principal:**

```bash
# Create service principal with Contributor role
az ad sp create-for-rbac \
  --name sp-terraform-automation \
  --role Contributor \
  --scopes /subscriptions/{subscription-id}/resourceGroups/rg-prod

# Output:
# {
#   "appId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
#   "displayName": "sp-terraform-automation",
#   "password": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
#   "tenant": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
# }

# Create with specific permissions
az ad sp create-for-rbac \
  --name sp-github-actions \
  --role "Website Contributor" \
  --scopes /subscriptions/{sub-id}/resourceGroups/rg-webapps \
  --sdk-auth \
  > github-sp-credentials.json

# Reset credentials
az ad sp credential reset \
  --id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx

# Delete service principal
az ad sp delete \
  --id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

---

### **Using Service Principal in Terraform:**

```hcl
# Configure Azure provider
provider "azurerm" {
  features {}
  
  subscription_id = var.subscription_id
  client_id       = var.client_id        # Service Principal App ID
  client_secret   = var.client_secret    # Service Principal Password
  tenant_id       = var.tenant_id
}
```

---

## 🛡️ Network Security

### **Network Security Groups (NSGs):**

```bash
# Create NSG
az network nsg create \
  --resource-group rg-networking \
  --name nsg-web-tier

# Add inbound rule (allow HTTPS)
az network nsg rule create \
  --resource-group rg-networking \
  --nsg-name nsg-web-tier \
  --name Allow-HTTPS \
  --priority 100 \
  --source-address-prefixes Internet \
  --source-port-ranges '*' \
  --destination-address-prefixes '*' \
  --destination-port-ranges 443 \
  --access Allow \
  --protocol Tcp \
  --description "Allow HTTPS traffic"

# Add rule to deny all inbound
az network nsg rule create \
  --resource-group rg-networking \
  --nsg-name nsg-web-tier \
  --name Deny-All-Inbound \
  --priority 4096 \
  --access Deny \
  --protocol '*'

# Associate NSG with subnet
az network vnet subnet update \
  --resource-group rg-networking \
  --vnet-name vnet-prod \
  --name subnet-web \
  --network-security-group nsg-web-tier
```

---

### **Private Endpoints:**

```bash
# Disable public access on storage account
az storage account update \
  --resource-group rg-storage \
  --name stproddata001 \
  --default-action Deny

# Create private endpoint
az network private-endpoint create \
  --resource-group rg-storage \
  --name pe-storage \
  --vnet-name vnet-prod \
  --subnet subnet-storage \
  --private-connection-resource-id $(az storage account show -g rg-storage -n stproddata001 --query id -o tsv) \
  --group-id blob \
  --connection-name storage-connection

# Create private DNS zone
az network private-dns zone create \
  --resource-group rg-storage \
  --name privatelink.blob.core.windows.net

# Link DNS zone to VNet
az network private-dns link vnet create \
  --resource-group rg-storage \
  --zone-name privatelink.blob.core.windows.net \
  --name storage-dns-link \
  --virtual-network vnet-prod \
  --registration-enabled false

# Create DNS record for private endpoint
az network private-endpoint dns-zone-group create \
  --resource-group rg-storage \
  --endpoint-name pe-storage \
  --name zone-group \
  --private-dns-zone privatelink.blob.core.windows.net \
  --zone-name privatelink.blob.core.windows.net
```

---

## 🌟 Real-World Security Patterns

### **Pattern 1: Secure Web Application**

```
Application → Managed Identity → Key Vault → Secrets
       ↓
   Private Endpoint
       ↓
  Database (No public access)
```

### **Pattern 2: Zero-Trust Network**

```
User → Azure AD (MFA) → Conditional Access
          ↓
    Private VNet
          ↓
   NSG + Firewall
          ↓
    Application
```

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)
