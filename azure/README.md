# Azure Resources - Complete Learning Hub

**Comprehensive DevOps Guide to Azure Services**

---

## 🎯 Overview

This Azure learning hub provides **dedicated comprehensive guides** for each major Azure service, covering configuration, deployment methods (Portal, CLI, PowerShell, ARM, Terraform), scaling, troubleshooting, and best practices.

---

## 📚 Azure Resource Guides

### **1. 💻 [Virtual Machines](virtual-machines/)**
Complete guide to Azure VMs covering:
- VM fundamentals and sizing
- Deployment methods: Portal, CLI, PowerShell, ARM Templates, Terraform
- VM Scale Sets (VMSS) and auto-scaling
- High availability with availability sets and zones
- Backup and disaster recovery
- Performance monitoring and optimization
- Comprehensive troubleshooting
- Production best practices

**Key Topics:** VM configuration, scaling types (vertical/horizontal), VMSS, availability zones, managed disks, custom images, cloud-init, extensions

---

### **2. 🌐 [Networking](networking/)**
Complete Azure networking guide covering:
- Virtual Networks (VNets) and subnets
- Network Security Groups (NSGs) and firewall rules
- Load Balancers (Standard and Basic)
- Application Gateway with WAF
- VPN Gateway for hybrid connectivity
- Azure Firewall
- Private Endpoints and Service Endpoints
- VNet Peering
- Network monitoring and diagnostics

**Key Topics:** IP addressing, NSG rules, load balancing, WAF, site-to-site VPN, Hub-spoke architecture, traffic routing, DNS

---

### **3. 💾 [Storage](storage/)**
Complete storage services guide covering:
- Storage Accounts (GPv2, Premium, Data Lake)
- Blob Storage (Hot, Cool, Archive tiers)
- Azure Files (SMB file shares)
- Queue Storage (messaging)
- Table Storage (NoSQL)
- Data Lake Storage Gen2
- Storage security and access control
- Lifecycle management and cost optimization
- Replication options (LRS, ZRS, GRS, GZRS)

**Key Topics:** Blob containers, file shares, SAS tokens, private endpoints, lifecycle policies, geo-replication, encryption

---

### **4. ⚡ [Azure Functions](azure-functions/)**
Serverless computing guide covering:
- Function Apps and hosting plans
- Triggers: HTTP, Timer, Queue, Blob, Event Grid
- Input and output bindings
- Durable Functions
- Deployment with CLI, ARM, Terraform
- Monitoring and diagnostics
- Performance optimization
- CI/CD integration

**Key Topics:** Consumption plan, Premium plan, event-driven architecture, bindings, function chaining, monitoring

---

### **5. 🌍 [Web Apps (App Service)](web-apps/)**
PaaS web hosting guide covering:
- App Service Plans and pricing tiers
- Web App deployment methods
- Deployment slots (blue-green deployments)
- Custom domains and SSL certificates
- Application settings and connection strings
- Auto-scaling and performance
- CI/CD with GitHub Actions and Azure Pipelines
- Container deployment

**Key Topics:** App Service Plan, deployment slots, auto-scale rules, custom domains, SSL, application logging, Kudu

---

### **6. 📊 [Monitoring](monitoring/)**
Complete observability guide covering:
- Azure Monitor overview
- Log Analytics Workspace and KQL queries
- Application Insights for APM
- Metrics and alerts
- Action Groups and notifications
- Workbooks and dashboards
- Diagnostic settings
- Cost monitoring

**Key Topics:** Log Analytics, KQL, Application Insights, metrics, alerts, dashboards, diagnostic logs, query optimization

---

### **7. 🗄️ [Databases](databases/)**
Managed database services guide covering:
- Azure SQL Database
- Azure Database for PostgreSQL
- Azure Database for MySQL
- Cosmos DB (multi-model NoSQL)
- Database security and firewall rules
- High availability and replication
- Backup and restore
- Performance tuning

**Key Topics:** SQL Database, PostgreSQL, MySQL, Cosmos DB, DTU vs vCore, read replicas, geo-replication, point-in-time restore

---

### **8. 🐳 [Containers](containers/)**
Container services guide covering:
- Azure Container Registry (ACR)
- Container image management
- Geo-replication for ACR
- Azure Container Instances (ACI)
- Container security scanning
- Integration with AKS
- CI/CD for containers

**Key Topics:** ACR, container registry, image scanning, ACI, container orchestration, Dockerfile, image tagging

---

### **9. 🔒 [Security](security/)**
Azure security services guide covering:
- Azure Key Vault (secrets, keys, certificates)
- Managed Identities (system and user-assigned)
- Role-Based Access Control (RBAC)
- Azure AD integration
- Service Principals
- Network security
- Azure Security Center/Defender
- Compliance and policies

**Key Topics:** Key Vault, managed identities, RBAC, Azure AD, service principals, secrets management, security best practices

---

### **10. ☸️ [Azure Kubernetes Service (AKS)](aks/)**
Managed Kubernetes guide covering:
- AKS cluster creation and configuration
- Node pools (system and user)
- Networking (Azure CNI vs Kubenet)
- Integration with ACR
- Auto-scaling (HPA and Cluster Autoscaler)
- Monitoring with Container Insights
- Security with Azure RBAC and Network Policies
- CI/CD deployment patterns

**Key Topics:** AKS setup, node pools, networking plugins, ACR integration, auto-scaling, monitoring, GitOps, Helm

---

## 🚀 Getting Started

