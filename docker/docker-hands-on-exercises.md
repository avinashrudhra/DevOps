# Docker Hands-On Exercises

Complete practical exercises to master Docker from beginner to advanced level.

---

## Beginner Level Exercises

### Exercise 1: Install Docker and Run Hello World
**Objective**: Set up Docker environment and verify installation

**Tasks:**
1. Install Docker on your system
2. Verify installation
3. Run hello-world container
4. Understand what happened

<details>
<summary>Solution</summary>

```bash
# Install Docker (Ubuntu/Debian)
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add user to docker group
sudo usermod -aG docker $USER
newgrp docker

# Verify installation
docker --version
docker version
docker info

# Run hello-world
docker run hello-world

# What happened:
# 1. Docker client contacted Docker daemon
# 2. Daemon pulled hello-world image from Docker Hub
# 3. Daemon created container from image
# 4. Daemon ran container, which printed message
# 5. Container exited

# View downloaded image
docker images

# View container (exited)
docker ps -a
```

**Verification:**
- Docker installed successfully
- hello-world message displayed
- Image appears in `docker images`
- Container appears in `docker ps -a`
</details>

---

### Exercise 2: Run Interactive Ubuntu Container
**Objective**: Work with interactive containers

**Tasks:**
1. Run Ubuntu container interactively
2. Install packages inside container
3. Create files
4. Exit and observe state

<details>
<summary>Solution</summary>

```bash
# Run Ubuntu interactively
docker run -it ubuntu bash

# Inside container:
# Check OS version
cat /etc/os-release

# Update packages
apt-get update

# Install curl
apt-get install -y curl

# Test curl
curl https://www.google.com

# Create a file
echo "Hello Docker" > /tmp/test.txt
cat /tmp/test.txt

# Check hostname (container ID)
hostname

# Exit container
exit

# Outside: Check container status
docker ps -a

# Try to restart and reconnect
CONTAINER_ID=$(docker ps -lq)
docker start $CONTAINER_ID
docker attach $CONTAINER_ID

# Your file is still there!
cat /tmp/test.txt

# Exit again
exit
```

**Learning Points:**
- `-it` = interactive with terminal
- Changes persist in stopped container
- Container has unique hostname
- Can restart and reattach to container
</details>

---

### Exercise 3: Run Web Server with Port Mapping
**Objective**: Expose container ports to host

**Tasks:**
1. Run Nginx container
2. Map port 8080 to port 80
3. Access web server from browser
4. View logs

<details>
<summary>Solution</summary>

```bash
# Run Nginx in background
docker run -d -p 8080:80 --name my-nginx nginx

# Check if running
docker ps

# Access web server
# Browser: http://localhost:8080
# Or CLI:
curl http://localhost:8080

# View logs
docker logs my-nginx

# Follow logs in real-time
docker logs -f my-nginx

# In another terminal, make requests
curl http://localhost:8080

# Check port mapping
docker port my-nginx

# Inspect container
docker inspect my-nginx | grep IPAddress

# Stop container
docker stop my-nginx

# Try to access - should fail
curl http://localhost:8080

# Start again
docker start my-nginx

# Access works again
curl http://localhost:8080

# Cleanup
docker stop my-nginx
docker rm my-nginx
```

**Learning Points:**
- `-d` runs in detached mode (background)
- `-p host:container` maps ports
- `docker logs` shows container output
- Container keeps running after exit
</details>

---

### Exercise 4: Work with Environment Variables
**Objective**: Pass configuration via environment variables

**Tasks:**
1. Run container with environment variables
2. Access env vars inside container
3. Use env file

<details>
<summary>Solution</summary>

```bash
# Run with single env var
docker run -e MY_VAR=hello ubuntu env | grep MY_VAR

# Run with multiple env vars
docker run \
  -e VAR1=value1 \
  -e VAR2=value2 \
  -e VAR3=value3 \
  ubuntu env

# Interactive with env vars
docker run -it \
  -e USER_NAME=john \
  -e USER_ROLE=admin \
  ubuntu bash

# Inside container:
echo $USER_NAME
echo $USER_ROLE
exit

# Create env file
cat > app.env <<EOF
DATABASE_HOST=db.example.com
DATABASE_PORT=5432
DATABASE_NAME=mydb
API_KEY=secret123
DEBUG=true
EOF

# Run with env file
docker run --env-file app.env ubuntu env

# Real example: PostgreSQL
docker run -d \
  --name postgres \
  -e POSTGRES_USER=admin \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=myapp \
  postgres:15

# Verify
docker exec postgres env | grep POSTGRES

# Cleanup
docker stop postgres
docker rm postgres
rm app.env
```
</details>

