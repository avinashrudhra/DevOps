# Azure Troubleshooting Guide for DevOps Engineers

**Real-World Debugging Scenarios with Root Cause Analysis & Solutions**

---

## 🎯 Guide Structure

Each troubleshooting scenario includes:
- **Symptom**: What you observe
- **Severity**: Critical/High/Medium/Low
- **Impact**: What's affected
- **Root Cause**: Why it's happening
- **Diagnostic Steps**: How to investigate
- **Solution**: How to fix
- **Prevention**: How to avoid future occurrences
- **Related Issues**: Similar problems

---

## 📚 Categories

1. **Networking** (Issues 1-15)
2. **Compute** (Issues 16-30)
3. **Storage** (Issues 31-40)
4. **Kubernetes/AKS** (Issues 41-60)
5. **Authentication & RBAC** (Issues 61-70)
6. **Performance** (Issues 71-80)
7. **Cost Management** (Issues 81-85)

---

## 🌐 NETWORKING ISSUES

### **Issue 1: VMs Cannot Communicate Between Subnets**

**Symptom:**
```bash
# From VM1 in subnet-web, cannot reach VM2 in subnet-app
ping 10.0.2.4
# Request timeout
```

**Severity:** High  
**Impact:** Application connectivity broken

**Root Cause:**
- Network Security Group (NSG) rules blocking traffic
- Route table misconfiguration
- Service endpoints interfering

**Diagnostic Steps:**
```bash
# 1. Check if VMs are in same VNet
az vm show -g rg-prod -n vm1 --query "networkProfile.networkInterfaces[].id" -o tsv | \
  xargs -I {} az network nic show --ids {} --query "ipConfigurations[].subnet.id"

# 2. Check NSG rules on source subnet
az network vnet subnet show -g rg-prod --vnet-name vnet-prod -n subnet-web \
  --query "networkSecurityGroup.id" -o tsv | \
  xargs -I {} az network nsg show --ids {}

# 3. Check NSG effective rules
az network nic list-effective-nsg \
  --resource-group rg-prod \
  --name vm1-nic

# 4. Check route table
az network vnet subnet show -g rg-prod --vnet-name vnet-prod -n subnet-web \
  --query "routeTable"

# 5. Test connectivity with Network Watcher
az network watcher test-connectivity \
  --resource-group NetworkWatcherRG \
  --source-resource vm1 \
  --dest-resource vm2 \
  --protocol Tcp \
  --dest-port 80
```

**Solution:**
```bash
# Option 1: Add NSG rule to allow traffic
az network nsg rule create \
  --resource-group rg-prod \
  --nsg-name nsg-web \
  --name Allow-Internal-Traffic \
  --priority 100 \
  --source-address-prefixes 10.0.0.0/16 \
  --destination-address-prefixes 10.0.0.0/16 \
  --destination-port-ranges '*' \
  --access Allow \
  --protocol '*'

# Option 2: Remove problematic NSG association
az network vnet subnet update \
  --resource-group rg-prod \
  --vnet-name vnet-prod \
  --name subnet-web \
  --network-security-group null
```

**Prevention:**
- Document NSG rules in repository
- Use NSG flow logs to monitor traffic
- Implement naming conventions for NSG rules
- Test connectivity after every NSG change

---

### **Issue 2: Private Endpoint DNS Resolution Failing**

**Symptom:**
```bash
# Cannot resolve private endpoint hostname
nslookup mystorageaccount.blob.core.windows.net
# Returns public IP instead of private IP
```

**Severity:** High  
**Impact:** Traffic going through public endpoint instead of private

**Root Cause:**
- Private DNS zone not linked to VNet
- DNS zone group not configured
- VM using external DNS servers

**Diagnostic Steps:**
```bash
# 1. Check private endpoint status
az network private-endpoint show \
  --resource-group rg-storage \
  --name pe-storage

# 2. Check DNS zone groups
az network private-endpoint dns-zone-group list \
  --resource-group rg-storage \
  --endpoint-name pe-storage

# 3. Check private DNS zone
az network private-dns zone show \
  --resource-group rg-storage \
  --name privatelink.blob.core.windows.net

# 4. Check VNet links
az network private-dns link vnet list \
  --resource-group rg-storage \
  --zone-name privatelink.blob.core.windows.net

# 5. Test from VM
ssh azureuser@vm-ip
nslookup mystorageaccount.blob.core.windows.net
dig mystorageaccount.blob.core.windows.net
```

