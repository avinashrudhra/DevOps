# DevOps Interview Session 2
## Conversational Format - Azure DevOps / Cloud / DevOps Engineer (3-7 Years)

---

### Question 1: Kubernetes Networking

**Interviewer:** Can you explain how pod-to-pod communication works in Kubernetes?

**Candidate:** Every pod gets its own IP address from the pod CIDR range. Pods can communicate directly with each other using these IPs without NAT. The Container Network Interface (CNI) plugin handles the networking - in AKS, it's typically Azure CNI or Kubenet.

**Interviewer:** What's the difference between Azure CNI and Kubenet?

**Candidate:** With Azure CNI, each pod gets an IP from the VNet subnet, making them directly routable. Kubenet uses a private IP range for pods and does NAT at the node level, which is more IP-efficient but adds a network hop.

---

### Question 2: Docker Compose vs Kubernetes

**Interviewer:** You have experience with Docker Compose. How would you explain the difference to Kubernetes?

**Candidate:** Docker Compose is for running multi-container applications on a single host - great for development and simple deployments. Kubernetes orchestrates containers across multiple nodes, handles scaling, self-healing, rolling updates, and load balancing. Compose is simpler but Kubernetes is production-grade for larger applications.

**Interviewer:** When would you still use Docker Compose?

**Candidate:** Local development, CI/CD test environments, or small single-server deployments where Kubernetes would be overkill. It's much faster to set up for testing microservices locally.

---

### Question 3: Azure Pipeline Variables

**Interviewer:** How do you handle secrets in Azure Pipelines?

**Candidate:** I use Azure Key Vault integration with variable groups. We link the variable group to Key Vault, authorize the pipeline, and reference secrets with `$(secret-name)`. They're automatically masked in logs. For less sensitive configs, I use pipeline variables marked as secret.

**Interviewer:** What if someone tries to echo a secret in the pipeline?

**Candidate:** Azure DevOps automatically masks secret variables in logs - you'll see `***` instead of the actual value. But you can't prevent someone from base64 encoding it or sending it elsewhere, so RBAC on who can edit pipelines is crucial.

---

### Question 4: Git Branching Strategy

**Interviewer:** What branching strategy have you used in your projects?

**Candidate:** We use Git Flow - `main` for production, `develop` for integration, feature branches for new work, and release branches for production prep. Feature branches are created from develop, merged back via PR with code review and automated tests.

**Interviewer:** How do you handle hotfixes?

**Candidate:** Hotfix branches come from `main`, get fixed and tested quickly, then merged to both `main` and `develop` to keep them in sync.

---

### Question 5: Terraform State Locking

**Interviewer:** Have you ever encountered a Terraform state lock issue?

**Candidate:** Yes, when a pipeline crashed mid-run and didn't release the lock. The state was locked in Azure Storage blob lease.

**Interviewer:** How did you resolve it?

**Candidate:** First verified no one else was running Terraform, then used `az storage blob lease break` to forcefully release the lock. After that, I ensured our pipeline had proper error handling to release locks on failure.

---

### Question 6: Kubernetes Resource Requests vs Limits

**Interviewer:** What's the difference between requests and limits?

**Candidate:** Requests are guaranteed resources - Kubernetes won't schedule a pod unless the node has that much available. Limits are the maximum a pod can use. If it exceeds memory limit, it gets OOMKilled. CPU limits just throttle the container.

**Interviewer:** What happens if you set limits but no requests?

**Candidate:** Requests default to limits. So if you set a high limit without a request, the pod might not get scheduled because it's asking for too much guaranteed resource even if it won't actually use it.

---

### Question 7: CI/CD Pipeline Optimization

**Interviewer:** Your Jenkins pipeline takes 20 minutes. How would you optimize it?

**Candidate:** First, I'd parallelize independent stages like unit tests, integration tests, and security scans. Then implement Docker layer caching for builds. Use incremental builds where possible. Cache dependencies - like Maven .m2 or npm node_modules. Finally, move heavy workloads to dedicated agents instead of running everything on master.

