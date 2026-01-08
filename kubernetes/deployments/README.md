# Kubernetes Deployments - Complete Theoretical Guide

**Understanding Declarative Application Management**

---

## 📚 Table of Contents

1. [What is a Deployment?](#what-is-deployment)
2. [Deployment vs ReplicaSet vs Pod](#deployment-vs-replicaset)
3. [Creating Deployments](#creating-deployments)
4. [Scaling](#scaling)
5. [Rolling Updates](#rolling-updates)
6. [Rollback](#rollback)
7. [Update Strategies](#update-strategies)
8. [Revision History](#revision-history)
9. [Pausing and Resuming](#pausing-resuming)

---

## 🚀 What is a Deployment?

A **Deployment** provides declarative updates for Pods and ReplicaSets.

### **Simple Analogy:**

```
Deployment = Production Manager

You tell manager: "I want 5 workers (pods) running app v2.0"

Manager handles:
✓ Hiring workers (creates pods)
✓ Replacing workers (updates pods)
✓ Maintaining headcount (keeps 5 running)
✓ Gradual transition (rolling updates)
✓ Undo changes (rollback)

You don't manage individual workers
You declare desired state
Manager makes it happen
```

---

### **Why Deployments?**

**❌ Managing Pods Directly:**
```
$ kubectl run app --image=app:1.0 --replicas=3

Problems:
1. Pod crashes? Manual restart needed
2. Update app? Delete and recreate all pods (downtime!)
3. Rollback? No history
4. Scale? Manual command
5. No automation
```

**✅ Using Deployments:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: app
        image: app:1.0

Benefits:
✓ Self-healing (auto-restart crashed pods)
✓ Rolling updates (zero downtime)
✓ Rollback (undo updates)
✓ Scaling (easy declarative)
✓ Revision history
```

---

## 📊 Deployment vs ReplicaSet vs Pod

Understanding the hierarchy.

### **The Hierarchy:**

```
Deployment (Manages)
    ↓
ReplicaSet (Manages)
    ↓
Pods (Run containers)
```

---

### **Detailed Flow:**

```
┌──────────────────────────────────────────────┐
│  Deployment: myapp                            │
│  - Replicas: 3                                │
│  - Image: app:2.0                             │
│  - Strategy: RollingUpdate                    │
└────────────────┬─────────────────────────────┘
                 │ Creates/manages
                 ↓
┌──────────────────────────────────────────────┐
│  ReplicaSet: myapp-5d4f7c8b9d                │
│  - Replicas: 3                                │
│  - Pod template hash: 5d4f7c8b9d             │
└────────────────┬─────────────────────────────┘
                 │ Creates/manages
                 ↓
┌──────────────────────────────────────────────┐
│  Pods:                                        │
│  - myapp-5d4f7c8b9d-abc12                    │
│  - myapp-5d4f7c8b9d-def34                    │
│  - myapp-5d4f7c8b9d-ghi56                    │
└──────────────────────────────────────────────┘

When you update Deployment image:
1. Deployment creates NEW ReplicaSet
2. New ReplicaSet creates new Pods
3. Old Pods gradually terminated
4. Rolling update!
```

---

### **Responsibilities:**

**Pod:**
- Runs containers
- Restarts containers if they crash
- Single instance

**ReplicaSet:**
- Maintains desired number of pod replicas
- Creates pods from template
- Self-healing (recreates deleted pods)

**Deployment:**
- Manages ReplicaSets
- Handles updates (creates new ReplicaSets)
- Rollback capability
- Revision history
- Pausing/resuming

---

## 🏗️ Creating Deployments

### **Basic Deployment:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3  # Desired number of pods
  
  selector:
    matchLabels:
      app: nginx  # Select pods with this label
  
  template:  # Pod template
    metadata:
      labels:
        app: nginx  # Must match selector
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
```

**What happens:**
```
1. Deployment created

2. Deployment creates ReplicaSet:
   nginx-deployment-5d4f7c8b9d

3. ReplicaSet creates 3 pods:
   nginx-deployment-5d4f7c8b9d-abc12
   nginx-deployment-5d4f7c8b9d-def34
   nginx-deployment-5d4f7c8b9d-ghi56

4. Pods start running nginx:1.21

5. Status:
   Deployment: 3/3 replicas available
   ReplicaSet: 3 current, 3 desired
   Pods: 3 running
```

---

### **Comprehensive Deployment Example:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
  namespace: production
  labels:
    app: webapp
    tier: frontend
spec:
  replicas: 5
  
  selector:
    matchLabels:
      app: webapp
  
  template:
    metadata:
      labels:
        app: webapp
        tier: frontend
    spec:
      containers:
      - name: webapp
        image: webapp:2.0
        ports:
        - containerPort: 8080
        
        # Resource limits
        resources:
          requests:
            memory: "128Mi"
            cpu: "250m"
          limits:
            memory: "256Mi"
            cpu: "500m"
        
        # Health probes
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
        
        # Environment variables
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: url
        
        # Volume mounts
        volumeMounts:
        - name: config
          mountPath: /config
      
      volumes:
      - name: config
        configMap:
          name: webapp-config
  
  # Update strategy
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # Max pods above replicas during update
      maxUnavailable: 0  # Max pods unavailable during update
  
  # Revision history limit
  revisionHistoryLimit: 10
  
  # Progress deadline
  progressDeadlineSeconds: 600  # 10 minutes
```

---

## 📈 Scaling

Changing the number of replicas.

### **Manual Scaling:**

**Method 1: kubectl scale:**
```bash
# Scale to 5 replicas
kubectl scale deployment webapp --replicas=5

# Check scaling
kubectl get deployment webapp
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
webapp   5/5     5            5           10m
```

**Method 2: Edit deployment:**
```bash
kubectl edit deployment webapp

# Change:
spec:
  replicas: 10  # Change to 10

# Save and exit
# Deployment automatically scales to 10
```

**Method 3: Patch deployment:**
```bash
kubectl patch deployment webapp -p '{"spec":{"replicas":10}}'
```

**Method 4: Update YAML:**
```bash
# Update replicas in deployment.yaml
spec:
  replicas: 10

# Apply
kubectl apply -f deployment.yaml
```

---

### **Scaling Process:**

**Scaling Up (3 → 5 replicas):**
```
T+0s:  kubectl scale deployment webapp --replicas=5
T+1s:  Deployment updates ReplicaSet desired count to 5
T+2s:  ReplicaSet creates 2 new pods
T+3s:  Pods: Pending (scheduler assigns nodes)
T+5s:  Pods: ContainerCreating (pulling images)
T+15s: Pods: Running
T+16s: Readiness probes succeed
T+17s: Pods: Ready (receiving traffic)
T+18s: Scaling complete: 5/5 ready
```

**Scaling Down (5 → 3 replicas):**
```
T+0s:  kubectl scale deployment webapp --replicas=3
T+1s:  Deployment updates ReplicaSet desired count to 3
T+2s:  ReplicaSet selects 2 pods to terminate
T+3s:  Selected pods: Terminating status
T+4s:  Pods removed from service endpoints (no new traffic)
T+5s:  SIGTERM sent to containers (graceful shutdown)
T+35s: Grace period ends (default 30s)
T+36s: SIGKILL sent if still running
T+37s: Pods deleted
T+38s: Scaling complete: 3/3 ready
```

---

## 🔄 Rolling Updates

Zero-downtime application updates.

### **What are Rolling Updates?**

```
Old version: app:1.0 (3 pods)
New version: app:2.0

Rolling Update:
1. Start 1 new pod (app:2.0)
2. Wait for new pod to be ready
3. Terminate 1 old pod (app:1.0)
4. Repeat until all pods updated

Benefits:
✓ Zero downtime
✓ Gradual rollout
✓ Can rollback if issues
✓ Always minimum pods available
```

---

### **Rolling Update Example:**

**Current state:**
```bash
kubectl get deployments
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
webapp  3/3     3            3           10m

kubectl get pods
NAME                      READY   STATUS    IMAGE
webapp-5d4f7c8b9d-abc12   1/1     Running   webapp:1.0
webapp-5d4f7c8b9d-def34   1/1     Running   webapp:1.0
webapp-5d4f7c8b9d-ghi56   1/1     Running   webapp:1.0
```

**Update image:**
```bash
kubectl set image deployment/webapp webapp=webapp:2.0
```

**Rolling update process:**
```
T+0s:  Image updated to webapp:2.0
       Deployment creates new ReplicaSet: webapp-7f8c9d5b6a

T+1s:  New ReplicaSet starts 1 pod
       Old: 3 pods (webapp:1.0)
       New: 1 pod  (webapp:2.0) - Pending

T+5s:  New pod: ContainerCreating

T+15s: New pod: Running, readiness probe starting

T+20s: New pod: Ready! ✓
       Available: 4 pods (3 old + 1 new)

T+21s: Old ReplicaSet terminates 1 pod
       Old: 2 pods
       New: 1 pod

T+51s: Old pod fully terminated
       Available: 3 pods (2 old + 1 new)

T+52s: New ReplicaSet starts another pod
       (Process repeats...)

T+180s: All pods updated
        Old: 0 pods
        New: 3 pods (all webapp:2.0)
        Rolling update complete! ✓
```

---

### **Watching Rolling Update:**

```bash
# Watch deployment status
kubectl rollout status deployment/webapp

# Output:
Waiting for deployment "webapp" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "webapp" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "webapp" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "webapp" rollout to finish: 2 old replicas are pending termination...
Waiting for deployment "webapp" rollout to finish: 3 out of 3 new replicas have been updated...
deployment "webapp" successfully rolled out
```

**Real-time pod status:**
```bash
kubectl get pods --watch

NAME                        READY   STATUS              AGE
webapp-5d4f7c8b9d-abc12     1/1     Running             10m
webapp-5d4f7c8b9d-def34     1/1     Running             10m
webapp-5d4f7c8b9d-ghi56     1/1     Running             10m
webapp-7f8c9d5b6a-aaa11     0/1     ContainerCreating   5s
webapp-7f8c9d5b6a-aaa11     1/1     Running             15s
webapp-5d4f7c8b9d-abc12     1/1     Terminating         10m
webapp-7f8c9d5b6a-bbb22     0/1     Pending             0s
...
```

---

## ⏪ Rollback

Reverting to a previous version.

### **Why Rollback?**

```
Deployed webapp:2.0
Problems:
- High error rate
- Performance issues
- Bug discovered

Solution:
Rollback to webapp:1.0 (previous working version)
```

---

### **Rollback Commands:**

**Undo last rollout:**
```bash
kubectl rollout undo deployment/webapp

# Output:
deployment.apps/webapp rolled back

# Deployment reverts to previous version
```

**Check rollback status:**
```bash
kubectl rollout status deployment/webapp

# Output:
Waiting for deployment "webapp" rollout to finish: 1 out of 3 new replicas have been updated...
deployment "webapp" successfully rolled out
```

---

### **Rollback to Specific Revision:**

**View rollout history:**
```bash
kubectl rollout history deployment/webapp

REVISION  CHANGE-CAUSE
1         <none>
2         Image updated to webapp:2.0
3         Image updated to webapp:2.1
4         Image updated to webapp:3.0
```

**Rollback to specific revision:**
```bash
# Rollback to revision 2
kubectl rollout undo deployment/webapp --to-revision=2

# Deployment reverts to revision 2 (webapp:2.0)
```

---

### **Rollback Example:**

```bash
# Current state: webapp:3.0 (broken)
kubectl get deployment webapp
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
webapp   2/3     3            2           20m  # Only 2 ready!

# Check history
kubectl rollout history deployment/webapp
REVISION  CHANGE-CAUSE
1         Initial deployment (webapp:1.0)
2         Updated to webapp:2.0
3         Updated to webapp:3.0 (current - broken)

# Rollback to revision 2
kubectl rollout undo deployment/webapp --to-revision=2

# Watch rollback
kubectl rollout status deployment/webapp
# deployment "webapp" successfully rolled out

# Verify
kubectl get deployment webapp
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
webapp   3/3     3            3           21m  # All ready!

kubectl get pods
NAME                      READY   STATUS    IMAGE
webapp-7f8c9d5b6a-aaa11   1/1     Running   webapp:2.0
webapp-7f8c9d5b6a-bbb22   1/1     Running   webapp:2.0
webapp-7f8c9d5b6a-ccc33   1/1     Running   webapp:2.0

# Rollback successful! ✓
```

---

## 🎯 Update Strategies

Controlling how updates are rolled out.

### **1. RollingUpdate (Default):**

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # Max pods above desired count
      maxUnavailable: 0  # Max pods below desired count
```

**Parameters:**

**maxSurge:**
```
Max pods above replicas during update

replicas: 3
maxSurge: 1
Max total pods during update: 4 (3 + 1)

maxSurge: 25% (of replicas)
replicas: 10
maxSurge: 25% = 2.5 → rounds up to 3
Max pods: 13
```

**maxUnavailable:**
```
Max pods unavailable during update

replicas: 3
maxUnavailable: 1
Min available pods during update: 2 (3 - 1)

maxUnavailable: 0
All replicas must be available during update
(Conservative, slower updates)
```

---

### **Update Strategy Examples:**

**Conservative (Zero downtime, slower):**
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1        # Add 1 new pod at a time
    maxUnavailable: 0  # Always keep all replicas available
```

**Process:**
```
replicas: 3

Step 1: 3 old + 1 new = 4 total (maxSurge: 1)
Step 2: Wait for new pod ready
Step 3: Terminate 1 old → 2 old + 1 new = 3 total
Step 4: Repeat
```

---

**Aggressive (Faster, brief capacity reduction):**
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 1  # Allow 1 pod unavailable
```

**Process:**
```
replicas: 3

Step 1: Terminate 1 old → 2 old running (maxUnavailable: 1)
Step 2: Start 1 new → 2 old + 1 new
Step 3: Wait for new pod ready
Step 4: Repeat
```

---

**Blue-Green style (All at once):**
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 100%     # Double the pods
    maxUnavailable: 0
```

**Process:**
```
replicas: 3

Step 1: Start 3 new pods → 3 old + 3 new = 6 total
Step 2: Wait for all new pods ready
Step 3: Terminate all old pods → 3 new
```

---

### **2. Recreate Strategy:**

```yaml
spec:
  strategy:
    type: Recreate  # Terminate all, then create new
```

**Process:**
```
1. Terminate all old pods
2. Wait for all to be deleted
3. Create all new pods

Downtime: YES (all pods terminated first)

Use case:
- Cannot run multiple versions simultaneously
- Database schema changes
- Shared state issues
```

---

## 📜 Revision History

Tracking deployment changes.

### **Viewing History:**

```bash
kubectl rollout history deployment/webapp

REVISION  CHANGE-CAUSE
1         kubectl apply --filename=deployment.yaml --record
2         kubectl set image deployment/webapp webapp=webapp:2.0 --record
3         kubectl set image deployment/webapp webapp=webapp:2.1 --record
```

**Detailed revision info:**
```bash
kubectl rollout history deployment/webapp --revision=2

deployment.apps/webapp with revision #2
Pod Template:
  Labels: app=webapp
          pod-template-hash=7f8c9d5b6a
  Containers:
   webapp:
    Image: webapp:2.0
    Port: 8080/TCP
    ...
```

---

### **Recording Changes:**

```bash
# Add --record flag to save command in history
kubectl set image deployment/webapp webapp=webapp:2.0 --record

# Or use kubernetes.io/change-cause annotation
kubectl annotate deployment/webapp kubernetes.io/change-cause="Updated to version 2.0 with bug fixes"
```

---

### **Revision History Limit:**

```yaml
spec:
  revisionHistoryLimit: 10  # Keep last 10 ReplicaSets
  
# Old ReplicaSets beyond limit are garbage collected
# Can't rollback beyond this limit
```

---

## ⏸️ Pausing and Resuming

Pausing rollouts for staged updates.

### **Why Pause?**

```
Use case: Multiple changes

Without pause:
1. Update image → Rolling update starts
2. Update resources → Another rolling update
3. Update env vars → Yet another rolling update
Total: 3 rolling updates!

With pause:
1. Pause deployment
2. Update image
3. Update resources
4. Update env vars
5. Resume deployment
Total: 1 rolling update!
```

---

### **Pausing Deployment:**

```bash
# Pause deployment
kubectl rollout pause deployment/webapp

# Make multiple changes
kubectl set image deployment/webapp webapp=webapp:3.0
kubectl set resources deployment/webapp -c=webapp --limits=cpu=500m,memory=512Mi
kubectl set env deployment/webapp API_URL=https://api.example.com

# No rollouts triggered yet!

# Resume deployment (single rollout with all changes)
kubectl rollout resume deployment/webapp

# One rolling update with all changes applied
```

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

