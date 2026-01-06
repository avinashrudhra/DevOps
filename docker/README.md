# Docker Learning Package 🐋

Complete learning resources for Docker - from basics to production-level containerization expertise.

---

## 📋 Overview

This comprehensive package covers **Docker** as part of the DevOps pipeline:
```
Git → Maven → SonarQube → [Jenkins] → Docker → Kubernetes
```

**Docker** is a platform for developing, shipping, and running applications in containers, providing consistency across environments and enabling microservices architecture.

---

## 📦 Package Contents

### 1. **Learning Roadmap** (`docker-learning-roadmap.md`)
- 12-week structured curriculum
- From Docker basics to production orchestration
- Covers images, containers, networking, volumes, security
- Week-by-week learning path with hands-on projects

### 2. **Quick Reference** (`docker-quick-reference.md`)
- Essential Docker commands
- Dockerfile best practices
- Docker Compose templates
- Networking and volume configurations
- Common patterns and solutions

### 3. **Hands-On Exercises** (`docker-hands-on-exercises.md`)
- 30+ practical exercises
- Container management
- Image creation and optimization
- Multi-container applications
- Docker Compose projects
- Production deployment patterns

### 4. **Troubleshooting Guide** (`docker-troubleshooting-guide.md`)
- 40+ common issues and solutions
- Container problems
- Image build failures
- Network connectivity issues
- Volume and storage problems
- Performance optimization

### 5. **Interview Questions** (`docker-interview-questions.md`)
- 70+ questions from basic to advanced
- For 7+ years experienced professionals
- Covers architecture, images, containers, networking, security
- Scenario-based questions
- Production best practices

---

## 🎯 Learning Objectives

After completing this package, you will:

✅ **Understand** Docker architecture and core concepts  
✅ **Build** optimized Docker images  
✅ **Manage** containers in production  
✅ **Configure** Docker networking and storage  
✅ **Implement** multi-container applications with Docker Compose  
✅ **Secure** Docker environments  
✅ **Optimize** image size and container performance  
✅ **Deploy** applications to production  
✅ **Integrate** with CI/CD pipelines  
✅ **Troubleshoot** common Docker issues

---

## 🚀 Quick Start

### Prerequisites
- Linux, macOS, or Windows 10/11
- Basic command-line knowledge
- Understanding of application deployment (basic)

### Install Docker

**Linux (Ubuntu/Debian):**
```bash
# Update package index
sudo apt-get update

# Install dependencies
sudo apt-get install ca-certificates curl gnupg lsb-release

# Add Docker's official GPG key
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Set up repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Verify installation
docker --version
docker compose version
```

**macOS:**
```bash
# Download Docker Desktop from:
# https://www.docker.com/products/docker-desktop

# Or using Homebrew
brew install --cask docker

# Start Docker Desktop
open -a Docker

# Verify
docker --version
```

**Windows:**
```powershell
# Download Docker Desktop from:
# https://www.docker.com/products/docker-desktop

# After installation, verify:
docker --version
```

### Run Your First Container
```bash
# Hello World
docker run hello-world

# Interactive Ubuntu container
docker run -it ubuntu bash

# Run Nginx web server
docker run -d -p 8080:80 nginx

# Access at http://localhost:8080
```

### Build Your First Image
```dockerfile
# Create Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

```bash
# Build image
docker build -t my-app:1.0 .

