# DevOps Interview Session 4
## Conversational Format - Scenario-Based Questions (3-7 Years)

---

### Question 1: Container Registry Migration

**Interviewer:** Your Docker Hub images are hitting rate limits. Plan a migration to Azure Container Registry.

**Candidate:** First, create ACR with geo-replication for HA. Then script the migration - pull images from Docker Hub, retag for ACR, push to ACR. Update all references in Kubernetes manifests and CI/CD pipelines. Test in dev environment first.

```bash
# Migration script
DOCKER_IMAGES=("nginx:latest" "redis:7" "postgres:14")
ACR_NAME="myregistry.azurecr.io"

az acr login --name myregistry

for image in "${DOCKER_IMAGES[@]}"; do
  docker pull $image
  docker tag $image $ACR_NAME/$image
  docker push $ACR_NAME/$image
done

# Update Kubernetes
kubectl set image deployment/myapp myapp=$ACR_NAME/myapp:v1.0
```

**Interviewer:** How do you handle authentication?

**Candidate:** For AKS, attach ACR with `az aks update --attach-acr`. For other clients, use service principal or admin credentials. In pipelines, use Azure service connection. No more Docker Hub tokens to manage.

---

### Question 2: Database Migration Zero Downtime

**Interviewer:** Migrate from PostgreSQL VM to Azure Database for PostgreSQL without downtime.

**Candidate:** Use Azure Database Migration Service. First, set up Azure DB, configure replication from source VM using logical replication slots. Once synced, switch application connection string during low-traffic window, verify, then decommission old VM.

**Interviewer:** What if something goes wrong?

**Candidate:** Keep old database running for 24 hours. Connection string is environment variable, easy to revert. Test rollback procedure beforehand. Monitor error rates and query performance closely after cutover.

---

### Question 3: Terraform Import Existing Resources

**Interviewer:** You have manually created Azure resources. How do you bring them under Terraform management?

**Candidate:** Write Terraform configuration matching the existing resource, then import it into state. For example:

```hcl
# Write the config
resource "azurerm_storage_account" "existing" {
  name                     = "existingstorage"
  resource_group_name      = "existing-rg"
  location                 = "eastus"
  account_tier             = "Standard"
  account_replication_type = "LRS"
}

# Import into state
terraform import azurerm_storage_account.existing /subscriptions/{sub-id}/resourceGroups/existing-rg/providers/Microsoft.Storage/storageAccounts/existingstorage

# Run plan to ensure match
terraform plan  # Should show no changes
```

**Interviewer:** What if the plan shows changes?

**Candidate:** Means my config doesn't match actual resource. Adjust configuration until `terraform plan` shows zero changes, then it's fully managed by Terraform.

---

### Question 4: Kubernetes Multi-Tenancy

**Interviewer:** Design a multi-tenant AKS cluster for different customers.

**Candidate:** Namespace per tenant with ResourceQuotas, Network Policies to isolate traffic, separate service accounts with RBAC, and LimitRanges for defaults. For stronger isolation, use node pools with taints/tolerations so each tenant runs on dedicated nodes.

```yaml
# Tenant namespace with isolation
apiVersion: v1
kind: Namespace
metadata:
  name: tenant-acme
  labels:
    tenant: acme

---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: tenant-quota
  namespace: tenant-acme
spec:
  hard:
    pods: "50"
    requests.cpu: "10"
    requests.memory: "20Gi"

---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-from-other-namespaces
  namespace: tenant-acme
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector: {}  # Only same namespace
```

**Interviewer:** Is this enough for strict isolation?

**Candidate:** For most cases yes, but truly hostile multi-tenancy needs separate clusters or virtual clusters with vCluster. Namespaces share the same nodes and kernel, so there's theoretical escape risk.

---

### Question 5: Azure Front Door Global Failover

**Interviewer:** Configure automatic failover between US and Europe regions.

**Candidate:** Front Door with backend pools in both regions, health probes enabled. If US backend fails health checks, Front Door automatically routes to Europe backend.

```hcl
resource "azurerm_frontdoor" "main" {
  name                = "myapp-frontdoor"
  resource_group_name = azurerm_resource_group.main.name

  backend_pool {
    name = "myapp-backends"
    backend {
      host_header = "myapp-us.azurewebsites.net"
      address     = "myapp-us.azurewebsites.net"
      http_port   = 80
      https_port  = 443
      priority    = 1  # Primary
      weight      = 100
    }
    backend {
      host_header = "myapp-eu.azurewebsites.net"
      address     = "myapp-eu.azurewebsites.net"
      http_port   = 80
      https_port  = 443
      priority    = 2  # Failover
      weight      = 100
    }
    load_balancing_name = "myapp-lb"
    health_probe_name   = "myapp-probe"
  }

  backend_pool_health_probe {
    name         = "myapp-probe"
    path         = "/health"
    protocol     = "Https"
    interval_in_seconds = 30
  }
}
```

**Interviewer:** How fast is failover?

