# Kubernetes Interview Questions
## From Basic to Advanced Production Level (7+ Years Experience)

Complete interview preparation guide covering fundamentals to production scenarios for experienced professionals.

---

## Table of Contents
1. [Fundamental Questions (Warm-up)](#fundamental-questions-warm-up)
2. [Architecture & Components](#architecture--components)
3. [Workloads & Controllers](#workloads--controllers)
4. [Networking](#networking)
5. [Storage](#storage)
6. [Configuration & Secrets](#configuration--secrets)
7. [Security & RBAC](#security--rbac)
8. [Scheduling & Resource Management](#scheduling--resource-management)
9. [Production & Operations](#production--operations)
10. [Troubleshooting & Debugging](#troubleshooting--debugging)
11. [Performance & Optimization](#performance--optimization)
12. [Advanced Architecture](#advanced-architecture)
13. [Scenario-Based Questions](#scenario-based-questions)
14. [Behavioral & Experience Questions](#behavioral--experience-questions)

---

## Fundamental Questions (Warm-up)

### Q1: What is Kubernetes and why do we need it?
**Expected Answer:**
Kubernetes is an open-source container orchestration platform that automates deployment, scaling, and management of containerized applications. 

**Why we need it:**
- **Automated deployment**: Rolling updates, rollbacks
- **Self-healing**: Restarts failed containers, replaces containers
- **Horizontal scaling**: Scale applications up/down automatically
- **Service discovery**: Built-in DNS and load balancing
- **Storage orchestration**: Automatic mounting of storage systems
- **Secret management**: Deploy and update secrets without rebuilding images
- **Resource optimization**: Bin packing to optimize resource utilization

**Follow-up**: What problems did you face before Kubernetes that it solved?

---

### Q2: Explain the Kubernetes architecture. What are the main components?
**Expected Answer:**

**Control Plane Components:**
1. **kube-apiserver**: Front-end for Kubernetes control plane, exposes REST API
2. **etcd**: Distributed key-value store for all cluster data
3. **kube-scheduler**: Assigns pods to nodes based on resource requirements
4. **kube-controller-manager**: Runs controller processes (Node, Replication, Endpoints, ServiceAccount)
5. **cloud-controller-manager**: Runs cloud-specific controllers (optional)

**Node Components:**
1. **kubelet**: Agent that ensures containers are running in pods
2. **kube-proxy**: Network proxy maintaining network rules
3. **container runtime**: Software to run containers (containerd, CRI-O, Docker)

**Add-ons:**
- DNS, Dashboard, Monitoring, Logging

**Follow-up**: What happens when you run `kubectl apply -f deployment.yaml`?

---

### Q3: What is a Pod? Why doesn't Kubernetes deploy containers directly?
**Expected Answer:**

**Pod** is the smallest deployable unit in Kubernetes that contains one or more containers.

**Why Pods, not containers directly:**
1. **Shared resources**: Containers in a pod share network namespace (IP, ports) and volumes
2. **Co-location**: Tightly coupled containers can be deployed together
3. **Atomic deployment**: All containers in pod deployed as a unit
4. **Shared lifecycle**: Started, stopped, and rescheduled together
5. **Design patterns**: Enables sidecar, ambassador, adapter patterns

**Example use cases:**
- Main app + logging sidecar
- Main app + proxy sidecar
- Main app + init containers for setup

**Follow-up**: Can you explain multi-container pod patterns you've used?

---

### Q4: What's the difference between a Deployment and a StatefulSet?
**Expected Answer:**

| Aspect | Deployment | StatefulSet |
|--------|-----------|-------------|
| **Identity** | Random pod names | Stable, predictable names (web-0, web-1) |
| **Network** | Random IPs | Stable network identity |
| **Storage** | Shared or random | Persistent per-pod storage |
| **Scaling** | Parallel | Ordered (sequential) |
| **Updates** | Rolling updates | Ordered updates |
| **Use case** | Stateless apps | Stateful apps (databases) |

**StatefulSet guarantees:**
- Stable, unique network identifiers
- Stable, persistent storage
- Ordered, graceful deployment and scaling
- Ordered, automated rolling updates

**Follow-up**: When would you choose StatefulSet over Deployment?

---

### Q5: Explain different types of Services in Kubernetes.
**Expected Answer:**

1. **ClusterIP** (default)
   - Exposes service on cluster-internal IP
   - Only reachable from within cluster
   - Use: Internal microservice communication

2. **NodePort**
   - Exposes service on each node's IP at static port (30000-32767)
   - Accessible from outside cluster via `<NodeIP>:<NodePort>`
   - Use: Development, testing, or when LoadBalancer unavailable

3. **LoadBalancer**
   - Creates external load balancer (cloud provider)
   - Assigns external IP
   - Use: Production external access

4. **ExternalName**
   - Maps service to DNS name
   - No proxy, returns CNAME record
   - Use: External service abstraction

**Follow-up**: How does kube-proxy implement Services?

---

## Architecture & Components

### Q6: Explain the flow when a pod is created. What happens internally?
**Expected Answer:**

1. **User** runs `kubectl create -f pod.yaml`
2. **kubectl** sends REST request to **API server**
3. **API server** validates and stores pod spec in **etcd**
4. **API server** returns response to kubectl
5. **Scheduler** watches API server, sees unscheduled pod
6. **Scheduler** determines best node based on:
   - Resource requirements
   - Node constraints
   - Affinity/anti-affinity rules
   - Taints and tolerations
7. **Scheduler** updates pod's `nodeName` field via API server
8. **Kubelet** on assigned node watches API server
9. **Kubelet** sees pod assigned to its node
10. **Kubelet** instructs container runtime to pull image and start container
11. **Kubelet** monitors container health and reports status to API server
12. **kube-proxy** updates iptables/IPVS rules if pod is part of service

**Follow-up**: What if scheduling fails? What if node goes down?

---

### Q7: How does etcd work in Kubernetes? What happens if etcd goes down?
**Expected Answer:**

**etcd in Kubernetes:**
- Distributed, consistent key-value store
- Single source of truth for cluster state
- Stores all cluster data (pods, services, secrets, etc.)
- Uses Raft consensus algorithm for consistency
- Typically runs in HA mode (3, 5, or 7 nodes)

**If etcd goes down:**
- **Existing workloads continue running** (kubelet has cached state)
- **Cannot create/update/delete resources** (API server can't persist)
- **No new pods scheduled**
- **No updates to cluster state**
- **Critical failure**: Cluster effectively read-only

**Recovery:**
- Restore from backup
- Rebuild etcd cluster
- Use etcd disaster recovery procedures

**Production best practices:**
- Run etcd as cluster (odd numbers: 3, 5)
- Regular backups (automated snapshots)
- Separate etcd from API server (not on same node)
- Monitor etcd health continuously
- Use dedicated storage (SSD preferred)

**Follow-up**: How do you backup and restore etcd?

---

### Q8: Explain how kube-proxy works. What are the different modes?
**Expected Answer:**

**kube-proxy** maintains network rules on nodes for service abstraction.

**Three modes:**

1. **iptables mode** (default)
   - Uses iptables rules for service routing
   - Random selection for load balancing
   - Better performance than userspace
   - No health checking (relies on readiness probes)

2. **IPVS mode** (recommended for large clusters)
   - Uses IP Virtual Server (Linux kernel feature)
   - Better performance and scalability
   - More load balancing algorithms (rr, lc, dh, sh, sed, nq)
   - Connection-based load balancing
   - Falls back to iptables if IPVS unavailable

3. **userspace mode** (legacy)
   - kube-proxy itself acts as proxy
   - Poor performance
   - Deprecated

**How it works:**
- Watches API server for Service/Endpoint changes
- Programs network rules based on service type
- Enables service discovery and load balancing

**Follow-up**: What's the difference between iptables and IPVS in large-scale deployments?

---

### Q9: What are admission controllers? Name some important ones.
**Expected Answer:**

**Admission controllers** are plugins that intercept API requests before persistence but after authentication and authorization.

**Two types:**
1. **Validating**: Validate requests, accept/reject
2. **Mutating**: Can modify requests before persistence

**Important admission controllers:**

1. **NamespaceLifecycle**: Prevents operations in terminating namespaces
2. **LimitRanger**: Enforces LimitRange constraints
3. **ResourceQuota**: Enforces ResourceQuota constraints
4. **ServiceAccount**: Automatically adds ServiceAccount to pods
5. **PodSecurityPolicy** (deprecated, replaced by Pod Security Admission)
6. **MutatingAdmissionWebhook**: Calls external webhooks to mutate objects
7. **ValidatingAdmissionWebhook**: Calls external webhooks to validate
8. **AlwaysPullImages**: Forces pulling images (security)
9. **NodeRestriction**: Limits kubelet modifications to own node
10. **PodSecurity**: Enforces Pod Security Standards

**Use cases:**
- Policy enforcement (OPA/Gatekeeper)
- Image scanning (require signed images)
- Resource validation
- Sidecar injection (Istio, Linkerd)
- Custom validation logic

**Follow-up**: Have you implemented custom admission webhooks?

---

### Q10: Explain the difference between API versions (alpha, beta, stable).
**Expected Answer:**

**Alpha (v1alpha1, v1alpha2):**
- Experimental features
- May have bugs
- Disabled by default
- May be dropped without notice
- Not recommended for production
- Example: `batch/v1alpha1`

**Beta (v1beta1, v1beta2):**
- Well-tested features
- Enabled by default
- Support guaranteed (won't be dropped)
- Details may change in future versions
- Safe for non-critical production
- Example: `autoscaling/v2beta2`

**Stable (v1, v2):**
- Production-ready
- Fully supported
- Backward compatible
- Will be supported for many versions
- Example: `apps/v1`, `v1`

**Version progression:**
`alpha` → `beta` → `stable (v1)`

**Follow-up**: How do you handle API deprecations in production?

---

## Workloads & Controllers

### Q11: Explain Deployment strategies. How do you implement blue-green and canary deployments?
**Expected Answer:**

**1. Rolling Update (default)**
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1        # Extra pods during update
    maxUnavailable: 0  # Maintain availability
```
- Gradual replacement of old pods with new
- Zero downtime if configured correctly
- Easy rollback

**2. Recreate**
```yaml
strategy:
  type: Recreate
```
- Terminate all old pods, then create new
- Downtime expected
- Use: Incompatible versions, stateful apps

**3. Blue-Green Deployment**
```yaml
# Two deployments: blue (current), green (new)
# Switch service selector to change traffic
# service.yaml
selector:
  app: myapp
  version: blue  # Change to 'green' to switch
```
- Zero downtime
- Instant rollback
- Requires double resources

**4. Canary Deployment**
```yaml
# Stable: 9 replicas (90%)
# Canary: 1 replica (10%)
# Both share same service label
```
- Gradual traffic shift
- Test new version with subset of users
- Can use service mesh for precise control (Istio)

**Production Implementation:**
- Use tools: Argo Rollouts, Flagger
- Implement automated testing
- Monitor metrics during rollout
- Define rollback criteria
- Use progressive delivery

**Follow-up**: How do you monitor canary deployments and decide to proceed or rollback?

---

### Q12: What are Init Containers? How are they different from sidecar containers?
**Expected Answer:**

**Init Containers:**
- Run before app containers
- Run to completion sequentially
- Must succeed before app containers start
- Don't support lifecycle, probes, or resource limits like app containers

**Use cases:**
- Wait for dependencies (service, database)
- Clone git repository
- Generate configuration files
- Set up permissions
- Database migrations

**Example:**
```yaml
spec:
  initContainers:
  - name: wait-for-db
    image: busybox
    command: ['sh', '-c', 'until nc -z db 5432; do sleep 2; done']
  - name: migrations
    image: myapp-migrations
    command: ['./migrate']
  containers:
  - name: app
    image: myapp
```

**Sidecar Containers:**
- Run alongside app container
- Runs continuously
- Shares pod lifecycle
- Can be added/removed without affecting main app

**Differences:**

| Aspect | Init Container | Sidecar Container |
|--------|---------------|-------------------|
| When runs | Before app start | With app |
| Lifecycle | Runs once | Runs continuously |
| Failure | Blocks app start | Pod restarts |
| Order | Sequential | Parallel |
| Use case | Setup, migration | Logging, proxy, monitoring |

**Follow-up**: How would you handle init container failures in production?

---

### Q13: Explain DaemonSet. When would you use it over Deployment?
**Expected Answer:**

**DaemonSet** ensures one pod copy runs on all (or selected) nodes.

**Characteristics:**
- Automatically adds pods to new nodes
- Removes pods when nodes are removed
- Deleting DaemonSet cleans up pods

**Use cases:**
1. **Node monitoring**: Prometheus node exporter, Datadog agent
2. **Log collection**: Fluentd, Filebeat
3. **Storage daemons**: Ceph, GlusterFS
4. **Network plugins**: CNI agents
5. **Security**: Vulnerability scanners

**When to use DaemonSet vs Deployment:**

**Use DaemonSet when:**
- Need exactly one pod per node
- System-level services (monitoring, logging)
- Node-specific operations
- Infrastructure services

**Use Deployment when:**
- Application workloads
- Need specific number of replicas
- Scaling based on load
- Stateless applications

**Advanced features:**
```yaml
spec:
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
  selector:
    matchLabels:
      app: logging
  template:
    spec:
      nodeSelector:
        disktype: ssd  # Run only on SSD nodes
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule  # Run on master nodes
```

**Follow-up**: How do you handle DaemonSet updates without disrupting node operations?

---

### Q14: What are Jobs and CronJobs? Explain backoff policies.
**Expected Answer:**

**Job** - Runs pods to completion

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: batch-job
spec:
  completions: 5          # Total successful completions needed
  parallelism: 2          # Run 2 pods at a time
  backoffLimit: 4         # Retry 4 times before marking as failed
  activeDeadlineSeconds: 600  # Timeout after 10 minutes
  template:
    spec:
      restartPolicy: OnFailure  # Never or OnFailure (not Always)
      containers:
      - name: worker
        image: batch-processor
```

**CronJob** - Scheduled Jobs
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: scheduled-job
spec:
  schedule: "*/5 * * * *"  # Every 5 minutes
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  concurrencyPolicy: Forbid  # Allow, Forbid, Replace
  startingDeadlineSeconds: 200
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: worker
            image: backup-processor
          restartPolicy: OnFailure
```

**Backoff Policy:**
- Default: exponential backoff (10s, 20s, 40s, ...)
- Maximum backoff: 6 minutes
- Controlled by `backoffLimit`
- Resets on successful completion

**Concurrency Policies:**
1. **Allow**: Multiple jobs can run concurrently
2. **Forbid**: Skip if previous job still running
3. **Replace**: Cancel existing and start new

**Production considerations:**
- Monitor failed jobs
- Set appropriate timeouts
- Clean up completed jobs
- Handle idempotency
- Use proper exit codes (0 = success)

**Follow-up**: How do you handle failed CronJobs in production? What about duplicate runs?

---

### Q15: How does Horizontal Pod Autoscaler (HPA) work? What metrics can it use?
**Expected Answer:**

**HPA** automatically scales pods based on observed metrics.

**How it works:**
1. HPA controller queries metrics every 15 seconds (default)
2. Calculates desired replicas: `ceil(current * (current metric / target metric))`
3. Updates Deployment/StatefulSet replica count
4. Checks scaling conditions (delays, limits)
5. Scales up/down pods

**Architecture:**
```
HPA Controller → Metrics API → Metrics Server → kubelet
```

**Metric types:**

**1. Resource Metrics** (CPU, Memory)
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 2
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
        type: AverageValue
        averageValue: 500Mi
```

**2. Custom Metrics** (from application)
```yaml
- type: Pods
  pods:
    metric:
      name: http_requests_per_second
    target:
      type: AverageValue
      averageValue: "1000"
```

**3. External Metrics** (from external systems)
```yaml
- type: External
  external:
    metric:
      name: queue_messages_ready
      selector:
        matchLabels:
          queue: worker_tasks
    target:
      type: AverageValue
      averageValue: "30"
```

**HPA Behavior (Advanced)**
```yaml
behavior:
  scaleDown:
    stabilizationWindowSeconds: 300  # Wait 5 min before scale down
    policies:
    - type: Percent
      value: 50      # Max 50% scale down
      periodSeconds: 60
  scaleUp:
    stabilizationWindowSeconds: 0
    policies:
    - type: Percent
      value: 100     # Double pods
      periodSeconds: 30
    - type: Pods
      value: 4       # Or add 4 pods
      periodSeconds: 60
    selectPolicy: Max  # Choose more aggressive policy
```

**Production best practices:**
- Set appropriate resource requests (HPA uses these)
- Use multiple metrics (CPU + custom)
- Configure scale down stabilization
- Set min/max replicas wisely
- Monitor HPA decisions
- Test scaling under load

**Follow-up**: What happens if pods don't have resource requests defined?

---

## Networking

### Q16: Explain Kubernetes networking model. What are the fundamental requirements?
**Expected Answer:**

**Kubernetes Networking Model Requirements:**

1. **All pods can communicate with all pods** without NAT
2. **All nodes can communicate with all pods** without NAT
3. **IP pod sees is same as others see** (no NAT)

**Network types:**

**1. Pod-to-Pod Communication**
- Every pod gets unique IP
- Pods on same node: via virtual bridge
- Pods on different nodes: via CNI plugin

**2. Pod-to-Service Communication**
- Service has stable virtual IP (ClusterIP)
- kube-proxy maintains iptables/IPVS rules
- Traffic load-balanced across pod endpoints

**3. External-to-Service Communication**
- NodePort: Exposes service on node's IP
- LoadBalancer: Cloud provider load balancer
- Ingress: HTTP(S) routing with single entry point

**CNI (Container Network Interface) Plugins:**
- **Calico**: L3 networking, network policies
- **Flannel**: Simple overlay network
- **Cilium**: eBPF-based, advanced features
- **Weave Net**: Simple setup, mesh network
- **AWS VPC CNI**: Native AWS VPC networking

**Network flow example:**
```
External Request
    ↓
LoadBalancer (Cloud)
    ↓
Ingress Controller (nginx)
    ↓
Service (ClusterIP)
    ↓
kube-proxy (iptables/IPVS)
    ↓
Pod (container)
```

**Follow-up**: How would you troubleshoot pod-to-pod communication failures?

---

### Q17: What is an Ingress? How does it differ from Service?
**Expected Answer:**

**Ingress** - HTTP(S) routing rules to services

**Service vs Ingress:**

| Aspect | Service | Ingress |
|--------|---------|---------|
| Layer | L4 (Transport) | L7 (Application) |
| Protocol | TCP/UDP | HTTP/HTTPS |
| Entry points | Many | Single (shared) |
| TLS | No (LoadBalancer handles) | Yes (built-in) |
| Routing | By port | By host/path |
| Cost | Per service (LoadBalancer) | Shared |

**Ingress Components:**

**1. Ingress Resource** (routing rules)
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - myapp.example.com
    secretName: tls-secret
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 8080
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
```

**2. Ingress Controller** (implements rules)
- **nginx-ingress**: Most popular, feature-rich
- **traefik**: Cloud-native, dynamic config
- **HAProxy**: High performance
- **AWS ALB Ingress**: Native AWS integration
- **Istio Gateway**: Service mesh integration

**Advanced Features:**

**Path types:**
- **Exact**: Exact match (/foo)
- **Prefix**: Prefix match (/foo/*)
- **ImplementationSpecific**: Controller-defined

**Annotations:**
```yaml
annotations:
  # Redirects
  nginx.ingress.kubernetes.io/ssl-redirect: "true"
  
  # Rate limiting
  nginx.ingress.kubernetes.io/limit-rps: "100"
  
  # Timeouts
  nginx.ingress.kubernetes.io/proxy-read-timeout: "600"
  
  # Authentication
  nginx.ingress.kubernetes.io/auth-type: basic
  nginx.ingress.kubernetes.io/auth-secret: basic-auth
  
  # CORS
  nginx.ingress.kubernetes.io/enable-cors: "true"
  
  # Canary
  nginx.ingress.kubernetes.io/canary: "true"
  nginx.ingress.kubernetes.io/canary-weight: "10"
```

**Production setup:**
- Use cert-manager for automatic TLS
- Configure rate limiting
- Set up monitoring and logging
- Use multiple replicas for HA
- Configure proper resource limits
- Implement WAF (Web Application Firewall)

**Follow-up**: How do you handle TLS certificate management at scale?

---

### Q18: Explain Network Policies. How do you implement zero-trust networking?
**Expected Answer:**

**Network Policy** defines how pods communicate with each other and other endpoints.

**Default behavior:** All pods can communicate (no restrictions)

**Zero-trust approach:**

**Step 1: Default Deny All**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: production
spec:
  podSelector: {}  # Applies to all pods
  policyTypes:
  - Ingress
  - Egress
```

**Step 2: Allow Specific Traffic**
```yaml
# Allow frontend → backend
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: backend
      tier: api
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    - namespaceSelector:
        matchLabels:
          name: production
    ports:
    - protocol: TCP
      port: 8080
```

**Step 3: Allow DNS (Always needed)**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-egress
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: UDP
      port: 53
```

**Complex example: Multi-tier application**
```yaml
# Database: Only from backend
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: database-policy
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
    ports:
    - protocol: TCP
      port: 5432

---
# Backend: From frontend, egress to DB and external APIs
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-policy
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432
  - to:  # External APIs
    - namespaceSelector: {}
    ports:
    - protocol: TCP
      port: 443
  - to:  # DNS
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: UDP
      port: 53
```

**Network Policy selectors:**
- **podSelector**: Select pods in same namespace
- **namespaceSelector**: Select all pods in namespace(s)
- **ipBlock**: CIDR-based selection
- **Combined**: podSelector + namespaceSelector (AND logic)

**Production best practices:**
- Start with default-deny
- Document policies clearly
- Test policies before production
- Use labels consistently
- Monitor denied connections
- Audit policies regularly
- Use tools: Cilium, Calico for advanced features

**Limitations:**
- Not all CNI plugins support Network Policies
- Cannot filter on service names (use pod selectors)
- Cannot deny based on L7 (use service mesh)
- Cannot filter on DNS names

**Follow-up**: How do you test Network Policies before deploying to production?

---

### Q19: Explain DNS in Kubernetes. How does service discovery work?
**Expected Answer:**

**Kubernetes DNS** (CoreDNS) provides service discovery.

**DNS naming:**

**1. Services:**
```
<service-name>.<namespace>.svc.cluster.local
```
Examples:
- `nginx.default.svc.cluster.local` (Full)
- `nginx.default` (Short, same cluster)
- `nginx` (Same namespace)

**2. Pods:**
```
<pod-ip-with-dashes>.<namespace>.pod.cluster.local
```
Example:
- `10-244-1-5.default.pod.cluster.local`

**3. Headless Services (StatefulSet):**
```
<pod-name>.<service-name>.<namespace>.svc.cluster.local
```
Example:
- `web-0.nginx.default.svc.cluster.local`

**DNS resolution flow:**
```
Pod
  ↓ (reads /etc/resolv.conf)
CoreDNS Service (kube-dns)
  ↓
CoreDNS Pod(s)
  ↓ (queries API server)
Service Endpoint
```

**/etc/resolv.conf in pod:**
```
nameserver 10.96.0.10  # CoreDNS service IP
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```

**DNS policies:**

**1. ClusterFirst** (default)
```yaml
dnsPolicy: ClusterFirst
```
- Queries sent to CoreDNS
- Falls back to node's DNS

**2. Default**
```yaml
dnsPolicy: Default
```
- Inherit node's DNS resolution

**3. None** (custom)
```yaml
dnsPolicy: None
dnsConfig:
  nameservers:
  - 8.8.8.8
  searches:
  - custom.dns.local
  options:
  - name: ndots
    value: "2"
```

**4. ClusterFirstWithHostNet**
- For pods using hostNetwork

**CoreDNS Configuration:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        health {
           lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
           ttl 30
        }
        prometheus :9153
        forward . /etc/resolv.conf {
           max_concurrent 1000
        }
        cache 30
        loop
        reload
        loadbalance
    }
```

**Troubleshooting DNS:**
```bash
# Test DNS from pod
kubectl exec -it <pod> -- nslookup kubernetes.default
kubectl exec -it <pod> -- nslookup myservice

# Check CoreDNS
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns

# DNS test pod
kubectl run -it --rm debug --image=nicolaka/netshoot -- bash
# Inside: nslookup, dig, host commands
```

**Performance tuning:**
- Increase CoreDNS replicas
- Enable DNS caching in applications
- Use NodeLocal DNSCache
- Tune `ndots` value

**Follow-up**: How would you troubleshoot intermittent DNS resolution failures?

---

### Q20: What is a Service Mesh? When would you use it?
**Expected Answer:**

**Service Mesh** is a dedicated infrastructure layer for handling service-to-service communication.

**Popular service meshes:**
- **Istio**: Feature-rich, complex
- **Linkerd**: Simple, lightweight
- **Consul**: HashiCorp's solution
- **AWS App Mesh**: AWS-native

**Key features:**

**1. Traffic Management**
- Intelligent routing (blue-green, canary)
- Traffic splitting
- Circuit breaking
- Retries, timeouts
- Load balancing

**2. Security**
- mTLS (mutual TLS) between services
- Certificate management
- Authentication/authorization
- Encryption at rest and in transit

**3. Observability**
- Distributed tracing
- Metrics collection
- Service topology
- Request logs

**4. Resilience**
- Circuit breakers
- Fault injection
- Retry logic
- Timeout policies

**Architecture (Istio example):**
```
Control Plane:
- istiod (Pilot, Citadel, Galley combined)

Data Plane:
- Envoy proxy sidecar in each pod
```

**Example Istio Configuration:**

**Virtual Service (routing)**
```yaml
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

**Destination Rule (policies)**
```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 50
        http2MaxRequests: 100
    outlierDetection:
      consecutiveErrors: 5
      interval: 30s
      baseEjectionTime: 30s
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
  - name: v3
    labels:
      version: v3
```

**When to use Service Mesh:**

**Use when:**
- Microservices architecture (many services)
- Need advanced traffic management
- Require mTLS without application changes
- Need distributed tracing
- Complex deployment strategies (canary, A/B)
- Multi-cluster communication
- Zero-trust security model

**Don't use when:**
- Simple monolithic application
- Few services (< 10)
- Limited resources (adds overhead)
- Team lacks expertise
- Don't need advanced features

**Trade-offs:**
- **Pros**: Security, observability, traffic control
- **Cons**: Complexity, resource overhead, learning curve, latency increase

**Production considerations:**
- Start small (pilot namespace)
- Monitor resource usage
- Train team on service mesh concepts
- Have rollback plan
- Monitor sidecar resource consumption
- Use managed service mesh if available (GKE, EKS, AKS)

**Follow-up**: How do you handle service mesh upgrades in production?

---

## Storage

### Q21: Explain PersistentVolume (PV), PersistentVolumeClaim (PVC), and StorageClass.
**Expected Answer:**

**Storage concepts:**

**1. PersistentVolume (PV)** - Cluster-level storage resource
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-volume
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: fast-ssd
  hostPath:
    path: /mnt/data
```

**2. PersistentVolumeClaim (PVC)** - User's storage request
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-claim
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: fast-ssd
```

**3. StorageClass** - Dynamic provisioning template
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  iopsPerGB: "10"
  fsType: ext4
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
```

**Relationship:**
```
StorageClass → (dynamic provisioning) → PV ← (bound to) ← PVC → Pod
```

**Access Modes:**
1. **ReadWriteOnce (RWO)**: Single node read-write
2. **ReadOnlyMany (ROX)**: Multiple nodes read-only
3. **ReadWriteMany (RWX)**: Multiple nodes read-write
4. **ReadWriteOncePod (RWOP)**: Single pod (K8s 1.27+)

**Reclaim Policies:**
1. **Retain**: Manual cleanup required
2. **Delete**: Auto-delete storage (default)
3. **Recycle**: Deprecated (basic scrub)

**Volume Binding Modes:**
1. **Immediate**: Bind as soon as PVC created
2. **WaitForFirstConsumer**: Wait until pod uses PVC (better for topology)

**Pod using PVC:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: storage
      mountPath: /data
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: pv-claim
```

**Production best practices:**
- Use StorageClass for dynamic provisioning
- Set appropriate access modes
- Use WaitForFirstConsumer for topology awareness
- Enable volume expansion
- Monitor storage usage
- Backup important data
- Use appropriate reclaim policy
- Set resource quotas for storage

**Follow-up**: How do you handle PVC expansion in production?

---

### Q22: What are Volume types in Kubernetes? Explain ephemeral vs persistent volumes.
**Expected Answer:**

**Volume Types:**

**Ephemeral Volumes** (lifecycle tied to pod)

**1. emptyDir**
```yaml
volumes:
- name: cache
  emptyDir:
    medium: Memory  # Or omit for disk
    sizeLimit: 1Gi
```
- Created when pod starts
- Deleted when pod removed
- Use: Scratch space, cache, temporary data

**2. configMap**
```yaml
volumes:
- name: config
  configMap:
    name: app-config
    items:
    - key: app.properties
      path: application.properties
```

**3. secret**
```yaml
volumes:
- name: certs
  secret:
    secretName: tls-secret
```

**4. downwardAPI**
```yaml
volumes:
- name: podinfo
  downwardAPI:
    items:
    - path: "labels"
      fieldRef:
        fieldPath: metadata.labels
    - path: "annotations"
      fieldRef:
        fieldPath: metadata.annotations
```

**Persistent Volumes**

**1. PersistentVolumeClaim**
```yaml
volumes:
- name: data
  persistentVolumeClaim:
    claimName: myapp-pvc
```

**2. Cloud Provider Volumes**

**AWS EBS**
```yaml
volumes:
- name: aws-ebs
  awsElasticBlockStore:
    volumeID: vol-12345678
    fsType: ext4
```

**GCE Persistent Disk**
```yaml
volumes:
- name: gce-pd
  gcePersistentDisk:
    pdName: my-disk
    fsType: ext4
```

**Azure Disk**
```yaml
volumes:
- name: azure-disk
  azureDisk:
    diskName: myDisk
    diskURI: /subscriptions/.../disks/myDisk
```

**Network Volumes**

**1. NFS**
```yaml
volumes:
- name: nfs
  nfs:
    server: nfs-server.example.com
    path: /exports/data
```

**2. Ceph RBD**
```yaml
volumes:
- name: ceph-rbd
  rbd:
    monitors:
    - '10.16.154.78:6789'
    - '10.16.154.82:6789'
    pool: kube
    image: foo
    fsType: ext4
    readOnly: false
```

**3. GlusterFS**
```yaml
volumes:
- name: glusterfs
  glusterfs:
    endpoints: glusterfs-cluster
    path: kube_vol
    readOnly: false
```

**CSI (Container Storage Interface)**
```yaml
volumes:
- name: csi-volume
  csi:
    driver: ebs.csi.aws.com
    volumeHandle: vol-12345678
    fsType: ext4
```

**Ephemeral vs Persistent:**

| Aspect | Ephemeral | Persistent |
|--------|-----------|------------|
| Lifecycle | Pod | Independent |
| Data persistence | Lost on pod deletion | Survives pod deletion |
| Sharing | Single pod | Multiple pods (if RWX) |
| Performance | Fast (local) | Varies |
| Use case | Cache, temp files | Databases, user data |
| Backup | Not needed | Required |

**Production patterns:**

**1. Stateless + ephemeral**
```yaml
# Web frontend
volumes:
- name: cache
  emptyDir: {}
- name: config
  configMap:
    name: app-config
```

**2. Stateful + persistent**
```yaml
# Database
volumeClaimTemplates:
- metadata:
    name: data
  spec:
    accessModes: ["ReadWriteOnce"]
    storageClassName: fast-ssd
    resources:
      requests:
        storage: 100Gi
```

**3. Hybrid**
```yaml
# Application with persistent data + cache
volumes:
- name: data
  persistentVolumeClaim:
    claimName: app-data
- name: cache
  emptyDir:
    medium: Memory
```

**Follow-up**: How do you handle data migration between different storage types?

---

### Q23: Explain CSI (Container Storage Interface). How is it better than in-tree volumes?
**Expected Answer:**

**CSI (Container Storage Interface)** - Standard for exposing storage systems to container orchestration platforms.

**In-tree vs CSI:**

**In-tree volumes (old way):**
- Code in Kubernetes core
- Tied to Kubernetes release cycle
- Limited to what Kubernetes developers implement
- Hard to add new storage systems
- Examples: AWS EBS, GCE PD, Azure Disk

**CSI (new way):**
- Out-of-tree plugin
- Independent release cycle
- Any vendor can write driver
- Easy to add new features
- Future of Kubernetes storage

**CSI Architecture:**
```
Pod → Kubelet → CSI Node Plugin
            ↓
        CSI Controller Plugin → Storage System
```

**CSI Components:**

**1. CSI Driver**
```yaml
apiVersion: storage.k8s.io/v1
kind: CSIDriver
metadata:
  name: ebs.csi.aws.com
spec:
  attachRequired: true
  podInfoOnMount: false
  volumeLifecycleModes:
  - Persistent
  - Ephemeral
```

**2. StorageClass (CSI)**
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-sc
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  encrypted: "true"
  kmsKeyId: arn:aws:kms:...
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
```

**3. Using CSI Volume**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ebs-claim
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: ebs-sc
  resources:
    requests:
      storage: 100Gi
```

**Popular CSI Drivers:**
- **AWS EBS CSI**: Amazon Elastic Block Store
- **GCP PD CSI**: Google Persistent Disk
- **Azure Disk CSI**: Azure managed disks
- **Ceph CSI**: Ceph RBD and CephFS
- **Longhorn**: Cloud-native distributed storage
- **OpenEBS**: Container-native storage
- **Portworx**: Enterprise storage platform
- **NetApp Trident**: NetApp storage

**CSI Features:**

**1. Volume Snapshots**
```yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: myapp-snapshot
spec:
  volumeSnapshotClassName: csi-snapclass
  source:
    persistentVolumeClaimName: myapp-pvc
```

**2. Volume Cloning**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: cloned-pvc
spec:
  storageClassName: ebs-sc
  dataSource:
    name: source-pvc
    kind: PersistentVolumeClaim
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 100Gi
```

**3. Volume Expansion**
```bash
# Edit PVC to increase size
kubectl edit pvc myapp-pvc
# Change storage: 100Gi to storage: 200Gi
```

**4. Ephemeral Volumes**
```yaml
volumes:
- name: scratch
  csi:
    driver: inline.storage.kubernetes.io
```

**5. Raw Block Volumes**
```yaml
spec:
  volumeMode: Block  # Instead of Filesystem
```

**Migration from in-tree to CSI:**
```yaml
# Enable CSIMigration feature gate
--feature-gates=CSIMigration=true,CSIMigrationAWS=true

# Old in-tree volume automatically uses CSI driver
```

**Production best practices:**
- Use CSI drivers instead of in-tree
- Enable volume snapshots for backups
- Test CSI driver before production
- Monitor CSI driver metrics
- Keep CSI drivers updated
- Use volume binding mode WaitForFirstConsumer
- Enable volume expansion

**Follow-up**: How do you backup and restore volumes using CSI snapshots?

---

## Configuration & Secrets

### Q24: ConfigMap vs Secret - What are the differences? How are secrets stored?
**Expected Answer:**

**ConfigMap vs Secret:**

| Aspect | ConfigMap | Secret |
|--------|-----------|--------|
| **Purpose** | Non-sensitive config | Sensitive data |
| **Storage** | Plain text in etcd | Base64 in etcd, encrypted optional |
| **Size limit** | 1MB | 1MB |
| **Immutable** | Optional (K8s 1.21+) | Optional (K8s 1.21+) |
| **Visibility** | `kubectl describe` shows data | `kubectl describe` hides data |
| **Use case** | Config files, env vars | Passwords, tokens, keys |

**Secret Storage & Security:**

**1. Storage in etcd:**
- Base64 encoded (NOT encrypted by default)
- Anyone with etcd access can read secrets
- Must enable encryption at rest

**2. Encryption at Rest:**
```yaml
# /etc/kubernetes/encryption-config.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
- resources:
  - secrets
  providers:
  - aescbc:
      keys:
      - name: key1
        secret: <base64-encoded-32-byte-key>
  - identity: {}  # Fallback to unencrypted
```

Enable in API server:
```bash
--encryption-provider-config=/etc/kubernetes/encryption-config.yaml
```

**3. Secret Types:**

```yaml
# Opaque (default) - arbitrary data
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: YWRtaW4=
  password: cGFzc3dvcmQ=

---
# TLS certificate
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
type: kubernetes.io/tls
data:
  tls.crt: <base64-cert>
  tls.key: <base64-key>

---
# Docker registry
apiVersion: v1
kind: Secret
metadata:
  name: registry-secret
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <base64-docker-config>

---
# Service Account Token
type: kubernetes.io/service-account-token

---
# Basic Auth
type: kubernetes.io/basic-auth
data:
  username: YWRtaW4=
  password: cGFzc3dvcmQ=

---
# SSH Auth
type: kubernetes.io/ssh-auth
data:
  ssh-privatekey: <base64-key>
```

**Using Secrets:**

**1. Environment Variables**
```yaml
env:
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: db-secret
      key: password
```

**2. Volume Mount**
```yaml
volumes:
- name: secret-volume
  secret:
    secretName: my-secret
    items:
    - key: username
      path: db-username
    - key: password
      path: db-password
volumeMounts:
- name: secret-volume
  mountPath: /etc/secrets
  readOnly: true
```

**3. imagePullSecrets**
```yaml
spec:
  imagePullSecrets:
  - name: registry-secret
```

**Immutable ConfigMaps/Secrets (K8s 1.21+):**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: immutable-secret
type: Opaque
immutable: true
data:
  key: value
```

**Benefits:**
- Prevents accidental updates
- Improves performance (kubelet doesn't watch)
- Reduces API server load

**Production Security Best Practices:**

**1. External Secret Management:**
```yaml
# Use External Secrets Operator
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: app-secret
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
    kind: SecretStore
  target:
    name: app-secret
  data:
  - secretKey: password
    remoteRef:
      key: prod/app/password
```

**2. Sealed Secrets (Bitnami):**
```bash
# Encrypt secret
kubectl create secret generic mysecret --dry-run=client --from-literal=password=mypass -o yaml | \
  kubeseal -o yaml > sealed-secret.yaml

# Safe to commit to Git
kubectl apply -f sealed-secret.yaml
```

**3. RBAC for Secrets:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  resourceNames: ["specific-secret"]  # Limit to specific secret
  verbs: ["get"]
```

**4. Secret Rotation:**
```bash
# Update secret
kubectl create secret generic db-secret \
  --from-literal=password=newpass \
  --dry-run=client -o yaml | kubectl apply -f -

# Restart pods to use new secret
kubectl rollout restart deployment myapp
```

**5. Audit Logging:**
```yaml
# Audit policy
- level: Metadata
  resources:
  - group: ""
    resources: ["secrets"]
```

**Security Checklist:**
- ✅ Enable encryption at rest
- ✅ Use external secret management (Vault, AWS Secrets Manager)
- ✅ Implement RBAC for secret access
- ✅ Rotate secrets regularly
- ✅ Use immutable secrets when possible
- ✅ Don't log secrets
- ✅ Audit secret access
- ✅ Use least privilege
- ✅ Scan images for hardcoded secrets

**Follow-up**: How do you implement automatic secret rotation without downtime?

---

### Q25: How do you manage application configuration across multiple environments?
**Expected Answer:**

**Multi-environment configuration strategies:**

**1. Namespace per Environment**
```
Cluster
├── dev namespace
├── staging namespace
└── prod namespace
```

**ConfigMap per environment:**
```yaml
# dev/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: dev
data:
  API_URL: "https://api-dev.example.com"
  LOG_LEVEL: "debug"
  CACHE_TTL: "60"

---
# prod/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: prod
data:
  API_URL: "https://api.example.com"
  LOG_LEVEL: "info"
  CACHE_TTL: "3600"
```

**2. Kustomize (Recommended)**
```
app/
├── base/
│   ├── kustomization.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   └── configmap.yaml
├── overlays/
│   ├── dev/
│   │   ├── kustomization.yaml
│   │   └── configmap.yaml
│   ├── staging/
│   │   ├── kustomization.yaml
│   │   └── configmap.yaml
│   └── prod/
│       ├── kustomization.yaml
│       ├── configmap.yaml
│       ├── replica-patch.yaml
│       └── resource-patch.yaml
```

**base/kustomization.yaml:**
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yaml
- service.yaml
- configmap.yaml
```

**overlays/prod/kustomization.yaml:**
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
bases:
- ../../base
namespace: production
replicas:
- name: myapp
  count: 5
configMapGenerator:
- name: app-config
  behavior: merge
  literals:
  - API_URL=https://api.example.com
  - LOG_LEVEL=info
patchesStrategicMerge:
- replica-patch.yaml
- resource-patch.yaml
```

**Deploy:**
```bash
kubectl apply -k overlays/dev
kubectl apply -k overlays/prod
```

**3. Helm Charts**
```
chart/
├── Chart.yaml
├── values.yaml              # Default values
├── values-dev.yaml          # Dev overrides
├── values-staging.yaml      # Staging overrides
├── values-prod.yaml         # Prod overrides
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    └── configmap.yaml
```

**values-prod.yaml:**
```yaml
replicaCount: 5

config:
  apiUrl: "https://api.example.com"
  logLevel: "info"
  cacheTTL: 3600

resources:
  limits:
    cpu: 1000m
    memory: 2Gi
  requests:
    cpu: 500m
    memory: 1Gi

autoscaling:
  enabled: true
  minReplicas: 5
  maxReplicas: 20
```

**Deploy:**
```bash
helm install myapp ./chart -f values-prod.yaml -n production
helm upgrade myapp ./chart -f values-prod.yaml -n production
```

**4. GitOps with ArgoCD**
```yaml
# dev-application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-dev
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/org/repo
    targetRevision: main
    path: overlays/dev
  destination:
    server: https://kubernetes.default.svc
    namespace: dev
  syncPolicy:
    automated:
      prune: true
      selfHeal: true

---
# prod-application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-prod
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/org/repo
    targetRevision: release
    path: overlays/prod
  destination:
    server: https://kubernetes.default.svc
    namespace: prod
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

**5. External Configuration (Production Pattern)**

**Using ConfigMap Reloader:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  annotations:
    configmap.reloader.stakater.com/reload: "app-config"
spec:
  template:
    spec:
      containers:
      - name: app
        image: myapp:v1
        envFrom:
        - configMapRef:
            name: app-config
```

**Using External Configuration Service:**
```yaml
# Spring Cloud Config, Consul, etcd
env:
- name: SPRING_CLOUD_CONFIG_URI
  value: "http://config-server:8888"
- name: SPRING_PROFILES_ACTIVE
  value: "production"
```

**6. Feature Flags**
```yaml
# ConfigMap with feature flags
apiVersion: v1
kind: ConfigMap
metadata:
  name: feature-flags
data:
  features.json: |
    {
      "newFeature": {
        "enabled": true,
        "rollout": 50
      },
      "betaFeature": {
        "enabled": false
      }
    }
```

**Production Best Practices:**

**1. Configuration hierarchy:**
```
Default values (in code)
  ↓
ConfigMap (environment-specific)
  ↓
Secrets (sensitive per-environment)
  ↓
Command-line args (override)
```

**2. Validation:**
```yaml
# Init container to validate config
initContainers:
- name: config-validator
  image: config-validator:v1
  command: ['validate', '/config/app.yaml']
  volumeMounts:
  - name: config
    mountPath: /config
```

**3. Version Control:**
```
Git Repository
├── apps/
│   └── myapp/
│       ├── base/          # Base manifests
│       └── overlays/
│           ├── dev/       # Dev config
│           ├── staging/   # Staging config
│           └── prod/      # Prod config (may be separate repo)
```

**4. Change Management:**
- Use GitOps (all changes via Git)
- Require PR reviews for prod changes
- Automated testing before merge
- Gradual rollout (dev → staging → prod)
- Rollback procedures documented

**5. Security:**
- Secrets in external vaults (Vault, AWS Secrets Manager)
- Encrypt secrets in Git (Sealed Secrets, SOPS)
- Separate repositories for prod secrets
- Audit all configuration changes
- Use RBAC to limit access

**Follow-up**: How do you handle database connection strings across environments?

---

## Security & RBAC

### Q26: Explain RBAC in Kubernetes. What's the difference between Role and ClusterRole?
**Expected Answer:**

**RBAC (Role-Based Access Control)** - Authorization mechanism to regulate access.

**Key concepts:**

**1. Role vs ClusterRole**

| Aspect | Role | ClusterRole |
|--------|------|-------------|
| Scope | Namespace | Cluster-wide |
| Resources | Namespaced resources | All resources |
| Binding | RoleBinding | ClusterRoleBinding |
| Use case | App-specific | Cluster admin, node access |

**Role (namespace-scoped):**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
- apiGroups: [""]  # "" = core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get"]
```

**ClusterRole (cluster-wide):**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["persistentvolumes"]
  verbs: ["get", "list"]
```

**2. RoleBinding vs ClusterRoleBinding**

**RoleBinding (binds in namespace):**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane@example.com
  apiGroup: rbac.authorization.k8s.io
- kind: ServiceAccount
  name: myapp
  namespace: default
- kind: Group
  name: developers
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

**ClusterRoleBinding (cluster-wide):**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-nodes
subjects:
- kind: User
  name: admin@example.com
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: node-reader
  apiGroup: rbac.authorization.k8s.io
```

**3. Subjects (who gets access):**
- **User**: Human user (from certificate CN)
- **Group**: Set of users (from certificate O)
- **ServiceAccount**: Pod identity

**4. Resources & Verbs:**

**Common verbs:**
```yaml
verbs:
- get            # Read single resource
- list           # List resources
- watch          # Watch for changes
- create         # Create new
- update         # Update existing
- patch          # Partial update
- delete         # Delete single
- deletecollection  # Delete multiple
```

**Resource examples:**
```yaml
resources:
- pods
- services
- deployments
- configmaps
- secrets
- nodes
- persistentvolumes
- namespaces
```

**Subresources:**
```yaml
resources:
- pods/log         # Pod logs
- pods/exec        # Execute in pod
- pods/portforward # Port forwarding
- deployments/scale  # Scaling
- services/proxy   # Service proxy
```

**5. Production RBAC Patterns:**

**Developer Role:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
  namespace: development
rules:
# Full access to most resources
- apiGroups: ["", "apps", "batch"]
  resources:
  - pods
  - pods/log
  - pods/exec
  - deployments
  - services
  - configmaps
  - jobs
  - cronjobs
  verbs: ["*"]
# Read-only secrets
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "list"]
# No access to resource quotas
```

**DevOps/SRE Role:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: devops
rules:
# Full access to workloads
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
# Except RBAC changes
- apiGroups: ["rbac.authorization.k8s.io"]
  resources: ["roles", "rolebindings", "clusterroles", "clusterrolebindings"]
  verbs: ["get", "list", "watch"]
```

**Read-Only Auditor:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: auditor
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["get", "list", "watch"]
```

**CI/CD Service Account:**
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cicd
  namespace: production
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: cicd-deployer
  namespace: production
rules:
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "update", "patch"]
- apiGroups: [""]
  resources: ["services"]
  verbs: ["get", "list"]
- apiGroups: [""]
  resources: ["configmaps"]
  resourceNames: ["app-config"]  # Specific ConfigMap only
  verbs: ["get", "update"]
```

**6. Testing RBAC:**
```bash
# Check if you can perform action
kubectl auth can-i create deployments
kubectl auth can-i delete pods --namespace=production

# Check as different user
kubectl auth can-i get pods --as=jane@example.com
kubectl auth can-i create secrets --as=system:serviceaccount:default:myapp

# Check all permissions
kubectl auth can-i --list
kubectl auth can-i --list --as=jane@example.com

# Dry-run impersonation
kubectl get pods --as=jane@example.com
kubectl get pods --as=system:serviceaccount:default:myapp
```

**7. Aggregated ClusterRoles:**
```yaml
# Base role
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: monitoring-reader
  labels:
    rbac.example.com/aggregate-to-monitoring: "true"
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list", "watch"]

---
# Aggregated role
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: monitoring
aggregationRule:
  clusterRoleSelectors:
  - matchLabels:
      rbac.example.com/aggregate-to-monitoring: "true"
rules: []  # Automatically filled
```

**Production Best Practices:**
- ✅ Principle of least privilege
- ✅ Use ServiceAccounts for pods (not default)
- ✅ Separate roles by responsibility
- ✅ Document RBAC policies
- ✅ Regular RBAC audits
- ✅ Use groups instead of individual users
- ✅ Test RBAC before applying
- ✅ Use resource names for sensitive resources
- ✅ Disable automountServiceAccountToken when not needed
- ✅ Monitor RBAC violations

**Common Pitfalls:**
- ❌ Using `verbs: ["*"]` unnecessarily
- ❌ Granting cluster-admin to everything
- ❌ Not testing RBAC changes
- ❌ Forgotten ClusterRoleBindings
- ❌ ServiceAccount token auto-mount
- ❌ No RBAC auditing

**Follow-up**: How do you implement RBAC for a multi-tenant cluster?

---

(Continue with remaining questions...)

### Q27: What are Pod Security Policies/Standards? How do you implement them?
**Expected Answer:**

**Pod Security** enforces security best practices at pod level.

**Evolution:**
1. **PodSecurityPolicy (PSP)** - Deprecated in 1.21, removed in 1.25
2. **Pod Security Admission (PSA)** - Current, built-in (1.23+)
3. **Pod Security Standards (PSS)** - Three profiles

**Pod Security Standards:**

**1. Privileged** - Unrestricted (for system pods)
```yaml
# No restrictions
# Use for: CNI, CSI, logging daemons
```

**2. Baseline** - Minimally restrictive
```yaml
# Prevents:
- Privileged containers
- HostNetwork, HostPID, HostIPC
- Privileged ports (< 1024)
- HostPath volumes (most types)
- Adding capabilities (except NET_BIND_SERVICE)
- Host namespaces
```

**3. Restricted** - Heavily restricted (recommended)
```yaml
# Additionally prevents:
- Running as root
- Privilege escalation
- All capabilities dropped
- ReadOnlyRootFilesystem required
- Seccomp profile required
- Non-default proc mount
```

**Implementation Methods:**

**Method 1: Namespace Labels (Pod Security Admission)**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    # Enforce restricted profile
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/enforce-version: latest
    
    # Audit violations but don't block
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/audit-version: latest
    
    # Warn about violations
    pod-security.kubernetes.io/warn: restricted
    pod-security.kubernetes.io/warn-version: latest
```

**Method 2: AdmissionConfiguration**
```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
- name: PodSecurity
  configuration:
    apiVersion: pod-security.admission.config.k8s.io/v1
    kind: PodSecurityConfiguration
    defaults:
      enforce: "baseline"
      enforce-version: "latest"
      audit: "restricted"
      audit-version: "latest"
      warn: "restricted"
      warn-version: "latest"
    exemptions:
      usernames: []
      runtimeClasses: []
      namespaces: ["kube-system", "monitoring"]
```

**Compliant Pod Example:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  # Security Context (Pod level)
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
    seccompProfile:
      type: RuntimeDefault
  
  containers:
  - name: app
    image: myapp:v1
    
    # Security Context (Container level)
    securityContext:
      runAsNonRoot: true
      runAsUser: 1000
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
    
    # Volumes for writable directories
    volumeMounts:
    - name: cache
      mountPath: /cache
    - name: tmp
      mountPath: /tmp
  
  volumes:
  - name: cache
    emptyDir: {}
  - name: tmp
    emptyDir: {}
```

**Method 3: OPA Gatekeeper (Advanced Policy Engine)**

**Install Gatekeeper:**
```bash
kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/master/deploy/gatekeeper.yaml
```

**Constraint Template:**
```yaml
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8sblockprivileged
spec:
  crd:
    spec:
      names:
        kind: K8sBlockPrivileged
  targets:
  - target: admission.k8s.gatekeeper.sh
    rego: |
      package k8sblockprivileged
      
      violation[{"msg": msg}] {
        container := input.review.object.spec.containers[_]
        container.securityContext.privileged
        msg := sprintf("Privileged container is not allowed: %v", [container.name])
      }
      
      violation[{"msg": msg}] {
        not has_securitycontext
        msg := "Containers must have securityContext"
      }
      
      has_securitycontext {
        input.review.object.spec.containers[_].securityContext
      }
```

**Constraint:**
```yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sBlockPrivileged
metadata:
  name: block-privileged-containers
spec:
  match:
    kinds:
    - apiGroups: [""]
      kinds: ["Pod"]
    excludedNamespaces: ["kube-system"]
```

**Method 4: Kyverno (Kubernetes Native Policy)**

**Install Kyverno:**
```bash
helm install kyverno kyverno/kyverno -n kyverno --create-namespace
```

**Policy:**
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-non-root
spec:
  validationFailureAction: enforce
  rules:
  - name: check-runAsNonRoot
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
      message: "Running as root is not allowed"
      pattern:
        spec:
          securityContext:
            runAsNonRoot: true
          containers:
          - securityContext:
              runAsNonRoot: true
              allowPrivilegeEscalation: false
              capabilities:
                drop:
                - ALL

---
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-readonly-rootfs
spec:
  validationFailureAction: enforce
  rules:
  - name: check-readonly-rootfs
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
      message: "Root filesystem must be read-only"
      pattern:
        spec:
          containers:
          - securityContext:
              readOnlyRootFilesystem: true
```

**Production Security Checklist:**

**Pod Level:**
```yaml
spec:
  # ✅ Run as non-root
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
    
    # ✅ Use seccomp
    seccompProfile:
      type: RuntimeDefault
      # OR: type: Localhost, localhostProfile: profiles/audit.json
  
  # ✅ Disable service account token
  automountServiceAccountToken: false
  
  # ✅ Use specific service account
  serviceAccountName: myapp-sa
  
  # ✅ Set host properties (if needed)
  hostNetwork: false
  hostPID: false
  hostIPC: false
  
  containers:
  - name: app
    image: myapp:v1
    
    # ✅ Container security context
    securityContext:
      runAsNonRoot: true
      runAsUser: 1000
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      
      # ✅ Drop all capabilities
      capabilities:
        drop:
        - ALL
        # Add only if absolutely needed
        add:
        - NET_BIND_SERVICE
    
    # ✅ Resource limits
    resources:
      limits:
        cpu: "1"
        memory: "1Gi"
      requests:
        cpu: "100m"
        memory: "128Mi"
```

**Validation Strategy:**
```bash
# Test pod against policies
kubectl run test --image=nginx --dry-run=server

# Check why pod was rejected
kubectl describe pod test

# Validate manifest before applying
kubectl apply --dry-run=server -f pod.yaml
```

**Migration Strategy:**
1. **Audit** - Enable audit mode, monitor violations
2. **Warn** - Enable warn mode, notify users
3. **Enforce** - Gradually enforce, namespace by namespace
4. **Fix** - Update applications to comply

**Follow-up**: How do you handle legacy applications that can't run as non-root?

---

### Q28: How do you implement mutual TLS (mTLS) between services?
**Expected Answer:**

**mTLS** provides encrypted, authenticated communication between services.

**Implementation Options:**

**1. Service Mesh (Istio) - Automatic mTLS**
```yaml
# PeerAuthentication - Enforce mTLS
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: production
spec:
  mtls:
    mode: STRICT  # STRICT, PERMISSIVE, or DISABLE


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

