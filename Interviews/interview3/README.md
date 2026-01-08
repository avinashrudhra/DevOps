# DevOps Interview Session 3
## Conversational Format - In-Depth Technical Discussion (3-7 Years)

---

### Question 1: Azure VNet Peering

**Interviewer:** Explain VNet peering and when you'd use it.

**Candidate:** VNet peering connects two Azure Virtual Networks, allowing resources to communicate using private IPs as if they're in the same network. There are two types - regional peering within the same region and global peering across regions.

**Interviewer:** What's the difference from VPN Gateway?

**Candidate:** Peering uses Azure backbone with higher bandwidth, lower latency, and no encryption overhead. VPN Gateway encrypts traffic and costs more but allows on-premises connectivity. For VNet-to-VNet, peering is faster and cheaper. We use peering for hub-spoke topology - shared services in hub VNet, applications in spoke VNets.

**Interviewer:** Any limitations?

**Candidate:** Address spaces can't overlap, no transitive routing by default - if VNet A peers with B and B peers with C, A can't reach C unless you use a gateway transit or NVA. Also, you can't peer more than 500 VNets to a single VNet.

---

### Question 2: Kubernetes RBAC Deep Dive

**Interviewer:** Walk me through implementing RBAC for a development team in Kubernetes.

**Candidate:** First, I create a namespace for their project, then define a Role with specific permissions - like getting, listing, and creating pods, deployments, and services in that namespace. Then I create a RoleBinding that links the Role to their Azure AD group.

**Interviewer:** Show me an example.

**Candidate:** 
```yaml
# Role - defines what can be done
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: dev-team-role
  namespace: development
rules:
- apiGroups: ["", "apps"]
  resources: ["pods", "deployments", "services", "configmaps"]
  verbs: ["get", "list", "create", "update", "delete"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get", "list"]

# RoleBinding - assigns role to users/groups
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-team-binding
  namespace: development
subjects:
- kind: Group
  name: "dev-team@company.com"
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: dev-team-role
  apiGroup: rbac.authorization.k8s.io
```

**Interviewer:** What if they need read-only access to production?

**Candidate:** I'd create a ClusterRole for read-only across namespaces and bind it specifically to production namespace using RoleBinding. ClusterRole is cluster-wide definition, but RoleBinding scopes it to one namespace.

---

### Question 3: PersistentVolumeClaims in Production

**Interviewer:** You need to deploy PostgreSQL in AKS with persistent storage. Walk me through it.

**Candidate:** I'd use a StatefulSet with volumeClaimTemplates. First, ensure there's a StorageClass - in AKS, managed-premium for SSD or azurefile for shared access. The PVC requests storage from this class, Kubernetes dynamically provisions an Azure Disk, and mounts it to the pod.

**Interviewer:** Show me the manifest.

**Candidate:**
```yaml
apiVersion: v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: disk.csi.azure.com
parameters:
  skuName: Premium_LRS
  kind: Managed
allowVolumeExpansion: true
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:14
        env:
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
        ports:
        - containerPort: 5432
        volumeMounts:
        - name: postgres-data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: postgres-data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: fast-ssd
      resources:
        requests:
          storage: 50Gi
```

**Interviewer:** What happens if the pod dies?

**Candidate:** StatefulSet reschedules it with the same name and reattaches the same PVC. Data persists because the Azure Disk remains. That's why we use StatefulSet for databases - stable network identity and persistent storage.

**Interviewer:** Can you expand the volume later?

**Candidate:** Yes, if `allowVolumeExpansion: true` in StorageClass. Edit the PVC to increase size, then restart the pod if the filesystem needs resizing. Azure Disk expands automatically.

---

### Question 4: Azure Subnets and NSG Design

**Interviewer:** Design a subnet structure for a three-tier application in Azure.

**Candidate:** I'd create a VNet with three subnets - frontend subnet for public-facing resources like Application Gateway, application subnet for AKS or App Services, and database subnet for managed databases. Each subnet has its own Network Security Group with specific rules.

**Interviewer:** What NSG rules would you configure?

**Candidate:** Frontend NSG allows 443 from internet, denies everything else. Application NSG allows traffic from frontend subnet on app ports, allows outbound to database subnet. Database NSG only allows 5432 or 1433 from application subnet, denies all internet access. Default deny-all rule at the bottom of each NSG.

**Interviewer:** How do you handle inter-subnet routing?

**Candidate:** By default, Azure routes traffic between subnets in the same VNet. For custom routing, I'd use Route Tables - for example, forcing all traffic through a firewall NVA. Also use service endpoints or private endpoints for accessing Azure PaaS services without internet exposure.

---

### Question 5: Docker Networking Configuration

**Interviewer:** You have a Docker Compose application where containers need to communicate. How do you configure networking?

**Candidate:**
```yaml
version: '3.8'
services:
  web:
    image: nginx
    networks:
      - frontend
      - backend
    ports:
      - "80:80"
  
  api:
    image: myapi:latest
    networks:
      - backend
    environment:
      DB_HOST: database
  
  database:
    image: postgres:14
    networks:
      - backend
    volumes:
      - db-data:/var/lib/postgresql/data

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true  # No external access

volumes:
  db-data:
```

**Interviewer:** Why separate networks?

**Candidate:** Defense in depth. Web server needs internet access, but database doesn't. Backend network is internal-only - even if web server is compromised, attacker can't directly reach database from outside. Web service acts as a gateway with limited API exposure.

---

### Question 6: Jenkins Pipeline Configuration

**Interviewer:** Configure a Jenkins pipeline for a microservice that builds, tests, scans with SonarQube, and deploys to AKS.

**Candidate:**
```groovy
pipeline {
    agent {
        label 'docker-agent'
    }
    
    environment {
        REGISTRY = 'myregistry.azurecr.io'
        IMAGE_NAME = 'myapp'
        IMAGE_TAG = "${BUILD_NUMBER}"
        SONAR_TOKEN = credentials('sonarqube-token')
        KUBECONFIG = credentials('aks-kubeconfig')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh 'git rev-parse HEAD > commit.txt'
            }
        }
        
        stage('Build & Test') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'npm install'
                        sh 'npm test'
                        junit 'test-results/*.xml'
                    }
                }
                stage('Security Scan') {
                    steps {
                        sh 'npm audit --audit-level=moderate'
                    }
                }
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'
                    withSonarQubeEnv('SonarQube') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                              -Dsonar.projectKey=myapp \
                              -Dsonar.sources=src \
                              -Dsonar.host.url=${SONAR_HOST_URL} \
                              -Dsonar.login=${SONAR_TOKEN}
                        """
                    }
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage('Docker Build & Push') {
            steps {
                script {
                    docker.withRegistry("https://${REGISTRY}", 'acr-credentials') {
                        def app = docker.build("${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}")
                        app.push()
                        app.push('latest')
                    }
                }
            }
        }
        
        stage('Deploy to AKS') {
            when {
                branch 'main'
            }
            steps {
                sh """
                    kubectl set image deployment/myapp \
                      myapp=${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} \
                      --record
                    kubectl rollout status deployment/myapp
                """
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            slackSend color: 'good', message: "Build ${BUILD_NUMBER} succeeded"
        }
        failure {
            slackSend color: 'danger', message: "Build ${BUILD_NUMBER} failed"
        }
    }
}
```

