# Azure for DevOps Engineers - 16-Week Learning Roadmap

**From Azure Fundamentals to Production-Grade Cloud Infrastructure**

For experienced DevOps engineers (3-7+ years) transitioning to or expanding Azure expertise.

---

## 📋 Prerequisites

- ✅ Strong Linux/Windows system administration
- ✅ Understanding of networking concepts
- ✅ Experience with virtualization
- ✅ Basic cloud concepts
- ✅ Command-line proficiency
- ✅ Git version control
- ✅ Infrastructure as Code exposure (Terraform/Ansible)

---

## 🎯 Learning Objectives

By completing this roadmap, you will:

✅ Design and deploy Azure infrastructure at scale  
✅ Implement hub-spoke network architectures  
✅ Deploy and manage production AKS clusters  
✅ Integrate Azure with CI/CD pipelines  
✅ Implement comprehensive monitoring and alerting  
✅ Optimize costs and performance  
✅ Secure Azure environments with best practices  
✅ Troubleshoot complex Azure issues  
✅ Design highly available, multi-region solutions  
✅ Automate everything with Infrastructure as Code  

---

## 🗓️ 16-Week Learning Plan

---

## **Week 1-2: Azure Fundamentals & Core Services**

### **Week 1: Azure Basics & Setup**

**Topics:**
- Azure global infrastructure (regions, availability zones)
- Azure subscription & resource hierarchy
- Azure Portal navigation
- Azure CLI installation & configuration
- Azure PowerShell setup
- Resource Groups concepts
- Tagging strategies

**Hands-On:**
```bash
# Install Azure CLI
winget install Microsoft.AzureCLI

# Login
az login

# List subscriptions
az account list --output table

# Set default subscription
az account set --subscription "YOUR_SUBSCRIPTION_NAME"

# Create first resource group
az group create \
  --name rg-learning-week1 \
  --location eastus \
  --tags Environment=Learning Week=1 Owner=YourName

# List all resource groups
az group list --output table

# Get resource group details
az group show \
  --name rg-learning-week1 \
  --output json

# Delete resource group
az group delete \
  --name rg-learning-week1 \
  --yes \
  --no-wait
```

**Key Concepts:**
- Resource Manager (ARM)
- Resource providers
- Regions vs Availability Zones
- Management Groups
- Azure hierarchy: Management Groups → Subscriptions → Resource Groups → Resources

**Practice Tasks:**
- [ ] Install Azure CLI and PowerShell
- [ ] Create 3 resource groups in different regions
- [ ] Implement tagging strategy
- [ ] Explore Azure Portal
- [ ] Set up Azure Cloud Shell

