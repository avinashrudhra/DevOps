# ArgoCD Learning Roadmap

Complete 12-week curriculum to master ArgoCD and GitOps for production Kubernetes deployments.

---

## 🎯 Learning Objectives

By completing this roadmap, you will:
- ✅ Master GitOps principles and workflows
- ✅ Deploy and manage ArgoCD in production
- ✅ Implement automated CD pipelines
- ✅ Manage multi-cluster deployments
- ✅ Configure RBAC and security
- ✅ Integrate with CI/CD tools
- ✅ Implement progressive delivery
- ✅ Troubleshoot production issues

**Target Audience:** Kubernetes administrators, DevOps engineers (3-7+ years experience)

---

## 📅 12-Week Structured Learning Path

### **Week 1: GitOps Fundamentals & ArgoCD Introduction**

#### **Day 1-2: GitOps Principles**
- What is GitOps?
- GitOps vs Traditional CD
- Benefits and challenges
- GitOps workflow patterns

**Key Concepts:**
```
┌─────────────┐
│  Git Repo   │ ← Single Source of Truth
│ (Declarative│
│   Config)   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  GitOps     │ ← Reconciliation Loop
│  Operator   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Kubernetes  │ ← Desired State Applied
│   Cluster   │
└─────────────┘
```

#### **Day 3-4: ArgoCD Architecture**
- Core components
- Controller architecture
- API server
- Repo server
- Application controller

**Architecture:**
```
┌──────────────┐
│  ArgoCD CLI  │
│   UI / API   │
└──────┬───────┘
       │
┌──────▼───────┐
│  API Server  │ ← Authentication, RBAC
└──────┬───────┘
       │
┌──────▼───────┐     ┌──────────────┐
│ Application  │────▶│  Repo Server │
│  Controller  │     │  (Git Sync)  │
└──────┬───────┘     └──────────────┘
       │
       ▼
┌──────────────┐
│  Kubernetes  │
│    Cluster   │
└──────────────┘
```

#### **Day 5-7: Installation & Setup**
- Installation methods
- High availability setup
- Ingress configuration
- Initial configuration

**Practice:**
```bash
# Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f \
  https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Install CLI
brew install argocd  # macOS
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd

# Access ArgoCD
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

**Weekly Goal:** ArgoCD installed and accessible

---

### **Week 2: Core Concepts & First Application**

#### **Day 8-10: Applications**
- Application CRD
- Source types (Git, Helm, Kustomize)
- Destination configuration
- Sync policies

**First Application:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: guestbook
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/argoproj/argocd-example-apps.git
    targetRevision: HEAD
    path: guestbook
  destination:
    server: https://kubernetes.default.svc
    namespace: guestbook
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

#### **Day 11-12: Projects**
- AppProject CRD
- Multi-tenancy
- Source restrictions
- Destination restrictions
- RBAC integration

**Example Project:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: production
  namespace: argocd
spec:
  description: Production applications
  sourceRepos:
    - 'https://github.com/company/prod-*'
  destinations:
    - namespace: 'prod-*'
      server: https://kubernetes.default.svc
  clusterResourceWhitelist:
    - group: 'apps'
      kind: Deployment
    - group: ''
      kind: Service
  namespaceResourceBlacklist:
    - group: ''
      kind: ResourceQuota
    - group: ''
      kind: LimitRange
```

#### **Day 13-14: Repository Management**
- Adding repositories
- Authentication methods
- SSH keys
- Personal access tokens
- Repository credentials

**Practice:**
```bash
# Add HTTPS repository
argocd repo add https://github.com/user/repo.git \
  --username user \
  --password ghp_xxxxxxxxxxxx

# Add SSH repository
argocd repo add git@github.com:user/repo.git \
  --ssh-private-key-path ~/.ssh/id_rsa

# List repositories
argocd repo list

# Remove repository
argocd repo rm https://github.com/user/repo.git
```

**Weekly Goal:** Deploy first application with ArgoCD

---

### **Week 3: Sync Strategies & Configuration Management**

#### **Day 15-16: Manual Sync**
- CLI sync
- UI sync
- Selective sync
- Sync options