**Interviewer:** What if SonarQube fails?

**Candidate:** The quality gate stage has `abortPipeline: true`, so the pipeline stops and doesn't deploy. This enforces code quality - no bad code goes to production.

---

### Question 7: Terraform Azure Infrastructure

**Interviewer:** Write Terraform to create an AKS cluster with a separate node pool and integrate with Azure Container Registry.

**Candidate:**
```hcl
# Resource Group
resource "azurerm_resource_group" "aks" {
  name     = "aks-production-rg"
  location = "East US"
}

# VNet for AKS
resource "azurerm_virtual_network" "aks" {
  name                = "aks-vnet"
  address_space       = ["10.1.0.0/16"]
  location            = azurerm_resource_group.aks.location
  resource_group_name = azurerm_resource_group.aks.name
}

resource "azurerm_subnet" "aks" {
  name                 = "aks-subnet"
  resource_group_name  = azurerm_resource_group.aks.name
  virtual_network_name = azurerm_virtual_network.aks.name
  address_prefixes     = ["10.1.0.0/24"]
}

# Container Registry
resource "azurerm_container_registry" "acr" {
  name                = "myappsacr"
  resource_group_name = azurerm_resource_group.aks.name
  location            = azurerm_resource_group.aks.location
  sku                 = "Standard"
  admin_enabled       = false
}

# AKS Cluster
resource "azurerm_kubernetes_cluster" "aks" {
  name                = "myapp-aks"
  location            = azurerm_resource_group.aks.location
  resource_group_name = azurerm_resource_group.aks.name
  dns_prefix          = "myapp"
  kubernetes_version  = "1.27.7"

  default_node_pool {
    name                = "system"
    node_count          = 3
    vm_size             = "Standard_D2s_v3"
    vnet_subnet_id      = azurerm_subnet.aks.id
    enable_auto_scaling = true
    min_count           = 2
    max_count           = 5
    type                = "VirtualMachineScaleSets"
    
    tags = {
      Environment = "Production"
      Type        = "System"
    }
  }

  identity {
    type = "SystemAssigned"
  }

  network_profile {
    network_plugin    = "azure"
    network_policy    = "azure"
    load_balancer_sku = "standard"
  }

  tags = {
    Environment = "Production"
  }
}

# Additional node pool for applications
resource "azurerm_kubernetes_cluster_node_pool" "apps" {
  name                  = "apps"
  kubernetes_cluster_id = azurerm_kubernetes_cluster.aks.id
  vm_size               = "Standard_D4s_v3"
  node_count            = 3
  enable_auto_scaling   = true
  min_count             = 2
  max_count             = 10
  
  node_labels = {
    "workload" = "applications"
  }

  tags = {
    Environment = "Production"
    Type        = "Application"
  }
}

# Role assignment for AKS to pull from ACR
resource "azurerm_role_assignment" "aks_acr" {
  principal_id                     = azurerm_kubernetes_cluster.aks.kubelet_identity[0].object_id
  role_definition_name             = "AcrPull"
  scope                            = azurerm_container_registry.acr.id
  skip_service_principal_aad_check = true
}

# Outputs
output "kube_config" {
  value     = azurerm_kubernetes_cluster.aks.kube_config_raw
  sensitive = true
}

output "acr_login_server" {
  value = azurerm_container_registry.acr.login_server
}
```

**Interviewer:** Why separate node pools?

**Candidate:** System node pool runs Kubernetes system pods like CoreDNS and metrics-server - you want these stable and not affected by application load. Application node pool can scale independently based on application demand. Also allows different VM sizes - smaller for system, larger for compute-intensive apps.

---

### Question 8: Azure Load Balancer vs Application Gateway

**Interviewer:** Explain the difference and when to use each.

**Candidate:** Load Balancer operates at Layer 4 (transport layer) - it distributes TCP/UDP traffic based on IP and port. Fast, simple, good for non-HTTP workloads. Application Gateway is Layer 7 (application layer) - understands HTTP/HTTPS, does URL path-based routing, SSL termination, Web Application Firewall.

**Interviewer:** Give me a use case for each.

**Candidate:** Load Balancer for distributing traffic to AKS node pools or load balancing database connections. Application Gateway for web applications where you need path-based routing - like sending `/api` to backend pods and `/static` to storage, plus WAF protection against SQL injection and XSS attacks. Application Gateway is more expensive but richer features for web apps.

---

### Question 9: Kubernetes Network Policies Implementation

**Interviewer:** Implement a network policy that allows only frontend pods to access backend on port 8080.

**Candidate:**
```yaml
# Backend network policy - deny all by default
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      tier: backend
  policyTypes:
  - Ingress
  - Egress
  ingress:
  # Allow from frontend pods
  - from:
    - podSelector:
        matchLabels:
          tier: frontend
    ports:
    - protocol: TCP
      port: 8080
  # Allow from monitoring (Prometheus)
  - from:
    - namespaceSelector:
        matchLabels:
          name: monitoring
    ports:
    - protocol: TCP
      port: 9090
  egress:
  # Allow DNS
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: UDP
      port: 53
  # Allow to database
  - to:
    - podSelector:
        matchLabels:
          tier: database
    ports:
    - protocol: TCP
      port: 5432
```

**Interviewer:** What happens to existing connections when you apply this?

**Candidate:** Existing connections continue, but new connections are blocked unless they match the policy. That's why we test network policies in dev first - applying in production could break things instantly.

---

### Question 10: Azure DevOps Service Connections

**Interviewer:** Configure a service connection for AKS deployment with proper security.

**Candidate:** Create a service principal with least privilege - only Kubernetes Cluster User role on the specific AKS cluster, not Contributor on the subscription. In Azure DevOps, create Azure Resource Manager service connection using this SP, scope it to the resource group, and enable workload identity federation for better security than secrets.

**Interviewer:** What's workload identity federation?

**Candidate:** Instead of long-lived secrets, Azure DevOps gets short-lived tokens from Azure AD during pipeline execution. No secrets to manage or rotate, more secure. The service connection trusts Azure DevOps' OIDC issuer, and tokens are valid only during pipeline run.

---

### Question 11: Helm Chart Configuration Management

**Interviewer:** You have an application deployed across dev, staging, and prod. How do you manage configurations with Helm?

**Candidate:**
```yaml
# values.yaml - base defaults
replicaCount: 2
image:
  repository: myapp
  tag: "1.0.0"
resources:
  requests:
    memory: "256Mi"
    cpu: "250m"
  limits:
    memory: "512Mi"
    cpu: "500m"
database:
  host: "db.default.svc.cluster.local"
ingress:
  enabled: true
  host: "myapp.example.com"

# values-dev.yaml - development overrides
replicaCount: 1
image:
  tag: "latest"
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
ingress:
  host: "myapp-dev.example.com"

# values-prod.yaml - production overrides
replicaCount: 5
image:
  tag: "1.0.0"  # Specific version, not latest
resources:
  requests:
    memory: "512Mi"
    cpu: "500m"
  limits:
    memory: "1Gi"
    cpu: "1000m"
database:
  host: "prod-db.database.azure.com"
ingress:
  host: "myapp.com"
  tls: true
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"

# Deploy commands
# helm install myapp ./mychart -f values-dev.yaml
# helm install myapp ./mychart -f values-prod.yaml
```

