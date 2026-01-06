# Maven Quick Reference Guide

## Essential Maven Commands

### Basic Commands
```bash
# Version
mvn --version
mvn -v

# Help
mvn help:help
mvn help:describe -Dplugin=compiler
mvn help:effective-pom

# Create project
mvn archetype:generate

# Lifecycle phases
mvn validate        # Validate project
mvn compile         # Compile source
mvn test           # Run tests
mvn package        # Create JAR/WAR
mvn verify         # Run integration tests
mvn install        # Install to local repo
mvn deploy         # Deploy to remote repo
mvn clean          # Clean build artifacts
mvn site           # Generate documentation

# Common combinations
mvn clean compile
mvn clean test
mvn clean package
mvn clean install
mvn clean deploy
```

### Skip Options
```bash
# Skip tests
mvn install -DskipTests              # Skip test execution
mvn install -Dmaven.test.skip=true   # Skip test compilation & execution

# Skip specific plugins
mvn install -Dcheckstyle.skip=true
mvn install -Dmaven.javadoc.skip=true
```

### Dependency Commands
```bash
# Dependency tree
mvn dependency:tree
mvn dependency:tree -Dverbose
mvn dependency:tree -Dincludes=groupId:artifactId

# Analyze dependencies
mvn dependency:analyze
mvn dependency:analyze-only
mvn dependency:analyze-duplicate

# Copy dependencies
mvn dependency:copy-dependencies
mvn dependency:copy-dependencies -DoutputDirectory=lib

# Check for updates
mvn versions:display-dependency-updates
mvn versions:display-plugin-updates

# Resolve dependencies
mvn dependency:resolve
mvn dependency:resolve-plugins

# Purge local repository
mvn dependency:purge-local-repository
```

### Build Options
```bash
# Offline mode
mvn clean install -o

# Parallel builds
mvn clean install -T 4      # 4 threads
mvn clean install -T 1C     # 1 thread per CPU core

# Debug mode
mvn clean install -X        # Debug output
mvn clean install -e        # Show stack traces

# Quiet mode
mvn clean install -q

# Update snapshots
mvn clean install -U

# Force update
mvn clean install -U -X

# Resume from module
mvn clean install -rf :module-name
```

### Profile Commands
```bash
# Activate profile
mvn clean install -Pprofile-name

# Multiple profiles
mvn clean install -Pdev,integration-test

# List active profiles
mvn help:active-profiles

# List all profiles
mvn help:all-profiles
```

### Plugin Execution
```bash
# Run specific plugin goal
mvn compiler:compile
mvn surefire:test
mvn jar:jar

# Run with configuration
mvn exec:java -Dexec.mainClass="com.example.App"
mvn exec:java -Dexec.args="arg1 arg2"

# Generate sources
mvn generate-sources

# Process resources
mvn process-resources
```

---

## POM Structure Reference

### Minimal POM
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.example</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0-SNAPSHOT</version>
</project>
```

### Complete POM Template
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- Coordinates -->
    <groupId>com.example</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <!-- Project Info -->
    <name>My Application</name>
    <description>Description of my application</description>
    <url>https://example.com</url>

    <!-- Properties -->
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <junit.version>5.9.3</junit.version>
    </properties>

    <!-- Dependencies -->
    <dependencies>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <!-- Build Configuration -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.11.0</version>
            </plugin>
        </plugins>
    </build>
</project>
```

---

## Common Plugins Configuration

### Compiler Plugin
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.11.0</version>
    <configuration>
        <source>11</source>
        <target>11</target>
        <encoding>UTF-8</encoding>
        <showWarnings>true</showWarnings>
        <showDeprecation>true</showDeprecation>
    </configuration>
</plugin>
```

### Surefire Plugin (Unit Tests)
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.1.2</version>
    <configuration>
        <includes>
            <include>**/*Test.java</include>
            <include>**/*Tests.java</include>
        </includes>
        <excludes>
            <exclude>**/*IntegrationTest.java</exclude>
        </excludes>
    </configuration>
</plugin>
```

### Failsafe Plugin (Integration Tests)
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-failsafe-plugin</artifactId>
    <version>3.1.2</version>
    <executions>
        <execution>
            <goals>
                <goal>integration-test</goal>
                <goal>verify</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