```bash
# Sync application
argocd app sync myapp

# Sync specific resource
argocd app sync myapp --resource apps:Deployment:myapp-deployment

# Sync with preview
argocd app sync myapp --dry-run --prune

# Force sync
argocd app sync myapp --force
```

#### **Day 17-18: Automated Sync**
- Auto-sync policies
- Prune resources
- Self-heal
- Sync windows

**Configuration:**
```yaml
syncPolicy:
  automated:
    prune: true        # Delete resources not in Git
    selfHeal: true     # Revert manual changes
    allowEmpty: false  # Don't sync if no resources
  syncOptions:
    - Validate=true
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

#### **Day 19-20: Helm Integration**
- Helm charts as source
- Values files
- Parameter overrides
- Helm hooks

**Helm Application:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-helm
spec:
  source:
    repoURL: https://github.com/user/charts.git
    path: myapp
    targetRevision: main
    helm:
      releaseName: myapp
      values: |
        replicaCount: 3
        image:
          repository: myapp
          tag: v1.0.0
        service:
          type: LoadBalancer
      parameters:
        - name: image.tag
          value: v1.0.1
      valueFiles:
        - values-production.yaml
```

#### **Day 21: Kustomize Integration**
- Kustomize overlays
- Base and overlays
- Image updates
- ConfigMap/Secret generators

**Kustomize Application:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-kustomize
spec:
  source:
    repoURL: https://github.com/user/kustomize.git
    path: overlays/production
    targetRevision: main
    kustomize:
      images:
        - myapp=myapp:v1.0.0
      commonLabels:
        env: production
      namePrefix: prod-
```

**Weekly Goal:** Master all sync strategies and config management

---

### **Week 4: CLI & Automation**

#### **Day 22-24: ArgoCD CLI**
- Installation
- Authentication
- Application management
- Project management
- Resource operations

**Essential Commands:**
```bash
# Login
argocd login argocd.example.com --username admin

# Application operations
argocd app list
argocd app get myapp
argocd app create myapp -f app.yaml
argocd app delete myapp
argocd app sync myapp
argocd app wait myapp --health

# Resource operations
argocd app resources myapp
argocd app manifests myapp
argocd app diff myapp
argocd app history myapp
argocd app rollback myapp 5

# Project operations
argocd proj list
argocd proj create myproject
argocd proj add-source myproject https://github.com/user/repo.git
argocd proj add-destination myproject https://kubernetes.default.svc production

# Cluster management
argocd cluster add prod-cluster
argocd cluster list
```

#### **Day 25-26: Declarative Setup**
- App of Apps pattern
- Declarative projects
- Declarative applications
- Bootstrap ArgoCD

**App of Apps:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: apps
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/company/argocd-apps.git
    targetRevision: HEAD
    path: apps
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

**Repository Structure:**
```
argocd-apps/
├── apps/
│   ├── app1.yaml
│   ├── app2.yaml
│   ├── app3.yaml
│   └── kustomization.yaml
└── projects/
    ├── dev-project.yaml
    └── prod-project.yaml
```

#### **Day 27-28: CI/CD Integration**
- Jenkins integration
- GitLab CI integration
- GitHub Actions integration
- Image updater

**Jenkins Pipeline:**
```groovy
pipeline {
    agent any
    stages {
        stage('Build & Push') {
            steps {
                sh 'docker build -t myapp:${BUILD_NUMBER} .'
                sh 'docker push myapp:${BUILD_NUMBER}'
            }
        }
        
        stage('Update ArgoCD') {
            steps {
                sh """
                    git clone https://github.com/company/k8s-manifests.git
                    cd k8s-manifests
                    sed -i 's|image:.*|image: myapp:${BUILD_NUMBER}|' deployment.yaml
                    git commit -am "Update to ${BUILD_NUMBER}"
                    git push
                """
            }
        }
        
        stage('Wait for Deployment') {
            steps {
                sh 'argocd app wait myapp --health --timeout 300'
            }
        }
    }
}
```

**Weekly Goal:** Automate ArgoCD operations with CI/CD

---

### **Week 5: Security & RBAC**

#### **Day 29-30: Authentication**
- Local users
- SSO integration
- LDAP/Active Directory
- SAML
- OAuth2/OIDC

**SSO Configuration:**
```yaml
# argocd-cm ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
data:
  url: https://argocd.example.com
  admin.enabled: "true"
  dex.config: |
    connectors:
      - type: github
        id: github
        name: GitHub
        config:
          clientID: $GITHUB_CLIENT_ID
          clientSecret: $GITHUB_CLIENT_SECRET
          orgs:
            - name: my-organization
              teams:
                - devops
                - developers
