# Kubernetes Nodes - Complete Theoretical Guide

**Understanding Worker Nodes: The Workhorses of Kubernetes**

---

## 📚 Table of Contents

1. [What is a Node?](#what-is-node)
2. [Node Components](#node-components)
3. [Node Conditions](#node-conditions)
4. [Node Capacity vs Allocatable](#capacity-allocatable)
5. [Node Management](#node-management)
6. [Node Troubleshooting](#troubleshooting)

---

## 🖥️ What is a Node?

A **Node** is a worker machine (VM or physical) in Kubernetes where your applications (Pods) actually run.

### **Simple Analogy:**

```
Kubernetes Cluster = Factory
Nodes = Assembly Lines
Pods = Products being built

Each assembly line (Node):
- Has workers (kubelet, kube-proxy)
- Has tools (container runtime)
- Produces products (runs Pods)
- Reports status to management (Control Plane)
```

---

### **Node Types:**

**1. Control Plane Nodes (Master Nodes):**
```
Run Control Plane components:
- kube-apiserver
- etcd
- kube-scheduler
- kube-controller-manager

Usually NOT used for application workloads
```

**2. Worker Nodes:**
```
Run your applications:
- Pods with containers
- Your microservices
- Your applications

This is where actual work happens
```

---

## 🔧 Node Components

Every worker node runs these components:

```
┌────────────────────────────────────────┐
│         Worker Node                     │
├────────────────────────────────────────┤
│  ┌──────────────────────────────────┐ │
│  │  kubelet                         │ │
│  │  - Registers node with cluster   │ │
│  │  - Manages pods on this node     │ │
│  │  - Reports status                │ │
│  └──────────────────────────────────┘ │
│                                        │
│  ┌──────────────────────────────────┐ │
│  │  kube-proxy                      │ │
│  │  - Network proxy                 │ │
│  │  - Routes traffic to pods        │ │
│  │  - Load balances                 │ │
│  └──────────────────────────────────┘ │
│                                        │
│  ┌──────────────────────────────────┐ │
│  │  Container Runtime               │ │
│  │  - Docker / containerd / CRI-O   │ │
│  │  - Runs containers               │ │
│  └──────────────────────────────────┘ │
│                                        │
│  ┌──────────────────────────────────┐ │
│  │  Pods (Your Applications)        │ │
│  │  - nginx pod                     │ │
│  │  - database pod                  │ │
│  │  - app pod                       │ │
│  └──────────────────────────────────┘ │
└────────────────────────────────────────┘
```

---

### **1. kubelet (Covered in detail in kubectl-kubelet guide)**

**Role:** Node agent that manages pods

**Key Functions:**
- Registers node with API Server
- Watches for pods assigned to this node
- Pulls container images
- Starts/stops containers
- Monitors container health
- Reports pod/node status

---

### **2. kube-proxy**

**Role:** Network proxy for Services

**What it does:**
```
Service: myapp (ClusterIP: 10.96.0.10)
Backend Pods:
  - Pod1: 10.244.1.5
  - Pod2: 10.244.2.8
  - Pod3: 10.244.3.2

kube-proxy configures:
"Traffic to 10.96.0.10 → Load balance to Pod1, Pod2, Pod3"

Uses: iptables or IPVS rules on each node
```

**Example flow:**
```
Application on Node A:
  curl http://myapp:80
        ↓
  kube-proxy on Node A:
    "myapp resolves to 10.96.0.10"
    "10.96.0.10 routes to Pod2 (10.244.2.8)"
        ↓
  Traffic sent to Pod2 on Node B
        ↓
  Pod2 responds
```

---

### **3. Container Runtime**

**Role:** Runs containers

**Options:**
- **containerd** (Most common, Docker's runtime)
- **CRI-O** (Lightweight, designed for Kubernetes)
- **Docker** (Deprecated in Kubernetes 1.24+)

**What it does:**
```
kubelet: "Run container with image nginx:1.21"
    ↓
Container Runtime:
  1. Pull image from registry
  2. Create container
  3. Start container process
  4. Monitor container
  5. Report status to kubelet
```

---

## 📊 Node Conditions

Nodes report various conditions to indicate health.

### **Node Conditions Explained:**

```yaml
# kubectl describe node worker-1
Conditions:
  Type                 Status  Reason                  Message
  ----                 ------  ------                  -------
  MemoryPressure       False   KubeletHasSufficientMemory
  DiskPressure         False   KubeletHasNoDiskPressure
  PIDPressure          False   KubeletHasSufficientPID
  Ready                True    KubeletReady
```

---

### **1. Ready Condition:**

**Status: True** = Node is healthy and ready to accept pods
**Status: False** = Node has problems, cannot accept pods
**Status: Unknown** = No heartbeat from kubelet (node might be down)

**What makes a node Ready:**
```
✓ kubelet is running
✓ Container runtime is working
✓ Sufficient disk space
✓ Sufficient memory
✓ Network is functioning
✓ Can communicate with API Server
```

**Timeline when node crashes:**
```
T+0s:  Node crashes, stops sending heartbeats
T+40s: Node Controller marks node as "Unknown"
T+5m:  Node Controller evicts all pods from node
T+6m:  Pods rescheduled on healthy nodes
```

---

### **2. MemoryPressure:**

**Status: True** = Node is running out of memory
**Status: False** = Node has sufficient memory

**What happens when True:**
```
1. Node stops accepting new pods
2. Kubelet may evict pods to free memory
3. Eviction order:
   - BestEffort pods first (no resource requests)
   - Burstable pods next (requests < limits)
   - Guaranteed pods last (requests = limits)
```

**Example:**
```
Node: 8GB RAM
Used: 7.5GB
Threshold: 90% (7.2GB)

MemoryPressure: True
Action: Evict lowest priority pods
```

---

### **3. DiskPressure:**

**Status: True** = Node is running out of disk space
**Status: False** = Node has sufficient disk

**What happens when True:**
```
1. Node stops accepting new pods
2. Kubelet evicts pods
3. Garbage collects unused images
4. Removes terminated containers
```

---

### **4. PIDPressure:**

**Status: True** = Too many processes running
**Status: False** = Sufficient PIDs available

**Cause:**
```
Linux has max PIDs limit (default ~32k)
If node runs too many containers/processes:
  PIDPressure: True
  Can't start new processes
```

---

## 💾 Node Capacity vs Allocatable

Understanding available resources on a node.

### **Capacity vs Allocatable:**

```
Node with 8GB RAM, 4 CPUs:

Capacity: Total hardware
  - Memory: 8GB
  - CPU: 4 cores

Reserved for System:
  - kubelet: 200MB RAM, 100m CPU
  - System daemons: 300MB RAM, 100m CPU
  - Eviction threshold: 500MB RAM

Allocatable: Available for Pods
  - Memory: 7GB  (8GB - 1GB reserved)
  - CPU: 3.8 cores  (4 - 0.2 reserved)
```

---

### **Resource Calculation Example:**

```yaml
# kubectl describe node worker-1

Capacity:
  cpu:                4
  memory:             8147356Ki  # ~8GB
  pods:               110

Allocatable:
  cpu:                3800m       # 3.8 cores
  memory:             7093612Ki   # ~7GB
  pods:               110

Allocated resources:
  Resource           Requests      Limits
  --------           --------      ------
  cpu                2000m (53%)   3000m (79%)
  memory             3Gi (43%)     5Gi (72%)
```

**What this means:**
```
Pods are using:
  - CPU Requests: 2 cores (53% of allocatable)
  - CPU Limits: 3 cores (can burst up to this)
  - Memory Requests: 3GB (43% of allocatable)
  - Memory Limits: 5GB (can use up to this)

Remaining capacity:
  - CPU: 1.8 cores available for new pods
  - Memory: 4GB available for new pods
```

---

## 🔧 Node Management

### **Viewing Nodes:**

```bash
# List all nodes
kubectl get nodes
NAME         STATUS   ROLES           AGE   VERSION
master-1     Ready    control-plane   50d   v1.28.0
worker-1     Ready    <none>          50d   v1.28.0
worker-2     Ready    <none>          50d   v1.28.0
worker-3     NotReady <none>          50d   v1.28.0

# Detailed node information
kubectl describe node worker-1

# Node resource usage (requires metrics-server)
kubectl top node
NAME       CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
master-1   250m         6%     1500Mi          19%
worker-1   1200m        30%    4Gi             57%
worker-2   800m         20%    3Gi             42%
```

---

### **Cordoning a Node (Prevent new pods):**

```bash
# Mark node as unschedulable
kubectl cordon worker-1

# What happens:
# - No new pods will be scheduled on worker-1
# - Existing pods continue running
# - Used during maintenance

# View cordoned nodes
kubectl get nodes
NAME       STATUS                     ROLES    AGE
worker-1   Ready,SchedulingDisabled   <none>   50d   # Cordoned

# Uncordon (allow scheduling again)
kubectl uncordon worker-1
```

**When to use:**
```
✓ Before node maintenance
✓ Before upgrading node
✓ When debugging node issues
✓ When replacing node
```

---

### **Draining a Node (Evict all pods):**

```bash
# Drain node (evict all pods)
kubectl drain worker-1 --ignore-daemonsets --delete-emptydir-data

# What happens:
# 1. Node is cordoned (no new pods)
# 2. All pods evicted (gracefully deleted)
# 3. Pods rescheduled on other nodes
# 4. Node is ready for maintenance

# After maintenance
kubectl uncordon worker-1
```

**Drain options:**
```bash
--ignore-daemonsets     # Don't evict DaemonSet pods
--delete-emptydir-data  # Delete pods with emptyDir volumes
--force                 # Force delete pods not managed by controllers
--grace-period=60       # Wait 60s before force kill
--timeout=5m            # Timeout for drain operation
```

---

### **Node Maintenance Example:**

```bash
# Scenario: Upgrade worker-1 OS

# Step 1: Cordon node
kubectl cordon worker-1
# New pods won't be scheduled here

# Step 2: Drain node
kubectl drain worker-1 --ignore-daemonsets --delete-emptydir-data
# Pods moved to other nodes

# Step 3: Perform maintenance
ssh worker-1
sudo apt update && sudo apt upgrade -y
sudo reboot

# Step 4: Wait for node to come back
# kubelet automatically registers node

# Step 5: Uncordon node
kubectl uncordon worker-1
# Node can accept pods again

# Verify
kubectl get nodes
NAME       STATUS   ROLES    AGE
worker-1   Ready    <none>   50d   # Ready again!
```

---

### **Removing a Node:**

```bash
# Drain first
kubectl drain worker-1 --ignore-daemonsets --force --delete-emptydir-data

# Remove from cluster
kubectl delete node worker-1

# On the node itself, stop kubelet
ssh worker-1
sudo systemctl stop kubelet
sudo systemctl disable kubelet

# Clean up (optional)
sudo kubeadm reset
```

---

## 🛠️ Node Troubleshooting

### **Issue 1: Node NotReady**

**Symptoms:**
```bash
kubectl get nodes
NAME       STATUS     ROLES    AGE
worker-1   NotReady   <none>   50d
```

**Check:**
```bash
# 1. Check node conditions
kubectl describe node worker-1 | grep Conditions -A 10

# 2. Check kubelet logs (on the node)
ssh worker-1
sudo journalctl -u kubelet -f

# Common causes:
# - kubelet not running
# - Network issues
# - Disk full
# - Out of memory
# - Container runtime issues
```

**Fix:**
```bash
# Restart kubelet
sudo systemctl restart kubelet

# Check status
sudo systemctl status kubelet

# Check disk space
df -h

# Check memory
free -h
```

---

### **Issue 2: Pods Not Scheduling on Node**

**Symptoms:**
```bash
kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
myapp     0/1     Pending   0          5m
```

**Check:**
```bash
# Check pod events
kubectl describe pod myapp

# Possible reasons:
# 1. Node is cordoned
kubectl get nodes
NAME       STATUS                     ROLES
worker-1   Ready,SchedulingDisabled   <none>

# 2. Insufficient resources
kubectl describe node worker-1
# Check "Allocated resources"

# 3. Taints on node
kubectl describe node worker-1 | grep Taints
Taints: key=value:NoSchedule

# 4. Node affinity mismatch
# Pod requires label that node doesn't have
```

---

### **Issue 3: High Resource Usage**

**Symptoms:**
```bash
kubectl top node
NAME       CPU    MEMORY
worker-1   95%    92%      # Very high!
```

**Investigate:**
```bash
# Check which pods are using resources
kubectl top pods --all-namespaces --sort-by=memory

# Check pod resource requests/limits
kubectl describe node worker-1

# If memory pressure:
kubectl describe node worker-1 | grep MemoryPressure
MemoryPressure   True   # Node evicting pods!
```

**Solutions:**
- Add more nodes
- Increase node size
- Optimize pod resource requests
- Evict non-critical pods

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

