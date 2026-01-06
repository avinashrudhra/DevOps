# Maven Hands-On Exercises

Complete practical exercises to master Maven from beginner to advanced level.

---

## Beginner Level Exercises

### Exercise 1: Create Your First Maven Project
**Objective**: Create and build a basic Maven project

1. Create a new Maven project using quickstart archetype
2. Examine the POM structure
3. Build the project
4. Run the generated application

<details>
<summary>Solution</summary>

```bash
# Create project
mvn archetype:generate \
  -DgroupId=com.example \
  -DartifactId=my-first-app \
  -DarchetypeArtifactId=maven-archetype-quickstart \
  -DarchetypeVersion=1.4 \
  -DinteractiveMode=false

# Navigate to project
cd my-first-app

# Examine POM
cat pom.xml

# Build project
mvn clean install

# Run application
java -cp target/my-first-app-1.0-SNAPSHOT.jar com.example.App
```
</details>

---

### Exercise 2: Add Dependencies
**Objective**: Add and manage project dependencies

1. Add Apache Commons Lang3 dependency
2. Add SLF4J logging dependency
3. View dependency tree
4. Use the dependencies in code

<details>
<summary>Solution</summary>

```xml
<!-- Add to pom.xml -->
<dependencies>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
        <version>3.12.0</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>2.0.7</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-simple</artifactId>
        <version>2.0.7</version>
    </dependency>
</dependencies>
```

```bash
# View dependency tree
mvn dependency:tree

# Rebuild
mvn clean install
```

```java
// Use in App.java
import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class App {
    private static final Logger logger = LoggerFactory.getLogger(App.class);
    
    public static void main(String[] args) {
        String text = "Hello Maven";
        logger.info("Original: {}", text);
        logger.info("Reversed: {}", StringUtils.reverse(text));
    }
}
```
</details>

---

### Exercise 3: Configure Compiler Plugin
**Objective**: Set Java version and compiler options

1. Configure Maven compiler plugin for Java 11
2. Enable compiler warnings
3. Set encoding to UTF-8
4. Build and verify

<details>
<summary>Solution</summary>

```xml
<!-- Add to pom.xml -->
<properties>
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>

<build>
    <plugins>
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
    </plugins>
</build>
```

```bash
# Build with warnings
mvn clean compile

# Verify effective POM
mvn help:effective-pom
```
</details>

---

### Exercise 4: Create Executable JAR
**Objective**: Package application as executable JAR

1. Configure manifest with main class
2. Create fat JAR with dependencies
3. Run the JAR file

<details>
<summary>Solution</summary>

```xml
<!-- Add Shade plugin to pom.xml -->
<build>
    <plugins>
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
                                <mainClass>com.example.App</mainClass>
                            </transformer>
                        </transformers>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

```bash
# Package
mvn clean package

# Run JAR
java -jar target/my-first-app-1.0-SNAPSHOT.jar
```
</details>

---

### Exercise 5: Add Unit Tests
**Objective**: Write and run unit tests with JUnit 5

1. Add JUnit 5 dependency
2. Write unit tests
3. Run tests
4. Generate test report

<details>
<summary>Solution</summary>

```xml
<!-- Add to pom.xml -->
<dependencies>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.9.3</version>
        <scope>test</scope>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.1.2</version>
        </plugin>
    </plugins>
</build>
```

```java
// src/test/java/com/example/AppTest.java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

public class AppTest {
    @Test
    public void testApp() {
        assertTrue(true);
        assertEquals(4, 2 + 2);
    }
}
```

```bash
# Run tests
mvn test

# Run tests with detailed output
mvn test -X

# Skip tests
mvn install -DskipTests
```
</details>

---

### Exercise 6: Use Properties
**Objective**: Define and use custom properties

1. Define version properties
2. Use properties in dependencies
3. Override properties from command line

<details>
<summary>Solution</summary>

```xml
<!-- Add to pom.xml -->
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
    
    <!-- Dependency versions -->
    <junit.version>5.9.3</junit.version>
    <commons.lang.version>3.12.0</commons.lang.version>
    <slf4j.version>2.0.7</slf4j.version>
    
    <!-- Custom properties -->
    <app.name>My Application</app.name>
    <app.version>1.0</app.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>${junit.version}</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
        <version>${commons.lang.version}</version>
    </dependency>
