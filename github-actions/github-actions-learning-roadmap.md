# GitHub Actions Learning Roadmap

**10-Week Comprehensive Curriculum: From Basics to Production Mastery**

---

## 🎯 Learning Objectives

By completing this roadmap, you will:
- Master GitHub Actions workflow creation and management
- Build production-ready CI/CD pipelines
- Create reusable workflows and custom actions
- Implement security best practices
- Optimize workflow performance and costs
- Deploy to multiple cloud platforms
- Troubleshoot complex workflow issues

**Target Audience:** DevOps Engineers, Software Engineers, SREs (3-7+ years experience)

**Prerequisites:**
- Git fundamentals
- Basic YAML syntax
- CI/CD concepts
- Docker basics
- Cloud platform knowledge (AWS/Azure/GCP)

---

## 📅 10-Week Learning Schedule

| Week | Focus Area | Time Commitment |
|------|-----------|----------------|
| 1-2 | Fundamentals & Basic Workflows | 10-12 hours/week |
| 3-4 | Advanced Workflows & Reusability | 10-12 hours/week |
| 5-6 | CI/CD Pipelines & Multi-Platform | 12-15 hours/week |
| 7-8 | Security, Secrets & Self-Hosted Runners | 10-12 hours/week |
| 9-10 | Production Patterns & Optimization | 12-15 hours/week |

**Total Learning Time:** ~110-130 hours over 10 weeks

---

## 📚 WEEK 1-2: GitHub Actions Fundamentals

### **Week 1: Core Concepts & Basic Workflows**

#### **Day 1-2: Introduction to GitHub Actions**
- **Concepts:**
  - What is GitHub Actions and why use it?
  - GitHub Actions vs other CI/CD tools
  - GitHub Actions pricing and limits
  - Architecture: workflows, jobs, steps, runners
  - Events and triggers

- **Hands-On:**
  ```yaml
  # Create your first workflow
  name: Hello GitHub Actions
  
  on: [push]
  
  jobs:
    hello-world:
      runs-on: ubuntu-latest
      steps:
        - name: Say hello
          run: echo "Hello, GitHub Actions!"
  ```

- **Practice:**
  - Create a simple workflow triggered by push
  - Add multiple jobs
  - Experiment with different `run` commands

#### **Day 3-4: Workflow Syntax & Events**
- **Concepts:**
  - YAML syntax for workflows
  - Common events: `push`, `pull_request`, `schedule`
  - Event filters: branches, paths, tags
  - Manual triggers with `workflow_dispatch`
  - Webhook events

- **Hands-On:**
  ```yaml
  name: Multi-Event Workflow
  
  on:
    push:
      branches: [ main, develop ]
      paths:
        - 'src/**'
    pull_request:
      branches: [ main ]
    schedule:
      - cron: '0 2 * * *'  # Daily at 2 AM
    workflow_dispatch:
      inputs:
        environment:
          description: 'Environment to deploy'
          required: true
          default: 'staging'
  ```

- **Practice:**
  - Create workflows with multiple triggers
  - Use branch and path filters
  - Set up a scheduled workflow
  - Add workflow_dispatch with inputs

#### **Day 5-6: Actions & Marketplace**
- **Concepts:**
  - Using actions from GitHub Marketplace
  - Action versioning strategies
  - Common actions: checkout, setup-*
  - Action inputs and outputs
  - JavaScript vs Docker actions

- **Hands-On:**
  ```yaml
  name: Using Marketplace Actions
  
  on: [push]
  
  jobs:
    build:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
        
        - uses: actions/setup-node@v4
          with:
            node-version: '20'
            cache: 'npm'
        
        - run: npm ci
        - run: npm test
        
        - uses: actions/upload-artifact@v4
          with:
            name: test-results
            path: test-results/
  ```

- **Practice:**
  - Use checkout action
  - Set up language runtimes (Node, Python, Java)
  - Upload and download artifacts
  - Explore marketplace actions

#### **Day 7: Context & Expressions**
- **Concepts:**
  - GitHub context: `github`, `env`, `job`, `steps`
  - Expression syntax: `${{ }}`
  - Functions: `contains`, `startsWith`, `endsWith`
  - Environment variables
  - Default environment variables

- **Hands-On:**
  ```yaml
  name: Using Contexts
  
  on: [push]
  
  jobs:
    info:
      runs-on: ubuntu-latest
      steps:
        - name: Show context information
          run: |
            echo "Repository: ${{ github.repository }}"
            echo "Branch: ${{ github.ref }}"
            echo "Actor: ${{ github.actor }}"
            echo "SHA: ${{ github.sha }}"
            echo "Run ID: ${{ github.run_id }}"
        
        - name: Conditional step
          if: github.ref == 'refs/heads/main'
          run: echo "Running on main branch!"
  ```

- **Practice:**
  - Access different context variables
  - Use expressions in conditionals
  - Set custom environment variables
  - Practice expression functions

---

### **Week 2: Jobs, Dependencies & Artifacts**

#### **Day 1-2: Job Dependencies & Outputs**
- **Concepts:**
  - Job dependencies with `needs`
  - Job outputs
  - Job matrix strategies
  - Job conditionals
  - Job container

