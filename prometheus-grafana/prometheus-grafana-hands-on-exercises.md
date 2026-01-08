# Prometheus & Grafana Hands-On Exercises

**30+ Practical Labs from Basic Setup to Production Monitoring**

---

## 🎯 Exercise Structure

Each exercise includes:
- **Objective**: What you'll learn
- **Prerequisites**: What you need
- **Steps**: Detailed instructions
- **Validation**: How to verify success
- **Challenge**: Extend your learning

---

## 📚 Lab Environment Setup

### **Lab 0: Setup Development Environment**

**Objective:** Create a complete monitoring lab environment

**Prerequisites:**
- Docker installed
- 8GB RAM minimum
- Basic Linux knowledge

**Setup:**

```bash
# Create project directory
mkdir prometheus-lab && cd prometheus-lab

# Create docker-compose.yml
cat > docker-compose.yml <<EOF
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus:/etc/prometheus
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.enable-lifecycle'
    networks:
      - monitoring

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_USERS_ALLOW_SIGN_UP=false
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

  alertmanager:
    image: prom/alertmanager:latest
    container_name: alertmanager
    ports:
      - "9093:9093"
    volumes:
      - ./alertmanager:/etc/alertmanager
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
    networks:
      - monitoring

  blackbox-exporter:
    image: prom/blackbox-exporter:latest
    container_name: blackbox-exporter
    ports:
      - "9115:9115"
    volumes:
      - ./blackbox:/etc/blackbox_exporter
    networks:
      - monitoring

volumes:
  prometheus-data:
  grafana-data:

networks:
  monitoring:
    driver: bridge
EOF

# Create directories
mkdir -p prometheus alertmanager blackbox

# Create basic prometheus config
cat > prometheus/prometheus.yml <<EOF
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
EOF

# Create basic alertmanager config
cat > alertmanager/alertmanager.yml <<EOF
global:
  resolve_timeout: 5m

route:
  receiver: 'default'
  
receivers:
  - name: 'default'
EOF

# Start the stack
docker-compose up -d

# Verify
docker-compose ps
```

**Validation:**
```bash
# Check Prometheus
curl http://localhost:9090/-/healthy

# Check Grafana
curl http://localhost:3000/api/health

# Access UIs
# Prometheus: http://localhost:9090
# Grafana: http://localhost:3000 (admin/admin)
```

---

## 🔰 Beginner Exercises

### **Exercise 1: Basic Metric Collection**

**Objective:** Configure Prometheus to scrape multiple targets

**Steps:**

```bash
# 1. Add a sample application with metrics
docker run -d --name sample-app \
  --network prometheus-lab_monitoring \
  -p 8080:8080 \
  -e METRICS_PORT=8080 \
  pierrev/sample-metrics-app

# 2. Update prometheus/prometheus.yml
cat >> prometheus/prometheus.yml <<EOF

  - job_name: 'sample-app'
    static_configs:
      - targets: ['sample-app:8080']
        labels:
          env: 'lab'
          app: 'sample'
EOF

# 3. Reload Prometheus
curl -X POST http://localhost:9090/-/reload

# 4. Verify in Prometheus UI
# Go to http://localhost:9090/targets
```

**Practice Queries:**
```promql
# 1. Check if target is up
up{job="sample-app"}

# 2. View all metrics from sample-app
{job="sample-app"}

# 3. Filter by label
up{env="lab"}
```

**Challenge:**
- Add 3 more targets with different labels
- Create a query that shows only production targets
- Group metrics by environment

---

### **Exercise 2: Understanding Metric Types**

**Objective:** Create and understand Counter, Gauge, Histogram, and Summary

**Steps:**

```python
# Create custom_exporter.py
from prometheus_client import Counter, Gauge, Histogram, Summary
from prometheus_client import start_http_server
import time
import random

# Define metrics
request_count = Counter('app_requests_total', 'Total requests', ['method', 'endpoint', 'status'])
active_users = Gauge('app_active_users', 'Currently active users')
request_duration = Histogram('app_request_duration_seconds', 'Request duration', 
                             buckets=[0.1, 0.5, 1.0, 2.0, 5.0])
response_size = Summary('app_response_size_bytes', 'Response size')

def simulate_traffic():
    while True:
        # Counter - increases only
        method = random.choice(['GET', 'POST', 'PUT'])
        endpoint = random.choice(['/api/users', '/api/products', '/api/orders'])
        status = random.choice(['200', '200', '200', '404', '500'])
        request_count.labels(method=method, endpoint=endpoint, status=status).inc()
        
        # Gauge - can go up or down
        active_users.set(random.randint(50, 200))
        
        # Histogram - observations
        request_duration.observe(random.uniform(0.1, 3.0))
        
        # Summary - observations
        response_size.observe(random.randint(100, 5000))
        
        time.sleep(1)

if __name__ == '__main__':
    start_http_server(8001)
    print("Custom exporter running on port 8001")
    simulate_traffic()
```

