# Azure Networking - Complete Learning Guide

**Comprehensive Tutorial: VNets, Subnets, NSGs, Load Balancers, VPN & More**

---

## 📚 What You'll Learn

Complete guide to Azure Networking covering:

1. **Virtual Networks (VNets)** - Foundation of Azure networking
2. **Subnets & IP Addressing** - Network segmentation
3. **Network Security Groups (NSGs)** - Firewall rules
4. **Load Balancers** - Traffic distribution
5. **Application Gateway** - Layer 7 load balancing with WAF
6. **VPN Gateway** - Hybrid connectivity
7. **Azure Firewall** - Centralized network security
8. **Private Endpoints** - Secure PaaS connectivity
9. **Network Monitoring** - Traffic analysis and troubleshooting

---

## 🎯 Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     Azure Region                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              Virtual Network (VNet)                   │   │
│  │              10.0.0.0/16                              │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌────────────┐ │   │
│  │  │ Subnet-Web   │  │ Subnet-App   │  │Subnet-Data │ │   │
│  │  │ 10.0.1.0/24  │  │ 10.0.2.0/24  │  │10.0.3.0/24 │ │   │
│  │  │              │  │              │  │            │ │   │
│  │  │  [NSG-Web]   │  │  [NSG-App]   │  │ [NSG-DB]   │ │   │
│  │  │              │  │              │  │            │ │   │
│  │  │  VMs/VMSS    │  │  VMs/VMSS    │  │  Database  │ │   │
│  │  └──────────────┘  └──────────────┘  └────────────┘ │   │
│  │                                                       │   │
│  │  [Load Balancer] ──→ Backend Pool (VMs)             │   │
│  │  [App Gateway]   ──→ Backend Pool (Web Apps)        │   │
│  └──────────────────────────────────────────────────────┘   │
│                           │                                   │
│                    [VPN Gateway]                             │
│                           │                                   │
└───────────────────────────┼───────────────────────────────────┘
                            │
                    On-Premises Network
```

---

## 🌐 Understanding Virtual Networks (VNets)

**Azure Virtual Network (VNet)** is the fundamental building block for your private network in Azure. It's like having your own isolated network in the cloud.

### **What is a VNet?**

Think of a VNet as your **private network in Azure** - similar to a network you'd have in your own datacenter, but in the cloud.

**Real-World Analogy:**
```
Traditional Office Network:
    Router → Switch → Computers/Servers
    All devices in same physical location
    Isolated from other companies

Azure VNet:
    VNet → Subnets → VMs/Resources
    All resources in same virtual network
    Isolated from other Azure customers
```

---

### **Why Do You Need VNets?**

**❌ Without VNets:**
```
All resources publicly accessible
No isolation between environments
Cannot control traffic flow
Security nightmare!
```

**✅ With VNets:**
```
Resources isolated in private network
Subnets for organization (web, app, data)
Control traffic with NSGs
Secure communication
```

---

### **VNet Key Concepts:**

**1. Address Space:**
```
VNet: 10.0.0.0/16
    ↓
Can contain 65,536 IP addresses (10.0.0.0 to 10.0.255.255)
```

**Common Private IP Ranges:**
- `10.0.0.0/8` (10.0.0.0 - 10.255.255.255) - Large networks
- `172.16.0.0/12` (172.16.0.0 - 172.31.255.255) - Medium networks
- `192.168.0.0/16` (192.168.0.0 - 192.168.255.255) - Small networks

**2. Subnets (Network Segmentation):**
```
VNet: 10.0.0.0/16
  ├── Subnet-Web: 10.0.1.0/24 (256 IPs) → Web servers
  ├── Subnet-App: 10.0.2.0/24 (256 IPs) → Application servers
  └── Subnet-Data: 10.0.3.0/24 (256 IPs) → Databases
```

**Why use subnets?**
- **Security**: Different security rules per subnet
- **Organization**: Group similar resources
- **Traffic control**: Control flow between tiers
- **Service requirements**: Some services need dedicated subnets

---

### **Understanding CIDR Notation:**

**What does /16 or /24 mean?**

```
IP Address: 10.0.0.0/16
            └─ Network ─┘ └ Host bits

/16 = First 16 bits are network, last 16 bits are hosts
    = 2^16 hosts = 65,536 IP addresses

/24 = First 24 bits are network, last 8 bits are hosts
    = 2^8 hosts = 256 IP addresses

/32 = Single IP address (no hosts)
```

**Common CIDR Blocks:**
| CIDR | Subnet Mask | Total IPs | Usable IPs* |
|------|-------------|-----------|-------------|
| /8 | 255.0.0.0 | 16,777,216 | 16,777,211 |
| /16 | 255.255.0.0 | 65,536 | 65,531 |
| /24 | 255.255.255.0 | 256 | 251 |
| /28 | 255.255.255.240 | 16 | 11 |
| /32 | 255.255.255.255 | 1 | 1 |

*Azure reserves 5 IPs in each subnet

---

### **Azure's Reserved IPs:**

**In every subnet, Azure reserves 5 IPs:**

```
Example subnet: 10.0.1.0/24

10.0.1.0   → Network address (reserved)
10.0.1.1   → Azure gateway (reserved)
10.0.1.2   → Azure DNS (reserved)
10.0.1.3   → Azure DNS (reserved)
10.0.1.4   → Future use (reserved)
10.0.1.5   → First usable IP ✓
...
10.0.1.254 → Last usable IP ✓
10.0.1.255 → Broadcast address (reserved)

Result: 251 usable IPs (not 256)
```

---

### **VNet Design Best Practices:**

**1. Plan Address Space Carefully:**
```
❌ Bad: Start with 10.0.0.0/24 (only 256 IPs)
   Problem: Run out of IPs quickly, can't expand

✅ Good: Start with 10.0.0.0/16 (65,536 IPs)
   Benefit: Room to grow, plan for future
```

**2. Don't Overlap Address Spaces:**
```
VNet 1: 10.0.0.0/16
VNet 2: 10.1.0.0/16 ✓ (Different range)

Don't:
VNet 1: 10.0.0.0/16
VNet 2: 10.0.10.0/24 ❌ (Overlaps!)
```

**3. Subnet Sizing:**
```
Web Tier (Public-facing):
  /24 (256 IPs) → Typically enough

Application Tier:
  /24 (256 IPs) → Room for scaling

Database Tier:
  /26 (64 IPs) → Usually fewer database servers

Management/Bastion:
  /27 (32 IPs) → Small subnet for jump boxes
