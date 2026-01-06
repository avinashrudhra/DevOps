# Docker Learning Roadmap
## 12-Week Journey from Beginner to Production Expert

Complete curriculum for mastering Docker containerization technology.

---

## 🎯 Learning Path Overview

```
Weeks 1-2:  Docker Fundamentals & Installation
Weeks 3-4:  Images & Dockerfiles
Weeks 5-6:  Networking & Storage
Weeks 7-8:  Docker Compose
Weeks 9-10: Security & Best Practices
Weeks 11-12: Production & Advanced Topics
```

**Time Commitment**: 10-15 hours per week  
**Prerequisites**: Basic Linux, command-line knowledge  
**Target**: Production-ready Docker expertise

---

## Week 1: Docker Fundamentals

### Learning Objectives
- Understand containerization vs virtualization
- Install Docker on your system
- Run your first containers
- Understand Docker architecture

### Topics

#### 1.1 What is Docker?

**Containerization:**
- Package application with dependencies
- Isolated, lightweight environments
- Consistent across development, testing, production
- Fast startup times (seconds vs minutes for VMs)

**Containers vs Virtual Machines:**
```
Virtual Machines:
┌─────────────────────────────────┐
│         Application             │
│         ┌─────────┐             │
│         │  Bins/  │             │
│         │  Libs   │             │
│         └─────────┘             │
│      Guest OS (GB)              │
│         Hypervisor              │
│         Host OS                 │
│         Infrastructure          │
└─────────────────────────────────┘

Containers:
┌─────────────────────────────────┐
│      Application                │
│      ┌─────────┐                │
│      │ Bins/   │                │
│      │ Libs    │                │
│      └─────────┘                │
│      Docker Engine              │
│      Host OS                    │
│      Infrastructure             │
└─────────────────────────────────┘
```

**Benefits:**
- **Portability**: Run anywhere
- **Efficiency**: Share OS kernel
- **Speed**: Fast startup/shutdown
- **Isolation**: Process-level isolation
- **Version Control**: Image versioning
- **Scalability**: Easy to replicate

#### 1.2 Docker Architecture

**Components:**
```
Docker Client (docker CLI)
    ↓ REST API
Docker Daemon (dockerd)
    ↓
containerd
    ↓
runc (container runtime)
```

**Key Elements:**
- **Docker Daemon**: Background service managing containers
- **Docker Client**: CLI tool to interact with daemon
- **Docker Images**: Read-only templates
- **Docker Containers**: Running instances
- **Docker Registry**: Image storage (Docker Hub)

#### 1.3 Installation

**Linux (Ubuntu/Debian):**
```bash
# Update package index
sudo apt-get update

# Install prerequisites
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# Add Docker's official GPG key
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Set up repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin

# Verify installation
sudo docker run hello-world

# Add user to docker group (optional, to run without sudo)
sudo usermod -aG docker $USER
newgrp docker
```

**macOS:**
```bash
# Download Docker Desktop
# https://www.docker.com/products/docker-desktop

# Or use Homebrew
brew install --cask docker

# Start Docker Desktop
open -a Docker

# Verify
docker version
```

**Windows:**
```powershell
# Download and install Docker Desktop
# https://www.docker.com/products/docker-desktop

# Enable WSL 2
wsl --install

# Verify
docker version
```

#### 1.4 First Container

```bash
# Hello World
docker run hello-world

# Interactive Ubuntu container
docker run -it ubuntu bash
# Inside container:
ls /
apt-get update
apt-get install -y curl
exit

# Run Nginx web server
docker run -d -p 8080:80 --name my-nginx nginx
# Access: http://localhost:8080

# View running containers
docker ps

# View logs
docker logs my-nginx

# Stop container
docker stop my-nginx

# Remove container
docker rm my-nginx
```

### Hands-On Practice
1. Install Docker on your machine
2. Run hello-world container
3. Start interactive Ubuntu container
4. Run Nginx and access web page
5. Explore container logs

### Resources
- Docker Docs: https://docs.docker.com/
- Docker Hub: https://hub.docker.com/

---

## Week 2: Container Lifecycle Management

### Learning Objectives
- Manage container lifecycle
- Understand container states
- Work with container logs and stats
- Execute commands in running containers

### Topics

#### 2.1 Container Lifecycle

**States:**
```
Created → Running → Paused → Stopped → Removed
```

