# Azure Storage - Complete Learning Guide

**Comprehensive Tutorial: Blob, Files, Queues, Tables & Data Lake**

---

## 📚 What You'll Learn

Complete guide to Azure Storage services:

1. **Storage Accounts** - Foundation and configuration
2. **Blob Storage** - Object storage for unstructured data
3. **Azure Files** - Managed file shares
4. **Queue Storage** - Message queuing
5. **Table Storage** - NoSQL key-value store
6. **Data Lake Storage Gen2** - Big data analytics
7. **Storage Security** - Access control and encryption
8. **Lifecycle Management** - Cost optimization
9. **Replication & Backup** - Data protection

---

## 🎯 What is Azure Storage?

**Azure Storage** is Microsoft's cloud storage solution providing highly available, durable, and scalable storage for various data types.

### **Core Storage Services:**

```
┌─────────────────────────────────────────────┐
│        Azure Storage Account                │
├─────────────────────────────────────────────┤
│  📦 Blob Storage  │  Unstructured data     │
│                   │  (files, images, logs)  │
├─────────────────────────────────────────────┤
│  📁 Azure Files   │  SMB file shares        │
│                   │  (shared folders)       │
├─────────────────────────────────────────────┤
│  📨 Queue Storage │  Message queuing        │
│                   │  (async processing)     │
├─────────────────────────────────────────────┤
│  🗄️ Table Storage │  NoSQL key-value        │
│                   │  (structured data)      │
├─────────────────────────────────────────────┤
│  🌊 Data Lake Gen2│  Big data analytics     │
│                   │  (hierarchical storage) │
└─────────────────────────────────────────────┘
```

---

## 📦 Understanding Blob Storage

**Blob Storage** stores unstructured data like documents, images, videos, backups, and logs.

### **What is a Blob?**

A **blob** (Binary Large Object) is any file stored in the cloud.

**Blob Types:**
- **Block Blobs**: Text and binary files (documents, media, backups)
- **Append Blobs**: Log files (optimized for append operations)
- **Page Blobs**: Virtual hard disks (VHDs) for VMs

### **Blob Storage Hierarchy:**

```
Storage Account
    └── Container (like a folder)
        └── Blob (the actual file)
            └── Snapshot (point-in-time copy)
```

**Real-World Example:**
```
stproddata001 (Storage Account)
  └── images (Container)
      ├── logo.png (Blob)
      ├── banner.jpg (Blob)
      └── profile-pics/
          └── user123.jpg (Blob with virtual directory)
```

### **Access Tiers (Cost vs Performance):**

| Tier | Access Time | Best For | Cost/GB |
|------|-------------|----------|---------|
| **Premium** | < 10ms | Databases, high IOPS | $$$$ |
| **Hot** | Milliseconds | Frequently accessed files | $$$ |
| **Cool** | Milliseconds | Infrequently accessed (30+ days) | $$ |
| **Archive** | Hours | Long-term archival (180+ days) | $ |

**When to Use Each:**
- **Hot**: Active website images, frequently accessed logs
- **Cool**: Backups accessed monthly, old reports
- **Archive**: Compliance data, historical records

---

## 📁 Understanding Azure Files

**Azure Files** provides fully managed file shares accessible via SMB (Server Message Block) protocol.

### **What is Azure Files?**

Think of it as **network drives in the cloud** that multiple VMs or users can access simultaneously.

**Key Difference from Blob Storage:**
- **Blob Storage**: Object storage, accessed via HTTP/REST API
- **Azure Files**: File system storage, accessed via SMB (like Windows file shares)

### **Common Use Cases:**

**1. Lift-and-Shift Applications:**
```
On-Premises App → Uses file share → Migrate to Azure
                                   → Mount Azure Files
                                   → No code changes needed!
```

**2. Shared Configuration Files:**
```
Multiple VMs/Containers
    ↓
Azure Files Share (config files)
    ↓
All apps read same config
```

**3. Development Tools:**
```
Developer Workstations → Mount Azure Files
    ↓
Shared codebase, libraries, tools
```

### **Azure Files Tiers:**

| Tier | Use Case | IOPS |
|------|----------|------|
| **Standard (Transaction optimized)** | General purpose | Up to 10,000 |
| **Standard (Hot)** | Frequent access | Up to 10,000 |
| **Standard (Cool)** | Archive, backup | Up to 10,000 |
| **Premium** | High-performance apps | Up to 100,000 |

---

## 📨 Understanding Queue Storage

**Queue Storage** provides reliable messaging between application components.

### **What is Queue Storage?**

