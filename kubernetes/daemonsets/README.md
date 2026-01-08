# Kubernetes DaemonSets - Complete Theoretical Guide

**Understanding Node-Level Services**

---

## 📚 Table of Contents

1. [What is a DaemonSet?](#what-is-daemonset)
2. [How DaemonSets Work](#how-work)
3. [Common Use Cases](#use-cases)
4. [Creating DaemonSets](#creating)
5. [Node Selection](#node-selection)
6. [Updates](#updates)
7. [DaemonSet vs Deployment](#vs-deployment)

---

## 🎯 What is a DaemonSet?

A **DaemonSet** ensures that all (or some) nodes run a copy of a pod.

### **Simple Analogy:**

```
DaemonSet = Security Guard at Every Building Floor

Company building has 10 floors
Need security guard on EVERY floor

DaemonSet ensures:
✓ One guard per floor
✓ New floor added? Guard assigned automatically
✓ Floor removed? Guard removed
✓ Guard leaves? Replacement immediately assigned

Floor = Node
Guard = Pod
```

---

### **Core Concept:**

```
Regular Deployment:
  "Run 5 replicas anywhere in cluster"
  Pods distributed across nodes
  Node A: 2 pods
  Node B: 3 pods
  Node C: 0 pods

DaemonSet:
  "Run 1 pod on EVERY node"
  Each node gets exactly 1 pod
  Node A: 1 pod
  Node B: 1 pod
  Node C: 1 pod
  New Node D added? → 1 pod automatically
```

---

## ⚙️ How DaemonSets Work

### **Automatic Pod Distribution:**

```
Cluster: 3 nodes

DaemonSet: fluentd (log collector)

┌─────────────────────┐
│  Node 1              │
│  Pod: fluentd-abc    │ ← DaemonSet creates
└─────────────────────┘

┌─────────────────────┐
│  Node 2              │
│  Pod: fluentd-def    │ ← DaemonSet creates
└─────────────────────┘

┌─────────────────────┐
│  Node 3              │
│  Pod: fluentd-ghi    │ ← DaemonSet creates
└─────────────────────┘

New node added:
┌─────────────────────┐
│  Node 4              │
│  Pod: fluentd-jkl    │ ← Auto-created!
└─────────────────────┘

Node 2 removed:
  fluentd-def deleted automatically
```

---

### **DaemonSet Controller:**

```
┌───────────────────────────────────────┐
│  DaemonSet Controller                  │
│  (Runs in kube-controller-manager)    │
└─────────────────┬─────────────────────┘
                  │
                  ↓
    ┌─────────────────────────────┐
    │  Watch nodes in cluster     │
    └─────────────┬───────────────┘
                  │
                  ↓
    ┌─────────────────────────────┐
    │  For each node:             │
    │  - Check if pod exists      │
    │  - Missing? Create pod      │
    │  - Extra? Delete pod        │
    └─────────────────────────────┘

Continuous reconciliation loop
```

---

## 💼 Common Use Cases

### **1. Log Collection:**

```
Every node needs log collector

fluentd DaemonSet:
  - Runs on every node
  - Collects logs from all pods on node
  - Forwards to centralized logging (Elasticsearch)

┌──────────────────────────────────────┐
│  Node                                 │
│  ┌─────────────────────────────────┐ │
│  │  fluentd (DaemonSet pod)        │ │
│  │  Reads: /var/log/containers/*   │ │
│  │  Forwards to: Elasticsearch     │ │
│  └─────────────────────────────────┘ │
│                                      │
│  ┌──────────┐  ┌──────────┐        │
│  │ Pod logs │  │ Pod logs │        │
│  └──────────┘  └──────────┘        │
└──────────────────────────────────────┘
```

---

### **2. Monitoring:**

```
Node exporter DaemonSet:
  - Monitors node metrics (CPU, memory, disk, network)
  - Exposes metrics for Prometheus
  - One per node for complete cluster visibility

node-exporter on each node:
  - Collects hardware & OS metrics
  - Prometheus scrapes from all nodes
```

---

### **3. Networking:**

```
CNI plugin DaemonSet (Calico, Cilium, Weave):
  - Provides networking for pods
  - Must run on every node
  - Sets up network interfaces

Kube-proxy DaemonSet:
  - Maintains network rules
  - Enables Service networking
  - One per node
```

---

### **4. Storage:**

```
CSI driver DaemonSet:
  - Storage interface plugin
  - Must be on every node
  - Mounts volumes to pods

Rook Ceph agent DaemonSet:
  - Distributed storage
  - Runs on nodes providing storage
```

---

### **5. Security:**

```
Security agents:
  - Falco (security monitoring)
  - Aqua Security
  - Twistlock

Run on every node to monitor:
  - System calls
  - File access
  - Network activity
  - Container behavior
```

---

## 🏗️ Creating DaemonSets

### **Basic DaemonSet:**

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-system
  labels:
    app: fluentd
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluentd:v1.14
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      
      # Run on host network for performance
      hostNetwork: true
      # Access host IPC
      hostIPC: true
      # Access host PID namespace
      hostPID: true
      
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

---

### **What Happens:**

```
kubectl apply -f daemon set.yaml

T+0s:   DaemonSet created

T+1s:   DaemonSet controller evaluates nodes:
        Node 1: No fluentd pod → Create
        Node 2: No fluentd pod → Create
        Node 3: No fluentd pod → Create

T+2s:   3 pods created:
        fluentd-abc12 → Node 1
        fluentd-def34 → Node 2
        fluentd-ghi56 → Node 3

T+15s:  All pods running

Result:
  1 pod per node
  Total: 3 pods (for 3 nodes)
```

---

### **Viewing DaemonSets:**

```bash
# List DaemonSets
kubectl get daemonsets -n kube-system
kubectl get ds -n kube-system  # Short form

NAME      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
fluentd   3         3         3       3            3           <none>          5m
kube-proxy 3        3         3       3            3           <none>          30d

# DESIRED: Number of nodes that should run pod
# CURRENT: Number of pods running
# READY: Number of pods ready

# Detailed information
kubectl describe daemonset fluentd -n kube-system

Name:           fluentd
Selector:       app=fluentd
Node-Selector:  <none>
Labels:         app=fluentd
Desired Number of Nodes Scheduled: 3
Current Number of Nodes Scheduled: 3
Number of Nodes Scheduled with Ready Pods: 3
...
```

---

## 🎯 Node Selection

Control which nodes run DaemonSet pods.

### **1. Run on ALL Nodes (Default):**

```yaml
spec:
  template:
    spec:
      containers:
      - name: agent
        image: monitoring-agent:latest
  # No nodeSelector or affinity
  # Runs on every node
```

---

### **2. nodeSelector (Simple):**

```yaml
spec:
  template:
    spec:
      nodeSelector:
        disktype: ssd  # Only on nodes with this label
      containers:
      - name: app
        image: myapp:latest
```

**Example:**
```bash
# Label nodes
kubectl label nodes node-1 disktype=ssd
kubectl label nodes node-2 disktype=ssd

# DaemonSet pods only on node-1 and node-2
# node-3 (no label) won't get pod
```

---

### **3. Node Affinity (Advanced):**

```yaml
spec:
  template:
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/os
                operator: In
                values:
                - linux
      containers:
      - name: agent
        image: linux-agent:latest
```

**Use case:**
```
Run DaemonSet only on Linux nodes
(Skip Windows nodes in mixed cluster)
```

---

### **4. Tolerations (Control Plane Nodes):**

```yaml
spec:
  template:
    spec:
      tolerations:
      # Allow scheduling on control plane nodes
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      containers:
      - name: monitoring
        image: node-exporter:latest
```

**Without toleration:**
```
Control plane nodes have taint:
  node-role.kubernetes.io/control-plane:NoSchedule

DaemonSet pods: Don't schedule there
Only on worker nodes
```

**With toleration:**
```
DaemonSet tolerates control plane taint
Pods scheduled on control plane + workers
Monitor entire cluster!
```

---

## 🔄 Updates

### **Update Strategies:**

**1. RollingUpdate (Default):**
```yaml
spec:
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1  # Max pods unavailable during update
```

**Update process:**
```
DaemonSet: fluentd (3 nodes)
Update: fluentd:v1.14 → fluentd:v1.15

maxUnavailable: 1

T+0s:   Delete fluentd pod on Node 1
T+30s:  Pod terminated
T+31s:  Create new pod on Node 1 (fluentd:v1.15)
T+45s:  New pod Ready ✓

T+46s:  Delete fluentd pod on Node 2
T+76s:  Pod terminated
T+77s:  Create new pod on Node 2 (fluentd:v1.15)
T+91s:  New pod Ready ✓

T+92s:  Delete fluentd pod on Node 3
T+122s: Pod terminated
T+123s: Create new pod on Node 3 (fluentd:v1.15)
T+137s: New pod Ready ✓

Update complete!
One node at a time (maxUnavailable: 1)
```

---

**maxUnavailable:**
```yaml
# Conservative (slower, more available)
rollingUpdate:
  maxUnavailable: 1  # Update 1 node at a time

# Aggressive (faster, less available)
rollingUpdate:
  maxUnavailable: 3  # Update up to 3 nodes simultaneously

# Percentage
rollingUpdate:
  maxUnavailable: 25%  # 25% of nodes at once
  # 10 nodes → 2-3 nodes updated simultaneously
```

---

**2. OnDelete:**
```yaml
spec:
  updateStrategy:
    type: OnDelete
```

**What happens:**
```
Update DaemonSet spec (change image)

Pods: No automatic update

Manual per-node update:
  kubectl delete pod fluentd-abc12
  New pod created with new image ✓
  
  kubectl delete pod fluentd-def34
  New pod created with new image ✓

Complete manual control
Useful for:
  - Testing updates on specific nodes
  - Staged rollouts
  - Critical infrastructure
```

---

## 🆚 DaemonSet vs Deployment

```
DaemonSet:
  ✓ One pod per node (or selected nodes)
  ✓ Automatically scales with cluster
  ✓ Node-level services
  ✗ Can't specify replica count
  
Use for:
  - Log collectors
  - Monitoring agents
  - Networking plugins
  - Storage drivers

Deployment:
  ✓ Specific replica count
  ✓ Flexible pod distribution
  ✓ Application workloads
  ✗ Doesn't guarantee node coverage
  
Use for:
  - Web servers
  - APIs
  - Databases (with StatefulSets)
  - Business applications

Example comparison:

10-node cluster:

DaemonSet (monitoring):
  Desired: 10 (one per node)
  Result: 1 pod on each node

Deployment (webapp):
  Replicas: 5
  Result: 5 pods distributed
    Maybe: 2 on Node1, 1 on Node2, 2 on Node3
           0 on other nodes
```

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

