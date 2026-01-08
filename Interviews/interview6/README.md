# DevOps Interview Session 6
## Production Issues & Troubleshooting (3-7 Years)

---

### Question 1: High CPU on Kubernetes Node

**Interviewer:** A Kubernetes node is showing 100% CPU. How do you investigate?

**Candidate:** First, check which pods are using CPU with `kubectl top pods --all-namespaces`. Then drill into the problematic pod with `kubectl top pod <pod-name> --containers`. Check if it's a legitimate spike or runaway process. Look at pod logs for errors. If it's a resource-intensive task, consider increasing CPU limits or scaling horizontally.

```bash
# Node CPU usage
kubectl top nodes

# Pods on that node
kubectl get pods --all-namespaces -o wide --field-selector spec.nodeName=node-1

# Pod CPU breakdown
kubectl top pods -n production --sort-by=cpu

# Describe pod for limits
kubectl describe pod high-cpu-pod

# Check for throttling
kubectl get --raw /apis/metrics.k8s.io/v1beta1/namespaces/production/pods/high-cpu-pod
```

**Interviewer:** What if it's a memory leak causing CPU thrashing?

**Candidate:** Check memory usage alongside CPU. Memory leak leads to swapping which spikes CPU. Look at `kubectl top` for memory, check OOM events in `kubectl describe pod`. Profile the application to find leak source.

---

### Question 2: Git Branch Protection Not Working

**Interviewer:** You set branch protection on main but developers can still push directly.

**Candidate:** Check if they're repository admins - admins bypass protection by default. Also verify protection rules are saved correctly - go to repository Settings → Branches → Branch protection rules. Make sure the pattern matches exactly (`main` not `master`). Check if "Include administrators" is unchecked - that's why admins bypass it.

**Interviewer:** How do you enforce it for everyone?

**Candidate:** Enable "Include administrators" in protection rules. For GitHub Enterprise, use organization-level policies. For Azure DevOps, use branch policies and ensure no one has "Exempt from policy enforcement" permission.

---

### Question 3: Maven Build Works Locally, Fails in CI

**Interviewer:** Developer says "it works on my machine" but CI build fails.

**Candidate:** Classic issue. Check Java version - `mvn -version` locally vs CI. Check Maven version. Look at CI logs for actual error. Often it's dependencies - local has cached snapshots, CI fetches fresh ones. Or tests rely on local files/database not in CI. Environmental differences like timezone, locale can affect tests too.

```bash
# CI pipeline debugging
- script: |
    java -version
    mvn -version
    echo "JAVA_HOME: $JAVA_HOME"
    mvn clean package -X  # Debug mode
```

**Interviewer:** Found it - tests fail due to database connection.

**Candidate:** Tests need proper setup. Either mock the database, use testcontainers for real DB in Docker, or configure CI to provide test database. Never have tests depend on local dev database.

---

### Question 4: Docker Image Pull Timeout

**Interviewer:** Kubernetes pods stuck in ImagePullBackOff with timeout errors.

**Candidate:** Network issue between nodes and registry. Check node can reach registry - `curl` the registry URL from node. Check if it's Docker Hub rate limits - switch to authenticated pulls or use Azure Container Registry. Verify image name is correct. Check if registry credentials are valid in imagePullSecrets.

```bash
# Debug from node
kubectl debug node/worker-1 -it --image=busybox
# Then test:
nslookup myregistry.azurecr.io
wget https://myregistry.azurecr.io/v2/

# Check events
kubectl get events --sort-by='.lastTimestamp' | grep Pull

# Verify secret
kubectl get secret acr-secret -o yaml
```

---

### Question 5: SonarQube Analysis Taking Too Long

**Interviewer:** SonarQube scan in pipeline takes 30 minutes, slowing releases.

**Candidate:** Check what's being scanned - exclude test files, generated code, vendor directories. Use incremental analysis for pull requests - only scan changed files. Increase SonarQube server resources if overloaded. Run analysis only on main branch and PRs, skip feature branches.

