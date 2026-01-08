# Prometheus & Grafana Troubleshooting Guide

**Solutions to common issues in production monitoring environments**

---

## 🎯 Troubleshooting Strategy

```
1. Identify the problem
2. Check logs
3. Verify configuration
4. Test connectivity
5. Review metrics
6. Apply fix
7. Validate solution
```

---

## 🔥 Prometheus Issues

### **Issue 1: Prometheus Not Starting**

**Symptoms:**
```bash
level=error ts=2026-01-07T10:00:00.000Z caller=main.go:123 err="opening storage failed"
```

**Root Causes & Solutions:**

**Cause 1: Configuration Error**
```bash
# Check configuration
promtool check config prometheus.yml

# Common errors:
# - Invalid YAML syntax
# - Missing required fields
# - Invalid regex patterns

# Fix example:
# Bad
scrape_configs:
  - job_name: 'app'
    static_configs
      - targets: ['localhost:8080']

# Good
scrape_configs:
  - job_name: 'app'
    static_configs:
      - targets: ['localhost:8080']
```

**Cause 2: Storage Permission Issues**
```bash
# Check permissions
ls -la /prometheus

# Fix permissions
chown -R 65534:65534 /prometheus
chmod -R 755 /prometheus

# Docker volume fix
docker-compose down
docker volume rm prometheus_data
docker-compose up -d
```

**Cause 3: Corrupted Data**
```bash
# Check TSDB integrity
promtool tsdb analyze /prometheus

# Recover from corruption
promtool tsdb create-blocks-from openmetrics /path/to/backup /prometheus
```

**Prevention:**
- Always validate config before deployment
- Use version control for configurations
- Regular backups of TSDB
- Monitor disk space

---

### **Issue 2: High Memory Usage**

**Symptoms:**
- Prometheus OOM killed
- Memory usage growing continuously
- Slow query performance

**Diagnosis:**
```bash
# Check memory usage
curl http://localhost:9090/api/v1/status/tsdb

# Check cardinality
curl http://localhost:9090/api/v1/label/__name__/values | jq '. | length'

# Find high cardinality metrics
curl http://localhost:9090/api/v1/status/tsdb | jq '.data.seriesCountByMetricName'
```

**Solutions:**

**Solution 1: Reduce Retention**
```yaml
# prometheus.yml
storage:
  tsdb:
    retention.time: 15d  # Reduce from 30d
    retention.size: 50GB
```

**Solution 2: Drop High Cardinality Metrics**
```yaml
scrape_configs:
  - job_name: 'app'
    static_configs:
      - targets: ['app:8080']
    metric_relabel_configs:
      - source_labels: [__name__]
        regex: 'high_cardinality_metric.*'
        action: drop
```

**Solution 3: Increase Resources**
```yaml
# docker-compose.yml
services:
  prometheus:
    deploy:
      resources:
        limits:
          memory: 8G
        reservations:
          memory: 4G
```

**Solution 4: Use Recording Rules**
```yaml
# Replace expensive queries with pre-aggregated rules
groups:
  - name: aggregation
    rules:
      - record: job:api_requests:rate5m
        expr: sum by (job) (rate(api_requests_total[5m]))
```

**Prevention:**
- Monitor metric cardinality
- Limit label values
- Use recording rules for expensive queries
- Regular capacity planning

---

### **Issue 3: Targets Not Scraped**

**Symptoms:**
- Target shows as "DOWN" in Prometheus UI
- No metrics from specific job
- `up{job="xxx"}` returns 0

**Diagnosis:**
```bash
# Check target status
curl http://localhost:9090/api/v1/targets | jq '.data.activeTargets[] | select(.health != "up")'

# Check Prometheus logs
docker logs prometheus 2>&1 | grep -i error

# Test connectivity
curl http://target:port/metrics

# Check DNS resolution
nslookup target-hostname

# Test from Prometheus container
docker exec prometheus wget -O- http://target:port/metrics
```

**Solutions:**

**Solution 1: Network Issues**
```bash
# Docker network
docker network ls
docker network inspect monitoring

# Verify target is on same network
docker inspect target-container | grep NetworkMode

# Fix: Add to correct network
docker network connect monitoring target-container
```

