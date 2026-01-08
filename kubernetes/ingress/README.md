# Kubernetes Ingress - Complete Theoretical Guide

**Understanding HTTP/HTTPS Routing and Load Balancing**

---

## 📚 Table of Contents

1. [What is Ingress?](#what-is-ingress)
2. [Ingress vs Service](#ingress-vs-service)
3. [Ingress Controller](#ingress-controller)
4. [Path-Based Routing](#path-routing)
5. [Host-Based Routing](#host-routing)
6. [TLS/HTTPS](#tls-https)
7. [Ingress Annotations](#annotations)
8. [Popular Ingress Controllers](#controllers)

---

## 🌐 What is Ingress?

An **Ingress** exposes HTTP and HTTPS routes from outside the cluster to services within the cluster.

### **Simple Analogy:**

```
Ingress = Receptionist at Company Building

External visitors (HTTP requests)
         ↓
Receptionist (Ingress):
  - "Looking for /api? → Go to API team (api-service)"
  - "Looking for /web? → Go to Frontend team (web-service)"
  - "Looking for blog.example.com? → Go to Blog team"
         ↓
Routes to correct internal team (Service → Pods)

One entry point, routes to many services
```

---

### **Without Ingress:**

```
Need to expose 5 services:
  web-service
  api-service
  blog-service
  docs-service
  admin-service

Option 1: NodePort for each
  web-service: NodePort 30080
  api-service: NodePort 30081
  blog-service: NodePort 30082
  ...
  Access: http://node-ip:30080, http://node-ip:30081...
  Problem: Ugly URLs, port management

Option 2: LoadBalancer for each
  5 services = 5 cloud load balancers
  Cost: $$$$$ (each LB costs money!)
```

---

### **With Ingress:**

```
Single Ingress:
  example.com/        → web-service
  example.com/api     → api-service
  blog.example.com    → blog-service
  docs.example.com    → docs-service
  admin.example.com   → admin-service

Benefits:
✓ Single load balancer (one IP/DNS)
✓ Clean URLs
✓ Path-based routing
✓ Host-based routing
✓ TLS termination
✓ Cost-effective

Cost: $ (single LB)
```

---

## 🆚 Ingress vs Service

```
Service (ClusterIP):
  Internal cluster access only
  No external exposure

Service (NodePort):
  External access via node IP + port
  http://node-ip:30080
  Limited, port range 30000-32767

Service (LoadBalancer):
  External cloud load balancer per service
  Expensive (multiple LBs)
  One LB per service

Ingress:
  Single load balancer
  Routes to multiple services
  HTTP/HTTPS routing
  Cost-effective
  Clean URLs
```

---

### **Architecture:**

```
Internet
   ↓
Cloud Load Balancer (1 LB)
   ↓
┌──────────────────────────────────────────┐
│  Kubernetes Cluster                       │
│                                           │
│  Ingress Controller (nginx/traefik)      │
│  ┌────────────────────────────────────┐  │
│  │  Ingress Rules:                    │  │
│  │  example.com/     → web-service    │  │
│  │  example.com/api  → api-service    │  │
│  │  blog.example.com → blog-service   │  │
│  └────────────────────────────────────┘  │
│         ↓             ↓          ↓        │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  │
│  │   web   │  │   api   │  │  blog   │  │
│  │ Service │  │ Service │  │ Service │  │
│  └─────────┘  └─────────┘  └─────────┘  │
│       ↓             ↓            ↓        │
│     Pods          Pods         Pods      │
└──────────────────────────────────────────┘

Single entry point, smart routing
```

---

## 🎮 Ingress Controller

Ingress resource needs an Ingress Controller to work.

### **What is Ingress Controller?**

```
Ingress Resource = Configuration (rules)
Ingress Controller = Software that implements rules

Like:
  Ingress = Recipe
  Ingress Controller = Chef who follows recipe
```

---

### **Popular Controllers:**

1. **NGINX Ingress Controller** (Most popular)
2. **Traefik**
3. **HAProxy**
4. **Contour** (Envoy-based)
5. **Kong**
6. **AWS ALB Ingress Controller**
7. **Azure Application Gateway Ingress Controller**
8. **GCE Ingress Controller**

---

### **Installing NGINX Ingress Controller:**

```bash
# Using Helm
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install ingress-nginx ingress-nginx/ingress-nginx

# Or using kubectl
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml

# Verify
kubectl get pods -n ingress-nginx

NAME                                        READY   STATUS
ingress-nginx-controller-abc123             1/1     Running

# Check service (gets external IP)
kubectl get svc -n ingress-nginx

NAME                                 TYPE           EXTERNAL-IP
ingress-nginx-controller             LoadBalancer   52.123.45.67
```

---

## 🛣️ Path-Based Routing

Route based on URL path.

### **Example:**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-based-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: example.com
    http:
      paths:
      # Route /web to web-service
      - path: /web
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
      
      # Route /api to api-service
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
      
      # Route /blog to blog-service
      - path: /blog
        pathType: Prefix
        backend:
          service:
            name: blog-service
            port:
              number: 80
```

---

### **How it works:**

```
User requests:

http://example.com/web/index.html
  ↓
Ingress: "Path starts with /web"
  ↓
Routes to: web-service:80
  ↓
web-service → web-pods

http://example.com/api/users
  ↓
Ingress: "Path starts with /api"
  ↓
Routes to: api-service:8080
  ↓
api-service → api-pods

http://example.com/blog/posts
  ↓
Ingress: "Path starts with /blog"
  ↓
Routes to: blog-service:80
  ↓
blog-service → blog-pods
```

---

### **pathType Options:**

**1. Prefix:**
```yaml
path: /api
pathType: Prefix

Matches:
  /api            ✓
  /api/          ✓
  /api/users     ✓
  /api/users/123 ✓

Doesn't match:
  /app           ✗
  /apiv2         ✗
```

**2. Exact:**
```yaml
path: /api
pathType: Exact

Matches:
  /api           ✓

Doesn't match:
  /api/          ✗
  /api/users     ✗
  /app           ✗
```

**3. ImplementationSpecific:**
```yaml
path: /api.*
pathType: ImplementationSpecific

# Behavior depends on Ingress Controller
# NGINX: Supports regex
# Others may not
```

---

## 🏠 Host-Based Routing

Route based on hostname.

### **Example:**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: host-based-ingress
spec:
  ingressClassName: nginx
  rules:
  # Route blog.example.com to blog-service
  - host: blog.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: blog-service
            port:
              number: 80
  
  # Route api.example.com to api-service
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
  
  # Route admin.example.com to admin-service
  - host: admin.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: admin-service
            port:
              number: 80
```

---

### **How it works:**

```
User requests:

http://blog.example.com/posts
  ↓
Ingress: "Host is blog.example.com"
  ↓
Routes to: blog-service:80

http://api.example.com/users
  ↓
Ingress: "Host is api.example.com"
  ↓
Routes to: api-service:8080

http://admin.example.com/dashboard
  ↓
Ingress: "Host is admin.example.com"
  ↓
Routes to: admin-service:80

Same ingress, different hosts
```

---

### **Combined Path + Host:**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: combined-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
  
  - host: admin.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: admin-service
            port:
              number: 80
```

**Routing:**
```
example.com/           → web-service
example.com/api        → api-service
admin.example.com/     → admin-service
```

---

## 🔒 TLS/HTTPS

Secure connections with HTTPS.

### **Creating TLS Secret:**

```bash
# Generate certificate (self-signed for testing)
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt \
  -subj "/CN=example.com"

# Create Kubernetes secret
kubectl create secret tls example-tls \
  --cert=tls.crt \
  --key=tls.key
```

---

### **Ingress with TLS:**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - example.com
    - www.example.com
    secretName: example-tls  # TLS secret
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

---

### **How TLS works:**

```
User → https://example.com
         ↓
  TLS handshake at Ingress
  (Certificate from example-tls secret)
         ↓
  Ingress terminates TLS
         ↓
  Plain HTTP to backend service
         ↓
  web-service → pods

TLS termination at ingress
Backend communication unencrypted (within cluster)
```

---

### **Multiple TLS Certificates:**

```yaml
spec:
  tls:
  - hosts:
    - example.com
    secretName: example-tls
  
  - hosts:
    - api.example.com
    secretName: api-tls
  
  - hosts:
    - blog.example.com
    secretName: blog-tls
  
  rules:
    # ... rules for each host
```

---

## 🏷️ Ingress Annotations

Controller-specific configuration.

### **Common NGINX Annotations:**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: annotated-ingress
  annotations:
    # Rewrite paths
    nginx.ingress.kubernetes.io/rewrite-target: /$2
    
    # SSL redirect
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    
    # Rate limiting
    nginx.ingress.kubernetes.io/limit-rps: "10"
    
    # CORS
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/cors-allow-origin: "*"
    
    # Custom timeouts
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "60"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "60"
    
    # Websocket support
    nginx.ingress.kubernetes.io/websocket-services: "chat-service"
    
    # Client body size
    nginx.ingress.kubernetes.io/proxy-body-size: "100m"
spec:
  # ... ingress spec
```

---

### **Rewrite Example:**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rewrite-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /api(/|$)(.*)
        pathType: ImplementationSpecific
        backend:
          service:
            name: api-service
            port:
              number: 8080
```

**How it works:**
```
User request: example.com/api/users
                            ↓
Regex match: /api(/|$)(.*)
  $1 = /
  $2 = users
                            ↓
Rewrite to: /$2 = /users
                            ↓
Backend receives: /users (not /api/users)
```

---

## 🎛️ Popular Ingress Controllers

### **1. NGINX Ingress Controller:**

```
Most popular
Mature and stable
Rich feature set
Good performance

Install:
  helm install ingress-nginx ingress-nginx/ingress-nginx

Use for:
  - General purpose
  - Production workloads
  - Complex routing
```

---

### **2. Traefik:**

```
Modern, cloud-native
Auto service discovery
Built-in dashboard
Easy configuration

Install:
  helm install traefik traefik/traefik

Use for:
  - Microservices
  - Dynamic environments
  - Need dashboard
```

---

### **3. Cloud-Specific:**

**AWS ALB Ingress Controller:**
```
Integrates with AWS ALB
Uses AWS features (WAF, Cognito)

Annotations:
  alb.ingress.kubernetes.io/scheme: internet-facing
  alb.ingress.kubernetes.io/target-type: ip
```

**Azure Application Gateway:**
```
Integrates with Azure App Gateway
Azure-native features

Annotations:
  appgw.ingress.kubernetes.io/backend-path-prefix: "/"
```

**GCE Ingress:**
```
Default on GKE
Google Cloud Load Balancer

Annotations:
  kubernetes.io/ingress.class: "gce"
```

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

