# GitHub Actions Hands-On Exercises

**35+ Practical Labs from Beginner to Production Expert**

---

## 🎯 How to Use This Guide

Each exercise includes:
- **Objective**: What you'll learn
- **Difficulty**: Beginner/Intermediate/Advanced
- **Duration**: Estimated completion time
- **Prerequisites**: What you need before starting
- **Step-by-Step Instructions**: Complete guide
- **Validation**: How to verify success
- **Bonus Challenges**: Additional tasks to extend learning

**Setup Requirements:**
- GitHub account
- Git installed locally
- Code editor (VS Code recommended)
- Basic command line knowledge

---

## 📚 Exercise Categories

1. **Getting Started** (Exercises 1-5) - Beginner
2. **CI/CD Basics** (Exercises 6-10) - Beginner
3. **Advanced Workflows** (Exercises 11-15) - Intermediate
4. **Multi-Language CI/CD** (Exercises 16-20) - Intermediate
5. **Docker & Containers** (Exercises 21-23) - Intermediate
6. **Cloud Deployments** (Exercises 24-27) - Advanced
7. **Security & Secrets** (Exercises 28-30) - Advanced
8. **Custom Actions** (Exercises 31-33) - Advanced
9. **Production Patterns** (Exercises 34-36) - Advanced

---

## 🌟 GETTING STARTED

### **Exercise 1: Your First Workflow**

**Objective:** Create and run a basic GitHub Actions workflow

**Difficulty:** 🟢 Beginner  
**Duration:** 15 minutes

**Steps:**

1. **Create a new repository on GitHub**
   - Go to GitHub and click "New Repository"
   - Name: `github-actions-lab`
   - Make it public
   - Initialize with README
   - Click "Create repository"

2. **Create workflow file**
   ```bash
   # Clone repository
   git clone https://github.com/YOUR_USERNAME/github-actions-lab.git
   cd github-actions-lab
   
   # Create workflows directory
   mkdir -p .github/workflows
   
   # Create workflow file
   cat > .github/workflows/hello-world.yml <<'EOF'
   name: Hello World
   
   on: [push]
   
   jobs:
     greet:
       runs-on: ubuntu-latest
       
       steps:
         - name: Print greeting
           run: echo "Hello, GitHub Actions!"
         
         - name: Show date
           run: date
         
         - name: List directory
           run: ls -la
   EOF
   ```

3. **Commit and push**
   ```bash
   git add .
   git commit -m "Add first workflow"
   git push origin main
   ```

4. **View workflow run**
   - Go to your repository on GitHub
   - Click "Actions" tab
   - You should see your workflow running
   - Click on the workflow run to see details

**Validation:**
- ✅ Workflow appears in Actions tab
- ✅ All steps show green checkmarks
- ✅ Logs show "Hello, GitHub Actions!"

**Bonus Challenges:**
- Add more steps that display environment variables
- Make the workflow run on pull requests too
- Add a step that fails and see what happens

---

### **Exercise 2: Multiple Triggers**

**Objective:** Configure workflow to run on different events

**Difficulty:** 🟢 Beginner  
**Duration:** 20 minutes

**Steps:**

1. **Create multi-trigger workflow**
   ```yaml
   name: Multi-Trigger Demo
   
   on:
     push:
       branches:
         - main
         - develop
     pull_request:
       branches:
         - main
     schedule:
       - cron: '0 0 * * 0'  # Every Sunday at midnight
     workflow_dispatch:
       inputs:
         message:
           description: 'Custom message'
           required: false
           default: 'Hello from manual trigger!'
   
   jobs:
     show-trigger:
       runs-on: ubuntu-latest
       steps:
         - name: Show event type
           run: |
             echo "Event: ${{ github.event_name }}"
             echo "Ref: ${{ github.ref }}"
             echo "Actor: ${{ github.actor }}"
         
         - name: Show manual input
           if: github.event_name == 'workflow_dispatch'
           run: echo "Message: ${{ inputs.message }}"
   ```

2. **Test different triggers**
   - Commit and push to main (push trigger)
   - Create a pull request (PR trigger)
   - Go to Actions tab → Select workflow → Click "Run workflow" (manual trigger)