**Solution 2: Authentication Required**
```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'secure-app'
    static_configs:
      - targets: ['app:8080']
    basic_auth:
      username: prometheus
      password: secret
```

**Solution 3: Wrong Port/Path**
```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'app'
    static_configs:
      - targets: ['app:8080']
    metrics_path: '/actuator/prometheus'  # Custom path
```

**Solution 4: TLS Issues**
```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'secure-app'
    scheme: https
    tls_config:
      ca_file: /etc/prometheus/certs/ca.crt
      cert_file: /etc/prometheus/certs/client.crt
      key_file: /etc/prometheus/certs/client.key
      insecure_skip_verify: false
```

---

### **Issue 4: Slow Queries**

**Symptoms:**
- Queries timeout
- Grafana dashboards load slowly
- High CPU usage

**Diagnosis:**
```promql
# Check query duration
rate(prometheus_engine_query_duration_seconds_sum[5m]) / rate(prometheus_engine_query_duration_seconds_count[5m])

# Find slow queries in logs
docker logs prometheus 2>&1 | grep "query timed out"
```

**Solutions:**

**Solution 1: Optimize PromQL**
```promql
# Bad - Aggregates too late
sum(rate(http_requests_total[5m])) by (job)

# Good - Aggregates early
sum by (job) (rate(http_requests_total[5m]))

# Bad - Large time range
rate(http_requests_total[1d])

# Good - Appropriate range
rate(http_requests_total[5m])

# Bad - Subquery with long range
max_over_time(rate(metric[5m])[1d:1m])

# Good - Use recording rule
max_over_time(job:metric:rate5m[1d:1m])
```

**Solution 2: Use Recording Rules**
```yaml
groups:
  - name: precomputed
    interval: 30s
    rules:
      - record: job:http_requests:rate5m
        expr: sum by (job) (rate(http_requests_total[5m]))
```

**Solution 3: Increase Query Timeout**
```yaml
# prometheus.yml
global:
  query_timeout: 2m
```

**Solution 4: Add More Resources**
```bash
# Increase CPU allocation
# Prometheus is CPU-intensive for queries
```

---

### **Issue 5: Alerts Not Firing**

**Symptoms:**
- Expected alerts not triggering
- Alerts stuck in "pending" state
- No notifications received

**Diagnosis:**
```bash
# Check alert rules
promtool check rules alerts.yml

# Check alert status
curl http://localhost:9090/api/v1/rules | jq '.data.groups[].rules[] | select(.type=="alerting")'

# Check Alertmanager connectivity
curl http://localhost:9090/api/v1/alertmanagers

# Check Alertmanager status
curl http://localhost:9093/api/v2/status

# Check Alertmanager logs
docker logs alertmanager
```

**Solutions:**

**Solution 1: Alert Rule Errors**
```yaml
# Bad - Query returns no data
- alert: HighCPU
  expr: non_existent_metric > 80

# Good - Verify query first
- alert: HighCPU
  expr: cpu_usage_percent > 80
  for: 5m
```

**Solution 2: `for` Duration Too Long**
```yaml
# Alert takes too long to fire
- alert: ServiceDown
  expr: up == 0
  for: 30m  # Too long!

# Better
- alert: ServiceDown
  expr: up == 0
  for: 1m
```

**Solution 3: Alertmanager Not Configured**
```yaml
# prometheus.yml
alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']
      timeout: 10s
```

**Solution 4: Alertmanager Routing Issues**
```yaml
# alertmanager.yml
route:
  receiver: 'default'
  group_by: ['alertname']
  routes:
    - match:
        severity: critical
      receiver: 'pagerduty'
      continue: true  # Important! Continue to other routes
```

---

### **Issue 6: Missing Metrics**

**Symptoms:**
- Some metrics not appearing
- Gaps in metric data
- Inconsistent metric collection

**Diagnosis:**
```bash
# Check metric ingestion
rate(prometheus_tsdb_head_samples_appended_total[5m])

# Check for dropped samples
rate(prometheus_tsdb_head_samples_appended_total[5m]) - rate(prometheus_tsdb_head_samples_appended_total[5m] offset 1h)

# Check scrape errors
up{job="target"} == 0
prometheus_target_scrapes_exceeded_sample_limit_total
```

