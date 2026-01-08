# GitHub Actions Quick Reference

**Essential Workflow Syntax, Actions & Patterns for Daily Use**

---

## 🚀 Basic Workflow Structure

```yaml
name: Workflow Name                    # Optional: Display name

on: [push, pull_request]               # Events that trigger workflow

jobs:
  job-id:                              # Unique job identifier
    runs-on: ubuntu-latest             # Runner environment
    steps:
      - name: Step name                # Optional: Display name
        run: echo "Hello World"        # Command to execute
```

---

## 📅 Workflow Triggers (on:)

### **Push Events**
```yaml
# Trigger on any push
on: push

# Trigger on specific branches
on:
  push:
    branches:
      - main
      - develop
      - 'releases/**'                  # Wildcard pattern
    branches-ignore:
      - 'feature/**'

# Trigger on specific paths
on:
  push:
    paths:
      - 'src/**'
      - '**.js'
    paths-ignore:
      - 'docs/**'
      - '**.md'

# Trigger on tags
on:
  push:
    tags:
      - 'v*'                           # v1.0, v2.0, etc.
      - 'v[0-9]+.[0-9]+.[0-9]+'       # Semantic versioning
```

### **Pull Request Events**
```yaml
on:
  pull_request:
    types:                             # Default: opened, synchronize, reopened
      - opened
      - edited
      - closed
      - reopened
      - synchronized
      - ready_for_review
    branches:
      - main

# Auto-merge PRs
on:
  pull_request_target:                 # Access to secrets (use carefully!)
    types: [opened, synchronize]
```

### **Schedule (Cron)**
```yaml
on:
  schedule:
    - cron: '0 0 * * *'                # Daily at midnight UTC
    - cron: '*/15 * * * *'             # Every 15 minutes
    - cron: '0 0 * * 0'                # Weekly on Sunday
    - cron: '0 0 1 * *'                # Monthly on 1st day

# Cron syntax: MIN HOUR DAY MONTH WEEKDAY
# * = any value
# */n = every n units
# 0-5 = range
```

### **Manual Trigger**
```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy'
        required: true
        type: choice
        options:
          - dev
          - staging
          - production
        default: 'dev'
      
      version:
        description: 'Version to deploy'
        required: false
        type: string
      
      debug:
        description: 'Enable debug logging'
        required: false
        type: boolean
        default: false

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - run: |
          echo "Deploying to ${{ inputs.environment }}"
          echo "Version: ${{ inputs.version }}"
          echo "Debug: ${{ inputs.debug }}"
```

### **Repository Events**
```yaml
on:
  release:
    types: [published, created, edited]
  
  issues:
    types: [opened, labeled]
  
  issue_comment:
    types: [created, edited]
  
  pull_request_review:
    types: [submitted, edited]
  
  fork:
  watch:
    types: [started]
  
  workflow_run:                        # Trigger after another workflow
    workflows: ["CI"]
    types: [completed]
    branches: [main]
```

### **Multiple Triggers**
```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 0 * * *'
  workflow_dispatch:
```

---

## 🎯 Jobs Configuration

### **Basic Job**
```yaml
jobs:
  job-name:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Hello"
```

### **Runner Selection**
```yaml
jobs:
  linux:
    runs-on: ubuntu-latest             # or ubuntu-20.04, ubuntu-22.04
  
  windows:
    runs-on: windows-latest            # or windows-2019, windows-2022
  
  macos:
    runs-on: macos-latest              # or macos-12, macos-13, macos-14
  
  self-hosted:
    runs-on: [self-hosted, linux, x64] # Custom labels
```

### **Job Dependencies**
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building..."
  
  test:
    needs: build                       # Wait for build to complete
    runs-on: ubuntu-latest
    steps:
      - run: echo "Testing..."
  
  deploy:
    needs: [build, test]               # Wait for multiple jobs
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying..."
```

### **Conditional Jobs**
```yaml
jobs:
  deploy:
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying to production"
  
  skip-draft-pr:
    if: github.event.pull_request.draft == false
    runs-on: ubuntu-latest
    steps:
      - run: echo "Not a draft PR"
```

### **Job Outputs**
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.get_version.outputs.version }}
      build-id: ${{ steps.build.outputs.id }}
    steps:
      - id: get_version
        run: echo "version=1.2.3" >> $GITHUB_OUTPUT
      
      - id: build
        run: echo "id=${{ github.run_number }}" >> $GITHUB_OUTPUT
  
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: |
          echo "Version: ${{ needs.build.outputs.version }}"
          echo "Build ID: ${{ needs.build.outputs.build-id }}"
```