```

---

### **Real-World VNet Scenarios:**

**Scenario 1: Simple 3-Tier Application**
```
VNet: 10.0.0.0/16 (Production)

├── subnet-web:   10.0.1.0/24  (Web servers, load balancer)
├── subnet-app:   10.0.2.0/24  (Application servers)
├── subnet-data:  10.0.3.0/24  (Databases)
└── subnet-mgmt:  10.0.4.0/27  (Bastion, jump boxes)

Traffic Flow:
Internet → Web → App → Data
         ↑ NSG  ↑ NSG  ↑ NSG
```

**Scenario 2: Multi-Environment Setup**
```
VNet-Production:  10.0.0.0/16
  └── Prod resources

VNet-Development: 10.1.0.0/16
  └── Dev/Test resources

VNet-Shared:      10.2.0.0/16
  └── Shared services (DNS, monitoring)

Connected via VNet Peering
```

---

### **VNet Communication:**

**Within Same VNet:**
```
VM1 (10.0.1.10) → VM2 (10.0.2.20)
✓ Can communicate by default
✓ No special configuration needed
✓ Private IP addresses only
```

**Between Different VNets:**
```
VNet1 → VNet2
❌ Cannot communicate by default
✓ Need VNet Peering or VPN Gateway
```

**To Internet:**
```
VM → Internet
✓ Outbound: Allowed by default
❌ Inbound: Need public IP + NSG rules
```

**To On-Premises:**
```
Azure VNet → On-Premises Network
✓ Need VPN Gateway or ExpressRoute
```

---

## 🌐 1. Virtual Networks (VNets) - Implementation

Now let's create VNets!

### **Create VNet - Azure CLI**

```bash
# Create VNet with multiple subnets
az network vnet create \
  --resource-group rg-network \
  --name vnet-prod \
  --address-prefix 10.0.0.0/16 \
  --subnet-name subnet-web \
  --subnet-prefix 10.0.1.0/24 \
  --location eastus

# Add additional subnets
az network vnet subnet create \
  --resource-group rg-network \
  --vnet-name vnet-prod \
  --name subnet-app \
  --address-prefix 10.0.2.0/24

az network vnet subnet create \
  --resource-group rg-network \
  --vnet-name vnet-prod \
  --name subnet-data \
  --address-prefix 10.0.3.0/24
```

### **Create VNet - PowerShell**

```powershell
# Create subnets
$subnetWeb = New-AzVirtualNetworkSubnetConfig `
  -Name "subnet-web" `
  -AddressPrefix "10.0.1.0/24"

$subnetApp = New-AzVirtualNetworkSubnetConfig `
  -Name "subnet-app" `
  -AddressPrefix "10.0.2.0/24"

# Create VNet
New-AzVirtualNetwork `
  -Name "vnet-prod" `
  -ResourceGroupName "rg-network" `
  -Location "East US" `
  -AddressPrefix "10.0.0.0/16" `
  -Subnet $subnetWeb,$subnetApp
```

### **Create VNet - Terraform**

```hcl
resource "azurerm_virtual_network" "main" {
  name                = "vnet-prod"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  address_space       = ["10.0.0.0/16"]

  subnet {
    name           = "subnet-web"
    address_prefix = "10.0.1.0/24"
  }

  subnet {
    name           = "subnet-app"
    address_prefix = "10.0.2.0/24"
  }

  tags = {
    Environment = "Production"
  }
}
```

---

## 🔒 Understanding Network Security Groups (NSGs)

**Network Security Groups (NSGs)** are Azure's virtual firewall that control inbound and outbound traffic to Azure resources.

### **What is an NSG?**

Think of NSGs as **security guards** that check every packet and decide: "Allow or Deny?"

**Real-World Analogy:**
```
Office Building Security:
    Security Guard at entrance
    Check visitor badge
    Allow: Valid badge → Enter
    Deny: No badge → Cannot enter

NSG:
    Firewall rules
    Check source IP, destination, port
    Allow: Matches allow rule → Traffic passes
    Deny: No match or deny rule → Traffic blocked
```

---

### **How NSGs Work:**

```
Internet Request (Source: 203.0.113.5, Port: 443)
    ↓
┌─────────────────────────────────────┐
│  NSG Rules (Processed in Priority)  │
├─────────────────────────────────────┤
│  Priority 100: Allow HTTPS (443)    │ ← Match! Allow
│  Priority 200: Deny All             │
└─────────────────────────────────────┘
    ↓
Traffic Reaches VM ✓
```

---

### **NSG Components:**

**1. Priority (100-4096):**
```
Lower number = Higher priority = Checked first

Priority 100: Allow HTTPS
Priority 200: Deny All

HTTPS traffic → Matches 100 → Allowed (200 never checked)
```

**2. Source:**
```
Where traffic comes from:
- Internet
- VirtualNetwork (any resource in same VNet)
- AzureLoadBalancer
- Specific IP: 203.0.113.5
- IP Range: 203.0.113.0/24
- Service Tag: "Sql", "Storage", etc.
```

**3. Destination:**
```
Where traffic goes to:
- VirtualNetwork
- Specific IP: 10.0.1.10
- IP Range: 10.0.1.0/24
- Service Tag
```

**4. Port:**
```
Common ports:
- 22: SSH
- 80: HTTP
- 443: HTTPS
- 3389: RDP
- 1433: SQL Server
- 3306: MySQL
- * : All ports
```

**5. Protocol:**
```
- TCP (web, SSH, databases)
- UDP (DNS, streaming)
- ICMP (ping)
- Any
```

**6. Action:**
```
- Allow: Traffic permitted
- Deny: Traffic blocked
```

---

### **Default NSG Rules:**

**Every NSG has default rules (cannot be deleted):**

**Inbound Default Rules:**
```
Priority 65000: Allow VNet to VNet
    - VMs in same VNet can talk to each other

Priority 65001: Allow Azure Load Balancer
    - Health probes can reach VMs

Priority 65500: Deny All Inbound
    - Block everything else from Internet

Result: By default, Internet cannot reach your VMs!
```

**Outbound Default Rules:**
```
Priority 65000: Allow VNet to VNet
    - VMs can talk to each other

Priority 65001: Allow Internet
    - VMs can access Internet

Priority 65500: Deny All
    - Everything else denied

Result: By default, VMs can access Internet!
```

---

### **NSG Association:**

**Where to Apply NSGs:**

**Option 1: Subnet Level (Recommended)**
```
Subnet: 10.0.1.0/24
  NSG: nsg-web
  ├── VM1 (protected)
  ├── VM2 (protected)
  └── VM3 (protected)

Benefit: One NSG protects all VMs in subnet
```