A **queue** is a message buffer between a sender and receiver, enabling **asynchronous processing**.

**How it Works:**

```
┌──────────────┐
│  Web Server  │  ←  User uploads image
└──────┬───────┘
       │ 1. Receive upload
       ↓ 2. Store in Blob Storage
┌─────────────────┐
│  Azure Queue    │  ←  Add message: "Process image123.jpg"
└──────┬──────────┘
       │ 3. Worker picks up message
       ↓
┌──────────────┐
│ Worker (VM)  │  ←  4. Resize image, create thumbnail
└──────┬───────┘
       │ 5. Delete message when done
       ↓
     ✓ Complete
```

### **Why Use Queues?**

**❌ Without Queues:**
```
User Upload → Web Server processes → User waits 30 seconds
                 (resizing, thumbnails)
```

**✅ With Queues:**
```
User Upload → Web Server stores → Returns immediately (1 second)
              Queue message → Background worker processes
```

**Benefits:**
- **Decoupling**: Web servers and workers independent
- **Scalability**: Add more workers during peak times
- **Reliability**: Messages persist until processed
- **Load Leveling**: Handle traffic spikes smoothly

### **Common Use Cases:**

1. **Order Processing**: User places order → Queue → Payment processing → Fulfillment
2. **Image Processing**: Upload photo → Queue → Resize → Thumbnail → Watermark
3. **Email Notifications**: Action occurs → Queue → Send email worker
4. **Batch Jobs**: Data arrives → Queue → Process in batches

### **Message Lifecycle:**

```
1. Enqueue: Add message to queue
2. Dequeue: Worker gets message (becomes invisible to others)
3. Process: Worker processes the task
4. Delete: Worker deletes message (job done)
   OR
   Timeout: Message becomes visible again (if worker fails)
```

---

## 🗄️ Understanding Table Storage

**Table Storage** is a NoSQL key-value store for structured, non-relational data.

### **What is Table Storage?**

A **schemaless** data store where each row can have different columns.

**Structure:**

```
Table: Users
┌────────────┬──────────┬─────────┬──────┬─────────┐
│ PartitionKey│ RowKey  │  Name   │ Age  │  City   │
├────────────┼──────────┼─────────┼──────┼─────────┤
│   USA      │  user001 │  John   │  30  │ Seattle │
│   USA      │  user002 │  Jane   │  25  │ Boston  │
│   UK       │  user003 │  Bob    │  35  │ London  │
└────────────┴──────────┴─────────┴──────┴─────────┘
```

**Key Concepts:**
- **PartitionKey**: Groups related rows together (affects performance)
- **RowKey**: Unique ID within partition
- **Together**: Primary key = PartitionKey + RowKey

### **When to Use Table Storage:**

**✅ Good For:**
- User profiles and session data
- IoT device telemetry
- Metadata for blobs
- Application logs
- Shopping cart data

**❌ Not Good For:**
- Complex queries with JOIN operations
- Transactions across partitions
- Full-text search
- ACID requirements (use Azure SQL instead)

### **Comparison:**

| Feature | Table Storage | Azure SQL | Cosmos DB |
|---------|---------------|-----------|-----------|
| **Type** | NoSQL | Relational | NoSQL |
| **Schema** | Schemaless | Fixed schema | Flexible |
| **Scalability** | High | Medium | Very High |
| **Cost** | $ | $$$ | $$$ |
| **Global Distribution** | No | No | Yes |
| **Query Complexity** | Simple | Complex (SQL) | Complex |

---

## 🌊 Understanding Data Lake Storage Gen2

**Data Lake Storage Gen2** combines blob storage with a hierarchical file system for big data analytics.

### **What is Data Lake Gen2?**

Think of it as **Blob Storage + File System Features** optimized for analytics workloads.

**Key Features:**
- **Hierarchical Namespace**: Real directories (not just virtual)
- **Hadoop Compatible**: Works with Apache Spark, Hive, Databricks
- **POSIX Permissions**: File and directory-level ACLs
- **Scalability**: Exabytes of data

### **Data Lake Structure:**

```
Storage Account (with Hierarchical Namespace enabled)
    └── Container (Filesystem)
        ├── raw/           (Landing zone)
        │   └── 2024/01/15/data.csv
        ├── bronze/        (Raw ingested data)
        │   └── cleaned_data.parquet
        ├── silver/        (Transformed data)
        │   └── aggregated.parquet
        └── gold/          (Business-ready data)
            └── reports.parquet
```

### **When to Use Data Lake Gen2:**