**Interviewer:** How do you handle secrets?

**Candidate:** Never in values files. Either use Helm secrets plugin with encrypted secrets, or better - use Azure Key Vault CSI driver to mount secrets directly from Key Vault. The Helm chart references the secret mount path, not the actual secret value.

---

### Question 12: Docker Multi-Stage Build Best Practices

**Interviewer:** Optimize this Dockerfile for a Java application.

**Candidate:**
```dockerfile
# Bad approach - single stage
FROM openjdk:11
COPY . /app
WORKDIR /app
RUN ./mvnw clean package
CMD ["java", "-jar", "target/app.jar"]
# Result: 800MB image with Maven, source code, etc.

# Optimized multi-stage build
# Stage 1: Build
FROM maven:3.8-openjdk-11 AS builder
WORKDIR /build
# Copy only pom.xml first for dependency caching
COPY pom.xml .
RUN mvn dependency:go-offline
# Now copy source and build
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2: Runtime
FROM openjdk:11-jre-slim
WORKDIR /app
# Copy only the jar from builder
COPY --from=builder /build/target/app.jar app.jar
# Create non-root user
RUN useradd -m appuser
USER appuser
EXPOSE 8080
ENTRYPOINT ["java", \
            "-XX:+UseContainerSupport", \
            "-XX:MaxRAMPercentage=75.0", \
            "-jar", "app.jar"]
# Result: 250MB image, 68% smaller, more secure
```

**Interviewer:** Why separate dependency download?

**Candidate:** Docker caches each layer. If only source code changes, the dependency layer stays cached and doesn't redownload. Saves time on every build. If pom.xml changes, everything rebuilds - which is correct because dependencies changed.

---

### Question 13: Azure Private Endpoints

**Interviewer:** Explain private endpoints and implement one for Azure SQL.

**Candidate:** Private endpoint gives a PaaS service like SQL Database a private IP in your VNet, so traffic never goes over internet. It's more secure than service endpoints because it's a dedicated IP, not just routing change.

**Interviewer:** Configure it in Terraform.

**Candidate:**
```hcl
resource "azurerm_private_endpoint" "sql" {
  name                = "sql-private-endpoint"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  subnet_id           = azurerm_subnet.private_endpoints.id

  private_service_connection {
    name                           = "sql-connection"
    private_connection_resource_id = azurerm_mssql_server.main.id
    is_manual_connection           = false
    subresource_names              = ["sqlServer"]
  }

  private_dns_zone_group {
    name                 = "sql-dns-zone-group"
    private_dns_zone_ids = [azurerm_private_dns_zone.sql.id]
  }
}

resource "azurerm_private_dns_zone" "sql" {
  name                = "privatelink.database.windows.net"
  resource_group_name = azurerm_resource_group.main.name
}

resource "azurerm_private_dns_zone_virtual_network_link" "sql" {
  name                  = "sql-dns-link"
  resource_group_name   = azurerm_resource_group.main.name
  private_dns_zone_name = azurerm_private_dns_zone.sql.name
  virtual_network_id    = azurerm_virtual_network.main.id
}
```

**Interviewer:** Why the DNS zone?

**Candidate:** Without it, `myserver.database.windows.net` resolves to public IP. The private DNS zone makes it resolve to the private endpoint IP, so applications connect privately without code changes.

---

### Question 14: Kubernetes Ingress with cert-manager

**Interviewer:** Set up Ingress with automatic SSL certificate management.

**Candidate:**
```yaml
# Install cert-manager first
# kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.yaml

# ClusterIssuer for Let's Encrypt
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx

---
# Ingress with TLS
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - myapp.example.com
    secretName: myapp-tls  # cert-manager creates this
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port:
              number: 80
```

**Interviewer:** How does cert-manager work?

**Candidate:** It watches Ingress resources with its annotation, sees a certificate is needed, creates a temporary pod to respond to Let's Encrypt's HTTP challenge, gets the cert, stores it in the specified secret, and renews it automatically before expiration. Fully automated SSL.

---

### Question 15: Azure Application Insights Integration

**Interviewer:** How do you monitor an application deployed in AKS with Application Insights?

**Candidate:** Install the Application Insights agent as a DaemonSet or use auto-instrumentation. The agent collects telemetry - requests, dependencies, exceptions, custom metrics. For Node.js or .NET, we add the SDK to the application code. Configure with instrumentation key or connection string from environment variable.

**Interviewer:** What about distributed tracing?

**Candidate:** Application Insights automatically correlates requests across microservices using W3C trace context headers. When frontend calls backend, we see the entire transaction flow, timing for each hop, and where failures occurred. Essential for debugging in microservice architectures.

---

### Question 16: Kubernetes StatefulSet vs Deployment

**Interviewer:** When do you use StatefulSet instead of Deployment?

**Candidate:** StatefulSet for applications requiring stable network identity, ordered deployment/scaling, or persistent storage tied to pod identity. Examples: databases like MySQL, Cassandra; message queues like Kafka; caching layers like Redis cluster.

**Interviewer:** What makes the identity stable?

**Candidate:** Pods get predictable names - `postgres-0`, `postgres-1`, `postgres-2`. If `postgres-1` dies, it's recreated with the same name and reconnects to the same PVC. Each has a stable DNS entry like `postgres-1.postgres.default.svc.cluster.local`. Deployment pods have random names and IPs - fine for stateless apps.

---

### Question 17: Azure DevOps Pipeline YAML Templates

**Interviewer:** Create reusable pipeline templates for your organization.

**Candidate:**
```yaml
# templates/docker-build.yml
parameters:
- name: imageName
  type: string
- name: dockerfilePath
  type: string
  default: 'Dockerfile'
- name: buildContext
  type: string
  default: '.'

steps:
- task: Docker@2
  displayName: 'Build ${{ parameters.imageName }}'
  inputs:
    command: build
    repository: ${{ parameters.imageName }}
    dockerfile: ${{ parameters.dockerfilePath }}
    buildContext: ${{ parameters.buildContext }}
    tags: |
      $(Build.BuildId)
      latest

- task: Docker@2
  displayName: 'Push ${{ parameters.imageName }}'
  inputs:
    command: push
    repository: ${{ parameters.imageName }}
    tags: |
      $(Build.BuildId)
      latest

---
# templates/deploy-to-aks.yml
parameters:
- name: environment
  type: string
- name: namespace
  type: string
- name: manifestPath
  type: string

steps:
- task: KubernetesManifest@0
  displayName: 'Deploy to ${{ parameters.environment }}'
  inputs:
    action: deploy
    namespace: ${{ parameters.namespace }}
    manifests: ${{ parameters.manifestPath }}

- script: |
    kubectl rollout status deployment/$(appName) -n ${{ parameters.namespace }}
  displayName: 'Wait for rollout'

---
# azure-pipelines.yml - consuming templates
trigger:
- main

stages:
- stage: Build
  jobs:
  - job: BuildApp
    steps:
    - template: templates/docker-build.yml
      parameters:
        imageName: 'myapp'
        dockerfilePath: 'app/Dockerfile'

- stage: DeployDev
  dependsOn: Build
  jobs:
  - deployment: DeployDevJob
    environment: 'dev'
    strategy:
      runOnce:
        deploy:
          steps:
          - template: templates/deploy-to-aks.yml
            parameters:
              environment: 'dev'
              namespace: 'development'
              manifestPath: 'k8s/*.yaml'
```

