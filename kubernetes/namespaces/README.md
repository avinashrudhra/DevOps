# Kubernetes Namespaces - Complete Theoretical Guide

**Understanding Resource Isolation and Multi-Tenancy**

---

## 📚 Table of Contents

1. [What is a Namespace?](#what-is-namespace)
2. [Default Namespaces](#default-namespaces)
3. [Creating Namespaces](#creating-namespaces)
4. [Resource Quotas](#resource-quotas)
5. [Limit Ranges](#limit-ranges)
6. [Network Policies](#network-policies)
7. [Best Practices](#best-practices)

---

## 📦 What is a Namespace?

A **Namespace** provides a mechanism for isolating groups of resources within a single cluster.

### **Simple Analogy:**

```
Namespace = Departments in a Company

Company (Cluster):
  ├── Engineering Department (Namespace: engineering)
  │   ├── Developers
  │   ├── Resources
  │   └── Budget
  │
  ├── Marketing Department (Namespace: marketing)
  │   ├── Marketers
  │   ├── Resources
  │   └── Budget
  │
  └── Finance Department (Namespace: finance)
      ├── Accountants
      ├── Resources
      └── Budget

Each department:
  - Separate resources
  - Separate budgets
  - Separate teams
  - Some shared services
```

---

### **Why Namespaces?**

```
Single cluster, multiple teams/environments:

Without Namespaces:
  - All resources in one pool
  - Name conflicts (both teams name service "api")
  - Hard to limit resources
  - Difficult to segregate
  - Security issues

With Namespaces:
  ✓ Resource isolation
  ✓ No name conflicts (api in dev, api in prod)
  ✓ Resource quotas per namespace
  ✓ Access control per namespace
  ✓ Logical separation
```

---

## 🏠 Default Namespaces

Kubernetes creates default namespaces.

### **System Namespaces:**

```bash
kubectl get namespaces

NAME              STATUS   AGE
default           Active   30d
kube-system       Active   30d
kube-public       Active   30d
kube-node-lease   Active   30d
```

---

### **1. default:**

```
Default namespace for resources

No namespace specified?
→ Goes to default namespace

kubectl run nginx --image=nginx
Pod created in: default namespace

Use for:
  - Quick testing
  - Small clusters
  - Non-production workloads

Avoid in production:
  Use explicit namespaces
```

---

### **2. kube-system:**

```
Kubernetes system components

Contains:
  - kube-dns / CoreDNS
  - kube-proxy
  - Metrics server
  - CNI plugins
  - Dashboard

⚠️ Don't deploy your apps here!
Reserved for system components
```

```bash
kubectl get pods -n kube-system

NAME                              READY   STATUS
coredns-5d78c9869d-abc12          1/1     Running
coredns-5d78c9869d-def34          1/1     Running
kube-proxy-ghi56                  1/1     Running
kube-proxy-jkl78                  1/1     Running
metrics-server-7b4f6b9d8c-mno90   1/1     Running
```

---

###**3. kube-public:**

```
Publicly readable namespace

Accessible to all users (including unauthenticated)

Used for:
  - Cluster information
  - Public ConfigMaps

Rarely used in practice
```

---

### **4. kube-node-lease:**

```
Node heartbeat information

Each node has a Lease object
Updated regularly to show node health

Used by:
  - Node controller
  - Performance improvement (vs node updates)
```

---

## 🏗️ Creating Namespaces

### **Method 1: kubectl create:**

```bash
# Create namespace
kubectl create namespace development

# Verify
kubectl get namespace development

NAME          STATUS   AGE
development   Active   5s
```

---

### **Method 2: YAML:**

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    environment: production
    team: platform
```

```bash
kubectl apply -f namespace.yaml
```

---

### **Using Namespaces:**

**Deploy to specific namespace:**
```bash
# Create deployment in namespace
kubectl create deployment nginx --image=nginx --namespace=development

# Or in YAML:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: development  # Specify namespace
spec:
  ...
```

**View resources in namespace:**
```bash
# List pods in namespace
kubectl get pods --namespace=development
kubectl get pods -n development  # Short form

# List all resources in namespace
kubectl get all -n development

# List pods in all namespaces
kubectl get pods --all-namespaces
kubectl get pods -A  # Short form
```

---

### **Set Default Namespace:**

```bash
# Set default namespace for current context
kubectl config set-context --current --namespace=development

# Now all commands use development namespace
kubectl get pods  # Shows pods in development
kubectl create deployment api --image=api:1.0  # Creates in development

# View current context
kubectl config view --minify | grep namespace
```

---

## 📊 Resource Quotas

Limit aggregate resource consumption per namespace.

### **What are Resource Quotas?**

```
Resource Quota = Budget per Department

Namespace: development
Budget:
  - Max 10 pods
  - Max 20 CPU cores
  - Max 40Gi memory
  - Max 5 services

Exceeding budget?
  Request denied ✗
```

---

### **Creating Resource Quota:**

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: development
spec:
  hard:
    # Compute resources
    requests.cpu: "10"        # Max 10 CPU cores requested
    requests.memory: 20Gi     # Max 20Gi memory requested
    limits.cpu: "20"          # Max 20 CPU cores limit
    limits.memory: 40Gi       # Max 40Gi memory limit
    
    # Object counts
    pods: "50"                # Max 50 pods
    services: "10"            # Max 10 services
    persistentvolumeclaims: "10"  # Max 10 PVCs
    
    # Storage
    requests.storage: 100Gi   # Max 100Gi storage requested
```

---

### **Resource Quota in Action:**

```
Namespace: development
Quota:
  pods: 50
  requests.cpu: 10
  requests.memory: 20Gi

Current usage:
  pods: 45
  requests.cpu: 8 (80% used)
  requests.memory: 18Gi (90% used)

Try to create deployment with:
  replicas: 10
  requests.cpu: 500m per pod
  requests.memory: 512Mi per pod

Calculation:
  New CPU: 10 * 0.5 = 5 cores
  New memory: 10 * 512Mi = 5Gi
  
  Total CPU: 8 + 5 = 13 cores (exceeds quota of 10!)
  Total memory: 18 + 5 = 23Gi (exceeds quota of 20Gi!)

Result: Request DENIED ✗
Error: exceeded quota
```

---

### **Viewing Quotas:**

```bash
# List quotas in namespace
kubectl get resourcequota -n development

NAME        AGE   REQUEST
dev-quota   5m    pods: 45/50, requests.cpu: 8/10, requests.memory: 18Gi/20Gi

# Detailed view
kubectl describe resourcequota dev-quota -n development

Name:            dev-quota
Namespace:       development
Resource         Used   Hard
--------         ----   ----
pods             45     50
requests.cpu     8      10
requests.memory  18Gi   20Gi
limits.cpu       15     20
limits.memory    35Gi   40Gi
```

---

## 📏 Limit Ranges

Set default, min, max constraints for containers/pods.

### **What are Limit Ranges?**

```
Resource Quota = Total budget for namespace
Limit Range = Per-pod/container constraints

Limit Range enforces:
  - Default requests/limits (if not specified)
  - Minimum requests/limits
  - Maximum requests/limits
  - Request/limit ratios
```

---

### **Creating Limit Range:**

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: dev-limits
  namespace: development
spec:
  limits:
  # Container limits
  - type: Container
    default:  # Default limits
      cpu: 500m
      memory: 512Mi
    defaultRequest:  # Default requests
      cpu: 250m
      memory: 256Mi
    max:  # Maximum
      cpu: 2
      memory: 2Gi
    min:  # Minimum
      cpu: 100m
      memory: 128Mi
  
  # Pod limits
  - type: Pod
    max:
      cpu: 4
      memory: 4Gi
```

---

### **Limit Range in Action:**

**Scenario 1: No resources specified**
```yaml
# Pod spec (no resources)
spec:
  containers:
  - name: app
    image: app:1.0
    # No resources specified

# Kubernetes applies defaults from LimitRange:
spec:
  containers:
  - name: app
    image: app:1.0
    resources:
      requests:
        cpu: 250m        # From defaultRequest
        memory: 256Mi
      limits:
        cpu: 500m        # From default
        memory: 512Mi
```

---

**Scenario 2: Exceeds maximum**
```yaml
# Try to create pod
spec:
  containers:
  - name: app
    image: app:1.0
    resources:
      requests:
        cpu: 3          # Exceeds max (2 CPU)
        memory: 3Gi     # Exceeds max (2Gi)

# Result: Request DENIED ✗
# Error: exceeded LimitRange maximum
```

---

**Scenario 3: Below minimum**
```yaml
# Try to create pod
spec:
  containers:
  - name: app
    image: app:1.0
    resources:
      requests:
        cpu: 50m        # Below min (100m)
        memory: 64Mi    # Below min (128Mi)

# Result: Request DENIED ✗
# Error: below LimitRange minimum
```

---

## 🌐 Network Policies

Control traffic between namespaces.

### **Cross-Namespace Communication:**

```
Default: All namespaces can talk to each other

Namespace: frontend
  Pod: web → Can access backend.backend-ns

Namespace: backend
  Pod: api → Can access database.database-ns

Namespace: database
  Pod: postgres → Can access anything
```

---

### **Restricting Access:**

```yaml
# Allow only frontend namespace to access backend
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-access
  namespace: backend
spec:
  podSelector: {}  # Apply to all pods in backend namespace
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: frontend  # Only from frontend namespace
    ports:
    - protocol: TCP
      port: 8080
```

---

## ✅ Best Practices

### **1. Use Namespaces for Environments:**

```
development namespace:
  - Dev deployments
  - Relaxed quotas
  - Quick iterations

staging namespace:
  - Pre-production testing
  - Production-like config
  - Medium quotas

production namespace:
  - Live services
  - Strict quotas
  - High availability
```

---

### **2. Use Namespaces for Teams:**

```
team-platform namespace:
  - Platform team services
  - Infrastructure
  - Quotas per team

team-data namespace:
  - Data team services
  - ML pipelines
  - Separate quotas

team-api namespace:
  - API team services
  - Microservices
  - Own quotas
```

---

### **3. Always Specify Namespace:**

```yaml
# ✓ Good: Explicit namespace
apiVersion: v1
kind: Service
metadata:
  name: api
  namespace: production  # Clear!
spec:
  ...

# ✗ Bad: No namespace (uses default)
apiVersion: v1
kind: Service
metadata:
  name: api
  # No namespace - confusing!
spec:
  ...
```

---

### **4. Set Resource Quotas:**

```yaml
# Always set quotas to prevent resource exhaustion
apiVersion: v1
kind: ResourceQuota
metadata:
  name: production-quota
  namespace: production
spec:
  hard:
    requests.cpu: "100"
    requests.memory: 200Gi
    pods: "500"
```

---

### **5. Use Labels:**

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    environment: production
    team: platform
    cost-center: engineering
    compliance: pci-dss
```

---

### **6. Namespace Naming:**

```
Good names:
  ✓ production
  ✓ development
  ✓ team-platform
  ✓ app-frontend
  ✓ monitoring

Bad names:
  ✗ prod (too short, unclear)
  ✗ ns1 (meaningless)
  ✗ john-test (user-specific)
  ✗ temp (unclear purpose)
```

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

