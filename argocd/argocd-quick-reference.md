# ArgoCD Quick Reference

Essential commands, configurations, and patterns for ArgoCD GitOps deployments.

---

## 📋 Table of Contents
1. [Installation](#installation)
2. [CLI Commands](#cli-commands)
3. [Application Management](#application-management)
4. [Project Management](#project-management)
5. [Repository Management](#repository-management)
6. [Cluster Management](#cluster-management)
7. [Sync Operations](#sync-operations)
8. [YAML Examples](#yaml-examples)
9. [Security & RBAC](#security--rbac)
10. [Troubleshooting](#troubleshooting)

---

## Installation

### **Quick Install**
```bash
# Create namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Install HA version
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/ha/install.yaml
```

### **Install CLI**
```bash
# macOS
brew install argocd

# Linux
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64

# Windows (PowerShell)
$url = "https://github.com/argoproj/argo-cd/releases/latest/download/argocd-windows-amd64.exe"
$output = "$env:ProgramFiles\argocd\argocd.exe"
Invoke-WebRequest -Uri $url -OutFile $output
```

### **Access ArgoCD**
```bash
# Port forward
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Get admin password
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d

# Expose via LoadBalancer
kubectl patch svc argocd-server -n argocd \
  -p '{"spec": {"type": "LoadBalancer"}}'

# Expose via Ingress
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-server-ingress
  namespace: argocd
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  ingressClassName: nginx
  rules:
    - host: argocd.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: argocd-server
                port:
                  name: https
  tls:
    - hosts:
        - argocd.example.com
      secretName: argocd-server-tls
EOF
```

---

## CLI Commands

### **Login & Context**
```bash
# Login
argocd login argocd.example.com --username admin

# Login with password
argocd login argocd.example.com --username admin --password 'password'

# Login with SSO
argocd login argocd.example.com --sso

# Change password
argocd account update-password

# Logout
argocd logout argocd.example.com

# Get current context
argocd context
```

### **Account Management**
```bash
# List accounts
argocd account list

# Get account info
argocd account get --account admin

# Generate token
argocd account generate-token --account admin

# Can I sync application?
argocd account can-i sync applications '*'
```

---

## Application Management

### **Create Application**
```bash
# From Git repository
argocd app create myapp \
  --repo https://github.com/user/repo.git \
  --path k8s/manifests \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace production

# From Helm chart
argocd app create myapp \
  --repo https://github.com/user/charts.git \
  --path myapp \
  --helm-set replicaCount=3 \
  --helm-set-string image.tag=v1.0.0 \
  --values values-prod.yaml \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace production

# From Kustomize
argocd app create myapp \
  --repo https://github.com/user/kustomize.git \
  --path overlays/production \
  --kustomize-image myapp=myapp:v1.0.0 \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace production

# From YAML file
argocd app create -f application.yaml
```

### **List & Get Applications**
```bash
# List all applications
argocd app list

# List with output format
argocd app list -o wide
argocd app list -o json
argocd app list -o yaml

# Get application details
argocd app get myapp

# Get application manifests
argocd app manifests myapp

# Get application resources
argocd app resources myapp

# Get application logs
argocd app logs myapp

# Get pod logs
argocd app logs myapp --kind Deployment --name myapp
```

### **Update Application**
```bash
# Set parameter
argocd app set myapp --parameter replicaCount=5

# Set Helm values
argocd app set myapp --helm-set replicaCount=5

# Set image
argocd app set myapp --kustomize-image myapp=myapp:v2.0.0

# Set revision
argocd app set myapp --revision v2.0.0

# Enable auto-sync
argocd app set myapp --sync-policy automated

# Disable auto-sync
argocd app set myapp --sync-policy none
```

### **Delete Application**
```bash
# Delete application (keeps resources)
argocd app delete myapp

# Delete application and resources
argocd app delete myapp --cascade

# Delete without confirmation
argocd app delete myapp --yes
```

---

## Sync Operations

### **Sync Application**
```bash
# Sync application
argocd app sync myapp

# Sync with prune
argocd app sync myapp --prune

# Force sync
argocd app sync myapp --force

# Dry run
argocd app sync myapp --dry-run

# Sync specific resources
argocd app sync myapp \
  --resource apps:Deployment:myapp-deployment \
  --resource :Service:myapp-service

# Sync and wait
argocd app sync myapp --timeout 300

# Local sync (without committing)
argocd app sync myapp --local ./manifests/
```

### **Diff & Wait**
```bash
# Show diff
argocd app diff myapp

# Show diff with local changes
argocd app diff myapp --local ./manifests/

# Wait for sync
argocd app wait myapp

# Wait for health
argocd app wait myapp --health

# Wait with timeout
argocd app wait myapp --timeout 300
```

### **Rollback & History**
```bash
# Show history
argocd app history myapp

# Rollback to specific revision
argocd app rollback myapp 5

# Rollback to previous
argocd app rollback myapp
```

### **Refresh & Terminate**
```bash
# Refresh application (check for changes)
argocd app refresh myapp

# Hard refresh (clear cache)
argocd app refresh myapp --hard

# Terminate operation
argocd app terminate-op myapp
```

---

## Project Management

### **Create Project**
```bash
# Create project
argocd proj create myproject \
  --description "My Project" \
  --src https://github.com/user/* \
  --dest https://kubernetes.default.svc,production

# Create from file
argocd proj create -f project.yaml
```

### **Manage Project**
```bash
# List projects
argocd proj list

# Get project details
argocd proj get myproject

# Update project
argocd proj set myproject --description "Updated description"

# Add source repository
argocd proj add-source myproject https://github.com/user/repo.git

# Remove source repository
argocd proj remove-source myproject https://github.com/user/repo.git

# Add destination
argocd proj add-destination myproject \
  https://kubernetes.default.svc production

# Remove destination
argocd proj remove-destination myproject \
  https://kubernetes.default.svc production

# Delete project
argocd proj delete myproject
```

### **Project RBAC**
```bash
# Add role
argocd proj role create myproject developer

# Add policy
argocd proj role add-policy myproject developer \
  --action get --permission allow --object '*'

# List roles
argocd proj role list myproject

# Get role
argocd proj role get myproject developer
```

---

## Repository Management

### **Add Repository**
```bash
# HTTPS with username/password
argocd repo add https://github.com/user/repo.git \
  --username user \
  --password ghp_xxxxxxxxxxxx

# HTTPS with token
argocd repo add https://github.com/user/repo.git \
  --username oauth2 \
  --password ghp_xxxxxxxxxxxx

# SSH
argocd repo add git@github.com:user/repo.git \
  --ssh-private-key-path ~/.ssh/id_rsa

# Helm repository
argocd repo add https://charts.helm.sh/stable \
  --type helm \
  --name stable
```

### **Manage Repositories**
```bash
# List repositories
argocd repo list

# Get repository
argocd repo get https://github.com/user/repo.git

# Remove repository
argocd repo rm https://github.com/user/repo.git
```

---

## Cluster Management

### **Add Cluster**
```bash
# Add cluster from kubeconfig context
argocd cluster add prod-cluster --name production

# Add with labels
argocd cluster add prod-cluster \
  --name production \
  --label env=production \
  --label region=us-east
```

### **Manage Clusters**
```bash
# List clusters
argocd cluster list

# Get cluster
argocd cluster get https://prod-cluster

# Update cluster
argocd cluster set production --label env=prod

# Remove cluster
argocd cluster rm production
```

---

## YAML Examples

### **Basic Application**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/user/repo.git
    targetRevision: HEAD
    path: k8s/manifests
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

### **Helm Application**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-helm
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/user/charts.git
    targetRevision: main
    path: myapp
    helm:
      releaseName: myapp
      values: |
        replicaCount: 3
        image:
          repository: myapp
          tag: v1.0.0
        service:
          type: LoadBalancer
          port: 80
      valueFiles:
        - values-production.yaml
      parameters:
        - name: image.tag
          value: v1.0.1
  destination:
    server: https://kubernetes.default.svc
    namespace: production
```

### **Kustomize Application**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-kustomize
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/user/kustomize.git
    targetRevision: main
    path: overlays/production
    kustomize:
      images:
        - myapp=myapp:v1.0.0
      commonLabels:
        env: production
      namePrefix: prod-
      nameSuffix: -v1
  destination:
    server: https://kubernetes.default.svc
    namespace: production
```

### **AppProject**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: production
  namespace: argocd
spec:
  description: Production applications
  sourceRepos:
    - 'https://github.com/company/*'
    - 'https://charts.helm.sh/stable'
  destinations:
    - namespace: 'prod-*'
      server: https://kubernetes.default.svc
    - namespace: production
      server: https://prod-cluster
  clusterResourceWhitelist:
    - group: '*'
      kind: '*'
  namespaceResourceBlacklist:
    - group: ''
      kind: ResourceQuota
    - group: ''
      kind: LimitRange
  roles:
    - name: developer
      policies:
        - p, proj:production:developer, applications, get, production/*, allow
        - p, proj:production:developer, applications, sync, production/*, allow
      groups:
        - dev-team
```

### **ApplicationSet - Cluster Generator**
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
        path: 'k8s/overlays/{{metadata.labels.env}}'
      destination:
        server: '{{server}}'
        namespace: myapp
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

### **ApplicationSet - Git Generator**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: git-generator
  namespace: argocd
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
      project: default
      source:
        repoURL: https://github.com/company/apps.git
        targetRevision: HEAD
        path: '{{path}}'
      destination:
        server: https://kubernetes.default.svc
        namespace: '{{path.basename}}'
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

---

## Security & RBAC

### **SSO Configuration**
```yaml
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
```

### **RBAC Policy**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-rbac-cm
  namespace: argocd
data:
  policy.default: role:readonly
  policy.csv: |
    p, role:admin, applications, *, */*, allow
    p, role:admin, clusters, *, *, allow
    p, role:developer, applications, get, */*, allow
    p, role:developer, applications, sync, */*, allow
    g, devops-team, role:admin
    g, dev-team, role:developer
```

---

## Troubleshooting

### **Check Status**
```bash
# Check ArgoCD pods
kubectl get pods -n argocd

# Check ArgoCD logs
kubectl logs -n argocd deployment/argocd-server
kubectl logs -n argocd deployment/argocd-repo-server
kubectl logs -n argocd deployment/argocd-application-controller

# Check application events
kubectl get events -n argocd --sort-by='.lastTimestamp'
```

### **Debug Application**
```bash
# Get application details
argocd app get myapp --show-operation

# Get sync status
argocd app get myapp --refresh

# Show diff
argocd app diff myapp

# Get application events
kubectl describe application myapp -n argocd
```

### **Common Issues**
```bash
# Application stuck in Progressing
argocd app terminate-op myapp
argocd app sync myapp --force

# Refresh cache
argocd app refresh myapp --hard

# Check repository access
argocd repo get https://github.com/user/repo.git

# Re-add repository
argocd repo rm https://github.com/user/repo.git
argocd repo add https://github.com/user/repo.git --username user --password token
```

---

## Environment Variables

```bash
# ArgoCD server
export ARGOCD_SERVER=argocd.example.com

# ArgoCD auth token
export ARGOCD_AUTH_TOKEN=ey...

# Skip TLS verification (not recommended for production)
export ARGOCD_OPTS='--insecure'
```

---

## Useful Scripts

### **Sync All Applications**
```bash
for app in $(argocd app list -o name); do
  argocd app sync $app --prune
done
```

### **Get All Application Status**
```bash
argocd app list -o json | jq -r '.[] | "\(.metadata.name): \(.status.sync.status) - \(.status.health.status)"'
```

### **Wait for All Apps to be Healthy**
```bash
for app in $(argocd app list -o name); do
  echo "Waiting for $app..."
  argocd app wait $app --health --timeout 300
done
```

---

**Quick reference complete! Check other guides for detailed examples and troubleshooting.** 🚀


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

