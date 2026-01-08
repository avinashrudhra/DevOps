# Prometheus & Grafana Quick Reference

**Essential commands, PromQL queries, and configurations for daily monitoring operations**

---

## 🚀 Quick Start

### **Start Stack with Docker Compose**

```bash
# docker-compose.yml
version: '3.8'
services:
  prometheus:
    image: prom/prometheus:latest
    ports: ["9090:9090"]
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
  
  grafana:
    image: grafana/grafana:latest
    ports: ["3000:3000"]
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin

# Start
docker-compose up -d

# URLs
# Prometheus: http://localhost:9090
# Grafana: http://localhost:3000 (admin/admin)
```

---

## 📊 Prometheus Commands

### **Binary Commands**

```bash
# Start Prometheus
./prometheus --config.file=prometheus.yml

# Check configuration
promtool check config prometheus.yml

# Check rules
promtool check rules alerts.yml

# Test query
promtool query instant http://localhost:9090 'up'

# Test query with time range
promtool query range http://localhost:9090 'up[5m]'

# Reload configuration (requires --web.enable-lifecycle)
curl -X POST http://localhost:9090/-/reload

# Check health
curl http://localhost:9090/-/healthy

# Check readiness
curl http://localhost:9090/-/ready
```

### **Configuration Validation**

```bash
# Validate prometheus.yml
promtool check config prometheus.yml

# Validate alert rules
promtool check rules /path/to/rules/*.yml

# Check metrics from target
curl http://localhost:9100/metrics

# Test scrape config
promtool check config --syntax-only prometheus.yml
```

---

## 🔍 PromQL Quick Reference

### **Basic Selectors**

```promql
# Instant vector (current value)
up
http_requests_total

# Filter by label
up{job="prometheus"}
http_requests_total{method="GET", status="200"}

# Regular expression matching
up{job=~"prometheus|node"}
http_requests_total{status=~"2.."}

# Negative matching
http_requests_total{status!="200"}
http_requests_total{job!~"test.*"}

# Range vector (values over time)
up[5m]
http_requests_total[1h]
```

### **Aggregation Operators**

```promql
# Sum
sum(http_requests_total)
sum by (status) (http_requests_total)
sum without (instance) (http_requests_total)

# Average
avg(cpu_usage_percent)
avg by (instance) (cpu_usage_percent)

# Count
count(up == 1)
count by (job) (up)

# Min/Max
min(memory_usage_bytes)
max(cpu_temperature_celsius)

# Standard deviation
stddev(response_time_seconds)

# Quantile
quantile(0.95, http_request_duration_seconds)

# Top K
topk(5, http_requests_total)
bottomk(3, memory_available_bytes)

# Count values
count_values("version", app_version_info)
```

### **Rate & Increase Functions**

```promql
# Rate - per-second average rate
rate(http_requests_total[5m])

# Irate - instant rate (more volatile)
irate(http_requests_total[5m])

# Increase - absolute increase
increase(http_requests_total[1h])

# Delta - difference for gauges
delta(cpu_temperature_celsius[1h])

# Idelta - instant delta
idelta(cpu_temperature_celsius[1h])
```

### **Histogram & Summary Functions**

```promql
# Quantile from histogram
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# Sum of histogram buckets
sum(rate(http_request_duration_seconds_bucket[5m])) by (le)

# Average from histogram
rate(http_request_duration_seconds_sum[5m]) / rate(http_request_duration_seconds_count[5m])
```

### **Time Functions**

```promql
# Current time (Unix timestamp)
time()

# Day of week (0=Sunday, 6=Saturday)
day_of_week()

# Day of month (1-31)
day_of_month()

# Hour of day (0-23)
hour()

# Minute (0-59)
minute()

# Month (1-12)
month()

# Year
year()

# Timestamp of sample
timestamp(metric_name)
```

### **Math & Comparison**

```promql
# Arithmetic operators
cpu_usage_percent + 10
memory_usage_bytes / 1024 / 1024  # Convert to MB
disk_used_bytes * 100 / disk_total_bytes

# Comparison operators
cpu_usage_percent > 80
memory_available_bytes < 1000000000

# Boolean operators
(cpu_usage_percent > 80) and (memory_usage_percent > 80)
(service_status == 1) or (service_status == 2)

# Unless (set difference)
metric_a unless metric_b
```

### **Vector Matching**