- **Hands-On:**
  ```yaml
  name: Job Dependencies
  
  on: [push]
  
  jobs:
    build:
      runs-on: ubuntu-latest
      outputs:
        version: ${{ steps.get_version.outputs.version }}
      steps:
        - uses: actions/checkout@v4
        - id: get_version
          run: echo "version=$(cat version.txt)" >> $GITHUB_OUTPUT
    
    test:
      needs: build
      runs-on: ubuntu-latest
      steps:
        - run: echo "Testing version ${{ needs.build.outputs.version }}"
    
    deploy:
      needs: [build, test]
      runs-on: ubuntu-latest
      if: github.ref == 'refs/heads/main'
      steps:
        - run: echo "Deploying version ${{ needs.build.outputs.version }}"
  ```

- **Practice:**
  - Create dependent jobs
  - Pass data between jobs using outputs
  - Implement conditional job execution
  - Use job containers

#### **Day 3-4: Matrix Strategies**
- **Concepts:**
  - Matrix builds for multi-version testing
  - Including and excluding matrix combinations
  - Matrix context variables
  - Fail-fast strategy
  - Max-parallel setting

- **Hands-On:**
  ```yaml
  name: Matrix Build
  
  on: [push]
  
  jobs:
    test:
      runs-on: ${{ matrix.os }}
      strategy:
        matrix:
          os: [ubuntu-latest, windows-latest, macos-latest]
          node-version: [16, 18, 20]
          include:
            - os: ubuntu-latest
              node-version: 20
              experimental: true
          exclude:
            - os: macos-latest
              node-version: 16
        fail-fast: false
        max-parallel: 3
      
      steps:
        - uses: actions/checkout@v4
        - uses: actions/setup-node@v4
          with:
            node-version: ${{ matrix.node-version }}
        - run: npm ci
        - run: npm test
  ```

- **Practice:**
  - Create multi-OS matrix builds
  - Test across multiple language versions
  - Use include/exclude for complex matrices
  - Experiment with fail-fast

#### **Day 5-6: Artifacts & Caching**
- **Concepts:**
  - Uploading artifacts
  - Downloading artifacts
  - Artifact retention
  - Caching dependencies
  - Cache keys and restore keys
  - Cache limits

- **Hands-On:**
  ```yaml
  name: Artifacts and Caching
  
  on: [push]
  
  jobs:
    build:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
        
        - uses: actions/cache@v4
          with:
            path: ~/.npm
            key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
            restore-keys: |
              ${{ runner.os }}-node-
        
        - uses: actions/setup-node@v4
          with:
            node-version: '20'
        
        - run: npm ci
        - run: npm run build
        
        - uses: actions/upload-artifact@v4
          with:
            name: build-output
            path: dist/
            retention-days: 7
    
    deploy:
      needs: build
      runs-on: ubuntu-latest
      steps:
        - uses: actions/download-artifact@v4
          with:
            name: build-output
            path: dist/
        
        - run: ls -la dist/
  ```

- **Practice:**
  - Upload build artifacts
  - Download artifacts in dependent jobs
  - Implement dependency caching
  - Optimize cache keys

#### **Day 7: Week 1-2 Review & Project**
- **Mini Project:** Build a complete CI pipeline
  - Multi-stage build (lint, test, build)
  - Matrix testing across OS and versions
  - Artifact management
  - Caching optimization

---

## 📚 WEEK 3-4: Advanced Workflows & Reusability

### **Week 3: Reusable Workflows & Composite Actions**

#### **Day 1-2: Reusable Workflows**
- **Concepts:**
  - Creating reusable workflows
  - Calling reusable workflows
  - Passing inputs and secrets
  - Outputs from reusable workflows
  - Limitations and best practices

- **Hands-On:**
  ```yaml
  # .github/workflows/reusable-build.yml
  name: Reusable Build Workflow
  
  on:
    workflow_call:
      inputs:
        node-version:
          required: true
          type: string
        environment:
          required: true
          type: string
      secrets:
        deploy-token:
          required: true
      outputs:
        build-id:
          description: "Build identifier"
          value: ${{ jobs.build.outputs.build-id }}
  
  jobs:
    build:
      runs-on: ubuntu-latest
      outputs:
        build-id: ${{ steps.build.outputs.id }}
      steps:
        - uses: actions/checkout@v4
        - uses: actions/setup-node@v4
          with:
            node-version: ${{ inputs.node-version }}
        - run: npm ci
        - run: npm run build
        - id: build
          run: echo "id=${{ github.run_number }}" >> $GITHUB_OUTPUT
  
  # .github/workflows/main.yml
  name: Main Workflow
  
  on: [push]
  
  jobs:
    call-build:
      uses: ./.github/workflows/reusable-build.yml
      with:
        node-version: '20'
        environment: 'production'
      secrets:
        deploy-token: ${{ secrets.DEPLOY_TOKEN }}
  ```

- **Practice:**
  - Create reusable workflow for common tasks
  - Call reusable workflows from multiple workflows
  - Pass inputs and secrets
  - Use outputs from reusable workflows

#### **Day 3-4: Composite Actions**
- **Concepts:**
  - Creating composite actions
  - Action metadata (action.yml)
  - Inputs, outputs, and runs
  - Sharing composite actions
  - Local vs published actions

