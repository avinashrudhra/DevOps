# Helm Interview Questions

80+ comprehensive interview questions for experienced professionals (5-7+ years) focusing on production Helm and Kubernetes package management.

---

## Table of Contents
1. [Fundamentals](#fundamentals)
2. [Chart Structure & Templates](#chart-structure--templates)
3. [Values & Configuration](#values--configuration)
4. [Dependencies](#dependencies)
5. [Hooks & Lifecycle](#hooks--lifecycle)
6. [Release Management](#release-management)
7. [Production Best Practices](#production-best-practices)
8. [Advanced Topics](#advanced-topics)
9. [Integration & CI/CD](#integration--cicd)
10. [Troubleshooting](#troubleshooting)

---

## Fundamentals

### Q1: What is Helm and why would you use it in a Kubernetes environment?

**Answer:**

**Helm** is the package manager for Kubernetes, often described as "the apt/yum for Kubernetes". It simplifies deploying and managing Kubernetes applications through packaged charts.

**Key Benefits:**

**1. Package Management:**
- Bundle multiple Kubernetes resources
- Single command deployment
- Versioned releases

**2. Templating:**
- Parameterized manifests
- Reusable configurations
- Environment-specific deployments

**3. Release Management:**
- Track deployment history
- Easy rollback
- Upgrade management

**4. Dependency Management:**
- Manage sub-charts
- Version dependencies
- Hierarchical values

**Without Helm:**
```bash
kubectl apply -f namespace.yaml
kubectl apply -f configmap.yaml
kubectl apply -f secret.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
# Repeat for each environment with different values...
```

**With Helm:**
```bash
helm install myapp ./myapp-chart --values production-values.yaml
```

**Production Use Cases:**

**1. Multi-Environment Deployments:**
```bash
helm install myapp-dev ./myapp -f values-dev.yaml -n dev
helm install myapp-prod ./myapp -f values-prod.yaml -n production
```

**2. Application Versioning:**
```bash
# Deploy version 1.0.0
helm install myapp ./myapp --version 1.0.0

# Upgrade to 1.1.0
helm upgrade myapp ./myapp --version 1.1.0

# Rollback if issues
helm rollback myapp 1
```

**3. Complex Applications with Dependencies:**
```yaml
# Chart.yaml
dependencies:
  - name: postgresql
    version: 11.x
  - name: redis
    version: 17.x
# One command installs app + dependencies
```

**Helm Architecture:**
```
┌─────────────┐
│  Helm CLI   │
└──────┬──────┘
       │
       ▼
┌─────────────┐     ┌─────────────┐
│ Kubernetes  │────▶│  Releases   │
│  API Server │     │  (Secrets)  │
└─────────────┘     └─────────────┘
```

**When to Use Helm:**
- ✅ Multiple environments (dev, staging, prod)
- ✅ Complex applications with many resources
- ✅ Need for rollback capability
- ✅ Reusable application packages
- ✅ Dependency management
- ✅ Standardized deployments

**When NOT to Use Helm:**
- ❌ Single static manifest
- ❌ No parameterization needed
- ❌ GitOps-only workflow (may prefer Kustomize)

---

### Q2: Explain Helm 3 architecture and how it differs from Helm 2

**Answer:**

**Helm 3 Architecture (Current):**

```
┌──────────────────┐
│   Helm Client    │
│   (helm CLI)     │
└────────┬─────────┘
         │
         │ Direct communication
         ▼
┌────────────────────────────┐
│    Kubernetes API Server   │
└────────┬───────────────────┘
         │
         ▼
┌────────────────────────────┐
│  Release Storage (Secrets) │
│  in Release Namespace      │
└────────────────────────────┘
```

**Helm 2 Architecture (Deprecated):**

```
┌──────────────────┐
│   Helm Client    │
└────────┬─────────┘
         │
         ▼
┌────────────────────────────┐
│  Tiller (Server Component) │
│  Runs in kube-system       │
└────────┬───────────────────┘
         │
         ▼
┌────────────────────────────┐
│    Kubernetes API Server   │
└────────────────────────────┘
```

**Key Differences:**

| Feature | Helm 2 | Helm 3 |
|---------|--------|--------|
| **Server Component** | Tiller (required) | None (removed) |
| **Security** | Tiller has cluster-admin | Uses user's kubeconfig RBAC |
| **Release Storage** | ConfigMaps in kube-system | Secrets in release namespace |
| **CRD Management** | Manual | Automatic with `crds/` directory |
| **Chart Validation** | Basic | JSON Schema validation |
| **Release Naming** | Auto-generated | Must be specified |
| **Namespaces** | `--tiller-namespace` | `--namespace` |

**Why Tiller was Removed:**

**1. Security Issues:**
```
# Helm 2
Tiller runs with cluster-admin privileges
Single point of compromise
Difficult to implement RBAC per user
```

**2. Complexity:**
```
# Helm 2 requires
- Tiller deployment
- Service account setup
- RBAC configuration
- Version compatibility
```

**3. Multi-Tenancy Problems:**
```
# Helm 2
All users share same Tiller
Cannot isolate permissions per team
Tiller namespace issues
```

**Helm 3 Improvements:**

**1. Security:**
```bash
# Uses kubectl credentials
helm install myapp ./chart

# User's RBAC applies directly
# No elevated privileges needed
```

**2. Release Storage:**
```bash
# Helm 2: ConfigMaps in kube-system
kubectl get configmaps -n kube-system | grep TILLER

# Helm 3: Secrets in release namespace
kubectl get secrets -n production -l owner=helm
```

**3. Three-Way Strategic Merge:**
```
Helm 2: Two-way (old manifest vs new manifest)
- Can't detect manual changes

Helm 3: Three-way (old manifest vs new manifest vs live state)
- Detects out-of-band changes
- Safer upgrades
```

**Migration from Helm 2 to 3:**
```bash
# Install Helm 3
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Migrate releases
helm 2to3 migrate myrelease

# Verify
helm list  # Helm 3 command
```

**Production Impact:**

**Positive:**
- ✅ Simpler architecture
- ✅ Better security
- ✅ Namespace-scoped releases
- ✅ No Tiller management overhead

**Breaking Changes:**
- ❌ Must specify release name
- ❌ Chart API v1 deprecated
- ❌ Some template functions changed
- ❌ `helm init` removed

---

### Q3: Explain Helm chart structure in detail

**Answer:**

**Complete Chart Structure:**

```
mychart/
├── Chart.yaml              # Chart metadata
├── values.yaml             # Default configuration values
├── values.schema.json      # JSON Schema for values validation
├── charts/                 # Chart dependencies (sub-charts)
│   ├── postgresql-11.6.26.tgz
│   └── redis-17.11.3.tgz
├── crds/                   # Custom Resource Definitions
│   └── mycrd.yaml
├── templates/              # Kubernetes manifest templates
│   ├── NOTES.txt          # Post-install notes
│   ├── _helpers.tpl       # Template helpers
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── serviceaccount.yaml
│   ├── hpa.yaml
│   ├── pdb.yaml
│   ├── hooks/             # Lifecycle hooks
│   │   ├── pre-install-job.yaml
│   │   └── post-upgrade-job.yaml
│   └── tests/             # Helm tests
│       └── test-connection.yaml
├── .helmignore            # Files to ignore when packaging
├── README.md              # Chart documentation
├── LICENSE                # Chart license
└── requirements.yaml      # Helm 2 dependencies (deprecated)
```

**1. Chart.yaml (Required):**
```yaml
apiVersion: v2                    # Chart API version (v2 for Helm 3)
name: myapp                       # Chart name
description: My Application       # Short description
type: application                 # application or library
version: 1.0.0                    # Chart version (SEMVER)
appVersion: "1.0.0"              # Application version
kubeVersion: ">=1.19.0-0"        # Required Kubernetes version
home: https://myapp.example.com   # Project home page
sources:                          # Source code URLs
  - https://github.com/user/myapp
keywords:                         # Keywords for searching
  - web
  - application
maintainers:                      # Chart maintainers
  - name: DevOps Team
    email: devops@example.com
    url: https://devops.example.com
icon: https://myapp.com/icon.png  # Chart icon URL
deprecated: false                 # Is chart deprecated?
annotations:                      # Additional metadata
  example.com/release-notes: |
    Changes in version 1.0.0
dependencies:                     # Chart dependencies
  - name: postgresql
    version: "11.x.x"
    repository: https://charts.bitnami.com/bitnami
    condition: postgresql.enabled
    tags:
      - database
```

**2. values.yaml (Default Values):**
```yaml
# Image configuration
image:
  registry: docker.io
  repository: myapp
  tag: "1.0.0"
  pullPolicy: IfNotPresent
  pullSecrets: []

# Deployment configuration
replicaCount: 2
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0

# Pod configuration
podAnnotations: {}
podLabels: {}
podSecurityContext:
  fsGroup: 1000
  runAsNonRoot: true
  runAsUser: 1000

# Container configuration
containerPort: 8080
securityContext:
  allowPrivilegeEscalation: false
  capabilities:
    drop:
      - ALL
  readOnlyRootFilesystem: true

# Resources
resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi

# Service
service:
  type: ClusterIP
  port: 80
  targetPort: http
  annotations: {}

# Ingress
ingress:
  enabled: false
  className: "nginx"
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
  hosts:
    - host: myapp.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: myapp-tls
      hosts:
        - myapp.example.com

# Autoscaling
autoscaling:
  enabled: false
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80

# Configuration
config: {}
secrets: {}
env: []
```

**3. templates/_helpers.tpl (Template Helpers):**
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

**4. templates/NOTES.txt (Post-Install Notes):**
```
Thank you for installing {{ .Chart.Name }}!

Your release is named {{ .Release.Name }}.

To learn more about the release, try:

  $ helm status {{ .Release.Name }}
  $ helm get all {{ .Release.Name }}

To access your application:

{{- if .Values.ingress.enabled }}
  http{{ if .Values.ingress.tls }}s{{ end }}://{{ (index .Values.ingress.hosts 0).host }}
{{- else if contains "NodePort" .Values.service.type }}
  export NODE_PORT=$(kubectl get --namespace {{ .Release.Namespace }} -o jsonpath="{.spec.ports[0].nodePort}" services {{ include "myapp.fullname" . }})
  export NODE_IP=$(kubectl get nodes --namespace {{ .Release.Namespace }} -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
{{- else if contains "LoadBalancer" .Values.service.type }}
  export SERVICE_IP=$(kubectl get svc --namespace {{ .Release.Namespace }} {{ include "myapp.fullname" . }} --template "{{"{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}"}}")
  echo http://$SERVICE_IP:{{ .Values.service.port }}
{{- else if contains "ClusterIP" .Values.service.type }}
  export POD_NAME=$(kubectl get pods --namespace {{ .Release.Namespace }} -l "app.kubernetes.io/name={{ include "myapp.name" . }},app.kubernetes.io/instance={{ .Release.Name }}" -o jsonpath="{.items[0].metadata.name}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace {{ .Release.Namespace }} port-forward $POD_NAME 8080:80
{{- end }}
```

**5. .helmignore:**
```
# Patterns to ignore when packaging
.git/
.gitignore
.DS_Store
*.swp
*.bak
*.tmp
*.orig
*~
.project
.idea/
*.tmproj
.vscode/
*.code-workspace
# CI files
.travis.yml
.gitlab-ci.yml
.circleci/
# Chart validation
values.schema.json.bak
# Test files
test/
```

**Best Practices:**

1. **Naming:** Use lowercase, hyphens for multi-word names
2. **Templates:** Keep templates simple, use helpers for complex logic
3. **Values:** Document all values with comments
4. **Testing:** Include test templates
5. **Documentation:** Always include README.md
6. **Versioning:** Follow semantic versioning

---

## Production Best Practices

### Q4: How would you design a production-ready Helm chart?

**Answer:**

**Complete Production-Ready Chart:**

**1. Security First:**

**Pod Security:**
```yaml
# values.yaml
podSecurityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 1000
  seccompProfile:
    type: RuntimeDefault

securityContext:
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  capabilities:
    drop:
      - ALL
```

**RBAC:**
```yaml
# templates/serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: {{ include "myapp.serviceAccountName" . }}
---
# templates/role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: {{ include "myapp.fullname" . }}
rules:
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["get", "list"]
---
# templates/rolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: {{ include "myapp.fullname" . }}
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: {{ include "myapp.fullname" . }}
subjects:
  - kind: ServiceAccount
    name: {{ include "myapp.serviceAccountName" . }}
```

**2. High Availability:**

```yaml
# values.yaml
replicaCount: 3

strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0  # Zero downtime

podDisruptionBudget:
  enabled: true
  minAvailable: 2

affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchLabels:
              app.kubernetes.io/name: myapp
          topologyKey: kubernetes.io/hostname  # Spread across nodes
```

**3. Resource Management:**

```yaml
resources:
  limits:
    cpu: 1000m
    memory: 1Gi
    ephemeral-storage: 1Gi
  requests:
    cpu: 500m
    memory: 512Mi
    ephemeral-storage: 500Mi

# Vertical Pod Autoscaler
vpa:
  enabled: false
  updateMode: Auto

# Horizontal Pod Autoscaler
autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
```

**4. Health Checks:**

```yaml
livenessProbe:
  httpGet:
    path: /health/live
    port: http
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  successThreshold: 1
  failureThreshold: 3

readinessProbe:
  httpGet:
    path: /health/ready
    port: http
  initialDelaySeconds: 5
  periodSeconds: 5
  timeoutSeconds: 3
  successThreshold: 1
  failureThreshold: 3

startupProbe:
  httpGet:
    path: /health/startup
    port: http
  initialDelaySeconds: 0
  periodSeconds: 5
  timeoutSeconds: 3
  successThreshold: 1
  failureThreshold: 30  # 2.5 minutes for slow starts
```

**5. Observability:**

```yaml
# Service Monitor for Prometheus
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: {{ include "myapp.fullname" . }}
spec:
  selector:
    matchLabels:
      {{- include "myapp.selectorLabels" . | nindent 6 }}
  endpoints:
    - port: metrics
      interval: 30s
      path: /metrics
```

**6. Configuration Management:**

```yaml
# Immutable ConfigMaps and Secrets
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "myapp.fullname" . }}-{{ .Release.Revision }}
  annotations:
    # Force pod restart on config change
    checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
immutable: true
```

**7. Network Policies:**

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: {{ include "myapp.fullname" . }}
spec:
  podSelector:
    matchLabels:
      {{- include "myapp.selectorLabels" . | nindent 6 }}
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: ingress-nginx
      ports:
        - protocol: TCP
          port: 8080
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: postgresql
      ports:
        - protocol: TCP
          port: 5432
    - to:  # Allow DNS
        - namespaceSelector: {}
          podSelector:
            matchLabels:
              k8s-app: kube-dns
      ports:
        - protocol: UDP
          port: 53
```

**8. Production Values Structure:**

```yaml
# values-production.yaml
global:
  environment: production
  region: us-east-1

replicaCount: 3

image:
  repository: registry.example.com/myapp
  tag: "1.0.0"  # Never use 'latest' in production
  pullPolicy: Always  # Always pull for production

resources:
  limits:
    cpu: 2000m
    memory: 2Gi
  requests:
    cpu: 1000m
    memory: 1Gi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 20

ingress:
  enabled: true
  className: "nginx"
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
  hosts:
    - host: myapp.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: myapp-tls
      hosts:
        - myapp.example.com

# External dependencies
postgresql:
  enabled: true
  auth:
    existingSecret: postgres-credentials
  primary:
    persistence:
      enabled: true
      size: 100Gi
      storageClass: ssd
    resources:
      limits:
        cpu: 2000m
        memory: 4Gi
      requests:
        cpu: 1000m
        memory: 2Gi

# Monitoring
metrics:
  enabled: true
  serviceMonitor:
    enabled: true
    interval: 30s

# Backup
backup:
  enabled: true
  schedule: "0 2 * * *"  # Daily at 2 AM
  retention: 30  # days
```

**9. CI/CD Integration:**

```bash
#!/bin/bash
# production-deploy.sh

set -e

CHART_PATH="./myapp"
RELEASE_NAME="myapp-prod"
NAMESPACE="production"
VERSION="$1"

# Validate inputs
if [ -z "$VERSION" ]; then
  echo "Usage: $0 <version>"
  exit 1
fi

# Lint chart
helm lint "$CHART_PATH" \
  -f "$CHART_PATH/values-production.yaml" \
  --set image.tag="$VERSION"

# Dry run
helm upgrade --install "$RELEASE_NAME" "$CHART_PATH" \
  -n "$NAMESPACE" \
  -f "$CHART_PATH/values-production.yaml" \
  --set image.tag="$VERSION" \
  --dry-run --debug

# Deploy with safety measures
helm upgrade --install "$RELEASE_NAME" "$CHART_PATH" \
  -n "$NAMESPACE" \
  -f "$CHART_PATH/values-production.yaml" \
  --set image.tag="$VERSION" \
  --atomic \
  --timeout 10m \
  --wait

# Run tests
helm test "$RELEASE_NAME" -n "$NAMESPACE"

# Verify deployment
kubectl rollout status deployment/"$RELEASE_NAME" -n "$NAMESPACE"

echo "✅ Deployment successful!"
```

This production-ready chart includes:
- ✅ Security hardening
- ✅ High availability
- ✅ Resource management
- ✅ Comprehensive health checks
- ✅ Observability
- ✅ Network policies
- ✅ Immutable configuration
- ✅ Automated testing
- ✅ CI/CD integration

---

**🎉 Complete Helm mastery achieved! You're ready for production Helm roles!** 🚀📦

For more resources:
- [Learning Roadmap](helm-learning-roadmap.md)
- [Quick Reference](helm-quick-reference.md)
- [Hands-On Exercises](helm-hands-on-exercises.md)
- [Troubleshooting Guide](helm-troubleshooting-guide.md)


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

