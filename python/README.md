# Python Scripting for DevOps

Complete guide to mastering Python for automation, scripting, and DevOps tasks.

---

## 🐍 What is Python?

**Python** is a high-level, interpreted programming language that's become the standard for:
- **DevOps Automation**: Infrastructure scripts, deployment automation
- **System Administration**: Server management, log analysis
- **Data Processing**: Log parsing, metrics collection
- **API Integration**: REST APIs, cloud service SDKs
- **Testing**: Unit tests, integration tests, infrastructure tests
- **Machine Learning/AI**: MLOps, AIOps

**Why Python for DevOps?**
- ✅ **Easy to Learn**: Clear, readable syntax
- ✅ **Powerful Libraries**: Extensive ecosystem
- ✅ **Cross-Platform**: Windows, Linux, macOS
- ✅ **Integration**: Works with all DevOps tools
- ✅ **Community**: Massive community support
- ✅ **Versatile**: From simple scripts to complex applications
- ✅ **Industry Standard**: Used by Google, Netflix, NASA, etc.

---

## 🎯 Python in DevOps Ecosystem

```
Python Powers Everything:

┌─────────────────────────────────────────┐
│          Python Scripting               │
│      (The Glue of DevOps)              │
└───────────┬─────────────────────────────┘
            │
    ┌───────┴────────┬─────────┬──────────┐
    │                │         │          │
    ▼                ▼         ▼          ▼
┌─────────┐    ┌─────────┐ ┌──────┐  ┌────────┐
│ Ansible │    │ Jenkins │ │Docker│  │  K8s   │
│ Modules │    │ Scripts │ │ SDK  │  │  API   │
└─────────┘    └─────────┘ └──────┘  └────────┘
    │                │         │          │
    └────────┬───────┴─────────┴──────────┘
             ▼
    ┌──────────────────┐
    │   Automation     │
    │   Monitoring     │
    │   Orchestration  │
    └──────────────────┘
```

---

## 🚀 Quick Start

### **Installation**

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install python3 python3-pip python3-venv

# RHEL/CentOS
sudo yum install python3 python3-pip

# macOS
brew install python3

# Windows (Use Python.org installer or)
choco install python

# Verify
python3 --version
pip3 --version
```

### **Virtual Environment (Best Practice)**

```bash
# Create virtual environment
python3 -m venv myenv

# Activate
source myenv/bin/activate  # Linux/macOS
myenv\Scripts\activate     # Windows

# Install packages
pip install requests ansible docker kubernetes

# Deactivate
deactivate

# Requirements file
pip freeze > requirements.txt
pip install -r requirements.txt
```

---

## 📦 Essential Python for DevOps

### **1. Basic Syntax**

```python
#!/usr/bin/env python3
"""
Simple Python script for DevOps
"""

# Variables
server = "web-server-01"
port = 8080
enabled = True

# String formatting
print(f"Connecting to {server} on port {port}")

# Conditionals
if enabled:
    print("Server is enabled")
else:
    print("Server is disabled")

# Loops
servers = ["web1", "web2", "web3"]
for server in servers:
    print(f"Checking {server}...")

# Functions
def check_server(hostname, port):
    """Check if server is reachable"""
    print(f"Checking {hostname}:{port}")
    return True

result = check_server("web1", 80)
```

### **2. File Operations**

```python
import os
import json
import yaml

# Read file
with open('config.txt', 'r') as f:
    content = f.read()

# Write file
with open('output.txt', 'w') as f:
    f.write("Hello DevOps\n")

# Append
with open('log.txt', 'a') as f:
    f.write("New log entry\n")

# Read JSON
with open('config.json', 'r') as f:
    config = json.load(f)

# Write JSON
data = {'server': 'web1', 'port': 80}
with open('config.json', 'w') as f:
    json.dump(data, f, indent=2)

# YAML
import yaml
with open('config.yaml', 'r') as f:
    config = yaml.safe_load(f)
```

### **3. System Commands**

```python
import subprocess
import os

# Run command
result = subprocess.run(['ls', '-la'], capture_output=True, text=True)
print(result.stdout)

# Check return code
if result.returncode == 0:
    print("Success")
else:
    print("Failed")

# Run shell command
output = subprocess.check_output('df -h', shell=True, text=True)
print(output)

# Environment variables
home = os.environ.get('HOME')
path = os.environ.get('PATH')

# Set environment variable
os.environ['MY_VAR'] = 'value'
```

### **4. Working with APIs**

```python
import requests

# GET request
response = requests.get('https://api.github.com/users/octocat')
if response.status_code == 200:
    data = response.json()
    print(data['name'])

# POST request
payload = {'key': 'value'}
response = requests.post('https://api.example.com/data', json=payload)

# Headers
headers = {'Authorization': 'Bearer token123'}
response = requests.get('https://api.example.com/data', headers=headers)

# Error handling
try:
    response = requests.get('https://api.example.com', timeout=5)
    response.raise_for_status()
except requests.exceptions.RequestException as e:
    print(f"Error: {e}")
```

### **5. Docker Integration**

```python
import docker

# Connect to Docker
client = docker.from_env()

# List containers
for container in client.containers.list():
    print(f"{container.name}: {container.status}")

# Run container
container = client.containers.run(
    'nginx',
    name='my-nginx',
    ports={'80/tcp': 8080},
    detach=True
)

# Stop container
container.stop()

# Build image
client.images.build(path='.', tag='myapp:latest')
```

### **6. Kubernetes Integration**

```python
from kubernetes import client, config

# Load config
config.load_kube_config()

# Create API client
v1 = client.CoreV1Api()

# List pods
pods = v1.list_namespaced_pod('default')
for pod in pods.items:
    print(f"{pod.metadata.name}: {pod.status.phase}")

