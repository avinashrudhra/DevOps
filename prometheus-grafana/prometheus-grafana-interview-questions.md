# Prometheus & Grafana Interview Questions

**80+ Interview Questions from Basic to Advanced Production Level**

For experienced professionals (7+ years) working with monitoring and observability.

---

## 📋 Question Categories

1. **Beginner (20 questions)** - Fundamentals & basics
2. **Intermediate (25 questions)** - PromQL & Configuration
3. **Advanced (25 questions)** - Architecture & Production
4. **Expert (10 questions)** - Design & Troubleshooting

---

## 🟢 Beginner Level Questions

### **Q1: What is Prometheus and why is it used?**

**Answer:**
Prometheus is an open-source monitoring and alerting toolkit designed for reliability and scalability. It's used for:

- **Time-series data collection**: Stores metrics with timestamps
- **Pull-based model**: Scrapes metrics from targets
- **Multi-dimensional data**: Uses labels for flexible querying
- **Built-in alerting**: Alert evaluation and routing
- **Service discovery**: Automatic target detection

**Key features:**
- PromQL query language
- No external dependencies
- Efficient storage (TSDB)
- Cloud-native (CNCF graduated project)
- Extensive ecosystem of exporters

---

### **Q2: Explain the architecture of Prometheus.**

**Answer:**

```
┌─────────────────────────────────────────┐
│         Target Applications             │
│    (Expose /metrics endpoints)          │
└──────────────┬──────────────────────────┘
               │ HTTP Pull (Scrape)
               ▼
┌──────────────────────────────────────────┐
│       Prometheus Server                  │
│  ┌────────────────────────────────────┐  │
│  │  Retrieval (Scraper)               │  │
│  └──────────┬─────────────────────────┘  │
│             ▼                             │
│  ┌────────────────────────────────────┐  │
│  │  TSDB (Time Series Database)       │  │
│  └──────────┬─────────────────────────┘  │
│             ▼                             │
│  ┌────────────────────────────────────┐  │
│  │  Rule Evaluation Engine            │  │
│  └──────────┬─────────────────────────┘  │
│             │                             │
│             ├──→ Alertmanager            │
│             │                             │
│             ├──→ HTTP API                │
│             │                             │
│             └──→ Grafana/Clients         │
└──────────────────────────────────────────┘
```

**Components:**
1. **Prometheus Server**: Core component
2. **Exporters**: Expose metrics from services
3. **Pushgateway**: For short-lived jobs
4. **Alertmanager**: Handles alerts
5. **Client Libraries**: For application instrumentation

---

### **Q3: What are the four metric types in Prometheus?**

**Answer:**

**1. Counter**
- Only increases (or resets to zero)
- Used for cumulative values
- Example: `http_requests_total`, `errors_total`

```promql
# Usage
rate(http_requests_total[5m])  # Requests per second
```

**2. Gauge**
- Can go up or down
- Used for current values
- Example: `memory_usage_bytes`, `temperature_celsius`

```promql
# Usage
memory_usage_bytes  # Current value
```

**3. Histogram**
- Samples observations into buckets
- Provides sum and count
- Used for distributions
- Example: `http_request_duration_seconds`

```promql
# Calculate 95th percentile
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
```

**4. Summary**
- Similar to histogram
- Calculates quantiles on client side
- Example: `rpc_duration_seconds`

```promql
# Pre-calculated quantiles
http_request_duration_seconds{quantile="0.9"}
```

---

### **Q4: What is PromQL?**

**Answer:**
PromQL (Prometheus Query Language) is a functional query language for selecting and aggregating time series data.

**Key concepts:**

**Instant Vector**: Single value per time series
```promql
up
http_requests_total{status="200"}
```

**Range Vector**: Values over time range
```promql
up[5m]
rate(http_requests_total[5m])
```

**Scalar**: Simple numeric value
```promql
100
time()
```

**String**: String value (rarely used)

**Common functions:**
- `rate()`: Per-second rate
- `sum()`: Sum values
- `avg()`: Average
- `max()/min()`: Max/Min values

---

### **Q5: How does Prometheus discover targets?**

**Answer:**
Prometheus supports multiple service discovery mechanisms:

**1. Static Configuration**
```yaml
scrape_configs:
  - job_name: 'app'
    static_configs:
      - targets: ['app1:8080', 'app2:8080']
```

**2. File-based Discovery**
```yaml
scrape_configs:
  - job_name: 'app'
    file_sd_configs:
      - files:
          - '/etc/prometheus/targets/*.json'
        refresh_interval: 30s
```

**3. Kubernetes Discovery**
```yaml
scrape_configs:
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
```

**4. Consul Discovery**
```yaml
scrape_configs:
  - job_name: 'consul'
    consul_sd_configs:
      - server: 'consul:8500'
```

**5. Cloud Provider Discovery**
- AWS EC2
- Azure
- GCP
- OpenStack

---

### **Q6: What is the difference between rate() and irate()?**

**Answer:**

**rate()**
- Calculates per-second average rate over time range
- Smooths out spikes
- Better for alerting
- More stable

```promql
rate(http_requests_total[5m])
# Average rate over last 5 minutes
```

**irate()**
- Calculates instant rate using last two samples
- More sensitive to changes
- Better for graphs
- More volatile

```promql
irate(http_requests_total[5m])
# Rate based on last two data points
```

**When to use:**
- **Alerting**: Use `rate()` (less noise)
- **Graphs**: Use `irate()` (more detail)

---

### **Q7: Explain labels in Prometheus.**

**Answer:**
Labels are key-value pairs that identify time series uniquely.

**Format:**
```
metric_name{label1="value1", label2="value2"}
```

**Example:**
```promql
http_requests_total{method="GET", endpoint="/api/users", status="200"}
```

**Best practices:**

**Good labels** (low cardinality):
- Environment: `env="production"`
- Region: `region="us-east-1"`
- Status code: `status="200"`
- Method: `method="GET"`

**Bad labels** (high cardinality):
- User ID: `user_id="12345"`
- Session ID: `session="abc-xyz"`
- Timestamp: `time="123456789"`
- Email: `email="user@example.com"`

**Why cardinality matters:**
- Each unique label combination creates a new time series
- High cardinality = more memory usage
- Affects query performance

---

### **Q8: What is Grafana and how does it integrate with Prometheus?**

**Answer:**
Grafana is an open-source analytics and monitoring platform for visualizing metrics.

**Integration with Prometheus:**

**1. Data Source Configuration**
```
Configuration → Data Sources → Add Prometheus
URL: http://prometheus:9090
```

