# Kubernetes: Beginner to Production Debug Expert
## Complete Learning Roadmap

> Based on official documentation from [kubernetes.io](https://kubernetes.io/)

---

## Table of Contents
1. [Introduction & Prerequisites](#introduction--prerequisites)
2. [Level 1: Beginner (Weeks 1-4)](#level-1-beginner-weeks-1-4)
3. [Level 2: Intermediate (Weeks 5-12)](#level-2-intermediate-weeks-5-12)
4. [Level 3: Advanced (Weeks 13-24)](#level-3-advanced-weeks-13-24)
5. [Level 4: Production & Debugging Expert (Weeks 25-52)](#level-4-production--debugging-expert-weeks-25-52)
6. [Practical Projects](#practical-projects)
7. [Certification Path](#certification-path)
8. [Resources & Tools](#resources--tools)

---

## Introduction & Prerequisites

### What is Kubernetes?
Kubernetes (K8s) is an **open-source system for automating deployment, scaling, and management of containerized applications**. It builds upon 15 years of experience running production workloads at Google.

### Key Benefits:
- **Planet Scale**: Designed to run billions of containers
- **Flexibility**: Run anywhere (on-premises, hybrid, or public cloud)
- **Never Outgrow**: Scales with your needs

### Prerequisites:
- [ ] Basic Linux command line skills
- [ ] Understanding of networking basics (IP, DNS, ports)
- [ ] Familiarity with Docker or containers
- [ ] Basic YAML syntax knowledge
- [ ] Git version control basics

---

## Level 1: Beginner (Weeks 1-4)

### Week 1: Foundation & Setup

#### 1.1 Understanding Containers
- **What are containers?**
  - Isolated, lightweight runtime environments
  - Package applications with all dependencies
  - Difference between containers and VMs

- **Docker Basics** (if not familiar)
  ```bash
  # Essential Docker commands
  docker run nginx
  docker ps
  docker images
  docker build -t myapp .
  docker stop <container-id>
  ```

#### 1.2 Kubernetes Architecture
- **Master Components (Control Plane)**:
  - `kube-apiserver`: Front-end for the Kubernetes control plane
  - `etcd`: Key-value store for all cluster data
  - `kube-scheduler`: Assigns pods to nodes
  - `kube-controller-manager`: Runs controller processes
  - `cloud-controller-manager`: Interacts with cloud providers

- **Node Components (Worker Nodes)**:
  - `kubelet`: Agent running on each node
  - `kube-proxy`: Network proxy
  - `container runtime`: Docker, containerd, CRI-O

#### 1.3 Setup Local Kubernetes
Choose one option:

**Option A: Minikube** (Recommended for beginners)
```bash
# Install Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-windows-amd64.exe
# Start cluster
minikube start
minikube status
```

**Option B: Docker Desktop**
- Enable Kubernetes in Docker Desktop settings

**Option C: Kind (Kubernetes in Docker)**
```bash
kind create cluster --name my-cluster
```

#### 1.4 Install kubectl
```bash
# Download kubectl
curl -LO "https://dl.k8s.io/release/v1.33.0/bin/windows/amd64/kubectl.exe"

# Verify installation
kubectl version --client
```

**✅ Week 1 Checkpoint**: You should have a local Kubernetes cluster running and kubectl installed.

---

### Week 2: Core Concepts

#### 2.1 Pods - The Smallest Deployable Unit
```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

```bash
# Create pod
kubectl apply -f pod.yaml

# List pods
kubectl get pods

# Describe pod
kubectl describe pod nginx-pod

# View logs
kubectl logs nginx-pod

# Execute command in pod
kubectl exec -it nginx-pod -- /bin/bash

# Delete pod
kubectl delete pod nginx-pod
```

#### 2.2 Essential kubectl Commands
```bash
# Get resources
kubectl get pods
kubectl get pods -o wide
kubectl get pods --all-namespaces
kubectl get all

# Describe resources (detailed info)
kubectl describe pod <pod-name>
kubectl describe node <node-name>

# Create resources
kubectl apply -f <file.yaml>
kubectl create deployment nginx --image=nginx

# Delete resources
kubectl delete pod <pod-name>
kubectl delete -f <file.yaml>

# Edit resources
kubectl edit pod <pod-name>

# Port forwarding
kubectl port-forward pod/nginx-pod 8080:80
```

#### 2.3 Namespaces
```bash
# List namespaces
kubectl get namespaces

# Create namespace
kubectl create namespace dev
kubectl create namespace prod

# Use namespace
kubectl get pods -n dev
kubectl apply -f pod.yaml -n dev

# Set default namespace
kubectl config set-context --current --namespace=dev
```

```yaml
# namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: development
```

#### 2.4 Labels & Selectors
```yaml
# pod-with-labels.yaml
apiVersion: v1
kind: Pod
metadata:
  name: labeled-pod
  labels:
    environment: production
    app: web
    tier: frontend
spec:
  containers:
  - name: nginx
    image: nginx
```

```bash
# Query by labels
kubectl get pods -l environment=production
kubectl get pods -l 'environment in (production, staging)'
kubectl get pods -l app=web,tier=frontend
```

**✅ Week 2 Checkpoint**: Create, manage, and query pods using labels and namespaces.

---

### Week 3: Workload Resources

#### 3.1 ReplicaSets
```yaml
# replicaset.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

```bash
kubectl apply -f replicaset.yaml
kubectl get rs
kubectl describe rs frontend
kubectl scale rs frontend --replicas=5
```

#### 3.2 Deployments (Recommended over ReplicaSets)
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
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
        image: nginx:1.16.0
        ports:
        - containerPort: 80
```

```bash
# Create deployment
kubectl apply -f deployment.yaml

# Check deployment
kubectl get deployments
kubectl get rs
kubectl get pods

# Scale deployment
kubectl scale deployment nginx-deployment --replicas=5

# Update image (rolling update)
kubectl set image deployment/nginx-deployment nginx=nginx:1.17.0

# Check rollout status
kubectl rollout status deployment/nginx-deployment

# Rollout history
kubectl rollout history deployment/nginx-deployment

# Rollback
kubectl rollout undo deployment/nginx-deployment
kubectl rollout undo deployment/nginx-deployment --to-revision=2
```

#### 3.3 Services - Exposing Applications

**ClusterIP (default)** - Internal cluster access only
```yaml
# service-clusterip.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

**NodePort** - Exposes on each node's IP
```yaml
# service-nodeport.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080  # 30000-32767
```

**LoadBalancer** - Cloud provider load balancer
```yaml
# service-loadbalancer.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-lb
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

```bash
# Create service
kubectl apply -f service-clusterip.yaml

# Get services
kubectl get svc
kubectl describe svc nginx-service

# Test service (from within cluster)
kubectl run curl-pod --image=curlimages/curl -it --rm -- sh
curl http://nginx-service
```

**✅ Week 3 Checkpoint**: Deploy applications using Deployments and expose them via Services.

---

### Week 4: Configuration & Storage

#### 4.1 ConfigMaps - Configuration Management
```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "postgres://db.example.com:5432"
  log_level: "info"
  config.json: |
    {
      "key": "value",
      "nested": {
        "setting": "enabled"
      }
    }
```

```yaml
# pod-with-configmap.yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ['sh', '-c', 'echo "Database: $DATABASE_URL" && sleep 3600']
    env:
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database_url
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: app-config
```

```bash
# Create from literal
kubectl create configmap app-config --from-literal=key1=value1

# Create from file
kubectl create configmap app-config --from-file=config.txt

# Get configmap
kubectl get configmap
kubectl describe configmap app-config
```

#### 4.2 Secrets - Sensitive Data
```yaml
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  username: YWRtaW4=  # base64 encoded
  password: cGFzc3dvcmQxMjM=  # base64 encoded
```

```bash
# Create secret from literal
kubectl create secret generic app-secret \
  --from-literal=username=admin \
  --from-literal=password=password123

# Create secret from file
kubectl create secret generic ssh-key-secret \
  --from-file=ssh-privatekey=~/.ssh/id_rsa

# Get secrets
kubectl get secrets
kubectl describe secret app-secret

# Decode secret
kubectl get secret app-secret -o jsonpath='{.data.password}' | base64 --decode
```

```yaml
# pod-with-secret.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
  - name: app
    image: nginx
    env:
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: password
```

#### 4.3 Volumes - Persistent Storage

**EmptyDir** - Temporary storage
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-pod
spec:
  containers:
  - name: writer
    image: busybox
    command: ['sh', '-c', 'echo "Hello" > /data/hello.txt && sleep 3600']
    volumeMounts:
    - name: shared-data
      mountPath: /data
  - name: reader
    image: busybox
    command: ['sh', '-c', 'cat /data/hello.txt && sleep 3600']
    volumeMounts:
    - name: shared-data
      mountPath: /data
  volumes:
  - name: shared-data
    emptyDir: {}
```

**HostPath** - Mount from host
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: host-volume
      mountPath: /usr/share/nginx/html
  volumes:
  - name: host-volume
    hostPath:
      path: /data
      type: Directory
```

**PersistentVolume & PersistentVolumeClaim**
```yaml
# persistent-volume.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-volume
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data

---
# persistent-volume-claim.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi

---
# pod-with-pvc.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pvc-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: persistent-storage
      mountPath: /usr/share/nginx/html
  volumes:
  - name: persistent-storage
    persistentVolumeClaim:
      claimName: pv-claim
```

**✅ Week 4 Checkpoint**: Configure applications using ConfigMaps, Secrets, and persistent storage.

---

## Level 2: Intermediate (Weeks 5-12)

### Week 5-6: Advanced Workloads

#### 5.1 StatefulSets - For Stateful Applications
```yaml
# statefulset.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-headless
spec:
  clusterIP: None
  selector:
    app: nginx-stateful
  ports:
  - port: 80

---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx-headless"
  replicas: 3
  selector:
    matchLabels:
      app: nginx-stateful
  template:
    metadata:
      labels:
        app: nginx-stateful
    spec:
      containers:
      - name: nginx
        image: nginx:1.16.0
        ports:
        - containerPort: 80
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

**Key Features**:
- Stable network identity (predictable pod names: web-0, web-1, web-2)
- Ordered deployment and scaling
- Persistent storage per pod

```bash
kubectl apply -f statefulset.yaml
kubectl get statefulsets
kubectl get pods -l app=nginx-stateful
kubectl delete statefulset web --cascade=false  # Keep pods
```

#### 5.2 DaemonSets - One Pod Per Node
```yaml
# daemonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```

**Use Cases**: Monitoring agents, log collectors, storage daemons

#### 5.3 Jobs & CronJobs
```yaml
# job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-calculation
spec:
  completions: 5
  parallelism: 2
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```

```yaml
# cronjob.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup-job
spec:
  schedule: "0 2 * * *"  # Every day at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: busybox
            command:
            - /bin/sh
            - -c
            - date; echo "Running backup"
          restartPolicy: OnFailure
```

```bash
kubectl apply -f job.yaml
kubectl get jobs
kubectl logs job/pi-calculation

kubectl apply -f cronjob.yaml
kubectl get cronjobs
kubectl get jobs --watch
```

---

### Week 7-8: Networking Deep Dive

#### 7.1 Network Policies
```yaml
# network-policy-deny-all.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress

---
# network-policy-allow-specific.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```

#### 7.2 Ingress Controllers
```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 8080
  tls:
  - hosts:
    - myapp.example.com
    secretName: tls-secret
```

**Install Nginx Ingress Controller**:
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml

# Verify
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx
```

#### 7.3 Service Mesh (Introduction)
- **Istio**: Traffic management, security, observability
- **Linkerd**: Lightweight service mesh
- **Consul**: HashiCorp's service mesh

---

### Week 9-10: Resource Management & Scheduling

#### 9.1 Resource Requests & Limits
```yaml
# resource-constraints.yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

**Understanding Units**:
- CPU: `1` = 1 core, `100m` = 0.1 core (100 millicores)
- Memory: `128Mi` = 128 Mebibytes, `1Gi` = 1 Gibibyte

#### 9.2 LimitRanges
```yaml
# limitrange.yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
  namespace: default
spec:
  limits:
  - default:
      memory: 512Mi
      cpu: 1
    defaultRequest:
      memory: 256Mi
      cpu: 0.5
    max:
      memory: 1Gi
      cpu: 2
    min:
      memory: 128Mi
      cpu: 0.25
    type: Container
```

#### 9.3 ResourceQuotas
```yaml
# resourcequota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 20Gi
    limits.cpu: "20"
    limits.memory: 40Gi
    pods: "50"
    services: "10"
    persistentvolumeclaims: "20"
```

#### 9.4 Node Affinity & Anti-Affinity
```yaml
# node-affinity.yaml
apiVersion: v1
kind: Pod
metadata:
  name: affinity-pod
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
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: disk-type
            operator: In
            values:
            - ssd
  containers:
  - name: nginx
    image: nginx
```

#### 9.5 Taints & Tolerations
```bash
# Add taint to node
kubectl taint nodes node1 key=value:NoSchedule

# Remove taint
kubectl taint nodes node1 key=value:NoSchedule-
```

```yaml
# toleration.yaml
apiVersion: v1
kind: Pod
metadata:
  name: toleration-pod
spec:
  tolerations:
  - key: "key"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule"
  containers:
  - name: nginx
    image: nginx
```

#### 9.6 Pod Priority & Preemption
```yaml
# priority-class.yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: "High priority for critical applications"

---
# pod-with-priority.yaml
apiVersion: v1
kind: Pod
metadata:
  name: critical-pod
spec:
  priorityClassName: high-priority
  containers:
  - name: nginx
    image: nginx
```

---

### Week 11-12: Security Fundamentals

#### 11.1 Service Accounts
```yaml
# serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-service-account
  namespace: default

---
# pod-with-sa.yaml
apiVersion: v1
kind: Pod
metadata:
  name: sa-pod
spec:
  serviceAccountName: app-service-account
  containers:
  - name: app
    image: nginx
```

#### 11.2 RBAC (Role-Based Access Control)

**Role & RoleBinding** (namespace-scoped)
```yaml
# role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]

---
# rolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: ServiceAccount
  name: app-service-account
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

**ClusterRole & ClusterRoleBinding** (cluster-wide)
```yaml
# clusterrole.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list", "watch"]

---
# clusterrolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-nodes
subjects:
- kind: ServiceAccount
  name: app-service-account
  namespace: default
roleRef:
  kind: ClusterRole
  name: node-reader
  apiGroup: rbac.authorization.k8s.io
```

```bash
# Check permissions
kubectl auth can-i create deployments
kubectl auth can-i delete pods --namespace=dev
kubectl auth can-i get pods --as=system:serviceaccount:default:app-service-account
```

#### 11.3 Pod Security Standards
```yaml
# pod-security-policy.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: secure-namespace
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted

---
# secure-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
  namespace: secure-namespace
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: nginx
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
    volumeMounts:
    - name: cache-volume
      mountPath: /var/cache/nginx
    - name: run-volume
      mountPath: /var/run
  volumes:
  - name: cache-volume
    emptyDir: {}
  - name: run-volume
    emptyDir: {}
```

#### 11.4 Network Security
```yaml
# secure-network-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress

---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-db-access
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: backend
    - namespaceSelector:
        matchLabels:
          name: production
    ports:
    - protocol: TCP
      port: 5432
```

**✅ Intermediate Checkpoint**: You should now be able to deploy complex applications with proper resource management, networking, and security configurations.

---

## Level 3: Advanced (Weeks 13-24)

### Week 13-15: Advanced Deployment Strategies

#### 13.1 Rolling Updates
```yaml
# rolling-update.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rolling-deployment
spec:
  replicas: 10
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: myapp:v1
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
```

#### 13.2 Blue-Green Deployment
```yaml
# blue-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: blue
  template:
    metadata:
      labels:
        app: myapp
        version: blue
    spec:
      containers:
      - name: app
        image: myapp:v1

---
# green-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: green
  template:
    metadata:
      labels:
        app: myapp
        version: green
    spec:
      containers:
      - name: app
        image: myapp:v2

---
# service.yaml (switch traffic by changing selector)
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
    version: blue  # Change to 'green' to switch traffic
  ports:
  - port: 80
    targetPort: 8080
```

#### 13.3 Canary Deployment
```yaml
# stable-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-stable
spec:
  replicas: 9
  selector:
    matchLabels:
      app: myapp
      track: stable
  template:
    metadata:
      labels:
        app: myapp
        track: stable
    spec:
      containers:
      - name: app
        image: myapp:v1

---
# canary-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-canary
spec:
  replicas: 1  # 10% traffic
  selector:
    matchLabels:
      app: myapp
      track: canary
  template:
    metadata:
      labels:
        app: myapp
        track: canary
    spec:
      containers:
      - name: app
        image: myapp:v2

---
# service.yaml (routes to both)
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp  # Matches both stable and canary
  ports:
  - port: 80
    targetPort: 8080
```

---

### Week 16-18: Observability & Monitoring

#### 16.1 Liveness & Readiness Probes
```yaml
# probes.yaml
apiVersion: v1
kind: Pod
metadata:
  name: probe-demo
spec:
  containers:
  - name: app
    image: myapp:latest
    ports:
    - containerPort: 8080
    
    # Liveness probe - restart if fails
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 30
      periodSeconds: 10
      timeoutSeconds: 5
      failureThreshold: 3
    
    # Readiness probe - remove from service if fails
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 10
      periodSeconds: 5
      timeoutSeconds: 3
      failureThreshold: 3
    
    # Startup probe - for slow-starting containers
    startupProbe:
      httpGet:
        path: /startup
        port: 8080
      initialDelaySeconds: 0
      periodSeconds: 10
      timeoutSeconds: 3
      failureThreshold: 30
```

**Probe Types**:
- `httpGet`: HTTP GET request
- `tcpSocket`: TCP socket check
- `exec`: Execute command in container

```yaml
# exec-probe.yaml
livenessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 5
```

#### 16.2 Metrics Server
```bash
# Install Metrics Server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# View resource usage
kubectl top nodes
kubectl top pods
kubectl top pods -n kube-system
kubectl top pods --all-namespaces
```

#### 16.3 Prometheus & Grafana Setup
```bash
# Using Helm
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install Prometheus
helm install prometheus prometheus-community/prometheus

# Install Grafana
helm repo add grafana https://grafana.github.io/helm-charts
helm install grafana grafana/grafana

# Get Grafana password
kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode

# Port forward Grafana
kubectl port-forward service/grafana 3000:80
```

**Prometheus ServiceMonitor**:
```yaml
# servicemonitor.yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: app-monitor
spec:
  selector:
    matchLabels:
      app: myapp
  endpoints:
  - port: metrics
    interval: 30s
```

#### 16.4 Logging with EFK Stack
**Elasticsearch, Fluentd, Kibana**

```yaml
# fluentd-daemonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-system
spec:
  selector:
    matchLabels:
      k8s-app: fluentd-logging
  template:
    metadata:
      labels:
        k8s-app: fluentd-logging
    spec:
      serviceAccount: fluentd
      serviceAccountName: fluentd
      containers:
      - name: fluentd
        image: fluent/fluentd-kubernetes-daemonset:v1-debian-elasticsearch
        env:
        - name: FLUENT_ELASTICSEARCH_HOST
          value: "elasticsearch.kube-system.svc.cluster.local"
        - name: FLUENT_ELASTICSEARCH_PORT
          value: "9200"
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

---

### Week 19-21: Helm Package Manager

#### 19.1 Install Helm
```bash
# Windows (PowerShell)
choco install kubernetes-helm

# Or download from https://github.com/helm/helm/releases
```

#### 19.2 Basic Helm Commands
```bash
# Add repository
helm repo add stable https://charts.helm.sh/stable
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Search charts
helm search repo nginx
helm search hub wordpress

# Install chart
helm install my-nginx bitnami/nginx
helm install my-release stable/mysql --set mysqlRootPassword=secretpassword

# List releases
helm list
helm list --all-namespaces

# Get values
helm show values bitnami/nginx

# Upgrade release
helm upgrade my-nginx bitnami/nginx --set replicaCount=3

# Rollback
helm rollback my-nginx 1

# Uninstall
helm uninstall my-nginx

# Get release info
helm status my-nginx
helm get values my-nginx
helm get manifest my-nginx
```

#### 19.3 Create Custom Helm Chart
```bash
# Create chart
helm create myapp

# Chart structure
myapp/
├── Chart.yaml          # Chart metadata
├── values.yaml         # Default configuration values
├── charts/             # Dependency charts
└── templates/          # Template files
    ├── deployment.yaml
    ├── service.yaml
    ├── ingress.yaml
    ├── _helpers.tpl
    └── NOTES.txt
```

**Chart.yaml**:
```yaml
apiVersion: v2
name: myapp
description: A Helm chart for my application
type: application
version: 0.1.0
appVersion: "1.0"
```

**values.yaml**:
```yaml
replicaCount: 3

image:
  repository: myapp
  pullPolicy: IfNotPresent
  tag: "latest"

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: false
  className: "nginx"
  hosts:
    - host: myapp.local
      paths:
        - path: /
          pathType: Prefix

resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 128Mi

autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 100
  targetCPUUtilizationPercentage: 80
```

**templates/deployment.yaml**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "myapp.fullname" . }}
  labels:
    {{- include "myapp.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "myapp.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "myapp.selectorLabels" . | nindent 8 }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - name: http
          containerPort: 8080
          protocol: TCP
        resources:
          {{- toYaml .Values.resources | nindent 12 }}
```

```bash
# Install custom chart
helm install my-release ./myapp

# Install with custom values
helm install my-release ./myapp -f custom-values.yaml

# Dry run (test without installing)
helm install my-release ./myapp --dry-run --debug

# Package chart
helm package myapp

# Install from package
helm install my-release myapp-0.1.0.tgz
```

---

### Week 22-24: Multi-Cluster & Advanced Topics

#### 22.1 Multi-Cluster Management
```bash
# View contexts
kubectl config get-contexts

# Switch context
kubectl config use-context prod-cluster

# Create context
kubectl config set-context dev-context --cluster=dev --user=dev-user --namespace=development

# Merge kubeconfig files
KUBECONFIG=~/.kube/config:~/.kube/config-cluster2 kubectl config view --flatten > ~/.kube/config-merged
```

#### 22.2 Kubernetes Federation
- Manage multiple clusters as a single entity
- Cross-cluster service discovery
- Disaster recovery and high availability

#### 22.3 Custom Resource Definitions (CRDs)
```yaml
# crd-definition.yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: applications.example.com
spec:
  group: example.com
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              replicas:
                type: integer
              image:
                type: string
  scope: Namespaced
  names:
    plural: applications
    singular: application
    kind: Application
    shortNames:
    - app
```

```yaml
# custom-resource.yaml
apiVersion: example.com/v1
kind: Application
metadata:
  name: my-application
spec:
  replicas: 3
  image: nginx:1.16.0
```

```bash
kubectl apply -f crd-definition.yaml
kubectl apply -f custom-resource.yaml
kubectl get applications
```

#### 22.4 Operators
**Operator Pattern**: Extends Kubernetes to manage complex applications

**Example: PostgreSQL Operator**
```bash
# Install Operator Lifecycle Manager
curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.25.0/install.sh | bash -s v0.25.0

# Install Postgres Operator
kubectl create -f https://operatorhub.io/install/postgresql.yaml
```

#### 22.5 GitOps with ArgoCD
```bash
# Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Access ArgoCD UI
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Get admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

**ArgoCD Application**:
```yaml
# application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/myapp
    targetRevision: HEAD
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

---

## Level 4: Production & Debugging Expert (Weeks 25-52)

### Week 25-28: Production Best Practices

#### 25.1 High Availability Setup
```yaml
# ha-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ha-app
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: ha-app
  template:
    metadata:
      labels:
        app: ha-app
    spec:
      # Spread pods across nodes
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - ha-app
            topologyKey: kubernetes.io/hostname
      
      # Spread across availability zones
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: DoNotSchedule
        labelSelector:
          matchLabels:
            app: ha-app
      
      containers:
      - name: app
        image: myapp:v1
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 512Mi
        
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
        
        # Graceful shutdown
        lifecycle:
          preStop:
            exec:
              command: ["/bin/sh", "-c", "sleep 15"]
      
      # Termination grace period
      terminationGracePeriodSeconds: 30
```

#### 25.2 Pod Disruption Budgets
```yaml
# pdb.yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: app-pdb
spec:
  minAvailable: 2
  # OR: maxUnavailable: 1
  selector:
    matchLabels:
      app: ha-app
```

#### 25.3 Horizontal Pod Autoscaler (HPA)
```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ha-app
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 30
      - type: Pods
        value: 4
        periodSeconds: 60
      selectPolicy: Max
```

```bash
kubectl apply -f hpa.yaml
kubectl get hpa
kubectl describe hpa app-hpa
kubectl autoscale deployment ha-app --cpu-percent=70 --min=3 --max=10
```

#### 25.4 Vertical Pod Autoscaler (VPA)
```yaml
# vpa.yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ha-app
  updatePolicy:
    updateMode: "Auto"  # Auto, Recreate, Initial, Off
  resourcePolicy:
    containerPolicies:
    - containerName: app
      minAllowed:
        cpu: 100m
        memory: 128Mi
      maxAllowed:
        cpu: 2
        memory: 2Gi
```

#### 25.5 Cluster Autoscaler
```yaml
# cluster-autoscaler.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cluster-autoscaler
  template:
    metadata:
      labels:
        app: cluster-autoscaler
    spec:
      serviceAccountName: cluster-autoscaler
      containers:
      - image: k8s.gcr.io/autoscaling/cluster-autoscaler:v1.27.0
        name: cluster-autoscaler
        command:
        - ./cluster-autoscaler
        - --cloud-provider=aws  # or gce, azure, etc.
        - --namespace=kube-system
        - --nodes=1:10:k8s-worker-asg-1
        - --scale-down-enabled=true
        - --scale-down-delay-after-add=10m
        - --scale-down-unneeded-time=10m
```

---

### Week 29-35: Production Debugging

#### 29.1 Essential Debugging Commands
```bash
# ============================================
# POD DEBUGGING
# ============================================

# Get pod status
kubectl get pods
kubectl get pods -o wide
kubectl get pods --all-namespaces
kubectl get pods -o yaml

# Describe pod (events, conditions, config)
kubectl describe pod <pod-name>

# View logs
kubectl logs <pod-name>
kubectl logs <pod-name> -c <container-name>  # Multi-container pod
kubectl logs <pod-name> --previous  # Previous instance
kubectl logs <pod-name> --tail=50
kubectl logs <pod-name> --since=1h
kubectl logs <pod-name> -f  # Follow logs

# Execute commands in pod
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec -it <pod-name> -c <container-name> -- /bin/sh
kubectl exec <pod-name> -- env
kubectl exec <pod-name> -- ps aux
kubectl exec <pod-name> -- cat /etc/resolv.conf

# Copy files to/from pod
kubectl cp <pod-name>:/path/to/file /local/path
kubectl cp /local/file <pod-name>:/path/in/container

# Port forward
kubectl port-forward pod/<pod-name> 8080:80
kubectl port-forward svc/<service-name> 8080:80

# ============================================
# NODE DEBUGGING
# ============================================

# Get node status
kubectl get nodes
kubectl get nodes -o wide
kubectl describe node <node-name>
kubectl top nodes

# Node conditions
kubectl get nodes -o json | jq '.items[].status.conditions'

# Cordon/uncordon node (prevent scheduling)
kubectl cordon <node-name>
kubectl uncordon <node-name>

# Drain node (evict pods)
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data

# ============================================
# RESOURCE DEBUGGING
# ============================================

# Get all resources
kubectl get all
kubectl get all -n <namespace>
kubectl api-resources  # List all resource types

# Events
kubectl get events
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get events --field-selector involvedObject.name=<pod-name>

# Check resource quotas
kubectl describe resourcequota -n <namespace>
kubectl describe limitrange -n <namespace>

# ============================================
# NETWORK DEBUGGING
# ============================================

# Check service endpoints
kubectl get endpoints
kubectl describe svc <service-name>

# DNS debugging
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup kubernetes.default
kubectl run -it --rm debug --image=nicolaka/netshoot --restart=Never -- /bin/bash

# Inside netshoot container:
# nslookup <service-name>
# ping <pod-ip>
# curl http://<service-name>
# traceroute <ip>
# tcpdump -i any port 80

# ============================================
# ADVANCED DEBUGGING
# ============================================

# Debug with ephemeral container (K8s 1.23+)
kubectl debug <pod-name> -it --image=busybox --target=<container-name>

# Create debugging pod on same node
kubectl debug node/<node-name> -it --image=ubuntu

# Check API server
kubectl cluster-info
kubectl cluster-info dump

# Check component health
kubectl get componentstatuses  # Deprecated but useful

# Check certificates
kubectl get csr

# Verbose output
kubectl get pods -v=8  # Verbosity level 0-10
```

#### 29.2 Common Issues & Solutions

**Issue 1: Pod in CrashLoopBackOff**
```bash
# Check logs
kubectl logs <pod-name> --previous

# Check events
kubectl describe pod <pod-name>

# Common causes:
# - Application crash on startup
# - Missing configuration (ConfigMap/Secret)
# - Liveness probe failing too quickly
# - Insufficient resources
# - Wrong command/entrypoint

# Solutions:
# 1. Fix application code
# 2. Adjust livenessProbe initialDelaySeconds
# 3. Increase resource limits
# 4. Check ConfigMap/Secret exists
```

**Issue 2: Pod in ImagePullBackOff**
```bash
# Check events
kubectl describe pod <pod-name>

# Common causes:
# - Wrong image name/tag
# - Private registry without imagePullSecrets
# - Network issues
# - Registry authentication failure

# Solutions:
# Create imagePullSecret
kubectl create secret docker-registry regcred \
  --docker-server=<registry> \
  --docker-username=<username> \
  --docker-password=<password>

# Add to pod spec:
spec:
  imagePullSecrets:
  - name: regcred
```

**Issue 3: Pod Pending**
```bash
# Check why pod is pending
kubectl describe pod <pod-name>

# Common causes:
# - Insufficient CPU/memory on nodes
# - No nodes matching nodeSelector
# - PersistentVolumeClaim not bound
# - Taint on nodes without tolerations

# Check node resources
kubectl top nodes
kubectl describe nodes | grep -A 5 "Allocated resources"

# Check PVC
kubectl get pvc
kubectl describe pvc <pvc-name>
```

**Issue 4: Service Not Accessible**
```bash
# 1. Check service exists
kubectl get svc <service-name>
kubectl describe svc <service-name>

# 2. Check endpoints
kubectl get endpoints <service-name>
# If no endpoints, selector doesn't match any pods

# 3. Check pod labels match service selector
kubectl get pods --show-labels
kubectl get svc <service-name> -o jsonpath='{.spec.selector}'

# 4. Test connectivity from another pod
kubectl run -it --rm debug --image=busybox --restart=Never -- wget -O- http://<service-name>

# 5. Check network policies
kubectl get networkpolicies
```

**Issue 5: High Memory/CPU Usage**
```bash
# Check resource usage
kubectl top pods
kubectl top pods --containers
kubectl top nodes

# Get resource limits
kubectl get pods <pod-name> -o jsonpath='{.spec.containers[*].resources}'

# Analyze metrics
kubectl describe hpa
kubectl get --raw /apis/metrics.k8s.io/v1beta1/pods
```

#### 29.3 Production Debugging Scenarios

**Scenario 1: Application Latency Issues**
```bash
# 1. Check pod restarts
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.containerStatuses[*].restartCount}{"\n"}{end}'

# 2. Check readiness probe
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.conditions[?(@.type=="Ready")].status}{"\n"}{end}'

# 3. Check HPA status
kubectl get hpa
kubectl describe hpa <hpa-name>

# 4. Analyze pod distribution
kubectl get pods -o wide

# 5. Check for resource throttling
kubectl top pods --containers

# 6. Check network latency
kubectl run -it --rm debug --image=nicolaka/netshoot --restart=Never -- /bin/bash
# Inside: ping <service-ip>

# 7. Check application metrics (if Prometheus installed)
kubectl port-forward -n monitoring svc/prometheus 9090:9090
# Open http://localhost:9090
```

**Scenario 2: Database Connection Issues**
```bash
# 1. Check database pod
kubectl get pods -l app=database
kubectl logs <db-pod-name>

# 2. Check service and endpoints
kubectl get svc database-service
kubectl get endpoints database-service

# 3. Test connectivity from app pod
kubectl exec -it <app-pod> -- nc -zv database-service 5432

# 4. Check secrets
kubectl get secret db-secret -o yaml

# 5. Check network policies
kubectl get networkpolicies
kubectl describe networkpolicy <policy-name>

# 6. DNS resolution
kubectl exec -it <app-pod> -- nslookup database-service
```

**Scenario 3: Storage Issues**
```bash
# 1. Check PVC status
kubectl get pvc
kubectl describe pvc <pvc-name>

# 2. Check PV
kubectl get pv
kubectl describe pv <pv-name>

# 3. Check StorageClass
kubectl get storageclass
kubectl describe storageclass <sc-name>

# 4. Check volume usage
kubectl exec -it <pod-name> -- df -h

# 5. Check for mount errors
kubectl describe pod <pod-name> | grep -A 10 "Events:"
```

**Scenario 4: Node Issues**
```bash
# 1. Check node status
kubectl get nodes
kubectl describe node <node-name>

# 2. Check node conditions
kubectl get nodes -o json | jq '.items[] | {name: .metadata.name, conditions: .status.conditions}'

# 3. Check system pods
kubectl get pods -n kube-system

# 4. SSH to node (if possible)
ssh <node-ip>
# Check:
# - systemctl status kubelet
# - systemctl status docker
# - df -h
# - free -m
# - dmesg | tail

# 5. Check node logs
kubectl logs -n kube-system <kubelet-pod>
```

#### 29.4 Advanced Debugging Tools

**Tool 1: Stern (Multi-pod log viewing)**
```bash
# Install stern
choco install stern

# View logs from multiple pods
stern <pod-prefix>
stern --namespace <namespace> <pod-prefix>
stern -l app=myapp

# With colors and timestamps
stern --color always --timestamps <pod-prefix>
```

**Tool 2: kubectx & kubens**
```bash
# Install
choco install kubectx

# Switch contexts
kubectx
kubectx <context-name>

# Switch namespaces
kubens
kubens <namespace>
```

**Tool 3: k9s (Terminal UI)**
```bash
# Install
choco install k9s

# Run
k9s

# Keyboard shortcuts:
# :pods     - View pods
# :svc      - View services
# :deploy   - View deployments
# /         - Filter
# l         - View logs
# d         - Describe
# e         - Edit
# Ctrl-d    - Delete
```

**Tool 4: Kube-ops-view**
```bash
# Deploy
kubectl apply -f https://raw.githubusercontent.com/hjacobs/kube-ops-view/master/deploy/kube-ops-view.yaml

# Access
kubectl port-forward service/kube-ops-view 8080:80
# Open http://localhost:8080
```

**Tool 5: Goldilocks (VPA recommendations)**
```bash
helm repo add fairwinds-stable https://charts.fairwinds.com/stable
helm install goldilocks fairwinds-stable/goldilocks --namespace goldilocks --create-namespace

# Enable for namespace
kubectl label namespace default goldilocks.fairwinds.com/enabled=true

# View dashboard
kubectl port-forward -n goldilocks svc/goldilocks-dashboard 8080:80
```

---

### Week 36-42: Disaster Recovery & Security

#### 36.1 Backup & Restore with Velero
```bash
# Install Velero CLI
choco install velero

# Install Velero in cluster
velero install \
  --provider aws \
  --plugins velero/velero-plugin-for-aws:v1.7.0 \
  --bucket velero-backups \
  --secret-file ./credentials-velero \
  --backup-location-config region=us-east-1

# Create backup
velero backup create full-backup
velero backup create namespace-backup --include-namespaces production

# Schedule backups
velero schedule create daily-backup --schedule="0 2 * * *"

# Restore
velero restore create --from-backup full-backup

# Check backup status
velero backup describe full-backup
velero backup logs full-backup

# Restore specific namespace
velero restore create --from-backup full-backup --include-namespaces production
```

#### 36.2 etcd Backup & Restore
```bash
# Backup etcd
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Verify snapshot
ETCDCTL_API=3 etcdctl snapshot status /backup/etcd-snapshot.db --write-out=table

# Restore etcd
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-snapshot.db \
  --data-dir=/var/lib/etcd-restore
```

#### 36.3 Security Scanning

**Trivy - Container vulnerability scanner**
```bash
# Install
choco install trivy

# Scan image
trivy image nginx:latest
trivy image --severity HIGH,CRITICAL myapp:v1

# Scan Kubernetes cluster
trivy k8s --report summary cluster
trivy k8s all --namespace production
```

**Kubesec - Kubernetes security analysis**
```bash
# Install
curl -sSL https://github.com/controlplaneio/kubesec/releases/download/v2.13.0/kubesec_windows_amd64.exe -o kubesec.exe

# Scan file
kubesec scan deployment.yaml

# Get recommendations
kubesec scan deployment.yaml | jq '.[] | .scoring'
```

**Kube-bench - CIS Kubernetes Benchmark**
```bash
# Run as job
kubectl apply -f https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job.yaml

# View results
kubectl logs job/kube-bench
```

**Kube-hunter - Penetration testing**
```bash
# Run as pod
kubectl create -f https://raw.githubusercontent.com/aquasecurity/kube-hunter/master/job.yaml

# View results
kubectl logs job/kube-hunter
```

#### 36.4 Secrets Management

**Sealed Secrets**
```bash
# Install controller
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.24.0/controller.yaml

# Install kubeseal CLI
choco install kubeseal

# Create sealed secret
kubectl create secret generic mysecret --dry-run=client --from-literal=password=mypassword -o yaml | \
  kubeseal -o yaml > sealed-secret.yaml

# Apply sealed secret
kubectl apply -f sealed-secret.yaml
```

**External Secrets Operator** (AWS Secrets Manager, Azure Key Vault, etc.)
```bash
# Install
helm repo add external-secrets https://charts.external-secrets.io
helm install external-secrets external-secrets/external-secrets -n external-secrets-system --create-namespace
```

```yaml
# external-secret.yaml
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: aws-secret-store
spec:
  provider:
    aws:
      service: SecretsManager
      region: us-east-1
      auth:
        secretRef:
          accessKeyIDSecretRef:
            name: aws-credentials
            key: access-key-id
          secretAccessKeySecretRef:
            name: aws-credentials
            key: secret-access-key

---
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: app-secret
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secret-store
    kind: SecretStore
  target:
    name: app-secret
    creationPolicy: Owner
  data:
  - secretKey: password
    remoteRef:
      key: prod/app/password
```

---

### Week 43-48: Performance Tuning & Optimization

#### 43.1 Performance Monitoring
```bash
# Resource usage over time
watch kubectl top nodes
watch kubectl top pods

# Get detailed metrics
kubectl get --raw /apis/metrics.k8s.io/v1beta1/nodes
kubectl get --raw /apis/metrics.k8s.io/v1beta1/pods

# Analyze resource usage
kubectl describe node <node-name> | grep -A 5 "Allocated resources"
```

#### 43.2 Performance Best Practices

**1. Resource Optimization**
```yaml
# right-sized-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: optimized-pod
spec:
  containers:
  - name: app
    image: myapp:v1
    resources:
      requests:
        cpu: "100m"      # Start small
        memory: "128Mi"
      limits:
        cpu: "500m"      # Allow bursting
        memory: "512Mi"  # Prevent OOM
    
    # Quality of Service: Guaranteed
    # (when requests == limits)
```

**2. Startup Optimization**
```yaml
# fast-startup.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fast-app
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2        # Deploy 2 extra during update
      maxUnavailable: 0  # Keep all running
  template:
    spec:
      containers:
      - name: app
        image: myapp:v1
        
        # Start fast with startup probe
        startupProbe:
          httpGet:
            path: /startup
            port: 8080
          initialDelaySeconds: 0
          periodSeconds: 5
          failureThreshold: 30  # 150s max startup time
        
        # Then use liveness
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          periodSeconds: 10
        
        # Use image pull policy
        imagePullPolicy: IfNotPresent
      
      # Prioritize critical workloads
      priorityClassName: high-priority
```

**3. Network Optimization**
```yaml
# network-optimized.yaml
apiVersion: v1
kind: Service
metadata:
  name: app-service
spec:
  # Use ClusterIP for internal services
  type: ClusterIP
  
  # Session affinity for stateful apps
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800
  
  # Use headless service for direct pod access
  # clusterIP: None
  
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
```

**4. Storage Optimization**
```yaml
# storage-optimized.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: fast-storage
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: ssd-storage  # Use fast storage class
  resources:
    requests:
      storage: 10Gi
```

#### 43.3 Cluster Optimization

**1. Node Pool Strategy**
```yaml
# Different node pools for different workloads
# - Compute-optimized for CPU-intensive
# - Memory-optimized for memory-intensive
# - GPU nodes for ML workloads
# - Spot/Preemptible for batch jobs

# Label nodes
kubectl label nodes node1 workload-type=compute-intensive
kubectl label nodes node2 workload-type=memory-intensive

# Use node selector
spec:
  nodeSelector:
    workload-type: compute-intensive
```

**2. Pod Placement**
```yaml
# optimal-placement.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: distributed-app
spec:
  replicas: 6
  template:
    spec:
      # Spread across zones
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: DoNotSchedule
        labelSelector:
          matchLabels:
            app: distributed-app
      
      # Spread across nodes
      - maxSkew: 2
        topologyKey: kubernetes.io/hostname
        whenUnsatisfiable: ScheduleAnyway
        labelSelector:
          matchLabels:
            app: distributed-app
      
      # Avoid co-location with same app
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - distributed-app
              topologyKey: kubernetes.io/hostname
      
      containers:
      - name: app
        image: myapp:v1
```

---

### Week 49-52: Expert-Level Topics

#### 49.1 Advanced Scheduling

**Priority and Preemption**
```yaml
# priority-classes.yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: system-critical
value: 2000000000
globalDefault: false
description: "System-critical pods"

---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: "High-priority application pods"

---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: low-priority
value: 100
globalDefault: true
description: "Low-priority batch jobs"
```

**Custom Scheduler**
```yaml
# custom-scheduler.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-custom-scheduler
spec:
  schedulerName: my-custom-scheduler
  containers:
  - name: app
    image: myapp:v1
```

#### 49.2 Advanced Networking

**Service Mesh with Istio**
```bash
# Install Istio
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.20.0
export PATH=$PWD/bin:$PATH

# Install Istio
istioctl install --set profile=demo -y

# Enable sidecar injection
kubectl label namespace default istio-injection=enabled

# Deploy sample app
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml

# Create gateway
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
```

**Traffic Management**
```yaml
# virtual-service.yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v3
      weight: 80
    - destination:
        host: reviews
        subset: v1
      weight: 20
```

#### 49.3 Chaos Engineering

**Chaos Mesh**
```bash
# Install Chaos Mesh
curl -sSL https://mirrors.chaos-mesh.org/latest/install.sh | bash

# Verify installation
kubectl get pods -n chaos-mesh
```

**Pod Failure Experiment**
```yaml
# pod-failure.yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: PodChaos
metadata:
  name: pod-failure-example
spec:
  action: pod-failure
  mode: one
  duration: "30s"
  selector:
    namespaces:
      - default
    labelSelectors:
      app: myapp
  scheduler:
    cron: "@every 2m"
```

**Network Delay Experiment**
```yaml
# network-delay.yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: NetworkChaos
metadata:
  name: network-delay
spec:
  action: delay
  mode: all
  selector:
    namespaces:
      - default
    labelSelectors:
      app: myapp
  delay:
    latency: "100ms"
    correlation: "100"
    jitter: "0ms"
  duration: "5m"
```

#### 49.4 Multi-Tenancy

**Hierarchical Namespaces**
```yaml
# hierarchical-namespace.yaml
apiVersion: hnc.x-k8s.io/v1alpha2
kind: SubnamespaceAnchor
metadata:
  name: team-a
  namespace: organization

---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-quota
  namespace: team-a
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 20Gi
    pods: "20"
```

**Tenant Isolation with Network Policies**
```yaml
# tenant-isolation.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-cross-namespace
  namespace: tenant-a
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          tenant: tenant-a
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          tenant: tenant-a
  - to:  # Allow DNS
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: UDP
      port: 53
```

#### 49.5 Advanced Observability

**Distributed Tracing with Jaeger**
```bash
# Install Jaeger Operator
kubectl create namespace observability
kubectl create -f https://github.com/jaegertracing/jaeger-operator/releases/download/v1.51.0/jaeger-operator.yaml -n observability

# Deploy Jaeger instance
kubectl apply -f - <<EOF
apiVersion: jaegertracing.io/v1
kind: Jaeger
metadata:
  name: simplest
  namespace: observability
EOF

# Access Jaeger UI
kubectl port-forward -n observability svc/simplest-query 16686:16686
```

**OpenTelemetry**
```yaml
# otel-collector.yaml
apiVersion: opentelemetry.io/v1alpha1
kind: OpenTelemetryCollector
metadata:
  name: otel-collector
spec:
  config: |
    receivers:
      otlp:
        protocols:
          grpc:
          http:
    processors:
      batch:
    exporters:
      prometheus:
        endpoint: "0.0.0.0:8889"
      jaeger:
        endpoint: jaeger-collector.observability.svc:14250
        tls:
          insecure: true
    service:
      pipelines:
        traces:
          receivers: [otlp]
          processors: [batch]
          exporters: [jaeger]
        metrics:
          receivers: [otlp]
          processors: [batch]
          exporters: [prometheus]
```

---

## Practical Projects

### Project 1: Microservices E-Commerce Platform
**Components**:
- Frontend (React)
- API Gateway
- Product Service
- Order Service
- Payment Service
- User Service
- PostgreSQL Database
- Redis Cache
- RabbitMQ Message Queue

**Requirements**:
1. Deploy all services
2. Implement service mesh (Istio)
3. Set up monitoring (Prometheus + Grafana)
4. Configure autoscaling
5. Implement CI/CD (GitOps with ArgoCD)
6. Set up logging (EFK stack)
7. Configure ingress with TLS
8. Implement backup strategy

### Project 2: ML Training Platform
**Components**:
- Jupyter Notebooks
- TensorFlow/PyTorch workers
- GPU nodes
- Model registry
- Data pipeline
- Result storage

**Requirements**:
1. GPU node pool configuration
2. Job scheduling for training
3. Persistent storage for models
4. Resource quotas per team
5. Monitoring GPU utilization
6. Cost optimization

### Project 3: Multi-Tenant SaaS Platform
**Components**:
- Tenant isolation
- Shared services
- Per-tenant databases
- Billing service
- Admin dashboard

**Requirements**:
1. Namespace per tenant
2. Network policies
3. Resource quotas
4. RBAC configuration
5. Monitoring per tenant
6. Backup per tenant

---

## Certification Path

### 1. Certified Kubernetes Application Developer (CKAD)
**Focus**: Application development on Kubernetes
**Topics**:
- Core concepts
- Configuration
- Multi-container pods
- Observability
- Pod design
- Services & networking

**Preparation**:
- Hands-on practice (70% of exam)
- Time management (2 hours, 15-20 tasks)
- kubectl speed and efficiency
- YAML proficiency

**Resources**:
- https://kubernetes.io/docs/
- https://github.com/dgkanatsios/CKAD-exercises

### 2. Certified Kubernetes Administrator (CKA)
**Focus**: Kubernetes administration
**Topics**:
- Cluster architecture
- Installation & configuration
- Workloads & scheduling
- Services & networking
- Storage
- Troubleshooting

**Preparation**:
- Cluster setup practice
- Backup and restore
- Troubleshooting scenarios
- Security configuration

**Resources**:
- https://kubernetes.io/docs/
- https://github.com/walidshaari/Kubernetes-Certified-Administrator

### 3. Certified Kubernetes Security Specialist (CKS)
**Focus**: Kubernetes security
**Topics**:
- Cluster setup & hardening
- System hardening
- Minimize microservice vulnerabilities
- Supply chain security
- Monitoring, logging & runtime security

**Prerequisites**: Must have CKA

**Resources**:
- https://kubernetes.io/docs/
- https://github.com/walidshaari/Certified-Kubernetes-Security-Specialist

---

## Resources & Tools

### Official Documentation
- **Kubernetes.io**: https://kubernetes.io/docs/
- **GitHub**: https://github.com/kubernetes/kubernetes
- **Slack**: https://slack.k8s.io/
- **Stack Overflow**: https://stackoverflow.com/questions/tagged/kubernetes

### Learning Platforms
- **Kubernetes Documentation**: https://kubernetes.io/docs/tutorials/
- **KillerCoda (formerly Katacoda)**: https://killercoda.com/
- **Play with Kubernetes**: https://labs.play-with-k8s.com/
- **Kubernetes Tutorials**: https://kubernetes.io/docs/tutorials/

### Books
- "Kubernetes Up & Running" by Kelsey Hightower
- "Kubernetes in Action" by Marko Lukša
- "Programming Kubernetes" by Michael Hausenblas
- "Production Kubernetes" by Josh Rosso

### Tools Checklist
- [x] kubectl - CLI tool
- [x] kubectx/kubens - Context/namespace switching
- [x] k9s - Terminal UI
- [x] stern - Multi-pod log viewer
- [x] Helm - Package manager
- [x] Kustomize - Configuration management
- [x] Lens - Desktop Kubernetes IDE
- [x] Octant - Web-based Kubernetes dashboard
- [x] Velero - Backup and restore
- [x] Prometheus - Monitoring
- [x] Grafana - Visualization
- [x] Jaeger - Distributed tracing
- [x] Istio - Service mesh
- [x] ArgoCD - GitOps
- [x] Trivy - Security scanner

### Community Resources
- **KubeCon + CloudNativeCon**: Annual conferences
  - Europe (Amsterdam, Mar 23-26, 2026)
  - North America (Salt Lake City, Nov 9-12, 2026)
- **Kubernetes Meetups**: Local community groups
- **CNCF Landscape**: https://landscape.cncf.io/
- **Awesome Kubernetes**: https://github.com/ramitsurana/awesome-kubernetes

### Practice Environments
1. **Minikube**: Local single-node cluster
2. **Kind**: Kubernetes in Docker
3. **K3s**: Lightweight Kubernetes
4. **Docker Desktop**: Built-in Kubernetes
5. **Cloud Providers**:
   - Google Kubernetes Engine (GKE) - Free tier
   - Amazon Elastic Kubernetes Service (EKS)
   - Azure Kubernetes Service (AKS)
   - DigitalOcean Kubernetes

### Debugging Cheat Sheet
```bash
# Quick reference commands
alias k=kubectl
alias kg='kubectl get'
alias kd='kubectl describe'
alias kl='kubectl logs'
alias ke='kubectl exec -it'
alias kx='kubectl explain'

# Get everything
k get all -A

# Quick pod logs
kl <pod> -f --tail=50

# Quick pod shell
ke <pod> -- /bin/bash

# Quick resource explanation
kx pod.spec.containers.resources
```

---

## Weekly Practice Routine

### Daily (30 minutes)
- [ ] Read 1 concept from official docs
- [ ] Practice 5 kubectl commands
- [ ] Review 1 YAML manifest

### Weekly (2-3 hours)
- [ ] Complete 1 hands-on lab
- [ ] Debug 2-3 scenarios
- [ ] Read 1 blog post/article
- [ ] Practice exam questions (if preparing for cert)

### Monthly
- [ ] Build 1 mini-project
- [ ] Contribute to open-source
- [ ] Attend 1 meetup/webinar
- [ ] Review and update notes

---

## Conclusion

This roadmap takes you from complete beginner to production Kubernetes expert. The key to mastery is:

1. **Hands-On Practice**: Theory is 30%, practice is 70%
2. **Break Things**: Learn by breaking and fixing
3. **Real Projects**: Apply knowledge to real scenarios
4. **Community**: Engage with the Kubernetes community
5. **Stay Updated**: Kubernetes evolves rapidly
6. **Production Experience**: Nothing beats real production debugging

### Next Steps:
1. ✅ Set up local Kubernetes cluster
2. ✅ Complete beginner section (Weeks 1-4)
3. ✅ Deploy first application
4. ✅ Practice daily kubectl commands
5. ✅ Join Kubernetes Slack community
6. ✅ Follow the roadmap progressively

**Remember**: Kubernetes is a journey, not a destination. Keep learning, keep practicing, and don't be afraid to experiment!

---

*Based on official documentation from [kubernetes.io](https://kubernetes.io/)*

**Good luck on your Kubernetes journey! 🚀**


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

