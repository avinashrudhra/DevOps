# DevOps Interview Session 11
## Monitoring, Observability & Performance (3-7 Years)

---

### Question 1: Prometheus Setup for Kubernetes

**Interviewer:** Set up Prometheus monitoring for a Kubernetes cluster.

**Candidate:** I use Helm to deploy kube-prometheus-stack which includes Prometheus, Grafana, and Alertmanager.

```bash
# Add Helm repo
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install with custom values
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --set prometheus.prometheusSpec.retention=15d \
  --set prometheus.prometheusSpec.storageSpec.volumeClaimTemplate.spec.resources.requests.storage=50Gi \
  --set grafana.adminPassword='SecurePass123!'
```

**Interviewer:** What metrics do you monitor?

**Candidate:** Node metrics (CPU, memory, disk), pod metrics (restarts, OOM kills), application metrics (request rate, errors, duration - RED method), cluster metrics (API server latency, etcd health).

---

### Question 2: Application Insights Integration

**Interviewer:** Integrate Application Insights with a Node.js app.

**Candidate:**
```javascript
// app.js
const appInsights = require('applicationinsights');
appInsights.setup(process.env.APPINSIGHTS_INSTRUMENTATION_KEY)
  .setAutoDependencyCorrelation(true)
  .setAutoCollectRequests(true)
  .setAutoCollectPerformance(true)
  .setAutoCollectExceptions(true)
  .setAutoCollectDependencies(true)
  .setUseDiskRetryCaching(true)
  .start();

const client = appInsights.defaultClient;

// Custom metrics
app.use((req, res, next) => {
  const start = Date.now();
  res.on('finish', () => {
    const duration = Date.now() - start;
    client.trackMetric({
      name: 'HTTP Request Duration',
      value: duration,
      properties: {
        path: req.path,
        method: req.method,
        statusCode: res.statusCode
      }
    });
  });
  next();
});

// Custom events
client.trackEvent({ name: 'UserLogin', properties: { userId: '123' }});
```

---

### Question 3: KQL Queries for Troubleshooting

**Interviewer:** Write KQL query to find slow API requests.

**Candidate:**
```kusto
// Requests over 2 seconds in last 24 hours
requests
| where timestamp > ago(24h)
| where duration > 2000
| project timestamp, name, url, duration, resultCode
| order by duration desc
| take 100

// Top 10 slowest operations
requests
| where timestamp > ago(1h)
| summarize 
    Count=count(),
    AvgDuration=avg(duration),
    P95Duration=percentile(duration, 95),
    P99Duration=percentile(duration, 99)
    by name
| order by P99Duration desc
| take 10

// Error rate by endpoint
requests
| where timestamp > ago(1h)
| summarize 
    Total=count(),
    Errors=countif(success == false)
    by name
| extend ErrorRate = (Errors * 100.0) / Total
| where ErrorRate > 1
| order by ErrorRate desc
```

---

### Question 4: Grafana Dashboard Design

**Interviewer:** Design a dashboard for API monitoring.

**Candidate:** Four main sections:
1. **Golden Signals** (top row)
   - Request rate (RPS)
   - Error rate (%)
   - Latency (p50, p95, p99)
   - Saturation (CPU/Memory)

2. **Detailed Metrics** (middle)
   - Requests by endpoint
   - Status codes distribution
   - Database query time
   - Cache hit rate

3. **Infrastructure** (bottom)
   - Pod count and health
   - Resource usage
   - Network I/O

4. **Alerts** (sidebar)
   - Active alerts
   - Recent incidents

```json
// Prometheus query for error rate
(sum(rate(http_requests_total{status=~"5.."}[5m])) 
/ 
sum(rate(http_requests_total[5m]))) * 100
```

---

### Question 5-40: Monitoring & Performance Questions

**Q5:** SLI vs SLO vs SLA?
**A:** SLI = metric (latency), SLO = target (p95 < 200ms), SLA = contract (99.9% uptime or refund).

**Q6:** Alert fatigue prevention?
**A:** Meaningful thresholds, group related alerts, silence known issues, on-call runbooks, escalation policies.

**Q7:** Distributed tracing setup?
**A:** Jaeger or Zipkin, instrument code with OpenTelemetry, correlation IDs across services, visualize request flow.

**Q8:** Log aggregation strategy?
**A:** Centralized logging (ELK/Azure Monitor), structured logs (JSON), log levels, retention policies, indexing.

**Q9:** Metric types in Prometheus?
**A:** Counter (incremental), Gauge (current value), Histogram (distribution), Summary (quantiles).