```yaml
# Optimized scanner properties
sonar.sources=src
sonar.exclusions=**/test/**,**/vendor/**,**/*.generated.js
sonar.coverage.exclusions=**/test/**

# PR analysis - faster
sonar.pullrequest.key=${PR_NUMBER}
sonar.pullrequest.branch=${BRANCH_NAME}
sonar.pullrequest.base=main

# Skip for feature branches
steps:
  - script: |
      if [[ "$BRANCH_NAME" == "main" ]] || [[ "$BRANCH_NAME" == PR-* ]]; then
        sonar-scanner
      else
        echo "Skipping SonarQube for feature branch"
      fi
```

---

### Question 6: Application Can't Connect to Database

**Interviewer:** Application deployed, logs show "Connection refused" to database.

**Candidate:** Check if database pod is running - `kubectl get pods`. Verify service exists - `kubectl get svc`. Test DNS resolution from app pod - `nslookup database-service`. Check if database is listening on expected port. Verify connection string uses correct service name and port. Check network policies aren't blocking traffic.

```bash
# Debug steps
kubectl exec -it app-pod -- sh

# Inside pod:
nslookup database-service  # DNS works?
ping database-service  # Network works?
nc -zv database-service 5432  # Port open?
env | grep DATABASE  # Connection string correct?

# Check database pod
kubectl logs database-pod
kubectl exec -it database-pod -- psql -U postgres -c "\l"
```

**Interviewer:** Database is running but still refused.

**Candidate:** Check database is listening on 0.0.0.0 not just localhost. PostgreSQL's `listen_addresses` must be '*'. Check pg_hba.conf allows connections. Verify credentials are correct.

---

### Question 7: Terraform State Locked, Team Blocked

**Interviewer:** Your team can't run Terraform, state is locked from yesterday's failed apply.

**Candidate:** Identify who has the lock with error message details. Check if that pipeline is actually still running. If not, force unlock with `terraform force-unlock <lock-id>`. For Azure backend, can break the blob lease with Azure CLI. Important - only force unlock if absolutely sure no one is running Terraform.

```bash
# Check lock info
terraform plan
# Shows lock ID and who locked it

# Force unlock (BE CAREFUL)
terraform force-unlock abc-123-def-456

# Azure Storage backend
az storage blob lease break \
  --blob-name prod.terraform.tfstate \
  --container-name tfstate \
  --account-name tfstateaccount

# Prevent future locks - add in pipeline
trap 'terraform force-unlock -force $LOCK_ID' EXIT ERR
```

---

### Question 8: Jenkins Out of Disk Space

**Interviewer:** Jenkins failing with "No space left on device".

**Candidate:** Check disk usage with `df -h`. Usually `/var/lib/jenkins` is full. Old builds, workspaces, and archived artifacts accumulate. Clean up - discard old builds, clean workspaces, move artifacts to external storage like Azure Blob.

```bash
# Find space hogs
du -sh /var/lib/jenkins/* | sort -rh | head -10

# Clean up
# 1. Configure job retention
# In job config → Discard Old Builds → Keep last 10 builds

# 2. Clean workspaces
# Add to pipeline:
post {
    always {
        cleanWs()
    }
}

# 3. Manual cleanup
find /var/lib/jenkins/workspace -type d -mtime +7 -exec rm -rf {} +
docker system prune -a --volumes -f  # If using Docker agents

# 4. Move artifacts externally
# Instead of archiveArtifacts, upload to Azure Storage
```

---

### Question 9: Load Balancer Not Distributing Traffic

**Interviewer:** Azure Load Balancer created but all traffic goes to one backend.

**Candidate:** Check health probes - if backend fails probe, it's removed from pool. Verify probe path is correct and returns 200. Check load balancing rule is configured properly. Look at session affinity - if enabled, same client goes to same backend. Review backend pool - ensure multiple VMs are actually added.

```bash
# Check health probe status
az network lb show --name myLB --resource-group myRG \
  --query "backendAddressPools[0].backendIPConfigurations[].id"

# Check probe configuration
az network lb probe show \
  --lb-name myLB \
  --name healthProbe \
  --resource-group myRG

# Test probe endpoint
curl http://backend-vm:8080/health
```

---

### Question 10: Python Script Permission Denied