```bash
# Run the exporter
python3 custom_exporter.py &

# Add to Prometheus
cat >> prometheus/prometheus.yml <<EOF

  - job_name: 'custom-exporter'
    static_configs:
      - targets: ['host.docker.internal:8001']
EOF

curl -X POST http://localhost:9090/-/reload
```

**Practice Queries:**
```promql
# Counter - rate of requests
rate(app_requests_total[5m])

# Counter - total requests by status
sum by (status) (app_requests_total)

# Gauge - current active users
app_active_users

# Histogram - 95th percentile duration
histogram_quantile(0.95, rate(app_request_duration_seconds_bucket[5m]))

# Histogram - average duration
rate(app_request_duration_seconds_sum[5m]) / rate(app_request_duration_seconds_count[5m])

# Summary - quantiles
app_response_size_bytes{quantile="0.5"}
app_response_size_bytes{quantile="0.9"}
app_response_size_bytes{quantile="0.99"}
```

**Challenge:**
- Create a business metric (e.g., revenue_total)
- Add more labels to track different dimensions
- Calculate error rate percentage

---

### **Exercise 3: PromQL Fundamentals**

**Objective:** Master basic PromQL queries

**Practice Queries:**

```promql
# 1. Label Selectors
up{job="prometheus"}
up{job!="prometheus"}
up{job=~"node.*"}
up{job=~"prometheus|node-exporter"}

# 2. Range Vectors
up[5m]
rate(app_requests_total[5m])

# 3. Aggregation
sum(up)
count(up)
avg(node_cpu_seconds_total)
max(app_active_users)
min(app_active_users)

# 4. Grouping
sum by (job) (up)
count by (status) (app_requests_total)
avg by (mode) (rate(node_cpu_seconds_total[5m]))

# 5. Math Operations
app_active_users * 2
app_active_users / 2
app_active_users + 100

# 6. Comparison
app_active_users > 100
up == 1
up != 0

# 7. Sorting
sort(app_requests_total)
sort_desc(app_requests_total)
topk(3, app_requests_total)
bottomk(2, app_requests_total)
```

**Challenge Tasks:**
1. Find request rate per second for each endpoint
2. Calculate the percentage of 5xx errors
3. List top 5 endpoints by request count
4. Find average request duration in milliseconds
5. Show only metrics where active_users > 150

**Solutions:**
```promql
# 1. Request rate per endpoint
sum by (endpoint) (rate(app_requests_total[5m]))

# 2. Error percentage
sum(rate(app_requests_total{status=~"5.."}[5m])) / sum(rate(app_requests_total[5m])) * 100

# 3. Top 5 endpoints
topk(5, sum by (endpoint) (app_requests_total))

# 4. Average duration in ms
rate(app_request_duration_seconds_sum[5m]) / rate(app_request_duration_seconds_count[5m]) * 1000

# 5. Filter active users
app_active_users > 150
```

---

## 🚀 Intermediate Exercises

### **Exercise 4: Create Your First Dashboard**

**Objective:** Build a comprehensive Grafana dashboard

**Steps:**

```bash
# 1. Login to Grafana (http://localhost:3000)
# Username: admin, Password: admin

# 2. Add Prometheus data source
# Configuration → Data Sources → Add data source
# Select Prometheus
# URL: http://prometheus:9090
# Click "Save & Test"

# 3. Create new dashboard
# Create → Dashboard → Add new panel
```

**Panel 1: Total Requests (Stat)**
```json
Panel Type: Stat
Query: sum(app_requests_total)
Title: Total Requests
Color Mode: Background
Graph Mode: Area
```

**Panel 2: Request Rate (Graph)**
```json
Panel Type: Time series
Query: sum(rate(app_requests_total[5m]))
Title: Request Rate (req/s)
Y-axis: reqps
Legend: {{job}}
```

**Panel 3: Active Users (Gauge)**
```json
Panel Type: Gauge
Query: app_active_users
Title: Active Users
Min: 0
Max: 300
Thresholds: 0=green, 100=yellow, 200=red
```

**Panel 4: Error Rate (Graph)**
```json
Panel Type: Time series
Query: sum(rate(app_requests_total{status=~"5.."}[5m])) / sum(rate(app_requests_total[5m])) * 100
Title: Error Rate (%)
Y-axis: percent
Color: Red
```