**Interviewer:** Benefits of templates?

**Candidate:** Consistency across teams - everyone uses the same build and deploy logic. One place to update security scanning or compliance checks. New projects just reference templates instead of copy-pasting 100 lines of YAML.

---

### Question 18: Git Workflow for Hotfixes

**Interviewer:** Production is down. Walk me through your hotfix process.

**Candidate:** Create hotfix branch from `main` - `git checkout -b hotfix/critical-bug main`. Fix the issue, commit, push. Create PR to `main` with expedited review. Once merged and deployed, merge `main` back to `develop` to keep them in sync. Tag the release `v1.2.1` for tracking.

**Interviewer:** What if develop has unreleased features?

**Candidate:** Cherry-pick only the hotfix commit to develop - `git checkout develop && git cherry-pick <hotfix-commit>`. This pulls just the fix without bringing production changes into develop. Alternatively, merge main into develop and resolve conflicts, keeping develop's new features.

---

### Question 19: Azure Key Vault CSI Driver

**Interviewer:** Implement secure secret management in AKS using Key Vault.

**Candidate:**
```yaml
# Install CSI driver
helm repo add csi-secrets-store https://kubernetes-sigs.github.io/secrets-store-csi-driver/charts
helm install csi-secrets-store csi-secrets-store/secrets-store-csi-driver --namespace kube-system

# Install Azure provider
kubectl apply -f https://raw.githubusercontent.com/Azure/secrets-store-csi-driver-provider-azure/master/deployment/provider-azure-installer.yaml

# SecretProviderClass
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: azure-keyvault
  namespace: production
spec:
  provider: azure
  parameters:
    usePodIdentity: "false"
    useVMManagedIdentity: "true"
    userAssignedIdentityID: "<managed-identity-client-id>"
    keyvaultName: "myapp-keyvault"
    tenantId: "<tenant-id>"
    objects: |
      array:
        - |
          objectName: database-password
          objectType: secret
        - |
          objectName: api-key
          objectType: secret

---
# Pod using secrets
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  namespace: production
spec:
  containers:
  - name: app
    image: myapp:1.0
    volumeMounts:
    - name: secrets-store
      mountPath: "/mnt/secrets"
      readOnly: true
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: keyvault-secrets
          key: database-password
  volumes:
  - name: secrets-store
    csi:
      driver: secrets-store.csi.k8s.io
      readOnly: true
      volumeAttributes:
        secretProviderClass: "azure-keyvault"
```

**Interviewer:** How does authentication work?

**Candidate:** AKS kubelet has a managed identity with Key Vault Reader permissions. When a pod starts, CSI driver uses that identity to fetch secrets from Key Vault and mount them. Secrets are pulled at runtime, never stored in cluster, and automatically updated when rotated.

---

### Question 20: Terraform Remote State Configuration

**Interviewer:** Configure Terraform with Azure Storage backend for state management.

**Candidate:**
```hcl
# backend.tf
terraform {
  required_version = ">= 1.5.0"
  
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
  
  backend "azurerm" {
    resource_group_name  = "terraform-state-rg"
    storage_account_name = "tfstatestorage123"
    container_name       = "tfstate"
    key                  = "production.terraform.tfstate"
    use_azuread_auth     = true  # Use Azure AD, not access keys
  }
}

provider "azurerm" {
  features {}
}
```

**Interviewer:** How do you bootstrap the backend storage?

**Candidate:** Chicken-and-egg problem - can't use Terraform to create the storage if state needs that storage. I create it manually first or use a separate bootstrap Terraform with local state:

```bash
# Create state storage manually
az group create --name terraform-state-rg --location eastus
az storage account create \
  --name tfstatestorage123 \
  --resource-group terraform-state-rg \
  --sku Standard_LRS \
  --encryption-services blob
az storage container create \
  --name tfstate \
  --account-name tfstatestorage123

# Enable versioning
az storage account blob-service-properties update \
  --account-name tfstatestorage123 \
  --enable-versioning true

# Then run terraform init
terraform init
```

---

### Question 21: Kubernetes ConfigMap Mounting Strategies

**Interviewer:** What are the different ways to use ConfigMaps?

**Candidate:** Three ways - environment variables, volume mounts, or command arguments. Environment variables for simple key-value configs, volumes when you need file-based config like nginx.conf, and command args for overriding defaults.

**Interviewer:** Show me volume mount example.

**Candidate:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  app.properties: |
    server.port=8080
    db.maxConnections=100
  log4j.properties: |
    log4j.rootLogger=INFO, stdout

---
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: app
    image: myapp:1.0
    volumeMounts:
    - name: config
      mountPath: /config
      readOnly: true
    # Config files appear as:
    # /config/app.properties
    # /config/log4j.properties
  volumes:
  - name: config
    configMap:
      name: app-config
```

**Interviewer:** Do changes update automatically?

**Candidate:** Yes for volume mounts - kubelet syncs changes periodically, but the app needs to reload the file. Environment variables don't update - you'd need to restart the pod. That's why volume mounts are better for configs that might change.

---

### Question 22: Azure Monitor and Log Analytics

**Interviewer:** Set up monitoring for AKS cluster with alerts.

**Candidate:** Enable Container Insights on AKS cluster - it sends logs and metrics to Log Analytics workspace. Then create alert rules based on KQL queries.

**Interviewer:** Give me an alert example.

**Candidate:**
```kusto
// Alert when pod restart count > 5 in 10 minutes
KubePodInventory
| where TimeGenerated > ago(10m)
| where Namespace == "production"
| where RestartCount > 5
| project TimeGenerated, ClusterName, Namespace, Name, RestartCount

// Alert when CPU exceeds 80%
Perf
| where TimeGenerated > ago(5m)
| where ObjectName == "K8SContainer"
| where CounterName == "cpuUsageNanoCores"
| summarize AvgCPU = avg(CounterValue) by bin(TimeGenerated, 1m), InstanceName
| where AvgCPU > 80
```

In Azure Portal, create Alert Rule using these queries, set threshold, configure action group to send email or Slack notification.

---

### Question 23: Docker Compose for Local Development

**Interviewer:** Set up a local development environment with Docker Compose for a full-stack application.

**Candidate:**
```yaml
version: '3.8'

