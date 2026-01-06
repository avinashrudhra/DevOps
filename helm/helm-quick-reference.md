# Helm Quick Reference

Essential commands, templates, and patterns for Helm package management.

---

## 📋 Installation

```bash
# macOS
brew install helm

# Linux
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Windows
choco install kubernetes-helm

# Verify
helm version
```

---

## 🗂️ Repository Commands

```bash
# Add repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add stable https://charts.helm.sh/stable

# List repositories
helm repo list

# Update repositories
helm repo update

# Remove repository
helm repo remove bitnami

# Search in repository
helm search repo nginx
helm search repo nginx --versions
helm search repo nginx --version "~1.0.0"

# Search Artifact Hub
helm search hub wordpress
```

---

## 📦 Chart Commands

### **Create & Package**
```bash
# Create new chart
helm create myapp

# Lint chart
helm lint ./myapp

# Package chart
helm package ./myapp

# Package with version
helm package ./myapp --version 1.0.1

# Package and sign
helm package ./myapp --sign --key mykey --keyring ~/.gnupg/secring.gpg
```

### **Dependencies**
```bash
# Update dependencies
helm dependency update ./myapp

# List dependencies
helm dependency list ./myapp

# Build dependencies
helm dependency build ./myapp
```

---

## 🚀 Release Commands

### **Install**
```bash
# Install chart
helm install myrelease ./myapp

# Install from repository
helm install myrelease bitnami/nginx

# Install with custom values
helm install myrelease ./myapp -f custom-values.yaml

# Install with inline overrides
helm install myrelease ./myapp --set replicaCount=3

# Install in specific namespace
helm install myrelease ./myapp -n production --create-namespace

# Dry run (validate without installing)
helm install myrelease ./myapp --dry-run --debug

# Generate name automatically
helm install ./myapp --generate-name

# Wait for resources to be ready
helm install myrelease ./myapp --wait --timeout 5m

# Atomic install (rollback on failure)
helm install myrelease ./myapp --atomic --timeout 5m
```

### **Upgrade**
```bash
# Upgrade release
helm upgrade myrelease ./myapp

# Upgrade with new values
helm upgrade myrelease ./myapp -f new-values.yaml

# Upgrade with inline overrides
helm upgrade myrelease ./myapp --set image.tag=v2.0.0

# Upgrade or install if not exists
helm upgrade --install myrelease ./myapp

# Force upgrade
helm upgrade myrelease ./myapp --force

# Recreate pods
helm upgrade myrelease ./myapp --recreate-pods

# Reset values
helm upgrade myrelease ./myapp --reset-values

# Reuse values from previous release
helm upgrade myrelease ./myapp --reuse-values

# Upgrade with wait
helm upgrade myrelease ./myapp --wait --timeout 5m

# Atomic upgrade (rollback on failure)
helm upgrade myrelease ./myapp --atomic
```

### **Rollback**
```bash
# Rollback to previous version
helm rollback myrelease

# Rollback to specific revision
helm rollback myrelease 3

# Rollback with wait
helm rollback myrelease --wait --timeout 5m

# Force rollback
helm rollback myrelease --force
```

### **Uninstall**
```bash
# Uninstall release
helm uninstall myrelease

# Uninstall with namespace
helm uninstall myrelease -n production

# Keep history
helm uninstall myrelease --keep-history

# Dry run
helm uninstall myrelease --dry-run
```

---

## 📊 Information Commands

```bash
# List releases
helm list
helm list -A  # All namespaces
helm list -n production

# List all (including deleted with history)
helm list --all

# Get release status
helm status myrelease

# Get release values
helm get values myrelease
helm get values myrelease --all  # Include defaults

# Get release manifest
helm get manifest myrelease

# Get release notes
helm get notes myrelease

# Get release hooks
helm get hooks myrelease

# Get all release information
helm get all myrelease

# Release history
helm history myrelease
helm history myrelease --max 10
```

---

## 🎨 Template Commands

```bash
# Render templates locally
helm template myrelease ./myapp

# With custom values
helm template myrelease ./myapp -f custom-values.yaml

# With inline values
helm template myrelease ./myapp --set replicaCount=3

# Show only specific template
helm template myrelease ./myapp -s templates/deployment.yaml

# Include hooks
helm template myrelease ./myapp --is-upgrade

# Debug mode
helm template myrelease ./myapp --debug

# Validate (like install --dry-run but against cluster)
helm install myrelease ./myapp --dry-run --debug
```

