# Helm Hands-On Exercises

30+ practical exercises to master Helm from beginner to production expert.

---

## Beginner Level Exercises

### Exercise 1: Install and Use Public Helm Chart
**Objective**: Deploy application from public Helm repository

**Tasks:**
1. Add Bitnami repository
2. Install NGINX
3. Access the application
4. Uninstall

<details>
<summary>Solution</summary>

```bash
# 1. Add repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# 2. Search for NGINX
helm search repo nginx

# 3. Install NGINX
helm install my-nginx bitnami/nginx

# 4. Check status
helm status my-nginx
helm list

# 5. Get service
kubectl get svc my-nginx

# 6. Port forward to access
kubectl port-forward svc/my-nginx 8080:80

# Open browser: http://localhost:8080

# 7. Get values
helm get values my-nginx

# 8. Uninstall
helm uninstall my-nginx
```

**Verification:**
- NGINX accessible via port-forward
- Release listed in `helm list`
- Successfully uninstalled
</details>

---

### Exercise 2: Create Your First Helm Chart
**Objective**: Create and deploy custom Helm chart

<details>
<summary>Solution</summary>

```bash
# 1. Create chart
helm create myapp

# 2. Explore structure
tree myapp/
# myapp/
# ├── Chart.yaml
# ├── values.yaml
# ├── templates/
# │   ├── deployment.yaml
# │   ├── service.yaml
# │   ├── ingress.yaml
# │   └── _helpers.tpl
# └── charts/

# 3. Modify values.yaml
cat <<EOF > myapp/values.yaml
replicaCount: 2

image:
  repository: nginx
  tag: "1.21"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 128Mi
EOF

# 4. Lint chart
helm lint myapp/

# 5. Dry run
helm install myapp-test myapp/ --dry-run --debug

# 6. Install chart
helm install myapp-test myapp/

# 7. Verify
kubectl get all -l app.kubernetes.io/name=myapp
helm status myapp-test

# 8. Upgrade
helm upgrade myapp-test myapp/ --set replicaCount=3

# 9. Check history
helm history myapp-test

# 10. Rollback
helm rollback myapp-test 1

# 11. Uninstall
helm uninstall myapp-test
```
</details>

---

### Exercise 3: Customize Chart with Values
**Objective**: Override default values using multiple methods

<details>
<summary>Solution</summary>

**Create custom values file:**
```yaml
# custom-values.yaml
replicaCount: 3

image:
  repository: nginx
  tag: "1.22"

service:
  type: LoadBalancer
  port: 8080

resources:
  limits:
    cpu: 200m
    memory: 256Mi
  requests:
    cpu: 100m
    memory: 128Mi

ingress:
  enabled: true
  hosts:
    - host: myapp.example.com
      paths:
        - path: /
          pathType: Prefix
```

**Install with custom values:**
```bash
# Method 1: Using values file
helm install myapp myapp/ -f custom-values.yaml

# Method 2: Command line override
helm install myapp myapp/ \
  --set replicaCount=3 \
  --set image.tag=1.22 \
  --set service.type=LoadBalancer

# Method 3: Multiple values files (right-most takes precedence)
helm install myapp myapp/ \
  -f values.yaml \
  -f custom-values.yaml \
  -f production-values.yaml

# Check applied values
helm get values myapp
helm get values myapp --all  # Include defaults
```
</details>

---

## Intermediate Level Exercises

### Exercise 4: Create Chart with ConfigMap and Secret
**Objective**: Template ConfigMaps and Secrets

<details>
<summary>Solution</summary>

**templates/configmap.yaml:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "myapp.fullname" . }}
  labels:
    {{- include "myapp.labels" . | nindent 4 }}
data:
  {{- range $key, $value := .Values.config }}
  {{ $key }}: {{ $value | quote }}
  {{- end }}
```

**templates/secret.yaml:**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: {{ include "myapp.fullname" . }}
  labels:
    {{- include "myapp.labels" . | nindent 4 }}
type: Opaque
data:
  {{- range $key, $value := .Values.secrets }}
  {{ $key }}: {{ $value | b64enc | quote }}
  {{- end }}
```

**values.yaml:**
```yaml
config:
  APP_ENV: production
  LOG_LEVEL: info
  API_URL: https://api.example.com

secrets:
  DB_PASSWORD: mypassword
  API_KEY: mysecretkey
```

**templates/deployment.yaml (add env):**
```yaml
spec:
  template:
    spec:
      containers:
        - name: {{ .Chart.Name }}
          envFrom:
            - configMapRef:
                name: {{ include "myapp.fullname" . }}
            - secretRef:
                name: {{ include "myapp.fullname" . }}
```

**Deploy:**
```bash
helm install myapp ./myapp

# Verify
kubectl get configmap
kubectl get secret
kubectl describe configmap myapp
```
</details>

---

### Exercise 5: Multi-Environment Deployment
**Objective**: Deploy to dev, staging, and production