services:
  database:
    image: postgres:14
    environment:
      POSTGRES_USER: devuser
      POSTGRES_PASSWORD: devpass
      POSTGRES_DB: myapp_dev
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./init-db.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "devuser"]
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.dev
    volumes:
      - ./backend:/app  # Mount code for hot reload
      - /app/node_modules  # Don't override node_modules
    environment:
      NODE_ENV: development
      DATABASE_URL: postgresql://devuser:devpass@database:5432/myapp_dev
      REDIS_URL: redis://redis:6379
    ports:
      - "3000:3000"
    depends_on:
      database:
        condition: service_healthy
      redis:
        condition: service_started
    command: npm run dev  # Hot reload

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.dev
    volumes:
      - ./frontend:/app
      - /app/node_modules
    environment:
      REACT_APP_API_URL: http://localhost:3000
    ports:
      - "8080:8080"
    depends_on:
      - backend

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  postgres-data:

networks:
  default:
    name: myapp-dev-network
```

**Interviewer:** How do developers use this?

**Candidate:** `docker-compose up` starts everything. Code changes hot-reload without rebuilding. Database initializes with schema. When done, `docker-compose down -v` removes everything. Clean, reproducible dev environment - no "works on my machine" issues.

---

### Question 24: Kubernetes Jobs and CronJobs

**Interviewer:** You need to run a database backup daily. How do you implement it?

**Candidate:**
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: database-backup
  namespace: production
spec:
  schedule: "0 2 * * *"  # 2 AM daily
  concurrencyPolicy: Forbid  # Don't start if previous is running
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 3
  jobTemplate:
    spec:
      backoffLimit: 2  # Retry twice
      template:
        spec:
          restartPolicy: OnFailure
          serviceAccountName: backup-sa
          containers:
          - name: backup
            image: postgres:14
            command:
            - /bin/sh
            - -c
            - |
              TIMESTAMP=$(date +%Y%m%d_%H%M%S)
              BACKUP_FILE="/backup/db_backup_${TIMESTAMP}.sql"
              pg_dump -h $DB_HOST -U $DB_USER $DB_NAME > $BACKUP_FILE
              gzip $BACKUP_FILE
              # Upload to Azure Storage
              az storage blob upload \
                --account-name $STORAGE_ACCOUNT \
                --container-name backups \
                --name db_backup_${TIMESTAMP}.sql.gz \
                --file ${BACKUP_FILE}.gz
              # Cleanup local file
              rm ${BACKUP_FILE}.gz
            env:
            - name: DB_HOST
              value: "postgres.production.svc.cluster.local"
            - name: DB_USER
              valueFrom:
                secretKeyRef:
                  name: postgres-creds
                  key: username
            - name: PGPASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-creds
                  key: password
            - name: DB_NAME
              value: "production"
            - name: STORAGE_ACCOUNT
              value: "mybackupstorage"
            volumeMounts:
            - name: backup-volume
              mountPath: /backup
          volumes:
          - name: backup-volume
            emptyDir: {}
```

**Interviewer:** How do you monitor if backups fail?

**Candidate:** Create an alert in Azure Monitor watching for CronJob failures:
```kusto
KubeEvents
| where ObjectKind == "CronJob"
| where Name == "database-backup"
| where Reason == "BackoffLimitExceeded" or Reason == "DeadlineExceeded"
```
Also, we have a separate job that verifies the latest backup exists in storage and sends alert if it's older than 25 hours.

---

### Question 25: Azure Application Gateway with WAF

**Interviewer:** Configure Application Gateway with Web Application Firewall for AKS.

**Candidate:** Application Gateway Ingress Controller (AGIC) integrates App Gateway with AKS. WAF protects against OWASP Top 10 attacks - SQL injection, XSS, etc.

```yaml
# Ingress using Application Gateway
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway
    appgw.ingress.kubernetes.io/ssl-redirect: "true"
    appgw.ingress.kubernetes.io/waf-policy-for-path: "/subscriptions/.../providers/Microsoft.Network/ApplicationGatewayWebApplicationFirewallPolicies/myWAFPolicy"
spec:
  tls:
  - hosts:
    - myapp.com
    secretName: myapp-tls
  rules:
  - host: myapp.com
    http:
      paths:
      - path: /api/*
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 8080
      - path: /*
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
```

**Interviewer:** What WAF modes are there?

**Candidate:** Detection mode logs threats but doesn't block - use this initially to tune rules and avoid false positives. Prevention mode actively blocks malicious requests. Start with detection, review logs, add exclusions for false positives, then switch to prevention.

---

### Question 26: Terraform Modules Best Practices

**Interviewer:** Structure a Terraform project with modules for reusability.

**Candidate:**
```
terraform/
├── modules/
│   ├── aks/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── README.md
│   ├── networking/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   └── storage/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── terraform.tfvars
│   │   └── backend.tf
│   ├── staging/
│   └── production/
└── shared/
    └── main.tf

# environments/production/main.tf
module "networking" {
  source = "../../modules/networking"
  
  vnet_name          = "prod-vnet"
  vnet_address_space = ["10.0.0.0/16"]
  subnets = {
    aks = {
      address_prefix = "10.0.1.0/24"
    }
    database = {
      address_prefix = "10.0.2.0/24"
    }
  }
  location = var.location
  environment = "production"
}

module "aks" {
  source = "../../modules/aks"
  
  cluster_name    = "prod-aks"
  subnet_id       = module.networking.subnet_ids["aks"]
  node_count      = 5
  vm_size         = "Standard_D4s_v3"
  location        = var.location
}

output "aks_fqdn" {
  value = module.aks.fqdn
}
```

**Interviewer:** Module versioning strategy?

**Candidate:** Tag modules in Git - `v1.0.0`, `v1.1.0`. Reference specific versions in module source - `source = "git::https://github.com/org/modules.git//aks?ref=v1.0.0"`. This prevents breaking changes from affecting existing environments. Test new module versions in dev first.

---

### Question 27: Azure DevOps Multi-Stage Deployment

**Interviewer:** Implement a pipeline that deploys to dev automatically, staging after tests, and prod with approval.

**Candidate:**
```yaml
trigger:
- main

variables:
  imageRepository: 'myapp'
  imageTag: '$(Build.BuildId)'

stages:
- stage: Build
  jobs:
  - job: BuildJob
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - task: Docker@2
      inputs:
        command: buildAndPush
        repository: $(imageRepository)
        tags: $(imageTag)

- stage: DeployDev
  dependsOn: Build
  condition: succeeded()
  jobs:
  - deployment: DevDeploy
    environment: 'development'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: Kubernetes@1
            inputs:
              command: 'set'
              arguments: 'image deployment/myapp myapp=$(imageRepository):$(imageTag)'
              namespace: 'development'

- stage: IntegrationTests
  dependsOn: DeployDev
  jobs:
  - job: RunTests
    steps:
    - script: |
        npm install
        npm run test:integration
      displayName: 'Integration Tests'

- stage: DeployStaging
  dependsOn: IntegrationTests
  condition: succeeded()
  jobs:
  - deployment: StagingDeploy
    environment: 'staging'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: Kubernetes@1
            inputs:
              command: 'set'
              arguments: 'image deployment/myapp myapp=$(imageRepository):$(imageTag)'
              namespace: 'staging'

- stage: ApprovalGate
  dependsOn: DeployStaging
  jobs:
  - job: WaitForApproval
    pool: server
    steps:
    - task: ManualValidation@0
      timeoutInMinutes: 1440
      inputs:
        notifyUsers: 'ops-team@example.com'
        instructions: 'Review staging and approve for production'

- stage: DeployProd
  dependsOn: ApprovalGate
  jobs:
  - deployment: ProdDeploy
    environment: 'production'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: Kubernetes@1
            inputs:
              command: 'set'
              arguments: 'image deployment/myapp myapp=$(imageRepository):$(imageTag)'
              namespace: 'production'
```

