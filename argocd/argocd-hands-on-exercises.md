# ArgoCD Hands-On Exercises

30+ practical exercises to master ArgoCD GitOps deployments from beginner to production expert.

---

## Beginner Level Exercises

### Exercise 1: Install ArgoCD and Access UI
**Objective**: Set up ArgoCD environment

**Tasks:**
1. Install ArgoCD on Kubernetes cluster
2. Access ArgoCD UI
3. Login with CLI
4. Change admin password

<details>
<summary>Solution</summary>

```bash
# 1. Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# 2. Wait for pods to be ready
kubectl wait --for=condition=Ready pods --all -n argocd --timeout=300s

# 3. Get admin password
ARGOCD_PASSWORD=$(kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d)
echo "Admin Password: $ARGOCD_PASSWORD"

# 4. Port forward to access UI
kubectl port-forward svc/argocd-server -n argocd 8080:443 &

# 5. Install CLI (Linux)
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64

# 6. Login with CLI
argocd login localhost:8080 \
  --username admin \
  --password $ARGOCD_PASSWORD \
  --insecure

# 7. Change password
argocd account update-password

# 8. Access UI
# Open browser: https://localhost:8080
# Username: admin
# Password: (new password)
```

**Verification:**
- ArgoCD UI accessible
- CLI login successful
- Password changed
</details>

---

### Exercise 2: Deploy First Application
**Objective**: Deploy a simple application using ArgoCD

**Tasks:**
1. Create application from guestbook example
2. Sync application
3. Verify deployment
4. Access application

<details>
<summary>Solution</summary>

**Create Application:**
```bash
# Method 1: Using CLI
argocd app create guestbook \
  --repo https://github.com/argoproj/argocd-example-apps.git \
  --path guestbook \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default

# Method 2: Using YAML
cat <<EOF | kubectl apply -f -
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
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
EOF
```

**Sync Application:**
```bash
# Sync manually
argocd app sync guestbook

# Wait for sync
argocd app wait guestbook --timeout 300

# Check status
argocd app get guestbook
```

**Verify:**
```bash
# Check resources
kubectl get all -n default -l app=guestbook

# Port forward to access
kubectl port-forward svc/guestbook-ui 8081:80 -n default

# Open browser: http://localhost:8081
```
</details>

---

### Exercise 3: Git-Based Application Deployment
**Objective**: Deploy your own application from Git

**Tasks:**
1. Create Git repository
2. Add Kubernetes manifests
3. Create ArgoCD application
4. Test GitOps workflow

<details>
<summary>Solution</summary>

**1. Create Repository:**
```bash
# Create local repository
mkdir myapp
cd myapp
git init

# Create manifests directory
mkdir -p k8s/manifests

# Create deployment
cat <<EOF > k8s/manifests/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 2
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
          image: nginx:1.21
          ports:
            - containerPort: 80
EOF

# Create service
cat <<EOF > k8s/manifests/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 80
  type: ClusterIP
EOF

# Commit and push
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/user/myapp.git
git push -u origin main
```

**2. Create ArgoCD Application:**
```bash
argocd app create myapp \
  --repo https://github.com/user/myapp.git \
  --path k8s/manifests \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace myapp \
  --sync-policy automated \
  --auto-prune \
  --self-heal

# Sync
argocd app sync myapp
```

**3. Test GitOps:**
```bash
# Update replicas
cd myapp
sed -i 's/replicas: 2/replicas: 3/' k8s/manifests/deployment.yaml
git commit -am "Scale to 3 replicas"
git push

# Watch ArgoCD sync
argocd app get myapp --refresh
argocd app wait myapp

# Verify
kubectl get pods -n myapp
```
</details>

---

## Intermediate Level Exercises

### Exercise 4: Helm Chart Deployment
**Objective**: Deploy application using Helm chart with ArgoCD

<details>
<summary>Solution</summary>

**Create Helm Chart:**
```bash
# Create chart structure
mkdir -p myapp-chart/templates
cd myapp-chart

# Chart.yaml
cat <<EOF > Chart.yaml
apiVersion: v2
name: myapp
description: My Application
type: application
version: 1.0.0
appVersion: "1.0.0"
EOF

# values.yaml
cat <<EOF > values.yaml
replicaCount: 2
image:
  repository: nginx
  tag: "1.21"
  pullPolicy: IfNotPresent
service:
  type: ClusterIP
  port: 80
resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 128Mi
EOF

# templates/deployment.yaml
cat <<'EOF' > templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "myapp.fullname" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ include "myapp.name" . }}
  template:
    metadata:
      labels:
        app: {{ include "myapp.name" . }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: 80
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
EOF

# Commit to Git
git add .
git commit -m "Add Helm chart"
git push
```

