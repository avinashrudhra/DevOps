# Ansible Hands-On Exercises

30+ practical exercises to master Ansible from beginner to production expert.

---

## Beginner Level Exercises

### Exercise 1: Setup Ansible and Test Connectivity
**Objective**: Install Ansible and verify connectivity to managed hosts

**Tasks:**
1. Install Ansible
2. Create inventory
3. Test connectivity with ping
4. Run ad-hoc commands

<details>
<summary>Solution</summary>

```bash
# 1. Install Ansible
# Ubuntu
sudo apt update
sudo apt install ansible -y

# macOS
brew install ansible

# Verify
ansible --version

# 2. Create inventory
cat > inventory.ini <<EOF
[local]
localhost ansible_connection=local

[webservers]
web1 ansible_host=192.168.1.10 ansible_user=admin
web2 ansible_host=192.168.1.11 ansible_user=admin
EOF

# 3. Test connectivity
ansible all -i inventory.ini -m ping

# 4. Run ad-hoc commands
ansible all -i inventory.ini -m command -a "uptime"
ansible webservers -i inventory.ini -m shell -a "df -h"
ansible localhost -i inventory.ini -m setup
```

**Verification:**
- Ansible installed successfully
- All hosts respond to ping
- Ad-hoc commands execute
</details>

---

### Exercise 2: First Playbook - Install and Configure Nginx
**Objective**: Write and execute first Ansible playbook

<details>
<summary>Solution</summary>

```yaml
# nginx.yaml
---
- name: Install and configure Nginx
  hosts: webservers
  become: yes
  
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
      when: ansible_os_family == "Debian"
    
    - name: Install Nginx
      package:
        name: nginx
        state: present
    
    - name: Start Nginx service
      service:
        name: nginx
        state: started
        enabled: yes
    
    - name: Deploy custom index page
      copy:
        content: |
          <html>
            <head><title>{{ inventory_hostname }}</title></head>
            <body>
              <h1>Hello from {{ inventory_hostname }}</h1>
              <p>Managed by Ansible</p>
            </body>
          </html>
        dest: /var/www/html/index.html
      notify: Reload Nginx
  
  handlers:
    - name: Reload Nginx
      service:
        name: nginx
        state: reloaded
```

**Run playbook:**
```bash
# Syntax check
ansible-playbook nginx.yaml --syntax-check

# Dry run
ansible-playbook -i inventory.ini nginx.yaml --check

# Execute
ansible-playbook -i inventory.ini nginx.yaml

# Verify
ansible webservers -i inventory.ini -m command -a "systemctl status nginx"
```
</details>

---

### Exercise 3: Working with Variables
**Objective**: Use variables from multiple sources

<details>
<summary>Solution</summary>

**Create variable files:**
```yaml
# vars/common.yaml
---
app_name: myapp
app_user: deploy
app_port: 8080

# vars/production.yaml
---
environment: production
debug_mode: false
max_connections: 1000

# vars/development.yaml
---
environment: development
debug_mode: true
max_connections: 10
```

**Playbook:**
```yaml
# deploy.yaml
---
- name: Deploy application
  hosts: webservers
  become: yes
  vars_files:
    - vars/common.yaml
    - "vars/{{ env }}.yaml"
  
  tasks:
    - name: Create application user
      user:
        name: "{{ app_user }}"
        state: present
        shell: /bin/bash
    
    - name: Create application directory
      file:
        path: "/opt/{{ app_name }}"
        state: directory
        owner: "{{ app_user }}"
        mode: '0755'
    
    - name: Deploy configuration
      template:
        src: templates/app.conf.j2
        dest: "/opt/{{ app_name }}/config.conf"
        owner: "{{ app_user }}"
```

**Template:**
```jinja2
{# templates/app.conf.j2 #}
[application]
name = {{ app_name }}
environment = {{ environment }}
port = {{ app_port }}
debug = {{ debug_mode }}
max_connections = {{ max_connections }}

[server]
hostname = {{ inventory_hostname }}
ip_address = {{ ansible_default_ipv4.address }}
```

**Run:**
```bash
ansible-playbook -i inventory.ini deploy.yaml -e "env=production"
ansible-playbook -i inventory.ini deploy.yaml -e "env=development"
```
</details>

---

## Intermediate Level Exercises