**Candidate:** Typically 30-60 seconds depending on health probe interval. Once backend is marked unhealthy, new requests route to Europe immediately. Existing connections to failed backend break.

---

### Question 6: Jenkins Plugin Vulnerability

**Interviewer:** Security scan flagged a Jenkins plugin vulnerability. How do you handle it?

**Candidate:** Check Jenkins security advisories for details and patch version. Update plugin through Manage Jenkins → Manage Plugins → Updates. Test in dev Jenkins instance first. For critical vulnerabilities, disable the plugin temporarily if patching isn't immediate.

**Interviewer:** How do you prevent this?

**Candidate:** Enable automatic plugin updates for security fixes only. Subscribe to Jenkins security mailing list. Use plugin compatibility checker before upgrading Jenkins itself. Regularly audit installed plugins and remove unused ones.

---

### Question 7: Kubernetes Sealed Secrets

**Interviewer:** Implement GitOps-friendly secret management.

**Candidate:** Install Sealed Secrets controller in cluster. Encrypt secrets with `kubeseal`, commit encrypted SealedSecret to Git. Controller decrypts and creates normal Secret in cluster.

```bash
# Install Sealed Secrets
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.18.0/controller.yaml

# Create and seal secret
kubectl create secret generic db-creds \
  --from-literal=password=SuperSecret \
  --dry-run=client -o yaml | \
  kubeseal -o yaml > sealed-secret.yaml

# sealed-secret.yaml is safe to commit - encrypted
git add sealed-secret.yaml
git commit -m "Add database credentials"
```

**Interviewer:** How does unsealing work?

**Candidate:** Controller has a private key stored in cluster. When Sealed Secret is applied, controller decrypts it using its private key, creates a normal Secret. Only the controller can decrypt - safe to store in Git.

---

### Question 8: Azure Site Recovery

**Interviewer:** Setup disaster recovery for critical VMs.

**Candidate:** Azure Site Recovery replicates VMs to secondary region. Configure recovery vault, enable replication for VMs, set recovery point objectives. Create recovery plan with failover order and scripts.

**Interviewer:** What's the RTO/RPO?

**Candidate:** RPO is typically 5 minutes - how much data loss you can tolerate. RTO depends on VM size and complexity, usually 15-30 minutes for failover to complete. Test failover regularly to verify RTO targets.

---

### Question 9: Docker Layer Caching in CI

**Interviewer:** Speed up Docker builds in Azure Pipelines.

**Candidate:**
```yaml
steps:
- task: Docker@2
  displayName: 'Pull previous image for cache'
  inputs:
    command: pull
    arguments: 'myregistry.azurecr.io/myapp:latest'
  continueOnError: true  # First build won't have cache

- task: Docker@2
  displayName: 'Build with cache'
  inputs:
    command: build
    Dockerfile: '**/Dockerfile'
    arguments: '--cache-from myregistry.azurecr.io/myapp:latest --build-arg BUILDKIT_INLINE_CACHE=1'
    tags: |
      $(Build.BuildId)
      latest

- task: Docker@2
  displayName: 'Push image'
  inputs:
    command: push
    tags: |
      $(Build.BuildId)
      latest
```

**Interviewer:** How much does this save?

**Candidate:** Depends on image, but typically 50-70% time reduction. Installing dependencies is cached - only application code layers rebuild. First build is still slow, subsequent builds fly.

---

### Question 10: Kubernetes Pod Priority and Preemption

**Interviewer:** Ensure critical pods always schedule, even if cluster is full.

**Candidate:**
```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: "High priority for critical services"

---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: low-priority
value: 100
globalDefault: false
preemptionPolicy: PreemptLowerPriority
description: "Low priority for batch jobs"

---
apiVersion: v1
kind: Pod
metadata:
  name: critical-app
spec:
  priorityClassName: high-priority
  containers:
  - name: app
    image: critical-app:1.0
```

**Interviewer:** What happens when cluster is full?

**Candidate:** High-priority pod preempts low-priority pods - Kubernetes evicts them to make room. Critical services get resources, batch jobs wait. Important for maintaining SLA during peak load.

---

### Question 11: Azure ExpressRoute

**Interviewer:** Explain ExpressRoute and when to use it over VPN.

**Candidate:** ExpressRoute is private connection from on-premises to Azure, doesn't go over internet. More reliable, faster (up to 100Gbps), lower latency than VPN. Expensive though - monthly circuit cost plus bandwidth.

**Interviewer:** Use case?

**Candidate:** Large data transfers - like database migration or backup/restore. Hybrid applications with tight latency requirements. Compliance requiring private connectivity. Small shops use Site-to-Site VPN, enterprises with serious traffic use ExpressRoute.

---

### Question 12: Git Bisect for Bug Hunting

**Interviewer:** A bug exists in production but you don't know which commit introduced it.

**Candidate:** Use `git bisect` to binary search through commits:

```bash
git bisect start
git bisect bad  # Current commit is bad
git bisect good v1.0.0  # Last known good version

# Git checks out middle commit
# Test it, then mark:
git bisect good  # or git bisect bad

# Repeat until Git identifies the exact commit
git bisect reset  # Done
```

**Interviewer:** How many tests does this take?

**Candidate:** With 100 commits between good and bad, only need ~7 tests (log2(100)). Much faster than testing each commit linearly.

---

### Question 13: Kubernetes Admission Controllers

**Interviewer:** Enforce that all pods must have resource limits set.

**Candidate:** Use OPA Gatekeeper or custom admission webhook. Gatekeeper is easier:

```yaml
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8srequiredresources
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredResources
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredresources
        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          not container.resources.limits
          msg := sprintf("Container %v must have resource limits", [container.name])
        }
---
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredResources
metadata:
  name: must-have-limits
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
```

**Interviewer:** What happens if someone tries to deploy without limits?

**Candidate:** Admission denied - pod creation fails with clear error message. Forces developers to set limits, improving cluster stability.

---

### Question 14: Azure Storage Lifecycle Management

**Interviewer:** Reduce storage costs for old data.

**Candidate:** Implement lifecycle policies to automatically tier storage:

```json
{
  "rules": [{
    "enabled": true,
    "name": "archive-old-blobs",
    "type": "Lifecycle",
    "definition": {
      "filters": {
        "blobTypes": ["blockBlob"],
        "prefixMatch": ["logs/"]
      },
      "actions": {
        "baseBlob": {
          "tierToCool": {
            "daysAfterModificationGreaterThan": 30
          },
          "tierToArchive": {
            "daysAfterModificationGreaterThan": 90
          },
          "delete": {
            "daysAfterModificationGreaterThan": 365
          }
        }
      }
    }
  }]
}
```

**Interviewer:** Cost savings?

**Candidate:** Cool tier is 50% cheaper than hot, archive is 90% cheaper. For logs older than 90 days that you rarely access, huge savings. Just remember archive has retrieval costs and hours of latency.

---

### Question 15: Helm Hooks

**Interviewer:** Run database migration before deploying application pods.

**Candidate:**
```yaml
# templates/db-migration-job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ .Release.Name }}-db-migration
  annotations:
    "helm.sh/hook": pre-install,pre-upgrade
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": before-hook-creation
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: migrate
        image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
        command: ["npm", "run", "migrate"]
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-creds
              key: url
```

**Interviewer:** What if migration fails?

**Candidate:** Helm install/upgrade fails, application pods don't deploy. Database stays at old schema, old version keeps running. Fix migration, retry Helm upgrade.

---

### Question 16: Azure Virtual Network Service Endpoints

**Interviewer:** Secure Azure SQL access from AKS without internet exposure.

**Candidate:** Enable service endpoint on AKS subnet, configure SQL firewall to allow that subnet. Traffic stays on Azure backbone, never touches internet.

```hcl
resource "azurerm_subnet" "aks" {
  name                 = "aks-subnet"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.1.0/24"]
  
  service_endpoints = ["Microsoft.Sql"]
}

resource "azurerm_sql_virtual_network_rule" "aks" {
  name                = "aks-access"
  resource_group_name = azurerm_resource_group.main.name
  server_name         = azurerm_sql_server.main.name
  subnet_id           = azurerm_subnet.aks.id
}
```

**Interviewer:** Service endpoint vs private endpoint?

**Candidate:** Service endpoint is routing - traffic stays private but SQL still has public IP. Private endpoint gives SQL a private IP in your VNet - more secure, no public endpoint at all. Endpoint is free, private endpoint costs $7/month.

---

### Question 17: Jenkins Declarative Pipeline Matrix

**Interviewer:** Test application against multiple Node.js versions in parallel.

**Candidate:**
```groovy
pipeline {
    agent none
    stages {
        stage('Test') {
            matrix {
                agent {
                    label 'docker'
                }
                axes {
                    axis {
                        name 'NODE_VERSION'
                        values '14', '16', '18', '20'
                    }
                    axis {
                        name 'OS'
                        values 'linux', 'windows'
                    }
                }
                stages {
                    stage('Run Tests') {
                        steps {
                            script {
                                docker.image("node:${NODE_VERSION}-${OS}").inside {
                                    sh 'npm install'
                                    sh 'npm test'
                                }
                            }
                        }
                    }
                }
            }
        }
    }
}
```

**Interviewer:** How many jobs does this create?

**Candidate:** 8 parallel jobs - 4 Node versions × 2 OSes. All run simultaneously if agents available, dramatically faster than sequential testing.

---

### Question 18: Kubernetes DaemonSet Use Cases

**Interviewer:** When would you use DaemonSet?

