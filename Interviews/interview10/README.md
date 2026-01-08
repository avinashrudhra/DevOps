# DevOps Interview Session 10
## Security, Compliance & Best Practices (3-7 Years)

---

### Question 1: Container Image Scanning

**Interviewer:** How do you implement container security scanning in CI/CD?

**Candidate:** I integrate Trivy or Snyk in the pipeline. After building the image, scan runs before push to registry. If HIGH or CRITICAL vulnerabilities found, pipeline fails.

```yaml
# Azure Pipeline
steps:
- task: Docker@2
  inputs:
    command: build
    repository: myapp
    tags: $(Build.BuildId)

- script: |
    docker run --rm \
      -v /var/run/docker.sock:/var/run/docker.sock \
      aquasec/trivy image \
      --severity HIGH,CRITICAL \
      --exit-code 1 \
      myapp:$(Build.BuildId)
  displayName: 'Security Scan'

# If scan passes, then push
- task: Docker@2
  inputs:
    command: push
```

**Interviewer:** What about existing images in production?

**Candidate:** Set up scheduled scans with Azure Defender for Containers or ACR task automation. Get alerts when new CVEs discovered in deployed images. Patch and redeploy proactively.

---

### Question 2: Kubernetes RBAC Implementation

**Interviewer:** Implement least privilege RBAC for a development team.

**Candidate:**
```yaml
# Role - namespace-specific permissions
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer-role
  namespace: development
rules:
- apiGroups: ["", "apps", "batch"]
  resources: ["pods", "deployments", "services", "jobs", "configmaps"]
  verbs: ["get", "list", "create", "update", "delete"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get", "list"]
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "list"]  # Read-only secrets

---
# RoleBinding - assign to Azure AD group
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer-binding
  namespace: development
subjects:
- kind: Group
  name: "dev-team@company.com"
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer-role
  apiGroup: rbac.authorization.k8s.io
```

**Interviewer:** How do you audit RBAC permissions?

**Candidate:** Use `kubectl auth can-i` to test permissions. Enable audit logging to track who accesses what. Review with `kubectl describe rolebinding` regularly.

---

### Question 3: Secrets Management Strategy

**Interviewer:** Describe your secrets management approach across environments.

**Candidate:** Never store secrets in code or CI/CD variables. Use Azure Key Vault as source of truth. In Kubernetes, use CSI driver to mount secrets from Key Vault. In Azure Pipelines, link variable groups to Key Vault.

```yaml
# K8s with Key Vault CSI Driver
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: azure-kv-secrets
spec:
  provider: azure
  parameters:
    keyvaultName: "production-kv"
    objects: |
      array:
        - objectName: "database-password"
          objectType: "secret"
    tenantId: "tenant-id"
```

**Interviewer:** Secret rotation strategy?

**Candidate:** Rotate every 90 days. Key Vault sends alerts 30 days before expiration. Update in Key Vault, CSI driver pulls new value automatically. For database passwords, use managed identities when possible to avoid secrets altogether.

---

### Question 4: Docker Image Security Hardening

**Interviewer:** Secure a Docker image for production.

**Candidate:**
```dockerfile
# Start with minimal base
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# Final stage - distroless for security
FROM gcr.io/distroless/nodejs18-debian11
WORKDIR /app

# Copy from builder
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules

# Non-root user (distroless already uses non-root)
USER nonroot

EXPOSE 8080
CMD ["dist/server.js"]
```

**Interviewer:** Why distroless?

**Candidate:** No shell, no package managers, minimal attack surface. Can't exec into container to run malicious commands. Only has your app and runtime. Reduces CVE exposure by 90%.

---

### Question 5: Network Policies for Microsegmentation

**Interviewer:** Implement zero-trust networking in Kubernetes.

**Candidate:**
```yaml
# Default deny all traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress

---
# Allow frontend -> backend
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: production
spec:
  podSelector:
    matchLabels:
      tier: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: frontend
    ports:
    - protocol: TCP
      port: 8080

---
# Allow backend -> database
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-backend-to-db
  namespace: production
spec:
  podSelector:
    matchLabels:
      tier: backend
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          tier: database
    ports:
    - protocol: TCP
      port: 5432
  # Allow DNS
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: UDP
      port: 53
```

---

### Question 6-40: Comprehensive Security Coverage

**Q6:** CI/CD pipeline security checklist?
**A:** Secret scanning (git-secrets, truffleHog), dependency scanning (Snyk), SAST (SonarQube), artifact signing, access control, audit logging, secure credential storage.

**Q7:** Azure Key Vault integration patterns?
**A:** Managed Identity for authentication, reference in App Settings, CSI driver for K8s, CLI for scripts. Never use access keys.