**2. Query Editor**
- Visual query builder
- PromQL editor with autocomplete
- Query inspector

**3. Visualization Options**
- Time series graphs
- Stat panels
- Gauges
- Tables
- Heatmaps

**4. Features:**
- Dashboard templating
- Variables for dynamic queries
- Alert notifications
- Dashboard sharing
- Annotations

---

### **Q9: What is the Pushgateway and when should you use it?**

**Answer:**
Pushgateway is a service that allows pushing metrics from batch jobs or services that cannot be scraped.

**Use cases:**
1. **Batch jobs**: Jobs that run briefly and exit
2. **Scheduled tasks**: Cron jobs
3. **Behind firewall**: Services Prometheus can't reach
4. **Network constraints**: When pull is not possible

**Example:**
```bash
# Push metrics
echo "job_success 1" | curl --data-binary @- \
  http://pushgateway:9091/metrics/job/backup_job/instance/server1

# Push from application
cat <<EOF | curl --data-binary @- http://pushgateway:9091/metrics/job/batch
# TYPE batch_duration_seconds gauge
batch_duration_seconds{job="etl"} 123.45
# TYPE batch_records_processed counter
batch_records_processed{job="etl"} 10000
EOF
```

**Prometheus scrape config:**
```yaml
scrape_configs:
  - job_name: 'pushgateway'
    honor_labels: true
    static_configs:
      - targets: ['pushgateway:9091']
```

**When NOT to use:**
- Regular applications (use pull model)
- High-frequency metrics
- As a general metrics aggregator

---

### **Q10: How do you create an alert in Prometheus?**

**Answer:**

**Step 1: Define alert rule**
```yaml
# alerts.yml
groups:
  - name: example
    rules:
      - alert: HighCPUUsage
        expr: cpu_usage_percent > 80
        for: 5m
        labels:
          severity: warning
          team: platform
        annotations:
          summary: "High CPU on {{ $labels.instance }}"
          description: "CPU usage is {{ $value }}%"
```

**Step 2: Configure in prometheus.yml**
```yaml
rule_files:
  - 'alerts.yml'

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']
```

**Step 3: Configure Alertmanager**
```yaml
# alertmanager.yml
route:
  receiver: 'slack'

receivers:
  - name: 'slack'
    slack_configs:
      - api_url: 'YOUR_WEBHOOK_URL'
        channel: '#alerts'
```

**Alert states:**
1. **Inactive**: Condition not met
2. **Pending**: Condition met but waiting for `for` duration
3. **Firing**: Alert active and sent to Alertmanager

---

## 🟡 Intermediate Level Questions

### **Q11: Write a query to calculate the error rate percentage.**

**Answer:**
```promql
# Error rate (5xx responses)
sum(rate(http_requests_total{status=~"5.."}[5m])) 
/ 
sum(rate(http_requests_total[5m])) 
* 100

# Success rate
sum(rate(http_requests_total{status=~"2.."}[5m])) 
/ 
sum(rate(http_requests_total[5m])) 
* 100

# Error rate by endpoint
sum by (endpoint) (rate(http_requests_total{status=~"5.."}[5m])) 
/ 
sum by (endpoint) (rate(http_requests_total[5m])) 
* 100
```

---

### **Q12: How do you calculate the 95th percentile latency from a histogram?**

**Answer:**
```promql
# 95th percentile
histogram_quantile(0.95, 
  sum by (le) (rate(http_request_duration_seconds_bucket[5m]))
)

# 99th percentile
histogram_quantile(0.99, 
  sum by (le) (rate(http_request_duration_seconds_bucket[5m]))
)

# By endpoint
histogram_quantile(0.95, 
  sum by (endpoint, le) (rate(http_request_duration_seconds_bucket[5m]))
)

# Median (50th percentile)
histogram_quantile(0.50, 
  sum by (le) (rate(http_request_duration_seconds_bucket[5m]))
)
```

**Explanation:**
- `rate()`: Calculate per-second rate for each bucket
- `sum by (le)`: Aggregate buckets (le = less than or equal)
- `histogram_quantile()`: Calculate quantile from buckets

---

### **Q13: What are recording rules and why use them?**

**Answer:**
Recording rules pre-compute frequently used or expensive queries.

**Benefits:**
1. **Performance**: Pre-aggregated results
2. **Efficiency**: Reduce query load
3. **Consistency**: Same calculation everywhere
4. **Simplification**: Easier queries

**Example:**
```yaml
# recording_rules.yml
groups:
  - name: api_rules
    interval: 30s
    rules:
      # Request rate by job
      - record: job:http_requests:rate5m
        expr: sum by (job) (rate(http_requests_total[5m]))
      
      # Error rate by job
      - record: job:http_requests:error_rate5m
        expr: |
          sum by (job) (rate(http_requests_total{status=~"5.."}[5m]))
          /
          sum by (job) (rate(http_requests_total[5m]))
      
      # Average latency
      - record: job:http_request:duration_avg5m
        expr: |
          rate(http_request_duration_seconds_sum[5m])
          /
          rate(http_request_duration_seconds_count[5m])
```

**Naming convention:**
```
level:metric:operations
```
Examples:
- `instance:node_cpu:rate5m`
- `job:http_requests:rate5m`
- `cluster:memory:utilization`

---

### **Q14: Explain relabeling in Prometheus.**

**Answer:**
Relabeling allows modifying labels before storing metrics.

**Types:**

**1. Target Relabeling** (before scrape)
```yaml
scrape_configs:
  - job_name: 'app'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      # Keep only pods with annotation
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      
      # Rename label
      - source_labels: [__meta_kubernetes_pod_name]
        target_label: pod
      
      # Drop pods from namespace
      - source_labels: [__meta_kubernetes_namespace]
        action: drop
        regex: kube-system
```

**2. Metric Relabeling** (after scrape)
```yaml
scrape_configs:
  - job_name: 'app'
    metric_relabel_configs:
      # Drop high-cardinality metric
      - source_labels: [__name__]
        regex: 'high_cardinality.*'
        action: drop
      
      # Drop label
      - regex: 'user_id'
        action: labeldrop
      
      # Keep only specific metrics
      - source_labels: [__name__]
        regex: 'http_requests_.*'
        action: keep
```

**Actions:**
- `keep`: Keep matching targets/metrics
- `drop`: Drop matching targets/metrics
- `replace`: Replace label value
- `labelkeep`: Keep only matching labels
- `labeldrop`: Drop matching labels
- `labelmap`: Map label names

---