### Exercise 4: Create Reusable Role
**Objective**: Build a production-ready Ansible role

<details>
<summary>Solution</summary>

```bash
# Create role structure
ansible-galaxy init roles/webserver

# roles/webserver/tasks/main.yaml
---
- name: Install web server packages
  package:
    name: "{{ item }}"
    state: present
  loop: "{{ webserver_packages }}"

- name: Configure web server
  template:
    src: "{{ webserver_config_template }}"
    dest: "{{ webserver_config_path }}"
  notify: Restart web server

- name: Ensure web server is running
  service:
    name: "{{ webserver_service }}"
    state: started
    enabled: yes

- name: Deploy site content
  copy:
    src: "{{ item }}"
    dest: "{{ webserver_docroot }}/"
  with_fileglob:
    - "files/site/*"

# roles/webserver/defaults/main.yaml
---
webserver_packages:
  - nginx
webserver_service: nginx
webserver_config_template: nginx.conf.j2
webserver_config_path: /etc/nginx/nginx.conf
webserver_docroot: /var/www/html
webserver_port: 80

# roles/webserver/handlers/main.yaml
---
- name: Restart web server
  service:
    name: "{{ webserver_service }}"
    state: restarted

# roles/webserver/templates/nginx.conf.j2
user www-data;
worker_processes {{ ansible_processor_vcpus }};

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    
    server {
        listen {{ webserver_port }};
        server_name {{ ansible_fqdn }};
        root {{ webserver_docroot }};
        index index.html;
    }
}

# Use role in playbook
---
- name: Configure web infrastructure
  hosts: webservers
  roles:
    - role: webserver
      webserver_port: 8080
```
</details>

---

### Exercise 5: Multi-Environment Deployment
**Objective**: Deploy to dev, staging, and production with different configs

<details>
<summary>Solution</summary>

**Directory structure:**
```
ansible-project/
├── inventory/
│   ├── dev/
│   │   ├── hosts
│   │   └── group_vars/
│   │       └── all.yaml
│   ├── staging/
│   │   ├── hosts
│   │   └── group_vars/
│   │       └── all.yaml
│   └── production/
│       ├── hosts
│       └── group_vars/
│           └── all.yaml
├── playbooks/
│   └── deploy.yaml
└── roles/
    └── application/
```

**inventory/dev/hosts:**
```ini
[webservers]
dev-web1.example.com
dev-web2.example.com

[databases]
dev-db1.example.com
```

**inventory/dev/group_vars/all.yaml:**
```yaml
---
environment: development
app_replicas: 1
db_size: small
enable_debug: true
```

**inventory/production/group_vars/all.yaml:**
```yaml
---
environment: production
app_replicas: 3
db_size: large
enable_debug: false
backup_enabled: true
monitoring_enabled: true
```

**Deploy:**
```bash
# Deploy to development
ansible-playbook -i inventory/dev playbooks/deploy.yaml

# Deploy to staging
ansible-playbook -i inventory/staging playbooks/deploy.yaml

# Deploy to production (with confirmation)
ansible-playbook -i inventory/production playbooks/deploy.yaml --step
```
</details>

---

### Exercise 6: Error Handling and Rollback
**Objective**: Implement deployment with automatic rollback on failure

<details>
<summary>Solution</summary>

