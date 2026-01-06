# Maven: Beginner to Expert
## Complete Learning Roadmap

> Master Apache Maven build automation from basics to advanced enterprise patterns

---

## Table of Contents
1. [Introduction](#introduction)
2. [Level 1: Beginner (Weeks 1-2)](#level-1-beginner-weeks-1-2)
3. [Level 2: Intermediate (Weeks 3-4)](#level-2-intermediate-weeks-3-4)
4. [Level 3: Advanced (Weeks 5-8)](#level-3-advanced-weeks-5-8)
5. [Level 4: Expert (Weeks 9-12)](#level-4-expert-weeks-9-12)
6. [Best Practices](#best-practices)
7. [Resources](#resources)

---

## Introduction

### What is Maven?
**Apache Maven** is a build automation and project management tool primarily used for Java projects.

### Key Benefits:
- **Convention over Configuration**: Standard project structure
- **Dependency Management**: Automatic downloads and transitive dependencies
- **Build Lifecycle**: Standardized build phases
- **Plugin Ecosystem**: Extensive plugin library
- **Project Documentation**: Automated site generation
- **Multi-Module Support**: Complex project structures

### Prerequisites:
- Java JDK installed (8+)
- Basic Java knowledge
- Command line familiarity
- IDE with Maven support

---

## Level 1: Beginner (Weeks 1-2)

### Week 1: Maven Fundamentals

#### 1.1 Installation & Setup

**Install Maven:**
```bash
# Verify Java installation
java -version

# Windows (Chocolatey)
choco install maven

# Mac
brew install maven

# Linux (Ubuntu/Debian)
sudo apt-get install maven

# Verify installation
mvn --version
```

**Configure Maven:**
```bash
# Maven home
echo $MAVEN_HOME
# or on Windows: echo %MAVEN_HOME%

# Local repository location (default)
~/.m2/repository  # Linux/Mac
%USERPROFILE%\.m2\repository  # Windows

# Settings file
~/.m2/settings.xml
```

**Create settings.xml:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
          http://maven.apache.org/xsd/settings-1.0.0.xsd">
    
    <!-- Local repository path -->
    <localRepository>${user.home}/.m2/repository</localRepository>
    
    <!-- Offline mode -->
    <offline>false</offline>
    
    <!-- Mirrors -->
    <mirrors>
        <mirror>
            <id>central-mirror</id>
            <mirrorOf>central</mirrorOf>
            <url>https://repo.maven.apache.org/maven2</url>
        </mirror>
    </mirrors>
</settings>
```

**✅ Week 1 Day 1 Checkpoint**: Maven installed and configured

---

#### 1.2 Your First Maven Project

**Create Project from Archetype:**
```bash
mvn archetype:generate \
  -DgroupId=com.example \
  -DartifactId=my-app \
  -DarchetypeArtifactId=maven-archetype-quickstart \
  -DarchetypeVersion=1.4 \
  -DinteractiveMode=false

cd my-app
```

**Project Structure:**
```
my-app/
├── pom.xml
└── src/
    ├── main/
    │   └── java/
    │       └── com/example/
    │           └── App.java
    └── test/
        └── java/
            └── com/example/
                └── AppTest.java
```

**Basic POM (pom.xml):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- Project Coordinates -->
    <groupId>com.example</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <!-- Project Information -->
    <name>my-app</name>
    <description>My First Maven Application</description>

    <!-- Properties -->
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>

    <!-- Dependencies -->
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

**Build Your Project:**
```bash
# Compile
mvn compile

# Run tests
mvn test

# Package (creates JAR)
mvn package

# Install to local repository
mvn install

# Clean build artifacts
mvn clean

# Clean and install
mvn clean install
```

---

#### 1.3 Understanding POM (Project Object Model)

**POM Coordinates (GAV):**
```xml
<groupId>com.example</groupId>        <!-- Organization/package -->
<artifactId>my-app</artifactId>       <!-- Project name -->
<version>1.0-SNAPSHOT</version>       <!-- Version -->
```

**Packaging Types:**
```xml
<packaging>jar</packaging>    <!-- Java library (default) -->
<packaging>war</packaging>    <!-- Web application -->
<packaging>ear</packaging>    <!-- Enterprise application -->
<packaging>pom</packaging>    <!-- Parent/aggregator -->
<packaging>maven-plugin</packaging>  <!-- Maven plugin -->
```

**Properties:**
```xml
<properties>
    <!-- Java version -->
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
    
    <!-- Encoding -->
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    
    <!-- Dependency versions -->
    <junit.version>5.9.3</junit.version>
    <spring.version>5.3.27</spring.version>
    
    <!-- Custom properties -->
    <my.custom.property>value</my.custom.property>
</properties>
```

**Using Properties:**
```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>${junit.version}</version>
    <scope>test</scope>
</dependency>
```

---

#### 1.4 Dependencies

**Adding Dependencies:**
```xml
<dependencies>
    <!-- Spring Framework -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>5.3.27</version>
    </dependency>
    
    <!-- Apache Commons -->
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
        <version>3.12.0</version>
    </dependency>
    
    <!-- Logging -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>2.0.7</version>
    </dependency>
    
    <!-- Test dependency -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.9.3</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

**Dependency Scopes:**
```xml
<!-- compile (default) - available in all classpaths -->
<scope>compile</scope>

<!-- provided - provided by JDK or container -->
<scope>provided</scope>

<!-- runtime - not needed for compilation -->
<scope>runtime</scope>

<!-- test - only for testing -->
<scope>test</scope>

<!-- system - must specify path (avoid!) -->
<scope>system</scope>

<!-- import - for dependency management -->
<scope>import</scope>
```

**Find Dependencies:**
```bash
# Maven Central: https://search.maven.org/
# MVN Repository: https://mvnrepository.com/
```

---

#### 1.5 Build Lifecycle

**Three Built-in Lifecycles:**

**1. Default Lifecycle (main build)**
```bash
mvn validate    # Validate project structure
mvn compile     # Compile source code
mvn test        # Run unit tests
mvn package     # Create JAR/WAR
mvn verify      # Run integration tests
mvn install     # Install to local repository
mvn deploy      # Deploy to remote repository
```

**2. Clean Lifecycle (cleanup)**
```bash
mvn clean       # Remove target directory
```

**3. Site Lifecycle (documentation)**
```bash
mvn site        # Generate project documentation
```

**Common Commands:**
```bash
# Clean and compile
mvn clean compile

# Clean, test, and package
mvn clean test package

# Clean and install
mvn clean install

# Clean, install, skip tests
mvn clean install -DskipTests

# Clean, install, skip test compilation
mvn clean install -Dmaven.test.skip=true
```

**Lifecycle Phases:**
```
validate → compile → test → package → verify → install → deploy
```

**✅ Week 1 Checkpoint**: Understand POM, dependencies, and lifecycle

---

### Week 2: Essential Maven Features

#### 2.1 Maven Repository

**Three Types:**

**1. Local Repository**
```bash
# Location: ~/.m2/repository/
# Structure: groupId/artifactId/version/

# Example:
~/.m2/repository/org/springframework/spring-core/5.3.27/spring-core-5.3.27.jar
```

**2. Central Repository**
```xml
<!-- Default - Maven Central -->
<!-- https://repo.maven.apache.org/maven2 -->
<!-- No configuration needed -->
```

**3. Remote Repository**
```xml
<repositories>
    <repository>
        <id>spring-milestones</id>
        <name>Spring Milestones</name>
        <url>https://repo.spring.io/milestone</url>
    </repository>
</repositories>
```

**Plugin Repositories:**
```xml
<pluginRepositories>
    <pluginRepository>
        <id>spring-plugins</id>
        <url>https://repo.spring.io/plugins-release</url>
    </pluginRepository>
</pluginRepositories>
```

---

#### 2.2 Basic Plugins

**Compiler Plugin:**
```xml
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
            </configuration>
        </plugin>
    </plugins>
</build>
```

**Surefire Plugin (Tests):**
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
    </configuration>
</plugin>
```

**JAR Plugin:**
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>3.3.0</version>
    <configuration>
        <archive>
            <manifest>
                <mainClass>com.example.App</mainClass>
                <addClasspath>true</addClasspath>
            </manifest>
        </archive>
    </configuration>
</plugin>
```

---

#### 2.3 Running Your Application

**Exec Maven Plugin:**
```xml
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>exec-maven-plugin</artifactId>
    <version>3.1.0</version>
    <configuration>
        <mainClass>com.example.App</mainClass>
    </configuration>
</plugin>
```

**Run:**
```bash
mvn exec:java
```

**Shade Plugin (Fat JAR):**
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
                        <mainClass>com.example.App</mainClass>
                    </transformer>
                </transformers>
            </configuration>
        </execution>
    </executions>
</plugin>
```

**Run Fat JAR:**
```bash
mvn clean package
java -jar target/my-app-1.0-SNAPSHOT.jar
```

---

#### 2.4 Dependency Management

**View Dependency Tree:**
```bash
mvn dependency:tree

# Output:
com.example:my-app:jar:1.0-SNAPSHOT
+- org.springframework:spring-core:jar:5.3.27:compile
|  +- org.springframework:spring-jcl:jar:5.3.27:compile
+- junit:junit:jar:4.13.2:test
   +- org.hamcrest:hamcrest-core:jar:1.3:test
```

**Analyze Dependencies:**
```bash
# Find used and unused dependencies
mvn dependency:analyze

# Copy dependencies to target/dependency
mvn dependency:copy-dependencies

# Display updates
mvn versions:display-dependency-updates
```

**Exclude Transitive Dependencies:**
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

**✅ Week 2 Checkpoint**: Master repositories, plugins, and dependency management

---

## Level 2: Intermediate (Weeks 3-4)

### Week 3: Advanced Build Configuration

#### 3.1 Profiles

**Purpose**: Different configurations for different environments

**Basic Profile:**
```xml
<profiles>
    <!-- Development Profile -->
    <profile>
        <id>dev</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <env>development</env>
            <db.url>jdbc:mysql://localhost:3306/devdb</db.url>
        </properties>
    </profile>
    
    <!-- Production Profile -->
    <profile>
        <id>prod</id>
        <properties>
            <env>production</env>
            <db.url>jdbc:mysql://prod-server:3306/proddb</db.url>
        </properties>
    </profile>
</profiles>
```

**Activate Profile:**
```bash
# Command line
mvn clean install -Pprod

# Multiple profiles
mvn clean install -Pdev,integration-test

# Check active profiles
mvn help:active-profiles
```

**Profile Activation:**
```xml
<profile>
    <id>windows</id>
    <activation>
        <!-- By OS -->
        <os>
            <family>windows</family>
        </os>
    </activation>
</profile>

<profile>
    <id>jdk11</id>
    <activation>
        <!-- By JDK version -->
        <jdk>11</jdk>
    </activation>
</profile>

<profile>
    <id>custom</id>
    <activation>
        <!-- By property -->
        <property>
            <name>environment</name>
            <value>test</value>
        </property>
    </activation>
</profile>
```

---

#### 3.2 Resource Filtering

**Enable Filtering:**
```xml
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
        </resource>
    </resources>
</build>
```

**application.properties:**
```properties
app.name=${project.name}
app.version=${project.version}
environment=${env}
database.url=${db.url}
```

**Result after filtering:**
```properties
app.name=my-app
app.version=1.0-SNAPSHOT
environment=development
database.url=jdbc:mysql://localhost:3306/devdb
```

---

#### 3.3 Dependency Management Section

**Parent POM Pattern:**
```xml
<project>
    <groupId>com.example</groupId>
    <artifactId>parent</artifactId>
    <version>1.0</version>
    <packaging>pom</packaging>
    
    <dependencyManagement>
        <dependencies>
            <!-- Define versions here -->
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
    
    <artifactId>child-module</artifactId>
    
    <dependencies>
        <!-- No version needed - inherited from parent -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
        </dependency>
    </dependencies>
</project>
```

---

#### 3.4 Plugin Management

**Plugin Management Section:**
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
    
    <plugins>
        <!-- Just reference, configuration inherited -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

---

### Week 4: Multi-Module Projects

#### 4.1 Creating Multi-Module Project

**Project Structure:**
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
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.example</groupId>
    <artifactId>parent-project</artifactId>
    <version>1.0</version>
    <packaging>pom</packaging>
    
    <modules>
        <module>module-a</module>
        <module>module-b</module>
        <module>module-c</module>
    </modules>
    
    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>
    
    <dependencyManagement>
        <dependencies>
            <!-- Shared dependency versions -->
        </dependencies>
    </dependencyManagement>
</project>
```

**Module POM (module-a):**
```xml
<project>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>parent-project</artifactId>
        <version>1.0</version>
    </parent>
    
    <artifactId>module-a</artifactId>
    
    <dependencies>
        <!-- Module dependencies -->
    </dependencies>
</project>
```

**Build All Modules:**
```bash
cd parent-project
mvn clean install

# Build specific module
mvn clean install -pl module-a

# Build with dependencies
mvn clean install -pl module-a -am

# Build modules that depend on it
mvn clean install -pl module-a -amd
```

---

#### 4.2 Inter-Module Dependencies

**Module A depends on Module B:**
```xml
<!-- module-a/pom.xml -->
<dependencies>
    <dependency>
        <groupId>com.example</groupId>
        <artifactId>module-b</artifactId>
        <version>${project.version}</version>
    </dependency>
</dependencies>
```

**Build Order:**
```bash
# Maven automatically determines build order based on dependencies
mvn clean install

# Reactor build order shown:
# [INFO] Reactor Build Order:
# [INFO] module-b
# [INFO] module-a
# [INFO] module-c
# [INFO] parent-project
```

---

## Level 3: Advanced (Weeks 5-8)

### Week 5-6: Advanced Plugin Configuration

#### 5.1 Maven Assembly Plugin

**Create Distribution:**
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
                <mainClass>com.example.App</mainClass>
            </manifest>
        </archive>
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
```

**Custom Assembly Descriptor:**
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
    </fileSets>
</assembly>
```

---

#### 5.2 Maven Release Plugin

**Configure Release Plugin:**
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-release-plugin</artifactId>
    <version>3.0.1</version>
    <configuration>
        <tagNameFormat>v@{project.version}</tagNameFormat>
        <autoVersionSubmodules>true</autoVersionSubmodules>
    </configuration>
</plugin>
```

**Release Process:**
```bash
# Prepare release
mvn release:prepare
# - Checks for uncommitted changes
# - Updates version (removes SNAPSHOT)
# - Creates Git tag
# - Updates to next SNAPSHOT version

# Perform release
mvn release:perform
# - Checks out tag
# - Builds and deploys artifacts

# Rollback if needed
mvn release:rollback
```

---

#### 5.3 Maven Site Plugin

**Generate Documentation:**
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-site-plugin</artifactId>
    <version>3.12.1</version>
</plugin>
```

**Reporting:**
```xml
<reporting>
    <plugins>
        <!-- Javadoc -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-javadoc-plugin</artifactId>
            <version>3.5.0</version>
        </plugin>
        
        <!-- Code coverage -->
        <plugin>
            <groupId>org.jacoco</groupId>
            <artifactId>jacoco-maven-plugin</artifactId>
            <version>0.8.10</version>
        </plugin>
        
        <!-- Checkstyle -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-checkstyle-plugin</artifactId>
            <version>3.3.0</version>
        </plugin>
    </plugins>
</reporting>
```

**Generate Site:**
```bash
mvn site
# View: target/site/index.html
```

---

### Week 7-8: Repository Management & CI/CD

#### 7.1 Deploying to Nexus/Artifactory

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

**Settings.xml (credentials):**
```xml
<servers>
    <server>
        <id>nexus-releases</id>
        <username>deployment-user</username>
        <password>encrypted-password</password>
    </server>
    <server>
        <id>nexus-snapshots</id>
        <username>deployment-user</username>
        <password>encrypted-password</password>
    </server>
</servers>
```

**Deploy:**
```bash
mvn clean deploy
```

---

#### 7.2 CI/CD Integration

**GitHub Actions:**
```yaml
# .github/workflows/maven.yml
name: Maven Build

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
    
    - name: Cache Maven packages
      uses: actions/cache@v3
      with:
        path: ~/.m2
        key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
        restore-keys: ${{ runner.os }}-m2
    
    - name: Build with Maven
      run: mvn clean verify
    
    - name: Upload artifacts
      uses: actions/upload-artifact@v3
      with:
        name: package
        path: target/*.jar
```

**Jenkins Pipeline:**
```groovy
pipeline {
    agent any
    
    tools {
        maven 'Maven 3.9.6'
        jdk 'JDK 11'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/user/project.git'
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('Package') {
            steps {
                sh 'mvn package'
            }
        }
        
        stage('Deploy') {
            steps {
                sh 'mvn deploy'
            }
        }
    }
    
    post {
        always {
            junit '**/target/surefire-reports/*.xml'
            archiveArtifacts artifacts: '**/target/*.jar'
        }
    }
}
```

---

## Level 4: Expert (Weeks 9-12)

### Week 9-10: Custom Plugin Development

#### 9.1 Create Maven Plugin

**Plugin Project Structure:**
```bash
mvn archetype:generate \
  -DgroupId=com.example.plugins \
  -DartifactId=my-maven-plugin \
  -DarchetypeArtifactId=maven-archetype-plugin
```

**Plugin POM:**
```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.example.plugins</groupId>
    <artifactId>my-maven-plugin</artifactId>
    <version>1.0</version>
    <packaging>maven-plugin</packaging>
    
    <dependencies>
        <dependency>
            <groupId>org.apache.maven</groupId>
            <artifactId>maven-plugin-api</artifactId>
            <version>3.9.6</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.maven.plugin-tools</groupId>
            <artifactId>maven-plugin-annotations</artifactId>
            <version>3.10.2</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
</project>
```

**Plugin Mojo:**
```java
package com.example.plugins;

import org.apache.maven.plugin.AbstractMojo;
import org.apache.maven.plugin.MojoExecutionException;
import org.apache.maven.plugins.annotations.Mojo;
import org.apache.maven.plugins.annotations.Parameter;

@Mojo(name = "greet")
public class GreetMojo extends AbstractMojo {
    
    @Parameter(property = "greet.name", defaultValue = "World")
    private String name;
    
    public void execute() throws MojoExecutionException {
        getLog().info("Hello, " + name + "!");
    }
}
```

**Use Plugin:**
```xml
<build>
    <plugins>
        <plugin>
            <groupId>com.example.plugins</groupId>
            <artifactId>my-maven-plugin</artifactId>
            <version>1.0</version>
            <executions>
                <execution>
                    <phase>validate</phase>
                    <goals>
                        <goal>greet</goal>
                    </goals>
                </execution>
            </executions>
            <configuration>
                <name>Maven</name>
            </configuration>
        </plugin>
    </plugins>
</build>
```

---

### Week 11-12: Performance & Best Practices

#### 11.1 Build Optimization

**Parallel Builds:**
```bash
# Build modules in parallel
mvn clean install -T 4  # 4 threads
mvn clean install -T 1C  # 1 thread per CPU core
```

**Offline Mode:**
```bash
# Don't check remote repositories
mvn clean install -o
```

**Skip Tests:**
```bash
# Skip test execution
mvn clean install -DskipTests

# Skip test compilation and execution
mvn clean install -Dmaven.test.skip=true
```

**Maven Daemon (mvnd):**
```bash
# Install mvnd (faster builds)
brew install mvndaemon/homebrew-mvnd/mvnd

# Use mvnd instead of mvn
mvnd clean install
```

---

#### 11.2 Best Practices

**POM Organization:**
```xml
<project>
    <!-- 1. Model version -->
    <modelVersion>4.0.0</modelVersion>
    
    <!-- 2. Parent -->
    <parent>...</parent>
    
    <!-- 3. Coordinates -->
    <groupId>...</groupId>
    <artifactId>...</artifactId>
    <version>...</version>
    <packaging>...</packaging>
    
    <!-- 4. Project info -->
    <name>...</name>
    <description>...</description>
    <url>...</url>
    
    <!-- 5. Properties -->
    <properties>...</properties>
    
    <!-- 6. Dependency Management -->
    <dependencyManagement>...</dependencyManagement>
    
    <!-- 7. Dependencies -->
    <dependencies>...</dependencies>
    
    <!-- 8. Build -->
    <build>...</build>
    
    <!-- 9. Profiles -->
    <profiles>...</profiles>
</project>
```

**Version Management:**
```xml
<properties>
    <!-- Use properties for versions -->
    <spring.version>5.3.27</spring.version>
    <junit.version>5.9.3</junit.version>
</properties>

<!-- Avoid version ranges -->
<version>[1.0,2.0)</version>  <!-- DON'T -->
<version>1.5.2</version>      <!-- DO -->
```

**Dependency Management:**
```xml
<!-- Use BOMs (Bill of Materials) -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.7.12</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

---

## Best Practices

### Project Structure
✅ Follow standard directory layout
✅ One artifact per module
✅ Keep POMs clean and organized
✅ Use meaningful artifact IDs
✅ Document custom conventions

### Dependency Management
✅ Use dependency management section
✅ Define versions in properties
✅ Import BOMs when available
✅ Minimize transitive dependencies
✅ Regular dependency updates

### Build Configuration
✅ Lock plugin versions
✅ Use plugin management
✅ Configure encoding explicitly
✅ Set Java version in properties
✅ Enable parallel builds

### Version Control
✅ Commit pom.xml
✅ Ignore target/ directory
✅ Ignore .m2/ directory
✅ Use .gitignore for Maven
✅ Version lock with Maven Wrapper

### CI/CD
✅ Cache .m2 repository
✅ Use profiles for environments
✅ Automate deployments
✅ Run tests in CI
✅ Generate reports

---

## Resources

### Official Documentation
- **Maven Documentation**: https://maven.apache.org/guides/
- **Maven Central**: https://search.maven.org/
- **Maven Plugins**: https://maven.apache.org/plugins/

### Books
- "Maven: The Complete Reference" by Sonatype
- "Apache Maven Cookbook" by Raghuram Bharathan

### Tools
- **Maven Wrapper**: Version-locked builds
- **Maven Daemon**: Faster builds
- **Nexus**: Repository manager
- **Artifactory**: Artifact repository

### Community
- **Stack Overflow**: [maven] tag
- **Maven Users List**: users@maven.apache.org

---

**Congratulations on completing the Maven Learning Roadmap! 🎉**

Continue practicing with real projects and explore advanced enterprise patterns!


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