---

### Question 8: Azure Storage Account Security

**Interviewer:** How do you secure an Azure Storage Account?

**Candidate:** Disable public blob access, use private endpoints for VNet access, enable firewall rules to restrict IPs, implement RBAC instead of access keys where possible, enable soft delete for recovery, use SAS tokens with expiration for temporary access, and enable Azure Defender for threat detection.

**Interviewer:** What about encryption?

**Candidate:** Storage accounts are encrypted at rest by default with Microsoft-managed keys. For higher security requirements, we use customer-managed keys stored in Azure Key Vault.

---

### Question 9: Kubernetes Service Types

**Interviewer:** Explain the different Kubernetes service types.

**Candidate:** ClusterIP is internal-only with a virtual IP. NodePort exposes the service on each node's IP at a static port - useful for dev but not production. LoadBalancer creates a cloud load balancer - in Azure, it provisions an Azure Load Balancer. ExternalName is a DNS CNAME record mapping.

**Interviewer:** When would you use headless services?

**Candidate:** For StatefulSets where you need to connect to specific pod instances directly, like for database replicas. You set clusterIP to None, and DNS returns all pod IPs instead of load balancing.

---

### Question 10: Real Issue - Pod Pending

**Interviewer:** A pod is stuck in Pending status. Walk me through your troubleshooting.

**Candidate:** First, `kubectl describe pod` to check events. Common causes: insufficient resources on nodes, PVC not bound, node selector not matching any nodes, or taints preventing scheduling.

**Interviewer:** The event says "0/3 nodes available: insufficient memory."

**Candidate:** The pod's resource requests exceed available node capacity. I'd either reduce the pod's memory request if it's overprovisioned, scale up the node pool, or check if other pods can be scaled down or moved.

---

### Question 11: Maven Multi-Module Build

**Interviewer:** How do you speed up Maven builds in a multi-module project?

**Candidate:** Use parallel builds with `-T 1C` flag, implement proper module dependencies so only changed modules rebuild, use `-pl` to build specific modules, cache the .m2 repository in CI/CD, and skip unnecessary phases like javadoc generation in dev builds with profiles.

---

### Question 12: Docker Image Layers

**Interviewer:** Why do we care about Docker image layers?

**Candidate:** Each RUN, COPY, ADD creates a new layer. Layers are cached - if nothing changed, Docker reuses the cached layer, making builds faster. Order matters - put frequently changing files like source code after dependencies so dependency layers stay cached.

**Interviewer:** How do you minimize layers?

**Candidate:** Combine related RUN commands with `&&`, use multi-stage builds to separate build and runtime, and add a .dockerignore file to exclude unnecessary files.

---

### Question 13: Ansible vs Terraform

**Interviewer:** When would you use Ansible over Terraform?

**Candidate:** Ansible for configuration management - installing packages, configuring services, managing files on existing servers. Terraform for infrastructure provisioning - creating VMs, networks, cloud resources. They complement each other: Terraform creates the infrastructure, Ansible configures it.

**Interviewer:** Can Ansible provision infrastructure?

**Candidate:** Yes, it has cloud modules, but Terraform is better for it because of state management and better dependency handling for infrastructure.

---

### Question 14: AKS Upgrade Strategy

**Interviewer:** How do you approach upgrading an AKS cluster?

**Candidate:** First, check release notes for breaking changes. Test the upgrade in dev/staging first. Backup critical resources. Do it during a maintenance window. Upgrade node pools one at a time if using multiple pools. Monitor applications closely after upgrade. Have a rollback plan - though rolling back isn't always straightforward.

**Interviewer:** Can you roll back an AKS upgrade?

**Candidate:** Not directly. You'd need to create a new cluster with the old version and migrate workloads. That's why testing in staging and having good backup/restore procedures is critical.

---

### Question 15: Prometheus Metrics

