# DevOps Interview Session 9
## Azure Cloud Services & Architecture (3-7 Years)

---

### Question 1: Azure AKS vs ACI

**Interviewer:** When would you choose ACI over AKS?

**Candidate:** **ACI (Container Instances)** for simple, short-lived containers - batch jobs, CI/CD build agents, event-driven tasks. No orchestration overhead, pay per second. **AKS** for production microservices needing scaling, load balancing, self-healing, and service discovery. ACI can extend AKS with virtual nodes for burst capacity.

---

### Question 2: Azure App Service Deployment Slots

**Interviewer:** How do deployment slots work?

**Candidate:** Slots are live instances with separate hostnames. Deploy new version to staging slot, test it, then swap with production. Swap is instant - just DNS change. Slot-specific settings (like connection strings) don't swap. Auto-swap available for CI/CD.

```bash
# Create slot
az webapp deployment slot create --name myapp --resource-group rg --slot staging

# Deploy to slot
az webapp deployment source config-zip --name myapp --slot staging --src package.zip

# Swap slots
az webapp deployment slot swap --name myapp --resource-group rg --slot staging
```

---

### Question 3: Azure Functions Best Practices

**Interviewer:** Production best practices for Azure Functions?

**Candidate:** Use Premium plan for production (no cold starts), implement retry logic, keep functions stateless, use durable functions for workflows, monitor with Application Insights, set proper timeouts, use managed identities for auth, version your functions.

**Q4-35: Rapid Coverage**

**Q4:** VNet integration for App Service?
**A:** Enable VNet integration, app can access private resources. Use with Private Endpoints for inbound security.

**Q5:** Azure Storage redundancy options?
**A:** LRS (3 copies local), ZRS (across zones), GRS (geo), RA-GRS (geo + read access).

**Q6:** Azure Key Vault integration patterns?
**A:** Managed Identity for auth, reference secrets in App Settings, mount as volume in AKS with CSI driver.

**Q7:** NSG vs Azure Firewall?
**A:** NSG for L4 filtering per subnet/NIC. Firewall for centralized L7 filtering, FQDN rules, threat intelligence.

**Q8:** Azure Load Balancer vs App Gateway?
**A:** LB for L4 TCP/UDP. App Gateway for L7 HTTP/HTTPS, WAF, SSL offload, path routing.

**Q9:** Azure SQL vs Cosmos DB?
**A:** SQL for relational, ACID transactions. Cosmos for globally distributed, multi-model NoSQL, low latency.

**Q10:** Private Endpoints benefit?
**A:** PaaS services get private IP in your VNet. No internet exposure, encrypted over Azure backbone.

**Q11:** Azure Monitor vs Application Insights?
**A:** Monitor for infrastructure metrics. App Insights for application telemetry, dependencies, traces.

**Q12:** Cost optimization strategies?
**A:** Reserved instances, auto-shutdown dev/test, rightsizing VMs, cleanup unused resources, Azure Advisor recommendations.

**Q13:** Azure DevOps vs GitHub Actions?
**A:** DevOps for enterprise features, better Azure integration. Actions for GitHub-native, marketplace, simpler.

**Q14:** Service Principal vs Managed Identity?
**A:** SP needs credential management. Managed Identity is Azure-managed, no credentials to store.

**Q15:** Azure Bastion purpose?
**A:** Secure RDP/SSH through portal over SSL. No public IPs on VMs needed.

**Q16:** VNet service endpoints vs Private Link?
**A:** Endpoints route optimization. Private Link gives private IP, true network isolation.

**Q17:** Azure Policy vs RBAC?
**A:** RBAC controls who can do what. Policy enforces standards on resources (tags, locations, SKUs).

**Q18:** App Service Environment use case?
**A:** Fully isolated, dedicated environment. For high scale, compliance, VNet injection needs.

**Q19:** Azure Front Door vs Traffic Manager?
**A:** Front Door is L7 proxy with CDN. Traffic Manager is DNS-based routing, works with any service.

**Q20:** AKS node pools strategy?
**A:** System pool for K8s services, user pools for apps. Separate pools for different VM sizes or zones.

**Q21:** Azure Container Registry geo-replication?
**A:** Replicate images across regions. Faster pulls, disaster recovery, compliance.

**Q22:** Azure Functions Durable Functions?
**A:** Stateful workflows, orchestration, human interaction patterns. Checkpointing for reliability.

**Q23:** Azure Site Recovery RTO/RPO?
**A:** RPO ~5 minutes, RTO ~15-30 minutes. Depends on VM size and config.

**Q24:** Azure Availability Zones?
**A:** Separate datacenters within region. Deploy across zones for 99.99% SLA vs 99.9% single zone.

**Q25:** Azure Kubernetes networking options?
**A:** Kubenet (basic, pod NAT) vs Azure CNI (pod gets VNet IP, more IPs needed).

**Q26:** Azure Logic Apps vs Functions?
**A:** Logic Apps for no-code workflows, integrations. Functions for custom code, better performance.

**Q27:** Azure Batch use case?
**A:** Large-scale parallel compute jobs. Auto-scaling compute nodes, job scheduling.

**Q28:** Azure API Management purpose?
**A:** API gateway - rate limiting, auth, caching, transformation. Facade for backend services.

**Q29:** Azure Service Bus vs Event Grid?
**A:** Service Bus for messaging (queues, pub-sub). Event Grid for event-driven architecture, pub-sub at scale.

**Q30:** Azure Blueprints?
**A:** Repeatable set of Azure resources, policies, RBAC. Like Terraform but Azure-native.

**Q31:** Azure Lighthouse?
**A:** Multi-tenant management. MSPs manage multiple customer tenants from single pane.

**Q32:** Azure Arc use case?
**A:** Manage on-prem, AWS, GCP resources through Azure. Hybrid cloud management.

**Q33:** Azure Advisor recommendations?
**A:** Cost, security, reliability, performance, operational excellence. AI-driven optimization suggestions.

**Q34:** Azure Service Health?
**A:** Personalized alerts for Azure service issues affecting your resources. Better than generic status page.

**Q35:** Azure Resource Manager templates vs Terraform?
**A:** ARM is Azure-native, immediate feature support. Terraform multi-cloud, better testing/modularity.

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

---

**✅ Session 9 Complete - Azure Services!**