# Run container
docker run -d -p 3000:3000 my-app:1.0
```

---

## 📚 Recommended Reading Order

**Week 1-2: Docker Fundamentals**
1. Start with Learning Roadmap (Weeks 1-2)
2. Review Quick Reference (Basics section)
3. Complete Exercises 1-5 (Container basics)

**Week 3-4: Images & Dockerfiles**
1. Learning Roadmap (Weeks 3-4)
2. Quick Reference (Dockerfile section)
3. Complete Exercises 6-10 (Image creation)

**Week 5-6: Networking & Storage**
1. Learning Roadmap (Weeks 5-6)
2. Quick Reference (Networking & Volumes)
3. Complete Exercises 11-15 (Multi-container apps)

**Week 7-8: Docker Compose**
1. Learning Roadmap (Weeks 7-8)
2. Quick Reference (Docker Compose)
3. Complete Exercises 16-20 (Compose projects)

**Week 9-10: Security & Production**
1. Learning Roadmap (Weeks 9-10)
2. Review Security sections
3. Complete Exercises 21-25 (Security)

**Week 11-12: Advanced & Optimization**
1. Learning Roadmap (Weeks 11-12)
2. Review Troubleshooting Guide
3. Study Interview Questions
4. Complete Exercises 26-30 (Production)

---

## 🎓 Key Features of This Package

### Comprehensive Coverage
- **100+ pages** of detailed content
- **70+ interview questions** with detailed answers
- **30+ hands-on exercises** with solutions
- **40+ troubleshooting scenarios**

### Production-Ready
- Enterprise patterns and best practices
- Security hardening
- Performance optimization
- CI/CD integration
- Multi-stage builds
- Health checks and monitoring

### Integration Focus
- Maven/Java applications
- Node.js applications
- Spring Boot deployment
- Multi-container architectures
- Docker Compose workflows
- Kubernetes preparation

### Real-World Scenarios
- Microservices deployment
- Database containerization
- CI/CD pipeline integration
- Production deployment
- Scaling strategies

---

## 📊 Learning Statistics

- **Total Learning Time**: 12 weeks (10-15 hours/week)
- **Exercises**: 30+ hands-on labs
- **Interview Prep**: 70+ questions
- **Topics Covered**: 60+
- **Troubleshooting Scenarios**: 40+

---

## 🔗 Integration with DevOps Pipeline

### Previous in Pipeline
- **Git**: Version control and source management
- **Maven**: Build automation and dependency management
- **SonarQube**: Code quality and security analysis

### Docker's Role
- **Containerization**: Package applications with dependencies
- **Consistency**: Same environment dev to prod
- **Isolation**: Separate application processes
- **Portability**: Run anywhere Docker is installed
- **Microservices**: Enable microservice architecture

### Next in Pipeline
- **Jenkins**: CI/CD automation (builds Docker images)
- **Kubernetes**: Container orchestration at scale

### Integration Examples

**Build with Maven, Containerize with Docker:**
```dockerfile
FROM maven:3.8-jdk-11 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

FROM openjdk:11-jre-slim
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Docker Compose with SonarQube:**
```yaml
version: '3'
services:
  app:
    build: .
    ports:
      - "8080:8080"
  
  sonarqube:
    image: sonarqube:lts
    ports:
      - "9000:9000"
    environment:
      - SONAR_JDBC_URL=jdbc:postgresql://db:5432/sonar
    depends_on:
      - db
  
  db:
    image: postgres:13
    environment:
      - POSTGRES_USER=sonar
      - POSTGRES_PASSWORD=sonar
```

---

## 🛠️ Core Concepts Covered

### Docker Architecture
- **Docker Engine**: Client-server application
- **Docker Daemon**: Manages containers
- **Docker Client**: CLI to interact with daemon
- **Docker Registry**: Stores images (Docker Hub)
- **Images**: Read-only templates
- **Containers**: Running instances of images

### Container Lifecycle
```
docker create  → Created
docker start   → Running
docker pause   → Paused
docker stop    → Stopped
docker rm      → Removed
```

### Image Layers
```
Base Image (OS)
    ↓
Dependencies Layer
    ↓
Application Layer
    ↓
Configuration Layer
```

### Networking Modes
- **Bridge**: Default, isolated network
- **Host**: Use host's network
- **None**: No networking
- **Overlay**: Multi-host networking (Swarm)
- **Macvlan**: Assign MAC address

### Storage Options
- **Volumes**: Managed by Docker
- **Bind Mounts**: Host filesystem paths
- **tmpfs**: Temporary in-memory storage

---

## 📖 Docker Commands Reference

### Essential Commands
```bash
# Images
docker images                    # List images
docker pull nginx:latest         # Pull image
docker build -t myapp:1.0 .     # Build image
docker rmi image-id             # Remove image

# Containers
docker ps                        # Running containers
docker ps -a                     # All containers
docker run -d -p 80:80 nginx    # Run container
docker stop container-id        # Stop container
docker rm container-id          # Remove container

# Logs & Inspection
docker logs container-id        # View logs
docker logs -f container-id     # Follow logs
docker inspect container-id     # Detailed info
docker stats                    # Resource usage

# Execute Commands
docker exec -it container-id bash   # Interactive shell
docker exec container-id ls /app    # Run command

# Networks
docker network ls               # List networks
docker network create mynet     # Create network
docker network inspect mynet    # Inspect network

# Volumes
docker volume ls                # List volumes
docker volume create myvolume   # Create volume
docker volume inspect myvolume  # Inspect volume

# System
docker system df                # Disk usage
docker system prune             # Clean up
docker info                     # System info
```

---

## 💡 Best Practices Covered

### Image Building
✅ Use official base images  
✅ Minimize layers  
✅ Multi-stage builds  
✅ Leverage build cache  
✅ Use .dockerignore  
✅ Don't run as root  
✅ Use specific tags, not `latest`  
✅ Scan for vulnerabilities