**Interviewer:** What if staging tests fail?

**Candidate:** The pipeline stops at that stage. Production never gets deployed. We get notified, fix the issue, and re-run the pipeline. Gates ensure bad code never reaches production.

---

### Question 28: Prometheus and Grafana Setup

**Interviewer:** Configure Prometheus to scrape metrics from your applications.

**Candidate:**
```yaml
# Application with metrics endpoint
apiVersion: v1
kind: Service
metadata:
  name: myapp
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "9090"
    prometheus.io/path: "/metrics"
spec:
  selector:
    app: myapp
  ports:
  - name: http
    port: 80
  - name: metrics
    port: 9090

---
# ServiceMonitor for Prometheus Operator
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: myapp-monitor
  labels:
    release: prometheus
spec:
  selector:
    matchLabels:
      app: myapp
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics

---
# PrometheusRule for alerts
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: myapp-alerts
spec:
  groups:
  - name: myapp
    interval: 30s
    rules:
    - alert: HighErrorRate
      expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "High error rate on {{ $labels.instance }}"
        description: "Error rate is {{ $value }} req/sec"
    
    - alert: HighMemoryUsage
      expr: container_memory_usage_bytes{pod=~"myapp-.*"} / container_spec_memory_limit_bytes > 0.9
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "High memory usage on {{ $labels.pod }}"
```

**Interviewer:** How do you visualize this in Grafana?

**Candidate:** Add Prometheus as a data source in Grafana, then create dashboards with panels querying metrics - `rate(http_requests_total[5m])` for request rate, `histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))` for 95th percentile latency. Import community dashboards like Kubernetes cluster monitoring for quick start.

---

### Question 29: Git Submodules and Monorepo

**Interviewer:** Your organization has shared libraries. How do you manage them across projects?

**Candidate:** Two approaches - Git submodules or monorepo. Submodules keep each project separate but reference shared code. Monorepo puts everything in one repo with shared tools and dependencies.

**Interviewer:** Which do you prefer?

**Candidate:** Monorepo for better developer experience - one `git clone`, atomic changes across projects, easier refactoring. But needs good tooling like Nx or Turborepo for selective builds. Submodules if teams are independent and need strict separation, but they're harder to manage - updating submodules is confusing for developers.

---

### Question 30: Azure Firewall with Force Tunneling

**Interviewer:** Explain Azure Firewall and force tunneling concept.

**Candidate:** Azure Firewall is a managed firewall service that filters outbound traffic from VNet. Force tunneling routes all internet-bound traffic through firewall for inspection. You create a route table that directs 0.0.0.0/0 to the firewall, associate it with subnets. Firewall applies FQDN rules - like allowing only `*.microsoft.com`, blocking everything else.

**Interviewer:** Why use this for AKS?

**Candidate:** Security compliance - inspect all egress traffic, prevent data exfiltration, control which external services pods can access. For example, pods can pull images from approved registries only, can't reach malicious sites even if compromised. In Terraform:

```hcl
resource "azurerm_route_table" "aks" {
  name                = "aks-route-table"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  route {
    name                   = "to-firewall"
    address_prefix         = "0.0.0.0/0"
    next_hop_type          = "VirtualAppliance"
    next_hop_in_ip_address = azurerm_firewall.main.ip_configuration[0].private_ip_address
  }
}

resource "azurerm_subnet_route_table_association" "aks" {
  subnet_id      = azurerm_subnet.aks.id
  route_table_id = azurerm_route_table.aks.id
}
```

---

### Question 31: Kubernetes ResourceQuotas and LimitRanges

**Interviewer:** Prevent teams from overconsuming cluster resources.

**Candidate:**
```yaml
# ResourceQuota - namespace-wide limits
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-quota
  namespace: development
spec:
  hard:
    requests.cpu: "20"
    requests.memory: "40Gi"
    limits.cpu: "40"
    limits.memory: "80Gi"
    persistentvolumeclaims: "10"
    services.loadbalancers: "2"
    pods: "50"

---
# LimitRange - default per-container limits
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: development
spec:
  limits:
  - default:  # Default limits if not specified
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:  # Default requests if not specified
      cpu: "200m"
      memory: "256Mi"
    max:  # Maximum allowed
      cpu: "2"
      memory: "4Gi"
    min:  # Minimum required
      cpu: "100m"
      memory: "128Mi"
    type: Container
```

**Interviewer:** What happens if they exceed quota?

**Candidate:** New pod creation fails with error "exceeded quota". They need to either scale down existing pods or request quota increase. This prevents one team from starving others and controls cloud costs.

---

### Question 32: Azure Policy for Governance

**Interviewer:** Enforce tagging and allowed VM sizes across subscriptions.

**Candidate:** Azure Policy defines rules - like "all resources must have CostCenter tag" or "only Standard_D series VMs allowed". Assign policies at subscription or resource group scope. Non-compliant resources are flagged or blocked.

**Interviewer:** Example policy?

**Candidate:**
```json
{
  "if": {
    "allOf": [
      {
        "field": "type",
        "equals": "Microsoft.Compute/virtualMachines"
      },
      {
        "not": {
          "field": "Microsoft.Compute/virtualMachines/sku.name",
          "in": ["Standard_D2s_v3", "Standard_D4s_v3"]
        }
      }
    ]
  },
  "then": {
    "effect": "deny"
  }
}
```

In Terraform, deploy with `azurerm_policy_definition` and `azurerm_policy_assignment`. This enforces governance without manual reviews - policies automatically audit and block non-compliant deployments.

---

### Question 33: Kubernetes Pod Disruption Budgets

**Interviewer:** Ensure high availability during cluster maintenance.

**Candidate:**
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: myapp-pdb
spec:
  minAvailable: 2  # Or use maxUnavailable: 1
  selector:
    matchLabels:
      app: myapp
```

**Interviewer:** How does this help?

**Candidate:** During node drains for upgrades or scaling, Kubernetes respects PDB. It won't evict pods if doing so would violate minimum availability. With `minAvailable: 2`, at least 2 pods stay running during maintenance. Prevents all pods going down simultaneously, maintaining service availability.

---

### Question 34: Azure Bastion for Secure Access

**Interviewer:** How do you provide secure SSH/RDP access to VMs without public IPs?

**Candidate:** Azure Bastion - fully managed PaaS that provides RDP/SSH through Azure Portal over SSL. No need to expose VMs publicly. Deploy Bastion into a dedicated subnet, it brokers connections securely.

```hcl
resource "azurerm_subnet" "bastion" {
  name                 = "AzureBastionSubnet"  # Name must be this
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.255.0/27"]  # Minimum /27 required
}