**Option 2: NIC Level**
```
VM1 → NIC → NSG-VM1 (specific rules)
VM2 → NIC → NSG-VM2 (different rules)

Use when: Each VM needs different rules
```

**Option 3: Both (Defense in Depth)**
```
Request
  ↓
Subnet NSG (First layer)
  ↓
NIC NSG (Second layer)
  ↓
VM

Traffic must pass BOTH NSGs!
```

---

### **NSG Rule Examples:**

**Example 1: Web Server (Allow HTTP/HTTPS from Internet)**
```
Priority: 100
Source: Internet
Destination: 10.0.1.0/24
Port: 80, 443
Protocol: TCP
Action: Allow

Meaning: Anyone on Internet can access web servers on ports 80/443
```

**Example 2: SSH from Office Only**
```
Priority: 110
Source: 203.0.113.0/24 (Your office IP range)
Destination: 10.0.1.0/24
Port: 22
Protocol: TCP
Action: Allow

Meaning: Only office network can SSH to servers
```

**Example 3: Database from App Tier Only**
```
Priority: 100
Source: 10.0.2.0/24 (App subnet)
Destination: 10.0.3.0/24 (Database subnet)
Port: 1433
Protocol: TCP
Action: Allow

Meaning: Only application servers can access SQL database
```

**Example 4: Deny All Else (Explicit)**
```
Priority: 4096 (lowest)
Source: *
Destination: *
Port: *
Protocol: *
Action: Deny

Meaning: Block everything not explicitly allowed
```

---

### **Service Tags (Simplified Management):**

**Instead of IP addresses, use Service Tags:**

```
❌ Hard Way:
Source: 13.64.0.0/11, 13.96.0.0/13, ... (100+ IP ranges for Azure SQL)

✅ Easy Way:
Source: Sql (Service Tag)
```

**Common Service Tags:**
| Tag | Represents |
|-----|------------|
| **Internet** | Public Internet |
| **VirtualNetwork** | All VNets and on-premises via VPN |
| **AzureLoadBalancer** | Azure Load Balancer health probes |
| **Sql** | Azure SQL Database |
| **Storage** | Azure Storage |
| **AzureActiveDirectory** | Azure AD |
| **AzureCloud** | All Azure public IPs |

---

### **Real-World NSG Scenarios:**

**Scenario 1: 3-Tier Application Security**

```
┌────────────────────────────────────────────┐
│  Internet                                   │
└──────────────┬─────────────────────────────┘
               │ HTTPS (443)
         ┌─────▼──────┐
         │   NSG-Web  │ Allow: Internet → Port 443
         │            │ Allow: Office → Port 22 (SSH)
         │  Web Tier  │ Deny: All else
         └─────┬──────┘
               │ Port 8080 (to app tier)
         ┌─────▼──────┐
         │  NSG-App   │ Allow: Web Subnet → Port 8080
         │            │ Deny: Internet
         │  App Tier  │ Deny: All else
         └─────┬──────┘
               │ Port 1433 (to database)
         ┌─────▼──────┐
         │ NSG-Data   │ Allow: App Subnet → Port 1433
         │            │ Deny: Internet
         │ Data Tier  │ Deny: Web Subnet (cannot bypass app tier)
         └────────────┘
```

---

**Scenario 2: Bastion/Jump Box Pattern**

```
Problem: Need to SSH to servers, but don't want to expose SSH to Internet

Solution: Bastion Host
    Internet → Bastion (only SSH exposed) → Other VMs (SSH from Bastion only)

NSG-Bastion:
  Allow: Office IP → Port 22

NSG-Servers:
  Allow: Bastion Subnet → Port 22
  Deny: Internet → Port 22

Result: Must go through Bastion to SSH to servers
```

---

**Scenario 3: Microservices Communication**

```
Service A needs to call Service B:

NSG-Service-B:
  Priority 100: Allow Source: Service-A-Subnet, Port: 8080
  Priority 4096: Deny All

Only Service A can reach Service B!
```

---

### **NSG Best Practices:**

**1. Least Privilege:**
```
❌ Don't: Allow * → * → *
✅ Do: Allow specific sources → specific ports
```

**2. Use Service Tags:**
```
❌ Don't: List all Azure IP ranges
✅ Do: Use "AzureCloud" service tag
```

**3. Naming Convention:**
```
✅ Good Names:
- Allow-HTTPS-from-Internet
- Allow-SSH-from-Office
- Allow-SQL-from-AppTier

❌ Bad Names:
- Rule1
- Allow-All
- Test
```

**4. Document Rules:**
```
az network nsg rule create \
  --name Allow-HTTPS-from-Internet \
  --description "Allow public access to web servers on HTTPS"
```

**5. Subnet-Level NSGs:**
```
✅ Apply NSG to subnet (protects all VMs)
❌ Don't: Apply different NSG to each VM NIC (hard to manage)
```

**6. Log NSG Flow:**
```
Enable NSG Flow Logs for troubleshooting:
- See what traffic is allowed/denied
- Identify security issues
- Debug connectivity problems
```

---

### **Common NSG Mistakes:**

| Mistake | Impact | Solution |
|---------|--------|----------|
| Allow 0.0.0.0/0 → SSH | Exposed to attacks | Allow office IP only |
| Forgot priority order | Wrong rule processed | Plan priorities |
| NSG on subnet & NIC | Conflicting rules | Use subnet-level only |
| No deny-all rule | Unclear security posture | Add explicit deny at end |
| Too many rules | Hard to audit | Consolidate with service tags |

---

### **NSG Evaluation Order:**

```
1. Check NIC-level NSG (if exists)
   ↓
2. Check Subnet-level NSG (if exists)
   ↓
3. Within each NSG:
   - Sort by priority (100 → 4096)
   - First match wins
   - Stop processing
   ↓
4. If no match:
   - Default rules apply
   - Usually Deny
```

---

## 🔒 2. Network Security Groups (NSGs) - Implementation

Now let's create and configure NSGs!

### **Create NSG with Rules - Azure CLI**