**Commands:**
```bash
# Create container (don't start)
docker create --name myapp nginx

# Start container
docker start myapp

# Stop container (graceful, sends SIGTERM)
docker stop myapp

# Kill container (force, sends SIGKILL)
docker kill myapp

# Restart container
docker restart myapp

# Pause container (freeze process)
docker pause myapp

# Unpause container
docker unpause myapp

# Remove container
docker rm myapp

# Remove running container (force)
docker rm -f myapp
```

#### 2.2 Running Containers

**Interactive vs Detached:**
```bash
# Interactive (foreground)
docker run -it ubuntu bash

# Detached (background)
docker run -d nginx

# Detached with name
docker run -d --name web nginx

# Auto-remove after exit
docker run --rm -it ubuntu bash
```

**Port Mapping:**
```bash
# Map host port 8080 to container port 80
docker run -d -p 8080:80 nginx

# Map to random port
docker run -d -P nginx

# Check mapped ports
docker port container-name
```

**Environment Variables:**
```bash
# Single variable
docker run -e MY_VAR=value myapp

# Multiple variables
docker run -e VAR1=val1 -e VAR2=val2 myapp

# From file
docker run --env-file .env myapp
```

#### 2.3 Container Inspection

```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Show last created container
docker ps -l

# Container details (JSON)
docker inspect container-name

# Specific field
docker inspect --format='{{.NetworkSettings.IPAddress}}' container-name

# Container processes
docker top container-name

# Resource usage
docker stats

# Real-time stats for specific container
docker stats container-name
```

#### 2.4 Logs and Debugging

```bash
# View logs
docker logs container-name

# Follow logs (like tail -f)
docker logs -f container-name

# Last 100 lines
docker logs --tail 100 container-name

# Logs since timestamp
docker logs --since 2023-01-01T00:00:00 container-name

# Logs with timestamps
docker logs -t container-name

# Execute command in running container
docker exec container-name ls /app

# Interactive shell
docker exec -it container-name bash

# Execute as specific user
docker exec -u www-data container-name whoami
```

### Hands-On Practice
1. Create and manage container lifecycle
2. Run containers with port mapping
3. Use environment variables
4. Inspect container details
5. Debug containers with logs and exec

---

## Week 3: Docker Images

### Learning Objectives
- Understand Docker images and layers
- Pull and push images
- Tag images
- Search for images on Docker Hub

### Topics

#### 3.1 Image Concepts

**What is an Image?**
- Read-only template for containers
- Contains OS, application, and dependencies
- Built in layers (union filesystem)
- Each layer is cached

**Image Layers:**
```
Layer 5: Application files        (100 MB)
Layer 4: Application dependencies  (200 MB)
Layer 3: Runtime (JDK, Node, etc) (300 MB)
Layer 2: Package manager updates   (50 MB)
Layer 1: Base OS (ubuntu, alpine)  (5 MB)
─────────────────────────────────────────
Total: Shared layers reduce storage
```

#### 3.2 Working with Images

```bash
# List images
docker images
docker image ls

# Pull image
docker pull ubuntu:22.04

# Pull all tags
docker pull --all-tags ubuntu

# Search Docker Hub
docker search nginx

# Image details
docker inspect nginx:latest

# Image history (layers)
docker history nginx:latest

# Remove image
docker rmi nginx:latest

# Remove unused images
docker image prune

# Remove all images
docker image prune -a
```

#### 3.3 Image Tags

```bash
# Tag image
docker tag ubuntu:22.04 myubuntu:latest

# Tag for registry
docker tag myapp:latest registry.example.com/myapp:1.0

# Multiple tags
docker tag myapp:latest myapp:1.0
docker tag myapp:latest myapp:prod
```

**Tagging Best Practices:**
- Use semantic versioning: `1.0.0`, `1.0`, `1`, `latest`
- Use environment tags: `dev`, `staging`, `prod`
- Use git commit SHA: `abc123f`
- Avoid `latest` in production

#### 3.4 Docker Hub

```bash
# Login to Docker Hub
docker login

# Push image
docker push username/myapp:1.0

# Pull private image
docker pull username/private-repo:latest

# Logout
docker logout
```

**Image Naming:**
```
registry/repository:tag
docker.io/library/nginx:latest
registry.example.com/myteam/myapp:1.0
```

### Hands-On Practice
1. Pull various images
2. Inspect image layers
3. Tag images appropriately
4. Push image to Docker Hub
5. Understand layer caching

---

## Week 4: Building Docker Images (Dockerfile)