**Q8:** SSL/TLS certificate automation?
**A:** cert-manager with Let's Encrypt for K8s. Auto-renewal, DNS01 or HTTP01 challenge. Store certificates in secrets.

**Q9:** Pod Security Standards?
**A:** Restricted (most secure), Baseline (minimal restrictions), Privileged (unrestricted). Enforce with admission controllers.

**Q10:** OAuth2 implementation for microservices?
**A:** Central identity provider (Azure AD), JWT tokens, API Gateway validates tokens, services trust gateway.

**Q11:** Vulnerability scanning in ACR?
**A:** Enable Azure Defender, automatic scanning on push, qualys integration, view vulnerabilities in portal.

**Q12:** Compliance as code example?
**A:** Azure Policy enforces tags, allowed regions, VM SKUs. OPA Gatekeeper for K8s enforces pod requirements.

**Q13:** Audit logging strategy?
**A:** K8s audit logs to Azure Monitor, Azure Activity Log for resource changes, Application Insights for app logs.

**Q14:** Just-in-time VM access?
**A:** Azure Security Center JIT - opens RDP/SSH only when needed for limited time. Reduces attack surface.

**Q15:** Database security best practices?
**A:** Private endpoints, firewall rules, SSL required, Azure AD auth, encryption at rest, TDE for SQL.

**Q16:** Container runtime security?
**A:** Use gVisor or Kata Containers for isolation. Seccomp profiles, AppArmor. Monitor with Falco.

**Q17:** Secret rotation automation?
**A:** Key Vault auto-rotation, Azure Functions to update downstream systems, test rotation procedures regularly.

**Q18:** SIEM integration?
**A:** Send logs to Azure Sentinel, Splunk, or ELK. Correlate events, detect anomalies, automated response.

**Q19:** Penetration testing approach?
**A:** Annual pen tests by third party, bug bounty program, automated DAST in pipeline, red team exercises.

**Q20:** Incident response plan?
**A:** Detection, containment, eradication, recovery, lessons learned. Documented runbooks, on-call rotation.

**Q21:** Backup encryption?
**A:** Azure Backup encrypts by default, customer-managed keys option, test restore procedures monthly.

**Q22:** Least privilege for service accounts?
**A:** Specific permissions only, no wildcard, regular review, temporary elevation for one-time tasks.

**Q23:** Container registry security?
**A:** Private access only, content trust (image signing), vulnerability scanning, retention policies.

**Q24:** API security?
**A:** Authentication (OAuth), authorization (RBAC), rate limiting, input validation, HTTPS only, API Gateway.

**Q25:** Secrets in logs prevention?
**A:** Mask sensitive data, never log passwords, use structured logging, log sanitization, audit log access.

**Q26:** Infrastructure immutability?
**A:** Never SSH to servers, rebuild instead of patch, version everything, automated deployments only.

**Q27:** Multi-factor authentication?
**A:** Enforce MFA for all users, conditional access policies, Privileged Identity Management for admins.

**Q28:** Data classification strategy?
**A:** Public, internal, confidential, restricted. Label data, apply appropriate controls per classification.

**Q29:** DDoS protection?
**A:** Azure DDoS Protection Standard, Application Gateway WAF, rate limiting, CDN caching.

**Q30:** Encryption at rest?
**A:** Azure Storage encryption (default), SQL TDE, disk encryption, Key Vault for key management.

**Q31:** Zero-trust network architecture?
**A:** Verify explicitly, least privilege, assume breach. Network segmentation, microsegmentation, continuous validation.

**Q32:** Security information sharing?
**A:** Threat intelligence feeds, security bulletins, vendor advisories, industry groups.

**Q33:** Compliance frameworks?
**A:** SOC 2, ISO 27001, HIPAA, PCI-DSS, GDPR. Document controls, audit evidence, regular assessments.

**Q34:** Disaster recovery security?
**A:** Encrypted backups, offline copies, tested restores, DR site security matches production.

**Q35:** Container escape prevention?
**A:** No privileged containers, read-only root filesystem, drop capabilities, seccomp/AppArmor profiles.

**Q36:** API keys vs OAuth tokens?
**A:** API keys for service-to-service, OAuth for user context. Both need rotation, rate limiting, revocation.

**Q37:** Security testing in CI/CD?
**A:** Unit tests, integration tests with security assertions, DAST against staging, compliance checks.

**Q38:** Third-party dependency risks?
**A:** Dependency scanning, lock files, private registry mirror, security policies, update regularly.

**Q39:** Insider threat mitigation?
**A:** Least privilege, activity monitoring, separation of duties, regular audits, offboarding process.

**Q40:** Security champions program?
**A:** Security advocates in each team, training, knowledge sharing, security reviews, culture building.

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

---

**✅ Session 10 Complete - Security & Compliance!**

