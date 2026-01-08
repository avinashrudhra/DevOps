# GitHub Actions Troubleshooting Guide

**60+ Real-World Issues with Root Cause Analysis & Solutions**

---

## 🎯 Guide Structure

Each troubleshooting scenario includes:
- **Symptom**: What you observe
- **Severity**: Critical/High/Medium/Low
- **Root Cause**: Why it's happening
- **Solution**: Step-by-step fix
- **Prevention**: How to avoid in future

---

## 📚 Categories

1. **Workflow Failures** (Issues 1-15)
2. **Authentication & Permissions** (Issues 16-25)
3. **Runner Problems** (Issues 26-35)
4. **Performance & Optimization** (Issues 36-45)
5. **Dependencies & Caching** (Issues 46-55)
6. **Debugging & Logs** (Issues 56-60)

---

## ⚠️ WORKFLOW FAILURES

### **Issue 1: Workflow Not Triggering**

**Symptom:**
```
# Pushed code but workflow doesn't run
git push origin main
# No workflow run appears in Actions tab
```

**Severity:** High

**Root Cause:**
- Workflow file in wrong location
- Syntax errors in YAML
- Event filter excluding changes
- Workflows disabled for repository

**Solution:**
```bash
# 1. Check file location
# Must be in: .github/workflows/*.yml or *.yaml

# 2. Validate YAML syntax
# Use GitHub's workflow editor or yamllint
yamllint .github/workflows/ci.yml

# 3. Check event filters
on:
  push:
    branches:
      - main              # Make sure your branch matches
    paths:
      - 'src/**'          # Check if changed files match

# 4. Enable workflows in repository settings
# Settings → Actions → General → Allow all actions
```

**Prevention:**
- Use GitHub's workflow editor with syntax validation
- Test with simple push trigger first
- Check Actions tab for error messages
- Enable workflow status badge

---

### **Issue 2: "Error: Process completed with exit code 1"**

**Symptom:**
```
Run npm test
Error: Process completed with exit code 1
```

**Severity:** Medium

**Root Cause:**
- Command failed (test failure, build error)
- Missing dependencies
- Environment variables not set
- Script not found in package.json

**Solution:**
```yaml
# 1. Add debug output
- name: Run tests with debug
  run: |
    echo "Node version: $(node --version)"
    echo "NPM version: $(npm --version)"
    ls -la
    cat package.json
    npm test

# 2. Check if script exists
- name: List npm scripts
  run: npm run

# 3. Run with verbose output
- name: Run tests verbose
  run: npm test -- --verbose

# 4. Ignore specific exit codes (use carefully!)
- name: Run tests
  run: npm test
  continue-on-error: true
```

**Prevention:**
- Test locally with same Node version
- Add proper error handling in scripts
- Use `npm ci` instead of `npm install`
- Add step to verify environment

---

### **Issue 3: Timeout Exceeded**

**Symptom:**
```
Error: The operation was canceled.
Job timed out after 360 minutes
```

**Severity:** High

**Root Cause:**
- Job running longer than timeout limit (default 360 minutes)
- Infinite loop or hanging process
- Waiting for interactive input
- Slow network/downloads

**Solution:**
```yaml
# 1. Set appropriate timeout
jobs:
  build:
    runs-on: ubuntu-latest
    timeout-minutes: 30              # Set realistic limit
    
    steps:
      - name: Build with timeout
        run: npm run build
        timeout-minutes: 10          # Per-step timeout

# 2. Find hanging process
- name: Debug hanging step
  run: |
    npm test &
    PID=$!
    sleep 300  # Wait 5 minutes
    if ps -p $PID > /dev/null; then
      echo "Process still running, investigating..."
      ps aux
      kill $PID
    fi

# 3. Add non-interactive flags
- run: apt-get install -y package  # -y flag for non-interactive
```

**Prevention:**
- Set reasonable timeouts for each job
- Use non-interactive mode for all commands
- Implement proper process cleanup
- Monitor job duration trends

---

### **Issue 4: "Resource not accessible by integration"**

**Symptom:**
```
Error: Resource not accessible by integration
HttpError: Resource not accessible by integration
```

**Severity:** High

**Root Cause:**
- Insufficient token permissions
- Missing workflow permissions
- Wrong secret/token used