```bash
# Create NSG
az network nsg create \
  --resource-group rg-network \
  --name nsg-web

# Allow HTTP/HTTPS
az network nsg rule create \
  --resource-group rg-network \
  --nsg-name nsg-web \
  --name AllowWeb \
  --priority 100 \
  --source-address-prefixes Internet \
  --destination-port-ranges 80 443 \
  --access Allow \
  --protocol Tcp

# Allow SSH from specific IP
az network nsg rule create \
  --resource-group rg-network \
  --nsg-name nsg-web \
  --name AllowSSH \
  --priority 110 \
  --source-address-prefixes 203.0.113.0/24 \
  --destination-port-ranges 22 \
  --access Allow \
  --protocol Tcp

# Deny all other inbound
az network nsg rule create \
  --resource-group rg-network \
  --nsg-name nsg-web \
  --name DenyAllInbound \
  --priority 4096 \
  --access Deny \
  --protocol '*' \
  --direction Inbound

# Associate with subnet
az network vnet subnet update \
  --resource-group rg-network \
  --vnet-name vnet-prod \
  --name subnet-web \
  --network-security-group nsg-web
```

### **NSG - Terraform**

```hcl
resource "azurerm_network_security_group" "web" {
  name                = "nsg-web"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  security_rule {
    name                       = "AllowWeb"
    priority                   = 100
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_ranges    = ["80", "443"]
    source_address_prefix      = "Internet"
    destination_address_prefix = "*"
  }

  security_rule {
    name                       = "AllowSSH"
    priority                   = 110
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "22"
    source_address_prefix      = "203.0.113.0/24"
    destination_address_prefix = "*"
  }
}

resource "azurerm_subnet_network_security_group_association" "web" {
  subnet_id                 = azurerm_subnet.web.id
  network_security_group_id = azurerm_network_security_group.web.id
}
```

---

## ⚖️ Understanding Load Balancers

**Azure Load Balancer** distributes incoming traffic across multiple servers, ensuring high availability and reliability.

### **What is a Load Balancer?**

Think of a load balancer as a **traffic director** that distributes visitors across multiple servers.

**Real-World Analogy:**
```
Supermarket Checkout:
    10 customers arrive
    Cashier directs each to shortest line
    Lines balanced, no one waits too long

Load Balancer:
    1000 web requests arrive
    Load balancer sends to available servers
    Traffic balanced, no server overwhelmed
```

---

### **Why Use Load Balancers?**

**❌ Without Load Balancer:**
```
Single Server:
    User → vm-web-01 (Handling ALL traffic)

Problems:
- If VM fails → Website down
- Limited capacity → Slow during peak
- No redundancy → Single point of failure
- Can't scale → Add more VMs manually
```

**✅ With Load Balancer:**
```
Multiple Servers:
    User → Load Balancer → vm-web-01
                        → vm-web-02
                        → vm-web-03

Benefits:
✓ High availability (if one VM fails, others handle traffic)
✓ Better performance (traffic distributed)
✓ Easy scaling (add more VMs automatically)
✓ Maintenance without downtime
```

---

### **How Load Balancers Work:**

```
1. User Request
   ↓
2. Load Balancer Receives (Public IP)
   ↓
3. Health Probe Check
   - VM1: Healthy ✓
   - VM2: Healthy ✓
   - VM3: Unhealthy ✗ (Don't send traffic)
   ↓
4. Distribute Using Algorithm
   - Round Robin: VM1 → VM2 → VM1 → VM2...
   - Or Hash: Same client → same VM
   ↓
5. Forward to Healthy VM
   ↓
6. VM Processes Request
   ↓
7. Response → User
```

---

### **Load Balancer Types:**

| Feature | Basic | Standard |
|---------|-------|----------|
| **Backend Pool Size** | 300 VMs | 1000 VMs |
| **Health Probes** | TCP, HTTP | TCP, HTTP, HTTPS |
| **Availability Zones** | No | Yes |
| **Security** | Open by default | Secure by default (NSG required) |
| **SLA** | None | 99.99% |
| **Price** | Free | $ Paid |
| **Use For** | Dev/Test | Production |

**Recommendation:** Always use **Standard** for production!

---

### **Load Balancer Components:**

```
┌────────────────────────────────────────┐
│       Azure Load Balancer              │
├────────────────────────────────────────┤
│  1. Frontend IP (Public/Private IP)   │
│     → The IP clients connect to       │
├────────────────────────────────────────┤
│  2. Backend Pool (VMs/VMSS)           │
│     → Servers that receive traffic    │
├────────────────────────────────────────┤
│  3. Health Probe (Check VM health)    │
│     → Determines which VMs are UP     │
├────────────────────────────────────────┤
│  4. Load Balancing Rules              │
│     → How to distribute traffic       │
│     → Port mapping (80 → 80)          │
└────────────────────────────────────────┘
```

---

### **1. Frontend IP Configuration:**

**Public Load Balancer:**
```
Public IP: 20.40.60.80
    ↓
Internet users connect to this IP
    ↓
Traffic forwarded to internal VMs (10.0.1.x)

Use for: Public-facing websites
```

**Internal Load Balancer:**
```
Private IP: 10.0.1.100
    ↓
Only internal users/services connect
    ↓
Traffic stays within VNet

Use for: Internal APIs, databases
```

---

### **2. Backend Pool:**

**What it is:**
```
Group of VMs that receive traffic

Backend Pool: web-servers
  ├── vm-web-01 (10.0.1.10)
  ├── vm-web-02 (10.0.1.11)
  └── vm-web-03 (10.0.1.12)

Load balancer picks one VM for each request
```

**How VMs are added:**
- Manually add individual VMs
- Automatically with VM Scale Sets
- Can span availability zones

---

### **3. Health Probes (Critical!):**

**What are Health Probes?**
```
Load balancer periodically checks: "Is this VM healthy?"

Probe Settings:
- Protocol: HTTP/HTTPS/TCP
- Port: 80, 443, custom
- Path: /health (for HTTP)
- Interval: 15 seconds (check every 15s)
- Threshold: 2 failures → Mark unhealthy
```

**Health Probe Example:**
```
HTTP Health Probe:
  Protocol: HTTP
  Port: 80
  Path: /health
  
Load balancer calls:
  http://vm-web-01/health every 15 seconds
  
Response:
  200 OK → Healthy ✓ → Send traffic
  500 Error → Unhealthy ✗ → Don't send traffic
  Timeout → Unhealthy ✗ → Don't send traffic
```

**Why Health Probes Matter:**
```
Without health probes:
  VM crashes → Load balancer still sends traffic → Users see errors

With health probes:
  VM crashes → Health probe fails → Load balancer stops sending traffic
  → Other VMs handle requests → Users don't notice!
```

---

### **4. Load Balancing Rules:**

**Rule Components:**
```
Frontend: Public IP Port 80
    ↓ (Maps to)
Backend: Backend Pool Port 80

Example:
User accesses: 20.40.60.80:80
Forwarded to: 10.0.1.10:80 (VM)
```