**Interviewer:** Cron job fails with "Permission denied" but script works when run manually.

**Candidate:** Cron runs with different user/environment. Check script has execute permission - `chmod +x script.py`. Use absolute paths in script, not relative. Specify Python interpreter explicitly in shebang - `#!/usr/bin/env python3`. In crontab, provide full path to script and python.

```bash
# Crontab entry
0 2 * * * /usr/bin/python3 /home/user/scripts/backup.py >> /var/log/backup.log 2>&1

# Or make script executable
chmod +x backup.py
0 2 * * * /home/user/scripts/backup.py >> /var/log/backup.log 2>&1

# Check cron logs
grep CRON /var/log/syslog
```

**Interviewer:** Script needs environment variables from .bashrc.

**Candidate:** Source environment in crontab or script. Cron doesn't load user's shell rc files.

```bash
# In crontab
0 2 * * * . $HOME/.bashrc; /home/user/scripts/backup.py

# Or in script itself
import os
os.environ['PATH'] = '/usr/local/bin:/usr/bin:/bin'
```

---

### Question 11: Helm Release Stuck in Pending-Upgrade

**Interviewer:** Helm upgrade failed mid-way, now stuck. Can't upgrade or rollback.

**Candidate:** Release is in limbo. Check release status with `helm list -a`. Look at release history - `helm history myapp`. Delete the pending release secret manually, then retry or rollback.

```bash
# Check status
helm list -n production -a

# View history
helm history myapp -n production

# Find pending secret
kubectl get secrets -n production | grep myapp

# Delete pending secret
kubectl delete secret sh.helm.release.v1.myapp.v5.pending -n production

# Now can rollback
helm rollback myapp 4 -n production

# Or force upgrade
helm upgrade myapp ./chart --force -n production
```

---

### Question 12: Ansible Playbook Hangs

**Interviewer:** Ansible playbook stops responding, no output.

**Candidate:** Usually waiting for sudo password or SSH key passphrase. Run with `-vvv` for verbose output to see where it's stuck. Check if `become` is used without proper sudo setup. Verify SSH keys are added to agent. Increase timeout if network is slow.

```bash
# Debug mode
ansible-playbook playbook.yml -vvv

# If waiting for sudo password
ansible-playbook playbook.yml --ask-become-pass

# SSH issues
ssh-add ~/.ssh/id_rsa
ansible-playbook playbook.yml

# Increase timeout
ansible-playbook playbook.yml -e "ansible_timeout=60"

# Test connection first
ansible all -m ping -i inventory.ini
```

---

### Question 13: ArgoCD App OutOfSync But Looks Identical

**Interviewer:** ArgoCD shows OutOfSync but Git and cluster look the same.

**Candidate:** Check ignored differences - some fields like replica count (managed by HPA) should be ignored. Look at status messages in ArgoCD UI for details. Could be a generated field like timestamps. Configure ArgoCD to ignore specific fields.

```yaml
# Application ignoreDifferences
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
spec:
  ignoreDifferences:
  - group: apps
    kind: Deployment
    jsonPointers:
    - /spec/replicas  # HPA manages this
  - kind: Secret
    jsonPointers:
    - /data  # Ignore secret data changes
```

**Interviewer:** Still OutOfSync after adding ignores.

**Candidate:** Hard refresh in ArgoCD UI or `argocd app get myapp --hard-refresh`. Could be a resource created manually not in Git - ArgoCD sees it as extra. Either add to Git or let ArgoCD prune it.

---

### Question 14: Prometheus Scrape Timeout

**Interviewer:** Prometheus targets showing UP but some metrics missing, logs show scrape timeout.

**Candidate:** Target responding too slowly. Check target's `/metrics` endpoint response time - should be under 10 seconds. Increase scrape timeout in Prometheus config if legitimate slow endpoint. Optimize metrics endpoint - reduce metric cardinality, remove unnecessary metrics.

```yaml
# Increase timeout
scrape_configs:
- job_name: 'slow-app'
  scrape_interval: 30s
  scrape_timeout: 15s  # Increased from 10s
  static_configs:
  - targets: ['app:9090']

# Check endpoint performance
time curl http://app:9090/metrics

# Reduce cardinality in app
# Instead of per-user metrics (million labels), aggregate by user tier
```

