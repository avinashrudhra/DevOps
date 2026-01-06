# Helm - The Kubernetes Package Manager

Complete guide to mastering Helm for production Kubernetes deployments and application packaging.

---

## 📚 What is Helm?

**Helm** is the package manager for Kubernetes, often called "the Kubernetes apt/yum". It simplifies deployment and management of Kubernetes applications through:

- **Charts**: Packaged Kubernetes applications
- **Templates**: Parameterized manifests
- **Releases**: Deployed instances
- **Repositories**: Chart distribution
- **Hooks**: Lifecycle management
- **Dependencies**: Chart composition

---

## 🎯 Why Helm?

### **Without Helm:**
```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f configmap.yaml
kubectl apply -f ingress.yaml
kubectl apply -f secret.yaml
# Repeat for each environment with different values...
```

### **With Helm:**
```bash
helm install myapp ./myapp-chart --values production-values.yaml
# One command, parameterized, reusable!
```

### **Key Benefits:**
- ✅ **Package Management**: Bundle related resources
- ✅ **Templating**: Reusable configurations
- ✅ **Version Control**: Track releases
- ✅ **Rollback**: Easy revert to previous versions
- ✅ **Dependency Management**: Manage sub-charts
- ✅ **Repository System**: Share and distribute charts
- ✅ **Release Management**: Track deployments
- ✅ **Lifecycle Hooks**: Pre/post install actions

---

## 🚀 Quick Start

### **Installation**

```bash
# macOS
brew install helm

# Linux
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Windows (PowerShell)
choco install kubernetes-helm

# Verify installation
helm version
```

### **First Chart Deployment**

```bash
# Add Helm repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Search for charts
helm search repo nginx

# Install chart
helm install my-nginx bitnami/nginx

# Check release
helm list

# Get release status
helm status my-nginx

# Uninstall
helm uninstall my-nginx
```

---

## 📦 Helm Chart Structure

```
mychart/
├── Chart.yaml              # Chart metadata
├── values.yaml             # Default configuration values
├── charts/                 # Dependencies (sub-charts)
├── templates/              # Kubernetes manifest templates
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── _helpers.tpl       # Template helpers
│   ├── NOTES.txt          # Usage notes
│   └── tests/             # Test files
│       └── test-connection.yaml
├── .helmignore            # Files to ignore
└── README.md              # Chart documentation
```

### **Chart.yaml**
```yaml
apiVersion: v2
name: myapp
description: My Application Helm Chart
type: application
version: 1.0.0        # Chart version
appVersion: "1.0.0"   # Application version
keywords:
  - myapp
  - web
home: https://myapp.example.com
sources:
  - https://github.com/user/myapp
maintainers:
  - name: DevOps Team
    email: devops@example.com
dependencies:
  - name: postgresql
    version: 11.x
    repository: https://charts.bitnami.com/bitnami
```

### **values.yaml**
```yaml
replicaCount: 2

image:
  repository: myapp
  tag: "1.0.0"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: false
  className: "nginx"
  annotations: {}
  hosts:
    - host: myapp.example.com
      paths:
        - path: /
          pathType: Prefix

resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 128Mi

autoscaling:
  enabled: false
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80
```

