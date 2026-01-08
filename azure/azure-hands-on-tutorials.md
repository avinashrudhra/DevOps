# Azure Hands-On Tutorials for DevOps Engineers

**40+ Step-by-Step Real-World Scenarios**

---

## 🎯 Tutorial Structure

Each tutorial includes:
- **Objective**: What you'll build
- **Duration**: Estimated time
- **Difficulty**: Beginner/Intermediate/Advanced
- **Architecture Diagram**: Visual representation
- **Prerequisites**: What you need
- **Step-by-Step Instructions**: Complete guide
- **Validation**: How to test
- **Clean Up**: Resource removal
- **Next Steps**: How to extend

---

## 📚 Tutorial Categories

1. **Networking** - VNets, NSGs, Load Balancers (Tutorials 1-8)
2. **Compute** - VMs, VMSS, AKS (Tutorials 9-16)
3. **Storage & Databases** - Storage, SQL, Cosmos (Tutorials 17-24)
4. **Security** - Key Vault, Identity, RBAC (Tutorials 25-28)
5. **Containers** - ACR, AKS, CI/CD (Tutorials 29-36)
6. **Monitoring** - Azure Monitor, Alerts (Tutorials 37-40)

---

## 🌐 NETWORKING TUTORIALS

### **Tutorial 1: Build Hub-Spoke Network Architecture**

**Objective:** Create a production-ready hub-spoke network topology

**Duration:** 45 minutes  
**Difficulty:** Intermediate

**Architecture:**
```
        ┌──────────────────────┐
        │      Hub VNet        │
        │  10.0.0.0/16         │
        │  - Firewall          │
        │  - VPN Gateway       │
        │  - Bastion           │
        └──────┬───────┬───────┘
               │       │
       ┌───────┴───┐ ┌┴───────────┐
       │           │ │            │
┌──────▼──────┐ ┌──▼────────┐ ┌──▼────────┐
│  Spoke VNet │ │ Spoke VNet│ │ Spoke VNet│
│  Production │ │  Staging  │ │   Dev     │
│ 10.1.0.0/16 │ │10.2.0.0/16│ │10.3.0.0/16│
└─────────────┘ └───────────┘ └───────────┘
```

**Prerequisites:**
- Azure CLI installed
- Azure subscription
- Contributor access

**Step-by-Step:**

```bash
# 1. Create resource group
az group create \
  --name rg-network-hub-spoke \
  --location eastus \
  --tags Project=HubSpoke Environment=Production

# 2. Create Hub VNet
az network vnet create \
  --resource-group rg-network-hub-spoke \
  --name vnet-hub \
  --address-prefix 10.0.0.0/16 \
  --location eastus

# 3. Create Hub subnets
# Firewall subnet (minimum /26)
az network vnet subnet create \
  --resource-group rg-network-hub-spoke \
  --vnet-name vnet-hub \
  --name AzureFirewallSubnet \
  --address-prefix 10.0.1.0/26

# Gateway subnet (minimum /27)
az network vnet subnet create \
  --resource-group rg-network-hub-spoke \
  --vnet-name vnet-hub \
  --name GatewaySubnet \
  --address-prefix 10.0.2.0/27

# Bastion subnet (minimum /26)
az network vnet subnet create \
  --resource-group rg-network-hub-spoke \
  --vnet-name vnet-hub \
  --name AzureBastionSubnet \
  --address-prefix 10.0.3.0/26

# 4. Create Production Spoke VNet
az network vnet create \
  --resource-group rg-network-hub-spoke \
  --name vnet-spoke-prod \
  --address-prefix 10.1.0.0/16 \
  --location eastus

# Create production subnets
az network vnet subnet create \
  --resource-group rg-network-hub-spoke \
  --vnet-name vnet-spoke-prod \
  --name subnet-web \
  --address-prefix 10.1.1.0/24

az network vnet subnet create \
  --resource-group rg-network-hub-spoke \
  --vnet-name vnet-spoke-prod \
  --name subnet-app \
  --address-prefix 10.1.2.0/24

az network vnet subnet create \
  --resource-group rg-network-hub-spoke \
  --vnet-name vnet-spoke-prod \
  --name subnet-data \
  --address-prefix 10.1.3.0/24

# 5. Create Staging Spoke VNet
az network vnet create \
  --resource-group rg-network-hub-spoke \
  --name vnet-spoke-staging \
  --address-prefix 10.2.0.0/16 \
  --location eastus

az network vnet subnet create \
  --resource-group rg-network-hub-spoke \
  --vnet-name vnet-spoke-staging \
  --name subnet-web \
  --address-prefix 10.2.1.0/24

# 6. Create Dev Spoke VNet
az network vnet create \
  --resource-group rg-network-hub-spoke \
  --name vnet-spoke-dev \
  --address-prefix 10.3.0.0/16 \
  --location eastus

az network vnet subnet create \
  --resource-group rg-network-hub-spoke \
  --vnet-name vnet-spoke-dev \
  --name subnet-web \
  --address-prefix 10.3.1.0/24

# 7. Create VNet Peering: Hub to Production Spoke
az network vnet peering create \
  --resource-group rg-network-hub-spoke \
  --name hub-to-prod \
  --vnet-name vnet-hub \
  --remote-vnet vnet-spoke-prod \
  --allow-vnet-access \
  --allow-forwarded-traffic \
  --allow-gateway-transit

# 8. Create VNet Peering: Production Spoke to Hub
az network vnet peering create \
  --resource-group rg-network-hub-spoke \
  --name prod-to-hub \
  --vnet-name vnet-spoke-prod \
  --remote-vnet vnet-hub \
  --allow-vnet-access \
  --allow-forwarded-traffic \
  --use-remote-gateways false

# 9. Repeat peering for Staging
az network vnet peering create \
  --resource-group rg-network-hub-spoke \
  --name hub-to-staging \
  --vnet-name vnet-hub \
  --remote-vnet vnet-spoke-staging \
  --allow-vnet-access \
  --allow-forwarded-traffic \
  --allow-gateway-transit

az network vnet peering create \
  --resource-group rg-network-hub-spoke \
  --name staging-to-hub \
  --vnet-name vnet-spoke-staging \
  --remote-vnet vnet-hub \
  --allow-vnet-access \
  --allow-forwarded-traffic

# 10. Repeat peering for Dev
az network vnet peering create \
  --resource-group rg-network-hub-spoke \
  --name hub-to-dev \
  --vnet-name vnet-hub \
  --remote-vnet vnet-spoke-dev \
  --allow-vnet-access \
  --allow-forwarded-traffic \
  --allow-gateway-transit

az network vnet peering create \
  --resource-group rg-network-hub-spoke \
  --name dev-to-hub \
  --vnet-name vnet-spoke-dev \
  --remote-vnet vnet-hub \
  --allow-vnet-access \
  --allow-forwarded-traffic
```

