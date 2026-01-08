# Kubernetes Control Plane - Complete Theoretical Guide

**Understanding the Brain of Kubernetes: API Server, etcd, Scheduler, Controller Manager**

---

## 📚 Table of Contents

1. [What is the Control Plane?](#what-is-control-plane)
2. [Control Plane Architecture](#architecture)
3. [kube-apiserver (The Gateway)](#kube-apiserver)
4. [etcd (The Database)](#etcd)
5. [kube-scheduler (The Matchmaker)](#kube-scheduler)
6. [kube-controller-manager (The Autopilot)](#kube-controller-manager)
7. [cloud-controller-manager](#cloud-controller-manager)
8. [How Components Work Together](#working-together)
9. [Real-World Examples](#real-world-examples)

---

## 🎯 What is the Control Plane?

The **Control Plane** is the "brain" of Kubernetes - it makes global decisions about the cluster and detects/responds to cluster events.

### **Simple Analogy:**

```
Kubernetes Cluster = City
Control Plane = City Hall (Government)
Worker Nodes = Buildings where people work

City Hall (Control Plane):
- Decides where to build new buildings (scheduling)
- Keeps records of all buildings (etcd)
- Ensures city rules are followed (controllers)
- Receives all requests from citizens (API server)

Buildings (Worker Nodes):
- Actually house the workers (Pods)
- Report status back to City Hall
```

---

### **Control Plane vs Worker Nodes:**

```
┌─────────────────────────────────────────┐
│         CONTROL PLANE                    │
│  (Makes Decisions, Manages State)       │
│  ┌──────────────────────────────────┐   │
│  │  kube-apiserver                  │   │
│  │  etcd                            │   │
│  │  kube-scheduler                  │   │
│  │  kube-controller-manager         │   │
│  └──────────────────────────────────┘   │
└──────────────┬──────────────────────────┘
               │ (Commands)
               ↓
┌──────────────────────────────────────────┐
│         WORKER NODES                      │
│  (Execute Tasks, Run Pods)               │
│  ┌──────────────────────────────────┐   │
│  │  kubelet                         │   │
│  │  kube-proxy                      │   │
│  │  Container Runtime               │   │
│  │  Pods (Your Applications)        │   │
│  └──────────────────────────────────┘   │
└──────────────────────────────────────────┘
```

---

## 🏗️ Control Plane Architecture

```
┌────────────────────────────────────────────────────────┐
│               KUBERNETES CONTROL PLANE                  │
├────────────────────────────────────────────────────────┤
│                                                         │
│  ┌────────────────────────────────────────────────┐   │
│  │         kube-apiserver (The Gateway)           │   │
│  │  - REST API interface                          │   │
│  │  - Authentication & Authorization              │   │
│  │  - Admission Control                           │   │
│  │  - Validates and processes requests            │   │
│  └────────────┬────────────────┬──────────────────┘   │
│               │                │                       │
│  ┌────────────▼─────┐   ┌─────▼──────────────────┐   │
│  │      etcd        │   │   kube-scheduler       │   │
│  │  (Database)      │   │   (Placement Engine)   │   │
│  │  - Stores state  │   │   - Assigns Pods       │   │
│  │  - Key-value DB  │   │   - Resource matching  │   │
│  └──────────────────┘   └────────────────────────┘   │
│                                                        │
│  ┌────────────────────────────────────────────────┐  │
│  │     kube-controller-manager                    │  │
│  │     (Maintains Desired State)                  │  │
│  │  - Node Controller                            │  │
│  │  - Deployment Controller                      │  │
│  │  - ReplicaSet Controller                      │  │
│  │  - Service Controller                         │  │
│  └────────────────────────────────────────────────┘  │
│                                                        │
└────────────────────────────────────────────────────────┘
```

---

## 🌐 kube-apiserver (The Gateway)

The **API Server** is the front door to Kubernetes. ALL communication goes through it.

### **What is kube-apiserver?**

Think of it as the **receptionist at a hotel**:
- Receives all requests
- Validates credentials
- Routes requests to appropriate departments
- Keeps record of everything

### **Key Responsibilities:**

**1. REST API Interface:**
```
Every Kubernetes operation goes through API:
- kubectl create deployment
- kubectl get pods
- kubectl delete service

All translate to HTTP requests to API server:
POST /apis/apps/v1/namespaces/default/deployments
GET /api/v1/namespaces/default/pods
DELETE /api/v1/namespaces/default/services/myapp
```

**2. Authentication (Who are you?):**
```
Methods:
- Client certificates
- Bearer tokens
- Service account tokens
- Basic auth (deprecated)
- OpenID Connect

Example:
User: "I'm John with certificate"
API Server: *validates certificate* "Yes, you're John"
```

**3. Authorization (What can you do?):**
```
After authentication:
API Server checks: "Can John create pods?"
Uses RBAC rules
Allow/Deny decision

Example:
John has "pod-reader" role
John tries: kubectl delete pod
API Server: "Denied! You can only read pods"
```

**4. Admission Control (Is request valid?):**
```
Final validation before processing:
- Resource quotas: "Does namespace have CPU quota?"
- Pod Security: "Is this pod trying privileged mode?"
- Default values: "No storage class? Use default"

Example validating admission controllers:
- LimitRanger: Enforces resource limits
- ResourceQuota: Enforces quotas
- PodSecurityPolicy: Security policies
```

---

### **API Server Request Flow:**

```
kubectl create deployment nginx --image=nginx
            ↓
1. Authentication
   "Who is making this request?"
   Check: Client certificate ✓
            ↓
2. Authorization
   "Can this user create deployments?"
   Check: RBAC rules ✓
            ↓
3. Admission Control
   "Is this deployment valid?"
   Check: Resource limits, policies ✓
            ↓
4. Validation
   "Is the YAML correct?"
   Check: Schema validation ✓
            ↓
5. Persist to etcd
   "Save deployment spec to database"
   etcd: Deployment saved ✓
            ↓
6. Return Response
   "deployment.apps/nginx created"
```

---

### **API Server in Action - Example:**

**Scenario: Creating a Pod**

```yaml
# User runs:
kubectl apply -f pod.yaml

# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.21
```

**Behind the Scenes:**

```
1. kubectl → API Server
   POST /api/v1/namespaces/default/pods
   Headers: Authorization: Bearer <token>
   Body: <pod YAML>

2. API Server authenticates
   Token valid? ✓

3. API Server authorizes
   Can this user create pods in default namespace? ✓

4. Admission Controllers run
   - NamespaceLifecycle: Namespace exists? ✓
   - LimitRanger: Resource limits OK? ✓
   - ResourceQuota: Namespace quota OK? ✓

5. API Server validates
   - apiVersion correct? ✓
   - Kind exists? ✓
   - Required fields present? ✓

6. API Server persists to etcd
   etcd.put("/registry/pods/default/nginx", pod_data)

7. API Server returns
   Response: pod/nginx created

8. Scheduler notices new pod (watches API)
   "New pod needs scheduling!"

9. Scheduler assigns node
   Updates pod: .spec.nodeName = "worker-node-1"

10. Kubelet on worker-node-1 notices (watches API)
    "I need to run this pod!"

11. Kubelet creates container
    Pulls nginx image, starts container

12. Kubelet reports status
    Updates pod: .status.phase = "Running"
```

---

## 💾 etcd (The Database)

**etcd** is Kubernetes' brain's memory - it stores the entire cluster state.

### **What is etcd?**

Think of it as a **highly reliable filing cabinet**:
- Stores all cluster data
- Consistent and reliable
- Distributed (no single point of failure)

### **Key Characteristics:**

**1. Distributed Key-Value Store:**
```
Key-Value pairs:
/registry/pods/default/nginx → {pod spec and status}
/registry/deployments/default/web → {deployment spec}
/registry/services/default/api → {service spec}

All Kubernetes objects stored as key-value pairs
```

**2. Consistency (Strong):**
```
Write to etcd → Confirmed → Data is safe
Never returns "saved" unless actually saved
Even if server crashes, data persists
```

**3. Watch Mechanism:**
```
Components can "watch" for changes:

Scheduler: "Tell me when new pods appear"
etcd: *Pod created* → "New pod!"
Scheduler: "I'll schedule it"

Real-time notifications of changes
```

---

### **What etcd Stores:**

```
Everything in Kubernetes is stored in etcd:

1. Cluster Configuration
   - Cluster settings
   - Component configs

2. Object Definitions
   - Pods
   - Deployments
   - Services
   - ConfigMaps
   - Secrets (encrypted)

3. Object Status
   - Pod running state
   - Node conditions
   - Resource usage

4. Cluster State
   - Which nodes are alive
   - Which pods are where
   - Service endpoints
```

---

### **etcd Data Structure Example:**

```
/registry/
  ├── pods/
  │   ├── default/
  │   │   ├── nginx-abc123 → {pod spec, status, node assignment}
  │   │   └── web-def456 → {pod data}
  │   └── kube-system/
  │       └── coredns-xyz789 → {pod data}
  │
  ├── deployments/
  │   └── default/
  │       └── nginx-deployment → {replicas: 3, image: nginx}
  │
  ├── services/
  │   └── default/
  │       └── nginx-service → {type: ClusterIP, port: 80}
  │
  └── nodes/
      ├── worker-node-1 → {status: Ready, resources}
      └── worker-node-2 → {status: Ready, resources}
```

---

### **etcd in Action - Example:**

**Scenario: Scaling a Deployment**

```bash
kubectl scale deployment nginx --replicas=5
```

**Behind the Scenes:**

```
1. kubectl → API Server
   PATCH /apis/apps/v1/namespaces/default/deployments/nginx
   {spec: {replicas: 5}}

2. API Server validates request
   Authentication ✓
   Authorization ✓

3. API Server writes to etcd
   etcd.update("/registry/deployments/default/nginx")
   Change: replicas: 3 → 5

4. Deployment Controller watches etcd
   "Deployment changed! Current: 3, Desired: 5"
   "Need to create 2 more ReplicaSets!"

5. Controller → API Server
   "Create 2 new pods"

6. API Server → etcd
   Creates 2 new pod objects

7. Scheduler watches etcd
   "2 new pods need nodes!"
   Assigns nodes to pods

8. Kubelet watches etcd
   "New pods assigned to me!"
   Creates containers
```

---

### **etcd High Availability:**

**Quorum-Based Consensus (Raft Algorithm):**

```
3-Node etcd Cluster:

Node 1 (Leader) ← Receives write
Node 2 (Follower)
Node 3 (Follower)

Write Process:
1. Leader receives write
2. Leader sends to followers
3. Wait for majority (2/3) to confirm
4. Commit write
5. Return success

If Leader fails:
- Followers elect new leader
- Cluster continues operating
- No data loss!

Quorum = (n/2) + 1
3 nodes: Need 2 to agree
5 nodes: Need 3 to agree
```

---

### **etcd Best Practices:**

**1. Run Odd Number of Nodes:**
```
✅ Good: 3, 5, 7 nodes
❌ Bad: 2, 4, 6 nodes

Why? Quorum calculation:
3 nodes: Survive 1 failure (need 2)
4 nodes: Survive 1 failure (need 3) ← No benefit over 3!
5 nodes: Survive 2 failures (need 3)
```

**2. Backup etcd Regularly:**
```bash
# Backup etcd
ETCDCTL_API=3 etcdctl snapshot save backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Restore from backup
ETCDCTL_API=3 etcdctl snapshot restore backup.db \
  --data-dir=/var/lib/etcd-restore
```

**3. Monitor etcd Performance:**
```
Key metrics:
- Latency (should be < 10ms)
- Storage size
- Number of watchers
- Leader elections (should be rare)
```

---

## 📅 kube-scheduler (The Matchmaker)

The **Scheduler** decides which node should run each pod.

### **What is kube-scheduler?**

Think of it as a **matchmaker** or **wedding planner**:
- Finds best match between pod and node
- Considers preferences and requirements
- Ensures optimal placement

### **Scheduling Process:**

```
New Pod Created (no node assigned)
        ↓
   Scheduler notices
        ↓
1. Filter Phase (Find Suitable Nodes)
   - Node has enough CPU? ✓
   - Node has enough memory? ✓
   - Pod requires SSD? Node has SSD? ✓
   - Node labels match pod's nodeSelector? ✓
   
   Result: List of feasible nodes
        ↓
2. Score Phase (Rank Nodes)
   - BalancedResourceAllocation score
   - LeastRequestedPriority score
   - NodeAffinityPriority score
   
   Result: Each node gets a score
        ↓
3. Select Best Node
   Pick node with highest score
        ↓
4. Bind Pod to Node
   Update pod: .spec.nodeName = "worker-node-2"
```

---

### **Scheduling Factors:**

**1. Resource Requirements:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:        # Minimum needed
        memory: "64Mi"
        cpu: "250m"
      limits:          # Maximum allowed
        memory: "128Mi"
        cpu: "500m"
```

**Scheduler checks:**
```
Node 1: Available CPU: 200m, Memory: 50Mi ✗ (Not enough)
Node 2: Available CPU: 1000m, Memory: 2Gi ✓ (Enough)
Node 3: Available CPU: 500m, Memory: 1Gi ✓ (Enough)

Feasible nodes: Node 2, Node 3
```

---

**2. Node Selectors:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  nodeSelector:
    disktype: ssd      # Only schedule on nodes with SSD
    zone: us-east-1a
  containers:
  - name: nginx
    image: nginx
```

**Scheduler filters:**
```
Node 1: disktype=hdd ✗
Node 2: disktype=ssd, zone=us-east-1a ✓
Node 3: disktype=ssd, zone=us-east-1b ✗

Selected: Node 2
```

---

**3. Affinity/Anti-Affinity:**
```yaml
# Pod Affinity: Run near other pods
apiVersion: v1
kind: Pod
metadata:
  name: backend
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - frontend
        topologyKey: kubernetes.io/hostname

# Meaning: Schedule backend on same node as frontend
```

---

**4. Taints and Tolerations:**
```bash
# Taint a node (repels pods)
kubectl taint nodes node1 gpu=true:NoSchedule

# Only pods with toleration can schedule there
```

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
  - name: cuda-app
    image: nvidia/cuda
```

---

### **Scheduler Scoring Example:**

**Scenario: Schedule pod requiring 500m CPU, 1Gi RAM**

```
Nodes after filtering:
- Node 1: 2 CPU, 4Gi RAM available
- Node 2: 1 CPU, 2Gi RAM available
- Node 3: 4 CPU, 8Gi RAM available

Scoring:

LeastRequestedPriority (prefers less utilized):
  Formula: (capacity - requests) / capacity
  
  Node 1: (2000m - 500m) / 2000m = 0.75 → Score: 75
  Node 2: (1000m - 500m) / 1000m = 0.50 → Score: 50
  Node 3: (4000m - 500m) / 4000m = 0.875 → Score: 87

BalancedResourceAllocation (prefers balanced CPU/memory):
  Node 1: CPU: 25%, Memory: 25% → Balanced ✓ → Score: 90
  Node 2: CPU: 50%, Memory: 50% → Balanced ✓ → Score: 80
  Node 3: CPU: 12%, Memory: 12% → Balanced ✓ → Score: 95

Total Scores:
  Node 1: (75 + 90) / 2 = 82.5
  Node 2: (50 + 80) / 2 = 65
  Node 3: (87 + 95) / 2 = 91 ← WINNER!

Result: Pod scheduled on Node 3
```

---

## 🎮 kube-controller-manager (The Autopilot)

The **Controller Manager** runs multiple controllers that maintain desired state.

### **What is kube-controller-manager?**

Think of it as **autopilot systems** in a car:
- Cruise control maintains speed
- Lane assist keeps you centered
- Controllers maintain cluster state

### **Controller Pattern:**

```
Observe → Compare → Act

Loop Forever:
1. Observe current state (read from API/etcd)
2. Compare with desired state (what should be)
3. If different, take action to reconcile
4. Wait and repeat
```

---

### **Major Controllers:**

**1. Node Controller:**
```
Responsibility: Monitor node health

Every 5 seconds:
  Check: Is node sending heartbeats?
  
  If no heartbeat for 40 seconds:
    Mark node as "Unknown"
  
  If unknown for 5 minutes:
    Evict all pods from node
    Schedule pods on healthy nodes
```

**Example:**
```
Timeline:
00:00 - Node healthy, running 10 pods
00:30 - Node crashes, stops sending heartbeats
01:10 - Node Controller: "No heartbeat for 40s, mark Unknown"
06:10 - Node Controller: "Unknown for 5 min, evict pods"
06:15 - Pods rescheduled on other nodes
06:20 - New pods running on healthy nodes
```

---

**2. ReplicaSet Controller:**
```
Responsibility: Maintain correct number of pod replicas

Desired: 3 replicas
Current: 2 replicas

Action: Create 1 new pod

Every 30 seconds:
  Count running pods
  Compare with desired replicas
  If fewer: Create missing pods
  If more: Delete extra pods
```

**Example:**
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
```

**Controller Loop:**
```
Iteration 1:
  Observe: 2 pods running
  Desired: 3 pods
  Action: Create 1 pod ✓

Iteration 2:
  Observe: 3 pods running
  Desired: 3 pods
  Action: Nothing (already correct)

User deletes 1 pod:

Iteration 3:
  Observe: 2 pods running
  Desired: 3 pods
  Action: Create 1 pod ✓ (self-healing!)
```

---

**3. Deployment Controller:**
```
Responsibility: Manage rollouts and rollbacks

Watches: Deployments
Creates: ReplicaSets
Manages: Rolling updates

Example: Update nginx:1.19 → nginx:1.20

Steps:
1. Create new ReplicaSet (nginx:1.20) with 0 replicas
2. Scale new RS to 1, scale old RS to 2
3. Scale new RS to 2, scale old RS to 1
4. Scale new RS to 3, scale old RS to 0
5. Keep old RS (for rollback)
```

---

**4. Service Controller:**
```
Responsibility: Create cloud load balancers for LoadBalancer services

When Service type=LoadBalancer created:
1. Call cloud provider API
2. Create load balancer (AWS ELB, Azure LB, GCP LB)
3. Configure backend pool with node IPs
4. Update Service with external IP
```

**Example:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  type: LoadBalancer      # Controller sees this
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80

# Service Controller:
# 1. Calls AWS API to create ELB
# 2. Updates Service: .status.loadBalancer.ingress[0].ip = "54.123.45.67"
```

---

**5. Endpoint Controller:**
```
Responsibility: Populate Endpoints objects

Watches: Services and Pods
Creates: Endpoints (list of pod IPs)

Service "nginx" with selector "app=nginx"
  ↓
Controller finds pods with label "app=nginx"
  ↓
Creates Endpoints object with pod IPs:
  - 10.244.1.5:80
  - 10.244.2.8:80
  - 10.244.3.2:80
```

---

## 🔗 How Components Work Together

### **Example: Creating a Deployment**

**User runs:**
```bash
kubectl create deployment nginx --image=nginx --replicas=3
```

**Complete Flow:**

```
1. kubectl → API Server
   "Create deployment 'nginx' with 3 replicas"
   
2. API Server
   - Authenticates user ✓
   - Authorizes action ✓
   - Validates deployment spec ✓
   - Writes to etcd ✓
   - Returns: "deployment.apps/nginx created"

3. Deployment Controller (watching etcd)
   "New deployment! Need to create ReplicaSet"
   → API Server: "Create ReplicaSet for nginx"

4. API Server
   - Writes ReplicaSet to etcd ✓

5. ReplicaSet Controller (watching etcd)
   "New ReplicaSet! Need 3 pods, have 0"
   → API Server: "Create 3 pods"

6. API Server
   - Writes 3 pod objects to etcd ✓
   - Pods have no node assignment yet

7. Scheduler (watching etcd)
   "3 new pods need scheduling!"
   - Filters nodes → finds 3 suitable nodes
   - Scores nodes → selects best nodes
   → API Server: "Assign pod1 to node-1, pod2 to node-2, pod3 to node-3"

8. API Server
   - Updates pods with node assignments ✓
   - Writes to etcd ✓

9. Kubelet on node-1 (watching API Server)
   "Pod assigned to me!"
   - Pulls nginx image
   - Creates container
   - Starts container
   → API Server: "Pod is running"

10. Kubelet on node-2 and node-3
    Same process as node-1

11. API Server
    - Updates pod status in etcd
    - Status: Running ✓

Result: 3 nginx pods running on 3 different nodes!
```

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