**✅ Perfect For:**
- Big data analytics (PB-scale)
- Data warehousing with Azure Synapse
- Machine learning data pipelines
- Batch and stream processing
- Multi-zone data processing

**❌ Not Ideal For:**
- Simple file storage (use Azure Files)
- Small-scale applications
- Traditional file sharing

---

## 📊 Storage Account Types

| Type | Performance | Use Case | Cost |
|------|-------------|----------|------|
| **Standard (GPv2)** | Standard | General purpose, most common | $ |
| **Premium Block Blobs** | Premium | High transaction rates, low latency | $$$ |
| **Premium File Shares** | Premium | Enterprise file shares, databases | $$$ |
| **Premium Page Blobs** | Premium | VM disks (managed disks) | $$$ |

### **Choosing Storage Account Type:**

**Standard GPv2 (General Purpose v2):**
- ✅ Most flexible
- ✅ Supports all storage types (Blob, Files, Queue, Table)
- ✅ All access tiers (Hot, Cool, Archive)
- ✅ Cost-effective for most workloads
- **Use for**: 90% of scenarios

**Premium Performance:**
- ✅ Low latency (< 10ms)
- ✅ High IOPS (100,000+)
- ✅ SSD-backed storage
- ❌ More expensive
- **Use for**: Databases, high-performance apps

---

## 💾 1. Create Storage Account

### **Azure CLI**

```bash
# Create storage account
az storage account create \
  --resource-group rg-storage \
  --name stproddata001 \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2 \
  --access-tier Hot \
  --https-only true \
  --min-tls-version TLS1_2 \
  --allow-blob-public-access false

# Get connection string
az storage account show-connection-string \
  --resource-group rg-storage \
  --name stproddata001 \
  --output tsv

# Get account keys
az storage account keys list \
  --resource-group rg-storage \
  --name stproddata001 \
  --output table
```

### **PowerShell**

```powershell
New-AzStorageAccount `
  -ResourceGroupName "rg-storage" `
  -Name "stproddata001" `
  -Location "East US" `
  -SkuName "Standard_LRS" `
  -Kind "StorageV2" `
  -AccessTier "Hot" `
  -EnableHttpsTrafficOnly $true `
  -MinimumTlsVersion "TLS1_2"
```

### **Terraform**

```hcl
resource "azurerm_storage_account" "main" {
  name                     = "stproddata001"
  resource_group_name      = azurerm_resource_group.main.name
  location                 = azurerm_resource_group.main.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
  account_kind             = "StorageV2"
  access_tier              = "Hot"
  
  enable_https_traffic_only = true
  min_tls_version          = "TLS1_2"
  allow_blob_public_access = false
  
  blob_properties {
    versioning_enabled = true
    
    delete_retention_policy {
      days = 7
    }
    
    container_delete_retention_policy {
      days = 7
    }
  }
  
  tags = {
    Environment = "Production"
  }
}
```

---

## 📦 2. Blob Storage

### **Create Container and Upload Blobs**

```bash
# Create container
az storage container create \
  --account-name stproddata001 \
  --name data \
  --public-access off

# Upload blob
az storage blob upload \
  --account-name stproddata001 \
  --container-name data \
  --name myfile.txt \
  --file ./myfile.txt \
  --tier Hot

# Upload directory
az storage blob upload-batch \
  --account-name stproddata001 \
  --destination data \
  --source ./local-folder \
  --pattern "*.txt"

# Download blob
az storage blob download \
  --account-name stproddata001 \
  --container-name data \
  --name myfile.txt \
  --file ./downloaded-file.txt

# List blobs
az storage blob list \
  --account-name stproddata001 \
  --container-name data \
  --output table

# Generate SAS token (7 days)
az storage blob generate-sas \
  --account-name stproddata001 \
  --container-name data \
  --name myfile.txt \
  --permissions r \
  --expiry $(date -u -d "7 days" '+%Y-%m-%dT%H:%MZ') \
  --output tsv
```

### **Blob Storage - Python SDK**

```python
from azure.storage.blob import BlobServiceClient, BlobClient, ContainerClient

# Connect to storage account
connection_string = "DefaultEndpointsProtocol=https;AccountName=..."
blob_service_client = BlobServiceClient.from_connection_string(connection_string)

# Create container
container_client = blob_service_client.create_container("data")

# Upload blob
with open("myfile.txt", "rb") as data:
    blob_client = blob_service_client.get_blob_client(
        container="data", 
        blob="myfile.txt"
    )
    blob_client.upload_blob(data, overwrite=True)

# Download blob
blob_client = blob_service_client.get_blob_client(
    container="data",
    blob="myfile.txt"
)
with open("downloaded.txt", "wb") as download_file:
    download_file.write(blob_client.download_blob().readall())

# List blobs
container_client = blob_service_client.get_container_client("data")
blob_list = container_client.list_blobs()
for blob in blob_list:
    print(f"Name: {blob.name}")
```