**Candidate:** Node-level services that must run on every node - log collectors like Fluentd, monitoring agents like Datadog, network plugins like Calico, security scanners. One pod per node automatically, including new nodes.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule  # Run even on master nodes
      containers:
      - name: fluentd
        image: fluent/fluentd:v1.14
        volumeMounts:
        - name: varlog
          mountPath: /var/log
          readOnly: true
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```

**Interviewer:** What if you don't want it on certain nodes?

**Candidate:** Use node selector or taints/tolerations. For example, skip GPU nodes with node anti-affinity, or only run on nodes with label `logging=enabled`.

---

### Question 19: Azure Backup and Recovery

**Interviewer:** Backup strategy for Azure VMs and databases.

**Candidate:** Azure Backup for VMs - daily snapshots retained per policy, stored in Recovery Services Vault across regions. For databases, Azure SQL automated backups with point-in-time restore up to 35 days. For critical data, additionally export to immutable blob storage.

**Interviewer:** How do you test restores?

**Candidate:** Monthly restore drills - restore to separate resource group, verify data integrity, document time taken, then delete test resources. Most backup failures are discovered during restore, not backup.

---

### Question 20: Docker Rootless Mode

**Interviewer:** What's rootless Docker and why use it?

**Candidate:** Docker daemon runs as non-root user, containers don't have root privileges on host. More secure - if container escapes, it's unprivileged user, not root. Useful for multi-tenant CI systems.

**Interviewer:** Any limitations?

**Candidate:** Can't bind to ports below 1024 without capabilities, some storage drivers unsupported, slight performance overhead. For Kubernetes, use pod security policies/standards instead - easier and more standard.

---

### Question 21: Azure Policy Initiatives

**Interviewer:** Enforce multiple related policies together.

**Candidate:** Initiative is a bundle of policies. For example, "CIS Azure Foundations" initiative contains 100+ policies for compliance. Assign initiative once, all policies apply.

```hcl
resource "azurerm_policy_set_definition" "security" {
  name         = "security-baseline"
  policy_type  = "Custom"
  display_name = "Security Baseline"

  parameters = <<PARAMETERS
    {
      "allowedLocations": {
        "type": "Array"
      }
    }
PARAMETERS

  policy_definition_reference {
    policy_definition_id = azurerm_policy_definition.require_tags.id
  }

  policy_definition_reference {
    policy_definition_id = azurerm_policy_definition.allowed_vm_sizes.id
  }

  policy_definition_reference {
    policy_definition_id = azurerm_policy_definition.require_https.id
  }
}
```

**Interviewer:** How do you handle exceptions?

**Candidate:** Policy exemptions - grant exemption to specific resource or resource group with justification and expiration date. Audited for compliance review.

---

### Question 22: Kubernetes Immutable ConfigMaps/Secrets

**Interviewer:** Prevent accidental ConfigMap changes breaking production.

**Candidate:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config-v2
immutable: true
data:
  app.properties: |
    version=2.0
    feature.newUI=enabled
```

**Interviewer:** What if you need to change it?

**Candidate:** Can't modify immutable ConfigMap - must create new version with different name, update deployment to use it, delete old version. Forces versioning and prevents accidental edits. Good practice for production.

---

### Question 23: Azure Container Instances for Burst Workloads

**Interviewer:** Handle occasional CPU-intensive jobs without keeping expensive VMs running.

**Candidate:** AKS Virtual Nodes with ACI - pods scheduled on virtual-kubelet burst to ACI serverless containers. Only pay for seconds used.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: batch-job
spec:
  nodeSelector:
    kubernetes.io/role: agent
    beta.kubernetes.io/os: linux
    type: virtual-kubelet
  tolerations:
  - key: virtual-kubelet.io/provider
    operator: Exists
  containers:
  - name: processor
    image: data-processor:1.0
    resources:
      requests:
        memory: "8Gi"
        cpu: "4"
```

**Interviewer:** When not to use ACI?

**Candidate:** Persistent services - ACI for short-lived jobs only. No persistent volumes support. Higher per-second cost than VMs, only cheaper for sporadic workloads. For steady state, regular nodes are cheaper.

---

### Question 24: Terraform Workspaces for Environment Management

**Interviewer:** You mentioned preferring directories over workspaces. Defend workspaces.

**Candidate:** Workspaces work well for identical infrastructure with different variable values. Single codebase, switch workspace, different state file.

```bash
terraform workspace new dev
terraform workspace new prod

terraform workspace select dev
terraform apply -var-file=dev.tfvars