**Solutions:**

**Solution 1: Sample Limit Exceeded**
```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  sample_limit: 0  # Unlimited (use with caution)

# Or per job
scrape_configs:
  - job_name: 'high-metric-app'
    sample_limit: 5000
    static_configs:
      - targets: ['app:8080']
```

**Solution 2: Scrape Timeout**
```yaml
scrape_configs:
  - job_name: 'slow-app'
    scrape_timeout: 30s  # Increase from default 10s
    static_configs:
      - targets: ['app:8080']
```

**Solution 3: Metric Name Issues**
```yaml
# Invalid metric names are dropped
# Bad: contains dash
http-requests-total

# Good: use underscore
http_requests_total

# Bad: starts with number
2xx_responses

# Good: starts with letter
responses_2xx
```

---

## 🎨 Grafana Issues

### **Issue 7: Dashboard Not Loading**

**Symptoms:**
- "No data" message
- Panels show empty
- Timeout errors

**Diagnosis:**
```bash
# Check Grafana logs
docker logs grafana

# Check data source connectivity
curl -H "Authorization: Bearer YOUR_API_KEY" \
  http://localhost:3000/api/datasources/1/health

# Test query manually
curl -X POST -H "Content-Type: application/json" \
  -d '{"query":"up","start":0,"end":0}' \
  http://localhost:9090/api/v1/query
```

**Solutions:**

**Solution 1: Data Source Misconfigured**
```bash
# Check data source URL
# In Docker: use container name not localhost
# Bad: http://localhost:9090
# Good: http://prometheus:9090
```

**Solution 2: Query Syntax Error**
```promql
# Bad
rate(http_requests_total)  # Missing time range

# Good
rate(http_requests_total[5m])
```

**Solution 3: Time Range Issues**
```json
// Dashboard time range too large
{
  "time": {
    "from": "now-90d",  // Too much data
    "to": "now"
  }
}

// Better
{
  "time": {
    "from": "now-6h",
    "to": "now"
  }
}
```

**Solution 4: Browser Console Errors**
```bash
# Check browser console (F12)
# Look for CORS errors, network timeouts

# Fix CORS in Grafana
# grafana.ini
[security]
allow_embedding = true
cookie_samesite = none
```

---

### **Issue 8: Grafana High Memory Usage**

**Symptoms:**
- Grafana becomes slow
- Dashboards timeout
- Container OOM killed

**Solutions:**

**Solution 1: Too Many Panels**
```
Reduce number of panels per dashboard
Split into multiple dashboards
Use row collapse feature
```

**Solution 2: Query Optimization**
```promql
# Bad - Too many time series
{__name__=~".+"}

# Good - Specific metrics
rate(http_requests_total[5m])

# Use Max data points setting
# Panel → Query options → Max data points: 1000
```

**Solution 3: Increase Resources**
```yaml
services:
  grafana:
    environment:
      - GF_DATABASE_MAX_OPEN_CONN=100
      - GF_DATABASE_MAX_IDLE_CONN=25
    deploy:
      resources:
        limits:
          memory: 2G
```

---

### **Issue 9: Alert Notifications Not Sent**

**Diagnosis:**
```bash
# Check notification channels
curl -H "Authorization: Bearer YOUR_API_KEY" \
  http://localhost:3000/api/alert-notifications

# Test notification
curl -X POST -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"id":1}' \
  http://localhost:3000/api/alert-notifications/1/test
```

**Solutions:**

**Solution 1: Webhook/Slack URL Wrong**
```bash
# Test webhook manually
curl -X POST -H "Content-Type: application/json" \
  -d '{"text":"Test message"}' \
  https://hooks.slack.com/services/YOUR/WEBHOOK/URL
```

**Solution 2: SMTP Configuration**
```ini
# grafana.ini
[smtp]
enabled = true
host = smtp.gmail.com:587
user = your-email@gmail.com
password = your-app-password
skip_verify = false
from_address = your-email@gmail.com
from_name = Grafana
```

---

## 🔐 Alertmanager Issues

### **Issue 10: Alerts Not Routed Correctly**