---

## 📁 3. Azure Files (File Shares)

### **Create File Share - CLI**

```bash
# Create file share
az storage share create \
  --account-name stproddata001 \
  --name share1 \
  --quota 100

# Upload file
az storage file upload \
  --account-name stproddata001 \
  --share-name share1 \
  --source ./localfile.txt \
  --path remotefile.txt

# Create directory
az storage directory create \
  --account-name stproddata001 \
  --share-name share1 \
  --name mydirectory

# List files
az storage file list \
  --account-name stproddata001 \
  --share-name share1 \
  --output table

# Mount file share on Linux
sudo mkdir /mnt/azurefileshare
sudo mount -t cifs //stproddata001.file.core.windows.net/share1 /mnt/azurefileshare \
  -o vers=3.0,username=stproddata001,password=<storage-key>,dir_mode=0777,file_mode=0777

# Mount on Windows
net use Z: \\stproddata001.file.core.windows.net\share1 /user:Azure\stproddata001 <storage-key>
```

### **File Share - Terraform**

```hcl
resource "azurerm_storage_share" "main" {
  name                 = "share1"
  storage_account_name = azurerm_storage_account.main.name
  quota                = 100
  
  metadata = {
    environment = "production"
  }
}
```

---

## 📨 4. Queue Storage (Messaging)

### **Work with Queues - CLI**

```bash
# Create queue
az storage queue create \
  --account-name stproddata001 \
  --name myqueue

# Send message
az storage message put \
  --account-name stproddata001 \
  --queue-name myqueue \
  --content "Hello, Queue!"

# Peek messages
az storage message peek \
  --account-name stproddata001 \
  --queue-name myqueue \
  --num-messages 5

# Get message (dequeue)
az storage message get \
  --account-name stproddata001 \
  --queue-name myqueue

# Clear queue
az storage message clear \
  --account-name stproddata001 \
  --queue-name myqueue
```

### **Queue Storage - Python**

```python
from azure.storage.queue import QueueClient

# Connect to queue
queue_client = QueueClient.from_connection_string(
    connection_string, 
    "myqueue"
)

# Send message
queue_client.send_message("Hello, Queue!")

# Receive messages
messages = queue_client.receive_messages(messages_per_page=5)
for message in messages:
    print(message.content)
    # Delete message after processing
    queue_client.delete_message(message)
```

---

## 🗄️ 5. Table Storage (NoSQL)

### **Create and Query Tables - CLI**

```bash
# Create table
az storage table create \
  --account-name stproddata001 \
  --name mytable

# Insert entity
az storage entity insert \
  --account-name stproddata001 \
  --table-name mytable \
  --entity PartitionKey=partition1 RowKey=row1 Name=John Age@odata.type=Edm.Int32 Age=30

# Query entities
az storage entity query \
  --account-name stproddata001 \
  --table-name mytable \
  --filter "PartitionKey eq 'partition1'"
```

---

## 🌊 6. Data Lake Storage Gen2

### **Enable Hierarchical Namespace**

```bash
# Create storage account with hierarchical namespace
az storage account create \
  --resource-group rg-storage \
  --name stdatalake001 \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2 \
  --hierarchical-namespace true

# Create filesystem (container)
az storage fs create \
  --account-name stdatalake001 \
  --name data \
  --auth-mode login

# Upload file with directory structure
az storage fs file upload \
  --account-name stdatalake001 \
  --file-system data \
  --source ./data.csv \
  --path /bronze/data.csv \
  --auth-mode login

# Set ACLs
az storage fs access set \
  --account-name stdatalake001 \
  --file-system data \
  --path /bronze \
  --acl "user::rwx,group::r-x,other::---" \
  --auth-mode login
```

---

## 🔒 7. Storage Security

### **Private Endpoints**

```bash
# Disable public network access
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
```

### **SAS Tokens**

```bash
# Generate account SAS
az storage account generate-sas \
  --account-name stproddata001 \
  --services b \
  --resource-types sco \
  --permissions rwdlac \
  --expiry $(date -u -d "30 days" '+%Y-%m-%dT%H:%MZ') \
  --output tsv

# Generate blob SAS with IP restrictions
az storage blob generate-sas \
  --account-name stproddata001 \
  --container-name data \
  --name myfile.txt \
  --permissions r \
  --expiry $(date -u -d "1 hour" '+%Y-%m-%dT%H:%MZ') \
  --ip "203.0.113.0/24" \
  --output tsv
```

