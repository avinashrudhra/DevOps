# SonarQube Hands-On Exercises

Complete practical exercises to master SonarQube from beginner to advanced level.

---

## Beginner Level Exercises

### Exercise 1: Install SonarQube Locally
**Objective**: Set up SonarQube development environment

**Tasks:**
1. Download SonarQube Community Edition
2. Install and start the server
3. Access web interface
4. Change default password
5. Generate authentication token

<details>
<summary>Solution</summary>

```bash
# Download
cd ~/tools
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.3.79811.zip
unzip sonarqube-9.9.3.79811.zip
cd sonarqube-9.9.3.79811

# Start server
bin/linux-x86-64/sonar.sh start

# Check logs
tail -f logs/sonar.log

# Wait for "SonarQube is operational"

# Access: http://localhost:9000
# Login: admin/admin
# Change password when prompted

# Generate token:
# My Account → Security → Generate Token
# Name: local-dev
# Type: User Token
# Save the generated token
```

**Verification:**
- Server accessible at http://localhost:9000
- Successfully logged in
- Token generated and saved
</details>

---

### Exercise 2: Analyze Your First Java Project
**Objective**: Perform first code analysis with Maven

**Tasks:**
1. Create/use existing Maven project
2. Add SonarQube plugin
3. Run analysis
4. Review results in UI

<details>
<summary>Solution</summary>

```bash
# Create sample project
mvn archetype:generate \
  -DgroupId=com.example \
  -DartifactId=sonar-demo \
  -DarchetypeArtifactId=maven-archetype-quickstart \
  -DinteractiveMode=false

cd sonar-demo
```

```xml
<!-- Add to pom.xml -->
<build>
    <plugins>
        <plugin>
            <groupId>org.sonarsource.scanner.maven</groupId>
            <artifactId>sonar-maven-plugin</artifactId>
            <version>3.10.0.2594</version>
        </plugin>
    </plugins>
</build>
```

```bash
# Run analysis
mvn clean verify sonar:sonar \
  -Dsonar.projectKey=sonar-demo \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=YOUR_TOKEN

# Check output for analysis results
# Open http://localhost:9000/dashboard?id=sonar-demo
```

**Review:**
- Check Bugs, Vulnerabilities, Code Smells
- View Lines of Code
- Explore Issues tab
- Check Quality Gate status
</details>

---

### Exercise 3: Configure Scanner Properties
**Objective**: Customize analysis with sonar-project.properties

**Tasks:**
1. Create sonar-project.properties
2. Configure project metadata
3. Set source directories
4. Add exclusions
5. Run analysis with CLI scanner

<details>
<summary>Solution</summary>

```properties
# sonar-project.properties
sonar.projectKey=my-custom-project
sonar.projectName=My Custom Project
sonar.projectVersion=1.0

# Source directories
sonar.sources=src/main/java
sonar.tests=src/test/java

# Exclusions
sonar.exclusions=**/generated/**,**/*.min.js

# Test exclusions
sonar.test.exclusions=**/test/**/*.java

# Source encoding
sonar.sourceEncoding=UTF-8

# Java specifics
sonar.java.binaries=target/classes
```

```bash
# Install SonarScanner CLI
wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.8.0.2856-linux.zip
unzip sonar-scanner-cli-4.8.0.2856-linux.zip
export PATH=$PATH:~/sonar-scanner-4.8.0.2856-linux/bin

# Run analysis
sonar-scanner \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=YOUR_TOKEN
```
</details>

---

### Exercise 4: Integrate Code Coverage
**Objective**: Add JaCoCo coverage and upload to SonarQube

**Tasks:**
1. Add JaCoCo plugin to Maven
2. Write unit tests
3. Generate coverage report
4. Upload to SonarQube
5. Verify coverage in dashboard

<details>
<summary>Solution</summary>