```promql
# One-to-one matching
method_code:http_errors:rate5m / ignoring(code) method:http_requests:rate5m

# Many-to-one matching
method_code:http_errors:rate5m / on(method) group_left method:http_requests:rate5m

# Group right
method:http_requests:rate5m * on(method) group_right app_info
```

### **Sorting & Limiting**

```promql
# Sort ascending
sort(http_requests_total)

# Sort descending
sort_desc(http_requests_total)

# Top K entries
topk(5, http_requests_total)

# Bottom K entries
bottomk(3, http_requests_total)
```

### **Label Manipulation**

```promql
# Label replace
label_replace(up, "host", "$1", "instance", "([^:]+):.*")

# Label join
label_join(up, "host_port", ":", "instance", "job")
```

---

## 🎯 Common Production Queries

### **System Metrics**

```promql
# CPU Usage Percentage
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory Usage Percentage
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Disk Usage Percentage
(1 - (node_filesystem_avail_bytes / node_filesystem_size_bytes)) * 100

# Disk I/O Rate
rate(node_disk_read_bytes_total[5m])
rate(node_disk_written_bytes_total[5m])

# Network Traffic
rate(node_network_receive_bytes_total[5m]) * 8  # bps
rate(node_network_transmit_bytes_total[5m]) * 8

# System Load
node_load1
node_load5
node_load15

# Process Count
node_processes_state{state="running"}
node_processes_state{state="blocked"}
```

### **Application Metrics**

```promql
# Request Rate
sum(rate(http_requests_total[5m]))

# Request Rate by Status
sum by (status) (rate(http_requests_total[5m]))

# Error Rate Percentage
sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m])) * 100

# Success Rate
sum(rate(http_requests_total{status="200"}[5m])) / sum(rate(http_requests_total[5m])) * 100

# Average Response Time
rate(http_request_duration_seconds_sum[5m]) / rate(http_request_duration_seconds_count[5m])

# 95th Percentile Latency
histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))

# 99th Percentile Latency
histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))

# Requests per minute
sum(increase(http_requests_total[1m]))
```

### **Kubernetes Metrics**

```promql
# Cluster Status
count(kube_node_status_condition{condition="Ready", status="true"})

# Pod Status
count by (phase) (kube_pod_status_phase)

# Container Restarts
sum by (namespace, pod) (rate(kube_pod_container_status_restarts_total[15m]))

# CPU Usage by Pod
sum by (namespace, pod) (rate(container_cpu_usage_seconds_total[5m]))

# Memory Usage by Pod
sum by (namespace, pod) (container_memory_usage_bytes)

# Pod CPU Request vs Actual
sum by (namespace) (container_cpu_usage_seconds_total) / sum by (namespace) (kube_pod_container_resource_requests_cpu_cores)

# Available vs Total CPU
sum(kube_node_status_allocatable_cpu_cores) - sum(kube_pod_container_resource_requests_cpu_cores)

# Pods not ready
count(kube_pod_status_phase{phase!="Running"})

# Node disk pressure
kube_node_status_condition{condition="DiskPressure", status="true"}
```

### **Prediction & Trending**

```promql
# Predict when disk will be full (4 hours ahead)
predict_linear(node_filesystem_free_bytes[1h], 4*3600) < 0

# Trend of request rate
deriv(rate(http_requests_total[5m])[30m:1m])

# Predict metric value in 1 hour
predict_linear(metric_name[30m], 3600)
```

### **SLI/SLO Calculations**

```promql
# Availability SLI (99.9% target)
sum(rate(http_requests_total{status!~"5.."}[30d])) / sum(rate(http_requests_total[30d])) * 100 > 99.9

# Error Budget Remaining
1 - ((1 - (sum(rate(http_requests_total{status!~"5.."}[30d])) / sum(rate(http_requests_total[30d])))) / (1 - 0.999))

# Latency SLI (95% under 500ms)
histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le)) < 0.5
```

---

## 🔧 Prometheus Configuration Templates

### **Basic prometheus.yml**

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    cluster: 'production'
    region: 'us-east-1'

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets: ['localhost:9093']

# Load rules
rule_files:
  - 'alerts/*.yml'
  - 'recording_rules/*.yml'

# Scrape configurations
scrape_configs:
  # Prometheus itself
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  
  # Node Exporter
  - job_name: 'node'
    static_configs:
      - targets: 
          - 'node1:9100'
          - 'node2:9100'
        labels:
          env: 'production'
  
  # Application with service discovery
  - job_name: 'app'
    consul_sd_configs:
      - server: 'consul:8500'
        services: ['app-service']
    relabel_configs:
      - source_labels: [__meta_consul_tags]
        regex: '.*,production,.*'
        action: keep
