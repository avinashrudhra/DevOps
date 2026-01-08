# kubectl & kubelet - Complete Theoretical Guide

**Understanding the Client Tool and Node Agent**

---

## 📚 Table of Contents

1. [kubectl - The Client Tool](#kubectl)
2. [kubelet - The Node Agent](#kubelet)
3. [How They Work Together](#working-together)
4. [kubectl Commands Deep Dive](#kubectl-commands)
5. [kubelet Pod Lifecycle](#kubelet-lifecycle)
6. [Real-World Examples](#real-world-examples)

---

## 🔧 kubectl (The Kubernetes Client)

**kubectl** is the command-line tool for interacting with Kubernetes clusters.

### **What is kubectl?**

Think of kubectl as your **remote control** for Kubernetes:
- You press buttons (run commands)
- It sends signals (HTTP requests) to API server
- Cluster responds (creates pods, shows status)

```
┌──────────────┐
│   kubectl    │  Command-line tool (runs on your laptop)
│ (Your Remote)│
└──────┬───────┘
       │ HTTPS
       ↓
┌──────────────────┐
│  API Server      │  Receives and processes requests
│ (Control Plane)  │
└──────────────────┘
```

---

### **kubectl Architecture:**

```
kubectl command
    ↓
1. Parse command
   "kubectl get pods" → GET /api/v1/namespaces/default/pods
    ↓
2. Read kubeconfig
   Find cluster endpoint, credentials
    ↓
3. Build HTTP request
   URL: https://api-server:6443/api/v1/namespaces/default/pods
   Headers: Authorization: Bearer <token>
    ↓
4. Send to API Server
   TLS connection, certificate validation
    ↓
5. Receive response
   JSON data with pod list
    ↓
6. Format output
   Table, YAML, JSON (based on -o flag)
    ↓
7. Display to user
```

---

### **kubeconfig File:**

**Location:** `~/.kube/config`

```yaml
apiVersion: v1
kind: Config
current-context: production    # Which cluster to use

clusters:                       # Available clusters
- name: production
  cluster:
    server: https://prod-api.example.com:6443
    certificate-authority-data: <base64-cert>

users:                          # Authentication credentials
- name: admin
  user:
    client-certificate-data: <base64-cert>
    client-key-data: <base64-key>

contexts:                       # Cluster + User + Namespace
- name: production
  context:
    cluster: production
    user: admin
    namespace: default
```

**What each part means:**
- **Cluster**: Where to connect (API server URL)
- **User**: How to authenticate (certificates, tokens)
- **Context**: Cluster + User + Default namespace
- **Current-context**: Which context kubectl uses

---

### **kubectl Command Structure:**

```bash
kubectl [command] [TYPE] [NAME] [flags]
```

**Examples:**
```bash
kubectl get pods                    # List pods
kubectl describe deployment nginx   # Show deployment details
kubectl delete service web          # Delete service
kubectl create -f pod.yaml          # Create from file
kubectl apply -f deployment.yaml    # Create or update

kubectl get pods -n kube-system     # Different namespace
kubectl get pods --all-namespaces   # All namespaces
kubectl get pods -o yaml            # YAML output
kubectl get pods -w                 # Watch for changes
```

---

### **Common kubectl Commands:**

**1. Get Resources (View):**
```bash
kubectl get pods                    # List pods
kubectl get deployments             # List deployments
kubectl get services                # List services
kubectl get nodes                   # List nodes
kubectl get all                     # All resources

# With details
kubectl get pods -o wide            # More columns (IP, node)
kubectl get pods -o yaml            # Full YAML
kubectl get pods -o json            # JSON format

# Watch for changes
kubectl get pods -w                 # Stream updates
```

**2. Describe Resources (Details):**
```bash
kubectl describe pod nginx-abc123
kubectl describe node worker-1
kubectl describe service web

# Shows:
# - Full spec
# - Current status
# - Events (what happened)
# - Conditions
```

**3. Create Resources:**
```bash
kubectl create deployment nginx --image=nginx
kubectl create service clusterip web --tcp=80:80

kubectl create -f pod.yaml          # From file
kubectl apply -f deployment.yaml    # Create or update
```

**4. Update Resources:**
```bash
kubectl edit deployment nginx       # Opens editor
kubectl scale deployment nginx --replicas=5
kubectl set image deployment/nginx nginx=nginx:1.21

kubectl apply -f deployment.yaml    # Declarative
```

**5. Delete Resources:**
```bash
kubectl delete pod nginx
kubectl delete deployment web
kubectl delete -f deployment.yaml   # Delete from file

kubectl delete pods --all           # Delete all pods
```

**6. Execute Commands:**
```bash
kubectl exec -it nginx -- /bin/bash    # Shell into container
kubectl exec nginx -- ls /var/log      # Run command

kubectl logs nginx                     # View logs
kubectl logs nginx -f                  # Stream logs
kubectl logs nginx --previous          # Previous container logs
```

**7. Copy Files:**
```bash
kubectl cp /local/file pod:/container/path
kubectl cp pod:/container/file /local/path
```

**8. Port Forwarding:**
```bash
kubectl port-forward pod/nginx 8080:80   # Local:Container
curl localhost:8080                      # Access pod
```

---

### **kubectl Behind the Scenes:**

**Example: `kubectl get pods`**

```
User runs:
$ kubectl get pods

Step-by-step:
1. kubectl reads ~/.kube/config
   - Gets API server: https://api.example.com:6443
   - Gets credentials: client-cert, client-key

2. kubectl builds HTTP request
   Method: GET
   URL: https://api.example.com:6443/api/v1/namespaces/default/pods
   Headers:
     - Authorization: Bearer <token>
     - Accept: application/json

3. kubectl sends request (TLS)
   - Validates server certificate
   - Presents client certificate

4. API Server receives request
   - Authenticates: Valid certificate? ✓
   - Authorizes: Can user list pods? ✓
   - Retrieves pod list from etcd

5. API Server responds
   HTTP 200 OK
   Body: {"kind": "PodList", "items": [{...}, {...}]}

6. kubectl processes response
   - Parses JSON
   - Formats as table

7. kubectl displays
   NAME        READY   STATUS    RESTARTS   AGE
   nginx-abc   1/1     Running   0          5m
   web-def     1/1     Running   0          3m
```

---

## 🤖 kubelet (The Node Agent)

**kubelet** is an agent that runs on each worker node and manages pods.

### **What is kubelet?**

Think of kubelet as the **construction supervisor** on a building site:
- Receives blueprints (pod specs) from control plane
- Builds the structures (starts containers)
- Reports progress (pod status)
- Maintains quality (health checks)

```
┌──────────────────────┐
│  Control Plane       │  "Run this pod on worker-1"
│  (API Server)        │
└───────┬──────────────┘
        │ (Assignment)
        ↓
┌──────────────────────┐
│  kubelet             │  "Got it! Starting container..."
│  (Worker Node)       │
└──────┬───────────────┘
       │ (Manages)
       ↓
┌──────────────────────┐
│  Container Runtime   │  Docker, containerd, CRI-O
│  (Containers)        │
└──────────────────────┘
```

---

### **kubelet Responsibilities:**

**1. Pod Lifecycle Management:**
```
1. Watch API Server for pods assigned to this node
2. Pull container images
3. Start containers
4. Monitor container health
5. Restart failed containers
6. Report status back to API Server
7. Clean up terminated pods
```

**2. Resource Monitoring:**
```
- CPU usage
- Memory usage
- Disk usage
- Network statistics

Reports to: API Server (for kubectl top)
```

**3. Health Checks:**
```
- Liveness Probes (is container alive?)
- Readiness Probes (is container ready for traffic?)
- Startup Probes (has container started?)
```

**4. Volume Management:**
```
- Mount volumes to pods
- Attach/detach storage
- Manage secret/configmap volumes
```

---

### **kubelet Workflow:**

```
Every kubelet runs this loop continuously:

1. Watch API Server
   GET /api/v1/pods?fieldSelector=spec.nodeName=worker-1
   "Give me all pods assigned to me"
        ↓
2. Compare desired vs actual
   Desired: 5 pods should run
   Actual: 4 pods running
   Difference: 1 pod missing!
        ↓
3. Sync pods
   - Start missing pod
   - Stop extra pods
   - Restart failed containers
        ↓
4. Container Runtime Interface (CRI)
   kubelet → CRI → containerd/Docker
   "Pull image nginx:1.21"
   "Create container with these specs"
        ↓
5. Monitor containers
   Check: Is container still running?
   Check: Are health probes passing?
        ↓
6. Report status to API Server
   POST /api/v1/namespaces/default/pods/nginx/status
   "Pod is Running, Container is Ready"
        ↓
7. Wait 10 seconds
   (Then repeat from step 1)
```

---

### **Pod Creation Flow (kubelet's perspective):**

**Scenario: Scheduler assigns pod to node**

```
Timeline:

T+0s: Scheduler assigns pod to worker-1
      Updates: pod.spec.nodeName = "worker-1"

T+1s: kubelet on worker-1 notices new pod
      "I have a new pod to run!"

T+2s: kubelet checks pod spec
      Image: nginx:1.21
      CPU: 100m, Memory: 128Mi
      Volumes: secret-volume

T+3s: kubelet pulls image
      containerd pull nginx:1.21
      "Pulling from library/nginx..."

T+15s: Image pull complete

T+16s: kubelet creates container
      CRI CreateContainer call
      - Set resource limits
      - Mount volumes
      - Set environment variables

T+17s: kubelet starts container
      CRI StartContainer call
      Container process starts

T+18s: Container running
      kubelet updates pod status
      API Server: "Pod phase: Running"

T+20s: kubelet runs liveness probe
      HTTP GET http://pod-ip:80/health
      Response: 200 OK ✓

T+21s: kubelet updates pod condition
      API Server: "Container Ready: True"

T+22s: Pod fully running and ready!
```

---

### **kubelet Health Probes:**

**1. Liveness Probe (Is container alive?):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - name: app
    image: myapp:1.0
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 30  # Wait 30s before first check
      periodSeconds: 10         # Check every 10s
      failureThreshold: 3       # Restart after 3 failures
```

**kubelet's liveness loop:**
```
Every 10 seconds:
  HTTP GET http://pod-ip:8080/health
  
  Response 200-399: Healthy ✓
    Continue monitoring
  
  Response 400+, timeout, or error: Unhealthy ✗
    Failure count: 1
  
  After 3 consecutive failures:
    "Container is dead! Restarting..."
    Kill container
    Start new container
    Reset failure count
```

---

**2. Readiness Probe (Ready for traffic?):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - name: app
    image: myapp:1.0
    readinessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
```

**Difference from liveness:**
```
Liveness Failed:
  → kubelet restarts container

Readiness Failed:
  → kubelet marks pod as NOT Ready
  → Service removes pod from endpoints
  → No traffic sent to pod
  → Container keeps running (not restarted)
```

---

**3. Startup Probe (For slow-starting containers):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: slow-app
spec:
  containers:
  - name: app
    image: slow-starting-app:1.0
    startupProbe:
      httpGet:
        path: /health
        port: 8080
      failureThreshold: 30      # Try 30 times
      periodSeconds: 10          # Every 10s
      # = 5 minutes total startup time allowed
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
      periodSeconds: 10
```

**Why startup probe?**
```
Problem without startup probe:
  App takes 2 minutes to start
  Liveness probe starts immediately
  Liveness kills "unhealthy" container before it finishes starting
  Container never starts successfully!

Solution with startup probe:
  Startup probe runs first (5 min allowance)
  Once startup succeeds once:
    → Liveness probe takes over
    → Normal health checking begins
```

---

## 🔗 kubectl & kubelet Working Together

**Complete Flow: Creating and Accessing a Pod**

```
User:
$ kubectl run nginx --image=nginx --port=80

┌─────────────────────────────────────────────────────────┐
│ 1. kubectl → API Server                                  │
│    POST /api/v1/namespaces/default/pods                  │
│    Body: {pod spec with nginx container}                │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│ 2. API Server → etcd                                     │
│    Stores pod object (no node assigned yet)             │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│ 3. Scheduler → API Server                                │
│    "Assign pod to worker-2"                             │
│    PATCH: pod.spec.nodeName = "worker-2"                │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│ 4. kubelet on worker-2 (watching API Server)            │
│    "New pod assigned to me!"                            │
│    - Pulls nginx image                                   │
│    - Creates container                                   │
│    - Starts container                                    │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│ 5. kubelet → API Server                                  │
│    PATCH /api/v1/namespaces/default/pods/nginx/status  │
│    "Container is running, pod is Ready"                 │
└─────────────────────────────────────────────────────────┘
                        ↓
User:
$ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          30s

┌─────────────────────────────────────────────────────────┐
│ 6. kubectl → API Server                                  │
│    GET /api/v1/namespaces/default/pods                  │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│ 7. API Server → etcd                                     │
│    Retrieves pod list with current status               │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│ 8. API Server → kubectl                                  │
│    Returns pod data                                      │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│ 9. kubectl                                               │
│    Formats and displays pod table                       │
└─────────────────────────────────────────────────────────┘
```

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