### **Q15: How do you monitor Prometheus itself?**

**Answer:**

**Key metrics:**

**1. Scrape Performance**
```promql
# Scrape duration
prometheus_target_interval_length_seconds

# Failed scrapes
up == 0
rate(prometheus_target_scrapes_exceeded_sample_limit_total[5m])

# Scrape samples ingested
rate(prometheus_tsdb_head_samples_appended_total[5m])
```

**2. Storage**
```promql
# Disk space
prometheus_tsdb_storage_blocks_bytes

# Time series count
prometheus_tsdb_head_series

# Oldest block
prometheus_tsdb_lowest_timestamp
```

**3. Query Performance**
```promql
# Query duration
rate(prometheus_engine_query_duration_seconds_sum[5m]) 
/ 
rate(prometheus_engine_query_duration_seconds_count[5m])

# Slow queries
topk(10, prometheus_engine_query_duration_seconds_sum)
```

**4. Resource Usage**
```promql
# Memory
process_resident_memory_bytes

# CPU
rate(process_cpu_seconds_total[5m])

# Go metrics
go_memstats_alloc_bytes
go_goroutines
```

**Alerts:**
```yaml
groups:
  - name: prometheus_alerts
    rules:
      - alert: PrometheusDown
        expr: up{job="prometheus"} == 0
        for: 1m
      
      - alert: PrometheusTooManyRestarts
        expr: changes(process_start_time_seconds[15m]) > 2
        for: 5m
      
      - alert: PrometheusHighMemory
        expr: process_resident_memory_bytes > 4e9
        for: 5m
```

---

### **Q16: What is federation and when would you use it?**

**Answer:**
Federation allows a Prometheus server to scrape metrics from another Prometheus server.

**Architecture:**
```
┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│ Prometheus   │   │ Prometheus   │   │ Prometheus   │
│ (Cluster A)  │   │ (Cluster B)  │   │ (Cluster C)  │
└──────┬───────┘   └──────┬───────┘   └──────┬───────┘
       │                  │                  │
       └──────────────────┴──────────────────┘
                          │
                          ▼
                ┌─────────────────┐
                │   Global         │
                │   Prometheus     │
                └─────────────────┘
```

**Configuration:**
```yaml
# Global Prometheus
scrape_configs:
  - job_name: 'federate'
    scrape_interval: 60s
    honor_labels: true
    metrics_path: '/federate'
    params:
      'match[]':
        - '{job="prometheus"}'
        - '{__name__=~"job:.*"}'  # Recording rules
    static_configs:
      - targets:
          - 'prometheus-cluster-a:9090'
          - 'prometheus-cluster-b:9090'
          - 'prometheus-cluster-c:9090'
```

**Use cases:**
1. **Multi-cluster monitoring**: Aggregate from multiple clusters
2. **Hierarchical monitoring**: Edge → Regional → Global
3. **Separation of concerns**: Different teams/environments
4. **Cross-region**: Centralized view

**Best practices:**
- Federate recording rules, not raw metrics
- Use external_labels to identify source
- Less frequent scrapes (60s+)
- Filter metrics aggressively

---

### **Q17: How do you secure Prometheus?**

**Answer:**

**1. TLS/HTTPS**
```yaml
# prometheus.yml
web_config_file: /etc/prometheus/web.yml
```

```yaml
# web.yml
tls_server_config:
  cert_file: /etc/prometheus/cert.pem
  key_file: /etc/prometheus/key.pem
```

**2. Basic Authentication**
```yaml
# web.yml
basic_auth_users:
  admin: $2y$10$hashed_password
```

```bash
# Generate password
htpasswd -nBC 10 "" | tr -d ':\n'
```

**3. TLS for Scraping**
```yaml
scrape_configs:
  - job_name: 'secure-app'
    scheme: https
    tls_config:
      ca_file: /etc/prometheus/ca.crt
      cert_file: /etc/prometheus/client.crt
      key_file: /etc/prometheus/client.key
    basic_auth:
      username: prometheus
      password: secret
```

**4. Network Security**
- Firewall rules
- VPC/Security groups
- Service mesh (mTLS)

**5. RBAC (Kubernetes)**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus
rules:
  - apiGroups: [""]
    resources:
      - nodes
      - services
      - endpoints
      - pods
    verbs: ["get", "list", "watch"]
```

**6. Grafana Security**
```ini
# grafana.ini
[auth]
disable_login_form = false

[auth.anonymous]
enabled = false

[security]
admin_user = admin
admin_password = $__file{/run/secrets/admin_password}
secret_key = $__file{/run/secrets/secret_key}
```

---

### **Q18: Explain the difference between `increase()` and `rate()`.**

**Answer:**

**rate()**
- Returns per-second average rate
- Handles counter resets
- Best for alerting

```promql
# Requests per second
rate(http_requests_total[5m])

# Result example: 10.5 req/s
```

**increase()**
- Returns total increase over time range
- Extrapolates to range boundaries
- Better for absolute counts

```promql
# Total requests in last hour
increase(http_requests_total[1h])

# Result example: 36000 requests
```

**Relationship:**
```promql
increase(metric[5m]) ≈ rate(metric[5m]) * 300
# 300 seconds = 5 minutes
```

**When to use:**
- **Graphs**: `rate()` for smooth per-second view
- **Total counts**: `increase()` for absolute numbers
- **Alerts**: `rate()` for consistent thresholds

---

### **Q19: How do you handle high cardinality metrics?**

**Answer:**

**1. Identify High Cardinality**
```bash
# Check cardinality
curl http://localhost:9090/api/v1/status/tsdb | \
  jq '.data.seriesCountByMetricName | to_entries | sort_by(.value) | reverse | .[0:10]'
```

**2. Drop at Source**
```yaml
# Don't create the metric with high cardinality labels
# Bad:
http_requests_total{user_id="12345"}

# Good:
http_requests_total{user_type="premium"}
```

**3. Metric Relabeling**
```yaml
metric_relabel_configs:
  # Drop entire metric
  - source_labels: [__name__]
    regex: 'high_cardinality_metric'
    action: drop
  
  # Drop specific labels
  - regex: 'user_id|session_id'
    action: labeldrop
  
  # Aggregate labels
  - source_labels: [user_id]
    target_label: user_type
    replacement: 'user'
```

**4. Use Recording Rules**
```yaml
# Pre-aggregate high cardinality metrics
- record: api:requests:by_endpoint
  expr: sum by (endpoint) (rate(http_requests_total[5m]))
