# ArgoCD Interview Questions

80+ comprehensive interview questions for experienced professionals (5-7+ years) focusing on production ArgoCD and GitOps deployments.

---

## Table of Contents
1. [Fundamentals & Concepts](#fundamentals--concepts)
2. [Architecture & Components](#architecture--components)
3. [GitOps Principles](#gitops-principles)
4. [Application Management](#application-management)
5. [Sync Strategies](#sync-strategies)
6. [Multi-Cluster & Multi-Tenancy](#multi-cluster--multi-tenancy)
7. [Security & RBAC](#security--rbac)
8. [Helm & Kustomize Integration](#helm--kustomize-integration)
9. [Performance & Scalability](#performance--scalability)
10. [Production Scenarios](#production-scenarios)

---

## Fundamentals & Concepts

### Q1: What is ArgoCD and how does it differ from traditional CI/CD tools?

**Answer:**

ArgoCD is a declarative GitOps continuous delivery tool for Kubernetes that automatically syncs application state from Git repositories to Kubernetes clusters.

**Key Differences from Traditional CI/CD:**

| Aspect | Traditional CI/CD (Jenkins, GitLab CI) | ArgoCD (GitOps) |
|--------|----------------------------------------|-----------------|
| **Model** | Push-based | Pull-based |
| **State Management** | Imperative | Declarative |
| **Source of Truth** | CI/CD scripts | Git repository |
| **Deployment** | External trigger | Continuous reconciliation |
| **Rollback** | Re-run pipeline | Git revert |
| **Drift Detection** | No automatic detection | Continuous monitoring |
| **Audit Trail** | CI/CD logs | Git history |

**Traditional CI/CD (Push):**
```
Developer → Git Commit → CI Build → CI Deploy → Kubernetes
                                         ↓
                                   Direct kubectl apply
```

**ArgoCD (Pull):**
```
Developer → Git Commit → Git Repository
                              ↓
              ArgoCD watches and pulls
                              ↓
                         Kubernetes
```

**ArgoCD Advantages:**
- **Declarative**: Entire application state in Git
- **Self-Healing**: Automatic correction of drift
- **Audit Trail**: Complete Git history
- **Security**: No CI/CD credentials needed in cluster
- **Visibility**: Real-time sync status
- **Multi-Cluster**: Centralized management

**Production Use Case:**
```yaml
# Git contains desired state
apiVersion: apps/v1
kind: Deployment
spec:
  replicas: 3
  
# If someone manually scales to 5
$ kubectl scale deployment myapp --replicas=5

# ArgoCD detects drift and reverts to 3 (if selfHeal enabled)
```

---

### Q2: Explain GitOps principles and how ArgoCD implements them

**Answer:**

**Four GitOps Principles:**

**1. Declarative Configuration:**
- Entire system described declaratively
- Kubernetes manifests, Helm charts, Kustomize

**ArgoCD Implementation:**
```yaml
# Everything in YAML
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
spec:
  source:
    repoURL: https://github.com/org/repo.git
    path: k8s/manifests
  destination:
    server: https://kubernetes.default.svc
    namespace: production
```

**2. Version Controlled:**
- Single source of truth in Git
- Complete audit trail

**ArgoCD Implementation:**
- Monitors Git repositories
- Tracks revisions
- Shows commit history
- Easy rollback via Git

**3. Automated Delivery:**
- Approved changes automatically applied
- Human intervention only for approval

**ArgoCD Implementation:**
```yaml
syncPolicy:
  automated:
    prune: true      # Auto-delete removed resources
    selfHeal: true   # Auto-correct drift
```

**4. Continuous Reconciliation:**
- Software agents ensure desired state
- Detect and correct drift

**ArgoCD Implementation:**
- Reconciliation loop every 3 minutes (configurable)
- Compares Git state with cluster state
- Auto-sync if discrepancies found

**Complete GitOps Flow:**
```
┌─────────────────┐
│   Developer     │
│  Updates Code   │
└────────┬────────┘
         │ git push
         ▼
┌─────────────────┐
│   Git Repo      │ ← Single Source of Truth
│  (Declarative)  │
└────────┬────────┘
         │ ArgoCD watches
         ▼
┌─────────────────┐
│   ArgoCD        │
│  (Reconciler)   │
└────────┬────────┘
         │ kubectl apply
         ▼
┌─────────────────┐
│  Kubernetes     │
│    Cluster      │
└─────────────────┘
         │
         └─► ArgoCD detects drift → Self-heal
```

---

### Q3: Describe ArgoCD architecture and its core components

**Answer:**

**ArgoCD Architecture:**

```
┌──────────────────────────────────────────────────┐
│                ArgoCD Control Plane               │
│                                                    │
│  ┌──────────┐  ┌──────────┐  ┌───────────────┐  │
│  │   CLI    │  │   UI     │  │  API Server   │  │
│  └────┬─────┘  └────┬─────┘  └───────┬───────┘  │
│       │             │                 │           │
│       └─────────────┴─────────────────┘           │
│                     │                             │
│       ┌─────────────▼──────────────┐              │
│       │      API Server             │              │
│       │  - Authentication           │              │
│       │  - RBAC                     │              │
│       │  - Webhook handling         │              │
│       └─────────────┬──────────────┘              │
│                     │                             │
│       ┌─────────────▼──────────────┐              │
│       │   Application Controller   │              │
│       │  - Watches applications    │              │
│       │  - Compares state          │              │
│       │  - Triggers sync           │              │
│       └─────────────┬──────────────┘              │
│                     │                             │
│       ┌─────────────▼──────────────┐              │
│       │      Repo Server           │              │
│       │  - Clones Git repos        │              │
│       │  - Generates manifests     │              │
│       │  - Helm/Kustomize support  │              │
│       └────────────────────────────┘              │
│                                                    │
└────────────────────────┬───────────────────────────┘
                         │
                         ▼
              ┌──────────────────┐
              │   Kubernetes     │
              │     Cluster      │
              └──────────────────┘
```

**Core Components:**

**1. API Server (`argocd-server`):**
- **Purpose**: Web UI, CLI, and API endpoint
- **Functions**:
  - Authentication (local, SSO, LDAP)
  - Authorization (RBAC)
  - Serve Web UI
  - Handle API requests
  - Webhook receiver
- **High Availability**: Can scale horizontally
- **Port**: 8080 (HTTP), 8083 (gRPC)

**2. Application Controller (`argocd-application-controller`):**
- **Purpose**: Core reconciliation engine
- **Functions**:
  - Watches Application CRDs
  - Monitors cluster state
  - Compares desired (Git) vs actual (cluster)
  - Triggers sync operations
  - Health assessment
  - Maintains operation state
- **Reconciliation Loop**: Every 3 minutes (default)
- **High Availability**: StatefulSet with leader election
- **Sharding**: Can be sharded for scale

**3. Repo Server (`argocd-repo-server`):**
- **Purpose**: Generate Kubernetes manifests
- **Functions**:
  - Clone Git repositories
  - Checkout specific revisions
  - Execute Helm template commands
  - Execute Kustomize build
  - Generate plain manifests
  - Cache repositories
- **High Availability**: Can scale horizontally
- **Performance**: In-memory caching

**4. Dex Server (`argocd-dex-server`):**
- **Purpose**: SSO integration
- **Functions**:
  - OIDC/OAuth2 identity provider
  - LDAP connector
  - SAML connector
  - GitHub/GitLab/Google authentication
- **Optional**: Not needed for local users

**5. Redis:**
- **Purpose**: Caching and temporary state
- **Functions**:
  - Cache Git repository data
  - Store temporary sync state
  - Session management
- **High Availability**: Redis Sentinel or Cluster

**6. ApplicationSet Controller (`argocd-applicationset-controller`):**
- **Purpose**: Multi-application management
- **Functions**:
  - Generate multiple Applications
  - Template-based generation
  - Cluster discovery
  - Git directory scanning

**Data Flow Example:**

```
1. User runs: argocd app sync myapp
                    ↓
2. API Server authenticates/authorizes request
                    ↓
3. Updates Application CRD
                    ↓
4. Application Controller detects change
                    ↓
5. Requests manifests from Repo Server
                    ↓
6. Repo Server clones Git, generates manifests
                    ↓
7. Application Controller compares with cluster
                    ↓
8. Applies changes to Kubernetes
                    ↓
9. Monitors rollout status
                    ↓
10. Updates Application CRD with status
```

**Production Considerations:**

**Resource Requirements:**
```yaml
# Typical production setup
API Server:
  CPU: 100m-1000m
  Memory: 128Mi-1Gi
  Replicas: 2-3 (HA)

Application Controller:
  CPU: 500m-2000m
  Memory: 1Gi-4Gi
  Replicas: 1 (with leader election) or sharded

Repo Server:
  CPU: 100m-1000m
  Memory: 256Mi-2Gi
  Replicas: 2-3 (HA)

Redis:
  CPU: 100m-500m
  Memory: 128Mi-512Mi
  Replicas: 3 (HA with Sentinel)
```

---

### Q4: How does ArgoCD handle application synchronization?

**Answer:**

**Sync Process:**

**1. Detection Phase:**
```
Application Controller polls every 3 minutes:
- Fetches desired state from Git
- Fetches actual state from Kubernetes
- Computes diff
```

**2. Sync Decision:**
```yaml
# Automated sync
syncPolicy:
  automated:
    prune: true
    selfHeal: true
    
# If automated → sync immediately
# If manual → wait for user trigger
```

**3. Sync Execution:**
```
Pre-Sync Hooks (if any)
    ↓
Apply resources in waves
    ↓
Wait for health check
    ↓
Post-Sync Hooks (if any)
```

**Sync Phases:**

**Phase 1: PreSync**
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
  annotations:
    argocd.argoproj.io/hook: PreSync
    argocd.argoproj.io/hook-delete-policy: HookSucceeded
```

**Phase 2: Sync (with Waves)**
```yaml
# Wave -1 (applied first)
apiVersion: v1
kind: Namespace
metadata:
  name: myapp
  annotations:
    argocd.argoproj.io/sync-wave: "-1"
---
# Wave 0 (default)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  annotations:
    argocd.argoproj.io/sync-wave: "0"
---
# Wave 1 (applied after Deployment)
apiVersion: batch/v1
kind: Job
metadata:
  name: data-loader
  annotations:
    argocd.argoproj.io/sync-wave: "1"
```

**Phase 3: PostSync**
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: smoke-test
  annotations:
    argocd.argoproj.io/hook: PostSync
```

**Sync Options:**

```yaml
syncPolicy:
  syncOptions:
    # Create namespace if doesn't exist
    - CreateNamespace=true
    
    # Validate resources before applying
    - Validate=true
    
    # Use kubectl apply instead of create/replace
    - ApplyOutOfSyncOnly=true
    
    # Respect ignore differences
    - RespectIgnoreDifferences=true
    
    # Prune resources in specific order
    - PrunePropagationPolicy=foreground
    
    # Prune last to prevent outages
    - PruneLast=true
    
    # Replace resources instead of apply
    - Replace=true
```

**Prune Behavior:**

```yaml
# In Git: deployment.yaml, service.yaml
# In Cluster: deployment, service, configmap (extra)

# With prune=false
# Result: deployment, service, configmap (stays)

# With prune=true
# Result: deployment, service (configmap deleted)
```

**Self-Heal Behavior:**

```yaml
# Git: replicas: 3
# Manual change: kubectl scale deployment myapp --replicas=5

# With selfHeal=false
# Application shows OutOfSync, waits for manual sync

# With selfHeal=true
# ArgoCD automatically reverts to replicas: 3 within 3 minutes
```

**Production Example:**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: production-app
spec:
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    syncOptions:
      - CreateNamespace=true
      - PrunePropagationPolicy=foreground
      - PruneLast=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

**Sync Performance:**
- Parallel resource application
- Configurable timeout per resource
- Wave-based sequential execution
- Health check polling

---

## Production Scenarios

### Q5: Design a complete GitOps workflow for a microservices architecture with dev, staging, and production environments

**Answer:**

**Repository Structure:**

```
company-microservices/
├── apps/
│   ├── service-a/
│   │   ├── base/
│   │   │   ├── deployment.yaml
│   │   │   ├── service.yaml
│   │   │   └── kustomization.yaml
│   │   └── overlays/
│   │       ├── dev/
│   │       │   ├── kustomization.yaml
│   │       │   └── patches/
│   │       ├── staging/
│   │       │   ├── kustomization.yaml
│   │       │   └── patches/
│   │       └── production/
│   │           ├── kustomization.yaml
│   │           └── patches/
│   ├── service-b/
│   └── service-c/
├── argocd/
│   ├── projects/
│   │   ├── dev-project.yaml
│   │   ├── staging-project.yaml
│   │   └── prod-project.yaml
│   └── applications/
│       ├── dev/
│       │   ├── service-a.yaml
│       │   ├── service-b.yaml
│       │   └── service-c.yaml
│       ├── staging/
│       └── production/
└── infrastructure/
    ├── monitoring/
    ├── ingress/
    └── cert-manager/
```

**Complete Workflow:**

```
┌─────────────┐
│  Developer  │
│   Feature   │
│   Branch    │
└──────┬──────┘
       │ PR
       ▼
┌─────────────┐
│   Jenkins   │
│   CI Build  │ ← Build, Test, Scan, Push Image
└──────┬──────┘
       │ Success
       ▼
┌─────────────┐
│ Update Git  │ ← Update image tag in manifests
│  (dev env)  │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  ArgoCD Dev │ ← Auto-sync to dev cluster
│   Cluster   │
└──────┬──────┘
       │ Tests pass
       ▼
┌─────────────┐
│ Promotion   │ ← Manual PR: dev → staging
│  to Staging │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ArgoCD Stage │ ← Auto-sync to staging
│   Cluster   │
└──────┬──────┘
       │ Approval
       ▼
┌─────────────┐
│ Promotion   │ ← Manual PR: staging → production
│  to Prod    │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ArgoCD Prod  │ ← Manual sync (approval gate)
│   Cluster   │
└─────────────┘
```

**Implementation:**

**1. Projects:**
```yaml
# argocd/projects/prod-project.yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: production
  namespace: argocd
spec:
  description: Production applications
  sourceRepos:
    - 'https://github.com/company/microservices.git'
  destinations:
    - namespace: 'prod-*'
      server: https://prod-cluster.example.com
  clusterResourceWhitelist:
    - group: '*'
      kind: '*'
  roles:
    - name: prod-deployer
      policies:
        - p, proj:production:prod-deployer, applications, sync, production/*, allow
        - p, proj:production:prod-deployer, applications, get, production/*, allow
      groups:
        - ops-team
```

**2. ApplicationSet for Multi-Environment:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: microservices
  namespace: argocd
spec:
  generators:
    - matrix:
        generators:
          # Environment generator
          - list:
              elements:
                - env: dev
                  cluster: https://dev-cluster
                  project: development
                  autoSync: "true"
                - env: staging
                  cluster: https://staging-cluster
                  project: staging
                  autoSync: "true"
                - env: production
                  cluster: https://prod-cluster
                  project: production
                  autoSync: "false"  # Manual approval
          # Service generator
          - git:
              repoURL: https://github.com/company/microservices.git
              revision: HEAD
              directories:
                - path: apps/*
  template:
    metadata:
      name: '{{path.basename}}-{{env}}'
    spec:
      project: '{{project}}'
      source:
        repoURL: https://github.com/company/microservices.git
        targetRevision: HEAD
        path: '{{path}}/overlays/{{env}}'
      destination:
        server: '{{cluster}}'
        namespace: '{{path.basename}}-{{env}}'
      syncPolicy:
        automated:
          prune: true
          selfHeal: '{{autoSync}}'
        syncOptions:
          - CreateNamespace=true
```

**3. CI/CD Integration (Jenkins):**
```groovy
pipeline {
    agent any
    stages {
        stage('Build & Push') {
            steps {
                sh 'docker build -t service-a:${BUILD_NUMBER} .'
                sh 'docker push registry.example.com/service-a:${BUILD_NUMBER}'
            }
        }
        
        stage('Update Dev Manifest') {
            steps {
                sh '''
                    git clone https://github.com/company/microservices.git
                    cd microservices/apps/service-a/overlays/dev
                    kustomize edit set image service-a=registry.example.com/service-a:${BUILD_NUMBER}
                    git commit -am "Update service-a to ${BUILD_NUMBER}"
                    git push
                '''
            }
        }
        
        stage('Wait for Dev Deployment') {
            steps {
                sh 'argocd app wait service-a-dev --health --timeout 300'
            }
        }
        
        stage('Run Integration Tests') {
            steps {
                sh './run-integration-tests.sh service-a-dev'
            }
        }
        
        stage('Promote to Staging') {
            when {
                branch 'main'
            }
            input {
                message "Promote to staging?"
            }
            steps {
                sh '''
                    cd microservices/apps/service-a/overlays/staging
                    kustomize edit set image service-a=registry.example.com/service-a:${BUILD_NUMBER}
                    git commit -am "Promote service-a to staging: ${BUILD_NUMBER}"
                    git push
                '''
            }
        }
    }
}
```

**4. Production Promotion:**
```bash
# Create PR from staging to production
gh pr create \
  --base production \
  --head staging \
  --title "Promote service-a to production" \
  --body "Version: ${VERSION}\nTests: Passed\nApproval: Required"

# After PR approval and merge, ArgoCD shows OutOfSync
# Manual sync required in production (approval gate)
argocd app sync service-a-production
```

**5. Monitoring & Notifications:**
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
  
  trigger.on-deployed: |
    - when: app.status.sync.status == 'Synced' and app.status.health.status == 'Healthy'
      send: [app-deployed]
  
  trigger.on-health-degraded: |
    - when: app.status.health.status == 'Degraded'
      send: [app-health-degraded]
  
  template.app-deployed: |
    message: |
      ✅ {{.app.metadata.name}} deployed successfully to {{.app.spec.destination.namespace}}
    slack:
      attachments: |
        [{
          "color": "good",
          "fields": [{
            "title": "Repository",
            "value": "{{.app.spec.source.repoURL}}",
            "short": true
          }, {
            "title": "Revision",
            "value": "{{.app.status.sync.revision}}",
            "short": true
          }]
        }]
```

This architecture provides:
- ✅ **Clear promotion path**: dev → staging → production
- ✅ **Automated dev/staging**: Fast feedback
- ✅ **Manual production**: Safety gate
- ✅ **Complete audit trail**: Git history
- ✅ **Easy rollback**: Git revert
- ✅ **Multi-tenancy**: Project isolation
- ✅ **Notifications**: Slack integration
- ✅ **Testing**: Automated at each stage

---

**🎉 Complete ArgoCD interview preparation! You're ready for production GitOps roles!** 🚀

For more resources:
- [Learning Roadmap](argocd-learning-roadmap.md)
- [Quick Reference](argocd-quick-reference.md)
- [Hands-On Exercises](argocd-hands-on-exercises.md)
- [Troubleshooting Guide](argocd-troubleshooting-guide.md)


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