```xml
<!-- Add to pom.xml -->
<build>
    <plugins>
        <!-- JaCoCo Plugin -->
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
    </plugins>
</build>

<properties>
    <sonar.coverage.jacoco.xmlReportPaths>
        ${project.build.directory}/site/jacoco/jacoco.xml
    </sonar.coverage.jacoco.xmlReportPaths>
</properties>
```

```java
// src/main/java/com/example/Calculator.java
public class Calculator {
    public int add(int a, int b) {
        return a + b;
    }
    
    public int subtract(int a, int b) {
        return a - b;
    }
}

// src/test/java/com/example/CalculatorTest.java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

public class CalculatorTest {
    @Test
    public void testAdd() {
        Calculator calc = new Calculator();
        assertEquals(5, calc.add(2, 3));
    }
    
    @Test
    public void testSubtract() {
        Calculator calc = new Calculator();
        assertEquals(1, calc.subtract(3, 2));
    }
}
```

```bash
# Run tests and generate coverage
mvn clean test

# Verify coverage report generated
ls -la target/site/jacoco/jacoco.xml

# Upload to SonarQube
mvn sonar:sonar \
  -Dsonar.projectKey=coverage-demo \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=YOUR_TOKEN

# Check coverage in UI
# Dashboard → Coverage
```
</details>

---

### Exercise 5: Navigate SonarQube UI
**Objective**: Master SonarQube web interface

**Tasks:**
1. Explore project dashboard
2. Review different issue types
3. Filter issues
4. Assign issues
5. Add comments

<details>
<summary>Solution</summary>

**Dashboard Exploration:**
```
1. Overview Tab:
   - View metrics summary
   - Check quality gate status
   - Review new code vs overall code
   - See activity graph

2. Issues Tab:
   - Filter by type (Bug, Vulnerability, Code Smell)
   - Filter by severity (Blocker, Critical, Major, Minor, Info)
   - Filter by resolution status
   - Sort by creation date

3. Security Hotspots Tab:
   - Review security-sensitive code
   - Mark as reviewed
   - Add review comments

4. Measures Tab:
   - Deep dive into metrics
   - View coverage details
   - Check duplication
   - Review complexity

5. Code Tab:
   - Browse source code
   - View issues inline
   - Check coverage per file
```

**Practice Actions:**
- Open an issue
- Change severity
- Assign to yourself
- Add comment
- Mark as false positive
- Close as won't fix
</details>

---

## Intermediate Level Exercises

### Exercise 6: Create Custom Quality Gate
**Objective**: Build organization-specific quality gate

**Tasks:**
1. Create new quality gate
2. Add conditions for New Code
3. Add conditions for Overall Code
4. Assign to project
5. Test by running analysis

<details>
<summary>Solution</summary>

```
Navigate: Quality Gates → Create

Name: "Company Standard"

Add Conditions on New Code:
1. Coverage is less than 85%
2. Duplicated Lines (%) is greater than 2%
3. Maintainability Rating is worse than A
4. Reliability Rating is worse than A
5. Security Rating is worse than A
6. Bugs is greater than 0
7. Vulnerabilities is greater than 0
8. Security Hotspots Reviewed is less than 100%

Add Conditions on Overall Code:
1. Security Rating is worse than B
2. Reliability Rating is worse than B
3. Duplicated Lines (%) is greater than 5%

Save and set as default or assign to specific projects

Test:
- Run analysis on a project
- Check if gate passes/fails
- Review conditions that failed
```

**Verification:**
```bash
# Check gate status via API
curl -u token: \
  "http://localhost:9000/api/qualitygates/project_status?projectKey=my-project" | jq .
```
</details>

---

### Exercise 7: Configure Custom Quality Profile
**Objective**: Create language-specific quality profile

**Tasks:**
1. Create Java quality profile
2. Extend from "Sonar way"
3. Activate security rules
4. Customize rule parameters
5. Set as default

<details>
<summary>Solution</summary>