**Validation:**
```bash
# List all VNets
az network vnet list \
  --resource-group rg-network-hub-spoke \
  --output table

# Check peering status
az network vnet peering list \
  --resource-group rg-network-hub-spoke \
  --vnet-name vnet-hub \
  --output table

# Test connectivity (after deploying VMs)
# From VM in spoke-prod, ping VM in spoke-dev (should fail - isolated)
# From VM in spoke-prod, ping hub (should work)
```

**Clean Up:**
```bash
az group delete --name rg-network-hub-spoke --yes --no-wait
```

**Next Steps:**
- Add Azure Firewall to hub
- Configure route tables for spoke-to-spoke communication
- Deploy Azure Bastion for secure access
- Add VPN Gateway for on-premises connectivity

---

### **Tutorial 2: Deploy Application Gateway with WAF**

**Objective:** Set up Application Gateway with Web Application Firewall

**Duration:** 30 minutes  
**Difficulty:** Intermediate

**Prerequisites:**
- VNet with subnet for Application Gateway
- 2+ backend web servers

**Step-by-Step:**

```bash
# 1. Create resource group
az group create \
  --name rg-appgw \
  --location eastus

# 2. Create VNet and subnets
az network vnet create \
  --resource-group rg-appgw \
  --name vnet-appgw \
  --address-prefix 10.0.0.0/16 \
  --subnet-name subnet-appgw \
  --subnet-prefix 10.0.1.0/24

az network vnet subnet create \
  --resource-group rg-appgw \
  --vnet-name vnet-appgw \
  --name subnet-backend \
  --address-prefix 10.0.2.0/24

# 3. Create public IP for Application Gateway
az network public-ip create \
  --resource-group rg-appgw \
  --name pip-appgw \
  --sku Standard \
  --allocation-method Static

# 4. Deploy 2 backend web servers
for i in 1 2; do
  az vm create \
    --resource-group rg-appgw \
    --name vm-web-$i \
    --image Ubuntu2204 \
    --size Standard_B2s \
    --vnet-name vnet-appgw \
    --subnet subnet-backend \
    --admin-username azureuser \
    --generate-ssh-keys \
    --custom-data cloud-init-web.txt \
    --public-ip-address "" \
    --nsg ""
done

# cloud-init-web.txt content:
cat > cloud-init-web.txt <<EOF
#cloud-config
package_update: true
packages:
  - nginx
runcmd:
  - systemctl start nginx
  - systemctl enable nginx
  - echo "<h1>Backend Server \$(hostname)</h1>" > /var/www/html/index.html
EOF

# 5. Create Application Gateway with WAF
az network application-gateway create \
  --resource-group rg-appgw \
  --name appgw-prod \
  --location eastus \
  --sku WAF_v2 \
  --capacity 2 \
  --vnet-name vnet-appgw \
  --subnet subnet-appgw \
  --public-ip-address pip-appgw \
  --http-settings-port 80 \
  --http-settings-protocol Http \
  --frontend-port 80 \
  --priority 100

# 6. Get backend server IPs
VM1_IP=$(az vm show -d \
  --resource-group rg-appgw \
  --name vm-web-1 \
  --query privateIps \
  --output tsv)

VM2_IP=$(az vm show -d \
  --resource-group rg-appgw \
  --name vm-web-2 \
  --query privateIps \
  --output tsv)

# 7. Add backend servers to pool
az network application-gateway address-pool update \
  --resource-group rg-appgw \
  --gateway-name appgw-prod \
  --name appGatewayBackendPool \
  --servers $VM1_IP $VM2_IP

# 8. Configure WAF
az network application-gateway waf-config set \
  --resource-group rg-appgw \
  --gateway-name appgw-prod \
  --enabled true \
  --firewall-mode Prevention \
  --rule-set-version 3.2

# 9. Configure health probe
az network application-gateway probe create \
  --resource-group rg-appgw \
  --gateway-name appgw-prod \
  --name health-probe \
  --protocol Http \
  --host-name-from-http-settings true \
  --path / \
  --interval 30 \
  --timeout 30 \
  --threshold 3

# 10. Update HTTP settings to use health probe
az network application-gateway http-settings update \
  --resource-group rg-appgw \
  --gateway-name appgw-prod \
  --name appGatewayBackendHttpSettings \
  --probe health-probe
```