resource "azurerm_public_ip" "bastion" {
  name                = "bastion-ip"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  allocation_method   = "Static"
  sku                 = "Standard"
}

resource "azurerm_bastion_host" "main" {
  name                = "myapp-bastion"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  ip_configuration {
    name                 = "configuration"
    subnet_id            = azurerm_subnet.bastion.id
    public_ip_address_id = azurerm_public_ip.bastion.id
  }
}
```

Users connect through Portal, no VPN needed, all traffic logged for audit.

---

### Question 35: Kubernetes Blue-Green Deployment

**Interviewer:** Implement zero-downtime blue-green deployment.

**Candidate:**
```yaml
# Blue deployment (current)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: blue
  template:
    metadata:
      labels:
        app: myapp
        version: blue
    spec:
      containers:
      - name: myapp
        image: myapp:v1.0

---
# Green deployment (new)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: green
  template:
    metadata:
      labels:
        app: myapp
        version: green
    spec:
      containers:
      - name: myapp
        image: myapp:v2.0

---
# Service - switch by changing selector
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp
    version: blue  # Switch to 'green' to cut over
  ports:
  - port: 80
    targetPort: 8080
```

**Interviewer:** How do you cutover?

**Candidate:** Deploy green, test it directly via pod IPs or temporary service. Once validated, update service selector to `version: green`. Traffic instantly switches. If issues, revert selector to `blue`. No downtime, instant rollback. Once confident, delete blue deployment.

---

### Question 36: Azure Front Door

**Interviewer:** Explain Azure Front Door and its use cases.

**Candidate:** Global HTTP load balancer with CDN capabilities. Routes requests to closest backend, does SSL offload, caching, WAF protection. Different from Traffic Manager which is DNS-based - Front Door is application-layer proxy with better performance.

**Interviewer:** When to use it?

**Candidate:** Multi-region applications for global users - routes to nearest region automatically, fails over if region is down. Combines CDN (cache static content), load balancer (route to backends), and WAF (security). More expensive than App Gateway but global reach. We use it for customer-facing web apps with users worldwide.

---

### Question 37: Ansible Roles for Configuration Management

**Interviewer:** Structure an Ansible playbook using roles.

**Candidate:**
```yaml
# playbooks/site.yml
---
- name: Configure Web Servers
  hosts: webservers
  become: yes
  roles:
    - common
    - nginx
    - ssl-certificates
    - monitoring-agent

- name: Configure Database Servers
  hosts: dbservers
  become: yes
  roles:
    - common
    - postgresql
    - backup-agent
    - monitoring-agent

# roles/nginx/tasks/main.yml
---
- name: Install Nginx
  apt:
    name: nginx
    state: present
    update_cache: yes

- name: Copy Nginx config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: Reload Nginx

- name: Start Nginx service
  systemd:
    name: nginx
    state: started
    enabled: yes

# roles/nginx/handlers/main.yml
---
- name: Reload Nginx
  systemd:
    name: nginx
    state: reloaded
```

**Interviewer:** Benefits of roles?

**Candidate:** Reusability - common role applied to all servers, nginx role only to web servers. Easy testing - test roles independently. Clear structure - each role is self-contained with tasks, templates, and handlers. Easy sharing - roles can be published to Ansible Galaxy.

---

### Question 38: Kubernetes Custom Resource Definitions (CRDs)

**Interviewer:** What are CRDs and when would you create one?

**Candidate:** CRDs extend Kubernetes API with custom resource types. Useful when you need to manage application-specific concepts declaratively. For example, instead of raw pods and services, you might define a `Tenant` CRD that automatically provisions namespace, RBAC, quotas, and network policies.

**Interviewer:** Basic example?

**Candidate:**
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: tenants.example.com
spec:
  group: example.com
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              name:
                type: string
              quota:
                type: object
                properties:
                  cpu:
                    type: string
                  memory:
                    type: string
  scope: Cluster
  names:
    plural: tenants
    singular: tenant
    kind: Tenant

---
# Using the CRD
apiVersion: example.com/v1
kind: Tenant
metadata:
  name: team-alpha
spec:
  name: team-alpha
  quota:
    cpu: "20"
    memory: "40Gi"
```

You'd pair this with a controller/operator that watches `Tenant` resources and creates necessary Kubernetes objects. Makes complex setups declarative and repeatable.

---

### Question 39: Azure DevOps Variable Groups with Key Vault

**Interviewer:** Link Azure Key Vault to Azure Pipelines securely.

**Candidate:** Create variable group in Azure DevOps, enable Link secrets from Azure Key Vault, select your Key Vault and subscription. Authorize the service connection. Select which secrets to include. In pipeline:

```yaml
variables:
- group: keyvault-secrets

stages:
- stage: Deploy
  jobs:
  - job: DeployJob
    steps:
    - script: |
        echo "Using secrets from Key Vault"
        # Reference as $(secret-name)
        kubectl create secret generic app-secrets \
          --from-literal=db-password=$(database-password) \
          --from-literal=api-key=$(external-api-key)
      displayName: 'Deploy with secrets'
```

**Interviewer:** Advantages over pipeline secrets?

**Candidate:** Centralized secret management - one place to update, audit, and rotate. Key Vault logs all access. Secrets automatically sync to pipeline. Multiple pipelines can share same variable group. Better compliance and security.

---

### Question 40: Docker Security Scanning

**Interviewer:** How do you ensure Docker images don't have vulnerabilities?

**Candidate:** Integrate security scanning in CI pipeline. Use tools like Trivy, Snyk, or Azure Container Registry scanning.

```yaml
# Azure Pipeline with Trivy
steps:
- script: |
    wget https://github.com/aquasecurity/trivy/releases/download/v0.45.0/trivy_0.45.0_Linux-64bit.tar.gz
    tar zxvf trivy_0.45.0_Linux-64bit.tar.gz
  displayName: 'Install Trivy'

- script: |
    ./trivy image --severity HIGH,CRITICAL \
      --exit-code 1 \
      myregistry.azurecr.io/myapp:$(Build.BuildId)
  displayName: 'Scan image for vulnerabilities'
```

**Interviewer:** What if a vulnerability is found?

**Candidate:** Pipeline fails with `--exit-code 1`. Review the CVEs, update base image or dependencies to patched versions, rebuild. For ACR, enable Azure Defender which continuously scans and alerts on new vulnerabilities even after deployment.

---

### Question 41: Kubernetes Operators Pattern

**Interviewer:** What's the Operator pattern?

**Candidate:** Operators are custom controllers that extend Kubernetes to manage complex applications. They encode operational knowledge - how to deploy, scale, backup, upgrade specific applications. Examples: MySQL Operator, Prometheus Operator.

**Interviewer:** How does it work?