```

### **Common Alert Rules**

```yaml
# alerts.yml
groups:
  - name: system_alerts
    rules:
      - alert: HighCPU
        expr: 100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU on {{ $labels.instance }}"
          description: "CPU usage is {{ $value }}%"
      
      - alert: HighMemory
        expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 85
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory on {{ $labels.instance }}"
      
      - alert: DiskSpaceLow
        expr: (node_filesystem_avail_bytes / node_filesystem_size_bytes) * 100 < 15
        for: 10m
        labels:
          severity: critical
        annotations:
          summary: "Low disk space on {{ $labels.instance }}"
      
      - alert: ServiceDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Service {{ $labels.job }} down"
      
      - alert: HighErrorRate
        expr: sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m])) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate (>5%)"
```

### **Recording Rules**

```yaml
# recording_rules.yml
groups:
  - name: cpu_rules
    interval: 30s
    rules:
      - record: instance:node_cpu_utilization:rate5m
        expr: 100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
      
      - record: job:node_cpu_utilization:avg
        expr: avg by (job) (instance:node_cpu_utilization:rate5m)
  
  - name: memory_rules
    interval: 30s
    rules:
      - record: instance:node_memory_utilization:ratio
        expr: 1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)
      
      - record: instance:node_memory_utilization:percent
        expr: instance:node_memory_utilization:ratio * 100
```

---

## 🎨 Grafana Quick Reference

### **Grafana CLI**

```bash
# Start Grafana
grafana-server --config=/etc/grafana/grafana.ini

# Reset admin password
grafana-cli admin reset-admin-password newpassword

# Install plugin
grafana-cli plugins install grafana-piechart-panel

# List installed plugins
grafana-cli plugins ls

# Update all plugins
grafana-cli plugins update-all

# Backup
grafana-cli admin data-export --output=/backup/grafana-backup.tar
```

### **Grafana API**

```bash
# Create API key
curl -X POST -H "Content-Type: application/json" \
  -d '{"name":"apikey", "role": "Admin"}' \
  http://admin:admin@localhost:3000/api/auth/keys

# List dashboards
curl -H "Authorization: Bearer YOUR_API_KEY" \
  http://localhost:3000/api/search?query=&

# Get dashboard by UID
curl -H "Authorization: Bearer YOUR_API_KEY" \
  http://localhost:3000/api/dashboards/uid/DASHBOARD_UID

# Create dashboard
curl -X POST -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d @dashboard.json \
  http://localhost:3000/api/dashboards/db

# Create datasource
curl -X POST -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{"name":"Prometheus","type":"prometheus","url":"http://prometheus:9090","access":"proxy"}' \
  http://localhost:3000/api/datasources

# Create user
curl -X POST -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{"name":"User","email":"user@example.com","login":"user","password":"password"}' \
  http://localhost:3000/api/admin/users
```

### **Dashboard Variables**

```json
// Query variable
{
  "name": "instance",
  "type": "query",
  "datasource": "Prometheus",
  "query": "label_values(up, instance)",
  "regex": "",
  "multi": true,
  "includeAll": true,
  "allValue": ".*"
}

// Interval variable
{
  "name": "interval",
  "type": "interval",
  "query": "1m,5m,10m,30m,1h",
  "auto": true,
  "auto_count": 30,
  "auto_min": "10s"
}

// Custom variable
{
  "name": "env",
  "type": "custom",
  "query": "production,staging,development"
}

// Datasource variable
{
  "name": "datasource",
  "type": "datasource",
  "query": "prometheus"
}
```

### **Panel Queries with Variables**

```promql
# Use variable in query
up{instance=~"$instance"}

# Multiple variables
rate(http_requests_total{instance=~"$instance", job="$job"}[$interval])

# All/Any selection
up{instance=~"${instance:pipe}"}
```

### **Common Panel Types**

```json
// Graph panel
{
  "type": "graph",
  "targets": [{
    "expr": "rate(http_requests_total[5m])"
  }],
  "yaxes": [{
    "format": "reqps"
  }]
}

// Stat panel
{
  "type": "stat",
  "targets": [{
    "expr": "up"
  }],
  "options": {
    "colorMode": "background",
    "graphMode": "area"
  }
}

