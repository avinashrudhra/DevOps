# Python Interview Questions for DevOps

100+ comprehensive interview questions for DevOps professionals (5-7+ years experience).

---

## Table of Contents
1. [Python Fundamentals](#python-fundamentals)
2. [Data Structures & Algorithms](#data-structures--algorithms)
3. [Object-Oriented Programming](#object-oriented-programming)
4. [System Administration](#system-administration)
5. [API & Web Services](#api--web-services)
6. [Docker & Kubernetes](#docker--kubernetes)
7. [Cloud Services](#cloud-services)
8. [DevOps Automation](#devops-automation)
9. [Production Scenarios](#production-scenarios)

---

## Python Fundamentals

### Q1: Explain Python's GIL (Global Interpreter Lock) and its implications for DevOps scripts

**Answer:**

The **GIL (Global Interpreter Lock)** is a mutex that protects access to Python objects, preventing multiple threads from executing Python bytecode simultaneously.

**How it Works:**

```python
import threading
import time

# This won't run in parallel due to GIL
def cpu_bound_task():
    count = 0
    for i in range(10000000):
        count += 1
    return count

# Two threads
thread1 = threading.Thread(target=cpu_bound_task)
thread2 = threading.Thread(target=cpu_bound_task)

start = time.time()
thread1.start()
thread2.start()
thread1.join()
thread2.join()
print(f"Time: {time.time() - start}s")
# Takes almost same time as sequential!
```

**Implications for DevOps:**

**1. CPU-Bound Tasks:**
- GIL prevents true parallelism for CPU-intensive tasks
- Use `multiprocessing` instead of `threading`

```python
from multiprocessing import Process
import time

def cpu_intensive_task():
    count = 0
    for i in range(10000000):
        count += 1

# Using processes (bypasses GIL)
if __name__ == '__main__':
    processes = []
    start = time.time()
    
    for _ in range(4):
        p = Process(target=cpu_intensive_task)
        p.start()
        processes.append(p)
    
    for p in processes:
        p.join()
    
    print(f"Time: {time.time() - start}s")
    # Much faster! True parallelism
```

**2. I/O-Bound Tasks (Not Affected):**
- Network requests, file I/O, database queries
- GIL is released during I/O operations
- Threading works well

```python
import threading
import requests

def check_server(url):
    response = requests.get(url)  # GIL released during I/O
    return response.status_code

# Threading is perfect for I/O
servers = ['http://server1.com', 'http://server2.com', 'http://server3.com']
threads = []

for server in servers:
    t = threading.Thread(target=check_server, args=(server,))
    t.start()
    threads.append(t)

for t in threads:
    t.join()
```

**3. DevOps Recommendations:**

| Task Type | Solution | Example |
|-----------|----------|---------|
| **CPU-Bound** | multiprocessing | Log parsing, data processing |
| **I/O-Bound** | threading or asyncio | API calls, file I/O |
| **Mixed** | multiprocessing + threading | Complex pipelines |

**Async Alternative (Modern Approach):**

```python
import asyncio
import aiohttp

async def check_server(session, url):
    async with session.get(url) as response:
        return response.status

async def check_all_servers(urls):
    async with aiohttp.ClientSession() as session:
        tasks = [check_server(session, url) for url in urls]
        results = await asyncio.gather(*tasks)
        return results

# Run
servers = ['http://server1.com', 'http://server2.com']
asyncio.run(check_all_servers(servers))
```

**Production Impact:**
- ✅ **Use threading** for monitoring scripts (API checks, health checks)
- ✅ **Use multiprocessing** for log analysis, batch processing
- ✅ **Use asyncio** for modern async I/O (thousands of connections)
- ❌ **Avoid threading** for CPU-intensive tasks

---

### Q2: What are Python's memory management mechanisms?

**Answer:**

Python uses **automatic memory management** with several key mechanisms:

**1. Reference Counting:**

Every object has a reference count. When it reaches zero, memory is freed.

```python
import sys

a = []  # Create list, refcount = 1
print(sys.getrefcount(a))  # Shows refcount (adds 1 for argument)

b = a  # Increment refcount to 2
print(sys.getrefcount(a))

del b  # Decrement refcount to 1
print(sys.getrefcount(a))

del a  # Refcount = 0, object destroyed
```

**2. Garbage Collection (Cyclic References):**

Handles circular references that reference counting can't clean.

```python
import gc

# Circular reference
class Node:
    def __init__(self):
        self.ref = None

a = Node()
b = Node()
a.ref = b
b.ref = a  # Circular reference!

# Even after deleting, objects exist due to circular ref
del a, b

# Garbage collector finds and cleans these
gc.collect()  # Force collection

# Check GC stats
print(gc.get_stats())
```

**3. Memory Pooling:**

Python uses object pools for small objects (< 512 bytes) for efficiency.

```python
# Small integers are cached (-5 to 256)
a = 100
b = 100
print(a is b)  # True - same object

a = 1000
b = 1000
print(a is b)  # False - different objects

# String interning
s1 = "hello"
s2 = "hello"
print(s1 is s2)  # True - interned
```

**DevOps Implications:**

**Memory Leaks in Long-Running Scripts:**

```python
# ❌ Bad - accumulates memory
cache = {}
def process_request(key, data):
    cache[key] = data  # Never cleaned!
    return cache[key]

# ✅ Good - bounded cache
from functools import lru_cache

@lru_cache(maxsize=1000)  # Limited size
def process_request(key):
    return expensive_operation(key)

# Or use explicit cleanup
from collections import OrderedDict

class LRUCache:
    def __init__(self, capacity):
        self.cache = OrderedDict()
        self.capacity = capacity
    
    def get(self, key):
        if key not in self.cache:
            return None
        self.cache.move_to_end(key)
        return self.cache[key]
    
    def put(self, key, value):
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value
        if len(self.cache) > self.capacity:
            self.cache.popitem(last=False)
```

**Monitoring Memory:**

```python
import tracemalloc
import psutil
import os

def monitor_memory():
    """Monitor memory usage"""
    process = psutil.Process(os.getpid())
    mem_info = process.memory_info()
    print(f"RSS: {mem_info.rss / 1024 / 1024:.2f} MB")
    print(f"VMS: {mem_info.vms / 1024 / 1024:.2f} MB")

# Trace memory allocations
tracemalloc.start()

# Your code
data = [i for i in range(1000000)]

# Get memory statistics
current, peak = tracemalloc.get_traced_memory()
print(f"Current: {current / 1024 / 1024:.2f} MB")
print(f"Peak: {peak / 1024 / 1024:.2f} MB")

tracemalloc.stop()
```

**Best Practices for DevOps:**

1. **Use context managers** for resource cleanup
2. **Implement bounded caches** in long-running services
3. **Monitor memory** in production
4. **Use generators** for large datasets
5. **Delete large objects** explicitly
6. **Profile memory** during development

---

## DevOps Automation

### Q3: Write a Python script to automate server health checks across multiple services

**Answer:**

```python
#!/usr/bin/env python3
"""
Production-grade server health check script
"""
import requests
import subprocess
import socket
import psutil
import smtplib
from email.mime.text import MIMEText
from datetime import datetime
from concurrent.futures import ThreadPoolExecutor, as_completed
from dataclasses import dataclass
from typing import List, Dict
import logging
import json

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('/var/log/health-check.log'),
        logging.StreamHandler()
    ]
)

@dataclass
class CheckResult:
    """Health check result"""
    check_name: str
    status: bool
    message: str
    response_time: float = 0.0
    timestamp: str = ""
    
    def __post_init__(self):
        if not self.timestamp:
            self.timestamp = datetime.now().isoformat()

class HealthChecker:
    """Comprehensive health checker for DevOps monitoring"""
    
    def __init__(self, config: Dict):
        self.config = config
        self.results = []
        self.failed_checks = []
    
    def check_http_endpoint(self, name: str, url: str, timeout: int = 5,
                           expected_status: int = 200) -> CheckResult:
        """Check HTTP/HTTPS endpoint"""
        try:
            start_time = datetime.now()
            response = requests.get(url, timeout=timeout)
            response_time = (datetime.now() - start_time).total_seconds()
            
            if response.status_code == expected_status:
                return CheckResult(
                    check_name=name,
                    status=True,
                    message=f"OK - Status {response.status_code}",
                    response_time=response_time
                )
            else:
                return CheckResult(
                    check_name=name,
                    status=False,
                    message=f"FAIL - Status {response.status_code}",
                    response_time=response_time
                )
        except requests.exceptions.Timeout:
            return CheckResult(
                check_name=name,
                status=False,
                message=f"FAIL - Timeout after {timeout}s"
            )
        except requests.exceptions.ConnectionError:
            return CheckResult(
                check_name=name,
                status=False,
                message="FAIL - Connection refused"
            )
        except Exception as e:
            return CheckResult(
                check_name=name,
                status=False,
                message=f"FAIL - {str(e)}"
            )
    
    def check_port(self, name: str, host: str, port: int, timeout: int = 3) -> CheckResult:
        """Check if port is accessible"""
        try:
            start_time = datetime.now()
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            sock.settimeout(timeout)
            result = sock.connect_ex((host, port))
            response_time = (datetime.now() - start_time).total_seconds()
            sock.close()
            
            if result == 0:
                return CheckResult(
                    check_name=name,
                    status=True,
                    message=f"OK - Port {port} is open",
                    response_time=response_time
                )
            else:
                return CheckResult(
                    check_name=name,
                    status=False,
                    message=f"FAIL - Port {port} is closed"
                )
        except Exception as e:
            return CheckResult(
                check_name=name,
                status=False,
                message=f"FAIL - {str(e)}"
            )
    
    def check_service(self, name: str, service_name: str) -> CheckResult:
        """Check systemd service status"""
        try:
            result = subprocess.run(
                ['systemctl', 'is-active', service_name],
                capture_output=True,
                text=True,
                timeout=5
            )
            
            if result.stdout.strip() == 'active':
                return CheckResult(
                    check_name=name,
                    status=True,
                    message=f"OK - Service {service_name} is running"
                )
            else:
                return CheckResult(
                    check_name=name,
                    status=False,
                    message=f"FAIL - Service {service_name} is {result.stdout.strip()}"
                )
        except Exception as e:
            return CheckResult(
                check_name=name,
                status=False,
                message=f"FAIL - {str(e)}"
            )
    
    def check_disk_space(self, name: str, path: str, threshold: int = 80) -> CheckResult:
        """Check disk space usage"""
        try:
            disk = psutil.disk_usage(path)
            if disk.percent < threshold:
                return CheckResult(
                    check_name=name,
                    status=True,
                    message=f"OK - Disk usage {disk.percent}% (threshold: {threshold}%)"
                )
            else:
                return CheckResult(
                    check_name=name,
                    status=False,
                    message=f"FAIL - Disk usage {disk.percent}% exceeds threshold {threshold}%"
                )
        except Exception as e:
            return CheckResult(
                check_name=name,
                status=False,
                message=f"FAIL - {str(e)}"
            )
    
    def check_memory(self, name: str, threshold: int = 90) -> CheckResult:
        """Check memory usage"""
        try:
            memory = psutil.virtual_memory()
            if memory.percent < threshold:
                return CheckResult(
                    check_name=name,
                    status=True,
                    message=f"OK - Memory usage {memory.percent}%"
                )
            else:
                return CheckResult(
                    check_name=name,
                    status=False,
                    message=f"FAIL - Memory usage {memory.percent}% exceeds threshold {threshold}%"
                )
        except Exception as e:
            return CheckResult(
                check_name=name,
                status=False,
                message=f"FAIL - {str(e)}"
            )
    
    def check_cpu(self, name: str, threshold: int = 80, interval: int = 1) -> CheckResult:
        """Check CPU usage"""
        try:
            cpu_percent = psutil.cpu_percent(interval=interval)
            if cpu_percent < threshold:
                return CheckResult(
                    check_name=name,
                    status=True,
                    message=f"OK - CPU usage {cpu_percent}%"
                )
            else:
                return CheckResult(
                    check_name=name,
                    status=False,
                    message=f"FAIL - CPU usage {cpu_percent}% exceeds threshold {threshold}%"
                )
        except Exception as e:
            return CheckResult(
                check_name=name,
                status=False,
                message=f"FAIL - {str(e)}"
            )
    
    def send_alert(self, subject: str, body: str):
        """Send email alert"""
        try:
            msg = MIMEText(body)
            msg['Subject'] = subject
            msg['From'] = self.config.get('alert_from', 'monitoring@example.com')
            msg['To'] = self.config.get('alert_to', 'admin@example.com')
            
            with smtplib.SMTP(self.config.get('smtp_server', 'localhost')) as server:
                server.send_message(msg)
            
            logging.info("Alert sent successfully")
        except Exception as e:
            logging.error(f"Failed to send alert: {e}")
    
    def run_checks(self):
        """Run all configured health checks in parallel"""
        logging.info("="*80)
        logging.info(f"Starting health checks at {datetime.now()}")
        logging.info("="*80)
        
        checks = []
        
        # HTTP endpoint checks
        for endpoint in self.config.get('http_endpoints', []):
            checks.append(('http', endpoint))
        
        # Port checks
        for port_check in self.config.get('port_checks', []):
            checks.append(('port', port_check))
        
        # Service checks
        for service in self.config.get('services', []):
            checks.append(('service', service))
        
        # System checks
        if self.config.get('check_disk'):
            checks.append(('disk', self.config['check_disk']))
        if self.config.get('check_memory'):
            checks.append(('memory', self.config['check_memory']))
        if self.config.get('check_cpu'):
            checks.append(('cpu', self.config['check_cpu']))
        
        # Run checks in parallel
        with ThreadPoolExecutor(max_workers=10) as executor:
            futures = []
            
            for check_type, check_config in checks:
                if check_type == 'http':
                    future = executor.submit(
                        self.check_http_endpoint,
                        check_config['name'],
                        check_config['url'],
                        check_config.get('timeout', 5),
                        check_config.get('expected_status', 200)
                    )
                elif check_type == 'port':
                    future = executor.submit(
                        self.check_port,
                        check_config['name'],
                        check_config['host'],
                        check_config['port']
                    )
                elif check_type == 'service':
                    future = executor.submit(
                        self.check_service,
                        check_config['name'],
                        check_config['service']
                    )
                elif check_type == 'disk':
                    future = executor.submit(
                        self.check_disk_space,
                        'Disk Space',
                        check_config['path'],
                        check_config.get('threshold', 80)
                    )
                elif check_type == 'memory':
                    future = executor.submit(
                        self.check_memory,
                        'Memory Usage',
                        check_config.get('threshold', 90)
                    )
                elif check_type == 'cpu':
                    future = executor.submit(
                        self.check_cpu,
                        'CPU Usage',
                        check_config.get('threshold', 80)
                    )
                
                futures.append(future)
            
            # Collect results
            for future in as_completed(futures):
                try:
                    result = future.result()
                    self.results.append(result)
                    
                    # Log result
                    status_symbol = "✓" if result.status else "✗"
                    log_method = logging.info if result.status else logging.error
                    log_method(f"{status_symbol} {result.check_name}: {result.message}")
                    
                    # Track failures
                    if not result.status:
                        self.failed_checks.append(result)
                except Exception as e:
                    logging.error(f"Check failed with exception: {e}")
        
        # Summary
        total_checks = len(self.results)
        passed_checks = sum(1 for r in self.results if r.status)
        failed_checks = total_checks - passed_checks
        
        logging.info("="*80)
        logging.info(f"Health Check Summary: {passed_checks}/{total_checks} passed, {failed_checks} failed")
        logging.info("="*80)
        
        # Send alert if any checks failed
        if self.failed_checks:
            alert_body = f"Health Check Alert\n\n"
            alert_body += f"Failed Checks ({len(self.failed_checks)}):\n"
            for check in self.failed_checks:
                alert_body += f"\n- {check.check_name}: {check.message}"
            
            self.send_alert("Health Check Alert", alert_body)
        
        # Save results to JSON
        self.save_results()
        
        return len(self.failed_checks) == 0
    
    def save_results(self):
        """Save results to JSON file"""
        try:
            results_dict = [
                {
                    'name': r.check_name,
                    'status': r.status,
                    'message': r.message,
                    'response_time': r.response_time,
                    'timestamp': r.timestamp
                }
                for r in self.results
            ]
            
            with open('/var/log/health-check-results.json', 'w') as f:
                json.dump(results_dict, f, indent=2)
        except Exception as e:
            logging.error(f"Failed to save results: {e}")

def main():
    """Main function"""
    config = {
        'http_endpoints': [
            {'name': 'Main Website', 'url': 'https://example.com', 'timeout': 5},
            {'name': 'API Health', 'url': 'https://api.example.com/health'},
            {'name': 'Admin Panel', 'url': 'https://admin.example.com'},
        ],
        'port_checks': [
            {'name': 'SSH', 'host': 'localhost', 'port': 22},
            {'name': 'HTTP', 'host': 'localhost', 'port': 80},
            {'name': 'HTTPS', 'host': 'localhost', 'port': 443},
            {'name': 'PostgreSQL', 'host': 'localhost', 'port': 5432},
            {'name': 'Redis', 'host': 'localhost', 'port': 6379},
        ],
        'services': [
            {'name': 'Nginx Service', 'service': 'nginx'},
            {'name': 'PostgreSQL Service', 'service': 'postgresql'},
            {'name': 'Redis Service', 'service': 'redis'},
        ],
        'check_disk': {'path': '/', 'threshold': 80},
        'check_memory': {'threshold': 90},
        'check_cpu': {'threshold': 80},
        'smtp_server': 'smtp.example.com',
        'alert_from': 'monitoring@example.com',
        'alert_to': 'admin@example.com'
    }
    
    checker = HealthChecker(config)
    success = checker.run_checks()
    
    # Exit with appropriate code
    import sys
    sys.exit(0 if success else 1)

if __name__ == '__main__':
    main()
```

**This script demonstrates:**
- ✅ Production-grade error handling
- ✅ Concurrent health checks
- ✅ Multiple check types (HTTP, ports, services, system resources)
- ✅ Logging and alerting
- ✅ JSON result persistence
- ✅ Proper exit codes
- ✅ Type hints and documentation
- ✅ Configuration management

---

**🎉 Complete Python interview preparation! You're ready for senior DevOps roles!** 🚀🐍

For more resources:
- [Learning Guide](python-learning-guide.md)
- [Quick Reference](python-quick-reference.md)
- [Hands-On Exercises](python-hands-on-exercises.md)
- [Troubleshooting Guide](python-troubleshooting-guide.md)

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