**Panel 5: Request Duration (Heatmap)**
```json
Panel Type: Heatmap
Query: rate(app_request_duration_seconds_bucket[5m])
Title: Request Duration Distribution
Format: Time series buckets
```

**Challenge:**
- Add 5 more panels
- Create dashboard variables
- Set up panel links
- Add text documentation
- Export dashboard as JSON

---

### **Exercise 5: Configure Alerting**

**Objective:** Set up alert rules and notifications

**Steps:**

```yaml
# 1. Create prometheus/alerts.yml
groups:
  - name: application_alerts
    rules:
      # High error rate
      - alert: HighErrorRate
        expr: |
          sum(rate(app_requests_total{status=~"5.."}[5m])) 
          / 
          sum(rate(app_requests_total[5m])) * 100 > 5
        for: 2m
        labels:
          severity: warning
          team: backend
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value | humanize }}%"
      
      # High latency
      - alert: HighLatency
        expr: |
          histogram_quantile(0.95, 
            sum(rate(app_request_duration_seconds_bucket[5m])) by (le)
          ) > 2
        for: 5m
        labels:
          severity: warning
          team: backend
        annotations:
          summary: "High latency detected"
          description: "95th percentile latency is {{ $value | humanize }}s"
      
      # Service down
      - alert: ServiceDown
        expr: up{job="sample-app"} == 0
        for: 1m
        labels:
          severity: critical
          team: sre
        annotations:
          summary: "Service {{ $labels.job }} is down"
          description: "{{ $labels.instance }} has been down for more than 1 minute"
      
      # High active users
      - alert: HighActiveUsers
        expr: app_active_users > 180
        for: 3m
        labels:
          severity: info
          team: product
        annotations:
          summary: "High number of active users"
          description: "Active users: {{ $value }}"
```

```yaml
# 2. Update prometheus/prometheus.yml
rule_files:
  - 'alerts.yml'

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']
```

```yaml
# 3. Configure alertmanager/alertmanager.yml
global:
  resolve_timeout: 5m

route:
  receiver: 'default'
  group_by: ['alertname', 'team']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  
  routes:
    - match:
        severity: critical
      receiver: 'critical-alerts'
      repeat_interval: 5m
    
    - match:
        severity: warning
      receiver: 'warning-alerts'
      repeat_interval: 30m
    
    - match:
        severity: info
      receiver: 'info-alerts'
      repeat_interval: 4h

receivers:
  - name: 'default'
    webhook_configs:
      - url: 'http://webhook.site/your-unique-id'
        send_resolved: true
  
  - name: 'critical-alerts'
    webhook_configs:
      - url: 'http://webhook.site/critical'
  
  - name: 'warning-alerts'
    webhook_configs:
      - url: 'http://webhook.site/warning'
  
  - name: 'info-alerts'
    webhook_configs:
      - url: 'http://webhook.site/info'

inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'instance']
```

```bash
# 4. Reload configurations
curl -X POST http://localhost:9090/-/reload
docker-compose restart alertmanager
```

**Validation:**
```bash
# Check alerts in Prometheus
# http://localhost:9090/alerts

# Check Alertmanager
# http://localhost:9093

# Trigger an alert (stop service)
docker stop sample-app

# Wait 1 minute and check alerts
# Restart service
docker start sample-app
```

**Challenge:**
- Add Slack integration
- Create silence rules
- Test inhibition rules
- Add email notifications
- Create custom alert templates

---

### **Exercise 6: Advanced PromQL**

**Objective:** Master complex queries and recording rules

**Advanced Queries:**

```promql
# 1. Calculate SLA/SLO
# Availability: 99.9% over 30 days
sum(rate(app_requests_total{status!~"5.."}[30d])) 
/ 
sum(rate(app_requests_total[30d])) * 100 > 99.9

# 2. Error budget consumption
1 - (
  (1 - (sum(rate(app_requests_total{status!~"5.."}[7d])) / sum(rate(app_requests_total[7d]))))
  /
  (1 - 0.999)
)

# 3. Predict when metric will cross threshold
predict_linear(app_active_users[30m], 3600) > 250

# 4. Rate of change
deriv(app_active_users[5m])

# 5. Absent metrics (detect missing metrics)
absent(up{job="sample-app"})

# 6. Multiple aggregations
sum without(instance) (
  rate(app_requests_total[5m])
) > 10

# 7. Vector matching
method_code:app_requests:rate5m / on(method) group_left method:app_requests:total > 0.05

# 8. Subqueries
max_over_time(
  rate(app_requests_total[5m])[30m:1m]
)
```