---

### Exercise 5: Manage Container Lifecycle
**Objective**: Understand container states

**Tasks:**
1. Create, start, stop, restart container
2. Pause and unpause
3. Remove container

<details>
<summary>Solution</summary>

```bash
# Create container (don't start)
docker create --name lifecycle-test nginx

# Check status (Created)
docker ps -a

# Start container
docker start lifecycle-test

# Check status (Running)
docker ps

# Stop container (graceful)
docker stop lifecycle-test

# Check status (Exited)
docker ps -a

# Start again
docker start lifecycle-test

# Restart (stop + start)
docker restart lifecycle-test

# Pause container (freeze processes)
docker pause lifecycle-test

# Check status (Paused)
docker ps

# Try to access - should timeout
curl http://localhost

# Unpause
docker unpause lifecycle-test

# Access works
curl http://localhost

# Kill container (force stop)
docker kill lifecycle-test

# Remove container
docker rm lifecycle-test

# Try to list - gone
docker ps -a | grep lifecycle-test

# All in one: run with auto-remove
docker run --rm nginx echo "This container will be removed"
docker ps -a | grep nginx  # Not found
```

**State Diagram:**
```
Created → Running → Paused → Running → Stopped → Removed
```
</details>

---

## Intermediate Level Exercises

### Exercise 6: Build Your First Docker Image
**Objective**: Create custom image with Dockerfile

**Tasks:**
1. Create simple Node.js app
2. Write Dockerfile
3. Build image
4. Run container from image

<details>
<summary>Solution</summary>

```bash
# Create project directory
mkdir my-node-app
cd my-node-app

# Create Node.js app
cat > server.js <<'EOF'
const http = require('http');
const port = 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello from Docker!\n');
});

server.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
EOF

# Create package.json
cat > package.json <<'EOF'
{
  "name": "my-node-app",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  }
}
EOF

# Create Dockerfile
cat > Dockerfile <<'EOF'
FROM node:18-alpine
WORKDIR /app
COPY package.json .
COPY server.js .
EXPOSE 3000
CMD ["npm", "start"]
EOF

# Build image
docker build -t my-node-app:1.0 .

# View image
docker images | grep my-node-app

# View image layers
docker history my-node-app:1.0

# Run container
docker run -d -p 3000:3000 --name nodeapp my-node-app:1.0

# Test
curl http://localhost:3000

# View logs
docker logs nodeapp

# Cleanup
docker stop nodeapp
docker rm nodeapp
cd ..
```
</details>

---

### Exercise 7: Optimize Docker Image Size
**Objective**: Reduce image size using best practices

**Tasks:**
1. Create image with large base
2. Optimize using Alpine
3. Use multi-stage build
4. Compare sizes

<details>
<summary>Solution</summary>

```bash
mkdir optimize-demo
cd optimize-demo

# Create Go application
cat > main.go <<'EOF'
package main
import "fmt"
func main() {
    fmt.Println("Hello from optimized container!")
}
EOF

# Version 1: Large base image
cat > Dockerfile.v1 <<'EOF'
FROM golang:1.20
WORKDIR /app
COPY main.go .
RUN go build -o app main.go
CMD ["./app"]
EOF

docker build -f Dockerfile.v1 -t goapp:v1 .
docker images goapp:v1
# Size: ~800MB

# Version 2: Alpine base
cat > Dockerfile.v2 <<'EOF'
FROM golang:1.20-alpine
WORKDIR /app
COPY main.go .
RUN go build -o app main.go
CMD ["./app"]
EOF

docker build -f Dockerfile.v2 -t goapp:v2 .
docker images goapp:v2
# Size: ~300MB

# Version 3: Multi-stage build
cat > Dockerfile.v3 <<'EOF'
# Build stage
FROM golang:1.20-alpine AS builder
WORKDIR /app
COPY main.go .
RUN go build -o app main.go

# Runtime stage
FROM alpine:3.18
WORKDIR /app
COPY --from=builder /app/app .
CMD ["./app"]
EOF

docker build -f Dockerfile.v3 -t goapp:v3 .
docker images goapp:v3
# Size: ~10MB

# Version 4: Scratch (smallest)
cat > Dockerfile.v4 <<'EOF'
FROM golang:1.20-alpine AS builder
WORKDIR /app
COPY main.go .
RUN CGO_ENABLED=0 go build -o app main.go

FROM scratch
COPY --from=builder /app/app /app
CMD ["/app"]
EOF

docker build -f Dockerfile.v4 -t goapp:v4 .
docker images goapp:v4
# Size: ~2MB

# Compare all versions
docker images goapp

# Test each version
docker run --rm goapp:v1
docker run --rm goapp:v2
docker run --rm goapp:v3
docker run --rm goapp:v4

# Cleanup
cd ..
```

