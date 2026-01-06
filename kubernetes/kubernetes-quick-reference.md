# Kubernetes Quick Reference Guide

## Essential kubectl Commands

### Cluster Information
```bash
kubectl version                           # Client and server version
kubectl cluster-info                      # Cluster information
kubectl get nodes                         # List all nodes
kubectl get nodes -o wide                # Detailed node information
kubectl describe node <node-name>        # Node details
```

### Working with Resources
```bash
# Get resources
kubectl get pods                          # List pods in default namespace
kubectl get pods -A                       # List pods in all namespaces
kubectl get pods -n <namespace>          # List pods in specific namespace
kubectl get all                          # List all resources
kubectl get deploy,svc,pods              # List multiple resource types
kubectl get pods -o wide                 # Show more details
kubectl get pods -o yaml                 # Output in YAML format
kubectl get pods -o json                 # Output in JSON format
kubectl get pods --show-labels           # Show labels
kubectl get pods -l app=nginx            # Filter by label

# Describe resources (detailed info + events)
kubectl describe pod <pod-name>
kubectl describe svc <service-name>
kubectl describe node <node-name>

# Create resources
kubectl create -f manifest.yaml          # Create from file
kubectl apply -f manifest.yaml           # Create or update from file
kubectl apply -f ./directory             # Apply all files in directory
kubectl create deployment nginx --image=nginx  # Quick deployment

# Edit resources
kubectl edit pod <pod-name>              # Edit with default editor
kubectl edit deploy <deployment-name>

# Delete resources
kubectl delete pod <pod-name>
kubectl delete -f manifest.yaml
kubectl delete deploy,svc <name>
kubectl delete pods --all                # Delete all pods in namespace
kubectl delete namespace <namespace>

# Scale
kubectl scale deployment nginx --replicas=3

# Rollout management
kubectl rollout status deployment/<name>
kubectl rollout history deployment/<name>
kubectl rollout undo deployment/<name>
kubectl rollout restart deployment/<name>
```

### Logs and Debugging
```bash
# Logs
kubectl logs <pod-name>                  # View logs
kubectl logs <pod-name> -c <container>  # Logs from specific container
kubectl logs <pod-name> -f              # Follow logs (stream)
kubectl logs <pod-name> --tail=50       # Last 50 lines
kubectl logs <pod-name> --since=1h      # Logs from last hour
kubectl logs <pod-name> --previous      # Logs from previous instance

# Execute commands in pod
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec -it <pod-name> -- env
kubectl exec <pod-name> -- ls /app

# Port forwarding
kubectl port-forward pod/<pod-name> 8080:80
kubectl port-forward svc/<service-name> 8080:80

# Copy files
kubectl cp <pod-name>:/path/file ./local-file
kubectl cp ./local-file <pod-name>:/path/file

# Top (requires metrics-server)
kubectl top nodes
kubectl top pods
kubectl top pods --containers

# Debug (K8s 1.23+)
kubectl debug <pod-name> -it --image=busybox
kubectl debug node/<node-name> -it --image=ubuntu
```

### Namespaces
```bash
kubectl get namespaces
kubectl create namespace dev
kubectl delete namespace dev
kubectl config set-context --current --namespace=dev  # Set default namespace
```

### Labels and Annotations
```bash
# Add/update label
kubectl label pods <pod-name> env=production
kubectl label pods <pod-name> env=dev --overwrite

# Remove label
kubectl label pods <pod-name> env-

# Add annotation
kubectl annotate pods <pod-name> description="My pod"
```

### Resource Quotas and Limits
```bash
kubectl describe resourcequota -n <namespace>
kubectl describe limitrange -n <namespace>
```

### Events
```bash
kubectl get events
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get events --field-selector involvedObject.name=<pod-name>
kubectl get events -w  # Watch events
```

### Config and Context
```bash
kubectl config view                      # View kubeconfig
kubectl config get-contexts              # List contexts
kubectl config current-context           # Show current context
kubectl config use-context <context>     # Switch context
kubectl config set-context <context> --namespace=<namespace>
```

### API Resources
```bash
kubectl api-resources                    # List all resource types
kubectl api-versions                     # List API versions
kubectl explain pod                      # Documentation for resource
kubectl explain pod.spec.containers     # Documentation for field
```