**Validation:**
- ✅ Workflow runs on push
- ✅ Workflow runs on PR creation
- ✅ Can manually trigger with custom message
- ✅ Different event names shown in logs

**Bonus Challenges:**
- Add path filtering to only trigger on specific file changes
- Add a webhook trigger
- Create a workflow that runs on issue creation

---

### **Exercise 3: Working with Artifacts**

**Objective:** Upload and download artifacts between jobs

**Difficulty:** 🟢 Beginner  
**Duration:** 25 minutes

**Steps:**

1. **Create artifact workflow**
   ```yaml
   name: Artifact Demo
   
   on: [push]
   
   jobs:
     build:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         
         - name: Create build artifacts
           run: |
             mkdir -p build
             echo "Build timestamp: $(date)" > build/info.txt
             echo "Git SHA: ${{ github.sha }}" >> build/info.txt
             echo "const version = '${{ github.sha }}';" > build/app.js
         
         - name: Upload artifacts
           uses: actions/upload-artifact@v4
           with:
             name: build-output
             path: build/
             retention-days: 7
     
     test:
       needs: build
       runs-on: ubuntu-latest
       steps:
         - name: Download artifacts
           uses: actions/download-artifact@v4
           with:
             name: build-output
             path: ./downloaded
         
         - name: Verify artifacts
           run: |
             ls -la downloaded/
             cat downloaded/info.txt
             cat downloaded/app.js
   ```

**Validation:**
- ✅ Build job completes successfully
- ✅ Artifacts visible in workflow run page
- ✅ Test job downloads and displays artifact contents
- ✅ Can download artifacts manually from UI

**Bonus Challenges:**
- Upload multiple artifacts with different names
- Add artifact retention policy
- Create artifacts with compression

---

### **Exercise 4: Job Dependencies**

**Objective:** Create workflows with dependent jobs

**Difficulty:** 🟢 Beginner  
**Duration:** 20 minutes

**Steps:**

1. **Create dependent jobs workflow**
   ```yaml
   name: Job Dependencies
   
   on: [push]
   
   jobs:
     setup:
       runs-on: ubuntu-latest
       outputs:
         build-id: ${{ steps.set-id.outputs.build-id }}
         timestamp: ${{ steps.set-time.outputs.timestamp }}
       steps:
         - id: set-id
           run: echo "build-id=${{ github.run_number }}" >> $GITHUB_OUTPUT
         
         - id: set-time
           run: echo "timestamp=$(date +%s)" >> $GITHUB_OUTPUT
     
     build:
       needs: setup
       runs-on: ubuntu-latest
       steps:
         - name: Show build info
           run: |
             echo "Build ID: ${{ needs.setup.outputs.build-id }}"
             echo "Timestamp: ${{ needs.setup.outputs.timestamp }}"
         
         - name: Simulate build
           run: sleep 5 && echo "Build complete"
     
     test:
       needs: build
       runs-on: ubuntu-latest
       steps:
         - name: Run tests
           run: echo "Testing build ${{ needs.setup.outputs.build-id }}"
     
     deploy:
       needs: [build, test]
       runs-on: ubuntu-latest
       steps:
         - name: Deploy
           run: |
             echo "Deploying build ${{ needs.setup.outputs.build-id }}"
             echo "All tests passed!"
   ```

**Validation:**
- ✅ Jobs run in correct order: setup → build → test → deploy
- ✅ Build ID passed correctly between jobs
- ✅ Deploy only runs after both build and test complete

**Bonus Challenges:**
- Add conditional job execution
- Create parallel jobs
- Handle job failures with `continue-on-error`

---

### **Exercise 5: Matrix Builds**

**Objective:** Test across multiple operating systems and versions

**Difficulty:** 🟢 Beginner  
**Duration:** 30 minutes

**Steps:**