### Container Management
✅ One process per container  
✅ Use health checks  
✅ Set resource limits  
✅ Use environment variables for config  
✅ Implement graceful shutdown  
✅ Store data in volumes  
✅ Use logging drivers  
✅ Monitor container metrics

### Security
✅ Use minimal base images  
✅ Scan images for vulnerabilities  
✅ Don't embed secrets in images  
✅ Use read-only root filesystem  
✅ Drop unnecessary capabilities  
✅ Use user namespaces  
✅ Implement network policies  
✅ Regular security updates

### Production Deployment
✅ Use orchestration (Kubernetes)  
✅ Implement auto-scaling  
✅ Configure health checks  
✅ Set up monitoring and logging  
✅ Use CI/CD for image builds  
✅ Implement blue-green deployment  
✅ Have rollback strategy  
✅ Document everything

---

## 🎯 Career Benefits

### Skills You'll Master
- Container technology and architecture
- Image creation and optimization
- Container orchestration basics
- Security best practices
- Production deployment
- CI/CD integration
- Troubleshooting and debugging

### Job Roles
- DevOps Engineer
- Container Platform Engineer
- Cloud Engineer
- Site Reliability Engineer (SRE)
- Platform Engineer
- Solutions Architect

---

## 🔧 Tools & Technologies

**Docker Ecosystem:**
- Docker Engine
- Docker Compose
- Docker Hub
- Docker Registry
- Docker Buildx
- Docker Scout (security scanning)

**Related Technologies:**
- containerd
- runc
- BuildKit
- Podman (Docker alternative)
- Kubernetes (orchestration)
- Docker Swarm (orchestration)

**CI/CD Integration:**
- Jenkins
- GitLab CI/CD
- GitHub Actions
- Azure DevOps
- CircleCI
- Travis CI

---

## 📝 Additional Resources

### Official Documentation
- [Docker Docs](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/)
- [Docker Blog](https://www.docker.com/blog/)

### Related Learning Packages
- [Git Package](../git/) - Version control
- [Maven Package](../maven/) - Build automation
- [SonarQube Package](../sonarqube/) - Code quality
- [Kubernetes Package](../kubernetes/) - Container orchestration

### Next Steps
After mastering Docker:
- Jenkins (CI/CD with Docker)
- Kubernetes (Container orchestration)
- Docker Swarm (Alternative orchestration)
- Service Mesh (Istio, Linkerd)

---

## 🆘 Getting Help

**Within This Package:**
1. Check Troubleshooting Guide for common issues
2. Review Quick Reference for commands
3. Revisit Learning Roadmap for concepts
4. Practice with Hands-On Exercises

**External Resources:**
- Docker Community Forums
- Stack Overflow [docker] tag
- Official documentation
- Docker Slack Community

---

## ✅ Learning Checklist

### Foundation
- [ ] Understand containers vs VMs
- [ ] Install Docker
- [ ] Run first container
- [ ] Build first image
- [ ] Understand Dockerfile

### Intermediate
- [ ] Create multi-stage builds
- [ ] Configure networking
- [ ] Manage volumes
- [ ] Use Docker Compose
- [ ] Implement health checks

### Advanced
- [ ] Optimize image size
- [ ] Implement security best practices
- [ ] Set up CI/CD with Docker
- [ ] Configure logging and monitoring
- [ ] Deploy to production

### Expert
- [ ] Create custom base images
- [ ] Implement advanced networking
- [ ] Master Docker security
- [ ] Optimize performance
- [ ] Prepare for Kubernetes

---

## 📈 Progress Tracking

Use the Learning Roadmap to track your weekly progress:
- Week 1-2: ⬜ Docker Fundamentals
- Week 3-4: ⬜ Images & Dockerfiles
- Week 5-6: ⬜ Networking & Storage
- Week 7-8: ⬜ Docker Compose
- Week 9-10: ⬜ Security & Best Practices
- Week 11-12: ⬜ Production & Advanced

---

## 🎊 Final Goal

By completing this package, you'll be proficient in:
- Docker architecture and core concepts
- Building optimized container images
- Managing containers in production
- Implementing security best practices
- Integrating Docker with CI/CD
- Deploying containerized applications
- Troubleshooting Docker issues
- Preparing for Kubernetes orchestration

**Ready to master Docker? Start with the [Learning Roadmap](docker-learning-roadmap.md)!**

---

## 📄 License & Usage

This learning package is created for educational purposes. Practice in safe environments before applying to production systems.

**Happy Learning! 🐋**

---

*Part of the DevOps Learning Series: Git → Maven → SonarQube → Docker → Kubernetes*


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