---

### Question 15: GitHub Actions Workflow Pending Forever

**Interviewer:** GitHub Actions workflow stuck in "Queued" status for hours.

**Candidate:** No available runners. Check organization's runner capacity. For self-hosted runners, check if runner service is up. Free tier has concurrent job limits. Could be waiting for required check from another workflow. Review workflow dependencies.

```yaml
# Check runner status in Settings → Actions → Runners

# Workflow waiting on another workflow
needs: [build]  # Waiting for 'build' job

# Use matrix to parallelize if hitting concurrency limits
strategy:
  matrix:
    shard: [1, 2, 3, 4]

# Or use self-hosted runners
runs-on: self-hosted
```

---

### Question 16: Docker Build Cache Not Working

**Interviewer:** Every Docker build downloads all dependencies again, ignoring cache.

**Candidate:** Layer invalidated by changes before dependency installation. Move dependency file copy before source code copy. Ensure COPY commands are in correct order. Use `.dockerignore` to exclude files that change frequently like logs.

```dockerfile
# Bad - cache breaks on any file change
COPY . /app
RUN npm install

# Good - cache works unless package.json changes
COPY package*.json /app/
RUN npm install
COPY . /app

# .dockerignore
node_modules
npm-debug.log
*.log
.git
```

**Interviewer:** Using BuildKit?

**Candidate:** Enable BuildKit for better caching - `DOCKER_BUILDKIT=1 docker build`. Use cache mounts for package managers:

```dockerfile
# Cache npm packages
RUN --mount=type=cache,target=/root/.npm npm install
```

---

### Question 17: Kubernetes DNS Resolution Intermittent

**Interviewer:** Pods sometimes can resolve service names, sometimes can't.

**Candidate:** CoreDNS pod issues. Check CoreDNS pods are healthy - `kubectl get pods -n kube-system -l k8s-app=kube-dns`. Look for errors in CoreDNS logs. Could be resource limits too low - scale up CoreDNS or increase resources. Check if ndots is causing issues with search domains.

```bash
# Check CoreDNS
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns

# Scale CoreDNS
kubectl scale deployment coredns -n kube-system --replicas=3

# Check pod's resolv.conf
kubectl exec myapp-pod -- cat /etc/resolv.conf

# Test DNS
kubectl run test --rm -it --image=busybox -- nslookup myservice
```

**Interviewer:** ndots issue?

**Candidate:** High ndots value causes many DNS queries. Pod tries all search domains before resolving. Use FQDN in applications to skip search - `myservice.namespace.svc.cluster.local` instead of just `myservice`.

---

### Question 18: Maven Central Repository Unreachable

**Interviewer:** Build fails with "Could not resolve dependencies" for Maven Central.

**Candidate:** Network issue or Maven Central is down. Configure mirror or use corporate repository manager like Artifactory. Check if proxy settings are correct. Add repository mirror in settings.xml.

```xml
<!-- settings.xml -->
<settings>
  <mirrors>
    <mirror>
      <id>company-nexus</id>
      <mirrorOf>central</mirrorOf>
      <url>https://nexus.company.com/repository/maven-public/</url>
    </mirror>
  </mirrors>
  
  <!-- If behind proxy -->
  <proxies>
    <proxy>
      <id>company-proxy</id>
      <active>true</active>
      <protocol>http</protocol>
      <host>proxy.company.com</host>
      <port>8080</port>
    </proxy>
  </proxies>
</settings>
```

---

### Question 19: Terraform Plan Shows Unexpected Changes

**Interviewer:** Running `terraform plan` shows it wants to recreate resources that shouldn't change.

**Candidate:** State drift - someone modified resources outside Terraform. Or Terraform version mismatch. Check what changed - plan output shows difference. If manual changes, import them to state or revert manual changes. Ensure team uses same Terraform version.