- **Hands-On:**
  ```yaml
  # .github/actions/setup-project/action.yml
  name: 'Setup Project'
  description: 'Setup Node.js project with caching'
  inputs:
    node-version:
      description: 'Node.js version'
      required: true
      default: '20'
    working-directory:
      description: 'Working directory'
      required: false
      default: '.'
  outputs:
    cache-hit:
      description: 'Whether cache was hit'
      value: ${{ steps.cache.outputs.cache-hit }}
  
  runs:
    using: 'composite'
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
      
      - id: cache
        uses: actions/cache@v4
        with:
          path: ~/.npm
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
      
      - run: npm ci
        shell: bash
        working-directory: ${{ inputs.working-directory }}
  
  # Using the composite action
  # .github/workflows/use-composite.yml
  name: Use Composite Action
  
  on: [push]
  
  jobs:
    build:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
        - uses: ./.github/actions/setup-project
          with:
            node-version: '20'
        - run: npm test
  ```

- **Practice:**
  - Create composite actions for repeated tasks
  - Define inputs and outputs
  - Use composite actions in workflows
  - Share actions across repositories

#### **Day 5-6: Secrets Management**
- **Concepts:**
  - Repository secrets
  - Organization secrets
  - Environment secrets
  - Using secrets in workflows
  - Secret security best practices
  - Masked values in logs

- **Hands-On:**
  ```yaml
  name: Secrets Management
  
  on: [push]
  
  jobs:
    deploy:
      runs-on: ubuntu-latest
      environment: production
      steps:
        - uses: actions/checkout@v4
        
        - name: Deploy to server
          env:
            SSH_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
            SERVER_HOST: ${{ secrets.SERVER_HOST }}
            SERVER_USER: ${{ secrets.SERVER_USER }}
          run: |
            echo "$SSH_KEY" > key.pem
            chmod 600 key.pem
            ssh -i key.pem $SERVER_USER@$SERVER_HOST "deploy.sh"
        
        - name: Notify Slack
          uses: slackapi/slack-github-action@v1
          with:
            webhook-url: ${{ secrets.SLACK_WEBHOOK_URL }}
            payload: |
              {
                "text": "Deployment completed for ${{ github.repository }}"
              }
  ```

- **Practice:**
  - Create and use repository secrets
  - Set up environment-specific secrets
  - Use secrets securely in workflows
  - Rotate secrets regularly

#### **Day 7: Environments & Approvals**
- **Concepts:**
  - Creating environments
  - Environment protection rules
  - Required reviewers
  - Wait timer
  - Environment-specific secrets
  - Deployment branches

- **Hands-On:**
  ```yaml
  name: Multi-Environment Deployment
  
  on:
    push:
      branches: [ main ]
  
  jobs:
    deploy-staging:
      runs-on: ubuntu-latest
      environment: staging
      steps:
        - uses: actions/checkout@v4
        - name: Deploy to staging
          run: echo "Deploying to staging..."
    
    deploy-production:
      needs: deploy-staging
      runs-on: ubuntu-latest
      environment:
        name: production
        url: https://prod.example.com
      steps:
        - uses: actions/checkout@v4
        - name: Deploy to production
          run: echo "Deploying to production..."
  ```

- **Practice:**
  - Create multiple environments
  - Set up approval gates
  - Configure environment secrets
  - Implement deployment strategies

---

### **Week 4: Custom JavaScript Actions**

#### **Day 1-3: JavaScript Action Basics**
- **Concepts:**
  - JavaScript action structure
  - Action toolkit (@actions/core, @actions/github)
  - Input handling
  - Output setting
  - Error handling
  - Logging

- **Hands-On:**
  ```javascript
  // index.js
  const core = require('@actions/core');
  const github = require('@actions/github');
  
  async function run() {
    try {
      // Get inputs
      const nameToGreet = core.getInput('who-to-greet', { required: true });
      const timeOfDay = core.getInput('time-of-day') || 'day';
      
      console.log(`Hello ${nameToGreet}! Good ${timeOfDay}!`);
      
      // Get GitHub context
      const context = github.context;
      console.log(`Repository: ${context.repo.owner}/${context.repo.repo}`);
      
      // Set outputs
      const time = (new Date()).toTimeString();
      core.setOutput('time', time);
      
      // Export variable
      core.exportVariable('GREETING_TIME', time);
      
      // Set secret (will be masked in logs)
      core.setSecret(nameToGreet);
      
    } catch (error) {
      core.setFailed(error.message);
    }
  }
  
  run();
  ```

  ```yaml
  # action.yml
  name: 'Greeting Action'
  description: 'Greet someone and record the time'
  inputs:
    who-to-greet:
      description: 'Who to greet'
      required: true
      default: 'World'
    time-of-day:
      description: 'Time of day'
      required: false
  outputs:
    time:
      description: 'The time we greeted you'
  runs:
    using: 'node20'
    main: 'dist/index.js'
  ```

- **Practice:**
  - Create a basic JavaScript action
  - Handle inputs and outputs
  - Use GitHub API
  - Publish to marketplace

#### **Day 4-7: Advanced JavaScript Actions**
- **Concepts:**
  - Octokit for GitHub API
  - Working with issues and PRs
  - Creating releases
  - Managing labels
  - Action distribution (ncc)
  - Testing actions

