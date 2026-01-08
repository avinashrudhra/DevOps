# Kubernetes ConfigMaps & Secrets - Complete Guide

**Understanding Configuration and Sensitive Data Management**

---

## 📚 Table of Contents

1. [What are ConfigMaps?](#what-are-configmaps)
2. [Creating ConfigMaps](#creating-configmaps)
3. [Using ConfigMaps](#using-configmaps)
4. [What are Secrets?](#what-are-secrets)
5. [Secret Types](#secret-types)
6. [Creating Secrets](#creating-secrets)
7. [Using Secrets](#using-secrets)
8. [Best Practices](#best-practices)

---

## 📋 What are ConfigMaps?

**ConfigMaps** store non-confidential configuration data as key-value pairs.

### **Simple Analogy:**

```
ConfigMap = Settings File

Without ConfigMap:
  - Configuration hardcoded in application
  - Change config = Rebuild image
  - Different environments = Different images

With ConfigMap:
  - Configuration external
  - Change config = Update ConfigMap (no rebuild!)
  - Same image, different configs
```

---

### **The Problem ConfigMaps Solve:**

**❌ Without ConfigMaps:**
```dockerfile
# Dockerfile
FROM node:18
COPY app.js /app/
ENV DATABASE_URL=postgres://prod-db:5432/mydb  # Hardcoded!
ENV LOG_LEVEL=info                             # Hardcoded!
CMD ["node", "/app/app.js"]

Problems:
1. Can't change config without rebuild
2. Same image for dev/prod (bad!)
3. Configuration mixed with code
4. Secrets in image (security risk!)
```

**✅ With ConfigMaps:**
```yaml
# ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DATABASE_URL: "postgres://dev-db:5432/mydb"
  LOG_LEVEL: "debug"

# Pod uses ConfigMap
spec:
  containers:
  - name: app
    image: myapp:1.0  # Same image everywhere!
    envFrom:
    - configMapRef:
        name: app-config  # Config injected at runtime

Benefits:
✓ Decouple configuration from code
✓ Same image, different configs
✓ Easy configuration changes
✓ Environment-specific configs
```

---

## 🏗️ Creating ConfigMaps

Multiple ways to create ConfigMaps.

### **Method 1: From Literal Values:**

```bash
kubectl create configmap app-config \
  --from-literal=APP_NAME=myapp \
  --from-literal=APP_VERSION=1.0 \
  --from-literal=LOG_LEVEL=info
```

**Result:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_NAME: "myapp"
  APP_VERSION: "1.0"
  LOG_LEVEL: "info"
```

---

### **Method 2: From File:**

**config.properties:**
```properties
database.host=postgres.default.svc.cluster.local
database.port=5432
database.name=mydb
app.timeout=30
app.retries=3
```

```bash
kubectl create configmap app-config --from-file=config.properties
```

**Result:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  config.properties: |
    database.host=postgres.default.svc.cluster.local
    database.port=5432
    database.name=mydb
    app.timeout=30
    app.retries=3
```

---

### **Method 3: From Multiple Files:**

```bash
kubectl create configmap app-config \
  --from-file=app.conf \
  --from-file=logging.conf \
  --from-file=database.conf
```

**Result:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  app.conf: |
    # app.conf contents
  logging.conf: |
    # logging.conf contents
  database.conf: |
    # database.conf contents
```

---

### **Method 4: From Directory:**

```bash
# All files in config/ directory
kubectl create configmap app-config --from-file=config/
```

---

### **Method 5: YAML Manifest (Recommended for production):**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: production
data:
  # Simple key-values
  APP_NAME: "myapp"
  LOG_LEVEL: "info"
  DATABASE_HOST: "postgres.default.svc.cluster.local"
  
  # Multi-line configuration file
  nginx.conf: |
    server {
      listen 80;
      server_name example.com;
      
      location / {
        proxy_pass http://backend:8080;
        proxy_set_header Host $host;
      }
    }
  
  # JSON configuration
  app-config.json: |
    {
      "database": {
        "host": "postgres",
        "port": 5432
      },
      "cache": {
        "enabled": true,
        "ttl": 300
      }
    }
```

---

## 💻 Using ConfigMaps

Several ways to use ConfigMaps in pods.

### **Method 1: As Environment Variables (All keys):**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: app
    image: myapp:1.0
    envFrom:
    - configMapRef:
        name: app-config  # All keys become env vars
```

**Result in container:**
```bash
# All ConfigMap keys available as environment variables
echo $APP_NAME       # myapp
echo $LOG_LEVEL      # info
echo $DATABASE_HOST  # postgres.default.svc.cluster.local
```

---

### **Method 2: As Environment Variables (Specific keys):**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: app
    image: myapp:1.0
    env:
    - name: APP_NAME              # Env var name in container
      valueFrom:
        configMapKeyRef:
          name: app-config        # ConfigMap name
          key: APP_NAME           # Key in ConfigMap
    
    - name: LOG_LEVEL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: LOG_LEVEL
    
    # Can also set static values
    - name: ENVIRONMENT
      value: "production"
```

---

### **Method 3: As Volume (Mount entire ConfigMap):**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    volumeMounts:
    - name: config-volume
      mountPath: /etc/nginx/conf.d  # Mount ConfigMap here
  
  volumes:
  - name: config-volume
    configMap:
      name: nginx-config  # ConfigMap with nginx.conf
```

**Result in container:**
```bash
# Files created from ConfigMap keys
/etc/nginx/conf.d/
  ├── nginx.conf       # From ConfigMap key: nginx.conf
  └── upstream.conf    # From ConfigMap key: upstream.conf

# nginx reads configuration from these files
```

---

### **Method 4: As Volume (Specific keys with custom paths):**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
  - name: app
    image: myapp:1.0
    volumeMounts:
    - name: config
      mountPath: /config
  
  volumes:
  - name: config
    configMap:
      name: app-config
      items:
      - key: app-config.json      # ConfigMap key
        path: config.json         # File name in container
      - key: database-config.json
        path: db/config.json      # Can use subdirectories
```

**Result:**
```bash
/config/
  ├── config.json          # From app-config.json key
  └── db/
      └── config.json      # From database-config.json key
```

---

### **Real-World Example:**

**ConfigMap:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: webapp-config
data:
  APP_ENV: "production"
  API_URL: "https://api.example.com"
  FEATURE_FLAG_NEW_UI: "true"
  
  app.properties: |
    database.pool.size=20
    cache.ttl=300
    timeout=30
```

**Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 3
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
        image: webapp:2.0
        
        # Environment variables from ConfigMap
        envFrom:
        - configMapRef:
            name: webapp-config
        
        # Mount properties file
        volumeMounts:
        - name: config
          mountPath: /app/config
      
      volumes:
      - name: config
        configMap:
          name: webapp-config
          items:
          - key: app.properties
            path: application.properties
```

---

## 🔐 What are Secrets?

**Secrets** store sensitive data (passwords, tokens, keys) in base64-encoded format.

### **Simple Analogy:**

```
Secret = Locked Safe

ConfigMap:
  - Public bulletin board
  - Everyone can see
  - Non-sensitive data

Secret:
  - Locked safe
  - Base64 encoded
  - RBAC protected
  - For sensitive data
```

---

### **⚠️ Important Security Notes:**

```
Secrets are NOT encrypted by default!
They are only base64-encoded (easily decoded)

Base64 is NOT encryption:
  echo "cGFzc3dvcmQ=" | base64 -d
  # Output: password

For true security:
✓ Enable encryption at rest in etcd
✓ Use RBAC to restrict access
✓ Use external secret managers (Vault, AWS Secrets Manager)
✓ Never commit secrets to Git!
```

---

## 🏷️ Secret Types

Kubernetes has several secret types.

### **Secret Types:**

**1. Opaque (Default):**
```
Generic secret for arbitrary data
Use: Database passwords, API keys
```

**2. kubernetes.io/service-account-token:**
```
Service account token
Use: Pod authentication
```

**3. kubernetes.io/dockerconfigjson:**
```
Docker registry credentials
Use: Pulling private images
```

**4. kubernetes.io/tls:**
```
TLS certificate and key
Use: HTTPS, Ingress TLS
```

**5. kubernetes.io/basic-auth:**
```
Basic authentication credentials
Use: Username/password pairs
```

**6. kubernetes.io/ssh-auth:**
```
SSH private key
Use: Git clone, SSH access
```

---

## 🔨 Creating Secrets

Multiple ways to create Secrets.

### **Method 1: From Literal Values:**

```bash
kubectl create secret generic db-credentials \
  --from-literal=username=admin \
  --from-literal=password=SuperSecret123!
```

**Result:**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: YWRtaW4=              # base64("admin")
  password: U3VwZXJTZWNyZXQxMjMh  # base64("SuperSecret123!")
```

---

### **Method 2: From Files:**

**password.txt:**
```
SuperSecret123!
```

```bash
kubectl create secret generic db-password --from-file=password.txt
```

---

### **Method 3: YAML Manifest:**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: YWRtaW4=              # base64 encoded "admin"
  password: U3VwZXJTZWNyZXQxMjMh  # base64 encoded "SuperSecret123!"
```

**Encoding values:**
```bash
echo -n "admin" | base64
# YWRtaW4=

echo -n "SuperSecret123!" | base64
# U3VwZXJTZWNyZXQxMjMh
```

---

### **Method 4: StringData (Plain text - Kubernetes encodes):**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
stringData:  # Plain text, Kubernetes will encode
  username: admin
  password: SuperSecret123!

# Kubernetes converts to:
# data:
#   username: YWRtaW4=
#   password: U3VwZXJTZWNyZXQxMjMh
```

---

### **Docker Registry Secret:**

```bash
kubectl create secret docker-registry regcred \
  --docker-server=docker.io \
  --docker-username=myuser \
  --docker-password=mypassword \
  --docker-email=user@example.com
```

**Using in pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: private-app
spec:
  imagePullSecrets:
  - name: regcred  # Use docker registry secret
  containers:
  - name: app
    image: myuser/private-app:1.0  # Private image
```

---

### **TLS Secret:**

```bash
kubectl create secret tls tls-secret \
  --cert=tls.crt \
  --key=tls.key
```

**Result:**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
type: kubernetes.io/tls
data:
  tls.crt: <base64-encoded-cert>
  tls.key: <base64-encoded-key>
```

**Using in Ingress:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
spec:
  tls:
  - hosts:
    - example.com
    secretName: tls-secret  # TLS secret
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web
            port:
              number: 80
```

---

## 🔑 Using Secrets

Similar to ConfigMaps but for sensitive data.

### **Method 1: As Environment Variables:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: db-app
spec:
  containers:
  - name: app
    image: myapp:1.0
    env:
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: username
    
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: password
```

**In container:**
```bash
echo $DB_USERNAME  # admin
echo $DB_PASSWORD  # SuperSecret123!
```

---

### **Method 2: As Volume (Mounted files):**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
  - name: app
    image: myapp:1.0
    volumeMounts:
    - name: secrets
      mountPath: /etc/secrets
      readOnly: true  # Read-only for security
  
  volumes:
  - name: secrets
    secret:
      secretName: db-credentials
```

**Result in container:**
```bash
/etc/secrets/
  ├── username  # Contains: admin
  └── password  # Contains: SuperSecret123!

# App reads from files
cat /etc/secrets/username  # admin
cat /etc/secrets/password  # SuperSecret123!
```

---

### **Real-World Example:**

**Secrets:**
```yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
stringData:
  host: postgres.default.svc.cluster.local
  username: dbadmin
  password: DbP@ssw0rd!
  database: production

---
apiVersion: v1
kind: Secret
metadata:
  name: api-keys
type: Opaque
stringData:
  stripe-key: sk_live_abc123def456
  sendgrid-key: SG.xyz789abc123
```

**Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 3
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
        
        # Database credentials as env vars
        env:
        - name: DB_HOST
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: host
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: username
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
        - name: DB_NAME
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: database
        
        # API keys as mounted files
        volumeMounts:
        - name: api-keys
          mountPath: /secrets/api
          readOnly: true
      
      volumes:
      - name: api-keys
        secret:
          secretName: api-keys
```

---

## ✅ Best Practices

### **ConfigMaps:**

**1. One ConfigMap per application/component:**
```yaml
# Good
app-frontend-config
app-backend-config
database-config

# Bad (mixing concerns)
all-configs
```

**2. Use namespaces for environments:**
```yaml
# Development
namespace: development
configMap: app-config

# Production
namespace: production
configMap: app-config  # Same name, different namespace
```

**3. Version ConfigMaps:**
```yaml
metadata:
  name: app-config-v2  # Version in name
  labels:
    version: "2.0"
```

**4. Use immutable ConfigMaps (Kubernetes 1.19+):**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
immutable: true  # Cannot be modified
data:
  APP_NAME: "myapp"

# Benefits:
# - Protection from accidental updates
# - Better performance (Kubernetes doesn't watch for changes)
# - To update, create new ConfigMap (app-config-v2)
```

---

### **Secrets:**

**1. Never commit secrets to Git!**
```bash
# .gitignore
secrets/
*.secret.yaml
*-secret.yaml
```

**2. Use external secret managers:**
```yaml
# Instead of storing secrets in Kubernetes:
# - AWS Secrets Manager
# - Azure Key Vault
# - HashiCorp Vault
# - Google Secret Manager

# Use Kubernetes External Secrets Operator
# Syncs from external vault to Kubernetes
```

**3. Enable encryption at rest:**
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
              secret: <32-byte-base64-key>
      - identity: {}

# API server flag:
# --encryption-provider-config=/etc/kubernetes/encryption-config.yaml
```

**4. Use RBAC to restrict access:**
```yaml
# Only specific service accounts can read secrets
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  resourceNames: ["db-credentials"]  # Specific secret
  verbs: ["get"]
```

**5. Rotate secrets regularly:**
```bash
# Update secret
kubectl create secret generic db-password \
  --from-literal=password=NewPassword123! \
  --dry-run=client -o yaml | kubectl apply -f -

# Rolling restart to pick up new secret
kubectl rollout restart deployment/myapp
```

**6. Use secret volumes instead of env vars:**
```
Env vars:
  - Visible in pod spec
  - Logged in crash dumps
  - Exposed to all processes

Volumes:
  - Files with restricted permissions
  - Not in environment
  - More secure
```

**7. Limit secret size:**
```
Max secret size: 1MB
Store large files elsewhere (S3, ConfigMap for non-sensitive)
```

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