---

## ♻️ 8. Lifecycle Management

### **Configure Lifecycle Policy**

```json
{
  "rules": [
    {
      "enabled": true,
      "name": "move-to-cool-after-30-days",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 30
            },
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 90
            },
            "delete": {
              "daysAfterModificationGreaterThan": 365
            }
          },
          "snapshot": {
            "delete": {
              "daysAfterCreationGreaterThan": 90
            }
          }
        },
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["logs/"]
        }
      }
    },
    {
      "enabled": true,
      "name": "delete-old-backups",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "delete": {
              "daysAfterModificationGreaterThan": 7
            }
          }
        },
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["backups/"]
        }
      }
    }
  ]
}
```

```bash
# Apply lifecycle policy
az storage account management-policy create \
  --account-name stproddata001 \
  --resource-group rg-storage \
  --policy @lifecycle-policy.json
```

---

## 🔄 9. Replication Options

| Type | Description | Durability | Cost |
|------|-------------|------------|------|
| **LRS** | 3 copies in single datacenter | 11 nines | Lowest |
| **ZRS** | 3 copies across availability zones | 12 nines | Low |
| **GRS** | LRS + async copy to secondary region | 16 nines | Medium |
| **GZRS** | ZRS + async copy to secondary region | 16 nines | Higher |
| **RA-GRS** | GRS with read access to secondary | 16 nines | Medium |
| **RA-GZRS** | GZRS with read access to secondary | 16 nines | Highest |

```bash
# Change replication type
az storage account update \
  --resource-group rg-storage \
  --name stproddata001 \
  --sku Standard_GRS
```

---

## 🎯 Real-World Use Cases

### **Use Case 1: Static Website Hosting**

```bash
# Enable static website hosting
az storage blob service-properties update \
  --account-name stproddata001 \
  --static-website \
  --index-document index.html \
  --error-document404-path 404.html

# Upload website files
az storage blob upload-batch \
  --account-name stproddata001 \
  --destination '$web' \
  --source ./website \
  --pattern "*.html" "*.css" "*.js"

# Get website URL
az storage account show \
  --name stproddata001 \
  --query "primaryEndpoints.web" \
  --output tsv
# Output: https://stproddata001.z13.web.core.windows.net/

# Configure custom domain (requires DNS CNAME)
# CNAME: www.mysite.com → stproddata001.z13.web.core.windows.net
az storage account update \
  --name stproddata001 \
  --resource-group rg-storage \
  --custom-domain www.mysite.com

# Use Azure CDN for global distribution
az cdn endpoint create \
  --resource-group rg-storage \
  --profile-name cdn-profile \
  --name cdn-mysite \
  --origin stproddata001.z13.web.core.windows.net \
  --origin-host-header stproddata001.z13.web.core.windows.net
```

---

### **Use Case 2: Backup Storage for VMs**

```bash
# Create backup container
az storage container create \
  --account-name stproddata001 \
  --name vm-backups \
  --public-access off

# Backup VM using Azure Backup (automated)
az backup protection enable-for-vm \
  --resource-group rg-compute \
  --vault-name vault-backup-prod \
  --vm vm-web-01 \
  --policy-name DefaultPolicy

# Or manual snapshot
az vm disk list \
  --resource-group rg-compute \
  --vm-name vm-web-01 \
  --query "[].{Name:name,ID:id}" \
  --output table

# Create disk snapshot
az snapshot create \
  --resource-group rg-storage \
  --name vm-web-01-snapshot \
  --source $(az vm show --resource-group rg-compute --name vm-web-01 --query "storageProfile.osDisk.managedDisk.id" --output tsv)

# Export snapshot to blob
az snapshot grant-access \
  --resource-group rg-storage \
  --name vm-web-01-snapshot \
  --duration-in-seconds 3600 \
  --query accessSas \
  --output tsv
```

---

### **Use Case 3: Log Aggregation**

