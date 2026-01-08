# DevOps Interview Session 7
## CI/CD & Automation Focus (3-7 Years)

---

### Question 1: Jenkins Pipeline Best Practices

**Interviewer:** What are your top 5 Jenkins pipeline best practices?

**Candidate:** 
1. **Use declarative over scripted** - easier to read and maintain
2. **Keep Jenkinsfile in source control** - version with code
3. **Use shared libraries** for reusable code
4. **Clean workspaces** - don't leave artifacts
5. **Fail fast** - validate early in pipeline

```groovy
pipeline {
    agent any
    options {
        timeout(time: 1, unit: 'HOURS')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }
    stages {
        stage('Validate') {
            steps {
                sh 'npm run lint'
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
```

---

### Question 2: GitHub Actions vs Azure Pipelines

**Interviewer:** Compare GitHub Actions and Azure Pipelines for CI/CD.

**Candidate:** **GitHub Actions** is tightly integrated with GitHub - automatically available, simpler for GitHub repos, marketplace with thousands of actions. **Azure Pipelines** has more enterprise features - better RBAC, approval gates, integration with Azure services, can connect to multiple repos.

For GitHub-based projects, Actions is simpler. For enterprises using Azure or needing complex approval workflows, Azure Pipelines is better. We use both - Actions for open source, Azure Pipelines for internal products.

---

### Question 3: Docker Multi-Stage Builds for CI

**Interviewer:** Why use multi-stage builds in CI/CD?

**Candidate:** Smaller final images mean faster deployment. Build stage has all dev tools, final stage only has runtime. Reduces attack surface. Speeds up container startup.

```dockerfile
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build
RUN npm test

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
USER node
CMD ["node", "dist/server.js"]
```

Results in 150MB image vs 1.2GB with single stage.

---

### Question 4: GitOps Deployment Strategy

**Interviewer:** Implement a GitOps workflow for Kubernetes deployments.

**Candidate:** Use ArgoCD or Flux. Git repo is single source of truth. Push to Git triggers automatic deployment. Changes are audited, rollback is just reverting Git commit.

```yaml
# Git repo structure
gitops-repo/
  ├── apps/
  │   ├── frontend/
  │   └── backend/
  └── infrastructure/

# CI pipeline updates image tag
- script: |
    cd gitops-repo/apps/backend
    kustomize edit set image backend=myregistry.azurecr.io/backend:${BUILD_ID}
    git commit -am "Update backend to ${BUILD_ID}"
    git push

# ArgoCD watches repo, deploys automatically
```

---

### Question 5: Blue-Green Deployment Implementation

**Interviewer:** Implement blue-green deployment in Azure Pipelines.

**Candidate:**
```yaml
stages:
- stage: DeployGreen
  jobs:
  - deployment: DeployToGreen
    environment: production-green
    strategy:
      runOnce:
        deploy:
          steps:
          - task: Kubernetes@1
            inputs:
              command: apply
              arguments: '-f k8s/deployment-green.yaml'
          
          - script: |
              kubectl wait --for=condition=available deployment/app-green
              curl -f http://green.example.com/health || exit 1
            displayName: 'Verify Green'

- stage: SwitchTraffic
  dependsOn: DeployGreen
  jobs:
  - deployment: SwitchToGreen
    environment: production
    strategy:
      runOnce:
        deploy:
          steps:
          - task: Kubernetes@1
            inputs:
              command: apply
              arguments: '-f k8s/service-green.yaml'  # Routes to green pods
```

---

### Question 6: Canary Deployments

**Interviewer:** How do you implement canary deployments?

**Candidate:** Deploy new version to small percentage of pods, monitor metrics, gradually increase if healthy. In Kubernetes, use two deployments with service routing percentage.

```yaml
# Stable deployment - 9 replicas
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-stable
spec:
  replicas: 9
---
# Canary deployment - 1 replica (10%)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-canary
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: myapp
        version: canary

# Service routes to both
apiVersion: v1
kind: Service
spec:
  selector:
    app: myapp  # Matches both stable and canary
```

Better approach: Use service mesh like Istio for percentage-based routing.

---

### Question 7: Terraform in CI/CD Pipeline

**Interviewer:** How do you safely run Terraform in CI/CD?