```

#### **Day 31-32: RBAC**
- Role definitions
- Policy configuration
- User groups
- Project-level RBAC
- Application-level permissions

**RBAC Policy:**
```yaml
# argocd-rbac-cm ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-rbac-cm
  namespace: argocd
data:
  policy.default: role:readonly
  policy.csv: |
    # Admin role
    p, role:admin, applications, *, */*, allow
    p, role:admin, clusters, *, *, allow
    p, role:admin, repositories, *, *, allow
    
    # Developer role
    p, role:developer, applications, get, */*, allow
    p, role:developer, applications, sync, */*, allow
    p, role:developer, applications, override, */*, allow
    p, role:developer, logs, get, */*, allow
    
    # Readonly role
    p, role:readonly, applications, get, */*, allow
    p, role:readonly, logs, get, */*, allow
    
    # Group mappings
    g, devops-team, role:admin
    g, dev-team, role:developer
    g, qa-team, role:readonly
```

#### **Day 33-34: Secrets Management**
- Sealed Secrets
- External Secrets Operator
- HashiCorp Vault
- AWS Secrets Manager
- Azure Key Vault

**Sealed Secrets:**
```bash
# Install Sealed Secrets
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.18.0/controller.yaml

# Create sealed secret
echo -n mypassword | kubectl create secret generic mysecret \
  --dry-run=client --from-file=password=/dev/stdin -o yaml | \
  kubeseal -o yaml > sealed-secret.yaml

# Commit to Git
git add sealed-secret.yaml
git commit -m "Add sealed secret"
git push
```

#### **Day 35: Network Policies**
- ArgoCD network security
- Egress policies
- Ingress policies
- TLS configuration

**Weekly Goal:** Implement complete security and RBAC

---

### **Week 6: Multi-Cluster Management**

#### **Day 36-37: Cluster Registration**
- Adding clusters
- Cluster credentials
- In-cluster vs external
- Cluster labels

```bash
# Add external cluster
kubectl config use-context prod-cluster
argocd cluster add prod-cluster --name production

# List clusters
argocd cluster list

# Update cluster
argocd cluster set production --label env=production

# Remove cluster
argocd cluster rm production
```

#### **Day 38-39: ApplicationSet**
- List generator
- Cluster generator
- Git generator
- Matrix generator
- Merge generator

**Cluster Generator:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: multi-cluster-app
  namespace: argocd
spec:
  generators:
    - clusters:
        selector:
          matchLabels:
            env: production
  template:
    metadata:
      name: 'myapp-{{name}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/company/myapp.git
        targetRevision: HEAD
        path: k8s/overlays/{{metadata.labels.env}}
      destination:
        server: '{{server}}'
        namespace: myapp
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

**Git Generator:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: git-generator
spec:
  generators:
    - git:
        repoURL: https://github.com/company/apps.git
        revision: HEAD
        directories:
          - path: apps/*
  template:
    metadata:
      name: '{{path.basename}}'
    spec:
      source:
        repoURL: https://github.com/company/apps.git
        targetRevision: HEAD
        path: '{{path}}'
      destination:
        server: https://kubernetes.default.svc
        namespace: '{{path.basename}}'
```

#### **Day 40-42: Multi-Environment Deployments**
- Environment strategies
- Promotion workflows
- Git branch strategies
- Kustomize overlays

**Environment Structure:**
```
k8s-manifests/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
└── overlays/
    ├── dev/
    │   ├── kustomization.yaml
    │   └── patches/
    ├── staging/
    │   ├── kustomization.yaml
    │   └── patches/
    └── production/
        ├── kustomization.yaml
        └── patches/
```

**Weekly Goal:** Deploy applications across multiple clusters

---

### **Week 7-8: Progressive Delivery & Advanced Patterns**

#### **Argo Rollouts Integration**
- Canary deployments
- Blue-Green deployments
- Analysis and metrics
- Traffic management

**Canary Rollout:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp
spec:
  replicas: 5
  strategy:
    canary:
      steps:
        - setWeight: 20
        - pause: {duration: 2m}
        - setWeight: 40
        - pause: {duration: 2m}
        - setWeight: 60
        - pause: {duration: 2m}
        - setWeight: 80
        - pause: {duration: 2m}
      trafficRouting:
        istio:
          virtualService:
            name: myapp-vsvc
          destinationRule:
            name: myapp-dest
            canarySubsetName: canary
            stableSubsetName: stable
      analysis:
        templates:
          - templateName: success-rate
        args:
          - name: service-name
            value: myapp
```

**Analysis Template:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: success-rate
spec:
  args:
    - name: service-name
  metrics:
    - name: success-rate
      interval: 1m
      count: 5
      successCondition: result[0] >= 0.95
      provider:
        prometheus:
          address: http://prometheus:9090
          query: |
            sum(rate(
              http_requests_total{service="{{args.service-name}}",status!~"5.."}[1m]
            ))
            /
            sum(rate(
              http_requests_total{service="{{args.service-name}}"}[1m]
            ))
```

**Weekly Goal:** Implement progressive delivery strategies

---

### **Week 9-10: Observability & Monitoring**

#### **Application Health**
- Health assessment
- Custom health checks
- Resource hooks
- Notifications

**Custom Health Check:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
data:
  resource.customizations.health.MyCustomResource: |
    hs = {}
    if obj.status ~= nil then
      if obj.status.phase == "Running" then
        hs.status = "Healthy"
        hs.message = "Resource is running"
      elseif obj.status.phase == "Failed" then
        hs.status = "Degraded"
        hs.message = "Resource failed"
      else
        hs.status = "Progressing"
        hs.message = "Resource is progressing"
      end
    end
    return hs
```

#### **Notifications**
- Slack integration
- Email notifications
- Webhook notifications
- Custom triggers

**Notification Configuration:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
data:
  service.slack: |
    token: $slack-token
  trigger.on-sync-succeeded: |
    - when: app.status.operationState.phase in ['Succeeded']
      send: [app-sync-succeeded]
  template.app-sync-succeeded: |
    message: |
      Application {{.app.metadata.name}} has been successfully synced.
      {{if eq .serviceType "slack"}}:white_check_mark:{{end}}
    slack:
      attachments: |
        [{
          "title": "{{.app.metadata.name}}",
          "title_link": "{{.context.argocdUrl}}/applications/{{.app.metadata.name}}",
          "color": "good",
          "fields": [{
            "title": "Sync Status",
            "value": "{{.app.status.sync.status}}",
            "short": true
          }, {
            "title": "Repository",
            "value": "{{.app.spec.source.repoURL}}",
            "short": true
          }]
        }]
```

**Weekly Goal:** Implement complete observability

---

### **Week 11-12: Production Best Practices & Optimization**

#### **High Availability**
- HA deployment
- Redis clustering
- Database backup
- Disaster recovery

**HA Installation:**
```bash
kubectl apply -n argocd -f \
  https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/ha/install.yaml
```

#### **Performance Optimization**
- Resource limits
- Caching strategies
- Webhook configuration
- Sharding

#### **Backup & Recovery**
```bash
# Backup ArgoCD configuration
kubectl get applications -n argocd -o yaml > apps-backup.yaml
kubectl get appprojects -n argocd -o yaml > projects-backup.yaml
kubectl get secrets -n argocd -o yaml > secrets-backup.yaml

# Restore
kubectl apply -f apps-backup.yaml
kubectl apply -f projects-backup.yaml
kubectl apply -f secrets-backup.yaml
```

**Weekly Goal:** Production-ready ArgoCD deployment

---

## 🎓 Certification & Next Steps

1. Complete [Hands-On Exercises](argocd-hands-on-exercises.md)
2. Review [Troubleshooting Guide](argocd-troubleshooting-guide.md)
3. Practice with [Interview Questions](argocd-interview-questions.md)
4. Study [Quick Reference](argocd-quick-reference.md)
5. Consider Argo Project certification

**You're now ready for production ArgoCD deployments! 🚀**


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