### Learning Objectives
- Write Dockerfiles
- Build custom images
- Understand build context
- Optimize image builds

### Topics

#### 4.1 Dockerfile Basics

**Dockerfile Structure:**
```dockerfile
# Comment
INSTRUCTION arguments
```

**Example Dockerfile:**
```dockerfile
# Base image
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy application
COPY . .

# Expose port
EXPOSE 3000

# Define entry point
CMD ["npm", "start"]
```

#### 4.2 Dockerfile Instructions

**FROM** - Base image
```dockerfile
FROM ubuntu:22.04
FROM node:18-alpine
FROM scratch  # Empty base for static binaries
```

**WORKDIR** - Set working directory
```dockerfile
WORKDIR /app
# All subsequent commands run from /app
```

**COPY** - Copy files
```dockerfile
COPY package.json /app/
COPY src/ /app/src/
COPY . .
```

**ADD** - Copy with features (extract tar, fetch URLs)
```dockerfile
ADD https://example.com/file.tar.gz /tmp/
ADD archive.tar.gz /app/
```

**RUN** - Execute commands during build
```dockerfile
RUN apt-get update && apt-get install -y \
    curl \
    vim \
    && rm -rf /var/lib/apt/lists/*

RUN npm install
```

**CMD** - Default command (can be overridden)
```dockerfile
CMD ["nginx", "-g", "daemon off;"]
CMD ["npm", "start"]
```

**ENTRYPOINT** - Main command (not easily overridden)
```dockerfile
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**ENV** - Environment variables
```dockerfile
ENV NODE_ENV=production
ENV APP_PORT=3000
```

**EXPOSE** - Document port
```dockerfile
EXPOSE 8080
EXPOSE 8080/tcp
EXPOSE 53/udp
```

**VOLUME** - Mount point
```dockerfile
VOLUME /data
VOLUME ["/var/log", "/var/db"]
```

**USER** - Set user
```dockerfile
USER appuser
USER 1000:1000
```

**ARG** - Build-time variables
```dockerfile
ARG VERSION=1.0
RUN echo "Building version $VERSION"
```

**LABEL** - Metadata
```dockerfile
LABEL maintainer="dev@example.com"
LABEL version="1.0"
```

**HEALTHCHECK** - Health check
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost/ || exit 1
```

#### 4.3 Building Images

```bash
# Build image
docker build -t myapp:1.0 .

# Build with different Dockerfile
docker build -t myapp -f Dockerfile.dev .

# Build with build args
docker build --build-arg VERSION=1.0 -t myapp .

# No cache build
docker build --no-cache -t myapp .

# View build progress
docker build --progress=plain -t myapp .
```

#### 4.4 Build Optimization

**Layer Caching:**
```dockerfile
# Bad: Changes in code invalidate npm install cache
COPY . .
RUN npm install

# Good: Leverage cache
COPY package*.json ./
RUN npm install
COPY . .
```

**Minimize Layers:**
```dockerfile
# Bad: Many layers
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y vim

# Good: Single layer
RUN apt-get update && apt-get install -y \
    curl \
    vim \
    && rm -rf /var/lib/apt/lists/*
```

**.dockerignore:**
```
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
*.md
dist
.DS_Store
```

### Hands-On Practice
1. Write Dockerfile for Node.js app
2. Build image with proper layering
3. Use .dockerignore effectively
4. Optimize build for caching
5. Understand build context

---

## Week 5: Multi-Stage Builds

### Learning Objectives
- Understand multi-stage builds
- Optimize image size
- Separate build and runtime
- Security improvements

### Topics

#### 5.1 Why Multi-Stage Builds?

**Problems with Single-Stage:**
- Large final image (includes build tools)
- Security risks (unnecessary tools in production)
- Slow image transfers

**Solution: Multi-Stage Builds**
- Separate build environment from runtime
- Copy only artifacts needed
- Smaller, more secure images

#### 5.2 Multi-Stage Example

**Java Application:**
```dockerfile
# Stage 1: Build
FROM maven:3.8-jdk-11 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2: Runtime
FROM openjdk:11-jre-slim
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Node.js Application:**
```dockerfile
# Stage 1: Build
FROM node:18 AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Runtime
FROM node:18-alpine
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

**Go Application:**
```dockerfile
# Stage 1: Build
FROM golang:1.20 AS build
WORKDIR /app
COPY go.* ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o app

# Stage 2: Runtime
FROM alpine:3.18
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=build /app/app .
EXPOSE 8080
CMD ["./app"]
```

