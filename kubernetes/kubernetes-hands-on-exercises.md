# Kubernetes Hands-On Exercises

## Beginner Level Exercises

### Exercise 1: First Pod
**Objective**: Create and manage a simple pod

1. Create a pod named `nginx-pod` with the nginx image
2. Verify the pod is running
3. Access the pod's shell
4. Delete the pod

<details>
<summary>Solution</summary>

```bash
# Create pod
kubectl run nginx-pod --image=nginx

# Verify
kubectl get pods
kubectl describe pod nginx-pod

# Access shell
kubectl exec -it nginx-pod -- /bin/bash
# Inside: curl localhost

# Delete
kubectl delete pod nginx-pod
```
</details>

---

### Exercise 2: Deployment and Scaling
**Objective**: Create a deployment and scale it

1. Create a deployment named `webapp` with nginx image and 2 replicas
2. Verify the deployment and pods are running
3. Scale the deployment to 5 replicas
4. Update the image to `nginx:1.19`
5. Check rollout status
6. Rollback the deployment

<details>
<summary>Solution</summary>

```bash
# Create deployment
kubectl create deployment webapp --image=nginx --replicas=2

# Verify
kubectl get deployment webapp
kubectl get pods -l app=webapp

# Scale
kubectl scale deployment webapp --replicas=5
kubectl get pods -w

# Update image
kubectl set image deployment/webapp nginx=nginx:1.19

# Check rollout
kubectl rollout status deployment/webapp
kubectl rollout history deployment/webapp

# Rollback
kubectl rollout undo deployment/webapp
```
</details>

---

### Exercise 3: Service Exposure
**Objective**: Expose a deployment via a service

1. Create a deployment named `myapp` with nginx image, 3 replicas
2. Expose it via a ClusterIP service on port 80
3. Create a NodePort service
4. Test connectivity from another pod

<details>
<summary>Solution</summary>

```bash
# Create deployment
kubectl create deployment myapp --image=nginx --replicas=3

# Expose as ClusterIP
kubectl expose deployment myapp --port=80 --type=ClusterIP

# Check service
kubectl get svc myapp
kubectl describe svc myapp
kubectl get endpoints myapp

# Create NodePort service
kubectl expose deployment myapp --port=80 --type=NodePort --name=myapp-nodeport

# Test from another pod
kubectl run test --image=busybox -it --rm -- wget -O- http://myapp
```
</details>

---

### Exercise 4: ConfigMap and Environment Variables
**Objective**: Use ConfigMap to inject configuration

1. Create a ConfigMap named `app-config` with key `DATABASE_URL=postgres://db:5432`
2. Create a pod that uses this ConfigMap as an environment variable
3. Verify the environment variable inside the pod

<details>
<summary>Solution</summary>

```bash
# Create ConfigMap
kubectl create configmap app-config --from-literal=DATABASE_URL=postgres://db:5432

# Verify
kubectl get configmap app-config
kubectl describe configmap app-config

# Create pod with ConfigMap
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: config-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ['sh', '-c', 'echo DATABASE_URL is $DATABASE_URL && sleep 3600']
    env:
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: DATABASE_URL
EOF

# Verify
kubectl logs config-pod
kubectl exec config-pod -- env | grep DATABASE_URL
```
</details>

---

### Exercise 5: Secrets
**Objective**: Create and use secrets

1. Create a secret named `db-secret` with username=admin and password=secretpass
2. Create a pod that mounts this secret as environment variables
3. Verify the values inside the pod

<details>
<summary>Solution</summary>

```bash
# Create secret
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=secretpass

# Verify (won't show values)
kubectl get secret db-secret
kubectl describe secret db-secret

# Create pod
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ['sh', '-c', 'echo "User: $DB_USER, Pass: $DB_PASS" && sleep 3600']
    env:
    - name: DB_USER
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: username
    - name: DB_PASS
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
EOF

# Verify
kubectl logs secret-pod
```
</details>

---

## Intermediate Level Exercises

### Exercise 6: Multi-Container Pod
**Objective**: Create a pod with multiple containers

1. Create a pod with two containers:
   - Container 1: nginx (serves files)
   - Container 2: busybox (writes to shared volume every 5 seconds)
2. Use an emptyDir volume shared between containers
3. Verify both containers are running
4. Check logs from both containers

<details>
<summary>Solution</summary>

```yaml
# multi-container-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container
spec:
  volumes:
  - name: shared-data
    emptyDir: {}
  
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html
  
  - name: writer
    image: busybox
    command: ['sh', '-c', 'while true; do date >> /data/index.html; sleep 5; done']
    volumeMounts:
    - name: shared-data
      mountPath: /data
```