terraform workspace select prod
terraform apply -var-file=prod.tfvars
```

Use workspace name in configs:
```hcl
resource "azurerm_resource_group" "main" {
  name     = "myapp-${terraform.workspace}-rg"
  location = var.location
}
```

**Interviewer:** So why do you prefer directories?

**Candidate:** Workspaces hide which environment you're in - easy to accidentally apply to wrong workspace. Separate directories are explicit - you're physically in the prod folder. But workspaces valid for quick dev/test environments.

---

### Question 25: GitOps Sync Policies

**Interviewer:** Configure ArgoCD automated sync with pruning.

**Candidate:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/org/repo
    targetRevision: HEAD
    path: k8s/overlays/prod
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true  # Delete resources not in Git
      selfHeal: true  # Revert manual changes
      allowEmpty: false
    syncOptions:
    - CreateNamespace=true
    - PrunePropagationPolicy=foreground
    - PruneLast=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

**Interviewer:** What's the risk of prune: true?

**Candidate:** Accidentally deleting wrong file in Git deletes resources in cluster. Use with caution in production. Start with prune: false and manual sync until confident in GitOps practices.

---

### Question 26: Azure Monitor Workbooks

**Interviewer:** Create custom monitoring dashboard.

**Candidate:** Workbooks are interactive dashboards combining metrics, logs, and parameters. Better than fixed dashboards - users can filter by time range, resource group, etc.

Create workbook with KQL queries:
```kusto
// Pod resource usage
KubePodInventory
| where Namespace == "production"
| join kind=inner (
    Perf
    | where ObjectName == "K8SContainer"
    | where CounterName == "cpuUsageNanoCores"
) on $left.ContainerID == $right.InstanceName
| summarize AvgCPU = avg(CounterValue) by PodName, bin(TimeGenerated, 5m)
| render timechart
```

Parameterize for interactivity - users select namespace dropdown, charts update automatically.

---

### Question 27: Kubernetes Liveness vs Readiness vs Startup Probes

**Interviewer:** Explain all three probe types.

**Candidate:** 
- **Liveness:** Is container alive? If fails, restart it. For detecting deadlocks/hangs
- **Readiness:** Is container ready for traffic? If fails, remove from service endpoints. For temporary unready states
- **Startup:** Has slow-starting container finished starting? If fails, restart. Protects liveness probe during startup

**Interviewer:** Configuration example?

**Candidate:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: app
    image: myapp:1.0
    startupProbe:
      httpGet:
        path: /startup
        port: 8080
      failureThreshold: 30  # 30 * 10s = 5 minutes to start
      periodSeconds: 10
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 0  # Startup probe handles this
      periodSeconds: 10
      failureThreshold: 3
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      periodSeconds: 5
      successThreshold: 1
```

---

### Question 28: Azure Service Fabric vs AKS

**Interviewer:** When would you choose Service Fabric over AKS?

**Candidate:** Honestly, rarely now. Service Fabric is Microsoft's older orchestrator, predates Kubernetes. Use it only if you have existing Service Fabric apps or Windows-specific requirements. AKS is industry standard, better community, more tools, easier hiring.

---

### Question 29: Maven Release Plugin

**Interviewer:** Automate versioning and releases with Maven.

**Candidate:**
```bash
# Release process
mvn release:prepare  # Updates versions, tags Git
mvn release:perform  # Builds and deploys release

# release:prepare does:
# 1. Removes -SNAPSHOT from version
# 2. Commits pom.xml
# 3. Creates Git tag
# 4. Increments version to next SNAPSHOT
# 5. Commits again
```

Configure in pom.xml:
```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-release-plugin</artifactId>
      <version>3.0.0</version>
      <configuration>
        <tagNameFormat>v@{project.version}</tagNameFormat>
        <autoVersionSubmodules>true</autoVersionSubmodules>
      </configuration>
    </plugin>
  </plugins>
</build>
```

**Interviewer:** Better than manual versioning?

**Candidate:** Eliminates human error, enforces consistency, automatic Git tagging. In CI/CD, makes releases traceable and repeatable.

---

### Question 30: Kubernetes Cluster Autoscaler Tuning

**Interviewer:** Cluster autoscaler is scaling too aggressively. How do you tune it?

**Candidate:** Adjust parameters:
- `scale-down-delay-after-add`: Wait time after scale-up before considering scale-down (default 10m)
- `scale-down-unneeded-time`: How long node is unneeded before removal (default 10m)
- `scale-down-utilization-threshold`: CPU/memory threshold for unneeded (default 0.5)

```yaml
# Cluster autoscaler deployment
spec:
  containers:
  - command:
    - ./cluster-autoscaler
    - --scale-down-delay-after-add=15m  # Slower scale-down
    - --scale-down-unneeded-time=15m
    - --scale-down-utilization-threshold=0.6  # More aggressive
```

**Interviewer:** What if it scales up too slowly?

**Candidate:** Check `--max-node-provision-time` - if node takes longer than this to become ready, autoscaler gives up. Increase if using large VM sizes with slow provisioning.

---

### Question 31: Azure NSG Flow Logs

**Interviewer:** Debug network connectivity issues in Azure.

**Candidate:** Enable NSG flow logs to capture all traffic hitting NSGs. Logs go to Storage Account, analyze with Network Watcher or Log Analytics.

```hcl
resource "azurerm_network_watcher_flow_log" "nsg" {
  network_watcher_name      = "NetworkWatcher_eastus"
  resource_group_name       = "NetworkWatcherRG"
  network_security_group_id = azurerm_network_security_group.main.id
  storage_account_id        = azurerm_storage_account.logs.id
  enabled                   = true
  version                   = 2

  retention_policy {
    enabled = true
    days    = 7
  }

  traffic_analytics {
    enabled               = true
    workspace_id          = azurerm_log_analytics_workspace.main.workspace_id
    workspace_region      = "eastus"
    workspace_resource_id = azurerm_log_analytics_workspace.main.id
    interval_in_minutes   = 10
  }
}
```