---

## 📝 Chart.yaml

```yaml
apiVersion: v2
name: myapp
description: My Application Helm Chart
type: application
version: 1.0.0              # Chart version (SEMVER)
appVersion: "1.0.0"         # Application version
kubeVersion: ">=1.19.0"     # Required Kubernetes version
home: https://myapp.example.com
sources:
  - https://github.com/user/myapp
keywords:
  - myapp
  - web
  - application
maintainers:
  - name: DevOps Team
    email: devops@example.com
    url: https://devops.example.com
icon: https://myapp.example.com/icon.png
deprecated: false

# Dependencies
dependencies:
  - name: postgresql
    version: "11.x.x"
    repository: https://charts.bitnami.com/bitnami
    condition: postgresql.enabled
    tags:
      - database
    import-values:
      - child: auth.password
        parent: dbPassword
  - name: redis
    version: "17.x.x"
    repository: https://charts.bitnami.com/bitnami
    condition: redis.enabled
```

---

## 💾 values.yaml Template

```yaml
# Common configurations
nameOverride: ""
fullnameOverride: ""

# Image
image:
  registry: docker.io
  repository: myapp
  tag: "1.0.0"
  pullPolicy: IfNotPresent
  pullSecrets: []

# Replicas
replicaCount: 2

# Pod configuration
podAnnotations: {}
podLabels: {}

podSecurityContext:
  fsGroup: 1000
  runAsNonRoot: true
  runAsUser: 1000

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

# Probes
livenessProbe:
  httpGet:
    path: /health
    port: http
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3

readinessProbe:
  httpGet:
    path: /ready
    port: http
  initialDelaySeconds: 5
  periodSeconds: 5
  timeoutSeconds: 3
  successThreshold: 1
  failureThreshold: 3

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
  targetMemoryUtilizationPercentage: 80

# Node selection
nodeSelector: {}
tolerations: []
affinity: {}

# Volumes
persistence:
  enabled: false
  storageClass: ""
  accessMode: ReadWriteOnce
  size: 10Gi
  existingClaim: ""

# Configuration
config: {}
secrets: {}
env: []
envFrom: []

# Additional resources
extraVolumes: []
extraVolumeMounts: []
initContainers: []
sidecars: []

# ServiceAccount
serviceAccount:
  create: true
  annotations: {}
  name: ""

# RBAC
rbac:
  create: true
  rules: []

# Pod Disruption Budget
podDisruptionBudget:
  enabled: false
  minAvailable: 1

# Network Policy
networkPolicy:
  enabled: false
  policyTypes:
    - Ingress
    - Egress
```

---

## 🎯 Template Functions

### **String Functions**
```yaml
{{ upper "hello" }}                    # HELLO
{{ lower "HELLO" }}                    # hello
{{ title "hello world" }}              # Hello World
{{ quote "value" }}                    # "value"
{{ trim " hello " }}                   # hello
{{ trunc 5 "hello world" }}           # hello
{{ trimSuffix "-" "hello-" }}          # hello
{{ trimPrefix "-" "-hello" }}          # hello
{{ replace "-" "_" "hello-world" }}    # hello_world
{{ repeat 3 "hello" }}                 # hellohellohello
{{ substr 0 5 "hello world" }}         # hello
{{ contains "hello" "hello world" }}   # true
{{ hasPrefix "hello" "hello world" }}  # true
{{ hasSuffix "world" "hello world" }}  # true
```

### **Math & Logic**
```yaml
{{ add 1 2 }}                          # 3
{{ sub 5 2 }}                          # 3
{{ mul 3 2 }}                          # 6
{{ div 10 2 }}                         # 5
{{ mod 10 3 }}                         # 1
{{ max 1 2 3 }}                        # 3
{{ min 1 2 3 }}                        # 1
{{ eq .Values.env "prod" }}            # true/false
{{ ne .Values.env "dev" }}             # true/false
{{ lt 1 2 }}                           # true
{{ gt 2 1 }}                           # true
{{ and true false }}                   # false
{{ or true false }}                    # true
{{ not false }}                        # true
```

### **Default & Required**
```yaml
{{ .Values.port | default 8080 }}
{{ .Values.name | default "myapp" }}
{{ required "image.tag is required!" .Values.image.tag }}
```