- **Hands-On:**
  ```javascript
  // Advanced action example
  const core = require('@actions/core');
  const github = require('@actions/github');
  
  async function run() {
    try {
      const token = core.getInput('github-token', { required: true });
      const octokit = github.getOctokit(token);
      
      const context = github.context;
      
      // Create a comment on PR
      if (context.payload.pull_request) {
        await octokit.rest.issues.createComment({
          ...context.repo,
          issue_number: context.payload.pull_request.number,
          body: '✅ CI passed! Ready for review.'
        });
      }
      
      // Add label to issue
      if (context.payload.issue) {
        await octokit.rest.issues.addLabels({
          ...context.repo,
          issue_number: context.payload.issue.number,
          labels: ['automated']
        });
      }
      
    } catch (error) {
      core.setFailed(error.message);
    }
  }
  
  run();
  ```

- **Practice:**
  - Use Octokit to interact with GitHub API
  - Create PR automation
  - Build issue management actions
  - Package action with @vercel/ncc

---

## 📚 WEEK 5-6: CI/CD Pipelines & Deployments

### **Week 5: Language-Specific CI/CD**

#### **Day 1: Node.js / JavaScript**
- **Complete CI/CD Pipeline:**
  ```yaml
  name: Node.js CI/CD
  
  on:
    push:
      branches: [ main, develop ]
    pull_request:
      branches: [ main ]
  
  jobs:
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
        - run: npm run lint
        - run: npm test
        - run: npm run build
        
        - uses: codecov/codecov-action@v3
          with:
            files: ./coverage/lcov.info
    
    security:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
        - run: npm audit
        - uses: snyk/actions/node@master
          env:
            SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
    
    publish:
      needs: [test, security]
      if: github.ref == 'refs/heads/main'
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
        - uses: actions/setup-node@v4
          with:
            node-version: '20'
            registry-url: 'https://registry.npmjs.org'
        - run: npm ci
        - run: npm publish
          env:
            NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
  ```

#### **Day 2: Python**
- **Complete CI/CD Pipeline:**
  ```yaml
  name: Python CI/CD
  
  on: [push, pull_request]
  
  jobs:
    test:
      runs-on: ${{ matrix.os }}
      strategy:
        matrix:
          os: [ubuntu-latest, macos-latest, windows-latest]
          python-version: ['3.9', '3.10', '3.11', '3.12']
      
      steps:
        - uses: actions/checkout@v4
        
        - uses: actions/setup-python@v5
          with:
            python-version: ${{ matrix.python-version }}
            cache: 'pip'
        
        - name: Install dependencies
          run: |
            python -m pip install --upgrade pip
            pip install -r requirements.txt
            pip install -r requirements-dev.txt
        
        - name: Lint with flake8
          run: flake8 . --count --show-source --statistics
        
        - name: Type check with mypy
          run: mypy .
        
        - name: Test with pytest
          run: pytest --cov=. --cov-report=xml
        
        - uses: codecov/codecov-action@v3
    
    publish:
      needs: test
      if: startsWith(github.ref, 'refs/tags/v')
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
        - uses: actions/setup-python@v5
          with:
            python-version: '3.12'
        - run: pip install build twine
        - run: python -m build
        - run: twine upload dist/*
          env:
            TWINE_USERNAME: __token__
            TWINE_PASSWORD: ${{ secrets.PYPI_TOKEN }}
  ```

#### **Day 3: Java / Maven**
- **Complete CI/CD Pipeline:**
  ```yaml
  name: Java CI/CD
  
  on: [push, pull_request]
  
  jobs:
    build:
      runs-on: ubuntu-latest
      
      steps:
        - uses: actions/checkout@v4
        
        - uses: actions/setup-java@v4
          with:
            distribution: 'temurin'
            java-version: '17'
            cache: 'maven'
        
        - name: Build with Maven
          run: mvn clean install -B
        
        - name: Run tests
          run: mvn test -B
        
        - name: Generate coverage report
          run: mvn jacoco:report
        
        - name: SonarCloud Scan
          uses: SonarSource/sonarcloud-github-action@master
          env:
            GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
            SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        
        - uses: actions/upload-artifact@v4
          with:
            name: jar-file
            path: target/*.jar
    
    deploy:
      needs: build
      if: github.ref == 'refs/heads/main'
      runs-on: ubuntu-latest
      steps:
        - uses: actions/download-artifact@v4
          with:
            name: jar-file
        - name: Deploy to server
          run: echo "Deploying JAR..."
  ```

#### **Day 4: Go**
- **Complete CI/CD Pipeline:**
  ```yaml
  name: Go CI/CD
  
  on: [push, pull_request]
  
  jobs:
    test:
      runs-on: ${{ matrix.os }}
      strategy:
        matrix:
          os: [ubuntu-latest, macos-latest, windows-latest]
          go-version: ['1.21', '1.22']
      
      steps:
        - uses: actions/checkout@v4
        
        - uses: actions/setup-go@v5
          with:
            go-version: ${{ matrix.go-version }}
            cache: true
        
        - run: go mod download
        - run: go vet ./...
        - run: go test -v -race -coverprofile=coverage.txt -covermode=atomic ./...
        
        - uses: codecov/codecov-action@v3
    
    build:
      needs: test
      runs-on: ubuntu-latest
      strategy:
        matrix:
          goos: [linux, windows, darwin]
          goarch: [amd64, arm64]
      
      steps:
        - uses: actions/checkout@v4
        - uses: actions/setup-go@v5
          with:
            go-version: '1.22'
        
        - name: Build
          env:
            GOOS: ${{ matrix.goos }}
            GOARCH: ${{ matrix.goarch }}
          run: go build -v -o myapp-${{ matrix.goos }}-${{ matrix.goarch }} .
        
        - uses: actions/upload-artifact@v4
          with:
            name: myapp-${{ matrix.goos }}-${{ matrix.goarch }}
            path: myapp-*
  ```