```bash
# Create container for logs
az storage container create \
  --account-name stproddata001 \
  --name application-logs \
  --public-access off

# Python script to upload logs
cat > upload_logs.py << 'EOF'
from azure.storage.blob import BlobServiceClient
from datetime import datetime
import logging

# Configure logging
logging.basicConfig(level=logging.INFO)

# Connect to storage
connection_string = "DefaultEndpointsProtocol=https;AccountName=..."
blob_service_client = BlobServiceClient.from_connection_string(connection_string)

# Upload log file
container_client = blob_service_client.get_container_client("application-logs")
log_filename = f"app-{datetime.now().strftime('%Y%m%d-%H%M%S')}.log"

with open("app.log", "rb") as data:
    blob_client = container_client.get_blob_client(log_filename)
    blob_client.upload_blob(data, overwrite=True)
    logging.info(f"Uploaded: {log_filename}")
EOF

# Run daily via cron or Azure Function
```

---

### **Use Case 4: Data Archive with Lifecycle Management**

```bash
# Automatically move old data to Archive tier
cat > lifecycle-policy.json << 'EOF'
{
  "rules": [
    {
      "enabled": true,
      "name": "archive-old-logs",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 30
            },
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 90
            },
            "delete": {
              "daysAfterModificationGreaterThan": 365
            }
          }
        },
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["logs/"]
        }
      }
    },
    {
      "enabled": true,
      "name": "delete-temp-files",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "delete": {
              "daysAfterModificationGreaterThan": 7
            }
          }
        },
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["temp/"]
        }
      }
    }
  ]
}
EOF

# Apply lifecycle policy
az storage account management-policy create \
  --account-name stproddata001 \
  --resource-group rg-storage \
  --policy @lifecycle-policy.json

# View policy
az storage account management-policy show \
  --account-name stproddata001 \
  --resource-group rg-storage
```

---

## ⚡ Performance Optimization

### **1. Choose Right Storage Tier:**

| Tier | Access Latency | Best For | Cost |
|------|----------------|----------|------|
| **Premium** | < 10ms | Databases, high IOPS | Highest |
| **Hot** | ms latency | Frequent access | High |
| **Cool** | ms latency | Infrequent access (30+ days) | Medium |
| **Archive** | hours | Long-term archive (180+ days) | Lowest |

---

### **2. Enable CDN for Global Access:**

```bash
# Create CDN profile
az cdn profile create \
  --resource-group rg-storage \
  --name cdn-global-profile \
  --sku Standard_Microsoft

# Create CDN endpoint
az cdn endpoint create \
  --resource-group rg-storage \
  --profile-name cdn-global-profile \
  --name cdn-storage-endpoint \
  --origin stproddata001.blob.core.windows.net \
  --origin-host-header stproddata001.blob.core.windows.net \
  --enable-compression true

# Purge CDN cache
az cdn endpoint purge \
  --resource-group rg-storage \
  --profile-name cdn-global-profile \
  --name cdn-storage-endpoint \
  --content-paths '/*'
```

---

### **3. Optimize Blob Access Patterns:**

**Python - Concurrent Upload:**
```python
from azure.storage.blob import BlobServiceClient
from concurrent.futures import ThreadPoolExecutor
import os

connection_string = "DefaultEndpointsProtocol=https;AccountName=..."
blob_service_client = BlobServiceClient.from_connection_string(connection_string)
container_client = blob_service_client.get_container_client("data")

def upload_file(file_path):
    blob_name = os.path.basename(file_path)
    blob_client = container_client.get_blob_client(blob_name)
    with open(file_path, "rb") as data:
        blob_client.upload_blob(data, overwrite=True, max_concurrency=4)
    print(f"Uploaded: {blob_name}")

# Upload multiple files concurrently
files = ["file1.dat", "file2.dat", "file3.dat"]
with ThreadPoolExecutor(max_workers=10) as executor:
    executor.map(upload_file, files)
```

---

### **4. Use Appropriate Block Sizes:**

```python
# Large file upload with optimal block size
blob_client = blob_service_client.get_blob_client(
    container="data",
    blob="largefile.zip"
)

with open("largefile.zip", "rb") as data:
    blob_client.upload_blob(
        data,
        overwrite=True,
        max_concurrency=8,
        blob_type="BlockBlob",
        max_block_size=4*1024*1024,  # 4MB blocks
        max_single_put_size=256*1024*1024  # 256MB threshold
    )
```

---

## 🔒 Advanced Security

### **1. Shared Access Signatures (SAS) - Best Practices:**

**Account SAS (Full Account Access):**
```bash
# Generate account SAS with minimal permissions
az storage account generate-sas \
  --account-name stproddata001 \
  --services b \
  --resource-types co \
  --permissions rl \
  --expiry $(date -u -d "7 days" '+%Y-%m-%dT%H:%MZ') \
  --https-only \
  --output tsv
```

