# Python Quick Reference for DevOps

Essential Python commands, patterns, and snippets for daily DevOps tasks.

---

## 📋 Basic Syntax

```python
# Variables
name = "value"
count = 42
price = 19.99
enabled = True

# Print
print("Hello")
print(f"Value: {count}")

# Input
name = input("Enter name: ")

# Comments
# Single line comment
"""
Multi-line comment
or docstring
"""
```

---

## 🔤 Strings

```python
# Creation
s = "Hello World"
s = 'Single quotes'
s = """Multi-line
string"""

# Operations
s.upper()                 # HELLO WORLD
s.lower()                 # hello world
s.strip()                 # Remove whitespace
s.replace("World", "Python")
s.split()                 # Split on whitespace
s.split(",")              # Split on comma
s.startswith("Hello")     # True
s.endswith("World")       # True
"Hello" in s              # True

# Formatting
name = "Alice"
age = 30
f"Name: {name}, Age: {age}"
"Name: {}, Age: {}".format(name, age)
"Name: %s, Age: %d" % (name, age)

# String methods
s.find("World")           # Index or -1
s.count("l")              # Count occurrences
s.isdigit()               # Check if digits
s.isalpha()               # Check if alphabetic
s.join(['a', 'b'])        # Join list
```

---

## 📊 Lists

```python
# Creation
lst = [1, 2, 3]
lst = list(range(5))      # [0, 1, 2, 3, 4]

# Access
lst[0]                    # First element
lst[-1]                   # Last element
lst[1:3]                  # Slice [1, 2]

# Modification
lst.append(4)             # Add to end
lst.insert(0, 0)          # Insert at position
lst.extend([5, 6])        # Add multiple
lst.remove(3)             # Remove by value
lst.pop()                 # Remove and return last
lst.pop(0)                # Remove by index
lst.clear()               # Remove all

# Operations
len(lst)                  # Length
lst.sort()                # Sort in place
sorted(lst)               # Return sorted copy
lst.reverse()             # Reverse in place
lst.index(2)              # Find index
lst.count(2)              # Count occurrences

# List comprehension
[x*2 for x in range(5)]           # [0, 2, 4, 6, 8]
[x for x in range(10) if x % 2 == 0]  # Even numbers
```

---

## 📖 Dictionaries

```python
# Creation
d = {'name': 'Alice', 'age': 30}
d = dict(name='Alice', age=30)

# Access
d['name']                 # 'Alice' (KeyError if missing)
d.get('name')             # 'Alice' (None if missing)
d.get('name', 'Unknown')  # With default

# Modification
d['city'] = 'NYC'         # Add/update
d.update({'age': 31, 'job': 'DevOps'})
del d['age']              # Delete key
d.pop('name')             # Remove and return

# Operations
d.keys()                  # Dict keys
d.values()                # Dict values
d.items()                 # Key-value pairs
'name' in d               # Check if key exists
len(d)                    # Number of keys

# Iteration
for key in d:
    print(key, d[key])

for key, value in d.items():
    print(key, value)

# Dict comprehension
{x: x**2 for x in range(5)}  # {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}
```

---

## 🎯 Control Flow

```python
# if-elif-else
if condition:
    # code
elif other_condition:
    # code
else:
    # code

# Ternary
value = "yes" if condition else "no"

# for loop
for item in iterable:
    print(item)

for i in range(5):
    print(i)

for i, item in enumerate(lst):
    print(i, item)

# while loop
while condition:
    # code

# break, continue
for item in lst:
    if item == target:
        break      # Exit loop
    if item == skip:
        continue   # Skip to next iteration
```

---

## 🔧 Functions

```python
# Basic function
def greet(name):
    return f"Hello, {name}"

# Default arguments
def deploy(app, env="dev"):
    return f"Deploying {app} to {env}"

# Variable arguments
def log(*messages):
    for msg in messages:
        print(msg)

def configure(**options):
    for key, value in options.items():
        print(f"{key}={value}")

# Lambda
square = lambda x: x**2
add = lambda x, y: x + y

# Type hints
def add(a: int, b: int) -> int:
    return a + b
```

---

## 📁 File Operations