</dependencies>
```

```bash
# Override property from command line
mvn clean install -Dapp.version=2.0
```
</details>

---

### Exercise 7: Resource Filtering
**Objective**: Filter resources with Maven properties

1. Enable resource filtering
2. Create properties file with placeholders
3. Build and verify filtered resources

<details>
<summary>Solution</summary>

```xml
<!-- Add to pom.xml -->
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
        </resource>
    </resources>
</build>

<properties>
    <app.name>My Application</app.name>
    <app.version>1.0-SNAPSHOT</app.version>
    <environment>development</environment>
</properties>
```

```properties
# src/main/resources/application.properties
app.name=${app.name}
app.version=${project.version}
environment=${environment}
build.timestamp=${maven.build.timestamp}
```

```bash
# Build
mvn clean package

# View filtered resource
cat target/classes/application.properties
```
</details>

---

### Exercise 8: Maven Profiles
**Objective**: Create and use build profiles

1. Create dev and prod profiles
2. Configure different properties per profile
3. Activate profiles

<details>
<summary>Solution</summary>

```xml
<!-- Add to pom.xml -->
<profiles>
    <profile>
        <id>dev</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <environment>development</environment>
            <db.url>jdbc:h2:mem:devdb</db.url>
        </properties>
    </profile>
    
    <profile>
        <id>prod</id>
        <properties>
            <environment>production</environment>
            <db.url>jdbc:mysql://prod-server:3306/proddb</db.url>
        </properties>
        <build>
            <plugins>
                <!-- Production-specific plugins -->
            </plugins>
        </build>
    </profile>
</profiles>
```

```bash
# Build with dev profile (default)
mvn clean install

# Build with prod profile
mvn clean install -Pprod

# Check active profiles
mvn help:active-profiles

# Multiple profiles
mvn clean install -Pprofile1,profile2
```
</details>

---

### Exercise 9: Dependency Analysis
**Objective**: Analyze and optimize dependencies

1. View dependency tree
2. Find unused dependencies
3. Detect dependency conflicts
4. Exclude transitive dependencies

<details>
<summary>Solution</summary>

```bash
# View dependency tree
mvn dependency:tree

# Verbose tree
mvn dependency:tree -Dverbose

# Analyze dependencies
mvn dependency:analyze

# Find duplicates
mvn dependency:analyze-duplicate

# Display updates
mvn versions:display-dependency-updates
```

```xml
<!-- Exclude transitive dependency -->
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
</details>

---

### Exercise 10: Generate Project Documentation
**Objective**: Generate Maven site with documentation

1. Configure site plugin
2. Add reporting plugins
3. Generate site
4. View documentation

<details>
<summary>Solution</summary>

```xml
<!-- Add to pom.xml -->
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-site-plugin</artifactId>
            <version>3.12.1</version>
        </plugin>
    </plugins>
</build>

<reporting>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-project-info-reports-plugin</artifactId>
            <version>3.4.5</version>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-javadoc-plugin</artifactId>
            <version>3.5.0</version>
        </plugin>
    </plugins>
</reporting>
```

```bash
# Generate site
mvn site

# View site
open target/site/index.html
```
</details>

---

## Intermediate Level Exercises

### Exercise 11: Multi-Module Project
**Objective**: Create multi-module Maven project

1. Create parent POM
2. Create multiple child modules
3. Define inter-module dependencies
4. Build all modules

<details>
<summary>Solution</summary>

```bash
# Create parent project
mkdir parent-project
cd parent-project

# Create parent POM manually
```

```xml
<!-- parent-project/pom.xml -->
<project>
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.example</groupId>
    <artifactId>parent-project</artifactId>
    <version>1.0</version>
    <packaging>pom</packaging>
    
    <modules>
        <module>core</module>
        <module>service</module>
        <module>web</module>
    </modules>
    
    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>
    
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.junit.jupiter</groupId>
                <artifactId>junit-jupiter</artifactId>
                <version>5.9.3</version>
                <scope>test</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

```bash
# Create modules
mvn archetype:generate -DgroupId=com.example -DartifactId=core -DinteractiveMode=false
mvn archetype:generate -DgroupId=com.example -DartifactId=service -DinteractiveMode=false
mvn archetype:generate -DgroupId=com.example -DartifactId=web -DarchetypeArtifactId=maven-archetype-webapp -DinteractiveMode=false
```

```xml
<!-- service/pom.xml - depends on core -->
<project>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>parent-project</artifactId>
        <version>1.0</version>
    </parent>
    
    <artifactId>service</artifactId>
    
    <dependencies>
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>core</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
</project>
```

```bash
# Build all modules
cd parent-project
mvn clean install