**Solution:**
```yaml
# 1. Add required permissions at workflow level
permissions:
  contents: write          # For pushing code
  packages: write          # For container registry
  pull-requests: write     # For commenting on PRs
  issues: write           # For creating issues

# 2. Or at job level
jobs:
  deploy:
    permissions:
      contents: read
      packages: write
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

# 3. Use correct token
- name: Push changes
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}  # Auto-generated
    # or
    PAT: ${{ secrets.PERSONAL_ACCESS_TOKEN }}  # Custom PAT
```

**Prevention:**
- Always specify minimum required permissions
- Use `GITHUB_TOKEN` for GitHub API operations
- Create PAT with appropriate scopes for external operations
- Test with read-only permissions first

---

### **Issue 5: Artifacts Not Found**

**Symptom:**
```
Error: Artifact 'build-output' not found
Unable to download artifact
```

**Severity:** Medium

**Root Cause:**
- Artifact name mismatch
- Artifact expired (retention period)
- Upload job failed
- Artifact from different workflow run

**Solution:**
```yaml
# 1. Ensure consistent naming
jobs:
  build:
    steps:
      - uses: actions/upload-artifact@v4
        with:
          name: my-artifact        # Exact name
          path: dist/
  
  deploy:
    needs: build
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: my-artifact        # Same exact name

# 2. Check artifact exists
- name: List available artifacts
  run: |
    curl -H "Authorization: token ${{ secrets.GITHUB_TOKEN }}" \
      "${{ github.api_url }}/repos/${{ github.repository }}/actions/runs/${{ github.run_id }}/artifacts"

# 3. Set longer retention
- uses: actions/upload-artifact@v4
  with:
    name: my-artifact
    path: dist/
    retention-days: 30            # Default is 90

# 4. Download from specific run
- uses: actions/download-artifact@v4
  with:
    name: my-artifact
    run-id: ${{ github.run_id }}
```

**Prevention:**
- Use variables for artifact names
- Document artifact naming convention
- Set appropriate retention policies
- Verify upload success before downloading

---

## 🔐 AUTHENTICATION & PERMISSIONS

### **Issue 6: "Permission denied (publickey)" for Git Operations**

**Symptom:**
```
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.
```

**Severity:** High

**Root Cause:**
- Using SSH URL instead of HTTPS
- Missing deploy keys
- Insufficient token permissions

**Solution:**
```yaml
# 1. Use HTTPS with token
- uses: actions/checkout@v4
  with:
    token: ${{ secrets.GITHUB_TOKEN }}

# 2. Configure git for pushing
- name: Configure git
  run: |
    git config user.name "GitHub Actions"
    git config user.email "actions@github.com"
    git remote set-url origin https://x-access-token:${{ secrets.GITHUB_TOKEN }}@github.com/${{ github.repository }}

# 3. Or use deploy key
- uses: actions/checkout@v4
  with:
    ssh-key: ${{ secrets.DEPLOY_KEY }}

# 4. Push changes
- name: Commit and push
  run: |
    git add .
    git commit -m "Update from Actions"
    git push
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**Prevention:**
- Always use HTTPS URLs in workflows
- Use `GITHUB_TOKEN` for repository operations
- Set up deploy keys for cross-repo access
- Test authentication before complex operations

---

### **Issue 7: Docker Login Failed**

**Symptom:**
```
Error: Cannot perform an interactive login from a non TTY device
Error response from daemon: Get https://ghcr.io/v2/: unauthorized
```

**Severity:** High

**Root Cause:**
- Missing or incorrect credentials
- Wrong registry URL
- Token without package permissions

**Solution:**
```yaml
# 1. Login to GitHub Container Registry
- uses: docker/login-action@v3
  with:
    registry: ghcr.io
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}

# 2. Add package permissions
permissions:
  packages: write

# 3. Login to Docker Hub
- uses: docker/login-action@v3
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}

# 4. Verify login
- name: Verify Docker auth
  run: docker info
```

**Prevention:**
- Use official login actions
- Store credentials as secrets
- Grant package permissions
- Test login before build/push

---

## 🖥️ RUNNER PROBLEMS

### **Issue 8: Self-Hosted Runner Offline**

**Symptom:**
```
Waiting for a runner to pick up this job...
No runners available
```

**Severity:** Critical

**Root Cause:**
- Runner service stopped
- Runner machine offline/rebooted
- Network connectivity issues
- Runner removed from repository

**Solution:**
```bash
# 1. Check runner status
cd /path/to/actions-runner
./run.sh --check