**Interviewer:** What key metrics do you monitor in Kubernetes?

**Candidate:** Node CPU and memory usage, pod CPU and memory usage, pod restart counts, deployment availability, PV usage, API server latency, container network I/O, and application-specific metrics like request rate, error rate, and response times - the RED method.

**Interviewer:** What's the RED method?

**Candidate:** Rate, Errors, Duration. For every service, monitor request rate, error rate, and response duration. It gives a complete picture of service health.

---

### Question 16: Git Merge vs Rebase

**Interviewer:** Merge or rebase - what do you prefer and why?

**Candidate:** For feature branches, I rebase onto develop to keep history linear and clean. For merging feature branches into develop, I use merge with --no-ff to preserve the feature branch context. Never rebase public/shared branches.

**Interviewer:** Why not rebase shared branches?

**Candidate:** Rebasing rewrites commit history. If someone else pulled the branch, their history conflicts with yours, causing a mess. Rebase is only safe for local or personal feature branches.

---

### Question 17: Azure App Service vs AKS

**Interviewer:** When would you choose App Service over AKS?

**Candidate:** App Service for simpler web apps where you don't need orchestration complexity - it's fully managed PaaS. AKS when you need container orchestration, multiple microservices, complex scaling rules, or need full control over the environment. App Service is faster to set up, AKS is more flexible.

---

### Question 18: Jenkins Shared Libraries

**Interviewer:** What are Jenkins shared libraries and why use them?

**Candidate:** They're reusable Groovy code stored in a Git repo that multiple pipelines can use. Instead of duplicating code across Jenkinsfiles, you centralize common functions like deployment steps, notifications, or security scans. Makes pipelines cleaner and easier to maintain.

**Interviewer:** Give an example.

**Candidate:** We had a shared library for Docker build and push - every team just calls `dockerBuildPush()` in their pipeline instead of repeating 20 lines of Docker commands.

---

### Question 19: Kubernetes ConfigMaps vs Secrets

**Interviewer:** Besides encryption, what's the real difference between ConfigMaps and Secrets?

**Candidate:** Honestly, the main difference is intent and base64 encoding. Secrets are base64 encoded (not encrypted by default!) and have stricter RBAC. Both can be mounted as volumes or environment variables. For real encryption, you need to enable encryption at rest in etcd.

**Interviewer:** So secrets aren't really that secret?

**Candidate:** Not by default. Base64 is just encoding. In production, we use external secret managers like Azure Key Vault with the CSI driver for actual encryption.

---

### Question 20: Terraform Modules

**Interviewer:** How do you structure Terraform code for reusability?

**Candidate:** I create modules for common patterns - like a VNet module, AKS module, or storage module. Each module has its own variables, resources, and outputs. Then different environments just call these modules with environment-specific values. Keeps code DRY and consistent.

---

### Question 21: Real Issue - Container Exit Code 137

**Interviewer:** A container keeps exiting with code 137. What's happening?

**Candidate:** Exit code 137 means it was OOMKilled - the container exceeded its memory limit. The kernel killed it.

**Interviewer:** How would you fix it?

**Candidate:** Check actual memory usage with `kubectl top pod` or container insights. If it's legitimately using more memory, increase the limit. If it's a memory leak, fix the application. Also ensure requests match actual usage for proper scheduling.

---

### Question 22: Azure DevOps Agents

**Interviewer:** Microsoft-hosted vs self-hosted agents - what's your experience?

**Candidate:** Microsoft-hosted are easy, no maintenance, but limited customization and you pay per minute. Self-hosted give you control - custom tools, caching, faster builds since the environment persists. We use self-hosted for production builds and Microsoft-hosted for PR validations.

**Interviewer:** What about security?

**Candidate:** Self-hosted agents need to be patched and maintained. We run them in a dedicated VNet with restricted access. For sensitive workloads, self-hosted in a secure environment is better than Microsoft-hosted.