<details>
<summary>Solution</summary>

**values-dev.yaml:**
```yaml
replicaCount: 1

image:
  tag: "dev-latest"

resources:
  limits:
    cpu: 100m
    memory: 128Mi

ingress:
  enabled: true
  hosts:
    - host: myapp-dev.example.com
```

**values-staging.yaml:**
```yaml
replicaCount: 2

image:
  tag: "v1.0.0-rc1"

resources:
  limits:
    cpu: 200m
    memory: 256Mi

ingress:
  enabled: true
  hosts:
    - host: myapp-staging.example.com
```

**values-production.yaml:**
```yaml
replicaCount: 3

image:
  tag: "v1.0.0"

resources:
  limits:
    cpu: 500m
    memory: 512Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10

ingress:
  enabled: true
  className: "nginx"
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
  hosts:
    - host: myapp.example.com
  tls:
    - secretName: myapp-tls
      hosts:
        - myapp.example.com
```

**Deploy to each environment:**
```bash
# Development
helm install myapp-dev ./myapp -f values-dev.yaml -n dev --create-namespace

# Staging
helm install myapp-staging ./myapp -f values-staging.yaml -n staging --create-namespace

# Production
helm install myapp-prod ./myapp -f values-production.yaml -n production --create-namespace

# List all
helm list -A
```
</details>

---

### Exercise 6: Chart with Dependencies
**Objective**: Create chart with PostgreSQL and Redis dependencies

<details>
<summary>Solution</summary>

**Chart.yaml:**
```yaml
apiVersion: v2
name: myapp
description: Application with database
version: 1.0.0
appVersion: "1.0.0"

dependencies:
  - name: postgresql
    version: "11.x.x"
    repository: https://charts.bitnami.com/bitnami
    condition: postgresql.enabled
  - name: redis
    version: "17.x.x"
    repository: https://charts.bitnami.com/bitnami
    condition: redis.enabled
```

**values.yaml:**
```yaml
postgresql:
  enabled: true
  auth:
    username: myapp
    password: mypassword
    database: myappdb
  primary:
    persistence:
      enabled: true
      size: 10Gi

redis:
  enabled: true
  auth:
    enabled: false
  master:
    persistence:
      enabled: true
      size: 5Gi
```

**Update dependencies:**
```bash
# Download dependencies
helm dependency update ./myapp

# Verify
ls myapp/charts/
# postgresql-11.x.x.tgz
# redis-17.x.x.tgz

# List dependencies
helm dependency list ./myapp

# Install with dependencies
helm install myapp ./myapp

# Verify all components
kubectl get pods
# myapp-xxx
# myapp-postgresql-0
# myapp-redis-master-0
```

**Connect to database from app:**
```yaml
# templates/deployment.yaml
env:
  - name: DB_HOST
    value: {{ include "myapp.fullname" . }}-postgresql
  - name: DB_PORT
    value: "5432"
  - name: DB_NAME
    value: {{ .Values.postgresql.auth.database }}
  - name: DB_USER
    value: {{ .Values.postgresql.auth.username }}
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: {{ include "myapp.fullname" . }}-postgresql
        key: password
  - name: REDIS_HOST
    value: {{ include "myapp.fullname" . }}-redis-master
  - name: REDIS_PORT
    value: "6379"
```
</details>

---

## Advanced Level Exercises

### Exercise 7: Helm Hooks for Database Migration
**Objective**: Run database migration before deployment

<details>
<summary>Solution</summary>

