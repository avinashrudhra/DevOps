# ArgoCD - Complete Theoretical Guide

**Understanding GitOps and Continuous Delivery for Kubernetes**

---

## 📚 Table of Contents

1. [What is ArgoCD?](#what-is-argocd)
2. [GitOps Principles](#gitops-principles)
3. [ArgoCD Architecture](#architecture)
4. [Applications](#applications)
5. [Sync Strategies](#sync-strategies)
6. [Health Status](#health-status)
7. [Rollbacks](#rollbacks)
8. [Best Practices](#best-practices)

---

## 🚀 What is ArgoCD?

**ArgoCD** is a declarative, GitOps continuous delivery tool for Kubernetes.

### **Simple Analogy:**

```
ArgoCD = Smart Home Automation System

Git Repository = Blueprint/Design
Kubernetes Cluster = Your House

ArgoCD:
  1. Watches blueprint (Git repo)
  2. Compares to actual house (Cluster)
  3. Detects differences
  4. Applies changes automatically
  5. Keeps house matching blueprint

Blueprint changes → House updates automatically
GitOps!
```

---

### **Traditional vs GitOps:**

**❌ Traditional CI/CD:**
```
Developer → Push code
         ↓
   CI builds image
         ↓
   CI pushes to registry
         ↓
   CI/CD runs kubectl apply  ← Problem!
         ↓
   Cluster updated

Issues:
- CI/CD needs cluster access (security risk)
- Push-based (cluster credentials everywhere)
- Hard to track who changed what
- Manual rollbacks
- No drift detection
```

**✅ GitOps with ArgoCD:**
```
Developer → Push code + manifests to Git
                      ↓
               Git is source of truth
                      ↓
            ArgoCD watches Git repo
                      ↓
            Detects changes
                      ↓
            Pulls and applies to cluster
                      ↓
            Cluster matches Git

Benefits:
✓ Pull-based (no cluster credentials in CI)
✓ Git audit trail (who changed what)
✓ Easy rollbacks (git revert)
✓ Drift detection (cluster vs Git)
✓ Declarative (desired state in Git)
```

---

## 📜 GitOps Principles

Four core principles of GitOps.

### **1. Declarative:**

```
Everything defined declaratively in Git

kubernetes manifests:
  deployment.yaml
  service.yaml
  ingress.yaml

Not:
  kubectl run ...
  kubectl scale ...

All configuration as code!
```

---

### **2. Versioned & Immutable:**

```
Git provides:
  - Version history
  - Immutable commits
  - Audit trail

Can answer:
  - What changed?
  - Who changed it?
  - When was it changed?
  - Why? (commit message)

Roll back to any previous version
```

---

### **3. Pulled Automatically:**

```
ArgoCD polls Git repository
Detects changes
Pulls new manifests
Applies to cluster

Not:
  CI pushes to cluster ✗

But:
  ArgoCD pulls from Git ✓
```

---

### **4. Continuously Reconciled:**

```
ArgoCD continuously compares:
  Desired state (Git)
  vs
  Actual state (Cluster)

Difference detected?
  - Alert user (OutOfSync)
  - Auto-sync (if enabled)

Handles:
  - Git changes
  - Manual kubectl changes (drift)
  - Accidental deletions
```

---

## 🏗️ ArgoCD Architecture

### **Components:**

```
┌─────────────────────────────────────────┐
│  Git Repository                          │
│  └── manifests/                         │
│      ├── deployment.yaml                │
│      ├── service.yaml                   │
│      └── ingress.yaml                   │
└─────────────────┬───────────────────────┘
                  │
                  │ (ArgoCD polls)
                  │
┌─────────────────┴───────────────────────┐
│  ArgoCD (Running in Kubernetes)         │
│                                          │
│  ┌────────────────────────────────────┐ │
│  │  API Server                        │ │
│  │  (UI, CLI, API)                    │ │
│  └────────────────────────────────────┘ │
│                                          │
│  ┌────────────────────────────────────┐ │
│  │  Application Controller            │ │
│  │  (Watches apps, syncs)             │ │
│  └────────────────────────────────────┘ │
│                                          │
│  ┌────────────────────────────────────┐ │
│  │  Repository Server                 │ │
│  │  (Git access, manifest generation) │ │
│  └────────────────────────────────────┘ │
└─────────────────┬───────────────────────┘
                  │
                  │ (Applies)
                  │
┌─────────────────┴───────────────────────┐
│  Kubernetes Cluster                      │
│  (Your applications)                     │
└──────────────────────────────────────────┘
```

---

### **Installation:**

```bash
# Create namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for pods
kubectl get pods -n argocd

# Get initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Access UI (port-forward)
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Open browser: https://localhost:8080
# Login: admin / <password-from-above>

# Install ArgoCD CLI
brew install argocd  # macOS
# Or download from https://github.com/argoproj/argo-cd/releases

# Login via CLI
argocd login localhost:8080
```

---

## 📱 Applications

Core resource in ArgoCD.

### **What is an Application?**

```
Application = ArgoCD's representation of your Kubernetes app

Defines:
  - Source: Git repo, path, branch
  - Destination: Cluster, namespace
  - Sync policy: Auto vs manual
```

---

### **Creating Application (UI):**

```
1. Click "+ NEW APP"
2. Fill in:
   - Application Name: myapp
   - Project: default
   - Sync Policy: Automatic (or Manual)

   Source:
   - Repository URL: https://github.com/myuser/myrepo
   - Revision: HEAD (or branch/tag)
   - Path: k8s/ (path to manifests)

   Destination:
   - Cluster: https://kubernetes.default.svc (in-cluster)
   - Namespace: production

3. Click "CREATE"
```

---

### **Creating Application (YAML):**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default
  
  # Source: Where are the manifests?
  source:
    repoURL: https://github.com/myuser/myrepo
    targetRevision: HEAD  # or branch: main, tag: v1.0.0
    path: k8s/production  # Path to Kubernetes manifests
  
  # Destination: Where to deploy?
  destination:
    server: https://kubernetes.default.svc  # In-cluster
    namespace: production
  
  # Sync policy
  syncPolicy:
    automated:
      prune: true      # Delete resources not in Git
      selfHeal: true   # Revert manual changes
      allowEmpty: false
    syncOptions:
    - CreateNamespace=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

```bash
kubectl apply -f application.yaml
```

---

### **Viewing Applications:**

```bash
# CLI
argocd app list

NAME   CLUSTER                         NAMESPACE    PROJECT  STATUS  HEALTH
myapp  https://kubernetes.default.svc  production   default  Synced  Healthy

# Get details
argocd app get myapp

# UI
Open https://localhost:8080
View application card
Click for detailed view
```

---

## 🔄 Sync Strategies

How ArgoCD applies changes.

### **1. Manual Sync:**

```yaml
syncPolicy: {}  # No automated sync
```

**Behavior:**
```
Git updated → ArgoCD detects change
Status: OutOfSync
Waits for manual trigger

User clicks "SYNC" button (or CLI)
ArgoCD applies changes
Status: Synced

Use case:
  - Production (manual approval)
  - Critical apps
  - Staged rollouts
```

---

### **2. Automatic Sync:**

```yaml
syncPolicy:
  automated:
    prune: false
    selfHeal: false
```

**Behavior:**
```
Git updated → ArgoCD detects change
Automatically syncs (applies)
Status: Synced

No manual intervention needed

Use case:
  - Development environments
  - Non-critical apps
  - Rapid iteration
```

---

### **3. Auto-Sync with Prune:**

```yaml
syncPolicy:
  automated:
    prune: true  # Delete resources not in Git
```

**Behavior:**
```
Resource in cluster but not in Git?
  ArgoCD deletes it

Example:
  T+0:  Git has: deployment.yaml, service.yaml
        Cluster has: deployment, service, configmap
  T+1:  ArgoCD sync
        Deletes configmap (not in Git)

Use case:
  - Keep cluster clean
  - Prevent drift
  - Ensure Git is single source of truth
```

**⚠️ Warning:**
```
Be careful with prune!
Can delete resources accidentally

Test in dev first
```

---

### **4. Auto-Sync with Self-Heal:**

```yaml
syncPolicy:
  automated:
    selfHeal: true  # Revert manual changes
```

**Behavior:**
```
Manual kubectl change → ArgoCD detects drift
Automatically reverts to Git state

Example:
  T+0:  Git: replicas: 3
        Cluster: replicas: 3 (Synced)
  T+1:  Admin: kubectl scale deployment myapp --replicas=5
        Cluster: replicas: 5
  T+2:  ArgoCD detects drift (OutOfSync)
  T+3:  ArgoCD reverts to replicas: 3 (selfHeal)

Use case:
  - Prevent manual changes
  - Enforce Git as source of truth
  - Compliance
```

---

## 🏥 Health Status

ArgoCD monitors application health.

### **Health Statuses:**

**Healthy:**
```
All resources healthy
Deployments: Available replicas = desired
Pods: Running and Ready
Services: Endpoints exist

Green checkmark ✓
```

**Progressing:**
```
Resources being updated
Deployment rolling out
Pods starting

Yellow spinner ⟳
```

**Degraded:**
```
Some issues detected
Pods crashing
Deployment not fully available
Services have no endpoints

Yellow warning ⚠️
```

**Suspended:**
```
Resource intentionally stopped
CronJob suspended
HPA suspended

Blue pause ||
```

**Missing:**
```
Resource defined in Git
Not found in cluster

Red X ✗
```

**Unknown:**
```
Cannot determine health
Custom resources without health check

Gray question ?
```

---

### **Health Check Example:**

```
Application: myapp

Resources:
  Deployment/myapp:  Healthy ✓
    Replicas: 3/3 available
  
  Service/myapp:     Healthy ✓
    Endpoints: 3
  
  Pod/myapp-abc12:   Healthy ✓
    Status: Running
    Readiness: 1/1
  
  Pod/myapp-def34:   Progressing ⟳
    Status: ContainerCreating
  
  Pod/myapp-ghi56:   Degraded ⚠️
    Status: CrashLoopBackOff

Overall: Degraded (1 pod unhealthy)
```

---

## ⏮️ Rollbacks

Easy rollbacks with GitOps.

### **Git-Based Rollback:**

```bash
# Scenario: Bad deployment in production

# Check Git history
git log --oneline

abc123 Update image to v2.0 (BAD!)
def456 Update image to v1.9
ghi789 Add health probes

# Rollback in Git
git revert abc123
# Or
git reset --hard def456
git push --force origin main

# ArgoCD detects change
# Automatically syncs (if auto-sync enabled)
# Or manually sync

# Application rolled back to v1.9!
```

---

### **ArgoCD History Rollback:**

```bash
# View sync history
argocd app history myapp

ID  DATE                           REVISION
10  2024-01-08 10:30:00 +0000 UTC  abc123 (HEAD)
9   2024-01-08 10:00:00 +0000 UTC  def456
8   2024-01-08 09:30:00 +0000 UTC  ghi789

# Rollback to previous sync
argocd app rollback myapp 9

# Or UI: Click "History and Rollback" → Select revision → Rollback
```

**What happens:**
```
ArgoCD applies manifests from sync #9
Cluster reverted to that state
Git still has latest commit (abc123)

Note: Git and cluster now out of sync!
Should revert in Git as well for consistency
```

---

## ✅ Best Practices

### **1. One App Per Environment:**

```
Git repository structure:

myapp/
  ├── base/           # Common manifests
  │   ├── deployment.yaml
  │   └── service.yaml
  │
  ├── overlays/
      ├── development/   # Dev-specific
      │   └── kustomization.yaml
      │
      ├── staging/       # Staging-specific
      │   └── kustomization.yaml
      │
      └── production/    # Prod-specific
          └── kustomization.yaml

ArgoCD Applications:
  - myapp-dev (points to overlays/development)
  - myapp-staging (points to overlays/staging)
  - myapp-prod (points to overlays/production)
```

---

### **2. Use Projects for Multi-Tenancy:**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: team-platform
  namespace: argocd
spec:
  description: Platform team applications
  
  # Allowed source repos
  sourceRepos:
  - https://github.com/myorg/platform-*
  
  # Allowed destination clusters/namespaces
  destinations:
  - namespace: 'platform-*'
    server: https://kubernetes.default.svc
  
  # Allowed resources
  clusterResourceWhitelist:
  - group: ''
    kind: Namespace
  namespaceResourceWhitelist:
  - group: 'apps'
    kind: Deployment
  - group: ''
    kind: Service
```

---

### **3. Sync Windows:**

```yaml
# Prevent syncs during business hours
syncPolicy:
  syncOptions:
  - CreateNamespace=true
  automated:
    prune: true
    selfHeal: true
  
syncWindows:
- kind: deny
  schedule: '0 9-17 * * 1-5'  # Mon-Fri 9am-5pm
  duration: 8h
  applications:
  - myapp-prod
  
- kind: allow
  schedule: '0 22 * * *'  # 10pm daily
  duration: 4h
  applications:
  - myapp-prod
```

---

### **4. Notifications:**

```yaml
# argocd-notifications ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
  namespace: argocd
data:
  service.slack: |
    token: $slack-token
  
  template.app-deployed: |
    message: |
      Application {{.app.metadata.name}} deployed!
      Revision: {{.app.status.sync.revision}}
  
  trigger.on-deployed: |
    - when: app.status.operationState.phase in ['Succeeded']
      send: [app-deployed]
  
  subscriptions: |
    - recipients:
      - slack:deployments
      triggers:
      - on-deployed
```

---

### **5. Use Helm/Kustomize:**

```
Don't commit plain YAML with hardcoded values

Use:
  - Helm charts with values.yaml
  - Kustomize with overlays

Easier to manage multiple environments
```

---

### **6. Separate App Definition from Manifests:**

```
Repo 1: Application definitions (ArgoCD apps)
  argocd-apps/
    ├── myapp-dev.yaml
    ├── myapp-staging.yaml
    └── myapp-prod.yaml

Repo 2: Kubernetes manifests
  myapp/
    ├── base/
    └── overlays/

Benefits:
  - Separate permissions
  - Platform team manages ArgoCD apps
  - Dev teams manage manifests
```

---

### **7. Monitor Sync Status:**

```bash
# Check all apps
argocd app list

# Apps out of sync?
argocd app list --sync-status OutOfSync

# Unhealthy apps?
argocd app list --health-status Degraded

# Set up alerts
# Email/Slack notifications for OutOfSync/Degraded
```

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