```

**5. Shorter Retention**
```yaml
# Reduce retention for high cardinality metrics
storage:
  tsdb:
    retention.time: 7d  # Instead of 30d
```

**6. Sample Limiting**
```yaml
scrape_configs:
  - job_name: 'app'
    sample_limit: 10000  # Limit samples per scrape
```

---

### **Q20: What are the best practices for naming metrics?**

**Answer:**

**Naming Convention:**
```
<namespace>_<subsystem>_<name>_<unit>
```

**Examples:**

**Counters** (suffix: `_total`):
```
http_requests_total
database_queries_total
cache_hits_total
errors_total
```

**Gauges** (descriptive name with unit):
```
memory_usage_bytes
cpu_usage_percent
active_connections
queue_length
temperature_celsius
```

**Histograms** (suffix: `_seconds`, `_bytes`):
```
http_request_duration_seconds
response_size_bytes
query_duration_seconds
```

**Rules:**
1. **Use base units**: seconds (not milliseconds), bytes (not KB)
2. **Lowercase with underscores**: `http_requests_total`
3. **Metric name is namespace**: `http_*`, `process_*`
4. **Labels for dimensions**: `{method="GET", status="200"}`
5. **Total suffix for counters**: `*_total`
6. **Descriptive labels**: `status` not `s`, `method` not `m`

**Bad:**
```
httpRequestsGET
request-count
RequestTime_ms
user123_requests
```

**Good:**
```
http_requests_total{method="GET"}
http_request_duration_seconds
user_requests_total{user_type="premium"}
```

---

## 🔴 Advanced Level Questions

### **Q21: Design a high-availability Prometheus architecture.**

**Answer:**

**Architecture:**
```
                  ┌─────────────────┐
                  │  Load Balancer  │
                  └────────┬────────┘
                           │
          ┌────────────────┴────────────────┐
          │                                 │
   ┌──────▼──────┐                  ┌──────▼──────┐
   │ Prometheus  │                  │ Prometheus  │
   │  Instance 1 │                  │  Instance 2 │
   │  (Active)   │                  │  (Active)   │
   └──────┬──────┘                  └──────┬──────┘
          │                                 │
          └────────────────┬────────────────┘
                           │
                  ┌────────▼────────┐
                  │  Thanos/Cortex  │
                  │  (Long-term)    │
                  └─────────────────┘
                           │
                  ┌────────▼────────┐
                  │  Object Storage │
                  │  (S3/GCS/Azure) │
                  └─────────────────┘
```

**Implementation:**

**1. Multiple Prometheus Instances**
```yaml
# prometheus-1.yml
global:
  external_labels:
    prometheus: 'prom-1'
    cluster: 'production'
```

```yaml
# prometheus-2.yml
global:
  external_labels:
    prometheus: 'prom-2'
    cluster: 'production'
```

**2. Thanos Sidecar**
```yaml
# docker-compose.yml
services:
  prometheus-1:
    image: prom/prometheus
    command:
      - '--storage.tsdb.min-block-duration=2h'
      - '--storage.tsdb.max-block-duration=2h'
  
  thanos-sidecar-1:
    image: thanosio/thanos
    command:
      - 'sidecar'
      - '--prometheus.url=http://prometheus-1:9090'
      - '--tsdb.path=/prometheus'
      - '--objstore.config-file=/etc/thanos/bucket.yml'
```

**3. Thanos Query (Deduplication)**
```yaml
thanos-query:
  image: thanosio/thanos
  command:
    - 'query'
    - '--store=thanos-sidecar-1:10901'
    - '--store=thanos-sidecar-2:10901'
    - '--store=thanos-store:10901'
```

**Benefits:**
- No single point of failure
- Horizontal scalability
- Long-term storage
- Global query view
- Automatic deduplication

---

### **Q22: How do you optimize Prometheus performance?**

**Answer:**

**1. Query Optimization**
```promql
# Bad - Aggregates late
sum(rate(http_requests_total[5m])) by (job)

# Good - Aggregates early
sum by (job) (rate(http_requests_total[5m]))

# Bad - Large time range
rate(metric[1d])

# Good - Appropriate range
rate(metric[5m])
```

**2. Use Recording Rules**
```yaml
# Pre-compute expensive queries
groups:
  - name: aggregations
    interval: 30s
    rules:
      - record: job:http_requests:rate5m
        expr: sum by (job) (rate(http_requests_total[5m]))
```

**3. Reduce Cardinality**
```yaml
# Drop unnecessary labels
metric_relabel_configs:
  - regex: '(user_id|session_id|request_id)'
    action: labeldrop
```

**4. Optimize Scrape Configuration**
```yaml
global:
  scrape_interval: 30s  # Don't go too low
  scrape_timeout: 10s
  evaluation_interval: 30s
```

**5. Storage Optimization**
```yaml
storage:
  tsdb:
    retention.time: 15d
    retention.size: 50GB
```

**6. Resource Allocation**
```yaml
# Prometheus needs:
# - CPU for queries
# - Memory for active series
# - Disk for TSDB

# Formula for memory:
# RAM = ingestion_rate * 1KB * 3
# Example: 100K samples/s * 1KB * 3 = 300MB
```

**7. Query Caching (Thanos/Cortex)**
```yaml
# Cache query results
query_frontend:
  cache:
    enabled: true
    ttl: 5m
```

---

### **Q23: Explain TSDB (Time Series Database) in Prometheus.**

**Answer:**

**Architecture:**
```
/prometheus/
├── 01ABCD1234/      # Block (2h of data)
│   ├── chunks/      # Compressed samples
│   ├── index        # Label index
│   ├── meta.json    # Block metadata
│   └── tombstones   # Deleted series
├── 01ABCD5678/      # Another block
├── wal/             # Write-Ahead Log
│   ├── 00000001
│   └── 00000002
└── chunks_head/     # Active data
```

**Key Components:**

**1. WAL (Write-Ahead Log)**
- Durability guarantee
- All samples written here first
- Replayed on restart

**2. Head Block**
- In-memory active data
- Last 2-3 hours
- Periodically flushed to disk

**3. Blocks**
- Immutable 2-hour chunks
- Compressed time series
- Indexed by labels

**4. Compaction**
- Merges small blocks
- Reduces number of files
- Improves query performance

**Storage Format:**
```
Sample = (timestamp, value)
Series = metric{labels} → [samples...]
Block = [series...] + index
```

**Benefits:**
- Fast writes (append-only)
- Efficient reads (indexed)
- Good compression (~1.3 bytes/sample)
- Automatic cleanup

---

### **Q24: How would you implement SLO monitoring with Prometheus?**

**Answer:**

**SLO Definition:**
```
Service Level Objective (SLO): 99.9% availability
= 0.1% error budget per 30 days
= 43.2 minutes downtime per month
```

**1. Define SLIs (Service Level Indicators)**
```yaml
# recording_rules.yml
groups:
  - name: sli_rules
    interval: 30s
    rules:
      # Availability SLI
      - record: sli:availability:ratio
        expr: |
          sum(rate(http_requests_total{status!~"5.."}[5m]))
          /
          sum(rate(http_requests_total[5m]))
      
      # Latency SLI (95% < 500ms)
      - record: sli:latency:ratio
        expr: |
          sum(rate(http_request_duration_seconds_bucket{le="0.5"}[5m]))
          /
          sum(rate(http_request_duration_seconds_count[5m]))