**Candidate:** Always run plan first, review output, then apply. Use remote state with locking. Separate state per environment. Use workspaces or directories.

```yaml
# Azure Pipeline
stages:
- stage: TerraformPlan
  jobs:
  - job: Plan
    steps:
    - script: terraform init
    - script: terraform plan -out=tfplan
    - publish: tfplan
      artifact: terraform-plan

- stage: ManualApproval
  dependsOn: TerraformPlan
  jobs:
  - job: Approval
    pool: server
    steps:
    - task: ManualValidation@0
      inputs:
        instructions: 'Review Terraform plan before applying'

- stage: TerraformApply
  dependsOn: ManualApproval
  jobs:
  - job: Apply
    steps:
    - download: current
      artifact: terraform-plan
    - script: terraform apply tfplan
```

---

### Question 8: Maven Release Pipeline

**Interviewer:** Automate Maven release process in CI/CD.

**Candidate:**
```yaml
trigger:
  tags:
    include:
    - 'v*'

steps:
- task: Maven@3
  inputs:
    goals: 'clean deploy'
    options: '-DskipTests'
    publishJUnitResults: false

- task: Maven@3
  displayName: 'Deploy to Nexus'
  inputs:
    goals: 'deploy'
    options: '-DaltDeploymentRepository=nexus::default::https://nexus.company.com/repository/releases/'

- task: GitHubRelease@1
  inputs:
    gitHubConnection: 'GitHub'
    repositoryName: '$(Build.Repository.Name)'
    action: 'create'
    target: '$(Build.SourceVersion)'
    tagSource: 'gitTag'
    assets: 'target/*.jar'
```

---

### Question 9: SonarQube in CI Pipeline

**Interviewer:** Integrate SonarQube with quality gates.

**Candidate:**
```yaml
steps:
- task: SonarQubePrepare@5
  inputs:
    SonarQube: 'SonarQube Server'
    scannerMode: 'CLI'
    configMode: 'manual'
    cliProjectKey: 'myproject'
    cliProjectName: 'My Project'

- script: mvn clean verify sonar:sonar

- task: SonarQubePublish@5
  inputs:
    pollingTimeoutSec: '300'

- task: sonar-quality-gate@1
  inputs:
    scannerMode: 'CLI'
    # Fails pipeline if quality gate fails
```

**Interviewer:** What if quality gate is too strict for this release?

**Candidate:** Don't bypass - that defeats the purpose. Fix the issues or get approval to adjust the gate threshold temporarily. Quality should never be compromised for speed.

---

### Question 10: Docker Image Scanning

**Interviewer:** Add security scanning to Docker image CI/CD.

**Candidate:**
```yaml
steps:
- task: Docker@2
  displayName: 'Build image'
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
  displayName: 'Scan for vulnerabilities'

# Fails build if HIGH or CRITICAL vulnerabilities found
```

---

### Question 11-50: Condensed Q&A Format

**Q11:** Rollback strategy in Kubernetes deployments?
**A:** `kubectl rollout undo deployment/myapp` or use ArgoCD to revert Git commit. Keep previous deployment for quick rollback.

**Q12:** Parallel stages in Azure Pipelines?
**A:** Use `jobs:` with multiple entries at same stage level. They run in parallel.

**Q13:** Secrets management in CI/CD?
**A:** Use Azure Key Vault, AWS Secrets Manager, or HashiCorp Vault. Never hardcode secrets. Reference by ID.

**Q14:** How to speed up Maven builds in CI?
**A:** Cache `.m2` directory, use `mvn -T 1C` for parallel, skip unnecessary plugins in CI profile.

**Q15:** GitHub Actions matrix builds?
**A:** `strategy: matrix:` with `node-version: [14, 16, 18]`. Runs 3 parallel jobs.

**Q16:** Docker layer caching in CI?
**A:** Pull previous image, use `--cache-from` flag. Or use registry cache with BuildKit.

**Q17:** Terraform state locking in CI?
**A:** Azure Storage backend provides automatic locking. DynamoDB for AWS.

**Q18:** Jenkins agent selection?
**A:** Use `agent { label 'docker' }` for specific agent. Or `kubernetes` for dynamic pods.

**Q19:** Artifact versioning strategy?
**A:** Semantic versioning - `major.minor.patch`. Use Git tags. Include commit hash for traceability.

