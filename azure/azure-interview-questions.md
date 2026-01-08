# Azure Interview Questions for DevOps Engineers

**120+ Questions from Basic to Advanced Production Level**

---

## 📚 Question Categories

1. **Azure Fundamentals** (Q1-Q15)
2. **Networking** (Q16-Q30)
3. **Compute** (Q31-Q45)
4. **Storage & Databases** (Q46-Q60)
5. **Kubernetes/AKS** (Q61-Q80)
6. **Security & Identity** (Q81-Q95)
7. **Monitoring & DevOps** (Q96-Q110)
8. **Architecture & Design** (Q111-Q120)

**Difficulty Levels:**
- 🟢 Basic (1-3 years experience)
- 🟡 Intermediate (3-5 years experience)
- 🔴 Advanced (5-7+ years experience)

---

## 🎯 AZURE FUNDAMENTALS

### **Q1: What is Azure Resource Manager (ARM) and what are its benefits?** 🟢

**Answer:**
Azure Resource Manager is the deployment and management service for Azure that provides a management layer that enables you to create, update, and delete resources in your Azure account.

**Key Benefits:**
- **Declarative templates**: Infrastructure as Code using ARM templates or Bicep
- **Resource grouping**: Logical organization of resources
- **RBAC integration**: Built-in access control
- **Tagging**: Resource organization and cost management
- **Dependency management**: Handles resource dependencies automatically
- **Idempotent deployments**: Same template can be deployed multiple times
- **Parallel deployment**: Resources deployed in parallel when possible

**Example:**
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2021-04-01",
      "name": "mystorageaccount",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "StorageV2"
    }
  ]
}
```

---

### **Q2: Explain the difference between Azure Regions and Availability Zones.** 🟢

**Answer:**
**Azure Regions:**
- Geographic areas containing one or more datacenters
- Separated by hundreds of miles
- Used for disaster recovery and compliance
- Example: East US, West Europe, Southeast Asia

**Availability Zones:**
- Physically separate locations within an Azure region
- Each zone has independent power, cooling, and networking
- Minimum of 3 zones in enabled regions
- Connected by high-speed, low-latency network (<2ms latency)
- Used for high availability within a region

**High Availability Strategy:**
```bash
# Deploy VM across availability zones
az vm create \
  --resource-group rg-prod \
  --name vm-web-01 \
  --image Ubuntu2204 \
  --zone 1 \
  --size Standard_D2s_v3
```

**Zone-Redundant Services:**
- Zone-redundant storage (ZRS)
- Zone-redundant Azure SQL
- Zone-redundant Load Balancer (Standard SKU)

---

### **Q3: What are Azure Resource Groups and what are the best practices?** 🟢

**Answer:**
A Resource Group is a logical container for resources deployed on Azure. All resources must exist in exactly one resource group.

**Best Practices:**
1. **Lifecycle alignment**: Group resources with similar lifecycles
2. **Naming conventions**: Use consistent naming (e.g., `rg-<workload>-<env>-<region>`)
3. **Tagging strategy**: Tag for cost allocation, ownership, environment
4. **RBAC at RG level**: Assign permissions at resource group level
5. **Lock critical RGs**: Prevent accidental deletion
6. **One region per RG**: Avoid cross-region dependencies

**Example Structure:**
```
rg-app-prod-eastus
  ├── VMs
  ├── Storage Accounts
  ├── SQL Databases
  └── Load Balancers

rg-network-prod-eastus
  ├── VNets
  ├── NSGs
  ├── Route Tables
  └── Firewalls

rg-monitoring-prod-eastus
  ├── Log Analytics
  ├── Application Insights
  └── Action Groups
```

---

### **Q4: Explain Azure subscription hierarchy and management groups.** 🟡

**Answer:**
Azure has a hierarchical structure for organizing and managing resources:

**Hierarchy (Top to Bottom):**
```
Management Group (Root)
  └── Management Group (Business Unit)
      └── Subscription
          └── Resource Group
              └── Resources
```

**Management Groups:**
- Organize subscriptions into hierarchies
- Apply policies and RBAC at scale
- Max depth: 6 levels
- Used for governance across multiple subscriptions

**Subscriptions:**
- Billing boundary
- Access control boundary
- Service limits apply at subscription level

**Use Cases:**
```bash
# Create management group
az account management-group create \
  --name "Production" \
  --display-name "Production Workloads"

# Move subscription to management group
az account management-group subscription add \
  --name "Production" \
  --subscription "sub-id"

# Assign policy at management group level (applies to all subscriptions)
az policy assignment create \
  --name "require-tags" \
  --scope "/providers/Microsoft.Management/managementGroups/Production" \
  --policy "require-tag-policy-id"
```

---

### **Q5: What is Azure Policy and how does it differ from RBAC?** 🟡

**Answer:**
**Azure Policy:**
- Focuses on resource properties (compliance)
- Enforces organizational standards
- Default action: Allow (unless explicitly denied by policy)
- Examples: Enforce tags, allowed VM sizes, allowed regions

**RBAC (Role-Based Access Control):**
- Focuses on user actions (permissions)
- Controls who can do what
- Default action: Deny (must be explicitly granted)
- Examples: Contributor, Reader, Owner roles

**Key Differences:**
| Azure Policy | RBAC |
|-------------|------|
| What can be deployed | Who can deploy |
| Resource compliance | User permissions |
| Audit and remediation | Grant/deny access |
| Applied to resources | Applied to users/groups |

**Example Scenario:**
```bash
# RBAC: Grant user permission to create VMs
az role assignment create \
  --role "Virtual Machine Contributor" \
  --assignee user@company.com \
  --scope /subscriptions/{sub-id}/resourceGroups/rg-prod

