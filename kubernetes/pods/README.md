# Kubernetes Pods - Complete Theoretical Guide

**Understanding the Smallest Deployable Unit in Kubernetes**

---

## 📚 Table of Contents

1. [What is a Pod?](#what-is-pod)
2. [Pod Lifecycle](#pod-lifecycle)
3. [Pod Phases](#pod-phases)
4. [Container States](#container-states)
5. [Init Containers](#init-containers)
6. [Sidecar Containers](#sidecar-containers)
7. [Multi-Container Patterns](#multi-container-patterns)
8. [Pod Networking](#pod-networking)
9. [Resource Management](#resource-management)
10. [Pod Quality of Service](#qos)
11. [Pod Disruption Budgets](#pdb)

---

## 📦 What is a Pod?

A **Pod** is the smallest deployable unit in Kubernetes - a wrapper around one or more containers.

### **Simple Analogy:**

```
Pod = Apartment/House
Containers = Roommates living together

Roommates in same apartment:
- Share the same address (IP)
- Share utilities (network, storage)
- Share resources (CPU, memory allocated to apartment)
- Can communicate easily (localhost)
- Live and die together (if apartment demolished, all move out)
```

---

### **Why Pods, Not Just Containers?**

**❌ Just Containers (Docker):**
```
Container 1: Web Server (nginx)
Container 2: Log Collector (fluentd)

Problems:
- Different IP addresses
- Can't easily share files
- Complex networking setup
- Hard to deploy together
```

**✅ Pods (Kubernetes):**
```
Pod = Web Server + Log Collector
- Same IP address
- Shared volumes
- Same network namespace
- Deploy/scale together
- Communicate via localhost
```

---

### **Single-Container vs Multi-Container Pods:**

**Single-Container Pod (Most Common ~90%):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
```

```
┌─────────────────────┐
│       Pod           │
│  ┌───────────────┐  │
│  │  nginx        │  │
│  │  container    │  │
│  └───────────────┘  │
│  IP: 10.244.1.5     │
└─────────────────────┘
```

---

**Multi-Container Pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-sidecar
spec:
  containers:
  - name: app
    image: myapp:1.0
    ports:
    - containerPort: 8080
  - name: log-collector
    image: fluentd:latest
```

```
┌────────────────────────────────┐
│          Pod                    │
│  ┌──────────┐  ┌─────────────┐ │
│  │   app    │  │log-collector│ │
│  │:8080     │  │             │ │
│  └──────────┘  └─────────────┘ │
│  IP: 10.244.1.5                │
│  Shared: localhost, volumes    │
└────────────────────────────────┘
```

**Containers in same pod:**
- Share IP address
- Share network namespace
- Share volumes
- Communicate via localhost
- Scheduled on same node

---

## 🔄 Pod Lifecycle

Understanding the complete journey of a pod from creation to deletion.

### **Pod Lifecycle Stages:**

```
1. Pod Definition (YAML)
        ↓
2. API Server Accepts Pod
        ↓
3. Scheduler Assigns Node
        ↓
4. kubelet Creates Pod
        ↓
5. Pod Running
        ↓
6. Pod Completes/Fails
        ↓
7. Pod Terminated
        ↓
8. Pod Deleted
```

---

### **Complete Pod Creation Flow:**

```
User:
$ kubectl apply -f pod.yaml

T+0s: kubectl → API Server
  POST /api/v1/namespaces/default/pods
  Body: Pod spec

T+0.1s: API Server validates and stores in etcd
  Pod created, Phase: Pending, No node assigned

T+0.2s: Scheduler notices new pod
  "New pod needs scheduling!"
  Filters nodes → Scores nodes
  Best node: worker-2

T+0.3s: Scheduler → API Server
  PATCH: pod.spec.nodeName = "worker-2"
  Pod still Pending (not running yet)

T+0.4s: kubelet on worker-2 notices pod
  "Pod assigned to me!"
  Checks pod spec...

T+1s: kubelet pulls image
  "Pulling nginx:1.21..."
  (May take seconds to minutes)

T+10s: Image pull complete
  kubelet → API Server
  Update: pod.status.conditions[ImagePulled] = True

T+11s: kubelet creates container
  CRI CreateContainer(nginx)
  Mounts volumes
  Sets environment variables

T+12s: kubelet starts container
  CRI StartContainer(nginx)
  Container process starts

T+13s: Container running
  kubelet → API Server
  Update: pod.status.phase = Running

T+15s: Readiness probe (if configured)
  HTTP GET http://pod-ip/health
  Response: 200 OK

T+16s: Pod ready for traffic
  kubelet → API Server
  Update: pod.status.conditions[Ready] = True

Pod lifecycle: Pending → Running → (Succeeded/Failed) → Terminated
```

---

## 📊 Pod Phases

Pod phase is a high-level summary of where the pod is in its lifecycle.

### **Pod Phases:**

**1. Pending:**
```
Meaning: Pod accepted but not running yet

Reasons:
- Waiting for scheduler to assign node
- Pulling container images
- Waiting for volumes to attach
- Insufficient resources on cluster

Example:
NAME    READY   STATUS    RESTARTS   AGE
myapp   0/1     Pending   0          30s

Check:
kubectl describe pod myapp
Events:
  FailedScheduling: 0/3 nodes available: insufficient memory
```

---

**2. Running:**
```
Meaning: Pod bound to node, all containers created, at least one running

Possible states:
- All containers running ✓
- Some containers running, some starting
- Some containers running, some crashed (restarting)

Example:
NAME    READY   STATUS    RESTARTS   AGE
myapp   1/1     Running   0          2m

All good! Pod is healthy and running.
```

---

**3. Succeeded:**
```
Meaning: All containers terminated successfully (exit code 0)

Use case: Jobs, batch processing

Example:
NAME          READY   STATUS      RESTARTS   AGE
batch-job     0/1     Completed   0          5m

Container ran, finished work, exited cleanly.
```

---

**4. Failed:**
```
Meaning: All containers terminated, at least one failed (exit code != 0)

Example:
NAME    READY   STATUS   RESTARTS   AGE
myapp   0/1     Error    0          1m

Check logs:
kubectl logs myapp
Error: Database connection failed

Fix issue and redeploy
```

---

**5. Unknown:**
```
Meaning: Pod state cannot be determined

Cause: Usually node communication lost

Example:
NAME    READY   STATUS    RESTARTS   AGE
myapp   0/1     Unknown   0          10m

Likely: Node crashed or network issue
Action: Check node status
```

---

## 🔍 Container States

Each container in a pod has a state.

### **Container States:**

**1. Waiting:**
```
Container not running yet

Reasons:
- Pulling image
- Waiting for init containers
- CrashLoopBackOff (restarting after crash)

Example:
kubectl describe pod myapp
Containers:
  app:
    State:      Waiting
    Reason:     ContainerCreating / ImagePullBackOff / CrashLoopBackOff
```

**Common Waiting Reasons:**

```yaml
# ContainerCreating
State: Waiting
Reason: ContainerCreating
# Container is being created, normal during startup

# ImagePullBackOff
State: Waiting
Reason: ImagePullBackOff
# Can't pull image (wrong name, no credentials, network issue)

# CrashLoopBackOff
State: Waiting
Reason: CrashLoopBackOff
# Container keeps crashing, Kubernetes backing off restarts
# Wait time: 10s, 20s, 40s, 80s, 160s (max 5 minutes)
```

---

**2. Running:**
```
Container executing

Example:
kubectl describe pod myapp
Containers:
  app:
    State:      Running
    Started:    2024-01-08 10:30:00
    
Good! Container is healthy.
```

---

**3. Terminated:**
```
Container finished execution or was killed

Example:
kubectl describe pod myapp
Containers:
  app:
    State:      Terminated
    Reason:     Completed / Error / OOMKilled
    Exit Code:  0 / 1 / 137
    Started:    2024-01-08 10:30:00
    Finished:   2024-01-08 10:35:00
```

**Common Exit Codes:**
```
0:   Success (clean exit)
1:   General error
2:   Misuse of shell command
126: Command cannot execute
127: Command not found
130: Terminated by Ctrl+C
137: Killed (SIGKILL) - usually OOM
143: Terminated (SIGTERM) - graceful shutdown
```

---

## 🚀 Init Containers

**Init containers** run before app containers and must complete successfully.

### **What are Init Containers?**

Think of them as **preparation crew**:
- Set up stage before main show
- Download dependencies
- Wait for services to be ready
- Initialize configuration

```
┌─────────────────────────────────────────┐
│              Pod Lifecycle               │
├─────────────────────────────────────────┤
│  1. Init Container 1 runs → Completes   │
│        ↓                                 │
│  2. Init Container 2 runs → Completes   │
│        ↓                                 │
│  3. Main Container starts                │
└─────────────────────────────────────────┘

Init containers run sequentially (one after another)
Main container only starts after ALL init containers succeed
```

---

### **Init Container Example:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  # Init containers run first
  initContainers:
  - name: wait-for-database
    image: busybox
    command: ['sh', '-c']
    args:
    - |
      echo "Waiting for database to be ready..."
      until nc -z database-service 5432; do
        echo "Database not ready, waiting..."
        sleep 2
      done
      echo "Database is ready!"
  
  - name: download-config
    image: busybox
    command: ['sh', '-c']
    args:
    - |
      echo "Downloading configuration..."
      wget -O /config/app.conf http://config-server/app.conf
    volumeMounts:
    - name: config
      mountPath: /config
  
  # Main container starts after init containers succeed
  containers:
  - name: app
    image: myapp:1.0
    volumeMounts:
    - name: config
      mountPath: /config
  
  volumes:
  - name: config
    emptyDir: {}
```

**Execution Flow:**
```
T+0s:  Pod created
T+1s:  init-wait-for-database starts
T+1s:  Checking database... not ready
T+3s:  Checking database... not ready
T+5s:  Checking database... ready! Init container exits (code 0)
T+6s:  init-download-config starts
T+8s:  Downloaded config file
T+9s:  Init container exits (code 0)
T+10s: Main app container starts
T+11s: Pod running with config ready!
```

---

### **Common Init Container Use Cases:**

**1. Wait for Service:**
```yaml
initContainers:
- name: wait-for-redis
  image: busybox
  command: ['sh', '-c', 'until nc -z redis 6379; do sleep 1; done']
```

**2. Clone Git Repository:**
```yaml
initContainers:
- name: git-clone
  image: alpine/git
  command: ['git', 'clone', 'https://github.com/user/repo.git', '/data']
  volumeMounts:
  - name: data
    mountPath: /data
```

**3. Set Permissions:**
```yaml
initContainers:
- name: fix-permissions
  image: busybox
  command: ['sh', '-c', 'chmod -R 777 /data']
  volumeMounts:
  - name: data
    mountPath: /data
```

---

## 🔗 Sidecar Containers

**Sidecar containers** run alongside main container to provide supporting functionality.

### **What are Sidecars?**

Think of them as **supporting actors**:
- Main actor does the main job
- Supporting actors help (lighting, props, etc.)
- All work together simultaneously

```
┌────────────────────────────────────┐
│           Pod                       │
│  ┌──────────┐  ┌─────────────────┐ │
│  │   Main   │  │    Sidecar      │ │
│  │    App   │  │ (Log collector) │ │
│  │          │  │                 │ │
│  │ Writes   │→→│  Reads logs     │ │
│  │ logs to  │  │  Sends to       │ │
│  │ file     │  │  centralized    │ │
│  └──────────┘  └─────────────────┘ │
│  Shared volume: /var/log           │
└────────────────────────────────────┘

Both containers run simultaneously
They complement each other
```

---

### **Sidecar Pattern Example:**

**Log Collection Sidecar:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-app-with-logging
spec:
  containers:
  # Main application container
  - name: webapp
    image: nginx:1.21
    ports:
    - containerPort: 80
    volumeMounts:
    - name: logs
      mountPath: /var/log/nginx
  
  # Sidecar: Log collector
  - name: log-collector
    image: fluentd:latest
    volumeMounts:
    - name: logs
      mountPath: /var/log/nginx
    env:
    - name: FLUENT_ELASTICSEARCH_HOST
      value: "elasticsearch.logging.svc"
  
  volumes:
  - name: logs
    emptyDir: {}
```

**How it works:**
```
1. nginx writes access logs to /var/log/nginx/access.log
2. fluentd reads logs from shared volume
3. fluentd forwards logs to Elasticsearch
4. Both containers run continuously
```

---

**Service Mesh Sidecar (Istio Example):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  # Main application
  - name: app
    image: myapp:1.0
    ports:
    - containerPort: 8080
  
  # Sidecar: Envoy proxy (injected by Istio)
  - name: istio-proxy
    image: istio/proxyv2:1.18.0
    # Intercepts all traffic
    # Provides: mTLS, retries, circuit breaking, telemetry
```

**Traffic flow with sidecar:**
```
External Request
    ↓
Service
    ↓
Envoy Sidecar (入) → Logs, applies policies
    ↓
App Container
    ↓
Envoy Sidecar (出) → Logs, applies policies
    ↓
Response
```

---

## 🏗️ Multi-Container Patterns

Common patterns for multi-container pods.

### **1. Sidecar Pattern:**
```
Main container + Helper container
Use: Logging, monitoring, proxying

Example: App + Log collector
```

### **2. Ambassador Pattern:**
```
Main container + Proxy container
Use: Connection pooling, circuit breaking

Example:
┌─────────────────────────────┐
│  Pod                         │
│  ┌────────┐  ┌────────────┐ │
│  │  App   │→→│ Ambassador │→→ External Service
│  │        │  │   Proxy    │   (Database, API)
│  └────────┘  └────────────┘ │
└─────────────────────────────┘

App thinks it's talking to localhost
Ambassador handles actual external connection
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-ambassador
spec:
  containers:
  - name: app
    image: myapp:1.0
    env:
    - name: DATABASE_URL
      value: "localhost:5432"  # Talks to ambassador
  
  - name: ambassador
    image: postgres-proxy:latest
    ports:
    - containerPort: 5432
    env:
    - name: REAL_DATABASE_URL
      value: "prod-db.example.com:5432"
```

---

### **3. Adapter Pattern:**
```
Main container + Adapter container
Use: Standardize output format

Example: App with custom logs + Adapter that converts to standard format

┌──────────────────────────────────┐
│  Pod                              │
│  ┌────────┐  ┌────────────────┐  │
│  │  App   │→→│   Adapter      │→→ Monitoring System
│  │Custom  │  │ (Standardizes) │   (Expects standard format)
│  │Logs    │  │                │  │
│  └────────┘  └────────────────┘  │
└──────────────────────────────────┘
```

---

## 🌐 Pod Networking

Every pod gets its own IP address.

### **Pod Networking Basics:**

```
Pod 1: 10.244.1.5
  Container A: localhost
  Container B: localhost
  (Containers in same pod share IP)

Pod 2: 10.244.2.8
  Container C: localhost

Pod 1 → Pod 2: Use 10.244.2.8
Container A in Pod 1 → Container B in Pod 1: Use localhost
```

---

### **Network Model:**

```
Kubernetes Network Requirements:
1. Every pod gets unique IP
2. Pods can communicate without NAT
3. All pods can reach all other pods
4. Containers in same pod use localhost

┌─────────────────────────────────────┐
│  Node 1                              │
│  ┌──────────┐  ┌──────────┐        │
│  │Pod A     │  │Pod B     │        │
│  │10.244.1.5│  │10.244.1.6│        │
│  └──────────┘  └──────────┘        │
└─────────────────────────────────────┘
          │
          │ (Pod network)
          │
┌─────────────────────────────────────┐
│  Node 2                              │
│  ┌──────────┐  ┌──────────┐        │
│  │Pod C     │  │Pod D     │        │
│  │10.244.2.5│  │10.244.2.6│        │
│  └──────────┘  └──────────┘        │
└─────────────────────────────────────┘

Pod A can directly reach Pod C at 10.244.2.5
No NAT, no port mapping needed!
```

---

## 💪 Resource Management

Pods can request and limit CPU/memory.

### **Requests vs Limits:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
spec:
  containers:
  - name: app
    image: myapp:1.0
    resources:
      requests:          # Minimum guaranteed
        memory: "128Mi"  # 128 Megabytes
        cpu: "250m"      # 250 millicores (0.25 CPU)
      limits:            # Maximum allowed
        memory: "256Mi"
        cpu: "500m"      # 500 millicores (0.5 CPU)
```

**What they mean:**
```
Requests (Minimum guarantee):
- Scheduler uses this for placement
- Node must have at least this much available
- Pod guaranteed to get these resources

Limits (Maximum cap):
- Pod cannot exceed these
- CPU: Throttled if exceeds
- Memory: OOMKilled if exceeds
```

---

### **CPU Units:**

```
1 CPU = 1000m (millicores)

Examples:
100m  = 0.1 CPU (10%)
250m  = 0.25 CPU (25%)
500m  = 0.5 CPU (50%)
1000m = 1 CPU (100%)
2000m = 2 CPUs
```

**What happens:**
```
Request: 250m, Limit: 500m

Normal operation: Uses 200m ✓
Spike: Uses 400m ✓
Heavy load: Tries to use 600m → Throttled to 500m ✓
```

---

### **Memory Units:**

```
Ki = Kibibyte = 1024 bytes
Mi = Mebibyte = 1024 Ki = 1,048,576 bytes
Gi = Gibibyte = 1024 Mi

Examples:
128Mi  = 128 Mebibytes = ~134 MB
256Mi  = 256 Mebibytes = ~268 MB
1Gi    = 1 Gibibyte = ~1.07 GB
```

**What happens:**
```
Request: 128Mi, Limit: 256Mi

Normal operation: Uses 100Mi ✓
Growing: Uses 200Mi ✓
Memory leak: Reaches 256Mi → OOMKilled! ✗
(Pod restarts)
```

---

## 🏆 Pod Quality of Service (QoS)

Kubernetes assigns QoS class based on resource requests/limits.

### **QoS Classes:**

**1. Guaranteed (Highest priority):**
```yaml
# All containers have:
# - Memory request = Memory limit
# - CPU request = CPU limit

spec:
  containers:
  - name: app
    resources:
      requests:
        memory: "256Mi"
        cpu: "500m"
      limits:
        memory: "256Mi"  # Same as request
        cpu: "500m"      # Same as request

QoS: Guaranteed
Priority: Highest (last to be evicted)
```

---

**2. Burstable (Medium priority):**
```yaml
# At least one container has request < limit
# OR has request but no limit

spec:
  containers:
  - name: app
    resources:
      requests:
        memory: "128Mi"
        cpu: "250m"
      limits:
        memory: "256Mi"  # Different from request
        cpu: "500m"

QoS: Burstable
Priority: Medium
```

---

**3. BestEffort (Lowest priority):**
```yaml
# No requests or limits specified

spec:
  containers:
  - name: app
    image: myapp:1.0
    # No resources specified

QoS: BestEffort
Priority: Lowest (first to be evicted)
```

---

### **Eviction Order (When node runs out of resources):**

```
Node running out of memory:

1. Kill BestEffort pods first
2. Then kill Burstable pods (exceeding requests)
3. Last: Kill Guaranteed pods

Example scenario:
Node: 8GB RAM
Used: 7.9GB (Memory pressure!)

Pods running:
- Pod A (BestEffort): Using 1GB → Killed first
- Pod B (Burstable): Using 2GB → Killed second
- Pod C (Guaranteed): Using 3GB → Protected
- Pod D (Guaranteed): Using 2GB → Protected

After killing Pod A:
Memory freed: 1GB
Pressure relieved ✓
Guaranteed pods safe!
```

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