### RBAC
```bash
kubectl get roles,rolebindings
kubectl get clusterroles,clusterrolebindings
kubectl auth can-i create pods           # Check if you can perform action
kubectl auth can-i '*' '*' --all-namespaces  # Check all permissions
```

### Helpful Flags
```bash
--dry-run=client -o yaml                # Generate YAML without creating
--force --grace-period=0                # Force delete immediately
-v=8                                    # Verbose output (0-10)
--watch                                 # Watch for changes
--all-namespaces                        # All namespaces (alias: -A)
-n <namespace>                          # Specify namespace
```

---

## Common YAML Manifests

### Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.19
    ports:
    - containerPort: 80
```

### Deployment
```yaml
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
        image: nginx:1.19
        ports:
        - containerPort: 80
```

### Service (ClusterIP)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
```

### ConfigMap
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "postgres://db:5432"
  log_level: "info"
```

### Secret
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  username: YWRtaW4=      # base64 encoded
  password: cGFzc3dvcmQ=  # base64 encoded
```

### Ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend
            port:
              number: 80
```

### PersistentVolumeClaim
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

---

## Troubleshooting Flowchart

### Pod Issues
```
Pod not running?
├─ Status: Pending
│  ├─ Check: kubectl describe pod
│  ├─ Reasons:
│  │  ├─ Insufficient resources → Check node capacity
│  │  ├─ PVC not bound → Check PVC status
│  │  └─ Node selector mismatch → Check node labels
│
├─ Status: ImagePullBackOff
│  ├─ Check: Image name/tag correct?
│  ├─ Check: imagePullSecrets configured?
│  └─ Check: Registry accessible?
│
├─ Status: CrashLoopBackOff
│  ├─ Check: kubectl logs pod --previous
│  ├─ Check: Application startup issues
│  ├─ Check: Missing ConfigMap/Secret
│  └─ Check: Liveness probe too aggressive
│
└─ Status: Error
   ├─ Check: kubectl describe pod
   └─ Check: Exit code and reason
```

### Service Issues
```
Service not accessible?
├─ Check service exists
│  └─ kubectl get svc
│
├─ Check endpoints
│  ├─ kubectl get endpoints <service>
│  └─ No endpoints? → Selector doesn't match pod labels
│
├─ Check connectivity
│  ├─ kubectl run test --image=busybox -it --rm
│  └─ wget -O- http://service-name
│
└─ Check NetworkPolicy
   └─ kubectl get networkpolicies
```

---

## Quick Debugging Commands

```bash
# Create debug pod
kubectl run debug --image=nicolaka/netshoot -it --rm -- /bin/bash

# Inside debug pod:
nslookup <service-name>              # DNS lookup
curl http://<service-name>           # HTTP test
ping <pod-ip>                        # Network test
traceroute <ip>                      # Network path
tcpdump -i any port 80               # Capture traffic

# Check DNS
kubectl exec -it <pod> -- nslookup kubernetes.default

# Check API server
kubectl cluster-info dump

# Check component health
kubectl get componentstatuses

# Drain node for maintenance
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data

# Cordon node (mark unschedulable)
kubectl cordon <node>

# Uncordon node
kubectl uncordon <node>
```

---

## Resource Units

### CPU
- `1` = 1 core
- `100m` = 0.1 core (100 millicores)
- `500m` = 0.5 core

### Memory
- `128Mi` = 128 Mebibytes (1 Mi = 1024 Ki)
- `1Gi` = 1 Gibibyte (1 Gi = 1024 Mi)
- `1G` = 1 Gigabyte (1000 MB)

---

## Useful Aliases

```bash
# Add to ~/.bashrc or ~/.zshrc
alias k='kubectl'
alias kg='kubectl get'
alias kgp='kubectl get pods'
alias kgs='kubectl get svc'
alias kgn='kubectl get nodes'
alias kd='kubectl describe'
alias kdp='kubectl describe pod'
alias kl='kubectl logs'
alias klf='kubectl logs -f'
alias ke='kubectl exec -it'
alias ka='kubectl apply -f'
alias kdel='kubectl delete'
alias kx='kubectl explain'