# Create deployment
apps_v1 = client.AppsV1Api()
deployment = {
    'apiVersion': 'apps/v1',
    'kind': 'Deployment',
    'metadata': {'name': 'nginx'},
    'spec': {
        'replicas': 3,
        'selector': {'matchLabels': {'app': 'nginx'}},
        'template': {
            'metadata': {'labels': {'app': 'nginx'}},
            'spec': {
                'containers': [{
                    'name': 'nginx',
                    'image': 'nginx:latest',
                    'ports': [{'containerPort': 80}]
                }]
            }
        }
    }
}
apps_v1.create_namespaced_deployment('default', deployment)
```

---

## 🔧 Common DevOps Scripts

### **System Monitor**

```python
#!/usr/bin/env python3
import psutil
import time

def monitor_system():
    """Monitor system resources"""
    while True:
        # CPU
        cpu_percent = psutil.cpu_percent(interval=1)
        
        # Memory
        memory = psutil.virtual_memory()
        memory_percent = memory.percent
        
        # Disk
        disk = psutil.disk_usage('/')
        disk_percent = disk.percent
        
        print(f"CPU: {cpu_percent}% | Memory: {memory_percent}% | Disk: {disk_percent}%")
        
        # Alert if high
        if cpu_percent > 80:
            print("⚠️ High CPU usage!")
        if memory_percent > 80:
            print("⚠️ High memory usage!")
            
        time.sleep(5)

if __name__ == '__main__':
    monitor_system()
```

### **Log Parser**

```python
#!/usr/bin/env python3
import re
from collections import Counter

def parse_nginx_logs(logfile):
    """Parse Nginx access logs"""
    ips = []
    status_codes = []
    
    with open(logfile, 'r') as f:
        for line in f:
            # Extract IP
            ip_match = re.search(r'(\d+\.\d+\.\d+\.\d+)', line)
            if ip_match:
                ips.append(ip_match.group(1))
            
            # Extract status code
            status_match = re.search(r'" (\d{3}) ', line)
            if status_match:
                status_codes.append(status_match.group(1))
    
    # Count occurrences
    ip_counts = Counter(ips)
    status_counts = Counter(status_codes)
    
    print("Top 10 IPs:")
    for ip, count in ip_counts.most_common(10):
        print(f"  {ip}: {count}")
    
    print("\nStatus Codes:")
    for code, count in status_counts.items():
        print(f"  {code}: {count}")

if __name__ == '__main__':
    parse_nginx_logs('/var/log/nginx/access.log')
```

### **Backup Script**

```python
#!/usr/bin/env python3
import os
import tarfile
from datetime import datetime
import boto3  # AWS S3

def backup_directory(source_dir, backup_dir, s3_bucket=None):
    """Backup directory and optionally upload to S3"""
    
    # Create backup filename
    timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
    backup_name = f"backup_{timestamp}.tar.gz"
    backup_path = os.path.join(backup_dir, backup_name)
    
    # Create tarball
    print(f"Creating backup: {backup_path}")
    with tarfile.open(backup_path, 'w:gz') as tar:
        tar.add(source_dir, arcname=os.path.basename(source_dir))
    
    print(f"Backup created: {backup_path}")
    
    # Upload to S3 if specified
    if s3_bucket:
        print(f"Uploading to S3: {s3_bucket}")
        s3 = boto3.client('s3')
        s3.upload_file(backup_path, s3_bucket, backup_name)
        print("Upload complete")
    
    return backup_path

if __name__ == '__main__':
    backup_directory('/opt/myapp', '/backup', 's3://my-backups')
```

---

## 📚 Learning Resources

### **📖 Documentation**
1. [Python Learning Guide](python-learning-guide.md) - Comprehensive topic-based guide
2. [Python Quick Reference](python-quick-reference.md) - Commands and patterns
3. [Hands-On Exercises](python-hands-on-exercises.md) - 40+ practical projects
4. [Troubleshooting Guide](python-troubleshooting-guide.md) - Common issues
5. [Interview Questions](python-interview-questions.md) - 100+ questions

### **🔗 Official Resources**
- [Python.org](https://www.python.org/)
- [Python Documentation](https://docs.python.org/3/)
- [PyPI (Python Package Index)](https://pypi.org/)
- [Real Python](https://realpython.com/)

---

## 🎯 Python for Each DevOps Tool

### **For Ansible**
```python
# Ansible dynamic inventory
import json
inventory = {
    'webservers': {'hosts': ['web1', 'web2']},
    'databases': {'hosts': ['db1']}
}
print(json.dumps(inventory))
```

### **For Jenkins**
```python
# Jenkins API automation
import requests
jenkins_url = 'http://jenkins.example.com'
job_name = 'deploy-prod'
requests.post(f'{jenkins_url}/job/{job_name}/build')
```

### **For Docker**
```python
# Docker automation
import docker
client = docker.from_env()
client.containers.run('nginx', detach=True)
```

### **For Kubernetes**
```python
# Kubernetes automation
from kubernetes import client, config
config.load_kube_config()
v1 = client.CoreV1Api()
v1.list_pod_for_all_namespaces()
```

---

## 🎓 Next Steps

1. **Start Learning**: Follow the [Learning Guide](python-learning-guide.md)
2. **Practice**: Complete [Hands-On Exercises](python-hands-on-exercises.md)
3. **Reference**: Use [Quick Reference](python-quick-reference.md) guide
4. **Troubleshoot**: Check [Troubleshooting Guide](python-troubleshooting-guide.md)
5. **Interview Prep**: Study [Interview Questions](python-interview-questions.md)

---

**Master Python for DevOps Automation! 🐍💪🚀**

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