# Build specific module with dependencies
mvn clean install -pl service -am

# Build reactor
mvn clean install
```
</details>

---

### Exercise 12: Spring Boot Application
**Objective**: Create Spring Boot application with Maven

1. Create Spring Boot project
2. Add Spring Boot dependencies
3. Create REST controller
4. Run application

<details>
<summary>Solution</summary>

```xml
<!-- pom.xml -->
<project>
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.12</version>
    </parent>
    
    <groupId>com.example</groupId>
    <artifactId>spring-boot-app</artifactId>
    <version>1.0</version>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
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
</project>
```

```java
// src/main/java/com/example/Application.java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

// src/main/java/com/example/HelloController.java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "Hello from Maven Spring Boot!";
    }
}
```

```bash
# Build and run
mvn clean package
mvn spring-boot:run

# Or run JAR
java -jar target/spring-boot-app-1.0.jar

# Test endpoint
curl http://localhost:8080/hello
```
</details>

---

### Exercise 13: Custom Assembly
**Objective**: Create custom distribution package

1. Configure assembly plugin
2. Create custom assembly descriptor
3. Generate distribution
4. Extract and verify

<details>
<summary>Solution</summary>

```xml
<!-- pom.xml -->
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-assembly-plugin</artifactId>
            <version>3.6.0</version>
            <configuration>
                <descriptors>
                    <descriptor>src/assembly/distribution.xml</descriptor>
                </descriptors>
            </configuration>
            <executions>
                <execution>
                    <id>make-assembly</id>
                    <phase>package</phase>
                    <goals>
                        <goal>single</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

```xml
<!-- src/assembly/distribution.xml -->
<assembly>
    <id>dist</id>
    <formats>
        <format>zip</format>
        <format>tar.gz</format>
    </formats>
    <fileSets>
        <fileSet>
            <directory>${project.basedir}</directory>
            <outputDirectory>/</outputDirectory>
            <includes>
                <include>README*</include>
                <include>LICENSE*</include>
            </includes>
        </fileSet>
        <fileSet>
            <directory>${project.build.directory}</directory>
            <outputDirectory>/lib</outputDirectory>
            <includes>
                <include>*.jar</include>
            </includes>
        </fileSet>
        <fileSet>
            <directory>src/main/scripts</directory>
            <outputDirectory>/bin</outputDirectory>
            <fileMode>0755</fileMode>
        </fileSet>
    </fileSets>
</assembly>
```

```bash
# Create distribution
mvn clean package

# Extract and verify
unzip target/my-app-1.0-dist.zip
ls -la my-app-1.0/
```
</details>

---

### Exercise 14: Integration Tests
**Objective**: Configure and run integration tests

1. Add Failsafe plugin
2. Write integration tests
3. Run integration tests separately
4. Generate test reports

<details>
<summary>Solution</summary>

```xml
<!-- pom.xml -->
<build>
    <plugins>
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
            <configuration>
                <includes>
                    <include>**/*IT.java</include>
                    <include>**/*IntegrationTest.java</include>
                </includes>
            </configuration>
        </plugin>
    </plugins>
</build>
```

```java
// src/test/java/com/example/AppIT.java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

public class AppIT {
    @Test
    public void integrationTest() {
        // Integration test logic
        assertTrue(true);
    }
}
```

```bash
# Run unit tests only
mvn test

# Run integration tests only
mvn verify

# Run all tests
mvn clean verify

# Skip integration tests
mvn install -DskipITs
```
</details>

---

### Exercise 15: Deploy to Nexus
**Objective**: Deploy artifacts to repository manager

1. Configure distribution management
2. Add server credentials
3. Deploy snapshot
4. Deploy release

<details>
<summary>Solution</summary>