**Size Comparison:**
- v1 (golang): ~800MB
- v2 (alpine): ~300MB
- v3 (multi-stage): ~10MB
- v4 (scratch): ~2MB
</details>

---

### Exercise 8: Docker Networking - Multi-Container Communication
**Objective**: Connect multiple containers

**Tasks:**
1. Create custom network
2. Run database container
3. Run application container
4. Test communication

<details>
<summary>Solution</summary>

```bash
# Create network
docker network create app-network

# Inspect network
docker network inspect app-network

# Run PostgreSQL on network
docker run -d \
  --name database \
  --network app-network \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=myapp \
  postgres:15

# Wait for database to be ready
sleep 5

# Run application container
docker run -d \
  --name webapp \
  --network app-network \
  -p 8080:80 \
  nginx

# Test connectivity from webapp to database
docker exec webapp ping -c 3 database

# DNS resolution works!
docker exec webapp nslookup database

# Install psql in webapp to test database
docker exec webapp apt-get update
docker exec webapp apt-get install -y postgresql-client

# Connect to database by name
docker exec webapp psql -h database -U postgres -d myapp -c "SELECT version();"

# Run another container on same network
docker run -it --rm \
  --network app-network \
  ubuntu bash

# Inside new container:
apt-get update && apt-get install -y postgresql-client iputils-ping
ping -c 3 database
ping -c 3 webapp
psql -h database -U postgres -d myapp
# \q to exit psql
exit

# View network connections
docker network inspect app-network

# Cleanup
docker stop webapp database
docker rm webapp database
docker network rm app-network
```

**Learning Points:**
- Containers on same network can communicate
- DNS automatically resolves container names
- No port mapping needed for inter-container communication
- Port mapping only needed for host access
</details>

---

### Exercise 9: Persistent Data with Volumes
**Objective**: Manage data persistence

**Tasks:**
1. Create named volume
2. Use volume in container
3. Verify data persistence
4. Backup and restore

<details>
<summary>Solution</summary>

```bash
# Create named volume
docker volume create mydata

# Inspect volume
docker volume inspect mydata

# Run container with volume
docker run -it --rm \
  -v mydata:/data \
  ubuntu bash

# Inside container:
cd /data
echo "Hello from container 1" > file1.txt
echo "Important data" > file2.txt
ls -la
exit

# Run another container with same volume
docker run -it --rm \
  -v mydata:/data \
  ubuntu bash

# Inside container:
cd /data
ls -la  # Files are there!
cat file1.txt
echo "More data" >> file1.txt
exit

# Database example with volume
docker run -d \
  --name postgres-db \
  -v pgdata:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=secret \
  postgres:15

# Create database and table
docker exec postgres-db psql -U postgres -c "CREATE DATABASE testdb;"
docker exec postgres-db psql -U postgres -d testdb -c "CREATE TABLE users (id INT, name VARCHAR(50));"
docker exec postgres-db psql -U postgres -d testdb -c "INSERT INTO users VALUES (1, 'John Doe');"

# Query data
docker exec postgres-db psql -U postgres -d testdb -c "SELECT * FROM users;"

# Stop and remove container
docker stop postgres-db
docker rm postgres-db

# Start new container with same volume
docker run -d \
  --name postgres-db-new \
  -v pgdata:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=secret \
  postgres:15

# Wait for startup
sleep 5

# Data is still there!
docker exec postgres-db-new psql -U postgres -d testdb -c "SELECT * FROM users;"

# Backup volume
docker run --rm \
  -v pgdata:/data \
  -v $(pwd):/backup \
  ubuntu \
  tar czf /backup/pgdata-backup.tar.gz /data

ls -lh pgdata-backup.tar.gz

# Create new volume
docker volume create pgdata-restored

# Restore to new volume
docker run --rm \
  -v pgdata-restored:/data \
  -v $(pwd):/backup \
  ubuntu \
  tar xzf /backup/pgdata-backup.tar.gz -C /

# Cleanup
docker stop postgres-db-new
docker rm postgres-db-new
docker volume rm pgdata pgdata-restored mydata
rm pgdata-backup.tar.gz
```
</details>