**Symptoms:**
- Alerts go to wrong receiver
- Duplicate notifications
- Alerts not grouped

**Diagnosis:**
```bash
# Check routing tree
curl http://localhost:9093/api/v2/status | jq '.config.route'

# Check active alerts
curl http://localhost:9093/api/v2/alerts | jq '.'

# Test routing
amtool config routes test --config.file=alertmanager.yml \
  severity=critical alertname=HighCPU
```

**Solutions:**

**Solution 1: Route Matching**
```yaml
# Bad - No match, goes to default
route:
  receiver: 'default'
  routes:
    - match:
        sevarity: critical  # Typo!
      receiver: 'pagerduty'

# Good
route:
  receiver: 'default'
  routes:
    - match:
        severity: critical
      receiver: 'pagerduty'
```

**Solution 2: Grouping Issues**
```yaml
# Too broad grouping
route:
  group_by: ['...']  # Groups everything

# Better
route:
  group_by: ['alertname', 'cluster', 'namespace']
```

---

### **Issue 11: Alert Silences Not Working**

**Solutions:**

```bash
# Create silence via API
curl -X POST -H "Content-Type: application/json" \
  -d '{
    "matchers": [
      {
        "name": "alertname",
        "value": "HighCPU",
        "isRegex": false
      }
    ],
    "startsAt": "2026-01-07T10:00:00Z",
    "endsAt": "2026-01-07T12:00:00Z",
    "createdBy": "admin",
    "comment": "Maintenance window"
  }' \
  http://localhost:9093/api/v2/silences

# List silences
curl http://localhost:9093/api/v2/silences | jq '.'

# Delete silence
curl -X DELETE http://localhost:9093/api/v2/silence/SILENCE_ID
```

---

## 🛠️ Common Production Issues

### **Issue 12: High Cardinality**

**Identification:**
```bash
# Find high cardinality metrics
curl http://localhost:9090/api/v1/status/tsdb | \
  jq '.data.seriesCountByMetricName | to_entries | sort_by(.value) | reverse | .[0:10]'
```

**Solutions:**
```yaml
# Drop unnecessary labels
metric_relabel_configs:
  - source_labels: [user_id]
    action: labeldrop

# Aggregate at source
# Instead of: http_requests{user_id="123", session="xyz"}
# Use: http_requests{user_type="premium"}
```

---

### **Issue 13: Federation Performance**

**Symptoms:**
- Federation scrapes timeout
- High CPU on global Prometheus

**Solution:**
```yaml
# Use external labels for filtering
global:
  external_labels:
    cluster: 'us-east-1'
    env: 'production'

# Federal only aggregated metrics
scrape_configs:
  - job_name: 'federate'
    scrape_interval: 60s  # Less frequent
    honor_labels: true
    metrics_path: '/federate'
    params:
      'match[]':
        - '{job="prometheus"}'
        - '{__name__=~"job:.*"}'  # Recording rules only
```

---

## 📊 Diagnostic Commands Reference

```bash
# Prometheus
promtool check config prometheus.yml
promtool check rules alerts.yml
promtool tsdb analyze /prometheus
curl http://localhost:9090/-/healthy
curl http://localhost:9090/api/v1/status/config
curl http://localhost:9090/api/v1/status/tsdb
curl http://localhost:9090/api/v1/targets

# Grafana
grafana-cli admin reset-admin-password newpassword
curl http://localhost:3000/api/health
curl http://localhost:3000/api/datasources

# Alertmanager
amtool config show --alertmanager.url=http://localhost:9093
amtool alert query --alertmanager.url=http://localhost:9093
amtool silence add --alertmanager.url=http://localhost:9093 alertname=Test
```

---

## 🎯 Prevention Best Practices

1. **Monitoring the Monitoring**
   - Monitor Prometheus itself
   - Track ingestion rate
   - Alert on failed scrapes

2. **Regular Maintenance**
   - Clean up old dashboards
   - Review alert rules
   - Update exporters

3. **Documentation**
   - Document custom metrics
   - Maintain runbooks
   - Keep architecture diagrams updated

4. **Testing**
   - Test alerts before deploying
   - Validate queries
   - Chaos engineering

---

**Happy troubleshooting! 🔧🚀**

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