**Service SAS (Container/Blob Level):**
```bash
# Generate blob SAS with IP restriction
az storage blob generate-sas \
  --account-name stproddata001 \
  --container-name data \
  --name report.pdf \
  --permissions r \
  --expiry $(date -u -d "24 hours" '+%Y-%m-%dT%H:%MZ') \
  --ip "203.0.113.0/24" \
  --https-only \
  --output tsv
```

**Stored Access Policy (Revocable SAS):**
```bash
# Create stored access policy
az storage container policy create \
  --account-name stproddata001 \
  --container-name data \
  --name read-policy \
  --permissions r \
  --expiry $(date -u -d "30 days" '+%Y-%m-%dT%H:%MZ')

# Generate SAS using policy
az storage blob generate-sas \
  --account-name stproddata001 \
  --container-name data \
  --name report.pdf \
  --policy-name read-policy \
  --output tsv

# Revoke access by deleting policy
az storage container policy delete \
  --account-name stproddata001 \
  --container-name data \
  --name read-policy
```

---

### **2. Encryption at Rest & Transit:**

```bash
# Enable infrastructure encryption (double encryption)
az storage account create \
  --resource-group rg-storage \
  --name stencrypted001 \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2 \
  --require-infrastructure-encryption true

# Customer-managed keys (CMK)
# 1. Create Key Vault and key
az keyvault create \
  --resource-group rg-storage \
  --name kv-storage-keys \
  --enable-soft-delete true \
  --enable-purge-protection true

az keyvault key create \
  --vault-name kv-storage-keys \
  --name storage-encryption-key \
  --kty RSA \
  --size 2048

# 2. Grant storage account access
STORAGE_IDENTITY=$(az storage account show \
  --name stproddata001 \
  --query identity.principalId \
  --output tsv)

az keyvault set-policy \
  --name kv-storage-keys \
  --object-id $STORAGE_IDENTITY \
  --key-permissions get unwrapKey wrapKey

# 3. Enable CMK encryption
az storage account update \
  --resource-group rg-storage \
  --name stproddata001 \
  --encryption-key-source Microsoft.Keyvault \
  --encryption-key-vault https://kv-storage-keys.vault.azure.net \
  --encryption-key-name storage-encryption-key
```

---

### **3. Immutable Storage (WORM - Write Once Read Many):**

```bash
# Configure immutable storage
az storage container immutability-policy create \
  --account-name stproddata001 \
  --container-name compliance-data \
  --period 365 \
  --allow-protected-append-writes false

# Lock policy (cannot be deleted)
az storage container immutability-policy lock \
  --account-name stproddata001 \
  --container-name compliance-data \
  --if-match "<etag>"

# Legal hold (indefinite retention)
az storage container legal-hold set \
  --account-name stproddata001 \
  --container-name compliance-data \
  --tags "investigation-2024"
```

---

## 🛠️ Comprehensive Troubleshooting

### **Issue 1: Access Denied (403 Forbidden)**

**Symptoms:**
- "This request is not authorized" error
- 403 status code

**Debugging:**
```bash
# 1. Check firewall rules
az storage account show \
  --resource-group rg-storage \
  --name stproddata001 \
  --query networkRuleSet

# 2. Check public access setting
az storage account show \
  --resource-group rg-storage \
  --name stproddata001 \
  --query allowBlobPublicAccess

# 3. Verify SAS token (if using)
# Check expiry, permissions, IP restrictions

# 4. Add IP to firewall
az storage account network-rule add \
  --resource-group rg-storage \
  --account-name stproddata001 \
  --ip-address $(curl -s ifconfig.me)

# 5. Check managed identity permissions
az role assignment list \
  --assignee <MANAGED_IDENTITY_ID> \
  --scope /subscriptions/{sub-id}/resourceGroups/rg-storage/providers/Microsoft.Storage/storageAccounts/stproddata001
```

**Common Causes:**
- IP not whitelisted in firewall
- SAS token expired
- Insufficient permissions
- CORS settings (for browser access)

---

### **Issue 2: Connection Timeout**

**Symptoms:**
- Timeout errors
- Cannot resolve storage account hostname

**Debugging:**
```bash
# 1. Test DNS resolution
nslookup stproddata001.blob.core.windows.net

# 2. Check if private endpoint resolves correctly
dig stproddata001.blob.core.windows.net

# 3. Test connectivity
curl -I https://stproddata001.blob.core.windows.net

# 4. Check private endpoint status
az network private-endpoint show \
  --resource-group rg-storage \
  --name pe-storage \
  --query "customDnsConfigs"

# 5. Verify private DNS zone
az network private-dns zone show \
  --resource-group rg-storage \
  --name privatelink.blob.core.windows.net
```