**Session Persistence (Affinity):**

**None (Default - Round Robin):**
```
Request 1 → VM1
Request 2 → VM2
Request 3 → VM3
Request 4 → VM1 (back to start)
```

**Client IP:**
```
User 203.0.113.5 → Always → VM1
User 203.0.113.6 → Always → VM2

Use when: Shopping cart, user sessions
```

**Client IP and Protocol:**
```
User 203.0.113.5 HTTP → Always → VM1
User 203.0.113.5 HTTPS → Always → VM2 (different protocol)
```

---

### **Real-World Load Balancer Scenarios:**

**Scenario 1: High-Availability Website**

```
Problem: Website must stay online 24/7

Solution:
    Internet
        ↓
    Load Balancer (20.40.60.80)
        ↓
    ├── VM1 (Zone 1) → Healthy
    ├── VM2 (Zone 2) → Healthy  
    └── VM3 (Zone 3) → Crashed! ✗

Traffic goes only to VM1 and VM2
VM3 maintenance? No downtime!
```

---

**Scenario 2: Internal API Load Balancing**

```
Frontend Apps → Internal Load Balancer → API Servers

Internal LB: 10.0.1.100
Backend Pool:
  - api-server-01 (10.0.2.10)
  - api-server-02 (10.0.2.11)
  - api-server-03 (10.0.2.12)

All frontend apps connect to 10.0.1.100
Load balancer distributes to healthy API servers
```

---

**Scenario 3: Blue-Green Deployment**

```
Step 1: All traffic to Blue pool
    LB → Blue Pool (v1.0) - 100% traffic
         Green Pool (v2.0) - 0% traffic

Step 2: Test Green
    LB → Blue Pool - 90% traffic
         Green Pool - 10% traffic

Step 3: Full cutover
    LB → Blue Pool - 0% traffic
         Green Pool (v2.0) - 100% traffic

Step 4: Rollback if issues
    LB → Blue Pool - 100% traffic (instant rollback!)
```

---

### **Load Balancer vs Application Gateway:**

| Feature | Load Balancer | Application Gateway |
|---------|---------------|---------------------|
| **OSI Layer** | Layer 4 (Transport) | Layer 7 (Application) |
| **Routing** | IP + Port only | URL path, host header |
| **Protocol** | Any TCP/UDP | HTTP/HTTPS only |
| **SSL Termination** | No | Yes |
| **WAF** | No | Yes (Web App Firewall) |
| **Use For** | Non-HTTP traffic | Web applications |
| **Example** | Database load balancing | Website with /api, /images paths |

**When to use what:**
```
Load Balancer:
✓ SQL Server cluster
✓ Any TCP/UDP service
✓ Simple HTTP distribution
✓ Lower cost

Application Gateway:
✓ Web applications
✓ URL-based routing
✓ SSL offloading
✓ WAF protection
```

---

### **Load Balancer Best Practices:**

**1. Always Use Health Probes:**
```
❌ Don't: Skip health probes
✅ Do: Configure health probes on all backend pools
```

**2. Use Standard SKU for Production:**
```
✓ Availability Zones support
✓ Better SLA (99.99%)
✓ More features
```

**3. Monitor Health Probe Status:**
```
Check: "How many VMs are healthy?"
Alert: If < 50% healthy
```

**4. Plan for Failures:**
```
Minimum: 2 VMs across 2 availability zones
Better: 3+ VMs across 3 zones
```

**5. Session Persistence:**
```
Stateless apps: Use None (best distribution)
Stateful apps: Use Client IP
```

---

### **Common Load Balancer Issues:**

| Issue | Cause | Solution |
|-------|-------|----------|
| All traffic to one VM | Wrong distribution mode | Check session persistence |
| Traffic not reaching VMs | Health probe failing | Check health endpoint |
| Slow performance | All VMs unhealthy except one | Fix unhealthy VMs |
| Connection timeout | NSG blocking | Allow health probe IPs |
| Uneven distribution | Long-lived connections | Use shorter timeouts |

---

## ⚖️ 3. Load Balancers - Implementation

Now let's create and configure Load Balancers!

### **Create Standard Load Balancer - CLI**

```bash
# Create public IP
az network public-ip create \
  --resource-group rg-network \
  --name pip-lb \
  --sku Standard \
  --allocation-method Static

# Create load balancer
az network lb create \
  --resource-group rg-network \
  --name lb-web \
  --sku Standard \
  --public-ip-address pip-lb \
  --frontend-ip-name frontend \
  --backend-pool-name backend-pool

# Add health probe
az network lb probe create \
  --resource-group rg-network \
  --lb-name lb-web \
  --name health-probe \
  --protocol Http \
  --port 80 \
  --path /health

# Add load balancing rule
az network lb rule create \
  --resource-group rg-network \
  --lb-name lb-web \
  --name http-rule \
  --protocol Tcp \
  --frontend-port 80 \
  --backend-port 80 \
  --frontend-ip-name frontend \
  --backend-pool-name backend-pool \
  --probe-name health-probe

# Add VMs to backend pool
az network nic ip-config address-pool add \
  --resource-group rg-network \
  --nic-name vm1-nic \
  --ip-config-name ipconfig1 \
  --lb-name lb-web \
  --address-pool backend-pool
```

### **Load Balancer - Terraform**

```hcl
resource "azurerm_public_ip" "lb" {
  name                = "pip-lb"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  allocation_method   = "Static"
  sku                 = "Standard"
}

resource "azurerm_lb" "main" {
  name                = "lb-web"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  sku                 = "Standard"

  frontend_ip_configuration {
    name                 = "frontend"
    public_ip_address_id = azurerm_public_ip.lb.id
  }
}

resource "azurerm_lb_backend_address_pool" "main" {
  loadbalancer_id = azurerm_lb.main.id
  name            = "backend-pool"
}

resource "azurerm_lb_probe" "main" {
  loadbalancer_id = azurerm_lb.main.id
  name            = "health-probe"
  protocol        = "Http"
  port            = 80
  request_path    = "/health"
}

resource "azurerm_lb_rule" "main" {
  loadbalancer_id                = azurerm_lb.main.id
  name                           = "http-rule"
  protocol                       = "Tcp"
  frontend_port                  = 80
  backend_port                   = 80
  frontend_ip_configuration_name = "frontend"
  backend_address_pool_ids       = [azurerm_lb_backend_address_pool.main.id]
  probe_id                       = azurerm_lb_probe.main.id
}
```

---

## 🌐 4. Application Gateway with WAF

