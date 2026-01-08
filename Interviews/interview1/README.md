# DevOps Interview Session 1
## For Azure DevOps Engineer / Cloud Engineer / DevOps Engineer (3-7 Years Experience)

---

### Question 1: Kubernetes - Explain the difference between Deployment and StatefulSet
**Answer:**
- **Deployment:** For stateless applications where pods are interchangeable
  - Random pod names (app-7f8c9d5b6a-abc12)
  - Any pod can be replaced by any other
  - No persistent identity
  - Scale up/down in any order
  - Use for: Web servers, APIs, microservices

- **StatefulSet:** For stateful applications requiring stable identity
  - Ordered pod names (database-0, database-1, database-2)
  - Each pod has unique identity
  - Stable network identity and storage
  - Ordered deployment (0→1→2)
  - Ordered scaling and deletion
  - Use for: Databases (MySQL, MongoDB), Kafka, Cassandra

---

### Question 2: Azure DevOps - How do you create and use a Service Principal for pipeline authentication?
**Answer:**
```bash
# Create Service Principal
az ad sp create-for-rbac \
  --name azure-devops-sp \
  --role Contributor \
  --scopes /subscriptions/{subscription-id}

# Output provides: appId, password, tenant

# In Azure DevOps:
1. Project Settings → Service connections
2. New service connection → Azure Resource Manager
3. Authentication: Service principal (manual)
4. Enter: Subscription ID, Service Principal ID, Key, Tenant ID
5. Verify and save

# Use in pipeline:
- task: AzureWebApp@1
  inputs:
    azureSubscription: 'azure-connection'
    appName: 'myapp'
```

**Best Practices:**
- Separate SPs for dev/staging/prod
- Grant minimum required permissions
- Rotate credentials every 90 days
- Never hardcode credentials

---

### Question 3: Docker - What happens when you run `docker run -p 8080:80 nginx`?
**Answer:**
1. **Image Pull:** Docker checks local images, pulls `nginx:latest` if not present
2. **Container Creation:** Creates container from nginx image
3. **Port Mapping:** Maps host port 8080 to container port 80
4. **Network:** Attaches container to default bridge network
5. **Process Start:** Starts nginx process (PID 1) inside container
6. **Accessibility:** nginx accessible at `http://localhost:8080`

**Command Breakdown:**
```bash
docker run      # Create and start container
-p 8080:80     # Host port 8080 → Container port 80
nginx          # Image name

# Alternative with name:
docker run -d --name mynginx -p 8080:80 nginx
```

---

### Question 4: Terraform - Explain the difference between `terraform plan` and `terraform apply`
**Answer:**
- **terraform plan:**
  - Dry-run, shows what will change
  - Doesn't modify infrastructure
  - Creates execution plan
  - Shows: Resources to add (+), change (~), destroy (-)
  - Safe to run anytime

- **terraform apply:**
  - Actually modifies infrastructure
  - Creates/updates/destroys resources
  - Prompts for confirmation (unless `-auto-approve`)
  - Runs plan first, then applies
  - Updates `terraform.tfstate`

```bash
# Safe preview
terraform plan -out=plan.out

# Review and apply
terraform apply plan.out

# Or apply directly (prompts for confirmation)
terraform apply

# Auto-approve (CI/CD use)
terraform apply -auto-approve
```

---

### Question 5: Linux - How do you find the top 10 processes consuming most memory?
**Answer:**
```bash
# Method 1: Using ps
ps aux --sort=-%mem | head -11

# Method 2: Using top (interactive)
top
# Press Shift+M to sort by memory

# Method 3: Specific memory usage
ps aux --sort=-%mem | awk 'NR<=10{print $0}'

# Method 4: With memory in human-readable format
ps aux --sort=-%mem | awk '{printf "%-10s %-8s %-8s %s\n", $1, $2, $4, $11}' | head -11

# Method 5: Using htop (if installed)
htop
# Press F6, select PERCENT_MEM, Enter
```

---

### Question 6: Jenkins - Explain Declarative vs Scripted Pipeline
**Answer:**
**Declarative Pipeline:**
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
    }
}
```
- **Pros:** Simpler syntax, easier to read, built-in directives
- **Cons:** Less flexible
- **Use:** Most common use cases

**Scripted Pipeline:**
```groovy
node {
    stage('Build') {
        sh 'npm install'
    }
    stage('Test') {
        sh 'npm test'
    }
}
```
- **Pros:** More flexible, full Groovy power
- **Cons:** Requires Groovy knowledge
- **Use:** Complex conditional logic needed

---

### Question 7: Kubernetes - How do you troubleshoot a pod stuck in CrashLoopBackOff?
**Answer:**
**Diagnosis Steps:**
```bash
# 1. Check pod status
kubectl describe pod <pod-name>

# 2. Check logs
kubectl logs <pod-name>

# 3. Check previous container logs (if crashed)
kubectl logs <pod-name> --previous

# 4. Check events
kubectl get events --sort-by='.lastTimestamp' | grep <pod-name>

# 5. Check resource limits
kubectl get pod <pod-name> -o yaml | grep -A 5 resources
```

**Common Causes:**
1. **Application Error:** Check logs for errors
2. **Missing Dependencies:** Database not ready, missing config
3. **OOMKilled:** Increase memory limit
4. **Wrong Command:** Incorrect entrypoint/command
5. **Liveness Probe Failure:** Adjust initialDelaySeconds

**Solution Example:**
```yaml
# Increase resources and adjust probes
resources:
  limits:
    memory: "512Mi"  # Increase from 256Mi
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 60  # Give app time to start
  periodSeconds: 10
```

---

### Question 8: Git - How do you revert a commit that has already been pushed to remote?
**Answer:**
```bash
# Method 1: git revert (Safe - creates new commit)
git revert <commit-hash>
git push origin main

# Method 2: git reset (Dangerous - rewrites history)
git reset --hard <commit-hash>
git push --force origin main  # Use with caution!

# Method 3: Interactive rebase
git rebase -i HEAD~3  # Last 3 commits
# Mark commit as 'drop' or 'edit'
git push --force origin main

# Safest for shared branches:
# Use git revert to maintain history
git log --oneline  # Find commit hash
git revert abc1234
git push origin main
```

**Best Practice:** Use `git revert` for shared branches (main/master/develop) to maintain history.

---

### Question 9: Azure - How do you create an AKS cluster using Azure CLI?
**Answer:**
```bash
# 1. Set variables
RESOURCE_GROUP="myapp-rg"
CLUSTER_NAME="myapp-aks"
LOCATION="eastus"

# 2. Create resource group
az group create \
  --name $RESOURCE_GROUP \
  --location $LOCATION

# 3. Create AKS cluster
az aks create \
  --resource-group $RESOURCE_GROUP \
  --name $CLUSTER_NAME \
  --node-count 3 \
  --node-vm-size Standard_D2s_v3 \
  --enable-managed-identity \
  --network-plugin azure \
  --network-policy azure \
  --load-balancer-sku standard \
  --enable-addons monitoring \
  --generate-ssh-keys

# 4. Get credentials
az aks get-credentials \
  --resource-group $RESOURCE_GROUP \
  --name $CLUSTER_NAME

# 5. Verify
kubectl get nodes
kubectl cluster-info
```

---

### Question 10: Python - Write a script to check if a service is running in Kubernetes
**Answer:**
```python
#!/usr/bin/env python3
from kubernetes import client, config
import sys

def check_service_status(namespace, service_name):
    """Check if a Kubernetes service is running and has endpoints"""
    try:
        # Load kubeconfig
        config.load_kube_config()
        
        # Create API clients
        v1 = client.CoreV1Api()
        
        # Get service
        try:
            service = v1.read_namespaced_service(service_name, namespace)
            print(f"✓ Service '{service_name}' exists in namespace '{namespace}'")
            print(f"  Type: {service.spec.type}")
            print(f"  ClusterIP: {service.spec.cluster_ip}")
            
            # Check if service has ports
            if service.spec.ports:
                print(f"  Ports: {[f'{p.port}:{p.target_port}' for p in service.spec.ports]}")
            
            # Check endpoints
            try:
                endpoints = v1.read_namespaced_endpoints(service_name, namespace)
                if endpoints.subsets and any(s.addresses for s in endpoints.subsets):
                    addresses = [addr.ip for subset in endpoints.subsets 
                               for addr in (subset.addresses or [])]
                    print(f"  Endpoints: {len(addresses)} pod(s) ready")
                    print(f"  Pod IPs: {addresses}")
                    return True
                else:
                    print(f"  ⚠ No endpoints available - no pods backing this service")
                    return False
            except client.exceptions.ApiException:
                print(f"  ⚠ No endpoints found")
                return False
                
        except client.exceptions.ApiException as e:
            if e.status == 404:
                print(f"✗ Service '{service_name}' not found in namespace '{namespace}'")
            else:
                print(f"✗ Error: {e}")
            return False
            
    except Exception as e:
        print(f"✗ Error: {e}")
        return False

if __name__ == "__main__":
    if len(sys.argv) != 3:
        print("Usage: python check_service.py <namespace> <service-name>")
        sys.exit(1)
    
    namespace = sys.argv[1]
    service_name = sys.argv[2]
    
    if check_service_status(namespace, service_name):
        sys.exit(0)
    else:
        sys.exit(1)

# Usage:
# python check_service.py default myapp-service
```

---

### Question 11: Azure DevOps - How do you implement approval gates in a release pipeline?
**Answer:**
**Using Environments:**
```yaml
stages:
- stage: DeployProduction
  jobs:
  - deployment: DeployProd
    environment: production  # Environment with approvals
    strategy:
      runOnce:
        deploy:
          steps:
          - script: echo Deploying to production
```

**Configure Approvals:**
```
1. Pipelines → Environments → production
2. Click "..." → Approvals and checks
3. Add → Approvals
4. Approvers: Select users/groups
5. Minimum number of approvers: 2
6. Timeout: 30 days
7. Instructions: "Review changes before prod deployment"
8. Advanced:
   - Requestor cannot approve
   - Revalidate after timeout
9. Create
```

**Multiple Gates:**
- Manual approval
- Azure Policy compliance
- Query Azure Monitor alerts
- Invoke Azure Function
- ServiceNow change request

---

### Question 12: Docker - Explain multi-stage builds and provide an example
**Answer:**
Multi-stage builds reduce image size by separating build and runtime environments.

```dockerfile
# Stage 1: Build
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:18-alpine
WORKDIR /app
# Copy only necessary files from builder stage
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package.json ./
EXPOSE 3000
USER node
CMD ["node", "dist/index.js"]

# Build:
# docker build -t myapp:1.0 .

# Result:
# - Builder stage: 1.2GB (includes dev dependencies, build tools)
# - Final image: 150MB (only runtime dependencies)
# - Secure (node user, alpine base)
```

**Benefits:**
- Smaller final image
- Faster deployment
- Separate build/runtime dependencies
- More secure (fewer packages)

---

### Question 13: Kubernetes - What is an Ingress and how is it different from a Service?
**Answer:**
**Service (Layer 4):**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: LoadBalancer  # Creates cloud LB (expensive!)
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
```
- Each service needs separate LoadBalancer ($$$)
- No HTTP routing (path/host-based)
- Port-based access only

**Ingress (Layer 7):**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
spec:
  rules:
  - host: myapp.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
      - path: /web
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```
- Single LoadBalancer for multiple services ($)
- HTTP/HTTPS routing (path and host-based)
- TLS termination
- Clean URLs

**Key Difference:** Ingress = smart HTTP router, Service = basic network endpoint

---

### Question 14: Ansible - Write a playbook to install and configure Nginx
**Answer:**
```yaml
---
- name: Install and Configure Nginx
  hosts: webservers
  become: yes
  vars:
    nginx_port: 80
    server_name: myapp.example.com
    
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600
      when: ansible_os_family == "Debian"
    
    - name: Install Nginx
      package:
        name: nginx
        state: present
    
    - name: Create web directory
      file:
        path: /var/www/myapp
        state: directory
        owner: www-data
        group: www-data
        mode: '0755'
    
    - name: Deploy index.html
      copy:
        content: |
          <!DOCTYPE html>
          <html>
          <body>
            <h1>Welcome to {{ server_name }}</h1>
          </body>
          </html>
        dest: /var/www/myapp/index.html
        owner: www-data
        group: www-data
        mode: '0644'
    
    - name: Configure Nginx site
      template:
        src: nginx-site.conf.j2
        dest: /etc/nginx/sites-available/myapp
      notify: Reload Nginx
    
    - name: Enable site
      file:
        src: /etc/nginx/sites-available/myapp
        dest: /etc/nginx/sites-enabled/myapp
        state: link
      notify: Reload Nginx
    
    - name: Remove default site
      file:
        path: /etc/nginx/sites-enabled/default
        state: absent
      notify: Reload Nginx
    
    - name: Ensure Nginx is started and enabled
      service:
        name: nginx
        state: started
        enabled: yes
    
    - name: Open firewall for HTTP
      ufw:
        rule: allow
        port: '80'
        proto: tcp
  
  handlers:
    - name: Reload Nginx
      service:
        name: nginx
        state: reloaded

# templates/nginx-site.conf.j2
server {
    listen {{ nginx_port }};
    server_name {{ server_name }};
    
    root /var/www/myapp;
    index index.html;
    
    location / {
        try_files $uri $uri/ =404;
    }
}

# Run:
# ansible-playbook -i inventory nginx-playbook.yml
```

---

### Question 15: Azure - Explain the difference between Azure Container Registry (ACR) and Docker Hub
**Answer:**
**Azure Container Registry (ACR):**
- **Private registry** in Azure
- Integrated with Azure services (AKS, App Service, etc.)
- Authentication via Azure AD, Service Principal, or managed identity
- Geo-replication available
- No rate limits
- Vulnerability scanning (Defender for Containers)
- Azure Key Vault integration for secrets
- **Pricing:** Pay for storage and bandwidth

```bash
# Create ACR
az acr create --resource-group myRG --name myregistry --sku Standard

# Login
az acr login --name myregistry

# Push image
docker tag myapp:1.0 myregistry.azurecr.io/myapp:1.0
docker push myregistry.azurecr.io/myapp:1.0

# Use in AKS (with managed identity)
az aks update -n myAKS -g myRG --attach-acr myregistry
```

**Docker Hub:**
- **Public registry** (or private with paid plan)
- Global, community-driven
- Authentication with Docker ID
- Rate limits (100 pulls/6hrs for free, 200 for authenticated)
- Public images for open-source projects
- **Pricing:** Free for public repos, paid for private

**When to use:**
- **ACR:** Enterprise applications, Azure deployments, private images, no rate limits
- **Docker Hub:** Open-source, public images, community sharing

---

### Question 16: Prometheus - How do you configure Prometheus to scrape metrics from a Kubernetes service?
**Answer:**
**Service with Prometheus Annotations:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "9090"
    prometheus.io/path: "/metrics"
spec:
  selector:
    app: myapp
  ports:
  - port: 8080
    name: http
  - port: 9090
    name: metrics
```

**Prometheus ServiceMonitor (Prometheus Operator):**
```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: myapp-monitor
  labels:
    release: prometheus
spec:
  selector:
    matchLabels:
      app: myapp
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics
```

**Prometheus ConfigMap (Manual):**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
    scrape_configs:
    - job_name: 'myapp'
      kubernetes_sd_configs:
      - role: service
      relabel_configs:
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_service_annotation_prometheus_io_port]
        action: replace
        target_label: __address__
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
```

---

### Question 17: Jenkins - How do you implement parallel stages in a pipeline?
**Answer:**
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }
        
        stage('Parallel Tests') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'npm run test:unit'
                        junit 'reports/unit/*.xml'
                    }
                }
                stage('Integration Tests') {
                    agent {
                        label 'integration-test'
                    }
                    steps {
                        sh 'npm run test:integration'
                        junit 'reports/integration/*.xml'
                    }
                }
                stage('Security Scan') {
                    steps {
                        sh 'npm audit'
                        sh 'snyk test'
                    }
                }
                stage('Linting') {
                    steps {
                        sh 'npm run lint'
                    }
                }
            }
        }
        
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sh 'kubectl apply -f k8s/'
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
}
```

**Benefits:**
- Faster pipeline execution
- Independent test stages
- Better resource utilization
- Clear failure points

---

### Question 18: Linux - How do you set up a cron job to run a script every day at 2 AM?
**Answer:**
```bash
# Edit crontab
crontab -e

# Add entry (runs at 2:00 AM daily)
0 2 * * * /path/to/script.sh

# Cron format:
# Minute Hour Day Month DayOfWeek Command
# 0      2    *   *     *         /path/to/script.sh

# Examples:
# Every hour
0 * * * * /path/to/script.sh

# Every day at midnight
0 0 * * * /path/to/script.sh

# Every Monday at 9 AM
0 9 * * 1 /path/to/script.sh

# Every 15 minutes
*/15 * * * * /path/to/script.sh

# First day of month
0 0 1 * * /path/to/script.sh

# Weekdays at 6 PM
0 18 * * 1-5 /path/to/script.sh

# With logging
0 2 * * * /path/to/script.sh >> /var/log/myscript.log 2>&1

# With environment variables
0 2 * * * export PATH=$PATH:/usr/local/bin && /path/to/script.sh

# View crontab
crontab -l

# Remove crontab
crontab -r

# System-wide cron jobs
# Edit: /etc/crontab
# Or: /etc/cron.d/
```

---

### Question 19: Kubernetes - Explain Horizontal Pod Autoscaler (HPA) with an example
**Answer:**
**HPA automatically scales pods based on CPU, memory, or custom metrics.**

```yaml
# Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 2  # Initial replicas
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: webapp:1.0
        resources:
          requests:
            cpu: 100m      # Required for HPA!
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 256Mi
---
# HPA
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: webapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70  # Target 70% CPU

# How it works:
# Current: 2 pods at 85% CPU (above 70% target)
# HPA calculation: ceil[2 * (85/70)] = 3 pods
# Result: Scales up to 3 pods
```

**Create HPA via CLI:**
```bash
kubectl autoscale deployment webapp --cpu-percent=70 --min=2 --max=10

# View HPA
kubectl get hpa

# Watch scaling
kubectl get hpa webapp --watch
```