#### **Day 5: .NET**
- **Complete CI/CD Pipeline:**
  ```yaml
  name: .NET CI/CD
  
  on: [push, pull_request]
  
  jobs:
    build:
      runs-on: ${{ matrix.os }}
      strategy:
        matrix:
          os: [ubuntu-latest, windows-latest, macos-latest]
          dotnet-version: ['6.0.x', '7.0.x', '8.0.x']
      
      steps:
        - uses: actions/checkout@v4
        
        - uses: actions/setup-dotnet@v4
          with:
            dotnet-version: ${{ matrix.dotnet-version }}
        
        - run: dotnet restore
        - run: dotnet build --no-restore
        - run: dotnet test --no-build --verbosity normal --collect:"XPlat Code Coverage"
        
        - name: Code Coverage Report
          uses: codecov/codecov-action@v3
          with:
            files: '**/coverage.cobertura.xml'
    
    publish:
      needs: build
      if: github.ref == 'refs/heads/main'
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
        - uses: actions/setup-dotnet@v4
          with:
            dotnet-version: '8.0.x'
        - run: dotnet publish -c Release -o ./publish
        - uses: actions/upload-artifact@v4
          with:
            name: published-app
            path: ./publish
  ```

#### **Day 6-7: Docker & Container Workflows**
- **Complete Docker CI/CD:**
  ```yaml
  name: Docker Build and Push
  
  on:
    push:
      branches: [ main ]
    pull_request:
      branches: [ main ]
  
  env:
    REGISTRY: ghcr.io
    IMAGE_NAME: ${{ github.repository }}
  
  jobs:
    build-and-push:
      runs-on: ubuntu-latest
      permissions:
        contents: read
        packages: write
      
      steps:
        - uses: actions/checkout@v4
        
        - name: Set up Docker Buildx
          uses: docker/setup-buildx-action@v3
        
        - name: Log in to Container Registry
          uses: docker/login-action@v3
          with:
            registry: ${{ env.REGISTRY }}
            username: ${{ github.actor }}
            password: ${{ secrets.GITHUB_TOKEN }}
        
        - name: Extract metadata
          id: meta
          uses: docker/metadata-action@v5
          with:
            images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
            tags: |
              type=ref,event=branch
              type=ref,event=pr
              type=semver,pattern={{version}}
              type=semver,pattern={{major}}.{{minor}}
              type=sha,prefix={{branch}}-
        
        - name: Build and push
          uses: docker/build-push-action@v5
          with:
            context: .
            push: ${{ github.event_name != 'pull_request' }}
            tags: ${{ steps.meta.outputs.tags }}
            labels: ${{ steps.meta.outputs.labels }}
            cache-from: type=gha
            cache-to: type=gha,mode=max
        
        - name: Scan image
          uses: aquasecurity/trivy-action@master
          with:
            image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
            format: 'sarif'
            output: 'trivy-results.sarif'
        
        - uses: github/codeql-action/upload-sarif@v3
          with:
            sarif_file: 'trivy-results.sarif'
  ```

---

### **Week 6: Cloud Deployments**

#### **Day 1-2: AWS Deployments**
- **Deploy to AWS ECS:**
  ```yaml
  name: Deploy to AWS ECS
  
  on:
    push:
      branches: [ main ]
  
  env:
    AWS_REGION: us-east-1
    ECR_REPOSITORY: myapp
    ECS_SERVICE: myapp-service
    ECS_CLUSTER: myapp-cluster
    ECS_TASK_DEFINITION: .aws/task-definition.json
    CONTAINER_NAME: myapp
  
  jobs:
    deploy:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
        
        - uses: aws-actions/configure-aws-credentials@v4
          with:
            aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
            aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
            aws-region: ${{ env.AWS_REGION }}
        
        - name: Login to Amazon ECR
          id: login-ecr
          uses: aws-actions/amazon-ecr-login@v2
        
        - name: Build and push image
          id: build-image
          env:
            ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
            IMAGE_TAG: ${{ github.sha }}
          run: |
            docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
            docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
            echo "image=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_OUTPUT
        
        - name: Fill in new image ID in task definition
          id: task-def
          uses: aws-actions/amazon-ecs-render-task-definition@v1
          with:
            task-definition: ${{ env.ECS_TASK_DEFINITION }}
            container-name: ${{ env.CONTAINER_NAME }}
            image: ${{ steps.build-image.outputs.image }}
        
        - name: Deploy to Amazon ECS
          uses: aws-actions/amazon-ecs-deploy-task-definition@v1
          with:
            task-definition: ${{ steps.task-def.outputs.task-definition }}
            service: ${{ env.ECS_SERVICE }}
            cluster: ${{ env.ECS_CLUSTER }}
            wait-for-service-stability: true
  ```