### **Deploy Application Gateway - CLI**

```bash
# Create public IP
az network public-ip create \
  --resource-group rg-network \
  --name pip-appgw \
  --sku Standard \
  --allocation-method Static

# Create Application Gateway
az network application-gateway create \
  --resource-group rg-network \
  --name appgw-web \
  --location eastus \
  --sku WAF_v2 \
  --capacity 2 \
  --vnet-name vnet-prod \
  --subnet subnet-appgw \
  --public-ip-address pip-appgw \
  --http-settings-port 80 \
  --http-settings-protocol Http \
  --frontend-port 80 \
  --priority 100

# Enable WAF
az network application-gateway waf-config set \
  --resource-group rg-network \
  --gateway-name appgw-web \
  --enabled true \
  --firewall-mode Prevention \
  --rule-set-version 3.2
```

---

## 🔐 5. VPN Gateway (Hybrid Connectivity)

### **Site-to-Site VPN - CLI**

```bash
# Create VPN Gateway subnet
az network vnet subnet create \
  --resource-group rg-network \
  --vnet-name vnet-prod \
  --name GatewaySubnet \
  --address-prefix 10.0.255.0/27

# Create public IP for VPN Gateway
az network public-ip create \
  --resource-group rg-network \
  --name pip-vpngw \
  --allocation-method Static \
  --sku Standard

# Create VPN Gateway (takes 30-45 minutes!)
az network vnet-gateway create \
  --resource-group rg-network \
  --name vgw-prod \
  --location eastus \
  --vnet vnet-prod \
  --public-ip-addresses pip-vpngw \
  --gateway-type Vpn \
  --vpn-type RouteBased \
  --sku VpnGw1 \
  --no-wait

# Create Local Network Gateway (on-premises)
az network local-gateway create \
  --resource-group rg-network \
  --name lgw-onprem \
  --gateway-ip-address 203.0.113.10 \
  --local-address-prefixes 192.168.0.0/16

# Create VPN Connection
az network vpn-connection create \
  --resource-group rg-network \
  --name connection-to-onprem \
  --vnet-gateway1 vgw-prod \
  --local-gateway2 lgw-onprem \
  --shared-key "YourSecureSharedKey123!"
```

---

## 🛡️ 6. Azure Firewall

### **Deploy Azure Firewall - Terraform**

```hcl
resource "azurerm_subnet" "firewall" {
  name                 = "AzureFirewallSubnet"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.254.0/26"]
}

resource "azurerm_public_ip" "firewall" {
  name                = "pip-firewall"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  allocation_method   = "Static"
  sku                 = "Standard"
}

resource "azurerm_firewall" "main" {
  name                = "fw-prod"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  sku_name            = "AZFW_VNet"
  sku_tier            = "Standard"

  ip_configuration {
    name                 = "configuration"
    subnet_id            = azurerm_subnet.firewall.id
    public_ip_address_id = azurerm_public_ip.firewall.id
  }
}

resource "azurerm_firewall_network_rule_collection" "allow_web" {
  name                = "allow-web"
  azure_firewall_name = azurerm_firewall.main.name
  resource_group_name = azurerm_resource_group.main.name
  priority            = 100
  action              = "Allow"

  rule {
    name = "allow-http-https"
    source_addresses = ["10.0.0.0/16"]
    destination_ports = ["80", "443"]
    destination_addresses = ["*"]
    protocols = ["TCP"]
  }
}
```

---

## 🔗 7. VNet Peering

### **Create VNet Peering - CLI**

```bash
# Peer VNet1 to VNet2
az network vnet peering create \
  --resource-group rg-network \
  --name vnet1-to-vnet2 \
  --vnet-name vnet-prod \
  --remote-vnet vnet-dev \
  --allow-vnet-access \
  --allow-forwarded-traffic \
  --allow-gateway-transit

# Peer VNet2 to VNet1
az network vnet peering create \
  --resource-group rg-network \
  --name vnet2-to-vnet1 \
  --vnet-name vnet-dev \
  --remote-vnet vnet-prod \
  --allow-vnet-access \
  --allow-forwarded-traffic \
  --use-remote-gateways false
```

---

## 🔍 8. Network Monitoring

### **Enable Network Watcher**

```bash
# Enable Network Watcher
az network watcher configure \
  --resource-group NetworkWatcherRG \
  --locations eastus \
  --enabled true

# Test connectivity
az network watcher test-connectivity \
  --resource-group rg-network \
  --source-resource vm-web-01 \
  --dest-resource vm-app-01 \
  --protocol Tcp \
  --dest-port 80

# Enable NSG Flow Logs
az network watcher flow-log create \
  --resource-group rg-network \
  --nsg nsg-web \
  --name flowlog-nsg-web \
  --storage-account stlogs \
  --enabled true \
  --retention 30
```

---

## 🔄 VNet Peering (Connecting VNets)

**VNet Peering** connects virtual networks directly via Azure backbone network.

### **Types:**

| Type | Use Case | Regions |
|------|----------|---------|
| **VNet Peering** | Same region | Same region only |
| **Global VNet Peering** | Cross-region | Any Azure region |

---

### **Create VNet Peering:**

```bash
# Create peering from VNet1 to VNet2
az network vnet peering create \
  --resource-group rg-network \
  --name peer-vnet1-to-vnet2 \
  --vnet-name vnet-prod-eastus \
  --remote-vnet /subscriptions/{sub-id}/resourceGroups/rg-network/providers/Microsoft.Network/virtualNetworks/vnet-prod-westus \
  --allow-vnet-access \
  --allow-forwarded-traffic

# Create peering from VNet2 to VNet1 (bidirectional)
az network vnet peering create \
  --resource-group rg-network \
  --name peer-vnet2-to-vnet1 \
  --vnet-name vnet-prod-westus \
  --remote-vnet /subscriptions/{sub-id}/resourceGroups/rg-network/providers/Microsoft.Network/virtualNetworks/vnet-prod-eastus \
  --allow-vnet-access \
  --allow-forwarded-traffic

# List peerings
az network vnet peering list \
  --resource-group rg-network \
  --vnet-name vnet-prod-eastus \
  --output table

# Delete peering
az network vnet peering delete \
  --resource-group rg-network \
  --vnet-name vnet-prod-eastus \
  --name peer-vnet1-to-vnet2
```

---

### **Hub-Spoke Topology:**

