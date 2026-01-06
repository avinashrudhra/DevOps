# Docker Interview Questions
## From Basic to Advanced - For 7+ Years Experience

Complete interview preparation covering fundamentals to expert-level Docker knowledge.

---

## Table of Contents
1. [Basic Questions](#basic-questions)
2. [Architecture & Internals](#architecture--internals)
3. [Images & Dockerfiles](#images--dockerfiles)
4. [Networking](#networking)
5. [Storage & Volumes](#storage--volumes)
6. [Security](#security)
7. [Production & Best Practices](#production--best-practices)
8. [Scenario-Based Questions](#scenario-based-questions)

---

## Basic Questions

### Q1: What is Docker and why use it?
**Expected Answer:**

**Docker** is a platform for developing, shipping, and running applications in containers.

**Key Benefits:**
- **Portability**: "Build once, run anywhere"
- **Consistency**: Same environment dev to prod
- **Isolation**: Process-level isolation
- **Efficiency**: Lightweight compared to VMs
- **Speed**: Fast startup (seconds vs minutes)
- **Scalability**: Easy to replicate
- **DevOps**: Enable CI/CD practices

**Use Cases:**
- Microservices architecture
- Continuous integration/deployment
- Development environments
- Application packaging
- Multi-tenancy
- Simplify configuration management

**Containers vs Virtual Machines:**

**VMs:**
- Full OS per instance
- Heavy (GBs)
- Minutes to start
- Hardware-level isolation

**Containers:**
- Share host OS kernel
- Lightweight (MBs)
- Seconds to start
- Process-level isolation

**Follow-up**: What's the difference between Docker and Kubernetes?

**Answer**: Docker is a containerization platform for building and running containers. Kubernetes is a container orchestration platform for managing multiple containers across multiple hosts. Docker focuses on single-host container management; Kubernetes manages containers at scale across clusters.

---

### Q2: Explain Docker architecture
**Expected Answer:**

**Docker Architecture:**

```
┌─────────────────────────────────────────┐
│           Docker Client (CLI)            │
│         docker build, run, push          │
└──────────────────┬──────────────────────┘
                   │ REST API
┌──────────────────▼──────────────────────┐
│          Docker Daemon (dockerd)         │
│    - Manages images, containers         │
│    - Handles builds, networks, volumes  │
└──────────────────┬──────────────────────┘
                   │
      ┌────────────┼────────────┐
      │            │            │
┌─────▼────┐  ┌───▼────┐  ┌───▼────┐
│ Images   │  │Containers│  │Networks│
└──────────┘  └─────────┘  └────────┘
```

**Components:**

**1. Docker Daemon (dockerd):**
- Server process
- Manages Docker objects
- Listens to Docker API requests
- Communicates with other daemons

**2. Docker Client (docker):**
- Command-line interface
- Sends commands to daemon
- Uses Docker API
- Can connect to remote daemons

**3. Docker Registry:**
- Stores Docker images
- Docker Hub (public)
- Private registries (Nexus, Harbor, ECR)

**4. Docker Objects:**

**Images:**
- Read-only templates
- Contains application + dependencies
- Built from Dockerfile
- Composed of layers

**Containers:**
- Runnable instances of images
- Isolated from other containers
- Can be stopped/started/moved/deleted

**Networks:**
- Connect containers
- Isolate container communication
- Multiple drivers (bridge, host, overlay)

**Volumes:**
- Persistent data storage
- Managed by Docker
- Shared between containers

**Underlying Technology:**
- **Namespaces**: Isolation (PID, NET, IPC, MNT, UTS)
- **Control Groups (cgroups)**: Resource limits
- **Union File Systems**: Layered file system
- **Container Runtime**: containerd, runc

**Follow-up**: What happens when you run `docker run nginx`?

**Answer**:
1. Client sends command to daemon
2. Daemon checks for nginx image locally
3. If not found, pulls from Docker Hub
4. Creates new container from image
5. Allocates read-write filesystem layer
6. Creates network interface
7. Starts container (runs nginx process)
8. Returns container ID to client

---

### Q3: What is the difference between CMD and ENTRYPOINT?
**Expected Answer:**

**CMD** - Default command, easily overridden  
**ENTRYPOINT** - Main command, not easily overridden

**CMD:**
```dockerfile
FROM ubuntu
CMD ["echo", "Hello World"]
```

```bash
docker run myimage               # Output: Hello World
docker run myimage echo "Bye"    # Output: Bye (CMD overridden)
```

**ENTRYPOINT:**
```dockerfile
FROM ubuntu
ENTRYPOINT ["echo"]
```

```bash
docker run myimage               # Output: (empty)
docker run myimage "Hello"       # Output: Hello
```

**Combined (Best Practice):**
```dockerfile
FROM ubuntu
ENTRYPOINT ["echo"]
CMD ["Hello World"]
```

```bash
docker run myimage               # Output: Hello World
docker run myimage "Custom"      # Output: Custom
```

**Exec Form vs Shell Form:**

**Exec form (preferred):**
```dockerfile
CMD ["executable", "param1", "param2"]
```
- Doesn't invoke shell
- Process runs as PID 1
- Handles signals properly

**Shell form:**
```dockerfile
CMD executable param1 param2
```
- Invokes `/bin/sh -c`
- Process not PID 1
- Signals not handled properly

**Real-world Example:**
```dockerfile
# Java application
FROM openjdk:11-jre-slim
WORKDIR /app
COPY app.jar .
ENTRYPOINT ["java", "-jar"]
CMD ["app.jar"]

# Can override JAR file:
# docker run myapp another-app.jar
```

**Follow-up**: How do you override ENTRYPOINT?

**Answer**: Use `--entrypoint` flag:
```bash
docker run --entrypoint /bin/bash myimage
```

---

## Architecture & Internals

### Q4: Explain Docker image layers
**Expected Answer:**

**Docker images are built in layers** using a union file system.

**How Layers Work:**

**Dockerfile:**
```dockerfile
FROM ubuntu:20.04          # Layer 1: Base OS (50 MB)
RUN apt-get update        # Layer 2: Package updates (30 MB)
RUN apt-get install curl  # Layer 3: Install curl (10 MB)
COPY app.py /app/         # Layer 4: Add application (1 MB)
CMD ["python", "app.py"]  # Layer 5: Metadata only
```

**Image Structure:**
```
Layer 5: CMD ["python", "app.py"]  (0 MB - metadata)
Layer 4: COPY app.py               (1 MB)
Layer 3: RUN apt-get install       (10 MB)
Layer 2: RUN apt-get update        (30 MB)
Layer 1: FROM ubuntu:20.04         (50 MB)
─────────────────────────────────────────
Total: 91 MB
```

**Benefits:**
- **Reusability**: Shared layers between images
- **Caching**: Faster builds
- **Storage efficiency**: Only changed layers stored
- **Transfer efficiency**: Only new layers pulled

**Layer Caching:**
```dockerfile
# Bad: Cache invalidated on any code change
COPY . /app
RUN npm install

# Good: Cache dependencies separately
COPY package*.json /app/
RUN npm install
COPY . /app
```

**Copy-on-Write:**
- Base layers are read-only
- Container adds writable layer on top
- Changes don't affect image
- Multiple containers share base layers

**Example:**
```
Container 1:  [RW Layer] ← Container-specific changes
              [Image Layer 3] ← Shared
              [Image Layer 2] ← Shared
              [Image Layer 1] ← Shared

Container 2:  [RW Layer] ← Different changes
              [Image Layer 3] ← Shared (same)
              [Image Layer 2] ← Shared (same)
              [Image Layer 1] ← Shared (same)
```

**View Image Layers:**
```bash
docker history myimage:latest
docker inspect myimage:latest | jq '.[].RootFS.Layers'
```

**Optimization Tips:**
- Minimize number of layers
- Order instructions by change frequency
- Combine RUN commands
- Use .dockerignore
- Multi-stage builds

**Follow-up**: What is the maximum number of layers?

**Answer**: Docker supports up to 127 layers (128 including base). However, best practice is to minimize layers for better performance and manageability.

---

### Q5: What are Docker namespaces and cgroups?
**Expected Answer:**

**Namespaces** and **cgroups** are Linux kernel features that enable containerization.

**Namespaces (Isolation):**

Provide isolated view of system resources:

**1. PID Namespace:**
- Process isolation
- Container has own PID 1
- Can't see host processes

```bash
# Host
ps aux  # Shows all processes

# In container
docker run -it ubuntu bash
ps aux  # Only sees container processes
```

**2. Network Namespace:**
- Network stack isolation
- Own IP address, routes, firewall rules

**3. Mount Namespace:**
- Filesystem isolation
- Own root filesystem
- Isolated mount points

**4. UTS Namespace:**
- Hostname and domain name isolation
- Container can have own hostname

**5. IPC Namespace:**
- Inter-process communication isolation
- Separate message queues, semaphores

**6. User Namespace:**
- User and group ID isolation
- Root in container ≠ root on host

**Example:**
```bash
# Container sees different hostname
docker run -it --hostname mycontainer ubuntu bash
hostname  # Output: mycontainer

# Host
hostname  # Output: host-machine
```

**Control Groups (cgroups) - Resource Limits:**

Limit and monitor resource usage:

**1. Memory:**
```bash
docker run -m 512m myapp        # Limit memory
docker run --memory-swap 1g myapp  # Memory + swap
```

**2. CPU:**
```bash
docker run --cpus=1.5 myapp     # Limit CPU
docker run --cpu-shares=512 myapp  # Relative weight
```

**3. Block I/O:**
```bash
docker run --device-read-bps /dev/sda:1mb myapp
```

**4. PIDs:**
```bash
docker run --pids-limit=100 myapp
```

**View cgroup Limits:**
```bash
# Memory limit
cat /sys/fs/cgroup/memory/docker/container-id/memory.limit_in_bytes

# CPU shares
cat /sys/fs/cgroup/cpu/docker/container-id/cpu.shares
```

**How Docker Uses Them:**
```
Container Process
    ↓
Namespaces (Isolation)
    - Own PID space
    - Own network stack
    - Own filesystem
    ↓
cgroups (Resource Limits)
    - Limited memory
    - Limited CPU
    - Limited I/O
    ↓
Isolated, Resource-Controlled Container
```

**Follow-up**: Can containers escape namespace isolation?

**Answer**: Yes, if running with `--privileged` or with excessive capabilities. This is why security best practices recommend:
- Don't run containers as root
- Drop unnecessary capabilities
- Use user namespaces
- Never use `--privileged` in production unless absolutely necessary

---

### Q6: What is the difference between ADD and COPY?
**Expected Answer:**

Both copy files from host to image, but with different capabilities:

**COPY (Recommended):**
- Simple file/directory copy
- Transparent behavior
- Preferred for most use cases

```dockerfile
COPY package.json /app/
COPY src/ /app/src/
COPY . /app
```

**ADD:**
- All COPY features plus:
  - Auto-extract tar archives
  - Download files from URLs

```dockerfile
# Auto-extract
ADD archive.tar.gz /app/
# Extracts contents to /app/

# Download from URL
ADD https://example.com/file.tar.gz /tmp/
```

**When to Use Each:**

**Use COPY:**
- 99% of the time
- Explicit and predictable
- Better for caching

**Use ADD only when:**
- Need to extract tar archives
- Need to download from URL (though wget/curl in RUN is better)

**Best Practice:**
```dockerfile
# Good: Explicit
COPY package.json /app/

# Avoid: Implicit extraction
ADD archive.tar.gz /app/  # Not obvious what happens

# Better: Explicit
COPY archive.tar.gz /app/
RUN tar -xzf /app/archive.tar.gz -C /app/ && rm /app/archive.tar.gz
```

**Caching Behavior:**
```dockerfile
# COPY invalidates cache if file changes
COPY package.json /app/
RUN npm install  # Cached unless package.json changes

# ADD with URL always downloads (no cache)
ADD https://example.com/file.tar.gz /tmp/
```

**Follow-up**: How does Docker determine if a file has changed for cache purposes?

**Answer**: Docker uses file checksums (content hash). If the checksum changes, the layer cache is invalidated. This is why timestamps don't affect caching, only actual content changes do.

---

## Images & Dockerfiles

### Q7: Explain multi-stage builds and their benefits
**Expected Answer:**

**Multi-stage builds** allow multiple FROM statements in a single Dockerfile, enabling separate build and runtime environments.

**Problem Without Multi-Stage:**
```dockerfile
FROM golang:1.20
WORKDIR /app
COPY . .
RUN go build -o app

# Final image includes:
# - Go compiler (not needed)
# - Source code (not needed)
# - Build tools (not needed)
# Size: ~800MB
```

**Solution With Multi-Stage:**
```dockerfile
# Stage 1: Build
FROM golang:1.20 AS builder
WORKDIR /app
COPY . .
RUN go build -o app

# Stage 2: Runtime
FROM alpine:3.18
WORKDIR /app
COPY --from=builder /app/app .
CMD ["./app"]

# Final image includes:
# - Only the binary
# Size: ~10MB
```

**Benefits:**
1. **Smaller Images**: Only runtime dependencies
2. **Security**: Fewer attack surfaces
3. **Faster Deployments**: Smaller images transfer faster
4. **Cleaner Separation**: Build vs runtime concerns
5. **No Build Artifacts**: Clean production images

**Real-World Examples:**

**Java Application:**
```dockerfile
# Build stage
FROM maven:3.8-jdk-11 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package -DskipTests

# Runtime stage
FROM openjdk:11-jre-slim
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]

# Build: ~700MB → Runtime: ~200MB
```

**Node.js Application:**
```dockerfile
# Build stage
FROM node:18 AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Runtime stage
FROM node:18-alpine
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
COPY package*.json ./
EXPOSE 3000
CMD ["node", "dist/server.js"]

# Build: ~1GB → Runtime: ~150MB
```

**Advanced: Multiple Stages for Testing:**
```dockerfile
# Base stage
FROM node:18 AS base
WORKDIR /app
COPY package*.json ./
RUN npm ci

# Test stage
FROM base AS test
COPY . .
RUN npm test

# Build stage
FROM base AS build
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine AS production
WORKDIR /app
COPY --from=build /app/dist ./dist
CMD ["node", "dist/server.js"]
```

**Build Specific Stage:**
```bash
# Build and run tests
docker build --target test -t myapp:test .

# Build production
docker build --target production -t myapp:prod .
```

**Size Comparison:**
```
Single-stage:    800 MB
Multi-stage:     10 MB
Savings:         790 MB (98.75% reduction)
```

**Follow-up**: Can you copy from external images?

**Answer**: Yes, using `COPY --from=external-image:tag`:
```dockerfile
FROM scratch
COPY --from=nginx:latest /usr/share/nginx/html /html
```

---

### Q8: How do you optimize Docker image size?
**Expected Answer:**

**Optimization Strategies:**

**1. Use Minimal Base Images:**
```dockerfile
# Large: 1GB
FROM ubuntu:22.04

# Medium: 200MB
FROM node:18

# Small: 150MB
FROM node:18-alpine

# Tiny: 5MB
FROM alpine:3.18

# Smallest: 0MB (static binaries only)
FROM scratch
```

**2. Multi-Stage Builds:**
```dockerfile
FROM golang:1.20 AS build
# Build application

FROM alpine:3.18
COPY --from=build /app/binary .
```

**3. Combine RUN Commands:**
```dockerfile
# Bad: 3 layers, larger image
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y vim

# Good: 1 layer, smaller image
RUN apt-get update && \
    apt-get install -y curl vim && \
    rm -rf /var/lib/apt/lists/*
```

**4. Remove Package Manager Cache:**
```dockerfile
# Alpine
RUN apk add --no-cache curl

# Ubuntu/Debian
RUN apt-get update && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*

# CentOS/RHEL
RUN yum install -y curl && yum clean all
```

**5. Use .dockerignore:**
```
# .dockerignore
node_modules
.git
*.log
.DS_Store
README.md
test
coverage
.env
```

**6. Order Layers by Change Frequency:**
```dockerfile
# Bad: Code changes invalidate dependency install
COPY . .
RUN npm install

# Good: Dependencies cached separately
COPY package*.json ./
RUN npm install
COPY . .
```

**7. Install Only Production Dependencies:**
```dockerfile
# Node.js
RUN npm ci --only=production

# Python
RUN pip install --no-cache-dir -r requirements.txt
```

**8. Use Specific Versions:**
```dockerfile
# Bad: Layers not cached
RUN apt-get install curl

# Good: Specific version, cacheable
RUN apt-get install curl=7.68.0-1ubuntu2.13
```

**9. Remove Temporary Files:**
```dockerfile
RUN wget https://example.com/package.tar.gz \
    && tar -xzf package.tar.gz \
    && mv package /opt/ \
    && rm package.tar.gz
```

**10. Leverage BuildKit Cache Mounts:**
```dockerfile
# Cache pip downloads
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install -r requirements.txt

# Cache npm modules
RUN --mount=type=cache,target=/root/.npm \
    npm install
```

**Real Example - Optimization Journey:**

**Version 1: Large Image**
```dockerfile
FROM ubuntu:22.04
RUN apt-get update
RUN apt-get install -y python3 python3-pip
COPY . /app
WORKDIR /app
RUN pip3 install -r requirements.txt
CMD ["python3", "app.py"]
# Size: 800MB
```

**Version 2: Alpine Base**
```dockerfile
FROM python:3.11-alpine
COPY . /app
WORKDIR /app
RUN pip install --no-cache-dir -r requirements.txt
CMD ["python", "app.py"]
# Size: 150MB
```

**Version 3: Multi-Stage + Optimizations**
```dockerfile
# Build stage
FROM python:3.11-alpine AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir --user -r requirements.txt

# Runtime stage
FROM python:3.11-alpine
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY app.py .
ENV PATH=/root/.local/bin:$PATH
CMD ["python", "app.py"]
# Size: 50MB
```

**Verification:**
```bash
# Check image size
docker images myapp

# View layers
docker history myapp:latest

# Analyze with dive
dive myapp:latest
```

**Follow-up**: What is Docker image squashing?

**Answer**: Squashing combines all layers into a single layer, which can reduce size by eliminating intermediate artifacts. However, it breaks layer caching and sharing. Generally not recommended except for final distribution.

```bash
# Experimental feature
docker build --squash -t myapp .
```

---

## Networking

### Q9: Explain Docker networking modes
**Expected Answer:**

Docker supports multiple network drivers:

**1. Bridge (Default):**

**Characteristics:**
- Private internal network on host
- Containers get IP from subnet
- NAT to access external network
- Containers on same bridge can communicate

**Usage:**
```bash
# Default bridge
docker run -d nginx

# Custom bridge
docker network create my-bridge
docker run -d --network my-bridge nginx

# Inspect
docker network inspect bridge
```

**Benefits:**
- Isolation from host network
- Container-to-container communication
- DNS resolution by container name

**2. Host:**

**Characteristics:**
- Use host's network directly
- No network isolation
- Container shares host IP/ports
- Better performance (no NAT)

**Usage:**
```bash
docker run -d --network host nginx
# Listens on host's port 80 directly
```

**Use Cases:**
- High-performance applications
- Need host network features
- Debugging network issues

**Limitations:**
- Port conflicts
- No isolation
- Can't run multiple containers on same port

**3. None:**

**Characteristics:**
- No networking
- Complete isolation
- Only loopback interface

**Usage:**
```bash
docker run -d --network none myapp
# No network access
```

**Use Cases:**
- Batch processing
- Security-sensitive workloads
- Testing

**4. Overlay:**

**Characteristics:**
- Multi-host networking
- For Docker Swarm
- Containers on different hosts communicate
- Encrypted by default

**Usage:**
```bash
docker network create --driver overlay my-overlay
docker service create --network my-overlay myapp
```

**Use Cases:**
- Docker Swarm
- Multi-host container communication
- Microservices across nodes

**5. Macvlan:**

**Characteristics:**
- Assign MAC address to container
- Container appears as physical device
- Direct host network access

**Usage:**
```bash
docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 \
  my-macvlan

docker run --network my-macvlan myapp
```

**Use Cases:**
- Legacy applications expecting physical network
- Network monitoring
- VLANs

**Network Comparison:**

| Feature | Bridge | Host | None | Overlay | Macvlan |
|---------|--------|------|------|---------|---------|
| Isolation | Yes | No | Complete | Yes | Yes |
| Performance | Good | Best | N/A | Good | Good |
| Port mapping | Needed | No | N/A | No | No |
| Multi-host | No | N/A | N/A | Yes | Yes |
| Use case | Most apps | Performance | Batch | Swarm | Legacy |

**Custom Bridge vs Default Bridge:**

```bash
# Default bridge - legacy
docker run -d --name web nginx

# Custom bridge - recommended
docker network create mynet
docker run -d --name web --network mynet nginx

# Advantages:
# - Automatic DNS resolution
# - Better isolation
# - User-defined subnet
# - Connect/disconnect on the fly
```

**Container Communication:**
```bash
# Create network
docker network create app-net

# Run containers
docker run -d --name db --network app-net postgres
docker run -d --name api --network app-net myapi

# API can reach DB by name
docker exec api ping db  # Works!
docker exec api curl http://db:5432  # Works!
```

**Follow-up**: How does Docker DNS work?

**Answer**: Docker has an embedded DNS server (127.0.0.11) that provides DNS resolution for container names on user-defined networks. When a container queries a name, Docker DNS resolves it to the container's IP address. This only works on custom networks, not the default bridge.

---

### Q10: How do you expose and publish container ports?
**Expected Answer:**

**EXPOSE vs -p:**

**EXPOSE (Dockerfile):**
- Documents which ports the container listens on
- Metadata only, doesn't actually publish
- Used for communication between containers

```dockerfile
FROM nginx
EXPOSE 80 443
```

**-p / --publish (Runtime):**
- Actually maps container port to host port
- Required to access from outside Docker network

```bash
# Format: -p [host-ip:]host-port:container-port[/protocol]

# Map host 8080 to container 80
docker run -p 8080:80 nginx

# Same port on host and container
docker run -p 80:80 nginx

# Random port on host
docker run -P nginx  # Maps to random high port

# Specific interface
docker run -p 127.0.0.1:8080:80 nginx

# Multiple ports
docker run -p 8080:80 -p 8443:443 nginx

# UDP port
docker run -p 53:53/udp dns-server
```

**Port Binding Examples:**

**1. Web Server:**
```bash
docker run -d -p 80:80 --name web nginx
curl http://localhost  # Accessible from host
```

**2. Development with Different Port:**
```bash
docker run -d -p 3001:3000 --name dev-app myapp
# App listens on 3000, accessible on host:3001
```

**3. Multiple Instances:**
```bash
docker run -d -p 8080:80 --name nginx1 nginx
docker run -d -p 8081:80 --name nginx2 nginx
docker run -d -p 8082:80 --name nginx3 nginx
```

**4. Bind to Specific Interface:**
```bash
# Only localhost
docker run -p 127.0.0.1:8080:80 nginx

# Specific IP
docker run -p 192.168.1.100:8080:80 nginx
```

**Check Port Mappings:**
```bash
# List port mappings
docker port container-name

# Inspect network settings
docker inspect container-name | grep -A 10 Ports
```

**Docker Compose:**
```yaml
version: '3.8'
services:
  web:
    image: nginx
    ports:
      - "8080:80"           # Host:Container
      - "8443:443"
      - "127.0.0.1:9000:9000"  # Specific interface
    expose:
      - "3000"              # Inter-container only
```

**Port Ranges:**
```bash
# Map port range
docker run -p 8080-8090:8080-8090 myapp

# Docker Compose
ports:
  - "8080-8090:8080-8090"
```

**Common Issues:**

**Port Already in Use:**
```bash
# Error: port is already allocated

# Find what's using port
sudo lsof -i :8080

# Solution: Use different port
docker run -p 8081:80 nginx
```

**Can't Access Container:**
```bash
# Container running but can't access

# Check:
1. Port is published: docker port container-name
2. Container is listening: docker exec container-name netstat -tuln
3. Firewall: sudo ufw status
4. Application is bound to 0.0.0.0, not 127.0.0.1
```

**Follow-up**: What's the difference between EXPOSE and publishing ports?

**Answer**:
- **EXPOSE**: Documentation, doesn't actually open ports. Tells other developers/containers what ports the app uses.
- **-p/--publish**: Actually binds container port to host port, making it accessible from outside.

Think of EXPOSE as a comment, -p as the action.

---

## Storage & Volumes

### Q11: What are the different types of Docker storage?
**Expected Answer:**

Docker provides three types of mounts for persisting data:

**1. Volumes (Recommended):**

**Characteristics:**
- Managed by Docker
- Stored in Docker area (`/var/lib/docker/volumes/`)
- Isolated from host filesystem
- Can be shared between containers
- Easy to backup
- Work on all platforms

**Usage:**
```bash
# Create volume
docker volume create mydata

# Use in container
docker run -v mydata:/app/data nginx

# List volumes
docker volume ls

# Inspect
docker volume inspect mydata

# Remove
docker volume rm mydata
```

**Benefits:**
- Docker manages lifecycle
- Platform independent
- Better performance on Docker Desktop
- Can use volume drivers (cloud storage)
- Safer (can't accidentally modify host)

**2. Bind Mounts:**

**Characteristics:**
- Map host directory to container
- Full access to host filesystem
- Path can be anywhere on host
- Performance issues on macOS/Windows

**Usage:**
```bash
# Mount current directory
docker run -v $(pwd):/app nginx

# Mount specific directory
docker run -v /host/path:/container/path nginx

# Read-only mount
docker run -v $(pwd):/app:ro nginx

# Windows
docker run -v C:\Users\data:/app nginx
```

**Use Cases:**
- Development (live code reload)
- Share configuration files
- Access host logs

**Limitations:**
- Tied to host file system structure
- Portability issues
- Potential security risks

**3. tmpfs Mounts:**

**Characteristics:**
- Stored in host memory only
- Never written to disk
- Temporary data only
- Fast performance
- Linux only

**Usage:**
```bash
# Mount tmpfs
docker run --tmpfs /app/cache nginx

# With size limit
docker run --tmpfs /tmp:size=100m nginx
```

**Use Cases:**
- Temporary files
- Secrets (shouldn't persist)
- Cache
- Session data

**Comparison:**

| Feature | Volume | Bind Mount | tmpfs |
|---------|--------|------------|-------|
| Managed by | Docker | Host | Memory |
| Location | Docker area | Anywhere | RAM |
| Portability | High | Low | N/A |
| Performance | Good | Varies | Best |
| Persistence | Yes | Yes | No |
| Sharing | Easy | Medium | No |
| Use case | Production | Development | Temporary |

**Real Examples:**

**Database (Volume):**
```bash
docker run -d \
  --name postgres \
  -v pgdata:/var/lib/postgresql/data \
  postgres:15

# Data persists even if container removed
docker rm postgres
docker run -d --name postgres2 -v pgdata:/var/lib/postgresql/data postgres:15
# Same data!
```

**Development (Bind Mount):**
```bash
# Live code reload
docker run -d \
  -v $(pwd)/src:/app/src \
  -p 3000:3000 \
  node:18 \
  npm run dev

# Edit files on host, changes reflected immediately
```

**Cache (tmpfs):**
```bash
# Temporary cache in memory
docker run -d \
  --tmpfs /app/cache:size=500m \
  myapp
```

**Docker Compose:**
```yaml
version: '3.8'
services:
  app:
    image: myapp
    volumes:
      # Named volume
      - app-data:/app/data
      
      # Bind mount
      - ./src:/app/src:ro
      
      # tmpfs
      - type: tmpfs
        target: /app/cache
        tmpfs:
          size: 100000000

volumes:
  app-data:
```

**Volume Backup/Restore:**
```bash
# Backup
docker run --rm \
  -v mydata:/data \
  -v $(pwd):/backup \
  ubuntu \
  tar czf /backup/backup.tar.gz /data

# Restore
docker run --rm \
  -v mydata:/data \
  -v $(pwd):/backup \
  ubuntu \
  tar xzf /backup/backup.tar.gz -C /
```

**Follow-up**: How do you share volumes between containers?

**Answer**: Multiple containers can mount the same volume:
```bash
# Create volume
docker volume create shared-data

# Container 1 writes
docker run -v shared-data:/data alpine sh -c "echo hello > /data/file.txt"

# Container 2 reads
docker run -v shared-data:/data alpine cat /data/file.txt
# Output: hello
```

This is useful for microservices sharing configuration or data.

---

## Security

### Q12: What are Docker security best practices?
**Expected Answer:**

**1. Use Official/Verified Images:**
```dockerfile
# Good: Official image
FROM node:18-alpine

# Avoid: Unknown source
FROM random-user/node
```

**2. Don't Run as Root:**
```dockerfile
FROM node:18-alpine

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Set ownership
COPY --chown=nodejs:nodejs . /app

# Switch to non-root
USER nodejs

CMD ["node", "server.js"]
```

**3. Use Minimal Base Images:**
```dockerfile
# Large attack surface
FROM ubuntu:22.04

# Minimal attack surface
FROM alpine:3.18
FROM gcr.io/distroless/node
FROM scratch  # For static binaries
```

**4. Scan Images for Vulnerabilities:**
```bash
# Docker Scout
docker scout cves myimage:latest

# Trivy
trivy image myimage:latest

# Snyk
snyk container test myimage:latest
```

**5. Use Specific Image Tags:**
```dockerfile
# Bad: Latest changes, unpredictable
FROM node:latest

# Good: Specific version
FROM node:18.17.0-alpine3.18
```

**6. Don't Embed Secrets:**
```dockerfile
# BAD: Secrets in image
ENV API_KEY=abc123
ENV DB_PASSWORD=secret

# GOOD: Runtime secrets
# Use environment variables at runtime
docker run -e API_KEY=$API_KEY myapp

# Or Docker secrets (Swarm)
docker secret create api_key key.txt
```

**7. Use Read-Only Root Filesystem:**
```bash
docker run --read-only \
  --tmpfs /tmp \
  myapp
```

**8. Drop Capabilities:**
```bash
docker run \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  myapp
```

**9. Limit Resources:**
```bash
docker run \
  --memory="512m" \
  --cpus="1.0" \
  --pids-limit=100 \
  myapp
```

**10. Security Options:**
```bash
docker run \
  --security-opt=no-new-privileges \
  --read-only \
  myapp
```

**11. Network Segmentation:**
```bash
# Separate networks for different tiers
docker network create frontend
docker network create backend

docker run --network frontend web
docker run --network backend db
```

**12. Regular Updates:**
```dockerfile
# Update packages
RUN apk upgrade --no-cache
RUN apt-get update && apt-get upgrade -y
```

**13. Use .dockerignore:**
```
.git
.env
secrets/
*.key
*.pem
node_modules
```

**14. Multi-Stage Builds:**
```dockerfile
# Build stage (with tools)
FROM golang:1.20 AS builder
RUN go build

# Runtime stage (minimal)
FROM alpine:3.18
COPY --from=builder /app/binary .
```

**15. Sign and Verify Images:**
```bash
# Enable content trust
export DOCKER_CONTENT_TRUST=1

# Push signed image
docker push myimage:1.0

# Pull verifies signature
docker pull myimage:1.0
```

**Complete Secure Dockerfile Example:**
```dockerfile
# Use specific version
FROM node:18.17.0-alpine3.18 AS base

# Install security updates
RUN apk upgrade --no-cache

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Build stage
FROM base AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Runtime stage
FROM base AS runtime
WORKDIR /app

# Copy with correct ownership
COPY --from=build --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --chown=nodejs:nodejs . .

# Switch to non-root user
USER nodejs

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD node healthcheck.js || exit 1

# Start application
EXPOSE 3000
CMD ["node", "server.js"]
```

**Runtime Security:**
```bash
docker run -d \
  --name secure-app \
  --read-only \
  --tmpfs /tmp \
  --security-opt=no-new-privileges \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  --memory="512m" \
  --cpus="1.0" \
  --pids-limit=100 \
  -p 3000:3000 \
  secure-app:latest
```

**Security Checklist:**
✅ Use official images  
✅ Don't run as root  
✅ Use minimal base  
✅ Scan for vulnerabilities  
✅ Use specific tags  
✅ No secrets in images  
✅ Read-only filesystem  
✅ Drop capabilities  
✅ Resource limits  
✅ Regular updates  
✅ Multi-stage builds  
✅ Sign images  

**Follow-up**: What is Docker Bench Security?

**Answer**: Docker Bench Security is a script that checks for common best practices around deploying Docker containers in production. It tests:
- Host configuration
- Docker daemon configuration
- Container images
- Container runtime
- Docker security operations

```bash
docker run -it --rm \
  --net host \
  --pid host \
  --cap-add audit_control \
  -v /var/lib:/var/lib \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /etc:/etc \
  docker/docker-bench-security
```

---

## Production & Best Practices

### Q13: How do you handle logging in Docker?
**Expected Answer:**

**Logging Strategies:**

**1. STDOUT/STDERR (Recommended):**

**Why:**
- Docker captures these automatically
- 12-factor app principle
- Easy to redirect

**Implementation:**
```dockerfile
# Application logs to stdout
FROM node:18-alpine
CMD ["node", "server.js"]  # Console.log goes to stdout
```

```python
# Python
import logging
logging.basicConfig(stream=sys.stdout, level=logging.INFO)
```

**View Logs:**
```bash
# View logs
docker logs container-name

# Follow logs
docker logs -f container-name

# Last 100 lines
docker logs --tail 100 container-name

# With timestamps
docker logs -t container-name

# Since specific time
docker logs --since 2024-01-01T00:00:00 container-name
```

**2. Logging Drivers:**

**Configure Driver:**
```bash
# At runtime
docker run --log-driver=json-file myapp

# Daemon-wide (/etc/docker/daemon.json)
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

**Available Drivers:**

**json-file (default):**
```bash
docker run \
  --log-driver=json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  myapp
```

**syslog:**
```bash
docker run \
  --log-driver=syslog \
  --log-opt syslog-address=tcp://192.168.0.100:514 \
  myapp
```

**journald:**
```bash
docker run \
  --log-driver=journald \
  myapp

# View with journalctl
journalctl CONTAINER_NAME=myapp
```

**fluentd:**
```bash
docker run \
  --log-driver=fluentd \
  --log-opt fluentd-address=localhost:24224 \
  --log-opt tag=docker.myapp \
  myapp
```

**gelf (Graylog):**
```bash
docker run \
  --log-driver=gelf \
  --log-opt gelf-address=udp://localhost:12201 \
  myapp
```

**awslogs (CloudWatch):**
```bash
docker run \
  --log-driver=awslogs \
  --log-opt awslogs-region=us-east-1 \
  --log-opt awslogs-group=myapp-logs \
  myapp
```

**3. Centralized Logging:**

**EFK Stack (Elasticsearch, Fluentd, Kibana):**
```yaml
version: '3.8'
services:
  app:
    image: myapp
    logging:
      driver: fluentd
      options:
        fluentd-address: localhost:24224
        tag: docker.myapp
  
  fluentd:
    image: fluent/fluentd
    ports:
      - "24224:24224"
    volumes:
      - ./fluentd/conf:/fluentd/etc
  
  elasticsearch:
    image: elasticsearch:8.10.0
  
  kibana:
    image: kibana:8.10.0
    ports:
      - "5601:5601"
```

**4. Log Rotation:**

**Configure Rotation:**
```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "5",
    "compress": "true"
  }
}
```

**5. Structured Logging:**

**JSON Format:**
```javascript
// Node.js with winston
const winston = require('winston');
const logger = winston.createLogger({
  format: winston.format.json(),
  transports: [
    new winston.transports.Console()
  ]
});

logger.info('User logged in', { userId: 123, ip: '1.2.3.4' });
// Output: {"level":"info","message":"User logged in","userId":123,"ip":"1.2.3.4"}
```

**6. Docker Compose Logging:**

```yaml
version: '3.8'
services:
  web:
    image: nginx
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
        labels: "service=web"
        env: "ENVIRONMENT"
```

**Best Practices:**
✅ Always log to STDOUT/STDERR  
✅ Use structured logging (JSON)  
✅ Configure log rotation  
✅ Use centralized logging in production  
✅ Include context (request ID, user ID)  
✅ Set appropriate log levels  
✅ Monitor log volume  
✅ Secure sensitive data  

**Follow-up**: How do you aggregate logs from multiple containers?

**Answer**: Use centralized logging solutions:
1. **EFK Stack**: Fluentd collects from all containers → Elasticsearch stores → Kibana visualizes
2. **Splunk**: Docker logging driver sends directly to Splunk
3. **CloudWatch**: AWS CloudWatch Logs for AWS deployments
4. **Datadog/New Relic**: SaaS APM solutions

All containers log to STDOUT, and logging driver/agent collects and centralizes them.

---

## Scenario-Based Questions

### Q14: How would you migrate a legacy application to Docker?
**Expected Answer:**

**Migration Strategy:**

**Phase 1: Assessment (Week 1)**
```
1. Analyze application:
   - Runtime dependencies (Java, Python, Node.js)
   - System dependencies (libraries, tools)
   - Configuration files
   - Data storage locations
   - Network requirements
   - External dependencies (databases, APIs)

2. Identify challenges:
   - Stateful components
   - Hard-coded paths
   - Environment-specific config
   - Logging locations
   - Security requirements
```

**Phase 2: Containerization (Week 2-3)**

**Step 1: Create Dockerfile**
```dockerfile
# Java legacy app example
FROM openjdk:11-jre

# Install system dependencies
RUN apt-get update && apt-get install -y \
    curl \
    netcat \
    && rm -rf /var/lib/apt/lists/*

# Create app user
RUN useradd -m -u 1001 appuser

# Set working directory
WORKDIR /app

# Copy application
COPY --chown=appuser:appuser app.jar .
COPY --chown=appuser:appuser config/ ./config/

# Switch to non-root user
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:8080/health || exit 1

# Expose port
EXPOSE 8080

# Run application
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Step 2: Externalize Configuration**
```bash
# Before: Hard-coded config in app
database.url=jdbc:postgresql://localhost:5432/db

# After: Environment variables
database.url=${DATABASE_URL}

# Run with config
docker run \
  -e DATABASE_URL=jdbc:postgresql://db:5432/prod \
  myapp
```

**Step 3: Handle State**
```bash
# Identify stateful components
- Database: Use Docker volume or external DB
- File uploads: Use volume or object storage (S3)
- Sessions: Use Redis or external session store
- Logs: Log to STDOUT, not files

# Example: Database with volume
docker run -v dbdata:/var/lib/postgresql/data postgres
```

**Phase 3: Testing (Week 4)**

**Test Scenarios:**
```bash
# 1. Functional testing
docker run myapp
curl http://localhost:8080/health

# 2. Integration testing
docker-compose -f docker-compose.test.yml up

# 3. Performance testing
# Compare response times: legacy vs containerized

# 4. Data migration testing
# Test database migration scripts in container
```

**Phase 4: Docker Compose (Week 5)**

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL=jdbc:postgresql://db:5432/appdb
      - REDIS_URL=redis://cache:6379
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_started
    volumes:
      - app-logs:/app/logs
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3
  
  db:
    image: postgres:15
    environment:
      - POSTGRES_DB=appdb
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
    volumes:
      - db-data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    secrets:
      - db_password
  
  cache:
    image: redis:7-alpine
    volumes:
      - cache-data:/data

volumes:
  db-data:
  cache-data:
  app-logs:

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

**Phase 5: CI/CD Integration (Week 6)**

**Dockerfile for CI/CD:**
```dockerfile
# Multi-stage for CI/CD
FROM maven:3.8-jdk-11 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package -DskipTests

FROM openjdk:11-jre-slim
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Jenkins Pipeline:**
```groovy
pipeline {
    agent any
    stages {
        stage('Build Image') {
            steps {
                sh 'docker build -t myapp:${BUILD_NUMBER} .'
            }
        }
        stage('Test') {
            steps {
                sh 'docker run --rm myapp:${BUILD_NUMBER} npm test'
            }
        }
        stage('Push') {
            steps {
                sh 'docker push registry.company.com/myapp:${BUILD_NUMBER}'
            }
        }
        stage('Deploy') {
            steps {
                sh 'docker-compose -f docker-compose.prod.yml up -d'
            }
        }
    }
}
```

**Phase 6: Production Deployment (Week 7-8)**

**Considerations:**
```bash
# 1. Resource limits
docker run \
  --memory="2g" \
  --cpus="2" \
  myapp

# 2. Monitoring
# Add Prometheus, Grafana

# 3. Logging
# Configure centralized logging

# 4. Backup strategy
# Automate volume backups

# 5. Disaster recovery
# Document recovery procedures
```

**Common Challenges & Solutions:**

**Challenge 1: Application expects specific file paths**
```
Solution: Use volumes to map paths
docker run -v /legacy/path:/app/data myapp
```

**Challenge 2: Application needs specific hostname**
```
Solution: Set hostname
docker run --hostname legacy-hostname myapp
```

**Challenge 3: Large image size**
```
Solution: Multi-stage builds, minimal base
```

**Challenge 4: Slow startup**
```
Solution: Optimize Dockerfile, pre-warm caches
```

**Rollback Strategy:**
```bash
# Keep legacy system running in parallel
# Gradual traffic shift
# Monitor metrics
# Quick rollback if issues

# Blue-green deployment
docker tag myapp:${BUILD_NUMBER} myapp:blue
# If issues:
docker tag myapp:previous myapp:green
```

**Success Metrics:**
- Deployment time reduction
- Environment consistency
- Resource utilization
- Incident reduction
- Developer productivity

**Follow-up**: How do you handle database migrations in containers?

**Answer**:
1. **Initialization scripts**: PostgreSQL/MySQL support `/docker-entrypoint-initdb.d/`
2. **Migration containers**: Run migration as separate container before app
3. **Tools**: Flyway, Liquibase in init containers
4. **Version control**: Track schema versions

```yaml
services:
  migrate:
    image: flyway/flyway
    command: migrate
    volumes:
      - ./migrations:/flyway/sql
    depends_on:
      - db
  
  app:
    image: myapp
    depends_on:
      - migrate
```

---

## Summary

### Key Topics to Master

**Fundamentals:**
- Docker architecture and components
- Images and containers
- Dockerfile instructions
- Container lifecycle

**Intermediate:**
- Networking modes and configuration
- Volumes and persistent storage
- Docker Compose
- Multi-stage builds

**Advanced:**
- Security best practices
- Production deployment patterns
- Performance optimization
- CI/CD integration
- Troubleshooting

---

## Interview Tips

**Preparation:**
1. Practice building images
2. Understand layer caching
3. Know networking inside out
4. Have real examples ready
5. Understand security implications

**During Interview:**
1. Explain with diagrams
2. Mention production experiences
3. Discuss trade-offs
4. Show problem-solving
5. Ask clarifying questions

**Common Follow-ups:**
- "Have you used this in production?"
- "What challenges did you face?"
- "How did you optimize performance?"
- "What about security?"

---

### Resources
- [Learning Roadmap](docker-learning-roadmap.md)
- [Quick Reference](docker-quick-reference.md)
- [Hands-On Exercises](docker-hands-on-exercises.md)
- [Troubleshooting Guide](docker-troubleshooting-guide.md)

**Good luck with your Docker interview! 🐋**


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