```bash
kubectl apply -f multi-container-pod.yaml
kubectl get pod multi-container
kubectl logs multi-container -c nginx
kubectl logs multi-container -c writer
kubectl exec multi-container -c nginx -- cat /usr/share/nginx/html/index.html
```
</details>

---

### Exercise 7: Resource Limits
**Objective**: Set resource requests and limits

1. Create a deployment with:
   - CPU request: 100m
   - CPU limit: 200m
   - Memory request: 128Mi
   - Memory limit: 256Mi
2. Generate load and observe behavior
3. Check resource usage

<details>
<summary>Solution</summary>

```yaml
# resource-limited-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: resource-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: resource-demo
  template:
    metadata:
      labels:
        app: resource-demo
    spec:
      containers:
      - name: app
        image: nginx
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "200m"
            memory: "256Mi"
```

```bash
kubectl apply -f resource-limited-deployment.yaml
kubectl get pods
kubectl top pods -l app=resource-demo
kubectl describe pod <pod-name> | grep -A 10 "Limits"
```
</details>

---

### Exercise 8: Persistent Volume
**Objective**: Create and use persistent storage

1. Create a PersistentVolume with 1Gi storage
2. Create a PersistentVolumeClaim requesting 500Mi
3. Create a pod that uses this PVC
4. Write data to the volume
5. Delete the pod and create a new one
6. Verify data persists

<details>
<summary>Solution</summary>

```yaml
# pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-volume
spec:
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteOnce
  hostPath:
    path: /tmp/data

---
# pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-claim
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi

---
# pod-with-pvc.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pvc-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ['sh', '-c', 'echo "Hello Persistent World" > /data/hello.txt && sleep 3600']
    volumeMounts:
    - name: storage
      mountPath: /data
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: pv-claim
```

```bash
kubectl apply -f pv.yaml
kubectl apply -f pvc.yaml
kubectl apply -f pod-with-pvc.yaml

# Verify
kubectl get pv
kubectl get pvc
kubectl exec pvc-pod -- cat /data/hello.txt

# Delete and recreate pod
kubectl delete pod pvc-pod
kubectl apply -f pod-with-pvc.yaml
kubectl exec pvc-pod -- cat /data/hello.txt  # Data should still be there
```
</details>

---

### Exercise 9: Network Policy
**Objective**: Implement network isolation

1. Create two deployments: `frontend` and `backend`
2. Create a NetworkPolicy that:
   - Denies all ingress traffic to backend
   - Allows traffic only from frontend
3. Test connectivity

<details>
<summary>Solution</summary>

```yaml
# deployments.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 1
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
        image: nginx

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: nginx
        image: nginx

---
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  selector:
    app: backend
  ports:
  - port: 80

---
# network-policy.yaml
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
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 80
```

```bash
kubectl apply -f deployments.yaml
kubectl apply -f network-policy.yaml

# Test from frontend (should work)
kubectl exec -it $(kubectl get pod -l app=frontend -o name) -- curl backend

# Test from random pod (should fail)
kubectl run test --image=busybox -it --rm -- wget -O- --timeout=2 http://backend
```
</details>

---

### Exercise 10: Horizontal Pod Autoscaler
**Objective**: Implement autoscaling based on CPU

1. Create a deployment with resource requests
2. Expose it via a service
3. Create an HPA that scales from 2 to 10 replicas at 50% CPU
4. Generate load and observe scaling

<details>
<summary>Solution</summary>

```yaml
# php-apache.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  replicas: 1
  selector:
    matchLabels:
      app: php-apache
  template:
    metadata:
      labels:
        app: php-apache
    spec:
      containers:
      - name: php-apache
        image: k8s.gcr.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 200m

---
apiVersion: v1
kind: Service
metadata:
  name: php-apache
spec:
  selector:
    app: php-apache
  ports:
  - port: 80
```

```bash
# Apply deployment
kubectl apply -f php-apache.yaml

# Create HPA
kubectl autoscale deployment php-apache --cpu-percent=50 --min=2 --max=10

# Verify
kubectl get hpa

# Generate load (in separate terminal)
kubectl run -it --rm load-generator --image=busybox -- /bin/sh
# Inside pod:
while true; do wget -q -O- http://php-apache; done

# Watch scaling
kubectl get hpa -w
kubectl get pods -w
```
</details>

---

## Advanced Level Exercises

### Exercise 11: StatefulSet with Headless Service
**Objective**: Deploy a stateful application