**Solution:**
```bash
# 1. Create private DNS zone if missing
az network private-dns zone create \
  --resource-group rg-storage \
  --name privatelink.blob.core.windows.net

# 2. Link DNS zone to VNet
az network private-dns link vnet create \
  --resource-group rg-storage \
  --zone-name privatelink.blob.core.windows.net \
  --name storage-dns-link \
  --virtual-network vnet-prod \
  --registration-enabled false

# 3. Create DNS zone group
az network private-endpoint dns-zone-group create \
  --resource-group rg-storage \
  --endpoint-name pe-storage \
  --name storage-dns-group \
  --private-dns-zone privatelink.blob.core.windows.net \
  --zone-name storage

# 4. Verify DNS A record created
az network private-dns record-set a list \
  --resource-group rg-storage \
  --zone-name privatelink.blob.core.windows.net
```

**Prevention:**
- Always create DNS zone groups with private endpoints
- Use automation scripts for consistent private endpoint setup
- Monitor DNS resolution with health checks
- Document private endpoint architecture

---

### **Issue 3: VPN Gateway Connection Failing**

**Symptom:**
```bash
az network vpn-connection show \
  --resource-group rg-network \
  --name connection-to-onprem \
  --query "connectionStatus"
# Returns: "NotConnected"
```

**Severity:** Critical  
**Impact:** No connectivity to on-premises resources

**Root Cause:**
- Mismatched PSK (Pre-Shared Key)
- Incorrect local network gateway configuration
- Firewall blocking UDP 500/4500
- BGP peer IP mismatch

**Diagnostic Steps:**
```bash
# 1. Check VPN gateway status
az network vnet-gateway show \
  --resource-group rg-network \
  --name vgw-prod

# 2. Check connection status
az network vpn-connection show \
  --resource-group rg-network \
  --name connection-to-onprem \
  --query "{Status:connectionStatus, IngressBytes:ingressBytesTransferred, EgressBytes:egressBytesTransferred}"

# 3. Check local network gateway
az network local-gateway show \
  --resource-group rg-network \
  --name lgw-onprem

# 4. Download VPN diagnostics
az network vnet-gateway vpn-client generate \
  --resource-group rg-network \
  --name vgw-prod

# 5. Check for errors in Activity Log
az monitor activity-log list \
  --resource-group rg-network \
  --resource-id $(az network vpn-connection show -g rg-network -n connection-to-onprem --query id -o tsv) \
  --start-time 2026-01-01T00:00:00Z
```

**Solution:**
```bash
# 1. Verify and update shared key
az network vpn-connection shared-key set \
  --resource-group rg-network \
  --connection-name connection-to-onprem \
  --value "YourComplexSharedKeyHere123!"

# 2. Update local network gateway IP
az network local-gateway update \
  --resource-group rg-network \
  --name lgw-onprem \
  --gateway-ip-address <CORRECT_ONPREM_IP>

# 3. Reset VPN connection
az network vpn-connection shared-key reset \
  --resource-group rg-network \
  --connection-name connection-to-onprem

# 4. If still failing, reset VPN gateway (causes downtime!)
az network vnet-gateway reset \
  --resource-group rg-network \
  --name vgw-prod
```

**Prevention:**
- Document PSK securely in Key Vault
- Set up monitoring alerts for connection status
- Keep on-premises firewall rules documented
- Test failover procedures regularly

---

## 💻 COMPUTE ISSUES

### **Issue 4: VM Won't Start - AllocationFailed**

**Symptom:**
```bash
az vm start -g rg-prod -n vm-web-01
# Error: AllocationFailed: Allocation failed. We do not have sufficient capacity...
```

**Severity:** High  
**Impact:** VM unavailable

**Root Cause:**
- Insufficient capacity in availability zone
- VM size not available in region
- Pinned to specific cluster that's full
- Availability set constraints

**Diagnostic Steps:**
```bash
# 1. Check VM configuration
az vm show -g rg-prod -n vm-web-01 --query "{Size:hardwareProfile.vmSize, Zone:zones, AvailabilitySet:availabilitySet.id}"

# 2. Check VM status
az vm get-instance-view -g rg-prod -n vm-web-01 --query "statuses"

# 3. List available sizes in region
az vm list-skus \
  --location eastus \
  --size Standard_D \
  --output table

# 4. Check quota
az vm list-usage --location eastus --output table
```

**Solution:**
```bash
# Option 1: Resize to available VM size
az vm resize \
  --resource-group rg-prod \
  --name vm-web-01 \
  --size Standard_D2s_v4

# Option 2: Deallocate and start (allows Azure to reallocate)
az vm deallocate -g rg-prod -n vm-web-01
az vm start -g rg-prod -n vm-web-01

# Option 3: Remove from availability set (DESTRUCTIVE - data loss!)
# Requires VM recreation
# 1. Export VM configuration
# 2. Delete VM (keep disks)
# 3. Recreate without availability set

# Option 4: Deploy to different zone
az vm update \
  --resource-group rg-prod \
  --name vm-web-01 \
  --set zones=["2"]
```

