# Kubernetes Horizontal Pod Autoscaler (HPA) - Complete Theoretical Guide

**Understanding Automatic Scaling Based on Metrics**

---

## 📚 Table of Contents

1. [What is HPA?](#what-is-hpa)
2. [How HPA Works](#how-works)
3. [Metrics Types](#metrics-types)
4. [Creating HPA](#creating-hpa)
5. [Scaling Algorithm](#scaling-algorithm)
6. [Custom Metrics](#custom-metrics)
7. [Best Practices](#best-practices)

---

## 📈 What is HPA?

**Horizontal Pod Autoscaler (HPA)** automatically scales the number of pods based on observed metrics.

### **Simple Analogy:**

```
HPA = Restaurant Manager Adjusting Staff

Lunch rush (high traffic):
  Manager: "We need more waiters!"
  Adds 3 more waiters
  
Slow period (low traffic):
  Manager: "We have too many waiters!"
  Sends 2 waiters home

HPA monitors metrics (CPU, memory, requests)
Adds pods when busy
Removes pods when idle

Automatic, dynamic scaling!
```

---

### **Why HPA?**

```
Fixed replicas:
  spec:
    replicas: 3

Problems:
  - Peak time: 3 pods overwhelmed 😰
  - Off-peak: 3 pods wasting resources 💸
  - Manual intervention needed
  - Can't handle unexpected spikes

With HPA:
  Min: 2 pods (off-peak)
  Max: 10 pods (peak)
  Target: 70% CPU

Benefits:
  ✓ Automatic scaling
  ✓ Cost optimization
  ✓ Handle traffic spikes
  ✓ Maintain performance
```

---

## ⚙️ How HPA Works

### **Control Loop:**

```
┌─────────────────────────────────────┐
│  HPA Controller                      │
│  (Checks every 15 seconds)          │
└────────────────┬────────────────────┘
                 │
                 ↓
    ┌────────────────────────────┐
    │  1. Get current metrics    │
    │     (from metrics-server)  │
    └────────────┬───────────────┘
                 │
                 ↓
    ┌────────────────────────────┐
    │  2. Compare to target      │
    │     Current: 85% CPU       │
    │     Target: 70% CPU        │
    │     Too high!              │
    └────────────┬───────────────┘
                 │
                 ↓
    ┌────────────────────────────┐
    │  3. Calculate replicas     │
    │     Current: 5 pods        │
    │     Needed: 7 pods         │
    └────────────┬───────────────┘
                 │
                 ↓
    ┌────────────────────────────┐
    │  4. Scale deployment       │
    │     5 → 7 pods             │
    └────────────────────────────┘

Repeats every 15 seconds (default)
```

---

### **Prerequisites:**

**1. Metrics Server:**
```bash
# Install metrics-server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Verify
kubectl get deployment metrics-server -n kube-system

# Check metrics
kubectl top nodes
kubectl top pods
```

**2. Resource Requests:**
```yaml
# Pods MUST have resource requests for CPU-based HPA
spec:
  containers:
  - name: app
    resources:
      requests:
        cpu: 100m      # Required for HPA!
        memory: 128Mi
```

---

## 📊 Metrics Types

### **1. Resource Metrics (CPU, Memory):**

Most common, requires metrics-server.

**CPU-based:**
```yaml
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
```

**How it works:**
```
Target: 70% CPU
Current: 5 pods averaging 85% CPU

Too high! Scale up!
New replicas: 7 pods

After scaling:
7 pods averaging 65% CPU
Within target ✓
```

---

**Memory-based:**
```yaml
metrics:
- type: Resource
  resource:
    name: memory
    target:
      type: Utilization
      averageUtilization: 80  # Target 80% memory
```

---

### **2. Custom Metrics:**

Application-specific metrics (requests/sec, queue length, etc.)

Requires custom metrics adapter (Prometheus Adapter, Datadog, etc.)

```yaml
metrics:
- type: Pods
  pods:
    metric:
      name: http_requests_per_second
    target:
      type: AverageValue
      averageValue: "1000"  # 1000 requests/sec per pod
```

---

### **3. External Metrics:**

Metrics from external systems (cloud monitoring, APM, etc.)

```yaml
metrics:
- type: External
  external:
    metric:
      name: sqs_queue_length
      selector:
        matchLabels:
          queue: "orders"
    target:
      type: Value
      value: "30"  # Keep queue at 30 items
```

---

## 🏗️ Creating HPA

### **Method 1: kubectl autoscale:**

```bash
# Simple CPU-based autoscaling
kubectl autoscale deployment webapp \
  --cpu-percent=70 \
  --min=2 \
  --max=10

# Result:
horizontalpodautoscaler.autoscaling/webapp autoscaled
```

---

### **Method 2: YAML Manifest (v2):**

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: webapp-hpa
  namespace: production
spec:
  # Target deployment
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp
  
  # Min/max replicas
  minReplicas: 2
  maxReplicas: 20
  
  # Metrics
  metrics:
  # CPU metric
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  
  # Memory metric
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  
  # Scaling behavior (v2 only)
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300  # Wait 5 min before scale down
      policies:
      - type: Percent
        value: 50          # Max 50% of pods at once
        periodSeconds: 60  # Per minute
    scaleUp:
      stabilizationWindowSeconds: 0  # Scale up immediately
      policies:
      - type: Percent
        value: 100         # Max double pods at once
        periodSeconds: 60  # Per minute
```

---

### **Viewing HPA:**

```bash
# List HPAs
kubectl get hpa

NAME         REFERENCE           TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
webapp-hpa   Deployment/webapp   45%/70%   2         10        5          10m

# TARGETS: current/target
# 45%/70% = Currently 45% CPU, target is 70%

# Detailed view
kubectl describe hpa webapp-hpa

Name:                        webapp-hpa
Namespace:                   default
Reference:                   Deployment/webapp
Metrics:                     ( current / target )
  resource cpu on pods:      45% (45m) / 70%
Min replicas:                2
Max replicas:                10
Deployment pods:             5 current / 5 desired
Conditions:
  Type            Status  Reason              Message
  ----            ------  ------              -------
  AbleToScale     True    ReadyForNewScale    ready for scaling
  ScalingActive   True    ValidMetricFound    HPA able to fetch metrics
  ScalingLimited  False   DesiredWithinRange  desired count within range
Events:
  Type    Reason             Age   Message
  ----    ------             ----  -------
  Normal  SuccessfulRescale  2m    New size: 5; reason: cpu resource utilization above target
```

---

## 🧮 Scaling Algorithm

### **Calculation Formula:**

```
desiredReplicas = ceil[currentReplicas * (currentMetricValue / targetMetricValue)]
```

---

### **Example 1: Scale Up:**

```
Current state:
  Replicas: 5 pods
  Current CPU: 85% (average across all pods)
  Target CPU: 70%

Calculation:
  desiredReplicas = ceil[5 * (85 / 70)]
                  = ceil[5 * 1.214]
                  = ceil[6.07]
                  = 7 pods

Action: Scale up from 5 to 7 pods
```

---

### **Example 2: Scale Down:**

```
Current state:
  Replicas: 7 pods
  Current CPU: 40%
  Target CPU: 70%

Calculation:
  desiredReplicas = ceil[7 * (40 / 70)]
                  = ceil[7 * 0.571]
                  = ceil[4]
                  = 4 pods

Action: Scale down from 7 to 4 pods
```

---

### **Scaling Delays:**

```yaml
behavior:
  scaleDown:
    stabilizationWindowSeconds: 300  # Wait 5 min
  scaleUp:
    stabilizationWindowSeconds: 0    # Immediate
```

**Why delays?**
```
Without stabilization:
  T+0:  CPU 80% → Scale up to 7
  T+30s: CPU 60% → Scale down to 5
  T+60s: CPU 75% → Scale up to 6
  T+90s: CPU 55% → Scale down to 4
  
  Thrashing! Constant scaling up/down

With stabilization (5 min scale down):
  T+0:  CPU 80% → Scale up to 7 (immediate)
  T+30s: CPU 60% → Wait (don't scale down yet)
  T+2m:  CPU 60% → Wait
  T+5m:  CPU still 60% → Scale down to 5
  
  Stable! Prevents flapping
```

---

### **Scaling Rate Limits:**

```yaml
behavior:
  scaleUp:
    policies:
    - type: Pods
      value: 4              # Add max 4 pods
      periodSeconds: 60     # Per minute
    - type: Percent
      value: 100            # Or double pods
      periodSeconds: 60     # Per minute
    selectPolicy: Max       # Use whichever allows more

# Scenario:
Current: 3 pods
CPU spike!

Minute 1:
  Policy 1: Add 4 pods → 7 total
  Policy 2: Double → 6 total
  Max policy: Choose 7 ✓

Minute 2:
  Current: 7 pods
  Still high CPU
  Policy 1: Add 4 → 11 total
  Policy 2: Double → 14 total
  Max policy: Choose 14 ✓
```

---

## 🎛️ Custom Metrics

### **Requests Per Second Example:**

**1. Application exports metrics:**
```go
// Application code
http_requests_total.Inc()
```

**2. Prometheus scrapes metrics**

**3. Install Prometheus Adapter:**
```bash
helm install prometheus-adapter prometheus-community/prometheus-adapter
```

**4. Configure adapter:**
```yaml
rules:
- seriesQuery: 'http_requests_total'
  resources:
    overrides:
      namespace: {resource: "namespace"}
      pod: {resource: "pod"}
  name:
    matches: "^(.*)_total$"
    as: "${1}_per_second"
  metricsQuery: 'rate(<<.Series>>{<<.LabelMatchers>>}[1m])'
```

**5. Create HPA:**
```yaml
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
  maxReplicas: 20
  metrics:
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "1000"  # 1000 req/s per pod

```

**How it works:**
```
Target: 1000 requests/sec per pod
Current: 5 pods handling 8000 requests/sec total
Per pod: 8000 / 5 = 1600 requests/sec

Too high! Scale up!
desiredReplicas = ceil[5 * (1600 / 1000)]
                = ceil[8]
                = 8 pods

After scaling:
8 pods handling 8000 requests/sec
Per pod: 1000 requests/sec ✓
```

---

## ✅ Best Practices

### **1. Set Appropriate Targets:**

```yaml
# ✗ Too low (constant scaling)
averageUtilization: 30  # Pods almost always over

# ✗ Too high (poor performance)
averageUtilization: 95  # No headroom for spikes

# ✓ Good balance
averageUtilization: 70  # Comfortable target with headroom
```

---

### **2. Always Set Min/Max:**

```yaml
spec:
  minReplicas: 2    # Never below 2 (availability)
  maxReplicas: 50   # Never above 50 (cost control)

# Without max: Runaway scaling $$$$
# Without min: Scale to 0 (downtime)
```

---

### **3. Resource Requests Required:**

```yaml
# ✓ Good: Has requests
spec:
  containers:
  - name: app
    resources:
      requests:
        cpu: 100m    # HPA needs this!
      limits:
        cpu: 200m

# ✗ Bad: No requests
spec:
  containers:
  - name: app
    # No resources
    # HPA won't work!
```

---

### **4. Combine Multiple Metrics:**

```yaml
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

# Scales if EITHER metric exceeds target
# Uses highest desired replicas
```

---

### **5. Tune Scaling Behavior:**

```yaml
behavior:
  scaleDown:
    stabilizationWindowSeconds: 300  # Conservative scale down
    policies:
    - type: Percent
      value: 50                      # Max 50% reduction
      periodSeconds: 60
  
  scaleUp:
    stabilizationWindowSeconds: 0    # Aggressive scale up
    policies:
    - type: Percent
      value: 100                     # Can double
      periodSeconds: 15              # Every 15 seconds
```

---

### **6. Monitor HPA:**

```bash
# Watch HPA in real-time
kubectl get hpa webapp-hpa --watch

# View events
kubectl describe hpa webapp-hpa

# Check metrics
kubectl top pods -l app=webapp
```

---

### **7. Test Scaling:**

```bash
# Generate load
kubectl run -it --rm load-generator --image=busybox -- /bin/sh
while true; do wget -q -O- http://webapp; done

# Watch scaling
kubectl get hpa webapp-hpa --watch
kubectl get deployment webapp --watch

# Observe:
# Replicas increase as CPU rises
# Replicas decrease when load stops
```

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