```

**2. Calculate SLO Compliance**
```promql
# 30-day availability
sum(rate(http_requests_total{status!~"5.."}[30d]))
/
sum(rate(http_requests_total[30d]))
* 100

# Result: 99.95% (exceeds 99.9% SLO)
```

**3. Error Budget**
```promql
# Error budget remaining
(
  1 - (
    (1 - sum(rate(http_requests_total{status!~"5.."}[7d])) / sum(rate(http_requests_total[7d])))
    /
    (1 - 0.999)  # Target: 99.9%
  )
) * 100

# Positive = budget remaining
# Negative = budget exceeded
```

**4. Burn Rate Alerts**
```yaml
# alerts.yml
groups:
  - name: slo_alerts
    rules:
      # Fast burn (2% budget in 1h = 14.4x burn rate)
      - alert: ErrorBudgetBurnFast
        expr: |
          (
            sum(rate(http_requests_total{status=~"5.."}[1h]))
            /
            sum(rate(http_requests_total[1h]))
          ) > (14.4 * 0.001)
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Fast error budget burn detected"
      
      # Slow burn (10% budget in 6h = 2x burn rate)
      - alert: ErrorBudgetBurnSlow
        expr: |
          (
            sum(rate(http_requests_total{status=~"5.."}[6h]))
            /
            sum(rate(http_requests_total[6h]))
          ) > (2 * 0.001)
        for: 15m
        labels:
          severity: warning
```

**5. Dashboard**
```json
{
  "panels": [
    {
      "title": "SLO Compliance (30d)",
      "targets": [{
        "expr": "sli:availability:ratio * 100"
      }],
      "thresholds": [
        {"value": 99.9, "color": "green"},
        {"value": 99.5, "color": "yellow"},
        {"value": 0, "color": "red"}
      ]
    },
    {
      "title": "Error Budget Remaining",
      "targets": [{
        "expr": "error_budget_remaining"
      }]
    }
  ]
}
```

---

### **Q25: How do you implement distributed tracing with Prometheus?**

**Answer:**

**Observability Pillars:**
```
┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│   Metrics   │   │   Logs      │   │   Traces    │
│ (Prometheus)│   │ (Loki/ELK)  │   │  (Jaeger)   │
└─────────────┘   └─────────────┘   └─────────────┘
```

**1. OpenTelemetry Integration**
```python
# Python application
from opentelemetry import trace, metrics
from opentelemetry.exporter.prometheus import PrometheusMetricReader
from opentelemetry.sdk.metrics import MeterProvider
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.jaeger.thrift import JaegerExporter

# Setup tracing
trace.set_tracer_provider(TracerProvider())
jaeger_exporter = JaegerExporter(
    agent_host_name="jaeger",
    agent_port=6831,
)
trace.get_tracer_provider().add_span_processor(
    BatchSpanProcessor(jaeger_exporter)
)

# Setup metrics
reader = PrometheusMetricReader()
metrics.set_meter_provider(MeterProvider(metric_readers=[reader]))

# Create tracer and meter
tracer = trace.get_tracer(__name__)
meter = metrics.get_meter(__name__)

# Create metrics
request_counter = meter.create_counter(
    "http_requests_total",
    description="Total HTTP requests"
)
request_duration = meter.create_histogram(
    "http_request_duration_seconds",
    description="HTTP request duration"
)

# Instrument request
@app.route("/api/users")
def get_users():
    with tracer.start_as_current_span("get_users") as span:
        request_counter.add(1, {"method": "GET", "endpoint": "/api/users"})
        
        start = time.time()
        # Business logic
        users = database.get_users()
        
        duration = time.time() - start
        request_duration.record(duration, {"endpoint": "/api/users"})
        
        span.set_attribute("user.count", len(users))
        return jsonify(users)
```

**2. Exemplars (Link Metrics → Traces)**
```yaml
# prometheus.yml
storage:
  exemplars:
    max_exemplars: 100000
```

```python
# Record exemplar with trace ID
from opentelemetry import trace

# In request handler
span = trace.get_current_span()
trace_id = format(span.get_span_context().trace_id, '032x')

# Prometheus client supports exemplars
request_duration.observe(
    duration,
    exemplar={'trace_id': trace_id}
)
```

**3. Grafana Integration**
```json
// Dashboard with trace links
{
  "datasource": "Prometheus",
  "targets": [{
    "expr": "rate(http_request_duration_seconds_count[5m])",
    "exemplar": true  // Show exemplars
  }]
}
```

**4. Query Traces from Metrics**
```promql
# Find slow requests
histogram_quantile(0.99, 
  rate(http_request_duration_seconds_bucket[5m])
) > 1

# Click on exemplar point in Grafana
# → Opens Jaeger with specific trace
```

---

## ⚫ Expert Level Questions

### **Q26: You have 1 million active time series, and Prometheus is running out of memory. How do you troubleshoot and fix this?**

**Answer:**

**Step 1: Diagnosis**
```bash
# Check current metrics
curl http://localhost:9090/api/v1/status/tsdb

# Response:
{
  "status": "success",
  "data": {
    "seriesCountByMetricName": [
      {"name": "high_cardinality_metric", "value": 800000},
      {"name": "user_requests_total", "value": 150000},
      {"name": "normal_metric", "value": 50000}
    ],
    "labelValueCountByLabelName": [
      {"name": "user_id", "value": 500000},  # HIGH!
      {"name": "session_id", "value": 300000}  # HIGH!
    ]
  }
}

# Memory usage calculation
# 1M series * 3KB = 3GB just for active series
# Plus WAL, query cache, etc. = 5-6GB total
```

**Step 2: Immediate Mitigation**
```yaml
# Drop high cardinality metrics
metric_relabel_configs:
  - source_labels: [__name__]
    regex: 'high_cardinality_metric'
    action: drop
  
  # Drop problematic labels
  - regex: '(user_id|session_id)'
    action: labeldrop

