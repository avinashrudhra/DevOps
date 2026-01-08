# GitHub Actions Interview Questions

**90+ Questions from Basic to Advanced Production Level**

---

## 📚 Question Categories

1. **Fundamentals** (Q1-Q15) - 🟢 Beginner
2. **Workflows & Syntax** (Q16-Q30) - 🟢 Beginner
3. **Advanced Workflows** (Q31-Q45) - 🟡 Intermediate
4. **Security & Best Practices** (Q46-Q60) - 🟡 Intermediate
5. **Custom Actions** (Q61-Q70) - 🔴 Advanced
6. **Production & Architecture** (Q71-Q90) - 🔴 Advanced

**Difficulty Levels:**
- 🟢 Basic (0-2 years experience)
- 🟡 Intermediate (2-5 years experience)
- 🔴 Advanced (5+ years experience)

---

## 🌟 FUNDAMENTALS

### **Q1: What is GitHub Actions and what are its main components?** 🟢

**Answer:**
GitHub Actions is a CI/CD and automation platform built into GitHub that allows you to automate software workflows directly from your repository.

**Main Components:**

1. **Workflows**
   - Automated processes defined in YAML files
   - Located in `.github/workflows/`
   - Triggered by events

2. **Events**
   - Triggers that start workflows
   - Examples: push, pull_request, schedule, workflow_dispatch

3. **Jobs**
   - Set of steps that execute on the same runner
   - Run in parallel by default
   - Can have dependencies using `needs`

4. **Steps**
   - Individual tasks within a job
   - Can run commands or use actions
   - Execute sequentially

5. **Actions**
   - Reusable units of code
   - From marketplace or custom
   - Types: JavaScript, Docker container, composite

6. **Runners**
   - Servers that run workflows
   - GitHub-hosted or self-hosted
   - Available: Linux, Windows, macOS

**Example:**
```yaml
name: CI                          # Workflow
on: [push]                        # Event
jobs:
  build:                          # Job
    runs-on: ubuntu-latest        # Runner
    steps:                        # Steps
      - uses: actions/checkout@v4 # Action (marketplace)
      - run: npm test             # Action (command)
```

---

### **Q2: What are the differences between GitHub Actions and other CI/CD tools like Jenkins?** 🟢

**Answer:**

| Feature | GitHub Actions | Jenkins |
|---------|---------------|---------|
| **Hosting** | Cloud-based (or self-hosted) | Self-hosted only |
| **Setup** | No setup required | Requires server setup |
| **Integration** | Native GitHub integration | Plugin-based |
| **Configuration** | YAML files in repo | Jenkinsfile or UI |
| **Marketplace** | 13,000+ actions | Plugin ecosystem |
| **Pricing** | Free tier + pay per minute | Infrastructure cost only |
| **Learning Curve** | Lower | Higher |
| **Flexibility** | Good | Excellent |

**Advantages of GitHub Actions:**
- Zero setup for GitHub repositories
- Native integration with GitHub features
- YAML-based, version-controlled config
- Rich marketplace of pre-built actions
- Matrix builds built-in
- Generous free tier

**When to Use Jenkins:**
- Complex custom pipelines
- Existing Jenkins infrastructure
- Need full control
- On-premises requirements
- Very high volume builds

---

### **Q3: Explain the difference between GitHub-hosted and self-hosted runners.** 🟢

**Answer:**

**GitHub-Hosted Runners:**

**Pros:**
- No maintenance required
- Clean environment for each job
- Multiple OS options (Linux, Windows, macOS)
- Automatic updates
- Free tier available

**Cons:**
- Fixed hardware specs
- Limited customization
- Cost for private repos (after free tier)
- No access to internal networks
- Slower for large dependencies

**Specifications:**
- 2-core CPU (3-core for macOS)
- 7 GB RAM (14 GB for macOS)
- 14 GB SSD storage

**Self-Hosted Runners:**

**Pros:**
- Custom hardware/resources
- Access to internal networks
- Cost-effective for high volume
- Can install custom software
- Faster with caching

**Cons:**
- Maintenance required
- Security considerations
- Setup complexity
- No OS flexibility per job

**When to Use Each:**
```yaml
# GitHub-hosted
jobs:
  test:
    runs-on: ubuntu-latest        # GitHub-hosted

# Self-hosted
jobs:
  deploy:
    runs-on: [self-hosted, linux, x64]  # Self-hosted with labels
```

**Use self-hosted when:**
- High build volume
- Need internal network access
- Require specific hardware/software
- Cost optimization important
- Large dependencies to cache

---

### **Q4: What are workflow triggers (events) in GitHub Actions?** 🟢