#### **Day 3-4: Azure Deployments**
- **Deploy to Azure AKS:**
  ```yaml
  name: Deploy to Azure AKS
  
  on:
    push:
      branches: [ main ]
  
  env:
    AZURE_CONTAINER_REGISTRY: myacr
    RESOURCE_GROUP: myapp-rg
    CLUSTER_NAME: myapp-aks
    DEPLOYMENT_NAME: myapp
    NAMESPACE: production
  
  jobs:
    deploy:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
        
        - uses: azure/login@v1
          with:
            creds: ${{ secrets.AZURE_CREDENTIALS }}
        
        - uses: azure/docker-login@v1
          with:
            login-server: ${{ env.AZURE_CONTAINER_REGISTRY }}.azurecr.io
            username: ${{ secrets.REGISTRY_USERNAME }}
            password: ${{ secrets.REGISTRY_PASSWORD }}
        
        - name: Build and push image
          run: |
            docker build -t ${{ env.AZURE_CONTAINER_REGISTRY }}.azurecr.io/myapp:${{ github.sha }} .
            docker push ${{ env.AZURE_CONTAINER_REGISTRY }}.azurecr.io/myapp:${{ github.sha }}
        
        - uses: azure/aks-set-context@v3
          with:
            resource-group: ${{ env.RESOURCE_GROUP }}
            cluster-name: ${{ env.CLUSTER_NAME }}
        
        - uses: azure/k8s-create-secret@v4
          with:
            namespace: ${{ env.NAMESPACE }}
            secret-type: 'docker-registry'
            secret-name: acr-secret
            docker-registry-url: ${{ env.AZURE_CONTAINER_REGISTRY }}.azurecr.io
            docker-username: ${{ secrets.REGISTRY_USERNAME }}
            docker-password: ${{ secrets.REGISTRY_PASSWORD }}
        
        - uses: azure/k8s-deploy@v4
          with:
            namespace: ${{ env.NAMESPACE }}
            manifests: |
              k8s/deployment.yml
              k8s/service.yml
            images: |
              ${{ env.AZURE_CONTAINER_REGISTRY }}.azurecr.io/myapp:${{ github.sha }}
            imagepullsecrets: |
              acr-secret
  ```

#### **Day 5-7: Kubernetes & Helm Deployments**
- **Complete Kubernetes Deployment:**
  ```yaml
  name: Deploy to Kubernetes with Helm
  
  on:
    push:
      branches: [ main ]
  
  jobs:
    deploy:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
        
        - name: Install kubectl
          uses: azure/setup-kubectl@v3
        
        - name: Install Helm
          uses: azure/setup-helm@v3
        
        - name: Configure kubeconfig
          run: |
            echo "${{ secrets.KUBE_CONFIG }}" | base64 -d > kubeconfig
            export KUBECONFIG=kubeconfig
        
        - name: Add Helm repo
          run: helm repo add myrepo https://charts.example.com
        
        - name: Update Helm dependencies
          run: helm dependency update ./helm/myapp
        
        - name: Deploy with Helm
          run: |
            helm upgrade --install myapp ./helm/myapp \
              --namespace production \
              --create-namespace \
              --set image.tag=${{ github.sha }} \
              --set ingress.host=myapp.example.com \
              --wait \
              --timeout 5m
        
        - name: Verify deployment
          run: |
            kubectl rollout status deployment/myapp -n production
            kubectl get pods -n production -l app=myapp
  ```

---

## 📚 WEEK 7-8: Security & Self-Hosted Runners

### **Week 7: Security Best Practices**

#### **Day 1-2: Code Scanning & SAST**
- **CodeQL Analysis:**
  ```yaml
  name: "CodeQL Security Scan"
  
  on:
    push:
      branches: [ main, develop ]
    pull_request:
      branches: [ main ]
    schedule:
      - cron: '0 0 * * 0'  # Weekly
  
  jobs:
    analyze:
      runs-on: ubuntu-latest
      permissions:
        actions: read
        contents: read
        security-events: write
      
      strategy:
        matrix:
          language: [ 'javascript', 'python' ]
      
      steps:
        - uses: actions/checkout@v4
        
        - uses: github/codeql-action/init@v3
          with:
            languages: ${{ matrix.language }}
        
        - uses: github/codeql-action/autobuild@v3
        
        - uses: github/codeql-action/analyze@v3
          with:
            category: "/language:${{ matrix.language }}"
  ```

#### **Day 3-4: Dependency Scanning**
- **Complete Dependency Security:**
  ```yaml
  name: Dependency Security Scan
  
  on:
    push:
      branches: [ main ]
    pull_request:
      branches: [ main ]
    schedule:
      - cron: '0 0 * * 1'  # Weekly on Monday
  
  jobs:
    dependency-review:
      runs-on: ubuntu-latest
      if: github.event_name == 'pull_request'
      steps:
        - uses: actions/checkout@v4
        - uses: actions/dependency-review-action@v4
          with:
            fail-on-severity: moderate
    
    snyk-scan:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
        
        - name: Run Snyk to check for vulnerabilities
          uses: snyk/actions/node@master
          env:
            SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
          with:
            args: --severity-threshold=high
        
        - uses: snyk/actions/node@master
          env:
            SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
          with:
            command: monitor
    
    trivy-scan:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
        
        - name: Run Trivy vulnerability scanner
          uses: aquasecurity/trivy-action@master
          with:
            scan-type: 'fs'
            scan-ref: '.'
            format: 'sarif'
            output: 'trivy-results.sarif'
        
        - uses: github/codeql-action/upload-sarif@v3
          with:
            sarif_file: 'trivy-results.sarif'
  ```