```bash
# See what's changing
terraform plan -out=plan.tfplan
terraform show plan.tfplan

# Check state
terraform state list
terraform state show azurerm_virtual_machine.example

# Refresh state without applying
terraform refresh

# If resources manually modified, import
terraform import azurerm_virtual_machine.example /subscriptions/xxx/resourceGroups/xxx/providers/Microsoft.Compute/virtualMachines/vm1
```

---

### Question 20: Linux Server Time Wrong

**Interviewer:** Server time is 5 hours off, causing authentication failures.

**Candidate:** Fix timezone and NTP. Check current time with `date`, set timezone with `timedatectl set-timezone`, enable NTP synchronization. For containers, mount host timezone or set TZ environment variable.

```bash
# Check time and timezone
date
timedatectl

# Set timezone
sudo timedatectl set-timezone America/New_York

# Enable NTP
sudo timedatectl set-ntp true

# Check NTP sync status
timedatectl status

# Force NTP sync
sudo systemctl restart systemd-timesyncd

# For containers
docker run -e TZ=America/New_York myapp
# Or mount
docker run -v /etc/localtime:/etc/localtime:ro myapp
```

---

### Question 21: SonarQube Quality Gate Fails on Coverage

**Interviewer:** Coverage is 79.8% but quality gate requires 80%, build fails.

**Candidate:** Either increase coverage or adjust gate threshold. Check if coverage excludes test files properly. Review which files are bringing down coverage - SonarQube shows this. Could mark generated code as excluded.

```properties
# sonar-project.properties
sonar.coverage.exclusions=**/test/**,**/generated/**,**/*Config.java

# Or adjust quality gate
# SonarQube UI → Quality Gates → Edit
# Coverage on New Code: 80% → 75%
```

**Interviewer:** Coverage should be 80%. How to improve?

**Candidate:** Write tests for uncovered code. Focus on new code first - SonarQube tracks coverage on new code separately. Review coverage report to find untested branches and methods.

---

### Question 22: Grafana Dashboard Shows No Data

**Interviewer:** Grafana dashboard was working, now shows "No data".

**Candidate:** Check data source connection - Configuration → Data Sources → Test. Verify Prometheus has data - query directly in Prometheus UI. Check time range in Grafana - might be looking at wrong time period. Review query syntax - Prometheus queries are case-sensitive.

```promql
# Test query in Prometheus UI first
up{job="myapp"}

# Check metric exists
{__name__=~"myapp.*"}

# If no data, check service discovery
# Prometheus → Status → Service Discovery
```

**Interviewer:** Prometheus shows data but Grafana doesn't.

**Candidate:** Data source configuration wrong. Verify URL is correct - `http://prometheus:9090` not `prometheus:9090`. Check if Grafana can reach Prometheus - network policy or firewall issue.

---

### Question 23: Git Merge Conflict in Binary File

**Interviewer:** Merge conflict in a binary file like .jar or image.

**Candidate:** Can't merge binary files. Pick one version - ours or theirs. Usually keep newer version or rebuild from source instead of committing binaries.

```bash
# Take ours
git checkout --ours path/to/file.jar
git add path/to/file.jar

# Take theirs
git checkout --theirs path/to/file.jar
git add path/to/file.jar

# Better solution: Don't commit binaries
# Add to .gitignore
*.jar
*.zip
*.tar.gz
dist/
build/
```

---

### Question 24: Kubernetes Ingress 502 Bad Gateway

**Interviewer:** Ingress configured, but accessing domain returns 502.

**Candidate:** Backend service isn't healthy. Check if pods are running and ready. Verify service endpoints exist - `kubectl get endpoints`. Test service directly from within cluster. Check ingress controller logs for errors.

```bash
# Check backend pods
kubectl get pods -l app=myapp

# Check service and endpoints
kubectl get svc myapp
kubectl get endpoints myapp  # Should have IPs

# Test from within cluster
kubectl run test --rm -it --image=busybox -- wget -O- http://myapp

# Check ingress controller logs
kubectl logs -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx

# Describe ingress
kubectl describe ingress myapp-ingress
```

---

### Question 25: Docker Permission Denied Error

**Interviewer:** User gets "permission denied" when running docker commands.

**Candidate:** User not in docker group. Add user to group or use sudo. For rootless Docker, ensure setup is complete.