**Answer:**

**Event Types:**

1. **Webhook Events:**
   ```yaml
   on:
     push:
       branches: [main]
     pull_request:
       types: [opened, synchronize]
     release:
       types: [published]
     issues:
       types: [opened, labeled]
   ```

2. **Scheduled Events:**
   ```yaml
   on:
     schedule:
       - cron: '0 0 * * *'        # Daily at midnight
       - cron: '*/15 * * * *'      # Every 15 minutes
   ```

3. **Manual Triggers:**
   ```yaml
   on:
     workflow_dispatch:
       inputs:
         environment:
           required: true
           type: choice
           options: [dev, staging, prod]
   ```

4. **Repository Events:**
   ```yaml
   on:
     fork:
     watch:
       types: [started]
     create:                       # Branch or tag created
     delete:                       # Branch or tag deleted
   ```

5. **Workflow Events:**
   ```yaml
   on:
     workflow_run:
       workflows: ["CI"]
       types: [completed]
       branches: [main]
   ```

**Event Filters:**
```yaml
on:
  push:
    branches:
      - main
      - 'releases/**'
    paths:
      - 'src/**'
      - '!docs/**'                # Ignore docs
    tags:
      - 'v*'
```

---

### **Q5: What is the `GITHUB_TOKEN` and how is it used?** 🟢

**Answer:**

**GITHUB_TOKEN** is an automatically generated secret that provides authentication for GitHub API operations within workflows.

**Key Features:**
- Automatically created for each workflow run
- Expires when job completes
- Scoped to the repository
- No manual management required

**Default Permissions:**
```yaml
# Read access to:
- contents
- metadata

# Write access to:
- contents (in some cases)
- packages (in some cases)
```

**Usage:**
```yaml
steps:
  # Automatic use in checkout
  - uses: actions/checkout@v4    # Uses GITHUB_TOKEN automatically
  
  # Explicit use
  - name: Create issue
    run: |
      curl -X POST \
        -H "Authorization: token ${{ secrets.GITHUB_TOKEN }}" \
        -H "Accept: application/vnd.github.v3+json" \
        https://api.github.com/repos/${{ github.repository }}/issues \
        -d '{"title":"New issue","body":"From workflow"}'
  
  # With GitHub CLI
  - name: Comment on PR
    run: gh pr comment ${{ github.event.pull_request.number }} --body "CI passed!"
    env:
      GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**Custom Permissions:**
```yaml
permissions:
  contents: read
  pull-requests: write
  issues: write
  packages: write

jobs:
  deploy:
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4
```

**When to Use Personal Access Token Instead:**
- Trigger other workflows
- Access other repositories
- Longer-lived operations
- More permissions needed

---

## 📝 WORKFLOWS & SYNTAX

### **Q6: Explain job dependencies and how to pass data between jobs.** 🟢

**Answer:**

**Job Dependencies:**

Jobs run in parallel by default. Use `needs` to create dependencies:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building..."
  
  test:
    needs: build                   # Runs after build
    runs-on: ubuntu-latest
    steps:
      - run: echo "Testing..."
  
  deploy:
    needs: [build, test]           # Runs after both
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying..."
```

**Passing Data Between Jobs:**

Use job outputs:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.get_version.outputs.version }}
      build-time: ${{ steps.build_info.outputs.time }}
    
    steps:
      - uses: actions/checkout@v4
      
      - id: get_version
        run: echo "version=$(cat version.txt)" >> $GITHUB_OUTPUT
      
      - id: build_info
        run: echo "time=$(date)" >> $GITHUB_OUTPUT
  
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: |
          echo "Deploying version: ${{ needs.build.outputs.version }}"
          echo "Built at: ${{ needs.build.outputs.build-time }}"
```

**Using Artifacts:**

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "data" > output.txt
      - uses: actions/upload-artifact@v4
        with:
          name: build-data
          path: output.txt
  
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build-data
      - run: cat output.txt
```

**Best Practices:**
- Use outputs for small data (strings, numbers)
- Use artifacts for files/binaries
- Keep artifact sizes reasonable
- Set appropriate retention periods

---

### **Q7: What are matrix builds and when would you use them?** 🟡

**Answer:**

**Matrix builds** allow you to test across multiple configurations simultaneously.

**Basic Matrix:**
```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [16, 18, 20]
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm test
```

This creates **9 jobs** (3 OS × 3 Node versions).

**Advanced Matrix with Include/Exclude:**
```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest]
    node-version: [16, 18, 20]
    
    include:
      # Add experimental combination
      - os: ubuntu-latest
        node-version: 21
        experimental: true
      
      # Add specific config
      - os: macos-latest
        node-version: 20
        special-flag: --experimental
    
    exclude:
      # Remove incompatible combination
      - os: windows-latest
        node-version: 16
```