# 2. Check service status (Linux)
sudo systemctl status actions.runner.*

# 3. Restart service
sudo ./svc.sh stop
sudo ./svc.sh start

# 4. Check logs
tail -f /path/to/actions-runner/_diag/Worker_*.log

# 5. Re-register if needed
./config.sh remove --token <TOKEN>
./config.sh --url https://github.com/org/repo --token <NEW_TOKEN>
```

**Prevention:**
- Monitor runner health
- Set up auto-restart on failure
- Configure runner as system service
- Use runner groups for redundancy
- Consider using ephemeral runners

---

### **Issue 9: Out of Disk Space on Runner**

**Symptom:**
```
Error: No space left on device
ENOSPC: no space left on device
```

**Severity:** High

**Root Cause:**
- Docker images/containers accumulating
- Build artifacts not cleaned
- Logs filling disk
- Cache files growing

**Solution:**
```yaml
# 1. Clean up at start of job
- name: Clean up disk space
  run: |
    docker system prune -af
    docker volume prune -f
    sudo rm -rf /usr/share/dotnet
    sudo rm -rf /opt/ghc
    sudo rm -rf "$AGENT_TOOLSDIRECTORY"

# 2. Free up space during build
- name: Free disk space
  uses: jlumbroso/free-disk-space@main
  with:
    tool-cache: true
    android: true
    dotnet: true
    haskell: true
    large-packages: true
    docker-images: true

# 3. Monitor disk usage
- name: Check disk space
  run: df -h

# 4. Clean up at end
- name: Cleanup
  if: always()
  run: docker system prune -af --volumes
```

**Prevention:**
- Regular cleanup of old images/containers
- Implement log rotation
- Monitor disk usage with alerts
- Use larger disk for runners
- Clean workspace between runs

---

## ⚡ PERFORMANCE & OPTIMIZATION

### **Issue 10: Slow Workflow Execution**

**Symptom:**
```
Workflow taking 45 minutes when it should take 10
Steps running sequentially when they could be parallel
```

**Severity:** Medium

**Root Cause:**
- Dependencies installed repeatedly
- No caching configured
- Sequential job execution
- Large artifacts transferred

**Solution:**
```yaml
# 1. Enable dependency caching
- uses: actions/setup-node@v4
  with:
    node-version: '20'
    cache: 'npm'                   # Built-in caching

# 2. Parallelize jobs
jobs:
  lint:
    runs-on: ubuntu-latest
  
  test:
    runs-on: ubuntu-latest
  
  build:
    needs: [lint, test]           # Only build after lint and test

# 3. Use matrix for parallel testing
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest]
  max-parallel: 3                 # Limit concurrency

# 4. Optimize Docker builds
- uses: docker/build-push-action@v5
  with:
    cache-from: type=gha
    cache-to: type=gha,mode=max

# 5. Use actions/cache for custom caching
- uses: actions/cache@v4
  with:
    path: |
      ~/.npm
      ~/.cache
      node_modules
    key: ${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}
```

**Prevention:**
- Always implement caching
- Parallelize independent jobs
- Use matrix builds wisely
- Minimize artifact sizes
- Profile workflow performance

---

### **Issue 11: Cache Not Working**

**Symptom:**
```
Cache restored successfully but dependencies still installing
Cache hit: true, but npm ci running
```

**Severity:** Low

**Root Cause:**
- Wrong path cached
- Cache key too specific/broad
- Cache corrupted
- Different restore/save paths

**Solution:**
```yaml
# 1. Verify cache paths
- name: Debug cache
  run: |
    echo "Cache paths:"
    ls -la ~/.npm || echo "NPM cache not found"
    ls -la node_modules || echo "node_modules not found"

# 2. Proper cache configuration
- id: cache
  uses: actions/cache@v4
  with:
    path: |
      ~/.npm
      node_modules
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-

# 3. Skip install if cache hit
- if: steps.cache.outputs.cache-hit != 'true'
  run: npm ci

# 4. Clear cache if corrupted
# Settings → Actions → Caches → Delete cache