```python
# Read file
with open('file.txt', 'r') as f:
    content = f.read()

# Read lines
with open('file.txt', 'r') as f:
    lines = f.readlines()

# Read line by line
with open('file.txt', 'r') as f:
    for line in f:
        print(line.strip())

# Write file
with open('file.txt', 'w') as f:
    f.write("content\n")

# Append
with open('file.txt', 'a') as f:
    f.write("more content\n")

# JSON
import json
data = json.load(open('data.json'))
json.dump(data, open('data.json', 'w'), indent=2)

# YAML
import yaml
data = yaml.safe_load(open('config.yaml'))
yaml.dump(data, open('config.yaml', 'w'))

# CSV
import csv
with open('data.csv') as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row)
```

---

## ⚠️ Error Handling

```python
# Try-except
try:
    risky_code()
except Exception as e:
    print(f"Error: {e}")

# Multiple exceptions
try:
    code()
except ValueError:
    print("Value error")
except KeyError:
    print("Key error")

# Try-except-else-finally
try:
    code()
except Exception as e:
    print(f"Error: {e}")
else:
    print("Success")
finally:
    print("Cleanup")

# Raise exception
raise ValueError("Invalid value")

# Custom exception
class CustomError(Exception):
    pass
```

---

## 🌐 HTTP Requests

```python
import requests

# GET
response = requests.get('https://api.example.com')
data = response.json()
print(response.status_code)

# POST
data = {'key': 'value'}
response = requests.post('https://api.example.com', json=data)

# Headers
headers = {'Authorization': 'Bearer token'}
response = requests.get('https://api.example.com', headers=headers)

# Timeout
response = requests.get('https://api.example.com', timeout=5)

# Error handling
try:
    response = requests.get('https://api.example.com')
    response.raise_for_status()
except requests.exceptions.RequestException as e:
    print(f"Error: {e}")
```

---

## 💻 System Commands

```python
import subprocess
import os

# Run command
result = subprocess.run(['ls', '-la'], capture_output=True, text=True)
print(result.stdout)
print(result.returncode)

# Run shell command
output = subprocess.check_output('ps aux | grep python', shell=True, text=True)

# Environment variables
home = os.environ.get('HOME')
os.environ['MY_VAR'] = 'value'

# Execute command
os.system('ls -la')

# Path operations
os.getcwd()                     # Current directory
os.chdir('/tmp')                # Change directory
os.path.exists('/path/to/file')
os.path.isfile('/path/to/file')
os.path.isdir('/path/to/dir')
os.listdir('/path')             # List directory

# pathlib (modern)
from pathlib import Path
p = Path('/etc/config')
p.exists()
p.is_file()
p.is_dir()
list(p.glob('*.conf'))
```

---

## 🐳 Docker

```python
import docker

# Connect
client = docker.from_env()

# List containers
for container in client.containers.list():
    print(container.name)

# Run container
container = client.containers.run(
    'nginx',
    name='my-nginx',
    ports={'80/tcp': 8080},
    detach=True
)

# Stop container
container.stop()

# List images
for image in client.images.list():
    print(image.tags)

# Build image
client.images.build(path='.', tag='myapp:latest')

# Remove container
container.remove(force=True)
```

---

## ☸️ Kubernetes

```python
from kubernetes import client, config

# Load config
config.load_kube_config()

# API clients
v1 = client.CoreV1Api()
apps_v1 = client.AppsV1Api()

# List pods
pods = v1.list_namespaced_pod('default')
for pod in pods.items:
    print(f"{pod.metadata.name}: {pod.status.phase}")

# Create deployment
deployment = client.V1Deployment(
    metadata=client.V1ObjectMeta(name='nginx'),
    spec=client.V1DeploymentSpec(
        replicas=3,
        selector=client.V1LabelSelector(
            match_labels={'app': 'nginx'}
        ),
        template=client.V1PodTemplateSpec(
            metadata=client.V1ObjectMeta(labels={'app': 'nginx'}),
            spec=client.V1PodSpec(
                containers=[
                    client.V1Container(
                        name='nginx',
                        image='nginx:latest',
                        ports=[client.V1ContainerPort(container_port=80)]
                    )
                ]
            )
        )
    )
)
apps_v1.create_namespaced_deployment('default', deployment)

# Delete pod
v1.delete_namespaced_pod('pod-name', 'default')
```

