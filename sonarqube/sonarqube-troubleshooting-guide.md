# SonarQube Troubleshooting Guide

Common SonarQube problems and their solutions.

---

## Table of Contents
1. [Installation Issues](#installation-issues)
2. [Scanner Problems](#scanner-problems)
3. [Server Issues](#server-issues)
4. [Database Problems](#database-problems)
5. [Analysis Failures](#analysis-failures)
6. [Quality Gate Issues](#quality-gate-issues)
7. [Integration Problems](#integration-problems)
8. [Performance Issues](#performance-issues)

---

## Installation Issues

### Issue 1: Server Won't Start
**Problem**: SonarQube server fails to start

**Symptoms:**
```
Wrapper  | --> Wrapper Started as Console
Wrapper  | Launching a JVM...
JVM exited while loading the application.
```

**Solutions:**

**Solution 1: Check Java Version**
```bash
# Check Java version
java -version

# SonarQube 9.9+ requires Java 11 or 17
# Install correct version
sudo apt install openjdk-11-jdk  # Ubuntu/Debian
brew install openjdk@11  # Mac
```

**Solution 2: Check Port Availability**
```bash
# Check if port 9000 is in use
netstat -tuln | grep 9000
# Or
lsof -i :9000

# Kill process using port
kill -9 <PID>

# Or change port in sonar.properties
sonar.web.port=9001
```

**Solution 3: Check Logs**
```bash
# View logs
tail -f logs/sonar.log
tail -f logs/web.log
tail -f logs/ce.log

# Common errors:
# - Java heap space: Increase memory
# - Port already in use: Change port
# - Database connection: Check DB credentials
```

**Solution 4: Increase Memory**
```properties
# conf/sonar.properties
sonar.web.javaOpts=-Xmx2048m -Xms1024m
sonar.ce.javaOpts=-Xmx2048m -Xms1024m
sonar.search.javaOpts=-Xmx1024m -Xms1024m
```

---

### Issue 2: "Elasticsearch failed to start"
**Problem**: Embedded Elasticsearch won't start

**Symptoms:**
```
[ERROR] Elasticsearch failed to start
Max virtual memory areas vm.max_map_count [65530] is too low
```

**Solutions:**

**Linux:**
```bash
# Temporary fix
sudo sysctl -w vm.max_map_count=262144

# Permanent fix
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

**Docker:**
```bash
# Add to docker-compose.yml
services:
  sonarqube:
    environment:
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    sysctls:
      - vm.max_map_count=262144
```

**Mac:**
```bash
# In Docker Desktop settings
# Increase memory to at least 4GB
```

---

### Issue 3: "Web server is down"
**Problem**: Server starts but web interface unreachable

**Symptoms:**
```
SonarQube is operational but web interface shows:
"Oops! We couldn't find the page you're looking for."
```

**Solutions:**

**Check Server Status:**
```bash
# Check if server is running
./bin/linux-x86-64/sonar.sh status

# Check if listening on port
curl http://localhost:9000/api/system/status
```

**Check Firewall:**
```bash
# Linux
sudo ufw allow 9000

# Check iptables
sudo iptables -L

# Windows
# Control Panel → Firewall → Allow app through firewall
```

**Check Context Path:**
```properties
# If using context path
sonar.web.context=/sonarqube

# Access at:
# http://localhost:9000/sonarqube
```

---

## Scanner Problems

### Issue 4: "Not authorized" During Analysis
**Problem**: Scanner fails with authorization error

**Symptoms:**
```
[ERROR] Not authorized. Please check the properties 
sonar.login and sonar.password
```

**Solutions:**

**Solution 1: Use Token Instead of Password**
```bash
# Generate token in SonarQube
# My Account → Security → Generate Token

# Use token (not password)
mvn sonar:sonar \
  -Dsonar.login=YOUR_TOKEN
# Don't use sonar.password with token!
```

**Solution 2: Check Token Permissions**
```
# In SonarQube UI:
# Administration → Security → Users
# Ensure user has "Execute Analysis" permission
```

**Solution 3: Project Permissions**
```
# Project Settings → Permissions
# Ensure user/group has "Execute Analysis" permission on project
```

---

### Issue 5: "Project Not Found" Error
**Problem**: Scanner creates multiple projects with different keys

**Symptoms:**
```
[ERROR] Project not found: my-project
Multiple projects created with similar names
```

**Solutions:**

**Consistent Project Key:**
```bash
# Always use the same project key
mvn sonar:sonar \
  -Dsonar.projectKey=com.example:my-project

# Or in pom.xml
<properties>
    <sonar.projectKey>com.example:my-project</sonar.projectKey>
</properties>

# Or in sonar-project.properties
sonar.projectKey=com.example:my-project
```

**Delete Duplicate Projects:**
```
# In UI:
# Administration → Projects → Management
# Delete duplicate projects
```

---

### Issue 6: Coverage Not Uploaded
**Problem**: Code coverage shows 0% despite tests running

**Symptoms:**
```
Tests run successfully
Coverage report generated locally
SonarQube shows 0.0% coverage
```

**Solutions:**

**Solution 1: Verify Coverage Report Path**
```xml
<!-- pom.xml -->
<properties>
    <sonar.coverage.jacoco.xmlReportPaths>
        ${project.build.directory}/site/jacoco/jacoco.xml
    </sonar.coverage.jacoco.xmlReportPaths>
</properties>
```

```bash
# Verify file exists
ls -la target/site/jacoco/jacoco.xml

# Check file contents
head -20 target/site/jacoco/jacoco.xml
```

**Solution 2: Run Tests Before Analysis**
```bash
# Correct order
mvn clean test  # Generate coverage
mvn sonar:sonar  # Upload coverage

# Or combined
mvn clean verify sonar:sonar
```

**Solution 3: Check JaCoCo Configuration**
```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.10</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

**Solution 4: Check Scanner Logs**
```bash
# Debug mode
mvn sonar:sonar -X | grep -i coverage

# Look for:
# - Coverage report parsing
# - File path resolution
# - Any coverage-related warnings
```

---

## Server Issues

### Issue 7: "Out of Memory" Errors
**Problem**: Server crashes with OutOfMemoryError

**Symptoms:**
```
java.lang.OutOfMemoryError: Java heap space
java.lang.OutOfMemoryError: GC overhead limit exceeded
```

**Solutions:**

**Increase Heap Size:**
```properties
# conf/sonar.properties

# Web Server
sonar.web.javaOpts=-Xmx4096m -Xms2048m

# Compute Engine
sonar.ce.javaOpts=-Xmx4096m -Xms2048m

# Elasticsearch
sonar.search.javaOpts=-Xmx2048m -Xms1024m
```

**Docker:**
```yaml
services:
  sonarqube:
    environment:
      - SONAR_WEB_JAVAOPTS=-Xmx4g
      - SONAR_CE_JAVAOPTS=-Xmx4g
      - SONAR_SEARCH_JAVAOPTS=-Xmx2g
    deploy:
      resources:
        limits:
          memory: 8G
```

**Monitor Memory Usage:**
```bash
# Check Java processes
ps aux | grep java

# Monitor with top
top -p <sonarqube-pid>
```

---

### Issue 8: Slow Dashboard Loading
**Problem**: Web UI is extremely slow

**Solutions:**

**Solution 1: Database Optimization**
```sql
-- PostgreSQL
-- Vacuum and analyze
VACUUM ANALYZE;

-- Reindex
REINDEX DATABASE sonarqube;

-- Check slow queries
SELECT * FROM pg_stat_activity 
WHERE state = 'active' 
ORDER BY query_start;
```

**Solution 2: Clean Old Data**
```
# In UI:
# Administration → Configuration → General → Housekeeping
Days before deleting closed issues: 30
Days before deleting inactive branches: 30
Days before deleting snapshots: 30

# Or via database
DELETE FROM snapshots 
WHERE created_at < NOW() - INTERVAL '30 days';
```

**Solution 3: Optimize Elasticsearch**
```properties
# Increase ES heap
sonar.search.javaOpts=-Xmx2048m -Xms1024m

# Check ES health
curl http://localhost:9000/api/system/health
```

---

## Database Problems

### Issue 9: Database Connection Failures
**Problem**: Can't connect to database

**Symptoms:**
```
[ERROR] Unable to connect to database
org.postgresql.util.PSQLException: Connection refused
```

**Solutions:**

**Check Database Running:**
```bash
# PostgreSQL
sudo systemctl status postgresql
sudo systemctl start postgresql

# Check if listening
netstat -tuln | grep 5432
```

**Check Credentials:**
```properties
# conf/sonar.properties
sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube
sonar.jdbc.username=sonarqube
sonar.jdbc.password=sonarqube
```

**Test Connection:**
```bash
# Command line
psql -h localhost -U sonarqube -d sonarqube

# If fails, reset password
sudo -u postgres psql
ALTER USER sonarqube WITH PASSWORD 'newpassword';
```

**Check pg_hba.conf:**
```bash
# /etc/postgresql/13/main/pg_hba.conf
# Add line:
host    sonarqube    sonarqube    127.0.0.1/32    md5

# Restart PostgreSQL
sudo systemctl restart postgresql
```

---

### Issue 10: Database Migration Errors
**Problem**: Migration fails after upgrade

**Symptoms:**
```
[ERROR] Database migration failure
```

**Solutions:**

**Backup First:**
```bash
# Always backup before upgrade!
pg_dump -U sonarqube sonarqube > backup.sql
```

**Check Migration Status:**
```bash
curl -u admin:admin \
  http://localhost:9000/api/system/db_migration_status
```

**Manual Migration:**
```
# In UI:
# Administration → System → Database Migration
# Click "Migrate Database"
```

**Rollback if Needed:**
```bash
# Stop SonarQube
./bin/linux-x86-64/sonar.sh stop

# Restore database
psql -U sonarqube sonarqube < backup.sql

# Downgrade SonarQube version
```

---

## Analysis Failures

### Issue 11: "Insufficient Memory" During Analysis
**Problem**: Scanner runs out of memory

**Symptoms:**
```
[ERROR] OutOfMemoryError: Java heap space
The scanner process exited with exit code: 1
```

**Solutions:**

**Increase Scanner Memory:**
```bash
# Maven
export MAVEN_OPTS="-Xmx2048m"
mvn sonar:sonar

# Gradle
org.gradle.jvmargs=-Xmx2048m

# SonarScanner CLI
export SONAR_SCANNER_OPTS="-Xmx2048m"
sonar-scanner
```

**Exclude Large Files:**
```properties
sonar.exclusions=**/node_modules/**,**/vendor/**,**/*.min.js
```

---

### Issue 12: "SCM Provider Not Found"
**Problem**: Scanner fails to detect SCM

**Symptoms:**
```
[WARN] SCM provider autodetection failed
```

**Solutions:**

**Disable SCM if Not Needed:**
```properties
sonar.scm.disabled=true
```

**Or Install Git:**
```bash
# Ubuntu/Debian
sudo apt install git

# Mac
brew install git

# Verify
git --version
```

---

## Quality Gate Issues

### Issue 13: Quality Gate Always Passes
**Problem**: Gate passes even with issues

**Symptoms:**
```
Quality Gate: PASSED
But project has many bugs/vulnerabilities
```

**Solutions:**

**Check Gate Conditions:**
```
Quality Gates → Your Gate
- Verify conditions are set
- Check thresholds
- Ensure gate is assigned to project
```

**Check Which Code is Evaluated:**
```
# Most conditions should be on "New Code"
# Check if new code has issues
Dashboard → New Code tab
```

**Verify Gate Assignment:**
```
Project Settings → Quality Gate
# Ensure correct gate is selected
```

---

### Issue 14: Can't Wait for Quality Gate in Jenkins
**Problem**: waitForQualityGate times out

**Symptoms:**
```
Timeout waiting for quality gate
```

**Solutions:**

**Check Webhook:**
```
# In SonarQube:
# Administration → Configuration → Webhooks
# Add webhook:
# URL: http://jenkins:8080/sonarqube-webhook/
```

**Increase Timeout:**
```groovy
timeout(time: 2, unit: 'HOURS') {
    waitForQualityGate abortPipeline: true
}
```

**Check CE Tasks:**
```bash
# Check if analysis is processing
curl -u token: \
  "http://localhost:9000/api/ce/activity"
```

---

## Integration Problems

### Issue 15: Jenkins Plugin Not Working
**Problem**: SonarQube plugin fails in Jenkins

**Symptoms:**
```
[ERROR] SonarQube server not found
```

**Solutions:**

**Check Plugin Installation:**
```
Jenkins → Manage Jenkins → Manage Plugins
- Ensure "SonarQube Scanner" is installed
- Ensure "Quality Gates" plugin is installed
```

**Configure Server:**
```
Jenkins → Manage Jenkins → Configure System
→ SonarQube servers

- Name: SonarQube
- Server URL: http://sonarqube:9000
- Add Server authentication token
```

**Test Connection:**
```groovy
stage('Test Connection') {
    steps {
        sh 'curl http://sonarqube:9000/api/system/status'
    }
}
```

---

### Issue 16: GitHub PR Decoration Not Working
**Problem**: Issues not appearing on pull requests

**Symptoms:**
```
Analysis completes
No comments on GitHub PR
```

**Solutions:**

**Check ALM Integration:**
```
Administration → Configuration → ALM Integrations
- Verify GitHub configuration
- Test connection
```

**Check Project Binding:**
```
Project Settings → General → ALM Integration
- Ensure repository is linked
```

**Check Permissions:**
```
# GitHub App needs permissions:
- Pull requests: Read & Write
- Contents: Read
- Metadata: Read
```

**Verify Analysis:**
```bash
# Ensure using pullrequest parameters
mvn sonar:sonar \
  -Dsonar.pullrequest.key=123 \
  -Dsonar.pullrequest.branch=feature/branch \
  -Dsonar.pullrequest.base=main
```

---

## Performance Issues

### Issue 17: Very Slow Analysis
**Problem**: Analysis takes too long

**Solutions:**

**Optimize Scanner:**
```properties
# Exclude unnecessary files
sonar.exclusions=**/test/**,**/generated/**,**/*.min.js

# Disable SCM if not needed
sonar.scm.disabled=true

# Reduce CPD (duplication) scope
sonar.cpd.cross_project=false
```

**Parallel Build:**
```bash
# Maven
mvn clean verify sonar:sonar -T 4

# Or per module
mvn clean verify sonar:sonar -pl module1,module2 -am
```

**Scanner Memory:**
```bash
export MAVEN_OPTS="-Xmx4096m"
mvn sonar:sonar
```

**Incremental Analysis (Enterprise):**
```bash
mvn sonar:sonar -Dsonar.incrementalAnalysis=true
```

---

### Issue 18: Server Performance Degradation
**Problem**: Server becomes slower over time

**Solutions:**

**Database Maintenance:**
```sql
-- PostgreSQL
VACUUM FULL ANALYZE;
REINDEX DATABASE sonarqube;

-- Check database size
SELECT pg_size_pretty(pg_database_size('sonarqube'));
```

**Clean Old Data:**
```
Administration → Projects → Management
- Delete old/unused projects

Administration → Configuration → General
- Reduce retention periods
```

**Monitor Resources:**
```bash
# CPU and Memory
top

# Disk space
df -h

# Database connections
# PostgreSQL
SELECT count(*) FROM pg_stat_activity;
```

---

## Quick Troubleshooting Checklist

### Scanner Issues
```bash
# 1. Check scanner version
mvn sonar:help -Ddetail=true

# 2. Debug mode
mvn sonar:sonar -X -e

# 3. Check token
curl -u YOUR_TOKEN: http://localhost:9000/api/authentication/validate

# 4. Test connection
curl http://localhost:9000/api/system/status

# 5. Clear cache
rm -rf ~/.sonar/cache
mvn clean
```

### Server Issues
```bash
# 1. Check logs
tail -f logs/sonar.log
tail -f logs/web.log
tail -f logs/ce.log

# 2. Check system health
curl http://localhost:9000/api/system/health

# 3. Check services
curl http://localhost:9000/api/system/status

# 4. Database connection
psql -h localhost -U sonarqube -d sonarqube

# 5. Restart server
./bin/linux-x86-64/sonar.sh restart
```

---

## Common Error Messages

| Error | Cause | Quick Fix |
|-------|-------|-----------|
| "Not authorized" | Invalid token | Regenerate token, check permissions |
| "Coverage 0%" | Coverage not uploaded | Check JaCoCo config and report path |
| "Elasticsearch failed" | Low vm.max_map_count | Increase vm.max_map_count |
| "OutOfMemoryError" | Insufficient heap | Increase Xmx settings |
| "Database connection failed" | DB not running/wrong credentials | Start DB, verify credentials |
| "Quality gate timeout" | Missing webhook | Configure webhook in SonarQube |
| "SCM provider not found" | Git not installed | Install git or disable SCM |
| "Project not found" | Inconsistent project key | Use consistent projectKey |

---

## Getting More Help

### Logs to Check
```bash
# Server logs
logs/sonar.log        # Main log
logs/web.log          # Web server
logs/ce.log           # Compute Engine
logs/es.log           # Elasticsearch
logs/access.log       # HTTP access

# Scanner logs
# Maven: Use -X flag
# SonarScanner: Use -X flag
```

### API for Diagnostics
```bash
# System health
curl http://localhost:9000/api/system/health

# System status
curl http://localhost:9000/api/system/status

# System info
curl -u admin:admin \
  http://localhost:9000/api/system/info

# CE tasks
curl -u token: \
  http://localhost:9000/api/ce/activity

# Database migration status
curl -u admin:admin \
  http://localhost:9000/api/system/db_migration_status
```

### Community Resources
- **SonarSource Community**: https://community.sonarsource.com/
- **Stack Overflow**: [sonarqube] tag
- **GitHub Issues**: https://github.com/SonarSource/sonarqube/issues
- **Documentation**: https://docs.sonarqube.org/

---

## Best Practices to Avoid Issues

✅ **Regular Maintenance**
- Clean old projects
- Vacuum database weekly
- Monitor disk space
- Review logs regularly

✅ **Proper Configuration**
- Use quality gates appropriately
- Configure webhooks for CI/CD
- Set proper exclusions
- Version control profiles/gates

✅ **Monitoring**
- Track analysis duration
- Monitor server resources
- Check CE task queue
- Review failed analyses

✅ **Updates**
- Keep SonarQube updated
- Update scanners regularly
- Update quality profiles
- Review new rules

---

**Tip**: When troubleshooting, always start with logs (-X flag) and check system health API endpoints!


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