### **Conditionals**
```yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
{{- end }}

{{- if eq .Values.env "production" }}
replicas: 3
{{- else if eq .Values.env "staging" }}
replicas: 2
{{- else }}
replicas: 1
{{- end }}
```

### **Loops**
```yaml
# Simple range
{{- range .Values.environments }}
- {{ . }}
{{- end }}

# Range with index
{{- range $index, $value := .Values.items }}
- name: item-{{ $index }}
  value: {{ $value }}
{{- end }}

# Range with key-value
{{- range $key, $value := .Values.config }}
{{ $key }}: {{ $value | quote }}
{{- end }}
```

### **Template Includes**
```yaml
# Include template
{{- include "myapp.labels" . }}

# Include with nindent
labels:
  {{- include "myapp.labels" . | nindent 2 }}

# Include with indent
metadata:
  labels:
{{ include "myapp.labels" . | indent 4 }}
```

### **toYaml**
```yaml
resources:
  {{- toYaml .Values.resources | nindent 2 }}

env:
  {{- toYaml .Values.env | nindent 2 }}
```

---

## 🔧 Common Helpers (_helpers.tpl)

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
Create chart name and version as used by the chart label.
*/}}
{{- define "myapp.chart" -}}
{{- printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" | trunc 63 | trimSuffix "-" }}
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

{{/*
Create the name of the service account to use
*/}}
{{- define "myapp.serviceAccountName" -}}
{{- if .Values.serviceAccount.create }}
{{- default (include "myapp.fullname" .) .Values.serviceAccount.name }}
{{- else }}
{{- default "default" .Values.serviceAccount.name }}
{{- end }}
{{- end }}
```

---

## 🪝 Helm Hooks

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ include "myapp.fullname" . }}-migration
  annotations:
    # Hook types
    "helm.sh/hook": pre-install,pre-upgrade
    
    # Hook weight (order of execution, lower first)
    "helm.sh/hook-weight": "1"
    
    # Delete policy
    "helm.sh/hook-delete-policy": before-hook-creation,hook-succeeded
```

**Hook Types:**
- `pre-install`, `post-install`
- `pre-delete`, `post-delete`
- `pre-upgrade`, `post-upgrade`
- `pre-rollback`, `post-rollback`
- `test`

**Delete Policies:**
- `before-hook-creation`
- `hook-succeeded`
- `hook-failed`

---

## 🧪 Tests

```yaml
# templates/tests/test-connection.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "myapp.fullname" . }}-test-connection"
  annotations:
    "helm.sh/hook": test
spec:
  containers:
    - name: wget
      image: busybox
      command: ['wget']
      args: ['{{ include "myapp.fullname" . }}:{{ .Values.service.port }}']
  restartPolicy: Never
```

**Run Tests:**
```bash
helm test myrelease
helm test myrelease --logs
```

---

## 🔒 Secrets

```bash
# Install with secret values
helm install myrelease ./myapp -f secrets.yaml

# Use --set for secrets (not recommended)
helm install myrelease ./myapp --set password=mysecret

# Use sealed secrets
kubeseal <secret.yaml >sealed-secret.yaml
```

---

## 🌐 Environment Variables

```bash
# Set Helm namespace
export HELM_NAMESPACE=production

# Set kubeconfig
export KUBECONFIG=~/.kube/config

# Debug mode
export HELM_DEBUG=true

# Plugin directory
export HELM_PLUGINS=~/.local/share/helm/plugins
```

---

## 📚 Useful Commands

```bash
# Show computed values
helm get values myrelease --all

# Diff between releases (with helm-diff plugin)
helm diff upgrade myrelease ./myapp -f new-values.yaml

# Push to OCI registry
helm push myapp-1.0.0.tgz oci://registry.example.com/charts

# Pull from OCI registry
helm pull oci://registry.example.com/charts/myapp --version 1.0.0

# Show chart metadata
helm show chart bitnami/nginx

# Show chart values
helm show values bitnami/nginx

# Show chart README
helm show readme bitnami/nginx

# Show all chart information
helm show all bitnami/nginx
```

---

**Quick reference complete! 📦✨**

For more resources:
- [Learning Roadmap](helm-learning-roadmap.md)
- [Hands-On Exercises](helm-hands-on-exercises.md)
- [Troubleshooting Guide](helm-troubleshooting-guide.md)
- [Interview Questions](helm-interview-questions.md)


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