**Prevention:**
- Use availability zones instead of availability sets
- Implement VM Scale Sets for flexibility
- Reserve capacity for critical workloads
- Have multi-region disaster recovery plan

---

### **Issue 5: SSH Connection Timeout to VM**

**Symptom:**
```bash
ssh azureuser@20.45.67.89
# ssh: connect to host 20.45.67.89 port 22: Connection timed out
```

**Severity:** High  
**Impact:** Cannot access VM

**Root Cause:**
- NSG blocking port 22
- VM stopped or deallocated
- Public IP not associated
- SSH service not running
- OS-level firewall blocking

**Diagnostic Steps:**
```bash
# 1. Check VM power state
az vm get-instance-view -g rg-prod -n vm-web-01 --query "instanceView.statuses[?starts_with(code, 'PowerState/')].displayStatus" -o tsv

# 2. Check public IP
az vm list-ip-addresses -g rg-prod -n vm-web-01 --query "[0].virtualMachine.network.publicIpAddresses[0].ipAddress" -o tsv

# 3. Check NSG rules
az network nsg rule list \
  --resource-group rg-prod \
  --nsg-name nsg-web \
  --query "[?destinationPortRange=='22' || destinationPortRange=='*']" \
  --output table

# 4. Check effective security rules
az network nic list-effective-nsg \
  --resource-group rg-prod \
  --name vm-web-01-nic \
  --query "value[].securityRuleProfiles.defaultSecurityRules[?destinationPortRange=='22']"

# 5. Use Serial Console (if enabled)
az vm serial-console connect -g rg-prod -n vm-web-01

# 6. Run command to check SSH status
az vm run-command invoke \
  --resource-group rg-prod \
  --name vm-web-01 \
  --command-id RunShellScript \
  --scripts "systemctl status sshd"
```

**Solution:**
```bash
# 1. Add NSG rule for SSH
az network nsg rule create \
  --resource-group rg-prod \
  --nsg-name nsg-web \
  --name Allow-SSH \
  --priority 1000 \
  --source-address-prefixes "YOUR_IP/32" \
  --destination-port-ranges 22 \
  --access Allow \
  --protocol Tcp

# 2. Start VM if stopped
az vm start -g rg-prod -n vm-web-01

# 3. Restart SSH service via run command
az vm run-command invoke \
  --resource-group rg-prod \
  --name vm-web-01 \
  --command-id RunShellScript \
  --scripts "sudo systemctl restart sshd"

# 4. Reset SSH configuration
az vm user update \
  --resource-group rg-prod \
  --name vm-web-01 \
  --username azureuser \
  --ssh-key-value "$(cat ~/.ssh/id_rsa.pub)"
```

**Prevention:**
- Use Azure Bastion for secure access
- Implement Just-In-Time (JIT) VM access
- Keep NSG rules documented
- Enable boot diagnostics and serial console
- Use managed SSH keys

---

### **Issue 6: High CPU Usage on VM**

**Symptom:**
```bash
# CPU consistently at 95-100%
top
# Shows runaway processes
```

**Severity:** Medium  
**Impact:** Poor application performance

**Diagnostic Steps:**
```bash
# 1. Check current CPU usage
top
htop  # if available

# 2. Identify top processes
ps aux --sort=-%cpu | head -10

# 3. Check system load
uptime
cat /proc/loadavg

# 4. Check Azure metrics
az monitor metrics list \
  --resource $(az vm show -g rg-prod -n vm-web-01 --query id -o tsv) \
  --metric "Percentage CPU" \
  --start-time 2026-01-07T00:00:00Z \
  --interval PT1M \
  --output table

# 5. Check for runaway Docker containers
docker stats

# 6. Check disk I/O (might be I/O wait)
iostat -x 1 5
```

**Solution:**
```bash
# 1. Kill specific runaway process
kill -9 <PID>

# 2. Restart problematic service
systemctl restart <service-name>

# 3. Limit process CPU (cgroups)
systemd-run --scope -p CPUQuota=50% <command>

# 4. Scale up VM (temporary fix)
az vm resize \
  --resource-group rg-prod \
  --name vm-web-01 \
  --size Standard_D4s_v3

# 5. Implement auto-scaling with VMSS
# Migrate to VM Scale Set for automatic scaling
```

**Prevention:**
- Implement proper application resource limits
- Use VM Scale Sets with auto-scaling
- Set up CPU usage alerts
- Profile application performance
- Implement horizontal scaling instead of vertical

---

## 💾 STORAGE ISSUES

### **Issue 7: Storage Account Throttling (Error 503)**

**Symptom:**
```bash
# Uploading files fails intermittently
az storage blob upload --account-name stprod --container-name data --file large-file.zip --name large-file.zip
# Error: (503) The server is busy.
```

**Severity:** Medium  
**Impact:** Failed uploads/downloads, slow operations