### **Job Timeout**
```yaml
jobs:
  long-running:
    runs-on: ubuntu-latest
    timeout-minutes: 60                # Default: 360 (6 hours)
    steps:
      - run: ./long-script.sh
```

### **Continue on Error**
```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    continue-on-error: true            # Job failure won't fail workflow
    steps:
      - run: npm test
```

---

## 🔄 Matrix Strategy

### **Basic Matrix**
```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [16, 18, 20]
    
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm test
```

### **Matrix with Include/Exclude**
```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest]
    node-version: [16, 18, 20]
    
    include:
      # Add experimental build
      - os: ubuntu-latest
        node-version: 21
        experimental: true
      
      # Add specific configuration
      - os: macos-latest
        node-version: 20
        arch: arm64
    
    exclude:
      # Remove incompatible combinations
      - os: windows-latest
        node-version: 16
```

### **Matrix Options**
```yaml
strategy:
  matrix:
    # ... matrix definition ...
  
  fail-fast: false                     # Don't cancel other jobs on failure
  max-parallel: 3                      # Limit concurrent jobs
```

---

## 📝 Steps Configuration

### **Run Commands**
```yaml
steps:
  # Single-line command
  - run: echo "Hello World"
  
  # Multi-line command
  - run: |
      echo "Line 1"
      echo "Line 2"
      echo "Line 3"
  
  # Multiple commands (different approach)
  - run: echo "Command 1" && echo "Command 2"
  
  # With working directory
  - run: npm install
    working-directory: ./frontend
  
  # With custom shell
  - run: Write-Output "Hello from PowerShell"
    shell: pwsh                        # bash, pwsh, python, sh, cmd, powershell
  
  # With environment variables
  - run: echo "Value is $MY_VAR"
    env:
      MY_VAR: "test-value"
  
  # With continue on error
  - run: npm test
    continue-on-error: true
  
  # With timeout
  - run: ./long-running-script.sh
    timeout-minutes: 10
```

### **Using Actions**
```yaml
steps:
  # Use specific version (recommended)
  - uses: actions/checkout@v4
  
  # Use major version
  - uses: actions/checkout@v4
  
  # Use commit SHA (most secure)
  - uses: actions/checkout@8ade135a41bc03ea155e62e844d188df1ea18608
  
  # With inputs
  - uses: actions/setup-node@v4
    with:
      node-version: '20'
      cache: 'npm'
  
  # From another repository
  - uses: username/repo-name@v1
  
  # From same repository
  - uses: ./.github/actions/my-action
  
  # With ID for referencing outputs
  - id: build-step
    uses: actions/build-action@v1
  
  # With conditional execution
  - uses: actions/upload-artifact@v4
    if: success()
    with:
      name: build-output
      path: dist/
```

### **Step Conditions**
```yaml
steps:
  # Run only on success
  - if: success()
    run: echo "Previous steps succeeded"
  
  # Run only on failure
  - if: failure()
    run: echo "Something failed"
  
  # Always run
  - if: always()
    run: echo "This always runs"
  
  # Run only when cancelled
  - if: cancelled()
    run: echo "Workflow was cancelled"
  
  # Complex conditions
  - if: github.ref == 'refs/heads/main' && success()
    run: echo "Deploy to production"
  
  # Check previous step outcome
  - id: test
    run: npm test
  
  - if: steps.test.outcome == 'success'
    run: echo "Tests passed"
  
  # Check previous step conclusion (includes cancelled/skipped)
  - if: steps.test.conclusion == 'success'
    run: echo "Tests concluded successfully"
```

---

## 🔧 Essential Actions

### **Checkout Code**
```yaml
- uses: actions/checkout@v4

# Checkout with submodules
- uses: actions/checkout@v4
  with:
    submodules: true

# Checkout specific ref
- uses: actions/checkout@v4
  with:
    ref: develop

# Checkout with full history
- uses: actions/checkout@v4
  with:
    fetch-depth: 0

# Checkout multiple repositories
- uses: actions/checkout@v4
  with:
    repository: org/other-repo
    token: ${{ secrets.PAT }}
    path: other-repo
```