#### 5.3 Size Comparison

```bash
# Before multi-stage
maven:3.8-jdk-11    700 MB
+ Application       100 MB
= Total            800 MB

# After multi-stage
openjdk:11-jre-slim 200 MB
+ Application        50 MB
= Total            250 MB

# Savings: 550 MB (69% reduction)
```

### Hands-On Practice
1. Convert single-stage to multi-stage
2. Compare image sizes
3. Build Java application with Maven
4. Build Node.js application
5. Optimize for minimal size

---

## Week 6: Docker Networking

### Learning Objectives
- Understand Docker networking
- Configure network drivers
- Container-to-container communication
- Connect to external networks

### Topics

#### 6.1 Network Drivers

**Bridge (Default):**
- Isolated network on single host
- Containers can communicate
- NAT to external network

**Host:**
- Use host's network directly
- No network isolation
- Better performance

**None:**
- No networking
- Complete isolation

**Overlay:**
- Multi-host networking
- Docker Swarm/Kubernetes

**Macvlan:**
- Assign MAC address to container
- Appear as physical device

#### 6.2 Bridge Network

```bash
# Create network
docker network create mynetwork

# Run containers on network
docker run -d --name web --network mynetwork nginx
docker run -d --name app --network mynetwork myapp

# Containers can communicate by name
docker exec app ping web

# Inspect network
docker network inspect mynetwork

# Connect existing container
docker network connect mynetwork existing-container

# Disconnect
docker network disconnect mynetwork container
```

#### 6.3 Port Publishing

```bash
# Publish single port
docker run -d -p 8080:80 nginx

# Publish to specific interface
docker run -d -p 127.0.0.1:8080:80 nginx

# Publish multiple ports
docker run -d -p 8080:80 -p 8443:443 nginx

# Publish to random port
docker run -d -P nginx

# Check published ports
docker port container-name
```

#### 6.4 DNS and Service Discovery

```bash
# Containers on same network can resolve by name
docker network create mynet
docker run -d --name db --network mynet postgres
docker run -d --name app --network mynet myapp

# From app container:
ping db  # Resolves to db container's IP
```

### Hands-On Practice
1. Create custom bridge network
2. Connect multiple containers
3. Test inter-container communication
4. Use DNS for service discovery
5. Publish ports correctly

---

## Week 7: Docker Volumes & Storage

### Learning Objectives
- Understand Docker storage
- Use volumes for persistent data
- Work with bind mounts
- Manage data lifecycle

### Topics

#### 7.1 Storage Types

**Volumes (Recommended):**
- Managed by Docker
- Stored in Docker area
- Can be shared
- Backed up easily

**Bind Mounts:**
- Map host directory
- Full host filesystem access
- Development use

**tmpfs Mounts:**
- Stored in memory
- Temporary data
- Not persisted

#### 7.2 Working with Volumes

```bash
# Create volume
docker volume create mydata

# List volumes
docker volume ls

# Inspect volume
docker volume inspect mydata

# Use volume
docker run -d -v mydata:/data nginx

# Anonymous volume
docker run -d -v /data nginx

# Remove volume
docker volume rm mydata

# Remove unused volumes
docker volume prune
```

#### 7.3 Bind Mounts

```bash
# Mount current directory
docker run -d -v $(pwd):/app nginx

# Mount specific directory
docker run -d -v /host/path:/container/path nginx

# Read-only mount
docker run -d -v $(pwd):/app:ro nginx
```

#### 7.4 Volume Examples

**Database:**
```bash
docker run -d \
  --name postgres \
  -v pgdata:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=secret \
  postgres:15
```

**Development:**
```bash
docker run -d \
  --name dev-app \
  -v $(pwd):/app \
  -p 3000:3000 \
  node:18 \
  npm run dev
```

**Backup Volume:**
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

### Hands-On Practice
1. Create and use volumes
2. Mount host directories
3. Share volumes between containers
4. Backup and restore volumes
5. Understand volume lifecycle

---

## Week 8: Docker Compose

### Learning Objectives
- Understand Docker Compose
- Write docker-compose.yml files
- Manage multi-container applications
- Use Compose in development

### Topics

#### 8.1 What is Docker Compose?

**Purpose:**
- Define multi-container applications
- Single configuration file (YAML)
- One command to start all services
- Development and testing environments