#### **Day 5-7: OIDC & Secure Cloud Authentication**
- **AWS OIDC Authentication:**
  ```yaml
  name: AWS OIDC Deployment
  
  on:
    push:
      branches: [ main ]
  
  permissions:
    id-token: write
    contents: read
  
  jobs:
    deploy:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
        
        - uses: aws-actions/configure-aws-credentials@v4
          with:
            role-to-assume: arn:aws:iam::123456789012:role/GitHubActionsRole
            aws-region: us-east-1
        
        - run: aws s3 cp ./build s3://my-bucket/ --recursive
  ```

- **Azure OIDC Authentication:**
  ```yaml
  name: Azure OIDC Deployment
  
  on:
    push:
      branches: [ main ]
  
  permissions:
    id-token: write
    contents: read
  
  jobs:
    deploy:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
        
        - uses: azure/login@v1
          with:
            client-id: ${{ secrets.AZURE_CLIENT_ID }}
            tenant-id: ${{ secrets.AZURE_TENANT_ID }}
            subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
        
        - run: az webapp deploy --resource-group myapp-rg --name myapp
  ```

---

### **Week 8: Self-Hosted Runners**

#### **Day 1-3: Setting Up Self-Hosted Runners**
- **Linux Runner Setup:**
  ```bash
  # Download and configure runner
  mkdir actions-runner && cd actions-runner
  curl -o actions-runner-linux-x64-2.311.0.tar.gz -L \
    https://github.com/actions/runner/releases/download/v2.311.0/actions-runner-linux-x64-2.311.0.tar.gz
  tar xzf ./actions-runner-linux-x64-2.311.0.tar.gz
  
  # Configure
  ./config.sh --url https://github.com/myorg/myrepo --token <TOKEN>
  
  # Install as service
  sudo ./svc.sh install
  sudo ./svc.sh start
  ```

- **Using Self-Hosted Runner:**
  ```yaml
  name: Self-Hosted Runner
  
  on: [push]
  
  jobs:
    build:
      runs-on: self-hosted
      steps:
        - uses: actions/checkout@v4
        - run: make build
  ```

#### **Day 4-5: Runner Autoscaling**
- **Using Actions Runner Controller (ARC) on Kubernetes:**
  ```yaml
  # Install ARC with Helm
  helm repo add actions-runner-controller \
    https://actions-runner-controller.github.io/actions-runner-controller
  
  helm upgrade --install --namespace actions-runner-system \
    --create-namespace \
    actions-runner-controller \
    actions-runner-controller/actions-runner-controller \
    --set authSecret.github_token=$GITHUB_TOKEN
  
  # Create runner deployment
  apiVersion: actions.summerwind.dev/v1alpha1
  kind: RunnerDeployment
  metadata:
    name: myapp-runner
  spec:
    replicas: 2
    template:
      spec:
        repository: myorg/myrepo
        labels:
          - self-hosted
          - linux
          - x64
  ```

#### **Day 6-7: Runner Security & Optimization**
- **Secure Runner Configuration:**
  - Ephemeral runners
  - Runner groups
  - Network isolation
  - Access controls
  - Image hardening
  - Monitoring and logging

---

## 📚 WEEK 9-10: Production Patterns & Optimization

### **Week 9: Advanced Patterns**

#### **Day 1-2: Monorepo Workflows**
- **Path-based Triggering:**
  ```yaml
  name: Monorepo CI
  
  on:
    push:
      paths:
        - 'services/frontend/**'
        - 'services/backend/**'
        - 'packages/**'
  
  jobs:
    detect-changes:
      runs-on: ubuntu-latest
      outputs:
        frontend: ${{ steps.changes.outputs.frontend }}
        backend: ${{ steps.changes.outputs.backend }}
      steps:
        - uses: actions/checkout@v4
        - uses: dorny/paths-filter@v2
          id: changes
          with:
            filters: |
              frontend:
                - 'services/frontend/**'
              backend:
                - 'services/backend/**'
    
    build-frontend:
      needs: detect-changes
      if: needs.detect-changes.outputs.frontend == 'true'
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
        - name: Build frontend
          working-directory: ./services/frontend
          run: npm ci && npm run build
    
    build-backend:
      needs: detect-changes
      if: needs.detect-changes.outputs.backend == 'true'
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
        - name: Build backend
          working-directory: ./services/backend
          run: go build .
  ```

#### **Day 3-4: Release Automation**
- **Semantic Release:**
  ```yaml
  name: Release
  
  on:
    push:
      branches: [ main ]
  
  jobs:
    release:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
          with:
            fetch-depth: 0
            token: ${{ secrets.GITHUB_TOKEN }}
        
        - uses: actions/setup-node@v4
          with:
            node-version: '20'
        
        - run: npm ci
        
        - name: Semantic Release
          uses: cycjimmy/semantic-release-action@v4
          env:
            GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
            NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
  ```