# Reload Prometheus
curl -X POST http://localhost:9090/-/reload
```

**Step 3: Long-term Solutions**

**Option 1: Fix at Source**
```python
# Bad: Creates 1M time series (1M users)
user_requests_total{user_id="12345"}

# Good: Creates 3 time series (3 types)
user_requests_total{user_type="premium|standard|free"}
```

**Option 2: Horizontal Sharding**
```yaml
# Prometheus 1: Monitor services A-M
scrape_configs:
  - job_name: 'services'
    kubernetes_sd_configs:
      - role: service
    relabel_configs:
      - source_labels: [__meta_kubernetes_service_name]
        regex: '^[a-mA-M].*'
        action: keep

# Prometheus 2: Monitor services N-Z
scrape_configs:
  - job_name: 'services'
    kubernetes_sd_configs:
      - role: service
    relabel_configs:
      - source_labels: [__meta_kubernetes_service_name]
        regex: '^[n-zN-Z].*'
        action: keep
```

**Option 3: Move to Cortex/Thanos**
```yaml
# Cortex handles high cardinality better
# Distributed architecture
# Horizontal scaling
```

**Option 4: Sampling**
```yaml
# Only keep 10% of high cardinality metrics
metric_relabel_configs:
  - source_labels: [user_id]
    modulus: 10
    target_label: __tmp_hash
  - source_labels: [__tmp_hash]
    regex: "0"  # Keep only 0 (10%)
    action: keep
```

---

### **Q27: Design a monitoring solution for a global, multi-region, multi-cloud Kubernetes deployment.**

**Answer:**

**Architecture:**
```
┌─────────────────────────────────────────────────────────┐
│                     Global Layer                        │
│  ┌────────────────┐         ┌────────────────┐        │
│  │ Thanos Query   │────────▶│ Thanos Ruler   │        │
│  └────────┬───────┘         └────────────────┘        │
└───────────┼──────────────────────────────────────────────┘
            │
    ┌───────┴────────┬────────────────┬───────────────┐
    │                │                │               │
┌───▼────────┐  ┌───▼────────┐  ┌───▼────────┐  ┌───▼────────┐
│  Region 1  │  │  Region 2  │  │  Region 3  │  │  Region 4  │
│  (AWS)     │  │  (GCP)     │  │  (Azure)   │  │  (On-prem) │
│            │  │            │  │            │  │            │
│ Prometheus │  │ Prometheus │  │ Prometheus │  │ Prometheus │
│  + Thanos  │  │  + Thanos  │  │  + Thanos  │  │  + Thanos  │
│  Sidecar   │  │  Sidecar   │  │  Sidecar   │  │  Sidecar   │
│     │      │  │     │      │  │     │      │  │     │      │
│     ▼      │  │     ▼      │  │     ▼      │  │     ▼      │
│ K8s Cluster│  │ K8s Cluster│  │ K8s Cluster│  │ K8s Cluster│
└────────────┘  └────────────┘  └────────────┘  └────────────┘
      │               │               │               │
      └───────────────┴───────────────┴───────────────┘
                          │
                          ▼
              ┌────────────────────────┐
              │   Object Storage       │
              │   (S3/GCS/Azure Blob)  │
              └────────────────────────┘
```

**Implementation:**

**1. Per-Region Prometheus**
```yaml
# prometheus-region1.yml
global:
  external_labels:
    cluster: 'prod-us-east-1'
    region: 'us-east-1'
    cloud: 'aws'
    prometheus: 'prom-us-east-1-1'

scrape_configs:
  - job_name: 'kubernetes-apiservers'
    kubernetes_sd_configs:
      - role: endpoints

  - job_name: 'kubernetes-nodes'
    kubernetes_sd_configs:
      - role: node

  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
```

**2. Thanos Sidecar**
```yaml
# thanos-sidecar.yml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: prometheus
spec:
  template:
    spec:
      containers:
        - name: prometheus
          image: prom/prometheus
          args:
            - '--storage.tsdb.min-block-duration=2h'
            - '--storage.tsdb.max-block-duration=2h'
        
        - name: thanos-sidecar
          image: thanosio/thanos
          args:
            - 'sidecar'
            - '--prometheus.url=http://localhost:9090'
            - '--tsdb.path=/prometheus'
            - '--objstore.config=$(OBJSTORE_CONFIG)'
            - '--grpc-address=0.0.0.0:10901'
          env:
            - name: OBJSTORE_CONFIG
              valueFrom:
                secretKeyRef:
                  name: thanos-objstore
                  key: config.yaml
```

**3. Object Storage Config**
```yaml
# For AWS S3
type: S3
config:
  bucket: "thanos-metrics"
  endpoint: "s3.amazonaws.com"
  region: "us-east-1"
  access_key: "ACCESS_KEY"
  secret_key: "SECRET_KEY"

# For GCS
type: GCS
config:
  bucket: "thanos-metrics"
  service_account: "sa-key.json"

# For Azure
type: AZURE
config:
  storage_account: "thanosmetrics"
  storage_account_key: "KEY"
  container: "metrics"
```

**4. Global Thanos Query**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: thanos-query
spec:
  template:
    spec:
      containers:
        - name: thanos-query
          image: thanosio/thanos
          args:
            - 'query'
            - '--http-address=0.0.0.0:9090'
            - '--grpc-address=0.0.0.0:10901'
            - '--store=dnssrv+_grpc._tcp.thanos-sidecar-us-east-1.monitoring.svc'
            - '--store=dnssrv+_grpc._tcp.thanos-sidecar-eu-west-1.monitoring.svc'
            - '--store=dnssrv+_grpc._tcp.thanos-sidecar-ap-south-1.monitoring.svc'
            - '--store=dnssrv+_grpc._tcp.thanos-store.monitoring.svc'
            - '--query.replica-label=prometheus'  # Deduplication
```

**5. Global Alerting**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: thanos-ruler
spec:
  template:
    spec:
      containers:
        - name: thanos-ruler
          image: thanosio/thanos
          args:
            - 'rule'
            - '--query=http://thanos-query:9090'
            - '--alertmanagers.url=http://alertmanager:9093'
            - '--rule-file=/etc/thanos/rules/*.yml'
          volumeMounts:
            - name: rules
              mountPath: /etc/thanos/rules