# Policy: Restrict VM sizes that can be created
az policy assignment create \
  --name "allowed-vm-sizes" \
  --policy "cccc23c7-8427-4f53-ad12-b6a63eb452b3" \
  --params '{
    "listOfAllowedSKUs": {
      "value": ["Standard_B2s", "Standard_D2s_v3"]
    }
  }' \
  --scope /subscriptions/{sub-id}/resourceGroups/rg-prod
```

---

## 🌐 NETWORKING

### **Q6: Explain the difference between Azure VNet, Subnet, and NSG.** 🟢

**Answer:**
**Virtual Network (VNet):**
- Logical isolation of Azure network
- Address space (e.g., 10.0.0.0/16)
- Regional resource (doesn't span regions)
- Enables private communication between resources

**Subnet:**
- Segment within VNet
- Smaller address range (e.g., 10.0.1.0/24)
- Used to organize and secure resources
- Can have different NSGs and route tables

**Network Security Group (NSG):**
- Firewall rules for network traffic
- Can be associated with subnet or NIC
- Contains inbound and outbound rules
- Stateful (return traffic automatically allowed)

**Example:**
```bash
# Create VNet
az network vnet create \
  --resource-group rg-network \
  --name vnet-prod \
  --address-prefix 10.0.0.0/16

# Create Subnet
az network vnet subnet create \
  --resource-group rg-network \
  --vnet-name vnet-prod \
  --name subnet-web \
  --address-prefix 10.0.1.0/24

# Create NSG with rule
az network nsg create \
  --resource-group rg-network \
  --name nsg-web

az network nsg rule create \
  --resource-group rg-network \
  --nsg-name nsg-web \
  --name Allow-HTTPS \
  --priority 100 \
  --destination-port-ranges 443 \
  --access Allow \
  --protocol Tcp

# Associate NSG with Subnet
az network vnet subnet update \
  --resource-group rg-network \
  --vnet-name vnet-prod \
  --name subnet-web \
  --network-security-group nsg-web
```

---

### **Q7: What is Azure Private Endpoint and when would you use it?** 🟡

**Answer:**
**Private Endpoint:**
A network interface that connects you privately and securely to a service powered by Azure Private Link. It uses a private IP address from your VNet, bringing the service into your VNet.

**Benefits:**
- **No public exposure**: Service accessed via private IP only
- **Data exfiltration protection**: Traffic stays on Microsoft backbone
- **Simple DNS integration**: Private DNS zones for name resolution
- **Granular access**: Per-service private connectivity

**Supported Services:**
- Storage (Blob, File, Queue, Table)
- Azure SQL Database
- Cosmos DB
- Key Vault
- Azure Kubernetes Service
- Azure Container Registry

**When to Use:**
- High security requirements
- Compliance needs (PCI-DSS, HIPAA)
- Hybrid connectivity
- Micro-segmentation

**Implementation:**
```bash
# Create private endpoint for storage account
az network private-endpoint create \
  --resource-group rg-storage \
  --name pe-storage-blob \
  --vnet-name vnet-prod \
  --subnet subnet-private-endpoints \
  --private-connection-resource-id $(az storage account show -g rg-storage -n stprod --query id -o tsv) \
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

# Create DNS zone group (automatic DNS registration)
az network private-endpoint dns-zone-group create \
  --resource-group rg-storage \
  --endpoint-name pe-storage-blob \
  --name storage-dns-group \
  --private-dns-zone privatelink.blob.core.windows.net \
  --zone-name blob
```

---

### **Q8: Describe Azure Load Balancer vs Application Gateway vs Traffic Manager.** 🟡

**Answer:**
| Feature | Load Balancer | Application Gateway | Traffic Manager |
|---------|--------------|-------------------|----------------|
| **OSI Layer** | Layer 4 (Transport) | Layer 7 (Application) | DNS-based (no layer) |
| **Protocols** | TCP, UDP | HTTP, HTTPS, WebSocket | Any |
| **Scope** | Regional | Regional | Global |
| **Use Case** | VM load balancing | Web app load balancing | Global traffic routing |
| **SSL Termination** | No | Yes | No |
| **URL Routing** | No | Yes | No |
| **WAF** | No | Yes | No |
| **Health Probes** | TCP, HTTP | HTTP, HTTPS | HTTP, HTTPS, TCP |

**Azure Load Balancer:**
```bash
# Best for: VM-to-VM traffic, non-HTTP workloads
az network lb create \
  --resource-group rg-prod \
  --name lb-internal \
  --sku Standard \
  --vnet-name vnet-prod \
  --subnet subnet-backend \
  --frontend-ip-name frontend \
  --backend-pool-name backend-pool
```

**Application Gateway:**
```bash
# Best for: HTTP/HTTPS web applications, microservices
az network application-gateway create \
  --resource-group rg-prod \
  --name appgw-web \
  --sku WAF_v2 \
  --capacity 2 \
  --vnet-name vnet-prod \
  --subnet subnet-appgw \
  --public-ip-address pip-appgw
```

**Traffic Manager:**
```bash
# Best for: Multi-region failover, geographic routing
az network traffic-manager profile create \
  --resource-group rg-global \
  --name tm-prod-app \
  --routing-method Performance \
  --unique-dns-name prod-app-global
```

**Decision Tree:**
1. **Global traffic?** → Traffic Manager
2. **HTTP/HTTPS only?** → Application Gateway
3. **Any protocol, regional?** → Load Balancer
4. **Combination:** Traffic Manager → Application Gateway → VMs

---

### **Q9: What is Service Endpoint vs Private Endpoint?** 🟡

**Answer:**
**Service Endpoints:**
- Extends VNet identity to Azure services
- Traffic stays on Azure backbone
- Service still has public IP
- Service-level access (entire storage account)
- No additional cost
- Subnet-level configuration

**Private Endpoints:**
- Brings service into your VNet with private IP
- No public IP needed
- Resource-level access (specific blob container)
- Billed per hour + data processing
- Per-resource configuration
- More secure

**Comparison:**
| Feature | Service Endpoint | Private Endpoint |
|---------|-----------------|------------------|
| **IP Address** | Public (but private routing) | Private |
| **DNS** | Public DNS name | Private DNS required |
| **Access Scope** | Entire service | Specific resource |
| **Cost** | Free | Paid |
| **Setup** | Simple | Complex |
| **Security** | Good | Excellent |

**Service Endpoint Example:**
```bash
# Enable service endpoint on subnet
az network vnet subnet update \
  --resource-group rg-network \
  --vnet-name vnet-prod \
  --name subnet-app \
  --service-endpoints Microsoft.Storage Microsoft.Sql