**templates/hooks/db-migration.yaml:**
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ include "myapp.fullname" . }}-db-migration
  labels:
    {{- include "myapp.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": pre-install,pre-upgrade
    "helm.sh/hook-weight": "1"
    "helm.sh/hook-delete-policy": before-hook-creation,hook-succeeded
spec:
  template:
    metadata:
      labels:
        app: db-migration
    spec:
      restartPolicy: Never
      containers:
        - name: migration
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          command:
            - /bin/sh
            - -c
            - |
              echo "Running database migrations..."
              ./migrate.sh
          env:
            - name: DB_HOST
              value: {{ include "myapp.fullname" . }}-postgresql
            - name: DB_NAME
              value: {{ .Values.postgresql.auth.database }}
            - name: DB_USER
              value: {{ .Values.postgresql.auth.username }}
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: {{ include "myapp.fullname" . }}-postgresql
                  key: password
      initContainers:
        - name: wait-for-db
          image: busybox:1.35
          command:
            - sh
            - -c
            - |
              until nc -z {{ include "myapp.fullname" . }}-postgresql 5432; do
                echo "Waiting for database..."
                sleep 2
              done
```

**Test:**
```bash
# Install (migration runs automatically)
helm install myapp ./myapp

# Check migration job
kubectl get jobs
kubectl logs job/myapp-db-migration

# Upgrade (migration runs again)
helm upgrade myapp ./myapp --set image.tag=v2.0.0
```
</details>

---

### Exercise 8: Chart Tests
**Objective**: Create automated tests for your chart

<details>
<summary>Solution</summary>

**templates/tests/test-connection.yaml:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "myapp.fullname" . }}-test-connection"
  labels:
    {{- include "myapp.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": test
spec:
  containers:
    - name: wget
      image: busybox:1.35
      command: ['wget']
      args: ['{{ include "myapp.fullname" . }}:{{ .Values.service.port }}']
  restartPolicy: Never
```

**templates/tests/test-database.yaml:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "myapp.fullname" . }}-test-database"
  labels:
    {{- include "myapp.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": test
spec:
  containers:
    - name: pg-ready
      image: postgres:13
      command:
        - sh
        - -c
        - |
          pg_isready -h {{ include "myapp.fullname" . }}-postgresql \
                     -U {{ .Values.postgresql.auth.username }}
  restartPolicy: Never
```

**Run tests:**
```bash
# Install chart
helm install myapp ./myapp

# Run tests
helm test myapp

# View test results
helm test myapp --logs

# Cleanup test pods
kubectl delete pod -l helm.sh/hook=test
```
</details>

---

### Exercise 9: Package and Publish Chart
**Objective**: Create Helm repository

<details>
<summary>Solution</summary>

**Package chart:**
```bash
# Package chart
helm package ./myapp
# Creates: myapp-1.0.0.tgz

# Package with specific version
helm package ./myapp --version 1.0.1

# Package and sign
helm package ./myapp --sign --key mykey --keyring ~/.gnupg/secring.gpg
```

**Create repository:**
```bash
# Create directory structure
mkdir helm-repo
mv myapp-*.tgz helm-repo/

# Generate index
cd helm-repo
helm repo index . --url https://charts.example.com

# Creates index.yaml
cat index.yaml

# Serve locally for testing
python3 -m http.server 8080
```

**Use your repository:**
```bash
# Add repository
helm repo add myrepo http://localhost:8080

# Search
helm search repo myrepo/

# Install from your repo
helm install myapp myrepo/myapp
```

**Publish to GitHub Pages:**
```bash
# Create gh-pages branch
git checkout -b gh-pages

# Add charts
mkdir charts
mv myapp-*.tgz charts/
helm repo index charts --url https://username.github.io/helm-charts/charts

# Commit and push
git add charts/
git commit -m "Add Helm charts"
git push origin gh-pages

# Use published repository
helm repo add myrepo https://username.github.io/helm-charts/charts
```
</details>

---

### Exercise 10: Advanced Templating
**Objective**: Implement complex template logic

<details>
<summary>Solution</summary>

**templates/deployment.yaml with advanced features:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "myapp.fullname" . }}
  labels:
    {{- include "myapp.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "myapp.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      annotations:
        checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
        {{- with .Values.podAnnotations }}
        {{- toYaml . | nindent 8 }}
        {{- end }}
      labels:
        {{- include "myapp.selectorLabels" . | nindent 8 }}
    spec:
      {{- if .Values.image.pullSecrets }}
      imagePullSecrets:
        {{- range .Values.image.pullSecrets }}
        - name: {{ . }}
        {{- end }}
      {{- end }}
      serviceAccountName: {{ include "myapp.serviceAccountName" . }}
      securityContext:
        {{- toYaml .Values.podSecurityContext | nindent 8 }}
      {{- if .Values.initContainers }}
      initContainers:
        {{- toYaml .Values.initContainers | nindent 8 }}
      {{- end }}
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: {{ .Values.service.targetPort | default .Values.service.port }}
              protocol: TCP
          {{- if .Values.livenessProbe }}
          livenessProbe:
            {{- toYaml .Values.livenessProbe | nindent 12 }}
          {{- end }}
          {{- if .Values.readinessProbe }}
          readinessProbe:
            {{- toYaml .Values.readinessProbe | nindent 12 }}
          {{- end }}
          {{- if or .Values.env .Values.envFrom }}
          {{- with .Values.env }}
          env:
            {{- toYaml . | nindent 12 }}
          {{- end }}
          {{- with .Values.envFrom }}
          envFrom:
            {{- toYaml . | nindent 12 }}
          {{- end }}
          {{- end }}
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
          {{- with .Values.volumeMounts }}
          volumeMounts:
            {{- toYaml . | nindent 12 }}
          {{- end }}
      {{- with .Values.volumes }}
      volumes:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.affinity }}
      affinity:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.tolerations }}
      tolerations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
```
</details>

---

**Complete all exercises to master Helm! 📦🚀**

For more resources:
- [Learning Roadmap](helm-learning-roadmap.md)
- [Quick Reference](helm-quick-reference.md)
- [Troubleshooting Guide](helm-troubleshooting-guide.md)
- [Interview Questions](helm-interview-questions.md)


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

