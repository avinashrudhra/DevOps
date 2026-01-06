# Helm Learning Roadmap

Complete 12-week curriculum to master Helm for production Kubernetes package management.

---

## 🎯 Learning Objectives

By completing this roadmap, you will:
- ✅ Master Helm charts and templating
- ✅ Create production-ready Helm charts
- ✅ Manage chart repositories
- ✅ Implement advanced templating techniques
- ✅ Handle dependencies and sub-charts
- ✅ Integrate Helm with CI/CD
- ✅ Troubleshoot Helm deployments
- ✅ Implement Helm best practices

**Target Audience:** Kubernetes administrators, DevOps engineers (3-7+ years experience)

---

## Week 1: Helm Fundamentals

### Day 1-2: Introduction & Installation
- What is Helm?
- Helm architecture
- Installation methods
- Basic concepts

**Installation:**
```bash
# macOS
brew install helm

# Linux
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Verify
helm version
```

### Day 3-4: First Helm Chart
- Using public charts
- Repository management
- Installing releases
- Basic operations

**Practice:**
```bash
# Add repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Search charts
helm search repo nginx

# Install chart
helm install my-nginx bitnami/nginx

# Check status
helm status my-nginx
helm list

# Uninstall
helm uninstall my-nginx
```

### Day 5-7: Chart Structure
- Chart.yaml
- values.yaml
- Templates directory
- Charts directory
- Creating first chart

**Create Chart:**
```bash
# Create new chart
helm create myapp

# Chart structure
myapp/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   └── _helpers.tpl
└── charts/
```

---

## Week 2: Templates & Values

### Day 8-10: Template Basics
- Template syntax
- Built-in objects
- Values access
- Functions and pipelines

**Template Examples:**
```yaml
# Access values
replicas: {{ .Values.replicaCount }}

# With default
image: {{ .Values.image.repository }}:{{ .Values.image.tag | default "latest" }}

# Built-in objects
namespace: {{ .Release.Namespace }}
name: {{ .Release.Name }}
chart: {{ .Chart.Name }}

# Conditionals
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
{{- end }}

# Loops
{{- range .Values.environments }}
- name: {{ . }}
{{- end }}
```

### Day 11-12: Functions
- String functions
- Math functions
- Logic functions
- Data structure functions

**Function Examples:**
```yaml
# String functions
{{ upper "hello" }}              # HELLO
{{ lower "HELLO" }}              # hello
{{ title "hello world" }}        # Hello World
{{ trim " hello " }}             # hello
{{ quote "value" }}              # "value"
{{ trunc 5 "hello world" }}      # hello

# Default values
{{ .Values.port | default 8080 }}

# Required values
{{ required "image.repository is required!" .Values.image.repository }}

# Include templates
{{- include "myapp.labels" . | nindent 4 }}

# toYaml
resources:
  {{- toYaml .Values.resources | nindent 2 }}
```

### Day 13-14: Named Templates
- _helpers.tpl
- Template definitions
- Template includes
- Template variables

**Helper Templates:**
```yaml
{{/* _helpers.tpl */}}
{{- define "myapp.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{- define "myapp.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}

{{- define "myapp.labels" -}}
helm.sh/chart: {{ include "myapp.chart" . }}
app.kubernetes.io/name: {{ include "myapp.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}
```

---

## Week 3: Advanced Templates

### Day 15-17: Control Structures
- if/else statements
- range loops
- with statements
- Variables

**Examples:**
```yaml
# if/else
{{- if eq .Values.service.type "NodePort" }}
  nodePort: {{ .Values.service.nodePort }}
{{- else if eq .Values.service.type "LoadBalancer" }}
  loadBalancerIP: {{ .Values.service.loadBalancerIP }}
{{- end }}

# range with index
{{- range $index, $env := .Values.environments }}
- name: ENV_{{ $index }}
  value: {{ $env }}
{{- end }}

# range with key-value
{{- range $key, $value := .Values.config }}
- name: {{ $key }}
  value: {{ $value }}
{{- end }}

# with (set context)
{{- with .Values.ingress }}
{{- if .enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    {{- toYaml .annotations | nindent 4 }}
{{- end }}
{{- end }}

# Variables
{{- $relname := .Release.Name }}
{{- $chartname := .Chart.Name }}
name: {{ printf "%s-%s" $relname $chartname }}
```

### Day 18-19: Flow Control
- Whitespace control
- Indentation
- Comments
- Template debugging

**Whitespace Control:**
```yaml
# Chomp left
{{- if .Values.enabled }}

# Chomp right
{{ .Values.name -}}

# Both
{{- .Values.name -}}

# Practical example
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "myapp.fullname" . }}
  labels:
    {{- include "myapp.labels" . | nindent 4 }}
data:
  {{- range $key, $value := .Values.config }}
  {{ $key }}: {{ $value | quote }}
  {{- end }}
```

### Day 20-21: Debugging
- helm lint
- helm template
- --dry-run --debug
- Troubleshooting templates