**Deploy with ArgoCD:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-helm
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/user/myapp.git
    targetRevision: main
    path: myapp-chart
    helm:
      values: |
        replicaCount: 3
        service:
          type: LoadBalancer
  destination:
    server: https://kubernetes.default.svc
    namespace: myapp
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```
</details>

---

### Exercise 5: Kustomize Overlays
**Objective**: Use Kustomize for multi-environment deployments

<details>
<summary>Solution</summary>

**Create Kustomize Structure:**
```bash
mkdir -p myapp-kustomize/{base,overlays/{dev,staging,production}}
cd myapp-kustomize

# Base deployment
cat <<EOF > base/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 1
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
          image: myapp:latest
          ports:
            - containerPort: 8080
EOF

# Base service
cat <<EOF > base/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080
EOF

# Base kustomization
cat <<EOF > base/kustomization.yaml
resources:
  - deployment.yaml
  - service.yaml
EOF

# Dev overlay
cat <<EOF > overlays/dev/kustomization.yaml
bases:
  - ../../base
namePrefix: dev-
nameSuffix: -v1
commonLabels:
  env: dev
images:
  - name: myapp
    newTag: dev-latest
replicas:
  - name: myapp
    count: 1
EOF

# Production overlay
cat <<EOF > overlays/production/kustomization.yaml
bases:
  - ../../base
namePrefix: prod-
commonLabels:
  env: production
images:
  - name: myapp
    newTag: v1.0.0
replicas:
  - name: myapp
    count: 3
patchesStrategicMerge:
  - resources.yaml
EOF

# Production resources patch
cat <<EOF > overlays/production/resources.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    spec:
      containers:
        - name: myapp
          resources:
            limits:
              cpu: 500m
              memory: 512Mi
            requests:
              cpu: 200m
              memory: 256Mi
EOF

# Commit
git add .
git commit -m "Add Kustomize structure"
git push
```

**Create Applications:**
```yaml
---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-dev
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/user/myapp-kustomize.git
    targetRevision: main
    path: overlays/dev
  destination:
    server: https://kubernetes.default.svc
    namespace: dev
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-prod
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/user/myapp-kustomize.git
    targetRevision: main
    path: overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```
</details>

---

## Advanced Level Exercises

### Exercise 6: App of Apps Pattern
**Objective**: Implement App of Apps for managing multiple applications

<details>
<summary>Solution</summary>

**Create Repository Structure:**
```bash
mkdir -p argocd-apps/{apps,projects}
cd argocd-apps

# Root application
cat <<EOF > apps/root-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: root-app
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
EOF

# Individual apps
cat <<EOF > apps/app1.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: app1
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/company/app1.git
    targetRevision: main
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: app1
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
EOF

cat <<EOF > apps/app2.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: app2
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/company/app2.git
    targetRevision: main
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: app2
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
EOF

# Commit
git add .
git commit -m "Add app of apps"
git push
```

**Deploy Root App:**
```bash
kubectl apply -f apps/root-app.yaml

# Watch all apps sync
argocd app list
```
</details>

---

### Exercise 7: Multi-Cluster Deployment with ApplicationSet
**Objective**: Deploy to multiple clusters using ApplicationSet

<details>
<summary>Solution</summary>

**Add Clusters:**
```bash
# Add dev cluster
kubectl config use-context dev-cluster
argocd cluster add dev-cluster --name dev

# Add prod cluster
kubectl config use-context prod-cluster
argocd cluster add prod-cluster --name production

# Label clusters
argocd cluster set dev --label env=dev
argocd cluster set production --label env=production
```

**Create ApplicationSet:**
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
        syncOptions:
          - CreateNamespace=true
```

**Apply and Verify:**
```bash
kubectl apply -f applicationset.yaml

# Verify applications created
argocd app list | grep myapp-
```
</details>

---

### Exercise 8: Progressive Delivery with Argo Rollouts
**Objective**: Implement canary deployment

<details>
<summary>Solution</summary>

**Install Argo Rollouts:**
```bash
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml
```

**Create Rollout:**
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
  revisionHistoryLimit: 2
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
          image: myapp:v1.0.0
          ports:
            - containerPort: 8080
```

**Deploy with ArgoCD:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-rollout
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/company/myapp.git
    targetRevision: main
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

**Trigger Rollout:**
```bash
# Update image in Git
# ArgoCD will sync and Argo Rollouts will manage canary

# Watch rollout
kubectl argo rollouts get rollout myapp -n production --watch

# Promote manually
kubectl argo rollouts promote myapp -n production
```
</details>

---

**Complete all exercises to master ArgoCD GitOps! 🚀**

For more resources:
- [Learning Roadmap](argocd-learning-roadmap.md)
- [Quick Reference](argocd-quick-reference.md)
- [Troubleshooting Guide](argocd-troubleshooting-guide.md)
- [Interview Questions](argocd-interview-questions.md)


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