### **Prerequisites**
- Azure subscription (create free account at [azure.com](https://azure.com/free))
- Azure CLI installed ([installation guide](https://docs.microsoft.com/cli/azure/install-azure-cli))
- PowerShell (for Windows users)
- Terraform installed (optional, for IaC)
- Git installed

### **Quick Setup**

```bash
# Login to Azure
az login

# Set default subscription
az account set --subscription "Your Subscription Name"

# Create resource group for learning
az group create \
  --name rg-learning \
  --location eastus

# Verify setup
az account show --output table
```

---

## 📖 Learning Path

### **Beginner Path (Start Here):**
1. **Virtual Machines** - Understand compute basics
2. **Networking** - Learn VNets and NSGs
3. **Storage** - Work with blobs and files
4. **Monitoring** - Set up basic logging

### **Intermediate Path:**
1. **Web Apps** - Deploy PaaS applications
2. **Databases** - Managed database services
3. **Containers** - ACR and container management
4. **Azure Functions** - Serverless computing

### **Advanced Path:**
1. **AKS** - Kubernetes orchestration
2. **Security** - Advanced security patterns
3. **Multi-service architecture** - Integrate all services

---

## 🛠️ Common Deployment Patterns

### **Pattern 1: 3-Tier Web Application**
```
┌─────────────────────────────────────────┐
│  Application Gateway + WAF              │
│  (Public endpoint with SSL)             │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│  Web Tier (App Service or VMSS)         │
│  - Auto-scaling enabled                 │
│  - Deployment slots for blue-green      │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│  Application Tier (AKS or Functions)    │
│  - Microservices architecture           │
│  - Horizontal pod autoscaling           │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│  Data Tier (Azure SQL + Storage)        │
│  - Geo-replication enabled              │
│  - Automated backups                    │
└─────────────────────────────────────────┘
```

### **Pattern 2: Serverless Architecture**
```
Azure Functions + API Management
    ↓
Cosmos DB + Azure Storage
    ↓
Event Grid + Service Bus
    ↓
Application Insights (Monitoring)
```

### **Pattern 3: Microservices on AKS**
```
AKS Cluster
├── Ingress Controller (NGINX)
├── Microservices (Pods)
├── Service Mesh (Istio/Linkerd)
├── Monitoring (Prometheus + Grafana)
└── CI/CD (Azure Pipelines + Helm)
```

---

## 📊 Cost Optimization Tips

1. **Use appropriate tiers** - Start with Basic/Standard, upgrade as needed
2. **Implement auto-scaling** - Scale down during off-hours
3. **Reserved instances** - Save up to 72% for predictable workloads
4. **Storage lifecycle management** - Move data to Cool/Archive tiers
5. **Monitor and analyze costs** - Use Azure Cost Management
6. **Delete unused resources** - Regular cleanup of dev/test resources
7. **Use Azure Hybrid Benefit** - If you have existing licenses
8. **Set up budgets and alerts** - Proactive cost management

---

## 🎯 Best Practices

### **Security**
- ✅ Use managed identities instead of keys/passwords
- ✅ Enable Azure AD authentication
- ✅ Implement network isolation with VNets and private endpoints
- ✅ Store secrets in Key Vault
- ✅ Enable encryption at rest and in transit
- ✅ Implement least privilege with RBAC
- ✅ Enable Azure Security Center recommendations

### **High Availability**
- ✅ Use availability zones for critical workloads
- ✅ Implement geo-replication for disaster recovery
- ✅ Configure health probes and auto-healing
- ✅ Use load balancers for traffic distribution
- ✅ Implement backup and restore strategies

### **Operations**
- ✅ Tag all resources for cost allocation
- ✅ Use resource naming conventions
- ✅ Implement infrastructure as code (Terraform/ARM)
- ✅ Set up comprehensive monitoring and alerting
- ✅ Document architecture and runbooks
- ✅ Automate deployments with CI/CD
- ✅ Regular security and compliance audits

---

## 🔗 Additional Resources

### **Official Documentation**
- [Azure Documentation](https://docs.microsoft.com/azure/)
- [Azure Architecture Center](https://docs.microsoft.com/azure/architecture/)
- [Azure CLI Reference](https://docs.microsoft.com/cli/azure/)
- [Azure PowerShell Reference](https://docs.microsoft.com/powershell/azure/)

### **Learning Platforms**
- [Microsoft Learn](https://docs.microsoft.com/learn/) - Free interactive learning
- [Azure Certifications](https://docs.microsoft.com/learn/certifications/) - Professional certifications
- [Azure Friday](https://azure.microsoft.com/resources/videos/azure-friday/) - Weekly video series

### **Community**
- [Azure Community](https://techcommunity.microsoft.com/t5/azure/ct-p/Azure)
- [Stack Overflow - Azure Tag](https://stackoverflow.com/questions/tagged/azure)
- [Azure Updates](https://azure.microsoft.com/updates/) - Latest features and announcements

---

## 📁 Repository Structure

```
azure/
├── README.md (this file)
├── virtual-machines/
│   └── README.md
├── networking/
│   └── README.md
├── storage/
│   └── README.md
├── azure-functions/
│   └── README.md
├── web-apps/
│   └── README.md
├── monitoring/
│   └── README.md
├── databases/
│   └── README.md
├── containers/
│   └── README.md
├── security/
│   └── README.md
└── aks/
    └── README.md
```

---

## 🎓 Certification Paths

- **AZ-900**: Azure Fundamentals
- **AZ-104**: Azure Administrator
- **AZ-204**: Azure Developer
- **AZ-305**: Azure Solutions Architect Expert
- **AZ-400**: DevOps Engineer Expert

---

**Master Azure one service at a time! Start with any guide above and build your cloud expertise! ☁️🚀**

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)
