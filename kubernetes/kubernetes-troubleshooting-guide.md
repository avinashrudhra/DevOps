# Kubernetes Production Troubleshooting Guide

## Table of Contents
1. [Pod Issues](#pod-issues)
2. [Service & Networking Issues](#service--networking-issues)
3. [Storage Issues](#storage-issues)
4. [Node Issues](#node-issues)
5. [Performance Issues](#performance-issues)
6. [Security Issues](#security-issues)
7. [Cluster Issues](#cluster-issues)
8. [Application Issues](#application-issues)

---

## Pod Issues

### Issue 1: Pod Stuck in Pending
**Symptoms**: Pod remains in Pending state indefinitely

**Common Causes**:
- Insufficient CPU/Memory on nodes
- PersistentVolumeClaim not bound
- Node selector doesn't match any nodes
- Taints without tolerations
- Resource quotas exceeded

**Debugging Steps**:
```bash
# Check pod status and events
kubectl describe pod <pod-name>

# Check node resources
kubectl top nodes
kubectl describe nodes | grep -A 5 "Allocated resources"

# Check PVC status
kubectl get pvc
kubectl describe pvc <pvc-name>

# Check resource quotas
kubectl describe resourcequota -n <namespace>

# Check if nodes have matching labels
kubectl get nodes --show-labels
kubectl get pod <pod-name> -o jsonpath='{.spec.nodeSelector}'

# Check node taints
kubectl describe nodes | grep -i taint
```

**Solutions**:
```bash
# If resource issue: Add more nodes or reduce requests
# If PVC issue: Check storage provisioner
kubectl get storageclass
kubectl logs -n kube-system -l app=storage-provisioner

# If node selector issue: Fix labels
kubectl label nodes <node-name> <key>=<value>

# If taint issue: Add toleration or remove taint
kubectl taint nodes <node-name> <key>:<effect>-
```

---

### Issue 2: CrashLoopBackOff
**Symptoms**: Pod continuously crashes and restarts

**Common Causes**:
- Application error/crash
- Missing dependencies (ConfigMap/Secret)
- Incorrect command/args
- Liveness probe failing too quickly
- Missing environment variables
- Port already in use

**Debugging Steps**:
```bash
# Check current logs
kubectl logs <pod-name>

# Check previous container logs (most important!)
kubectl logs <pod-name> --previous

# Check events
kubectl describe pod <pod-name>

# Check if ConfigMap/Secret exists
kubectl get configmap
kubectl get secret

# Check environment variables
kubectl exec <pod-name> -- env

# Check running processes
kubectl exec <pod-name> -- ps aux
```

**Solutions**:
```yaml
# Fix 1: Increase initialDelaySeconds for probes
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 60  # Increase this
  periodSeconds: 10
  failureThreshold: 3

# Fix 2: Add missing ConfigMap/Secret
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  config.json: |
    { "key": "value" }

# Fix 3: Fix command
spec:
  containers:
  - name: app
    image: myapp:v1
    command: ["./app"]  # Correct command
    args: ["--port=8080"]
```

---

### Issue 3: ImagePullBackOff
**Symptoms**: Cannot pull container image

**Common Causes**:
- Wrong image name or tag
- Image doesn't exist
- Private registry without imagePullSecrets
- Network issues
- Rate limiting

**Debugging Steps**:
```bash
# Check pod events
kubectl describe pod <pod-name>

# Check image name
kubectl get pod <pod-name> -o jsonpath='{.spec.containers[*].image}'

# Check if imagePullSecret is configured
kubectl get pod <pod-name> -o jsonpath='{.spec.imagePullSecrets}'

# Check secret
kubectl get secret <secret-name> -o yaml
```

**Solutions**:
```bash
# Create docker registry secret
kubectl create secret docker-registry regcred \
  --docker-server=<registry-url> \
  --docker-username=<username> \
  --docker-password=<password> \
  --docker-email=<email>

# Add to deployment
spec:
  imagePullSecrets:
  - name: regcred
  containers:
  - name: app
    image: registry.example.com/myapp:v1

# Or fix image name/tag
kubectl set image deployment/<name> <container>=<correct-image>
```

---

### Issue 4: OOMKilled (Out of Memory)
**Symptoms**: Pod killed due to memory limit exceeded

**Debugging Steps**:
```bash
# Check if pod was OOMKilled
kubectl describe pod <pod-name> | grep -i "OOMKilled"

# Check memory limits
kubectl get pod <pod-name> -o jsonpath='{.spec.containers[*].resources}'

# Check memory usage
kubectl top pod <pod-name>

# Check events
kubectl get events --field-selector involvedObject.name=<pod-name>
```

**Solutions**:
```yaml
# Solution 1: Increase memory limits
spec:
  containers:
  - name: app
    resources:
      requests:
        memory: "256Mi"
      limits:
        memory: "512Mi"  # Increase this

# Solution 2: Fix memory leak in application

# Solution 3: Use VPA for automatic adjustment
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  updatePolicy:
    updateMode: "Auto"
```

---

### Issue 5: CreateContainerConfigError
**Symptoms**: Error creating container configuration

**Common Causes**:
- Referenced ConfigMap doesn't exist
- Referenced Secret doesn't exist
- Wrong key name in ConfigMap/Secret

**Debugging Steps**:
```bash
# Check pod events
kubectl describe pod <pod-name>

# List ConfigMaps
kubectl get configmap

# List Secrets
kubectl get secret

# Check ConfigMap content
kubectl describe configmap <configmap-name>

# Check Secret content
kubectl describe secret <secret-name>
```

**Solutions**:
```bash
# Create missing ConfigMap
kubectl create configmap app-config --from-literal=key=value

# Create missing Secret
kubectl create secret generic app-secret --from-literal=password=secret

# Or fix the reference
spec:
  containers:
  - name: app
    env:
    - name: CONFIG
      valueFrom:
        configMapKeyRef:
          name: app-config  # Must exist
          key: key          # Must be correct key name
```

---

## Service & Networking Issues

### Issue 6: Service Not Accessible
**Symptoms**: Cannot access service from within cluster

**Debugging Steps**:
```bash
# 1. Check service exists
kubectl get svc <service-name>
kubectl describe svc <service-name>

# 2. Check endpoints (most important!)
kubectl get endpoints <service-name>
# Empty endpoints = selector doesn't match pods

# 3. Check pod labels
kubectl get pods --show-labels

# 4. Check service selector
kubectl get svc <service-name> -o jsonpath='{.spec.selector}'

# 5. Test DNS resolution
kubectl run test --image=busybox -it --rm -- nslookup <service-name>

# 6. Test connectivity
kubectl run test --image=busybox -it --rm -- wget -O- --timeout=5 http://<service-name>

# 7. Check NetworkPolicies
kubectl get networkpolicies
kubectl describe networkpolicy <policy-name>
```

**Solutions**:
```bash
# Fix 1: Correct label selector
kubectl patch service <service-name> -p '{"spec":{"selector":{"app":"correct-label"}}}'

# Fix 2: Add labels to pods
kubectl label pods <pod-name> app=myapp

# Fix 3: Create service if missing
kubectl expose deployment <deployment-name> --port=80 --target-port=8080

# Fix 4: Update NetworkPolicy
kubectl delete networkpolicy <policy-name>
# Or allow traffic from specific pods
```

---

### Issue 7: Ingress Not Working
**Symptoms**: Cannot access application via Ingress

**Debugging Steps**:
```bash
# Check Ingress exists
kubectl get ingress
kubectl describe ingress <ingress-name>

# Check Ingress controller is running
kubectl get pods -n ingress-nginx
kubectl logs -n ingress-nginx <ingress-controller-pod>

# Check service backends
kubectl get svc
kubectl get endpoints

# Check Ingress address
kubectl get ingress <ingress-name> -o jsonpath='{.status.loadBalancer.ingress}'

# Test from within cluster
kubectl run test --image=curlimages/curl -it --rm -- curl -H "Host: myapp.example.com" http://<service-ip>
```

**Solutions**:
```yaml
# Ensure Ingress is properly configured
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx  # Make sure this matches your controller
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port:
              number: 80
```

```bash
# Restart Ingress controller if needed
kubectl rollout restart deployment -n ingress-nginx ingress-nginx-controller
```

---

### Issue 8: DNS Resolution Failures
**Symptoms**: Pods cannot resolve service names

**Debugging Steps**:
```bash
# Test DNS from pod
kubectl exec -it <pod-name> -- nslookup kubernetes.default
kubectl exec -it <pod-name> -- nslookup <service-name>

# Check CoreDNS is running
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns

# Check DNS configuration in pod
kubectl exec <pod-name> -- cat /etc/resolv.conf

# Check CoreDNS ConfigMap
kubectl get configmap coredns -n kube-system -o yaml
```

**Solutions**:
```bash
# Restart CoreDNS
kubectl rollout restart deployment coredns -n kube-system

# Check/Fix DNS policy
spec:
  dnsPolicy: ClusterFirst  # Should be this for service discovery

# Manually specify DNS
spec:
  dnsConfig:
    nameservers:
    - 8.8.8.8
    searches:
    - default.svc.cluster.local
    - svc.cluster.local
    - cluster.local
```

---

### Issue 9: Network Policy Blocking Traffic
**Symptoms**: Pods cannot communicate despite service configuration

**Debugging Steps**:
```bash
# List NetworkPolicies
kubectl get networkpolicies

# Describe policy
kubectl describe networkpolicy <policy-name>

# Check if default deny-all exists
kubectl get networkpolicy -o yaml | grep -A 10 "podSelector: {}"

# Test connectivity
kubectl exec -it <source-pod> -- nc -zv <target-service> <port>
```

**Solutions**:
```yaml
# Allow specific traffic
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

# Allow DNS (always needed)
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: UDP
      port: 53
```

---

## Storage Issues

### Issue 10: PVC Pending
**Symptoms**: PersistentVolumeClaim stuck in Pending

**Debugging Steps**:
```bash
# Check PVC status
kubectl get pvc
kubectl describe pvc <pvc-name>

# Check PV
kubectl get pv

# Check StorageClass
kubectl get storageclass
kubectl describe storageclass <storageclass-name>

# Check provisioner logs
kubectl logs -n kube-system -l app=storage-provisioner
```

**Solutions**:
```bash
# Solution 1: Create PV manually if no dynamic provisioner
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-volume
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce
  hostPath:
    path: /data

# Solution 2: Fix StorageClass
kubectl get sc
kubectl patch storageclass <name> -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'

# Solution 3: Check if enough storage available
kubectl describe nodes | grep -A 5 "Storage"
```

---

### Issue 11: Volume Mount Failures
**Symptoms**: Pod cannot mount volume

**Debugging Steps**:
```bash
# Check pod events
kubectl describe pod <pod-name>

# Check PVC status
kubectl get pvc <pvc-name>

# Check if volume is already attached to another node
kubectl get volumeattachment

# Check CSI driver logs (if using CSI)
kubectl logs -n kube-system -l app=csi-driver
```

**Solutions**:
```bash
# If RWO (ReadWriteOnce) volume is attached elsewhere, delete the old pod first

# Check access mode matches usage
spec:
  accessModes:
  - ReadWriteOnce   # Single node
  # OR
  - ReadWriteMany   # Multiple nodes
  # OR
  - ReadOnlyMany    # Multiple nodes read-only
```

---

## Node Issues

### Issue 12: Node NotReady
**Symptoms**: Node showing NotReady status

**Debugging Steps**:
```bash
# Check node status
kubectl get nodes
kubectl describe node <node-name>

# Check node conditions
kubectl get nodes -o json | jq '.items[] | {name: .metadata.name, conditions: .status.conditions}'

# Check system pods on the node
kubectl get pods -n kube-system -o wide | grep <node-name>

# SSH to node (if possible) and check:
# - kubelet status
systemctl status kubelet
journalctl -u kubelet -f

# - Container runtime
systemctl status docker  # or containerd

# - Disk space
df -h

# - Memory
free -m
```

**Solutions**:
```bash
# Restart kubelet (on node)
systemctl restart kubelet

# Restart container runtime
systemctl restart docker  # or containerd

# Clear disk space if full
docker system prune -a

# Uncordon if node was cordoned
kubectl uncordon <node-name>
```

---

### Issue 13: Node Disk Pressure
**Symptoms**: Pods being evicted due to disk pressure

**Debugging Steps**:
```bash
# Check node conditions
kubectl describe node <node-name> | grep -i "DiskPressure"

# Check disk usage
kubectl exec -it <pod-on-node> -- df -h

# SSH to node
df -h
du -sh /var/lib/docker/*
du -sh /var/lib/kubelet/*
```

**Solutions**:
```bash
# Clean up unused images (on node)
docker system prune -a

# Clean up logs
find /var/log -name "*.log" -mtime +7 -delete

# Increase disk size or add volume

# Set eviction thresholds (kubelet config)
evictionHard:
  nodefs.available: "10%"
  imagefs.available: "15%"
```

---

## Performance Issues

### Issue 14: High CPU Usage
**Symptoms**: Pods consuming excessive CPU

**Debugging Steps**:
```bash
# Check pod CPU usage
kubectl top pods
kubectl top pods --containers

# Check node CPU
kubectl top nodes

# Get detailed metrics
kubectl get --raw /apis/metrics.k8s.io/v1beta1/pods

# Check for CPU throttling
kubectl describe pod <pod-name> | grep -i "cpu"

# Check application metrics (if available)
kubectl port-forward <pod-name> 9090:9090
# Access Prometheus metrics
```

**Solutions**:
```yaml
# Solution 1: Increase CPU limits
resources:
  requests:
    cpu: "500m"
  limits:
    cpu: "2"      # Increase this

# Solution 2: Enable HPA
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70

# Solution 3: Optimize application code
```

---

### Issue 15: Slow Application Response
**Symptoms**: Application responding slowly

**Debugging Steps**:
```bash
# Check if pods are being throttled
kubectl top pods
kubectl describe hpa

# Check readiness probe
kubectl describe pod <pod-name> | grep -A 10 "Readiness"

# Check for pod restarts
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.containerStatuses[*].restartCount}{"\n"}{end}'

# Check network latency
kubectl run test --image=nicolaka/netshoot -it --rm -- ping <service-ip>

# Check database connections
kubectl logs <pod-name> | grep -i "connection"

# Check application metrics
kubectl port-forward <pod-name> 8080:8080
# Access /metrics endpoint
```

**Solutions**:
```yaml
# Fix 1: Adjust probe timings
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
  timeoutSeconds: 3      # Increase if slow
  failureThreshold: 3

# Fix 2: Add more replicas
kubectl scale deployment <name> --replicas=5

# Fix 3: Implement caching
# Add Redis cache

# Fix 4: Use connection pooling
# Configure database connection pool in application

# Fix 5: Add resource limits
resources:
  requests:
    cpu: "500m"
    memory: "512Mi"
  limits:
    cpu: "1"
    memory: "1Gi"
```

---

## Security Issues

### Issue 16: Unauthorized Access
**Symptoms**: User cannot access resources

**Debugging Steps**:
```bash
# Check if user can perform action
kubectl auth can-i create pods
kubectl auth can-i get pods --namespace=dev

# Check as specific user
kubectl auth can-i get pods --as=user@example.com

# Check as service account
kubectl auth can-i get pods --as=system:serviceaccount:default:myapp

# List roles
kubectl get roles,rolebindings -n <namespace>
kubectl get clusterroles,clusterrolebindings

# Describe role
kubectl describe role <role-name>
kubectl describe rolebinding <rolebinding-name>
```

**Solutions**:
```yaml
# Create Role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]

---
# Create RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: user@example.com
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

---

### Issue 17: Secret Exposure
**Symptoms**: Secrets accidentally exposed

**Solutions**:
```bash
# Rotate secrets immediately
kubectl create secret generic new-secret --from-literal=password=newpassword

# Update deployments
kubectl set env deployment/<name> PASSWORD=<new-value>

# Use external secret management
# - HashiCorp Vault
# - AWS Secrets Manager
# - Azure Key Vault

# Scan for exposed secrets
trivy k8s --report summary cluster
```

---

## Cluster Issues

### Issue 18: API Server Unreachable
**Symptoms**: Cannot connect to cluster

**Debugging Steps**:
```bash
# Check kubectl config
kubectl config view
kubectl cluster-info

# Test API server connectivity
curl -k https://<api-server>:6443/healthz

# Check certificate validity
openssl s_client -connect <api-server>:6443 2>/dev/null | openssl x509 -noout -dates

# Check kubeconfig
cat ~/.kube/config
```

**Solutions**:
```bash
# Update kubeconfig
kubectl config set-cluster <cluster-name> --server=https://<new-api-server>:6443

# Merge kubeconfigs
KUBECONFIG=~/.kube/config:~/.kube/config2 kubectl config view --flatten > ~/.kube/config-merged

# Check if API server pods are running (on master)
kubectl get pods -n kube-system
```

---

### Issue 19: etcd Issues
**Symptoms**: Cluster state inconsistent

**Debugging Steps**:
```bash
# Check etcd pods
kubectl get pods -n kube-system -l component=etcd

# Check etcd health (from master node)
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  endpoint health

# Check etcd logs
kubectl logs -n kube-system etcd-<master-node>
```

**Solutions**:
```bash
# Backup etcd
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot.db

# Restore etcd
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-snapshot.db

# Restart etcd (if needed)
systemctl restart etcd
```

---

## Application Issues

### Issue 20: Database Connection Failures
**Symptoms**: Application cannot connect to database

**Debugging Steps**:
```bash
# Test DNS resolution
kubectl exec -it <app-pod> -- nslookup <db-service>

# Test connectivity
kubectl exec -it <app-pod> -- nc -zv <db-service> 5432

# Check database pod
kubectl get pods -l app=database
kubectl logs <db-pod>

# Check database service
kubectl get svc <db-service>
kubectl get endpoints <db-service>

# Check credentials
kubectl get secret <db-secret> -o yaml
```

**Solutions**:
```bash
# Verify connection string
kubectl exec -it <app-pod> -- env | grep DATABASE

# Test connection manually
kubectl exec -it <db-pod> -- psql -U postgres -c "SELECT 1"

# Check network policy
kubectl get networkpolicy
# Ensure app can access database
```

---

## Debugging Tools & Commands Reference

### Essential Commands
```bash
# Get everything
kubectl get all -A

# Watch resources
kubectl get pods -w

# Sort by creation time
kubectl get pods --sort-by=.metadata.creationTimestamp

# Get with custom columns
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName

# JSON path queries
kubectl get pods -o jsonpath='{.items[*].metadata.name}'

# Show labels
kubectl get pods --show-labels

# Filter by label
kubectl get pods -l app=myapp,env=prod

# Get events sorted
kubectl get events --sort-by=.metadata.creationTimestamp
```

### Advanced Debugging
```bash
# Debug with ephemeral container (K8s 1.23+)
kubectl debug <pod-name> -it --image=busybox --target=<container-name>

# Debug node
kubectl debug node/<node-name> -it --image=ubuntu

# Copy pod for debugging
kubectl debug <pod-name> -it --copy-to=<new-pod-name> --container=<container-name>

# Run privileged container on node
kubectl run debug --image=nicolaka/netshoot --rm -it --privileged -- /bin/bash
```

### Logging
```bash
# Follow logs from multiple pods
kubectl logs -f -l app=myapp --all-containers=true

# Logs with timestamps
kubectl logs <pod-name> --timestamps

# Logs since time
kubectl logs <pod-name> --since-time=2023-01-01T00:00:00Z

# Logs with grep
kubectl logs <pod-name> | grep ERROR
```

---

## Production Incident Response Checklist

### Immediate Actions
1. [ ] Identify affected services
2. [ ] Check recent changes (deployments, configs)
3. [ ] Review monitoring dashboards
4. [ ] Check logs for errors
5. [ ] Verify external dependencies

### Investigation
1. [ ] Run `kubectl get pods -A` to see overall status
2. [ ] Check events: `kubectl get events --sort-by=.metadata.creationTimestamp`
3. [ ] Review logs: `kubectl logs <pod-name> --previous`
4. [ ] Check resource usage: `kubectl top nodes` and `kubectl top pods`
5. [ ] Verify networking: Test service connectivity

### Mitigation
1. [ ] Scale up healthy pods if possible
2. [ ] Rollback recent changes if suspected
3. [ ] Adjust resource limits if resource issue
4. [ ] Restart pods if transient issue
5. [ ] Implement workaround if available

### Resolution
1. [ ] Identify root cause
2. [ ] Implement permanent fix
3. [ ] Test thoroughly
4. [ ] Deploy fix
5. [ ] Monitor for stability

### Post-Incident
1. [ ] Document incident
2. [ ] Perform root cause analysis
3. [ ] Update runbooks
4. [ ] Implement preventive measures
5. [ ] Share learnings with team

---

## Useful Tools

### CLI Tools
- **kubectl** - Official CLI
- **k9s** - Terminal UI
- **stern** - Multi-pod log viewer
- **kubectx/kubens** - Context/namespace switcher
- **krew** - kubectl plugin manager

### Debugging Tools
- **nicolaka/netshoot** - Network debugging
- **busybox** - Basic debugging
- **curl/wget** - HTTP testing
- **dig/nslookup** - DNS debugging

### Monitoring Tools
- **Prometheus** - Metrics collection
- **Grafana** - Visualization
- **Lens** - Desktop Kubernetes IDE
- **Octant** - Web UI

---

**Remember**: Always check the basics first (pods, events, logs) before diving into complex debugging!

Reference: [kubernetes.io](https://kubernetes.io/)


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

