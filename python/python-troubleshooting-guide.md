# Python Troubleshooting Guide

Comprehensive solutions to common Python issues in DevOps environments.

---

## Table of Contents
1. [Installation & Environment Issues](#installation--environment-issues)
2. [Import Errors](#import-errors)
3. [Syntax & Runtime Errors](#syntax--runtime-errors)
4. [File Operations](#file-operations)
5. [API & Network Issues](#api--network-issues)
6. [Memory & Performance](#memory--performance)
7. [Docker & Kubernetes](#docker--kubernetes)
8. [Database Issues](#database-issues)
9. [Cloud Service Errors](#cloud-service-errors)
10. [Production Debugging](#production-debugging)

---

## Installation & Environment Issues

### Issue 1: ModuleNotFoundError

**Symptoms:**
```
ModuleNotFoundError: No module named 'requests'
```

**Solutions:**

**1. Install the module:**
```bash
pip install requests

# Or with specific version
pip install requests==2.28.0

# From requirements.txt
pip install -r requirements.txt
```

**2. Check which pip you're using:**
```bash
# Verify pip version
which pip
pip --version

# Use python -m pip (safer)
python3 -m pip install requests
```

**3. Virtual environment issues:**
```bash
# Create virtual environment
python3 -m venv myenv

# Activate
source myenv/bin/activate  # Linux/macOS
myenv\Scripts\activate     # Windows

# Verify you're in venv
which python
which pip

# Install in venv
pip install requests
```

---

### Issue 2: Permission Denied

**Symptoms:**
```
ERROR: Could not install packages due to an PermissionError
```

**Solutions:**

**1. Use user install:**
```bash
pip install --user requests
```

**2. Use virtual environment (recommended):**
```bash
python3 -m venv myenv
source myenv/bin/activate
pip install requests
```

**3. Fix permissions (Linux):**
```bash
# Change ownership of Python site-packages
sudo chown -R $USER:$USER ~/.local/lib/python3.*/
```

---

### Issue 3: Multiple Python Versions

**Diagnosis:**
```bash
# Check installed Python versions
python --version
python2 --version
python3 --version

# Check which python
which python
which python3

# List all Python installations
ls -la /usr/bin/python*
```

**Solutions:**

**1. Use python3 explicitly:**
```bash
# Always use python3
python3 script.py
python3 -m pip install package

# Create alias (add to ~/.bashrc)
alias python=python3
alias pip=pip3
```

**2. Use pyenv (version manager):**
```bash
# Install pyenv
curl https://pyenv.run | bash

# Install specific Python version
pyenv install 3.11.0
pyenv global 3.11.0
pyenv local 3.11.0
```

---

## Import Errors

### Issue 4: Circular Import

**Symptoms:**
```
ImportError: cannot import name 'function' from partially initialized module
```

**Example Problem:**
```python
# file_a.py
from file_b import func_b

def func_a():
    return func_b()

# file_b.py
from file_a import func_a  # Circular import!

def func_b():
    return func_a()
```

**Solutions:**

**1. Restructure imports:**
```python
# file_a.py
def func_a():
    from file_b import func_b  # Import inside function
    return func_b()

# Or move to a third file
# utils.py
def func_a():
    pass

def func_b():
    pass
```

**2. Use import at the end:**
```python
# file_a.py
def func_a():
    pass

from file_b import func_b  # At the end
```

---

### Issue 5: Module Not in Path

**Symptoms:**
```
ModuleNotFoundError: No module named 'mymodule'
```

**Diagnosis:**
```python
import sys
print('\n'.join(sys.path))
```

**Solutions:**

**1. Add to PYTHONPATH:**
```bash
export PYTHONPATH="/path/to/module:$PYTHONPATH"

# Or in script
import sys
sys.path.insert(0, '/path/to/module')
```

**2. Use relative imports:**
```python
# In package structure
from . import module_name
from ..parent import module_name
```

**3. Install as package:**
```bash
# Create setup.py
pip install -e .  # Editable install
```

---

## Syntax & Runtime Errors

### Issue 6: IndentationError

**Symptoms:**
```
IndentationError: unexpected indent
```

**Solutions:**

**1. Check consistent indentation:**
```python
# ❌ Bad - mixing tabs and spaces
def function():
    if condition:
        return True  # 4 spaces
	return False     # Tab character

# ✅ Good - consistent spaces
def function():
    if condition:
        return True  # 4 spaces
    return False     # 4 spaces
```

**2. Configure editor:**
```bash
# Set editor to use spaces
# .vimrc
set expandtab
set tabstop=4
set shiftwidth=4

# VS Code: settings.json
"editor.insertSpaces": true,
"editor.tabSize": 4
```

---

### Issue 7: TypeError

**Common TypeErrors:**

**1. None has no attribute:**
```python
# Problem
result = some_function()  # Returns None
result.method()  # TypeError: 'NoneType' object has no attribute 'method'

# Solution
result = some_function()
if result is not None:
    result.method()
```

**2. Unsupported operand types:**
```python
# Problem
"Port: " + 8080  # TypeError: can only concatenate str to str

# Solution
"Port: " + str(8080)
f"Port: {8080}"
```

**3. Missing required argument:**
```python
# Problem
def deploy(app, env):
    pass

deploy("myapp")  # TypeError: missing 1 required positional argument

# Solution
deploy("myapp", "prod")

# Or use default
def deploy(app, env="dev"):
    pass
```

---

## File Operations

### Issue 8: File Not Found

**Symptoms:**
```
FileNotFoundError: [Errno 2] No such file or directory: 'config.txt'
```

**Solutions:**

**1. Check file exists:**
```python
import os

file_path = 'config.txt'
if os.path.exists(file_path):
    with open(file_path) as f:
        content = f.read()
else:
    print(f"File not found: {file_path}")
```

**2. Use absolute paths:**
```python
import os

# Get script directory
script_dir = os.path.dirname(os.path.abspath(__file__))
config_path = os.path.join(script_dir, 'config.txt')

with open(config_path) as f:
    content = f.read()
```

**3. Use pathlib:**
```python
from pathlib import Path

config_file = Path(__file__).parent / 'config.txt'
if config_file.exists():
    content = config_file.read_text()
```

---

### Issue 9: Permission Denied on File

**Symptoms:**
```
PermissionError: [Errno 13] Permission denied: '/etc/config'
```

**Solutions:**

**1. Check permissions:**
```bash
ls -l /etc/config
# -rw-r--r-- root root /etc/config
```

**2. Run with sudo (if necessary):**
```bash
sudo python3 script.py
```

**3. Handle exception:**
```python
try:
    with open('/etc/config', 'r') as f:
        content = f.read()
except PermissionError:
    print("Permission denied. Run with sudo.")
    sys.exit(1)
```

---

## API & Network Issues

### Issue 10: Connection Timeout

**Symptoms:**
```
requests.exceptions.ConnectionError: Connection timed out
```

**Solutions:**

**1. Increase timeout:**
```python
import requests

try:
    response = requests.get('https://api.example.com', timeout=30)
except requests.exceptions.Timeout:
    print("Request timed out")
```

**2. Retry logic:**
```python
from requests.adapters import HTTPAdapter
from requests.packages.urllib3.util.retry import Retry

session = requests.Session()
retry = Retry(
    total=3,
    backoff_factor=1,
    status_forcelist=[500, 502, 503, 504]
)
adapter = HTTPAdapter(max_retries=retry)
session.mount('http://', adapter)
session.mount('https://', adapter)

response = session.get('https://api.example.com')
```

**3. Check connectivity:**
```python
import socket

def check_connectivity(host, port):
    try:
        socket.create_connection((host, port), timeout=5)
        return True
    except OSError:
        return False

if check_connectivity('api.example.com', 443):
    # Make request
    pass
```

---

### Issue 11: SSL Certificate Error

**Symptoms:**
```
SSLError: [SSL: CERTIFICATE_VERIFY_FAILED]
```

**Solutions:**

**1. Update certificates:**
```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install ca-certificates

# macOS
/Applications/Python\ 3.x/Install\ Certificates.command
```

**2. Disable verification (not recommended for production):**
```python
import requests
import urllib3

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
response = requests.get('https://api.example.com', verify=False)
```

**3. Specify certificate bundle:**
```python
response = requests.get(
    'https://api.example.com',
    verify='/path/to/certfile'
)
```

---

## Memory & Performance

### Issue 12: Memory Error

**Symptoms:**
```
MemoryError: Unable to allocate array
```

**Solutions:**

**1. Process file in chunks:**
```python
# ❌ Bad - loads entire file
with open('large_file.txt') as f:
    content = f.read()  # Loads all into memory

# ✅ Good - process line by line
with open('large_file.txt') as f:
    for line in f:
        process(line)  # Processes one line at a time
```

**2. Use generators:**
```python
# ❌ Bad - creates full list in memory
def get_numbers():
    return [x for x in range(1000000)]

# ✅ Good - yields one at a time
def get_numbers():
    for x in range(1000000):
        yield x

for num in get_numbers():
    process(num)
```

**3. Clear large objects:**
```python
import gc

large_data = process_data()
# Use large_data
del large_data
gc.collect()  # Force garbage collection
```

---

### Issue 13: Slow Script Performance

**Diagnosis:**
```python
import cProfile
import pstats

# Profile your code
cProfile.run('your_function()', 'output.stats')

# View results
stats = pstats.Stats('output.stats')
stats.sort_stats('cumulative')
stats.print_stats(10)  # Top 10 functions
```

**Solutions:**

**1. Use list comprehensions:**
```python
# Slower
result = []
for i in range(1000):
    result.append(i * 2)

# Faster
result = [i * 2 for i in range(1000)]
```

**2. Use local variables:**
```python
# Slower - module lookup each time
for i in range(1000):
    math.sqrt(i)

# Faster - local reference
sqrt = math.sqrt
for i in range(1000):
    sqrt(i)
```

**3. Use appropriate data structures:**
```python
# Slow for membership testing
items_list = [1, 2, 3, 4, 5]
if 3 in items_list:  # O(n)
    pass

# Fast for membership testing
items_set = {1, 2, 3, 4, 5}
if 3 in items_set:  # O(1)
    pass
```

---

## Docker & Kubernetes

### Issue 14: Docker Connection Error

**Symptoms:**
```
docker.errors.DockerException: Error while fetching server API version
```

**Solutions:**

**1. Check Docker is running:**
```bash
systemctl status docker
sudo systemctl start docker
```

**2. Check permissions:**
```bash
# Add user to docker group
sudo usermod -aG docker $USER
# Log out and back in

# Or use sudo
sudo python3 script.py
```

**3. Check Docker socket:**
```bash
ls -la /var/run/docker.sock
# srw-rw---- 1 root docker /var/run/docker.sock
```

---

### Issue 15: Kubernetes Authentication Error

**Symptoms:**
```
kubernetes.config.config_exception.ConfigException: Invalid kube-config file
```

**Solutions:**

**1. Check kubeconfig:**
```bash
kubectl config view
kubectl cluster-info

# Set KUBECONFIG
export KUBECONFIG=~/.kube/config
```

**2. Use in-cluster config:**
```python
from kubernetes import client, config

try:
    config.load_incluster_config()  # Running in cluster
except:
    config.load_kube_config()  # Running outside cluster
```

**3. Check permissions:**
```bash
ls -la ~/.kube/config
# Should be 600
chmod 600 ~/.kube/config
```

---

## Production Debugging

### Issue 16: Script Fails Silently

**Solutions:**

**1. Enable logging:**
```python
import logging

logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    filename='/var/log/myapp.log'
)

logger = logging.getLogger(__name__)

try:
    result = risky_operation()
    logger.info(f"Operation succeeded: {result}")
except Exception as e:
    logger.error(f"Operation failed: {e}", exc_info=True)
```

**2. Add exception handling:**
```python
import sys
import traceback

try:
    main()
except Exception as e:
    print(f"Fatal error: {e}", file=sys.stderr)
    traceback.print_exc()
    sys.exit(1)
```

**3. Use exit codes:**
```python
import sys

def main():
    try:
        # Your code
        return 0  # Success
    except Exception as e:
        print(f"Error: {e}", file=sys.stderr)
        return 1  # Failure

if __name__ == '__main__':
    sys.exit(main())
```

---

**Complete Python troubleshooting expertise! 🔧🐍**

For more resources:
- [Learning Guide](python-learning-guide.md)
- [Quick Reference](python-quick-reference.md)
- [Hands-On Exercises](python-hands-on-exercises.md)
- [Interview Questions](python-interview-questions.md)

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