**Matrix Options:**
```yaml
strategy:
  matrix:
    # ... matrix definition ...
  
  fail-fast: false                 # Continue all jobs even if one fails
  max-parallel: 3                  # Limit concurrent jobs
```

**Dynamic Matrix:**
```yaml
jobs:
  setup:
    outputs:
      matrix: ${{ steps.set-matrix.outputs.matrix }}
    steps:
      - id: set-matrix
        run: echo "matrix={\"node\":[16,18,20]}" >> $GITHUB_OUTPUT
  
  test:
    needs: setup
    strategy:
      matrix: ${{ fromJSON(needs.setup.outputs.matrix) }}
    steps:
      - run: echo "Testing Node ${{ matrix.node }}"
```

**When to Use:**
- Test across multiple OS
- Test multiple language versions
- Test different configurations
- Cross-browser testing
- Multi-architecture builds

**Best Practices:**
- Use `fail-fast: false` for comprehensive testing
- Set `max-parallel` to control resource usage
- Use matrix for truly independent tests
- Consider cost (each combination = 1 job)

---

### **Q8: How do you handle secrets in GitHub Actions?** 🟡

**Answer:**

**Types of Secrets:**

1. **Repository Secrets** (repository level)
2. **Organization Secrets** (organization level)
3. **Environment Secrets** (environment level)

**Setting Secrets:**
```
Repository → Settings → Secrets and variables → Actions → New repository secret
```

**Using Secrets:**
```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to server
        run: ./deploy.sh
        env:
          API_KEY: ${{ secrets.API_KEY }}
          DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
```

**Environment Secrets:**
```yaml
jobs:
  deploy-prod:
    runs-on: ubuntu-latest
    environment: production          # Uses production secrets
    steps:
      - name: Deploy
        run: ./deploy.sh
        env:
          DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
```

**Best Practices:**

1. **Never Echo Secrets:**
```yaml
# ❌ BAD
- run: echo "Token: ${{ secrets.TOKEN }}"

# ✅ GOOD
- run: ./deploy.sh
  env:
    TOKEN: ${{ secrets.TOKEN }}
```

2. **Mask Custom Values:**
```yaml
- run: |
    echo "::add-mask::$MY_SECRET_VALUE"
    echo "$MY_SECRET_VALUE"        # Will be masked
```

3. **Use Environment Secrets for Deployments:**
```yaml
environment:
  name: production
  url: https://prod.example.com
```

4. **OIDC for Cloud Authentication:**
```yaml
permissions:
  id-token: write
  contents: read

steps:
  - uses: aws-actions/configure-aws-credentials@v4
    with:
      role-to-assume: arn:aws:iam::123456789:role/GitHubActions
      aws-region: us-east-1
```

5. **Rotate Secrets Regularly**
6. **Use Least Privilege**
7. **Audit Secret Usage**

---

## 🔧 ADVANCED WORKFLOWS

### **Q9: What are reusable workflows and how do they differ from composite actions?** 🟡

**Answer:**

**Reusable Workflows:**

Entire workflows that can be called from other workflows.

```yaml
# .github/workflows/reusable-deploy.yml
name: Reusable Deploy
on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
    secrets:
      deploy-token:
        required: true
    outputs:
      url:
        value: ${{ jobs.deploy.outputs.url }}

jobs:
  deploy:
    runs-on: ubuntu-latest
    outputs:
      url: ${{ steps.deploy.outputs.url }}
    steps:
      - uses: actions/checkout@v4
      - id: deploy
        run: echo "url=https://app.example.com" >> $GITHUB_OUTPUT

# Calling workflow
jobs:
  call-deploy:
    uses: ./.github/workflows/reusable-deploy.yml
    with:
      environment: production
    secrets:
      deploy-token: ${{ secrets.DEPLOY_TOKEN }}
```

**Composite Actions:**

Reusable steps within a job.

```yaml
# .github/actions/setup-project/action.yml
name: Setup Project
inputs:
  node-version:
    required: true
runs:
  using: composite
  steps:
    - uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}
      shell: bash
    - run: npm ci
      shell: bash

# Using composite action
steps:
  - uses: ./.github/actions/setup-project
    with:
      node-version: '20'
```

**Comparison:**