```
         ┌─────────────────┐
         │   Hub VNet      │
         │  (Shared Svcs)  │
         │  - VPN Gateway  │
         │  - Firewall     │
         │  - DNS          │
         └────────┬────────┘
                  │
      ┌───────────┼───────────┐
      │           │           │
┌─────▼────┐ ┌───▼─────┐ ┌───▼─────┐
│ Spoke 1  │ │ Spoke 2 │ │ Spoke 3 │
│  (Prod)  │ │  (Dev)  │ │ (Test)  │
└──────────┘ └─────────┘ └─────────┘
```

**Terraform - Hub-Spoke:**

```hcl
# Hub VNet
resource "azurerm_virtual_network" "hub" {
  name                = "vnet-hub"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  address_space       = ["10.0.0.0/16"]
}

# Spoke VNet 1
resource "azurerm_virtual_network" "spoke1" {
  name                = "vnet-spoke1-prod"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  address_space       = ["10.1.0.0/16"]
}

# Peering: Hub to Spoke1
resource "azurerm_virtual_network_peering" "hub_to_spoke1" {
  name                         = "hub-to-spoke1"
  resource_group_name          = azurerm_resource_group.main.name
  virtual_network_name         = azurerm_virtual_network.hub.name
  remote_virtual_network_id    = azurerm_virtual_network.spoke1.id
  allow_virtual_network_access = true
  allow_forwarded_traffic      = true
  allow_gateway_transit        = true
}

# Peering: Spoke1 to Hub
resource "azurerm_virtual_network_peering" "spoke1_to_hub" {
  name                         = "spoke1-to-hub"
  resource_group_name          = azurerm_resource_group.main.name
  virtual_network_name         = azurerm_virtual_network.spoke1.name
  remote_virtual_network_id    = azurerm_virtual_network.hub.id
  allow_virtual_network_access = true
  allow_forwarded_traffic      = true
  use_remote_gateways          = true
}
```

---

## 🌐 Azure DNS

### **Private DNS Zones:**

```bash
# Create private DNS zone
az network private-dns zone create \
  --resource-group rg-network \
  --name contoso.internal

# Link to VNet
az network private-dns link vnet create \
  --resource-group rg-network \
  --zone-name contoso.internal \
  --name link-to-vnet-prod \
  --virtual-network vnet-prod \
  --registration-enabled false

# Add DNS record
az network private-dns record-set a add-record \
  --resource-group rg-network \
  --zone-name contoso.internal \
  --record-set-name webapp \
  --ipv4-address 10.0.1.10

# Query DNS
nslookup webapp.contoso.internal
```

---

## 🔥 Azure Firewall (Advanced)

**Azure Firewall** is a managed cloud-based network security service.

### **Create Azure Firewall:**

```bash
# Create firewall subnet (must be named AzureFirewallSubnet)
az network vnet subnet create \
  --resource-group rg-network \
  --vnet-name vnet-hub \
  --name AzureFirewallSubnet \
  --address-prefix 10.0.100.0/26

# Create public IP for firewall
az network public-ip create \
  --resource-group rg-network \
  --name pip-firewall \
  --sku Standard \
  --allocation-method Static

# Create firewall
az network firewall create \
  --resource-group rg-network \
  --name fw-hub \
  --location eastus

# Configure firewall IP
az network firewall ip-config create \
  --resource-group rg-network \
  --firewall-name fw-hub \
  --name fw-config \
  --public-ip-address pip-firewall \
  --vnet-name vnet-hub

# Get firewall private IP
FW_PRIVATE_IP=$(az network firewall show \
  --resource-group rg-network \
  --name fw-hub \
  --query "ipConfigurations[0].privateIpAddress" \
  --output tsv)

echo "Firewall Private IP: $FW_PRIVATE_IP"
```

---

### **Firewall Rules:**

**Application Rules (FQDN-based):**
```bash
# Allow outbound HTTP/HTTPS to specific domains
az network firewall application-rule create \
  --resource-group rg-network \
  --firewall-name fw-hub \
  --collection-name allow-web \
  --name allow-microsoft \
  --protocols Http=80 Https=443 \
  --source-addresses 10.1.0.0/16 \
  --target-fqdns "*.microsoft.com" "*.azure.com" \
  --priority 100 \
  --action Allow
```

**Network Rules (IP-based):**
```bash
# Allow specific ports between subnets
az network firewall network-rule create \
  --resource-group rg-network \
  --firewall-name fw-hub \
  --collection-name allow-internal \
  --name allow-sql \
  --protocols TCP \
  --source-addresses 10.1.0.0/24 \
  --destination-addresses 10.1.2.0/24 \
  --destination-ports 1433 \
  --priority 200 \
  --action Allow
```

**NAT Rules (DNAT for inbound):**
```bash
# Forward external traffic to internal server
az network firewall nat-rule create \
  --resource-group rg-network \
  --firewall-name fw-hub \
  --collection-name inbound-nat \
  --name allow-rdp \
  --protocols TCP \
  --source-addresses "*" \
  --destination-addresses <FIREWALL_PUBLIC_IP> \
  --destination-ports 3389 \
  --translated-address 10.1.1.10 \
  --translated-port 3389 \
  --priority 300
```

---

### **Route Traffic Through Firewall:**

```bash
# Create route table
az network route-table create \
  --resource-group rg-network \
  --name rt-spoke-to-firewall

# Add route to firewall
az network route-table route create \
  --resource-group rg-network \
  --route-table-name rt-spoke-to-firewall \
  --name route-to-firewall \
  --address-prefix 0.0.0.0/0 \
  --next-hop-type VirtualAppliance \
  --next-hop-ip-address $FW_PRIVATE_IP

# Associate with subnet
az network vnet subnet update \
  --resource-group rg-network \
  --vnet-name vnet-spoke1-prod \
  --name subnet-app \
  --route-table rt-spoke-to-firewall
```

---

## 🎯 Real-World Network Architectures

### **Architecture 1: 3-Tier Application**

```
Internet → App Gateway (WAF) → Web Tier (Subnet 1)
                                    ↓
                              App Tier (Subnet 2)
                                    ↓
                              Data Tier (Subnet 3)
                                    ↓
                             Private Endpoint
                                    ↓
                            Azure SQL Database
```

### **Architecture 2: Hybrid Cloud**

```
On-Premises ← VPN Gateway → Hub VNet → Firewall
                              ↓
                        (VNet Peering)
                              ↓
                    ┌─────────┼─────────┐
                    ↓         ↓         ↓
                Spoke 1   Spoke 2   Spoke 3
```

### **Architecture 3: Multi-Region with Traffic Manager**