**Testing HPA:**
```bash
# Generate load
kubectl run -it load-generator --image=busybox --rm -- /bin/sh -c "while true; do wget -q -O- http://webapp; done"

# Watch pods scale
watch kubectl get pods
watch kubectl get hpa
```

---

### Question 20: Azure DevOps - How do you securely store secrets in Azure Pipelines?
**Answer:**
**Method 1: Variable Groups with Azure Key Vault:**
```
1. Create Key Vault:
   az keyvault create --name mykeyvault --resource-group myRG --location eastus

2. Add secrets:
   az keyvault secret set --vault-name mykeyvault --name "db-password" --value "SuperSecret123"

3. Azure DevOps → Pipelines → Library → Variable groups
4. "+ Variable group"
5. Name: keyvault-secrets
6. Link to Azure Key Vault: Toggle ON
7. Azure subscription: Select
8. Key vault name: mykeyvault
9. Authorize
10. Add secrets: Select "db-password"
11. Save

Pipeline usage:
variables:
- group: keyvault-secrets

steps:
- script: |
    echo "Connecting to database..."
    # $(db-password) fetched from Key Vault
    mysql -h $(db-host) -u $(db-user) -p$(db-password)
```

**Method 2: Pipeline Variables (Secret):**
```
1. Edit pipeline → Variables
2. "+ New variable"
3. Name: API_KEY
4. Value: secret-key-value
5. Keep this value secret: ✓ (Lock icon)
6. Save

Pipeline:
variables:
  API_KEY: $(API_KEY)  # Masked in logs

steps:
- script: |
    curl -H "Authorization: Bearer $(API_KEY)" https://api.example.com
```

**Method 3: Secure Files:**
```
1. Pipelines → Library → Secure files
2. "+ Secure file"
3. Upload: certificate.pfx, kubeconfig, etc.
4. Authorize for specific pipelines

Pipeline:
- task: DownloadSecureFile@1
  name: kubeconfig
  inputs:
    secureFile: 'prod-kubeconfig'

- script: |
    kubectl --kubeconfig=$(kubeconfig.secureFilePath) get pods
```

**Best Practices:**
- Use Azure Key Vault for production secrets
- Never commit secrets to code
- Rotate secrets regularly
- Use managed identities when possible
- Limit secret access to specific pipelines

---

### Question 21: Helm - What is Helm and how do you create a basic Helm chart?
**Answer:**
**Helm = Package manager for Kubernetes**

**Create Helm Chart:**
```bash
# Create chart structure
helm create myapp

# Structure:
myapp/
  Chart.yaml          # Chart metadata
  values.yaml         # Default values
  charts/             # Dependencies
  templates/          # Kubernetes manifests
    deployment.yaml
    service.yaml
    ingress.yaml
    _helpers.tpl      # Template helpers
```

**Chart.yaml:**
```yaml
apiVersion: v2
name: myapp
description: My Application Helm Chart
version: 1.0.0
appVersion: "1.0"
```

**values.yaml:**
```yaml
replicaCount: 3

image:
  repository: myregistry.azurecr.io/myapp
  tag: "1.0"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: true
  hosts:
    - host: myapp.example.com
      paths:
        - path: /
          pathType: Prefix

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi
```