**Interviewer:** What can you find with this?

**Candidate:** Blocked connections showing why traffic isn't reaching destination, top talkers consuming bandwidth, geographic traffic patterns, malicious connection attempts. Essential for troubleshooting "why can't X reach Y?"

---

### Question 32: Docker BuildKit Features

**Interviewer:** What's BuildKit and why enable it?

**Candidate:** Next-gen Docker build engine with better performance, caching, and features.

```bash
# Enable BuildKit
export DOCKER_BUILDKIT=1

# Or in Dockerfile
# syntax=docker/dockerfile:1.4
```

Features: parallel build stages, improved caching, secrets mounting without leaving traces, SSH agent forwarding, faster builds.

```dockerfile
# Mount secret during build without adding to layers
RUN --mount=type=secret,id=npmrc,dst=/root/.npmrc npm install

# docker build --secret id=npmrc,src=$HOME/.npmrc
```

**Interviewer:** Performance gains?

**Candidate:** 30-50% faster on average. Parallel builds of independent stages. Much better layer caching. Should be default for all modern Docker builds.

---

### Question 33: Kubernetes Custom Metrics for Autoscaling

**Interviewer:** Scale based on queue length, not CPU.

**Candidate:** Use metrics-server with custom metrics adapter.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: queue-processor
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: queue-processor
  minReplicas: 2
  maxReplicas: 20
  metrics:
  - type: External
    external:
      metric:
        name: queue_length
        selector:
          matchLabels:
            queue: "orders"
      target:
        type: AverageValue
        averageValue: "30"  # Scale when queue > 30 per pod
```

**Interviewer:** How does HPA get queue length?

**Candidate:** Install KEDA (Kubernetes Event-Driven Autoscaling) - supports Azure Queue, Service Bus, Kafka, etc. KEDA queries queue, exposes as Kubernetes metric, HPA scales based on it.

---

### Question 34: Azure Logic Apps for Automation

**Interviewer:** Automate incident response workflow.

**Candidate:** Logic App triggers on Azure Monitor alert, posts to Teams, creates ticket in ServiceNow, calls remediation API. All no-code connectors.

Workflow:
1. Azure Monitor Alert fires
2. Logic App HTTP trigger receives alert
3. Parse JSON payload
4. Post to Teams channel
5. Create ServiceNow incident
6. If critical severity, page on-call via PagerDuty
7. Call auto-remediation webhook

**Interviewer:** When not to use Logic Apps?

**Candidate:** Complex business logic - use Azure Functions with code instead. Logic Apps for simple workflows, Functions for complex logic. Logic Apps are expensive at high volumes compared to Functions.

---

### Question 35: Git Worktree Advanced Usage

**Interviewer:** You're comparing behavior between two branches extensively.

**Candidate:**
```bash
# Add worktree for feature branch
git worktree add ../feature-test feature/new-api

# Now have two directories:
# ./         (main branch)
# ../feature-test (feature branch)

# Run both simultaneously
cd ../ && npm start  # Terminal 1
cd ../feature-test && npm start  # Terminal 2

# Compare behavior, test integration

# Remove worktree when done
git worktree remove ../feature-test
```

**Interviewer:** When is this better than branches?

**Candidate:** Testing two versions side-by-side, long-running comparisons, parallel builds without stashing/rebuilding. Especially useful for monorepos with slow build times.

---

### Question 36: Azure Availability Zones

**Interviewer:** Design highly available architecture using availability zones.

**Candidate:** Availability zones are separate datacenters within a region. Deploy VMs, databases, load balancers across zones for resilience against datacenter failures.

```hcl
# VMs in different zones
resource "azurerm_linux_virtual_machine" "web1" {
  name                = "web1"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  zone                = "1"
  size                = "Standard_D2s_v3"
  # ...
}

resource "azurerm_linux_virtual_machine" "web2" {
  name                = "web2"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  zone                = "2"
  size                = "Standard_D2s_v3"
  # ...
}

# Zone-redundant load balancer
resource "azurerm_lb" "main" {
  name                = "web-lb"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  sku                 = "Standard"  # Standard LB is zone-redundant
}
```

**Interviewer:** Cost impact?

**Candidate:** Same VM cost, but cross-zone bandwidth charged (small). Worth it for 99.99% SLA vs 99.9% for single-zone. For AKS, use node pools spread across zones automatically.

---

### Question 37: Kubernetes Affinity and Anti-Affinity

**Interviewer:** Ensure frontend and backend pods are co-located for low latency.

**Candidate:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: backend
        tier: backend
    spec:
      containers:
      - name: backend
        image: backend:1.0

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: frontend
    spec:
      affinity:
        podAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: tier
                  operator: In
                  values:
                  - backend
              topologyKey: kubernetes.io/hostname
      containers:
      - name: frontend
        image: frontend:1.0
```

