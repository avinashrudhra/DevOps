# DevOps Interview Session 8
## Kubernetes Deployment Strategies & Advanced Concepts (3-7 Years)

---

### Question 1: Kubernetes Deployment Strategies Overview

**Interviewer:** Explain different deployment strategies in Kubernetes.

**Candidate:** Six main strategies:
1. **Recreate** - terminate all old pods, then create new ones. Downtime but simple.
2. **Rolling Update** - gradually replace pods. Default strategy, zero downtime.
3. **Blue-Green** - two identical environments, switch traffic. Instant rollback.
4. **Canary** - deploy to small subset, gradually increase. Test with real traffic.
5. **A/B Testing** - route specific users to new version. Feature testing.
6. **Shadow** - mirror traffic to new version without affecting users. Risk-free testing.

---

### Question 2: Rolling Update Strategy

**Interviewer:** Implement rolling update with specific controls.

**Candidate:**
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
      maxUnavailable: 2  # Max 2 pods down during update
      maxSurge: 3        # Max 3 extra pods during update
  template:
    spec:
      containers:
      - name: app
        image: myapp:v2.0
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
```

**Interviewer:** How does Kubernetes decide update speed?

**Candidate:** Based on maxUnavailable and maxSurge. With 10 replicas, maxUnavailable=2 means minimum 8 must be available. maxSurge=3 means maximum 13 total pods during update. Kubernetes terminates 2 old, starts 3 new, waits for readiness, repeats.

---

### Question 3: Blue-Green Deployment in Kubernetes

**Interviewer:** Implement complete blue-green deployment.

**Candidate:**
```yaml
# Blue deployment (current production)
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

---
# Green deployment (new version)
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

---
# Service - switch by changing selector
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
    version: blue  # Change to 'green' to switch traffic
  ports:
  - port: 80
    targetPort: 8080
```

**Interviewer:** Instant rollback?

**Candidate:** Just change service selector back to `version: blue`. Traffic switches immediately. Keep blue deployment running for quick rollback.

---

### Question 4: Canary Deployment Implementation

**Interviewer:** Deploy canary with 10% traffic to new version.

**Candidate:**
```yaml
# Stable - 9 replicas (90%)
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
    spec:
      containers:
      - name: app
        image: myapp:v1.0

---
# Canary - 1 replica (10%)
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
```

**Interviewer:** How do you gradually increase canary percentage?

**Candidate:** Increase canary replicas, decrease stable replicas proportionally. 10% → 25% → 50% → 100%. Monitor metrics at each stage. If errors spike, rollback by scaling canary to 0.

---

### Question 5: Istio for Advanced Traffic Management

**Interviewer:** Use service mesh for percentage-based canary.

**Candidate:**
```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapp
spec:
  hosts:
  - myapp
  http:
  - match:
    - headers:
        user-agent:
          regex: ".*Chrome.*"  # Chrome users get canary
    route:
    - destination:
        host: myapp
        subset: canary
      weight: 100
  - route:
    - destination:
        host: myapp
        subset: stable
      weight: 90
    - destination:
        host: myapp
        subset: canary
      weight: 10  # 10% traffic to canary

---
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: myapp
spec:
  host: myapp
  subsets:
  - name: stable
    labels:
      version: v1.0
  - name: canary
    labels:
      version: v2.0
```

---

### Question 6: Recreate Strategy Use Case

**Interviewer:** When would you use Recreate strategy?

**Candidate:** When application can't run multiple versions simultaneously - like stateful apps with database schema changes, or development environments where downtime is acceptable. Simple but has downtime.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: legacy-app
spec:
  replicas: 1
  strategy:
    type: Recreate  # All pods terminated before new ones created
  template:
    spec:
      containers:
      - name: app
        image: legacy-app:v2.0
```

---

### Question 7-40: Rapid Fire Deployment & K8s Questions

**Q7:** DaemonSet deployment strategy?
**A:** No strategy - updates all nodes immediately. Can use `updateStrategy: RollingUpdate` with `maxUnavailable` for controlled updates.

**Q8:** StatefulSet deployment order?
**A:** Sequential - 0, 1, 2. Waits for each pod ready before starting next. Reverse order for termination.

**Q9:** Deployment rollback command?
**A:** `kubectl rollout undo deployment/myapp` or `kubectl rollout undo deployment/myapp --to-revision=3`

**Q10:** Check rollout status?
**A:** `kubectl rollout status deployment/myapp` - waits until rollout complete.