```yaml
---
- name: Deploy with rollback capability
  hosts: webservers
  serial: 1
  
  vars:
    app_path: /opt/myapp
    backup_path: /backup
  
  tasks:
    - name: Deployment block
      block:
        - name: Create backup directory
          file:
            path: "{{ backup_path }}"
            state: directory
        
        - name: Backup current version
          archive:
            path: "{{ app_path }}"
            dest: "{{ backup_path }}/myapp-{{ ansible_date_time.epoch }}.tar.gz"
          register: backup_result
          when: stat_result.stat.exists
        
        - name: Check current version
          stat:
            path: "{{ app_path }}"
          register: stat_result
        
        - name: Stop application
          systemd:
            name: myapp
            state: stopped
        
        - name: Deploy new version
          unarchive:
            src: "https://releases.example.com/myapp-{{ version }}.tar.gz"
            dest: "{{ app_path }}"
            remote_src: yes
        
        - name: Start application
          systemd:
            name: myapp
            state: started
        
        - name: Wait for application to be ready
          uri:
            url: "http://localhost:8080/health"
            status_code: 200
          register: health_check
          until: health_check.status == 200
          retries: 10
          delay: 5
      
      rescue:
        - name: Rollback on failure
          debug:
            msg: "Deployment failed, initiating rollback"
        
        - name: Stop failed version
          systemd:
            name: myapp
            state: stopped
          ignore_errors: yes
        
        - name: Restore from backup
          unarchive:
            src: "{{ backup_result.dest }}"
            dest: "{{ app_path | dirname }}"
            remote_src: yes
          when: backup_result is defined
        
        - name: Start previous version
          systemd:
            name: myapp
            state: started
        
        - name: Verify rollback
          uri:
            url: "http://localhost:8080/health"
            status_code: 200
          register: rollback_health
          until: rollback_health.status == 200
          retries: 5
          delay: 5
        
        - name: Notify rollback
          debug:
            msg: "Rollback completed successfully"
      
      always:
        - name: Cleanup old backups
          shell: "ls -t {{ backup_path }}/myapp-*.tar.gz | tail -n +6 | xargs rm -f"
          ignore_errors: yes
```
</details>

---

## Advanced Level Exercises

### Exercise 7: Dynamic Inventory with AWS
**Objective**: Use dynamic inventory to manage AWS EC2 instances

<details>
<summary>Solution</summary>

**Install AWS collection:**
```bash
ansible-galaxy collection install amazon.aws
pip install boto3 botocore
```

**Configure AWS credentials:**
```bash
export AWS_ACCESS_KEY_ID="your-key"
export AWS_SECRET_ACCESS_KEY="your-secret"
export AWS_REGION="us-east-1"
```

**Create dynamic inventory:**
```yaml
# aws_ec2.yaml
---
plugin: amazon.aws.aws_ec2
regions:
  - us-east-1
filters:
  tag:Environment: production
  instance-state-name: running
keyed_groups:
  - key: tags.Role
    prefix: role
  - key: placement.availability_zone
    prefix: az
hostnames:
  - private-ip-address
```

**Use dynamic inventory:**
```bash
# List hosts
ansible-inventory -i aws_ec2.yaml --list

# Run playbook
ansible-playbook -i aws_ec2.yaml playbook.yaml

# Target specific group
ansible-playbook -i aws_ec2.yaml playbook.yaml --limit role_webserver
```
</details>

---

### Exercise 8: Ansible with Docker
**Objective**: Manage Docker containers and images

<details>
<summary>Solution</summary>

```yaml
---
- name: Manage Docker containers
  hosts: docker_hosts
  become: yes
  
  tasks:
    - name: Install Docker
      apt:
        name:
          - docker.io
          - python3-docker
        state: present
    
    - name: Start Docker service
      service:
        name: docker
        state: started
        enabled: yes
    
    - name: Pull images
      docker_image:
        name: "{{ item }}"
        source: pull
      loop:
        - nginx:latest
        - postgres:13
        - redis:6
    
    - name: Create Docker network
      docker_network:
        name: app_network
        state: present
    
    - name: Run PostgreSQL container
      docker_container:
        name: postgres
        image: postgres:13
        state: started
        restart_policy: always
        networks:
          - name: app_network
        env:
          POSTGRES_PASSWORD: "{{ db_password }}"
          POSTGRES_DB: myapp
        volumes:
          - postgres_data:/var/lib/postgresql/data
    
    - name: Run Redis container
      docker_container:
        name: redis
        image: redis:6
        state: started
        restart_policy: always
        networks:
          - name: app_network
    
    - name: Run application container
      docker_container:
        name: myapp
        image: "myapp:{{ app_version }}"
        state: started
        restart_policy: always
        networks:
          - name: app_network
        ports:
          - "80:8080"
        env:
          DB_HOST: postgres
          REDIS_HOST: redis
        links:
          - postgres
          - redis
```
</details>

---

### Exercise 9: Ansible with Kubernetes
**Objective**: Deploy applications to Kubernetes using Ansible

<details>
<summary>Solution</summary>

**Install Kubernetes collection:**
```bash
ansible-galaxy collection install kubernetes.core
pip install kubernetes
```