**Root Cause:**
- Exceeding storage account scalability targets
- Too many requests per second
- Large number of small files
- Single storage account bottleneck

**Diagnostic Steps:**
```bash
# 1. Check storage metrics
az monitor metrics list \
  --resource $(az storage account show -g rg-storage -n stprod --query id -o tsv) \
  --metric "Transactions" \
  --start-time 2026-01-07T00:00:00Z \
  --interval PT1M

# 2. Check for throttling
az monitor metrics list \
  --resource $(az storage account show -g rg-storage -n stprod --query id -o tsv) \
  --metric "SuccessServerLatency" "SuccessE2ELatency" \
  --start-time 2026-01-07T00:00:00Z

# 3. Review activity logs for throttling events
az monitor activity-log list \
  --resource-group rg-storage \
  --start-time 2026-01-07T00:00:00Z \
  --query "[?contains(status.value, 'Fail')]"

# 4. Check current usage against limits
# Limits: https://docs.microsoft.com/azure/storage/common/scalability-targets-standard-account
# - 20,000 requests/sec for blobs
# - 20,000 requests/sec for files
# - 20,000 requests/sec for queues
```

**Solution:**
```bash
# 1. Implement exponential backoff in application
# Python example:
cat > upload_with_retry.py <<'EOF'
from azure.storage.blob import BlobServiceClient
from azure.core.exceptions import ServiceRequestError
import time

def upload_with_retry(container_client, file_name, max_retries=5):
    for attempt in range(max_retries):
        try:
            with open(file_name, "rb") as data:
                container_client.upload_blob(name=file_name, data=data)
            return True
        except ServiceRequestError as e:
            if e.status_code == 503:
                wait_time = (2 ** attempt) + random.random()
                time.sleep(wait_time)
            else:
                raise
    return False
EOF

# 2. Use multiple storage accounts for load distribution
az storage account create \
  --resource-group rg-storage \
  --name stprod02 \
  --sku Standard_LRS

# 3. Upgrade to premium storage for higher limits
az storage account create \
  --resource-group rg-storage \
  --name stprodpremium \
  --sku Premium_LRS \
  --kind BlockBlobStorage

# 4. Implement batch operations
# Use blob batch API for multiple operations

# 5. Enable CDN for read-heavy workloads
az cdn profile create \
  --resource-group rg-storage \
  --name cdn-storage

az cdn endpoint create \
  --resource-group rg-storage \
  --profile-name cdn-storage \
  --name stprod-endpoint \
  --origin stprod.blob.core.windows.net
```

**Prevention:**
- Design for partition-level scalability
- Use multiple storage accounts
- Implement client-side retry logic
- Cache frequently accessed data
- Use CDN for static content
- Monitor storage metrics proactively

---

### **Issue 8: Cannot Access Blob with Private Endpoint**

**Symptom:**
```bash
az storage blob list --account-name stprod --container-name data --auth-mode login
# Error: (403) This request is not authorized to perform this operation.
```

**Severity:** High  
**Impact:** Application cannot access storage

**Root Cause:**
- Public network access disabled
- Private endpoint DNS not resolving
- RBAC permissions missing
- Network connectivity issue

**Diagnostic Steps:**
```bash
# 1. Check storage account network rules
az storage account show \
  --resource-group rg-storage \
  --name stprod \
  --query "networkRuleSet"

# 2. Check private endpoint status
az network private-endpoint show \
  --resource-group rg-storage \
  --name pe-storage

# 3. Test DNS resolution
nslookup stprod.blob.core.windows.net
# Should resolve to private IP (10.x.x.x)

# 4. Check RBAC role assignments
az role assignment list \
  --scope $(az storage account show -g rg-storage -n stprod --query id -o tsv) \
  --assignee $(az ad signed-in-user show --query id -o tsv)

# 5. Test from VM in same VNet
ssh azureuser@vm-test
az storage blob list --account-name stprod --container-name data --auth-mode login
```

**Solution:**
```bash
# 1. Add firewall exception for your IP (temporary testing)
az storage account network-rule add \
  --resource-group rg-storage \
  --account-name stprod \
  --ip-address $(curl -s ifconfig.me)

# 2. Grant RBAC permissions
az role assignment create \
  --role "Storage Blob Data Contributor" \
  --assignee $(az ad signed-in-user show --query id -o tsv) \
  --scope $(az storage account show -g rg-storage -n stprod --query id -o tsv)

# 3. Fix DNS resolution
# Ensure private DNS zone is linked to VNet

# 4. Add VNet to allowed networks
az storage account network-rule add \
  --resource-group rg-storage \
  --account-name stprod \
  --vnet-name vnet-prod \
  --subnet subnet-app

# 5. Enable from selected networks
az storage account update \
  --resource-group rg-storage \
  --name stprod \
  --default-action Deny \
  --bypass AzureServices
```