---

### Exercise 10: Docker Compose - Multi-Service Application
**Objective**: Define and run multi-container app

**Tasks:**
1. Create docker-compose.yml
2. Define multiple services
3. Use networks and volumes
4. Manage application

<details>
<summary>Solution</summary>

```bash
mkdir fullstack-app
cd fullstack-app

# Create docker-compose.yml
cat > docker-compose.yml <<'EOF'
version: '3.8'

services:
  # Frontend
  frontend:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html:ro
    depends_on:
      - backend
    networks:
      - frontend

  # Backend API
  backend:
    image: node:18-alpine
    working_dir: /app
    volumes:
      - ./backend:/app
    command: node server.js
    environment:
      - DATABASE_URL=postgresql://postgres:secret@database:5432/appdb
      - REDIS_URL=redis://cache:6379
    depends_on:
      - database
      - cache
    networks:
      - frontend
      - backend

  # Database
  database:
    image: postgres:15
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=secret
      - POSTGRES_DB=appdb
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - backend

  # Cache
  cache:
    image: redis:7-alpine
    networks:
      - backend

networks:
  frontend:
  backend:

volumes:
  db_data:
EOF

# Create HTML
mkdir html
cat > html/index.html <<'EOF'
<!DOCTYPE html>
<html>
<head><title>Fullstack App</title></head>
<body>
  <h1>Hello from Docker Compose!</h1>
</body>
</html>
EOF

# Create backend
mkdir backend
cat > backend/server.js <<'EOF'
const http = require('http');
const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'application/json');
  res.end(JSON.stringify({ message: 'Hello from backend!' }));
});
server.listen(3000, () => console.log('Backend running on port 3000'));
EOF

# Start all services
docker compose up -d

# View running services
docker compose ps

# View logs
docker compose logs

# Follow logs
docker compose logs -f backend

# Test services
curl http://localhost
curl http://localhost:3000  # If backend exposed

# Execute command in service
docker compose exec backend sh
# Inside: apk add curl
# curl http://database:5432  # Test connectivity
# exit

# Stop services
docker compose stop

# Start again
docker compose start

# Restart specific service
docker compose restart backend

# View service logs
docker compose logs database

# Scale service (if supported)
docker compose up -d --scale backend=3

# Stop and remove everything
docker compose down

# Stop and remove volumes
docker compose down -v

# Cleanup
cd ..
```
</details>

---

## Advanced Level Exercises

### Exercise 11: Multi-Stage Build for Java Application
**Objective**: Build Java app with Maven multi-stage

**Tasks:**
1. Create Spring Boot application
2. Multi-stage Dockerfile
3. Optimize layer caching
4. Run application

<details>
<summary>Solution</summary>

```bash
mkdir java-app
cd java-app

# Create pom.xml
cat > pom.xml <<'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>1.0.0</version>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.12</version>
    </parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
EOF

# Create Java source
mkdir -p src/main/java/com/example/demo
cat > src/main/java/com/example/demo/DemoApplication.java <<'EOF'
package com.example.demo;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.*;

@SpringBootApplication
@RestController
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
    
    @GetMapping("/")
    public String hello() {
        return "Hello from Spring Boot in Docker!";
    }
}
EOF

# Create Dockerfile
cat > Dockerfile <<'EOF'
# Build stage
FROM maven:3.8-jdk-11 AS build
WORKDIR /app

# Copy pom.xml and download dependencies (cached layer)
COPY pom.xml .
RUN mvn dependency:go-offline

# Copy source and build
COPY src ./src
RUN mvn clean package -DskipTests

# Runtime stage
FROM openjdk:11-jre-slim
WORKDIR /app

# Create non-root user
RUN groupadd -r spring && useradd -r -g spring spring
USER spring:spring

# Copy JAR from build stage
COPY --from=build /app/target/*.jar app.jar

# Health check
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -f http://localhost:8080/ || exit 1

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
EOF

# Build image
docker build -t spring-boot-app:1.0 .

# Check image size
docker images spring-boot-app:1.0

# Run container
docker run -d -p 8080:8080 --name springapp spring-boot-app:1.0

# Wait for startup
sleep 10

# Test
curl http://localhost:8080

# View logs
docker logs springapp

# Health check
docker inspect --format='{{.State.Health.Status}}' springapp

# Cleanup
docker stop springapp
docker rm springapp
cd ..
```
</details>