**Create Recording Rules:**

```yaml
# prometheus/recording_rules.yml
groups:
  - name: app_rules
    interval: 30s
    rules:
      # Request rate by endpoint
      - record: endpoint:app_requests:rate5m
        expr: sum by (endpoint) (rate(app_requests_total[5m]))
      
      # Total request rate
      - record: job:app_requests:rate5m
        expr: sum(rate(app_requests_total[5m]))
      
      # Error rate
      - record: job:app_requests:error_rate5m
        expr: |
          sum(rate(app_requests_total{status=~"5.."}[5m]))
          /
          sum(rate(app_requests_total[5m]))
      
      # Average latency
      - record: job:app_request:duration_avg5m
        expr: |
          rate(app_request_duration_seconds_sum[5m])
          /
          rate(app_request_duration_seconds_count[5m])
      
      # 95th percentile latency
      - record: job:app_request:duration_p95_5m
        expr: |
          histogram_quantile(0.95,
            sum(rate(app_request_duration_seconds_bucket[5m])) by (le)
          )
```

```yaml
# Update prometheus/prometheus.yml
rule_files:
  - 'alerts.yml'
  - 'recording_rules.yml'
```

**Challenge:**
- Create 10 more recording rules
- Use recording rules in alerts
- Create multi-level recording rules
- Optimize expensive queries

---

## 🎓 Advanced Exercises

### **Exercise 7: Kubernetes Monitoring**

**Objective:** Monitor Kubernetes cluster with Prometheus

**Prerequisites:**
- Kubernetes cluster (Minikube, Kind, or Cloud)
- kubectl configured

**Steps:**

```bash
# 1. Deploy Prometheus Operator
kubectl create namespace monitoring

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --set prometheus.prometheusSpec.retention=7d \
  --set prometheus.prometheusSpec.storageSpec.volumeClaimTemplate.spec.resources.requests.storage=20Gi

# 2. Port-forward to access
kubectl port-forward -n monitoring svc/prometheus-kube-prometheus-prometheus 9090:9090
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80

# 3. Deploy sample application
kubectl create deployment sample-app --image=nginx --replicas=3
kubectl expose deployment sample-app --port=80 --type=ClusterIP

# 4. Create ServiceMonitor
cat <<EOF | kubectl apply -f -
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: sample-app
  labels:
    app: sample-app
spec:
  selector:
    matchLabels:
      app: sample-app
  endpoints:
    - port: http
      interval: 30s
      path: /metrics
EOF
```

**Key Kubernetes Queries:**

```promql
# Cluster health
kube_node_status_condition{condition="Ready", status="true"}

# Pod status
count by (phase) (kube_pod_status_phase)

# CPU usage by namespace
sum by (namespace) (rate(container_cpu_usage_seconds_total[5m]))

# Memory usage by pod
sum by (namespace, pod) (container_memory_usage_bytes)

# Container restarts
sum by (namespace, pod) (rate(kube_pod_container_status_restarts_total[15m]))

# Pods not ready
count(kube_pod_status_phase{phase!="Running"})

# Resource requests vs limits
sum(kube_pod_container_resource_requests_cpu_cores) / sum(kube_node_status_allocatable_cpu_cores) * 100
```

**Challenge:**
- Create Kubernetes-specific dashboards
- Set up alerts for pod failures
- Monitor ingress controllers
- Track PVC usage

---

### **Exercise 8: High Availability Setup**

**Objective:** Configure HA Prometheus with Thanos

**Steps:**

```yaml
# thanos-docker-compose.yml
version: '3.8'

services:
  prometheus-1:
    image: prom/prometheus:latest
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.min-block-duration=2h'
      - '--storage.tsdb.max-block-duration=2h'
    volumes:
      - ./prometheus:/etc/prometheus
      - prom1-data:/prometheus
    
  prometheus-2:
    image: prom/prometheus:latest
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.min-block-duration=2h'
      - '--storage.tsdb.max-block-duration=2h'
    volumes:
      - ./prometheus:/etc/prometheus
      - prom2-data:/prometheus
  
  thanos-sidecar-1:
    image: thanosio/thanos:latest
    command:
      - 'sidecar'
      - '--prometheus.url=http://prometheus-1:9090'
      - '--tsdb.path=/prometheus'
      - '--grpc-address=0.0.0.0:10901'
      - '--http-address=0.0.0.0:10902'
    volumes:
      - prom1-data:/prometheus
  
  thanos-query:
    image: thanosio/thanos:latest
    command:
      - 'query'
      - '--http-address=0.0.0.0:9090'
      - '--store=thanos-sidecar-1:10901'
      - '--store=thanos-sidecar-2:10901'
    ports:
      - '9090:9090'

volumes:
  prom1-data:
  prom2-data:
```

