# Maven Troubleshooting Guide

Common Maven problems and their solutions.

---

## Table of Contents
1. [Build Issues](#build-issues)
2. [Dependency Problems](#dependency-problems)
3. [Plugin Issues](#plugin-issues)
4. [Repository Problems](#repository-problems)
5. [Compilation Errors](#compilation-errors)
6. [Test Failures](#test-failures)
7. [Performance Issues](#performance-issues)
8. [Configuration Problems](#configuration-problems)

---

## Build Issues

### Issue 1: Build Fails with "Maven not found"
**Problem**: Command `mvn` not recognized

**Symptoms:**
```bash
mvn clean install
# bash: mvn: command not found
```

**Solutions:**
```bash
# Check if Maven is installed
mvn --version

# Windows - Add to PATH
setx MAVEN_HOME "C:\Program Files\apache-maven-3.9.6"
setx PATH "%PATH%;%MAVEN_HOME%\bin"

# Mac/Linux - Add to ~/.bash_profile or ~/.zshrc
export MAVEN_HOME=/usr/local/maven
export PATH=$MAVEN_HOME/bin:$PATH

# Verify
mvn --version
```

---

### Issue 2: "Non-resolvable parent POM"
**Problem**: Cannot resolve parent POM

**Symptoms:**
```
[ERROR] Non-resolvable parent POM: Could not find artifact
com.example:parent:pom:1.0
```

**Solutions:**
```bash
# Solution 1: Build parent first
cd parent-project
mvn clean install

# Solution 2: Update repositories in settings.xml
<repositories>
    <repository>
        <id>central</id>
        <url>https://repo.maven.apache.org/maven2</url>
    </repository>
</repositories>

# Solution 3: Check parent coordinates
<parent>
    <groupId>com.example</groupId>
    <artifactId>parent</artifactId>
    <version>1.0</version>  <!-- Verify this exists -->
</parent>

# Solution 4: Force update
mvn clean install -U
```

---

### Issue 3: "Project build error: Unknown packaging"
**Problem**: Invalid packaging type

**Symptoms:**
```
[ERROR] Unknown packaging: invalid-type
```

**Solutions:**
```xml
<!-- Use valid packaging types -->
<packaging>jar</packaging>      <!-- Java library -->
<packaging>war</packaging>      <!-- Web application -->
<packaging>ear</packaging>      <!-- Enterprise application -->
<packaging>pom</packaging>      <!-- Parent/aggregator -->
<packaging>maven-plugin</packaging>  <!-- Maven plugin -->

<!-- If using custom packaging, ensure plugin is configured -->
```

---

## Dependency Problems

### Issue 4: "Dependency not found" Error
**Problem**: Maven cannot find dependency

**Symptoms:**
```
[ERROR] Failed to execute goal on project: Could not resolve dependencies
for project com.example:app:jar:1.0: Could not find artifact
org.example:library:jar:1.0
```

**Solutions:**
```bash
# Solution 1: Force update dependencies
mvn clean install -U

# Solution 2: Check dependency coordinates
mvn dependency:get -Dartifact=groupId:artifactId:version

# Solution 3: Clear local repository
rm -rf ~/.m2/repository/org/example/library
mvn clean install

# Solution 4: Add repository where artifact is hosted
```

```xml
<repositories>
    <repository>
        <id>custom-repo</id>
        <url>https://custom-repo.example.com/maven2</url>
    </repository>
</repositories>
```

```bash
# Solution 5: Check if artifact exists
# Search on Maven Central: https://search.maven.org/
```

---

### Issue 5: Dependency Conflict
**Problem**: Multiple versions of same dependency

**Symptoms:**
```
[WARNING] Some problems were encountered while building the effective model
[WARNING] 'dependencies.dependency.version' for org.example:library:jar
is either LATEST or RELEASE (both of them are being deprecated)
```

**Solutions:**
```bash
# View dependency tree
mvn dependency:tree

# Find conflicts
mvn dependency:tree -Dverbose

# Analyze dependencies
mvn dependency:analyze
```

```xml
<!-- Solution 1: Exclude conflicting transitive dependency -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>5.3.27</version>
    <exclusions>
        <exclusion>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<!-- Solution 2: Use dependencyManagement to force version -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
            <version>1.2</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

---

### Issue 6: "Missing artifact" in Local Repository
**Problem**: Artifact exists but Maven can't find it

**Symptoms:**
```
[ERROR] Failed to read artifact descriptor: Could not find artifact
```

**Solutions:**
```bash
# Solution 1: Delete and re-download
rm -rf ~/.m2/repository/problematic/artifact
mvn clean install -U

# Solution 2: Purge local repository
mvn dependency:purge-local-repository

# Solution 3: Check for corrupted files
find ~/.m2/repository -name "*.lastUpdated" -delete
mvn clean install -U

# Solution 4: Verify local repository location
mvn help:effective-settings | grep localRepository
```

---

## Plugin Issues

### Issue 7: Plugin Not Found
**Problem**: Maven cannot find plugin

**Symptoms:**
```
[ERROR] Plugin org.apache.maven.plugins:maven-compiler-plugin:999.0
or one of its dependencies could not be resolved
```

**Solutions:**
```xml
<!-- Solution 1: Specify correct plugin version -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.11.0</version>  <!-- Use valid version -->
</plugin>

<!-- Solution 2: Add plugin repository if needed -->
<pluginRepositories>
    <pluginRepository>
        <id>central</id>
        <url>https://repo.maven.apache.org/maven2</url>
    </pluginRepository>
</pluginRepositories>
```

```bash
# Solution 3: Update plugin
mvn versions:display-plugin-updates

# Solution 4: Force update
mvn clean install -U
```

---

### Issue 8: Plugin Execution Failed
**Problem**: Plugin fails during execution

**Symptoms:**
```
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.11.0:compile
```

**Solutions:**
```bash
# Debug mode for details
mvn clean install -X

# Check plugin configuration
mvn help:describe -Dplugin=compiler -Ddetail

# View effective POM
mvn help:effective-pom
```

```xml
<!-- Common fixes -->
<!-- Fix 1: Correct configuration -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.11.0</version>
    <configuration>
        <source>11</source>
        <target>11</target>
    </configuration>
</plugin>

<!-- Fix 2: Skip if not needed -->
<plugin>
    <configuration>
        <skip>true</skip>
    </configuration>
</plugin>
```

---

## Repository Problems

### Issue 9: "Connection refused" to Repository
**Problem**: Cannot connect to Maven repository

**Symptoms:**
```
[ERROR] Failed to retrieve plugin descriptor: Connection refused
```

**Solutions:**
```bash
# Solution 1: Check internet connection
ping repo.maven.apache.org

# Solution 2: Use offline mode (if dependencies already downloaded)
mvn clean install -o

# Solution 3: Configure proxy in settings.xml
```

```xml
<settings>
    <proxies>
        <proxy>
            <id>http-proxy</id>
            <active>true</active>
            <protocol>http</protocol>
            <host>proxy.example.com</host>
            <port>8080</port>
            <username>proxyuser</username>
            <password>proxypass</password>
            <nonProxyHosts>localhost|127.0.0.1</nonProxyHosts>
        </proxy>
    </proxies>
</settings>
```

```bash
# Solution 4: Use mirror
```

```xml
<mirrors>
    <mirror>
        <id>central-mirror</id>
        <mirrorOf>central</mirrorOf>
        <url>https://repo.maven.apache.org/maven2</url>
    </mirror>
</mirrors>
```

---

### Issue 10: "401 Unauthorized" Error
**Problem**: Authentication required for repository

**Symptoms:**
```
[ERROR] Failed to deploy artifacts: Return code is: 401, ReasonPhrase: Unauthorized
```

**Solutions:**
```xml
<!-- Add server credentials in settings.xml -->
<settings>
    <servers>
        <server>
            <id>nexus</id>
            <username>deployment-user</username>
            <password>your-password</password>
        </server>
    </servers>
</settings>
```

```bash
# Encrypt password
mvn --encrypt-master-password
mvn --encrypt-password your-password

# Use encrypted password in settings.xml
```

---

## Compilation Errors

### Issue 11: "Invalid target release" Error
**Problem**: Java version mismatch

**Symptoms:**
```
[ERROR] Fatal error compiling: invalid target release: 17
```

**Solutions:**
```bash
# Check Java version
java -version
javac -version

# Ensure Java 17+ installed if targeting 17
```

```xml
<!-- Solution: Match Java version with Maven compiler settings -->
<properties>
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
</properties>

<!-- Or use plugin configuration -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.11.0</version>
    <configuration>
        <source>11</source>
        <target>11</target>
        <release>11</release>  <!-- Java 9+ -->
    </configuration>
</plugin>
```

---

### Issue 12: "Package does not exist" Error
**Problem**: Cannot find imported classes

**Symptoms:**
```
[ERROR] package org.example does not exist
```

**Solutions:**
```bash
# Solution 1: Verify dependency is added
mvn dependency:tree | grep example

# Solution 2: Force dependency update
mvn clean install -U

# Solution 3: Reimport in IDE
# IntelliJ: File > Invalidate Caches / Restart
# Eclipse: Project > Clean

# Solution 4: Check dependency scope
```

```xml
<!-- Ensure dependency has correct scope -->
<dependency>
    <groupId>org.example</groupId>
    <artifactId>library</artifactId>
    <version>1.0</version>
    <!-- Should be 'compile' or no scope for normal use -->
    <scope>compile</scope>
</dependency>
```

---

## Test Failures

### Issue 13: Tests Not Running
**Problem**: Maven doesn't execute tests

**Symptoms:**
```
[INFO] Tests are skipped
```

**Solutions:**
```bash
# Check if tests are skipped
mvn test

# Don't skip tests
mvn test -DskipTests=false

# Check test file naming
# Surefire looks for:
# **/Test*.java
# **/*Test.java
# **/*Tests.java
# **/*TestCase.java
```

```xml
<!-- Configure Surefire plugin -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.1.2</version>
    <configuration>
        <includes>
            <include>**/*Test.java</include>
            <include>**/*Tests.java</include>
        </includes>
    </configuration>
</plugin>
```

---

### Issue 14: "No tests were executed"
**Problem**: Test files found but not executed

**Symptoms:**
```
[INFO] No tests were executed!
```

**Solutions:**
```xml
<!-- Solution 1: Add JUnit dependency -->
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.9.3</version>
    <scope>test</scope>
</dependency>

<!-- Solution 2: Configure Surefire for JUnit 5 -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.1.2</version>
</plugin>
```

```java
// Ensure tests have @Test annotation
import org.junit.jupiter.api.Test;

public class MyTest {
    @Test  // Don't forget this!
    public void testSomething() {
        // test code
    }
}
```

---

## Performance Issues

### Issue 15: Slow Build Times
**Problem**: Maven builds take too long

**Solutions:**
```bash
# Solution 1: Parallel builds
mvn clean install -T 4     # 4 threads
mvn clean install -T 1C    # 1 thread per CPU core

# Solution 2: Offline mode
mvn clean install -o

# Solution 3: Skip tests when appropriate
mvn clean install -DskipTests

# Solution 4: Use Maven Daemon
mvnd clean install  # Requires mvnd installation

# Solution 5: Optimize dependency resolution
```

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-dependency-plugin</artifactId>
            <configuration>
                <ignoreNonCompile>true</ignoreNonCompile>
            </configuration>
        </plugin>
    </plugins>
</build>
```

---

### Issue 16: Out of Memory Error
**Problem**: Maven runs out of heap memory

**Symptoms:**
```
[ERROR] Java heap space
[ERROR] GC overhead limit exceeded
```

**Solutions:**
```bash
# Increase heap size
export MAVEN_OPTS="-Xmx2048m -XX:MaxPermSize=512m"

# Windows
set MAVEN_OPTS=-Xmx2048m -XX:MaxPermSize=512m

# Or in .mavenrc (Unix) / .mavenrc.cmd (Windows)
MAVEN_OPTS="-Xmx4096m -XX:MaxMetaspaceSize=1024m"
```

---

## Configuration Problems

### Issue 17: "settings.xml" Not Found
**Problem**: Maven cannot find settings file

**Solutions:**
```bash
# Check settings location
~/.m2/settings.xml  # User settings
$MAVEN_HOME/conf/settings.xml  # Global settings

# Create if missing
mkdir -p ~/.m2
touch ~/.m2/settings.xml
```

```xml
<!-- Minimal settings.xml -->
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
          http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <localRepository>${user.home}/.m2/repository</localRepository>
</settings>
```

---

### Issue 18: Profile Not Activating
**Problem**: Maven profile not being used

**Solutions:**
```bash
# Check active profiles
mvn help:active-profiles

# List all profiles
mvn help:all-profiles

# Explicitly activate
mvn clean install -Pprod

# Multiple profiles
mvn clean install -Pprofile1,profile2
```

```xml
<!-- Check activation conditions -->
<profile>
    <id>dev</id>
    <activation>
        <activeByDefault>true</activeByDefault>
        <!-- OR -->
        <property>
            <name>env</name>
            <value>dev</value>
        </property>
    </activation>
</profile>
```

---

## Quick Troubleshooting

### General Debug Steps
```bash
# 1. Debug mode (verbose output)
mvn clean install -X

# 2. Show error traces
mvn clean install -e

# 3. Force update dependencies
mvn clean install -U

# 4. View effective POM
mvn help:effective-pom

# 5. View effective settings
mvn help:effective-settings

# 6. Analyze dependencies
mvn dependency:tree
mvn dependency:analyze

# 7. Clean local repository
mvn dependency:purge-local-repository

# 8. Verify project
mvn validate

# 9. Check plugin info
mvn help:describe -Dplugin=<plugin-name>
```

---

## Common Error Messages

| Error | Likely Cause | Quick Fix |
|-------|-------------|-----------|
| "Non-resolvable parent POM" | Parent not in local repo | Build parent first |
| "Dependency not found" | Wrong coordinates or offline | Check coordinates, use -U |
| "Plugin not found" | Wrong version or offline | Check version, use -U |
| "Invalid target release" | Java version mismatch | Match Java and Maven versions |
| "Connection refused" | Network/proxy issue | Check network, configure proxy |
| "401 Unauthorized" | Missing credentials | Add to settings.xml |
| "Tests not running" | Wrong naming or missing plugin | Check file names, add Surefire |
| "Out of memory" | Heap too small | Increase MAVEN_OPTS |

---

## Preventive Measures

### Best Practices
✅ Lock plugin versions
✅ Use dependency management
✅ Regular dependency updates
✅ Clean builds periodically
✅ Maintain clean local repository
✅ Use Maven Wrapper
✅ Enable parallel builds
✅ Configure proper memory
✅ Use profiles correctly
✅ Document custom configurations

### Maintenance Commands
```bash
# Weekly
mvn dependency:analyze
mvn versions:display-dependency-updates

# Monthly
mvn dependency:purge-local-repository
find ~/.m2/repository -name "*.lastUpdated" -delete

# Before release
mvn clean install -U
mvn help:effective-pom > effective-pom.xml
mvn dependency:tree > dependency-tree.txt
```

---

## Getting More Help

```bash
# Maven help
mvn help:help

# Plugin help
mvn help:describe -Dplugin=compiler -Ddetail

# System info
mvn --version
mvn -v

# Debug logging
mvn clean install -X > build-debug.log 2>&1
```

### Resources
- **Maven Documentation**: https://maven.apache.org/guides/
- **Stack Overflow**: [maven] tag
- **Maven Users List**: users@maven.apache.org

---

**Tip**: When stuck, start with `mvn clean install -X -U` for detailed output and forced updates!


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

