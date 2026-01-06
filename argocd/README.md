# ArgoCD - GitOps Continuous Delivery for Kubernetes

Complete guide to mastering ArgoCD for production Kubernetes deployments using GitOps principles.

---

## 📚 What is ArgoCD?

**ArgoCD** is a declarative, GitOps continuous delivery tool for Kubernetes. It automatically syncs your Kubernetes cluster state with configuration stored in Git repositories, enabling:

- **GitOps Workflow**: Git as single source of truth
- **Automated Deployment**: Continuous synchronization
- **Declarative Setup**: Infrastructure as Code
- **Multi-Cluster Management**: Manage multiple clusters
- **Advanced Deployment Strategies**: Blue-Green, Canary, Progressive
- **RBAC Integration**: Fine-grained access control
- **Audit Trail**: Complete deployment history

---

## 🎯 Why ArgoCD?

### **GitOps Benefits**
- ✅ **Version Control**: All changes tracked in Git
- ✅ **Rollback**: Easy revert to previous state
- ✅ **Audit Trail**: Who changed what, when
- ✅ **Collaboration**: Pull request workflow
- ✅ **Disaster Recovery**: Git as backup

### **ArgoCD Advantages**
- ✅ **Automated Sync**: Continuous reconciliation
- ✅ **Multi-Tenancy**: Multiple teams, projects
- ✅ **SSO Integration**: LDAP, SAML, OAuth2
- ✅ **Webhook Support**: Automated triggers
- ✅ **Web UI**: Visual application management
- ✅ **Health Assessment**: Application status monitoring

---

## 🚀 Quick Start

### **Installation**

```bash
# Create namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Verify installation
kubectl get pods -n argocd

# Get admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Access UI (port-forward)
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Or expose via LoadBalancer
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

# Login with CLI
argocd login localhost:8080
# Username: admin
# Password: (from above command)

# Change admin password
argocd account update-password
```

---

## 📖 Core Concepts

### **1. Application**
ArgoCD's primary resource representing a deployed application:

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
```

### **2. Project**
Logical grouping of applications with RBAC:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: my-project
  namespace: argocd
spec:
  description: Production applications
  sourceRepos:
    - 'https://github.com/organization/*'
  destinations:
    - namespace: production
      server: https://kubernetes.default.svc
  clusterResourceWhitelist:
    - group: '*'
      kind: '*'
```

### **3. Repository**
Git repository containing Kubernetes manifests:

```bash
# Add repository
argocd repo add https://github.com/user/repo.git \
  --username user \
  --password token

# List repositories
argocd repo list
```

---

## 🔄 GitOps Workflow

```
┌──────────────┐
│ Developer    │
│ Commits Code │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  Git Repo    │
│  (Manifests) │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│   ArgoCD     │◄─── Sync Status
│   Monitors   │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  Kubernetes  │
│   Cluster    │
└──────────────┘
```

---

## 📦 Application Sources

### **1. Kubernetes Manifests**
```yaml
source:
  repoURL: https://github.com/user/repo.git
  path: k8s/manifests
  targetRevision: main
```

### **2. Helm Charts**
```yaml
source:
  repoURL: https://github.com/user/repo.git
  path: charts/myapp
  targetRevision: main
  helm:
    values: |
      replicaCount: 3
      image:
        tag: v1.0.0
```

### **3. Kustomize**
```yaml
source:
  repoURL: https://github.com/user/repo.git
  path: overlays/production
  targetRevision: main
  kustomize:
    images:
      - myapp:v1.0.0
```

---

## 🎯 Sync Strategies

### **Manual Sync**
```bash
argocd app sync myapp
```

### **Automated Sync**
```yaml
syncPolicy:
  automated:
    prune: true      # Delete resources not in Git
    selfHeal: true   # Revert manual changes
    allowEmpty: false
  syncOptions:
    - CreateNamespace=true
```