---

### Exercise 12: Docker Security Hardening
**Objective**: Implement security best practices

**Tasks:**
1. Create secure Dockerfile
2. Run with security options
3. Scan for vulnerabilities
4. Implement least privilege

<details>
<summary>Solution</summary>

```bash
mkdir secure-app
cd secure-app

# Create secure Dockerfile
cat > Dockerfile <<'EOF'
# Use specific version, not latest
FROM node:18.17.0-alpine3.18

# Install security updates
RUN apk upgrade --no-cache

# Create non-root user
RUN addgroup -g 1001 -S appgroup && \
    adduser -u 1001 -S appuser -G appgroup

WORKDIR /app

# Copy package files
COPY --chown=appuser:appgroup package*.json ./

# Install dependencies
RUN npm ci --only=production && \
    npm cache clean --force

# Copy application
COPY --chown=appuser:appgroup . .

# Switch to non-root user
USER appuser

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD node healthcheck.js || exit 1

# Start application
CMD ["node", "server.js"]
EOF

# Create app files
cat > package.json <<'EOF'
{
  "name": "secure-app",
  "version": "1.0.0",
  "main": "server.js",
  "dependencies": {}
}
EOF

cat > server.js <<'EOF'
const http = require('http');
const server = http.createServer((req, res) => {
  res.end('Secure app running!\n');
});
server.listen(3000);
EOF

cat > healthcheck.js <<'EOF'
const http = require('http');
const options = { host: 'localhost', port: 3000, timeout: 2000 };
const request = http.request(options, (res) => {
  process.exit(res.statusCode == 200 ? 0 : 1);
});
request.on('error', () => process.exit(1));
request.end();
EOF

# Build image
docker build -t secure-app:1.0 .

# Scan for vulnerabilities
docker scout cves secure-app:1.0

# Or use Trivy
# trivy image secure-app:1.0

# Run with security options
docker run -d \
  --name secureapp \
  -p 3000:3000 \
  --read-only \
  --tmpfs /tmp \
  --security-opt=no-new-privileges \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  --memory="256m" \
  --cpus="0.5" \
  --pids-limit=100 \
  secure-app:1.0

# Verify security settings
docker inspect secureapp | grep -A 10 SecurityOpt
docker inspect secureapp | grep -A 5 CapDrop

# Test
curl http://localhost:3000

# Try to write (should fail - read-only)
docker exec secureapp touch /test.txt  # Error: read-only

# Can write to /tmp (tmpfs)
docker exec secureapp touch /tmp/test.txt  # Success

# Check user
docker exec secureapp whoami  # appuser

# Cleanup
docker stop secureapp
docker rm secureapp
cd ..
```

**Security Checklist:**
✅ Use specific image versions
✅ Run as non-root user
✅ Read-only root filesystem
✅ Drop all capabilities
✅ Resource limits
✅ Security options
✅ Regular vulnerability scans
</details>

---

### Exercise 13: Health Checks and Self-Healing
**Objective**: Implement container health monitoring

**Tasks:**
1. Create app with health endpoint
2. Configure Docker health check
3. Test failure scenarios
4. Auto-restart on failure

<details>
<summary>Solution</summary>

```bash
mkdir health-demo
cd health-demo

# Create Node.js app with health endpoint
cat > server.js <<'EOF'
const http = require('http');
let isHealthy = true;
let requestCount = 0;

const server = http.createServer((req, res) => {
  if (req.url === '/health') {
    if (isHealthy) {
      res.statusCode = 200;
      res.end('OK\n');
    } else {
      res.statusCode = 503;
      res.end('Unhealthy\n');
    }
  } else if (req.url === '/fail') {
    isHealthy = false;
    res.end('Health set to fail\n');
  } else if (req.url === '/recover') {
    isHealthy = true;
    res.end('Health recovered\n');
  } else {
    requestCount++;
    res.end(`Request ${requestCount}\n`);
  }
});

server.listen(3000, () => console.log('Server running'));
EOF

# Create Dockerfile with health check
cat > Dockerfile <<'EOF'
FROM node:18-alpine
WORKDIR /app
COPY server.js .
EXPOSE 3000

HEALTHCHECK --interval=10s --timeout=3s --start-period=5s --retries=3 \
  CMD wget -q --spider http://localhost:3000/health || exit 1

CMD ["node", "server.js"]
EOF

# Build
docker build -t health-app .

# Run with restart policy
docker run -d \
  -p 3000:3000 \
  --name healthapp \
  --restart=unless-stopped \
  health-app

# Monitor health status
watch -n 2 "docker ps | grep healthapp"

# In another terminal:
# Check health
curl http://localhost:3000/health  # OK

# Check container health
docker inspect --format='{{.State.Health.Status}}' healthapp

# Make app unhealthy
curl http://localhost:3000/fail

# Watch health checks fail (3 retries)
docker inspect healthapp | grep -A 20 Health

# Check status changes to unhealthy
docker ps | grep healthapp

# Container restarts automatically
docker logs healthapp

# Recover health
curl http://localhost:3000/recover

# Health recovers
docker inspect --format='{{.State.Health.Status}}' healthapp

# Cleanup
docker stop healthapp
docker rm healthapp
cd ..
```
</details>

