# Docker Quick Reference Guide

Essential Docker commands, configurations, and templates for daily operations.

---

## Table of Contents
1. [Installation](#installation)
2. [Container Commands](#container-commands)
3. [Image Commands](#image-commands)
4. [Dockerfile Reference](#dockerfile-reference)
5. [Docker Compose](#docker-compose)
6. [Networking](#networking)
7. [Volumes & Storage](#volumes--storage)
8. [System Commands](#system-commands)

---

## Installation

### Linux (Ubuntu/Debian)
```bash
# Quick install script
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add user to docker group
sudo usermod -aG docker $USER
newgrp docker

# Verify
docker --version
docker run hello-world
```

### macOS
```bash
# Using Homebrew
brew install --cask docker

# Or download Docker Desktop
# https://www.docker.com/products/docker-desktop
```

### Windows
```powershell
# Download Docker Desktop
# https://www.docker.com/products/docker-desktop

# Enable WSL 2
wsl --install
```

---

## Container Commands

### Running Containers
```bash
# Run container
docker run nginx

# Run in background (-d detached)
docker run -d nginx

# Run with name
docker run -d --name my-nginx nginx

# Run interactively (-it)
docker run -it ubuntu bash

# Run with port mapping
docker run -d -p 8080:80 nginx

# Run with environment variables
docker run -d -e MY_VAR=value myapp

# Run with volume
docker run -d -v mydata:/data nginx

# Run with auto-remove
docker run --rm -it ubuntu bash

# Run with resource limits
docker run -d --memory="512m" --cpus="1.0" nginx

# Run with restart policy
docker run -d --restart=always nginx

# Run as specific user
docker run --user 1000:1000 myapp
```

### Managing Containers
```bash
# List running containers
docker ps

# List all containers
docker ps -a

# List container IDs only
docker ps -q

# Start container
docker start container-name

# Stop container
docker stop container-name

# Restart container
docker restart container-name

# Pause container
docker pause container-name

# Unpause container
docker unpause container-name

# Kill container
docker kill container-name

# Remove container
docker rm container-name

# Remove running container (force)
docker rm -f container-name

# Remove all stopped containers
docker container prune

# Stop all running containers
docker stop $(docker ps -q)

# Remove all containers
docker rm $(docker ps -aq)
```

### Container Inspection
```bash
# View logs
docker logs container-name

# Follow logs
docker logs -f container-name

# Last 100 lines
docker logs --tail 100 container-name

# Logs with timestamps
docker logs -t container-name

# Container details
docker inspect container-name

# Specific field
docker inspect -f '{{.NetworkSettings.IPAddress}}' container-name

# Container processes
docker top container-name

# Resource usage
docker stats

# Real-time stats
docker stats container-name

# Container changes
docker diff container-name

# Port mappings
docker port container-name
```

### Executing Commands
```bash
# Execute command
docker exec container-name ls /app

# Interactive shell
docker exec -it container-name bash

# Execute as specific user
docker exec -u www-data container-name whoami

# Execute with environment variables
docker exec -e VAR=value container-name env

# Attach to container
docker attach container-name

# Copy files to container
docker cp file.txt container-name:/path/

# Copy files from container
docker cp container-name:/path/file.txt .
```

---

## Image Commands

### Working with Images
```bash
# List images
docker images
docker image ls

# Pull image
docker pull nginx:latest

# Pull specific version
docker pull nginx:1.25

# Search Docker Hub
docker search nginx

# Image details
docker inspect nginx:latest

# Image history (layers)
docker history nginx:latest

# Remove image
docker rmi nginx:latest

# Remove by ID
docker rmi abc123

# Force remove
docker rmi -f nginx:latest

# Remove unused images
docker image prune

# Remove all images
docker image prune -a

# Save image to tar
docker save nginx:latest > nginx.tar
docker save -o nginx.tar nginx:latest

# Load image from tar
docker load < nginx.tar
docker load -i nginx.tar

# Export container to tar
docker export container-name > container.tar

# Import tar as image
docker import container.tar myimage:latest
```

### Building Images
```bash
# Build from Dockerfile
docker build -t myapp:1.0 .

# Build with different Dockerfile
docker build -t myapp -f Dockerfile.dev .

# Build with build args
docker build --build-arg VERSION=1.0 -t myapp .

# No cache build
docker build --no-cache -t myapp .

# Build with tag
docker build -t myapp:latest -t myapp:1.0 .

# Build with target stage
docker build --target production -t myapp .

# Pull base image before building
docker build --pull -t myapp .

# View build output
docker build --progress=plain -t myapp .
```

### Tagging & Pushing
```bash
# Tag image
docker tag myapp:latest myapp:1.0

# Tag for registry
docker tag myapp:latest registry.example.com/myapp:1.0

# Login to registry
docker login
docker login registry.example.com

# Push image
docker push myapp:1.0

# Push to private registry
docker push registry.example.com/myapp:1.0

# Logout
docker logout
```

---

## Dockerfile Reference

### Basic Dockerfile
```dockerfile
# Base image
FROM node:18-alpine

# Metadata
LABEL maintainer="dev@example.com"
LABEL version="1.0"

# Set working directory
WORKDIR /app

# Copy files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application
COPY . .

# Expose port
EXPOSE 3000

# Set environment variables
ENV NODE_ENV=production

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
USER nodejs

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD node healthcheck.js || exit 1

# Start command
CMD ["node", "server.js"]
```

### Multi-Stage Build
```dockerfile
# Build stage
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

### Java Application
```dockerfile
# Build stage
FROM maven:3.8-jdk-11 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package -DskipTests

# Runtime stage
FROM openjdk:11-jre-slim
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Python Application
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["python", "app.py"]
```

### Dockerfile Instructions
```dockerfile
# FROM - Base image
FROM ubuntu:22.04
FROM node:18-alpine AS builder

# WORKDIR - Set working directory
WORKDIR /app

# COPY - Copy files
COPY file.txt /app/
COPY src/ /app/src/
COPY --from=builder /app/dist /app/

# ADD - Copy with extract and URL support
ADD https://example.com/file.tar.gz /tmp/
ADD archive.tar.gz /app/

# RUN - Execute commands
RUN apt-get update && apt-get install -y curl
RUN npm install

# ENV - Set environment variables
ENV NODE_ENV=production
ENV PORT=3000

# ARG - Build-time variables
ARG VERSION=1.0
ARG BUILD_DATE

# EXPOSE - Document ports
EXPOSE 8080
EXPOSE 8080/tcp 53/udp

# VOLUME - Mount points
VOLUME /data
VOLUME ["/var/log", "/var/db"]

# USER - Set user/group
USER appuser
USER 1000:1000

# CMD - Default command
CMD ["nginx", "-g", "daemon off;"]
CMD npm start

# ENTRYPOINT - Main executable
ENTRYPOINT ["python", "app.py"]

# HEALTHCHECK - Container health
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -f http://localhost/health || exit 1

# LABEL - Metadata
LABEL version="1.0"
LABEL description="My application"

# SHELL - Default shell
SHELL ["/bin/bash", "-c"]

# STOPSIGNAL - Stop signal
STOPSIGNAL SIGTERM

# ONBUILD - Trigger on child build
ONBUILD COPY . /app
```

### .dockerignore
```
# Version control
.git
.gitignore
.gitattributes

# Dependencies
node_modules
vendor
__pycache__
*.pyc

# Build artifacts
dist
build
target
*.jar
*.war

# Logs
*.log
logs

# OS files
.DS_Store
Thumbs.db

# IDE
.vscode
.idea
*.swp

# Environment
.env
.env.local

# Documentation
README.md
LICENSE
*.md

# Test files
test
tests
*.test.js
```

---

## Docker Compose

### Basic Compose File
```yaml
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html
    networks:
      - frontend
    restart: unless-stopped
  
  app:
    build:
      context: ./app
      dockerfile: Dockerfile
    environment:
      - DATABASE_URL=postgresql://db:5432/mydb
      - REDIS_URL=redis://cache:6379
    depends_on:
      - db
      - cache
    networks:
      - frontend
      - backend
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 3s
      retries: 3
  
  db:
    image: postgres:15
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: secret
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - backend
  
  cache:
    image: redis:7-alpine
    networks:
      - backend

networks:
  frontend:
  backend:

volumes:
  postgres_data:
```

### Compose Commands
```bash
# Start services
docker compose up

# Start in background
docker compose up -d

# Build and start
docker compose up --build

# Start specific service
docker compose up web

# Scale services
docker compose up --scale app=3

# Stop services
docker compose stop

# Stop and remove
docker compose down

# Remove volumes too
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
docker compose exec web sh

# Run one-off command
docker compose run web npm test

# Restart service
docker compose restart web

# Build services
docker compose build

# Pull images
docker compose pull

# Validate compose file
docker compose config

# View processes
docker compose top
```

### Advanced Compose
```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        - VERSION=1.0
        - BUILD_DATE=2024-01-01
      target: production
    image: myapp:latest
    container_name: myapp
    hostname: app-server
    ports:
      - "3000:3000"
      - "3001:3001"
    expose:
      - "3000"
    environment:
      NODE_ENV: production
      LOG_LEVEL: info
    env_file:
      - .env
      - .env.production
    volumes:
      - ./app:/app:ro
      - app_data:/app/data
      - type: tmpfs
        target: /tmp
    networks:
      - frontend
      - backend
    depends_on:
      db:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    restart: unless-stopped
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 512M
        reservations:
          cpus: '1.0'
          memory: 256M
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
```

---

## Networking

### Network Commands
```bash
# List networks
docker network ls

# Create network
docker network create mynetwork

# Create with driver
docker network create --driver bridge mynetwork

# Create with subnet
docker network create --subnet=172.18.0.0/16 mynetwork

# Inspect network
docker network inspect mynetwork

# Connect container
docker network connect mynetwork container-name

# Disconnect container
docker network disconnect mynetwork container-name

# Remove network
docker network rm mynetwork

# Remove unused networks
docker network prune
```

### Network Types
```bash
# Bridge (default)
docker network create --driver bridge my-bridge

# Host (use host network)
docker run --network host nginx

# None (no networking)
docker run --network none ubuntu

# Overlay (multi-host, Swarm)
docker network create --driver overlay my-overlay

# Macvlan (assign MAC address)
docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 \
  my-macvlan
```

### Container Communication
```bash
# Create network
docker network create app-network

# Run containers on network
docker run -d --name web --network app-network nginx
docker run -d --name app --network app-network myapp

# Containers can communicate by name
docker exec app ping web
docker exec app curl http://web:80
```

---

## Volumes & Storage

### Volume Commands
```bash
# Create volume
docker volume create mydata

# List volumes
docker volume ls

# Inspect volume
docker volume inspect mydata

# Remove volume
docker volume rm mydata

# Remove unused volumes
docker volume prune

# Remove all volumes
docker volume rm $(docker volume ls -q)
```

### Using Volumes
```bash
# Named volume
docker run -d -v mydata:/data nginx

# Anonymous volume
docker run -d -v /data nginx

# Bind mount (host directory)
docker run -d -v $(pwd):/app nginx

# Bind mount (Windows)
docker run -d -v C:\Users\myuser\app:/app nginx

# Read-only mount
docker run -d -v mydata:/data:ro nginx

# Volume from another container
docker run -d --volumes-from container1 nginx

# tmpfs mount (temporary, in-memory)
docker run -d --tmpfs /tmp nginx
```

### Volume Backup/Restore
```bash
# Backup volume
docker run --rm \
  -v mydata:/data \
  -v $(pwd):/backup \
  ubuntu \
  tar czf /backup/backup.tar.gz /data

# Restore volume
docker run --rm \
  -v mydata:/data \
  -v $(pwd):/backup \
  ubuntu \
  tar xzf /backup/backup.tar.gz -C /
```

---

## System Commands

### System Information
```bash
# Docker version
docker --version
docker version

# Docker info
docker info

# Disk usage
docker system df

# Detailed disk usage
docker system df -v

# System events (real-time)
docker events

# System events (filtered)
docker events --filter 'type=container'
```

### Cleanup Commands
```bash
# Remove stopped containers
docker container prune

# Remove unused images
docker image prune

# Remove unused networks
docker network prune

# Remove unused volumes
docker volume prune

# Remove everything unused
docker system prune

# Remove everything (including volumes)
docker system prune -a --volumes

# Remove all stopped containers
docker rm $(docker ps -aq)

# Remove all images
docker rmi $(docker images -q)

# Remove dangling images
docker rmi $(docker images -f "dangling=true" -q)
```

### Resource Management
```bash
# View resource usage
docker stats

# Limit memory
docker run -m 512m myapp

# Limit CPU
docker run --cpus=1.5 myapp

# Limit CPU shares
docker run --cpu-shares=512 myapp

# Limit PIDs
docker run --pids-limit=100 myapp

# Update container resources
docker update --memory 1g --cpus 2 container-name
```

---

## Common Patterns

### Development Setup
```bash
# Run with live reload
docker run -d \
  -v $(pwd):/app \
  -p 3000:3000 \
  -e NODE_ENV=development \
  node:18 \
  npm run dev
```

### Database Container
```bash
# PostgreSQL
docker run -d \
  --name postgres \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=mydb \
  -v pgdata:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:15

# MySQL
docker run -d \
  --name mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=mydb \
  -v mysql_data:/var/lib/mysql \
  -p 3306:3306 \
  mysql:8

# MongoDB
docker run -d \
  --name mongodb \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=secret \
  -v mongo_data:/data/db \
  -p 27017:27017 \
  mongo:7

# Redis
docker run -d \
  --name redis \
  -v redis_data:/data \
  -p 6379:6379 \
  redis:7-alpine
```

### CI/CD Image Build
```bash
# Build with metadata
docker build \
  --build-arg VERSION=1.0.0 \
  --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
  --build-arg VCS_REF=$(git rev-parse --short HEAD) \
  -t myapp:1.0.0 \
  -t myapp:latest \
  .

# Push to registry
docker push myapp:1.0.0
docker push myapp:latest
```

---

## Security Best Practices

### Secure Run
```bash
docker run -d \
  --read-only \
  --security-opt=no-new-privileges \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  --user 1000:1000 \
  -m 512m \
  --cpus=1.0 \
  myapp
```

### Scan Images
```bash
# Docker Scout (built-in)
docker scout cves myapp:latest

# Trivy
trivy image myapp:latest

# Snyk
snyk container test myapp:latest
```

---

## Troubleshooting Commands

```bash
# Check container logs
docker logs --tail 100 container-name

# Follow logs
docker logs -f container-name

# Inspect container
docker inspect container-name

# Check processes
docker top container-name

# Container IP address
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container-name

# Container environment variables
docker inspect -f '{{range .Config.Env}}{{println .}}{{end}}' container-name

# Execute debug commands
docker exec -it container-name bash
docker exec -it container-name sh

# Check Docker daemon logs
# Linux
sudo journalctl -u docker

# macOS/Windows
# Check Docker Desktop logs
```

---

## Useful Aliases

```bash
# Add to ~/.bashrc or ~/.zshrc

alias d='docker'
alias dc='docker compose'
alias dps='docker ps'
alias dpsa='docker ps -a'
alias di='docker images'
alias dex='docker exec -it'
alias dlog='docker logs -f'
alias dstop='docker stop $(docker ps -q)'
alias drm='docker rm $(docker ps -aq)'
alias drmi='docker rmi $(docker images -q)'
alias dprune='docker system prune -af --volumes'
```

---

## Environment Variables

### Docker Daemon
```bash
# Docker host
export DOCKER_HOST=tcp://192.168.1.10:2375

# Enable BuildKit
export DOCKER_BUILDKIT=1

# Compose file
export COMPOSE_FILE=docker-compose.yml

# Compose project name
export COMPOSE_PROJECT_NAME=myproject
```

---

**Tip**: Bookmark this page for quick access to essential Docker commands!


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