# With these aliases:
k get pods              # Instead of: kubectl get pods
kgp -A                  # Instead of: kubectl get pods -A
kl my-pod -f            # Instead of: kubectl logs my-pod -f
```

---

## Common Error Messages

### "ImagePullBackOff"
**Cause**: Cannot pull container image  
**Fix**: Check image name, tag, and registry credentials

### "CrashLoopBackOff"
**Cause**: Container keeps crashing  
**Fix**: Check logs with `kubectl logs <pod> --previous`

### "Pending"
**Cause**: Pod cannot be scheduled  
**Fix**: Check node resources, taints, and node selectors

### "Error: ErrImagePull"
**Cause**: Initial attempt to pull image failed  
**Fix**: Check image name and registry access

### "CreateContainerConfigError"
**Cause**: ConfigMap or Secret referenced but not found  
**Fix**: Verify ConfigMap/Secret exists

### "OOMKilled"
**Cause**: Container exceeded memory limit  
**Fix**: Increase memory limits or fix memory leak

### "FailedScheduling"
**Cause**: Insufficient resources or constraints  
**Fix**: Check node capacity and pod requirements

---

## Production Readiness Checklist

### Resource Management
- [ ] Resources requests and limits defined
- [ ] ResourceQuota configured per namespace
- [ ] LimitRange configured per namespace
- [ ] PodDisruptionBudget for critical apps

### Health Checks
- [ ] Liveness probes configured
- [ ] Readiness probes configured
- [ ] Startup probes for slow-starting apps

### Security
- [ ] Run as non-root user
- [ ] ReadOnlyRootFilesystem enabled
- [ ] Drop unnecessary capabilities
- [ ] Network policies defined
- [ ] RBAC configured properly
- [ ] Secrets encrypted at rest
- [ ] Pod security standards enforced

### High Availability
- [ ] Multiple replicas (3+ for critical apps)
- [ ] PodDisruptionBudget configured
- [ ] Anti-affinity rules for spreading
- [ ] Horizontal Pod Autoscaler configured

### Monitoring & Logging
- [ ] Prometheus metrics exposed
- [ ] Logs structured (JSON)
- [ ] Distributed tracing configured
- [ ] Alerts configured
- [ ] Dashboards created

### Backup & Disaster Recovery
- [ ] Backup strategy defined
- [ ] etcd backup automated
- [ ] Application data backup configured
- [ ] Restore procedures tested

### Networking
- [ ] Ingress controller configured
- [ ] TLS certificates managed
- [ ] Network policies enforced
- [ ] Service mesh (if needed)

### Deployment
- [ ] Rolling update strategy defined
- [ ] maxSurge and maxUnavailable configured
- [ ] CI/CD pipeline automated
- [ ] GitOps implemented

---

## Performance Tips

1. **Use resource requests and limits** to prevent resource contention
2. **Enable HPA** for automatic scaling based on metrics
3. **Use ClusterIP services** for internal communication (faster than NodePort/LoadBalancer)
4. **Implement pod anti-affinity** to spread pods across nodes
5. **Use PersistentVolumes** with appropriate StorageClass (SSD for databases)
6. **Configure proper probe timings** (don't make them too aggressive)
7. **Use image pull policy IfNotPresent** to reduce image pull time
8. **Implement caching** (Redis, CDN) to reduce backend load
9. **Use connection pooling** for database connections
10. **Monitor and optimize** based on metrics (Prometheus/Grafana)

---

## Security Best Practices

1. **Run as non-root**: Set `runAsNonRoot: true`
2. **Read-only root filesystem**: Set `readOnlyRootFilesystem: true`
3. **Drop capabilities**: Drop all and add only needed ones
4. **Use NetworkPolicies**: Implement default deny-all policy
5. **Enable Pod Security Standards**: Use "restricted" profile
6. **Scan images**: Use Trivy/Snyk to scan for vulnerabilities
7. **Rotate credentials**: Use external secret management (Vault, AWS Secrets Manager)
8. **Implement RBAC**: Principle of least privilege
9. **Enable audit logging**: Track all API server requests
10. **Keep Kubernetes updated**: Apply security patches regularly

---

## Next Steps

1. Practice commands daily (30 min)
2. Deploy sample applications
3. Break things and fix them
4. Join Kubernetes Slack: https://slack.k8s.io/
5. Read official docs: https://kubernetes.io/docs/
6. Consider certification: CKAD → CKA → CKS

---

**Reference**: Based on official [Kubernetes documentation](https://kubernetes.io/)


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