#### **Day 5-7: GitOps Workflows**
- **Update Kubernetes Manifests:**
  ```yaml
  name: Update Kubernetes Manifests
  
  on:
    push:
      branches: [ main ]
  
  jobs:
    update-manifests:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
          with:
            repository: myorg/gitops-repo
            token: ${{ secrets.PAT }}
        
        - name: Update image tag
          run: |
            sed -i "s|image: myapp:.*|image: myapp:${{ github.sha }}|" \
              k8s/production/deployment.yaml
        
        - name: Commit and push
          run: |
            git config user.name "GitHub Actions"
            git config user.email "actions@github.com"
            git add k8s/production/deployment.yaml
            git commit -m "Update image to ${{ github.sha }}"
            git push
  ```

---

### **Week 10: Performance & Optimization**

#### **Day 1-3: Workflow Optimization**
- **Optimization Techniques:**
  - Dependency caching
  - Artifact caching
  - Parallel jobs
  - Job matrix optimization
  - Conditional execution
  - Workflow splitting

- **Example Optimized Workflow:**
  ```yaml
  name: Optimized CI
  
  on: [push, pull_request]
  
  jobs:
    setup:
      runs-on: ubuntu-latest
      outputs:
        cache-key: ${{ steps.cache-key.outputs.key }}
      steps:
        - uses: actions/checkout@v4
        - id: cache-key
          run: echo "key=${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}" >> $GITHUB_OUTPUT
    
    lint:
      needs: setup
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
        - uses: actions/setup-node@v4
          with:
            node-version: '20'
            cache: 'npm'
        - run: npm ci --prefer-offline
        - run: npm run lint
    
    test:
      needs: setup
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
        - uses: actions/setup-node@v4
          with:
            node-version: '20'
            cache: 'npm'
        - run: npm ci --prefer-offline
        - run: npm test
    
    build:
      needs: [lint, test]
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
        - uses: actions/setup-node@v4
          with:
            node-version: '20'
            cache: 'npm'
        - run: npm ci --prefer-offline
        - run: npm run build
        - uses: actions/cache@v4
          with:
            path: dist
            key: build-${{ github.sha }}
  ```

#### **Day 4-5: Cost Optimization**
- **Strategies:**
  - Use self-hosted runners for high-volume
  - Optimize caching
  - Skip redundant jobs
  - Use concurrency control
  - Monitor minutes usage

- **Concurrency Control:**
  ```yaml
  name: Deploy with Concurrency
  
  on:
    push:
      branches: [ main ]
  
  concurrency:
    group: production-deployment
    cancel-in-progress: false
  
  jobs:
    deploy:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
        - name: Deploy
          run: ./deploy.sh
  ```

#### **Day 6-7: Monitoring & Observability**
- **Workflow Monitoring:**
  - Workflow run analytics
  - Success/failure metrics
  - Performance tracking
  - Custom notifications

- **Advanced Notifications:**
  ```yaml
  name: Workflow with Notifications
  
  on: [push]
  
  jobs:
    build:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
        - run: npm ci
        - run: npm test
    
    notify:
      needs: build
      if: always()
      runs-on: ubuntu-latest
      steps:
        - name: Send Slack notification
          uses: slackapi/slack-github-action@v1
          with:
            webhook-url: ${{ secrets.SLACK_WEBHOOK }}
            payload: |
              {
                "text": "Workflow ${{ github.workflow }} - ${{ job.status }}",
                "blocks": [
                  {
                    "type": "section",
                    "text": {
                      "type": "mrkdwn",
                      "text": "*Workflow:* ${{ github.workflow }}\n*Status:* ${{ job.status }}\n*Repository:* ${{ github.repository }}\n*Branch:* ${{ github.ref_name }}\n*Commit:* ${{ github.sha }}"
                    }
                  }
                ]
              }
  ```

---

## 🎓 Learning Resources

### **Official Documentation**
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Workflow Syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [GitHub Actions Marketplace](https://github.com/marketplace?type=actions)

### **Community Resources**
- [Awesome Actions](https://github.com/sdras/awesome-actions)
- [GitHub Actions Community Forum](https://github.community/c/code-to-cloud/github-actions/)
- [GitHub Actions Skills](https://skills.github.com/)

### **Video Courses**
- GitHub Actions Official Training
- DevOps with GitHub Actions (Pluralsight)
- CI/CD with GitHub Actions (Udemy)

---

## ✅ Certification Path

While GitHub doesn't offer a specific GitHub Actions certification, consider:
1. **GitHub Foundations Certification**
2. **GitHub Actions Associate (Community)**
3. **GitHub Advanced Security Certification**

---

## 🎯 Final Project

**Build a Complete Production CI/CD Pipeline:**
- Multi-language monorepo
- Matrix testing across OS and versions
- Security scanning (SAST, dependency, container)
- Multi-environment deployments (dev, staging, prod)
- Approval gates for production
- Reusable workflows
- Custom actions
- Self-hosted runners
- Monitoring and notifications

---

**Congratulations on completing the GitHub Actions Learning Roadmap! 🎉**

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