**Resources:**
- [Azure Global Infrastructure](https://azure.microsoft.com/global-infrastructure/)
- [Azure CLI Reference](https://docs.microsoft.com/cli/azure/)

---

### **Week 2: Compute & Storage Basics**

**Topics:**
- Virtual Machines (Linux & Windows)
- VM sizes and families
- Storage Accounts (Blob, File, Table, Queue)
- Managed Disks
- Availability Sets vs Availability Zones

**Hands-On - Deploy Linux VM:**
```bash
# Create resource group
az group create \
  --name rg-vm-tutorial \
  --location eastus

# Create virtual network
az network vnet create \
  --resource-group rg-vm-tutorial \
  --name vnet-tutorial \
  --address-prefix 10.0.0.0/16 \
  --subnet-name subnet-web \
  --subnet-prefix 10.0.1.0/24

# Create network security group
az network nsg create \
  --resource-group rg-vm-tutorial \
  --name nsg-web

# Add SSH rule
az network nsg rule create \
  --resource-group rg-vm-tutorial \
  --nsg-name nsg-web \
  --name AllowSSH \
  --priority 1000 \
  --source-address-prefixes '*' \
  --destination-port-ranges 22 \
  --access Allow \
  --protocol Tcp

# Create public IP
az network public-ip create \
  --resource-group rg-vm-tutorial \
  --name pip-vm-web \
  --sku Standard \
  --allocation-method Static

# Create network interface
az network nic create \
  --resource-group rg-vm-tutorial \
  --name nic-vm-web \
  --vnet-name vnet-tutorial \
  --subnet subnet-web \
  --public-ip-address pip-vm-web \
  --network-security-group nsg-web

# Create VM
az vm create \
  --resource-group rg-vm-tutorial \
  --name vm-web-01 \
  --nics nic-vm-web \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --admin-username azureuser \
  --generate-ssh-keys \
  --custom-data cloud-init.txt

# Get public IP
az network public-ip show \
  --resource-group rg-vm-tutorial \
  --name pip-vm-web \
  --query ipAddress \
  --output tsv

# SSH to VM
ssh azureuser@<PUBLIC_IP>

# Install web server
sudo apt update
sudo apt install -y nginx
sudo systemctl start nginx

# Test
curl http://<PUBLIC_IP>
```

**Storage Account:**
```bash
# Create storage account
az storage account create \
  --name stlearning$(date +%s) \
  --resource-group rg-vm-tutorial \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2

# Create blob container
az storage container create \
  --account-name stlearning... \
  --name data \
  --public-access off

# Upload file
az storage blob upload \
  --account-name stlearning... \
  --container-name data \
  --name myfile.txt \
  --file ./myfile.txt

# List blobs
az storage blob list \
  --account-name stlearning... \
  --container-name data \
  --output table
```

**Practice Tasks:**
- [ ] Deploy 3 VMs in an Availability Set
- [ ] Deploy 2 VMs across Availability Zones
- [ ] Create and attach managed disks
- [ ] Set up Azure File Share
- [ ] Configure blob storage with lifecycle policies

---

## **Week 3-4: Networking Fundamentals**

### **Week 3: Virtual Networks & Connectivity**

**Topics:**
- Virtual Networks (VNets)
- Subnets and address spaces
- Network Security Groups (NSGs)
- Application Security Groups (ASGs)
- VNet Peering
- VPN Gateway basics
- Azure Bastion

**Hub-Spoke Network Architecture:**
```bash
# Hub VNet
az network vnet create \
  --resource-group rg-network \
  --name vnet-hub \
  --address-prefix 10.0.0.0/16 \
  --subnet-name AzureFirewallSubnet \
  --subnet-prefix 10.0.1.0/26

az network vnet subnet create \
  --resource-group rg-network \
  --vnet-name vnet-hub \
  --name GatewaySubnet \
  --address-prefix 10.0.2.0/27

az network vnet subnet create \
  --resource-group rg-network \
  --vnet-name vnet-hub \
  --name AzureBastionSubnet \
  --address-prefix 10.0.3.0/27

# Spoke VNet - Production
az network vnet create \
  --resource-group rg-network \
  --name vnet-spoke-prod \
  --address-prefix 10.1.0.0/16 \
  --subnet-name subnet-web \
  --subnet-prefix 10.1.1.0/24

az network vnet subnet create \
  --resource-group rg-network \
  --vnet-name vnet-spoke-prod \
  --name subnet-app \
  --address-prefix 10.1.2.0/24

az network vnet subnet create \
  --resource-group rg-network \
  --vnet-name vnet-spoke-prod \
  --name subnet-data \
  --address-prefix 10.1.3.0/24

# Spoke VNet - Development
az network vnet create \
  --resource-group rg-network \
  --name vnet-spoke-dev \
  --address-prefix 10.2.0.0/16 \
  --subnet-name subnet-web \
  --subnet-prefix 10.2.1.0/24

# VNet Peering: Hub to Prod
az network vnet peering create \
  --resource-group rg-network \
  --name hub-to-prod \
  --vnet-name vnet-hub \
  --remote-vnet vnet-spoke-prod \
  --allow-forwarded-traffic \
  --allow-gateway-transit

# VNet Peering: Prod to Hub
az network vnet peering create \
  --resource-group rg-network \
  --name prod-to-hub \
  --vnet-name vnet-spoke-prod \
  --remote-vnet vnet-hub \
  --allow-forwarded-traffic \
  --use-remote-gateways=false

# Repeat for Dev spoke...
```

**Practice Tasks:**
- [ ] Build complete hub-spoke topology
- [ ] Configure NSG rules for 3-tier app
- [ ] Set up VNet peering with service chaining
- [ ] Deploy Azure Bastion for secure access
- [ ] Configure User Defined Routes (UDR)

---

### **Week 4: Advanced Networking**

**Topics:**
- Azure Load Balancer
- Application Gateway
- Azure Firewall
- Private Endpoints
- Service Endpoints
- Azure DNS
- Traffic Manager

**Application Gateway Setup:**
```bash
# Create public IP for App Gateway
az network public-ip create \
  --resource-group rg-network \
  --name pip-appgw \
  --sku Standard \
  --allocation-method Static

# Create subnet for App Gateway
az network vnet subnet create \
  --resource-group rg-network \
  --vnet-name vnet-spoke-prod \
  --name subnet-appgw \
  --address-prefix 10.1.10.0/24

# Create Application Gateway
az network application-gateway create \
  --resource-group rg-network \
  --name appgw-prod \
  --location eastus \
  --sku Standard_v2 \
  --capacity 2 \
  --vnet-name vnet-spoke-prod \
  --subnet subnet-appgw \
  --public-ip-address pip-appgw \
  --http-settings-port 80 \
  --http-settings-protocol Http \
  --frontend-port 80

# Add backend pool
az network application-gateway address-pool create \
  --resource-group rg-network \
  --gateway-name appgw-prod \
  --name pool-web \
  --servers 10.1.1.4 10.1.1.5

# Configure health probe
az network application-gateway probe create \
  --resource-group rg-network \
  --gateway-name appgw-prod \
  --name probe-web \
  --protocol Http \
  --host-name-from-http-settings true \
  --path /health
```

**Azure Firewall:**
```bash
# Create public IP
az network public-ip create \
  --resource-group rg-network \
  --name pip-firewall \
  --sku Standard \
  --allocation-method Static

# Create Azure Firewall
az network firewall create \
  --resource-group rg-network \
  --name fw-hub \
  --location eastus

# Configure firewall
az network firewall ip-config create \
  --resource-group rg-network \
  --firewall-name fw-hub \
  --name fw-config \
  --public-ip-address pip-firewall \
  --vnet-name vnet-hub

# Add application rule
az network firewall application-rule create \
  --resource-group rg-network \
  --firewall-name fw-hub \
  --collection-name app-rules \
  --name AllowWeb \
  --protocols Http=80 Https=443 \
  --source-addresses 10.1.0.0/16 10.2.0.0/16 \
  --target-fqdns *.microsoft.com *.ubuntu.com

# Add network rule
az network firewall network-rule create \
  --resource-group rg-network \
  --firewall-name fw-hub \
  --collection-name net-rules \
  --name AllowDNS \
  --protocols UDP \
  --source-addresses 10.1.0.0/16 10.2.0.0/16 \
  --destination-addresses 168.63.129.16 \
  --destination-ports 53
```

**Practice Tasks:**
- [ ] Configure Application Gateway with WAF
- [ ] Deploy Azure Firewall with custom rules
- [ ] Set up Private Link for Azure SQL
- [ ] Configure Traffic Manager for multi-region
- [ ] Implement Azure Front Door

---

## **Week 5-6: Identity, Security & Governance**

### **Week 5: Azure Active Directory & RBAC**

**Topics:**
- Azure Active Directory basics
- Service Principals
- Managed Identities (System & User assigned)
- Role-Based Access Control (RBAC)
- Azure Policy
- Custom roles

**Service Principal Creation:**
```bash
# Create service principal for CI/CD
az ad sp create-for-rbac \
  --name sp-cicd-pipeline \
  --role Contributor \
  --scopes /subscriptions/{subscription-id}

# Create with specific permissions
az ad sp create-for-rbac \
  --name sp-aks-devops \
  --role "Azure Kubernetes Service Cluster User Role" \
  --scopes /subscriptions/{sub-id}/resourceGroups/rg-aks

# List service principals
az ad sp list --display-name sp-cicd-pipeline

# Delete service principal
az ad sp delete --id {app-id}
```

**Managed Identity:**
```bash
# Create VM with system-assigned managed identity
az vm create \
  --resource-group rg-identity \
  --name vm-identity-demo \
  --image Ubuntu2204 \
  --assign-identity \
  --role Contributor \
  --scope /subscriptions/{sub-id}/resourceGroups/rg-identity

# Create user-assigned managed identity
az identity create \
  --resource-group rg-identity \
  --name id-app-identity

# Assign to VM
az vm identity assign \
  --resource-group rg-identity \
  --name vm-identity-demo \
  --identities id-app-identity

# Grant permissions
az role assignment create \
  --assignee {identity-principal-id} \
  --role "Storage Blob Data Contributor" \
  --scope /subscriptions/{sub-id}/resourceGroups/rg-storage
```

**RBAC Configuration:**
```bash
# Assign role to user
az role assignment create \
  --assignee user@domain.com \
  --role "Virtual Machine Contributor" \
  --resource-group rg-vms

# Assign role to group
az role assignment create \
  --assignee {group-object-id} \
  --role "Network Contributor" \
  --scope /subscriptions/{sub-id}

# List role assignments
az role assignment list \
  --resource-group rg-vms \
  --output table

# Create custom role
az role definition create --role-definition @custom-role.json
```

**Practice Tasks:**
- [ ] Create service principals for different purposes
- [ ] Configure managed identities for VMs and AKS
- [ ] Implement least-privilege RBAC
- [ ] Create custom roles
- [ ] Set up Azure AD groups for team access

---

### **Week 6: Azure Key Vault & Security**

**Topics:**
- Azure Key Vault (secrets, keys, certificates)
- Key Vault access policies
- Key Vault RBAC
- Azure Security Center
- Azure Defender
- Security best practices

**Key Vault Setup:**
```bash
# Create Key Vault
az keyvault create \
  --resource-group rg-security \
  --name kv-prod-secrets-$(date +%s) \
  --location eastus \
  --enable-soft-delete true \
  --enable-purge-protection true \
  --enabled-for-deployment true \
  --enabled-for-template-deployment true

# Set secret
az keyvault secret set \
  --vault-name kv-prod-secrets-... \
  --name db-connection-string \
  --value "Server=...;Database=...;User=...;Password=..."

# Get secret
az keyvault secret show \
  --vault-name kv-prod-secrets-... \
  --name db-connection-string \
  --query value \
  --output tsv

# Grant access to service principal
az keyvault set-policy \
  --vault-name kv-prod-secrets-... \
  --spn {service-principal-id} \
  --secret-permissions get list

# Grant access to managed identity
az keyvault set-policy \
  --vault-name kv-prod-secrets-... \
  --object-id {managed-identity-principal-id} \
  --secret-permissions get list

# Create certificate
az keyvault certificate create \
  --vault-name kv-prod-secrets-... \
  --name ssl-cert \
  --policy @cert-policy.json

# Enable Key Vault logging
az monitor diagnostic-settings create \
  --resource {keyvault-id} \
  --name kv-diagnostics \
  --workspace {log-analytics-id} \
  --logs '[{"category": "AuditEvent","enabled": true}]'
```

**Azure Policy:**
```bash
# List built-in policies
az policy definition list --query "[?policyType=='BuiltIn'].{name:name, displayName:displayName}" --output table

# Assign policy (require tags)
az policy assignment create \
  --name require-tag-policy \
  --policy {policy-definition-id} \
  --scope /subscriptions/{sub-id} \
  --params '{"tagName":{"value":"Environment"}}'

# Create custom policy
az policy definition create \
  --name allowed-vm-sizes \
  --rules @policy-rules.json \
  --params @policy-params.json \
  --mode All

# Check compliance
az policy state list \
  --resource-group rg-prod \
  --output table
```

**Practice Tasks:**
- [ ] Deploy Key Vault with access policies
- [ ] Store and retrieve secrets programmatically
- [ ] Configure Key Vault for VM disk encryption
- [ ] Implement Azure Policies for compliance
- [ ] Set up Security Center recommendations

---

## **Week 7-10: Containers & Kubernetes**

### **Week 7: Azure Container Registry & Docker**

**Topics:**
- Azure Container Registry (ACR)
- Container image management
- ACR Tasks
- Geo-replication
- Content trust
- Vulnerability scanning

**ACR Setup:**
```bash
# Create container registry
az acr create \
  --resource-group rg-containers \
  --name acrprodregistry$(date +%s) \
  --sku Premium \
  --location eastus \
  --admin-enabled false

# Login to ACR
az acr login --name acrprodregistry...

# Build and push image
docker build -t myapp:v1 .
docker tag myapp:v1 acrprodregistry.azurecr.io/myapp:v1
docker push acrprodregistry.azurecr.io/myapp:v1

# List images
az acr repository list \
  --name acrprodregistry... \
  --output table

# Show image tags
az acr repository show-tags \
  --name acrprodregistry... \
  --repository myapp \
  --output table

# ACR Task (Build in cloud)
az acr task create \
  --registry acrprodregistry... \
  --name build-myapp \
  --image myapp:{{.Run.ID}} \
  --context https://github.com/user/repo.git \
  --file Dockerfile \
  --git-access-token {token}

# Enable geo-replication
az acr replication create \
  --registry acrprodregistry... \
  --location westus

# Enable vulnerability scanning
az acr task create \
  --registry acrprodregistry... \
  --name security-scan \
  --context /dev/null \
  --cmd "mcr.microsoft.com/azure-security-pack:latest {{.Run.Registry}}/myapp:latest" \
  --schedule "0 0 * * *"
```

**Practice Tasks:**
- [ ] Set up ACR with geo-replication
- [ ] Configure ACR Tasks for CI
- [ ] Implement image retention policies
- [ ] Set up webhook for image push
- [ ] Configure ACR with AKS integration

---

### **Week 8-9: Azure Kubernetes Service (AKS) - Part 1**

**Topics:**
- AKS architecture
- AKS networking (kubenet vs Azure CNI)
- Node pools and scaling
- AKS upgrade strategies
- Azure AD integration
- RBAC in AKS

**Production AKS Cluster:**
```bash
# Create AKS cluster
az aks create \
  --resource-group rg-aks \
  --name aks-prod-cluster \
  --location eastus \
  --kubernetes-version 1.28.0 \
  --node-count 3 \
  --node-vm-size Standard_D4s_v3 \
  --network-plugin azure \
  --network-policy calico \
  --service-cidr 172.16.0.0/16 \
  --dns-service-ip 172.16.0.10 \
  --vnet-subnet-id {subnet-id} \
  --enable-managed-identity \
  --enable-aad \
  --enable-azure-rbac \
  --attach-acr acrprodregistry... \
  --load-balancer-sku standard \
  --zones 1 2 3 \
  --enable-cluster-autoscaler \
  --min-count 3 \
  --max-count 10 \
  --enable-addons monitoring \
  --workspace-resource-id {log-analytics-id}

# Get credentials
az aks get-credentials \
  --resource-group rg-aks \
  --name aks-prod-cluster

# Verify
kubectl get nodes
kubectl get pods --all-namespaces

# Add user node pool
az aks nodepool add \
  --resource-group rg-aks \
  --cluster-name aks-prod-cluster \
  --name userpool \
  --node-count 2 \
  --node-vm-size Standard_D8s_v3 \
  --mode User \
  --enable-cluster-autoscaler \
  --min-count 2 \
  --max-count 20 \
  --node-taints workload=user-apps:NoSchedule \
  --labels workload=user-apps

# Scale node pool
az aks nodepool scale \
  --resource-group rg-aks \
  --cluster-name aks-prod-cluster \
  --name userpool \
  --node-count 5

# Upgrade cluster
az aks upgrade \
  --resource-group rg-aks \
  --name aks-prod-cluster \
  --kubernetes-version 1.29.0
```

**Deploy Application:**
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: acrprodregistry.azurecr.io/myapp:v1
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 512Mi
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
  namespace: production
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 80
```

```bash
kubectl apply -f deployment.yaml
kubectl get pods -n production
kubectl get svc -n production
```

**Practice Tasks:**
- [ ] Deploy production AKS with Azure CNI
- [ ] Configure multiple node pools
- [ ] Set up HPA and cluster autoscaler
- [ ] Integrate with ACR
- [ ] Configure Azure AD RBAC

---

### **Week 10: AKS - Part 2 (Advanced)**

**Topics:**
- Ingress controllers (NGINX, Application Gateway)
- AKS monitoring (Azure Monitor, Prometheus)
- Secrets management (Key Vault CSI driver)
- Network policies
- Pod Identity
- GitOps with Flux/ArgoCD

**Ingress Setup:**
```bash
# Install NGINX Ingress
kubectl create namespace ingress-nginx

helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --set controller.replicaCount=2 \
  --set controller.service.loadBalancerIP=<STATIC_IP>

# Create ingress
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  namespace: production
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port:
              number: 80
EOF
```

**Key Vault CSI Driver:**
```bash
# Enable Key Vault provider
az aks enable-addons \
  --resource-group rg-aks \
  --name aks-prod-cluster \
  --addons azure-keyvault-secrets-provider

# Create SecretProviderClass
kubectl apply -f - <<EOF
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: azure-kv-sync
  namespace: production
spec:
  provider: azure
  parameters:
    usePodIdentity: "false"
    useVMManagedIdentity: "true"
    userAssignedIdentityID: ""
    keyvaultName: "kv-prod-secrets"
    objects: |
      array:
        - |
          objectName: db-connection-string
          objectType: secret
          objectVersion: ""
    tenantId: "{tenant-id}"
EOF
```

**Practice Tasks:**
- [ ] Deploy ingress with TLS
- [ ] Configure Azure Monitor for containers
- [ ] Integrate Key Vault with AKS
- [ ] Implement network policies
- [ ] Set up GitOps with Flux

---

## **Week 11-12: Databases & Data Services**

### **Week 11: Azure SQL & Managed Databases**

**Topics:**
- Azure SQL Database
- Azure Database for PostgreSQL/MySQL
- Cosmos DB basics
- Backup and restore
- High availability
- Private endpoints for databases

**Azure SQL:**
```bash
# Create SQL Server
az sql server create \
  --resource-group rg-databases \
  --name sql-prod-server-$(date +%s) \
  --location eastus \
  --admin-user sqladmin \
  --admin-password 'P@ssw0rd1234!' \
  --enable-public-network false

# Create SQL Database
az sql db create \
  --resource-group rg-databases \
  --server sql-prod-server-... \
  --name sqldb-prod \
  --service-objective S1 \
  --zone-redundant false \
  --backup-storage-redundancy Local

# Configure firewall (if needed)
az sql server firewall-rule create \
  --resource-group rg-databases \
  --server sql-prod-server-... \
  --name AllowAzureServices \
  --start-ip-address 0.0.0.0 \
  --end-ip-address 0.0.0.0

# Create private endpoint
az network private-endpoint create \
  --resource-group rg-databases \
  --name pe-sql-server \
  --vnet-name vnet-spoke-prod \
  --subnet subnet-data \
  --private-connection-resource-id {sql-server-id} \
  --group-id sqlServer \
  --connection-name sql-connection

# Configure backup retention
az sql db ltr-policy set \
  --resource-group rg-databases \
  --server sql-prod-server-... \
  --database sqldb-prod \
  --weekly-retention P4W \
  --monthly-retention P12M \
  --yearly-retention P5Y \
  --week-of-year 1
```

**PostgreSQL:**
```bash
# Create PostgreSQL server
az postgres flexible-server create \
  --resource-group rg-databases \
  --name psql-prod-server \
  --location eastus \
  --admin-user pgadmin \
  --admin-password 'P@ssw0rd1234!' \
  --sku-name Standard_D2s_v3 \
  --tier GeneralPurpose \
  --storage-size 128 \
  --version 14 \
  --high-availability ZoneRedundant \
  --zone 1

# Create database
az postgres flexible-server db create \
  --resource-group rg-databases \
  --server-name psql-prod-server \
  --database-name appdb
```

**Practice Tasks:**
- [ ] Deploy Azure SQL with geo-replication
- [ ] Configure automatic backups
- [ ] Set up private endpoints
- [ ] Implement database monitoring
- [ ] Configure read replicas

---

### **Week 12: Monitoring & Logging**

**Topics:**
- Azure Monitor
- Log Analytics
- Application Insights
- Azure Metrics
- Alert rules
- Dashboards

**Log Analytics Setup:**
```bash
# Create Log Analytics Workspace
az monitor log-analytics workspace create \
  --resource-group rg-monitoring \
  --workspace-name law-prod-monitoring \
  --location eastus \
  --sku PerGB2018

# Enable diagnostics for resource
az monitor diagnostic-settings create \
  --resource {resource-id} \
  --name diag-settings \
  --workspace {workspace-id} \
  --logs '[{"category": "Administrative","enabled": true}]' \
  --metrics '[{"category": "AllMetrics","enabled": true}]'

# Query logs (KQL)
az monitor log-analytics query \
  --workspace {workspace-id} \
  --analytics-query "AzureActivity | where TimeGenerated > ago(1h) | project TimeGenerated, Caller, OperationName" \
  --output table
```

**Alerts:**
```bash
# Create action group
az monitor action-group create \
  --resource-group rg-monitoring \
  --name ag-devops-team \
  --short-name DevOps \
  --email-receiver name=DevOpsTeam email=devops@company.com

# Create metric alert
az monitor metrics alert create \
  --resource-group rg-monitoring \
  --name alert-high-cpu \
  --scopes {vm-id} \
  --condition "avg Percentage CPU > 80" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --action ag-devops-team

# Create log alert
az monitor scheduled-query create \
  --resource-group rg-monitoring \
  --name alert-failed-deployments \
  --scopes {workspace-id} \
  --condition "count > 5" \
  --condition-query "AzureActivity | where OperationName contains 'deployment' and ActivityStatus == 'Failed'" \
  --action ag-devops-team
```

**Practice Tasks:**
- [ ] Set up centralized logging
- [ ] Create custom dashboards
- [ ] Configure alerting for critical metrics
- [ ] Implement Application Insights
- [ ] Set up log retention policies

---

## **Week 13-14: DevOps & CI/CD**

### **Week 13: Azure DevOps Basics**

**Topics:**
- Azure DevOps setup
- Azure Repos
- Azure Pipelines (YAML)
- Build pipelines
- Artifact management
- Service connections

**Basic Pipeline:**
```yaml
# azure-pipelines.yml
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
  azureSubscription: 'Azure-Service-Connection'
  resourceGroup: 'rg-app-prod'

stages:
  - stage: Build
    jobs:
      - job: BuildApp
        steps:
          - task: Docker@2
            displayName: 'Build Docker Image'
            inputs:
              command: build
              dockerfile: 'Dockerfile'
              tags: '$(Build.BuildId)'
          
          - task: Docker@2
            displayName: 'Push to ACR'
            inputs:
              command: push
              containerRegistry: 'ACR-Service-Connection'
              repository: 'myapp'
              tags: '$(Build.BuildId)'

  - stage: Deploy
    dependsOn: Build
    jobs:
      - deployment: DeployToAKS
        environment: 'production'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: KubernetesManifest@0
                  displayName: 'Deploy to AKS'
                  inputs:
                    action: 'deploy'
                    kubernetesServiceConnection: 'AKS-Service-Connection'
                    namespace: 'production'
                    manifests: |
                      manifests/deployment.yaml
                      manifests/service.yaml
```

**Practice Tasks:**
- [ ] Create complete CI/CD pipeline
- [ ] Configure multi-stage deployments
- [ ] Implement approval gates
- [ ] Set up artifact versioning
- [ ] Configure pipeline triggers

---

### **Week 14: Infrastructure as Code with Terraform**

**Topics:**
- Terraform with Azure
- Azure DevOps integration
- State management
- Modules
- CI/CD for infrastructure

**Terraform Pipeline:**
```yaml
# terraform-pipeline.yml
trigger:
  branches:
    include:
      - main
  paths:
    include:
      - terraform/**

stages:
  - stage: Validate
    jobs:
      - job: Validate
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          - task: TerraformInstaller@0
            inputs:
              terraformVersion: '1.7.0'
          
          - task: TerraformTaskV4@4
            displayName: 'Terraform Init'
            inputs:
              provider: 'azurerm'
              command: 'init'
              backendServiceArm: 'Azure-Service-Connection'
              backendAzureRmResourceGroupName: 'rg-terraform-state'
              backendAzureRmStorageAccountName: 'tfstate'
              backendAzureRmContainerName: 'tfstate'
              backendAzureRmKey: 'prod.terraform.tfstate'
          
          - task: TerraformTaskV4@4
            displayName: 'Terraform Validate'
            inputs:
              provider: 'azurerm'
              command: 'validate'

  - stage: Plan
    dependsOn: Validate
    jobs:
      - job: Plan
        steps:
          - task: TerraformTaskV4@4
            displayName: 'Terraform Plan'
            inputs:
              provider: 'azurerm'
              command: 'plan'
              environmentServiceNameAzureRM: 'Azure-Service-Connection'
              commandOptions: '-out=tfplan'

  - stage: Apply
    dependsOn: Plan
    jobs:
      - deployment: Apply
        environment: 'production-infrastructure'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: TerraformTaskV4@4
                  displayName: 'Terraform Apply'
                  inputs:
                    provider: 'azurerm'
                    command: 'apply'
                    environmentServiceNameAzureRM: 'Azure-Service-Connection'
```

**Practice Tasks:**
- [ ] Build complete infrastructure with Terraform
- [ ] Set up remote state in Azure Storage
- [ ] Create reusable modules
- [ ] Implement CI/CD for infrastructure
- [ ] Configure drift detection

---

## **Week 15-16: Advanced Topics & Production**

### **Week 15: High Availability & Disaster Recovery**

**Topics:**
- Multi-region architecture
- Traffic Manager
- Azure Front Door
- Backup strategies
- Site Recovery
- Business continuity

**Multi-Region Setup:**
```bash
# Deploy to East US
az group create --name rg-app-eastus --location eastus
# ... deploy resources

# Deploy to West US
az group create --name rg-app-westus --location westus
# ... deploy resources

# Create Traffic Manager profile
az network traffic-manager profile create \
  --resource-group rg-global \
  --name tm-global-app \
  --routing-method Priority \
  --unique-dns-name myapp-global

# Add endpoints
az network traffic-manager endpoint create \
  --resource-group rg-global \
  --profile-name tm-global-app \
  --name endpoint-eastus \
  --type azureEndpoints \
  --target-resource-id {app-gateway-eastus-id} \
  --priority 1

az network traffic-manager endpoint create \
  --resource-group rg-global \
  --profile-name tm-global-app \
  --name endpoint-westus \
  --type azureEndpoints \
  --target-resource-id {app-gateway-westus-id} \
  --priority 2
```

**Practice Tasks:**
- [ ] Design multi-region architecture
- [ ] Implement automated failover
- [ ] Configure backup strategies
- [ ] Test disaster recovery procedures
- [ ] Document recovery procedures

---

### **Week 16: Cost Optimization & Best Practices**

**Topics:**
- Cost management
- Azure Advisor
- Resource optimization
- Reserved instances
- Tagging strategies
- Governance

**Cost Management:**
```bash
# Create budget
az consumption budget create \
  --resource-group rg-app-prod \
  --budget-name monthly-budget \
  --amount 10000 \
  --time-grain Monthly \
  --start-date 2026-01-01 \
  --end-date 2026-12-31

# Add notification
az consumption budget notification create \
  --resource-group rg-app-prod \
  --budget-name monthly-budget \
  --notification-key Warning80 \
  --threshold 80 \
  --contact-emails devops@company.com

# Get cost analysis
az consumption usage list \
  --start-date 2026-01-01 \
  --end-date 2026-01-31
```

**Practice Tasks:**
- [ ] Implement cost tagging
- [ ] Set up budget alerts
- [ ] Review and optimize resources
- [ ] Implement auto-shutdown for dev resources
- [ ] Create cost reports

---

## 📚 Additional Resources

### **Books:**
- "Azure for Architects" by Ritesh Modi
- "Azure DevOps Explained" by Sjoukje Zaal
- "Designing Distributed Systems" by Brendan Burns

### **Certifications:**
- **AZ-104**: Azure Administrator Associate
- **AZ-204**: Azure Developer Associate
- **AZ-400**: DevOps Engineer Expert
- **AZ-305**: Azure Solutions Architect Expert

### **Online Resources:**
- [Microsoft Learn - Azure](https://learn.microsoft.com/azure/)
- [Azure Architecture Center](https://docs.microsoft.com/azure/architecture/)
- [Azure Updates](https://azure.microsoft.com/updates/)

---

## 🎯 Validation & Next Steps

After completing this roadmap:

**Skills Validation:**
- [ ] Can design and deploy Azure infrastructure
- [ ] Implement production AKS clusters
- [ ] Configure networking and security
- [ ] Build CI/CD pipelines
- [ ] Monitor and troubleshoot Azure workloads
- [ ] Optimize costs and performance

**Next Steps:**
1. Get Azure certifications (AZ-104, AZ-400)
2. Build real-world projects
3. Contribute to open-source Azure tools
4. Mentor junior team members
5. Stay updated with Azure announcements

---

**Master Azure for DevOps! Start your journey today! ☁️🚀**

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