// Table panel
{
  "type": "table",
  "targets": [{
    "expr": "up",
    "format": "table",
    "instant": true
  }],
  "transformations": [{
    "id": "organize",
    "options": {
      "excludeByName": {},
      "indexByName": {},
      "renameByName": {}
    }
  }]
}

// Gauge panel
{
  "type": "gauge",
  "targets": [{
    "expr": "cpu_usage_percent"
  }],
  "options": {
    "thresholds": {
      "steps": [
        {"value": 0, "color": "green"},
        {"value": 70, "color": "yellow"},
        {"value": 85, "color": "red"}
      ]
    }
  }
}
```

---

## 📊 Exporters Quick Reference

### **Node Exporter**

```bash
# Run Node Exporter
docker run -d \
  --name=node-exporter \
  -p 9100:9100 \
  --pid=host \
  -v "/:/host:ro,rslave" \
  prom/node-exporter \
  --path.rootfs=/host

# Key metrics
node_cpu_seconds_total
node_memory_MemAvailable_bytes
node_filesystem_avail_bytes
node_network_receive_bytes_total
node_disk_io_time_seconds_total
```

### **Blackbox Exporter**

```yaml
# blackbox.yml
modules:
  http_2xx:
    prober: http
    timeout: 5s
    http:
      preferred_ip_protocol: "ip4"
      valid_status_codes: [200]
  
  tcp_connect:
    prober: tcp
    timeout: 5s
  
  icmp:
    prober: icmp
    timeout: 5s

# Prometheus config
scrape_configs:
  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
          - https://example.com
          - https://api.example.com
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox-exporter:9115
```

### **MySQL Exporter**

```bash
# Run MySQL Exporter
docker run -d \
  -p 9104:9104 \
  -e DATA_SOURCE_NAME="user:password@(localhost:3306)/" \
  prom/mysqld-exporter

# Key metrics
mysql_up
mysql_global_status_threads_connected
mysql_global_status_queries
mysql_global_status_slow_queries
```

---

## 🔐 Security Best Practices

### **TLS Configuration**

```yaml
# prometheus.yml with TLS
scrape_configs:
  - job_name: 'secure-app'
    scheme: https
    tls_config:
      ca_file: /etc/prometheus/certs/ca.crt
      cert_file: /etc/prometheus/certs/client.crt
      key_file: /etc/prometheus/certs/client.key
      insecure_skip_verify: false
```

### **Basic Auth**

```yaml
# prometheus.yml with auth
scrape_configs:
  - job_name: 'protected-app'
    basic_auth:
      username: prometheus
      password: secure_password
```

### **Grafana Security**

```ini
# grafana.ini
[security]
admin_user = admin
admin_password = $__file{/run/secrets/admin_password}
secret_key = $__file{/run/secrets/secret_key}

[auth]
disable_login_form = false
oauth_auto_login = false

[auth.anonymous]
enabled = false
```

---

## 🚨 Alertmanager Configuration

```yaml
# alertmanager.yml
global:
  resolve_timeout: 5m

route:
  receiver: 'default'
  group_by: ['alertname', 'cluster']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h
  
  routes:
    - match:
        severity: critical
      receiver: 'pagerduty'
    
    - match:
        severity: warning
      receiver: 'slack'

receivers:
  - name: 'default'
    webhook_configs:
      - url: 'http://webhook-receiver:5001/'
  
  - name: 'slack'
    slack_configs:
      - api_url: 'YOUR_SLACK_WEBHOOK'
        channel: '#alerts'
        title: '{{ .GroupLabels.alertname }}'
  
  - name: 'pagerduty'
    pagerduty_configs:
      - service_key: 'YOUR_PAGERDUTY_KEY'

inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'instance']
```

---

## 💡 Performance Tips

### **Query Optimization**

```promql
# Bad - calculates for every series
sum(rate(http_requests_total[5m]))

# Good - aggregates first
sum by (job) (rate(http_requests_total[5m]))

# Bad - large range
rate(http_requests_total[1d])

# Good - appropriate range
rate(http_requests_total[5m])

# Use recording rules for expensive queries
```

### **Storage Optimization**

```yaml
# prometheus.yml
global:
  scrape_interval: 30s  # Don't go too low
  scrape_timeout: 10s

storage:
  tsdb:
    retention.time: 15d  # Adjust based on needs
    retention.size: 50GB
```

---

**Save this reference for quick access during monitoring operations! 📊🚀**

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