**Q10:** Alertmanager configuration?
**A:** Route alerts by severity, group by cluster, inhibit dependent alerts, notification channels (Slack/PagerDuty).

**Q11:** Custom metrics in Kubernetes?
**A:** Expose /metrics endpoint, ServiceMonitor CRD, metrics-server for HPA, custom metrics API.

**Q12:** Performance profiling tools?
**A:** Application Insights Profiler, Node.js --inspect, Python cProfile, continuous profiling in production.

**Q13:** Load testing strategy?
**A:** k6 or JMeter, ramp-up pattern, sustained load, spike testing, stress testing until failure.

**Q14:** Capacity planning approach?
**A:** Historical trends, growth projections, peak load analysis, resource headroom (20-30%), cost optimization.

**Q15:** Database monitoring?
**A:** Query performance, slow queries, connection pool, deadlocks, replication lag, index usage.

**Q16:** APM tool selection?
**A:** Application Insights for Azure, New Relic, Datadog. Auto-instrumentation, distributed tracing, alerting.

**Q17:** On-call best practices?
**A:** Rotation schedule, escalation path, runbooks, blameless culture, post-incident reviews, compensatory time off.

**Q18:** Incident severity levels?
**A:** P0 (outage), P1 (major impact), P2 (minor impact), P3 (cosmetic). Response time SLAs per level.

**Q19:** Post-mortem template?
**A:** Timeline, impact, root cause, resolution, action items, lessons learned. Blameless, focus on systems.

**Q20:** Uptime calculation?
**A:** (Total time - Downtime) / Total time * 100. 99.9% = 8.76 hours/year downtime.

**Q21:** Health check implementation?
**A:** /health endpoint, check dependencies (DB, cache), timeout quickly, K8s liveness/readiness probes.

**Q22:** Rate limiting monitoring?
**A:** Track rejected requests, per-client metrics, DDoS detection, automatic throttling.

**Q23:** Cost monitoring dashboards?
**A:** Azure Cost Management, resource tagging, showback/chargeback, budget alerts, optimization recommendations.

**Q24:** Black-box vs white-box monitoring?
**A:** Black-box = external probes (uptime checks). White-box = internal metrics (logs, traces).

**Q25:** Service mesh observability?
**A:** Istio provides automatic metrics, Jaeger tracing, Kiali visualization, Grafana dashboards.

**Q26:** Anomaly detection?
**A:** Machine learning models, baseline behavior, statistical thresholds, Azure Monitor smart alerts.

**Q27:** Synthetic monitoring?
**A:** Azure Monitor availability tests, Pingdom, simulated user journeys, multi-region checks.

**Q28:** Log retention policies?
**A:** Hot (7 days, expensive), warm (30 days, archive), cold (365 days, compliance).

**Q29:** Real User Monitoring (RUM)?
**A:** Client-side JavaScript, page load times, user interactions, geographic distribution.

**Q30:** Alerting on logs?
**A:** KQL scheduled queries, threshold triggers, log-based metrics, anomaly detection.

**Q31:** Dashboard sharing strategy?
**A:** Team dashboards, executive summaries, public status page, embed in wiki.

**Q32:** Metric cardinality issues?
**A:** Too many label combinations, Prometheus memory issues, use recording rules, limit labels.

**Q33:** Continuous profiling?
**A:** Always-on profiling in production, flame graphs, identify bottlenecks, compare before/after deployments.

**Q34:** Chaos engineering monitoring?
**A:** Inject failures, observe system behavior, validate alerts fire, measure MTTR.

**Q35:** Multi-cluster monitoring?
**A:** Centralized Prometheus, Thanos for long-term storage, federated queries, cross-cluster dashboards.

**Q36:** Logging best practices?
**A:** Structured JSON, correlation IDs, appropriate log levels, PII masking, contextual information.

**Q37:** Error budget tracking?
**A:** (1 - SLO) * time period = allowed downtime. Track consumption, gate deployments if budget exhausted.

**Q38:** Network performance monitoring?
**A:** Connection tracking, latency between services, packet loss, bandwidth utilization.

**Q39:** Storage monitoring?
**A:** Disk usage, IOPS, throughput, latency, PVC status, volume expansion.

**Q40:** Observability vs monitoring?
**A:** Monitoring = known unknowns (dashboards, alerts). Observability = unknown unknowns (explore, debug novel issues).

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

---

**✅ Session 11 Complete - Monitoring & Observability!**