### **Setup Language Runtimes**
```yaml
# Node.js
- uses: actions/setup-node@v4
  with:
    node-version: '20'
    cache: 'npm'                       # or 'yarn', 'pnpm'
    registry-url: 'https://registry.npmjs.org'

# Python
- uses: actions/setup-python@v5
  with:
    python-version: '3.12'
    cache: 'pip'

# Java
- uses: actions/setup-java@v4
  with:
    distribution: 'temurin'            # or 'zulu', 'adopt', 'microsoft'
    java-version: '17'
    cache: 'maven'                     # or 'gradle'

# Go
- uses: actions/setup-go@v5
  with:
    go-version: '1.22'
    cache: true

# .NET
- uses: actions/setup-dotnet@v4
  with:
    dotnet-version: '8.0.x'

# Ruby
- uses: ruby/setup-ruby@v1
  with:
    ruby-version: '3.2'
    bundler-cache: true
```

### **Artifacts**
```yaml
# Upload artifact
- uses: actions/upload-artifact@v4
  with:
    name: build-output
    path: |
      dist/
      build/
      !dist/**/*.map                   # Exclude files
    retention-days: 7                  # Default: 90
    if-no-files-found: error          # or 'warn', 'ignore'
    compression-level: 6               # 0-9, default: 6

# Download artifact
- uses: actions/download-artifact@v4
  with:
    name: build-output
    path: ./downloaded

# Download all artifacts
- uses: actions/download-artifact@v4
```

### **Caching**
```yaml
# Cache dependencies
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-
      ${{ runner.os }}-

# Multiple paths
- uses: actions/cache@v4
  with:
    path: |
      ~/.npm
      ~/.cache
      node_modules
    key: ${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}

# Cache hit output
- id: cache
  uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}

- if: steps.cache.outputs.cache-hit != 'true'
  run: npm ci
```

### **Docker**
```yaml
# Build and push
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
    tags: |
      ghcr.io/${{ github.repository }}:latest
      ghcr.io/${{ github.repository }}:${{ github.sha }}
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

---

## 💾 Environment Variables & Secrets

### **Default Environment Variables**
```yaml
steps:
  - run: |
      echo "Workflow: $GITHUB_WORKFLOW"
      echo "Action: $GITHUB_ACTION"
      echo "Actor: $GITHUB_ACTOR"
      echo "Repository: $GITHUB_REPOSITORY"
      echo "Event: $GITHUB_EVENT_NAME"
      echo "SHA: $GITHUB_SHA"
      echo "Ref: $GITHUB_REF"
      echo "Ref Name: $GITHUB_REF_NAME"
      echo "Head Ref: $GITHUB_HEAD_REF"
      echo "Base Ref: $GITHUB_BASE_REF"
      echo "Workspace: $GITHUB_WORKSPACE"
      echo "Run ID: $GITHUB_RUN_ID"
      echo "Run Number: $GITHUB_RUN_NUMBER"
```

### **Set Environment Variables**
```yaml
# Workflow-level
env:
  NODE_ENV: production
  API_URL: https://api.example.com

jobs:
  build:
    # Job-level
    env:
      BUILD_TYPE: release
    
    runs-on: ubuntu-latest
    steps:
      # Step-level
      - run: echo "Building..."
        env:
          VERBOSE: true
      
      # Set for subsequent steps
      - run: echo "MY_VAR=my-value" >> $GITHUB_ENV
      
      - run: echo $MY_VAR              # Outputs: my-value
```

### **Using Secrets**
```yaml
steps:
  - run: echo "Deploying..."
    env:
      API_KEY: ${{ secrets.API_KEY }}
      DB_PASSWORD: ${{ secrets.DB_PASSWORD }}

# Secrets are automatically masked in logs
```

### **Environment Protection**
```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://prod.example.com
    steps:
      - run: echo "Deploying to ${{ secrets.SERVER_URL }}"
        env:
          DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
```

---

## 📊 Context & Expressions

### **GitHub Context**
```yaml
steps:
  - run: |
      echo "Event: ${{ github.event_name }}"
      echo "Repository: ${{ github.repository }}"
      echo "Owner: ${{ github.repository_owner }}"
      echo "Repo Name: ${{ github.event.repository.name }}"
      echo "Actor: ${{ github.actor }}"
      echo "SHA: ${{ github.sha }}"
      echo "Ref: ${{ github.ref }}"
      echo "Ref Name: ${{ github.ref_name }}"
      echo "Workflow: ${{ github.workflow }}"
      echo "Action: ${{ github.action }}"
      echo "Job: ${{ github.job }}"
      echo "Run ID: ${{ github.run_id }}"
      echo "Run Number: ${{ github.run_number }}"
      echo "API URL: ${{ github.api_url }}"
      echo "Server URL: ${{ github.server_url }}"
      echo "Workspace: ${{ github.workspace }}"

