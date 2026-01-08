# Kubernetes Services - Complete Theoretical Guide

**Understanding Pod Networking and Service Discovery**

---

## 📚 Table of Contents

1. [What is a Service?](#what-is-service)
2. [Why Services?](#why-services)
3. [Service Types](#service-types)
4. [ClusterIP Service](#clusterip)
5. [NodePort Service](#nodeport)
6. [LoadBalancer Service](#loadbalancer)
7. [ExternalName Service](#externalname)
8. [Headless Service](#headless)
9. [Service Discovery](#service-discovery)
10. [Endpoints](#endpoints)

---

## 🌐 What is a Service?

A **Service** is a stable network endpoint for accessing a set of pods.

### **Simple Analogy:**

```
Service = Restaurant Phone Number
Pods = Restaurant Staff

Without Service:
Customer: "Who's working today?"
         "What's their phone number?"
         "They changed shift!"
         "Lost the number!"

With Service:
Customer: Calls restaurant number (Service)
Service: Routes to available staff (Pods)
Staff changes? No problem!
Customer always uses same number
```

---

### **The Problem Services Solve:**

**❌ Without Services:**
```
You have 3 web app pods:
- Pod 1: IP 10.244.1.5
- Pod 2: IP 10.244.2.8  
- Pod 3: IP 10.244.3.2

Problems:
1. Which IP to use? 🤔
2. Pod crashes, new IP: 10.244.1.9 📱
3. Pod scales to 5, more IPs! 📈
4. How to load balance? ⚖️
5. IP changes every deployment! 🔄

Your app: "Which IP should I connect to???"
```

**✅ With Services:**
```
Service: myapp-service
  ClusterIP: 10.96.0.50 (Stable!)
  
Backend Pods: 3 → 5 → 2 (changes)
Pod IPs change? No problem!
  
Your app: "Connect to myapp-service:8080"
Service handles:
  - Load balancing
  - Pod discovery
  - Health checking
```

---

## 🤔 Why Services?

### **Services Provide:**

**1. Stable IP Address:**
```
Pods come and go: 10.244.1.5 → 10.244.2.9 → 10.244.1.11
Service IP stays: 10.96.0.50 (Never changes!)
```

**2. Load Balancing:**
```
Request → Service → Distributes to healthy pods
         → Pod 1 (Round robin)
         → Pod 2
         → Pod 3
```

**3. Service Discovery:**
```
Instead of: curl http://10.244.1.5:8080
Use:       curl http://myapp-service:8080
DNS resolves service name to IP
```

**4. Health Checking:**
```
Service only routes to healthy pods
Pod crashes? Removed from service
Pod recovers? Added back to service
```

---

## 📋 Service Types

Kubernetes has 5 service types for different use cases.

```
Service Types:

1. ClusterIP     (Default) - Internal cluster access only
2. NodePort      - Expose on each node's IP
3. LoadBalancer  - Cloud provider's load balancer
4. ExternalName  - DNS alias to external service
5. Headless      - Direct pod IPs (no load balancing)
```

---

## 🔒 ClusterIP Service

**Most common type** - Exposes service on internal cluster IP.

### **What is ClusterIP?**

```
ClusterIP = Internal phone number (only works inside company)

┌─────────────────────────────────────────┐
│  Kubernetes Cluster                      │
│                                          │
│  Pod A → Can access service ✓           │
│  Pod B → Can access service ✓           │
│                                          │
│  Service: myapp (10.96.0.50)            │
│    ↓                                     │
│  Backend Pods: 3 pods                    │
└─────────────────────────────────────────┘

Outside cluster → Cannot access ✗
```

---

### **ClusterIP Example:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: ClusterIP  # Default, can be omitted
  selector:
    app: backend   # Select pods with this label
  ports:
  - protocol: TCP
    port: 80       # Service port
    targetPort: 8080  # Pod port
```

**How it works:**
```
1. Service gets ClusterIP: 10.96.0.50

2. Finds pods matching selector:
   app=backend
   
   Found pods:
   - backend-pod-1: 10.244.1.5:8080
   - backend-pod-2: 10.244.2.8:8080
   - backend-pod-3: 10.244.3.2:8080

3. Creates load balancing rules:
   Traffic to 10.96.0.50:80 → 
     Round robin to pods on port 8080

4. Any pod in cluster can access:
   curl http://backend-service:80
   curl http://10.96.0.50:80
```

---

### **Traffic Flow:**

```
Frontend Pod wants to call backend:

1. DNS Query:
   "What's IP of backend-service?"
   DNS: "10.96.0.50"

2. Connection:
   Frontend Pod → 10.96.0.50:80
   
3. kube-proxy (on node):
   "10.96.0.50:80 routes to backend pods"
   Picks: backend-pod-2 (10.244.2.8:8080)
   
4. Packet sent:
   From: Frontend Pod
   To: 10.244.2.8:8080 (backend-pod-2)
   
5. Response:
   backend-pod-2 → Frontend Pod
```

---

### **Port Configuration:**

```yaml
spec:
  ports:
  - protocol: TCP
    port: 80           # Port on Service IP
    targetPort: 8080   # Port on Pod
    name: http         # Optional: name for this port
```

**Multiple ports:**
```yaml
spec:
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 8080
  - name: https
    protocol: TCP
    port: 443
    targetPort: 8443
  - name: metrics
    protocol: TCP
    port: 9090
    targetPort: 9090
```

**Named targetPort:**
```yaml
# In Pod:
spec:
  containers:
  - name: app
    ports:
    - name: web-port
      containerPort: 8080

# In Service:
spec:
  ports:
  - port: 80
    targetPort: web-port  # Reference by name
```

---

## 🚪 NodePort Service

Exposes service on each node's IP at a static port.

### **What is NodePort?**

```
NodePort = Every node listens on same port

┌──────────────────────────────────────────────┐
│  Node 1 (192.168.1.10)                       │
│    Listening on: 192.168.1.10:30080         │
└──────────────────────────────────────────────┘
         ↓
┌──────────────────────────────────────────────┐
│  Node 2 (192.168.1.11)                       │
│    Listening on: 192.168.1.11:30080         │
└──────────────────────────────────────────────┘
         ↓
    Service (10.96.0.50:80)
         ↓
    Backend Pods

Access from outside:
- http://192.168.1.10:30080  → Works! ✓
- http://192.168.1.11:30080  → Works! ✓
Both route to same service
```

---

### **NodePort Example:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: NodePort
  selector:
    app: web
  ports:
  - protocol: TCP
    port: 80          # Service port
    targetPort: 8080  # Pod port
    nodePort: 30080   # Port on each node (30000-32767)
```

**What gets created:**
```
1. ClusterIP: 10.96.0.50:80
   (Internal access)

2. NodePort: 30080 on ALL nodes
   Node1: 192.168.1.10:30080
   Node2: 192.168.1.11:30080
   Node3: 192.168.1.12:30080

3. All route to same backend pods
```

---

### **NodePort Traffic Flow:**

```
External User → 192.168.1.10:30080
                      ↓
          Node 1 receives traffic on port 30080
                      ↓
          kube-proxy on Node 1:
          "30080 → web-service → backend pods"
                      ↓
          Selects pod: web-pod-2 on Node 2
                      ↓
          Forwards: 192.168.1.11:8080
                      ↓
          web-pod-2 processes request
                      ↓
          Response back to user
```

**Key point:**
```
Request to Node 1:30080
Can route to pod on Node 2!

NodePort just exposes service
Actual pod can be on any node
```

---

### **NodePort Range:**

```
Default range: 30000-32767

Auto-assign:
  nodePort field omitted
  Kubernetes picks random port

Manual assign:
  nodePort: 30080
  Must be in range 30000-32767
  Must be unique across cluster
```

---

### **When to use NodePort:**

```
✓ Development/testing
✓ On-premise clusters (no cloud LB)
✓ Custom load balancer setup
✓ Direct node access available

✗ Production (use LoadBalancer instead)
✗ Need automatic DNS
✗ Cloud environments
```

---

## ☁️ LoadBalancer Service

Provisions cloud provider's load balancer.

### **What is LoadBalancer?**

```
LoadBalancer = Cloud provider's L4 load balancer

Cloud Provider (AWS, Azure, GCP):
  Creates: External Load Balancer
  Assigns: Public IP
  Routes: To your cluster nodes
  
┌─────────────────────────────────────┐
│  Cloud Load Balancer                │
│  Public IP: 52.123.45.67            │
│  (Managed by cloud provider)        │
└─────────────────────────────────────┘
          ↓
┌─────────────────────────────────────┐
│  Kubernetes Cluster                  │
│  NodePort Service: 30080             │
│         ↓                            │
│  Backend Pods                        │
└─────────────────────────────────────┘

Users access: http://52.123.45.67
```

---

### **LoadBalancer Example:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

**What gets created:**
```
1. Cloud load balancer provisioned
   Public IP: 52.123.45.67
   (Takes 1-2 minutes)

2. NodePort created: 30080
   (Automatic)

3. ClusterIP created: 10.96.0.50
   (Automatic)

4. Cloud LB → NodePort → Service → Pods
```

---

### **LoadBalancer Traffic Flow:**

```
Internet User → http://52.123.45.67:80
                      ↓
        Cloud Load Balancer (AWS ELB/ALB)
        Performs health checks on nodes
        Distributes traffic
                      ↓
        Node 1:30080 or Node 2:30080 or Node 3:30080
                      ↓
        Service: web-service (10.96.0.50:80)
                      ↓
        Backend Pods (round robin)
                      ↓
        Response back through same path
```

---

### **Checking LoadBalancer Status:**

```bash
kubectl get service web-service

NAME          TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)
web-service   LoadBalancer   10.96.0.50     52.123.45.67     80:30080/TCP
                                             ↑
                                    Public IP assigned!

# If pending:
NAME          TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)
web-service   LoadBalancer   10.96.0.50     <pending>     80:30080/TCP
                                             ↑
                              Waiting for cloud provider...
```

---

### **Cloud Provider Examples:**

**AWS (Elastic Load Balancer):**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"  # Network LB
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
```

**Azure (Azure Load Balancer):**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-internal: "false"  # Public
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
```

**GCP (Cloud Load Balancer):**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
  annotations:
    cloud.google.com/load-balancer-type: "External"
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
```

---

## 🔗 ExternalName Service

Creates DNS alias to external service.

### **What is ExternalName?**

```
ExternalName = DNS CNAME record

Your Pod: "Connect to database-service"
DNS: "database-service" → "prod-db.example.com"
Pod connects to: prod-db.example.com

Use case: External database/service
```

---

### **ExternalName Example:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-database
spec:
  type: ExternalName
  externalName: prod-db.company.com  # External DNS name
```

**How pods use it:**
```yaml
# Pod configuration
spec:
  containers:
  - name: app
    env:
    - name: DATABASE_HOST
      value: external-database  # Service name
    - name: DATABASE_PORT
      value: "5432"
```

**What happens:**
```
1. App tries to connect: external-database:5432

2. DNS query: external-database
   Response: prod-db.company.com

3. App actually connects: prod-db.company.com:5432

4. No load balancing, no proxying
   Just DNS resolution
```

---

### **When to use ExternalName:**

```
✓ External managed database (RDS, Cloud SQL)
✓ Third-party APIs
✓ Services in different clusters
✓ Migration from external to internal service

Example:
Start: ExternalName → prod-db.aws.com
Later: Change to ClusterIP → internal-db
       (Pods don't need to change!)
```

---

## 👻 Headless Service

Service without ClusterIP - returns pod IPs directly.

### **What is Headless Service?**

```
Normal Service:
  DNS: myapp → 10.96.0.50 (Service IP)
  
Headless Service:
  DNS: myapp → 10.244.1.5, 10.244.2.8, 10.244.3.2 (Pod IPs)
  
Returns all pod IPs directly
No load balancing by service
Client chooses which pod to connect to
```

---

### **Headless Service Example:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: stateful-service
spec:
  clusterIP: None  # Makes it headless!
  selector:
    app: stateful
  ports:
  - port: 3306
    targetPort: 3306
```

**DNS Resolution:**
```bash
# Normal service:
nslookup myapp
Answer: 10.96.0.50  (Service IP)

# Headless service:
nslookup stateful-service
Answer: 10.244.1.5   (Pod 1 IP)
        10.244.2.8   (Pod 2 IP)
        10.244.3.2   (Pod 3 IP)
```

---

### **StatefulSet with Headless Service:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: cassandra
spec:
  clusterIP: None  # Headless
  selector:
    app: cassandra
  ports:
  - port: 9042

---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: cassandra
spec:
  serviceName: cassandra  # References headless service
  replicas: 3
  selector:
    matchLabels:
      app: cassandra
  template:
    metadata:
      labels:
        app: cassandra
    spec:
      containers:
      - name: cassandra
        image: cassandra:4.0
```

**DNS Names (Predictable!):**
```
cassandra-0.cassandra.default.svc.cluster.local → 10.244.1.5
cassandra-1.cassandra.default.svc.cluster.local → 10.244.2.8
cassandra-2.cassandra.default.svc.cluster.local → 10.244.3.2

Pods can directly address each other:
cassandra-0 → cassandra-1
```

---

### **When to use Headless Service:**

```
✓ StatefulSets (databases, Kafka, etc.)
✓ Need to connect to specific pod
✓ Peer discovery in clustered apps
✓ Client-side load balancing

✗ Stateless web apps
✗ Need automatic load balancing
```

---

## 🔍 Service Discovery

How pods find services.

### **Two Methods:**

**1. Environment Variables:**
```
When pod starts, Kubernetes injects variables:

BACKEND_SERVICE_HOST=10.96.0.50
BACKEND_SERVICE_PORT=80

Your app:
host = os.environ['BACKEND_SERVICE_HOST']
```

**Limitation:**
```
Service must exist BEFORE pod starts
Pod restart needed if service created later
```

---

**2. DNS (Recommended):**

```
Kubernetes DNS (CoreDNS):
  Automatically creates DNS records for services

Service: backend-service
  Namespace: default
  
DNS Names:
  backend-service                              (same namespace)
  backend-service.default                      (with namespace)
  backend-service.default.svc.cluster.local    (fully qualified)
```

**DNS Resolution Example:**
```bash
# From any pod:
curl http://backend-service          # Same namespace
curl http://backend-service.default  # Explicit namespace
curl http://backend-service.production  # Different namespace

# DNS resolves to ClusterIP:
backend-service → 10.96.0.50
```

---

### **Cross-Namespace Access:**

```
┌─────────────────────────────────┐
│  Namespace: frontend             │
│                                  │
│  Pod: web                        │
│  Wants to call backend           │
│                                  │
│  curl http://api.backend:80      │
│         ↓                        │
│  DNS: api.backend → 10.96.0.50  │
└─────────────────────────────────┘
         ↓
┌─────────────────────────────────┐
│  Namespace: backend              │
│                                  │
│  Service: api (10.96.0.50)      │
│        ↓                         │
│  Pods: 3 backend pods            │
└─────────────────────────────────┘

Format: <service-name>.<namespace-name>
```

---

## 🎯 Endpoints

Endpoints track which pod IPs are behind a service.

### **What are Endpoints?**

```
Service: backend-service
  Selector: app=backend
  
Endpoints object (automatically created):
  backend-service endpoints:
    - 10.244.1.5:8080 (backend-pod-1)
    - 10.244.2.8:8080 (backend-pod-2)
    - 10.244.3.2:8080 (backend-pod-3)

When pod added/removed:
  Endpoints automatically updated
```

---

### **Viewing Endpoints:**

```bash
# Get endpoints for service
kubectl get endpoints backend-service

NAME              ENDPOINTS
backend-service   10.244.1.5:8080,10.244.2.8:8080,10.244.3.2:8080

# Detailed view
kubectl describe endpoints backend-service

Name:         backend-service
Namespace:    default
Subsets:
  Addresses:
    - IP: 10.244.1.5
      TargetRef:
        Kind: Pod
        Name: backend-pod-1
    - IP: 10.244.2.8
      TargetRef:
        Kind: Pod
        Name: backend-pod-2
  Ports:
    Port: 8080
```

---

### **Troubleshooting with Endpoints:**

**Problem: Service not working**
```bash
# Check service
kubectl get service myapp
NAME    TYPE        CLUSTER-IP    PORT(S)
myapp   ClusterIP   10.96.0.50    80/TCP

# Check endpoints
kubectl get endpoints myapp
NAME    ENDPOINTS
myapp   <none>       # No endpoints! Problem!

# Why no endpoints?
# 1. Check selector
kubectl describe service myapp
Selector: app=myapp

# 2. Check if pods match selector
kubectl get pods -l app=myapp
No resources found.  # No matching pods!

# Solution: Deploy pods with correct labels!
```

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