#### 8.2 Compose File Structure

```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html
    networks:
      - frontend
    depends_on:
      - app
  
  app:
    build: ./app
    environment:
      - DATABASE_URL=postgresql://db:5432/mydb
    networks:
      - frontend
      - backend
    depends_on:
      - db
  
  db:
    image: postgres:15
    environment:
      - POSTGRES_PASSWORD=secret
    volumes:
      - dbdata:/var/lib/postgresql/data
    networks:
      - backend

networks:
  frontend:
  backend:

volumes:
  dbdata:
```

#### 8.3 Compose Commands

```bash
# Start services
docker compose up

# Start in background
docker compose up -d

# Start specific service
docker compose up web

# Build and start
docker compose up --build

# Stop services
docker compose stop

# Stop and remove containers
docker compose down

# Stop and remove volumes
docker compose down -v

# View logs
docker compose logs

# Follow logs
docker compose logs -f

# Service logs
docker compose logs web

# List services
docker compose ps

# Execute command
docker compose exec web bash

# Restart service
docker compose restart web
```

#### 8.4 Real-World Example

**Full Stack Application:**
```yaml
version: '3.8'

services:
  # Frontend
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - REACT_APP_API_URL=http://localhost:8000
    volumes:
      - ./frontend/src:/app/src
    depends_on:
      - backend
  
  # Backend API
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://postgres:secret@db:5432/appdb
      - REDIS_URL=redis://cache:6379
    depends_on:
      - db
      - cache
    volumes:
      - ./backend:/app
    command: python manage.py runserver 0.0.0.0:8000
  
  # Database
  db:
    image: postgres:15
    environment:
      - POSTGRES_DB=appdb
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=secret
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
  
  # Redis Cache
  cache:
    image: redis:7-alpine
    ports:
      - "6379:6379"
  
  # Nginx Reverse Proxy
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - frontend
      - backend

volumes:
  postgres_data:
```

### Hands-On Practice
1. Create docker-compose.yml
2. Define multi-service application
3. Use networks and volumes
4. Manage application lifecycle
5. Debug with compose logs

---

## Week 9: Docker Security

### Learning Objectives
- Understand Docker security model
- Implement security best practices
- Scan images for vulnerabilities
- Secure containers in production

### Topics

#### 9.1 Image Security

**Use Official Images:**
```dockerfile
FROM node:18-alpine  # Official, minimal
```

**Scan for Vulnerabilities:**
```bash
# Docker Scout (built-in)
docker scout cves myimage:latest

# Trivy
trivy image myimage:latest

# Snyk
snyk container test myimage:latest
```

**Minimal Base Images:**
```dockerfile
FROM alpine:3.18      # 5 MB
FROM scratch          # 0 MB (static binaries)
FROM distroless/java  # Google distroless
```

#### 9.2 Dockerfile Security

```dockerfile
# Don't run as root
FROM node:18-alpine
RUN addgroup -g 1001 -S appgroup && \
    adduser -u 1001 -S appuser -G appgroup
USER appuser

# Use specific versions
FROM node:18.17.0-alpine3.18

# Don't embed secrets
# Bad:
ENV API_KEY=abc123

# Good: Use runtime secrets
ENV API_KEY=${API_KEY}

# Remove unnecessary tools
RUN apk del apk-tools

# Use read-only root filesystem
docker run --read-only myapp
```

#### 9.3 Runtime Security

```bash
# Run as non-root user
docker run --user 1000:1000 myapp

# Drop capabilities
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE myapp

# Read-only root filesystem
docker run --read-only myapp

# No new privileges
docker run --security-opt=no-new-privileges myapp

# Resource limits
docker run \
  --memory="512m" \
  --cpus="1.0" \
  --pids-limit=100 \
  myapp
```

#### 9.4 Network Security

```bash
# Disable inter-container communication
docker network create --internal isolated

# Use specific networks
docker run --network=backend-only myapp

# Limit port exposure
# Only expose what's necessary
```

### Hands-On Practice
1. Scan images for vulnerabilities
2. Create non-root user in Dockerfile
3. Implement resource limits
4. Use security options
5. Follow security best practices

---

## Week 10: Production Best Practices

### Learning Objectives
- Prepare Docker for production
- Implement health checks
- Configure logging
- Monitor containers

### Topics

#### 10.1 Health Checks