---

### Exercise 14: Docker Registry - Private Image Repository
**Objective**: Set up and use private registry

**Tasks:**
1. Run local Docker registry
2. Tag and push images
3. Pull from private registry
4. Secure with authentication

<details>
<summary>Solution</summary>

```bash
# Run local registry
docker run -d \
  -p 5000:5000 \
  --name registry \
  -v registry_data:/var/lib/registry \
  registry:2

# Build sample image
mkdir myapp
echo 'FROM nginx:alpine' > myapp/Dockerfile
docker build -t myapp:1.0 myapp/

# Tag for local registry
docker tag myapp:1.0 localhost:5000/myapp:1.0

# Push to local registry
docker push localhost:5000/myapp:1.0

# Remove local image
docker rmi myapp:1.0 localhost:5000/myapp:1.0

# Pull from registry
docker pull localhost:5000/myapp:1.0

# List images in registry
curl http://localhost:5000/v2/_catalog

# View tags
curl http://localhost:5000/v2/myapp/tags/list

# Cleanup
docker stop registry
docker rm registry
docker volume rm registry_data
rm -rf myapp
```
</details>

---

### Exercise 15: Production Deployment with Docker Compose
**Objective**: Deploy production-ready stack

**Tasks:**
1. Create production compose file
2. Include monitoring
3. Configure logging
4. Implement backups

<details>
<summary>Solution</summary>

```bash
mkdir production-stack
cd production-stack

# Create production docker-compose.yml
cat > docker-compose.yml <<'EOF'
version: '3.8'

services:
  app:
    image: myapp:latest
    deploy:
      replicas: 2
      resources:
        limits:
          cpus: '1'
          memory: 512M
      restart_policy:
        condition: on-failure
        max_attempts: 3
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://db:5432/prod
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
    networks:
      - backend
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
      POSTGRES_DB: prod
    volumes:
      - db_data:/var/lib/postgresql/data
      - ./backups:/backups
    networks:
      - backend
    secrets:
      - db_password

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
    networks:
      - backend
    depends_on:
      - app

networks:
  backend:
    driver: bridge

volumes:
  db_data:
    driver: local

secrets:
  db_password:
    file: ./secrets/db_password.txt
EOF

# Create secrets
mkdir -p secrets
echo "strong-password-here" > secrets/db_password.txt

# Deploy
docker compose up -d

# Monitor
docker compose ps
docker compose logs -f

# Backup database
docker compose exec db pg_dump -U postgres prod > backups/backup.sql

# Cleanup
docker compose down -v
cd ..
```
</details>

---

## Tips for Practice

1. **Start Simple**: Master basics before advanced topics
2. **Read Documentation**: Official Docker docs are excellent
3. **Practice Regularly**: Build muscle memory
4. **Clean Up**: Remove unused containers/images
5. **Use `.dockerignore`**: Optimize build context
6. **Version Images**: Use specific tags
7. **Monitor Resources**: Watch disk space
8. **Security First**: Follow best practices
9. **Log Everything**: Use proper logging
10. **Test Locally**: Before deploying to production

---

## Next Steps

1. Complete all exercises in order
2. Practice with real applications
3. Study [Learning Roadmap](docker-learning-roadmap.md)
4. Review [Troubleshooting Guide](docker-troubleshooting-guide.md)
5. Prepare with [Interview Questions](docker-interview-questions.md)
6. Move to Kubernetes for orchestration

**Happy Dockering! 🐋**


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