### **Sync Windows**
```yaml
syncPolicy:
  syncOptions:
    - CreateNamespace=true
  retry:
    limit: 5
    backoff:
      duration: 5s
      factor: 2
      maxDuration: 3m
```

---

## 🔐 Security & RBAC

### **SSO Integration**
```yaml
# argocd-cm ConfigMap
data:
  url: https://argocd.example.com
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

### **RBAC Configuration**
```yaml
# argocd-rbac-cm ConfigMap
data:
  policy.csv: |
    p, role:developers, applications, get, */*, allow
    p, role:developers, applications, sync, */*, allow
    p, role:ops, applications, *, */*, allow
    g, dev-team, role:developers
    g, ops-team, role:ops
```

---

## 🎨 Advanced Features

### **ApplicationSet**
Deploy to multiple clusters/environments:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: myapp-set
spec:
  generators:
    - list:
        elements:
          - cluster: dev
            url: https://dev-cluster
          - cluster: prod
            url: https://prod-cluster
  template:
    metadata:
      name: 'myapp-{{cluster}}'
    spec:
      source:
        repoURL: https://github.com/user/repo.git
        path: 'overlays/{{cluster}}'
      destination:
        server: '{{url}}'
        namespace: myapp
```

### **Progressive Delivery**
Integration with Argo Rollouts:

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
        - pause: {duration: 1h}
        - setWeight: 40
        - pause: {duration: 1h}
        - setWeight: 60
        - pause: {duration: 1h}
        - setWeight: 80
        - pause: {duration: 1h}
```

---

## 📚 Learning Resources

### **📖 Documentation**
1. [ArgoCD Learning Roadmap](argocd-learning-roadmap.md) - 12-week structured curriculum
2. [ArgoCD Quick Reference](argocd-quick-reference.md) - Commands and patterns
3. [Hands-On Exercises](argocd-hands-on-exercises.md) - 30+ practical labs
4. [Troubleshooting Guide](argocd-troubleshooting-guide.md) - Common issues
5. [Interview Questions](argocd-interview-questions.md) - 80+ questions

### **🔗 Official Resources**
- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [ArgoCD GitHub](https://github.com/argoproj/argo-cd)
- [ArgoCD Best Practices](https://argo-cd.readthedocs.io/en/stable/user-guide/best_practices/)

---

## 🎯 Common Use Cases

### **1. Multi-Environment Deployment**
- Development, Staging, Production
- Environment-specific configurations
- Progressive rollout

### **2. Multi-Cluster Management**
- Centralized control plane
- Deploy to edge clusters
- Disaster recovery clusters

### **3. Multi-Tenancy**
- Team isolation with Projects
- RBAC per team
- Resource quotas

### **4. Disaster Recovery**
- Git as backup
- Quick cluster recreation
- State restoration

---

## 🚀 Integration with DevOps Pipeline

```
Git ──▶ Jenkins ──▶ Docker ──▶ Git (Update Manifest) ──▶ ArgoCD ──▶ Kubernetes
         Build       Package      Trigger Sync            Deploy
```

**Jenkins Pipeline Example:**
```groovy
stage('Update ArgoCD Manifest') {
    steps {
        sh """
            git clone https://github.com/org/k8s-manifests.git
            cd k8s-manifests
            sed -i 's|image:.*|image: myapp:${BUILD_NUMBER}|' deployment.yaml
            git commit -am "Update image to ${BUILD_NUMBER}"
            git push
        """
    }
}
```

---

## 🎓 Next Steps

1. **Start Learning**: Follow the [Learning Roadmap](argocd-learning-roadmap.md)
2. **Practice**: Complete [Hands-On Exercises](argocd-hands-on-exercises.md)
3. **Reference**: Use [Quick Reference](argocd-quick-reference.md) guide
4. **Troubleshoot**: Check [Troubleshooting Guide](argocd-troubleshooting-guide.md)
5. **Interview Prep**: Study [Interview Questions](argocd-interview-questions.md)

---

**Master GitOps with ArgoCD! 🚀**


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

