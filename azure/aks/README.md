# Azure Kubernetes Service (AKS) - Complete Learning Guide

**Managed Kubernetes: Deploy, Scale & Manage Containerized Applications**

---

## 📚 Table of Contents

1. [AKS Overview](#aks-overview)
2. [Create AKS Cluster](#create-aks-cluster)
3. [Node Pools](#node-pools)
4. [Networking](#networking)
5. [ACR Integration](#acr-integration)
6. [Deploy Applications](#deploy-applications)
7. [Scaling](#scaling)
8. [Monitoring](#monitoring)
9. [Security](#security)
10. [Troubleshooting](#troubleshooting)

---

## 🎯 AKS Overview

**Azure Kubernetes Service (AKS)** is a managed Kubernetes container orchestration service.

### **Why AKS?**

**✅ Benefits:**
- Managed control plane (free)
- Automatic updates and patches
- Built-in monitoring
- Integration with Azure services
- Enterprise-grade security
- Azure support

**💰 Cost Model:**
- Control plane: **Free**
- Worker nodes: Pay for VMs
- Storage and networking: Pay-as-you-go

---

## 🚀 Create AKS Cluster

### **Basic Cluster:**

```bash
# Create resource group
az group create \
  --name rg-aks \
  --location eastus

# Create basic AKS cluster
az aks create \
  --resource-group rg-aks \
  --name aks-prod-cluster \
  --kubernetes-version 1.28.3 \
  --node-count 3 \
  --node-vm-size Standard_D2s_v3 \
  --generate-ssh-keys

# Get credentials
az aks get-credentials \
  --resource-group rg-aks \
  --name aks-prod-cluster \
  --overwrite-existing

# Verify connection
kubectl get nodes
kubectl cluster-info
```

---

### **Production-Ready Cluster:**

```bash
# Create production AKS cluster
az aks create \
  --resource-group rg-aks \
  --name aks-prod-cluster \
  --kubernetes-version 1.28.3 \
  --node-count 3 \
  --node-vm-size Standard_D4s_v3 \
  --zones 1 2 3 \
  --network-plugin azure \
  --network-policy calico \
  --enable-managed-identity \
  --enable-aad \
  --enable-azure-rbac \
  --enable-addons monitoring \
  --workspace-resource-id /subscriptions/{sub-id}/resourceGroups/rg-monitoring/providers/Microsoft.OperationalInsights/workspaces/law-prod \
  --attach-acr acrprodregistry001 \
  --enable-cluster-autoscaler \
  --min-count 3 \
  --max-count 10 \
  --max-pods 50 \
  --load-balancer-sku standard \
  --vm-set-type VirtualMachineScaleSets \
  --generate-ssh-keys

# Get credentials
az aks get-credentials \
  --resource-group rg-aks \
  --name aks-prod-cluster \
  --overwrite-existing

# Verify
kubectl get nodes
kubectl get pods --all-namespaces
```

**Parameter Explanation:**
- `--zones 1 2 3` - Deploy across availability zones for HA
- `--network-plugin azure` - Azure CNI (each pod gets VNet IP)
- `--network-policy calico` - Network policies for pod-to-pod traffic control
- `--enable-managed-identity` - Use managed identity (recommended)
- `--enable-aad` - Azure AD integration for authentication
- `--enable-azure-rbac` - Use Azure RBAC for Kubernetes authorization
- `--enable-addons monitoring` - Container Insights for monitoring
- `--attach-acr` - Grant AKS access to ACR
- `--enable-cluster-autoscaler` - Automatically scale nodes

---

### **Terraform AKS Deployment:**

```hcl
resource "azurerm_kubernetes_cluster" "main" {
  name                = "aks-prod-cluster"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  dns_prefix          = "aks-prod"
  kubernetes_version  = "1.28.3"
  
  # System node pool (required)
  default_node_pool {
    name                = "system"
    node_count          = 3
    vm_size             = "Standard_D2s_v3"
    zones               = ["1", "2", "3"]
    enable_auto_scaling = true
    min_count           = 3
    max_count           = 6
    max_pods            = 50
    os_disk_size_gb     = 128
    type                = "VirtualMachineScaleSets"
    
    upgrade_settings {
      max_surge = "33%"
    }
  }
  
  # Identity
  identity {
    type = "SystemAssigned"
  }
  
  # Network
  network_profile {
    network_plugin     = "azure"
    network_policy     = "calico"
    load_balancer_sku  = "standard"
    outbound_type      = "loadBalancer"
    service_cidr       = "10.0.0.0/16"
    dns_service_ip     = "10.0.0.10"
  }
  
  # Azure AD RBAC
  azure_active_directory_role_based_access_control {
    managed                = true
    azure_rbac_enabled     = true
    admin_group_object_ids = [var.admin_group_id]
  }
  
  # Monitoring
  oms_agent {
    log_analytics_workspace_id = azurerm_log_analytics_workspace.main.id
  }
  
  # Auto-scaler
  auto_scaler_profile {
    balance_similar_node_groups = true
    max_graceful_termination_sec = 600
    scale_down_delay_after_add = "10m"
    scale_down_unneeded = "10m"
    scan_interval = "10s"
  }
  
  tags = {
    Environment = "Production"
  }
}

# ACR access
resource "azurerm_role_assignment" "acr_pull" {
  principal_id                     = azurerm_kubernetes_cluster.main.kubelet_identity[0].object_id
  role_definition_name             = "AcrPull"
  scope                            = azurerm_container_registry.main.id
  skip_service_principal_aad_check = true
}
```

---

## 🖥️ Node Pools

**Node pools** are groups of nodes with the same configuration.

### **Types:**

| Type | Purpose | Mode |
|------|---------|------|
| **System** | Kubernetes system pods | System |
| **User** | Application workloads | User |

---

### **Add User Node Pool:**

```bash
# Add user node pool for applications
az aks nodepool add \
  --resource-group rg-aks \
  --cluster-name aks-prod-cluster \
  --name userpool \
  --node-count 3 \
  --node-vm-size Standard_D4s_v3 \
  --zones 1 2 3 \
  --mode User \
  --enable-cluster-autoscaler \
  --min-count 3 \
  --max-count 10 \
  --max-pods 50 \
  --labels environment=production tier=frontend

# Add GPU node pool
az aks nodepool add \
  --resource-group rg-aks \
  --cluster-name aks-prod-cluster \
  --name gpupool \
  --node-count 1 \
  --node-vm-size Standard_NC6s_v3 \
  --mode User \
  --node-taints sku=gpu:NoSchedule \
  --labels gpu=true

# List node pools
az aks nodepool list \
  --resource-group rg-aks \
  --cluster-name aks-prod-cluster \
  --output table

# Scale node pool
az aks nodepool scale \
  --resource-group rg-aks \
  --cluster-name aks-prod-cluster \
  --name userpool \
  --node-count 5

# Delete node pool
az aks nodepool delete \
  --resource-group rg-aks \
  --cluster-name aks-prod-cluster \
  --name gpupool
```

---

## 🌐 Networking

### **Network Plugins:**

| Plugin | IP Assignment | Use Case |
|--------|---------------|----------|
| **Kubenet** | NAT for pods | Simple, smaller clusters |
| **Azure CNI** | VNet IPs for pods | Enterprise, VNet integration |

---

### **Create with Custom VNet:**

```bash
# Create VNet
az network vnet create \
  --resource-group rg-aks \
  --name vnet-aks \
  --address-prefixes 10.1.0.0/16 \
  --subnet-name subnet-aks \
  --subnet-prefixes 10.1.0.0/22

# Get subnet ID
SUBNET_ID=$(az network vnet subnet show \
  --resource-group rg-aks \
  --vnet-name vnet-aks \
  --name subnet-aks \
  --query id \
  --output tsv)

# Create AKS with custom VNet
az aks create \
  --resource-group rg-aks \
  --name aks-prod-cluster \
  --network-plugin azure \
  --vnet-subnet-id $SUBNET_ID \
  --service-cidr 10.0.0.0/16 \
  --dns-service-ip 10.0.0.10 \
  --node-count 3
```

---

## 📦 ACR Integration

```bash
# Attach ACR to existing AKS
az aks update \
  --resource-group rg-aks \
  --name aks-prod-cluster \
  --attach-acr acrprodregistry001

# Verify access
kubectl run test-pod --image=acrprodregistry001.azurecr.io/myapp:v1.0
kubectl get pods
```

---

## 🚀 Deploy Applications

### **Sample Deployment:**

**`deployment.yaml`**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: production
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: acrprodregistry001.azurecr.io/nginx:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: production
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: v1
kind: Namespace
metadata:
  name: production
```

**Deploy:**
```bash
# Create namespace
kubectl create namespace production

# Deploy application
kubectl apply -f deployment.yaml

# Check deployment
kubectl get deployments -n production
kubectl get pods -n production
kubectl get services -n production

# Get external IP
kubectl get service nginx-service -n production
```

---

### **Ingress Controller (NGINX):**

```bash
# Install NGINX Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml

# Wait for external IP
kubectl get service ingress-nginx-controller -n ingress-nginx --watch

# Create ingress
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  namespace: production
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: myapp-tls
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
EOF
```

---

## 📈 Scaling

### **Horizontal Pod Autoscaler (HPA):**

```bash
# Create HPA
kubectl autoscale deployment nginx-deployment \
  -n production \
  --cpu-percent=50 \
  --min=3 \
  --max=10

# View HPA
kubectl get hpa -n production

# Or via YAML
cat <<EOF | kubectl apply -f -
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 70
EOF
```

---

### **Cluster Autoscaler:**

```bash
# Enable cluster autoscaler
az aks update \
  --resource-group rg-aks \
  --name aks-prod-cluster \
  --enable-cluster-autoscaler \
  --min-count 3 \
  --max-count 10

# Update autoscaler settings
az aks update \
  --resource-group rg-aks \
  --name aks-prod-cluster \
  --cluster-autoscaler-profile \
    scale-down-delay-after-add=10m \
    scan-interval=10s
```

---

## 📊 Monitoring

### **Container Insights:**

```bash
# Enable Container Insights
az aks enable-addons \
  --resource-group rg-aks \
  --name aks-prod-cluster \
  --addons monitoring \
  --workspace-resource-id /subscriptions/{sub-id}/resourceGroups/rg-monitoring/providers/Microsoft.OperationalInsights/workspaces/law-prod
```

### **Useful KQL Queries:**

**Pod Performance:**
```kql
Perf
| where ObjectName == "K8SContainer"
| where CounterName == "cpuUsageNanoc ores"
| summarize AvgCPU = avg(CounterValue) by Computer, InstanceName
| order by AvgCPU desc
```

**Container Logs:**
```kql
ContainerLog
| where LogEntry contains "error"
| project TimeGenerated, LogEntry, ContainerID
| order by TimeGenerated desc
```

---

## 🔒 Security

### **Azure AD Integration:**

```bash
# Get AKS cluster ID
AKS_ID=$(az aks show \
  --resource-group rg-aks \
  --name aks-prod-cluster \
  --query id \
  --output tsv)

# Grant admin access to Azure AD group
az role assignment create \
  --assignee <AAD_GROUP_OBJECT_ID> \
  --role "Azure Kubernetes Service Cluster Admin Role" \
  --scope $AKS_ID

# Grant user access
az role assignment create \
  --assignee <AAD_GROUP_OBJECT_ID> \
  --role "Azure Kubernetes Service Cluster User Role" \
  --scope $AKS_ID
```

---

### **Network Policies:**

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-nginx-ingress
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: nginx
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
    ports:
    - protocol: TCP
      port: 80
```

---

## 🛠️ Troubleshooting

### **Common Issues:**

**1. Pods not starting:**
```bash
# Describe pod
kubectl describe pod <POD_NAME> -n production

# View logs
kubectl logs <POD_NAME> -n production

# Previous logs (if crashed)
kubectl logs <POD_NAME> -n production --previous
```

**2. Service not accessible:**
```bash
# Check service
kubectl get svc -n production
kubectl describe svc nginx-service -n production

# Check endpoints
kubectl get endpoints nginx-service -n production
```

**3. Node issues:**
```bash
# Check node status
kubectl get nodes
kubectl describe node <NODE_NAME>

# View node logs
az aks get-credentials --resource-group rg-aks --name aks-prod-cluster
kubectl logs -n kube-system <SYSTEM_POD_NAME>
```

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)