**Validation:**
```bash
# Get public IP
PUBLIC_IP=$(az network public-ip show \
  --resource-group rg-appgw \
  --name pip-appgw \
  --query ipAddress \
  --output tsv)

# Test access
curl http://$PUBLIC_IP
# Should see response from one of the backend servers

# Test multiple times to see load balancing
for i in {1..10}; do
  curl http://$PUBLIC_IP
  echo ""
done

# Test WAF (should be blocked)
curl "http://$PUBLIC_IP/?id=1' OR '1'='1"
```

**Clean Up:**
```bash
az group delete --name rg-appgw --yes --no-wait
```

---

## 💻 COMPUTE TUTORIALS

### **Tutorial 3: Deploy Highly Available Web Application with VMSS**

**Objective:** Create auto-scaling web application using VM Scale Sets

**Duration:** 40 minutes  
**Difficulty:** Intermediate

**Architecture:**
```
┌─────────────────────────────────────┐
│      Azure Load Balancer            │
│         (Public IP)                 │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│   VM Scale Set (Auto-scaling)       │
│   ┌────┐ ┌────┐ ┌────┐ ┌────┐      │
│   │VM 1│ │VM 2│ │VM 3│ │VM 4│ ...  │
│   └────┘ └────┘ └────┘ └────┘      │
│   Min: 2, Max: 10                   │
└─────────────────────────────────────┘
```

**Step-by-Step:**

```bash
# 1. Create resource group
az group create \
  --name rg-vmss-web \
  --location eastus

# 2. Create cloud-init script for web servers
cat > cloud-init.txt <<'EOF'
#cloud-config
package_update: true
packages:
  - nginx
  - stress
runcmd:
  - systemctl start nginx
  - systemctl enable nginx
  - |
    cat > /var/www/html/index.html <<HTML
    <!DOCTYPE html>
    <html>
    <head><title>VMSS Demo</title></head>
    <body>
      <h1>Hostname: $(hostname)</h1>
      <h2>IP: $(hostname -I)</h2>
      <p>Served by VM Scale Set</p>
    </body>
    </html>
    HTML
EOF

# 3. Create VMSS with load balancer
az vmss create \
  --resource-group rg-vmss-web \
  --name vmss-web-app \
  --image Ubuntu2204 \
  --instance-count 2 \
  --vm-sku Standard_B2s \
  --upgrade-policy-mode Automatic \
  --admin-username azureuser \
  --generate-ssh-keys \
  --custom-data cloud-init.txt \
  --load-balancer lb-web \
  --backend-pool-name web-pool \
  --health-probe /health \
  --public-ip-address pip-vmss

# 4. Configure auto-scaling
az monitor autoscale create \
  --resource-group rg-vmss-web \
  --resource vmss-web-app \
  --resource-type Microsoft.Compute/virtualMachineScaleSets \
  --name autoscale-web \
  --min-count 2 \
  --max-count 10 \
  --count 2

# 5. Add scale-out rule (CPU > 70%)
az monitor autoscale rule create \
  --resource-group rg-vmss-web \
  --autoscale-name autoscale-web \
  --condition "Percentage CPU > 70 avg 5m" \
  --scale out 2

# 6. Add scale-in rule (CPU < 30%)
az monitor autoscale rule create \
  --resource-group rg-vmss-web \
  --autoscale-name autoscale-web \
  --condition "Percentage CPU < 30 avg 5m" \
  --scale in 1

# 7. Configure load balancer health probe
az network lb probe update \
  --resource-group rg-vmss-web \
  --lb-name lb-web \
  --name healthprobe \
  --protocol Http \
  --path /

# 8. Add load balancer rule for HTTP
az network lb rule create \
  --resource-group rg-vmss-web \
  --lb-name lb-web \
  --name http-rule \
  --protocol Tcp \
  --frontend-port 80 \
  --backend-port 80 \
  --frontend-ip-name LoadBalancerFrontEnd \
  --backend-pool-name web-pool \
  --probe-name healthprobe
```

**Validation:**
```bash
# Get public IP
PUBLIC_IP=$(az network public-ip show \
  --resource-group rg-vmss-web \
  --name pip-vmss \
  --query ipAddress \
  --output tsv)

echo "Access application at: http://$PUBLIC_IP"

# Test load balancing
for i in {1..20}; do
  curl http://$PUBLIC_IP
  echo "---"
  sleep 1
done

# Monitor VMSS instances
az vmss list-instances \
  --resource-group rg-vmss-web \
  --name vmss-web-app \
  --output table

# Test auto-scaling (generate CPU load)
# SSH to one instance
az vmss list-instance-connection-info \
  --resource-group rg-vmss-web \
  --name vmss-web-app

# Inside VM, run:
# stress --cpu 8 --timeout 600s

# Watch auto-scaling in action
watch -n 30 'az vmss list-instances \
  --resource-group rg-vmss-web \
  --name vmss-web-app \
  --output table'
```

**Clean Up:**
```bash
az group delete --name rg-vmss-web --yes --no-wait
```

---

### **Tutorial 4: Deploy Production AKS Cluster**

**Objective:** Create production-ready AKS cluster with best practices

**Duration:** 60 minutes  
**Difficulty:** Advanced