# Add firewall rule on storage
az storage account network-rule add \
  --resource-group rg-storage \
  --account-name stprod \
  --vnet-name vnet-prod \
  --subnet subnet-app
```

**Private Endpoint Example:**
```bash
# Create private endpoint (shown in Q7)
# More secure but more complex setup
```

**When to Use:**
- **Service Endpoints**: Cost-sensitive, simpler setup, service-level access OK
- **Private Endpoints**: High security needs, specific resource access, compliance requirements

---

### **Q10: Explain VNet Peering vs VPN Gateway.** 🟢

**Answer:**
**VNet Peering:**
- Direct connection between VNets
- Uses Azure backbone network
- Low latency, high bandwidth
- No encryption (traffic on Microsoft network)
- No gateway required
- Regional or Global
- Pay per GB transferred

**VPN Gateway:**
- IPsec/IKE VPN tunnel
- Encrypted traffic
- Higher latency
- Gateway required (cost)
- Cross-premises or VNet-to-VNet
- Up to 10 Gbps (VpnGw5AZ)

**When to Use:**
| Scenario | Recommendation |
|----------|---------------|
| Connect VNets in same/different regions | VNet Peering |
| Connect on-premises to Azure | VPN Gateway |
| Need encrypted traffic | VPN Gateway |
| Cost-sensitive, high bandwidth | VNet Peering |
| Transitive routing needed | VPN Gateway + BGP |

**VNet Peering Setup:**
```bash
# Peer VNet1 to VNet2
az network vnet peering create \
  --resource-group rg-network \
  --name vnet1-to-vnet2 \
  --vnet-name vnet-prod \
  --remote-vnet vnet-dev \
  --allow-vnet-access \
  --allow-forwarded-traffic

# Peer VNet2 to VNet1 (bidirectional)
az network vnet peering create \
  --resource-group rg-network \
  --name vnet2-to-vnet1 \
  --vnet-name vnet-dev \
  --remote-vnet vnet-prod \
  --allow-vnet-access \
  --allow-forwarded-traffic
```

---

## 💻 COMPUTE

### **Q11: Explain the difference between VM, VMSS, and AKS.** 🟢

**Answer:**
**Virtual Machine (VM):**
- Single compute instance
- Manual scaling
- Full OS control
- Best for: Legacy apps, specific OS requirements, lift-and-shift

**VM Scale Set (VMSS):**
- Group of identical VMs
- Auto-scaling (0-1000 instances)
- Load balancer integration
- Best for: Scalable stateless workloads, web farms

**Azure Kubernetes Service (AKS):**
- Managed Kubernetes cluster
- Container orchestration
- Automatic updates and patching
- Best for: Microservices, cloud-native apps, DevOps workflows

**Comparison:**
| Feature | VM | VMSS | AKS |
|---------|----|----- |-----|
| **Management** | Manual | Semi-automated | Highly automated |
| **Scaling** | Manual | Auto | Auto (HPA, CA) |
| **Deployment** | Minutes | Minutes | Seconds |
| **Updates** | Manual | Rolling | Rolling/Blue-Green |
| **Cost** | Medium | Medium | Low (container density) |

---

### **Q12: What are Availability Sets vs Availability Zones?** 🟡

**Answer:**
**Availability Sets:**
- Logical grouping within a datacenter
- Protects against rack and hardware failures
- Up to 3 fault domains (racks)
- Up to 20 update domains
- 99.95% SLA
- Free (no additional cost)
- Limitation: All VMs in same datacenter

**Availability Zones:**
- Physically separate datacenters within region
- Protects against datacenter failures
- 3 zones per supported region
- Independent power, cooling, networking
- 99.99% SLA
- Small data transfer cost between zones

**SLA Comparison:**
| Configuration | SLA | Downtime/Year |
|--------------|-----|---------------|
| Single VM (Premium SSD) | 99.9% | 8.76 hours |
| Availability Set | 99.95% | 4.38 hours |
| Availability Zone | 99.99% | 52.56 minutes |

**Best Practice:**
```bash
# Use Availability Zones for production
az vm create \
  --resource-group rg-prod \
  --name vm-web-01 \
  --image Ubuntu2204 \
  --zone 1 \
  --size Standard_D2s_v3

az vm create \
  --resource-group rg-prod \
  --name vm-web-02 \
  --image Ubuntu2204 \
  --zone 2 \
  --size Standard_D2s_v3

az vm create \
  --resource-group rg-prod \
  --name vm-web-03 \
  --image Ubuntu2204 \
  --zone 3 \
  --size Standard_D2s_v3
```

---

### **Q13: How do you implement auto-scaling for VMs in Azure?** 🟡

**Answer:**
Auto-scaling for VMs requires **VM Scale Sets (VMSS)** with autoscale rules.

**Components:**
1. **VMSS**: Group of identical VMs
2. **Autoscale Profile**: Rules for scaling
3. **Metrics**: Trigger for scaling (CPU, memory, custom)
4. **Scale Actions**: Scale out/in with cooldown

**Implementation:**
```bash
# Create VMSS
az vmss create \
  --resource-group rg-prod \
  --name vmss-web \
  --image Ubuntu2204 \
  --instance-count 2 \
  --vm-sku Standard_B2s \
  --upgrade-policy-mode Automatic \
  --admin-username azureuser \
  --generate-ssh-keys \
  --load-balancer lb-web