1. **Create simple Node.js project**
   ```bash
   npm init -y
   npm install --save-dev jest
   
   # Create test file
   mkdir src
   cat > src/sum.js <<'EOF'
   function sum(a, b) {
     return a + b;
   }
   module.exports = sum;
   EOF
   
   cat > src/sum.test.js <<'EOF'
   const sum = require('./sum');
   
   test('adds 1 + 2 to equal 3', () => {
     expect(sum(1, 2)).toBe(3);
   });
   EOF
   
   # Update package.json test script
   npm pkg set scripts.test="jest"
   ```

2. **Create matrix workflow**
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
       
       steps:
         - uses: actions/checkout@v4
         
         - uses: actions/setup-node@v4
           with:
             node-version: ${{ matrix.node-version }}
             cache: 'npm'
         
         - name: Install dependencies
           run: npm ci
         
         - name: Run tests
           run: npm test
         
         - name: Show matrix info
           run: |
             echo "OS: ${{ matrix.os }}"
             echo "Node: ${{ matrix.node-version }}"
             node --version
             npm --version
   ```

**Validation:**
- ✅ Workflow creates 8 jobs (3 OS × 3 Node versions - 1 exclusion + 1 inclusion)
- ✅ Tests run on all combinations
- ✅ Matrix visualization shows in Actions UI

**Bonus Challenges:**
- Add `fail-fast: false` to see all results
- Limit with `max-parallel: 3`
- Create dynamic matrix from file or API

---

## 🔧 CI/CD BASICS

### **Exercise 6: Node.js CI Pipeline**

**Objective:** Build complete CI pipeline for Node.js application

**Difficulty:** 🟢 Beginner  
**Duration:** 40 minutes

**Steps:**

1. **Create Express application**
   ```bash
   npm init -y
   npm install express
   npm install --save-dev jest supertest eslint
   
   # Create app
   cat > src/app.js <<'EOF'
   const express = require('express');
   const app = express();
   
   app.get('/', (req, res) => {
     res.json({ message: 'Hello World!' });
   });
   
   app.get('/health', (req, res) => {
     res.json({ status: 'healthy' });
   });
   
   module.exports = app;
   EOF
   
   # Create server
   cat > src/server.js <<'EOF'
   const app = require('./app');
   const PORT = process.env.PORT || 3000;
   
   app.listen(PORT, () => {
     console.log(`Server running on port ${PORT}`);
   });
   EOF
   
   # Create tests
   cat > src/app.test.js <<'EOF'
   const request = require('supertest');
   const app = require('./app');
   
   describe('API Tests', () => {
     test('GET / returns hello message', async () => {
       const response = await request(app).get('/');
       expect(response.statusCode).toBe(200);
       expect(response.body.message).toBe('Hello World!');
     });
     
     test('GET /health returns healthy status', async () => {
       const response = await request(app).get('/health');
       expect(response.statusCode).toBe(200);
       expect(response.body.status).toBe('healthy');
     });
   });
   EOF
   
   # Setup scripts
   npm pkg set scripts.test="jest"
   npm pkg set scripts.start="node src/server.js"
   npm pkg set scripts.lint="eslint src/"
   
   # Create .eslintrc.json
   cat > .eslintrc.json <<'EOF'
   {
     "env": {
       "node": true,
       "es2021": true,
       "jest": true
     },
     "extends": "eslint:recommended"
   }
   EOF
   ```

2. **Create CI workflow**
   ```yaml
   name: Node.js CI
   
   on:
     push:
       branches: [ main ]
     pull_request:
       branches: [ main ]
   
   jobs:
     lint:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         
         - uses: actions/setup-node@v4
           with:
             node-version: '20'
             cache: 'npm'
         
         - run: npm ci
         
         - name: Run ESLint
           run: npm run lint
     
     test:
       runs-on: ubuntu-latest
       strategy:
         matrix:
           node-version: [18, 20]
       
       steps:
         - uses: actions/checkout@v4
         
         - uses: actions/setup-node@v4
           with:
             node-version: ${{ matrix.node-version }}
             cache: 'npm'
         
         - run: npm ci
         
         - name: Run tests
           run: npm test
         
         - name: Upload coverage
           if: matrix.node-version == '20'
           uses: actions/upload-artifact@v4
           with:
             name: coverage
             path: coverage/
     
     build:
       needs: [lint, test]
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         
         - uses: actions/setup-node@v4
           with:
             node-version: '20'
             cache: 'npm'
         
         - run: npm ci
         
         - name: Create build info
           run: |
             mkdir -p dist
             echo "Build: ${{ github.run_number }}" > dist/build-info.txt
             echo "SHA: ${{ github.sha }}" >> dist/build-info.txt
             cp -r src dist/
         
         - uses: actions/upload-artifact@v4
           with:
             name: dist
             path: dist/
   ```

**Validation:**
- ✅ Lint job catches style issues
- ✅ Tests pass on Node 18 and 20
- ✅ Build creates dist artifact
- ✅ Jobs run in parallel then sequentially

**Bonus Challenges:**
- Add code coverage reporting
- Integrate with SonarCloud
- Add dependency security scanning
- Create Docker image in build step

---

### **Exercise 7: Python CI Pipeline**

**Objective:** Build CI pipeline for Python application with testing and linting

**Difficulty:** 🟢 Beginner  
**Duration:** 35 minutes

**Steps:**

1. **Create Python project**
   ```bash
   # Create project structure
   mkdir -p src tests
   
   # Create main application
   cat > src/calculator.py <<'EOF'
   """Simple calculator module."""
   
   def add(a: float, b: float) -> float:
       """Add two numbers."""
       return a + b
   
   def subtract(a: float, b: float) -> float:
       """Subtract b from a."""
       return a - b
   
   def multiply(a: float, b: float) -> float:
       """Multiply two numbers."""
       return a * b
   
   def divide(a: float, b: float) -> float:
       """Divide a by b."""
       if b == 0:
           raise ValueError("Cannot divide by zero")
       return a / b
   EOF
   
   # Create tests
   cat > tests/test_calculator.py <<'EOF'
   """Tests for calculator module."""
   import pytest
   from src.calculator import add, subtract, multiply, divide
   
   def test_add():
       assert add(2, 3) == 5
       assert add(-1, 1) == 0
   
   def test_subtract():
       assert subtract(5, 3) == 2
       assert subtract(0, 1) == -1
   
   def test_multiply():
       assert multiply(2, 3) == 6
       assert multiply(-2, 3) == -6
   
   def test_divide():
       assert divide(6, 2) == 3
       assert divide(5, 2) == 2.5
   
   def test_divide_by_zero():
       with pytest.raises(ValueError):
           divide(5, 0)
   EOF
   
   # Create requirements
   cat > requirements.txt <<'EOF'
   # Production dependencies
   EOF
   
   cat > requirements-dev.txt <<'EOF'
   pytest>=7.4.0
   pytest-cov>=4.1.0
   flake8>=6.0.0
   black>=23.0.0
   mypy>=1.4.0
   EOF
   ```

2. **Create Python CI workflow**
   ```yaml
   name: Python CI
   
   on: [push, pull_request]
   
   jobs:
     lint:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         
         - uses: actions/setup-python@v5
           with:
             python-version: '3.12'
             cache: 'pip'
         
         - name: Install dependencies
           run: |
             python -m pip install --upgrade pip
             pip install -r requirements-dev.txt
         
         - name: Run flake8
           run: flake8 src/ tests/
         
         - name: Run black check
           run: black --check src/ tests/
         
         - name: Run mypy
           run: mypy src/
     
     test:
       runs-on: ${{ matrix.os }}
       strategy:
         matrix:
           os: [ubuntu-latest, windows-latest, macos-latest]
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
         
         - name: Run tests with coverage
           run: pytest --cov=src --cov-report=xml --cov-report=html
         
         - name: Upload coverage
           if: matrix.os == 'ubuntu-latest' && matrix.python-version == '3.12'
           uses: actions/upload-artifact@v4
           with:
             name: coverage-report
             path: htmlcov/
   ```

**Validation:**
- ✅ Linting passes (flake8, black, mypy)
- ✅ Tests pass on all OS and Python versions
- ✅ Coverage report generated

**Bonus Challenges:**
- Add safety check for dependencies
- Integrate with CodeCov
- Add bandit for security scanning
- Build and publish package to PyPI (test)

---

### **Exercise 8: Docker Build and Push**

**Objective:** Build Docker images and push to GitHub Container Registry

**Difficulty:** 🟡 Intermediate  
**Duration:** 45 minutes

**Steps:**

1. **Create Dockerfile**
   ```dockerfile
   FROM node:20-alpine
   
   WORKDIR /app
   
   COPY package*.json ./
   RUN npm ci --only=production
   
   COPY src ./src
   
   EXPOSE 3000
   
   USER node
   
   CMD ["node", "src/server.js"]
   ```

2. **Create Docker workflow**
   ```yaml
   name: Docker Build and Push
   
   on:
     push:
       branches: [ main ]
       tags:
         - 'v*'
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
           if: github.event_name != 'pull_request'
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
               type=raw,value=latest,enable={{is_default_branch}}
         
         - name: Build and push
           uses: docker/build-push-action@v5
           with:
             context: .
             push: ${{ github.event_name != 'pull_request' }}
             tags: ${{ steps.meta.outputs.tags }}
             labels: ${{ steps.meta.outputs.labels }}
             cache-from: type=gha
             cache-to: type=gha,mode=max
             platforms: linux/amd64,linux/arm64
         
         - name: Scan image with Trivy
           uses: aquasecurity/trivy-action@master
           with:
             image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
             format: 'sarif'
             output: 'trivy-results.sarif'
         
         - name: Upload Trivy results
           uses: github/codeql-action/upload-sarif@v3
           with:
             sarif_file: 'trivy-results.sarif'
   ```

**Validation:**
- ✅ Image builds successfully
- ✅ Image pushed to ghcr.io
- ✅ Multiple tags created
- ✅ Security scan runs
- ✅ Multi-platform build works

**Bonus Challenges:**
- Add Docker Compose for local testing
- Implement image signing
- Add SBOM generation
- Create vulnerability report

---

## 🚀 ADVANCED WORKFLOWS

### **Exercise 9: Reusable Workflows**

**Objective:** Create and use reusable workflows

**Difficulty:** 🟡 Intermediate  
**Duration:** 40 minutes

**Steps:**

1. **Create reusable build workflow**
   ```yaml
   # .github/workflows/reusable-build.yml
   name: Reusable Build
   
   on:
     workflow_call:
       inputs:
         node-version:
           required: false
           type: string
           default: '20'
         environment:
           required: true
           type: string
         skip-tests:
           required: false
           type: boolean
           default: false
       secrets:
         deploy-token:
           required: true
       outputs:
         build-version:
           description: "Version of the build"
           value: ${{ jobs.build.outputs.version }}
         artifact-name:
           description: "Name of the build artifact"
           value: ${{ jobs.build.outputs.artifact }}
   
   jobs:
     build:
       runs-on: ubuntu-latest
       outputs:
         version: ${{ steps.version.outputs.version }}
         artifact: build-${{ github.sha }}
       
       steps:
         - uses: actions/checkout@v4
         
         - uses: actions/setup-node@v4
           with:
             node-version: ${{ inputs.node-version }}
             cache: 'npm'
         
         - run: npm ci
         
         - name: Run tests
           if: ${{ !inputs.skip-tests }}
           run: npm test
         
         - id: version
           run: echo "version=$(node -p "require('./package.json').version")" >> $GITHUB_OUTPUT
         
         - name: Build
           run: npm run build
           env:
             NODE_ENV: ${{ inputs.environment }}
         
         - uses: actions/upload-artifact@v4
           with:
             name: build-${{ github.sha }}
             path: dist/
   ```

2. **Create caller workflow**
   ```yaml
   # .github/workflows/ci-cd.yml
   name: CI/CD Pipeline
   
   on:
     push:
       branches: [ main, develop ]
     pull_request:
       branches: [ main ]
   
   jobs:
     build-dev:
       if: github.ref == 'refs/heads/develop'
       uses: ./.github/workflows/reusable-build.yml
       with:
         node-version: '20'
         environment: 'development'
         skip-tests: false
       secrets:
         deploy-token: ${{ secrets.DEV_DEPLOY_TOKEN }}
     
     build-prod:
       if: github.ref == 'refs/heads/main'
       uses: ./.github/workflows/reusable-build.yml
       with:
         node-version: '20'
         environment: 'production'
         skip-tests: false
       secrets:
         deploy-token: ${{ secrets.PROD_DEPLOY_TOKEN }}
     
     deploy:
       needs: [build-prod]
       if: github.ref == 'refs/heads/main'
       runs-on: ubuntu-latest
       steps:
         - run: |
             echo "Deploying version: ${{ needs.build-prod.outputs.build-version }}"
             echo "Artifact: ${{ needs.build-prod.outputs.artifact-name }}"
   ```

**Validation:**
- ✅ Reusable workflow called successfully
- ✅ Inputs passed correctly
- ✅ Secrets work in reusable workflow
- ✅ Outputs returned to caller

**Bonus Challenges:**
- Create reusable deployment workflow
- Add reusable security scanning workflow
- Create workflow library in separate repository

---

### **Exercise 10: Composite Actions**

**Objective:** Create custom composite action

**Difficulty:** 🟡 Intermediate  
**Duration:** 35 minutes

**Steps:**

1. **Create composite action**
   ```yaml
   # .github/actions/setup-and-build/action.yml
   name: 'Setup and Build Node.js Project'
   description: 'Setup Node.js with caching and build the project'
   
   inputs:
     node-version:
       description: 'Node.js version to use'
       required: false
       default: '20'
     working-directory:
       description: 'Working directory'
       required: false
       default: '.'
     cache-key-prefix:
       description: 'Prefix for cache key'
       required: false
       default: 'node-deps'
   
   outputs:
     build-time:
       description: 'Build completion time'
       value: ${{ steps.build-info.outputs.time }}
     cache-hit:
       description: 'Whether cache was hit'
       value: ${{ steps.cache.outputs.cache-hit }}
   
   runs:
     using: 'composite'
     steps:
       - name: Setup Node.js
         uses: actions/setup-node@v4
         with:
           node-version: ${{ inputs.node-version }}
         shell: bash
       
       - name: Cache dependencies
         id: cache
         uses: actions/cache@v4
         with:
           path: ${{ inputs.working-directory }}/node_modules
           key: ${{ inputs.cache-key-prefix }}-${{ runner.os }}-${{ hashFiles(format('{0}/package-lock.json', inputs.working-directory)) }}
           restore-keys: |
             ${{ inputs.cache-key-prefix }}-${{ runner.os }}-
         shell: bash
       
       - name: Install dependencies
         if: steps.cache.outputs.cache-hit != 'true'
         working-directory: ${{ inputs.working-directory }}
         run: npm ci
         shell: bash
       
       - name: Build project
         working-directory: ${{ inputs.working-directory }}
         run: npm run build
         shell: bash
       
       - name: Set build info
         id: build-info
         run: echo "time=$(date -u +%Y-%m-%dT%H:%M:%SZ)" >> $GITHUB_OUTPUT
         shell: bash
   ```

2. **Use composite action**
   ```yaml
   # .github/workflows/use-composite.yml
   name: Use Composite Action
   
   on: [push]
   
   jobs:
     build:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         
         - name: Setup and build
           id: build
           uses: ./.github/actions/setup-and-build
           with:
             node-version: '20'
             working-directory: '.'
             cache-key-prefix: 'my-app'
         
         - name: Show build info
           run: |
             echo "Build time: ${{ steps.build.outputs.build-time }}"
             echo "Cache hit: ${{ steps.build.outputs.cache-hit }}"
   ```

**Validation:**
- ✅ Composite action executes all steps
- ✅ Inputs work correctly
- ✅ Outputs returned properly
- ✅ Cache functioning

**Bonus Challenges:**
- Add error handling
- Create action for deployment
- Publish action to marketplace
- Add comprehensive documentation

---

**Continue with remaining 26 exercises covering:**
- Multi-environment deployments
- Cloud platform integrations (AWS, Azure, GCP)
- Security scanning and secrets management
- Self-hosted runners
- Monorepo patterns
- Release automation
- Performance optimization

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