**Q20:** Branch-based deployments?
**A:** `main` → production, `develop` → staging, feature branches → dev or PR preview environments.

**Q21:** Integration testing in CI?
**A:** Use testcontainers for dependencies, run after unit tests, separate stage. Cleanup after.

**Q22:** Performance testing automation?
**A:** Add stage with JMeter or k6. Run against staging. Fail if response time > threshold.

**Q23:** Database migrations in CI/CD?
**A:** Use Flyway or Liquibase. Run as init container or pre-deployment job. Rollback plan required.

**Q24:** Multi-repo pipeline coordination?
**A:** Trigger downstream pipelines with API. Or use monorepo with selective build tools.

**Q25:** Container registry cleanup?
**A:** Automated retention policies - delete images older than X days, keep last N versions.

**Q26:** Smoke tests post-deployment?
**A:** HTTP health checks, critical API endpoints, database connectivity. Quick validation.

**Q27:** Deployment notifications?
**A:** Slack, email, or Teams. Include version, environment, who deployed, link to build.

**Q28:** Feature flags in deployments?
**A:** Deploy code but keep features off. Enable gradually. Rollback is just toggling flag.

**Q29:** A/B testing deployment?
**A:** Deploy both versions, route traffic percentage-based. Measure metrics, promote winner.

**Q30:** Scheduled pipeline runs?
**A:** Use `schedules:` trigger in Azure Pipelines or cron in GitHub Actions for nightly builds.

**Q31:** Pipeline as code benefits?
**A:** Version controlled, code reviewed, reproducible, can be tested. Infrastructure as code for CI/CD.

**Q32:** Failed deployment recovery time?
**A:** Automated rollback reduces to minutes. Manual can take hours. Practice rollback procedures.

**Q33:** Zero-downtime deployment verification?
**A:** Rolling updates, readiness probes, gradual traffic shift. Monitor error rates during deployment.

**Q34:** Build artifact storage?
**A:** Nexus, Artifactory, or Azure Artifacts. Keep for release versions, clean old ones.

**Q35:** Compliance scanning in CI?
**A:** License scanning, dependency checking, policy violations. Fail build if non-compliant.

**Q36:** Multi-cloud CI/CD?
**A:** Abstract cloud-specific code, use Terraform for IaC, containerize applications. Same pipeline, different targets.

**Q37:** Microservices deployment order?
**A:** Deploy dependencies first (databases, message queues), then services. Use health checks between stages.

**Q38:** Pipeline failure notifications?
**A:** Instant alerts to team channel, email to committer. Include logs, failure stage, and commit details.

**Q39:** Environment parity?
**A:** Use same IaC for all environments. Different variable values only. Staging should match production closely.

**Q40:** Continuous deployment vs delivery?
**A:** Delivery = automatically ready to deploy, requires manual approval. Deployment = automatically deploys to production.

**Q41:** Pipeline security best practices?
**A:** Least privilege access, signed commits, immutable artifacts, audit logs, secret scanning.

**Q42:** Build time optimization?
**A:** Parallel execution, caching, incremental builds, smaller test sets in PR validation.

**Q43:** Infrastructure drift detection?
**A:** Run `terraform plan` in scheduled pipeline. Alert if drift detected. Reconcile manually or auto-apply.

**Q44:** Container vulnerability management?
**A:** Scan on build, rescan periodically, patch base images, update dependencies, automated PR for updates.

**Q45:** Pipeline cost optimization?
**A:** Use caching, parallel jobs, skip unnecessary stages, self-hosted agents, spot instances.

**Q46:** Deployment frequency target?
**A:** Multiple times per day for mature teams. Start with once per sprint, gradually increase.

**Q47:** Failed test handling?
**A:** Block deployment, notify developer, create bug ticket. Fix before next deployment.

**Q48:** Config management across environments?
**A:** ConfigMaps/Secrets per environment, environment variables, external config servers like Spring Cloud Config.

**Q49:** Release notes automation?
**A:** Generate from Git commits, JIRA tickets, or changelog file. Auto-publish to GitHub releases or wiki.

**Q50:** CI/CD pipeline monitoring?
**A:** Track build success rate, duration, failure reasons. Dashboard with trends. Alert on pattern changes.

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

---

**✅ Session 7 Complete - CI/CD & Automation Focus!**