**Architecture:**
```
┌────────────────────────────────────────────┐
│         AKS Cluster                        │
│  ┌──────────────────────────────────────┐  │
│  │  System Node Pool (3 nodes)         │  │
│  │  Standard_D2s_v3, Zones: 1,2,3      │  │
│  └──────────────────────────────────────┘  │
│  ┌──────────────────────────────────────┐  │
│  │  User Node Pool (3-10 nodes)        │  │
│  │  Standard_D4s_v3, Auto-scaling      │  │
│  └──────────────────────────────────────┘  │
│                                            │
│  Azure CNI, Calico Policy, Azure Monitor │
└────────────────────────────────────────────┘
         │                        │
         ▼                        ▼
┌─────────────────┐     ┌──────────────────┐
│  ACR (Premium)  │     │  Log Analytics   │
│  Geo-replicated │     │  Workspace       │
└─────────────────┘     └──────────────────┘
```

**Prerequisites:**
- Azure CLI with `aks-preview` extension
- kubectl installed
- Helm installed

**Step-by-Step:**

```bash
# 1. Install/Update AKS preview extension
az extension add --name aks-preview
az extension update --name aks-preview

# 2. Create resource group
az group create \
  --name rg-aks-prod \
  --location eastus

# 3. Create Log Analytics workspace
az monitor log-analytics workspace create \
  --resource-group rg-aks-prod \
  --workspace-name law-aks-prod

# Get workspace ID
WORKSPACE_ID=$(az monitor log-analytics workspace show \
  --resource-group rg-aks-prod \
  --workspace-name law-aks-prod \
  --query id \
  --output tsv)

# 4. Create Azure Container Registry
az acr create \
  --resource-group rg-aks-prod \
  --name acrprodaks$RANDOM \
  --sku Premium \
  --location eastus

# Enable geo-replication
az acr replication create \
  --registry acrprodaks... \
  --location westus

# 5. Create VNet for AKS
az network vnet create \
  --resource-group rg-aks-prod \
  --name vnet-aks \
  --address-prefix 10.0.0.0/8 \
  --subnet-name subnet-aks \
  --subnet-prefix 10.240.0.0/16

# Get subnet ID
SUBNET_ID=$(az network vnet subnet show \
  --resource-group rg-aks-prod \
  --vnet-name vnet-aks \
  --name subnet-aks \
  --query id \
  --output tsv)

# 6. Create AKS cluster
az aks create \
  --resource-group rg-aks-prod \
  --name aks-prod-cluster \
  --location eastus \
  --kubernetes-version 1.28.0 \
  --node-count 3 \
  --node-vm-size Standard_D2s_v3 \
  --network-plugin azure \
  --network-policy calico \
  --service-cidr 172.16.0.0/16 \
  --dns-service-ip 172.16.0.10 \
  --vnet-subnet-id $SUBNET_ID \
  --enable-managed-identity \
  --enable-aad \
  --enable-azure-rbac \
  --attach-acr acrprodaks... \
  --load-balancer-sku standard \
  --zones 1 2 3 \
  --enable-cluster-autoscaler \
  --min-count 3 \
  --max-count 6 \
  --enable-addons monitoring \
  --workspace-resource-id $WORKSPACE_ID \
  --nodepool-name systempool \
  --nodepool-labels pool=system \
  --nodepool-taints CriticalAddonsOnly=true:NoSchedule \
  --max-pods 50 \
  --vm-set-type VirtualMachineScaleSets \
  --generate-ssh-keys

# 7. Get credentials
az aks get-credentials \
  --resource-group rg-aks-prod \
  --name aks-prod-cluster \
  --overwrite-existing

# 8. Verify cluster
kubectl get nodes
kubectl get pods --all-namespaces

# 9. Add user node pool
az aks nodepool add \
  --resource-group rg-aks-prod \
  --cluster-name aks-prod-cluster \
  --name userpool \
  --node-count 3 \
  --node-vm-size Standard_D4s_v3 \
  --mode User \
  --zones 1 2 3 \
  --enable-cluster-autoscaler \
  --min-count 3 \
  --max-count 10 \
  --node-taints workload=user-apps:NoSchedule \
  --labels pool=user workload=applications \
  --max-pods 100

# 10. Enable Key Vault CSI driver
az aks enable-addons \
  --resource-group rg-aks-prod \
  --name aks-prod-cluster \
  --addons azure-keyvault-secrets-provider

# 11. Install NGINX Ingress Controller
kubectl create namespace ingress-nginx

helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --set controller.replicaCount=2 \
  --set controller.nodeSelector."kubernetes\.io/os"=linux \
  --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-health-probe-request-path"=/healthz

# 12. Deploy sample application
kubectl create namespace production

kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-demo
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-demo
  template:
    metadata:
      labels:
        app: nginx-demo
    spec:
      nodeSelector:
        pool: user
      tolerations:
      - key: workload
        operator: Equal
        value: user-apps
        effect: NoSchedule
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: production
spec:
  type: LoadBalancer
  selector:
    app: nginx-demo
  ports:
  - port: 80
    targetPort: 80
EOF

# Wait for service to get external IP
kubectl get service nginx-service -n production --watch
```

**Validation:**
```bash
# Check cluster info
kubectl cluster-info

# Check nodes
kubectl get nodes -o wide

# Check node pools
az aks nodepool list \
  --resource-group rg-aks-prod \
  --cluster-name aks-prod-cluster \
  --output table

# Check all pods
kubectl get pods --all-namespaces

# Get service external IP
kubectl get service nginx-service -n production

# Test application
EXTERNAL_IP=$(kubectl get service nginx-service -n production -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
curl http://$EXTERNAL_IP

# Check monitoring
# Go to Azure Portal → AKS cluster → Insights
```