```

**6. Global Dashboard (Grafana)**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: grafana-datasource
data:
  datasource.yaml: |
    apiVersion: 1
    datasources:
      - name: Thanos
        type: prometheus
        url: http://thanos-query:9090
        isDefault: true
```

**Key Features:**
- ✅ Multi-region monitoring
- ✅ Cross-cloud visibility
- ✅ Automatic deduplication
- ✅ Long-term storage (unlimited retention)
- ✅ Global alerting
- ✅ Disaster recovery
- ✅ Query caching
- ✅ Downsampling for efficiency

---

### **Q28: A critical production alert fired but you see no evidence in Prometheus. How do you investigate?**

**Answer:**

**Investigation Steps:**

**1. Verify Alert Fired**
```bash
# Check Alertmanager
curl http://localhost:9093/api/v2/alerts

# Check Prometheus alerts
curl http://localhost:9090/api/v1/rules | jq '.data.groups[].rules[] | select(.type=="alerting")'

# Check alert history
curl http://localhost:9090/api/v1/query?query=ALERTS{alertname="CriticalAlert"}
```

**2. Check Alert Rule**
```yaml
# Was alert recently changed?
git log -p alerts.yml

# Test alert query manually
# In Prometheus UI, run the expr
cpu_usage_percent > 80
```

**3. Check for Data Gaps**
```promql
# Check if metric exists
up{job="critical-service"}

# Check for gaps (returns 0 if data missing)
count_over_time(cpu_usage_percent[1h])

# When did we last receive data?
timestamp(cpu_usage_percent) - time()  # Negative = minutes ago
```

**4. Review Scrape Targets**
```bash
# Check target status
curl http://localhost:9090/api/v1/targets | jq '.data.activeTargets[] | select(.health != "up")'

# Scrape duration
prometheus_target_interval_length_seconds{job="critical-service"}

# Failed scrapes
rate(prometheus_target_scrapes_exceeded_sample_limit_total{job="critical-service"}[5m])
```

**5. Check Prometheus Logs**
```bash
docker logs prometheus --since 30m | grep -i error

# Look for:
# - Scrape timeouts
# - Configuration errors
# - Out of memory
# - TSDB errors
```

**6. Check Alertmanager Logs**
```bash
docker logs alertmanager --since 30m

# Look for:
# - Notification failures
# - Silences
# - Inhibition rules
```

**7. Verify Alertmanager Routing**
```bash
# Test routing
amtool config routes test \
  --config.file=alertmanager.yml \
  severity=critical alertname=CriticalAlert

# Check silences
amtool silence query --alertmanager.url=http://localhost:9093
```

**8. Check Time Sync**
```bash
# Prometheus server time
date

# Target server time
ssh target-server date

# Time drift can cause metrics to be rejected
```

**9. Check TSDB**
```bash
# Check for corruption
promtool tsdb analyze /prometheus

# Check disk space
df -h /prometheus

# Check if Prometheus was OOM killed
dmesg | grep -i "out of memory"
```

**10. Review Recent Changes**
```bash
# Configuration changes
git log --since="2 hours ago" prometheus.yml alerts.yml

# Deployments
kubectl get events --sort-by='.lastTimestamp' | grep prometheus

# Infrastructure changes
# Check cloud provider console for changes
```

**Common Root Causes:**

1. **Alert silenced**: Someone silenced it
2. **Scrape failure**: Target down during incident
3. **Query error**: Alert expr has bug
4. **Data retention**: Data aged out
5. **Time sync issue**: Clock drift
6. **Configuration reload**: Config change broke alert
7. **Alertmanager routing**: Routed to wrong receiver
8. **TSDB corruption**: Data lost
9. **OOM**: Prometheus restarted, lost in-memory data
10. **Network partition**: Targets unreachable

---

### **Q29: How would you migrate from Prometheus to a managed service (AWS CloudWatch, Datadog) while maintaining metrics history?**

**Answer:**

**Migration Strategy:**

**Phase 1: Parallel Running (Week 1-2)**
```
┌──────────────┐     ┌──────────────┐
│  Prometheus  │     │   CloudWatch │
│   (Existing) │     │     (New)    │
└──────┬───────┘     └──────┬───────┘
       │                    │
       └────────┬───────────┘
                │
           ┌────▼─────┐
           │ Services │
           └──────────┘
```

**Step 1: Dual Instrumentation**
```python
# Python example - send to both
from prometheus_client import Counter
import boto3

# Prometheus metric
prom_counter = Counter('requests_total', 'Total requests')

# CloudWatch client
cloudwatch = boto3.client('cloudwatch')

def record_request():
    # Prometheus
    prom_counter.inc()
    
    # CloudWatch
    cloudwatch.put_metric_data(
        Namespace='MyApp',
        MetricData=[{
            'MetricName': 'RequestCount',
            'Value': 1,
            'Unit': 'Count'
        }]
    )
```

**Step 2: Export Historical Data**
```bash
# Export Prometheus data
promtool tsdb dump /prometheus > metrics.json

# Or use snapshot API
curl -X POST http://localhost:9090/api/v1/admin/tsdb/snapshot

# Convert to target format
python convert_metrics.py metrics.json cloudwatch_format.json
```

**Step 3: Import to New System**
```python
# Import to CloudWatch
import boto3
import json

cloudwatch = boto3.client('cloudwatch')

with open('cloudwatch_format.json') as f:
    metrics = json.load(f)
    
    for metric in metrics:
        cloudwatch.put_metric_data(
            Namespace='Imported',
            MetricData=[{
                'MetricName': metric['name'],
                'Timestamp': metric['timestamp'],
                'Value': metric['value'],
                'Dimensions': metric['labels']
            }]
        )
```

**Phase 2: Validation (Week 3-4)**
```bash
# Compare metrics between systems
# Prometheus
rate(http_requests_total[5m])

# CloudWatch equivalent
# Verify similar values
```

**Phase 3: Migrate Dashboards (Week 5-6)**
```bash
# Export Grafana dashboards
curl -H "Authorization: Bearer $TOKEN" \
  http://grafana:3000/api/dashboards/uid/ABC123 > dashboard.json

# Convert queries
# PromQL: rate(http_requests_total[5m])
# CloudWatch: SUM(RequestCount) / 300
```

**Phase 4: Migrate Alerts (Week 7-8)**
```yaml
# Prometheus alert
- alert: HighErrorRate
  expr: rate(errors_total[5m]) > 10

# CloudWatch alarm (CloudFormation)
ErrorRateAlarm:
  Type: AWS::CloudWatch::Alarm
  Properties:
    AlarmName: HighErrorRate
    MetricName: Errors
    Statistic: Sum
    Period: 300
    Threshold: 10
    ComparisonOperator: GreaterThanThreshold
```