```bash
# Add user to docker group
sudo usermod -aG docker $USER

# Log out and back in for group to take effect
# Or:
newgrp docker

# Verify
groups

# Check Docker socket permissions
ls -l /var/run/docker.sock
# Should be owned by docker group

# If socket permissions wrong
sudo chown root:docker /var/run/docker.sock
sudo chmod 660 /var/run/docker.sock
```

---

### Question 26: Jenkins Plugin Compatibility Issue

**Interviewer:** After Jenkins upgrade, some pipelines fail with plugin errors.

**Candidate:** Plugin incompatible with new Jenkins version. Check plugin manager for warnings. Update plugins or downgrade Jenkins. Test in staging Jenkins first before upgrading production.

```groovy
// Check plugin version in Jenkinsfile
@Library('shared-library@v1.0') _  // Pin version

// Pipeline compatibility
// In Manage Jenkins → System Information
// Check Jenkins version and plugin versions
```

**Interviewer:** How to prevent this?

**Candidate:** Use Plugin Manager's compatibility checker before upgrade. Test upgrades in non-prod Jenkins. Pin plugin versions with Plugin Installation Manager Tool. Have rollback plan - backup Jenkins home before upgrade.

---

### Question 27: Ansible Connection Timeout

**Interviewer:** Ansible playbook fails immediately with "Failed to connect to the host via ssh".

**Candidate:** SSH key issue or host unreachable. Verify SSH key is correct and added to target. Check target IP/hostname is right. Ensure SSH service is running on target. Try manual SSH first.

```bash
# Test SSH manually
ssh -i ~/.ssh/id_rsa user@target-host

# Ansible with verbose output
ansible-playbook -i inventory playbook.yml -vvv

# Check inventory
ansible-inventory -i inventory --list

# Test ping
ansible all -m ping -i inventory

# Specify SSH key
ansible-playbook playbook.yml --private-key=~/.ssh/id_rsa

# If SSH on non-standard port
# In inventory:
[webservers]
web1 ansible_host=192.168.1.10 ansible_port=2222
```

---

### Question 28: Helm Chart Values Not Applied

**Interviewer:** Changed values.yaml but deployed application doesn't reflect changes.

**Candidate:** Need to upgrade release, not just edit values file. Use `helm upgrade` to apply changes. Or specified wrong values file during install. Check installed values with `helm get values`.

```bash
# Check current values
helm get values myapp -n production

# Upgrade with new values
helm upgrade myapp ./mychart -n production -f values-prod.yaml

# Or set values inline
helm upgrade myapp ./mychart --set image.tag=v2.0

# Verify upgrade
helm history myapp -n production
```

---

### Question 29: Prometheus High Memory Usage

**Interviewer:** Prometheus pod using 8GB memory and growing.

**Candidate:** Too much data being scraped or retained. Reduce retention time, decrease scrape frequency, or limit targets. Check for metrics with high cardinality - labels with many unique values.

```yaml
# Reduce retention
args:
- --storage.tsdb.retention.time=15d  # From 30d
- --storage.tsdb.retention.size=50GB

# Reduce scrape frequency
scrape_configs:
- job_name: 'myapp'
  scrape_interval: 30s  # From 15s

# Find high cardinality metrics in Prometheus UI
# Status → TSDB Status → Top 10 series
```

**Interviewer:** Which metrics to remove?

**Candidate:** User-specific or very granular metrics. Aggregate at collection time instead of Prometheus. Use recording rules to pre-aggregate expensive queries.

---

### Question 30: Azure VM Can't Access Internet

**Interviewer:** VM created but can't access internet, can't install packages.

**Candidate:** Check if VM has public IP or NAT Gateway. Verify NSG allows outbound traffic. Check route table - traffic needs route to internet. For VMs without public IP, need NAT Gateway or Azure Firewall.