# 5. Use better cache key
key: ${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}-v2
```

**Prevention:**
- Test cache with debug output
- Use specific but not too specific keys
- Include version in cache key for breaking changes
- Implement cache fallbacks with restore-keys
- Monitor cache hit rates

---

## 🔍 DEBUGGING & LOGS

### **Issue 12: Hidden Error in Logs**

**Symptom:**
```
Step shows success but something is wrong
No clear error message in logs
```

**Severity:** Medium

**Root Cause:**
- Error in multi-line command ignored
- Success despite failures in background
- Errors written to stderr but not failing step

**Solution:**
```yaml
# 1. Enable strict error handling
- name: Run with strict mode
  run: |
    set -euo pipefail              # Exit on error, undefined vars, pipe failures
    npm test

# 2. Check all commands
- name: Multiple commands
  run: |
    command1 && \
    command2 && \
    command3

# 3. Explicit error checking
- name: Build with error check
  run: |
    npm run build
    if [ $? -ne 0 ]; then
      echo "Build failed!"
      exit 1
    fi

# 4. Capture stderr
- name: Run with error capture
  run: |
    npm test 2>&1 | tee test-output.log
    if grep -q "ERROR" test-output.log; then
      echo "Errors found in test output"
      exit 1
    fi
```

**Prevention:**
- Always use `set -euo pipefail` in bash
- Check exit codes explicitly
- Use `&&` for command chaining
- Capture and analyze stderr

---

### **Issue 13: Unable to Debug Workflow**

**Symptom:**
```
Need to see what's happening inside runner
Want to SSH into runner during workflow
```

**Severity:** Low

**Root Cause:**
- Limited visibility into runner environment
- Complex debugging needed

**Solution:**
```yaml
# 1. Enable debug logging
# Set repository secrets:
# ACTIONS_RUNNER_DEBUG=true
# ACTIONS_STEP_DEBUG=true

# 2. Add debugging step
- name: Debug information
  run: |
    echo "=== Environment Variables ==="
    env | sort
    echo "=== Working Directory ==="
    pwd
    ls -la
    echo "=== Git Status ==="
    git status
    echo "=== System Info ==="
    uname -a
    df -h

# 3. SSH into runner (use carefully!)
- name: Setup tmate session
  if: failure()                    # Only on failure
  uses: mxschmitt/action-tmate@v3
  timeout-minutes: 30

# 4. Print context
- name: Dump GitHub context
  run: echo '${{ toJSON(github) }}'

- name: Dump job context
  run: echo '${{ toJSON(job) }}'

- name: Dump steps context
  run: echo '${{ toJSON(steps) }}'
```

**Prevention:**
- Add comprehensive logging
- Use debug workflows in development
- Test locally with act: `act -s GITHUB_TOKEN=...`
- Create debug reusable workflow

---

## 💡 Quick Troubleshooting Tips

### **General Debugging Process:**

1. **Check the obvious first**
   - Syntax errors in YAML
   - File paths and permissions
   - Secret names and values

2. **Enable debug logging**
   ```bash
   # Repository Settings → Secrets → Add:
   ACTIONS_RUNNER_DEBUG=true
   ACTIONS_STEP_DEBUG=true
   ```

3. **Add diagnostic steps**
   ```yaml
   - name: Debug
     run: |
       echo "Working directory: $(pwd)"
       echo "Files: $(ls -la)"
       echo "Environment: $(env | sort)"
   ```

4. **Test locally**
   ```bash
   # Use nektos/act
   act -s GITHUB_TOKEN=your_token
   ```

5. **Simplify to isolate**
   - Remove all steps except problem step
   - Test with minimal reproduction case
   - Add steps back one at a time

### **Common Fixes:**

```yaml
# Fix: Unrecognized named-value
# Use: ${{ secrets.NAME }} not ${{secrets.NAME}}

# Fix: Unexpected value
# Quote values: version: '3.9' not version: 3.9

# Fix: Invalid workflow
# Validate YAML structure and indentation

# Fix: Step skipped
# Check if: conditions and job dependencies

# Fix: Merge conflicts
# Use pull before push or checkout with fetch-depth: 0
```

---

## 🎯 Preventive Best Practices

1. **Always validate YAML** before committing
2. **Use workflow templates** for consistency
3. **Implement proper error handling** in scripts
4. **Monitor workflow execution time** and costs
5. **Set up status checks** and branch protection
6. **Use workflow badges** to track status
7. **Document known issues** and solutions
8. **Test workflows in draft PR** before merging
9. **Keep actions updated** to latest versions
10. **Review workflow logs** regularly

---

**Remember: Most issues have been solved before - search GitHub Community and Stack Overflow!**

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