```
Navigate: Quality Profiles → Create

Profile Details:
- Name: "Company Java Profile"
- Language: Java
- Parent: Sonar way

Activate Additional Rules:
1. Click "Activate More"
2. Filter by tag: "security"
3. Activate all security rules
4. Filter by tag: "owasp"
5. Activate OWASP rules

Customize Rules:
1. Search: "Cognitive Complexity of methods"
2. Change parameter from 15 to 20

3. Search: "Methods should not be too complex"
4. Change cyclomatic complexity from 10 to 15

5. Search: "Classes should not have too many methods"
6. Adjust threshold as needed

Set as Default:
- Quality Profiles → Company Java Profile → Set as Default

Export Profile:
- Back Up → Download XML
- Save to version control
```

**Test:**
```bash
# Analyze with new profile
mvn sonar:sonar \
  -Dsonar.profile="Company Java Profile"

# Verify in UI that profile is used
```
</details>

---

### Exercise 8: Set Up Docker-Based SonarQube
**Objective**: Run SonarQube in Docker with PostgreSQL

**Tasks:**
1. Create docker-compose.yml
2. Configure PostgreSQL
3. Start services
4. Configure SonarQube
5. Run first analysis

<details>
<summary>Solution</summary>

```yaml
# docker-compose.yml
version: '3'

services:
  sonarqube:
    image: sonarqube:9.9.3-community
    container_name: sonarqube
    depends_on:
      - db
    environment:
      SONAR_JDBC_URL: jdbc:postgresql://db:5432/sonar
      SONAR_JDBC_USERNAME: sonar
      SONAR_JDBC_PASSWORD: sonar
    volumes:
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
      - sonarqube_logs:/opt/sonarqube/logs
    ports:
      - "9000:9000"
    networks:
      - sonarnet

  db:
    image: postgres:13
    container_name: postgres
    environment:
      POSTGRES_USER: sonar
      POSTGRES_PASSWORD: sonar
      POSTGRES_DB: sonar
    volumes:
      - postgresql:/var/lib/postgresql
      - postgresql_data:/var/lib/postgresql/data
    networks:
      - sonarnet

volumes:
  sonarqube_data:
  sonarqube_extensions:
  sonarqube_logs:
  postgresql:
  postgresql_data:

networks:
  sonarnet:
    driver: bridge
```

```bash
# Start services
docker-compose up -d

# Check logs
docker-compose logs -f sonarqube

# Wait for startup (may take 2-3 minutes)
# Access: http://localhost:9000

# Stop services
docker-compose down

# Remove volumes (clean install)
docker-compose down -v
```

**Verification:**
- Access UI at http://localhost:9000
- Login with admin/admin
- Run analysis against Docker instance
</details>

---

### Exercise 9: Jenkins Integration
**Objective**: Integrate SonarQube with Jenkins pipeline

**Tasks:**
1. Install Jenkins plugins
2. Configure SonarQube server
3. Create Jenkinsfile
4. Run pipeline
5. Verify quality gate enforcement

<details>
<summary>Solution</summary>

**Install Plugins:**
```
Jenkins → Manage Jenkins → Manage Plugins
- SonarQube Scanner for Jenkins
- SonarQube Quality Gates Plugin
```

**Configure Server:**
```
Jenkins → Manage Jenkins → Configure System → SonarQube servers
- Name: SonarQube
- Server URL: http://sonarqube:9000
- Server authentication token: (add token from SonarQube)
```

**Jenkinsfile:**
```groovy
pipeline {
    agent any
    
    tools {
        maven 'Maven 3.8'
        jdk 'JDK 11'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/your-org/your-repo.git'
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
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        mvn sonar:sonar \
                          -Dsonar.projectKey=jenkins-demo \
                          -Dsonar.projectName="Jenkins Demo"
                    '''
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    script {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }
        
        stage('Deploy') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                echo 'Deploying application...'
                // Add deployment steps
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
```

