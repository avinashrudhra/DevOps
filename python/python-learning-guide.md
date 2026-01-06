# Python Learning Guide for DevOps

Comprehensive topic-based guide to mastering Python for automation and DevOps tasks.

---

## 📚 Table of Contents

1. [Python Fundamentals](#python-fundamentals)
2. [Data Structures](#data-structures)
3. [Functions & Modules](#functions--modules)
4. [File Operations](#file-operations)
5. [Error Handling](#error-handling)
6. [Working with APIs](#working-with-apis)
7. [System Administration](#system-administration)
8. [Database Operations](#database-operations)
9. [Network Programming](#network-programming)
10. [Cloud Integration](#cloud-integration)
11. [Container & Orchestration](#container--orchestration)
12. [Testing & Debugging](#testing--debugging)

---

## Python Fundamentals

### Variables & Data Types

```python
# Basic types
name = "DevOps Engineer"          # String
age = 30                          # Integer
salary = 120000.50                # Float
is_active = True                  # Boolean
nothing = None                    # NoneType

# Type checking
print(type(name))                 # <class 'str'>
print(isinstance(age, int))       # True

# Type conversion
port = "8080"
port_num = int(port)             # Convert to int
version = str(1.0)               # Convert to string
```

### String Operations

```python
# String methods
server = "web-server-01"
print(server.upper())             # WEB-SERVER-01
print(server.lower())             # web-server-01
print(server.replace("-", "_"))  # web_server_01
print(server.split("-"))          # ['web', 'server', '01']

# String formatting
name = "nginx"
version = "1.21"

# f-strings (Python 3.6+, recommended)
message = f"Running {name} version {version}"

# format() method
message = "Running {} version {}".format(name, version)

# % formatting (old style)
message = "Running %s version %s" % (name, version)

# Multi-line strings
script = """
#!/bin/bash
echo "Hello"
exit 0
"""

# Raw strings (for regex)
pattern = r'\d{3}-\d{3}-\d{4}'
```

### Operators

```python
# Arithmetic
result = 10 + 5    # Addition
result = 10 - 5    # Subtraction
result = 10 * 5    # Multiplication
result = 10 / 5    # Division (float)
result = 10 // 5   # Floor division
result = 10 % 3    # Modulus
result = 10 ** 2   # Power

# Comparison
is_equal = (5 == 5)        # True
is_not_equal = (5 != 3)    # True
greater = (10 > 5)         # True
less_equal = (5 <= 10)     # True

# Logical
result = True and False    # False
result = True or False     # True
result = not True          # False

# Membership
servers = ['web1', 'web2']
'web1' in servers          # True
'db1' not in servers       # True
```

### Control Flow

```python
# if-elif-else
cpu_usage = 85

if cpu_usage > 90:
    print("Critical: CPU usage very high")
elif cpu_usage > 75:
    print("Warning: CPU usage high")
else:
    print("OK: CPU usage normal")

# Ternary operator
status = "up" if cpu_usage < 80 else "down"

# match-case (Python 3.10+)
http_code = 404
match http_code:
    case 200:
        print("OK")
    case 404:
        print("Not Found")
    case 500:
        print("Server Error")
    case _:
        print("Unknown code")
```

### Loops

```python
# for loop
servers = ['web1', 'web2', 'web3']
for server in servers:
    print(f"Checking {server}")

# for loop with index
for index, server in enumerate(servers):
    print(f"{index}: {server}")

# for loop with range
for i in range(5):              # 0 to 4
    print(i)

for i in range(1, 10, 2):      # 1, 3, 5, 7, 9
    print(i)

# while loop
count = 0
while count < 5:
    print(count)
    count += 1

# break and continue
for i in range(10):
    if i == 3:
        continue  # Skip 3
    if i == 7:
        break     # Stop at 7
    print(i)

# Loop with else
for server in servers:
    if check_server(server):
        print(f"{server} is up")
        break
else:
    print("No servers are up")  # Executes if no break
```

---

## Data Structures

### Lists (Arrays)

```python
# Creating lists
servers = ['web1', 'web2', 'web3']
ports = [80, 443, 8080]
mixed = ['server', 80, True, None]

# Accessing elements
first = servers[0]              # web1
last = servers[-1]              # web3
slice = servers[0:2]            # ['web1', 'web2']

# Modifying lists
servers.append('web4')          # Add to end
servers.insert(0, 'lb1')        # Insert at position
servers.remove('web2')          # Remove by value
servers.pop()                   # Remove and return last
servers.pop(0)                  # Remove by index

# List operations
length = len(servers)           # Number of elements
servers.sort()                  # Sort in place
sorted_list = sorted(servers)   # Return sorted copy
servers.reverse()               # Reverse in place

# List comprehension (powerful!)
# Create list of uppercase server names
upper_servers = [s.upper() for s in servers]

# Filter list
active_servers = [s for s in servers if 'web' in s]

# With conditional
statuses = ['up' if s.startswith('web') else 'down' for s in servers]
```

### Dictionaries (Hash Maps)

```python
# Creating dictionaries
server = {
    'hostname': 'web-01',
    'ip': '192.168.1.10',
    'port': 80,
    'status': 'running'
}

# Accessing values
hostname = server['hostname']           # Raises KeyError if missing
hostname = server.get('hostname')       # Returns None if missing
hostname = server.get('hostname', 'unknown')  # With default

# Modifying dictionaries
server['cpu'] = 45                      # Add/update key
del server['port']                      # Delete key
server.pop('status')                    # Remove and return value
server.update({'memory': 8192, 'disk': 500})  # Update multiple

# Dictionary operations
keys = server.keys()                    # Dict keys
values = server.values()                # Dict values
items = server.items()                  # Key-value pairs

# Iterating dictionaries
for key in server:
    print(f"{key}: {server[key]}")

for key, value in server.items():
    print(f"{key}: {value}")

# Dictionary comprehension
ports_dict = {f'server{i}': 8000+i for i in range(3)}
# {'server0': 8000, 'server1': 8001, 'server2': 8002}

# Nested dictionaries
config = {
    'webservers': {
        'web1': {'ip': '192.168.1.10', 'port': 80},
        'web2': {'ip': '192.168.1.11', 'port': 80}
    },
    'databases': {
        'db1': {'ip': '192.168.1.20', 'port': 5432}
    }
}
```

### Sets

```python
# Creating sets (unique, unordered)
active_servers = {'web1', 'web2', 'web3'}
monitored_servers = {'web2', 'web3', 'db1'}

# Set operations
union = active_servers | monitored_servers        # All servers
intersection = active_servers & monitored_servers # Common servers
difference = active_servers - monitored_servers   # In active but not monitored
symmetric_diff = active_servers ^ monitored_servers  # In one but not both

# Set methods
active_servers.add('web4')              # Add element
active_servers.remove('web1')           # Remove (raises KeyError if missing)
active_servers.discard('web1')          # Remove (no error if missing)

# Set comprehension
ports = {8000 + i for i in range(5)}    # {8000, 8001, 8002, 8003, 8004}
```

### Tuples (Immutable Lists)

```python
# Creating tuples
server = ('web1', '192.168.1.10', 80)
singleton = (42,)  # Note the comma

# Accessing elements
hostname = server[0]
ip = server[1]

# Tuple unpacking
hostname, ip, port = server

# Named tuples (more readable)
from collections import namedtuple

Server = namedtuple('Server', ['hostname', 'ip', 'port'])
server = Server('web1', '192.168.1.10', 80)
print(server.hostname)  # web1
print(server.ip)        # 192.168.1.10
```

---

## Functions & Modules

### Defining Functions

```python
# Basic function
def greet(name):
    """Greet a user"""
    return f"Hello, {name}!"

# Function with default arguments
def deploy(app_name, version="latest", env="dev"):
    """Deploy application"""
    print(f"Deploying {app_name}:{version} to {env}")
    return True

deploy("myapp")                          # Uses defaults
deploy("myapp", "1.0.0")                 # Overrides version
deploy("myapp", version="2.0", env="prod")  # Named arguments

# Variable arguments
def log_message(*args):
    """Log multiple messages"""
    for msg in args:
        print(msg)

log_message("Starting", "Processing", "Complete")

# Keyword arguments
def configure_server(**kwargs):
    """Configure server with arbitrary options"""
    for key, value in kwargs.items():
        print(f"{key} = {value}")

configure_server(hostname="web1", port=80, ssl=True)

# Type hints (Python 3.5+)
def check_port(host: str, port: int) -> bool:
    """Check if port is open"""
    # Implementation
    return True

# Lambda functions (anonymous)
square = lambda x: x ** 2
print(square(5))  # 25

# Sort with lambda
servers = [{'name': 'web2', 'priority': 2}, 
           {'name': 'web1', 'priority': 1}]
sorted_servers = sorted(servers, key=lambda s: s['priority'])
```

### Decorators

```python
# Function decorator
def log_execution(func):
    """Decorator to log function execution"""
    def wrapper(*args, **kwargs):
        print(f"Executing {func.__name__}")
        result = func(*args, **kwargs)
        print(f"Completed {func.__name__}")
        return result
    return wrapper

@log_execution
def deploy_app(app_name):
    print(f"Deploying {app_name}")

deploy_app("myapp")
# Output:
# Executing deploy_app
# Deploying myapp
# Completed deploy_app

# Timer decorator
import time
def timer(func):
    """Measure execution time"""
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} took {end-start:.2f} seconds")
        return result
    return wrapper

@timer
def process_logs():
    time.sleep(2)
    print("Processing logs")

# Retry decorator
def retry(max_attempts=3):
    """Retry decorator with configurable attempts"""
    def decorator(func):
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    print(f"Attempt {attempt + 1} failed, retrying...")
            return None
        return wrapper
    return decorator

@retry(max_attempts=3)
def flaky_api_call():
    # Might fail
    return requests.get('https://api.example.com')
```

### Modules & Packages

```python
# Importing modules
import os                    # Import entire module
import sys as system         # Import with alias
from os import path          # Import specific function
from os import *             # Import everything (avoid!)

# Using imported modules
current_dir = os.getcwd()
file_exists = path.exists('/tmp/file.txt')

# Creating your own module
# File: server_utils.py
def check_server(hostname):
    """Check if server is reachable"""
    return True

def restart_server(hostname):
    """Restart server"""
    return True

# Using your module
# In another file:
import server_utils
server_utils.check_server('web1')

# Or:
from server_utils import check_server
check_server('web1')

# Package structure
# myproject/
#   ├── __init__.py
#   ├── servers/
#   │   ├── __init__.py
#   │   ├── web.py
#   │   └── database.py
#   └── monitoring/
#       ├── __init__.py
#       └── metrics.py

# Usage
from myproject.servers import web
from myproject.monitoring import metrics
```

---

## File Operations

### Reading Files

```python
# Read entire file
with open('config.txt', 'r') as f:
    content = f.read()
    print(content)

# Read lines
with open('config.txt', 'r') as f:
    lines = f.readlines()  # List of lines
    for line in lines:
        print(line.strip())

# Read line by line (memory efficient)
with open('large_log.txt', 'r') as f:
    for line in f:
        if 'ERROR' in line:
            print(line.strip())

# Read specific number of bytes
with open('data.bin', 'rb') as f:
    chunk = f.read(1024)  # Read 1KB
```

### Writing Files

```python
# Write to file (overwrites)
with open('output.txt', 'w') as f:
    f.write("Line 1\n")
    f.write("Line 2\n")

# Append to file
with open('log.txt', 'a') as f:
    f.write(f"Log entry at {time.time()}\n")

# Write lines
lines = ['Line 1\n', 'Line 2\n', 'Line 3\n']
with open('output.txt', 'w') as f:
    f.writelines(lines)

# Binary mode
with open('image.png', 'rb') as f_in:
    with open('copy.png', 'wb') as f_out:
        f_out.write(f_in.read())
```

### Working with JSON

```python
import json

# Read JSON
with open('config.json', 'r') as f:
    config = json.load(f)

# Parse JSON string
json_string = '{"name": "web1", "port": 80}'
data = json.loads(json_string)

# Write JSON
data = {
    'servers': ['web1', 'web2'],
    'port': 80,
    'enabled': True
}
with open('config.json', 'w') as f:
    json.dump(data, f, indent=2)

# Convert to JSON string
json_string = json.dumps(data, indent=2)

# Pretty print JSON
print(json.dumps(data, indent=2, sort_keys=True))
```

### Working with YAML

```python
import yaml

# Read YAML
with open('config.yaml', 'r') as f:
    config = yaml.safe_load(f)

# Write YAML
data = {
    'servers': {
        'web': ['web1', 'web2'],
        'db': ['db1']
    },
    'port': 80
}
with open('config.yaml', 'w') as f:
    yaml.dump(data, f, default_flow_style=False)
```

### CSV Operations

```python
import csv

# Read CSV
with open('servers.csv', 'r') as f:
    reader = csv.reader(f)
    headers = next(reader)  # Skip header
    for row in reader:
        print(row)

# Read CSV as dictionary
with open('servers.csv', 'r') as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row['hostname'], row['ip'])

# Write CSV
servers = [
    ['hostname', 'ip', 'port'],
    ['web1', '192.168.1.10', '80'],
    ['web2', '192.168.1.11', '80']
]
with open('servers.csv', 'w', newline='') as f:
    writer = csv.writer(f)
    writer.writerows(servers)

# Write CSV from dictionaries
servers = [
    {'hostname': 'web1', 'ip': '192.168.1.10'},
    {'hostname': 'web2', 'ip': '192.168.1.11'}
]
with open('servers.csv', 'w', newline='') as f:
    fieldnames = ['hostname', 'ip']
    writer = csv.DictWriter(f, fieldnames=fieldnames)
    writer.writeheader()
    writer.writerows(servers)
```

### Path Operations

```python
import os
from pathlib import Path

# os.path methods
current_dir = os.getcwd()
home_dir = os.path.expanduser('~')
abs_path = os.path.abspath('file.txt')
dir_name = os.path.dirname('/path/to/file.txt')
base_name = os.path.basename('/path/to/file.txt')
exists = os.path.exists('/path/to/file')
is_file = os.path.isfile('/path/to/file')
is_dir = os.path.isdir('/path/to/directory')

# Join paths (OS-independent)
config_path = os.path.join('/etc', 'myapp', 'config.ini')

# pathlib (modern, recommended)
p = Path('/etc/myapp/config.ini')
print(p.parent)         # /etc/myapp
print(p.name)           # config.ini
print(p.suffix)         # .ini
print(p.stem)           # config
print(p.exists())       # True/False

# Create directory
Path('/tmp/myapp').mkdir(parents=True, exist_ok=True)

# List files
for file in Path('/etc').glob('*.conf'):
    print(file)

# Recursive search
for file in Path('/var/log').rglob('*.log'):
    print(file)
```

---

## Error Handling

### Try-Except Blocks

```python
# Basic try-except
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero")

# Multiple exceptions
try:
    file = open('config.txt')
    data = json.load(file)
except FileNotFoundError:
    print("File not found")
except json.JSONDecodeError:
    print("Invalid JSON")

# Catch multiple exception types
try:
    # Some code
    pass
except (FileNotFoundError, PermissionError) as e:
    print(f"File error: {e}")

# Catch all exceptions (use sparingly)
try:
    risky_operation()
except Exception as e:
    print(f"Error occurred: {e}")

# Try-except-else-finally
try:
    file = open('config.txt')
    data = file.read()
except FileNotFoundError:
    print("File not found")
else:
    print("File read successfully")
    # Executes if no exception
finally:
    print("Cleanup code")
    # Always executes
    try:
        file.close()
    except:
        pass

# Raise exceptions
def check_port(port):
    if not (1 <= port <= 65535):
        raise ValueError(f"Invalid port: {port}")
    return port

# Custom exceptions
class ServerConnectionError(Exception):
    """Custom exception for server connection errors"""
    pass

def connect_to_server(hostname):
    if not hostname:
        raise ServerConnectionError("Hostname cannot be empty")
```

---

## Working with APIs

### REST API with requests

```python
import requests

# GET request
response = requests.get('https://api.github.com/users/octocat')
print(response.status_code)  # 200
print(response.headers)
print(response.json())       # Parse JSON response

# Query parameters
params = {'q': 'python', 'sort': 'stars'}
response = requests.get('https://api.github.com/search/repositories', params=params)

# POST request
data = {'username': 'admin', 'password': 'secret'}
response = requests.post('https://api.example.com/login', json=data)

# Headers
headers = {
    'Authorization': 'Bearer token123',
    'Content-Type': 'application/json'
}
response = requests.get('https://api.example.com/data', headers=headers)

# Upload file
files = {'file': open('report.pdf', 'rb')}
response = requests.post('https://api.example.com/upload', files=files)

# Session (persist cookies/headers)
session = requests.Session()
session.headers.update({'Authorization': 'Bearer token123'})
response = session.get('https://api.example.com/data')

# Timeouts
try:
    response = requests.get('https://api.example.com', timeout=5)
except requests.exceptions.Timeout:
    print("Request timed out")

# Error handling
try:
    response = requests.get('https://api.example.com')
    response.raise_for_status()  # Raises HTTPError for bad status
except requests.exceptions.HTTPError as e:
    print(f"HTTP Error: {e}")
except requests.exceptions.ConnectionError:
    print("Connection Error")
except requests.exceptions.RequestException as e:
    print(f"Error: {e}")
```

---

**Continue to other sections for System Administration, Docker, Kubernetes integration, and more!** 🐍💪

For complete reference:
- [Quick Reference](python-quick-reference.md)
- [Hands-On Exercises](python-hands-on-exercises.md)
- [Troubleshooting Guide](python-troubleshooting-guide.md)
- [Interview Questions](python-interview-questions.md)

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