# PR-specific
- run: |
    echo "PR Number: ${{ github.event.pull_request.number }}"
    echo "PR Title: ${{ github.event.pull_request.title }}"
    echo "PR Head SHA: ${{ github.event.pull_request.head.sha }}"
    echo "PR Base Ref: ${{ github.event.pull_request.base.ref }}"
```

### **Other Contexts**
```yaml
# Runner context
- run: |
    echo "OS: ${{ runner.os }}"        # Linux, Windows, macOS
    echo "Arch: ${{ runner.arch }}"    # X86, X64, ARM, ARM64
    echo "Name: ${{ runner.name }}"
    echo "Temp: ${{ runner.temp }}"
    echo "Tool Cache: ${{ runner.tool_cache }}"

# Job context
- run: echo "Status: ${{ job.status }}" # success, failure, cancelled

# Steps context
- id: build
  run: npm build

- run: |
    echo "Outcome: ${{ steps.build.outcome }}"         # success, failure, cancelled, skipped
    echo "Conclusion: ${{ steps.build.conclusion }}"   # success, failure, cancelled, skipped, neutral
    echo "Outputs: ${{ steps.build.outputs.version }}"

# Needs context (in dependent jobs)
- run: echo "Build output: ${{ needs.build.outputs.version }}"

# Secrets context
- run: echo "API Key length: ${{ length(secrets.API_KEY) }}"
```

### **Expression Functions**
```yaml
# String functions
- if: contains(github.ref, 'refs/heads/')
- if: startsWith(github.ref, 'refs/tags/v')
- if: endsWith(github.ref, '-dev')
- if: format('v{0}.{1}', 1, 2)         # Returns: v1.2

# Comparison
- if: github.event.pull_request.number > 100
- if: runner.os == 'Linux'
- if: job.status == 'success'

# Logical operators
- if: github.ref == 'refs/heads/main' && success()
- if: github.event_name == 'push' || github.event_name == 'pull_request'
- if: "!cancelled()"

# Status functions
- if: success()                        # All previous steps succeeded
- if: failure()                        # Any previous step failed
- if: always()                         # Always run
- if: cancelled()                      # Workflow was cancelled

# Type conversion
- run: echo "${{ toJSON(github) }}"    # Convert to JSON
- run: echo "${{ fromJSON('{"key":"value"}').key }}"

# Array functions
- if: contains(fromJSON('["dev", "prod"]'), github.ref_name)

# Hash functions
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ hashFiles('**/package-lock.json') }}
```

---

## 🔐 Security Best Practices

### **Pin Actions to SHA**
```yaml
# ❌ Bad - uses mutable tag
- uses: actions/checkout@v4

# ✅ Good - pinned to commit SHA
- uses: actions/checkout@8ade135a41bc03ea155e62e844d188df1ea18608  # v4.1.0

# ✅ Also acceptable - major version
- uses: actions/checkout@v4
```

### **Limit Permissions**
```yaml
# Workflow-level
permissions:
  contents: read
  pull-requests: write
  issues: write

jobs:
  deploy:
    # Job-level permissions
    permissions:
      contents: read
      id-token: write                  # For OIDC
    
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
```

### **Secure Secrets Usage**
```yaml
steps:
  # ❌ Bad - secret in output
  - run: echo "Token: ${{ secrets.TOKEN }}"

  # ✅ Good - secret in env var
  - run: ./deploy.sh
    env:
      DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}

  # ✅ Good - mask custom values
  - run: |
      echo "::add-mask::$MY_SECRET"
      echo "$MY_SECRET"
```

### **Code Scanning**
```yaml
# Enable CodeQL
- uses: github/codeql-action/init@v3
  with:
    languages: javascript, python

- uses: github/codeql-action/autobuild@v3

- uses: github/codeql-action/analyze@v3
```

---

## 🎯 Common Patterns

### **Build Once, Deploy Everywhere**
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: build
          path: dist/
  
  deploy-dev:
    needs: build
    runs-on: ubuntu-latest
    environment: dev
    steps:
      - uses: actions/download-artifact@v4
      - run: ./deploy.sh dev
  
  deploy-prod:
    needs: build
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/download-artifact@v4
      - run: ./deploy.sh production
```