1. Create a headless service
2. Create a StatefulSet with 3 replicas
3. Verify stable network identities
4. Test pod-to-pod communication using DNS

<details>
<summary>Solution</summary>

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
  serviceName: nginx-headless
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
        image: nginx:1.19
        ports:
        - containerPort: 80
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

```bash
kubectl apply -f statefulset.yaml

# Verify stable identities
kubectl get pods -l app=nginx-stateful

# Should see: web-0, web-1, web-2

# Test DNS
kubectl run -it --rm debug --image=busybox --restart=Never -- sh
# Inside:
nslookup web-0.nginx-headless
nslookup web-1.nginx-headless
nslookup web-2.nginx-headless
```
</details>

---

### Exercise 12: Rolling Update with Zero Downtime
**Objective**: Perform zero-downtime deployment

1. Create a deployment with 5 replicas
2. Configure proper readiness probes
3. Set rolling update strategy (maxSurge: 1, maxUnavailable: 0)
4. Update the image while monitoring availability

<details>
<summary>Solution</summary>

```yaml
# zero-downtime.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: zero-downtime-app
spec:
  replicas: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: nginx
        image: nginx:1.18
        ports:
        - containerPort: 80
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
        lifecycle:
          preStop:
            exec:
              command: ["/bin/sh", "-c", "sleep 15"]
      terminationGracePeriodSeconds: 30

---
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
  - port: 80
```

```bash
kubectl apply -f zero-downtime.yaml

# Monitor in one terminal
kubectl get pods -w

# Generate continuous traffic in another terminal
while true; do curl http://myapp-service 2>&1 | head -1; sleep 0.5; done

# Update image in third terminal
kubectl set image deployment/zero-downtime-app nginx=nginx:1.19

# Watch rollout
kubectl rollout status deployment/zero-downtime-app
```
</details>

---

### Exercise 13: Blue-Green Deployment
**Objective**: Implement blue-green deployment pattern

1. Deploy blue version (v1)
2. Deploy green version (v2)
3. Switch traffic from blue to green
4. Rollback if needed

<details>
<summary>Solution</summary>

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
      - name: nginx
        image: nginx:1.18
        command: ["/bin/sh", "-c", "echo 'Blue Version' > /usr/share/nginx/html/index.html && nginx -g 'daemon off;'"]

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
      - name: nginx
        image: nginx:1.19
        command: ["/bin/sh", "-c", "echo 'Green Version' > /usr/share/nginx/html/index.html && nginx -g 'daemon off;'"]

---
# service-blue.yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
    version: blue  # Switch to 'green' to change version
  ports:
  - port: 80
```

```bash
# Deploy blue
kubectl apply -f blue-deployment.yaml
kubectl apply -f service-blue.yaml

# Test
kubectl run test --image=busybox -it --rm -- wget -O- http://myapp-service
# Output: Blue Version

# Deploy green
kubectl apply -f green-deployment.yaml

# Switch service to green
kubectl patch service myapp-service -p '{"spec":{"selector":{"version":"green"}}}'

# Test again
kubectl run test --image=busybox -it --rm -- wget -O- http://myapp-service
# Output: Green Version

# Rollback to blue if needed
kubectl patch service myapp-service -p '{"spec":{"selector":{"version":"blue"}}}'
```
</details>

---

### Exercise 14: Job and CronJob
**Objective**: Run batch workloads

1. Create a Job that runs 5 completions with parallelism of 2
2. Create a CronJob that runs every minute
3. Monitor job execution

<details>
<summary>Solution</summary>

```yaml
# job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: batch-job
spec:
  completions: 5
  parallelism: 2
  template:
    spec:
      containers:
      - name: worker
        image: busybox
        command: ['sh', '-c', 'echo "Processing batch job $(date)" && sleep 10']
      restartPolicy: Never
  backoffLimit: 4

---
# cronjob.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: scheduled-job
spec:
  schedule: "*/1 * * * *"  # Every minute
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: worker
            image: busybox
            command: ['sh', '-c', 'echo "Cron job executed at $(date)"']
          restartPolicy: OnFailure
```

```bash
kubectl apply -f job.yaml
kubectl get jobs -w
kubectl get pods -l job-name=batch-job
kubectl logs <pod-name>

kubectl apply -f cronjob.yaml
kubectl get cronjobs
kubectl get jobs -w  # Wait for a minute
```
</details>

---

### Exercise 15: RBAC Configuration
**Objective**: Implement role-based access control

1. Create a namespace called `dev`
2. Create a ServiceAccount called `developer`
3. Create a Role that allows getting, listing, and watching pods
4. Bind the role to the service account
5. Test permissions

<details>
<summary>Solution</summary>

```yaml
# rbac.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: developer
  namespace: dev