**Test:**
```bash
# Create Jenkins job
# - New Item → Pipeline
# - Pipeline script from SCM
# - Add repository URL
# - Save and Build Now

# Verify:
# - Analysis runs
# - Results appear in SonarQube
# - Quality gate checked
# - Build fails if gate fails
```
</details>

---

### Exercise 10: Multi-Module Project Analysis
**Objective**: Analyze Maven multi-module project

**Tasks:**
1. Create multi-module structure
2. Configure parent POM
3. Set up module dependencies
4. Run analysis
5. Review results per module

<details>
<summary>Solution</summary>

**Project Structure:**
```
parent-project/
├── pom.xml (parent)
├── core/
│   └── pom.xml
├── service/
│   └── pom.xml
└── web/
    └── pom.xml
```

**Parent POM:**
```xml
<project>
    <groupId>com.example</groupId>
    <artifactId>parent</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    
    <modules>
        <module>core</module>
        <module>service</module>
        <module>web</module>
    </modules>
    
    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <sonar.projectKey>com.example:parent</sonar.projectKey>
    </properties>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.sonarsource.scanner.maven</groupId>
                <artifactId>sonar-maven-plugin</artifactId>
                <version>3.10.0.2594</version>
            </plugin>
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
        </plugins>
    </build>
</project>
```

```bash
# From parent directory
mvn clean verify sonar:sonar \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=YOUR_TOKEN

# SonarQube will show:
# - Overall project metrics
# - Individual module metrics
# - Module-specific issues
```

**Review:**
- Check each module separately
- Compare metrics across modules
- Identify which modules need attention
</details>

---

## Advanced Level Exercises

### Exercise 11: Branch Analysis Configuration
**Objective**: Set up branch analysis (Developer Edition+)

**Tasks:**
1. Configure branch analysis
2. Analyze main branch
3. Analyze feature branch
4. Compare branches
5. Set up branch policies

<details>
<summary>Solution</summary>

```bash
# Analyze main branch
git checkout main
mvn sonar:sonar \
  -Dsonar.branch.name=main \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=YOUR_TOKEN

# Create and analyze feature branch
git checkout -b feature/new-feature
# Make some changes
mvn sonar:sonar \
  -Dsonar.branch.name=feature/new-feature \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=YOUR_TOKEN
```

**UI Review:**
```
Project → Branches
- View all branches
- Compare metrics
- Check new issues
- Review quality gate per branch
```

**Note:** Branch analysis requires Developer Edition or higher
</details>

---

### Exercise 12: Pull Request Decoration
**Objective**: Configure PR decoration with GitHub

**Tasks:**
1. Install SonarQube GitHub App
2. Configure ALM integration
3. Link repository
4. Analyze pull request
5. Review decoration in GitHub

<details>
<summary>Solution</summary>

**Configure GitHub Integration:**
```
1. Install SonarQube GitHub App:
   - https://github.com/apps/sonarqube
   - Install for your organization

2. In SonarQube:
   Administration → Configuration → General Settings → ALM Integrations
   - GitHub → Add configuration
   - Configuration name: GitHub
   - GitHub App ID: (from GitHub)
   - Client ID: (from GitHub)
   - Client Secret: (from GitHub)
   - Private Key: (from GitHub App)

3. Link Project:
   Project Settings → General Settings → ALM Integration
   - Select GitHub configuration
   - Repository: your-org/your-repo
```

**Jenkins Pipeline for PR:**
```groovy
stage('PR Analysis') {
    when {
        changeRequest()
    }
    steps {
        withSonarQubeEnv('SonarQube') {
            sh """
                mvn sonar:sonar \
                  -Dsonar.pullrequest.key=${env.CHANGE_ID} \
                  -Dsonar.pullrequest.branch=${env.CHANGE_BRANCH} \
                  -Dsonar.pullrequest.base=${env.CHANGE_TARGET}
            """
        }
    }
}
```