**Challenge:**
- Add S3 backend for long-term storage
- Configure compaction
- Set up Thanos Query Frontend
- Implement deduplication

---

### **Exercise 9: Custom Exporter Development**

**Objective:** Build a production-grade custom exporter

**Requirements:**
- Monitor custom application
- Expose business metrics
- Handle errors gracefully
- Include labels for filtering

**Implementation:**

```python
# production_exporter.py
from prometheus_client import start_http_server, Gauge, Counter, Histogram, Info
from prometheus_client import generate_latest, REGISTRY
import time
import logging
import argparse
import signal
import sys

# Configure logging
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')
logger = logging.getLogger(__name__)

# Define metrics
app_info = Info('app_version', 'Application version info')
scrape_duration = Histogram('app_scrape_duration_seconds', 'Time spent collecting metrics')
scrape_errors = Counter('app_scrape_errors_total', 'Total scrape errors')
business_revenue = Gauge('business_revenue_usd', 'Total revenue', ['product', 'region'])
business_orders = Counter('business_orders_total', 'Total orders', ['product', 'status'])

class CustomExporter:
    def __init__(self, port=8000):
        self.port = port
        self.running = True
        
        # Set app info
        app_info.info({'version': '1.0.0', 'environment': 'production'})
    
    @scrape_duration.time()
    def collect_metrics(self):
        try:
            # Collect business metrics
            # In real scenario, fetch from database/API
            business_revenue.labels(product='product-a', region='us-east').set(125000.50)
            business_revenue.labels(product='product-b', region='eu-west').set(89000.75)
            
            business_orders.labels(product='product-a', status='completed').inc(5)
            business_orders.labels(product='product-b', status='pending').inc(2)
            
            logger.info("Metrics collected successfully")
        except Exception as e:
            scrape_errors.inc()
            logger.error(f"Error collecting metrics: {e}")
    
    def run(self):
        start_http_server(self.port)
        logger.info(f"Exporter running on port {self.port}")
        
        while self.running:
            self.collect_metrics()
            time.sleep(15)
    
    def shutdown(self, signum, frame):
        logger.info("Shutting down exporter...")
        self.running = False
        sys.exit(0)

if __name__ == '__main__':
    parser = argparse.ArgumentParser(description='Custom Prometheus Exporter')
    parser.add_argument('--port', type=int, default=8000, help='Port to run exporter on')
    args = parser.parse_args()
    
    exporter = CustomExporter(port=args.port)
    
    # Handle graceful shutdown
    signal.signal(signal.SIGINT, exporter.shutdown)
    signal.signal(signal.SIGTERM, exporter.shutdown)
    
    exporter.run()
```

**Challenge:**
- Add health check endpoint
- Implement metric caching
- Add authentication
- Create Helm chart for deployment

---

### **Exercise 10: Full Stack Monitoring Project**

**Objective:** Build complete monitoring for microservices application

**Architecture:**
```
Frontend → API Gateway → Services → Database
                ↓
          Prometheus
                ↓
            Grafana
```

**Requirements:**
1. Monitor all components
2. Create comprehensive dashboards
3. Set up intelligent alerting
4. Implement SLO tracking
5. Add distributed tracing

**Deliverables:**
- [ ] Full monitoring stack deployed
- [ ] 5+ dashboards created
- [ ] 20+ alert rules configured
- [ ] SLI/SLO tracking implemented
- [ ] Documentation completed

---

## 🎯 Validation Checklist

After completing exercises, you should be able to:

**Beginner Level:**
- [ ] Install and configure Prometheus
- [ ] Understand all metric types
- [ ] Write basic PromQL queries
- [ ] Create simple Grafana dashboards
- [ ] Configure basic alerts

**Intermediate Level:**
- [ ] Write complex PromQL queries
- [ ] Create advanced dashboards with variables
- [ ] Configure Alertmanager with routing
- [ ] Set up recording rules
- [ ] Monitor Kubernetes clusters

**Advanced Level:**
- [ ] Design HA monitoring architecture
- [ ] Build custom exporters
- [ ] Optimize query performance
- [ ] Implement long-term storage
- [ ] Create comprehensive monitoring solutions

---

**Practice makes perfect! Complete all exercises for mastery! 📊🚀**

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