**Playbook:**
```yaml
---
- name: Deploy to Kubernetes
  hosts: localhost
  collections:
    - kubernetes.core
  
  vars:
    namespace: production
    app_name: myapp
    app_version: "1.0.0"
  
  tasks:
    - name: Create namespace
      k8s:
        api_version: v1
        kind: Namespace
        name: "{{ namespace }}"
        state: present
    
    - name: Create ConfigMap
      k8s:
        state: present
        definition:
          apiVersion: v1
          kind: ConfigMap
          metadata:
            name: "{{ app_name }}-config"
            namespace: "{{ namespace }}"
          data:
            APP_ENV: production
            LOG_LEVEL: info
    
    - name: Deploy application
      k8s:
        state: present
        definition:
          apiVersion: apps/v1
          kind: Deployment
          metadata:
            name: "{{ app_name }}"
            namespace: "{{ namespace }}"
          spec:
            replicas: 3
            selector:
              matchLabels:
                app: "{{ app_name }}"
            template:
              metadata:
                labels:
                  app: "{{ app_name }}"
              spec:
                containers:
                  - name: "{{ app_name }}"
                    image: "{{ app_name }}:{{ app_version }}"
                    ports:
                      - containerPort: 8080
                    envFrom:
                      - configMapRef:
                          name: "{{ app_name }}-config"
    
    - name: Create Service
      k8s:
        state: present
        definition:
          apiVersion: v1
          kind: Service
          metadata:
            name: "{{ app_name }}"
            namespace: "{{ namespace }}"
          spec:
            selector:
              app: "{{ app_name }}"
            ports:
              - port: 80
                targetPort: 8080
            type: LoadBalancer
    
    - name: Wait for deployment
      k8s_info:
        kind: Deployment
        name: "{{ app_name }}"
        namespace: "{{ namespace }}"
      register: deployment
      until: deployment.resources[0].status.readyReplicas == 3
      retries: 30
      delay: 10
```
</details>

---

### Exercise 10: Complete CI/CD Pipeline Integration
**Objective**: Integrate Ansible with Jenkins for full automation

<details>
<summary>Solution</summary>

**Jenkinsfile:**
```groovy
pipeline {
    agent any
    
    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'production'])
        string(name: 'VERSION', defaultValue: 'latest')
    }
    
    environment {
        ANSIBLE_HOST_KEY_CHECKING = 'False'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/company/ansible-playbooks.git'
            }
        }
        
        stage('Lint Playbooks') {
            steps {
                sh 'ansible-lint playbooks/*.yaml'
            }
        }
        
        stage('Syntax Check') {
            steps {
                sh """
                    ansible-playbook \
                        -i inventory/${params.ENVIRONMENT} \
                        playbooks/deploy.yaml \
                        --syntax-check
                """
            }
        }
        
        stage('Dry Run') {
            steps {
                sh """
                    ansible-playbook \
                        -i inventory/${params.ENVIRONMENT} \
                        playbooks/deploy.yaml \
                        -e "version=${params.VERSION}" \
                        --check --diff
                """
            }
        }
        
        stage('Deploy') {
            when {
                expression { params.ENVIRONMENT == 'production' }
            }
            steps {
                input message: 'Deploy to production?', ok: 'Deploy'
                sh """
                    ansible-playbook \
                        -i inventory/${params.ENVIRONMENT} \
                        playbooks/deploy.yaml \
                        -e "version=${params.VERSION}"
                """
            }
        }
        
        stage('Verify Deployment') {
            steps {
                sh """
                    ansible-playbook \
                        -i inventory/${params.ENVIRONMENT} \
                        playbooks/verify.yaml
                """
            }
        }
    }
    
    post {
        success {
            slackSend color: 'good',
                     message: "✅ Deployment to ${params.ENVIRONMENT} succeeded"
        }
        failure {
            slackSend color: 'danger',
                     message: "❌ Deployment to ${params.ENVIRONMENT} failed"
        }
    }
}
```
</details>

---

**Complete all exercises to master Ansible! ⚙️🚀**

For more resources:
- [Learning Roadmap](ansible-learning-roadmap.md)
- [Quick Reference](ansible-quick-reference.md)
- [Troubleshooting Guide](ansible-troubleshooting-guide.md)
- [Interview Questions](ansible-interview-questions.md)


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

