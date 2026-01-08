# Kubernetes Troubleshooting - Complete Practical Guide

**Debugging Common Issues: OOM, CrashLoopBackOff, ImagePullBackOff, and More**

---

## 📚 Table of Contents

1. [Pod Status Issues](#pod-status-issues)
2. [OOMKilled (Out of Memory)](#oomkilled)
3. [CrashLoopBackOff](#crashloopbackoff)
4. [ImagePullBackOff](#imagepullbackoff)
5. [Pending Pods](#pending-pods)
6. [Networking Issues](#networking-issues)
7. [Troubleshooting Commands](#troubleshooting-commands)

---

## 🚨 Pod Status Issues

Common pod statuses and what they mean.

### **Pod Lifecycle:**

```
Pending → ContainerCreating → Running → Succeeded/Failed/CrashLoopBackOff
```

---

### **Status Overview:**

| Status | Meaning | Action |
|--------|---------|--------|
| Pending | Waiting to be scheduled | Check resources, node selectors |
| ContainerCreating | Pulling image, starting | Wait or check image issues |
| Running | All containers running | Monitor logs |
| Succeeded | Completed successfully (Jobs) | Normal for Jobs |
| Failed | Terminated with error | Check logs, fix error |
| CrashLoopBackOff | Keeps crashing and restarting | Check logs, fix application |
| Error | Pod encountered error | Check describe output |
| Unknown | Cannot determine status | Check node health |
| ImagePullBackOff | Cannot pull container image | Check image name, credentials |
| OOMKilled | Out of memory | Increase memory limits |

---

## 💀 OOMKilled (Out of Memory)

Pod killed because it exceeded memory limit.

### **Symptoms:**

```bash
kubectl get pods

NAME      READY   STATUS      RESTARTS   AGE
myapp     0/1     OOMKilled   5          10m

# Or CrashLoopBackOff with OOMKilled in history
NAME      READY   STATUS             RESTARTS   AGE
myapp     0/1     CrashLoopBackOff   10         20m
```

---

### **Diagnosis:**

```bash
# Check pod status
kubectl describe pod myapp

Last State:     Terminated
  Reason:       OOMKilled    # ← Clear indicator
  Exit Code:    137          # ← OOM exit code
  Started:      ...
  Finished:     ...

# Check resource limits
kubectl get pod myapp -o yaml | grep -A 5 resources

resources:
  limits:
    memory: 256Mi  # Too low?
  requests:
    memory: 128Mi
```

---

### **Understanding OOMKilled:**

```
Pod resource spec:
  requests:
    memory: 128Mi  # Scheduler guarantee
  limits:
    memory: 256Mi  # Hard cap!

Pod memory usage:
  T+0s:   50Mi   ✓ (within limit)
  T+30s:  150Mi  ✓ (within limit)
  T+60s:  250Mi  ✓ (near limit)
  T+61s:  256Mi  ✓ (at limit)
  T+62s:  Tries to allocate more...
  T+63s:  KILLED by OOMKiller! ✗
  
Exit code: 137
Reason: OOMKilled
```

---

### **Common Causes:**

**1. Memory Leak:**
```
Application has memory leak
Usage grows over time
Eventually hits limit
Fix: Fix application code
```

**2. Insufficient Limits:**
```
Application legitimately needs more memory
Limit too low
Fix: Increase memory limit
```

**3. Spike in Traffic:**
```
Sudden traffic spike
More data in memory
Fix: Increase limits or add autoscaling
```

---

### **Solutions:**

**1. Increase Memory Limit:**
```yaml
spec:
  containers:
  - name: app
    resources:
      limits:
        memory: 512Mi  # Was 256Mi
      requests:
        memory: 256Mi  # Was 128Mi
```

**2. Add HPA:**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    kind: Deployment
    name: myapp
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 70
```

**3. Fix Memory Leak:**
```
Profile application
Find leak
Fix in code
Redeploy
```

---

## 🔄 CrashLoopBackOff

Container keeps crashing and Kubernetes keeps restarting it.

### **Symptoms:**

```bash
kubectl get pods

NAME      READY   STATUS             RESTARTS   AGE
myapp     0/1     CrashLoopBackOff   10         15m

# RESTARTS increasing
# Status: CrashLoopBackOff or Error
```

---

### **Diagnosis:**

```bash
# Check pod events
kubectl describe pod myapp

Events:
  Type     Reason     Message
  ----     ------     -------
  Normal   Created    Created container app
  Normal   Started    Started container app
  Warning  BackOff    Back-off restarting failed container

# Check logs
kubectl logs myapp

# If container crashed, check previous logs
kubectl logs myapp --previous

Error: Database connection failed
Cannot connect to postgres:5432
```

---

### **Understanding Backoff:**

```
Restart attempts with increasing delays:

Attempt 1: Immediate restart
Attempt 2: 10s delay
Attempt 3: 20s delay
Attempt 4: 40s delay
Attempt 5: 80s delay
Attempt 6: 160s delay (max)
...
Continues with 160s delay

"CrashLoopBackOff" = Waiting in backoff period
```

---

### **Common Causes:**

**1. Application Error:**
```bash
# Logs show:
Error: Missing environment variable DATABASE_URL
panic: runtime error

Fix: Fix application error, add missing config
```

**2. Missing Dependencies:**
```bash
# Logs show:
Error: Cannot connect to database
Connection refused: postgres:5432

Fix: Ensure dependencies are running
```

**3. Wrong Command/Args:**
```yaml
# Bad:
spec:
  containers:
  - name: app
    image: myapp:1.0
    command: ["node"]
    args: ["server.js"]  # File doesn't exist!

Fix: Correct command/args
```

**4. Insufficient Resources:**
```bash
# Describe shows:
Last State:     Terminated
  Reason:       OOMKilled
  
Fix: Increase resources
```

**5. Liveness Probe Failure:**
```yaml
# Probe failing immediately
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 5  # App needs 30s to start!
  
Fix: Increase initialDelaySeconds
```

---

### **Solutions:**

**1. Check Logs:**
```bash
# Current logs
kubectl logs myapp

# Previous container logs (if crashed)
kubectl logs myapp --previous

# Follow logs
kubectl logs myapp -f

# Logs from specific container (multi-container pod)
kubectl logs myapp -c sidecar
```

**2. Fix Application:**
```
Based on logs:
  - Fix code errors
  - Add missing configuration
  - Fix dependencies
```

**3. Adjust Resource Limits:**
```yaml
resources:
  limits:
    memory: 512Mi  # Increase if OOMKilled
    cpu: 1000m
  requests:
    memory: 256Mi
    cpu: 500m
```

**4. Fix Probes:**
```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30  # Give app time to start
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3
```

---

## 🖼️ ImagePullBackOff

Cannot pull container image.

### **Symptoms:**

```bash
kubectl get pods

NAME      READY   STATUS             RESTARTS   AGE
myapp     0/1     ImagePullBackOff   0          5m
# or
NAME      READY   STATUS         RESTARTS   AGE
myapp     0/1     ErrImagePull   0          30s
```

---

### **Diagnosis:**

```bash
kubectl describe pod myapp

Events:
  Type     Reason     Message
  ----     ------     -------
  Normal   Scheduled  Successfully assigned default/myapp to node-1
  Normal   Pulling    Pulling image "myapp:v2.0"
  Warning  Failed     Failed to pull image "myapp:v2.0": rpc error: code = NotFound desc = failed to pull and unpack image "docker.io/library/myapp:v2.0": pull access denied, repository does not exist or may require docker login
  Warning  Failed     Error: ErrImagePull
  Normal   BackOff    Back-off pulling image "myapp:v2.0"
  Warning  Failed     Error: ImagePullBackOff
```

---

### **Common Causes:**

**1. Image Doesn't Exist:**
```yaml
spec:
  containers:
  - name: app
    image: myapp:v99.0  # Typo or doesn't exist!

Fix: Correct image name/tag
```

**2. Private Registry - No Credentials:**
```yaml
spec:
  containers:
  - name: app
    image: myregistry.com/myapp:1.0  # Private registry

Fix: Add imagePullSecrets
```

**3. Network Issues:**
```
Cannot reach registry
Firewall blocking
DNS issues

Fix: Check network connectivity
```

**4. Registry Down:**
```
Container registry unavailable
DockerHub rate limiting

Fix: Wait or use different registry
```

---

### **Solutions:**

**1. Verify Image:**
```bash
# Check if image exists
docker pull myapp:v2.0

# Or
kubectl run test --image=myapp:v2.0 --dry-run=client
```

**2. Private Registry - Add Credentials:**
```bash
# Create docker registry secret
kubectl create secret docker-registry regcred \
  --docker-server=myregistry.com \
  --docker-username=myuser \
  --docker-password=mypassword \
  --docker-email=user@example.com
```

```yaml
spec:
  imagePullSecrets:
  - name: regcred  # Use the secret
  containers:
  - name: app
    image: myregistry.com/myapp:1.0
```

**3. Use Public Image for Testing:**
```yaml
spec:
  containers:
  - name: app
    image: nginx:1.21  # Public image to test
```

---

## ⏳ Pending Pods

Pod stuck in Pending status.

### **Symptoms:**

```bash
kubectl get pods

NAME      READY   STATUS    RESTARTS   AGE
myapp     0/1     Pending   0          10m
```

---

### **Diagnosis:**

```bash
kubectl describe pod myapp

Events:
  Type     Reason            Message
  ----     ------            -------
  Warning  FailedScheduling  0/3 nodes are available: 1 node(s) had taint {node-role.kubernetes.io/master: }, that the pod didn't tolerate, 2 Insufficient cpu.
```

---

### **Common Causes:**

**1. Insufficient Resources:**
```
Events:
  0/3 nodes available: insufficient cpu
  OR
  0/3 nodes available: insufficient memory

Cluster doesn't have enough resources

Fix: Add nodes or reduce resource requests
```

**2. Node Selector Mismatch:**
```yaml
spec:
  nodeSelector:
    disktype: ssd  # No nodes have this label!

Fix: Remove nodeSelector or label nodes
```

**3. Taints on Nodes:**
```
Events:
  0/3 nodes: 3 node(s) had taints that the pod didn't tolerate

Fix: Add tolerations or remove taints
```

**4. PVC Not Bound:**
```yaml
spec:
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: my-pvc  # PVC not bound yet

Fix: Check PVC status, create PV
```

**5. Pod Affinity Not Met:**
```yaml
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - cache
        topologyKey: kubernetes.io/hostname

# No nodes have pods matching this requirement

Fix: Adjust affinity rules
```

---

### **Solutions:**

**1. Check Cluster Resources:**
```bash
# Node resources
kubectl describe nodes | grep -A 5 "Allocated resources"

# Check if nodes full
kubectl top nodes
```

**2. Reduce Resource Requests:**
```yaml
resources:
  requests:
    cpu: 100m      # Was 1000m
    memory: 128Mi  # Was 1Gi
```

**3. Add Nodes:**
```bash
# Scale node group (cloud-specific)
# AWS:
eksctl scale nodegroup --cluster=my-cluster --nodes=5 ng-1

# GKE:
gcloud container clusters resize my-cluster --num-nodes=5
```

**4. Fix PVC:**
```bash
# Check PVC status
kubectl get pvc

NAME     STATUS    VOLUME   CAPACITY
my-pvc   Pending   

# Check why pending
kubectl describe pvc my-pvc

# Create matching PV or use dynamic provisioning
```

---

## 🌐 Networking Issues

Pods cannot communicate.

### **Symptoms:**

```bash
# Cannot reach service
curl http://myservice
curl: (7) Failed to connect

# DNS not working
nslookup myservice
;; connection timed out
```

---

### **Common Issues:**

**1. Service Selector Mismatch:**
```yaml
# Service
spec:
  selector:
    app: web  # Looking for app=web

# Pod
metadata:
  labels:
    app: webapp  # Doesn't match!

Fix: Match labels
```

**2. No Endpoints:**
```bash
kubectl get endpoints myservice

NAME        ENDPOINTS
myservice   <none>  # No pods matched!

Fix: Fix service selector or pod labels
```

**3. Network Policy Blocking:**
```yaml
# Network policy denying traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress  # Blocks all ingress!

Fix: Adjust network policy or add allow rules
```

**4. DNS Issues:**
```bash
# Test DNS
kubectl run test --image=busybox -it --rm -- nslookup myservice

# If fails, check CoreDNS
kubectl get pods -n kube-system | grep coredns
```

---

### **Troubleshooting:**

```bash
# 1. Check service
kubectl get svc myservice
kubectl describe svc myservice

# 2. Check endpoints
kubectl get endpoints myservice

# 3. Test from another pod
kubectl run test --image=busybox -it --rm -- sh
wget -O- http://myservice:8080

# 4. Check network policies
kubectl get networkpolicies

# 5. Check pod connectivity
kubectl exec -it mypod -- curl http://other-pod-ip:8080
```

---

## 🛠️ Troubleshooting Commands

Essential commands for debugging.

### **Pod Inspection:**

```bash
# Get pods
kubectl get pods
kubectl get pods -o wide  # With node and IP
kubectl get pods --all-namespaces  # All namespaces

# Describe pod (most useful!)
kubectl describe pod myapp

# Pod YAML
kubectl get pod myapp -o yaml

# Pod logs
kubectl logs myapp
kubectl logs myapp --previous  # Previous container
kubectl logs myapp -f  # Follow logs
kubectl logs myapp -c sidecar  # Specific container

# Execute command in pod
kubectl exec myapp -- ls /app
kubectl exec -it myapp -- /bin/sh  # Interactive shell

# Copy files
kubectl cp myapp:/app/config.json ./config.json
```

---

### **Resource Inspection:**

```bash
# Deployments
kubectl get deployments
kubectl describe deployment myapp
kubectl rollout status deployment/myapp
kubectl rollout history deployment/myapp

# Services
kubectl get services
kubectl describe service myservice
kubectl get endpoints myservice

# Nodes
kubectl get nodes
kubectl describe node node-1
kubectl top nodes  # Resource usage
```

---

### **Events:**

```bash
# All events
kubectl get events --sort-by='.lastTimestamp'

# Events for specific resource
kubectl get events --field-selector involvedObject.name=myapp

# Watch events
kubectl get events --watch
```

---

### **Resource Usage:**

```bash
# Requires metrics-server
kubectl top nodes
kubectl top pods
kubectl top pods --containers
kubectl top pods -l app=myapp  # Specific label
```

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