**Prevention:**
- Document network architecture
- Test private endpoints from all consuming resources
- Use managed identities for authentication
- Implement monitoring for access failures

---

## ☸️ KUBERNETES/AKS ISSUES

### **Issue 9: AKS Pods Stuck in ContainerCreating**

**Symptom:**
```bash
kubectl get pods -n production
# NAME                     READY   STATUS              RESTARTS   AGE
# app-7d9c5f8b-9xkmt       0/1     ContainerCreating   0          5m
```

**Severity:** High  
**Impact:** Application not running

**Root Cause:**
- Image pull failure from ACR
- Volume mount issues
- Network policy blocking
- Node resource constraints

**Diagnostic Steps:**
```bash
# 1. Describe pod for events
kubectl describe pod app-7d9c5f8b-9xkmt -n production

# 2. Check pod events specifically
kubectl get events -n production --field-selector involvedObject.name=app-7d9c5f8b-9xkmt

# 3. Check node status
kubectl get nodes
kubectl describe node <node-name>

# 4. Check if ACR is attached
az aks show -g rg-aks -n aks-cluster --query "servicePrincipalProfile.clientId"

# 5. Check image pull secret (if using secret)
kubectl get secrets -n production
kubectl describe secret acr-secret -n production

# 6. Check network policies
kubectl get networkpolicies -n production

# 7. Check node disk space
kubectl describe node <node-name> | grep -A 5 "Conditions:"
```

**Solution:**
```bash
# Solution 1: Attach ACR to AKS
az aks update \
  --resource-group rg-aks \
  --name aks-cluster \
  --attach-acr acrprod

# Solution 2: Create image pull secret
kubectl create secret docker-registry acr-secret \
  --docker-server=acrprod.azurecr.io \
  --docker-username=<ACR_USERNAME> \
  --docker-password=<ACR_PASSWORD> \
  --namespace=production

# Update deployment to use secret
kubectl patch serviceaccount default \
  -n production \
  -p '{"imagePullSecrets":[{"name":"acr-secret"}]}'

# Solution 3: Fix volume mount issues
# Check PVC status
kubectl get pvc -n production
# If PVC is pending, check storage class
kubectl get sc

# Solution 4: Increase node resources
az aks nodepool scale \
  --resource-group rg-aks \
  --cluster-name aks-cluster \
  --name nodepool1 \
  --node-count 4

# Solution 5: Delete and recreate pod
kubectl delete pod app-7d9c5f8b-9xkmt -n production
```

**Prevention:**
- Always attach ACR during AKS creation
- Use managed identities for ACR auth
- Monitor node resource usage
- Set resource requests/limits properly
- Test image pulls during deployment

---

### **Issue 10: AKS LoadBalancer Service Stuck in Pending**

**Symptom:**
```bash
kubectl get svc -n production
# NAME           TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
# app-service    LoadBalancer   10.0.45.23     <pending>     80:30123/TCP   10m
```

**Severity:** High  
**Impact:** Service not accessible externally

**Root Cause:**
- Subnet running out of IPs
- Azure Load Balancer quota exceeded
- Network plugin issues
- Service annotation problems

**Diagnostic Steps:**
```bash
# 1. Check service details
kubectl describe svc app-service -n production

# 2. Check events
kubectl get events -n production | grep app-service

# 3. Check AKS network configuration
az aks show -g rg-aks -n aks-cluster --query "networkProfile"

# 4. Check available IPs in subnet
az network vnet subnet show \
  --resource-group rg-aks \
  --vnet-name aks-vnet \
  --name aks-subnet \
  --query "addressPrefix"

# 5. Check load balancer quota
az network lb list --output table

# 6. Check kube-controller-manager logs
kubectl logs -n kube-system -l component=kube-controller-manager
```

**Solution:**
```bash
# Solution 1: Use internal load balancer (if public not needed)
kubectl annotate service app-service \
  service.beta.kubernetes.io/azure-load-balancer-internal="true" \
  -n production

# Solution 2: Expand subnet
az network vnet subnet update \
  --resource-group rg-aks \
  --vnet-name aks-vnet \
  --name aks-subnet \
  --address-prefixes 10.240.0.0/16

# Solution 3: Use existing public IP
# Create public IP
az network public-ip create \
  --resource-group MC_rg-aks_aks-cluster_eastus \
  --name pip-app-service \
  --sku Standard

# Annotate service
kubectl annotate service app-service \
  service.beta.kubernetes.io/azure-load-balancer-resource-group="MC_rg-aks_aks-cluster_eastus" \
  service.beta.kubernetes.io/azure-pip-name="pip-app-service" \
  -n production

# Solution 4: Delete and recreate service
kubectl delete svc app-service -n production
kubectl apply -f service.yaml

# Solution 5: Check and fix Azure CNI IP exhaustion
az aks nodepool update \
  --resource-group rg-aks \
  --cluster-name aks-cluster \
  --name nodepool1 \
  --max-pods 110  # Adjust based on needs
```