### **templates/deployment.yaml**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "myapp.fullname" . }}
  labels:
    {{- include "myapp.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "myapp.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "myapp.selectorLabels" . | nindent 8 }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
```

---

## 🔧 Helm Commands

### **Repository Management**
```bash
# Add repository
helm repo add stable https://charts.helm.sh/stable

# Update repositories
helm repo update

# List repositories
helm repo list

# Remove repository
helm repo remove stable

# Search in repository
helm search repo nginx
helm search repo nginx --versions
```

### **Chart Management**
```bash
# Create new chart
helm create mychart

# Lint chart
helm lint mychart

# Package chart
helm package mychart

# Install chart
helm install myrelease ./mychart

# Install with custom values
helm install myrelease ./mychart -f custom-values.yaml

# Install with inline values
helm install myrelease ./mychart --set replicaCount=3

# Dry run (validate without installing)
helm install myrelease ./mychart --dry-run --debug

# Generate manifests
helm template myrelease ./mychart
```

### **Release Management**
```bash
# List releases
helm list
helm list --all-namespaces

# Get release status
helm status myrelease

# Get release values
helm get values myrelease

# Get release manifest
helm get manifest myrelease

# Upgrade release
helm upgrade myrelease ./mychart

# Upgrade with wait
helm upgrade myrelease ./mychart --wait --timeout 5m

# Rollback release
helm rollback myrelease 1

# Uninstall release
helm uninstall myrelease

# Uninstall and keep history
helm uninstall myrelease --keep-history

# Release history
helm history myrelease
```

---

## 🎨 Template Functions

### **Built-in Functions**
```yaml
# String operations
{{ upper "hello" }}                    # HELLO
{{ lower "HELLO" }}                    # hello
{{ quote "value" }}                    # "value"
{{ trim " hello " }}                   # hello
{{ trunc 5 "hello world" }}           # hello

# Conditionals
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
{{- end }}

# Loops
{{- range .Values.environments }}
- name: {{ . }}
{{- end }}

# Default values
image: {{ .Values.image.repository }}:{{ .Values.image.tag | default "latest" }}

# Include templates
{{- include "myapp.labels" . | nindent 4 }}

# toYaml
resources:
  {{- toYaml .Values.resources | nindent 2 }}
```

### **Common Helpers (_helpers.tpl)**
```yaml
{{/*
Expand the name of the chart.
*/}}
{{- define "myapp.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Create a default fully qualified app name.
*/}}
{{- define "myapp.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- if contains $name .Release.Name }}
{{- .Release.Name | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
{{- end }}

{{/*
Common labels
*/}}
{{- define "myapp.labels" -}}
helm.sh/chart: {{ include "myapp.chart" . }}
{{ include "myapp.selectorLabels" . }}
{{- if .Chart.AppVersion }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
{{- end }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}

{{/*
Selector labels
*/}}
{{- define "myapp.selectorLabels" -}}
app.kubernetes.io/name: {{ include "myapp.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}
```

---

## 🔗 Integration with DevOps Tools

### **With Jenkins**
```groovy
stage('Deploy with Helm') {
    steps {
        sh """
            helm upgrade --install myapp ./chart \
              --namespace production \
              --values values-production.yaml \
              --set image.tag=${BUILD_NUMBER} \
              --wait \
              --timeout 5m
        """
    }
}
```

### **With ArgoCD**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  source:
    repoURL: https://github.com/user/charts.git
    path: myapp
    targetRevision: main
    helm:
      values: |
        replicaCount: 3
        image:
          tag: v1.0.0
  destination:
    server: https://kubernetes.default.svc
    namespace: production
```

### **With Docker**
```bash
# Build image
docker build -t myapp:v1.0.0 .

# Update Helm values
helm upgrade myapp ./chart --set image.tag=v1.0.0
```

---

## 📚 Learning Resources

### **📖 Documentation**
1. [Helm Learning Roadmap](helm-learning-roadmap.md) - 12-week structured curriculum
2. [Helm Quick Reference](helm-quick-reference.md) - Commands and templates
3. [Hands-On Exercises](helm-hands-on-exercises.md) - 30+ practical labs
4. [Troubleshooting Guide](helm-troubleshooting-guide.md) - Common issues
5. [Interview Questions](helm-interview-questions.md) - 80+ questions

### **🔗 Official Resources**
- [Helm Documentation](https://helm.sh/docs/)
- [Helm GitHub](https://github.com/helm/helm)
- [Artifact Hub](https://artifacthub.io/) - Public Helm charts
- [Helm Best Practices](https://helm.sh/docs/chart_best_practices/)

---

## 🎯 Common Use Cases

### **1. Application Deployment**
```bash
helm install wordpress bitnami/wordpress \
  --set wordpressUsername=admin \
  --set wordpressPassword=password \
  --set service.type=LoadBalancer
```

### **2. Multi-Environment Deployment**
```bash
# Development
helm install myapp ./chart -f values-dev.yaml

# Staging
helm install myapp ./chart -f values-staging.yaml

# Production
helm install myapp ./chart -f values-production.yaml
```

### **3. Upgrade Strategy**
```bash
# Canary upgrade
helm upgrade myapp ./chart --set replicaCount=1

# Full upgrade after validation
helm upgrade myapp ./chart
```

### **4. Disaster Recovery**
```bash
# Backup release
helm get values myapp > myapp-values-backup.yaml

# Restore
helm install myapp ./chart -f myapp-values-backup.yaml
```

---

## 🎓 Next Steps

1. **Start Learning**: Follow the [Learning Roadmap](helm-learning-roadmap.md)
2. **Practice**: Complete [Hands-On Exercises](helm-hands-on-exercises.md)
3. **Reference**: Use [Quick Reference](helm-quick-reference.md) guide
4. **Troubleshoot**: Check [Troubleshooting Guide](helm-troubleshooting-guide.md)
5. **Interview Prep**: Study [Interview Questions](helm-interview-questions.md)

---

**Master Kubernetes package management with Helm! 📦🚀**


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

