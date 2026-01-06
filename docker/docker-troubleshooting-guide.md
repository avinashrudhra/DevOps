# Docker Troubleshooting Guide

Common Docker problems and their solutions.

---

## Table of Contents
1. [Installation Issues](#installation-issues)
2. [Container Problems](#container-problems)
3. [Image Issues](#image-issues)
4. [Network Problems](#network-problems)
5. [Volume & Storage Issues](#volume--storage-issues)
6. [Performance Problems](#performance-problems)
7. [Docker Compose Issues](#docker-compose-issues)
8. [Security & Permissions](#security--permissions)

---

## Installation Issues

### Issue 1: "Cannot connect to Docker daemon"
**Problem**: Docker client can't reach daemon

**Symptoms:**
```
Cannot connect to the Docker daemon at unix:///var/run/docker.sock.
Is the docker daemon running?
```

**Solutions:**

**Solution 1: Start Docker Service**
```bash
# Linux
sudo systemctl start docker
sudo systemctl enable docker

# macOS/Windows
# Start Docker Desktop application

# Verify
sudo systemctl status docker
```

**Solution 2: Add User to Docker Group**
```bash
# Add current user to docker group
sudo usermod -aG docker $USER

# Log out and back in, or:
newgrp docker

# Verify
docker ps
```

**Solution 3: Check Docker Socket**
```bash
# Check socket permissions
ls -l /var/run/docker.sock

# Fix permissions if needed
sudo chmod 666 /var/run/docker.sock

# Or restart Docker
sudo systemctl restart docker
```

---

### Issue 2: "Got permission denied"
**Problem**: Permission errors when running Docker

**Symptoms:**
```
Got permission denied while trying to connect to the Docker daemon socket
```

**Solutions:**
```bash
# Option 1: Add user to docker group (recommended)
sudo usermod -aG docker $USER
newgrp docker

# Option 2: Run with sudo (not recommended)
sudo docker ps

# Option 3: Fix socket permissions
sudo chmod 666 /var/run/docker.sock

# Verify group membership
groups $USER | grep docker
```

---

### Issue 3: Docker Desktop Won't Start (macOS/Windows)
**Problem**: Docker Desktop fails to start

**Solutions:**

**macOS:**
```bash
# Check logs
cat ~/Library/Containers/com.docker.docker/Data/log/vm/dockerd.log

# Reset Docker Desktop
# Settings → Troubleshoot → Reset to factory defaults

# Reinstall
brew uninstall docker
brew install --cask docker
```

**Windows:**
```powershell
# Enable WSL 2
wsl --install
wsl --set-default-version 2

# Check Hyper-V
# Windows Features → Enable Hyper-V

# Reset Docker Desktop
# Settings → Troubleshoot → Reset to factory defaults
```

---

## Container Problems

### Issue 4: Container Exits Immediately
**Problem**: Container starts but exits right away

**Symptoms:**
```bash
docker run myimage
# Container exits immediately

docker ps -a
# Status: Exited (0) or Exited (1)
```

**Solutions:**

**Solution 1: Check Logs**
```bash
# View container logs
docker logs container-name

# Check exit code
docker inspect container-name --format='{{.State.ExitCode}}'
```

**Solution 2: Keep Container Running**
```bash
# Interactive mode
docker run -it myimage /bin/bash

# Keep stdin open
docker run -d -i myimage

# Use proper CMD
# Dockerfile: CMD ["nginx", "-g", "daemon off;"]
```

**Solution 3: Fix Application**
```dockerfile
# Bad: Process exits
CMD ["echo", "hello"]

# Good: Long-running process
CMD ["nginx", "-g", "daemon off;"]
CMD ["tail", "-f", "/dev/null"]
```

---

### Issue 5: "Port is already allocated"
**Problem**: Port conflict on host

**Symptoms:**
```
Error: bind: address already in use
docker: Error response from daemon: driver failed programming external 
connectivity on endpoint: Bind for 0.0.0.0:8080 failed: port is already allocated
```

**Solutions:**

**Solution 1: Find Process Using Port**
```bash
# Linux/macOS
sudo lsof -i :8080
sudo netstat -tuln | grep 8080

# Kill process
sudo kill -9 <PID>

# Windows
netstat -ano | findstr :8080
taskkill /PID <PID> /F
```

**Solution 2: Use Different Port**
```bash
# Use different host port
docker run -p 8081:80 nginx
```

**Solution 3: Stop Conflicting Container**
```bash
# Find container using port
docker ps
docker port container-name

# Stop container
docker stop container-name
```

---

### Issue 6: Container Out of Memory
**Problem**: Container killed by OOM

**Symptoms:**
```bash
docker logs container-name
# Killed

docker inspect container-name
# OOMKilled: true
```

**Solutions:**

**Solution 1: Increase Memory Limit**
```bash
# Set memory limit
docker run -m 1g myapp

# Set memory + swap
docker run -m 1g --memory-swap 2g myapp

# No limit (use with caution)
docker run -m 0 myapp
```

**Solution 2: Monitor Memory**
```bash
# Real-time stats
docker stats container-name

# Check memory usage
docker inspect -f '{{.HostConfig.Memory}}' container-name
```

**Solution 3: Fix Application**
```javascript
// Bad: Memory leak
let cache = [];
setInterval(() => {
    cache.push(new Array(1000000));
}, 1000);

// Good: Limit cache size
const cache = new Map();
const MAX_SIZE = 1000;
function addToCache(key, value) {
    if (cache.size >= MAX_SIZE) {
        const firstKey = cache.keys().next().value;
        cache.delete(firstKey);
    }
    cache.set(key, value);
}
```

---

### Issue 7: Can't Access Container by IP
**Problem**: Cannot reach container IP from host

**Symptoms:**
```bash
docker inspect container-name | grep IPAddress
# IP: 172.17.0.2

curl http://172.17.0.2
# Timeout or connection refused
```

**Solutions:**

**Solution 1: Use Port Mapping**
```bash
# Map container port to host
docker run -p 8080:80 nginx

# Access via localhost
curl http://localhost:8080
```

**Solution 2: Check Network Mode**
```bash
# Bridge network (default, isolated)
docker run --network bridge nginx

# Host network (shares host network)
docker run --network host nginx
```

**Solution 3: Docker Desktop Limitation**
```bash
# macOS/Windows: Can't access bridge network directly
# Solution: Always use port mapping

docker run -p 8080:80 nginx
curl http://localhost:8080
```

---

## Image Issues

### Issue 8: "No such image" or Pull Error
**Problem**: Can't pull or find image

**Symptoms:**
```
Unable to find image 'myimage:latest' locally
Error response from daemon: pull access denied for myimage
```

**Solutions:**

**Solution 1: Check Image Name**
```bash
# Correct format: [registry/]repository[:tag]
docker pull nginx:latest              # Official
docker pull myuser/myimage:1.0       # User repository
docker pull gcr.io/project/image:tag # GCR

# Search for image
docker search nginx
```

**Solution 2: Login to Registry**
```bash
# Docker Hub
docker login
docker pull myuser/private-image

# Private registry
docker login registry.example.com
docker pull registry.example.com/image
```

**Solution 3: Check Network/Proxy**
```bash
# Test connectivity
ping registry-1.docker.io

# Configure proxy
# /etc/systemd/system/docker.service.d/http-proxy.conf
[Service]
Environment="HTTP_PROXY=http://proxy:8080"
Environment="HTTPS_PROXY=http://proxy:8080"

sudo systemctl daemon-reload
sudo systemctl restart docker
```

---

### Issue 9: "Insufficient space" During Build
**Problem**: Out of disk space

**Symptoms:**
```
no space left on device
ERROR: failed to solve: failed to copy files
```

**Solutions:**

**Solution 1: Clean Up**
```bash
# Remove stopped containers
docker container prune

# Remove unused images
docker image prune -a

# Remove unused volumes
docker volume prune

# Remove everything
docker system prune -a --volumes

# Check disk usage
docker system df
```

**Solution 2: Increase Docker Storage**
```bash
# Docker Desktop: Settings → Resources → Disk image size

# Linux: Change data-root
# /etc/docker/daemon.json
{
  "data-root": "/mnt/docker-data"
}

sudo systemctl restart docker
```

**Solution 3: Optimize Build**
```dockerfile
# Use .dockerignore
node_modules
.git
*.log

# Multi-stage builds
FROM node:18 AS build
# ...build steps
FROM node:18-alpine
COPY --from=build /app/dist /app/

# Remove cache
RUN apt-get update && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*
```

---

### Issue 10: Image Build is Very Slow
**Problem**: Docker build takes too long

**Solutions:**

**Solution 1: Use BuildKit**
```bash
# Enable BuildKit
export DOCKER_BUILDKIT=1
docker build -t myapp .

# Or in daemon.json
{
  "features": {
    "buildkit": true
  }
}
```

**Solution 2: Optimize Layer Caching**
```dockerfile
# Bad: Invalidates cache on any code change
COPY . .
RUN npm install

# Good: Cache dependencies separately
COPY package*.json ./
RUN npm install
COPY . .
```

**Solution 3: Use Cache Mounts**
```dockerfile
# Cache package manager downloads
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install -r requirements.txt

RUN --mount=type=cache,target=/root/.npm \
    npm install
```

**Solution 4: Parallel Builds**
```bash
# Build multiple images in parallel
docker build -t app1 ./app1 &
docker build -t app2 ./app2 &
wait
```

---

## Network Problems

### Issue 11: Containers Can't Communicate
**Problem**: Inter-container communication fails

**Symptoms:**
```bash
docker exec app1 ping app2
# ping: app2: Name or service not known
```

**Solutions:**

**Solution 1: Use Same Network**
```bash
# Create network
docker network create mynetwork

# Run containers on network
docker run -d --name app1 --network mynetwork image1
docker run -d --name app2 --network mynetwork image2

# Test
docker exec app1 ping app2  # Works!
```

**Solution 2: Use Docker Compose**
```yaml
version: '3.8'
services:
  app1:
    image: image1
    networks:
      - mynetwork
  app2:
    image: image2
    networks:
      - mynetwork

networks:
  mynetwork:
```

**Solution 3: Legacy Link (Not Recommended)**
```bash
# Old method (deprecated)
docker run -d --name app2 image2
docker run -d --name app1 --link app2 image1
```

---

### Issue 12: DNS Resolution Fails
**Problem**: Container can't resolve hostnames

**Symptoms:**
```bash
docker exec container ping google.com
# ping: google.com: Temporary failure in name resolution
```

**Solutions:**

**Solution 1: Configure DNS**
```bash
# Run with custom DNS
docker run --dns 8.8.8.8 --dns 8.8.4.4 myapp

# Daemon-wide DNS
# /etc/docker/daemon.json
{
  "dns": ["8.8.8.8", "8.8.4.4"]
}

sudo systemctl restart docker
```

**Solution 2: Check Docker Network**
```bash
# Inspect network DNS
docker network inspect bridge

# Create network with custom DNS
docker network create \
  --dns 8.8.8.8 \
  mynetwork
```

---

### Issue 13: "Cannot connect to host.docker.internal"
**Problem**: Can't reach host from container

**Solutions:**

**macOS/Windows:**
```bash
# Use special hostname
docker run -e API_URL=http://host.docker.internal:8000 myapp
```

**Linux:**
```bash
# Add extra host
docker run --add-host=host.docker.internal:host-gateway myapp

# Or use host IP
HOST_IP=$(ip route | grep docker0 | awk '{print $9}')
docker run -e API_URL=http://$HOST_IP:8000 myapp

# Or use host network mode
docker run --network host myapp
```

---

## Volume & Storage Issues

### Issue 14: Permission Denied on Mounted Volume
**Problem**: Can't access mounted directory

**Symptoms:**
```bash
docker run -v $(pwd):/app myapp
# Error: Permission denied
```

**Solutions:**

**Solution 1: Fix Ownership**
```bash
# Change ownership on host
sudo chown -R $USER:$USER $(pwd)

# Or in Dockerfile
USER node
# Files will be created as node user
```

**Solution 2: User Namespace Remapping**
```bash
# /etc/docker/daemon.json
{
  "userns-remap": "default"
}

sudo systemctl restart docker
```

**Solution 3: SELinux Context (Fedora/RHEL)**
```bash
# Add :z or :Z flag
docker run -v $(pwd):/app:z myapp

# z = shared between containers
# Z = private to container
```

---

### Issue 15: Volume Data Not Persisting
**Problem**: Data lost after container restart

**Solutions:**

**Solution 1: Use Named Volumes**
```bash
# Bad: Anonymous volume (random name)
docker run -v /data myapp

# Good: Named volume
docker run -v mydata:/data myapp

# Volume persists
docker volume ls
```

**Solution 2: Check Volume Mount**
```bash
# Inspect container
docker inspect container-name | grep -A 10 Mounts

# Verify volume location
docker volume inspect mydata
```

**Solution 3: Backup Volume**
```bash
# Backup to tar
docker run --rm \
  -v mydata:/data \
  -v $(pwd):/backup \
  ubuntu \
  tar czf /backup/backup.tar.gz /data
```

---

## Performance Problems

### Issue 16: Slow Performance on macOS/Windows
**Problem**: Container performance is slow

**Solutions:**

**Solution 1: Avoid Bind Mounts for Code**
```yaml
# Bad: Slow on macOS/Windows
volumes:
  - ./src:/app/src

# Better: Use volumes + rsync
volumes:
  - node_modules:/app/node_modules
```

**Solution 2: Use Delegated Mode**
```yaml
# Faster on macOS
volumes:
  - ./:/app:delegated
```

**Solution 3: Allocate More Resources**
```
# Docker Desktop
Settings → Resources
- CPUs: 4+
- Memory: 4GB+
- Disk: 60GB+
```

---

### Issue 17: High CPU Usage
**Problem**: Container consuming too much CPU

**Solutions:**

**Solution 1: Monitor & Identify**
```bash
# Real-time stats
docker stats

# Container processes
docker top container-name
```

**Solution 2: Limit CPU**
```bash
# Limit to 1 CPU
docker run --cpus=1.0 myapp

# CPU shares (relative)
docker run --cpu-shares=512 myapp

# Update running container
docker update --cpus 0.5 container-name
```

**Solution 3: Profile Application**
```bash
# Node.js profiling
docker exec container-name node --prof app.js

# Python profiling
docker exec container-name python -m cProfile app.py
```

---

## Docker Compose Issues

### Issue 18: "Service 'X' failed to build"
**Problem**: Docker Compose build fails

**Symptoms:**
```
ERROR: Service 'app' failed to build
```

**Solutions:**

**Solution 1: Build Services Separately**
```bash
# Build with detailed output
docker compose build --no-cache --progress=plain

# Build specific service
docker compose build app

# Use docker build directly
docker build -f Dockerfile -t myapp .
```

**Solution 2: Check Build Context**
```yaml
services:
  app:
    build:
      context: ./app      # Path to Dockerfile directory
      dockerfile: Dockerfile
```

**Solution 3: Clean Build Cache**
```bash
# Remove build cache
docker builder prune -a

# Force rebuild
docker compose build --no-cache
```

---

### Issue 19: "Error: No such service"
**Problem**: Compose can't find service

**Symptoms:**
```
ERROR: No such service: app
```

**Solutions:**

**Solution 1: Check Service Name**
```bash
# List services
docker compose config --services

# Check compose file syntax
docker compose config
```

**Solution 2: Correct Compose File**
```yaml
# Ensure proper indentation
version: '3.8'
services:
  app:    # Service name
    image: myapp
```

**Solution 3: Specify Compose File**
```bash
# Use specific file
docker compose -f docker-compose.prod.yml up

# Multiple files
docker compose -f docker-compose.yml -f docker-compose.override.yml up
```

---

### Issue 20: "Connection refused" Between Services
**Problem**: Services can't connect to each other

**Solutions:**

**Solution 1: Use Service Names**
```yaml
# Service names are DNS names
services:
  web:
    environment:
      - API_URL=http://api:3000  # Use service name
  api:
    image: myapi
```

**Solution 2: Wait for Service**
```dockerfile
# Install wait-for-it
ADD https://github.com/vishnubob/wait-for-it/raw/master/wait-for-it.sh /wait-for-it.sh
RUN chmod +x /wait-for-it.sh

# Wait for database
CMD ["/wait-for-it.sh", "db:5432", "--", "npm", "start"]
```

**Solution 3: Use depends_on with healthcheck**
```yaml
services:
  app:
    depends_on:
      db:
        condition: service_healthy
  db:
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "postgres"]
      interval: 5s
```

---

## Security & Permissions

### Issue 21: "Operation not permitted" as Root
**Problem**: Can't perform operation even as root

**Solutions:**

**Solution 1: Add Capabilities**
```bash
# Add specific capability
docker run --cap-add=NET_ADMIN myapp

# Add all capabilities (not recommended)
docker run --cap-add=ALL myapp

# Drop capabilities
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE myapp
```

**Solution 2: Privileged Mode**
```bash
# Run privileged (use with caution!)
docker run --privileged myapp
```

---

### Issue 22: Image Scan Finds Vulnerabilities
**Problem**: Security vulnerabilities in image

**Solutions:**

**Solution 1: Update Base Image**
```dockerfile
# Use latest patch version
FROM node:18.17.0-alpine3.18

# Update packages
RUN apk upgrade --no-cache
```

**Solution 2: Use Minimal Images**
```dockerfile
# Smaller = fewer vulnerabilities
FROM alpine:3.18
FROM scratch
FROM gcr.io/distroless/nodejs
```

**Solution 3: Scan & Fix**
```bash
# Scan image
docker scout cves myimage

# Fix specific CVEs
RUN apk upgrade --no-cache \
    && apk add --no-cache package-name=version
```

---

## Quick Troubleshooting Commands

```bash
# Container logs
docker logs --tail 100 container-name
docker logs -f container-name

# Container inspect
docker inspect container-name

# Container processes
docker top container-name

# Container stats
docker stats container-name

# Network inspection
docker network inspect bridge

# Volume inspection
docker volume inspect myvolume

# System info
docker info
docker version

# Daemon logs (Linux)
sudo journalctl -u docker -f

# Check Docker health
docker run --rm hello-world

# Clean up
docker system prune -a --volumes
```

---

## Common Error Messages

| Error | Cause | Quick Fix |
|-------|-------|-----------|
| "Cannot connect to daemon" | Daemon not running | `systemctl start docker` |
| "Permission denied" | User not in docker group | `usermod -aG docker $USER` |
| "Port already allocated" | Port conflict | Use different port or stop container |
| "No space left" | Disk full | `docker system prune -a` |
| "OOMKilled" | Out of memory | Increase memory limit |
| "Name already in use" | Container name conflict | Remove old container or use different name |
| "Network not found" | Network doesn't exist | Create network first |
| "Volume not found" | Volume doesn't exist | Create volume first |

---

## Getting More Help

**Logs:**
```bash
# Container logs
docker logs container-name

# Daemon logs (Linux)
sudo journalctl -u docker

# macOS/Windows
# Docker Desktop → Troubleshoot → View Logs
```

**Community Resources:**
- Docker Forums: https://forums.docker.com/
- Stack Overflow: [docker] tag
- Docker Slack: https://dockercommunity.slack.com/
- Official Docs: https://docs.docker.com/

---

**Tip**: When troubleshooting, always check logs first with `docker logs -f container-name`!


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