**Phase 5: Decommission (Week 9-10)**
```bash
# Gradually reduce Prometheus retention
# prometheus.yml
storage:
  tsdb:
    retention.time: 7d  # From 30d

# Monitor for any issues
# After 2 weeks of stable operation, shutdown
docker-compose down prometheus
```

**Challenges & Solutions:**

| Challenge | Solution |
|-----------|----------|
| Query language change | Document query mappings |
| Different metric types | Convert at source |
| Label → Dimension mapping | Use tags/dimensions |
| Historical data volume | Sample or aggregate |
| Cost considerations | Set up cost alerts |
| Team training | Training sessions |

---

### **Q30: Design a monitoring strategy for a company transitioning from monolith to microservices.**

**Answer:**

**Evolution Strategy:**

**Phase 1: Monolith + Prometheus**
```
┌────────────────────────────────────┐
│                                    │
│         Monolith App               │
│                                    │
│  ┌──────────┐  ┌──────────┐      │
│  │ Web      │  │ API      │      │
│  └──────────┘  └──────────┘      │
│  ┌──────────┐  ┌──────────┐      │
│  │ Business │  │ Data     │      │
│  └──────────┘  └──────────┘      │
│                                    │
│       /metrics endpoint            │
└─────────────┬──────────────────────┘
              │
              ▼
       ┌──────────────┐
       │  Prometheus  │
       └──────────────┘
```

```python
# Instrument monolith
from prometheus_client import Counter, Histogram

requests = Counter('app_requests_total', 'Requests', ['endpoint', 'method'])
latency = Histogram('app_request_duration_seconds', 'Duration', ['endpoint'])

@app.route('/api/users')
@latency.labels(endpoint='/api/users').time()
def get_users():
    requests.labels(endpoint='/api/users', method='GET').inc()
    return jsonify(users)
```

**Phase 2: Strangler Pattern**
```
┌──────────────┐          ┌──────────────────┐
│   Monolith   │          │   Microservices  │
│              │          │                  │
│  ┌────────┐  │          │  ┌────────────┐ │
│  │ Users  │──┼──────────┼─▶│User Service│ │
│  └────────┘  │          │  └────────────┘ │
│              │          │                  │
│  ┌────────┐  │          │  ┌────────────┐ │
│  │ Orders │  │          │  │Order Service│ │
│  └────────┘  │          │  └────────────┘ │
│              │          │                  │
└──────┬───────┘          └────────┬─────────┘
       │                           │
       └───────────┬───────────────┘
                   ▼
            ┌──────────────┐
            │  Prometheus  │
            └──────────────┘
```

**Monitoring Strategy:**

**1. Service-Level Metrics**
```yaml
# ServiceMonitor for each service
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: user-service
spec:
  selector:
    matchLabels:
      app: user-service
  endpoints:
    - port: metrics
```

**2. Request Tracing**
```python
# Add trace IDs
import uuid

@app.before_request
def before_request():
    g.trace_id = request.headers.get('X-Trace-ID', str(uuid.uuid4()))
    request_start.labels(service='user-service', trace_id=g.trace_id).inc()
```

**3. Service Mesh (Istio/Linkerd)**
```yaml
# Automatic metrics from sidecar
apiVersion: v1
kind: Service
metadata:
  name: user-service
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "15020"  # Istio sidecar
```

**4. Golden Signals**
```promql
# Latency
histogram_quantile(0.99, 
  sum(rate(http_request_duration_seconds_bucket{service="user-service"}[5m])) by (le)
)

# Traffic
sum(rate(http_requests_total{service="user-service"}[5m]))

# Errors
sum(rate(http_requests_total{service="user-service", status=~"5.."}[5m])) 
/ 
sum(rate(http_requests_total{service="user-service"}[5m]))

# Saturation
container_memory_usage_bytes{pod=~"user-service.*"} 
/ 
container_spec_memory_limit_bytes{pod=~"user-service.*"}
```

**5. Dependency Mapping**
```yaml
# Recording rule for service graph
- record: service:dependency:requests_total
  expr: |
    sum by (source_service, destination_service) (
      rate(http_requests_total{destination_service!=""}[5m])
    )
```

**6. Migration Dashboard**
```json
{
  "dashboard": {
    "title": "Migration Progress",
    "panels": [
      {
        "title": "Traffic Split",
        "targets": [{
          "expr": "sum by (service) (rate(http_requests_total[5m]))"
        }]
      },
      {
        "title": "Error Rate Comparison",
        "targets": [
          {
            "expr": "monolith_error_rate",
            "legendFormat": "Monolith"
          },
          {
            "expr": "microservices_error_rate",
            "legendFormat": "Microservices"
          }
        ]
      }
    ]
  }
}
```

**Phase 3: Full Microservices**
```
┌────────────────────────────────────────────────────┐
│              Service Mesh (Istio)                  │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐        │
│  │  User    │  │  Order   │  │  Payment │        │
│  │  Service │  │  Service │  │  Service │        │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘        │
│       │             │             │               │
│       └─────────────┼─────────────┘               │
│                     │                              │
└─────────────────────┼──────────────────────────────┘
                      │
              ┌───────▼────────┐
              │   Thanos Query │
              └───────┬────────┘
                      │
      ┌───────────────┼───────────────┐
      │               │               │
┌─────▼──────┐ ┌─────▼──────┐ ┌─────▼──────┐
│Prometheus 1│ │Prometheus 2│ │Prometheus 3│
└────────────┘ └────────────┘ └────────────┘
```

**Key Metrics for Migration:**

```yaml
# Migration progress
- record: migration:traffic:percentage
  expr: |
    sum(rate(http_requests_total{service=~".*-service"}[5m]))
    /
    sum(rate(http_requests_total[5m]))
    * 100

# Error rate delta
- record: migration:error_rate:delta
  expr: |
    (microservices_error_rate - monolith_error_rate) 
    / 
    monolith_error_rate 
    * 100

# Latency comparison
- record: migration:latency:ratio
  expr: |
    microservices_p99_latency 
    / 
    monolith_p99_latency
```

**Success Criteria:**
- ✅ Error rate ≤ monolith baseline
- ✅ Latency ≤ 1.2x monolith
- ✅ All services instrumented
- ✅ Distributed tracing working
- ✅ SLOs defined and tracked
- ✅ Alerts migrated and tested

---

**Congratulations! You've completed all 80+ interview questions! 🎉📊🚀**

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