**Candidate:** You define CRD for your application, Operator watches for these resources, reconciles actual state to desired state. For example, MySQL Operator watches `MySQLCluster` CRD - if you create one, Operator provisions StatefulSet, PVCs, Services, and configures replication automatically. If a pod dies, Operator recovers it, maintaining cluster topology.

---

### Question 42: Azure Traffic Manager

**Interviewer:** Explain Traffic Manager and its routing methods.

**Candidate:** DNS-based global load balancer. Returns different IPs based on routing policy. Four methods: Priority (active-passive failover), Performance (route to closest), Geographic (route by user location), Weighted (distribute percentage-based).

**Interviewer:** Use case?

**Candidate:** Multi-region disaster recovery - primary in East US, failover in West Europe using Priority routing. Or distribute load globally using Performance routing - European users hit Europe datacenter, Asian users hit Asia. It's DNS-level, so works with any service - VMs, App Services, external endpoints.

---

### Question 43: Git Stash and Worktree

**Interviewer:** You're working on a feature but need to urgently fix a bug on main. What do you do?

**Candidate:** `git stash` saves uncommitted changes, then `git checkout main` to switch branches, fix and commit, then return to feature branch with `git stash pop` to restore work. Alternatively, `git worktree add ../hotfix main` creates separate working directory for main branch - work on both simultaneously.

**Interviewer:** When is worktree better?

**Candidate:** When switching branches is expensive - like long rebuild times or complex test setups. With worktree, each has its own working directory and build artifacts. No need to rebuild when switching contexts.

---

### Question 44: Kubernetes Finalizers

**Interviewer:** What are finalizers?

**Candidate:** Metadata that prevents deletion until certain cleanup completes. When you delete a resource with finalizers, Kubernetes marks it for deletion but waits. A controller performs cleanup (like releasing external resources), removes the finalizer, then Kubernetes deletes the object.

**Interviewer:** Example use?

**Candidate:** Custom controller that provisions Azure storage for PVCs. When PVC is deleted, finalizer ensures controller deletes the Azure disk first, removes finalizer, then PVC is deleted. Prevents orphaned cloud resources costing money.

---

### Question 45: Azure Managed Identity for AKS

**Interviewer:** How does pod-managed identity work in AKS?

**Candidate:** AAD Pod Identity lets pods use Azure managed identities. You create identity, assign Azure roles, bind identity to Kubernetes service account, assign service account to pod. Pod gets Azure AD token automatically to access Azure services without secrets.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: myapp-sa
  annotations:
    azure.workload.identity/client-id: "<managed-identity-client-id>"

---
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  labels:
    azure.workload.identity/use: "true"
spec:
  serviceAccountName: myapp-sa
  containers:
  - name: app
    image: myapp:1.0
    env:
    - name: AZURE_CLIENT_ID
      value: "<managed-identity-client-id>"
```

Application SDK uses DefaultAzureCredential, automatically authenticates with managed identity - no connection strings or keys.

---

### Question 46: Terraform Lifecycle Meta-Arguments

**Interviewer:** Prevent accidental resource destruction in Terraform.

**Candidate:**
```hcl
resource "azurerm_postgresql_server" "main" {
  name                = "myapp-db-server"
  resource_group_name = azurerm_resource_group.main.name
  # ... other config

  lifecycle {
    prevent_destroy = true  # Terraform errors if you try to destroy
    ignore_changes = [
      tags,  # Ignore tag changes made outside Terraform
      administrator_login_password  # Don't update password
    ]
    create_before_destroy = true  # Create new before deleting old
  }
}
```

**Interviewer:** When to use create_before_destroy?

**Candidate:** Resources that can't have downtime - like databases or load balancers. Terraform creates new instance, updates dependencies to use it, then deletes old. Brief overlap but zero downtime.

---

### Question 47: Azure Service Bus

**Interviewer:** Explain Service Bus and when to use it over Storage Queues.

**Candidate:** Both are message queues, but Service Bus is enterprise-grade. Supports topics/subscriptions (pub-sub), sessions for ordered processing, dead-letter queues, scheduled messages, and transactions. Storage Queue is simpler, cheaper, but basic - just FIFO queue.

**Interviewer:** Architecture example?

**Candidate:** Order processing system - frontend pushes to Service Bus topic, multiple subscribers: inventory service, shipping service, notification service each get copy. If a service fails processing, message goes to dead-letter queue for investigation. Sessions ensure orders from same customer process sequentially.

---

### Question 48: Kubernetes Autoscaling Deep Dive

**Interviewer:** Explain all three autoscaling types in Kubernetes.

**Candidate:** 
1. **HPA (Horizontal Pod Autoscaler)** - scales pod replicas based on CPU, memory, or custom metrics
2. **VPA (Vertical Pod Autoscaler)** - adjusts pod resource requests/limits based on actual usage
3. **Cluster Autoscaler** - adds/removes nodes based on pending pods

**Interviewer:** How do they work together?

**Candidate:** HPA scales pods horizontally when load increases. If nodes are full and pods are pending, Cluster Autoscaler adds nodes. VPA runs separately, recommending or automatically adjusting resource requests for better efficiency. Typically use HPA + Cluster Autoscaler together; VPA for workloads with unpredictable resource needs.

---

### Question 49: Azure Cost Management

**Interviewer:** How do you monitor and optimize Azure spending?

**Candidate:** Use Cost Management + Billing - set budgets with alert thresholds, analyze spending by resource group/tag. Use Azure Advisor for optimization recommendations - resize underutilized VMs, delete unattached disks, reserve instances for predictable workloads.

**Interviewer:** Practical cost-saving strategies?

**Candidate:** Auto-shutdown dev/test resources nightly, use Azure Dev/Test pricing, Spot VMs for fault-tolerant workloads, reserved instances for production (1-year saves 40%, 3-year saves 60%), delete old snapshots and images, use lifecycle management to archive cool storage. Tag everything with CostCenter for chargeback.

---

### Question 50: DevOps Metrics and KPIs

**Interviewer:** What metrics do you track to measure DevOps success?

**Candidate:** DORA metrics - Deployment Frequency (how often deploy to prod), Lead Time for Changes (code commit to prod), Mean Time to Recovery (MTTR after incidents), Change Failure Rate (percentage of deployments causing failures).

**Interviewer:** Your team's current metrics?

**Candidate:** We deploy to production 3-5 times per day, lead time is under 4 hours from PR merge, MTTR averages 15 minutes thanks to automated rollback, change failure rate is around 5%. We track these in Grafana dashboards, review trends monthly to identify improvements. Also monitor CI/CD pipeline success rate and test coverage as leading indicators.

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

---

**✅ Interview Session 3 Complete - 50 Unique In-Depth Technical Questions!**

**Topics Covered:** VNet Peering, Subnets, NSGs, RBAC, PVC/Storage, Docker Networking, Jenkins/Azure Pipelines Configuration, Terraform Modules, Kubernetes Deep Dive (Network Policies, Ingress, StatefulSets), Azure Services (Key Vault, App Gateway, Front Door, Bastion), Monitoring, and DevOps Best Practices.

**Total Questions: 50/50** | **Format:** Conversational with Technical Depth