```
User → Traffic Manager (DNS-based routing)
          ↓                    ↓
    Region 1 (Primary)    Region 2 (DR)
       App Gateway          App Gateway
          ↓                    ↓
        AKS Cluster          AKS Cluster
```

---

## 🛠️ Comprehensive Troubleshooting

### **Issue 1: Cannot Connect Between Subnets**

**Symptoms:**
- VMs in different subnets can't communicate
- Timeout errors

**Debugging:**
```bash
# 1. Check NSG effective rules
az network nic list-effective-nsg \
  --resource-group rg-network \
  --name vm1-nic \
  --output table

# 2. Check route table
az network nic show-effective-route-table \
  --resource-group rg-network \
  --name vm1-nic \
  --output table

# 3. Test connectivity
az network watcher test-connectivity \
  --resource-group rg-network \
  --source-resource vm-web-01 \
  --dest-resource vm-app-01 \
  --dest-port 80

# 4. Enable NSG Flow Logs
az network watcher flow-log create \
  --resource-group rg-network \
  --nsg nsg-web \
  --name flowlog-nsg-web \
  --storage-account stlogs \
  --enabled true
```

**Common Causes:**
- NSG blocking traffic
- Missing route in route table
- Incorrect subnet configuration
- Service endpoint misconfiguration

**Solutions:**
- Add NSG allow rule
- Update route table
- Check firewall rules

---

### **Issue 2: VPN Connection Not Working**

**Symptoms:**
- VPN status shows "Not Connected"
- Cannot access on-premises resources

**Debugging:**
```bash
# Check VPN status
az network vpn-connection show \
  --resource-group rg-network \
  --name connection-to-onprem \
  --query "{Name:name, Status:connectionStatus, IngressBytes:ingressBytesTransferred, EgressBytes:egressBytesTransferred}"

# View BGP status (if using BGP)
az network vnet-gateway list-bgp-peer-status \
  --resource-group rg-network \
  --name vpn-gateway-hub

# Check logs
az network vpn-connection show \
  --resource-group rg-network \
  --name connection-to-onprem \
  --query "connectionStatus"

# Reset connection
az network vpn-connection shared-key reset \
  --resource-group rg-network \
  --connection-name connection-to-onprem
```

**Common Causes:**
- Shared key mismatch
- On-premises device configuration
- Firewall blocking UDP 500/4500
- IP address conflicts

---

### **Issue 3: High Latency or Packet Loss**

**Debugging:**
```bash
# Test latency
ping <DESTINATION_IP>

# Traceroute
traceroute <DESTINATION_IP>

# Azure Network Watcher
az network watcher test-connectivity \
  --resource-group rg-network \
  --source-resource vm-web-01 \
  --dest-address 10.2.1.10 \
  --protocol TCP \
  --dest-port 443

# Check for packet loss
az network watcher show-next-hop \
  --resource-group rg-network \
  --vm vm-web-01 \
  --dest-ip 10.2.1.10
```

**Solutions:**
- Enable Accelerated Networking on VMs
- Use proximity placement groups
- Check NSG rules causing drops
- Upgrade to higher network tier

---

### **Issue 4: Load Balancer Not Distributing Traffic**

**Debugging:**
```bash
# Check backend pool health
az network lb show \
  --resource-group rg-network \
  --name lb-web \
  --query "backendAddressPools[].backendIPConfigurations[].id"

# View health probe status
az network lb probe show \
  --resource-group rg-network \
  --lb-name lb-web \
  --name health-probe

# Check load balancing rules
az network lb rule list \
  --resource-group rg-network \
  --lb-name lb-web \
  --output table

# View metrics
az monitor metrics list \
  --resource $(az network lb show --resource-group rg-network --name lb-web --query id --output tsv) \
  --metric "VipAvailability" "DipAvailability" \
  --start-time 2024-01-01T00:00:00Z
```

**Common Causes:**
- Health probe failing (check backend servers)
- Incorrect load balancing rules
- Session persistence causing uneven distribution
- Backend pool members unhealthy

---

## ✅ Network Security Best Practices

### **1. Defense in Depth:**

```
Layer 1: Azure DDoS Protection
Layer 2: Azure Firewall / NVA
Layer 3: Network Security Groups
Layer 4: Application Gateway WAF
Layer 5: Private Endpoints
Layer 6: Service Endpoints
```

### **2. NSG Rules Best Practices:**

```bash
# Deny all by default, allow specific
az network nsg rule create \
  --resource-group rg-network \
  --nsg-name nsg-web \
  --name DenyAllInbound \
  --priority 4096 \
  --access Deny \
  --protocol '*' \
  --direction Inbound \
  --source-address-prefixes '*' \
  --destination-address-prefixes '*'

# Allow only necessary ports
az network nsg rule create \
  --resource-group rg-network \
  --nsg-name nsg-web \
  --name AllowHTTPS \
  --priority 100 \
  --access Allow \
  --protocol Tcp \
  --direction Inbound \
  --source-address-prefixes 'Internet' \
  --destination-port-ranges 443
```

### **3. Use Service Tags:**

```bash
# Allow Azure services without specific IPs
az network nsg rule create \
  --resource-group rg-network \
  --nsg-name nsg-app \
  --name AllowAzureSQL \
  --priority 200 \
  --destination-address-prefixes 'Sql' \
  --destination-port-ranges 1433 \
  --protocol Tcp \
  --access Allow
```

**Common Service Tags:**
- `AzureCloud` - All Azure IPs
- `Internet` - Public Internet
- `Sql` - Azure SQL Database
- `Storage` - Azure Storage
- `AzureActiveDirectory` - Azure AD

---

### **4. Network Segmentation:**

```
Production VNet     Dev/Test VNet     Management VNet
     ↓                  ↓                   ↓
  No Peering      Peering to Hub     Peering to Hub
     ↓                  ↓                   ↓
  Isolated        Controlled Access   Restricted Access
```

---

## 💰 Cost Optimization

**1. Use Standard Load Balancer Wisely:**
- Standard Load Balancer has per-rule cost
- Basic Load Balancer is free (but limited features)

**2. Optimize VPN Gateway:**
- Use VpnGw1 for lower bandwidth needs
- Consider ExpressRoute for high bandwidth

**3. Minimize Data Transfer:**
- Keep resources in same region when possible
- Use Azure CDN for static content
- Implement caching strategies

**4. Monitor Bandwidth Usage:**
```bash
az monitor metrics list \
  --resource $(az network vnet-gateway show --resource-group rg-network --name vpn-gateway-hub --query id --output tsv) \
  --metric "BandwidthUtilization" \
  --start-time 2024-01-01T00:00:00Z
```

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