### JAR Plugin
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>3.3.0</version>
    <configuration>
        <archive>
            <manifest>
                <addClasspath>true</addClasspath>
                <mainClass>com.example.Main</mainClass>
            </manifest>
        </archive>
    </configuration>
</plugin>
```

### Shade Plugin (Fat JAR)
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>3.5.0</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <transformers>
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                        <mainClass>com.example.Main</mainClass>
                    </transformer>
                </transformers>
            </configuration>
        </execution>
    </executions>
</plugin>
```

### Spring Boot Plugin
```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <version>2.7.12</version>
    <executions>
        <execution>
            <goals>
                <goal>repackage</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

### Exec Plugin
```xml
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>exec-maven-plugin</artifactId>
    <version>3.1.0</version>
    <configuration>
        <mainClass>com.example.Main</mainClass>
        <arguments>
            <argument>arg1</argument>
            <argument>arg2</argument>
        </arguments>
    </configuration>
</plugin>
```

### Assembly Plugin
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>3.6.0</version>
    <configuration>
        <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>
        <archive>
            <manifest>
                <mainClass>com.example.Main</mainClass>
            </manifest>
        </archive>
    </configuration>
</plugin>
```

---

## Dependency Management

### Dependency Scopes
```xml
<!-- compile (default) - all classpaths -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
    <scope>compile</scope>
</dependency>

<!-- provided - JDK or container provides -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
    <scope>provided</scope>
</dependency>

<!-- runtime - not needed for compilation -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.33</version>
    <scope>runtime</scope>
</dependency>

<!-- test - testing only -->
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.9.3</version>
    <scope>test</scope>
</dependency>
```

### Excluding Transitive Dependencies
```xml
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
```

### Dependency Management Section
```xml
<dependencyManagement>
    <dependencies>
        <!-- Define versions here -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.7.12</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <!-- Use without version -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

---

## Profiles

### Basic Profile
```xml
<profiles>
    <profile>
        <id>dev</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <env>development</env>
        </properties>
    </profile>
    
    <profile>
        <id>prod</id>
        <properties>
            <env>production</env>
        </properties>
        <build>
            <plugins>
                <!-- Production-specific plugins -->
            </plugins>
        </build>
    </profile>
</profiles>
```

### Profile Activation
```xml
<!-- By JDK -->
<activation>
    <jdk>11</jdk>
</activation>

<!-- By OS -->
<activation>
    <os>
        <family>windows</family>
    </os>
</activation>

<!-- By Property -->
<activation>
    <property>
        <name>environment</name>
        <value>test</value>
    </property>
</activation>

<!-- By File -->
<activation>
    <file>
        <exists>src/main/config/dev.properties</exists>
    </file>
</activation>
```

---

## Multi-Module Project

### Parent POM
```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.example</groupId>
    <artifactId>parent</artifactId>
    <version>1.0</version>
    <packaging>pom</packaging>
    
    <modules>
        <module>module-a</module>
        <module>module-b</module>
    </modules>
    
    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>
    
    <dependencyManagement>
        <dependencies>
            <!-- Shared versions -->
        </dependencies>
    </dependencyManagement>
</project>
```

### Module POM
```xml
<project>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>parent</artifactId>
        <version>1.0</version>
    </parent>
    
    <artifactId>module-a</artifactId>
    
    <dependencies>
        <!-- Module-specific dependencies -->
    </dependencies>
</project>
```

---

## Settings.xml Configuration

### User Settings (~/.m2/settings.xml)
```xml
<settings>
    <!-- Local Repository -->
    <localRepository>${user.home}/.m2/repository</localRepository>
    
    <!-- Offline Mode -->
    <offline>false</offline>
    
    <!-- Mirrors -->
    <mirrors>
        <mirror>
            <id>central-mirror</id>
            <mirrorOf>central</mirrorOf>
            <url>https://repo.maven.apache.org/maven2</url>
        </mirror>
    </mirrors>
    
    <!-- Servers (credentials) -->
    <servers>
        <server>
            <id>nexus</id>
            <username>deployment</username>
            <password>password123</password>
        </server>
    </servers>
    
    <!-- Proxies -->
    <proxies>
        <proxy>
            <id>http-proxy</id>
            <active>true</active>
            <protocol>http</protocol>
            <host>proxy.example.com</host>
            <port>8080</port>
            <username>proxyuser</username>
            <password>proxypass</password>
        </proxy>
    </proxies>
    
    <!-- Active Profiles -->
    <activeProfiles>
        <activeProfile>dev</activeProfile>
    </activeProfiles>