### **Conditional Deployment**
```yaml
jobs:
  deploy:
    if: |
      github.event_name == 'push' &&
      github.ref == 'refs/heads/main' &&
      !contains(github.event.head_commit.message, '[skip ci]')
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh
```

### **Retry on Failure**
```yaml
- uses: nick-fields/retry@v2
  with:
    timeout_minutes: 10
    max_attempts: 3
    retry_wait_seconds: 30
    command: npm test
```

### **Concurrency Control**
```yaml
# Cancel in-progress runs
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

# Don't cancel (for deployments)
concurrency:
  group: production-deploy
  cancel-in-progress: false
```

### **Dynamic Matrix**
```yaml
jobs:
  setup:
    runs-on: ubuntu-latest
    outputs:
      matrix: ${{ steps.set-matrix.outputs.matrix }}
    steps:
      - id: set-matrix
        run: |
          echo "matrix={\"node\":[16,18,20]}" >> $GITHUB_OUTPUT
  
  test:
    needs: setup
    runs-on: ubuntu-latest
    strategy:
      matrix: ${{ fromJSON(needs.setup.outputs.matrix) }}
    steps:
      - run: echo "Testing Node ${{ matrix.node }}"
```

---

## 📋 GitHub CLI in Actions

```yaml
steps:
  # CLI is pre-installed and authenticated
  - run: gh --version
  
  # Create issue
  - run: gh issue create --title "Bug" --body "Description"
    env:
      GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  
  # Comment on PR
  - run: |
      gh pr comment ${{ github.event.pull_request.number }} \
        --body "✅ Build succeeded!"
  
  # Create release
  - run: |
      gh release create v${{ github.run_number }} \
        --title "Release v${{ github.run_number }}" \
        --notes "Automated release"
  
  # Approve PR
  - run: gh pr review ${{ github.event.pull_request.number }} --approve
```

---

## 🔍 Debugging Workflows

### **Enable Debug Logging**
```bash
# Set repository secrets:
ACTIONS_RUNNER_DEBUG=true
ACTIONS_STEP_DEBUG=true
```

### **Debug Steps**
```yaml
steps:
  # Print all context
  - run: echo '${{ toJSON(github) }}'
  
  # Print environment variables
  - run: env | sort
  
  # List files
  - run: ls -la
  
  # Check working directory
  - run: pwd
  
  # Debug action
  - uses: actions/checkout@v4
    with:
      fetch-depth: 0
  - run: |
      echo "Branch: $(git branch --show-current)"
      echo "Commits: $(git log --oneline -5)"
```

---

## 📝 Complete Example Workflow

```yaml
name: Complete CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  packages: write
  pull-requests: write

env:
  NODE_VERSION: '20'
  REGISTRY: ghcr.io

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - run: npm ci
      - run: npm run lint
  
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
        node-version: [18, 20]
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      - run: npm ci
      - run: npm test
      
      - uses: codecov/codecov-action@v3
        if: matrix.os == 'ubuntu-latest' && matrix.node-version == '20'
  
  build:
    needs: [lint, test]
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.version.outputs.version }}
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - id: version
        run: echo "version=$(node -p "require('./package.json').version")" >> $GITHUB_OUTPUT
      
      - run: npm ci
      - run: npm run build
      
      - uses: actions/upload-artifact@v4
        with:
          name: build-${{ github.sha }}
          path: dist/
          retention-days: 7
  
  docker:
    needs: build
    if: github.event_name == 'push'
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: docker/setup-buildx-action@v3
      
      - uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            ${{ env.REGISTRY }}/${{ github.repository }}:latest
            ${{ env.REGISTRY }}/${{ github.repository }}:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
  
  deploy:
    needs: [build, docker]
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://app.example.com
    
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build-${{ github.sha }}
      
      - name: Deploy to production
        run: echo "Deploying version ${{ needs.build.outputs.version }}"
        env:
          DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
      
      - name: Notify Slack
        if: always()
        uses: slackapi/slack-github-action@v1
        with:
          webhook-url: ${{ secrets.SLACK_WEBHOOK }}
          payload: |
            {
              "text": "Deployment ${{ job.status }}: ${{ github.repository }}@${{ github.sha }}"
            }
```

---

**Quick reference complete! Bookmark this for daily GitHub Actions work! 🚀**

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

