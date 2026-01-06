# Maven Interview Questions
## From Basic to Advanced

Complete interview preparation covering fundamentals to expert-level Maven knowledge.

---

## Table of Contents
1. [Basic Questions](#basic-questions)
2. [POM & Dependencies](#pom--dependencies)
3. [Build Lifecycle & Plugins](#build-lifecycle--plugins)
4. [Multi-Module Projects](#multi-module-projects)
5. [Advanced Topics](#advanced-topics)
6. [Best Practices](#best-practices)
7. [Scenario-Based Questions](#scenario-based-questions)
8. [Behavioral Questions](#behavioral-questions)

---

## Basic Questions

### Q1: What is Maven and why use it?
**Expected Answer:**

Maven is a build automation and project management tool for Java projects.

**Key Benefits:**
- **Dependency Management**: Automatic download and management
- **Build Automation**: Standardized build lifecycle
- **Convention over Configuration**: Standard project structure
- **Plugin Ecosystem**: Extensive plugins for various tasks
- **Project Documentation**: Automated site generation
- **Reproducible Builds**: Same build everywhere

**Core Concepts:**
- POM (Project Object Model)
- Build lifecycle
- Dependency management
- Plugin architecture
- Repository system

**Follow-up**: What's the difference between Maven and Gradle?

**Answer**: Maven uses XML (declarative), standardized, convention-based. Gradle uses Groovy/Kotlin DSL (imperative), more flexible, faster for large projects.

---

### Q2: Explain the Maven project structure
**Expected Answer:**

**Standard Directory Layout:**
```
project/
├── pom.xml                    # Project Object Model
├── src/
│   ├── main/
│   │   ├── java/              # Java source code
│   │   ├── resources/         # Resources (properties, XML)
│   │   └── webapp/            # Web resources (for WAR)
│   └── test/
│       ├── java/              # Test source code
│       └── resources/         # Test resources
└── target/                    # Build output (generated)
    ├── classes/               # Compiled classes
    ├── test-classes/          # Compiled test classes
    └── *.jar / *.war          # Final artifact
```

**Benefits of Convention:**
- No configuration needed for standard layouts
- Everyone knows where to find things
- IDE integration works automatically
- Build tools know what to do

**Follow-up**: Can you customize the structure?

**Answer**: Yes, using `<build><sourceDirectory>` and `<testSourceDirectory>` in POM, but discouraged unless necessary.

---

### Q3: What is POM? Explain its structure.
**Expected Answer:**

**POM (Project Object Model)** is the fundamental unit of work in Maven - an XML file (pom.xml) containing project information and configuration.

**Key Sections:**
```xml
<project>
    <!-- 1. Model Version -->
    <modelVersion>4.0.0</modelVersion>
    
    <!-- 2. Coordinates (GAV) -->
    <groupId>com.example</groupId>        <!-- Organization -->
    <artifactId>my-app</artifactId>       <!-- Project name -->
    <version>1.0-SNAPSHOT</version>       <!-- Version -->
    <packaging>jar</packaging>            <!-- Type -->
    
    <!-- 3. Project Info -->
    <name>My Application</name>
    <description>Application description</description>
    <url>http://example.com</url>
    
    <!-- 4. Properties -->
    <properties>
        <maven.compiler.source>11</maven.compiler.source>
    </properties>
    
    <!-- 5. Dependencies -->
    <dependencies>...</dependencies>
    
    <!-- 6. Build Configuration -->
    <build>...</build>
    
    <!-- 7. Profiles -->
    <profiles>...</profiles>
</project>
```

**POM Types:**
- **Super POM**: Maven's default POM
- **Parent POM**: Inherited by child projects
- **Effective POM**: Result of merging all POMs

**Follow-up**: What is the effective POM?

**Answer**: `mvn help:effective-pom` - shows the final POM after inheriting from parent and super POM.

---

### Q4: Explain Maven coordinates (GAV)
**Expected Answer:**

**GAV** - GroupId, ArtifactId, Version - uniquely identifies a Maven artifact.

**Components:**
```xml
<groupId>com.example</groupId>          <!-- Organization/package -->
<artifactId>my-app</artifactId>         <!-- Project name -->
<version>1.0-SNAPSHOT</version>         <!-- Version -->
<packaging>jar</packaging>              <!-- Optional, defaults to jar -->
```

**GroupId:**
- Reverse domain name convention
- Examples: `org.apache.commons`, `com.google.guava`
- Should be owned/controlled by organization

**ArtifactId:**
- Project name
- Should be lowercase
- No special characters except hyphens

**Version:**
- Semantic versioning: MAJOR.MINOR.PATCH
- `SNAPSHOT`: Development version
- Without SNAPSHOT: Release version

**Repository Path:**
```
~/.m2/repository/com/example/my-app/1.0-SNAPSHOT/my-app-1.0-SNAPSHOT.jar
                 └─groupId─┘ └artifactId┘└─version──┘└──────filename────┘
```

---

### Q5: What is SNAPSHOT in Maven?
**Expected Answer:**

**SNAPSHOT** is a special version indicating the project is in development.

**Characteristics:**
- Updated frequently during development
- Maven checks for newer SNAPSHOT on each build (with -U)
- Never deployed to Maven Central
- Should not be used in production

**Example:**
```xml
<version>1.0-SNAPSHOT</version>
```

**Release Process:**
```
1.0-SNAPSHOT  (development)
     ↓
1.0           (release)
     ↓
1.1-SNAPSHOT  (next development cycle)
```

**Update SNAPSHOT:**
```bash
mvn clean install -U  # Force update SNAPSHOT dependencies
```

**Best Practices:**
- Use SNAPSHOT during development
- Remove SNAPSHOT for releases
- Don't depend on SNAPSHOTs in production
- Use release plugin to manage versions

---

## POM & Dependencies

### Q6: Explain dependency scopes in Maven
**Expected Answer:**

**Six Dependency Scopes:**

**1. compile (default)**
```xml
<scope>compile</scope>
```
- Available in all classpaths
- Transitive to dependent projects
- Use for: Main application dependencies

**2. provided**
```xml
<scope>provided</scope>
```
- Available at compile time
- NOT included in package (container provides)
- Use for: Servlet API, Java EE APIs

**3. runtime**
```xml
<scope>runtime</scope>
```
- NOT available at compile time
- Available at runtime and test
- Use for: JDBC drivers, logging implementations

**4. test**
```xml
<scope>test</scope>
```
- Only available for test compilation and execution
- Not transitive
- Use for: JUnit, Mockito

**5. system**
```xml
<scope>system</scope>
<systemPath>/path/to/jar</systemPath>
```
- Similar to provided but must specify path
- Not recommended (breaks portability)

**6. import**
```xml
<scope>import</scope>
<type>pom</type>
```
- Only in `<dependencyManagement>`
- Imports dependency management from other POMs
- Use for: BOMs (Bill of Materials)

**Follow-up**: What happens if no scope is specified?

**Answer**: Defaults to `compile` scope.

---

### Q7: What is dependencyManagement?
**Expected Answer:**

**dependencyManagement** section declares dependencies and versions without actually adding them to the project.

**Purpose:**
- Centralize dependency versions
- Ensure consistency across modules
- Child projects inherit versions

**Parent POM:**
```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>5.3.27</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

**Child POM:**
```xml
<dependencies>
    <!-- No version needed - inherited from parent -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
    </dependency>
</dependencies>
```

**Benefits:**
- Single place to manage versions
- Avoid version conflicts
- Easy to update dependencies
- Enforce consistency

**Follow-up**: Difference between dependencies and dependencyManagement?

**Answer**: `dependencies` actually adds the dependency. `dependencyManagement` only declares version/scope for child projects to use.

---

### Q8: How do you handle dependency conflicts?
**Expected Answer:**

**Dependency Conflict** occurs when different versions of the same artifact are required.

**Maven's Resolution Strategy:**
- **Nearest Definition**: Closest to project in dependency tree
- **First Declaration**: If equal distance, first declared wins

**Troubleshooting:**
```bash
# View dependency tree
mvn dependency:tree

# Verbose (shows conflicts)
mvn dependency:tree -Dverbose

# Find specific dependency
mvn dependency:tree -Dincludes=groupId:artifactId
```

**Solutions:**

**1. Exclude Transitive Dependency:**
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

**2. Use dependencyManagement:**
```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
            <version>1.2</version>  <!-- Force this version -->
        </dependency>
    </dependencies>
</dependencyManagement>
```

**3. Declare Direct Dependency:**
```xml
<dependencies>
    <!-- Explicit dependency takes precedence -->
    <dependency>
        <groupId>commons-logging</groupId>
        <artifactId>commons-logging</artifactId>
        <version>1.2</version>
    </dependency>
</dependencies>
```

---

## Build Lifecycle & Plugins

### Q9: Explain Maven build lifecycle
**Expected Answer:**

Maven has **three built-in lifecycles**:

**1. default** (main build)
```
validate → compile → test → package → verify → install → deploy
```

**Key Phases:**
- `validate`: Validate project structure
- `compile`: Compile source code
- `test`: Run unit tests
- `package`: Create JAR/WAR
- `verify`: Run integration tests
- `install`: Install to local repository
- `deploy`: Deploy to remote repository

**2. clean** (cleanup)
```
pre-clean → clean → post-clean
```
- `clean`: Delete target directory

**3. site** (documentation)
```
pre-site → site → post-site → site-deploy
```
- `site`: Generate project documentation

**Execution:**
```bash
mvn clean install
# Runs: clean, then all default phases up to install

mvn test
# Runs: validate, compile, test (and all phases before test)
```

**Important**: Each phase executes all previous phases.

---

### Q10: What are Maven plugins? Name important ones.
**Expected Answer:**

**Plugins** provide goals that perform specific tasks in Maven build.

**Plugin Types:**
- **Build plugins**: Execute during build (configured in `<build><plugins>`)
- **Reporting plugins**: Execute during site generation (configured in `<reporting><plugins>`)

**Essential Plugins:**

**1. Compiler Plugin**
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.11.0</version>
    <configuration>
        <source>11</source>
        <target>11</target>
    </configuration>
</plugin>
```

**2. Surefire (Unit Tests)**
```xml
<plugin>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.1.2</version>
</plugin>
```

**3. Failsafe (Integration Tests)**
```xml
<plugin>
    <artifactId>maven-failsafe-plugin</artifactId>
    <version>3.1.2</version>
</plugin>
```

**4. JAR Plugin**
```xml
<plugin>
    <artifactId>maven-jar-plugin</artifactId>
    <version>3.3.0</version>
</plugin>
```

**5. Shade Plugin (Fat JAR)**
```xml
<plugin>
    <artifactId>maven-shade-plugin</artifactId>
    <version>3.5.0</version>
</plugin>
```

**Execute Plugin Goal:**
```bash
mvn compiler:compile
mvn surefire:test
mvn <plugin>:<goal>
```

---

### Q11: What is pluginManagement?
**Expected Answer:**

**pluginManagement** declares plugin configuration without activating it.

**Purpose:**
- Centralize plugin versions and configuration
- Child modules inherit configuration
- Avoid duplication

**Parent POM:**
```xml
<build>
    <pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.11.0</version>
                <configuration>
                    <source>11</source>
                    <target>11</target>
                </configuration>
            </plugin>
        </plugins>
    </pluginManagement>
</build>
```

**Child POM:**
```xml
<build>
    <plugins>
        <!-- Just reference - configuration inherited -->
        <plugin>
            <artifactId>maven-compiler-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

**Similar to dependencyManagement** but for plugins.

---

## Multi-Module Projects

### Q12: Explain multi-module Maven projects
**Expected Answer:**

**Multi-module project** (reactor build) contains parent POM and multiple child modules.

**Structure:**
```
parent-project/
├── pom.xml (parent)
├── module-a/
│   └── pom.xml
├── module-b/
│   └── pom.xml
└── module-c/
    └── pom.xml
```

**Parent POM:**
```xml
<project>
    <groupId>com.example</groupId>
    <artifactId>parent</artifactId>
    <version>1.0</version>
    <packaging>pom</packaging>  <!-- Must be pom -->
    
    <modules>
        <module>module-a</module>
        <module>module-b</module>
        <module>module-c</module>
    </modules>
    
    <dependencyManagement>
        <!-- Shared dependency versions -->
    </dependencyManagement>
</project>
```

**Child POM:**
```xml
<project>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>parent</artifactId>
        <version>1.0</version>
    </parent>
    
    <artifactId>module-a</artifactId>
    <!-- Inherits groupId and version from parent -->
</project>
```

**Build:**
```bash
# Build all modules
mvn clean install

# Build specific module
mvn clean install -pl module-a

# Build module with dependencies
mvn clean install -pl module-a -am

# Build dependents
mvn clean install -pl module-a -amd
```

**Benefits:**
- Share configuration
- Build order handled automatically
- Inter-module dependencies
- Consistent versioning

---

### Q13: What is reactor in Maven?
**Expected Answer:**

**Reactor** is Maven's mechanism for handling multi-module builds.

**Responsibilities:**
- Collect all modules
- Sort by dependencies
- Build in correct order

**Reactor Build Order:**
```
[INFO] Reactor Build Order:
[INFO] parent
[INFO] module-core
[INFO] module-service (depends on core)
[INFO] module-web (depends on service)
```

**Reactor Options:**
```bash
# Build from specific module
mvn install -rf :module-b

# Build specific modules
mvn install -pl module-a,module-b

# Build module and dependencies
mvn install -pl module-c -am  # also makes

# Build module and dependents
mvn install -pl module-a -amd  # also make dependents

# Resume from failed module
mvn install -rf :failed-module
```

**Follow-up**: What if there's a circular dependency?

**Answer**: Maven will fail with "The projects in the reactor contain a cyclic reference" error.

---

## Advanced Topics

### Q14: What are Maven profiles?
**Expected Answer:**

**Profiles** allow different build configurations for different environments.

**Use Cases:**
- Different configurations per environment (dev, test, prod)
- OS-specific builds
- JDK-specific configurations
- Custom build scenarios

**Basic Profile:**
```xml
<profiles>
    <profile>
        <id>dev</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <env>development</env>
            <db.url>jdbc:h2:mem:devdb</db.url>
        </properties>
    </profile>
    
    <profile>
        <id>prod</id>
        <properties>
            <env>production</env>
            <db.url>jdbc:mysql://prod:3306/proddb</db.url>
        </properties>
        <build>
            <plugins>
                <!-- Production-specific plugins -->
            </plugins>
        </build>
    </profile>
</profiles>
```

**Activation:**
```bash
# Command line
mvn clean install -Pprod

# Multiple profiles
mvn clean install -Pprofile1,profile2
```

**Auto-Activation:**
```xml
<activation>
    <!-- By JDK -->
    <jdk>11</jdk>
    
    <!-- By OS -->
    <os>
        <family>windows</family>
    </os>
    
    <!-- By property -->
    <property>
        <name>env</name>
        <value>test</value>
    </property>
    
    <!-- By file -->
    <file>
        <exists>src/main/config/dev.properties</exists>
    </file>
</activation>
```

---

### Q15: What is Maven repository? Types?
**Expected Answer:**

**Maven Repository** stores project artifacts (JARs, POMs, etc.).

**Three Types:**

**1. Local Repository**
```
~/.m2/repository/
```
- On developer's machine
- First place Maven checks
- Downloaded artifacts cached here

**2. Central Repository**
```
https://repo.maven.apache.org/maven2
```
- Public repository
- Maintained by Maven community
- No configuration needed
- Contains millions of artifacts

**3. Remote Repository**
```xml
<repositories>
    <repository>
        <id>company-repo</id>
        <url>https://repo.company.com/maven2</url>
    </repository>
</repositories>
```
- Corporate/custom repositories
- Nexus, Artifactory, etc.
- Requires configuration

**Search Order:**
```
Local → Remote (settings.xml) → Remote (pom.xml) → Central
```

**Repository Managers:**
- **Nexus**: Most popular
- **Artifactory**: JFrog solution
- **Archiva**: Apache solution

**Benefits:**
- Cache dependencies
- Host internal artifacts
- Proxy external repositories
- Control dependencies

---

### Q16: How do you deploy artifacts to repository?
**Expected Answer:**

**Configure Distribution Management:**
```xml
<distributionManagement>
    <repository>
        <id>nexus-releases</id>
        <name>Nexus Release Repository</name>
        <url>http://nexus.example.com/repository/maven-releases/</url>
    </repository>
    <snapshotRepository>
        <id>nexus-snapshots</id>
        <name>Nexus Snapshot Repository</name>
        <url>http://nexus.example.com/repository/maven-snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```

**Add Credentials in settings.xml:**
```xml
<servers>
    <server>
        <id>nexus-releases</id>
        <username>deployment-user</username>
        <password>deployment-password</password>
    </server>
    <server>
        <id>nexus-snapshots</id>
        <username>deployment-user</username>
        <password>deployment-password</password>
    </server>
</servers>
```

**Deploy:**
```bash
# Deploy to remote repository
mvn clean deploy

# Deploy without rebuilding
mvn deploy:deploy-file \
  -DgroupId=com.example \
  -DartifactId=my-app \
  -Dversion=1.0 \
  -Dpackaging=jar \
  -Dfile=target/my-app-1.0.jar \
  -DrepositoryId=nexus-releases \
  -Durl=http://nexus.example.com/repository/maven-releases/
```

---

## Best Practices

### Q17: What are Maven best practices?
**Expected Answer:**

**POM Best Practices:**
- ✅ Use dependencyManagement in parent
- ✅ Define versions as properties
- ✅ Lock plugin versions
- ✅ Use BOM for complex dependencies
- ✅ Organize POM consistently
- ✅ Don't use SNAPSHOT in production

**Dependency Management:**
- ✅ Minimize dependencies
- ✅ Exclude unused transitive dependencies
- ✅ Regular dependency updates
- ✅ Analyze dependencies: `mvn dependency:analyze`
- ✅ Use correct scopes

**Build Performance:**
- ✅ Enable parallel builds: `mvn -T 1C`
- ✅ Use Maven daemon (mvnd)
- ✅ Skip tests when appropriate: `-DskipTests`
- ✅ Use offline mode: `-o`
- ✅ Configure build caching

**Version Control:**
- ✅ Commit pom.xml
- ✅ Ignore target/
- ✅ Ignore .m2/
- ✅ Use Maven Wrapper

**Project Structure:**
- ✅ Follow standard directory layout
- ✅ One artifact per module
- ✅ Keep modules focused
- ✅ Clear module dependencies

---

## Scenario-Based Questions

### Q18: How would you migrate from Ant/Gradle to Maven?
**Expected Answer:**

**Migration Strategy:**

**1. Analysis Phase:**
- Analyze current build
- Identify dependencies
- Document custom tasks
- Plan module structure

**2. Create Maven Structure:**
```bash
# Create standard layout
src/main/java
src/main/resources
src/test/java
src/test/resources
```

**3. Create Initial POM:**
- Add coordinates
- Define properties
- Add dependencies
- Configure plugins

**4. Migrate Dependencies:**
- Find Maven equivalents
- Add to POM
- Test compilation

**5. Migrate Build Tasks:**
- Map to Maven plugins
- Configure plugins
- Custom plugins if needed

**6. Testing:**
- Run unit tests
- Run integration tests
- Compare artifacts
- Verify functionality

**7. CI/CD Integration:**
- Update build scripts
- Configure Maven in CI
- Test deployments

**Challenges:**
- Custom build logic
- Non-Maven dependencies
- Team training
- Tool integration

---

### Q19: How do you optimize Maven builds in CI/CD?
**Expected Answer:**

**Optimization Strategies:**

**1. Caching:**
```yaml
# GitHub Actions
- uses: actions/cache@v3
  with:
    path: ~/.m2/repository
    key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
```

**2. Parallel Builds:**
```bash
mvn clean install -T 1C
```

**3. Selective Builds:**
```bash
# Only changed modules
mvn install -pl changed-module -am
```

**4. Skip Unnecessary Steps:**
```bash
# Skip tests in non-test builds
mvn install -DskipTests

# Skip documentation
mvn install -Dmaven.javadoc.skip=true
```

**5. Maven Daemon:**
```bash
# Use mvnd instead of mvn
mvnd clean install
```

**6. Incremental Builds:**
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
        <useIncrementalCompilation>true</useIncrementalCompilation>
    </configuration>
</plugin>
```

**7. Repository Manager:**
- Use Nexus/Artifactory
- Cache dependencies
- Reduce network calls

---

### Q20: How do you handle a dependency not in Maven Central?
**Expected Answer:**

**Solutions:**

**1. Install to Local Repository:**
```bash
mvn install:install-file \
  -Dfile=path/to/library.jar \
  -DgroupId=com.example \
  -DartifactId=library \
  -Dversion=1.0 \
  -Dpackaging=jar
```

**2. Deploy to Company Repository:**
```bash
mvn deploy:deploy-file \
  -Dfile=path/to/library.jar \
  -DgroupId=com.example \
  -DartifactId=library \
  -Dversion=1.0 \
  -Dpackaging=jar \
  -DrepositoryId=nexus \
  -Durl=http://nexus.company.com/repository/releases/
```

**3. System Scope (Not Recommended):**
```xml
<dependency>
    <groupId>com.example</groupId>
    <artifactId>library</artifactId>
    <version>1.0</version>
    <scope>system</scope>
    <systemPath>${project.basedir}/lib/library.jar</systemPath>
</dependency>
```

**4. Project Repository:**
```xml
<project>
    <repositories>
        <repository>
            <id>project-repo</id>
            <url>file://${project.basedir}/lib</url>
        </repository>
    </repositories>
</project>
```

**Best Practice**: Deploy to company repository manager.

---

## Behavioral Questions

### Q21: Describe experience with Maven in large projects
**Expected Answer Structure:**

**Situation:**
"In a microservices project with 50+ modules..."

**Task:**
"Needed to manage dependencies, ensure consistency, optimize build times..."

**Action:**
- Implemented parent POM with dependencyManagement
- Created BOMs for shared dependencies
- Established multi-module structure
- Configured CI/CD with caching
- Set up Nexus for artifact management

**Result:**
- Build time reduced from 30 to 10 minutes
- Zero dependency conflicts
- Consistent versions across services
- Easy onboarding for new developers

**Lessons Learned:**
- Importance of dependency management
- Value of repository manager
- Team agreement on conventions

---

## Summary

### Key Concepts to Remember

**Fundamentals:**
- POM structure
- GAV coordinates
- Build lifecycle
- Dependencies and scopes

**Advanced:**
- Multi-module projects
- Profiles
- Repository management
- Plugin configuration

**Best Practices:**
- Version management
- Dependency analysis
- Build optimization
- Clean structure

---

### Interview Tips

**Preparation:**
1. Practice basic commands
2. Understand POM structure
3. Know dependency management
4. Understand build lifecycle
5. Have project examples ready

**During Interview:**
1. Explain clearly with examples
2. Mention real projects
3. Discuss trade-offs
4. Show problem-solving
5. Ask clarifying questions

---

**Resources:**
- [Learning Roadmap](maven-learning-roadmap.md)
- [Quick Reference](maven-quick-reference.md)
- [Hands-On Exercises](maven-hands-on-exercises.md)
- [Troubleshooting Guide](maven-troubleshooting-guide.md)

**Good luck with your Maven interview! 🚀**


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