```xml
<!-- pom.xml -->
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

```xml
<!-- ~/.m2/settings.xml -->
<settings>
    <servers>
        <server>
            <id>nexus-releases</id>
            <username>deployment-user</username>
            <password>{encrypted-password}</password>
        </server>
        <server>
            <id>nexus-snapshots</id>
            <username>deployment-user</username>
            <password>{encrypted-password}</password>
        </server>
    </servers>
</settings>
```

```bash
# Encrypt password
mvn --encrypt-master-password <password>
mvn --encrypt-password <password>

# Deploy snapshot
mvn clean deploy

# Prepare release
mvn versions:set -DnewVersion=1.0

# Deploy release
mvn clean deploy
```
</details>

---

## Advanced Level Exercises

### Exercise 16: Custom Maven Plugin
**Objective**: Develop custom Maven plugin

1. Create plugin project
2. Implement Mojo
3. Build and install plugin
4. Use plugin in project

<details>
<summary>Solution</summary>

```bash
# Create plugin project
mvn archetype:generate \
  -DgroupId=com.example.plugins \
  -DartifactId=custom-maven-plugin \
  -DarchetypeArtifactId=maven-archetype-plugin
```

```java
// src/main/java/com/example/plugins/GreetMojo.java
import org.apache.maven.plugin.AbstractMojo;
import org.apache.maven.plugin.MojoExecutionException;
import org.apache.maven.plugins.annotations.Mojo;
import org.apache.maven.plugins.annotations.Parameter;

@Mojo(name = "greet")
public class GreetMojo extends AbstractMojo {
    
    @Parameter(property = "greet.name", defaultValue = "World")
    private String name;
    
    @Parameter(property = "greet.message", defaultValue = "Hello")
    private String message;
    
    public void execute() throws MojoExecutionException {
        getLog().info(message + ", " + name + "!");
    }
}
```

```bash
# Build plugin
mvn clean install

# Use in project
```

```xml
<!-- Use plugin in project pom.xml -->
<build>
    <plugins>
        <plugin>
            <groupId>com.example.plugins</groupId>
            <artifactId>custom-maven-plugin</artifactId>
            <version>1.0-SNAPSHOT</version>
            <executions>
                <execution>
                    <phase>validate</phase>
                    <goals>
                        <goal>greet</goal>
                    </goals>
                    <configuration>
                        <name>Maven</name>
                        <message>Greetings</message>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

```bash
# Run plugin
mvn com.example.plugins:custom-maven-plugin:1.0-SNAPSHOT:greet -Dgreet.name=Developer
```
</details>

---

### Exercise 17: Maven Wrapper
**Objective**: Add Maven Wrapper to project

1. Generate wrapper files
2. Commit wrapper to version control
3. Use wrapper for builds
4. Configure wrapper version

<details>
<summary>Solution</summary>

```bash
# Generate wrapper
mvn wrapper:wrapper

# Or specify Maven version
mvn wrapper:wrapper -Dmaven=3.9.6

# Generated files:
# mvnw (Unix)
# mvnw.cmd (Windows)
# .mvn/wrapper/maven-wrapper.properties
# .mvn/wrapper/maven-wrapper.jar
```

```bash
# Use wrapper (Unix/Mac)
./mvnw clean install

# Use wrapper (Windows)
mvnw.cmd clean install

# Commit to Git
git add mvnw mvnw.cmd .mvn/
git commit -m "Add Maven Wrapper"
```

```properties
# .mvn/wrapper/maven-wrapper.properties
distributionUrl=https://repo.maven.apache.org/maven2/org/apache/maven/apache-maven/3.9.6/apache-maven-3.9.6-bin.zip
wrapperUrl=https://repo.maven.apache.org/maven2/org/apache/maven/wrapper/maven-wrapper/3.2.0/maven-wrapper-3.2.0.jar
```
</details>

---

### Exercise 18: BOM (Bill of Materials)
**Objective**: Create and use BOM for dependency management

1. Create BOM POM
2. Define dependency versions
3. Import BOM in projects
4. Use dependencies without versions

<details>
<summary>Solution</summary>