---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: dev
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: dev
subjects:
- kind: ServiceAccount
  name: developer
  namespace: dev
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

```bash
kubectl apply -f rbac.yaml

# Test permissions
kubectl auth can-i get pods --as=system:serviceaccount:dev:developer -n dev
# Output: yes

kubectl auth can-i create pods --as=system:serviceaccount:dev:developer -n dev
# Output: no

kubectl auth can-i delete pods --as=system:serviceaccount:dev:developer -n dev
# Output: no
```
</details>

---

## Production Debugging Exercises

### Exercise 16: Debug CrashLoopBackOff
**Scenario**: A pod is in CrashLoopBackOff state

<details>
<summary>Debugging Steps</summary>

```yaml
# Create problematic pod
apiVersion: v1
kind: Pod
metadata:
  name: crash-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ['sh', '-c', 'exit 1']
```

```bash
kubectl apply -f crash-pod.yaml

# Debugging steps
kubectl get pods  # See CrashLoopBackOff status
kubectl describe pod crash-pod  # Check events
kubectl logs crash-pod  # Check current logs
kubectl logs crash-pod --previous  # Check previous instance logs

# Fix by updating command
kubectl edit pod crash-pod
# Change command to: ['sh', '-c', 'sleep 3600']
```
</details>

---

### Exercise 17: Debug Service Connectivity
**Scenario**: Service is not accessible

<details>
<summary>Debugging Steps</summary>

```bash
# Check if service exists
kubectl get svc myapp-service
kubectl describe svc myapp-service

# Check endpoints
kubectl get endpoints myapp-service
# If no endpoints, selector doesn't match pods

# Check pod labels
kubectl get pods --show-labels

# Verify selector matches
kubectl get svc myapp-service -o jsonpath='{.spec.selector}'

# Test DNS resolution
kubectl run test --image=busybox -it --rm -- nslookup myapp-service

# Test connectivity
kubectl run test --image=busybox -it --rm -- wget -O- http://myapp-service

# Check network policies
kubectl get networkpolicies
kubectl describe networkpolicy <policy-name>
```
</details>

---

### Exercise 18: Debug High Memory Usage
**Scenario**: Pod is consuming too much memory

<details>
<summary>Debugging Steps</summary>

```bash
# Check resource usage
kubectl top pods
kubectl top pods --containers

# Check resource limits
kubectl get pod <pod-name> -o jsonpath='{.spec.containers[*].resources}'

# Describe pod
kubectl describe pod <pod-name> | grep -A 10 "Limits"

# Check if pod was OOMKilled
kubectl describe pod <pod-name> | grep -i "OOMKilled"

# Get detailed metrics
kubectl get --raw /apis/metrics.k8s.io/v1beta1/namespaces/default/pods/<pod-name>

# Check application logs for memory issues
kubectl logs <pod-name> | grep -i memory
```
</details>

---

## Challenge Exercises

### Challenge 1: Complete Microservices Deployment
Deploy a complete microservices application with:
- Frontend (React)
- Backend API (Node.js)
- Database (PostgreSQL)
- Redis cache
- Proper networking
- Persistent storage
- ConfigMaps and Secrets
- Ingress
- Monitoring

### Challenge 2: Disaster Recovery
1. Set up a production cluster
2. Deploy applications with persistent data
3. Implement backup strategy
4. Simulate node failure
5. Perform full cluster restore

### Challenge 3: Security Hardening
1. Deploy an application
2. Implement all security best practices
3. Run security scans
4. Fix all vulnerabilities
5. Implement network policies
6. Configure RBAC

---

## Tips for Practice

1. **Start Simple**: Master basic commands before advanced concepts
2. **Break Things**: Learn by breaking and fixing
3. **Use Labels**: Practice querying with label selectors
4. **Read Docs**: Use `kubectl explain` to understand resources
5. **Check Events**: Always check events when debugging
6. **Time Yourself**: Practice for certification exams
7. **Clean Up**: Delete resources after practice to save resources
8. **Take Notes**: Document solutions and learnings
9. **Join Community**: Ask questions in Kubernetes Slack
10. **Build Projects**: Apply knowledge to real projects

---

## Next Steps

1. Complete all beginner exercises
2. Move to intermediate exercises
3. Practice debugging scenarios
4. Build a real project
5. Consider certification (CKAD/CKA)
6. Contribute to open source

**Happy Learning! 🚀**

Reference: [kubernetes.io](https://kubernetes.io/)


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