</settings>
```

---

## Common Archetypes

```bash
# Java Application
mvn archetype:generate \
  -DgroupId=com.example \
  -DartifactId=my-app \
  -DarchetypeArtifactId=maven-archetype-quickstart

# Web Application
mvn archetype:generate \
  -DarchetypeArtifactId=maven-archetype-webapp

# Maven Plugin
mvn archetype:generate \
  -DarchetypeArtifactId=maven-archetype-plugin

# Spring Boot
mvn archetype:generate \
  -DarchetypeGroupId=org.springframework.boot \
  -DarchetypeArtifactId=spring-boot-starter-parent
```

---

## Useful Properties

```xml
<properties>
    <!-- Maven Properties -->
    ${project.groupId}
    ${project.artifactId}
    ${project.version}
    ${project.name}
    ${project.basedir}
    ${project.build.directory}        <!-- target/ -->
    ${project.build.outputDirectory}   <!-- target/classes -->
    
    <!-- System Properties -->
    ${user.home}
    ${user.name}
    ${java.version}
    
    <!-- Environment Variables -->
    ${env.PATH}
    ${env.JAVA_HOME}
    
    <!-- Settings -->
    ${settings.localRepository}
</properties>
```

---

## Maven Wrapper

### Generate Wrapper
```bash
mvn wrapper:wrapper
# OR
mvn -N wrapper:wrapper
```

### Use Wrapper
```bash
# Unix/Mac
./mvnw clean install

# Windows
mvnw.cmd clean install
```

### Specify Maven Version
```bash
mvn wrapper:wrapper -Dmaven=3.9.6
```

---

## Troubleshooting Commands

```bash
# Force update dependencies
mvn clean install -U

# Debug mode
mvn clean install -X

# Show effective POM
mvn help:effective-pom

# Show effective settings
mvn help:effective-settings

# Dependency tree
mvn dependency:tree

# Analyze dependency issues
mvn dependency:analyze

# Purge local repository
mvn dependency:purge-local-repository

# Display plugin info
mvn help:describe -Dplugin=compiler

# Verify checksums
mvn -C clean install
```

---

## Performance Tips

```bash
# Parallel builds
mvn clean install -T 1C

# Offline mode
mvn clean install -o

# Skip tests
mvn clean install -DskipTests

# Maven daemon (faster builds)
mvnd clean install
```

---

## Common Patterns

### Spring Boot Application
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.7.12</version>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

### REST API Project
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

---

## Maven Lifecycle Phases

### Default Lifecycle
```
validate
initialize
generate-sources
process-sources
generate-resources
process-resources
compile
process-classes
generate-test-sources
process-test-sources
generate-test-resources
process-test-resources
test-compile
process-test-classes
test
prepare-package
package
pre-integration-test
integration-test
post-integration-test
verify
install
deploy
```

### Clean Lifecycle
```
pre-clean
clean
post-clean
```

### Site Lifecycle
```
pre-site
site
post-site
site-deploy
```

---

## Environment Variables

```bash
# Maven Home
export MAVEN_HOME=/usr/local/maven
export PATH=$MAVEN_HOME/bin:$PATH

# Maven Options
export MAVEN_OPTS="-Xmx2048m -XX:MaxPermSize=512m"

# Skip Tests
export MAVEN_SKIP_TESTS=true
```

---

## Quick Troubleshooting

| Problem | Solution |
|---------|----------|
| Dependency not found | `mvn clean install -U` |
| Plugin not found | Check plugin repository |
| Build slow | Use `-T 4` for parallel |
| Tests failing | `mvn clean test -X` |
| Corrupted .m2 | Delete and rebuild |
| Version conflict | `mvn dependency:tree` |

---

**Tip**: Use `mvn help:describe -Dplugin=<plugin-name>` to get detailed plugin information!


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