**Interviewer:** Preferred vs required affinity?

**Candidate:** Preferred is soft - scheduler tries but doesn't guarantee. Required is hard - pod won't schedule if rule can't be satisfied. Use preferred for performance optimization, required for hard constraints.

---

### Question 38: Azure Defender for Cloud

**Interviewer:** Enable security monitoring across Azure resources.

**Candidate:** Defender for Cloud continuously scans for vulnerabilities, misconfigurations, and threats. Enable for VMs, containers, databases, storage.

Provides:
- Secure score with recommendations
- Threat protection and alerts
- Compliance dashboard
- JIT VM access
- Adaptive application controls
- File integrity monitoring

**Interviewer:** Cost?

**Candidate:** Free tier gives recommendations. Standard tier ($15/VM/month) adds threat protection, JIT access, advanced features. Worth it for production environments - catches misconfigurations and threats quickly.

---

### Question 39: Terraform Sentinel Policies

**Interviewer:** Prevent expensive VM sizes from being deployed.

**Candidate:** Terraform Cloud/Enterprise feature - write policies in Sentinel language to enforce governance.

```python
import "tfplan/v2" as tfplan

# Get all VM resources
vms = filter tfplan.resource_changes as _, rc {
    rc.type is "azurerm_linux_virtual_machine" and
    rc.mode is "managed" and
    rc.change.actions contains "create"
}

# Allowed VM sizes
allowed_sizes = ["Standard_B2s", "Standard_D2s_v3", "Standard_D4s_v3"]

# Validate each VM
vm_size_valid = rule {
    all vms as _, vm {
        vm.change.after.size in allowed_sizes
    }
}

# Main rule
main = rule {
    vm_size_valid
}
```

**Interviewer:** What if policy fails?

**Candidate:** Terraform apply blocked. Advisory policies warn but allow, mandatory policies hard-block. Governance team defines policies, developers can't bypass without approval.

---

### Question 40: Kubernetes Init Containers

**Interviewer:** Wait for database to be ready before starting application.

**Candidate:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  initContainers:
  - name: wait-for-db
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup database.default.svc.cluster.local; do echo waiting for database; sleep 2; done']
  
  - name: run-migrations
    image: myapp:1.0
    command: ['npm', 'run', 'migrate']
    env:
    - name: DATABASE_URL
      valueFrom:
        secretKeyRef:
          name: db-creds
          key: url
  
  containers:
  - name: app
    image: myapp:1.0
```

**Interviewer:** Init container execution order?

**Candidate:** Sequential - first init container runs to completion, then second, then third. Only after all init containers succeed do regular containers start. If any init container fails, pod restarts.

---

### Question 41: Azure Chaos Studio

**Interviewer:** Test application resilience with chaos engineering.

**Candidate:** Azure Chaos Studio injects faults - kill VMs, add network latency, CPU pressure, disconnect storage. Verify application handles failures gracefully.

Experiments:
- VM shutdown
- Network disconnect
- DNS failure
- High CPU/memory
- Storage throttling

**Interviewer:** Have you run chaos experiments?

**Candidate:** Yes, tested our AKS deployment. Killed random pods to verify HPA scaled correctly and no user impact. Found bug where application didn't retry database connections - fixed before production incident. Chaos engineering finds issues before customers do.

---

### Question 42: Docker Multi-Platform Builds

**Interviewer:** Build images for both AMD64 and ARM64.

**Candidate:**
```bash
# Create builder
docker buildx create --name multiplatform --use

# Build for multiple platforms
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t myregistry.azurecr.io/myapp:1.0 \
  --push \
  .

# Buildx creates manifest list - correct image pulled automatically
```

**Interviewer:** Why support ARM64?

**Candidate:** AWS Graviton, Azure ARM VMs cost 20% less than x86. Same image works on both - Kubernetes automatically pulls right architecture. Good for cost optimization without code changes.

---

### Question 43: Kubernetes Topology Spread Constraints

**Interviewer:** Evenly distribute pods across zones.

**Candidate:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 9
  template:
    spec:
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: DoNotSchedule
        labelSelector:
          matchLabels:
            app: myapp
      containers:
      - name: app
        image: myapp:1.0
```

**Interviewer:** What does maxSkew: 1 mean?

**Candidate:** Difference between zones can't exceed 1. With 9 replicas and 3 zones, each zone gets 3 pods. If one zone fails, pods remain balanced across other zones. Better than plain anti-affinity - more flexible spreading.

---

### Question 44: Azure SignalR Service

**Interviewer:** Build real-time web application with WebSockets.

**Candidate:** Azure SignalR Service manages WebSocket connections at scale. Your web app doesn't handle persistent connections - SignalR does. Client connects to SignalR, your app backend sends messages through SignalR to all clients.

Use cases: Live dashboards, chat, notifications, collaborative editing, real-time gaming.

**Interviewer:** Why not self-host SignalR?

