# Kubernetes Taints & Tolerations - Complete Theoretical Guide

**Understanding Pod Placement Control**

---

## 📚 Table of Contents

1. [What are Taints?](#what-are-taints)
2. [What are Tolerations?](#what-are-tolerations)
3. [Taint Effects](#taint-effects)
4. [Creating Taints](#creating-taints)
5. [Creating Tolerations](#creating-tolerations)
6. [Common Use Cases](#use-cases)
7. [Node Affinity vs Taints](#affinity-vs-taints)

---

## 🚫 What are Taints?

**Taints** allow a node to repel a set of pods.

### **Simple Analogy:**

```
Taint = "No Entry" Sign on Building

Building (Node) with sign (Taint):
  "No entry for general public"

General public (Pods):
  Cannot enter ✗

People with special pass (Toleration):
  Can enter ✓

Taint = Restriction on node
Toleration = Permission to bypass restriction
```

---

### **How Taints Work:**

```
Node without taint:
  Any pod can be scheduled ✓

Node with taint:
  Only pods with matching toleration can be scheduled
  Pods without toleration: Rejected ✗
```

---

## ✅ What are Tolerations?

**Tolerations** allow pods to schedule onto nodes with matching taints.

### **Relationship:**

```
Taint on node:
  key=gpu, value=true, effect=NoSchedule

Pod without toleration:
  Cannot schedule on this node ✗

Pod with matching toleration:
  Can schedule on this node ✓

Think of it as:
  Taint = Lock
  Toleration = Key
```

---

## ⚡ Taint Effects

Three types of taint effects.

### **1. NoSchedule:**

```
Effect: NoSchedule

Pods without matching toleration:
  Will NOT be scheduled on node

Existing pods (before taint applied):
  Continue running (not evicted)

Use case:
  Prevent new pods from scheduling
  Don't affect existing pods
```

**Example:**
```
T+0s:   Node has 5 pods running
T+1s:   Add taint: gpu=true:NoSchedule
T+2s:   Existing 5 pods still running ✓
T+3s:   Try to schedule new pod (no toleration)
T+4s:   Scheduler rejects: Node tainted ✗
```

---

### **2. PreferNoSchedule:**

```
Effect: PreferNoSchedule

Soft preference (not enforced)

Pods without toleration:
  Try to avoid this node
  But can schedule if no other option

Use case:
  Prefer not to schedule
  But allow if necessary
```

**Example:**
```
3 nodes:
  node-1: No taint
  node-2: No taint
  node-3: Taint (PreferNoSchedule)

Schedule 10 pods (no toleration):
  Result:
    node-1: 5 pods
    node-2: 5 pods
    node-3: 0 pods (avoided)

Scale to 30 pods:
  Result:
    node-1: 10 pods (full)
    node-2: 10 pods (full)
    node-3: 10 pods (had to use it)
    
Prefers not to use node-3, but will if needed
```

---

### **3. NoExecute:**

```
Effect: NoExecute

Strongest effect!

Pods without matching toleration:
  Will NOT be scheduled (like NoSchedule)
  EXISTING pods EVICTED (removed)

Use case:
  Evacuate node
  Immediate enforcement
  Maintenance
```

**Example:**
```
T+0s:   Node has 5 pods running
T+1s:   Add taint: maintenance=true:NoExecute
T+2s:   Check all pods for toleration:
        - pod-1: No toleration → Evict ✗
        - pod-2: No toleration → Evict ✗
        - pod-3: Has toleration → Keep ✓
        - pod-4: No toleration → Evict ✗
        - pod-5: No toleration → Evict ✗
T+3s:   4 pods evicted and rescheduled elsewhere
        1 pod remains (had toleration)
```

---

## 🏗️ Creating Taints

### **Taint Syntax:**

```bash
kubectl taint nodes <node-name> <key>=<value>:<effect>
```

---

### **Examples:**

**1. Taint for GPU nodes:**
```bash
kubectl taint nodes gpu-node-1 gpu=true:NoSchedule

# Result:
# Only pods with toleration for "gpu=true" can schedule
```

**2. Taint for dedicated nodes:**
```bash
kubectl taint nodes dedicated-1 dedicated=team-platform:NoSchedule

# Only team-platform pods (with toleration) can use this node
```

**3. Taint for maintenance:**
```bash
kubectl taint nodes worker-1 maintenance=true:NoExecute

# Evict all pods without toleration immediately
```

---

### **Viewing Taints:**

```bash
# Describe node to see taints
kubectl describe node gpu-node-1

Name:               gpu-node-1
Taints:             gpu=true:NoSchedule
...

# No taints:
Taints:             <none>
```

---

### **Removing Taints:**

```bash
# Remove taint (add minus sign at end)
kubectl taint nodes gpu-node-1 gpu=true:NoSchedule-

# Remove all taints with key "gpu"
kubectl taint nodes gpu-node-1 gpu-
```

---

## 🎟️ Creating Tolerations

Tolerations are specified in pod spec.

### **Toleration Syntax:**

```yaml
spec:
  tolerations:
  - key: "<key>"
    operator: "Equal"  # or "Exists"
    value: "<value>"
    effect: "<effect>"  # NoSchedule, PreferNoSchedule, NoExecute
```

---

### **Example 1: GPU Toleration:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod
spec:
  tolerations:
  - key: "gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
  containers:
  - name: tensorflow
    image: tensorflow/tensorflow:latest-gpu
```

**Matching:**
```
Node taint: gpu=true:NoSchedule
Pod toleration: gpu=true:NoSchedule

Match! ✓ Pod can schedule on node
```

---

### **Example 2: Tolerate Any Value:**

```yaml
spec:
  tolerations:
  - key: "dedicated"
    operator: "Exists"  # Tolerate any value
    effect: "NoSchedule"
```

**Matches:**
```
dedicated=team-a:NoSchedule    ✓
dedicated=team-b:NoSchedule    ✓
dedicated=anything:NoSchedule  ✓

Key "dedicated" with any value
```

---

### **Example 3: Tolerate All Taints:**

```yaml
spec:
  tolerations:
  - operator: "Exists"  # Tolerate everything
```

**Matches:**
```
Any taint on any node ✓

Use case: DaemonSets (must run on every node)
```

---

### **Example 4: Toleration with Timeout (NoExecute):**

```yaml
spec:
  tolerations:
  - key: "maintenance"
    operator: "Equal"
    value: "true"
    effect: "NoExecute"
    tolerationSeconds: 3600  # Stay for 1 hour, then evict
```

**Behavior:**
```
Node gets taint: maintenance=true:NoExecute

Pod with toleration (tolerationSeconds: 3600):
  T+0s:   Taint applied
  T+0s:   Pod tolerates, keeps running
  T+3600s: Timeout! Pod evicted

Pod without toleration:
  T+0s:   Taint applied
  T+1s:   Pod evicted immediately

Use case: Graceful node shutdown
```

---

## 💼 Common Use Cases

### **1. Dedicated Nodes for Team:**

```bash
# Taint nodes for team-platform
kubectl taint nodes node-1 node-2 node-3 dedicated=team-platform:NoSchedule
```

```yaml
# team-platform pods
apiVersion: apps/v1
kind: Deployment
metadata:
  name: platform-app
spec:
  template:
    spec:
      tolerations:
      - key: "dedicated"
        operator: "Equal"
        value: "team-platform"
        effect: "NoSchedule"
      containers:
      - name: app
        image: platform-app:1.0
```

**Result:**
```
team-platform pods → Only on node-1, node-2, node-3
Other teams' pods → Cannot use those nodes
Dedicated resources for team!
```

---

### **2. GPU Nodes:**

```bash
# Taint GPU nodes
kubectl taint nodes gpu-1 gpu-2 gpu=true:NoSchedule
```

```yaml
# ML training pod
apiVersion: v1
kind: Pod
metadata:
  name: ml-training
spec:
  tolerations:
  - key: "gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
  containers:
  - name: trainer
    image: ml-trainer:1.0
    resources:
      limits:
        nvidia.com/gpu: 1
```

**Result:**
```
GPU pods → Can schedule on GPU nodes
Regular pods → Cannot use expensive GPU nodes
Efficient GPU utilization!
```

---

### **3. Node Maintenance:**

```bash
# Evacuate node for maintenance
kubectl taint nodes worker-5 maintenance=true:NoExecute

# What happens:
# - All pods without toleration evicted
# - Pods rescheduled on other nodes
# - Node ready for maintenance
# - No new pods scheduled

# After maintenance:
kubectl taint nodes worker-5 maintenance=true:NoExecute-

# Node back in rotation
```

---

### **4. Control Plane Nodes:**

```
Control plane nodes have default taint:
  node-role.kubernetes.io/control-plane:NoSchedule

Regular pods: Cannot schedule on control plane ✓
System pods (kube-proxy, etc.): Have toleration ✓

Prevents user workloads from consuming control plane resources
```

**Override (not recommended):**
```yaml
spec:
  tolerations:
  - key: "node-role.kubernetes.io/control-plane"
    operator: "Exists"
    effect: "NoSchedule"
```

---

### **5. Spot/Preemptible Nodes:**

```bash
# Taint spot instances
kubectl taint nodes spot-1 spot-2 instance-type=spot:NoSchedule
```

```yaml
# Fault-tolerant batch job
apiVersion: batch/v1
kind: Job
metadata:
  name: batch-job
spec:
  template:
    spec:
      tolerations:
      - key: "instance-type"
        operator: "Equal"
        value: "spot"
        effect: "NoSchedule"
      containers:
      - name: processor
        image: batch-processor:1.0
```

**Result:**
```
Batch jobs → Can use cheap spot instances
Critical services → Stay on reliable nodes
Cost optimization!
```

---

## 🆚 Node Affinity vs Taints

### **Node Affinity:**

```yaml
# Pod says: "I WANT to run on GPU nodes"
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: gpu
            operator: In
            values:
            - "true"

Pod perspective: "I choose nodes"
```

---

### **Taints & Tolerations:**

```yaml
# Node says: "Only GPU pods allowed"
# Pod says: "I tolerate GPU nodes"

Node taint: gpu=true:NoSchedule

Pod toleration:
spec:
  tolerations:
  - key: "gpu"
    value: "true"
    effect: "NoSchedule"

Node perspective: "I choose pods"
```

---

### **Combining Both:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gpu-app
spec:
  template:
    spec:
      # Toleration: Allows scheduling on GPU nodes
      tolerations:
      - key: "gpu"
        operator: "Equal"
        value: "true"
        effect: "NoSchedule"
      
      # Node Affinity: Requires GPU nodes
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: gpu
                operator: In
                values:
                - "true"
      
      containers:
      - name: app
        image: gpu-app:1.0
```

**Result:**
```
Toleration alone:
  Pod CAN run on GPU nodes
  But might run on non-GPU nodes too

Affinity alone:
  Pod WANTS to run on GPU nodes
  But GPU nodes might have taints (rejected!)

Both together:
  Pod MUST run on GPU nodes (affinity)
  Pod CAN run on GPU nodes (toleration)
  Guaranteed GPU node placement! ✓
```

---

### **Key Differences:**

```
Taints & Tolerations:
  - Node decides who can come
  - Repels pods by default
  - Pod needs permission (toleration)
  - Doesn't guarantee placement

Node Affinity:
  - Pod decides where to go
  - Attracts pod to nodes
  - Pod chooses nodes
  - Can guarantee placement

Use together:
  - Taint: Keep non-GPU pods away
  - Affinity: Ensure GPU pods go to GPU nodes
  - Best of both worlds!
```

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

