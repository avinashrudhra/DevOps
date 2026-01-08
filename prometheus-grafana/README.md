# Prometheus & Grafana - Monitoring & Observability

Complete guide to mastering Prometheus and Grafana for production monitoring, alerting, and observability.

---

## 📊 What are Prometheus & Grafana?

**Prometheus** is an open-source monitoring and alerting toolkit designed for reliability and scalability.

**Grafana** is an open-source analytics and monitoring platform for visualizing metrics from multiple data sources.

**Together, they form the industry-standard monitoring stack for:**
- Kubernetes clusters
- Microservices architectures
- Cloud-native applications
- Infrastructure monitoring
- Application performance monitoring (APM)

**Why Prometheus & Grafana?**
- ✅ **Industry Standard**: Used by Google, Uber, DigitalOcean
- ✅ **Cloud Native**: CNCF graduated project
- ✅ **Powerful Query Language**: PromQL for complex queries
- ✅ **Multi-Dimensional Data**: Labels for filtering and aggregation
- ✅ **Pull-Based Model**: Scrapes metrics from targets
- ✅ **Beautiful Dashboards**: Grafana's rich visualization
- ✅ **Alert Management**: Built-in alerting with Alertmanager
- ✅ **Extensive Exporters**: Monitor anything

---

## 🎯 Monitoring Stack Overview

```
┌─────────────────────────────────────────────┐
│         Application & Infrastructure         │
│  (Kubernetes, Servers, Apps, Databases)     │
└─────────────┬───────────────────────────────┘
              │ Exposes Metrics
              ▼
┌─────────────────────────────────────────────┐
│            Prometheus Server                 │
│  • Scrapes metrics from targets             │
│  • Stores time-series data                  │
│  • Evaluates alerting rules                 │
│  • Provides PromQL query interface          │
└─────────────┬──────────────┬────────────────┘
              │              │
              ▼              ▼
┌──────────────────┐  ┌─────────────────────┐
│   Alertmanager   │  │      Grafana        │
│  • Routes alerts │  │  • Visualizations   │
│  • Deduplicates  │  │  • Dashboards       │
│  • Notifies      │  │  • Queries PromQL   │
└──────────────────┘  └─────────────────────┘
```

---

## 🚀 Quick Start

### **Installation**

**Docker Compose (Recommended for Learning):**
```yaml
# docker-compose.yml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
    networks:
      - monitoring

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana-data:/var/lib/grafana
    networks:
      - monitoring

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    ports:
      - "9100:9100"
    networks:
      - monitoring

volumes:
  prometheus-data:
  grafana-data:

networks:
  monitoring:
```

**Prometheus Configuration:**
```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
```

**Start the Stack:**
```bash
docker-compose up -d

# Access
# Prometheus: http://localhost:9090
# Grafana: http://localhost:3000 (admin/admin)
```

---

## 📊 Core Concepts

### **1. Prometheus Architecture**

**Components:**
- **Prometheus Server**: Core component that scrapes and stores metrics
- **Exporters**: Expose metrics from systems (Node Exporter, MySQL Exporter, etc.)
- **Pushgateway**: For short-lived jobs that can't be scraped
- **Alertmanager**: Handles alerts from Prometheus
- **Client Libraries**: Instrument your applications

**Metrics Types:**
```python
from prometheus_client import Counter, Gauge, Histogram, Summary

# Counter - only increases
requests_total = Counter('http_requests_total', 'Total HTTP requests')
requests_total.inc()  # Increment by 1

# Gauge - can go up or down
temperature = Gauge('room_temperature_celsius', 'Room temperature')
temperature.set(23.5)

# Histogram - observations in buckets
request_duration = Histogram('http_request_duration_seconds', 'HTTP request duration')
request_duration.observe(0.5)  # 500ms

# Summary - similar to histogram but calculates quantiles
response_size = Summary('response_size_bytes', 'Response size')
response_size.observe(1024)
```

### **2. PromQL (Prometheus Query Language)**

**Basic Queries:**
```promql
# Instant vector - current value
up

# Range vector - values over time
up[5m]

# Rate of increase
rate(http_requests_total[5m])

# Sum by label
sum(rate(http_requests_total[5m])) by (status)

# Calculate 95th percentile
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
```

**Common Patterns:**
```promql
# CPU usage percentage
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory usage percentage
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100

# Request rate
sum(rate(http_requests_total[5m]))

# Error rate
sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))

# Average response time
rate(http_request_duration_seconds_sum[5m]) / rate(http_request_duration_seconds_count[5m])
```