**Candidate:** Scaling WebSockets is hard - sticky sessions, distributed state. SignalR Service handles millions of connections, automatic scaling, geo-distribution. Focus on application logic, not WebSocket infrastructure.

---

### Question 45: Terraform Dynamic Blocks

**Interviewer:** Create variable number of subnets based on input.

**Candidate:**
```hcl
variable "subnets" {
  type = map(string)
  default = {
    "web"      = "10.0.1.0/24"
    "app"      = "10.0.2.0/24"
    "database" = "10.0.3.0/24"
  }
}

resource "azurerm_virtual_network" "main" {
  name                = "main-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = var.location
  resource_group_name = var.resource_group_name

  dynamic "subnet" {
    for_each = var.subnets
    content {
      name           = subnet.key
      address_prefix = subnet.value
    }
  }
}
```

**Interviewer:** Benefits over static configuration?

**Candidate:** Reusable modules - different environments define different subnet maps. No code duplication. Easy to add new subnet - just update variable.

---

### Question 46: Kubernetes Ephemeral Containers

**Interviewer:** Debug a running pod without modifying its image.

**Candidate:**
```bash
# Attach ephemeral debug container to running pod
kubectl debug mypod -it --image=busybox --target=myapp

# Now inside pod's namespace, can inspect:
ps aux
netstat -tlnp
curl localhost:8080
cat /proc/1/environ
```

**Interviewer:** How is this different from exec?

**Candidate:** `kubectl exec` only works if container has shell. Many production images are distroless - no shell, no debugging tools. Ephemeral containers add debugging tools temporarily without rebuilding image. Container disappears when you exit.

---

### Question 47: Azure Arc

**Interviewer:** Manage on-premises Kubernetes with Azure.

**Candidate:** Azure Arc brings Azure management to anywhere - on-prem Kubernetes, AWS, GCP. Install Arc agent on cluster, it appears in Azure Portal. Apply Azure Policy, monitor with Azure Monitor, deploy using GitOps.

**Interviewer:** Use case?

**Candidate:** Hybrid applications - some workloads must stay on-prem (compliance, latency), but you want consistent tooling. Arc provides single pane of glass for all clusters regardless of location.

---

### Question 48: Kubernetes Resource Bin Packing

**Interviewer:** Optimize node utilization.

**Candidate:** Cluster autoscaler uses bin packing to fit pods efficiently. Set pod requests accurately - if too high, waste resources; too low, nodes overcommit and pods get OOMKilled.

Use VPA in recommendation mode to see actual usage:
```bash
kubectl describe vpa myapp-vpa

# Shows recommended requests based on observed usage
Recommendation:
  Container Recommendations:
    Container Name: app
    Lower Bound:
      Cpu:     100m
      Memory:  256Mi
    Target:
      Cpu:     200m
      Memory:  512Mi
    Uncapped Target:
      Cpu:     250m
      Memory:  650Mi
    Upper Bound:
      Cpu:     500m
      Memory:  1Gi
```

Set requests to Target values for optimal bin packing.

---

### Question 49: Azure Network Security Groups - Flow Evaluation

**Interviewer:** NSG rules aren't working as expected. How does Azure evaluate them?

**Candidate:** Rules processed in priority order (100, 200, 300...). First matching rule wins - remaining rules ignored. Default rules at end deny all inbound, allow all outbound.

**Interviewer:** Debug example?

**Candidate:** Traffic denied unexpectedly - check for deny rule with lower priority number than intended allow rule. Common mistake: Deny rule 100, Allow rule 200 - deny wins, traffic blocked. Fix: Swap priorities or restructure rules.

---

### Question 50: GitOps Repository Structure Best Practices

**Interviewer:** Structure GitOps repository for multiple applications and environments.

**Candidate:**
```
gitops-repo/
├── apps/
│   ├── frontend/
│   │   ├── base/
│   │   │   ├── kustomization.yaml
│   │   │   ├── deployment.yaml
│   │   │   └── service.yaml
│   │   └── overlays/
│   │       ├── dev/
│   │       │   └── kustomization.yaml
│   │       ├── staging/
│   │       │   └── kustomization.yaml
│   │       └── production/
│   │           └── kustomization.yaml
│   ├── backend/
│   └── database/
├── infrastructure/
│   ├── ingress-nginx/
│   ├── cert-manager/
│   └── external-dns/
└── argocd/
    ├── applications/
    │   ├── frontend-dev.yaml
    │   ├── frontend-prod.yaml
    │   └── ...
    └── projects/
```

**Interviewer:** Why separate apps and infrastructure?

**Candidate:** Different ownership and update cadence. Infrastructure rarely changes, applications update constantly. Separate repos or folders with different RBAC. Infrastructure team owns ingress/cert-manager, app teams own their apps.

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

---

**✅ Interview Session 4 Complete - 50 Unique Scenario-Based Questions!**

**Focus:** Real-world scenarios, migration strategies, troubleshooting, advanced configurations, cost optimization, and production best practices.

**Total Questions: 50/50** | **Format:** Conversational Scenario-Based