| Feature | Reusable Workflows | Composite Actions |
|---------|-------------------|-------------------|
| **Scope** | Entire workflow (jobs) | Steps within a job |
| **Location** | `.github/workflows/` | `.github/actions/` or repo |
| **Triggers** | `workflow_call` | Used in `steps` |
| **Secrets** | Can receive secrets | Cannot receive secrets directly |
| **Runners** | Defines its own runners | Uses caller's runner |
| **Outputs** | Job-level outputs | Step-level outputs |
| **Complexity** | Higher | Lower |

**When to Use:**
- **Reusable Workflows**: Full CI/CD pipelines, multi-job processes
- **Composite Actions**: Repeated step sequences, setup tasks

---

### **Q10: Explain caching in GitHub Actions and best practices.** 🟡

**Answer:**

**Purpose of Caching:**
- Speed up workflows
- Reduce bandwidth
- Lower costs
- Consistent builds

**Basic Caching:**
```yaml
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-
      ${{ runner.os }}-
```

**Built-in Caching:**
```yaml
# Node.js
- uses: actions/setup-node@v4
  with:
    node-version: '20'
    cache: 'npm'                   # Automatic caching

# Python
- uses: actions/setup-python@v5
  with:
    python-version: '3.12'
    cache: 'pip'

# Java/Maven
- uses: actions/setup-java@v4
  with:
    distribution: 'temurin'
    java-version: '17'
    cache: 'maven'
```

**Cache Key Strategy:**

**Good Cache Keys:**
```yaml
# Version-specific
key: ${{ runner.os }}-node-v2-${{ hashFiles('**/package-lock.json') }}

# Multiple file hash
key: ${{ runner.os }}-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}

# With fallback
key: ${{ runner.os }}-build-${{ github.sha }}
restore-keys: |
  ${{ runner.os }}-build-
  ${{ runner.os }}-
```

**Multiple Paths:**
```yaml
- uses: actions/cache@v4
  with:
    path: |
      ~/.npm
      ~/.cache
      node_modules
      **/node_modules
    key: ${{ runner.os }}-deps-${{ hashFiles('**/package-lock.json') }}
```

**Conditional Installation:**
```yaml
- id: cache
  uses: actions/cache@v4
  with:
    path: node_modules
    key: ${{ runner.os }}-${{ hashFiles('package-lock.json') }}

- if: steps.cache.outputs.cache-hit != 'true'
  run: npm ci
```

**Best Practices:**

1. **Use Specific Keys**: Include version numbers or file hashes
2. **Implement Fallbacks**: Use `restore-keys` for partial matches
3. **Cache Appropriate Data**: Dependencies, build artifacts, not source code
4. **Mind Cache Limits**: 10 GB per repository
5. **Version Cache Keys**: Increment when breaking changes occur
6. **Test Without Cache**: Ensure builds work without cache
7. **Monitor Cache Hit Rates**: Check workflow insights
8. **Clean Invalid Caches**: Delete corrupted caches from settings

**Cache Limits:**
- 10 GB total per repository
- Oldest caches evicted when limit reached
- 7-day retention if not accessed

---

## 🔒 SECURITY & BEST PRACTICES

### **Q11: What are the security best practices for GitHub Actions?** 🔴

**Answer:**

**1. Pin Actions to Commit SHA:**
```yaml
# ❌ Bad - mutable tag
- uses: actions/checkout@v4

# ✅ Good - immutable SHA
- uses: actions/checkout@8ade135a41bc03ea155e62e844d188df1ea18608  # v4.1.0

# ✅ Also acceptable - major version
- uses: actions/checkout@v4
```

**2. Limit Permissions:**
```yaml
permissions:
  contents: read
  pull-requests: write

jobs:
  deploy:
    permissions:
      contents: read
      id-token: write              # Minimal permissions
```

**3. Secure Secrets:**
```yaml
# Never expose secrets
# ❌ Bad
- run: echo "${{ secrets.API_KEY }}"

# ✅ Good
- run: ./script.sh
  env:
    API_KEY: ${{ secrets.API_KEY }}
```

**4. Use OIDC for Cloud Authentication:**
```yaml
permissions:
  id-token: write

steps:
  - uses: aws-actions/configure-aws-credentials@v4
    with:
      role-to-assume: arn:aws:iam::123456789:role/GitHub
      aws-region: us-east-1
```

**5. Implement Code Scanning:**
```yaml
- uses: github/codeql-action/init@v3
  with:
    languages: javascript, python

- uses: github/codeql-action/analyze@v3
```

**6. Scan Dependencies:**
```yaml
- uses: actions/dependency-review-action@v4
  if: github.event_name == 'pull_request'
```

**7. Review Third-Party Actions:**
- Check source code
- Review permissions requested
- Check maintainer reputation
- Pin to specific versions