**Prevention:**
- Plan subnet sizing properly (Azure CNI needs many IPs)
- Use kubenet if IP space is limited
- Monitor IP usage with alerts
- Use internal load balancers where possible
- Pre-create public IPs for predictability

---

### **Issue 11: AKS Node NotReady**

**Symptom:**
```bash
kubectl get nodes
# NAME                                STATUS     ROLES   AGE   VERSION
# aks-nodepool1-12345678-vmss000000   NotReady   agent   5d    v1.28.0
```

**Severity:** Critical  
**Impact:** Pods cannot schedule on node

**Root Cause:**
- Kubelet stopped/crashed
- Disk pressure
- Memory pressure
- Network issues

**Diagnostic Steps:**
```bash
# 1. Describe node
kubectl describe node aks-nodepool1-12345678-vmss000000

# 2. Check node conditions
kubectl get node aks-nodepool1-12345678-vmss000000 -o json | jq '.status.conditions'

# 3. SSH to node (if possible)
# Get node resource group
NODE_RG=$(az aks show -g rg-aks -n aks-cluster --query nodeResourceGroup -o tsv)

# Get VMSS instance
az vmss list-instances \
  --resource-group $NODE_RG \
  --name aks-nodepool1-12345678 \
  --output table

# Run command on node
az vmss run-command invoke \
  --resource-group $NODE_RG \
  --name aks-nodepool1-12345678 \
  --instance-id 0 \
  --command-id RunShellScript \
  --scripts "systemctl status kubelet; df -h; free -m"

# 4. Check Azure activity logs
az monitor activity-log list \
  --resource-group $NODE_RG \
  --start-time 2026-01-07T00:00:00Z \
  --query "[?contains(resourceId, 'vmss')]"
```

**Solution:**
```bash
# Solution 1: Restart kubelet
az vmss run-command invoke \
  --resource-group $NODE_RG \
  --name aks-nodepool1-12345678 \
  --instance-id 0 \
  --command-id RunShellScript \
  --scripts "sudo systemctl restart kubelet"

# Solution 2: Clean up disk space
az vmss run-command invoke \
  --resource-group $NODE_RG \
  --name aks-nodepool1-12345678 \
  --instance-id 0 \
  --command-id RunShellScript \
  --scripts "
    docker system prune -af
    sudo journalctl --vacuum-time=1d
    sudo rm -rf /var/log/*.log
  "

# Solution 3: Drain and delete node
kubectl drain aks-nodepool1-12345678-vmss000000 --ignore-daemonsets --delete-emptydir-data
kubectl delete node aks-nodepool1-12345678-vmss000000

# VMSS will recreate node automatically

# Solution 4: Reimage node
az vmss reimage \
  --resource-group $NODE_RG \
  --name aks-nodepool1-12345678 \
  --instance-id 0

# Solution 5: Scale nodepool
az aks nodepool scale \
  --resource-group rg-aks \
  --cluster-name aks-cluster \
  --name nodepool1 \
  --node-count 4  # Increase to replace bad node
```

**Prevention:**
- Enable AKS diagnostics
- Monitor node health metrics
- Set up alerts for NotReady nodes
- Implement cluster autoscaler
- Use larger node disks
- Regular cluster upgrades

---

## 🔐 AUTHENTICATION & RBAC ISSUES

### **Issue 12: "Unauthorized" Error Accessing AKS**

**Symptom:**
```bash
kubectl get pods
# Error from server (Forbidden): pods is forbidden: User "user@company.com" cannot list resource "pods"
```

**Severity:** High  
**Impact:** Cannot manage cluster

**Root Cause:**
- Missing Azure AD RBAC role
- Kubernetes RBAC not configured
- Azure RBAC disabled on cluster

**Diagnostic Steps:**
```bash
# 1. Check current user identity
az account show

# 2. Check AKS configuration
az aks show -g rg-aks -n aks-cluster \
  --query "{aadProfile:aadProfile, enableRbac:enableRbac, enableAzureRbac:enableAzureRbac}"

# 3. List role assignments
az role assignment list \
  --scope $(az aks show -g rg-aks -n aks-cluster --query id -o tsv) \
  --assignee $(az ad signed-in-user show --query id -o tsv)

# 4. Check Kubernetes RBAC
kubectl get clusterrolebindings
kubectl get rolebindings --all-namespaces

# 5. Test with admin credentials
az aks get-credentials -g rg-aks -n aks-cluster --admin
kubectl get pods
```