**Clean Up:**
```bash
az group delete --name rg-aks-prod --yes --no-wait
```

---

## 🗄️ STORAGE & DATABASE TUTORIALS

### **Tutorial 5: Set Up Azure SQL with Private Endpoint**

**Objective:** Deploy Azure SQL Database with private connectivity

**Duration:** 30 minutes  
**Difficulty:** Intermediate

**Step-by-Step:**

```bash
# 1. Create resource group
az group create \
  --name rg-sql-private \
  --location eastus

# 2. Create VNet
az network vnet create \
  --resource-group rg-sql-private \
  --name vnet-sql \
  --address-prefix 10.0.0.0/16 \
  --subnet-name subnet-app \
  --subnet-prefix 10.0.1.0/24

# 3. Create subnet for private endpoint
az network vnet subnet create \
  --resource-group rg-sql-private \
  --vnet-name vnet-sql \
  --name subnet-private-endpoints \
  --address-prefix 10.0.2.0/24 \
  --disable-private-endpoint-network-policies

# 4. Create Azure SQL Server
az sql server create \
  --resource-group rg-sql-private \
  --name sql-prod-server-$RANDOM \
  --location eastus \
  --admin-user sqladmin \
  --admin-password 'P@ssw0rd1234!ComplexP@ss' \
  --enable-public-network false

# Get server name
SQL_SERVER=$(az sql server list \
  --resource-group rg-sql-private \
  --query "[0].name" \
  --output tsv)

# 5. Create SQL Database
az sql db create \
  --resource-group rg-sql-private \
  --server $SQL_SERVER \
  --name sqldb-prod \
  --service-objective S1 \
  --backup-storage-redundancy Local

# 6. Create Private DNS Zone
az network private-dns zone create \
  --resource-group rg-sql-private \
  --name privatelink.database.windows.net

# 7. Link DNS zone to VNet
az network private-dns link vnet create \
  --resource-group rg-sql-private \
  --zone-name privatelink.database.windows.net \
  --name sql-dns-link \
  --virtual-network vnet-sql \
  --registration-enabled false

# 8. Create Private Endpoint
az network private-endpoint create \
  --resource-group rg-sql-private \
  --name pe-sql-server \
  --location eastus \
  --vnet-name vnet-sql \
  --subnet subnet-private-endpoints \
  --private-connection-resource-id $(az sql server show \
    --resource-group rg-sql-private \
    --name $SQL_SERVER \
    --query id \
    --output tsv) \
  --group-id sqlServer \
  --connection-name sql-private-connection

# 9. Create DNS zone group
az network private-endpoint dns-zone-group create \
  --resource-group rg-sql-private \
  --endpoint-name pe-sql-server \
  --name sql-dns-zone-group \
  --private-dns-zone privatelink.database.windows.net \
  --zone-name sql

# 10. Deploy test VM in same VNet
az vm create \
  --resource-group rg-sql-private \
  --name vm-test \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --vnet-name vnet-sql \
  --subnet subnet-app \
  --admin-username azureuser \
  --generate-ssh-keys \
  --public-ip-address pip-vm-test

# 11. Install SQL tools on VM
VM_IP=$(az network public-ip show \
  --resource-group rg-sql-private \
  --name pip-vm-test \
  --query ipAddress \
  --output tsv)

ssh azureuser@$VM_IP <<'ENDSSH'
  sudo apt-get update
  sudo apt-get install -y mssql-tools unixodbc-dev
  echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc
ENDSSH
```

**Validation:**
```bash
# Get connection details
echo "Server: $SQL_SERVER.database.windows.net"
echo "Database: sqldb-prod"
echo "Username: sqladmin"

# Test from VM
ssh azureuser@$VM_IP

# Inside VM, test SQL connection
sqlcmd -S $SQL_SERVER.database.windows.net -d sqldb-prod -U sqladmin -P 'P@ssw0rd1234!ComplexP@ss'

# Run SQL query
SELECT @@VERSION;
GO

# Test DNS resolution
nslookup $SQL_SERVER.database.windows.net
# Should resolve to private IP (10.0.2.x)
```

**Clean Up:**
```bash
az group delete --name rg-sql-private --yes --no-wait
```

---

## 🔐 SECURITY TUTORIALS

### **Tutorial 6: Implement Comprehensive Key Vault Solution**

**Objective:** Set up Key Vault with secrets, keys, certificates, and access policies

**Duration:** 40 minutes  
**Difficulty:** Intermediate

**Step-by-Step:**