**Debugging Commands:**
```bash
# Lint chart
helm lint ./myapp

# Generate templates locally
helm template myrelease ./myapp

# With values
helm template myrelease ./myapp -f custom-values.yaml

# With debug output
helm install myrelease ./myapp --dry-run --debug

# Show only specific template
helm template myrelease ./myapp -s templates/deployment.yaml
```

---

## Week 4: Values & Configuration

### Day 22-24: values.yaml Best Practices
- Structure organization
- Documentation
- Default values
- Value types

**Well-Structured values.yaml:**
```yaml
# Image configuration
image:
  registry: docker.io
  repository: myapp
  tag: "1.0.0"
  pullPolicy: IfNotPresent
  pullSecrets: []

# Pod configuration
replicaCount: 2
podAnnotations: {}
podSecurityContext:
  runAsNonRoot: true
  runAsUser: 1000

# Container configuration
containerPort: 8080
resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi

livenessProbe:
  httpGet:
    path: /health
    port: http
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: http
  initialDelaySeconds: 5
  periodSeconds: 5

# Service configuration
service:
  type: ClusterIP
  port: 80
  targetPort: http
  annotations: {}

# Ingress configuration
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

# Configuration
config: {}
secrets: {}

# Additional volumes
volumes: []
volumeMounts: []
```

### Day 25-26: Multiple Environments
- Environment-specific values
- Values file hierarchy
- Merging values
- Override patterns

**Environment Values:**
```bash
# values.yaml (base)
replicaCount: 1
resources:
  limits:
    cpu: 100m
    memory: 128Mi

# values-dev.yaml
replicaCount: 1
ingress:
  host: myapp-dev.example.com

# values-production.yaml
replicaCount: 3
resources:
  limits:
    cpu: 1000m
    memory: 1Gi
ingress:
  host: myapp.example.com

# Deploy
helm install myapp ./chart -f values-production.yaml
```

### Day 27-28: Values from Different Sources
- Command line overrides
- Values files
- Environment variables
- Secrets management

**Override Examples:**
```bash
# Multiple values files (right-most takes precedence)
helm install myapp ./chart \
  -f values.yaml \
  -f values-production.yaml \
  -f values-secrets.yaml

# Command line overrides
helm install myapp ./chart \
  --set replicaCount=3 \
  --set image.tag=v2.0.0

# Nested values
helm install myapp ./chart \
  --set service.type=LoadBalancer \
  --set ingress.enabled=true

# Array values
helm install myapp ./chart \
  --set-string "environments[0]=dev" \
  --set-string "environments[1]=staging"

# File content as value
helm install myapp ./chart \
  --set-file config.json=config.json
```

---

## Week 5-6: Dependencies & Packaging

### Dependencies (Sub-charts)
```yaml
# Chart.yaml
dependencies:
  - name: postgresql
    version: 11.x.x
    repository: https://charts.bitnami.com/bitnami
    condition: postgresql.enabled
  - name: redis
    version: 17.x.x
    repository: https://charts.bitnami.com/bitnami
    condition: redis.enabled
```

**Manage Dependencies:**
```bash
# Download dependencies
helm dependency update ./myapp

# List dependencies
helm dependency list ./myapp

# Build dependencies
helm dependency build ./myapp
```

### Packaging & Distribution
```bash
# Package chart
helm package ./myapp

# Package with version
helm package ./myapp --version 1.0.1

# Package and sign
helm package ./myapp --sign --key mykey

# Create repository index
helm repo index .

# Push to OCI registry
helm push myapp-1.0.0.tgz oci://registry.example.com/charts
```

---

## Week 7-8: Hooks & Tests

### Helm Hooks
```yaml
# Pre-install hook (database migration)
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ include "myapp.fullname" . }}-db-migration
  annotations:
    "helm.sh/hook": pre-install,pre-upgrade
    "helm.sh/hook-weight": "1"
    "helm.sh/hook-delete-policy": before-hook-creation
spec:
  template:
    spec:
      containers:
        - name: migration
          image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
          command: ["./migrate"]
      restartPolicy: Never
```

### Chart Tests
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
```

---

## Week 9-10: Production Best Practices

### Security
- RBAC
- Pod Security
- Network Policies
- Secret management

### High Availability
- Multiple replicas
- Pod Disruption Budgets
- Anti-affinity rules
- Resource limits

### Monitoring & Observability
- Service monitors
- Dashboards
- Alerts
- Logging

---

## Week 11-12: CI/CD Integration & Advanced Topics

### Jenkins Integration
```groovy
stage('Deploy with Helm') {
    steps {
        sh """
            helm upgrade --install ${APP_NAME} ./chart \
              --namespace ${NAMESPACE} \
              --set image.tag=${BUILD_NUMBER} \
              --wait --timeout 5m
        """
    }
}
```

### ArgoCD Integration
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
spec:
  source:
    helm:
      values: |
        replicaCount: 3
      parameters:
        - name: image.tag
          value: v1.0.0
```

**Complete Helm mastery in 12 weeks! 🎓📦**

For more resources:
- [Quick Reference](helm-quick-reference.md)
- [Hands-On Exercises](helm-hands-on-exercises.md)
- [Troubleshooting Guide](helm-troubleshooting-guide.md)
- [Interview Questions](helm-interview-questions.md)


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

