# ArgoCD Troubleshooting Guide

Comprehensive solutions to common ArgoCD issues in production GitOps environments.

---

## Table of Contents
1. [Installation Issues](#installation-issues)
2. [Authentication Problems](#authentication-problems)
3. [Git Repository Access](#git-repository-access)
4. [Sync Failures](#sync-failures)
5. [Application Health Issues](#application-health-issues)
6. [Performance Problems](#performance-problems)
7. [RBAC and Permissions](#rbac-and-permissions)
8. [Multi-Cluster Issues](#multi-cluster-issues)
9. [Helm-Specific Issues](#helm-specific-issues)
10. [Production Incidents](#production-incidents)

---

## Installation Issues

### Issue 1: ArgoCD Pods Not Starting

**Symptoms:**
```bash
$ kubectl get pods -n argocd
NAME                                  READY   STATUS             RESTARTS
argocd-server-xxx                     0/1     CrashLoopBackOff   5
argocd-repo-server-xxx                0/1     ImagePullBackOff   0
```

**Diagnosis:**
```bash
# Check pod status
kubectl describe pod argocd-server-xxx -n argocd

# Check logs
kubectl logs argocd-server-xxx -n argocd

# Check events
kubectl get events -n argocd --sort-by='.lastTimestamp'
```

**Solutions:**

**1. Image Pull Issues:**
```bash
# Check if images are accessible
docker pull quay.io/argoproj/argocd:latest

# Use private registry
kubectl create secret docker-registry argocd-registry \
  --docker-server=registry.example.com \
  --docker-username=user \
  --docker-password=pass \
  -n argocd
```

**2. Insufficient Resources:**
```yaml
# Increase resource limits
apiVersion: apps/v1
kind: Deployment
metadata:
  name: argocd-server
spec:
  template:
    spec:
      containers:
        - name: argocd-server
          resources:
            requests:
              cpu: 100m
              memory: 256Mi
            limits:
              cpu: 500m
              memory: 512Mi
```

---

### Issue 2: Unable to Access ArgoCD UI

**Symptoms:**
- Browser shows "Connection refused"
- Timeout when accessing UI
- 502 Bad Gateway

**Solutions:**

**1. Check Service:**
```bash
# Verify service
kubectl get svc argocd-server -n argocd

# Check endpoints
kubectl get endpoints argocd-server -n argocd
```

**2. Port Forward:**
```bash
# Kill existing port-forward
pkill -f "port-forward.*argocd"

# Create new port-forward
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

**3. Ingress Issues:**
```bash
# Check ingress
kubectl get ingress -n argocd
kubectl describe ingress argocd-server-ingress -n argocd

# Verify TLS certificate
kubectl get secret argocd-server-tls -n argocd
```

**4. Fix Ingress Configuration:**
```yaml
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
```

---

## Authentication Problems

### Issue 3: Lost Admin Password

**Symptoms:**
- Cannot login to ArgoCD
- Forgot admin password
- Initial password not working

**Solutions:**

**1. Reset Admin Password:**
```bash
# Get current password hash
kubectl -n argocd get secret argocd-secret \
  -o jsonpath="{.data.admin\.password}" | base64 -d

# Generate new password
NEW_PASSWORD="newpassword"
BCRYPT_HASH=$(htpasswd -nbBC 10 "" $NEW_PASSWORD | tr -d ':\n' | sed 's/$2y/$2a/')

# Update secret
kubectl -n argocd patch secret argocd-secret \
  -p "{\"data\": {\"admin.password\": \"$(echo -n $BCRYPT_HASH | base64)\"}}"

# Restart server
kubectl -n argocd delete pod -l app.kubernetes.io/name=argocd-server
```

**2. Disable Authentication (Temporary):**
```bash
# NOT RECOMMENDED FOR PRODUCTION
kubectl patch cm argocd-cm -n argocd --type merge \
  -p '{"data": {"users.anonymous.enabled": "true"}}'
```

---

### Issue 4: SSO Authentication Failing

**Symptoms:**
```
Failed to authenticate: invalid_grant
OIDC authentication failed
```

**Diagnosis:**
```bash
# Check Dex logs
kubectl logs deployment/argocd-dex-server -n argocd

# Check configuration
kubectl get cm argocd-cm -n argocd -o yaml
```

**Solutions:**

**1. Fix OIDC Configuration:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
data:
  url: https://argocd.example.com
  dex.config: |
    connectors:
      - type: oidc
        id: google
        name: Google
        config:
          issuer: https://accounts.google.com
          clientID: $GOOGLE_CLIENT_ID
          clientSecret: $GOOGLE_CLIENT_SECRET
          redirectURI: https://argocd.example.com/api/dex/callback
```

**2. Check Callback URL:**
- Ensure redirectURI matches in OAuth provider
- Must be exact match including protocol
- Check for trailing slashes

---

## Git Repository Access

### Issue 5: Repository Authentication Failed

**Symptoms:**
```
ComparisonError: repository not found
authentication required
permission denied
```

**Diagnosis:**
```bash
# Check repository
argocd repo get https://github.com/user/repo.git

# List repositories
argocd repo list
```

**Solutions:**

**1. Re-add Repository with Token:**
```bash
# Remove old repository
argocd repo rm https://github.com/user/repo.git

# Add with personal access token
argocd repo add https://github.com/user/repo.git \
  --username oauth2 \
  --password ghp_xxxxxxxxxxxx
```

**2. SSH Key Issues:**
```bash
# Generate new SSH key
ssh-keygen -t ed25519 -f ~/.ssh/argocd_rsa

# Add to GitHub
cat ~/.ssh/argocd_rsa.pub
# Copy to GitHub Settings → SSH Keys

# Add to ArgoCD
argocd repo add git@github.com:user/repo.git \
  --ssh-private-key-path ~/.ssh/argocd_rsa
```

**3. Certificate Issues:**
```bash
# Skip TLS verification (not recommended)
argocd repo add https://github.com/user/repo.git \
  --insecure-skip-server-verification

# Add custom CA certificate
kubectl create secret generic repo-cert \
  --from-file=cert=ca.crt \
  -n argocd
```

---

### Issue 6: Large Repository Clone Timeout

**Symptoms:**
```
fatal: The remote end hung up unexpectedly
fatal: early EOF
ComparisonError: timeout
```

**Solutions:**

**1. Increase Timeout:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
data:
  timeout.reconciliation: 300s
```

**2. Shallow Clone:**
```yaml
# In Application spec
spec:
  source:
    repoURL: https://github.com/user/large-repo.git
    targetRevision: HEAD
    path: k8s
  syncOptions:
    - CreateNamespace=true
```

**3. Use Submodules with Shallow:**
```bash
# Configure in argocd-cm
data:
  resource.customizations.ignoreDifferences.extensions_Ingress: |
    jsonPointers:
    - /spec/rules/0/host
```

---

## Sync Failures

### Issue 7: Application Stuck in Progressing

**Symptoms:**
```
Sync Status: OutOfSync
Health Status: Progressing
Operation stuck for hours
```

**Diagnosis:**
```bash
# Check application status
argocd app get myapp

# Check sync operation
argocd app get myapp --show-operation

# Check resource status
kubectl get all -n production -l app=myapp
```

**Solutions:**

**1. Terminate Operation:**
```bash
# Terminate stuck operation
argocd app terminate-op myapp

# Force sync
argocd app sync myapp --force --prune
```

**2. Check Resource Hooks:**
```bash
# View hooks
kubectl get job -n production
kubectl logs job/myapp-migration -n production

# Delete failed hooks
kubectl delete job myapp-migration -n production

# Sync again
argocd app sync myapp
```

---

### Issue 8: Sync Fails with Validation Error

**Symptoms:**
```
error: error validating data
CustomResourceDefinition.apiextensions.k8s.io is invalid
```

**Solutions:**

**1. Skip Validation:**
```yaml
syncPolicy:
  syncOptions:
    - Validate=false
```

**2. Fix CRD Order:**
```yaml
# Add sync waves to control order
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: mycrd.example.com
  annotations:
    argocd.argoproj.io/sync-wave: "-1"  # Apply CRD first
---
apiVersion: example.com/v1
kind: MyCRD
metadata:
  name: myresource
  annotations:
    argocd.argoproj.io/sync-wave: "0"  # Apply CR after CRD
```

---

## Application Health Issues

### Issue 9: Application Shows Degraded

**Symptoms:**
```
Health Status: Degraded
Some pods not running
Resources unhealthy
```

**Diagnosis:**
```bash
# Check application resources
argocd app resources myapp

# Check pod status
kubectl get pods -n production

# Check events
kubectl get events -n production --sort-by='.lastTimestamp'
```

**Solutions:**

**1. Check Pod Logs:**
```bash
# Get pod logs
kubectl logs deployment/myapp -n production

# Check previous logs
kubectl logs deployment/myapp -n production --previous
```

**2. Custom Health Check:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
data:
  resource.customizations.health.apps_Deployment: |
    hs = {}
    if obj.status ~= nil then
      if obj.status.replicas ~= nil and obj.status.updatedReplicas ~= nil then
        if obj.status.replicas == obj.status.updatedReplicas and 
           obj.status.availableReplicas == obj.status.replicas then
          hs.status = "Healthy"
          hs.message = "Deployment is healthy"
        else
          hs.status = "Progressing"
          hs.message = "Waiting for rollout to finish"
        end
      end
    end
    return hs
```

---

## Performance Problems

### Issue 10: Slow Sync Operations

**Symptoms:**
- Sync takes very long time
- High CPU usage
- Memory pressure

**Solutions:**

**1. Increase Resources:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: argocd-repo-server
  namespace: argocd
spec:
  template:
    spec:
      containers:
        - name: argocd-repo-server
          resources:
            requests:
              cpu: 500m
              memory: 1Gi
            limits:
              cpu: 1000m
              memory: 2Gi
```

**2. Enable Caching:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
data:
  resource.compareoptions: |
    ignoreAggregatedRoles: true
  resource.exclusions: |
    - apiGroups:
      - "*"
      kinds:
      - Event
```

**3. Use Application Sharding:**
```yaml
# For large number of applications
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: argocd-application-controller
spec:
  replicas: 3  # Shard across replicas
  template:
    spec:
      containers:
        - name: argocd-application-controller
          env:
            - name: ARGOCD_CONTROLLER_REPLICAS
              value: "3"
```

---

## RBAC and Permissions

### Issue 11: User Cannot Sync Application

**Symptoms:**
```
permission denied
User does not have access to sync application
```

**Solutions:**

**Update RBAC Policy:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-rbac-cm
  namespace: argocd
data:
  policy.csv: |
    p, role:developer, applications, sync, default/*, allow
    p, role:developer, applications, get, default/*, allow
    g, dev-team, role:developer
```

---

## Multi-Cluster Issues

### Issue 12: Cannot Add Cluster

**Symptoms:**
```
Unable to create cluster secret
Connection refused
```

**Solutions:**

```bash
# Verify kubeconfig context
kubectl config current-context

# Test cluster connectivity
kubectl --context prod-cluster get nodes

# Add cluster with service account
argocd cluster add prod-cluster \
  --name production \
  --service-account argocd-manager
```

---

## Helm-Specific Issues

### Issue 13: Helm Values Not Applied

**Symptoms:**
- Values override not working
- Default values used instead

**Solutions:**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
spec:
  source:
    helm:
      # Use values block instead of valueFiles for inline values
      values: |
        replicaCount: 3
        image:
          tag: v2.0.0
      # Or use valueFiles
      valueFiles:
        - values-production.yaml
      # Parameters for specific overrides
      parameters:
        - name: image.tag
          value: v2.0.1
          forceString: true
```

---

## Production Incidents

### Issue 14: Emergency Rollback

**Immediate Actions:**

```bash
# Check application history
argocd app history myapp

# Rollback to previous version
argocd app rollback myapp

# Rollback to specific revision
argocd app rollback myapp 5

# Verify rollback
argocd app get myapp
kubectl get pods -n production -w
```

---

**Complete troubleshooting guide for production ArgoCD! 🔧**

For more resources:
- [Learning Roadmap](argocd-learning-roadmap.md)
- [Quick Reference](argocd-quick-reference.md)
- [Hands-On Exercises](argocd-hands-on-exercises.md)
- [Interview Questions](argocd-interview-questions.md)


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