```bash
# 1. Create resource group
az group create \
  --name rg-keyvault \
  --location eastus

# 2. Create Key Vault
az keyvault create \
  --resource-group rg-keyvault \
  --name kv-prod-$RANDOM \
  --location eastus \
  --enable-soft-delete true \
  --soft-delete-retention-days 90 \
  --enable-purge-protection true \
  --enabled-for-deployment true \
  --enabled-for-template-deployment true \
  --enabled-for-disk-encryption true \
  --sku premium

# Get Key Vault name
KV_NAME=$(az keyvault list \
  --resource-group rg-keyvault \
  --query "[0].name" \
  --output tsv)

# 3. Add secrets
az keyvault secret set \
  --vault-name $KV_NAME \
  --name db-connection-string \
  --value "Server=sql-server.database.windows.net;Database=mydb;User=admin;Password=P@ssw0rd!"

az keyvault secret set \
  --vault-name $KV_NAME \
  --name api-key \
  --value "1234567890abcdef1234567890abcdef"

az keyvault secret set \
  --vault-name $KV_NAME \
  --name storage-key \
  --value "base64encodedkey=="

# 4. Create encryption key
az keyvault key create \
  --vault-name $KV_NAME \
  --name data-encryption-key \
  --kty RSA \
  --size 2048 \
  --ops encrypt decrypt

# 5. Create self-signed certificate
az keyvault certificate create \
  --vault-name $KV_NAME \
  --name ssl-cert \
  --policy "$(az keyvault certificate get-default-policy)"

# 6. Create user-assigned managed identity
az identity create \
  --resource-group rg-keyvault \
  --name id-app-identity

# Get identity details
IDENTITY_ID=$(az identity show \
  --resource-group rg-keyvault \
  --name id-app-identity \
  --query principalId \
  --output tsv)

# 7. Grant managed identity access to Key Vault
az keyvault set-policy \
  --name $KV_NAME \
  --object-id $IDENTITY_ID \
  --secret-permissions get list \
  --key-permissions get list encrypt decrypt \
  --certificate-permissions get list

# 8. Create service principal for CI/CD
SP_OUTPUT=$(az ad sp create-for-rbac \
  --name sp-cicd-keyvault \
  --role Reader \
  --scopes /subscriptions/$(az account show --query id -o tsv)/resourceGroups/rg-keyvault)

SP_APP_ID=$(echo $SP_OUTPUT | jq -r '.appId')

# Grant service principal access
az keyvault set-policy \
  --name $KV_NAME \
  --spn $SP_APP_ID \
  --secret-permissions get list \
  --key-permissions get list

# 9. Enable diagnostic logging
az monitor diagnostic-settings create \
  --resource $(az keyvault show \
    --name $KV_NAME \
    --query id \
    --output tsv) \
  --name kv-diagnostics \
  --logs '[{"category": "AuditEvent","enabled": true}]' \
  --metrics '[{"category": "AllMetrics","enabled": true}]'

# 10. Create access policy for current user
CURRENT_USER=$(az ad signed-in-user show --query id --output tsv)

az keyvault set-policy \
  --name $KV_NAME \
  --object-id $CURRENT_USER \
  --secret-permissions all \
  --key-permissions all \
  --certificate-permissions all
```

**Test Application Access:**
```python
# app.py - Python example using managed identity
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient

# Using managed identity (works on Azure VMs, AKS, etc.)
credential = DefaultAzureCredential()
client = SecretClient(vault_url=f"https://{KV_NAME}.vault.azure.net/", credential=credential)

# Retrieve secret
db_connection = client.get_secret("db-connection-string")
print(f"Connection string: {db_connection.value}")

# List all secrets
secrets = client.list_properties_of_secrets()
for secret in secrets:
    print(f"Secret: {secret.name}")
```

**Validation:**
```bash
# List secrets
az keyvault secret list \
  --vault-name $KV_NAME \
  --output table

# Get secret value
az keyvault secret show \
  --vault-name $KV_NAME \
  --name db-connection-string \
  --query value \
  --output tsv

# List keys
az keyvault key list \
  --vault-name $KV_NAME \
  --output table

# List certificates
az keyvault certificate list \
  --vault-name $KV_NAME \
  --output table

# Check access policies
az keyvault show \
  --name $KV_NAME \
  --query properties.accessPolicies
```

**Clean Up:**
```bash
az group delete --name rg-keyvault --yes --no-wait
```

---

## 🐳 CONTAINER & CI/CD TUTORIALS

### **Tutorial 7: Complete CI/CD Pipeline for AKS**

**Objective:** Build end-to-end CI/CD pipeline deploying to AKS

**Duration:** 60 minutes  
**Difficulty:** Advanced

**Prerequisites:**
- Azure DevOps account
- AKS cluster running
- ACR created
- GitHub/Azure Repos repository