### **3. Grafana Dashboards**

**Dashboard Components:**
- **Panels**: Individual visualizations (Graph, Stat, Table, etc.)
- **Rows**: Organize panels horizontally
- **Variables**: Dynamic dashboards with dropdowns
- **Annotations**: Mark events on graphs
- **Alerts**: Trigger notifications based on queries

**Example Panel Configuration:**
```json
{
  "title": "CPU Usage",
  "targets": [
    {
      "expr": "100 - (avg by (instance) (rate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)",
      "legendFormat": "{{instance}}"
    }
  ],
  "type": "graph",
  "yaxes": [
    {
      "format": "percent",
      "max": 100,
      "min": 0
    }
  ]
}
```

### **4. Alerting**

**Alert Rules (Prometheus):**
```yaml
# alerts.yml
groups:
  - name: example
    rules:
      - alert: HighCPUUsage
        expr: 100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage on {{ $labels.instance }}"
          description: "CPU usage is {{ $value }}%"

      - alert: ServiceDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Service {{ $labels.job }} is down"
```

**Alertmanager Configuration:**
```yaml
# alertmanager.yml
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname', 'cluster']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h
  receiver: 'slack'

receivers:
  - name: 'slack'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK'
        channel: '#alerts'
        title: '{{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'

  - name: 'email'
    email_configs:
      - to: 'team@example.com'
        from: 'prometheus@example.com'
        smarthost: 'smtp.gmail.com:587'
        auth_username: 'your-email@gmail.com'
        auth_password: 'your-app-password'
```

---

## 🔧 Common Use Cases

### **1. Kubernetes Monitoring**

```yaml
# ServiceMonitor for Kubernetes
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: app-monitor
spec:
  selector:
    matchLabels:
      app: myapp
  endpoints:
    - port: metrics
      interval: 30s
```

### **2. Application Monitoring**

```python
# Python Flask application
from prometheus_client import Counter, Histogram, generate_latest
from flask import Flask, Response
import time

app = Flask(__name__)

request_count = Counter('app_requests_total', 'Total requests', ['method', 'endpoint'])
request_duration = Histogram('app_request_duration_seconds', 'Request duration')

@app.route('/api/data')
@request_duration.time()
def get_data():
    request_count.labels(method='GET', endpoint='/api/data').inc()
    return {"data": "example"}

@app.route('/metrics')
def metrics():
    return Response(generate_latest(), mimetype='text/plain')
```

### **3. Infrastructure Monitoring**

**Node Exporter Metrics:**
- CPU usage
- Memory usage
- Disk I/O
- Network traffic
- System load

**Common Exporters:**
- **Node Exporter**: System metrics (CPU, memory, disk)
- **Blackbox Exporter**: HTTP/HTTPS/DNS/TCP probing
- **MySQL Exporter**: MySQL database metrics
- **PostgreSQL Exporter**: PostgreSQL metrics
- **Redis Exporter**: Redis metrics
- **Nginx Exporter**: Nginx metrics
- **Elasticsearch Exporter**: Elasticsearch cluster metrics

---

## 📚 Learning Resources

### **📖 Documentation**
1. [Learning Roadmap](prometheus-grafana-learning-roadmap.md) - 12-week structured curriculum
2. [Quick Reference](prometheus-grafana-quick-reference.md) - PromQL & commands
3. [Hands-On Exercises](prometheus-grafana-hands-on-exercises.md) - 30+ practical labs
4. [Troubleshooting Guide](prometheus-grafana-troubleshooting-guide.md) - Common issues
5. [Interview Questions](prometheus-grafana-interview-questions.md) - 80+ questions

### **🔗 Official Resources**
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [PromQL Basics](https://prometheus.io/docs/prometheus/latest/querying/basics/)
- [Grafana Dashboards](https://grafana.com/grafana/dashboards/)

---

## 🎯 Next Steps

1. **Start Learning**: Follow the [Learning Roadmap](prometheus-grafana-learning-roadmap.md)
2. **Practice**: Complete [Hands-On Exercises](prometheus-grafana-hands-on-exercises.md)
3. **Reference**: Use [Quick Reference](prometheus-grafana-quick-reference.md) guide
4. **Troubleshoot**: Check [Troubleshooting Guide](prometheus-grafana-troubleshooting-guide.md)
5. **Interview Prep**: Study [Interview Questions](prometheus-grafana-interview-questions.md)

---

**Master Monitoring & Observability with Prometheus & Grafana! 📊📈🚀**

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