**templates/deployment.yaml:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "myapp.fullname" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ include "myapp.name" . }}
  template:
    metadata:
      labels:
        app: {{ include "myapp.name" . }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        ports:
        - containerPort: 8080
        resources:
          {{- toYaml .Values.resources | nindent 12 }}
```

**Install Chart:**
```bash
# Dry run (test)
helm install myapp ./myapp --dry-run --debug

# Install
helm install myapp ./myapp

# Install with custom values
helm install myapp ./myapp --set replicaCount=5

# Install with values file
helm install myapp ./myapp -f prod-values.yaml

# Upgrade
helm upgrade myapp ./myapp

# Rollback
helm rollback myapp 1

# List releases
helm list

# Uninstall
helm uninstall myapp
```

---

### Question 22: ArgoCD - How do you deploy an application using ArgoCD?
**Answer:**
**1. Install ArgoCD:**
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Get admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Port forward
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Login: https://localhost:8080 (admin / password)
```

**2. Create Application:**
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
    path: k8s/production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true

# Apply:
kubectl apply -f application.yaml
```

**3. Sync Application:**
```bash
# CLI login
argocd login localhost:8080

# Sync app
argocd app sync myapp

# Get app status
argocd app get myapp

# Watch sync
argocd app sync myapp --watch
```

**GitOps Workflow:**
```
1. Developer pushes code + K8s manifests to Git
2. ArgoCD detects change in Git repo
3. ArgoCD compares desired state (Git) vs actual state (cluster)
4. ArgoCD syncs cluster to match Git
5. Application deployed/updated

Git = Single source of truth
```

---

### Question 23: GitHub Actions - Write a workflow to build and push Docker image
**Answer:**
```yaml
# .github/workflows/docker-build.yml
name: Docker Build and Push

on:
  push:
    branches: [main, develop]
    tags: ['v*']
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Log in to Container Registry
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha
      
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
      
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:sha-${{ github.sha }}
          format: 'sarif'
          output: 'trivy-results.sarif'
      
      - name: Upload Trivy results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'
```

---

### Question 24: SonarQube - How do you integrate SonarQube with Jenkins pipeline?
**Answer:**
```groovy
pipeline {
    agent any
    
    environment {
        SONAR_HOST_URL = 'http://sonarqube.example.com'
        SONAR_PROJECT_KEY = 'myapp'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/myorg/myapp.git'
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('SonarQube Analysis') {
            environment {
                SONAR_TOKEN = credentials('sonarqube-token')
            }
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'
                    withSonarQubeEnv('SonarQube') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                              -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                              -Dsonar.sources=src \
                              -Dsonar.java.binaries=target/classes \
                              -Dsonar.host.url=${SONAR_HOST_URL} \
                              -Dsonar.login=${SONAR_TOKEN}
                        """
                    }
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sh 'kubectl apply -f k8s/'
            }
        }
    }
    
    post {
        always {
            junit 'target/surefire-reports/*.xml'
            jacoco()
        }
    }
}
```

**Jenkins Configuration:**
1. Manage Jenkins → Configure System → SonarQube servers
2. Add SonarQube: Name, URL, Token
3. Manage Jenkins → Global Tool Configuration → SonarQube Scanner
4. Add SonarScanner installation

---

### Question 25: Terraform - How do you manage multiple environments (dev/staging/prod)?
**Answer:**
**Method 1: Workspaces:**
```bash
# Create workspaces
terraform workspace new dev
terraform workspace new staging
terraform workspace new prod

# List workspaces
terraform workspace list

# Switch workspace
terraform workspace select prod

# Use in config
resource "azurerm_resource_group" "rg" {
  name     = "myapp-${terraform.workspace}-rg"
  location = "East US"
}

# Deploy
terraform workspace select dev
terraform apply  # Creates myapp-dev-rg

terraform workspace select prod
terraform apply  # Creates myapp-prod-rg
```

**Method 2: Separate directories:**
```
terraform/
  ├── modules/
  │   ├── network/
  │   ├── compute/
  │   └── database/
  ├── environments/
  │   ├── dev/
  │   │   ├── main.tf
  │   │   ├── variables.tf
  │   │   └── terraform.tfvars
  │   ├── staging/
  │   │   ├── main.tf
  │   │   ├── variables.tf
  │   │   └── terraform.tfvars
  │   └── prod/
  │       ├── main.tf
  │       ├── variables.tf
  │       └── terraform.tfvars
```

**environments/dev/terraform.tfvars:**
```hcl
environment = "dev"
vm_size     = "Standard_B2s"
vm_count    = 1
```

**environments/prod/terraform.tfvars:**
```hcl
environment = "prod"
vm_size     = "Standard_D4s_v3"
vm_count    = 3
```

**Deploy:**
```bash
cd environments/dev
terraform init
terraform apply

cd ../prod
terraform init
terraform apply
```

**Method 3: tfvars files:**
```bash
# dev.tfvars
environment = "dev"
vm_count    = 1

# prod.tfvars
environment = "prod"
vm_count    = 5

# Deploy
terraform apply -var-file="dev.tfvars"
terraform apply -var-file="prod.tfvars"
```

---

## Practical Scenario Questions

### Question 26: Scenario - Your Kubernetes pods are getting OOMKilled. How do you diagnose and fix?
**Answer:**
**Diagnosis:**
```bash
# 1. Confirm OOMKilled
kubectl describe pod <pod-name>
# Look for: Reason: OOMKilled, Exit Code: 137

# 2. Check current memory usage
kubectl top pod <pod-name>

# 3. Check resource limits
kubectl get pod <pod-name> -o yaml | grep -A 5 resources

# 4. Check metrics history (if Prometheus available)
kubectl port-forward -n monitoring svc/prometheus 9090:9090
# Query: container_memory_usage_bytes{pod="<pod-name>"}
```

**Solution Options:**
```yaml
# Option 1: Increase memory limit
resources:
  limits:
    memory: "1Gi"  # Increase from 512Mi
  requests:
    memory: "512Mi"

# Option 2: Fix memory leak (if application issue)
# Review application logs, profile memory usage

# Option 3: Use HPA to distribute load
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    kind: Deployment
    name: myapp
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 70

# Option 4: Add memory-based eviction threshold
# (Node-level tuning)
kubelet flags:
  --eviction-hard=memory.available<100Mi
  --eviction-soft=memory.available<500Mi
```

---

### Question 27: Scenario - Jenkins pipeline is slow. How do you optimize it?
**Answer:**
**Optimization Strategies:**

**1. Parallel Stages:**
```groovy
stage('Tests') {
    parallel {
        stage('Unit') { steps { sh 'npm test:unit' } }
        stage('Integration') { steps { sh 'npm test:integration' } }
        stage('Security') { steps { sh 'npm audit' } }
    }
}
```

**2. Use Docker Caching:**
```groovy
stage('Build') {
    steps {
        script {
            docker.build("myapp:${BUILD_ID}", "--cache-from myapp:latest .")
        }
    }
}
```

**3. Skip Unnecessary Stages:**
```groovy
stage('Deploy') {
    when {
        anyOf {
            branch 'main'
            branch 'develop'
        }
        changeset "src/**"  // Only if src/ changed
    }
    steps {
        sh 'kubectl apply -f k8s/'
    }
}
```

**4. Use Build Agents with More Resources:**
```groovy
agent {
    label 'high-performance'  // Dedicated agent
}
```

**5. Incremental Builds:**
```groovy
stage('Build') {
    steps {
        sh 'mvn clean install -U -DskipTests'  // Skip tests in build
    }
}
stage('Test') {
    steps {
        sh 'mvn test'  // Test separately
    }
}
```

**6. Artifact Caching:**
```groovy
stage('Dependencies') {
    steps {
        cache(path: '.m2', key: "maven-${hashFiles('pom.xml')}") {
            sh 'mvn dependency:resolve'
        }
    }
}
```

---

### Question 28: Scenario - Azure DevOps pipeline failing to connect to AKS. How do you troubleshoot?
**Answer:**
**Troubleshooting Steps:**

**1. Verify Service Connection:**
```bash
# Azure DevOps UI:
Project Settings → Service connections → aks-connection
Click "Verify" button

# Check service principal permissions:
az role assignment list --assignee <sp-app-id> --output table

# Should have at least "Azure Kubernetes Service Cluster User Role"
az role assignment create \
  --assignee <sp-app-id> \
  --role "Azure Kubernetes Service Cluster User Role" \
  --scope /subscriptions/{sub-id}/resourceGroups/{rg}/providers/Microsoft.ContainerService/managedClusters/{aks-name}
```

**2. Test Connection Locally:**
```bash
# Login with service principal
az login --service-principal \
  -u <app-id> \
  -p <password> \
  --tenant <tenant-id>

# Get AKS credentials
az aks get-credentials --resource-group myRG --name myAKS

# Test kubectl
kubectl get nodes
```

**3. Check AKS Network Security:**
```bash
# Check if AKS has authorized IP ranges
az aks show -g myRG -n myAKS --query apiServerAccessProfile

# Add Azure DevOps agent IP (if using Microsoft-hosted)
# Or disable authorized IP ranges for testing
az aks update -g myRG -n myAKS --api-server-authorized-ip-ranges ""
```

**4. Verify RBAC:**
```bash
# Check if service principal has RBAC permissions
kubectl auth can-i get pods --as <sp-app-id>

# Grant necessary permissions
kubectl create clusterrolebinding sp-cluster-admin \
  --clusterrole=cluster-admin \
  --user=<sp-app-id>
```

**5. Check Pipeline Task Configuration:**
```yaml
- task: Kubernetes@1
  inputs:
    connectionType: 'Azure Resource Manager'
    azureSubscriptionEndpoint: 'azure-connection'  # Verify name
    azureResourceGroup: 'myRG'
    kubernetesCluster: 'myAKS'
    namespace: 'production'
    command: 'apply'
    arguments: '-f $(System.DefaultWorkingDirectory)/k8s/'
```

---

### Question 29: Scenario - Docker build is taking 15 minutes. How do you speed it up?
**Answer:**
**Optimization Techniques:**

**1. Use Multi-Stage Builds:**
```dockerfile
# Before: Single stage (slow)
FROM node:18
WORKDIR /app
COPY . .
RUN npm install  # Installs everything
RUN npm run build
CMD ["npm", "start"]

# After: Multi-stage (fast)
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production  # Cached layer
COPY . .
RUN npm run build

FROM node:18-alpine  # Smaller base
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/index.js"]
```

**2. Optimize Layer Caching:**
```dockerfile
# Bad: Changes to any file invalidate all layers
COPY . .
RUN npm install

# Good: Cache dependencies separately
COPY package*.json ./
RUN npm ci  # Cached unless package.json changes
COPY . .
```

**3. Use .dockerignore:**
```
# .dockerignore
node_modules
npm-debug.log
.git
.env
*.md
tests/
.dockerignore
Dockerfile
.github/
```

**4. Use BuildKit:**
```bash
# Enable BuildKit
export DOCKER_BUILDKIT=1

# Parallel builds
docker build --target builder -t myapp:builder .
docker build --target runtime -t myapp:latest .

# Or in Dockerfile
# syntax=docker/dockerfile:1.4
```

**5. Use Build Cache:**
```bash
# Azure Pipelines
- task: Docker@2
  inputs:
    command: build
    arguments: '--cache-from myregistry.azurecr.io/myapp:latest --build-arg BUILDKIT_INLINE_CACHE=1'

# Docker Buildx with cache
docker buildx build \
  --cache-from type=registry,ref=myregistry.azurecr.io/myapp:cache \
  --cache-to type=registry,ref=myregistry.azurecr.io/myapp:cache \
  --tag myapp:latest .
```

---

### Question 30: Scenario - Kubernetes service is not accessible. How do you debug?
**Answer:**
**Debugging Steps:**

**1. Check Service:**
```bash
# Verify service exists
kubectl get svc myapp-service

# Check service details
kubectl describe svc myapp-service

# Check endpoints
kubectl get endpoints myapp-service
# If no endpoints, pods aren't matching selector!
```

**2. Verify Pod Labels Match Service Selector:**
```bash
# Check service selector
kubectl get svc myapp-service -o yaml | grep -A 3 selector

# Check pod labels
kubectl get pods -l app=myapp -o wide

# If no pods match, fix labels:
kubectl label pods myapp-pod app=myapp
```

**3. Test Service DNS:**
```bash
# Create test pod
kubectl run test --image=busybox -it --rm -- sh

# Test DNS resolution
nslookup myapp-service
nslookup myapp-service.default.svc.cluster.local

# Test connectivity
wget -O- http://myapp-service:80
```

**4. Check Pod Health:**
```bash
# Are pods running?
kubectl get pods -l app=myapp

# Check pod logs
kubectl logs <pod-name>

# Check if pods are ready
kubectl get pods -l app=myapp -o jsonpath='{.items[*].status.conditions[?(@.type=="Ready")].status}'
```

**5. Verify Network Policies:**
```bash
# Check if network policies are blocking
kubectl get networkpolicies

# Describe network policy
kubectl describe networkpolicy <policy-name>

# Temporarily delete to test
kubectl delete networkpolicy <policy-name>
```

**6. Check Service Type:**
```bash
# ClusterIP: Only internal
# NodePort: Accessible via node IP
# LoadBalancer: External IP

# For external access, use NodePort or LoadBalancer
kubectl patch svc myapp-service -p '{"spec":{"type":"LoadBalancer"}}'
```

---

### Question 31: Theory - Explain the difference between Service Principal (Azure) and Service Account (Kubernetes)
**Answer:**

**Service Principal (Azure AD):**
- **Identity for applications** to access Azure resources
- Lives in Azure Active Directory
- Has credentials (client ID + secret or certificate)
- Used for authentication to Azure services
- Granted permissions via Azure RBAC
- Example: CI/CD pipeline deploying to Azure

```bash
# Create Service Principal
az ad sp create-for-rbac --name myapp-sp

# Grant permissions
az role assignment create \
  --assignee <app-id> \
  --role Contributor \
  --scope /subscriptions/{sub-id}

# Use in Azure DevOps/scripts
az login --service-principal \
  -u <app-id> \
  -p <password> \
  --tenant <tenant-id>
```

**Service Account (Kubernetes):**
- **Identity for pods** to access Kubernetes API
- Lives inside Kubernetes cluster
- Token automatically mounted to pod
- Used for pod-to-API-server communication
- Granted permissions via Kubernetes RBAC
- Example: Pod listing other pods, creating ConfigMaps

```yaml
# Create Service Account
apiVersion: v1
kind: ServiceAccount
metadata:
  name: myapp-sa
---
# Grant permissions
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
subjects:
- kind: ServiceAccount
  name: myapp-sa
roleRef:
  kind: Role
  name: pod-reader
---
# Use in Pod
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  serviceAccountName: myapp-sa
  containers:
  - name: app
    image: myapp:1.0
```

**Key Differences:**

| Aspect | Service Principal | Service Account |
|--------|------------------|-----------------|
| Scope | Azure AD (cloud) | Kubernetes (cluster) |
| Purpose | Azure resource access | K8s API access |
| Authentication | Client ID + Secret/Cert | Token (auto-mounted) |
| Permissions | Azure RBAC | Kubernetes RBAC |
| Use Case | CI/CD, Azure automation | Pod-to-API communication |
| Location | External to cluster | Internal to cluster |

**Common Scenario:**
```
CI/CD Pipeline needs to:
1. Deploy to Azure (uses Service Principal)
2. Deploy to AKS (uses Service Principal to authenticate to AKS)
3. Pod in AKS needs to list other pods (uses Service Account)

Flow:
Azure DevOps → Service Principal → Authenticate to Azure
              → Get AKS credentials
              → Deploy pod with Service Account
              → Pod uses Service Account → Access K8s API
```

---

### Question 32: Real-World Issue - Describe a common issue you faced with persistent volumes in Kubernetes and how you resolved it
**Answer:**

**Issue: PVC Stuck in Pending State**

**Scenario:**
```bash
kubectl get pvc
NAME        STATUS    VOLUME   CAPACITY
data-pvc    Pending            10Gi

# Application pods couldn't start because PVC wasn't bound
```

**Root Cause Analysis:**
```bash
# 1. Describe PVC
kubectl describe pvc data-pvc
# Events showed: "no persistent volumes available"

# 2. Check available PVs
kubectl get pv
# No PVs available or no matching PVs

# 3. Check StorageClass
kubectl get storageclass
# StorageClass not set as default or provisioner not working
```

**Resolution Steps:**

**Problem 1: No StorageClass or not default**
```bash
# Check StorageClass
kubectl get storageclass

# Set default StorageClass
kubectl patch storageclass managed-premium -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'

# Update PVC to use StorageClass
kubectl edit pvc data-pvc
# Add: storageClassName: managed-premium
```

**Problem 2: Dynamic provisioning not working (AKS)**
```bash
# Check Azure CSI driver
kubectl get pods -n kube-system | grep csi

# If missing, install
helm repo add azuredisk-csi-driver https://raw.githubusercontent.com/kubernetes-sigs/azuredisk-csi-driver/master/charts
helm install azuredisk-csi-driver azuredisk-csi-driver/azuredisk-csi-driver --namespace kube-system

# Verify provisioner
kubectl get storageclass managed-premium -o yaml | grep provisioner
# Should show: disk.csi.azure.com
```

**Problem 3: Access mode mismatch**
```yaml
# Original PVC (wrong)
spec:
  accessModes:
  - ReadWriteMany  # Azure Disk doesn't support RWX!

# Fixed PVC
spec:
  accessModes:
  - ReadWriteOnce  # Use RWO for Azure Disk
  # Or use Azure Files for RWX
```

**Problem 4: Insufficient permissions**
```bash
# Check service principal permissions
az role assignment list --assignee <sp-id>

# Grant required role
az role assignment create \
  --assignee <sp-id> \
  --role "Contributor" \
  --scope /subscriptions/{sub-id}/resourceGroups/{aks-rg}
```

**Final Solution:**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: managed-premium  # Explicit StorageClass
  resources:
    requests:
      storage: 10Gi
```

**Lessons Learned:**
- Always specify StorageClass explicitly
- Understand access mode limitations per storage type
- Verify CSI drivers are running
- Check service principal permissions for dynamic provisioning
- Monitor PVC events immediately after creation

---

### Question 33: Azure DevOps - How do you implement blue-green deployment in Azure Pipelines with AKS?
**Answer:**
```yaml
# Blue-Green Deployment Strategy
trigger:
- main

variables:
  imageTag: '$(Build.BuildId)'
  blueNamespace: 'production-blue'
  greenNamespace: 'production-green'

stages:
- stage: Build
  jobs:
  - job: BuildImage
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - task: Docker@2
      inputs:
        command: buildAndPush
        repository: 'myapp'
        tags: |
          $(imageTag)
          latest

- stage: DeployGreen
  dependsOn: Build
  jobs:
  - deployment: DeployToGreen
    environment: 'green-environment'
    pool:
      vmImage: 'ubuntu-latest'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: KubernetesManifest@0
            displayName: 'Deploy to Green'
            inputs:
              action: 'deploy'
              kubernetesServiceConnection: 'aks-connection'
              namespace: $(greenNamespace)
              manifests: |
                $(Pipeline.Workspace)/manifests/deployment.yaml
                $(Pipeline.Workspace)/manifests/service-green.yaml
          
          - script: |
              # Wait for deployment to be ready
              kubectl rollout status deployment/myapp -n $(greenNamespace)
            displayName: 'Wait for rollout'
          
          - script: |
              # Run smoke tests against green
              GREEN_URL=$(kubectl get svc myapp-green -n $(greenNamespace) -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
              curl -f http://$GREEN_URL/health || exit 1
            displayName: 'Smoke tests on Green'

- stage: SwitchTraffic
  dependsOn: DeployGreen
  jobs:
  - job: ApprovalGate
    pool: server
    steps:
    - task: ManualValidation@0
      inputs:
        notifyUsers: 'ops-team@example.com'
        instructions: 'Validate green environment before switching traffic'
  
  - deployment: SwitchToGreen
    dependsOn: ApprovalGate
    environment: 'production'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: Kubernetes@1
            displayName: 'Switch traffic to Green'
            inputs:
              connectionType: 'Azure Resource Manager'
              azureSubscriptionEndpoint: 'aks-connection'
              namespace: 'production'
              command: 'apply'
              arguments: '-f $(Pipeline.Workspace)/manifests/service-main.yaml'
              # service-main.yaml points to green pods

- stage: CleanupBlue
  dependsOn: SwitchTraffic
  jobs:
  - job: RemoveBlue
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - task: Kubernetes@1
      displayName: 'Scale down Blue'
      inputs:
        command: 'scale'
        arguments: 'deployment/myapp --replicas=0 -n $(blueNamespace)'
```

**Kubernetes Manifests:**
```yaml
# deployment.yaml (Green)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: production-green
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
      - name: myapp
        image: myregistry.azurecr.io/myapp:{{BUILD_ID}}

---
# service-main.yaml (Traffic routing)
apiVersion: v1
kind: Service
metadata:
  name: myapp
  namespace: production
spec:
  selector:
    app: myapp
    version: green  # Switch between blue/green
  ports:
  - port: 80
    targetPort: 8080
  type: LoadBalancer
```

---

### Question 34: Terraform - Common issue with state file locking. How do you handle it?
**Answer:**

**Issue: State Lock Error**
```bash
terraform apply

Error: Error acquiring the state lock
Lock Info:
  ID: abc123-def456
  Path: terraform.tfstate
  Operation: OperationTypeApply
  Who: user@hostname
  Version: 1.5.0
  Created: 2024-01-08 10:30:00
  Info: 

Terraform acquires a state lock to protect the state from being written
by multiple users at the same time. Please resolve the issue above and try
again. For most commands, you can disable locking with the "-lock=false"
flag, but this is not recommended.
```

**Root Causes:**
1. Another terraform process running
2. Previous terraform crashed without releasing lock
3. Multiple users/pipelines running simultaneously
4. Network issues during remote state operations

**Resolution:**

**Method 1: Wait for Lock Release (if legitimate process)**
```bash
# Check who has the lock
terraform force-unlock <lock-id>

# But first, verify no one is actually running terraform!
# Contact the user shown in lock info
```

**Method 2: Force Unlock (if stuck lock)**
```bash
# Only if you're SURE no other process is running
terraform force-unlock abc123-def456

# Confirm by typing: yes
```

**Method 3: Azure Backend - Manual Unlock**
```bash
# Check blob lease (Azure Storage backend)
az storage blob lease break \
  --blob-name terraform.tfstate \
  --container-name tfstate \
  --account-name mystorageaccount
```

**Prevention:**

**1. Use Remote Backend with Locking:**
```hcl
# Azure Storage backend
terraform {
  backend "azurerm" {
    resource_group_name  = "terraform-rg"
    storage_account_name = "tfstatestore"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
    use_azuread_auth     = true
  }
}

# This provides automatic state locking
```

**2. In CI/CD - Use Single Agent:**
```yaml
# Azure Pipelines - prevent parallel runs
trigger:
  batch: true  # Batch changes
  
# Or use exclusive lock
lockBehavior: sequential
```

**3. Set Timeouts:**
```bash
# Increase timeout if network is slow
export TF_LOCK_TIMEOUT=10m
terraform apply
```

**4. Cleanup on Pipeline Failure:**
```yaml
steps:
- script: terraform apply -auto-approve
  displayName: 'Terraform Apply'
  
# Cleanup on failure
- script: |
    LOCK_ID=$(terraform force-unlock 2>&1 | grep "Lock ID" | cut -d: -f2 | xargs)
    if [ ! -z "$LOCK_ID" ]; then
      terraform force-unlock -force $LOCK_ID
    fi
  condition: failed()
  displayName: 'Cleanup lock on failure'
```

**Best Practices:**
- Never use `-lock=false` in production
- Always use remote backend with locking
- Coordinate with team before force-unlock
- Check lock info before forcing unlock
- Implement proper CI/CD queuing
- Use workspace isolation for parallel work

---

### Question 35: Docker - You have a 2GB Docker image. How do you optimize it?
**Answer:**

**Initial Dockerfile (2GB):**
```dockerfile
FROM ubuntu:latest
RUN apt-get update && apt-get install -y \
    python3 python3-pip curl wget git vim \
    && pip3 install flask pandas numpy scikit-learn
COPY . /app
WORKDIR /app
CMD ["python3", "app.py"]
```

**Optimization Strategies:**

**1. Use Smaller Base Image:**
```dockerfile
# Before: ubuntu:latest (77MB base)
# After: python:3.11-slim (126MB) or python:3.11-alpine (49MB)

FROM python:3.11-slim
# Saves: 50-100MB
```

**2. Multi-Stage Build:**
```dockerfile
# Stage 1: Build dependencies
FROM python:3.11 AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Stage 2: Runtime
FROM python:3.11-slim
WORKDIR /app

# Copy only necessary files from builder
COPY --from=builder /root/.local /root/.local
COPY app.py .
COPY templates/ ./templates/

# Make sure scripts are in PATH
ENV PATH=/root/.local/bin:$PATH

CMD ["python", "app.py"]

# Result: 2GB → 500MB
```

**3. Layer Optimization:**
```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies (cached layer)
RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Copy only requirements first (cached layer)
COPY requirements.txt .

# Install Python dependencies (cached unless requirements.txt changes)
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code (changes frequently)
COPY . .

# Remove build dependencies
RUN apt-get purge -y --auto-remove gcc

CMD ["python", "app.py"]
```

**4. Remove Unnecessary Files:**
```dockerfile
# .dockerignore
__pycache__/
*.pyc
*.pyo
*.pyd
.Python
env/
venv/
.venv
.git/
.gitignore
*.md
tests/
docs/
*.log
.DS_Store
node_modules/
```

**5. Use Specific Package Versions:**
```dockerfile
# Before: Install everything
RUN pip install flask pandas numpy scikit-learn jupyter matplotlib

# After: Only what you need
RUN pip install --no-cache-dir \
    flask==2.3.0 \
    pandas==2.0.0
```

**6. Minimize Layers:**
```dockerfile
# Bad: Multiple RUN commands (more layers)
RUN apt-get update
RUN apt-get install -y python3
RUN apt-get install -y pip
RUN pip install flask

# Good: Single RUN command
RUN apt-get update && apt-get install -y \
    python3 python3-pip \
    && pip install flask \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*
```

**Final Optimized Dockerfile:**
```dockerfile
FROM python:3.11-slim AS builder

WORKDIR /build

# Install build dependencies
RUN apt-get update && apt-get install -y --no-install-recommends gcc \
    && rm -rf /var/lib/apt/lists/*

# Install Python packages
COPY requirements.txt .
RUN pip wheel --no-cache-dir --no-deps --wheel-dir /build/wheels -r requirements.txt

# Final stage
FROM python:3.11-slim

WORKDIR /app

# Copy wheels and install
COPY --from=builder /build/wheels /wheels
RUN pip install --no-cache-dir /wheels/* \
    && rm -rf /wheels

# Copy application
COPY app.py .
COPY templates/ ./templates/

# Create non-root user
RUN useradd -m -u 1000 appuser && chown -R appuser:appuser /app
USER appuser

EXPOSE 5000

CMD ["python", "app.py"]

# Result: 2GB → 180MB (91% reduction!)
```

**Analysis:**
```bash
# Compare images
docker images

REPOSITORY   TAG       SIZE
myapp        before    2.0GB
myapp        after     180MB

# Reduction: 1.82GB (91% smaller)
# Build time: 50% faster (cached layers)
# Security: Non-root user, minimal packages
```

---

### Question 36: Kubernetes - Explain a scenario where you had pod eviction issues and how you debugged it
**Answer:**

**Scenario: Pods Being Evicted Unexpectedly**

**Initial Problem:**
```bash
kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
app-1                   1/1     Running   5          10m
app-2                   0/1     Evicted   0          2m
app-3                   1/1     Running   3          8m
db-1                    0/1     Evicted   0          5m
```

**Investigation Steps:**

**1. Check Pod Events:**
```bash
kubectl describe pod app-2

Events:
  Type     Reason                Message
  ----     ------                -------
  Warning  Evicted               The node was low on resource: memory. Container app was using 1024Mi, which exceeds its request of 512Mi.
  Normal   Killing               Stopping container app
```

**2. Check Node Resources:**
```bash
kubectl describe node worker-1

Conditions:
  Type             Status
  ----             ------
  MemoryPressure   True   # Node under memory pressure!
  DiskPressure     False
  Ready            True

Allocated resources:
  Resource           Requests      Limits
  --------           --------      ------
  cpu                7200m (90%)   14000m (175%)
  memory             28Gi (95%)    40Gi (136%)

# Node is overcommitted!
```

**3. Identify Resource Hogs:**
```bash
kubectl top pods --all-namespaces --sort-by=memory

NAMESPACE     NAME                    CPU    MEMORY
production    app-memory-leak-1       250m   1950Mi  # Memory leak!
production    app-1                   100m   1024Mi
production    db-1                    50m    800Mi
```

**4. Check Pod QoS Classes:**
```bash
kubectl get pod app-2 -o jsonpath='{.status.qosClass}'
# Burstable

# Eviction order:
# 1. BestEffort pods (no requests/limits)
# 2. Burstable pods exceeding requests
# 3. Guaranteed pods (last to evict)
```

**Root Causes Found:**
1. Memory leak in `app-memory-leak-1`
2. Pods without proper resource limits
3. Node overcommitment
4. No resource quotas per namespace

**Solutions Implemented:**

**Solution 1: Fix Memory Leak**
```bash
# Investigate memory leak
kubectl exec -it app-memory-leak-1 -- sh
# Profile application, find leak
# Deploy fixed version

# Immediate mitigation: Restart pod
kubectl delete pod app-memory-leak-1
```

**Solution 2: Set Proper Resource Limits**
```yaml
# Before (caused eviction)
spec:
  containers:
  - name: app
    image: myapp:1.0
    resources:
      requests:
        memory: "512Mi"
      # No limits! Can use unlimited memory

# After (prevents eviction)
spec:
  containers:
  - name: app
    image: myapp:1.0
    resources:
      requests:
        memory: "512Mi"
        cpu: "250m"
      limits:
        memory: "1Gi"      # Hard limit
        cpu: "500m"
      # QoS: Burstable (safer)
```

**Solution 3: Use Guaranteed QoS for Critical Pods**
```yaml
# Database pod - critical, don't evict
spec:
  containers:
  - name: postgres
    image: postgres:14
    resources:
      requests:
        memory: "2Gi"
        cpu: "1000m"
      limits:
        memory: "2Gi"    # Same as requests
        cpu: "1000m"     # Same as requests
      # QoS: Guaranteed (last to evict)
```

**Solution 4: Implement Resource Quotas**
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: production-quota
  namespace: production
spec:
  hard:
    requests.cpu: "20"
    requests.memory: 40Gi
    limits.cpu: "40"
    limits.memory: 80Gi
    pods: "50"
    persistentvolumeclaims: "10"

# Prevents namespace from consuming all node resources
```

**Solution 5: Configure Kubelet Eviction Thresholds**
```yaml
# Kubelet configuration (on nodes)
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
evictionHard:
  memory.available: "500Mi"
  nodefs.available: "10%"
evictionSoft:
  memory.available: "1Gi"
evictionSoftGracePeriod:
  memory.available: "2m"
```

**Solution 6: Add More Nodes or Upgrade**
```bash
# Scale node pool
az aks nodepool scale \
  --resource-group myRG \
  --cluster-name myAKS \
  --name nodepool1 \
  --node-count 5  # Increase from 3

# Or upgrade node size
az aks nodepool update \
  --resource-group myRG \
  --cluster-name myAKS \
  --name nodepool1 \
  --enable-cluster-autoscaler \
  --min-count 3 \
  --max-count 10
```

**Monitoring Added:**
```yaml
# Prometheus alert for memory pressure
alert: NodeMemoryPressure
expr: kube_node_status_condition{condition="MemoryPressure",status="true"} == 1
for: 5m
annotations:
  summary: "Node {{ $labels.node }} under memory pressure"
  
# Alert for pod evictions
alert: PodEvictionRate
expr: rate(kube_pod_status_reason{reason="Evicted"}[5m]) > 0
annotations:
  summary: "Pods being evicted in {{ $labels.namespace }}"
```

**Lessons Learned:**
- Always set resource limits, not just requests
- Use Guaranteed QoS for critical workloads
- Implement namespace resource quotas
- Monitor node resource usage proactively
- Fix application memory leaks promptly
- Enable cluster autoscaler for dynamic scaling
- Set up alerts for memory pressure and evictions

---

### Question 37: Jenkins - How do you secure Jenkins in production?
**Answer:**

**Security Hardening Checklist:**

**1. Enable Security Realm:**
```
Manage Jenkins → Configure Global Security
Security Realm: Jenkins' own user database
Authorization: Matrix-based security

Create admin user, disable anonymous access
```

**2. Configure Authorization Matrix:**
```
Overall:
  Admin: Read, Administer
  Developers: Read
  
Job:
  Developers: Build, Cancel, Read
  Viewers: Read only
  
Credentials:
  Admin: Create, Update, View, Delete
  Developers: View only (for specific credentials)
```

**3. Use HTTPS:**
```bash
# Install SSL certificate
keytool -genkey -keyalg RSA -alias jenkins -keystore jenkins.jks

# Jenkins configuration
JENKINS_ARGS="--httpPort=-1 --httpsPort=8443 --httpsKeyStore=/path/to/jenkins.jks --httpsKeyStorePassword=password"

# Or use reverse proxy (nginx)
server {
    listen 443 ssl;
    server_name jenkins.example.com;
    
    ssl_certificate /etc/ssl/certs/jenkins.crt;
    ssl_certificate_key /etc/ssl/private/jenkins.key;
    
    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

**4. Secure Credentials:**
```groovy
// Use Jenkins Credentials Plugin
pipeline {
    agent any
    environment {
        // Reference credential by ID
        DB_CREDS = credentials('database-credentials')
        DOCKER_CREDS = credentials('dockerhub-credentials')
    }
    steps {
        script {
            // Credentials masked in logs
            sh 'echo $DB_CREDS_USR'  // Username
            sh 'echo $DB_CREDS_PSW'  // Password (masked)
        }
    }
}

// Never hardcode:
// ✗ sh 'docker login -u admin -p password123'
// ✓ sh 'docker login -u $DOCKER_CREDS_USR -p $DOCKER_CREDS_PSW'
```

**5. Restrict Script Execution:**
```
Manage Jenkins → Configure Global Security
CSRF Protection: ✓ Enable
Markup Formatter: Plain text (not Raw HTML)
Script Security for Job DSL: ✓ Enable

In-process Script Approval:
Review and approve all scripts before execution
```

**6. Install Security Plugins:**
```
Recommended plugins:
- OWASP Markup Formatter
- Strict Crumb Issuer
- Role-based Authorization Strategy
- Audit Trail
- Job Restrictions
```

**7. Secure Jenkins Configuration:**
```groovy
// Jenkinsfile security practices
pipeline {
    agent {
        label 'trusted-agent'  // Use specific agents
    }
    
    options {
        // Prevent concurrent builds
        disableConcurrentBuilds()
        
        // Timeout to prevent hanging
        timeout(time: 1, unit: 'HOURS')
        
        // Limit build history
        buildDiscarder(logRotator(
            numToKeepStr: '10',
            artifactNumToKeepStr: '5'
        ))
    }
    
    stages {
        stage('Build') {
            steps {
                // Use specific tool versions
                tool name: 'Maven-3.8', type: 'maven'
                
                // Validate input
                script {
                    if (!env.BRANCH_NAME.matches(/^(main|develop|feature\/.*)$/)) {
                        error "Invalid branch name"
                    }
                }
            }
        }
    }
}
```

**8. Network Security:**
```bash
# Firewall rules (iptables)
iptables -A INPUT -p tcp --dport 8080 -s 10.0.0.0/8 -j ACCEPT  # Internal only
iptables -A INPUT -p tcp --dport 8080 -j DROP  # Block external

# Or use Azure NSG
az network nsg rule create \
  --resource-group myRG \
  --nsg-name jenkins-nsg \
  --name Allow-Jenkins \
  --priority 100 \
  --source-address-prefixes 10.0.0.0/8 \
  --destination-port-ranges 8080 \
  --access Allow
```

**9. Audit Logging:**
```
Install Audit Trail plugin

Manage Jenkins → Configure System → Audit Trail
Log location: /var/log/jenkins/audit.log
Pattern: /.*/ (log everything)

Monitor:
- User logins
- Job executions
- Configuration changes
- Credential access
```

**10. Regular Updates:**
```bash
# Automated Jenkins updates (with backup)
#!/bin/bash
# backup-and-update-jenkins.sh

# Backup
BACKUP_DIR="/backup/jenkins/$(date +%Y%m%d)"
mkdir -p $BACKUP_DIR
cp -r /var/lib/jenkins/ $BACKUP_DIR/

# Update Jenkins
wget http://updates.jenkins.io/latest/jenkins.war
cp jenkins.war /usr/share/jenkins/

# Update plugins
java -jar jenkins-cli.jar -s http://localhost:8080/ install-plugin \
  $(java -jar jenkins-cli.jar -s http://localhost:8080/ list-plugins | grep -e ')$' | awk '{print $1}')

# Restart
systemctl restart jenkins
```

**11. Agent Security:**
```
Master-Agent security:
- Use SSH-based agents (not JNLP)
- Different credentials per agent
- Limit agent permissions

// Pipeline
agent {
    label 'secure-agent'
    customWorkspace '/secure/workspace'
}
```

**12. Secrets Scanning:**
```groovy
// Pre-commit hook or pipeline stage
stage('Security Scan') {
    steps {
        // Scan for hardcoded secrets
        sh '''
            git secrets --scan
            trufflehog --regex --entropy=False .
        '''
        
        // Dependency vulnerability scan
        sh 'npm audit'
        sh 'safety check'  // Python
        sh 'snyk test'
    }
}
```

---

### Question 38: Real Production Issue - Kubernetes cluster upgrade broke applications. How do you handle it?
**Answer:**

**Scenario: AKS Cluster Upgrade from 1.24 to 1.25**

**Initial Problem:**
```bash
# After upgrade
kubectl get pods
NAME                    READY   STATUS             RESTARTS
app-1                   0/1     CrashLoopBackOff   5
app-2                   0/1     ImagePullBackOff   0
ingress-controller      0/1     Error              10
```

**Breaking Changes in K8s 1.25:**
1. PodSecurityPolicy removed (replaced with Pod Security Standards)
2. APIs removed (batch/v1beta1 CronJob, autoscaling/v2beta1 HPA)
3. Dockershim removed (container runtime)

**Investigation Steps:**

**1. Check API Version Deprecations:**
```bash
# Check for deprecated APIs
kubectl get --raw /metrics | grep apiserver_requested_deprecated_apis

# List resources using old APIs
kubectl get cronjobs.v1beta1.batch --all-namespaces
# Error: the server doesn't have a resource type "cronjobs"
```

**2. Review PSP Impact:**
```bash
# Old PSPs no longer work
kubectl get psp
# No resources found (PSPs removed in 1.25)

# Pods failing due to missing PSP
kubectl describe pod app-1
Events:
  Warning FailedCreate unable to validate against any pod security policy
```

**3. Check Container Runtime:**
```bash
kubectl get nodes -o wide
# CONTAINER-RUNTIME column shows containerd (good)

# If using Docker, migration needed
```

**Resolution Steps:**

**Step 1: Prepare Migration Plan**
```bash
# BEFORE upgrade (should have done):
1. Run kubectl-convert to update manifests
2. Test in staging environment
3. Review release notes for breaking changes
4. Backup all resources
```

**Step 2: Fix API Versions**
```yaml
# Old (broken)
apiVersion: batch/v1beta1
kind: CronJob
---
apiVersion: autoscaling/v2beta1
kind: HorizontalPodAutoscaler

# New (fixed)
apiVersion: batch/v1  # Updated API version
kind: CronJob
spec:
  schedule: "0 0 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: job
            image: myapp:1.0
          restartPolicy: OnFailure
---
apiVersion: autoscaling/v2  # Updated API version
kind: HorizontalPodAutoscaler
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
```

**Step 3: Migrate from PSP to Pod Security Standards**
```yaml
# Label namespaces with Pod Security Standards
kubectl label namespace production \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/audit=restricted \
  pod-security.kubernetes.io/warn=restricted

# Update pods to meet restricted standards
apiVersion: v1
kind: Pod
metadata:
  name: secure-app
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: myapp:1.0
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop:
        - ALL
      readOnlyRootFilesystem: true
```

**Step 4: Fix Ingress Controller**
```bash
# Ingress controller using old API
kubectl get ingress
# apiVersion: networking.k8s.io/v1beta1 (deprecated)

# Update to new API
kubectl get ingress myapp -o yaml | \
  sed 's/networking.k8s.io\/v1beta1/networking.k8s.io\/v1/' | \
  kubectl apply -f -

# Update ingress controller itself
helm upgrade ingress-nginx ingress-nginx/ingress-nginx \
  --version 4.7.0 \
  --namespace ingress-nginx
```

**Step 5: Rollback if Critical (Emergency)**
```bash
# If applications completely broken, rollback cluster
az aks upgrade \
  --resource-group myRG \
  --name myAKS \
  --kubernetes-version 1.24.10

# Note: Rollback may not always be possible!
# Some changes are irreversible
```

**Step 6: Gradual Migration (Blue-Green Cluster)**
```bash
# Better approach: Create new cluster with 1.25
az aks create \
  --resource-group myRG \
  --name myAKS-1-25 \
  --kubernetes-version 1.25.6 \
  --node-count 3

# Migrate applications gradually
# Test each application in new cluster
# Switch traffic with DNS/Load Balancer
# Decommission old cluster once stable
```

**Prevention for Future Upgrades:**

**1. Pre-Upgrade Validation:**
```bash
# Use kubectl-convert
kubectl-convert -f old-manifest.yaml --output-version apps/v1

# Use Pluto to detect deprecated APIs
pluto detect-files -d manifests/

# Use kube-no-trouble
kubent

# Output:
# CronJob (batch/v1beta1) → Use batch/v1
# HPA (autoscaling/v2beta1) → Use autoscaling/v2
```

**2. Staging Environment Testing:**
```bash
# Always test upgrades in staging first
# Staging should match production version

az aks create \
  --resource-group staging-rg \
  --name staging-aks \
  --kubernetes-version 1.25.6

# Deploy all apps, run tests
# Validate for 1-2 weeks
# Then upgrade production
```

**3. Automated Pre-Upgrade Checks:**
```yaml
# Azure Pipeline - Pre-upgrade validation
stages:
- stage: ValidateUpgrade
  jobs:
  - job: CheckDeprecations
    steps:
    - script: |
        # Install tools
        wget https://github.com/FairwindsOps/pluto/releases/download/v5.0.0/pluto_5.0.0_linux_amd64.tar.gz
        tar xzf pluto_5.0.0_linux_amd64.tar.gz
        
        # Scan manifests
        ./pluto detect-files -d k8s/ -o wide
        
        # Fail if deprecated APIs found
        if ./pluto detect-files -d k8s/ | grep -q "DEPRECATED"; then
          echo "Found deprecated APIs!"
          exit 1
        fi
      displayName: 'Check for deprecated APIs'
    
    - script: |
        # Check PSP usage
        kubectl get psp -o yaml > psp-backup.yaml
        
        # Generate PSS migration
        kubectl label namespace production \
          pod-security.kubernetes.io/warn=restricted --dry-run=client
      displayName: 'PSP to PSS migration check'
```

**Lessons Learned:**
- Always review Kubernetes release notes for breaking changes
- Test upgrades in staging environment first
- Use automated tools to detect deprecated APIs
- Have rollback plan (or blue-green cluster strategy)
- Upgrade incrementally (skip no more than 1 minor version)
- Monitor applications closely post-upgrade
- Schedule upgrades during maintenance windows
- Backup all resources before upgrade

---

### Question 39: Azure DevOps - How do you implement deployment slots with Azure Web Apps in pipelines?
**Answer:**

**Deployment Slots Strategy:**
Deployment slots allow zero-downtime deployments with easy rollback.

```yaml
# azure-pipelines-slots.yml
trigger:
- main

variables:
  azureSubscription: 'azure-prod-connection'
  webAppName: 'mywebapp'
  resourceGroup: 'myapp-rg'
  slotName: 'staging'

stages:
- stage: Build
  jobs:
  - job: BuildJob
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - task: NodeTool@0
      inputs:
        versionSpec: '18.x'
    
    - script: |
        npm ci
        npm run build
        npm test
      displayName: 'Build and test'
    
    - task: ArchiveFiles@2
      inputs:
        rootFolderOrFile: '$(System.DefaultWorkingDirectory)'
        includeRootFolder: false
        archiveType: 'zip'
        archiveFile: '$(Build.ArtifactStagingDirectory)/$(Build.BuildId).zip'
    
    - publish: $(Build.ArtifactStagingDirectory)/$(Build.BuildId).zip
      artifact: drop

- stage: DeployToSlot
  dependsOn: Build
  jobs:
  - deployment: DeployStaging
    environment: 'staging'
    pool:
      vmImage: 'ubuntu-latest'
    strategy:
      runOnce:
        deploy:
          steps:
          - download: current
            artifact: drop
          
          - task: AzureWebApp@1
            displayName: 'Deploy to Staging Slot'
            inputs:
              azureSubscription: '$(azureSubscription)'
              appType: 'webAppLinux'
              appName: '$(webAppName)'
              deployToSlotOrASE: true
              resourceGroupName: '$(resourceGroup)'
              slotName: '$(slotName)'
              package: '$(Pipeline.Workspace)/drop/$(Build.BuildId).zip'
          
          - task: AzureCLI@2
            displayName: 'Warm up staging slot'
            inputs:
              azureSubscription: '$(azureSubscription)'
              scriptType: 'bash'
              scriptLocation: 'inlineScript'
              inlineScript: |
                # Get staging slot URL
                SLOT_URL=$(az webapp show \
                  --name $(webAppName) \
                  --resource-group $(resourceGroup) \
                  --slot $(slotName) \
                  --query defaultHostName -o tsv)
                
                # Warm up application
                for i in {1..5}; do
                  curl -f https://$SLOT_URL/health || echo "Warming up..."
                  sleep 2
                done
          
          - task: AzureCLI@2
            displayName: 'Run smoke tests on staging'
            inputs:
              azureSubscription: '$(azureSubscription)'
              scriptType: 'bash'
              scriptLocation: 'inlineScript'
              inlineScript: |
                SLOT_URL=$(az webapp show \
                  --name $(webAppName) \
                  --resource-group $(resourceGroup) \
                  --slot $(slotName) \
                  --query defaultHostName -o tsv)
                
                # Health check
                if ! curl -f https://$SLOT_URL/health; then
                  echo "Health check failed!"
                  exit 1
                fi
                
                # Test critical endpoints
                if ! curl -f https://$SLOT_URL/api/status; then
                  echo "API test failed!"
                  exit 1
                fi
                
                echo "All smoke tests passed!"

- stage: ApprovalGate
  dependsOn: DeployToSlot
  jobs:
  - job: WaitForValidation
    pool: server
    steps:
    - task: ManualValidation@0
      timeoutInMinutes: 1440  # 24 hours
      inputs:
        notifyUsers: 'devops-team@example.com'
        instructions: |
          Please validate the staging slot before swapping to production.
          
          Staging URL: https://mywebapp-staging.azurewebsites.net
          
          Test:
          1. Functionality
          2. Performance
          3. UI/UX
          4. API endpoints
          
          Click Resume to proceed with swap.

- stage: SwapSlots
  dependsOn: ApprovalGate
  jobs:
  - deployment: SwapToProduction
    environment: 'production'
    pool:
      vmImage: 'ubuntu-latest'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureAppServiceManage@0
            displayName: 'Swap Staging to Production'
            inputs:
              azureSubscription: '$(azureSubscription)'
              action: 'Swap Slots'
              webAppName: '$(webAppName)'
              resourceGroupName: '$(resourceGroup)'
              sourceSlot: '$(slotName)'
              targetSlot: 'production'
          
          - task: AzureCLI@2
            displayName: 'Verify production'
            inputs:
              azureSubscription: '$(azureSubscription)'
              scriptType: 'bash'
              scriptLocation: 'inlineScript'
              inlineScript: |
                PROD_URL=$(az webapp show \
                  --name $(webAppName) \
                  --resource-group $(resourceGroup) \
                  --query defaultHostName -o tsv)
                
                # Wait for DNS propagation
                sleep 30
                
                # Verify production
                if ! curl -f https://$PROD_URL/health; then
                  echo "Production verification failed! Rolling back..."
                  
                  # Rollback - swap back
                  az webapp deployment slot swap \
                    --name $(webAppName) \
                    --resource-group $(resourceGroup) \
                    --slot $(slotName) \
                    --target-slot production
                  
                  exit 1
                fi
                
                echo "Production deployment successful!"
```

**Manual Slot Management:**
```bash
# Create deployment slot
az webapp deployment slot create \
  --name mywebapp \
  --resource-group myapp-rg \
  --slot staging \
  --configuration-source mywebapp

# Deploy to slot
az webapp deployment source config-zip \
  --name mywebapp \
  --resource-group myapp-rg \
  --slot staging \
  --src package.zip

# Swap slots
az webapp deployment slot swap \
  --name mywebapp \
  --resource-group myapp-rg \
  --slot staging \
  --target-slot production

# Rollback (swap back)
az webapp deployment slot swap \
  --name mywebapp \
  --resource-group myapp-rg \
  --slot production \
  --target-slot staging
```

**Slot Configuration:**
```bash
# Configure slot-specific settings (don't swap with prod)
az webapp config appsettings set \
  --name mywebapp \
  --resource-group myapp-rg \
  --slot staging \
  --settings \
    ENVIRONMENT=staging \
    DATABASE_URL=staging-db.database.azure.com \
  --slot-settings ENVIRONMENT DATABASE_URL  # Mark as slot-specific

# These settings stay with the slot during swap
```

---

### Question 40: Linux - A server is running slow. Walk me through your troubleshooting process
**Answer:**

**Systematic Troubleshooting Approach:**

**1. Initial Assessment:**
```bash
# Check system load
uptime
# 10:30:51 up 45 days, 3 users, load average: 15.50, 12.30, 8.90
# Load > CPU count = overloaded!

# Check running processes
top
# Press 1 for per-CPU view
# Press M to sort by memory
# Press P to sort by CPU

# Quick system overview
vmstat 1 5  # 5 samples, 1 second apart
```

**2. Check CPU Usage:**
```bash
# Identify CPU hogs
ps aux --sort=-%cpu | head -10

# Check per-CPU usage
mpstat -P ALL 1

# Identify what's using CPU
pidstat 1 5

# If high system CPU (not user)
# Check for I/O wait
iostat -x 1 5

# sy (system) high = kernel issue
# wa (iowait) high = disk I/O bottleneck
```

**3. Check Memory:**
```bash
# Memory overview
free -h
#               total        used        free      shared  buff/cache   available
# Mem:           31Gi        25Gi       500Mi       2.0Gi        5.5Gi        3.8Gi
# Swap:          8.0Gi       6.0Gi       2.0Gi  # Swap being used = bad!

# Identify memory hogs
ps aux --sort=-%mem | head -10

# Check for OOM killer activity
dmesg | grep -i "out of memory"
grep -i "killed process" /var/log/messages

# Detailed memory breakdown
cat /proc/meminfo

# Check swap usage
swapon --show
vmstat 1 5  # Check 'si' and 'so' columns (swap in/out)
```

**4. Check Disk I/O:**
```bash
# Disk usage
df -h
# If any filesystem > 90% full = problem!

# Identify I/O bottleneck
iostat -x 1 5
# Look at %util column
# > 80% = disk saturated

# Which processes doing I/O
iotop -o  # Only show active I/O

# Check inode usage (can be full even if disk has space)
df -i

# Find large files
du -h / | sort -rh | head -20
find / -type f -size +1G -exec ls -lh {} \; 2>/dev/null
```

**5. Check Network:**
```bash
# Network connections
netstat -tulanp | grep ESTABLISHED | wc -l
# Too many connections?

# Network throughput
iftop  # Or
nload

# Check for packet drops
netstat -i
# Look for errors, dropped packets

# DNS issues
time nslookup google.com
# If slow, DNS problem
```

**6. Check Logs:**
```bash
# System logs
tail -f /var/log/syslog
tail -f /var/log/messages

# Check for errors
journalctl -p err -b  # Errors from current boot

# Application logs
tail -f /var/log/nginx/error.log
tail -f /var/log/mysql/error.log

# Kernel messages
dmesg | tail -50
```

**7. Process-Specific Investigation:**
```bash
# If specific process identified (e.g., PID 1234)

# Process details
ps -p 1234 -f

# Open files
lsof -p 1234 | wc -l  # File descriptor leak?

# Network connections
lsof -i -p 1234

# System calls (strace - careful in production)
strace -p 1234 -c  # Summary
strace -p 1234 -e trace=open,read,write  # Specific calls

# Memory maps
pmap -x 1234

# Thread count
ps -eLf | grep 1234 | wc -l
```

**Common Issues & Solutions:**

**Issue 1: High CPU from runaway process**
```bash
# Identify process
top  # Note PID of high-CPU process

# Check what it's doing
strace -p <PID> -c

# Kill if necessary
kill <PID>  # Graceful
kill -9 <PID>  # Force (last resort)

# Restart service
systemctl restart service-name
```

**Issue 2: Memory leak**
```bash
# Identify leaking process
ps aux --sort=-%mem | head -5

# Monitor memory growth
watch -n 1 'ps aux | grep <process-name>'

# Restart to free memory (temporary fix)
systemctl restart <service>

# Permanent fix: Fix application memory leak
```

**Issue 3: Disk full**
```bash
# Find what's using space
du -h --max-depth=1 / | sort -rh | head -10

# Common culprits:
# - Log files
find /var/log -type f -size +100M -exec ls -lh {} \;

# - Old docker images/containers
docker system df
docker system prune -a

# - Core dumps
find / -name "core.*" -size +100M

# Clean up
rm /var/log/old-logs/*
journalctl --vacuum-size=500M
docker system prune -a --volumes
```

**Issue 4: Too many connections**
```bash
# Check connection limit
ulimit -n  # File descriptors
sysctl net.core.somaxconn  # Connection backlog

# Increase limits
# /etc/security/limits.conf
* soft nofile 65536
* hard nofile 65536

# /etc/sysctl.conf
net.core.somaxconn = 4096
net.ipv4.tcp_max_syn_backlog = 4096

# Apply
sysctl -p
```

**Issue 5: I/O bottleneck**
```bash
# Check which process
iotop -o

# If database
# - Add indexes
# - Optimize queries
# - Increase buffer pool

# If logs
# - Move logs to faster disk
# - Reduce log level
# - Implement log rotation

# If general I/O
# - Upgrade to SSD
# - Add more disks (RAID)
# - Tune filesystem (noatime)
```

**Monitoring Script:**
```bash
#!/bin/bash
# server-health-check.sh

echo "=== System Health Check ==="
echo "Date: $(date)"
echo ""

echo "=== Uptime & Load ==="
uptime
echo ""

echo "=== CPU Usage ==="
mpstat 1 1 | tail -1
echo ""

echo "=== Memory Usage ==="
free -h
echo ""

echo "=== Disk Usage ==="
df -h | grep -v "tmpfs"
echo ""

echo "=== Top CPU Processes ==="
ps aux --sort=-%cpu | head -6
echo ""

echo "=== Top Memory Processes ==="
ps aux --sort=-%mem | head -6
echo ""

echo "=== Network Connections ==="
netstat -tulanp | grep ESTABLISHED | wc -l
echo " active connections"
echo ""

echo "=== Disk I/O ==="
iostat -x 1 1 | tail -n +4
echo ""

echo "=== Recent Errors ==="
journalctl -p err --since "1 hour ago" --no-pager | tail -10
```

---

### Question 41: Python - Write a script to automate AKS cluster scaling based on time
**Answer:**
```python
#!/usr/bin/env python3
"""
Automated AKS Node Pool Scaling Script
Scale up during business hours, scale down after hours
"""
from azure.identity import DefaultAzureCredential
from azure.mgmt.containerservice import ContainerServiceClient
from datetime import datetime
import logging

# Configuration
SUBSCRIPTION_ID = "your-subscription-id"
RESOURCE_GROUP = "myapp-rg"
CLUSTER_NAME = "myapp-aks"
NODEPOOL_NAME = "nodepool1"

# Scaling configuration
BUSINESS_HOURS_CONFIG = {
    "start_hour": 8,  # 8 AM
    "end_hour": 18,   # 6 PM
    "weekdays_only": True,
    "scale_up_count": 5,
    "scale_down_count": 2
}

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s'
)

def is_business_hours():
    """Check if current time is within business hours"""
    now = datetime.now()
    current_hour = now.hour
    current_weekday = now.weekday()  # 0=Monday, 6=Sunday
    
    # Check if it's a weekday (if configured)
    if BUSINESS_HOURS_CONFIG["weekdays_only"] and current_weekday >= 5:
        return False
    
    # Check if within business hours
    if (BUSINESS_HOURS_CONFIG["start_hour"] <= current_hour < 
        BUSINESS_HOURS_CONFIG["end_hour"]):
        return True
    
    return False

def get_current_node_count(client, rg, cluster, nodepool):
    """Get current node count for the node pool"""
    try:
        agent_pool = client.agent_pools.get(
            resource_group_name=rg,
            resource_name=cluster,
            agent_pool_name=nodepool
        )
        return agent_pool.count
    except Exception as e:
        logging.error(f"Error getting node count: {e}")
        raise

def scale_nodepool(client, rg, cluster, nodepool, count):
    """Scale the AKS node pool"""
    try:
        logging.info(f"Scaling {nodepool} to {count} nodes...")
        
        # Start scaling operation
        poller = client.agent_pools.begin_create_or_update(
            resource_group_name=rg,
            resource_name=cluster,
            agent_pool_name=nodepool,
            parameters={
                "count": count
            }
        )
        
        # Wait for completion
        result = poller.result()
        logging.info(f"Successfully scaled to {count} nodes")
        return result
        
    except Exception as e:
        logging.error(f"Error scaling node pool: {e}")
        raise

def main():
    """Main execution function"""
    try:
        # Authenticate
        credential = DefaultAzureCredential()
        client = ContainerServiceClient(credential, SUBSCRIPTION_ID)
        
        # Get current node count
        current_count = get_current_node_count(
            client, 
            RESOURCE_GROUP, 
            CLUSTER_NAME, 
            NODEPOOL_NAME
        )
        logging.info(f"Current node count: {current_count}")
        
        # Determine desired count
        if is_business_hours():
            desired_count = BUSINESS_HOURS_CONFIG["scale_up_count"]
            logging.info("Business hours detected - scaling UP")
        else:
            desired_count = BUSINESS_HOURS_CONFIG["scale_down_count"]
            logging.info("Off-hours detected - scaling DOWN")
        
        # Scale if needed
        if current_count != desired_count:
            scale_nodepool(
                client,
                RESOURCE_GROUP,
                CLUSTER_NAME,
                NODEPOOL_NAME,
                desired_count
            )
            logging.info(f"Scaled from {current_count} to {desired_count} nodes")
        else:
            logging.info(f"Already at desired count ({desired_count}), no action needed")
            
    except Exception as e:
        logging.error(f"Script failed: {e}")
        raise

if __name__ == "__main__":
    main()

# Deploy as CronJob (runs every hour)
# 0 * * * * /usr/bin/python3 /scripts/aks_autoscale.py >> /var/log/aks_scale.log 2>&1
```

**Deploy as Azure Function (Better approach):**
```python
# function_app.py
import azure.functions as func
import logging
from azure.identity import DefaultAzureCredential
from azure.mgmt.containerservice import ContainerServiceClient
from datetime import datetime

app = func.FunctionApp()

@app.schedule(schedule="0 0 * * * *", arg_name="myTimer", 
              run_on_startup=False)
def aks_autoscaler(myTimer: func.TimerRequest) -> None:
    """Runs every hour to check and scale AKS"""
    if myTimer.past_due:
        logging.info('The timer is past due!')
    
    logging.info('Python timer trigger function executed.')
    
    # Scaling logic here (same as above)
    scale_aks_based_on_time()
```

---

### Question 42: Real Issue - Docker registry pull rate limit errors. How did you handle it?
**Answer:**

**Problem: Docker Hub Rate Limiting**
```bash
# Error in pod
kubectl describe pod myapp-pod

Events:
  Warning  Failed     2m   kubelet  Failed to pull image "nginx:latest": 
           rpc error: code = Unknown desc = Error response from daemon: 
           toomanyrequests: You have reached your pull rate limit. 
           You may increase the limit by authenticating and upgrading.
```

**Docker Hub Limits:**
- Anonymous users: 100 pulls / 6 hours
- Authenticated free users: 200 pulls / 6 hours
- Pro/Team: Higher limits

**Solutions Implemented:**

**Solution 1: Use Azure Container Registry (ACR)**
```bash
# Create ACR
az acr create \
  --resource-group myapp-rg \
  --name myappregistry \
  --sku Standard

# Import commonly used images
az acr import \
  --name myappregistry \
  --source docker.io/library/nginx:latest \
  --image nginx:latest

az acr import \
  --name myappregistry \
  --source docker.io/library/redis:7 \
  --image redis:7

# Attach ACR to AKS
az aks update \
  --name myapp-aks \
  --resource-group myapp-rg \
  --attach-acr myappregistry

# Update deployments
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  template:
    spec:
      containers:
      - name: nginx
        image: myappregistry.azurecr.io/nginx:latest  # Use ACR!
```

**Solution 2: Authenticate to Docker Hub**
```bash
# Create Docker Hub secret
kubectl create secret docker-registry dockerhub-secret \
  --docker-server=docker.io \
  --docker-username=myusername \
  --docker-password=mypassword \
  --docker-email=my@email.com \
  --namespace=default

# Use in pod
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: app
    image: docker.io/myorg/myapp:1.0
  imagePullSecrets:
  - name: dockerhub-secret  # Authenticate pulls
```

**Solution 3: Image Pull Policy**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    spec:
      containers:
      - name: app
        image: nginx:latest
        imagePullPolicy: IfNotPresent  # Don't pull if exists locally
        # Options:
        # - Always: Pull every time (default for :latest)
        # - IfNotPresent: Pull only if not exists
        # - Never: Never pull, use local only
```

**Solution 4: Use Image Caching (Node-level)**
```bash
# Pre-pull images to all nodes
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: image-prepuller
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: image-prepuller
  template:
    metadata:
      labels:
        name: image-prepuller
    spec:
      initContainers:
      - name: prepull-nginx
        image: nginx:latest
        command: ['sh', '-c', 'echo Image pulled']
      - name: prepull-redis
        image: redis:7
        command: ['sh', '-c', 'echo Image pulled']
      containers:
      - name: pause
        image: gcr.io/google_containers/pause:3.1
EOF
```

**Solution 5: Use Alternative Registries**
```yaml
# Use GitHub Container Registry (ghcr.io)
image: ghcr.io/myorg/myapp:1.0

# Use AWS ECR
image: 123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:1.0

# Use Google Container Registry
image: gcr.io/myproject/myapp:1.0

# Use Quay.io
image: quay.io/myorg/myapp:1.0
```

**Solution 6: Implement Image Mirror/Proxy**
```bash
# Deploy Docker Registry as cache
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: docker-registry-proxy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: registry-proxy
  template:
    metadata:
      labels:
        app: registry-proxy
    spec:
      containers:
      - name: registry
        image: registry:2
        env:
        - name: REGISTRY_PROXY_REMOTEURL
          value: https://registry-1.docker.io
        - name: REGISTRY_PROXY_USERNAME
          value: dockerhubuser
        - name: REGISTRY_PROXY_PASSWORD
          valueFrom:
            secretKeyRef:
              name: dockerhub-creds
              key: password
        ports:
        - containerPort: 5000
        volumeMounts:
        - name: registry-storage
          mountPath: /var/lib/registry
      volumes:
      - name: registry-storage
        persistentVolumeClaim:
          claimName: registry-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: docker-registry-proxy
spec:
  selector:
    app: registry-proxy
  ports:
  - port: 5000
    targetPort: 5000
EOF

# Configure containerd to use proxy
# /etc/containerd/config.toml on nodes
[plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
  endpoint = ["http://docker-registry-proxy:5000"]
```

**Long-term Solution:**
```
1. Migrate all public images to ACR/ECR/GCR
2. Use authenticated Docker Hub for remaining
3. Implement image scanning in CI/CD
4. Build and push images to private registry
5. Never use :latest in production (use specific tags)
6. Monitor registry quotas and usage
```

---

### Question 43: GitOps - Explain how you implemented GitOps with ArgoCD and encountered issues
**Answer:**

**Implementation:**

**1. Repository Structure:**
```
gitops-repo/
├── apps/
│   ├── frontend/
│   │   ├── base/
│   │   │   ├── kustomization.yaml
│   │   │   ├── deployment.yaml
│   │   │   └── service.yaml
│   │   └── overlays/
│   │       ├── dev/
│   │       ├── staging/
│   │       └── prod/
│   ├── backend/
│   └── database/
├── infrastructure/
│   ├── ingress/
│   ├── monitoring/
│   └── cert-manager/
└── argocd/
    ├── applications/
    └── projects/
```

**2. ArgoCD Application:**
```yaml
# argocd/applications/frontend-prod.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: frontend-prod
  namespace: argocd
  finalizers:
  - resources-finalizer.argocd.argoproj.io
spec:
  project: production
  source:
    repoURL: https://github.com/myorg/gitops-repo
    targetRevision: main
    path: apps/frontend/overlays/prod
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    syncOptions:
    - CreateNamespace=true
    - PrunePropagationPolicy=foreground
    - PruneLast=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

**Issues Encountered & Solutions:**

**Issue 1: Sync Loops / Infinite Updates**
```
Problem: ArgoCD kept syncing every few seconds

Root Cause: Fields being modified by external controllers (HPA, cluster-autoscaler)

Solution: Ignore specific fields
```

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: frontend-prod
spec:
  ignoreDifferences:
  - group: apps
    kind: Deployment
    jsonPointers:
    - /spec/replicas  # Ignore replicas (managed by HPA)
  - group: apps
    kind: Deployment
    managedFieldsManagers:
    - kube-controller-manager  # Ignore changes by K8s controllers
```

**Issue 2: Secret Management**
```
Problem: Can't commit secrets to Git (security risk)

Solution: Use Sealed Secrets or External Secrets Operator
```

```bash
# Install Sealed Secrets
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.18.0/controller.yaml

# Create sealed secret
kubectl create secret generic db-creds \
  --from-literal=password=SuperSecret123 \
  --dry-run=client -o yaml | \
  kubeseal -o yaml > sealed-secret.yaml

# Commit sealed-secret.yaml to Git (safe!)
```

```yaml
# Or use External Secrets Operator with Azure Key Vault
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: db-credentials
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: azure-keyvault-store
    kind: SecretStore
  target:
    name: db-creds
    creationPolicy: Owner
  data:
  - secretKey: password
    remoteRef:
      key: db-password  # Key Vault secret name
```

**Issue 3: Application Dependencies**
```
Problem: Backend deployed before database ready

Solution: Use ArgoCD Sync Waves
```

```yaml
# Database (wave 0 - deploys first)
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
  annotations:
    argocd.argoproj.io/sync-wave: "0"

---
# Backend (wave 1 - after database)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  annotations:
    argocd.argoproj.io/sync-wave: "1"
    argocd.argoproj.io/sync-options: SkipDryRunOnMissingResource=true

---
# Frontend (wave 2 - after backend)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  annotations:
    argocd.argoproj.io/sync-wave: "2"
```

**Issue 4: Large Monorepo Performance**
```
Problem: ArgoCD slow with large repository

Solution: Split into multiple apps or use Application Sets
```

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: microservices
  namespace: argocd
spec:
  generators:
  - git:
      repoURL: https://github.com/myorg/gitops-repo
      revision: HEAD
      directories:
      - path: apps/*
  template:
    metadata:
      name: '{{path.basename}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/myorg/gitops-repo
        targetRevision: HEAD
        path: '{{path}}'
      destination:
        server: https://kubernetes.default.svc
        namespace: '{{path.basename}}'
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

**Issue 5: Rollback Complexity**
```
Problem: How to rollback when Git is source of truth?

Solution: Revert Git commit or use ArgoCD history
```

```bash
# Method 1: Git revert
git log --oneline  # Find commit to revert
git revert abc1234
git push origin main
# ArgoCD auto-syncs to reverted state

# Method 2: ArgoCD rollback
argocd app history frontend-prod
# ID  DATE                  REVISION
# 5   2024-01-08 10:30:00  abc1234 (current)
# 4   2024-01-08 09:00:00  def5678
# 3   2024-01-08 08:00:00  ghi9012

argocd app rollback frontend-prod 4
# Note: This doesn't update Git! Manual reconciliation needed
```

**Issue 6: Environment Promotion**
```
Problem: Safely promote from dev → staging → prod

Solution: Use separate branches or directories with PR approvals
```

```yaml
# Approach 1: Branch per environment
# dev branch → dev cluster
# staging branch → staging cluster  
# main branch → prod cluster

# Approach 2: Directory overlays with Kustomize
apps/
  frontend/
    base/
    overlays/
      dev/     # argocd app pointing here for dev
      staging/ # PR required to promote from dev
      prod/    # PR + approval required
```

**CI/CD Integration:**
```yaml
# Azure Pipeline - Update image tag in Git
trigger:
- main

stages:
- stage: Build
  jobs:
  - job: BuildAndPush
    steps:
    - task: Docker@2
      inputs:
        command: buildAndPush
        repository: myapp
        tags: $(Build.BuildId)

- stage: UpdateGitOps
  jobs:
  - job: UpdateManifest
    steps:
    - checkout: gitops-repo
    - script: |
        # Update image tag in kustomization
        cd apps/frontend/overlays/dev
        kustomize edit set image myapp=myregistry.azurecr.io/myapp:$(Build.BuildId)
        
        git config user.email "pipeline@azuredevops.com"
        git config user.name "Azure Pipeline"
        git add .
        git commit -m "Update frontend image to $(Build.BuildId)"
        git push origin main
      displayName: 'Update GitOps repo'
    
# ArgoCD detects change and auto-deploys!
```

**Lessons Learned:**
- Use ignoreDifferences for fields managed by controllers
- Implement proper secret management (Sealed Secrets)
- Use sync waves for deployment ordering
- Split large repos for performance
- Implement branch protection for production
- Monitor ArgoCD sync status with alerts
- Document rollback procedures clearly
- Use ApplicationSets for repetitive patterns

---

### Question 44: Terraform - Explain how you handle sensitive outputs and state file security
**Answer:**

**Problem: Sensitive Data Exposure**

Terraform state files contain:
- Resource IDs
- Connection strings
- Passwords
- API keys
- Private IPs
- Certificate data

**Security Solutions:**

**1. Remote Backend with Encryption:**
```hcl
# backend.tf
terraform {
  backend "azurerm" {
    resource_group_name  = "terraform-state-rg"
    storage_account_name = "tfstatesecure"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
    
    # Enable encryption at rest
    use_azuread_auth = true  # Use Azure AD instead of access key
  }
}

# Azure Storage Account configuration
resource "azurerm_storage_account" "tfstate" {
  name                     = "tfstatesecure"
  resource_group_name      = azurerm_resource_group.state.name
  location                 = "East US"
  account_tier             = "Standard"
  account_replication_type = "GRS"
  
  # Enable encryption
  encryption {
    services {
      blob {
        enabled = true
      }
    }
  }
  
  # Enable blob versioning (for recovery)
  blob_properties {
    versioning_enabled = true
  }
  
  # Restrict network access
  network_rules {
    default_action = "Deny"
    ip_rules       = ["1.2.3.4"]  # Your IPs
    virtual_network_subnet_ids = [
      azurerm_subnet.devops.id
    ]
  }
  
  # Enable advanced threat protection
  advanced_threat_protection_enabled = true
}

# Lock state blob
resource "azurerm_storage_container" "tfstate" {
  name                  = "tfstate"
  storage_account_name  = azurerm_storage_account.tfstate.name
  container_access_type = "private"  # No public access!
}

# RBAC for state access
resource "azurerm_role_assignment" "tfstate_contributor" {
  scope                = azurerm_storage_account.tfstate.id
  role_definition_name = "Storage Blob Data Contributor"
  principal_id         = data.azuread_service_principal.terraform.object_id
}
```

**2. Mark Sensitive Outputs:**
```hcl
# outputs.tf
output "database_password" {
  value     = random_password.db_password.result
  sensitive = true  # Won't show in logs or plan output
}

output "connection_string" {
  value     = "Server=${azurerm_sql_server.main.fully_qualified_domain_name};..."
  sensitive = true
}

# When running:
# terraform output database_password  # Shows <sensitive>
# terraform output -json database_password  # Shows actual value (use carefully!)
```

**3. Use Azure Key Vault for Secrets:**
```hcl
# Don't store secrets in variables or tfvars!

# Bad:
# variable "db_password" {
#   default = "SuperSecret123"  # Never do this!
# }

# Good: Fetch from Key Vault
data "azurerm_key_vault" "main" {
  name                = "myapp-keyvault"
  resource_group_name = "myapp-rg"
}

data "azurerm_key_vault_secret" "db_password" {
  name         = "database-password"
  key_vault_id = data.azurerm_key_vault.main.id
}

resource "azurerm_sql_server" "main" {
  name                = "myapp-sqlserver"
  administrator_login = "sqladmin"
  administrator_login_password = data.azurerm_key_vault_secret.db_password.value
  # Password never in state or logs
}
```

**4. Restrict State File Access:**
```bash
# Azure Storage - Disable public access
az storage account update \
  --name tfstatesecure \
  --resource-group terraform-state-rg \
  --allow-blob-public-access false

# Enable soft delete (recovery)
az storage blob service-properties delete-policy update \
  --account-name tfstatesecure \
  --enable true \
  --days-retained 30

# Enable versioning
az storage account blob-service-properties update \
  --account-name tfstatesecure \
  --resource-group terraform-state-rg \
  --enable-versioning true

# Lock the storage account
az lock create \
  --name "state-lock" \
  --lock-type CanNotDelete \
  --resource-group terraform-state-rg \
  --resource tfstatesecure \
  --resource-type Microsoft.Storage/storageAccounts
```

**5. Encrypt Sensitive Variables:**
```hcl
# Use encrypted variables in CI/CD

# Azure Pipelines
variables:
- group: terraform-secrets  # Linked to Key Vault
- name: ARM_CLIENT_SECRET
  value: $(keyvault-client-secret)  # Fetched from KV

# Never commit .tfvars with secrets!
# .gitignore
*.tfvars
*.tfstate
*.tfstate.backup
.terraform/
```

**6. Audit State Access:**
```bash
# Enable Azure Storage diagnostics
az monitor diagnostic-settings create \
  --name tfstate-audit \
  --resource /subscriptions/{sub-id}/resourceGroups/terraform-state-rg/providers/Microsoft.Storage/storageAccounts/tfstatesecure \
  --logs '[{"category":"StorageRead","enabled":true},{"category":"StorageWrite","enabled":true}]' \
  --workspace /subscriptions/{sub-id}/resourceGroups/monitoring-rg/providers/Microsoft.OperationalInsights/workspaces/logs

# Query logs
# Log Analytics query:
StorageBlobLogs
| where AccountName == "tfstatesecure"
| where OperationName in ("GetBlob", "PutBlob")
| project TimeGenerated, CallerIpAddress, OperationName, Uri
| order by TimeGenerated desc
```

**7. Rotate State Access Keys:**
```bash
# Rotate storage account keys regularly
az storage account keys renew \
  --account-name tfstatesecure \
  --resource-group terraform-state-rg \
  --key primary

# Update pipelines/local config with new key
# Or better: Use Azure AD (no keys to rotate)
```

**8. State File Encryption at Rest (Additional Layer):**
```hcl
# Use customer-managed keys
resource "azurerm_storage_account" "tfstate" {
  name                     = "tfstatesecure"
  resource_group_name      = azurerm_resource_group.state.name
  location                 = "East US"
  account_tier             = "Standard"
  account_replication_type = "GRS"
  
  identity {
    type = "SystemAssigned"
  }
  
  customer_managed_key {
    key_vault_key_id          = azurerm_key_vault_key.storage.id
    user_assigned_identity_id = azurerm_user_assigned_identity.storage.id
  }
}

resource "azurerm_key_vault_key" "storage" {
  name         = "tfstate-encryption-key"
  key_vault_id = azurerm_key_vault.main.id
  key_type     = "RSA"
  key_size     = 2048
  
  key_opts = [
    "decrypt",
    "encrypt",
    "unwrapKey",
    "wrapKey"
  ]
}
```

**9. Pipeline Security:**
```yaml
# Azure Pipeline with secure state access
trigger:
- main

variables:
- group: terraform-production  # Contains state access creds

pool:
  vmImage: 'ubuntu-latest'

steps:
- task: AzureCLI@2
  inputs:
    azureSubscription: 'terraform-sp'
    scriptType: 'bash'
    scriptLocation: 'inlineScript'
    inlineScript: |
      # Use managed identity or service principal (no secrets in logs)
      export ARM_USE_MSI=true
      export ARM_SUBSCRIPTION_ID=$(az account show --query id -o tsv)
      export ARM_TENANT_ID=$(az account show --query tenantId -o tsv)
      
      # Initialize with remote backend
      terraform init \
        -backend-config="storage_account_name=tfstatesecure" \
        -backend-config="container_name=tfstate" \
        -backend-config="key=prod.terraform.tfstate"
      
      # Plan (output redacted)
      terraform plan -out=tfplan
      
      # Apply (sensitive values hidden)
      terraform apply -auto-approve tfplan
      
      # Don't output sensitive values in logs!
  displayName: 'Terraform Apply'
  env:
    # Sensitive variables masked in logs
    TF_VAR_db_password: $(db-password)
    TF_VAR_api_key: $(api-key)
```

**10. Regular Security Audits:**
```bash
# Scan state for secrets (locally, never commit)
grep -i "password\|secret\|key" terraform.tfstate

# Use tfsec for security scanning
tfsec .

# Use Checkov
checkov -d . --framework terraform

# Scan for exposed secrets
trufflehog filesystem --directory .
```

**Best Practices Checklist:**
- ✅ Use remote backend with encryption
- ✅ Mark all sensitive outputs as sensitive = true
- ✅ Never commit state files to Git
- ✅ Use Azure Key Vault for secrets
- ✅ Implement RBAC on state storage
- ✅ Enable audit logging
- ✅ Use Azure AD authentication (not access keys)
- ✅ Enable soft delete and versioning
- ✅ Restrict network access to state storage
- ✅ Rotate credentials regularly
- ✅ Scan for exposed secrets in CI/CD
- ✅ Use customer-managed encryption keys
- ✅ Implement state file locking
- ✅ Regular security audits

---

### Question 45: Maven - You have a multi-module Maven project. How do you optimize build times?
**Answer:**

**Project Structure:**
```
myapp/
├── pom.xml (parent)
├── common-lib/
│   └── pom.xml
├── backend-api/
│   └── pom.xml
├── frontend-service/
│   └── pom.xml
├── notification-service/
│   └── pom.xml
└── integration-tests/
    └── pom.xml
```

**Optimization Strategies:**

**1. Parallel Builds:**
```bash
# Build modules in parallel
mvn clean install -T 4  # Use 4 threads

# Or use all available cores
mvn clean install -T 1C  # 1 thread per CPU core
mvn clean install -T 2C  # 2 threads per CPU core

# Parallel builds only work if modules are independent!
```

**2. Skip Unnecessary Phases:**
```bash
# Skip tests (use in CI after running tests separately)
mvn clean install -DskipTests

# Skip test compilation too
mvn clean install -Dmaven.test.skip=true

# Skip JavaDoc generation
mvn clean install -Dmaven.javadoc.skip=true

# Combined
mvn clean install -DskipTests -Dmaven.javadoc.skip=true -T 1C
```

**3. Build Only Changed Modules:**
```bash
# Build from specific module and its dependents
mvn install -pl backend-api -am
# -pl: project list (which modules)
# -am: also make (include dependencies)

# Build specific modules only (not dependencies)
mvn install -pl backend-api,frontend-service

# Build modules affected by changes
mvn install -pl :backend-api -amd
# -amd: also make dependents
```

**4. Incremental Builds:**
```bash
# Only recompile changed classes
mvn compile -Dcompiler.incremental=true

# Or in pom.xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-compiler-plugin</artifactId>
  <version>3.11.0</version>
  <configuration>
    <useIncrementalCompilation>true</useIncrementalCompilation>
  </configuration>
</plugin>
```

**5. Dependency Caching:**
```xml
<!-- Parent pom.xml -->
<project>
  <properties>
    <maven.repo.local>${user.home}/.m2/repository</maven.repo.local>
  </properties>
  
  <dependencyManagement>
    <dependencies>
      <!-- Define versions once for all modules -->
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-dependencies</artifactId>
        <version>3.2.0</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>
</project>
```

**CI/CD Optimization:**
```yaml
# Azure Pipelines with Maven caching
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

variables:
  MAVEN_CACHE_FOLDER: $(Pipeline.Workspace)/.m2/repository
  MAVEN_OPTS: '-Dmaven.repo.local=$(MAVEN_CACHE_FOLDER) -Xmx3072m'

steps:
- task: Cache@2
  inputs:
    key: 'maven | "$(Agent.OS)" | **/pom.xml'
    restoreKeys: |
      maven | "$(Agent.OS)"
      maven
    path: $(MAVEN_CACHE_FOLDER)
  displayName: Cache Maven local repo

- task: Maven@3
  inputs:
    mavenPomFile: 'pom.xml'
    goals: 'clean install'
    options: '-T 1C -DskipTests'
    publishJUnitResults: false
    javaHomeOption: 'JDKVersion'
    jdkVersionOption: '17'
    mavenVersionOption: 'Default'
    mavenOptions: '$(MAVEN_OPTS)'
  displayName: 'Maven Build (Parallel)'

- task: Maven@3
  inputs:
    mavenPomFile: 'pom.xml'
    goals: 'test'
    options: '-T 1C'
  displayName: 'Maven Test (Parallel)'
```

**6. Optimize Dependencies:**
```xml
<!-- Remove unused dependencies -->
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-dependency-plugin</artifactId>
  <executions>
    <execution>
      <id>analyze</id>
      <goals>
        <goal>analyze-only</goal>
      </goals>
      <configuration>
        <failOnWarning>true</failOnWarning>
        <ignoreNonCompile>true</ignoreNonCompile>
      </configuration>
    </execution>
  </executions>
</plugin>

<!-- Run: mvn dependency:analyze -->
```

**7. Use Maven Daemon (mvnd):**
```bash
# Install Maven Daemon (faster than regular Maven)
sdk install mvnd

# Use mvnd instead of mvn
mvnd clean install -T 1C

# Benefits:
# - JVM reuse (no startup cost)
# - Smart parallel builds
# - Faster incremental builds
```

**8. Profile-Based Builds:**
```xml
<!-- pom.xml -->
<profiles>
  <!-- Fast build for development -->
  <profile>
    <id>fast</id>
    <properties>
      <skipTests>true</skipTests>
      <maven.javadoc.skip>true</maven.javadoc.skip>
      <checkstyle.skip>true</checkstyle.skip>
    </properties>
  </profile>
  
  <!-- Full build for CI/CD -->
  <profile>
    <id>ci</id>
    <properties>
      <skipTests>false</skipTests>
      <maven.javadoc.skip>false</maven.javadoc.skip>
    </properties>
    <build>
      <plugins>
        <!-- SonarQube, Jacoco, etc. -->
      </plugins>
    </build>
  </profile>
</profiles>

<!-- Usage: -->
<!-- mvn clean install -Pfast         (dev) -->
<!-- mvn clean install -Pci -T 1C     (CI/CD) -->
```

**9. Smart Reactor Ordering:**
```xml
<!-- Parent pom.xml -->
<project>
  <modules>
    <!-- Order matters! Build dependencies first -->
    <module>common-lib</module>          <!-- No dependencies -->
    <module>backend-api</module>         <!-- Depends on common-lib -->
    <module>frontend-service</module>    <!-- Depends on common-lib -->
    <module>notification-service</module><!-- Depends on common-lib -->
    <module>integration-tests</module>   <!-- Depends on all above -->
  </modules>
</project>

<!-- Maven automatically determines build order based on dependencies -->
```

**10. Optimize Test Execution:**
```xml
<!-- Parallel test execution -->
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-surefire-plugin</artifactId>
  <version>3.0.0</version>
  <configuration>
    <!-- Run tests in parallel -->
    <parallel>classes</parallel>
    <threadCount>4</threadCount>
    <perCoreThreadCount>true</perCoreThreadCount>
    
    <!-- Fork JVM for tests (isolate) -->
    <forkCount>2</forkCount>
    <reuseForks>true</reuseForks>
    
    <!-- Exclude slow tests in fast builds -->
    <excludedGroups>integration,slow</excludedGroups>
  </configuration>
</plugin>
```

**Results:**
```
Before Optimization:
- Full build: 15 minutes
- Parallel: N/A
- Cache: No
- Test time: 8 minutes

After Optimization:
- Full build: 4 minutes (73% faster)
- Parallel: -T 2C (8 cores)
- Cache: Maven + Docker layers
- Test time: 2 minutes (parallel execution)
- Incremental builds: 30 seconds (changed module only)
```

---

### Question 46: Real Issue - Kubernetes DNS resolution failures. How did you troubleshoot?
**Answer:**

**Problem Symptoms:**
```bash
# Pods can't resolve service names
kubectl logs myapp-pod
Error: failed to connect to database-service: no such host

# But direct IP works
curl http://10.0.0.5:5432  # Works
curl http://database-service:5432  # Fails
```

**Troubleshooting Steps:**

**1. Check DNS Service Status:**
```bash
# Check CoreDNS pods
kubectl get pods -n kube-system -l k8s-app=kube-dns
NAME                       READY   STATUS    RESTARTS   AGE
coredns-5d78c9869d-abc12   1/1     Running   0          10d
coredns-5d78c9869d-def34   1/1     Running   0          10d

# Check CoreDNS logs
kubectl logs -n kube-system -l k8s-app=kube-dns --tail=50

# Common errors:
# - "plugin/forward: no servers"
# - "read udp timeout"
# - "no nameservers defined"
```

**2. Test DNS from Pod:**
```bash
# Create debug pod
kubectl run -it --rm debug --image=busybox --restart=Never -- sh

# Inside pod:
# Test service DNS
nslookup database-service
nslookup database-service.default.svc.cluster.local

# Test external DNS
nslookup google.com

# Check /etc/resolv.conf
cat /etc/resolv.conf
# Should show:
# nameserver 10.0.0.10  (ClusterIP of kube-dns service)
# search default.svc.cluster.local svc.cluster.local cluster.local
# options ndots:5
```

**3. Verify DNS Service:**
```bash
# Get DNS service ClusterIP
kubectl get svc -n kube-system kube-dns
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)         AGE
kube-dns   ClusterIP   10.0.0.10    <none>        53/UDP,53/TCP   30d

# Test DNS service directly
kubectl run -it --rm debug --image=tutum/dnsutils --restart=Never -- sh
dig @10.0.0.10 database-service.default.svc.cluster.local

# If this fails, DNS service issue
# If this works, pod's resolv.conf issue
```

**4. Check CoreDNS Configuration:**
```bash
# Get CoreDNS ConfigMap
kubectl get configmap -n kube-system coredns -o yaml

# Should contain:
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

**Root Causes Found & Solutions:**

**Issue 1: CoreDNS Pods Not Ready**
```bash
# Scale up CoreDNS if needed
kubectl scale deployment coredns -n kube-system --replicas=3

# Or check resource limits
kubectl describe pod -n kube-system -l k8s-app=kube-dns

# If OOMKilled, increase limits
kubectl edit deployment coredns -n kube-system
# Increase:
resources:
  limits:
    memory: 512Mi  # Increase from 170Mi
  requests:
    memory: 256Mi
```

**Issue 2: Network Policy Blocking DNS**
```bash
# Check if network policies exist
kubectl get networkpolicies --all-namespaces

# Allow DNS traffic
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
  namespace: default
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
    - protocol: TCP
      port: 53
EOF
```

**Issue 3: Node-Level DNS Issues**
```bash
# Check node DNS resolution
kubectl debug node/<node-name> -it --image=busybox

# On node:
cat /etc/resolv.conf
nslookup google.com

# If node DNS broken, fix node networking
# Azure AKS: Check virtual network DNS settings
az network vnet show \
  --resource-group myapp-rg \
  --name aks-vnet \
  --query dhcpOptions
```

**Issue 4: CoreDNS Forward Loop**
```bash
# Check CoreDNS logs
kubectl logs -n kube-system -l k8s-app=kube-dns | grep loop

# If loop detected:
# Edit CoreDNS ConfigMap
kubectl edit configmap coredns -n kube-system

# Remove or fix forward directive
# From:
forward . /etc/resolv.conf

# To:
forward . 8.8.8.8 8.8.4.4  # Use external DNS directly
```

**Issue 5: Pod DNS Policy Incorrect**
```yaml
# Check pod DNS policy
kubectl get pod myapp-pod -o yaml | grep dnsPolicy

# Options:
# - ClusterFirst: Use cluster DNS (default, correct)
# - Default: Use node's /etc/resolv.conf
# - None: Manual DNS config

# Fix if wrong:
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  dnsPolicy: ClusterFirst  # Correct setting
  containers:
  - name: app
    image: myapp:1.0
```

**Issue 6: High DNS Query Volume (DDoS)**
```bash
# Check CoreDNS metrics
kubectl port-forward -n kube-system svc/coredns 9153:9153
curl http://localhost:9153/metrics | grep coredns_dns_request_count_total

# High request rate can overwhelm CoreDNS

# Solution: Increase replicas and resources
kubectl scale deployment coredns -n kube-system --replicas=5

# Or use node-local DNS cache
kubectl apply -f https://k8s.io/examples/admin/dns/nodelocaldns.yaml
```

**Issue 7: ndots Configuration**
```yaml
# Problem: Too many DNS queries due to ndots
# Each lookup tries:
# myservice
# myservice.default
# myservice.default.svc
# myservice.default.svc.cluster
# myservice.default.svc.cluster.local

# Solution: Use FQDN or reduce ndots
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  dnsConfig:
    options:
    - name: ndots
      value: "1"  # Reduce from default 5
  containers:
  - name: app
    image: myapp:1.0
    env:
    - name: DATABASE_HOST
      value: "database-service.default.svc.cluster.local"  # Use FQDN
```

**Permanent Fix - Node-Local DNS Cache:**
```yaml
# Deploy NodeLocal DNSCache for better performance
apiVersion: v1
kind: ConfigMap
metadata:
  name: nodelocaldns
  namespace: kube-system
data:
  Corefile: |
    cluster.local:53 {
        errors
        cache {
            success 9984 30
            denial 9984 5
        }
        reload
        loop
        bind 169.254.20.10
        forward . 10.0.0.10 {
            force_tcp
        }
        prometheus :9253
    }

---
# Deploy as DaemonSet (runs on every node)
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nodelocaldns
  namespace: kube-system
spec:
  selector:
    matchLabels:
      k8s-app: nodelocaldns
  template:
    metadata:
      labels:
        k8s-app: nodelocaldns
    spec:
      hostNetwork: true
      containers:
      - name: node-cache
        image: k8s.gcr.io/dns/k8s-dns-node-cache:1.22.13
        resources:
          requests:
            cpu: 25m
            memory: 5Mi
        volumeMounts:
        - name: config-volume
          mountPath: /etc/coredns
      volumes:
      - name: config-volume
        configMap:
          name: nodelocaldns
          items:
          - key: Corefile
            path: Corefile.base

# Benefits:
# - Reduced latency (local cache)
# - Reduced load on CoreDNS
# - Better reliability
```

**Monitoring & Alerts:**
```yaml
# Prometheus alert for DNS failures
alert: HighDNSErrorRate
expr: |
  rate(coredns_dns_request_count_total{rcode="SERVFAIL"}[5m]) > 10
for: 5m
annotations:
  summary: "High DNS error rate detected"
  description: "DNS SERVFAIL rate is {{ $value }} per second"

# Alert for CoreDNS pods down
alert: CoreDNSDown
expr: |
  kube_deployment_status_replicas_available{deployment="coredns"} < 2
for: 5m
annotations:
  summary: "CoreDNS has less than 2 replicas available"
```

**Lessons Learned:**
- Always test DNS resolution when debugging connectivity
- Monitor CoreDNS resource usage and scale appropriately
- Use node-local DNS cache for large clusters
- Be careful with network policies (don't block DNS)
- Use FQDN in applications to reduce DNS queries
- Set up alerts for DNS failures
- Keep CoreDNS logs accessible for troubleshooting

---

### Question 47: Azure Pipelines - Multi-stage pipeline with manual approval and automatic rollback. How do you implement it?
**Answer:**

```yaml
# azure-pipelines-advanced.yml
trigger:
- main

variables:
  azureSubscription: 'azure-prod-connection'
  kubernetesCluster: 'myapp-aks'
  resourceGroup: 'myapp-rg'
  imageRepository: 'myapp'
  containerRegistry: 'myregistry.azurecr.io'
  imageTag: '$(Build.BuildId)'

stages:
# Stage 1: Build and Test
- stage: Build
  displayName: 'Build and Test'
  jobs:
  - job: BuildJob
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - task: Docker@2
      displayName: 'Build Docker Image'
      inputs:
        command: build
        repository: $(imageRepository)
        dockerfile: Dockerfile
        tags: |
          $(imageTag)
          latest
    
    - script: |
        docker run --rm $(imageRepository):$(imageTag) npm test
      displayName: 'Run Unit Tests'
    
    - task: Docker@2
      displayName: 'Push to ACR'
      inputs:
        command: push
        repository: $(imageRepository)
        containerRegistry: $(containerRegistry)
        tags: |
          $(imageTag)
          latest
    
    - script: |
        # Save image tag for later stages
        echo "$(imageTag)" > $(Build.ArtifactStagingDirectory)/image-tag.txt
      displayName: 'Save Image Tag'
    
    - publish: $(Build.ArtifactStagingDirectory)
      artifact: build-artifacts

# Stage 2: Deploy to Dev (Automatic)
- stage: DeployDev
  displayName: 'Deploy to Development'
  dependsOn: Build
  jobs:
  - deployment: DeployDevJob
    environment: 'development'
    pool:
      vmImage: 'ubuntu-latest'
    strategy:
      runOnce:
        deploy:
          steps:
          - download: current
            artifact: build-artifacts
          
          - task: Kubernetes@1
            displayName: 'Deploy to Dev'
            inputs:
              connectionType: 'Azure Resource Manager'
              azureSubscriptionEndpoint: $(azureSubscription)
              azureResourceGroup: $(resourceGroup)
              kubernetesCluster: $(kubernetesCluster)
              namespace: 'development'
              command: 'apply'
              arguments: '-f k8s/deployment.yaml'
              secretType: 'dockerRegistry'
              containerRegistryType: 'Azure Container Registry'
          
          - script: |
              # Health check
              kubectl wait --for=condition=available --timeout=300s \
                deployment/myapp -n development
              
              # Get service URL
              SERVICE_IP=$(kubectl get svc myapp -n development -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
              echo "Development URL: http://$SERVICE_IP"
              
              # Smoke test
              for i in {1..10}; do
                if curl -f http://$SERVICE_IP/health; then
                  echo "Health check passed"
                  exit 0
                fi
                sleep 5
              done
              echo "Health check failed"
              exit 1
            displayName: 'Verify Deployment'

# Stage 3: Deploy to Staging (Automatic)
- stage: DeployStaging
  displayName: 'Deploy to Staging'
  dependsOn: DeployDev
  condition: succeeded()
  jobs:
  - deployment: DeployStagingJob
    environment: 'staging'
    pool:
      vmImage: 'ubuntu-latest'
    strategy:
      runOnce:
        deploy:
          steps:
          - download: current
            artifact: build-artifacts
          
          - task: Kubernetes@1
            displayName: 'Deploy to Staging'
            inputs:
              connectionType: 'Azure Resource Manager'
              azureSubscriptionEndpoint: $(azureSubscription)
              azureResourceGroup: $(resourceGroup)
              kubernetesCluster: $(kubernetesCluster)
              namespace: 'staging'
              command: 'apply'
              arguments: '-f k8s/deployment.yaml'
          
          - script: |
              # Save current version for rollback
              kubectl get deployment myapp -n staging -o yaml > $(Pipeline.Workspace)/staging-backup.yaml
              
              # Wait for rollout
              kubectl rollout status deployment/myapp -n staging --timeout=5m
              
              # Run integration tests
              SERVICE_IP=$(kubectl get svc myapp -n staging -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
              
              # Run test suite
              docker run --rm -e BASE_URL=http://$SERVICE_IP \
                myregistry.azurecr.io/integration-tests:latest
            displayName: 'Deploy and Test'
          
          - publish: $(Pipeline.Workspace)/staging-backup.yaml
            artifact: staging-backup

# Stage 4: Manual Approval for Production
- stage: ApprovalGate
  displayName: 'Production Approval'
  dependsOn: DeployStaging
  condition: succeeded()
  jobs:
  - job: WaitForApproval
    displayName: 'Wait for Production Approval'
    pool: server
    timeoutInMinutes: 1440  # 24 hours
    steps:
    - task: ManualValidation@0
      timeoutInMinutes: 1440
      inputs:
        notifyUsers: |
          ops-team@example.com
          tech-lead@example.com
        instructions: |
          ## Production Deployment Approval Required
          
          **Build:** $(Build.BuildId)
          **Image:** $(containerRegistry)/$(imageRepository):$(imageTag)
          **Branch:** $(Build.SourceBranchName)
          **Commit:** $(Build.SourceVersion)
          
          ### Pre-Deployment Checklist:
          - [ ] Staging tests passed
          - [ ] Performance metrics reviewed
          - [ ] Security scan completed
          - [ ] Change ticket approved
          - [ ] Rollback plan confirmed
          
          ### Staging Environment:
          Review application at: http://staging.example.com
          
          **Click Resume to proceed with production deployment.**
          **Click Reject to cancel deployment.**

# Stage 5: Deploy to Production (Blue-Green)
- stage: DeployProduction
  displayName: 'Deploy to Production'
  dependsOn: ApprovalGate
  condition: succeeded()
  jobs:
  - deployment: DeployProdJob
    environment: 'production'
    pool:
      vmImage: 'ubuntu-latest'
    strategy:
      runOnce:
        preDeploy:
          steps:
          - script: |
              echo "Starting production deployment at $(date)"
              echo "Image: $(containerRegistry)/$(imageRepository):$(imageTag)"
            displayName: 'Pre-Deploy Log'
          
          - task: Kubernetes@1
            displayName: 'Backup Current Production'
            inputs:
              connectionType: 'Azure Resource Manager'
              azureSubscriptionEndpoint: $(azureSubscription)
              azureResourceGroup: $(resourceGroup)
              kubernetesCluster: $(kubernetesCluster)
              namespace: 'production'
              command: 'get'
              arguments: 'deployment myapp -o yaml > $(Pipeline.Workspace)/prod-backup.yaml'
          
          - publish: $(Pipeline.Workspace)/prod-backup.yaml
            artifact: prod-backup
        
        deploy:
          steps:
          - download: current
            artifact: build-artifacts
          
          - script: |
              # Save current metrics for comparison
              SERVICE_IP=$(kubectl get svc myapp -n production -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
              
              # Baseline metrics
              curl -s http://$SERVICE_IP/metrics > $(Pipeline.Workspace)/baseline-metrics.txt
              
              echo "Current version info:"
              kubectl get deployment myapp -n production -o jsonpath='{.spec.template.spec.containers[0].image}'
            displayName: 'Capture Baseline Metrics'
          
          - task: Kubernetes@1
            displayName: 'Deploy to Production'
            inputs:
              connectionType: 'Azure Resource Manager'
              azureSubscriptionEndpoint: $(azureSubscription)
              azureResourceGroup: $(resourceGroup)
              kubernetesCluster: $(kubernetesCluster)
              namespace: 'production'
              command: 'apply'
              arguments: '-f k8s/deployment.yaml'
          
          - script: |
              # Wait for rollout
              echo "Waiting for deployment to complete..."
              kubectl rollout status deployment/myapp -n production --timeout=10m
              
              if [ $? -ne 0 ]; then
                echo "Deployment failed to roll out"
                exit 1
              fi
              
              echo "Deployment successful"
            displayName: 'Wait for Rollout'
        
        routeTraffic:
          steps:
          - script: |
              echo "Starting traffic routing to new version..."
              SERVICE_IP=$(kubectl get svc myapp -n production -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
              
              # Gradual traffic routing could be implemented here with service mesh
              # For now, all-at-once routing
              
              echo "All traffic now routing to new version"
            displayName: 'Route Traffic'
        
        postRouteTraffic:
          steps:
          - script: |
              SERVICE_IP=$(kubectl get svc myapp -n production -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
              
              echo "Running post-deployment validation..."
              
              # Health check
              for i in {1..20}; do
                if curl -f http://$SERVICE_IP/health; then
                  echo "Health check $i/20 passed"
                else
                  echo "Health check $i/20 FAILED"
                  exit 1
                fi
                sleep 3
              done
              
              # Check error rate
              ERROR_RATE=$(curl -s http://$SERVICE_IP/metrics | grep error_rate | awk '{print $2}')
              echo "Current error rate: $ERROR_RATE"
              
              if (( $(echo "$ERROR_RATE > 0.01" | bc -l) )); then
                echo "ERROR: Error rate too high: $ERROR_RATE"
                exit 1
              fi
              
              # Check response time
              AVG_RESPONSE=$(curl -s http://$SERVICE_IP/metrics | grep avg_response_time | awk '{print $2}')
              echo "Average response time: $AVG_RESPONSE ms"
              
              if (( $(echo "$AVG_RESPONSE > 1000" | bc -l) )); then
                echo "WARNING: Response time degraded: $AVG_RESPONSE ms"
                exit 1
              fi
              
              echo "All validation checks passed!"
            displayName: 'Post-Deployment Validation'
            timeoutInMinutes: 10
        
        on:
          failure:
            steps:
            - script: |
                echo "##[error]Deployment failed! Initiating automatic rollback..."
              displayName: 'Failure Detected'
            
            - download: current
              artifact: prod-backup
            
            - task: Kubernetes@1
              displayName: 'Rollback to Previous Version'
              inputs:
                connectionType: 'Azure Resource Manager'
                azureSubscriptionEndpoint: $(azureSubscription)
                azureResourceGroup: $(resourceGroup)
                kubernetesCluster: $(kubernetesCluster)
                namespace: 'production'
                command: 'apply'
                arguments: '-f $(Pipeline.Workspace)/prod-backup.yaml'
            
            - script: |
                echo "Waiting for rollback to complete..."
                kubectl rollout status deployment/myapp -n production --timeout=5m
                
                SERVICE_IP=$(kubectl get svc myapp -n production -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
                
                # Verify rollback
                for i in {1..10}; do
                  if curl -f http://$SERVICE_IP/health; then
                    echo "Rollback successful - application healthy"
                    break
                  fi
                  sleep 5
                done
              displayName: 'Verify Rollback'
            
            - task: SendEmail@1
              inputs:
                To: 'ops-team@example.com'
                Subject: 'Production Deployment Failed - Rollback Executed'
                Body: |
                  Production deployment FAILED and automatic rollback was executed.
                  
                  Build: $(Build.BuildId)
                  Failed Image: $(containerRegistry)/$(imageRepository):$(imageTag)
                  
                  Please investigate logs: $(System.TeamFoundationCollectionUri)$(System.TeamProject)/_build/results?buildId=$(Build.BuildId)
              displayName: 'Send Failure Notification'

# Stage 6: Post-Deployment Monitoring
- stage: PostDeploymentMonitoring
  displayName: 'Post-Deployment Monitoring'
  dependsOn: DeployProduction
  condition: succeeded()
  jobs:
  - job: MonitoringJob
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - script: |
        echo "Monitoring production for 10 minutes..."
        
        SERVICE_IP=$(kubectl get svc myapp -n production -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
        
        for i in {1..120}; do  # 10 minutes
          HEALTH=$(curl -s -o /dev/null -w "%{http_code}" http://$SERVICE_IP/health)
          
          if [ "$HEALTH" != "200" ]; then
            echo "##[error]Health check failed at $(date)"
            echo "##vso[task.logissue type=error]Production health check failed"
            exit 1
          fi
          
          if [ $((i % 12)) -eq 0 ]; then  # Every minute
            echo "Health check OK at $(date)"
          fi
          
          sleep 5
        done
        
        echo "Monitoring complete - No issues detected"
      displayName: 'Monitor Production Health'
      timeoutInMinutes: 15
    
    - task: SendEmail@1
      condition: succeeded()
      inputs:
        To: 'ops-team@example.com'
        Subject: 'Production Deployment Successful'
        Body: |
          Production deployment completed successfully!
          
          Build: $(Build.BuildId)
          Image: $(containerRegistry)/$(imageRepository):$(imageTag)
          
          Post-deployment monitoring: PASSED
          
          Application URL: https://production.example.com
      displayName: 'Send Success Notification'
```

**Key Features:**
1. **Multi-stage pipeline** with clear progression
2. **Automatic rollback** on failure
3. **Manual approval gate** before production
4. **Health checks** at each stage
5. **Backup and restore** capability
6. **Post-deployment monitoring**
7. **Email notifications** for success/failure
8. **Metrics validation** (error rate, response time)

---

### Question 48: Scenario - High memory usage in Jenkins causing builds to fail. How do you optimize?
**Answer:**

**Symptoms:**
```
- Jenkins UI slow/unresponsive
- Builds failing with OutOfMemoryError
- Agents disconnecting
- Pipeline jobs stuck
```

**Investigation:**

**1. Check Jenkins Memory Usage:**
```bash
# SSH to Jenkins server
top
# Check JAVA process memory

# Check Jenkins heap usage
# Manage Jenkins → System Information → Memory Usage
# Or via Groovy Console:
Runtime.getRuntime().totalMemory() / (1024 * 1024) + " MB"
Runtime.getRuntime().freeMemory() / (1024 * 1024) + " MB"
Runtime.getRuntime().maxMemory() / (1024 * 1024) + " MB"
```

**Solutions:**

**1. Increase JVM Heap Size:**
```bash
# Edit Jenkins service
sudo systemctl edit jenkins

# Add/modify:
[Service]
Environment="JAVA_OPTS=-Xms2g -Xmx4g -XX:MaxMetaspaceSize=512m -XX:+UseG1GC"

# Restart Jenkins
sudo systemctl restart jenkins

# Or in Docker:
docker run -e JAVA_OPTS="-Xmx4g -Xms2g" jenkins/jenkins:lts
```

**2. Limit Build History:**
```groovy
// In Jenkinsfile
options {
    buildDiscarder(logRotator(
        numToKeepStr: '10',          // Keep last 10 builds
        artifactNumToKeepStr: '5',   // Keep artifacts for 5 builds
        daysToKeepStr: '30'          // Keep builds for 30 days
    ))
}

// Or globally in Jenkins config
// Manage Jenkins → Configure System → Discard Old Builds
```

**3. Clean Up Workspace:**
```groovy
pipeline {
    agent any
    
    options {
        skipDefaultCheckout()  // Don't checkout automatically
    }
    
    stages {
        stage('Checkout') {
            steps {
                cleanWs()  // Clean workspace before build
                checkout scm
            }
        }
    }
    
    post {
        always {
            cleanWs()  // Clean workspace after build
        }
    }
}
```

**4. Optimize Plugin Usage:**
```
# Disable unused plugins
Manage Jenkins → Manage Plugins → Installed
# Uninstall plugins you don't need

# Common memory-hungry plugins:
# - Monitoring plugin (can use external monitoring instead)
# - Heavy UI plugins
# - Unused SCM plugins
```

**5. Use External Agents:**
```groovy
// Don't run builds on master
pipeline {
    agent {
        label 'build-agent'  // Use dedicated agent
    }
    // ...
}

// Configure master to not run builds
// Manage Jenkins → Manage Nodes → Built-In Node
// Set "# of executors" to 0
```

**6. Garbage Collection Tuning:**
```bash
# Use G1GC (better for large heaps)
JAVA_OPTS="-Xmx4g -XX:+UseG1GC -XX:MaxGCPauseMillis=200"

# Enable GC logging
JAVA_OPTS="-Xmx4g -XX:+PrintGCDetails -XX:+PrintGCDateStamps -Xloggc:/var/log/jenkins/gc.log"

# Analyze GC logs
tail -f /var/log/jenkins/gc.log
```

**7. Archive Large Artifacts Externally:**
```groovy
pipeline {
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Archive') {
            steps {
                // Don't use Jenkins archiveArtifacts for large files
                // Use external storage instead
                sh '''
                    az storage blob upload \
                      --account-name artifacts \
                      --container builds \
                      --name ${BUILD_ID}/app.jar \
                      --file target/app.jar
                '''
            }
        }
    }
}
```

**8. Reduce Concurrent Builds:**
```
Manage Jenkins → Configure System
# Limit concurrent builds per agent
# Reduce "# of executors" from 10 to 2-3
```

**9. Monitor and Alert:**
```groovy
// Install Monitoring plugin or use Prometheus
// Create alert for high memory usage

// Groovy script to monitor (run periodically):
def runtime = Runtime.getRuntime()
def usedMemory = (runtime.totalMemory() - runtime.freeMemory()) / (1024 * 1024)
def maxMemory = runtime.maxMemory() / (1024 * 1024)
def percentage = (usedMemory / maxMemory) * 100

if (percentage > 80) {
    // Send alert
    println "WARNING: Memory usage at ${percentage}%"
}
```

**10. Upgrade Jenkins and Plugins:**
```bash
# Newer versions often have memory optimizations
# Backup first!
sudo systemctl stop jenkins
sudo cp /var/lib/jenkins/config.xml /backup/
sudo apt-get update
sudo apt-get upgrade jenkins
sudo systemctl start jenkins
```

---

### Question 49: Python - Write a script to rotate Azure container registry images older than 30 days
**Answer:**
```python
#!/usr/bin/env python3
"""
Azure Container Registry Image Cleanup Script
Deletes images older than specified days to save storage costs
"""
from azure.identity import DefaultAzureCredential
from azure.mgmt.containerregistry import ContainerRegistryManagementClient
from azure.containerregistry import ContainerRegistryClient
from datetime import datetime, timedelta, timezone
import argparse
import logging

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s'
)

def parse_args():
    """Parse command line arguments"""
    parser = argparse.ArgumentParser(
        description='Clean up old ACR images'
    )
    parser.add_argument(
        '--registry-name',
        required=True,
        help='ACR registry name'
    )
    parser.add_argument(
        '--days',
        type=int,
        default=30,
        help='Delete images older than N days (default: 30)'
    )
    parser.add_argument(
        '--keep-latest',
        type=int,
        default=5,
        help='Always keep N latest images (default: 5)'
    )
    parser.add_argument(
        '--dry-run',
        action='store_true',
        help='Show what would be deleted without actually deleting'
    )
    parser.add_argument(
        '--exclude-repos',
        nargs='+',
        default=[],
        help='Repository names to exclude from cleanup'
    )
    parser.add_argument(
        '--exclude-tags',
        nargs='+',
        default=['latest', 'stable', 'prod'],
        help='Tag names to never delete'
    )
    return parser.parse_args()

def get_image_age(created_date):
    """Calculate image age in days"""
    if isinstance(created_date, str):
        created_date = datetime.fromisoformat(created_date.replace('Z', '+00:00'))
    
    now = datetime.now(timezone.utc)
    age = now - created_date
    return age.days

def list_repositories(client, registry_url):
    """List all repositories in ACR"""
    try:
        repositories = client.list_repository_names()
        return list(repositories)
    except Exception as e:
        logging.error(f"Error listing repositories: {e}")
        return []

def list_manifests(client, repository):
    """List all manifests (images) in a repository"""
    try:
        manifests = []
        for manifest in client.list_manifest_properties(repository):
            manifests.append({
                'digest': manifest.digest,
                'tags': manifest.tags or [],
                'created': manifest.created_on,
                'last_updated': manifest.last_updated_on
            })
        return manifests
    except Exception as e:
        logging.error(f"Error listing manifests for {repository}: {e}")
        return []

def should_delete_image(manifest, args, manifests_count, manifest_index):
    """Determine if image should be deleted based on criteria"""
    # Never delete if it's one of the latest N images
    if manifest_index < args.keep_latest:
        logging.debug(f"Keeping {manifest['tags']} - in latest {args.keep_latest}")
        return False
    
    # Never delete protected tags
    for tag in manifest['tags']:
        if tag in args.exclude_tags:
            logging.debug(f"Keeping {tag} - protected tag")
            return False
    
    # Check age
    age_days = get_image_age(manifest['created'])
    if age_days <= args.days:
        logging.debug(f"Keeping {manifest['tags']} - only {age_days} days old")
        return False
    
    return True

def delete_image(client, repository, digest, tags, dry_run=False):
    """Delete an image by digest"""
    try:
        if dry_run:
            logging.info(f"[DRY RUN] Would delete {repository}:{tags} ({digest})")
            return True
        else:
            client.delete_manifest(repository, digest)
            logging.info(f"Deleted {repository}:{tags} ({digest})")
            return True
    except Exception as e:
        logging.error(f"Error deleting {repository}:{tags}: {e}")
        return False

def cleanup_registry(args):
    """Main cleanup logic"""
    try:
        # Authenticate
        credential = DefaultAzureCredential()
        registry_url = f"https://{args.registry_name}.azurecr.io"
        
        logging.info(f"Connecting to registry: {registry_url}")
        client = ContainerRegistryClient(registry_url, credential)
        
        # Get all repositories
        repositories = list_repositories(client, registry_url)
        logging.info(f"Found {len(repositories)} repositories")
        
        total_deleted = 0
        total_kept = 0
        
        # Process each repository
        for repo in repositories:
            if repo in args.exclude_repos:
                logging.info(f"Skipping excluded repository: {repo}")
                continue
            
            logging.info(f"Processing repository: {repo}")
            
            # Get all manifests (images) in repository
            manifests = list_manifests(client, repo)
            
            if not manifests:
                logging.warning(f"No manifests found in {repo}")
                continue
            
            # Sort by last_updated (newest first)
            manifests.sort(key=lambda x: x['last_updated'], reverse=True)
            
            logging.info(f"Found {len(manifests)} manifests in {repo}")
            
            # Process each manifest
            for idx, manifest in enumerate(manifests):
                age_days = get_image_age(manifest['created'])
                tags = manifest['tags'] or ['<untagged>']
                
                logging.debug(f"Checking {repo}:{tags} - {age_days} days old")
                
                if should_delete_image(manifest, args, len(manifests), idx):
                    if delete_image(client, repo, manifest['digest'], tags, args.dry_run):
                        total_deleted += 1
                else:
                    total_kept += 1
        
        # Summary
        logging.info("=" * 60)
        logging.info(f"Cleanup {'simulation' if args.dry_run else 'completed'}")
        logging.info(f"Repositories processed: {len(repositories)}")
        logging.info(f"Images deleted: {total_deleted}")
        logging.info(f"Images kept: {total_kept}")
        
        if args.dry_run:
            logging.info("This was a DRY RUN - no images were actually deleted")
            logging.info("Run without --dry-run to actually delete images")
        
        return total_deleted
        
    except Exception as e:
        logging.error(f"Error during cleanup: {e}")
        raise

def main():
    """Main entry point"""
    args = parse_args()
    
    logging.info("Starting ACR image cleanup")
    logging.info(f"Registry: {args.registry_name}")
    logging.info(f"Delete images older than: {args.days} days")
    logging.info(f"Always keep latest: {args.keep_latest} images")
    logging.info(f"Protected tags: {args.exclude_tags}")
    logging.info(f"Excluded repos: {args.exclude_repos}")
    logging.info(f"Dry run: {args.dry_run}")
    logging.info("=" * 60)
    
    try:
        deleted_count = cleanup_registry(args)
        
        if deleted_count > 0:
            logging.info(f"Successfully cleaned up {deleted_count} old images")
        else:
            logging.info("No old images found to clean up")
        
        return 0
        
    except Exception as e:
        logging.error(f"Cleanup failed: {e}")
        return 1

if __name__ == "__main__":
    exit(main())

# Usage examples:
#
# Dry run (preview what would be deleted):
# python acr_cleanup.py --registry-name myregistry --dry-run
#
# Delete images older than 30 days (keep latest 5):
# python acr_cleanup.py --registry-name myregistry --days 30 --keep-latest 5
#
# Exclude specific repositories:
# python acr_cleanup.py --registry-name myregistry --exclude-repos frontend backend
#
# Custom protected tags:
# python acr_cleanup.py --registry-name myregistry --exclude-tags latest stable v1.0
#
# Schedule with cron (weekly cleanup):
# 0 2 * * 0 /usr/bin/python3 /scripts/acr_cleanup.py --registry-name myregistry --days 30
```

**Azure Function version (serverless):**
```python
# function_app.py
import azure.functions as func
import logging
from datetime import datetime

app = func.FunctionApp()

@app.schedule(schedule="0 0 2 * * 0", arg_name="myTimer",  # Weekly at 2 AM Sunday
              run_on_startup=False)
def acr_cleanup_timer(myTimer: func.TimerRequest) -> None:
    """Scheduled ACR cleanup function"""
    if myTimer.past_due:
        logging.info('The timer is past due!')
    
    logging.info(f'ACR cleanup function executed at {datetime.now()}')
    
    # Use the cleanup logic from above
    # cleanup_registry(args)
    
    logging.info('ACR cleanup completed')
```

---

### Question 50: Theory - Explain Infrastructure as Code (IaC) principles and compare Terraform vs ARM Templates vs Bicep
**Answer:**

**Infrastructure as Code (IaC) Principles:**

**Definition:**
Infrastructure as Code is managing and provisioning infrastructure through machine-readable configuration files rather than manual processes.

**Core Principles:**
1. **Declarative vs Imperative:**
   - Declarative: Describe desired state (Terraform, ARM, Bicep)
   - Imperative: Describe steps to achieve state (Scripts, Ansible playbooks)

2. **Idempotency:**
   - Running same code multiple times produces same result
   - No unintended side effects

3. **Version Control:**
   - Infrastructure code stored in Git
   - Track changes, review, rollback

4. **Automation:**
   - Eliminate manual steps
   - CI/CD for infrastructure

5. **Reproducibility:**
   - Same code creates identical infrastructure
   - Disaster recovery, multiple environments

**Comparison: Terraform vs ARM Templates vs Bicep**

**1. Terraform (HashiCorp)**

**Pros:**
- Multi-cloud (Azure, AWS, GCP, etc.)
- Large provider ecosystem
- Mature, widely adopted
- State management
- Plan preview before apply
- Module reusability

**Cons:**
- State file management complexity
- Learning curve for HCL
- Licensing changes (BSL license)
- Not Azure-native

**Example:**
```hcl
resource "azurerm_resource_group" "main" {
  name     = "myapp-rg"
  location = "East US"
}

resource "azurerm_virtual_network" "main" {
  name                = "myapp-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}

resource "azurerm_subnet" "main" {
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.1.0/24"]
}
```

**2. ARM Templates (Azure Resource Manager)**

**Pros:**
- Azure-native, full Azure feature support
- No external tools needed
- First-party support from Microsoft
- Immediate new feature availability
- No state file to manage

**Cons:**
- JSON syntax (verbose, hard to read)
- Azure-only (vendor lock-in)
- Limited error messages
- Steeper learning curve
- Complex expressions

**Example:**
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Network/virtualNetworks",
      "apiVersion": "2021-02-01",
      "name": "myapp-vnet",
      "location": "[parameters('location')]",
      "properties": {
        "addressSpace": {
          "addressPrefixes": ["10.0.0.0/16"]
        },
        "subnets": [
          {
            "name": "internal",
            "properties": {
              "addressPrefix": "10.0.1.0/24"
            }
          }
        ]
      }
    }
  ]
}
```

**3. Bicep (Microsoft)**

**Pros:**
- Azure-native, like ARM
- Clean, readable syntax (similar to HCL)
- Transpiles to ARM templates
- Type safety and IntelliSense
- Easier to learn than ARM
- No state file management
- Modules for reusability

**Cons:**
- Azure-only (like ARM)
- Relatively new (less mature than Terraform)
- Smaller community
- Some features lag behind ARM

**Example:**
```bicep
param location string = resourceGroup().location

resource vnet 'Microsoft.Network/virtualNetworks@2021-02-01' = {
  name: 'myapp-vnet'
  location: location
  properties: {
    addressSpace: {
      addressPrefixes: [
        '10.0.0.0/16'
      ]
    }
    subnets: [
      {
        name: 'internal'
        properties: {
          addressPrefix: '10.0.1.0/24'
        }
      }
    ]
  }
}

output vnetId string = vnet.id
```

**Side-by-Side Comparison:**

| Feature | Terraform | ARM Templates | Bicep |
|---------|-----------|---------------|-------|
| **Cloud Support** | Multi-cloud | Azure only | Azure only |
| **Syntax** | HCL (clean) | JSON (verbose) | Clean, concise |
| **State Management** | Required | No state file | No state file |
| **Learning Curve** | Moderate | Steep | Easy |
| **Maturity** | Very mature | Mature | New (2020) |
| **Community** | Large | Large | Growing |
| **Tooling** | Excellent | Good | Excellent (VS Code) |
| **Preview** | terraform plan | what-if | what-if |
| **Module Support** | Yes | Linked templates | Yes (modules) |
| **IDE Support** | Good | Basic | Excellent |
| **Azure Features** | Slight lag | Immediate | Immediate |
| **Cost** | Free (OSS/BSL) | Free | Free |

**When to Use Which:**

**Use Terraform when:**
- Multi-cloud strategy
- Already using Terraform
- Need mature ecosystem
- Want provider diversity

**Use ARM Templates when:**
- Legacy infrastructure
- Need immediate new Azure features
- Complex nested deployments
- Already heavily invested in ARM

**Use Bicep when:**
- Azure-only deployment
- Want clean, readable syntax
- Starting new IaC project
- Want Azure-native experience with good DX
- Team prefers simpler syntax

**Real-World Recommendation:**
For Azure-focused organizations: **Bicep** (best of both worlds - clean syntax like Terraform, Azure-native like ARM)

For multi-cloud: **Terraform** (industry standard, mature)

**Hybrid Approach:**
Many organizations use:
- Terraform for multi-cloud resources (AKS, networking, VMs)
- Bicep/ARM for Azure-specific features (Policy, Blueprints)

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)