**Solution:**
```bash
# Solution 1: Grant Azure Kubernetes Service Cluster User Role
az role assignment create \
  --role "Azure Kubernetes Service Cluster User Role" \
  --assignee user@company.com \
  --scope $(az aks show -g rg-aks -n aks-cluster --query id -o tsv)

# Solution 2: Grant Azure Kubernetes Service RBAC Admin
az role assignment create \
  --role "Azure Kubernetes Service RBAC Cluster Admin" \
  --assignee user@company.com \
  --scope $(az aks show -g rg-aks -n aks-cluster --query id -o tsv)

# Solution 3: Create Kubernetes RoleBinding
kubectl create clusterrolebinding user-admin-binding \
  --clusterrole=cluster-admin \
  --user=user@company.com

# Solution 4: Create namespace-specific access
kubectl create rolebinding dev-admin \
  --clusterrole=admin \
  --user=user@company.com \
  --namespace=development

# Solution 5: Enable Azure RBAC on existing cluster
az aks update \
  --resource-group rg-aks \
  --name aks-cluster \
  --enable-azure-rbac
```

**Prevention:**
- Document RBAC strategy
- Use Azure AD groups for access management
- Implement least privilege principle
- Regular access reviews
- Use Azure RBAC for consistent authorization

---

## 📊 PERFORMANCE ISSUES

### **Issue 13: Slow Azure SQL Database Queries**

**Symptom:**
```sql
-- Query taking 30+ seconds
SELECT * FROM Orders WHERE CustomerId = 12345;
```

**Severity:** Medium  
**Impact:** Poor application performance

**Diagnostic Steps:**
```bash
# 1. Check DTU/vCore usage
az sql db show \
  --resource-group rg-sql \
  --server sql-server \
  --name sqldb-prod \
  --query "{ServiceObjective:currentServiceObjectiveName, DTU:currentSku.capacity}"

# 2. Check Query Performance Insight (Azure Portal)
# Portal → SQL Database → Performance Overview

# 3. Check for blocking
# Run in SQL Server Management Studio or Azure Data Studio:
```

```sql
-- Check current blocking
SELECT
    blocking_session_id,
    wait_duration_ms,
    wait_type,
    s.text
FROM sys.dm_exec_requests r
CROSS APPLY sys.dm_exec_sql_text(r.sql_handle) s
WHERE blocking_session_id > 0;

-- Check missing indexes
SELECT
    migs.avg_total_user_cost * migs.avg_user_impact * (migs.user_seeks + migs.user_scans) AS improvement_measure,
    'CREATE INDEX [IX_' + OBJECT_NAME(mid.object_id, mid.database_id) + '_'
    + REPLACE(REPLACE(REPLACE(ISNULL(mid.equality_columns,''), ', ', '_'), '[', ''), ']', '')
    + CASE WHEN mid.inequality_columns IS NOT NULL THEN '_' + REPLACE(REPLACE(REPLACE(mid.inequality_columns, ', ', '_'), '[', ''), ']', '') ELSE '' END
    + '] ON ' + mid.statement + ' (' + ISNULL (mid.equality_columns,'')
    + CASE WHEN mid.inequality_columns IS NOT NULL THEN ',' + mid.inequality_columns ELSE '' END + ')'
    + ISNULL (' INCLUDE (' + mid.included_columns + ')', '') AS create_index_statement
FROM sys.dm_db_missing_index_groups mig
INNER JOIN sys.dm_db_missing_index_group_stats migs ON migs.group_handle = mig.index_group_handle
INNER JOIN sys.dm_db_missing_index_details mid ON mig.index_handle = mid.index_handle
ORDER BY improvement_measure DESC;

-- Check expensive queries
SELECT TOP 10
    qt.text AS QueryText,
    qs.execution_count,
    qs.total_worker_time / qs.execution_count AS AvgCPU,
    qs.total_elapsed_time / qs.execution_count AS AvgDuration
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) qt
ORDER BY AvgDuration DESC;
```

**Solution:**
```bash
# Solution 1: Scale up database
az sql db update \
  --resource-group rg-sql \
  --server sql-server \
  --name sqldb-prod \
  --service-objective S3

# Solution 2: Enable read replica
az sql db replica create \
  --resource-group rg-sql \
  --server sql-server \
  --name sqldb-prod \
  --partner-server sql-server-secondary

# Solution 3: Implement connection pooling
# In application config (example for .NET)
```

```csharp
// connection string
"Server=tcp:sql-server.database.windows.net;Database=sqldb-prod;User ID=admin;Password=***;Min Pool Size=10;Max Pool Size=100;Pooling=true;"
```