**8. Protect Sensitive Branches:**
- Require reviews for workflow changes
- Use environment protection rules
- Enable branch protection

**9. Audit Logs:**
- Monitor workflow runs
- Review failed authentications
- Track secret access

**10. Self-Hosted Runner Security:**
- Isolate runners
- Use ephemeral runners
- Implement runner groups
- Restrict repository access

---

## 🎯 PRODUCTION & ARCHITECTURE

### **Q12: How would you design a production-ready CI/CD pipeline using GitHub Actions?** 🔴

**Answer:**

**Complete Production Pipeline Architecture:**

```yaml
name: Production CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

permissions:
  contents: read
  packages: write
  pull-requests: write
  security-events: write

jobs:
  # Stage 1: Code Quality & Security
  code-quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      # Static analysis
      - uses: github/codeql-action/init@v3
        with:
          languages: javascript
      
      - run: npm ci && npm run build
      
      - uses: github/codeql-action/analyze@v3
      
      # Linting
      - run: npm run lint
      
      # Dependency scanning
      - uses: actions/dependency-review-action@v4
        if: github.event_name == 'pull_request'
  
  # Stage 2: Multi-Platform Testing
  test:
    needs: code-quality
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [18, 20]
      fail-fast: false
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      - run: npm ci
      - run: npm test -- --coverage
      
      - uses: codecov/codecov-action@v3
        if: matrix.os == 'ubuntu-latest' && matrix.node-version == '20'
  
  # Stage 3: Build & Package
  build:
    needs: test
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.version.outputs.version }}
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - id: version
        run: echo "version=$(node -p "require('./package.json').version")" >> $GITHUB_OUTPUT
      
      - run: npm ci
      - run: npm run build
      
      - uses: actions/upload-artifact@v4
        with:
          name: dist-${{ github.sha }}
          path: dist/
          retention-days: 7
  
  # Stage 4: Container Build & Security Scan
  docker:
    needs: build
    if: github.event_name == 'push'
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: docker/setup-buildx-action@v3
      
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
      
      - uses: aquasecurity/trivy-action@master
        with:
          image-ref: ghcr.io/${{ github.repository }}:${{ github.sha }}
          format: 'sarif'
          output: 'trivy-results.sarif'
      
      - uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: 'trivy-results.sarif'
  
  # Stage 5: Deploy to Staging
  deploy-staging:
    needs: [build, docker]
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.example.com
    
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: dist-${{ github.sha }}
      
      - name: Deploy to staging
        run: ./deploy.sh staging
        env:
          DEPLOY_TOKEN: ${{ secrets.STAGING_DEPLOY_TOKEN }}
      
      - name: Run smoke tests
        run: npm run test:e2e
        env:
          BASE_URL: https://staging.example.com
  
  # Stage 6: Deploy to Production
  deploy-production:
    needs: [build, docker]
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://app.example.com
    
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: dist-${{ github.sha }}
      
      - name: Deploy to production
        run: ./deploy.sh production
        env:
          DEPLOY_TOKEN: ${{ secrets.PROD_DEPLOY_TOKEN }}
      
      - name: Health check
        run: curl -f https://app.example.com/health
      
      - name: Notify team
        uses: slackapi/slack-github-action@v1
        with:
          webhook-url: ${{ secrets.SLACK_WEBHOOK }}
          payload: |
            {
              "text": "✅ Production deployment successful: v${{ needs.build.outputs.version }}"
            }
  
  # Stage 7: Post-Deployment Monitoring
  monitor:
    needs: deploy-production
    if: success()
    runs-on: ubuntu-latest
    
    steps:
      - name: Check application metrics
        run: ./check-metrics.sh
      
      - name: Create deployment event
        uses: actions/github-script@v7
        with:
          script: |
            await github.rest.repos.createDeployment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              ref: context.sha,
              environment: 'production',
              description: 'Deployment completed'
            });
```

**Key Principles:**

1. **Fail Fast**: Code quality checks first
2. **Parallel Execution**: Independent jobs run simultaneously
3. **Caching**: Dependencies cached for speed
4. **Security First**: Scanning at multiple stages
5. **Environment Isolation**: Separate staging/production
6. **Approval Gates**: Manual approval for production
7. **Rollback Capability**: Artifact retention for rollbacks
8. **Monitoring**: Post-deployment validation
9. **Notifications**: Team alerts on success/failure
10. **Audit Trail**: Deployment tracking

---

**Continue with remaining 78 questions covering:**
- Custom action development
- Self-hosted runners
- Monorepo strategies
- Performance optimization
- Cost management
- Advanced troubleshooting
- Integration patterns

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