**Verify:**
- Create PR in GitHub
- Jenkins runs analysis
- Quality gate status appears on PR
- Issues decorated inline in PR
</details>

---

### Exercise 13: Security Analysis Deep Dive
**Objective**: Master security vulnerability detection

**Tasks:**
1. Activate all security rules
2. Introduce security issues
3. Analyze and review
4. Fix vulnerabilities
5. Review security hotspots

<details>
<summary>Solution</summary>

**Activate Security Rules:**
```
Quality Profiles → Java → Activate More
- Tag: security → Activate All
- Tag: owasp → Activate All
- Tag: cwe → Activate All
```

**Create Vulnerable Code:**
```java
// SQL Injection
public class VulnerableCode {
    public void sqlInjection(String userId) {
        String query = "SELECT * FROM users WHERE id = " + userId;
        // Execute query...
    }
    
    // Hardcoded credentials
    public void hardcodedPassword() {
        String password = "admin123";
        // Use password...
    }
    
    // XSS vulnerability
    public void xssVulnerability(HttpServletResponse response, 
                                  HttpServletRequest request) throws IOException {
        String name = request.getParameter("name");
        response.getWriter().write("Hello " + name);
    }
    
    // Weak cryptography
    public String weakCrypto(String data) throws Exception {
        MessageDigest md = MessageDigest.getInstance("MD5");
        byte[] hash = md.digest(data.getBytes());
        return new String(hash);
    }
}
```

```bash
# Analyze
mvn sonar:sonar

# Review in UI:
# - Security Vulnerabilities tab
# - Check OWASP categorization
# - Review remediation advice
```

**Fix Issues:**
```java
// Fixed code
public class SecureCode {
    // Use prepared statements
    public void secureSql(String userId, Connection conn) throws SQLException {
        String query = "SELECT * FROM users WHERE id = ?";
        PreparedStatement stmt = conn.prepareStatement(query);
        stmt.setString(1, userId);
        ResultSet rs = stmt.executeQuery();
    }
    
    // Use environment variables
    public void securePassword() {
        String password = System.getenv("DB_PASSWORD");
        // Use password...
    }
    
    // Encode output
    public void secureOutput(HttpServletResponse response, 
                             HttpServletRequest request) throws IOException {
        String name = request.getParameter("name");
        String encodedName = HtmlUtils.htmlEscape(name);
        response.getWriter().write("Hello " + encodedName);
    }
    
    // Strong cryptography
    public String strongCrypto(String data) throws Exception {
        MessageDigest md = MessageDigest.getInstance("SHA-256");
        byte[] hash = md.digest(data.getBytes());
        return Base64.getEncoder().encodeToString(hash);
    }
}
```

**Re-analyze and verify fixes**
</details>

---

### Exercise 14: Performance Optimization
**Objective**: Optimize SonarQube analysis performance

**Tasks:**
1. Measure baseline analysis time
2. Configure scanner optimization
3. Exclude unnecessary files
4. Enable incremental analysis
5. Measure improvements

<details>
<summary>Solution</summary>

**Baseline Measurement:**
```bash
time mvn clean verify sonar:sonar
# Note the time taken
```

**Optimization 1: Exclude Files:**
```xml
<!-- pom.xml -->
<properties>
    <sonar.exclusions>
        **/generated/**,
        **/dto/**,
        **/model/**,
        **/*.min.js,
        **/vendor/**
    </sonar.exclusions>
    
    <sonar.test.exclusions>
        **/test/**
    </sonar.test.exclusions>
    
    <sonar.cpd.exclusions>
        **/generated/**
    </sonar.cpd.exclusions>
</properties>
```

**Optimization 2: Scanner Performance:**
```properties
# sonar-project.properties
sonar.scm.disabled=true  # If not using SCM blame
sonar.cpd.cross_project=false  # Disable cross-project duplication
```

**Optimization 3: Parallel Analysis:**
```bash
# Maven parallel build
mvn clean verify sonar:sonar -T 4
```

