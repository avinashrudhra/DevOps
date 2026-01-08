# Kubernetes StatefulSets - Complete Theoretical Guide

**Understanding Stateful Application Management**

---

## 📚 Table of Contents

1. [What is a StatefulSet?](#what-is-statefulset)
2. [StatefulSet vs Deployment](#statefulset-vs-deployment)
3. [Key Features](#key-features)
4. [Pod Identity](#pod-identity)
5. [Ordered Operations](#ordered-operations)
6. [Persistent Storage](#persistent-storage)
7. [Headless Service](#headless-service)
8. [Scaling](#scaling)
9. [Updates](#updates)

---

## 🎯 What is a StatefulSet?

A **StatefulSet** manages stateful applications that require stable network identity, stable persistent storage, and ordered deployment/scaling.

### **Simple Analogy:**

```
Deployment = Hotel Rooms
  - Guests come and go
  - Any room is fine
  - Room 101 = Room 102 (interchangeable)
  - Check in/out in any order

StatefulSet = Assigned Lockers
  - Each person has specific locker
  - Locker 1 always for Person 1
  - Locker number doesn't change
  - Open in order: 1, 2, 3
  - Close in reverse: 3, 2, 1
```

---

### **When to Use:**

```
✅ Use StatefulSet:
  - Databases (MySQL, PostgreSQL, MongoDB)
  - Distributed systems (Kafka, Cassandra, Elasticsearch)
  - Applications needing:
    • Stable pod name
    • Stable network identity
    • Stable persistent storage
    • Ordered deployment/scaling

✅ Use Deployment:
  - Stateless applications
  - Web servers
  - API services
  - Any pod can replace another
```

---

## 🆚 StatefulSet vs Deployment

### **Comparison:**

```
Deployment Pods:
  webapp-5d4f7c8b9d-abc12  (random hash)
  webapp-5d4f7c8b9d-def34  (random hash)
  webapp-5d4f7c8b9d-ghi56  (random hash)
  
  - Random names
  - Any pod interchangeable
  - Scale up/down: random order
  - No persistent pod identity

StatefulSet Pods:
  cassandra-0  (stable name)
  cassandra-1  (stable name)
  cassandra-2  (stable name)
  
  - Predictable names (0, 1, 2...)
  - Each pod unique
  - Ordered operations (0 → 1 → 2)
  - Persistent identity
```

---

### **Key Differences:**

| Feature | Deployment | StatefulSet |
|---------|------------|-------------|
| Pod names | Random | Ordered (0, 1, 2...) |
| Network identity | No stability | Stable DNS name |
| Storage | Shared or none | Per-pod persistent volume |
| Startup order | Parallel | Sequential |
| Scaling | Random | Ordered |
| Updates | Rolling | Ordered |
| Use case | Stateless | Stateful |

---

## 🔑 Key Features

### **1. Stable Pod Identity:**

```
StatefulSet: database

Pods:
  database-0  → Always database-0
  database-1  → Always database-1
  database-2  → Always database-2

Pod database-1 crashes:
  Replacement also named: database-1
  Same network identity
  Same persistent storage
  
Identity persists across:
  - Container restarts
  - Pod rescheduling
  - Node failures
```

---

### **2. Stable Network Identity:**

```
Each pod gets stable DNS name:
  <pod-name>.<service-name>.<namespace>.svc.cluster.local

Example:
  StatefulSet: cassandra
  Headless Service: cassandra
  Namespace: default

Pod DNS names:
  cassandra-0.cassandra.default.svc.cluster.local
  cassandra-1.cassandra.default.svc.cluster.local
  cassandra-2.cassandra.default.svc.cluster.local

Pods can directly address each other:
  cassandra-0 connects to cassandra-1
  Even if cassandra-1 dies and recreates
  DNS name remains the same!
```

---

### **3. Ordered Deployment:**

```
Creating StatefulSet with 3 replicas:

T+0s:   Create cassandra-0
T+15s:  cassandra-0 Running & Ready ✓
T+16s:  Create cassandra-1 (waits for 0)
T+31s:  cassandra-1 Running & Ready ✓
T+32s:  Create cassandra-2 (waits for 1)
T+47s:  cassandra-2 Running & Ready ✓

Sequential: 0 → 1 → 2
Each waits for previous to be Ready
```

---

### **4. Ordered Scaling:**

**Scale Up (2 → 5):**
```
Current: cassandra-0, cassandra-1
Add:     cassandra-2, cassandra-3, cassandra-4

Sequential:
  Create cassandra-2 → Wait Ready ✓
  Create cassandra-3 → Wait Ready ✓
  Create cassandra-4 → Wait Ready ✓
```

**Scale Down (5 → 2):**
```
Current: 0, 1, 2, 3, 4
Remove:  4, 3, 2 (reverse order!)

Sequential:
  Delete cassandra-4 → Wait Terminated ✓
  Delete cassandra-3 → Wait Terminated ✓
  Delete cassandra-2 → Wait Terminated ✓
  
Remaining: cassandra-0, cassandra-1
```

---

### **5. Persistent Per-Pod Storage:**

```
Each pod gets its own PersistentVolumeClaim:

database-0 → database-data-database-0 (10Gi)
database-1 → database-data-database-1 (10Gi)
database-2 → database-data-database-2 (10Gi)

Pod database-1 deleted and recreated:
  New database-1 pod
  Still uses database-data-database-1
  Data persists! ✓
```

---

## 🆔 Pod Identity

### **Predictable Naming:**

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  replicas: 3
  serviceName: "web-service"
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
```

**Resulting pods:**
```bash
kubectl get pods

NAME    READY   STATUS    AGE
web-0   1/1     Running   2m
web-1   1/1     Running   1m
web-2   1/1     Running   30s

# Names: <statefulset-name>-<ordinal>
# Ordinals: 0, 1, 2, 3...
```

---

### **Stable Storage:**

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:14
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
  
  # PVC template - creates PVC per pod
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
```

**What gets created:**
```
Pods:                        PVCs:
postgres-0  →  data-postgres-0 (10Gi)
postgres-1  →  data-postgres-1 (10Gi)
postgres-2  →  data-postgres-2 (10Gi)

Each pod has dedicated PVC
PVCs persist even if pods deleted
```

---

## 📋 Ordered Operations

### **Deployment Guarantees:**

```
StatefulSet guarantees:

1. Pods created sequentially: 0, then 1, then 2...
2. Pod N starts only after Pod N-1 is Running & Ready
3. Pods deleted in reverse: ...2, then 1, then 0
4. Pod N deleted only after Pod N+1 is terminated
```

---

### **Creation Example:**

```
StatefulSet: kafka
Replicas: 3

Timeline:
T+0s:   Create kafka-0
        Status: Pending

T+5s:   kafka-0: ContainerCreating

T+15s:  kafka-0: Running
        Readiness probe not yet passed

T+20s:  kafka-0: Ready ✓
        Now create kafka-1

T+21s:  Create kafka-1
        Status: Pending

T+26s:  kafka-1: ContainerCreating

T+36s:  kafka-1: Running & Ready ✓
        Now create kafka-2

T+37s:  Create kafka-2
        ...process continues

T+60s:  All 3 pods Running & Ready ✓
```

---

### **Deletion Example:**

```
kubectl delete statefulset kafka

Timeline:
T+0s:   Start deletion

T+1s:   Delete kafka-2 (highest ordinal first)
        kafka-2: Terminating

T+31s:  kafka-2: Terminated ✓
        Now delete kafka-1

T+32s:  Delete kafka-1
        kafka-1: Terminating

T+62s:  kafka-1: Terminated ✓
        Now delete kafka-0

T+63s:  Delete kafka-0
        kafka-0: Terminating

T+93s:  kafka-0: Terminated ✓
        StatefulSet deleted

Reverse order: 2 → 1 → 0
```

---

## 💾 Persistent Storage

### **volumeClaimTemplates:**

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongodb
spec:
  serviceName: mongodb
  replicas: 3
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo:5.0
        volumeMounts:
        - name: mongo-data
          mountPath: /data/db
  
  volumeClaimTemplates:
  - metadata:
      name: mongo-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: fast-ssd
      resources:
        requests:
          storage: 20Gi
```

---

### **PVC Creation:**

```
StatefulSet: mongodb (3 replicas)

Automatically creates PVCs:
  mongo-data-mongodb-0 (20Gi, ReadWriteOnce)
  mongo-data-mongodb-1 (20Gi, ReadWriteOnce)
  mongo-data-mongodb-2 (20Gi, ReadWriteOnce)

Each PVC bound to a PV:
  PV-001 ← PVC mongo-data-mongodb-0 ← Pod mongodb-0
  PV-002 ← PVC mongo-data-mongodb-1 ← Pod mongodb-1
  PV-003 ← PVC mongo-data-mongodb-2 ← Pod mongodb-2
```

---

### **PVC Lifecycle:**

```
Scenario: Pod mongodb-1 deleted

T+0s:   Delete pod mongodb-1
T+30s:  Pod deleted
        PVC mongo-data-mongodb-1 remains! ✓

Later: StatefulSet recreates pod

T+60s:  New pod mongodb-1 created
T+61s:  Binds to existing PVC: mongo-data-mongodb-1
T+62s:  Mounts existing PV
T+63s:  All data intact! ✓

PVCs are NOT deleted when pods deleted
Allows data to persist across pod recreation
```

---

### **⚠️ Deleting StatefulSet:**

```bash
# Delete StatefulSet
kubectl delete statefulset mongodb

Result:
  - Pods deleted ✓
  - StatefulSet deleted ✓
  - PVCs remain! (Not auto-deleted)

# View PVCs
kubectl get pvc

NAME                    STATUS   VOLUME   CAPACITY
mongo-data-mongodb-0    Bound    pv-001   20Gi
mongo-data-mongodb-1    Bound    pv-002   20Gi
mongo-data-mongodb-2    Bound    pv-003   20Gi

# Manual cleanup required
kubectl delete pvc mongo-data-mongodb-0 mongo-data-mongodb-1 mongo-data-mongodb-2
```

---

## 🌐 Headless Service

StatefulSets require a headless service for stable network identity.

### **Why Headless Service?**

```
Normal Service:
  DNS: mongodb → 10.96.0.50 (ClusterIP)
  Load balances to any pod

Headless Service (clusterIP: None):
  DNS: mongodb → All pod IPs
  Individual pod DNS:
    mongodb-0.mongodb → Direct to mongodb-0
    mongodb-1.mongodb → Direct to mongodb-1
    mongodb-2.mongodb → Direct to mongodb-2

Stateful apps need to address specific pods
Example: MongoDB replica set needs to connect to primary
```

---

### **Headless Service Example:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongodb
spec:
  clusterIP: None  # Makes it headless
  selector:
    app: mongodb
  ports:
  - port: 27017
    name: mongodb

---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongodb
spec:
  serviceName: mongodb  # Reference to headless service
  replicas: 3
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo:5.0
```

---

### **DNS Resolution:**

```bash
# From any pod:

# Query headless service
nslookup mongodb.default.svc.cluster.local

# Returns all pod IPs:
Server: 10.96.0.10
Address: 10.96.0.10:53

Name: mongodb.default.svc.cluster.local
Address: 10.244.1.5  (mongodb-0)
Address: 10.244.2.8  (mongodb-1)
Address: 10.244.3.2  (mongodb-2)

# Query specific pod
nslookup mongodb-0.mongodb.default.svc.cluster.local

# Returns specific pod IP:
Name: mongodb-0.mongodb.default.svc.cluster.local
Address: 10.244.1.5  (only mongodb-0)
```

---

## 📊 Scaling

### **Scale Up:**

```bash
# Scale from 3 to 5
kubectl scale statefulset mongodb --replicas=5

# What happens:
T+0s:   Current: mongodb-0, mongodb-1, mongodb-2
        Desired: 5 replicas

T+1s:   Create mongodb-3
T+20s:  mongodb-3 Running & Ready ✓

T+21s:  Create mongodb-4 (waits for 3)
T+40s:  mongodb-4 Running & Ready ✓

T+41s:  Scaling complete
        Pods: mongodb-0, 1, 2, 3, 4

PVCs created:
  mongo-data-mongodb-3
  mongo-data-mongodb-4
```

---

### **Scale Down:**

```bash
# Scale from 5 to 3
kubectl scale statefulset mongodb --replicas=3

# What happens:
T+0s:   Current: 0, 1, 2, 3, 4
        Desired: 3 replicas

T+1s:   Delete mongodb-4 (highest first)
T+31s:  mongodb-4 terminated ✓

T+32s:  Delete mongodb-3
T+62s:  mongodb-3 terminated ✓

T+63s:  Scaling complete
        Pods: mongodb-0, 1, 2

⚠️ PVCs NOT deleted:
  mongo-data-mongodb-3 (orphaned)
  mongo-data-mongodb-4 (orphaned)
  
Scale back up: Pods reattach to PVCs ✓
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
      partition: 0  # Update all pods
```

**Update process:**
```
Current: 0, 1, 2 (image: mongo:5.0)
Update:  image: mongo:6.0

Reverse order (highest first):
T+0s:   Delete mongodb-2
T+30s:  Recreate mongodb-2 with mongo:6.0
T+50s:  mongodb-2 Ready ✓

T+51s:  Delete mongodb-1
T+81s:  Recreate mongodb-1 with mongo:6.0
T+101s: mongodb-1 Ready ✓

T+102s: Delete mongodb-0
T+132s: Recreate mongodb-0 with mongo:6.0
T+152s: mongodb-0 Ready ✓

Update complete: 2 → 1 → 0
```

---

**2. Partitioned Updates:**
```yaml
spec:
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      partition: 2  # Only update pods with ordinal >= 2
```

**What happens:**
```
Pods: 0, 1, 2, 3, 4 (all mongo:5.0)
Update to: mongo:6.0
Partition: 2

Updates:
  mongodb-4 → mongo:6.0 ✓
  mongodb-3 → mongo:6.0 ✓
  mongodb-2 → mongo:6.0 ✓
  mongodb-1 → mongo:5.0 (no update, < partition)
  mongodb-0 → mongo:5.0 (no update, < partition)

Result:
  0, 1: Old version (5.0)
  2, 3, 4: New version (6.0)

Use case: Canary deployment
  Test new version on some pods
  Then set partition=0 to update all
```

---

**3. OnDelete:**
```yaml
spec:
  updateStrategy:
    type: OnDelete
```

**What happens:**
```
Update spec (change image)
Pods: No automatic update

Manual deletion:
  kubectl delete pod mongodb-2
  Pod recreated with new image ✓

Gives full manual control over updates
```

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