```xml
<!-- bom/pom.xml -->
<project>
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.example</groupId>
    <artifactId>bom</artifactId>
    <version>1.0</version>
    <packaging>pom</packaging>
    
    <dependencyManagement>
        <dependencies>
            <!-- Define all versions here -->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-core</artifactId>
                <version>5.3.27</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-web</artifactId>
                <version>5.3.27</version>
            </dependency>
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>4.13.2</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

```xml
<!-- project/pom.xml - Use BOM -->
<project>
    <dependencyManagement>
        <dependencies>
            <!-- Import BOM -->
            <dependency>
                <groupId>com.example</groupId>
                <artifactId>bom</artifactId>
                <version>1.0</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    
    <dependencies>
        <!-- Use without version -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
        </dependency>
    </dependencies>
</project>
```

```bash
# Build BOM
cd bom
mvn clean install

# Use in projects
cd ../project
mvn clean install
```
</details>

---

### Exercise 19: CI/CD with Maven
**Objective**: Set up CI/CD pipeline for Maven project

1. Create GitHub Actions workflow
2. Configure caching
3. Run tests and build
4. Deploy artifacts

<details>
<summary>Solution</summary>

```yaml
# .github/workflows/maven.yml
name: Maven CI/CD

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up JDK 11
      uses: actions/setup-java@v3
      with:
        java-version: '11'
        distribution: 'temurin'
        cache: maven
    
    - name: Build with Maven
      run: mvn clean verify
    
    - name: Upload coverage
      uses: codecov/codecov-action@v3
      with:
        files: ./target/site/jacoco/jacoco.xml
    
    - name: Upload artifacts
      uses: actions/upload-artifact@v3
      with:
        name: package
        path: target/*.jar
    
    - name: Deploy to Nexus
      if: github.ref == 'refs/heads/main'
      run: mvn deploy -s .maven-settings.xml
      env:
        NEXUS_USERNAME: ${{ secrets.NEXUS_USERNAME }}
        NEXUS_PASSWORD: ${{ secrets.NEXUS_PASSWORD }}
```
</details>

---

### Exercise 20: Performance Optimization
**Objective**: Optimize Maven build performance

1. Enable parallel builds
2. Configure Maven daemon
3. Use offline mode
4. Optimize dependencies

<details>
<summary>Solution</summary>

```bash
# Parallel builds
mvn clean install -T 4      # 4 threads
mvn clean install -T 1C     # 1 thread per CPU core

# Offline mode (skip remote repository checks)
mvn clean install -o

# Skip tests for faster builds
mvn clean install -DskipTests

# Maven daemon (install first)
brew install mvndaemon/homebrew-mvnd/mvnd
mvnd clean install  # Much faster!
```

```xml
<!-- .mvn/maven.config -->
-T 1C
-Dmaven.artifact.threads=10
-Daether.connector.http.connectionMaxTtl=30
```

```xml
<!-- pom.xml - Optimize dependencies -->
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-dependency-plugin</artifactId>
            <version>3.6.0</version>
            <configuration>
                <ignoreNonCompile>true</ignoreNonCompile>
            </configuration>
        </plugin>
    </plugins>
</build>
```

```bash
# Measure build time
time mvn clean install

# Analyze what takes time
mvn clean install -Dorg.slf4j.simpleLogger.showDateTime=true

# Clean local repository of unused dependencies
mvn dependency:purge-local-repository
```
</details>

---

## Tips for Practice

1. **Start Simple**: Begin with basic projects
2. **Read Error Messages**: Maven provides detailed errors
3. **Use Help Plugin**: `mvn help:describe -Dplugin=<plugin>`
4. **Check Effective POM**: `mvn help:effective-pom`
5. **Analyze Dependencies**: Regular dependency analysis
6. **Keep POMs Clean**: Organize and comment
7. **Use Profiles**: Different configurations
8. **Version Management**: Central version control
9. **Test Regularly**: Run tests frequently
10. **Document**: Comment complex configurations

---

## Next Steps

1. Complete all exercises in order
2. Practice with real projects
3. Study Maven best practices
4. Learn plugin development
5. Explore enterprise patterns
6. Check [Learning Roadmap](maven-learning-roadmap.md)
7. Review [Troubleshooting Guide](maven-troubleshooting-guide.md)

**Good luck with your Maven practice! 🚀**


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