```bash
# Solution 4: Enable automatic tuning
az sql db update \
  --resource-group rg-sql \
  --server sql-server \
  --name sqldb-prod \
  --auto-tuning CREATE_INDEX ENABLED \
  --auto-tuning DROP_INDEX ENABLED

# Solution 5: Add missing indexes (from query above)
# Execute CREATE INDEX statements identified
```

**Prevention:**
- Regular query performance reviews
- Implement proper indexing strategy
- Use query store for analysis
- Set up performance alerts
- Monitor DTU/vCore usage
- Design for horizontal scaling

---

## 💰 COST MANAGEMENT ISSUES

### **Issue 14: Unexpected High Azure Costs**

**Symptom:**
```bash
# Monthly bill doubled without obvious changes
az consumption usage list --start-date 2026-01-01 --end-date 2026-01-31
```

**Severity:** High  
**Impact:** Budget overrun

**Root Cause:**
- Resources left running (dev/test)
- Oversized VMs
- Unused public IPs
- Excess storage
- Data egress charges

**Diagnostic Steps:**
```bash
# 1. Get cost analysis by resource group
az consumption usage list \
  --start-date 2026-01-01 \
  --end-date 2026-01-31 \
  --query "[].{ResourceGroup:instanceName, Cost:pretaxCost}" \
  --output table

# 2. List all running VMs
az vm list --query "[].{Name:name, Size:hardwareProfile.vmSize, ResourceGroup:resourceGroup, PowerState:powerState}" --output table --show-details

# 3. Find unused public IPs
az network public-ip list \
  --query "[?ipConfiguration==null].{Name:name, ResourceGroup:resourceGroup, Location:location}" \
  --output table

# 4. Find unused disks
az disk list \
  --query "[?diskState=='Unattached'].{Name:name, Size:diskSizeGb, ResourceGroup:resourceGroup, SKU:sku.name}" \
  --output table

# 5. Check storage account usage
for rg in $(az group list --query "[].name" -o tsv); do
  echo "Resource Group: $rg"
  az storage account list -g $rg --query "[].{Name:name, SKU:sku.name, Kind:kind}" --output table
done

# 6. Use Azure Cost Management
az cost-management query \
  --type Usage \
  --dataset-aggregation '{\"totalCost\":{\"name\":\"Cost\",\"function\":\"Sum\"}}' \
  --dataset-grouping name="ResourceType" type="Dimension" \
  --timeframe MonthToDate
```

**Solution:**
```bash
# Solution 1: Delete unused public IPs
az network public-ip list \
  --query "[?ipConfiguration==null].id" \
  --output tsv | \
  xargs -I {} az network public-ip delete --ids {}

# Solution 2: Delete unattached disks
az disk list \
  --query "[?diskState=='Unattached'].id" \
  --output tsv | \
  xargs -I {} az disk delete --ids {} --yes

# Solution 3: Stop deallocated VMs in dev/test
az vm list \
  --resource-group rg-dev \
  --query "[].id" \
  --output tsv | \
  xargs -I {} az vm deallocate --ids {}

# Solution 4: Implement auto-shutdown for dev VMs
az vm auto-shutdown \
  --resource-group rg-dev \
  --name vm-dev-01 \
  --time 1800 \
  --timezone "UTC"

# Solution 5: Move to reserved instances
az reservations reservation-order list

# Solution 6: Implement lifecycle management for blobs
az storage account management-policy create \
  --account-name stprod \
  --resource-group rg-storage \
  --policy @policy.json
```

```json
// policy.json
{
  "rules": [
    {
      "enabled": true,
      "name": "move-to-cool",
      "type": "Lifecycle",
      "definition": {
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
        },
        "filters": {
          "blobTypes": ["blockBlob"]
        }
      }
    }
  ]
}
```

**Prevention:**
- Set up budgets and alerts
- Tag resources by environment and cost center
- Regular cost reviews
- Implement auto-shutdown policies
- Use Azure Advisor recommendations
- Consider reserved instances for steady workloads
- Implement resource governance with Azure Policy

---

## 🎯 Quick Troubleshooting Checklist

### **Network Issues:**
- [ ] Check NSG rules
- [ ] Verify route tables
- [ ] Test DNS resolution
- [ ] Check service endpoints
- [ ] Verify firewall rules

### **Authentication Issues:**
- [ ] Verify credentials
- [ ] Check RBAC roles
- [ ] Test with --admin flag
- [ ] Review service principal
- [ ] Check managed identity

### **Performance Issues:**
- [ ] Check resource metrics (CPU, memory, disk)
- [ ] Review application logs
- [ ] Check for throttling
- [ ] Verify proper sizing
- [ ] Look for misconfigurations

### **Deployment Issues:**
- [ ] Check activity logs
- [ ] Review error messages
- [ ] Verify quotas
- [ ] Check resource dependencies
- [ ] Test in different region

---

**Remember: Always check Azure Service Health and Activity Logs first!**

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