# Create autoscale settings
az monitor autoscale create \
  --resource-group rg-prod \
  --resource vmss-web \
  --resource-type Microsoft.Compute/virtualMachineScaleSets \
  --name autoscale-web \
  --min-count 2 \
  --max-count 10 \
  --count 2

# Scale out rule (CPU > 70%)
az monitor autoscale rule create \
  --resource-group rg-prod \
  --autoscale-name autoscale-web \
  --condition "Percentage CPU > 70 avg 5m" \
  --scale out 2 \
  --cooldown 5

# Scale in rule (CPU < 30%)
az monitor autoscale rule create \
  --resource-group rg-prod \
  --autoscale-name autoscale-web \
  --condition "Percentage CPU < 30 avg 5m" \
  --scale in 1 \
  --cooldown 10
```

**Custom Metrics:**
```bash
# Scale based on queue length
az monitor autoscale rule create \
  --resource-group rg-prod \
  --autoscale-name autoscale-web \
  --condition "ApproximateMessageCount > 100 avg 5m" \
  --scale out 3 \
  --cooldown 5 \
  --source $(az storage queue show --account-name stqueue --name myqueue --query id -o tsv)
```

**Best Practices:**
- Min count = 2 (high availability)
- Max count = based on capacity planning
- Scale out faster than scale in
- Use cooldown periods
- Monitor scale actions
- Test scaling triggers

---

## 💾 STORAGE & DATABASES

### **Q14: Explain Azure Storage redundancy options.** 🟢

**Answer:**
**LRS (Locally Redundant Storage):**
- 3 copies within single datacenter
- 99.999999999% (11 9's) durability
- Cheapest option
- Protects against: Rack/disk failures
- Does NOT protect against: Datacenter failure

**ZRS (Zone-Redundant Storage):**
- 3 copies across 3 availability zones
- 99.9999999999% (12 9's) durability
- Regional high availability
- Protects against: Datacenter failure
- Does NOT protect against: Region failure

**GRS (Geo-Redundant Storage):**
- LRS in primary + LRS in secondary region
- 99.99999999999999% (16 9's) durability
- Data in secondary is not accessible for read
- Protects against: Regional disaster

**GZRS (Geo-Zone-Redundant Storage):**
- ZRS in primary + LRS in secondary region
- 99.99999999999999% (16 9's) durability
- Best of both worlds
- Most expensive

**RA-GRS / RA-GZRS:**
- Same as GRS/GZRS but with read access to secondary region
- Use different endpoint for read: `<account>-secondary.blob.core.windows.net`

**Selection Guide:**
| Requirement | Recommendation |
|------------|----------------|
| Cost-sensitive, dev/test | LRS |
| HA within region | ZRS |
| DR to different region | GRS |
| HA + DR | GZRS |
| Read from secondary | RA-GRS / RA-GZRS |

---

### **Q15: What is Azure Blob Storage lifecycle management?** 🟡

**Answer:**
Lifecycle management allows automatic transition of blobs between access tiers or deletion based on rules.

**Access Tiers:**
- **Hot**: Frequently accessed data (highest storage cost, lowest access cost)
- **Cool**: Infrequently accessed, stored for 30+ days
- **Archive**: Rarely accessed, stored for 180+ days (hours to retrieve)

**Lifecycle Actions:**
- `tierToCool`: Move to Cool tier
- `tierToArchive`: Move to Archive tier
- `delete`: Delete blob

**Use Cases:**
- Cost optimization
- Compliance (data retention)
- Automated cleanup

**Implementation:**
```json
{
  "rules": [
    {
      "enabled": true,
      "name": "move-to-cool-and-archive",
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

**Apply Policy:**
```bash
az storage account management-policy create \
  --account-name stprod \
  --resource-group rg-storage \
  --policy @policy.json
```

---

### **Q16: Explain Azure SQL Database service tiers.** 🟢

**Answer:**
Azure SQL Database offers three purchasing models:

**1. DTU-Based Model:**
- **Basic**: Dev/test, small databases (<2GB)
  - Max DTU: 5
  - Max storage: 2GB
  - Use case: Non-production
  
- **Standard**: General purpose workloads
  - Max DTU: 3000
  - Max storage: 1TB
  - Use case: Most web applications
  
- **Premium**: Mission-critical workloads
  - Max DTU: 4000
  - Max storage: 4TB
  - Use case: High transaction volume, low latency

**2. vCore-Based Model:**
- **General Purpose**: Balanced compute/storage
  - 2-80 vCores
  - Up to 32TB storage
  - Use case: Most business workloads
  
- **Business Critical**: Low latency, high IOPS
  - 2-80 vCores
  - Read replicas included
  - Use case: OLTP applications
  
- **Hyperscale**: Highly scalable storage
  - Up to 100TB storage
  - Fast scaling
  - Use case: Large databases, variable workload

**3. Serverless:**
- Auto-pause during inactive periods
- Auto-scale compute
- Pay per second
- Use case: Intermittent workloads

**Comparison:**
```bash
# Create Basic DTU database
az sql db create \
  --resource-group rg-sql \
  --server sql-server \
  --name sqldb-dev \
  --service-objective Basic \
  --max-size 2GB

# Create General Purpose vCore database
az sql db create \
  --resource-group rg-sql \
  --server sql-server \
  --name sqldb-prod \
  --edition GeneralPurpose \
  --family Gen5 \
  --capacity 4 \
  --compute-model Provisioned

# Create Serverless database
az sql db create \
  --resource-group rg-sql \
  --server sql-server \
  --name sqldb-test \
  --edition GeneralPurpose \
  --family Gen5 \
  --compute-model Serverless \
  --auto-pause-delay 60 \
  --min-capacity 0.5 \
  --capacity 4
```

---

## ☸️ KUBERNETES/AKS

### **Q17: What is the difference between Azure CNI and kubenet in AKS?** 🟡

**Answer:**
**Kubenet (Basic Networking):**
- Default networking option
- Pods get IP from separate address space (overlay network)
- NAT used for pod communication
- Less IP addresses required
- Limitations:
  - No direct pod-to-pod across clusters
  - Service mesh limitations
  - Network policies require Calico

**Azure CNI (Advanced Networking):**
- Pods get IP from VNet subnet
- Direct pod connectivity
- No NAT overhead
- Full integration with Azure networking
- Requires more IP addresses

**Comparison:**
| Feature | Kubenet | Azure CNI |
|---------|---------|-----------|
| **IPs Required** | Few (only nodes) | Many (nodes + pods) |
| **Performance** | Good | Better |
| **Network Policies** | Requires Calico | Native support |
| **Service Endpoints** | Not supported | Supported |
| **Private Endpoints** | Not supported | Supported |
| **Complexity** | Simple | Complex |

**IP Planning:**
```bash
# Kubenet: (Nodes × 2) + 10 IPs
# Example: 10 nodes = 30 IPs needed

# Azure CNI: (Nodes × max-pods-per-node) + Nodes
# Example: 10 nodes × 30 pods = 310 IPs needed
```

**Creation:**
```bash
# Create with kubenet
az aks create \
  --resource-group rg-aks \
  --name aks-kubenet \
  --network-plugin kubenet \
  --node-count 3

# Create with Azure CNI
az aks create \
  --resource-group rg-aks \
  --name aks-azurecni \
  --network-plugin azure \
  --vnet-subnet-id /subscriptions/.../subnets/aks-subnet \
  --service-cidr 172.16.0.0/16 \
  --dns-service-ip 172.16.0.10 \
  --node-count 3
```

**When to Use:**
- **Kubenet**: Simple deployments, limited IP space, cost-sensitive
- **Azure CNI**: Production workloads, need VNet integration, service mesh, network policies

---

### **Q18: Explain AKS node pools and their use cases.** 🟡

**Answer:**
Node pools are groups of nodes with same configuration in an AKS cluster.

**Types:**
1. **System Node Pool**:
   - Runs critical system pods (CoreDNS, metrics-server)
   - Minimum 1 node (recommended 3)
   - Cannot be deleted
   - Taints: `CriticalAddonsOnly=true:NoSchedule`

2. **User Node Pools**:
   - Run application workloads
   - Can add/remove pools
   - Different VM sizes per pool
   - Can be scaled to 0

**Use Cases:**
```bash
# 1. Different VM sizes for different workloads
az aks nodepool add \
  --resource-group rg-aks \
  --cluster-name aks-prod \
  --name gpupool \
  --node-count 2 \
  --node-vm-size Standard_NC6s_v3 \
  --node-taints sku=gpu:NoSchedule \
  --labels workload=ml

# 2. Windows node pool for .NET apps
az aks nodepool add \
  --resource-group rg-aks \
  --cluster-name aks-prod \
  --name winpool \
  --node-count 3 \
  --os-type Windows

# 3. Spot instances for cost savings
az aks nodepool add \
  --resource-group rg-aks \
  --cluster-name aks-prod \
  --name spotpool \
  --priority Spot \
  --eviction-policy Delete \
  --spot-max-price -1 \
  --node-count 5

# 4. Availability zones for HA
az aks nodepool add \
  --resource-group rg-aks \
  --cluster-name aks-prod \
  --name hapool \
  --node-count 3 \
  --zones 1 2 3
```

**Best Practices:**
- Separate system and user pools
- Use taints and labels for workload isolation
- Different pools for different workload types
- Enable cluster autoscaler per pool

---

### **Q19: How do you implement zero-downtime deployments in AKS?** 🔴

**Answer:**
Several strategies for zero-downtime deployments:

**1. Rolling Updates (Default):**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1  # Max pods that can be unavailable
      maxSurge: 1        # Max extra pods during update
  template:
    spec:
      containers:
      - name: myapp
        image: myapp:v2
        readinessProbe:  # Critical for zero downtime
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
```

**2. Blue-Green Deployment:**
```bash
# Deploy green version
kubectl apply -f deployment-green.yaml

# Test green version
kubectl port-forward deployment/myapp-green 8080:80

# Switch traffic (update service selector)
kubectl patch service myapp -p '{"spec":{"selector":{"version":"green"}}}'

# Delete blue version after validation
kubectl delete deployment myapp-blue
```

**3. Canary Deployment (using Ingress):**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "10"  # 10% to canary
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-canary
            port:
              number: 80
```

**4. Using Helm:**
```bash
# Upgrade with rollback capability
helm upgrade myapp ./myapp-chart \
  --set image.tag=v2 \
  --wait \
  --timeout 5m

# Rollback if issues
helm rollback myapp
```

**5. Pre-stop Hook (Graceful Shutdown):**
```yaml
lifecycle:
  preStop:
    exec:
      command: ["/bin/sh", "-c", "sleep 15"]  # Allow load balancer to update
```

**Key Requirements:**
- Proper health checks (readiness/liveness)
- PodDisruptionBudget
- Resource requests/limits
- Graceful shutdown handling
- Connection draining

---

### **Q20: What is the AKS upgrade process and best practices?** 🔴

**Answer:**
AKS provides managed Kubernetes upgrades with control plane and node pool updates.

**Upgrade Components:**
1. **Control Plane**: Managed by Azure
2. **Node Pools**: Managed by you

**Upgrade Process:**
```bash
# 1. Check available versions
az aks get-upgrades \
  --resource-group rg-aks \
  --name aks-prod \
  --output table

# 2. Upgrade control plane first
az aks upgrade \
  --resource-group rg-aks \
  --name aks-prod \
  --kubernetes-version 1.28.0 \
  --control-plane-only

# 3. Upgrade node pools one by one
az aks nodepool upgrade \
  --resource-group rg-aks \
  --cluster-name aks-prod \
  --name systempool \
  --kubernetes-version 1.28.0

# 4. Verify cluster status
kubectl get nodes
kubectl get pods --all-namespaces
```

**Best Practices:**

**1. Maintenance Window:**
```bash
# Set maintenance window
az aks maintenanceconfiguration add \
  --resource-group rg-aks \
  --cluster-name aks-prod \
  --name default \
  --weekday Monday \
  --start-hour 2
```

**2. Test in Non-Production First:**
```bash
# Create test cluster with target version
az aks create \
  --resource-group rg-aks-test \
  --name aks-test \
  --kubernetes-version 1.28.0

# Deploy application and test
kubectl apply -f manifests/
```

**3. Use PodDisruptionBudgets:**
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: myapp-pdb
spec:
  minAvailable: 2  # Always keep 2 pods running
  selector:
    matchLabels:
      app: myapp
```

**4. Blue-Green Node Pool Upgrade:**
```bash
# Create new node pool with new version
az aks nodepool add \
  --resource-group rg-aks \
  --cluster-name aks-prod \
  --name newpool \
  --kubernetes-version 1.28.0 \
  --node-count 3

# Cordon old nodes
kubectl cordon <old-node-name>

# Drain workloads
kubectl drain <old-node-name> --ignore-daemonsets --delete-emptydir-data

# Delete old pool after validation
az aks nodepool delete \
  --resource-group rg-aks \
  --cluster-name aks-prod \
  --name oldpool
```

**5. Auto-upgrade Channels:**
```bash
# Enable auto-upgrade (patch version only)
az aks update \
  --resource-group rg-aks \
  --name aks-prod \
  --auto-upgrade-channel patch
```

**Upgrade Checklist:**
- [ ] Review release notes
- [ ] Test in non-production
- [ ] Backup cluster configurations
- [ ] Set PodDisruptionBudgets
- [ ] Schedule maintenance window
- [ ] Monitor during upgrade
- [ ] Validate application functionality
- [ ] Have rollback plan

---

## 🔐 SECURITY & IDENTITY

### **Q21: Explain managed identities in Azure.** 🟡

**Answer:**
Managed identities provide automatic credential management for Azure resources.

**Types:**

**1. System-Assigned Managed Identity:**
- Tied to resource lifecycle
- Deleted when resource is deleted
- 1:1 relationship with resource
- Cannot be shared

**2. User-Assigned Managed Identity:**
- Independent lifecycle
- Can be shared across resources
- Persists after resource deletion
- Reusable

**Use Cases:**
- Access Key Vault secrets
- Access Azure Storage
- Access Azure SQL
- Access ACR from AKS

**Implementation:**

```bash
# System-assigned for VM
az vm identity assign \
  --resource-group rg-compute \
  --name vm-web-01

# User-assigned identity
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
  --assignee <MANAGED_IDENTITY_PRINCIPAL_ID> \
  --role "Storage Blob Data Contributor" \
  --scope /subscriptions/{sub-id}/resourceGroups/rg-storage
```

**Application Code (Python):**
```python
from azure.identity import DefaultAzureCredential
from azure.storage.blob import BlobServiceClient

# Automatically uses managed identity
credential = DefaultAzureCredential()
blob_service = BlobServiceClient(
    account_url="https://stprod.blob.core.windows.net",
    credential=credential
)
```

**Benefits:**
- No credential management
- Automatic rotation
- No secrets in code
- Azure AD integrated
- Auditable

---

### **Q22: How do you secure secrets in Azure?** 🔴

**Answer:**
Multiple approaches for secret management:

**1. Azure Key Vault:**
```bash
# Create Key Vault
az keyvault create \
  --resource-group rg-security \
  --name kv-prod-secrets \
  --enable-soft-delete true \
  --enable-purge-protection true

# Store secret
az keyvault secret set \
  --vault-name kv-prod-secrets \
  --name db-password \
  --value "P@ssw0rd123!"

# Grant access via managed identity
az keyvault set-policy \
  --name kv-prod-secrets \
  --object-id <MANAGED_IDENTITY_ID> \
  --secret-permissions get list
```

**2. AKS Integration with Key Vault:**

**Using CSI Driver:**
```yaml
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: azure-keyvault-secrets
spec:
  provider: azure
  parameters:
    usePodIdentity: "true"
    keyvaultName: "kv-prod-secrets"
    objects: |
      array:
        - |
          objectName: db-password
          objectType: secret
          objectVersion: ""
    tenantId: "tenant-id"
---
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: myapp
    image: myapp:latest
    volumeMounts:
    - name: secrets-store
      mountPath: "/mnt/secrets-store"
      readOnly: true
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: db-password
  volumes:
  - name: secrets-store
    csi:
      driver: secrets-store.csi.k8s.io
      readOnly: true
      volumeAttributes:
        secretProviderClass: "azure-keyvault-secrets"
```

**3. Azure DevOps Variable Groups:**
```bash
# Link Key Vault to variable group
# Azure DevOps → Pipelines → Library → Variable Groups → Link secrets from Azure Key Vault
```

**4. GitHub Actions:**
```yaml
- name: Azure Login
  uses: azure/login@v1
  with:
    creds: ${{ secrets.AZURE_CREDENTIALS }}

- name: Get secrets from Key Vault
  uses: Azure/get-keyvault-secrets@v1
  with:
    keyvault: "kv-prod-secrets"
    secrets: 'db-password, api-key'
  id: keyvault

- name: Use secret
  run: |
    echo "Password retrieved"
  env:
    DB_PASSWORD: ${{ steps.keyvault.outputs.db-password }}
```

**Best Practices:**
- Never commit secrets to source control
- Use managed identities
- Enable soft delete and purge protection
- Rotate secrets regularly
- Audit secret access
- Use separate Key Vaults per environment
- Limit secret access with RBAC
- Enable diagnostic logging

---

## 📊 MONITORING & DEVOPS

### **Q23: How do you implement comprehensive monitoring in Azure?** 🔴

**Answer:**
Comprehensive monitoring requires multiple Azure services:

**1. Azure Monitor:**
- Centralized monitoring platform
- Collects metrics and logs
- Alerting and dashboards

**2. Log Analytics:**
```bash
# Create workspace
az monitor log-analytics workspace create \
  --resource-group rg-monitoring \
  --workspace-name law-prod \
  --location eastus

# Enable diagnostics for resource
az monitor diagnostic-settings create \
  --resource <RESOURCE_ID> \
  --name diag-settings \
  --workspace <WORKSPACE_ID> \
  --logs '[{"category": "Administrative","enabled": true}]' \
  --metrics '[{"category": "AllMetrics","enabled": true}]'

# Query logs
az monitor log-analytics query \
  --workspace <WORKSPACE_ID> \
  --analytics-query "
    AzureActivity
    | where TimeGenerated > ago(1h)
    | summarize count() by OperationName
  "
```

**3. Application Insights:**
```bash
# Create App Insights
az monitor app-insights component create \
  --resource-group rg-monitoring \
  --app ai-prod-app \
  --location eastus \
  --workspace <WORKSPACE_ID>

# Get instrumentation key
AI_KEY=$(az monitor app-insights component show \
  --resource-group rg-monitoring \
  --app ai-prod-app \
  --query instrumentationKey \
  --output tsv)
```

**4. Alerts:**
```bash
# Create action group
az monitor action-group create \
  --resource-group rg-monitoring \
  --name ag-devops \
  --short-name DevOps \
  --email-receiver name=Team email=devops@company.com

# Create metric alert
az monitor metrics alert create \
  --resource-group rg-monitoring \
  --name alert-high-cpu \
  --scopes <RESOURCE_ID> \
  --condition "avg Percentage CPU > 80" \
  --window-size 5m \
  --action ag-devops
```

**5. AKS Monitoring:**
```bash
# Enable Container Insights
az aks enable-addons \
  --resource-group rg-aks \
  --name aks-prod \
  --addons monitoring \
  --workspace-resource-id <WORKSPACE_ID>

# Query container logs
az monitor log-analytics query \
  --workspace <WORKSPACE_ID> \
  --analytics-query "
    ContainerLog
    | where ContainerName == 'myapp'
    | where TimeGenerated > ago(1h)
    | order by TimeGenerated desc
  "
```

**Monitoring Stack:**
```
Application Insights (APM)
    │
    ├─→ Distributed tracing
    ├─→ Performance monitoring
    └─→ Exception tracking

Log Analytics (Logs & Queries)
    │
    ├─→ VM logs
    ├─→ Container logs
    ├─→ Activity logs
    └─→ Custom logs

Azure Monitor (Metrics & Alerts)
    │
    ├─→ Platform metrics
    ├─→ Custom metrics
    ├─→ Alerts
    └─→ Dashboards

Workbooks (Visualization)
    └─→ Custom reports and dashboards
```

---

## 🎨 ARCHITECTURE & DESIGN

### **Q24: Design a highly available multi-region architecture on Azure.** 🔴

**Answer:**
High-level architecture for global application:

```
                        ┌──────────────────────┐
                        │   Traffic Manager    │
                        │  (Global DNS-based   │
                        │   Load Balancing)    │
                        └──────────┬───────────┘
                                   │
                   ┌───────────────┼───────────────┐
                   │                               │
          ┌────────▼─────────┐          ┌────────▼─────────┐
          │   East US Region │          │  West US Region  │
          │                  │          │                  │
          │  ┌────────────┐  │          │  ┌────────────┐  │
          │  │  App Gtw   │  │          │  │  App Gtw   │  │
          │  │   (WAF)    │  │          │  │   (WAF)    │  │
          │  └─────┬──────┘  │          │  └─────┬──────┘  │
          │        │         │          │        │         │
          │  ┌─────▼──────┐  │          │  ┌─────▼──────┐  │
          │  │    AKS     │  │          │  │    AKS     │  │
          │  │  Cluster   │  │          │  │  Cluster   │  │
          │  │  (3 Zones) │  │          │  │  (3 Zones) │  │
          │  └─────┬──────┘  │          │  └─────┬──────┘  │
          │        │         │          │        │         │
          │  ┌─────▼──────┐  │          │  ┌─────▼──────┐  │
          │  │  SQL       │◄─┼──────────┼─►│  SQL       │  │
          │  │ (Geo-Rep)  │  │          │  │ (Replica)  │  │
          │  └────────────┘  │          │  └────────────┘  │
          │                  │          │                  │
          │  ┌────────────┐  │          │  ┌────────────┐  │
          │  │   ACR      │◄─┼──────────┼─►│    ACR     │  │
          │  │(Geo-Rep)   │  │          │  │ (Replica)  │  │
          │  └────────────┘  │          │  └────────────┘  │
          │                  │          │                  │
          │  ┌────────────┐  │          │  ┌────────────┐  │
          │  │ Storage    │◄─┼──────────┼─►│  Storage   │  │
          │  │  (GRS)     │  │          │  │  (GRS)     │  │
          │  └────────────┘  │          │  └────────────┘  │
          └──────────────────┘          └──────────────────┘
```

**Implementation:**

```bash
# 1. Traffic Manager
az network traffic-manager profile create \
  --resource-group rg-global \
  --name tm-prod-app \
  --routing-method Performance \
  --unique-dns-name prod-app-global

# 2. Primary Region (East US)
az aks create \
  --resource-group rg-eastus \
  --name aks-eastus \
  --location eastus \
  --zones 1 2 3 \
  --node-count 3

# 3. Secondary Region (West US)
az aks create \
  --resource-group rg-westus \
  --name aks-westus \
  --location westus \
  --zones 1 2 3 \
  --node-count 3

# 4. Geo-replicated ACR
az acr create \
  --resource-group rg-eastus \
  --name acrprod \
  --sku Premium \
  --location eastus

az acr replication create \
  --registry acrprod \
  --location westus

# 5. SQL Database with failover group
az sql server create \
  --resource-group rg-eastus \
  --name sql-eastus \
  --location eastus

az sql server create \
  --resource-group rg-westus \
  --name sql-westus \
  --location westus

az sql failover-group create \
  --resource-group rg-eastus \
  --server sql-eastus \
  --name fg-prod-sql \
  --partner-server sql-westus \
  --failover-policy Automatic \
  --grace-period 1

# 6. Storage with GRS
az storage account create \
  --resource-group rg-eastus \
  --name stprodgrs \
  --sku Standard_GRS \
  --location eastus
```

**RTO/RPO:**
- **RTO (Recovery Time Objective)**: 5 minutes
- **RPO (Recovery Point Objective)**: Near zero

**Cost Optimization:**
- Active-Active vs Active-Passive
- Reserved instances
- Auto-scaling

---

### **Q25: Describe your CI/CD pipeline architecture for Azure.** 🔴

**Answer:**
End-to-end CI/CD pipeline for AKS deployment:

**Pipeline Stages:**
```
┌──────────────┐
│  Developer   │
│  Commits     │
└──────┬───────┘
       │
┌──────▼───────┐
│   Git Repo   │ (GitHub/Azure Repos)
└──────┬───────┘
       │ Trigger
┌──────▼───────────────────────────────────┐
│         CI Pipeline                      │
│  ┌────────────────────────────────────┐  │
│  │ 1. Code Checkout                   │  │
│  │ 2. Build & Unit Tests              │  │
│  │ 3. Code Quality (SonarQube)        │  │
│  │ 4. Security Scan (Trivy)           │  │
│  │ 5. Build Docker Image              │  │
│  │ 6. Push to ACR                     │  │
│  │ 7. Scan Image (Azure Defender)     │  │
│  │ 8. Publish Artifacts               │  │
│  └────────────────────────────────────┘  │
└──────┬───────────────────────────────────┘
       │
┌──────▼───────────────────────────────────┐
│         CD Pipeline                      │
│  ┌────────────────────────────────────┐  │
│  │ Dev Environment                    │  │
│  │  - Deploy to AKS-Dev              │  │
│  │  - Integration Tests              │  │
│  │  - Smoke Tests                    │  │
│  └────────────────────────────────────┘  │
│  ┌────────────────────────────────────┐  │
│  │ QA Environment                     │  │
│  │  - Deploy to AKS-QA               │  │
│  │  - E2E Tests                      │  │
│  │  - Performance Tests              │  │
│  └────────────────────────────────────┘  │
│  ┌────────────────────────────────────┐  │
│  │ Production Environment             │  │
│  │  - Manual Approval                │  │
│  │  - Blue-Green Deployment          │  │
│  │  - Health Checks                  │  │
│  │  - Monitoring & Alerts            │  │
│  └────────────────────────────────────┘  │
└──────────────────────────────────────────┘
```

**Azure DevOps YAML:**
```yaml
trigger:
  branches:
    include:
    - main
  paths:
    include:
    - src/**

pool:
  vmImage: 'ubuntu-latest'

variables:
  dockerRegistryServiceConnection: 'ACR-Connection'
  imageRepository: 'myapp'
  containerRegistry: 'acrprod.azurecr.io'
  dockerfilePath: 'Dockerfile'
  tag: '$(Build.BuildId)'

stages:
- stage: Build
  displayName: 'Build and Push'
  jobs:
  - job: Build
    steps:
    - task: SonarQubePrepare@5
      inputs:
        SonarQube: 'SonarQube-Connection'
        scannerMode: 'CLI'

    - task: Docker@2
      displayName: 'Build Docker Image'
      inputs:
        command: build
        repository: $(imageRepository)
        dockerfile: $(dockerfilePath)
        tags: |
          $(tag)
          latest

    - task: Trivy@1
      displayName: 'Security Scan'
      inputs:
        image: '$(containerRegistry)/$(imageRepository):$(tag)'
        exitCode: '1'
        severity: 'CRITICAL,HIGH'

    - task: Docker@2
      displayName: 'Push to ACR'
      inputs:
        command: push
        repository: $(imageRepository)
        containerRegistry: $(dockerRegistryServiceConnection)
        tags: |
          $(tag)
          latest

- stage: Deploy_Dev
  displayName: 'Deploy to Dev'
  dependsOn: Build
  jobs:
  - deployment: DeployDev
    environment: 'dev'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: KubernetesManifest@0
            displayName: 'Deploy to AKS'
            inputs:
              action: 'deploy'
              kubernetesServiceConnection: 'AKS-Dev'
              namespace: 'dev'
              manifests: |
                manifests/deployment.yaml
                manifests/service.yaml
              containers: |
                $(containerRegistry)/$(imageRepository):$(tag)

- stage: Deploy_Prod
  displayName: 'Deploy to Production'
  dependsOn: Deploy_Dev
  jobs:
  - deployment: DeployProd
    environment: 'production'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: Kubernetes@1
            displayName: 'Blue-Green Deployment'
            inputs:
              connectionType: 'Kubernetes Service Connection'
              kubernetesServiceEndpoint: 'AKS-Prod'
              command: 'apply'
              useConfigurationFile: true
              configuration: 'manifests/'
              arguments: '-f manifests/deployment.yaml'
```

**Key Components:**
- Source control (Git)
- Build automation (Azure Pipelines)
- Container registry (ACR)
- Security scanning (Trivy, Defender)
- Quality gates (SonarQube)
- Infrastructure as Code (Helm/Kustomize)
- Monitoring (Application Insights)
- Secrets management (Key Vault)

---

**Continue with remaining 95+ questions covering all Azure topics...**

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

