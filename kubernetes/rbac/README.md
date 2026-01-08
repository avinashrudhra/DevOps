# Kubernetes RBAC - Complete Theoretical Guide

**Understanding Role-Based Access Control**

---

## 📚 Table of Contents

1. [What is RBAC?](#what-is-rbac)
2. [RBAC Components](#components)
3. [Roles vs ClusterRoles](#roles-vs-clusterroles)
4. [RoleBindings](#rolebindings)
5. [Service Accounts](#service-accounts)
6. [Common Patterns](#common-patterns)
7. [Best Practices](#best-practices)

---

## 🔐 What is RBAC?

**RBAC (Role-Based Access Control)** regulates access to Kubernetes resources based on roles assigned to users or service accounts.

### **Simple Analogy:**

```
RBAC = Building Access System

Building (Cluster):
  ├── Floor 1 (Namespace: dev)
  │   - Developers: Full access
  │   - Interns: Read-only
  │   - External: No access
  │
  └── Floor 2 (Namespace: prod)
      - Ops team: Full access
      - Developers: Read-only
      - Others: No access

RBAC defines:
  - Who (User/ServiceAccount)
  - Can do what (Verbs: get, create, delete)
  - On which resources (Pods, Services, Secrets)
  - In which scope (Namespace or Cluster-wide)
```

---

### **Why RBAC?**

```
Without RBAC:
  - Everyone has full access
  - Developers can delete production
  - Pods can access any secret
  - No audit trail
  - Security nightmare!

With RBAC:
  ✓ Least privilege principle
  ✓ Developers: Own namespace only
  ✓ Pods: Only needed secrets
  ✓ Audit who did what
  ✓ Secure multi-tenancy
```

---

## 🧩 RBAC Components

Four main components work together.

### **1. Role / ClusterRole:**

```
Defines WHAT can be done

Role: Namespace-scoped
ClusterRole: Cluster-wide

Permissions:
  - Which resources (pods, services, secrets)
  - Which verbs (get, list, create, delete)
```

### **2. RoleBinding / ClusterRoleBinding:**

```
Defines WHO can do it

Binds Role to:
  - Users
  - Groups
  - ServiceAccounts

Links:
  Role + Subject = Permission
```

### **3. ServiceAccount:**

```
Identity for processes running in pods

Like:
  User = Human
  ServiceAccount = Application/Pod
```

---

### **RBAC Flow:**

```
1. Create Role:
   "Can read pods and services"

2. Create ServiceAccount:
   "app-sa"

3. Create RoleBinding:
   "Bind role to app-sa"

4. Pod uses ServiceAccount:
   Pod runs with app-sa identity
   
Result:
   Pod can read pods and services ✓
   Pod cannot create/delete ✗
```

---

## 📋 Roles vs ClusterRoles

### **Role (Namespace-scoped):**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: development  # Specific namespace
rules:
- apiGroups: [""]  # "" = core API group
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

**Scope:**
```
Only in "development" namespace
Can read pods in development
Cannot read pods in production
```

---

### **ClusterRole (Cluster-wide):**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-reader-global
  # No namespace - cluster-wide
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

**Scope:**
```
All namespaces
Can read pods everywhere
Cluster-scoped resources (nodes, PVs, etc.)
```

---

### **Common Verbs:**

```yaml
verbs:
  - get        # Get single resource
  - list       # List multiple resources
  - watch      # Watch for changes
  - create     # Create new resources
  - update     # Update existing (full replacement)
  - patch      # Partial update
  - delete     # Delete single resource
  - deletecollection  # Delete multiple resources
  - "*"        # All verbs (use carefully!)
```

---

### **API Groups:**

```yaml
# Core API group (pods, services, secrets, etc.)
- apiGroups: [""]
  resources: ["pods", "services"]

# Apps API group (deployments, statefulsets, etc.)
- apiGroups: ["apps"]
  resources: ["deployments", "statefulsets"]

# Batch API group (jobs, cronjobs)
- apiGroups: ["batch"]
  resources: ["jobs", "cronjobs"]

# RBAC API group
- apiGroups: ["rbac.authorization.k8s.io"]
  resources: ["roles", "rolebindings"]
```

---

### **Example Roles:**

**1. Read-only access:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer-readonly
  namespace: development
rules:
- apiGroups: ["", "apps", "batch"]
  resources: ["pods", "deployments", "services", "jobs"]
  verbs: ["get", "list", "watch"]  # Only read
```

---

**2. Full access to pods:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-admin
  namespace: development
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["*"]  # All operations
- apiGroups: [""]
  resources: ["pods/log"]  # Also pod logs
  verbs: ["get", "list"]
```

---

**3. Deployment manager:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: deployment-manager
  namespace: development
rules:
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]  # View pods
```

---

## 🔗 RoleBindings

Bind Roles to subjects (users, groups, service accounts).

### **RoleBinding:**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer-readonly-binding
  namespace: development
subjects:
- kind: User
  name: john@example.com
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer-readonly  # References Role
  apiGroup: rbac.authorization.k8s.io
```

**Result:**
```
User: john@example.com
Namespace: development
Permissions: Read-only (from developer-readonly role)
```

---

### **ClusterRoleBinding:**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-binding
subjects:
- kind: User
  name: admin@example.com
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-admin  # Built-in ClusterRole
  apiGroup: rbac.authorization.k8s.io
```

**Result:**
```
User: admin@example.com
Scope: Cluster-wide
Permissions: Full admin access everywhere
```

---

### **Multiple Subjects:**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: team-access
  namespace: development
subjects:
- kind: User
  name: alice@example.com
  apiGroup: rbac.authorization.k8s.io
- kind: User
  name: bob@example.com
  apiGroup: rbac.authorization.k8s.io
- kind: ServiceAccount
  name: ci-sa
  namespace: development
- kind: Group
  name: developers
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer-readonly
  apiGroup: rbac.authorization.k8s.io
```

**Result:**
```
All of these get same permissions:
  - alice@example.com (user)
  - bob@example.com (user)
  - ci-sa (service account)
  - developers group (all members)
```

---

## 👤 Service Accounts

Identity for applications/pods.

### **What are Service Accounts?**

```
Users: For humans
ServiceAccounts: For applications/pods

Pod needs to access Kubernetes API:
  - List other pods
  - Create ConfigMaps
  - Read secrets

Pod authenticates using ServiceAccount
```

---

### **Creating Service Account:**

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  namespace: development
```

```bash
kubectl create serviceaccount app-sa -n development
```

---

### **Using in Pod:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  namespace: development
spec:
  serviceAccountName: app-sa  # Use this ServiceAccount
  containers:
  - name: app
    image: myapp:1.0
```

**What happens:**
```
1. Pod starts with app-sa identity
2. Kubernetes mounts ServiceAccount token:
   /var/run/secrets/kubernetes.io/serviceaccount/token
3. App can use token to call Kubernetes API
4. API Server checks RBAC for app-sa permissions
5. Actions allowed/denied based on RBAC
```

---

### **Complete Example:**

```yaml
---
# 1. ServiceAccount
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  namespace: development

---
# 2. Role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role
  namespace: development
rules:
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "list"]

---
# 3. RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-rolebinding
  namespace: development
subjects:
- kind: ServiceAccount
  name: app-sa
  namespace: development
roleRef:
  kind: Role
  name: app-role
  apiGroup: rbac.authorization.k8s.io

---
# 4. Pod
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  namespace: development
spec:
  serviceAccountName: app-sa
  containers:
  - name: app
    image: myapp:1.0
```

**Result:**
```
Pod myapp can:
  ✓ Get ConfigMaps in development
  ✓ List ConfigMaps in development
  ✗ Create ConfigMaps (not allowed)
  ✗ Access secrets (not allowed)
  ✗ Access production namespace (not allowed)
```

---

## 🎯 Common Patterns

### **1. Namespace Admin:**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: namespace-admin
  namespace: development
subjects:
- kind: User
  name: team-lead@example.com
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: admin  # Built-in ClusterRole
  apiGroup: rbac.authorization.k8s.io
```

**Permissions:**
```
Full access in development namespace
Cannot access other namespaces
Cannot manage cluster resources (nodes, PVs)
```

---

### **2. Read-Only Cluster View:**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: view-all
subjects:
- kind: Group
  name: junior-devs
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: view  # Built-in ClusterRole
  apiGroup: rbac.authorization.k8s.io
```

**Permissions:**
```
Read-only access to all namespaces
Cannot modify anything
Good for auditors, new team members
```

---

### **3. CI/CD Pipeline:**

```yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: ci-deployer
  namespace: production

---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: deployer
  namespace: production
rules:
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "update", "patch"]
- apiGroups: [""]
  resources: ["services"]
  verbs: ["get", "list"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: ci-deployer-binding
  namespace: production
subjects:
- kind: ServiceAccount
  name: ci-deployer
  namespace: production
roleRef:
  kind: Role
  name: deployer
  apiGroup: rbac.authorization.k8s.io
```

**Usage in CI/CD:**
```
Pipeline uses ci-deployer ServiceAccount:
  ✓ Update deployments (rolling updates)
  ✓ View services
  ✗ Delete resources (safer!)
  ✗ Access secrets
```

---

## ✅ Best Practices

### **1. Least Privilege:**

```yaml
# ✗ Bad: Too permissive
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]

# ✓ Good: Only what's needed
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```

---

### **2. Use ServiceAccounts for Pods:**

```yaml
# ✗ Bad: No ServiceAccount (uses default)
spec:
  containers:
  - name: app
    image: app:1.0

# ✓ Good: Explicit ServiceAccount
spec:
  serviceAccountName: app-sa
  containers:
  - name: app
    image: app:1.0
```

---

### **3. Namespace Isolation:**

```yaml
# ✓ Use Roles (namespace-scoped)
kind: Role
metadata:
  namespace: team-a

# Rather than ClusterRoles
# Unless truly needed cluster-wide
```

---

### **4. Test Permissions:**

```bash
# Check what a user can do
kubectl auth can-i list pods --as=john@example.com -n development
# yes

kubectl auth can-i delete pods --as=john@example.com -n development
# no

# Check all permissions
kubectl auth can-i --list --as=john@example.com -n development
```

---

### **5. Audit RBAC:**

```bash
# List all RoleBindings
kubectl get rolebindings --all-namespaces

# Find who has access to a resource
kubectl get rolebindings -n production -o yaml | grep -A5 "name: admin"

# Check ClusterRoleBindings (powerful!)
kubectl get clusterrolebindings
```

---

### **6. Built-in Roles:**

Use built-in ClusterRoles when possible:

```
cluster-admin: Full cluster access (dangerous!)
admin: Full namespace access
edit: Read/write in namespace (no RBAC changes)
view: Read-only in namespace

kubectl get clusterroles
```

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