---

### Question 23: Docker Networking Modes

**Interviewer:** Explain Docker network types.

**Candidate:** Bridge is default - containers get IPs on a private network and can talk to each other. Host mode shares the host's network stack - no isolation but better performance. None has no networking. Overlay is for Swarm multi-host networking.

**Interviewer:** When would you use host mode?

**Candidate:** For performance-critical applications where the network overhead matters, but you lose port isolation. Or when the container needs to see the host's network interfaces directly.

---

### Question 24: Kubernetes Namespaces Strategy

**Interviewer:** How do you use namespaces in production?

**Candidate:** We separate by environment and application - like `production-frontend`, `production-backend`, `staging-frontend`, etc. Each namespace has its own resource quotas, network policies, and RBAC. Keeps teams isolated and prevents resource hogging.

**Interviewer:** Why not just `production` namespace?

**Candidate:** Too broad. If one app has a memory leak, it could starve others. Separate namespaces with quotas contain the blast radius. Also easier RBAC - frontend team only accesses frontend namespace.

---

### Question 25: SonarQube Quality Gates

**Interviewer:** How do you enforce code quality with SonarQube?

**Candidate:** We have a quality gate that fails the pipeline if coverage drops below 80%, new code has any critical vulnerabilities, or code smells exceed threshold. It's integrated in the CI pipeline - code can't be merged to develop without passing the gate.

**Interviewer:** What if someone needs to bypass it urgently?

**Candidate:** We have an exception process requiring tech lead approval and a follow-up task to fix it within a sprint. But it's rare - usually means the standards are too strict or there's a legitimate edge case.

---

### Question 26: Azure Key Vault Integration

**Interviewer:** How do you use Key Vault in AKS?

**Candidate:** We use the CSI driver to mount secrets as volumes in pods. The pod's managed identity authenticates to Key Vault, secrets get mounted at pod startup. This way secrets never touch CI/CD or Kubernetes etcd - they're pulled at runtime.

**Interviewer:** What about secret rotation?

**Candidate:** The CSI driver can poll Key Vault for updates. When a secret rotates, the mount updates automatically. The application needs to re-read the file, or we do a rolling restart.

---

### Question 27: Helm Chart Values

**Interviewer:** How do you manage Helm values across environments?

**Candidate:** We have a base `values.yaml` with defaults, then environment-specific values files like `values-dev.yaml`, `values-prod.yaml`. Deploy with `helm install -f values-prod.yaml`. Production has higher replicas, different resource limits, and production-specific configs.

---

### Question 28: Git Conflicts Resolution

**Interviewer:** You have a complex merge conflict. How do you approach it?

**Candidate:** First, understand what both sides are trying to do. Use `git diff` to see the changes. For complex conflicts, I use a merge tool like VS Code or Beyond Compare. If it's too messy, I talk to the other developer - sometimes it's easier to pair on it. Always test after resolving to ensure nothing broke.

---

### Question 29: Container Resource Limits Best Practices

**Interviewer:** How do you determine resource limits for a new application?

**Candidate:** Start by monitoring it in dev without limits to see actual usage. Then set requests to average usage and limits to 1.5-2x that for bursts. Monitor in production and adjust. Guaranteed QoS needs requests equal to limits, but that's usually overkill.

---

### Question 30: Jenkins Pipeline Error Handling

**Interviewer:** How do you handle failures in Jenkins pipelines?

**Candidate:** Use try-catch blocks for recoverable errors, post blocks for cleanup - `always` runs regardless, `failure` only on failure, `success` only on success. For critical failures, we send Slack notifications with logs and fail the build. Non-critical issues log warnings but don't fail.

---

### Question 31: Kubernetes Ingress Controllers

**Interviewer:** Which Ingress controller have you used and why?

**Candidate:** Nginx Ingress Controller - it's widely supported, lots of annotations for customization, handles path-based routing and SSL termination well. In Azure, Application Gateway Ingress Controller integrates better with Azure services but is more complex.