**Solutions:**
- Fix private DNS zone configuration
- Update DNS link to VNet
- Check NSG rules on subnet
- Verify route table

---

### **Issue 3: Slow Upload/Download Performance**

**Debugging:**
```bash
# 1. Check storage account metrics
az monitor metrics list \
  --resource $(az storage account show --resource-group rg-storage --name stproddata001 --query id --output tsv) \
  --metric "Transactions" "UsedCapacity" "Egress" "Ingress" \
  --start-time 2024-01-01T00:00:00Z

# 2. Test with AzCopy (optimized tool)
azcopy bench "https://stproddata001.blob.core.windows.net/data?<SAS>" \
  --file-count 100 \
  --size-per-file 10M

# 3. Check if being throttled
az monitor metrics list \
  --resource $(az storage account show --resource-group rg-storage --name stproddata001 --query id --output tsv) \
  --metric "SuccessE2ELatency" "SuccessServerLatency" \
  --start-time 2024-01-01T00:00:00Z
```

**Solutions:**
- Use Premium storage for high IOPS
- Increase concurrency in uploads
- Use CDN for downloads
- Implement retry logic
- Check network bandwidth

---

### **Issue 4: Blob Not Found (404)**

**Debugging:**
```bash
# List blobs with prefix
az storage blob list \
  --account-name stproddata001 \
  --container-name data \
  --prefix "folder/" \
  --output table

# Check blob properties
az storage blob show \
  --account-name stproddata001 \
  --container-name data \
  --name myfile.txt

# List deleted blobs (if soft delete enabled)
az storage blob list \
  --account-name stproddata001 \
  --container-name data \
  --include d \
  --output table

# Restore deleted blob
az storage blob undelete \
  --account-name stproddata001 \
  --container-name data \
  --name myfile.txt
```

---

## 💰 Cost Optimization Strategies

### **1. Use Appropriate Storage Tiers:**

```bash
# Move cold data to Cool tier
az storage blob set-tier \
  --account-name stproddata001 \
  --container-name data \
  --name oldfile.zip \
  --tier Cool

# Bulk tier change
az storage blob list \
  --account-name stproddata001 \
  --container-name data \
  --query "[?properties.lastModified<'2023-01-01'].name" \
  --output tsv | \
while read blob; do
  az storage blob set-tier \
    --account-name stproddata001 \
    --container-name data \
    --name "$blob" \
    --tier Archive
done
```

---

### **2. Implement Lifecycle Management:**

**Cost Savings Example:**
```
Hot Storage: $0.018/GB/month
Cool Storage: $0.01/GB/month
Archive Storage: $0.002/GB/month

For 1TB of data:
- Hot: $18.40/month
- Cool: $10.24/month (44% savings)
- Archive: $2.05/month (89% savings)
```

---

### **3. Monitor and Analyze Costs:**

```bash
# View storage account costs
az consumption usage list \
  --start-date 2024-01-01 \
  --end-date 2024-01-31 \
  --query "[?contains(instanceName, 'stproddata001')]" \
  --output table

# Set up cost alerts
az consumption budget create \
  --resource-group rg-storage \
  --budget-name storage-budget \
  --amount 100 \
  --time-grain Monthly \
  --start-date 2024-01-01
```

---

### **4. Optimize Data Transfer:**

- ✅ Use **Azure Data Box** for large initial uploads (>40TB)
- ✅ Compress data before uploading
- ✅ Use **AzCopy** for efficient transfers
- ✅ Leverage **private endpoints** to avoid egress charges

---

## ✅ Storage Best Practices

### **1. Security:**
- ✅ Disable public blob access by default
- ✅ Use private endpoints for PaaS connections
- ✅ Enable soft delete (7-365 days)
- ✅ Use managed identities for authentication
- ✅ Enable versioning for critical data
- ✅ Implement RBAC instead of shared keys

### **2. Reliability:**
- ✅ Use ZRS or GZRS for production workloads
- ✅ Enable soft delete and versioning
- ✅ Implement backup strategy
- ✅ Test disaster recovery procedures
- ✅ Monitor replication status

### **3. Performance:**
- ✅ Use Premium for high IOPS workloads
- ✅ Enable CDN for static content
- ✅ Implement caching strategies
- ✅ Use appropriate block sizes
- ✅ Maximize concurrency

### **4. Cost Management:**
- ✅ Implement lifecycle management
- ✅ Use appropriate storage tiers
- ✅ Delete unnecessary snapshots
- ✅ Monitor and analyze usage
- ✅ Set up budget alerts

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