---

## 📊 System Monitoring

```python
import psutil

# CPU
cpu_percent = psutil.cpu_percent(interval=1)
cpu_count = psutil.cpu_count()

# Memory
memory = psutil.virtual_memory()
print(f"Total: {memory.total}")
print(f"Available: {memory.available}")
print(f"Percent: {memory.percent}")

# Disk
disk = psutil.disk_usage('/')
print(f"Total: {disk.total}")
print(f"Used: {disk.used}")
print(f"Percent: {disk.percent}")

# Network
net_io = psutil.net_io_counters()
print(f"Bytes sent: {net_io.bytes_sent}")
print(f"Bytes recv: {net_io.bytes_recv}")

# Processes
for proc in psutil.process_iter(['pid', 'name', 'cpu_percent']):
    print(proc.info)
```

---

## 🗄️ Database

```python
# PostgreSQL
import psycopg2
conn = psycopg2.connect(
    host="localhost",
    database="mydb",
    user="user",
    password="password"
)
cur = conn.cursor()
cur.execute("SELECT * FROM users")
rows = cur.fetchall()
cur.close()
conn.close()

# MySQL
import mysql.connector
conn = mysql.connector.connect(
    host="localhost",
    user="user",
    password="password",
    database="mydb"
)
cur = conn.cursor()
cur.execute("SELECT * FROM users")
rows = cur.fetchall()

# MongoDB
from pymongo import MongoClient
client = MongoClient('mongodb://localhost:27017/')
db = client['mydb']
collection = db['users']
for doc in collection.find():
    print(doc)

# Redis
import redis
r = redis.Redis(host='localhost', port=6379, db=0)
r.set('key', 'value')
value = r.get('key')
```

---

## ☁️ AWS (boto3)

```python
import boto3

# S3
s3 = boto3.client('s3')
s3.upload_file('local.txt', 'bucket-name', 'remote.txt')
s3.download_file('bucket-name', 'remote.txt', 'local.txt')
s3.list_objects(Bucket='bucket-name')

# EC2
ec2 = boto3.client('ec2')
instances = ec2.describe_instances()
ec2.start_instances(InstanceIds=['i-1234567890abcdef0'])
ec2.stop_instances(InstanceIds=['i-1234567890abcdef0'])

# Lambda
lambda_client = boto3.client('lambda')
response = lambda_client.invoke(
    FunctionName='my-function',
    InvocationType='RequestResponse',
    Payload=json.dumps({'key': 'value'})
)
```

---

## 🔄 Common Patterns

### Configuration Management
```python
import configparser

config = configparser.ConfigParser()
config.read('config.ini')
host = config['database']['host']
port = config.getint('database', 'port')
```

### Logging
```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s',
    filename='app.log'
)

logging.debug("Debug message")
logging.info("Info message")
logging.warning("Warning message")
logging.error("Error message")
logging.critical("Critical message")
```

### Date & Time
```python
from datetime import datetime, timedelta
import time

# Current time
now = datetime.now()
print(now.strftime('%Y-%m-%d %H:%M:%S'))

# Parse date
dt = datetime.strptime('2024-01-01', '%Y-%m-%d')

# Time operations
tomorrow = now + timedelta(days=1)
yesterday = now - timedelta(days=1)

# Unix timestamp
timestamp = time.time()
dt = datetime.fromtimestamp(timestamp)

# Sleep
time.sleep(5)  # Sleep 5 seconds
```

### Regular Expressions
```python
import re

# Match
match = re.search(r'\d+', 'Server 123')
if match:
    print(match.group())  # 123

# Find all
numbers = re.findall(r'\d+', 'Server 123 Port 8080')
# ['123', '8080']

# Replace
text = re.sub(r'\d+', 'XXX', 'Server 123')
# 'Server XXX'

# Compile for reuse
pattern = re.compile(r'\d+\.\d+\.\d+\.\d+')
ips = pattern.findall(log_text)
```

---

**Quick reference complete! 🐍✨**

For more resources:
- [Learning Guide](python-learning-guide.md)
- [Hands-On Exercises](python-hands-on-exercises.md)
- [Troubleshooting Guide](python-troubleshooting-guide.md)
- [Interview Questions](python-interview-questions.md)

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