```bash
# Check NSG rules
az network nsg show --resource-group myRG --name myNSG --query "securityRules[?direction=='Outbound']"

# Check VM NIC
az network nic show --resource-group myRG --name myNIC --query "ipConfigurations[0].publicIPAddress"

# Check effective routes
az network nic show-effective-route-table --resource-group myRG --name myNIC

# Add NAT Gateway if needed
az network nat gateway create \
  --resource-group myRG \
  --name myNATGateway \
  --public-ip-addresses myPublicIP

az network vnet subnet update \
  --resource-group myRG \
  --vnet-name myVNet \
  --name mySubnet \
  --nat-gateway myNATGateway
```

---

### Question 31-50: Additional Troubleshooting Scenarios

**Q31:** Container keeps restarting, logs empty?
**A:** Check `kubectl logs --previous` for previous container logs. Likely crash before logging initialized. Use `kubectl describe pod` for exit codes.

**Q32:** GitHub webhook not triggering build?
**A:** Check webhook delivery history in GitHub repo settings. Verify webhook URL is accessible from internet. Check Jenkins job has "GitHub hook trigger" enabled.

**Q33:** Maven dependency version conflict?
**A:** Use `mvn dependency:tree` to see conflict. Add `<exclusions>` to remove unwanted transitive dependency. Or explicitly declare version in `<dependencyManagement>`.

**Q34:** Docker volume not persisting data?
**A:** Check volume mount path matches where app writes data. Named volume vs bind mount confusion. Verify volume permissions - container user might not have write access.

**Q35:** Terraform destroys resources unexpectedly?
**A:** Resource renamed in code without `moved` block. Check plan carefully before apply. Use `prevent_destroy` lifecycle for critical resources.

**Q36:** Kubernetes pod can't write to mounted volume?
**A:** Volume permissions issue. Set `fsGroup` in pod security context or change volume ownership with initContainer.

**Q37:** Jenkins build hangs at "Checking out git repository"?
**A:** Git credential issue or large repository. Check credentials are valid. Use shallow clone with `depth: 1` for large repos.

**Q38:** ArgoCD won't sync with "permission denied"?
**A:** Service account lacks RBAC permissions. Grant necessary roles in target namespace. Check ArgoCD service account in `argocd` namespace.

**Q39:** Ansible playbook works but idempotency fails?
**A:** Task always reports "changed" even when nothing changed. Use proper modules instead of shell commands. Modules like `copy`, `template` are idempotent.

**Q40:** Prometheus alerts not firing?
**A:** Check alert rule syntax in PrometheusRule. Verify Alertmanager is configured and reachable. Check alert labels match routing rules.

**Q41:** Linux service fails to start after reboot?
**A:** Service dependencies not met. Fix with `After=` in systemd unit. Or service not enabled - `systemctl enable service`.

**Q42:** Docker network can't communicate?
**A:** Containers on different networks. Use `docker network connect` to attach to common network. Or use host network mode.

**Q43:** Helm release shows FAILED status?
**A:** Deployment failed but Helm marked release. Check pod errors. Can rollback with `helm rollback`. Or uninstall and reinstall.

**Q44:** Git cherry-pick conflicts?
**A:** Commit depends on other commits. Cherry-pick multiple commits, or merge properly instead of cherry-picking.

**Q45:** SonarQube analysis outdated?
**A:** Cache issue. Run `mvn clean` before analysis. Or SonarQube server needs indexing.

**Q46:** Python script works in IDE but not CLI?
**A:** Virtual environment not activated. Or wrong Python interpreter. Check `which python` matches expected path.

**Q47:** Kubernetes service IP not accessible?
**A:** Service type is ClusterIP, only internal. Change to NodePort or LoadBalancer for external access.

**Q48:** Grafana variable not updating dashboard?
**A:** Variable query syntax error. Test query in Explore first. Check datasource is correct.

**Q49:** Azure resource quota exceeded?
**A:** Region or subscription limit hit. Request quota increase through Azure Portal. Or use different region.

**Q50:** Maven build stops at "Downloading..."?
**A:** Repository unreachable or very slow. Add timeout config. Use corporate proxy/mirror.

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

---

**✅ Interview Session 6 Complete - Production Troubleshooting!**

**Focus:** Real production issues, debugging techniques, common failures, resolution strategies

**Total Questions: 50/50** | **Format:** Problem-Solution Based

