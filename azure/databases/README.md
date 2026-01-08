# Azure Databases - Complete Learning Guide

**SQL Database, PostgreSQL, MySQL, Cosmos DB & Database Management**

---

## 📚 Table of Contents

1. [Azure Database Services Overview](#overview)
2. [Azure SQL Database](#azure-sql-database)
3. [Azure Database for PostgreSQL](#azure-postgresql)
4. [Azure Database for MySQL](#azure-mysql)
5. [Azure Cosmos DB](#azure-cosmos-db)
6. [High Availability & Replication](#high-availability)
7. [Backup and Restore](#backup-restore)
8. [Security Best Practices](#security-practices)
9. [Performance Tuning](#performance-tuning)
10. [Cost Optimization](#cost-optimization)

---

## 🎯 Azure Database Services Overview

Azure offers multiple managed database services:

| Service | Type | Use Case | Max Size |
|---------|------|----------|----------|
| **Azure SQL Database** | Relational (MS SQL) | General purpose, enterprise | 4 TB |
| **Azure PostgreSQL** | Relational (PostgreSQL) | Open source, JSON support | 16 TB |
| **Azure MySQL** | Relational (MySQL) | WordPress, LAMP stack | 16 TB |
| **Azure Cosmos DB** | NoSQL (Multi-model) | Global distribution, low latency | Unlimited |
| **Azure Database for MariaDB** | Relational (MariaDB) | MySQL fork | 16 TB |
| **Azure SQL Managed Instance** | SQL Server | Lift-and-shift | 16 TB |

---

## 💾 Azure SQL Database

**Azure SQL Database** is a fully managed relational database (PaaS) based on Microsoft SQL Server.

### **Deployment Options:**

| Option | Description | Use Case |
|--------|-------------|----------|
| **Single Database** | Isolated database | Single application |
| **Elastic Pool** | Shared resources | Multiple databases, unpredictable usage |
| **Managed Instance** | Full SQL Server compatibility | Lift-and-shift migrations |

---

### **Pricing Tiers:**

**DTU-Based (Simple):**
- **Basic** - 5 DTUs, 2 GB, $4.99/month
- **Standard** - 10-3000 DTUs, up to 1 TB
- **Premium** - 125-4000 DTUs, up to 4 TB

**vCore-Based (Flexible):**
- **General Purpose** - Balanced compute/storage
- **Business Critical** - High performance, read replicas
- **Hyperscale** - Up to 100 TB, fast scaling

---

### **Create SQL Server and Database:**

```bash
# Create SQL Server
az sql server create \
  --resource-group rg-sql \
  --name sql-prod-server-001 \
  --location eastus \
  --admin-user sqladmin \
  --admin-password 'P@ssw0rd1234!SecurePassword'

# Configure Azure AD admin
az sql server ad-admin create \
  --resource-group rg-sql \
  --server-name sql-prod-server-001 \
  --display-name "DBA Team" \
  --object-id <AAD_GROUP_OBJECT_ID>

# Create database (vCore model)
az sql db create \
  --resource-group rg-sql \
  --server sql-prod-server-001 \
  --name sqldb-appdata \
  --edition GeneralPurpose \
  --family Gen5 \
  --capacity 2 \
  --compute-model Provisioned \
  --backup-storage-redundancy Local \
  --zone-redundant false

# Create database (DTU model)
az sql db create \
  --resource-group rg-sql \
  --server sql-prod-server-001 \
  --name sqldb-webapp \
  --service-objective S2
```

---

### **Firewall Rules:**

```bash
# Allow Azure services
az sql server firewall-rule create \
  --resource-group rg-sql \
  --server sql-prod-server-001 \
  --name AllowAzureServices \
  --start-ip-address 0.0.0.0 \
  --end-ip-address 0.0.0.0

# Allow specific IP
az sql server firewall-rule create \
  --resource-group rg-sql \
  --server sql-prod-server-001 \
  --name AllowMyIP \
  --start-ip-address 203.0.113.5 \
  --end-ip-address 203.0.113.5

# Allow IP range
az sql server firewall-rule create \
  --resource-group rg-sql \
  --server sql-prod-server-001 \
  --name AllowOfficeNetwork \
  --start-ip-address 203.0.113.0 \
  --end-ip-address 203.0.113.255

# List firewall rules
az sql server firewall-rule list \
  --resource-group rg-sql \
  --server sql-prod-server-001 \
  --output table

# Delete firewall rule
az sql server firewall-rule delete \
  --resource-group rg-sql \
  --server sql-prod-server-001 \
  --name AllowMyIP
```

---

### **Connection Strings:**

```bash
# Get connection string
az sql db show-connection-string \
  --client ado.net \
  --name sqldb-appdata \
  --server sql-prod-server-001

# Output:
# Server=tcp:sql-prod-server-001.database.windows.net,1433;
# Database=sqldb-appdata;
# User ID=sqladmin;
# Password={your_password};
# Encrypt=true;
# Connection Timeout=30;
```

**Connection Strings for Different Languages:**

**C# / .NET:**
```csharp
Server=tcp:sql-prod-server-001.database.windows.net,1433;
Initial Catalog=sqldb-appdata;
Persist Security Info=False;
User ID=sqladmin;
Password={your_password};
MultipleActiveResultSets=False;
Encrypt=True;
TrustServerCertificate=False;
Connection Timeout=30;
```

**Python:**
```python
import pyodbc

connection_string = (
    "DRIVER={ODBC Driver 18 for SQL Server};"
    "SERVER=sql-prod-server-001.database.windows.net;"
    "DATABASE=sqldb-appdata;"
    "UID=sqladmin;"
    "PWD={your_password}"
)

conn = pyodbc.connect(connection_string)
cursor = conn.cursor()
cursor.execute("SELECT @@VERSION")
print(cursor.fetchone())
```

**Node.js:**
```javascript
const sql = require('mssql');

const config = {
    server: 'sql-prod-server-001.database.windows.net',
    database: 'sqldb-appdata',
    user: 'sqladmin',
    password: '{your_password}',
    options: {
        encrypt: true,
        enableArithAbort: true
    }
};

sql.connect(config).then(pool => {
    return pool.request().query('SELECT @@VERSION')
}).then(result => {
    console.log(result);
}).catch(err => {
    console.error(err);
});
```

---

### **Read Replicas (Business Critical tier):**

```bash
# Create read replica
az sql db replica create \
  --resource-group rg-sql \
  --server sql-prod-server-001 \
  --name sqldb-appdata \
  --partner-server sql-prod-server-002 \
  --partner-resource-group rg-sql-secondary \
  --service-objective P2

# Failover to secondary
az sql db replica set-primary \
  --resource-group rg-sql-secondary \
  --server sql-prod-server-002 \
  --name sqldb-appdata

# Remove replication
az sql db replica delete-link \
  --resource-group rg-sql \
  --server sql-prod-server-001 \
  --name sqldb-appdata \
  --partner-server sql-prod-server-002 \
  --partner-resource-group rg-sql-secondary
```

---

## 🐘 Azure Database for PostgreSQL

Azure offers two deployment options:

| Option | Use Case |
|--------|----------|
| **Single Server** | Simple deployment (being retired) |
| **Flexible Server** | Recommended, zone-redundant HA |

---

### **Create PostgreSQL Flexible Server:**

```bash
# Create PostgreSQL server
az postgres flexible-server create \
  --resource-group rg-sql \
  --name psql-prod-server-001 \
  --location eastus \
  --admin-user psqladmin \
  --admin-password 'P@ssw0rd1234!' \
  --sku-name Standard_D2s_v3 \
  --tier GeneralPurpose \
  --storage-size 128 \
  --version 15 \
  --high-availability ZoneRedundant \
  --zone 1 \
  --standby-zone 2

# Create database
az postgres flexible-server db create \
  --resource-group rg-sql \
  --server-name psql-prod-server-001 \
  --database-name appdb

# Show server details
az postgres flexible-server show \
  --resource-group rg-sql \
  --name psql-prod-server-001 \
  --output table
```

---

### **Configure Firewall:**

```bash
# Allow Azure services
az postgres flexible-server firewall-rule create \
  --resource-group rg-sql \
  --name psql-prod-server-001 \
  --rule-name AllowAzureServices \
  --start-ip-address 0.0.0.0 \
  --end-ip-address 0.0.0.0

# Allow specific IP
az postgres flexible-server firewall-rule create \
  --resource-group rg-sql \
  --name psql-prod-server-001 \
  --rule-name AllowMyIP \
  --start-ip-address 203.0.113.5 \
  --end-ip-address 203.0.113.5

# List firewall rules
az postgres flexible-server firewall-rule list \
  --resource-group rg-sql \
  --name psql-prod-server-001 \
  --output table
```

---

### **Connect to PostgreSQL:**

```bash
# Connection string
psql "host=psql-prod-server-001.postgres.database.azure.com \
      port=5432 \
      dbname=appdb \
      user=psqladmin \
      password=P@ssw0rd1234! \
      sslmode=require"

# Or set environment variable
export PGPASSWORD='P@ssw0rd1234!'
psql -h psql-prod-server-001.postgres.database.azure.com \
     -U psqladmin \
     -d appdb
```

**Python Example:**
```python
import psycopg2

conn = psycopg2.connect(
    host="psql-prod-server-001.postgres.database.azure.com",
    database="appdb",
    user="psqladmin",
    password="P@ssw0rd1234!",
    sslmode="require"
)

cursor = conn.cursor()
cursor.execute("SELECT version();")
print(cursor.fetchone())
cursor.close()
conn.close()
```

---

### **Server Parameters:**

```bash
# Update max_connections
az postgres flexible-server parameter set \
  --resource-group rg-sql \
  --server-name psql-prod-server-001 \
  --name max_connections \
  --value 200

# Update work_mem
az postgres flexible-server parameter set \
  --resource-group rg-sql \
  --server-name psql-prod-server-001 \
  --name work_mem \
  --value 16384

# List all parameters
az postgres flexible-server parameter list \
  --resource-group rg-sql \
  --server-name psql-prod-server-001 \
  --output table
```

---

## 🐬 Azure Database for MySQL

### **Create MySQL Flexible Server:**

```bash
# Create MySQL server
az mysql flexible-server create \
  --resource-group rg-sql \
  --name mysql-prod-server-001 \
  --location eastus \
  --admin-user mysqladmin \
  --admin-password 'P@ssw0rd1234!' \
  --sku-name Standard_D2ds_v4 \
  --tier GeneralPurpose \
  --storage-size 128 \
  --version 8.0.21 \
  --high-availability ZoneRedundant

# Create database
az mysql flexible-server db create \
  --resource-group rg-sql \
  --server-name mysql-prod-server-001 \
  --database-name wordpressdb

# Connect
mysql -h mysql-prod-server-001.mysql.database.azure.com \
      -u mysqladmin \
      -p \
      --ssl-mode=REQUIRED \
      wordpressdb
```

---

## 🌍 Azure Cosmos DB

**Cosmos DB** is a globally distributed, multi-model NoSQL database.

### **APIs Supported:**

| API | Use Case |
|-----|----------|
| **SQL (Core)** | Document database (JSON) |
| **MongoDB** | MongoDB compatibility |
| **Cassandra** | Wide-column store |
| **Gremlin** | Graph database |
| **Table** | Key-value store |

---

### **Create Cosmos DB Account:**

```bash
# Create Cosmos DB account (SQL API)
az cosmosdb create \
  --resource-group rg-cosmos \
  --name cosmos-prod-account-001 \
  --locations regionName=eastus failoverPriority=0 isZoneRedundant=True \
  --locations regionName=westus failoverPriority=1 isZoneRedundant=True \
  --default-consistency-level Session \
  --enable-automatic-failover true \
  --enable-multiple-write-locations false

# Create database
az cosmosdb sql database create \
  --account-name cosmos-prod-account-001 \
  --resource-group rg-cosmos \
  --name appdb

# Create container
az cosmosdb sql container create \
  --account-name cosmos-prod-account-001 \
  --resource-group rg-cosmos \
  --database-name appdb \
  --name users \
  --partition-key-path "/userId" \
  --throughput 400

# Get connection string
az cosmosdb keys list \
  --name cosmos-prod-account-001 \
  --resource-group rg-cosmos \
  --type connection-strings \
  --query "connectionStrings[0].connectionString" \
  --output tsv
```

---

### **Python Example (SQL API):**

```python
from azure.cosmos import CosmosClient, PartitionKey

# Connect
endpoint = "https://cosmos-prod-account-001.documents.azure.com:443/"
key = "your-primary-key"
client = CosmosClient(endpoint, key)

# Get database and container
database = client.get_database_client("appdb")
container = database.get_container_client("users")

# Create item
user = {
    "id": "user1",
    "userId": "user1",
    "name": "John Doe",
    "email": "john@example.com"
}
container.create_item(body=user)

# Query items
query = "SELECT * FROM c WHERE c.email = @email"
parameters = [{"name": "@email", "value": "john@example.com"}]
results = container.query_items(query=query, parameters=parameters, enable_cross_partition_query=True)

for item in results:
    print(item)

# Read item
item = container.read_item(item="user1", partition_key="user1")
print(item)

# Update item
item["email"] = "john.doe@example.com"
container.replace_item(item=item["id"], body=item)

# Delete item
container.delete_item(item="user1", partition_key="user1")
```

---

## 🔄 High Availability & Replication

### **SQL Database - Geo-Replication:**

```bash
# Create geo-replica in West US
az sql db replica create \
  --resource-group rg-sql \
  --server sql-prod-server-001 \
  --name sqldb-appdata \
  --partner-server sql-westus-server \
  --partner-resource-group rg-sql-westus \
  --service-objective P2

# Manual failover
az sql db replica set-primary \
  --resource-group rg-sql-westus \
  --server sql-westus-server \
  --name sqldb-appdata
```

---

### **PostgreSQL - Zone-Redundant HA:**

```bash
# Enable high availability
az postgres flexible-server update \
  --resource-group rg-sql \
  --name psql-prod-server-001 \
  --high-availability ZoneRedundant \
  --standby-zone 2

# Disable high availability
az postgres flexible-server update \
  --resource-group rg-sql \
  --name psql-prod-server-001 \
  --high-availability Disabled
```

---

## 💾 Backup and Restore

### **SQL Database - Automated Backups:**

```bash
# View backup policy
az sql db show \
  --resource-group rg-sql \
  --server sql-prod-server-001 \
  --name sqldb-appdata \
  --query "earliestRestoreDate"

# Restore to point-in-time
az sql db restore \
  --resource-group rg-sql \
  --server sql-prod-server-001 \
  --name sqldb-appdata-restored \
  --source-database sqldb-appdata \
  --time "2024-01-15T10:30:00Z"

# Restore deleted database
az sql db restore \
  --resource-group rg-sql \
  --server sql-prod-server-001 \
  --name sqldb-appdata-restored \
  --deleted-time "2024-01-15T12:00:00Z" \
  --source-database sqldb-appdata
```

---

### **PostgreSQL - Backup and Restore:**

```bash
# List backups
az postgres flexible-server backup list \
  --resource-group rg-sql \
  --name psql-prod-server-001 \
  --output table

# Restore to point-in-time
az postgres flexible-server restore \
  --resource-group rg-sql \
  --name psql-restored-server \
  --source-server psql-prod-server-001 \
  --restore-time "2024-01-15T10:30:00Z"
```

---

## 🔒 Security Best Practices

**1. Use Azure AD Authentication**
```bash
az sql server ad-admin create \
  --resource-group rg-sql \
  --server-name sql-prod-server-001 \
  --display-name "DBA Team" \
  --object-id <GROUP_OBJECT_ID>
```

**2. Enable Transparent Data Encryption (TDE)**
```bash
az sql db tde set \
  --resource-group rg-sql \
  --server sql-prod-server-001 \
  --database sqldb-appdata \
  --status Enabled
```

**3. Private Endpoints**
```bash
az sql server update \
  --resource-group rg-sql \
  --name sql-prod-server-001 \
  --public-network-access Disabled
```

---

## ⚡ Performance Tuning

**1. Enable Query Performance Insights**
```bash
az sql db show \
  --resource-group rg-sql \
  --server sql-prod-server-001 \
  --name sqldb-appdata \
  --query queryInsightsEnabled
```

**2. Auto-tuning**
```bash
az sql db update \
  --resource-group rg-sql \
  --server sql-prod-server-001 \
  --name sqldb-appdata \
  --auto-create-indexes Enabled \
  --auto-drop-indexes Enabled
```

---

## 💰 Cost Optimization

**1. Use Serverless for Dev/Test**
```bash
az sql db update \
  --resource-group rg-sql \
  --server sql-prod-server-001 \
  --name sqldb-dev \
  --compute-model Serverless \
  --auto-pause-delay 60
```

**2. Reserved Capacity (1-3 year commit)**
- Save up to 80% vs pay-as-you-go

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)