**Optimization 4: Incremental Analysis (Enterprise+):**
```bash
mvn sonar:sonar -Dsonar.incrementalAnalysis=true
```

**Measure Results:**
```bash
time mvn clean verify sonar:sonar
# Compare with baseline
```
</details>

---

### Exercise 15: Custom Rules with API
**Objective**: Use SonarQube API for automation

**Tasks:**
1. Create project via API
2. Configure quality gate via API
3. Retrieve metrics via API
4. Export data to report
5. Automate project setup

<details>
<summary>Solution</summary>

**API Scripts:**
```bash
#!/bin/bash
SONAR_HOST="http://localhost:9000"
SONAR_TOKEN="YOUR_TOKEN"

# Create project
create_project() {
    curl -u $SONAR_TOKEN: -X POST \
      "$SONAR_HOST/api/projects/create?name=API-Project&project=api-project"
}

# Set quality gate
set_quality_gate() {
    curl -u $SONAR_TOKEN: -X POST \
      "$SONAR_HOST/api/qualitygates/select?gateId=1&projectKey=api-project"
}

# Get metrics
get_metrics() {
    curl -u $SONAR_TOKEN: \
      "$SONAR_HOST/api/measures/component?component=api-project&metricKeys=bugs,vulnerabilities,code_smells,coverage" \
      | jq .
}

# Get quality gate status
get_gate_status() {
    curl -u $SONAR_TOKEN: \
      "$SONAR_HOST/api/qualitygates/project_status?projectKey=api-project" \
      | jq .
}

# Export to CSV
export_metrics() {
    curl -u $SONAR_TOKEN: \
      "$SONAR_HOST/api/measures/component?component=api-project&metricKeys=ncloc,bugs,vulnerabilities,code_smells,coverage,duplicated_lines_density" \
      | jq -r '.component.measures[] | [.metric, .value] | @csv' \
      > metrics.csv
}

# Run functions
create_project
set_quality_gate
get_metrics
get_gate_status
export_metrics
```

**Python Script:**
```python
import requests
import json

class SonarQubeAPI:
    def __init__(self, url, token):
        self.url = url
        self.auth = (token, '')
    
    def get_project_metrics(self, project_key):
        endpoint = f"{self.url}/api/measures/component"
        params = {
            'component': project_key,
            'metricKeys': 'bugs,vulnerabilities,code_smells,coverage'
        }
        response = requests.get(endpoint, auth=self.auth, params=params)
        return response.json()
    
    def get_issues(self, project_key):
        endpoint = f"{self.url}/api/issues/search"
        params = {
            'componentKeys': project_key,
            'resolved': 'false'
        }
        response = requests.get(endpoint, auth=self.auth, params=params)
        return response.json()

# Usage
sonar = SonarQubeAPI('http://localhost:9000', 'YOUR_TOKEN')
metrics = sonar.get_project_metrics('my-project')
print(json.dumps(metrics, indent=2))
```
</details>

---

## Tips for Practice

1. **Start Simple**: Begin with basic installation and analysis
2. **Incremental Learning**: Master one feature before moving to next
3. **Real Projects**: Practice on actual codebases
4. **Explore UI**: Spend time navigating all sections
5. **Read Documentation**: Official docs are comprehensive
6. **Community**: Engage with SonarSource community
7. **Automation**: Automate repetitive tasks
8. **Security Focus**: Pay special attention to security features
9. **Quality Gates**: Experiment with different thresholds
10. **Integration**: Practice CI/CD integrations

---

## Next Steps

1. Complete all exercises in order
2. Practice with real projects
3. Study [Learning Roadmap](sonarqube-learning-roadmap.md)
4. Review [Troubleshooting Guide](sonarqube-troubleshooting-guide.md)
5. Prepare with [Interview Questions](sonarqube-interview-questions.md)
6. Move to Jenkins (next in pipeline)

**Happy practicing! 🚀**


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