**Interviewer:** What about SSL certificates?

**Candidate:** We use cert-manager with Let's Encrypt for automatic SSL certificate generation and renewal. It watches Ingress resources and provisions certificates automatically.

---

### Question 32: Azure Pipeline Templates

**Interviewer:** What are pipeline templates and when do you use them?

**Candidate:** Templates are reusable YAML files that multiple pipelines can reference. We have templates for common patterns like Docker build, Kubernetes deploy, SonarQube scan. Teams reference these templates instead of duplicating code, and updates propagate automatically.

---

### Question 33: Docker Volume Management

**Interviewer:** How do you handle persistent data in Docker?

**Candidate:** Named volumes for production - Docker manages them and they persist after container removal. Bind mounts for development to sync code changes. For databases, always use volumes. In Kubernetes, we use PersistentVolumes instead.

**Interviewer:** What about volume backups?

**Candidate:** For Docker, we backup volume data to Azure Storage. For Kubernetes PVs, we use Velero for cluster backup including volumes, or snapshot the underlying Azure disks.

---

### Question 34: Terraform Workspaces vs Directories

**Interviewer:** How do you manage multiple environments in Terraform?

**Candidate:** I prefer separate directories per environment - `environments/dev`, `environments/prod`. Clearer separation, different state files, less risk of accidentally destroying prod. Workspaces work but can be confusing when switching contexts.

---

### Question 35: Real Issue - Slow Kubernetes API

**Interviewer:** The Kubernetes API server is responding slowly. How do you troubleshoot?

**Candidate:** Check API server logs for errors, look at etcd performance since API server queries etcd, check if there's excessive watch requests or list operations without selectors. Also monitor API server CPU and memory.

**Interviewer:** What could cause excessive API calls?

**Candidate:** Badly written controllers or operators polling frequently, applications listing all pods repeatedly, or too many CRDs. We'd identify the client with high request rates in API audit logs and optimize it.

---

### Question 36: Python for DevOps Automation

**Interviewer:** Give me an example of a Python script you wrote for automation.

**Candidate:** I wrote a script to check Azure VMs for unattached disks and alert on unused resources. It uses the Azure SDK to list all disks, checks if they're attached, calculates their cost, and sends a weekly report to help reduce cloud spend.

---

### Question 37: Maven Dependency Conflicts

**Interviewer:** How do you resolve Maven dependency conflicts?

**Candidate:** Use `mvn dependency:tree` to see the dependency graph, identify where the conflict is coming from. Then use `<exclusions>` to exclude the unwanted version and explicitly declare the version you want in `<dependencyManagement>`.

---

### Question 38: Azure Managed Identity

**Interviewer:** What's a managed identity and why use it?

**Candidate:** It's an Azure AD identity automatically managed by Azure. Instead of storing credentials, your VM or App Service gets an identity that can authenticate to Azure services. No secrets to manage or rotate - Azure handles it.

**Interviewer:** System-assigned vs user-assigned?

**Candidate:** System-assigned is tied to the resource lifecycle - it's created and deleted with the resource. User-assigned is independent, can be shared across resources, and persists even if resources are deleted. I use user-assigned when multiple resources need the same identity.

---

### Question 39: Kubernetes HPA with Custom Metrics

**Interviewer:** HPA with CPU is basic. Have you used custom metrics?

**Candidate:** Yes, we autoscale based on HTTP requests per second using the metrics-server and custom metrics API. For example, scale up when requests exceed 100/sec per pod. More relevant than CPU for our web apps.

---

### Question 40: GitLab CI vs Azure Pipelines

**Interviewer:** You've used both. What are the differences?

**Candidate:** GitLab CI is in the same platform as your code, tighter integration, simpler setup. Azure Pipelines integrates better with Azure services, more enterprise features, better Windows support. GitLab's `.gitlab-ci.yml` is simpler than Azure's YAML. Both work well - depends on your ecosystem.