**Dockerfile:**
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1
```

**Docker Run:**
```bash
docker run -d \
  --health-cmd="curl -f http://localhost/health || exit 1" \
  --health-interval=30s \
  --health-timeout=3s \
  --health-retries=3 \
  myapp
```

**Check Health:**
```bash
docker ps  # Shows health status
docker inspect --format='{{.State.Health.Status}}' container
```

#### 10.2 Logging

**Logging Drivers:**
```bash
# json-file (default)
docker run --log-driver=json-file myapp

# syslog
docker run --log-driver=syslog myapp

# journald
docker run --log-driver=journald myapp

# fluentd
docker run --log-driver=fluentd myapp

# none (disable logging)
docker run --log-driver=none myapp
```

**Log Options:**
```bash
docker run \
  --log-driver=json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  myapp
```

#### 10.3 Resource Management

```bash
# Memory limits
docker run -m 512m myapp

# CPU limits
docker run --cpus=1.5 myapp

# CPU shares (relative weight)
docker run --cpu-shares=512 myapp

# Limit PIDs
docker run --pids-limit=100 myapp

# Disk I/O
docker run --device-read-bps /dev/sda:1mb myapp
```

#### 10.4 Restart Policies

```bash
# Always restart
docker run --restart=always myapp

# Restart on failure
docker run --restart=on-failure:3 myapp

# Unless stopped manually
docker run --restart=unless-stopped myapp
```

### Hands-On Practice
1. Implement health checks
2. Configure logging
3. Set resource limits
4. Test restart policies
5. Monitor container health

---

## Week 11: CI/CD Integration

### Learning Objectives
- Build images in CI/CD
- Tag images appropriately
- Push to registries
- Deploy containers

### Topics

#### 11.1 Building in CI/CD

**GitHub Actions:**
```yaml
name: Docker Build

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: |
            myapp:latest
            myapp:${{ github.sha }}
          cache-from: type=registry,ref=myapp:latest
          cache-to: type=inline
```

#### 11.2 Image Tagging Strategy

```bash
# Semantic versioning
myapp:1.0.0
myapp:1.0
myapp:1
myapp:latest

# Git commit SHA
myapp:abc123f

# Build number
myapp:build-456

# Environment
myapp:dev
myapp:staging
myapp:prod
```

### Hands-On Practice
1. Build images in GitHub Actions
2. Implement tagging strategy
3. Push to Docker Hub
4. Automate deployment

---

## Week 12: Advanced Topics & Optimization

### Learning Objectives
- Optimize image size
- Improve build performance
- Use BuildKit features
- Prepare for Kubernetes

### Topics

#### 12.1 Image Optimization

**Techniques:**
```dockerfile
# 1. Use smaller base images
FROM alpine:3.18

# 2. Multi-stage builds
FROM node:18 AS build
# ... build steps
FROM node:18-alpine
COPY --from=build /app/dist /app/dist

# 3. Remove cache
RUN apt-get update && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*

# 4. Combine commands
RUN apk add --no-cache curl vim

# 5. Order layers by change frequency
COPY package*.json ./
RUN npm install
COPY . .
```

#### 12.2 BuildKit Features

```bash
# Enable BuildKit
export DOCKER_BUILDKIT=1

# Secrets
docker build --secret id=mysecret,src=secret.txt .

# SSH forwarding
docker build --ssh default .

# Cache mounts
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install -r requirements.txt
```

#### 12.3 Kubernetes Preparation

**Convert to K8s:**
- Docker Compose → Kompose → Kubernetes YAML
- Understand Pods vs Containers
- ConfigMaps and Secrets
- Services and Ingress

### Hands-On Practice
1. Optimize image to < 50MB
2. Use BuildKit features
3. Implement caching strategies
4. Prepare for Kubernetes migration

---

## Certification & Next Steps

### Certifications
- Docker Certified Associate (DCA)

### Continue Learning
1. **Kubernetes** - Container orchestration
2. **Docker Swarm** - Docker's orchestrator
3. **Service Mesh** - Istio, Linkerd
4. **Monitoring** - Prometheus, Grafana

---

## Summary

After 12 weeks, you will have mastered:
✅ Docker architecture and fundamentals  
✅ Image creation and optimization  
✅ Container lifecycle management  
✅ Networking and storage  
✅ Docker Compose  
✅ Security best practices  
✅ Production deployment  
✅ CI/CD integration

**Next Package**: [Kubernetes](../kubernetes/) for container orchestration at scale!

**Happy Learning! 🐋**


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