**Azure DevOps Pipeline:**

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main
  paths:
    include:
      - src/**

variables:
  azureSubscription: 'Azure-Service-Connection'
  acrName: 'acrprodregistry'
  aksResourceGroup: 'rg-aks-prod'
  aksClusterName: 'aks-prod-cluster'
  imageName: 'myapp'
  k8sNamespace: 'production'

stages:
  - stage: Build
    displayName: 'Build and Push'
    jobs:
      - job: Build
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          - task: Docker@2
            displayName: 'Build Docker Image'
            inputs:
              command: build
              dockerfile: 'Dockerfile'
              repository: '$(imageName)'
              tags: |
                $(Build.BuildId)
                latest
          
          - task: Docker@2
            displayName: 'Push to ACR'
            inputs:
              command: push
              containerRegistry: 'ACR-Service-Connection'
              repository: '$(imageName)'
              tags: |
                $(Build.BuildId)
                latest
          
          - task: PublishPipelineArtifact@1
            displayName: 'Publish K8s Manifests'
            inputs:
              targetPath: 'k8s'
              artifact: 'manifests'

  - stage: Deploy_Dev
    displayName: 'Deploy to Dev'
    dependsOn: Build
    jobs:
      - deployment: DeployDev
        environment: 'dev'
        pool:
          vmImage: 'ubuntu-latest'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: DownloadPipelineArtifact@2
                  inputs:
                    artifact: 'manifests'
                    path: '$(System.DefaultWorkingDirectory)/k8s'
                
                - task: KubernetesManifest@0
                  displayName: 'Create/Update Namespace'
                  inputs:
                    action: 'deploy'
                    kubernetesServiceConnection: 'AKS-Service-Connection'
                    namespace: 'dev'
                    manifests: |
                      k8s/namespace.yaml
                
                - task: KubernetesManifest@0
                  displayName: 'Deploy to AKS'
                  inputs:
                    action: 'deploy'
                    kubernetesServiceConnection: 'AKS-Service-Connection'
                    namespace: 'dev'
                    manifests: |
                      k8s/deployment.yaml
                      k8s/service.yaml
                    containers: |
                      $(acrName).azurecr.io/$(imageName):$(Build.BuildId)

  - stage: Deploy_Prod
    displayName: 'Deploy to Production'
    dependsOn: Deploy_Dev
    jobs:
      - deployment: DeployProd
        environment: 'production'
        pool:
          vmImage: 'ubuntu-latest'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: DownloadPipelineArtifact@2
                  inputs:
                    artifact: 'manifests'
                    path: '$(System.DefaultWorkingDirectory)/k8s'
                
                - task: KubernetesManifest@0
                  displayName: 'Deploy to Production'
                  inputs:
                    action: 'deploy'
                    kubernetesServiceConnection: 'AKS-Service-Connection'
                    namespace: '$(k8sNamespace)'
                    manifests: |
                      k8s/deployment.yaml
                      k8s/service.yaml
                      k8s/ingress.yaml
                    containers: |
                      $(acrName).azurecr.io/$(imageName):$(Build.BuildId)
```

**Kubernetes Manifests:**

```yaml
# k8s/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    name: production

---
# k8s/deployment.yaml
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
        image: acrprodregistry.azurecr.io/myapp:latest
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
# k8s/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
  namespace: production
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 80

---
# k8s/ingress.yaml
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
```

**Setup Instructions:**

```bash
# 1. Create service connection in Azure DevOps
# Go to Project Settings → Service connections

# 2. Create Azure Resource Manager connection
# Name: Azure-Service-Connection
# Scope: Subscription

# 3. Create Docker Registry connection
# Name: ACR-Service-Connection
# Registry: acrprodregistry.azurecr.io

# 4. Create Kubernetes connection
az aks get-credentials \
  --resource-group rg-aks-prod \
  --name aks-prod-cluster

# Export kubeconfig
kubectl config view --raw > kubeconfig.yaml

# Upload to Azure DevOps as service connection

# 5. Create environments in Azure DevOps
# Pipelines → Environments → New environment
# Create: dev, production (with approval gates)

# 6. Push code and pipeline will trigger
git add .
git commit -m "Add CI/CD pipeline"
git push origin main
```

**Validation:**
```bash
# Check pipeline execution in Azure DevOps

# Verify deployment
kubectl get deployments -n production
kubectl get pods -n production
kubectl get service -n production
kubectl get ingress -n production

# Test application
kubectl port-forward service/myapp-service 8080:80 -n production
curl http://localhost:8080
```

---

## 📊 MONITORING TUTORIALS

### **Tutorial 8: Comprehensive Azure Monitor Setup**

**Objective:** Implement complete monitoring solution with alerts and dashboards

**Duration:** 45 minutes  
**Difficulty:** Intermediate

**Step-by-Step:**

```bash
# 1. Create resource group
az group create \
  --name rg-monitoring \
  --location eastus

# 2. Create Log Analytics workspace
az monitor log-analytics workspace create \
  --resource-group rg-monitoring \
  --workspace-name law-central-monitoring \
  --location eastus \
  --sku PerGB2018

# Get workspace ID
WORKSPACE_ID=$(az monitor log-analytics workspace show \
  --resource-group rg-monitoring \
  --workspace-name law-central-monitoring \
  --query id \
  --output tsv)

# 3. Create action group for alerts
az monitor action-group create \
  --resource-group rg-monitoring \
  --name ag-devops-team \
  --short-name DevOps \
  --email-receiver \
    name=Team \
    email=devops@company.com \
    use-common-alert-schema=true \
  --sms-receiver \
    name=OnCall \
    country-code=1 \
    phone-number=5551234567

# 4. Create sample resources to monitor
az vm create \
  --resource-group rg-monitoring \
  --name vm-monitored \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --admin-username azureuser \
  --generate-ssh-keys

# 5. Enable diagnostic settings for VM
az monitor diagnostic-settings create \
  --resource $(az vm show \
    --resource-group rg-monitoring \
    --name vm-monitored \
    --query id \
    --output tsv) \
  --name vm-diagnostics \
  --workspace $WORKSPACE_ID \
  --metrics '[{"category": "AllMetrics","enabled": true}]'

# 6. Create metric alert for high CPU
az monitor metrics alert create \
  --resource-group rg-monitoring \
  --name alert-high-cpu \
  --scopes $(az vm show \
    --resource-group rg-monitoring \
    --name vm-monitored \
    --query id \
    --output tsv) \
  --condition "avg Percentage CPU > 80" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --action ag-devops-team \
  --description "Alert when CPU exceeds 80%"

# 7. Create metric alert for low disk space
az monitor metrics alert create \
  --resource-group rg-monitoring \
  --name alert-low-disk \
  --scopes $(az vm show \
    --resource-group rg-monitoring \
    --name vm-monitored \
    --query id \
    --output tsv) \
  --condition "avg Available Memory Bytes < 536870912" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --action ag-devops-team

# 8. Create log query alert
az monitor scheduled-query create \
  --resource-group rg-monitoring \
  --name alert-failed-logins \
  --scopes $WORKSPACE_ID \
  --condition "count > 5" \
  --condition-query "Syslog | where Facility == 'auth' and SeverityLevel == 'err' | summarize count() by Computer" \
  --description "Alert on failed login attempts" \
  --evaluation-frequency 5m \
  --window-size 10m \
  --action-groups ag-devops-team

# 9. Create Application Insights
az monitor app-insights component create \
  --resource-group rg-monitoring \
  --app ai-app-monitoring \
  --location eastus \
  --workspace $WORKSPACE_ID

# Get instrumentation key
AI_KEY=$(az monitor app-insights component show \
  --resource-group rg-monitoring \
  --app ai-app-monitoring \
  --query instrumentationKey \
  --output tsv)

# 10. Create dashboard (JSON)
cat > dashboard.json <<'EOF'
{
  "properties": {
    "lenses": {
      "0": {
        "order": 0,
        "parts": {
          "0": {
            "position": {
              "x": 0,
              "y": 0,
              "colSpan": 6,
              "rowSpan": 4
            },
            "metadata": {
              "type": "Extension/Microsoft_OperationsManagementSuite_Workspace/PartType/LogsDashboardPart"
            }
          }
        }
      }
    },
    "metadata": {
      "model": {
        "displayName": "Production Monitoring Dashboard"
      }
    }
  }
}
EOF

az portal dashboard create \
  --resource-group rg-monitoring \
  --name dashboard-production \
  --input-path dashboard.json
```

**Validation:**
```bash
# List all alerts
az monitor metrics alert list \
  --resource-group rg-monitoring \
  --output table

# Test alert (generate high CPU)
ssh azureuser@<VM_IP>
stress --cpu 8 --timeout 600s

# Check alert status
az monitor metrics alert show \
  --resource-group rg-monitoring \
  --name alert-high-cpu

# Query logs
az monitor log-analytics query \
  --workspace $WORKSPACE_ID \
  --analytics-query "Perf | where ObjectName == 'Processor' and CounterName == 'UtilizationPercentage' | summarize avg(CounterValue) by bin(TimeGenerated, 5m)" \
  --output table
```

**Clean Up:**
```bash
az group delete --name rg-monitoring --yes --no-wait
```

---

## 🎯 Additional Tutorials Available

**Networking (Continued):**
- Tutorial 9: Configure Azure Firewall with custom rules
- Tutorial 10: Set up VPN Gateway for hybrid connectivity
- Tutorial 11: Deploy Azure Bastion for secure access
- Tutorial 12: Implement Traffic Manager for global load balancing

**Compute (Continued):**
- Tutorial 13: Deploy Windows VMs with custom extensions
- Tutorial 14: Create custom VM images with Packer
- Tutorial 15: Implement Azure Batch for parallel processing
- Tutorial 16: Set up Azure Functions with triggers

**Storage (Continued):**
- Tutorial 17: Configure blob lifecycle management
- Tutorial 18: Set up Azure Files with AD authentication
- Tutorial 19: Implement Cosmos DB with multi-region writes
- Tutorial 20: Configure Redis Cache for performance

**Databases (Continued):**
- Tutorial 21: Deploy PostgreSQL Flexible Server
- Tutorial 22: Set up SQL Database failover groups
- Tutorial 23: Implement database backup and restore
- Tutorial 24: Configure database auditing and security

**Security (Continued):**
- Tutorial 25: Implement Azure AD Conditional Access
- Tutorial 26: Set up Azure Policy for compliance
- Tutorial 27: Configure Security Center recommendations
- Tutorial 28: Implement Just-In-Time VM access

**Containers (Continued):**
- Tutorial 29: Deploy multi-container app with Docker Compose
- Tutorial 30: Implement AKS with Istio service mesh
- Tutorial 31: Set up Helm charts for applications
- Tutorial 32: Configure AKS with Azure AD workload identity
- Tutorial 33: Implement GitOps with Flux/ArgoCD
- Tutorial 34: Set up AKS with Azure Application Gateway Ingress
- Tutorial 35: Configure container vulnerability scanning
- Tutorial 36: Implement blue-green deployments on AKS

**Monitoring (Continued):**
- Tutorial 37: Create custom Azure Monitor workbooks
- Tutorial 38: Set up distributed tracing with App Insights
- Tutorial 39: Implement log aggregation and analysis
- Tutorial 40: Configure cost alerts and budgets

---

## 💡 Tutorial Best Practices

### **Before Starting:**
1. Always set a budget alert
2. Use consistent naming conventions
3. Tag all resources
4. Work in a dedicated resource group
5. Have a cleanup plan

### **During Tutorial:**
1. Read error messages carefully
2. Validate after each major step
3. Document any deviations
4. Save commands in a script
5. Take notes on learnings

### **After Completion:**
1. Test thoroughly
2. Document architecture
3. Clean up resources promptly
4. Review costs incurred
5. Share learnings with team

---

## 🎓 Next Steps

After completing these tutorials:

1. **Combine Multiple Tutorials** - Build complex architectures
2. **Automate with IaC** - Convert to Terraform/ARM templates
3. **Add CI/CD** - Implement pipeline automation
4. **Implement Monitoring** - Add comprehensive observability
5. **Optimize Costs** - Review and reduce spending
6. **Security Hardening** - Implement security best practices
7. **Document Everything** - Create runbooks and diagrams

---

**Happy Learning! Build amazing things on Azure! ☁️🚀**

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