---

### Question 41: Docker Build Cache Optimization

**Interviewer:** Your Docker builds are slow. How do you optimize?

**Candidate:** Order Dockerfile instructions from least to most frequently changing. Copy package files first, install dependencies, then copy source code. Use BuildKit with `--cache-from` to reuse layers from previous builds. In CI, pull the latest image to use as cache.

---

### Question 42: Kubernetes Rolling Update

**Interviewer:** Explain how rolling updates work.

**Candidate:** Kubernetes gradually replaces old pods with new ones. MaxUnavailable controls how many can be down during update, maxSurge controls how many extra can be created. New pods start, pass readiness probes, then old pods terminate. If new pods fail health checks, rollout pauses.

**Interviewer:** How do you rollback?

**Candidate:** `kubectl rollout undo deployment/myapp` reverts to previous revision. Kubernetes keeps rollout history by default for rollbacks.

---

### Question 43: Azure Storage Redundancy

**Interviewer:** Explain storage redundancy options in Azure.

**Candidate:** LRS replicates 3 times in one datacenter - cheapest but single location. ZRS replicates across availability zones in one region. GRS replicates to a secondary region. RA-GRS does geo-replication and allows reading from secondary. Choice depends on RTO/RPO requirements and cost.

---

### Question 44: Jenkins Credentials Management

**Interviewer:** How do you manage credentials in Jenkins securely?

**Candidate:** Use the Credentials plugin to store secrets encrypted in Jenkins. Reference them by ID in pipelines - they're masked in logs. For Azure, we use service principals with certificates, not passwords. We also integrate Jenkins with Azure Key Vault for centralized secret management.

---

### Question 45: Kubernetes Network Policies

**Interviewer:** What are Network Policies?

**Candidate:** They're firewall rules for pods. By default, all pods can talk to each other. Network Policies restrict traffic based on labels - for example, only allow frontend pods to talk to backend on port 8080. Essential for zero-trust security.

**Interviewer:** How do you test them?

**Candidate:** Create a test pod and try to curl the target service. If the policy works, the request times out. Always test both allowed and denied traffic to confirm the policy is correct.

---

### Question 46: Terraform Import

**Interviewer:** When do you use `terraform import`?

**Candidate:** When you have existing infrastructure not managed by Terraform and want to bring it under management. You write the Terraform code, then import the real resource into state. After that, Terraform manages it. Common after manual resource creation or when adopting Terraform mid-project.

---

### Question 47: Azure DevOps Branch Policies

**Interviewer:** What branch policies do you enforce?

**Candidate:** Require minimum two reviewers, require linked work items, require build validation, and don't allow commits directly to main - only PRs. This ensures code quality and maintains audit trail.

---

### Question 48: Real Issue - Certificate Expiration

**Interviewer:** Your SSL certificate expired and the site went down. How do you prevent this?

**Candidate:** Use cert-manager with Let's Encrypt for automatic 90-day renewals. Set up monitoring alerts 30 days before expiration. For purchased certificates, use Azure Key Vault with notifications. Also document renewal procedures and ownership.

---

### Question 49: Linux Performance Tuning

**Interviewer:** A Linux server has high CPU iowait. What does that mean and how do you investigate?

**Candidate:** High iowait means CPU is idle waiting for I/O operations - usually disk. I'd check with `iostat` to see which disk is saturated, use `iotop` to find which process is doing heavy I/O. Could be database queries, log writes, or disk being too slow. Solution depends on the cause - optimize queries, add indexes, or upgrade to SSD.

---

### Question 50: DevOps Culture Question

**Interviewer:** What does DevOps mean to you beyond tools?

**Candidate:** DevOps is about breaking silos between dev and ops, shared ownership of production, automation to reduce manual errors, and continuous improvement. It's measuring everything, failing fast, learning from failures, and treating infrastructure as code. Tools enable it, but culture and collaboration are what make it work.

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)