**Q11:** Pause deployment mid-rollout?
**A:** `kubectl rollout pause deployment/myapp` - useful for canary validation before continuing.

**Q12:** Job deployment strategy?
**A:** Jobs don't have strategies - run once to completion. Use `ttlSecondsAfterFinished` for cleanup.

**Q13:** CronJob deployment?
**A:** No deployment - scheduled execution. Update with `kubectl apply -f cronjob.yaml`.

**Q14:** Progressive delivery vs continuous delivery?
**A:** Progressive = gradual rollout with automated gates. Continuous = deploy frequently. Progressive adds safety layers.

**Q15:** Feature flags in deployments?
**A:** Deploy code, features off by default. Enable gradually via config. Decouples deployment from release.

**Q16:** Shadow deployment benefit?
**A:** Test new version with production traffic copy. Zero risk - responses discarded. Finds production-only issues.

**Q17:** Traffic mirroring in Istio?
**A:** `mirror:` directive sends copy of traffic to shadow version. Original responses returned to users.

**Q18:** Deployment annotations for tracking?
**A:** `kubernetes.io/change-cause` shows in rollout history. Set with `--record` flag.

**Q19:** Init containers in deployments?
**A:** Run before main containers. Use for migrations, setup. Block deployment if init fails.

**Q20:** Pod Disruption Budget with deployments?
**A:** Ensures minimum replicas during voluntary disruptions. Prevents all pods terminating during updates.

**Q21:** HPA behavior during deployments?
**A:** HPA continues operating. Can scale deployment being updated. Set appropriate min/max replicas.

**Q22:** Resource requests impact on rollout?
**A:** If cluster can't schedule new pods (insufficient resources), rollout blocks. Monitor node capacity.

**Q23:** Liveness vs readiness in deployments?
**A:** Readiness blocks traffic during updates. Liveness restarts failed pods. Both critical for zero-downtime.

**Q24:** Image pull policy impact?
**A:** `Always` ensures new image fetched. `IfNotPresent` can use stale cache. Use `Always` with `:latest`.

**Q25:** Deployment revision history?
**A:** `kubectl rollout history deployment/myapp` shows all revisions. Configurable with `revisionHistoryLimit`.

**Q26:** ConfigMap update propagation?
**A:** Not automatic - restart pods or use reloader tool. Or mount as volume (syncs every ~60s).

**Q27:** Secret rotation during deployment?
**A:** Use external secrets operator or update secret + rolling restart deployment.

**Q28:** Multi-cluster deployments?
**A:** Use ArgoCD ApplicationSet or Fleet. Deploy same config to multiple clusters simultaneously.

**Q29:** Namespace-based deployments?
**A:** Isolate environments by namespace. Same manifests, different namespaces for dev/staging/prod.

**Q30:** Affinity rules in deployments?
**A:** `podAntiAffinity` spreads replicas across nodes/zones. Improves availability during updates.

**Q31:** Sidecar container updates?
**A:** Update deployment - all containers updated together. Can't update sidecar independently.

**Q32:** Volume mount updates?
**A:** Config changes trigger rolling update. PVC changes require pod recreation.

**Q33:** Deployment selector immutability?
**A:** Can't change `spec.selector` - must delete and recreate deployment. Labels are immutable.

**Q34:** Progressive rollout monitoring?
**A:** Watch error rates, latency, resource usage during rollout. Automated rollback if thresholds exceeded.

**Q35:** Deployment vs ReplicaSet difference?
**A:** Deployment manages ReplicaSets, provides rollout/rollback. ReplicaSet just maintains pod count.

**Q36:** Kubectl wait for deployment?
**A:** `kubectl wait --for=condition=available deployment/myapp --timeout=300s`

**Q37:** Deployment strategy for stateful apps?
**A:** Use StatefulSet with rolling updates and careful ordering. Or external coordination (Consul, etcd).

**Q38:** Pre-stop hooks in deployments?
**A:** `preStop` hook allows graceful shutdown. Drain connections before termination.

**Q39:** Readiness gate advanced use?
**A:** Custom readiness conditions beyond probes. External system validates readiness.

**Q40:** Deployment troubleshooting steps?
**A:** Check events, describe deployment, check pod status, review logs, verify image exists, check resources.

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

---

**✅ Session 8 Complete - K8s Deployment Strategies!**
**Total Questions: 40/40** | **Format:** Deployment Focus

