# Kubernetes Deployment Strategies - Complete Guide

## Table of Contents
1. [Overview](#overview)
2. [Rolling Update Strategy](#rolling-update)
3. [Recreate Strategy](#recreate)
4. [Blue-Green Deployment](#blue-green)
5. [Canary Deployment](#canary)
6. [A/B Testing](#ab-testing)
7. [Shadow Deployment](#shadow)
8. [Production Best Practices](#best-practices)

---

## Overview

Kubernetes deployment strategies determine how applications are updated in production. Choosing the right strategy balances risk, downtime, and rollback capabilities.

### Strategy Comparison

| Strategy | Downtime | Risk | Rollback Speed | Resource Cost | Complexity |
|----------|----------|------|----------------|---------------|------------|
| Rolling Update | None | Low | Fast | Low | Low |
| Recreate | Yes | Medium | Medium | Low | Very Low |
| Blue-Green | None | Low | Instant | High (2x) | Medium |
| Canary | None | Very Low | Fast | Medium | High |
| A/B Testing | None | Low | Fast | Medium | High |
| Shadow | None | Very Low | N/A | High | Very High |

---

## Rolling Update Strategy

**Default Kubernetes strategy. Gradually replaces old pods with new ones.**

### How It Works
1. Create new pods with updated version
2. Wait for new pods to be ready (readiness probe)
3. Terminate old pods
4. Repeat until all pods updated

### Configuration

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 10
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%  # Max 25% pods can be unavailable
      maxSurge: 25%        # Max 25% extra pods during update
  template:
    metadata:
      labels:
        app: myapp
        version: v2.0
    spec:
      containers:
      - name: app
        image: myapp:v2.0
        ports:
        - containerPort: 8080
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
```

### Key Parameters

**maxUnavailable:**
- Absolute number (e.g., 2) or percentage (e.g., 25%)
- Ensures minimum availability during update
- Lower = slower rollout, higher availability

**maxSurge:**
- Maximum extra pods created during update
- Higher = faster rollout, more resources needed
- 0 = no extra pods, only replacement

### Example Scenarios

```bash
# 10 replicas, maxUnavailable=2, maxSurge=3

# Step 1: Create 3 new pods (surge)
# Running: 10 old + 3 new = 13 total

# Step 2: Once new pods ready, terminate 2 old
# Running: 8 old + 3 new = 11 total

# Step 3: Create 2 more new pods
# Running: 8 old + 5 new = 13 total

# Continues until all updated
```

### Rollback

```bash
# View rollout history
kubectl rollout history deployment/myapp

# Rollback to previous revision
kubectl rollout undo deployment/myapp

# Rollback to specific revision
kubectl rollout undo deployment/myapp --to-revision=3

# Check rollout status
kubectl rollout status deployment/myapp
```

### Pros & Cons

**Pros:**
- Zero downtime
- Gradual rollout reduces blast radius
- Built-in Kubernetes feature
- Easy rollback

**Cons:**
- Both versions run simultaneously (compatibility needed)
- Can't control which users get new version
- Takes time to complete

---

## Recreate Strategy

**Terminates all old pods before creating new ones. Has downtime but simpler.**

### Configuration

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 5
  strategy:
    type: Recreate  # Terminate all, then create new
  template:
    spec:
      containers:
      - name: app
        image: myapp:v2.0
```

### When to Use
- Development/test environments
- Applications that can't run multiple versions
- Database schema migrations requiring specific order
- Cost-sensitive scenarios (no extra pods)

### Pros & Cons

**Pros:**
- Simple
- No version compatibility concerns
- Lower resource usage

**Cons:**
- Downtime during deployment
- All-or-nothing (high risk)
- Slower rollback

---

## Blue-Green Deployment

**Two identical environments. Switch traffic instantly between them.**

### Architecture

```
┌─────────────┐
│   Ingress   │
│  /Service   │
└──────┬──────┘
       │
       ├──[Switch]──┐
       │            │
   ┌───▼───┐   ┌───▼───┐
   │ Blue  │   │ Green │
   │  v1.0 │   │  v2.0 │
   │ 5 pods│   │ 5 pods│
   └───────┘   └───────┘
```

### Implementation

```yaml
# Blue Deployment (Current Production - v1.0)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-blue
  labels:
    app: myapp
    version: blue
spec:
  replicas: 5
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
        image: myapp:v1.0
        ports:
        - containerPort: 8080

---
# Green Deployment (New Version - v2.0)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-green
  labels:
    app: myapp
    version: green
spec:
  replicas: 5
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
        image: myapp:v2.0
        ports:
        - containerPort: 8080

---
# Service - Traffic Router
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
    version: blue  # Switch to 'green' to cut over
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
```

### Deployment Process

```bash
# Step 1: Deploy green environment
kubectl apply -f deployment-green.yaml

# Step 2: Wait for green pods ready
kubectl wait --for=condition=available deployment/myapp-green --timeout=300s

# Step 3: Test green environment directly
kubectl port-forward deployment/myapp-green 8080:8080
# Test: http://localhost:8080

# Step 4: Switch traffic to green
kubectl patch service myapp-service -p '{"spec":{"selector":{"version":"green"}}}'

# Traffic instantly switched to green

# Step 5: Monitor for issues
# If problems, instant rollback:
kubectl patch service myapp-service -p '{"spec":{"selector":{"version":"blue"}}}'

# Step 6: Once confident, delete blue
kubectl delete deployment myapp-blue
```

### Pros & Cons

**Pros:**
- Instant cutover and rollback
- Test production environment before switch
- Zero downtime
- Simple rollback (just switch back)

**Cons:**
- Requires 2x resources
- Database migrations complex (shared state)
- Cost (two full environments)

---

## Canary Deployment

**Deploy new version to small subset of users. Gradually increase if healthy.**

### Architecture

```
         ┌─────────┐
         │ Service │
         └────┬────┘
              │
       ┌──────┴──────┐
       │             │
   ┌───▼──┐      ┌──▼───┐
   │Stable│ 90%  │Canary│ 10%
   │ v1.0 │      │ v2.0 │
   │9 pods│      │1 pod │
   └──────┘      └──────┘
```

### Implementation (Basic)

```yaml
# Stable Deployment - 9 replicas (90%)
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
        version: v1.0
    spec:
      containers:
      - name: app
        image: myapp:v1.0

---
# Canary Deployment - 1 replica (10%)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-canary
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
      track: canary
  template:
    metadata:
      labels:
        app: myapp
        track: canary
        version: v2.0
    spec:
      containers:
      - name: app
        image: myapp:v2.0

---
# Service routes to both
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp  # Matches both stable and canary
  ports:
  - port: 80
    targetPort: 8080
```

### Progressive Rollout

```bash
# Phase 1: 10% canary
kubectl scale deployment myapp-canary --replicas=1
kubectl scale deployment myapp-stable --replicas=9

# Monitor metrics for 15 minutes
# If healthy, proceed

# Phase 2: 25% canary
kubectl scale deployment myapp-canary --replicas=3
kubectl scale deployment myapp-stable --replicas=7

# Monitor

# Phase 3: 50% canary
kubectl scale deployment myapp-canary --replicas=5
kubectl scale deployment myapp-stable --replicas=5

# Phase 4: 100% canary
kubectl scale deployment myapp-canary --replicas=10
kubectl scale deployment myapp-stable --replicas=0

# Delete stable deployment
kubectl delete deployment myapp-stable
```

### Metrics to Monitor

```yaml
# Prometheus queries for canary validation

# Error rate
sum(rate(http_requests_total{status=~"5..",version="v2.0"}[5m])) 
  / 
sum(rate(http_requests_total{version="v2.0"}[5m]))

# Latency p95
histogram_quantile(0.95, 
  rate(http_request_duration_seconds_bucket{version="v2.0"}[5m]))

# Compare canary vs stable
rate(http_requests_total{version="v2.0",status=~"5.."}[5m]) 
  >
rate(http_requests_total{version="v1.0",status=~"5.."}[5m]) * 1.5
```

### Automated Canary with Flagger

```yaml
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: myapp
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  service:
    port: 80
  analysis:
    interval: 1m
    threshold: 5
    maxWeight: 50
    stepWeight: 10
    metrics:
    - name: request-success-rate
      thresholdRange:
        min: 99
      interval: 1m
    - name: request-duration
      thresholdRange:
        max: 500
      interval: 1m
```

---

## A/B Testing

**Route specific users to new version for feature testing.**

### Implementation with Istio

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapp
spec:
  hosts:
  - myapp
  http:
  # Route premium users to v2.0
  - match:
    - headers:
        user-tier:
          exact: premium
    route:
    - destination:
        host: myapp
        subset: v2
      weight: 100
  
  # Route Chrome users to v2.0  
  - match:
    - headers:
        user-agent:
          regex: ".*Chrome.*"
    route:
    - destination:
        host: myapp
        subset: v2
      weight: 100
  
  # Everyone else to v1.0
  - route:
    - destination:
        host: myapp
        subset: v1
      weight: 100

---
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: myapp
spec:
  host: myapp
  subsets:
  - name: v1
    labels:
      version: v1.0
  - name: v2
    labels:
      version: v2.0
```

---

## Shadow Deployment

**Mirror production traffic to new version without affecting users.**

### Implementation with Istio

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapp
spec:
  hosts:
  - myapp
  http:
  - route:
    - destination:
        host: myapp
        subset: stable
      weight: 100
    mirror:
      host: myapp
      subset: shadow
    mirrorPercentage:
      value: 100.0  # Mirror 100% of traffic
```

**Shadow version receives traffic copy but responses are discarded. Users only see stable version responses.**

---

## Production Best Practices

### 1. Always Use Readiness Probes

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
  failureThreshold: 3
  successThreshold: 1
```

### 2. Implement Health Checks

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
```

### 3. Use Pod Disruption Budgets

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: myapp-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: myapp
```

### 4. Monitor Key Metrics

- Request rate
- Error rate
- Latency (p50, p95, p99)
- Resource usage
- Pod restart count

### 5. Automated Rollback

```yaml
# Set deployment deadline
spec:
  progressDeadlineSeconds: 600  # 10 minutes
  # Automatically rolls back if deadline exceeded
```

### 6. Version Everything

```yaml
metadata:
  labels:
    app: myapp
    version: v2.0.5
    commit: abc1234
```

### 7. Test Rollback Procedures

```bash
# Practice rollback regularly
kubectl rollout undo deployment/myapp
```

---

## Choosing the Right Strategy

### Quick Decision Matrix

**Use Rolling Update when:**
- Need zero downtime
- Can tolerate both versions running
- Standard stateless applications
- ✅ Most common choice

**Use Recreate when:**
- Downtime acceptable
- Can't run multiple versions
- Development environments
- Cost optimization priority

**Use Blue-Green when:**
- Need instant rollback
- Can afford 2x resources
- Want to test in production
- High-risk deployments

**Use Canary when:**
- Want gradual rollout
- Have good monitoring
- Need to limit blast radius
- User-facing changes

**Use A/B Testing when:**
- Testing specific features
- Want targeted user groups
- Have service mesh
- Measuring feature impact

**Use Shadow when:**
- Testing major changes
- Want zero-risk validation
- Need production-like testing
- Have spare capacity

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

---

**✅ Complete Kubernetes Deployment Strategies Guide!**

