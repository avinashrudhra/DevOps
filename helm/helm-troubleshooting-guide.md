# Helm Troubleshooting Guide

Comprehensive solutions to common Helm issues in production environments.

---

## Table of Contents
1. [Installation Issues](#installation-issues)
2. [Chart Creation Problems](#chart-creation-problems)
3. [Template Errors](#template-errors)
4. [Release Management](#release-management)
5. [Dependency Issues](#dependency-issues)
6. [Values & Configuration](#values--configuration)
7. [Hooks Problems](#hooks-problems)
8. [Performance Issues](#performance-issues)
9. [Repository Problems](#repository-problems)
10. [Production Incidents](#production-incidents)

---

## Installation Issues

### Issue 1: Helm Command Not Found

**Symptoms:**
```bash
$ helm version
bash: helm: command not found
```

**Solutions:**

**macOS:**
```bash
brew install helm
```

**Linux:**
```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Verify
helm version
```

**Manual Installation:**
```bash
# Download
wget https://get.helm.sh/helm-v3.12.0-linux-amd64.tar.gz

# Extract
tar -zxvf helm-v3.12.0-linux-amd64.tar.gz

# Move to PATH
sudo mv linux-amd64/helm /usr/local/bin/helm

# Verify
helm version
```

---

### Issue 2: Cannot Connect to Kubernetes Cluster

**Symptoms:**
```
Error: Kubernetes cluster unreachable
Unable to connect to the server
```

**Diagnosis:**
```bash
# Check kubectl connectivity
kubectl cluster-info

# Check kubeconfig
echo $KUBECONFIG
kubectl config view

# Check current context
kubectl config current-context
```

**Solutions:**

**1. Set Kubeconfig:**
```bash
export KUBECONFIG=~/.kube/config

# Or specify in helm command
helm list --kubeconfig ~/.kube/config
```

**2. Switch Context:**
```bash
kubectl config get-contexts
kubectl config use-context my-cluster
```

**3. Update Cluster Certificate:**
```bash
kubectl config set-cluster my-cluster --insecure-skip-tls-verify=true
```

---

## Chart Creation Problems

### Issue 3: Chart Lint Failures

**Symptoms:**
```bash
$ helm lint ./myapp
==> Linting ./myapp
[ERROR] Chart.yaml: version is required
[ERROR] templates/: object name does not conform to Kubernetes naming requirements
```

**Solutions:**

**1. Fix Chart.yaml:**
```yaml
apiVersion: v2
name: myapp
description: My Application
version: 1.0.0        # Must be SEMVER
appVersion: "1.0.0"   # Application version
type: application
```

**2. Fix Resource Names:**
```yaml
# ❌ Bad
metadata:
  name: My_App

# ✅ Good
metadata:
  name: my-app
```

**3. Common Lint Issues:**
```bash
# Run with debug
helm lint ./myapp --debug

# Check specific issues
# - Invalid YAML syntax
# - Missing required fields
# - Invalid Kubernetes resource names
# - Template syntax errors
```

---

## Template Errors

### Issue 4: Template Parsing Error

**Symptoms:**
```
Error: template: myapp/templates/deployment.yaml:10:14: executing "myapp/templates/deployment.yaml" at <.Values.replicaCount>: nil pointer evaluating interface {}.replicaCount
```

**Diagnosis:**
```bash
# Generate templates locally
helm template myapp ./myapp --debug

# Check specific template
helm template myapp ./myapp -s templates/deployment.yaml
```

**Solutions:**

**1. Use Default Values:**
```yaml
# ❌ Bad
replicas: {{ .Values.replicaCount }}

# ✅ Good
replicas: {{ .Values.replicaCount | default 1 }}
```

**2. Check if Value Exists:**
```yaml
{{- if .Values.ingress }}
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
{{- end }}
{{- end }}
```

**3. Use Required:**
```yaml
image: {{ required "image.repository is required!" .Values.image.repository }}
```

---

### Issue 5: Whitespace Issues

**Symptoms:**
```yaml
# Generated manifest has extra blank lines
apiVersion: v1
kind: ConfigMap

data:

  key: value
```

**Solutions:**

**Use Whitespace Control:**
```yaml
# Remove left whitespace
{{- if .Values.enabled }}

# Remove right whitespace
{{ .Values.name -}}

# Remove both
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

---

### Issue 6: Indentation Errors

**Symptoms:**
```
Error: YAML parse error
error converting YAML to JSON: yaml: line 12: mapping values are not allowed in this context
```

**Solutions:**

**Use nindent:**
```yaml
# ❌ Bad
labels:
{{ include "myapp.labels" . | indent 2 }}  # Wrong indentation

# ✅ Good
labels:
  {{- include "myapp.labels" . | nindent 2 }}

# For YAML blocks
resources:
  {{- toYaml .Values.resources | nindent 2 }}
```

---

## Release Management

### Issue 7: Release Already Exists

**Symptoms:**
```
Error: INSTALLATION FAILED: cannot re-use a name that is still in use
```

**Solutions:**

```bash
# Check existing releases
helm list
helm list --all

# Uninstall existing release
helm uninstall myrelease

# Or use upgrade --install
helm upgrade --install myrelease ./myapp
```

---

### Issue 8: Upgrade Failed, Release in Pending State

**Symptoms:**
```bash
$ helm list
NAME      STATUS          CHART
myapp     pending-upgrade myapp-1.0.0
```

**Diagnosis:**
```bash
# Check release status
helm status myapp

# Get release history
helm history myapp

# Check pods
kubectl get pods -n default
```

**Solutions:**

**1. Rollback:**
```bash
# Rollback to previous version
helm rollback myapp

# Or to specific revision
helm rollback myapp 2
```

**2. Force Delete (if rollback fails):**
```bash
# Delete release record (use with caution!)
kubectl delete secret -l owner=helm,name=myapp -n default

# Clean up resources
kubectl delete all -l app.kubernetes.io/instance=myapp

# Reinstall
helm install myapp ./myapp
```

---

### Issue 9: Cannot Rollback Release

**Symptoms:**
```
Error: UPGRADE FAILED: unable to rollback
```

**Solutions:**

**1. Check History:**
```bash
helm history myapp

# If no history
helm uninstall myapp
helm install myapp ./myapp
```

**2. Force Rollback:**
```bash
helm rollback myapp --force --wait
```

---

## Dependency Issues

### Issue 10: Dependency Update Fails

**Symptoms:**
```
Error: can't get a valid version for repositories
Unable to find chart postgresql
```

**Solutions:**

**1. Update Repositories:**
```bash
helm repo update
helm dependency update ./myapp
```

**2. Check Chart.yaml:**
```yaml
dependencies:
  - name: postgresql
    version: "11.x.x"  # Use version constraints
    repository: https://charts.bitnami.com/bitnami  # Correct URL
```

**3. Manual Download:**
```bash
# Download specific version
helm pull bitnami/postgresql --version 11.6.26

# Move to charts directory
mv postgresql-11.6.26.tgz myapp/charts/
```

---

### Issue 11: Sub-chart Values Not Applied

**Symptoms:**
- Sub-chart uses default values
- Custom values not passed to dependency

**Solutions:**

**values.yaml:**
```yaml
# Pass values to postgresql sub-chart
postgresql:
  enabled: true
  auth:
    username: myapp
    password: mypassword
    database: myappdb
  primary:
    resources:
      limits:
        cpu: 500m
        memory: 512Mi
```

**Import Values from Sub-chart:**
```yaml
# Chart.yaml
dependencies:
  - name: postgresql
    version: "11.x.x"
    repository: https://charts.bitnami.com/bitnami
    import-values:
      - child: auth.password
        parent: dbPassword
```

---

## Values & Configuration

### Issue 12: Values Not Being Applied

**Symptoms:**
```bash
$ helm get values myapp
# Shows empty or unexpected values
```

**Diagnosis:**
```bash
# Check values hierarchy
helm get values myapp
helm get values myapp --all  # Include defaults

# Test template rendering
helm template myapp ./myapp -f custom-values.yaml --debug
```

**Solutions:**

**1. Check Values File Path:**
```bash
# ❌ Wrong
helm install myapp ./myapp -f values/production.yaml

# ✅ Correct (relative to chart directory)
helm install myapp ./myapp -f values-production.yaml
```

**2. Values Override Order:**
```bash
# Last value wins
helm install myapp ./myapp \
  -f values.yaml \           # Base values
  -f values-prod.yaml \      # Environment values
  --set replicaCount=5       # Command line (highest priority)
```

**3. Check YAML Syntax:**
```bash
# Validate YAML
yamllint values.yaml
```

---

## Hooks Problems

### Issue 13: Pre-Install Hook Failing

**Symptoms:**
```
Error: pre-install hooks failed
Job failed: BackoffLimitExceeded
```

**Diagnosis:**
```bash
# Check hook jobs
kubectl get jobs -l helm.sh/hook

# Check pod logs
kubectl logs job/myapp-pre-install
```

**Solutions:**

**1. Increase Timeout:**
```bash
helm install myapp ./myapp --timeout 10m
```

**2. Fix Hook Definition:**
```yaml
metadata:
  annotations:
    "helm.sh/hook": pre-install
    "helm.sh/hook-weight": "0"
    "helm.sh/hook-delete-policy": hook-succeeded,hook-failed  # Clean up
spec:
  backoffLimit: 3  # Retry failed hooks
  activeDeadlineSeconds: 300  # 5 minutes timeout
```

**3. Skip Hooks (temporary):**
```bash
helm install myapp ./myapp --no-hooks
```

---

## Performance Issues

### Issue 14: Slow Helm Operations

**Symptoms:**
- `helm list` takes very long
- `helm install` timeout
- High CPU usage

**Solutions:**

**1. Increase Timeout:**
```bash
helm install myapp ./myapp --timeout 10m --wait
```

**2. Optimize Templates:**
```bash
# Check template complexity
helm template myapp ./myapp --debug | wc -l

# Reduce template processing
# - Remove unnecessary loops
# - Cache helper results
# - Simplify conditionals
```

**3. Clean Old Releases:**
```bash
# List all releases (including deleted)
helm list --all

# Uninstall old releases
helm uninstall old-release --keep-history=false
```

---

## Repository Problems

### Issue 15: Cannot Add Repository

**Symptoms:**
```
Error: looks like "https://charts.example.com" is not a valid chart repository
```

**Solutions:**

**1. Verify Repository URL:**
```bash
# Check index.yaml exists
curl https://charts.example.com/index.yaml

# Add repository
helm repo add myrepo https://charts.example.com
```

**2. Fix Repository Index:**
```bash
# Generate index for repository
helm repo index ./charts --url https://charts.example.com
```

**3. Skip TLS Verification (not recommended):**
```bash
helm repo add myrepo https://charts.example.com --insecure-skip-tls-verify
```

---

## Production Incidents

### Issue 16: Emergency Rollback

**Immediate Actions:**

```bash
# 1. Check current status
helm list
helm status myapp

# 2. Check history
helm history myapp

# 3. Quick rollback
helm rollback myapp

# 4. Or rollback to specific working version
helm rollback myapp 3

# 5. Verify rollback
kubectl get pods -w
helm status myapp

# 6. Check application health
kubectl describe pod myapp-xxx
kubectl logs myapp-xxx
```

---

### Issue 17: Release Stuck, Cluster Issues

**Recovery Steps:**

```bash
# 1. Check Kubernetes cluster health
kubectl get nodes
kubectl get pods --all-namespaces

# 2. If release stuck, force delete
kubectl delete secret -l owner=helm,name=myapp

# 3. Clean up resources
kubectl delete all,cm,secret -l app.kubernetes.io/instance=myapp

# 4. Reinstall
helm install myapp ./myapp --wait --timeout 10m
```

---

## Debugging Tips

### General Debugging Commands

```bash
# 1. Dry run with debug
helm install myapp ./myapp --dry-run --debug

# 2. Template rendering
helm template myapp ./myapp --debug

# 3. Lint chart
helm lint ./myapp --strict

# 4. Get manifest
helm get manifest myapp

# 5. Check values
helm get values myapp --all

# 6. Release history
helm history myapp

# 7. Kubernetes events
kubectl get events --sort-by='.lastTimestamp'

# 8. Pod logs
kubectl logs -l app.kubernetes.io/instance=myapp --all-containers
```

---

**Complete Helm troubleshooting guide! 🔧📦**

For more resources:
- [Learning Roadmap](helm-learning-roadmap.md)
- [Quick Reference](helm-quick-reference.md)
- [Hands-On Exercises](helm-hands-on-exercises.md)
- [Interview Questions](helm-interview-questions.md)


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

