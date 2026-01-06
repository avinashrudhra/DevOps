# Ansible Learning Roadmap

Complete 12-week curriculum to master Ansible for infrastructure automation and configuration management.

---

## 🎯 Learning Objectives

By completing this roadmap, you will:
- ✅ Master Ansible fundamentals and architecture
- ✅ Write production-ready playbooks
- ✅ Create reusable roles
- ✅ Manage complex inventories
- ✅ Integrate Ansible with CI/CD
- ✅ Automate cloud infrastructure
- ✅ Implement security best practices
- ✅ Troubleshoot Ansible issues

**Target Audience:** DevOps engineers, SysAdmins, Infrastructure engineers (3-7+ years experience)

---

## 📅 12-Week Structured Learning Path

### **Week 1: Ansible Fundamentals**

#### **Day 1-2: Introduction & Installation**
- What is Ansible?
- Ansible architecture
- Agentless design
- Installation methods

**Installation:**
```bash
# macOS
brew install ansible

# Ubuntu
sudo apt update
sudo apt install ansible

# Python pip
pip install ansible

# Verify
ansible --version
```

#### **Day 3-4: Inventory Basics**
- Inventory file formats (INI, YAML)
- Host patterns
- Groups and subgroups
- Inventory variables

**Basic Inventory:**
```ini
# inventory.ini
[webservers]
web1.example.com
web2.example.com ansible_host=192.168.1.10

[databases]
db1.example.com

[production:children]
webservers
databases

[production:vars]
ansible_user=admin
ansible_port=22
```

#### **Day 5-7: Ad-Hoc Commands**
- Module syntax
- Common modules
- Running commands
- Gathering facts

**Practice:**
```bash
# Ping all hosts
ansible all -i inventory.ini -m ping

# Check uptime
ansible webservers -i inventory.ini -m command -a "uptime"

# Install package
ansible webservers -i inventory.ini -m apt -a "name=nginx state=present" --become

# Gather facts
ansible web1.example.com -m setup

# Copy file
ansible webservers -m copy -a "src=file.txt dest=/tmp/file.txt"
```

**Weekly Goal:** Understand Ansible basics and run ad-hoc commands

---

### **Week 2: Playbooks & Tasks**

#### **Day 8-10: First Playbook**
- YAML syntax
- Playbook structure
- Tasks
- Plays

**First Playbook:**
```yaml
---
- name: Configure web servers
  hosts: webservers
  become: yes
  
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
        update_cache: yes
    
    - name: Start Nginx
      service:
        name: nginx
        state: started
        enabled: yes
    
    - name: Deploy index page
      copy:
        content: "<h1>Hello from {{ inventory_hostname }}</h1>"
        dest: /var/www/html/index.html
```

**Run Playbook:**
```bash
# Syntax check
ansible-playbook playbook.yaml --syntax-check

# Dry run
ansible-playbook playbook.yaml --check

# Execute
ansible-playbook -i inventory.ini playbook.yaml

# Verbose output
ansible-playbook playbook.yaml -vvv
```

#### **Day 11-12: Variables**
- Variable precedence
- Defining variables
- Variable files
- Special variables

**Variables Example:**
```yaml
---
- name: Deploy application
  hosts: webservers
  vars:
    app_name: myapp
    app_port: 8080
    app_version: "1.0.0"
  vars_files:
    - vars/common.yaml
    - vars/{{ env }}.yaml
  
  tasks:
    - name: Create app directory
      file:
        path: "/opt/{{ app_name }}"
        state: directory
    
    - name: Deploy application
      template:
        src: app.conf.j2
        dest: "/etc/{{ app_name }}/config.conf"
```

#### **Day 13-14: Handlers & Conditionals**
- Handlers
- Notify
- When conditions
- Loops

**Example:**
```yaml
---
- name: Configure services
  hosts: webservers
  tasks:
    - name: Update nginx config
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Restart Nginx
    
    - name: Install packages
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - nginx
        - python3
        - git
      when: ansible_os_family == "Debian"
  
  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```

**Weekly Goal:** Write and execute playbooks with variables and handlers

---

### **Week 3: Templates & Files**

#### **Day 15-17: Jinja2 Templates**
- Template syntax
- Variables in templates
- Filters
- Control structures

**Template Example:**
```jinja2
{# nginx.conf.j2 #}
user {{ nginx_user }};
worker_processes {{ ansible_processor_vcpus }};

http {
    upstream backend {
        {% for host in groups['webservers'] %}
        server {{ hostvars[host]['ansible_host'] }}:{{ app_port }};
        {% endfor %}
    }
    
    server {
        listen {{ http_port }};
        server_name {{ domain }};
        
        {% if ssl_enabled %}
        listen 443 ssl;
        ssl_certificate {{ ssl_cert }};
        ssl_certificate_key {{ ssl_key }};
        {% endif %}
        
        location / {
            proxy_pass http://backend;
        }
    }
}
```

#### **Day 18-19: File Operations**
- copy module
- template module
- file module
- lineinfile module
- blockinfile module

**File Operations:**
```yaml
tasks:
  # Copy file
  - name: Copy configuration
    copy:
      src: files/config.ini
      dest: /etc/myapp/config.ini
      owner: root
      group: root
      mode: '0644'
  
  # Template file
  - name: Generate config from template
    template:
      src: templates/app.conf.j2
      dest: /etc/myapp/app.conf
  
  # Create directory
  - name: Create directories
    file:
      path: "{{ item }}"
      state: directory
      mode: '0755'
    loop:
      - /var/log/myapp
      - /var/lib/myapp
      - /etc/myapp
  
  # Modify file
  - name: Add line to config
    lineinfile:
      path: /etc/myapp/config.ini
      line: "debug=true"
      create: yes
  
  # Add block
  - name: Add configuration block
    blockinfile:
      path: /etc/hosts
      block: |
        192.168.1.10 web1.local
        192.168.1.11 web2.local
```

#### **Day 20-21: Facts & Gathering**
- System facts
- Custom facts
- Fact caching
- Magic variables

**Facts Example:**
```yaml
tasks:
  - name: Display OS information
    debug:
      msg: "This is {{ ansible_distribution }} {{ ansible_distribution_version }}"
  
  - name: Configure based on memory
    template:
      src: app.conf.j2
      dest: /etc/app.conf
    vars:
      memory_limit: "{{ (ansible_memtotal_mb * 0.7) | int }}"
  
  - name: Set custom fact
    set_fact:
      deployment_time: "{{ ansible_date_time.iso8601 }}"
```

**Weekly Goal:** Master templates and file operations

---

### **Week 4: Roles & Best Practices**

#### **Day 22-24: Creating Roles**
- Role structure
- Tasks organization
- Defaults and variables
- Role dependencies

**Role Structure:**
```
roles/
└── webserver/
    ├── tasks/
    │   └── main.yaml
    ├── handlers/
    │   └── main.yaml
    ├── templates/
    │   └── nginx.conf.j2
    ├── files/
    │   └── index.html
    ├── vars/
    │   └── main.yaml
    ├── defaults/
    │   └── main.yaml
    ├── meta/
    │   └── main.yaml
    └── README.md
```

**roles/webserver/tasks/main.yaml:**
```yaml
---
- name: Install Nginx
  apt:
    name: nginx
    state: present

- name: Configure Nginx
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: Restart Nginx

- name: Start Nginx
  service:
    name: nginx
    state: started
    enabled: yes
```

**Using Roles:**
```yaml
---
- name: Configure web infrastructure
  hosts: webservers
  roles:
    - role: common
    - role: webserver
      vars:
        nginx_port: 8080
    - role: application
```

#### **Day 25-26: Ansible Galaxy**
- Installing roles
- Creating roles
- Publishing roles
- Requirements file

**Galaxy Usage:**
```bash
# Install role
ansible-galaxy install geerlingguy.nginx

# Install from requirements
cat > requirements.yaml <<EOF
roles:
  - name: geerlingguy.nginx
  - name: geerlingguy.docker
collections:
  - name: community.general
  - name: ansible.posix
EOF

ansible-galaxy install -r requirements.yaml

# Create role
ansible-galaxy init myrole

# Search roles
ansible-galaxy search nginx
```

#### **Day 27-28: Project Structure**
- Directory layout
- Best practices
- Version control
- Documentation

**Production Structure:**
```
ansible-project/
├── inventory/
│   ├── production/
│   │   ├── hosts
│   │   └── group_vars/
│   ├── staging/
│   │   ├── hosts
│   │   └── group_vars/
│   └── development/
│       ├── hosts
│       └── group_vars/
├── playbooks/
│   ├── site.yaml
│   ├── webservers.yaml
│   └── databases.yaml
├── roles/
│   ├── common/
│   ├── webserver/
│   └── database/
├── group_vars/
│   ├── all.yaml
│   └── webservers.yaml
├── host_vars/
│   └── web1.yaml
├── files/
├── templates/
├── ansible.cfg
├── requirements.yaml
└── README.md
```

**Weekly Goal:** Create reusable roles and organize projects

---

### **Week 5-6: Advanced Playbooks**

#### **Complex Workflows**
- Blocks
- Error handling
- Delegation
- Serial execution
- Tags

**Advanced Playbook:**
```yaml
---
- name: Deploy application with rollback
  hosts: webservers
  serial: 2  # Deploy 2 at a time
  max_fail_percentage: 25
  
  tasks:
    - name: Deployment block
      block:
        - name: Stop application
          service:
            name: myapp
            state: stopped
        
        - name: Backup current version
          archive:
            path: /opt/myapp
            dest: "/backup/myapp-{{ ansible_date_time.epoch }}.tar.gz"
        
        - name: Deploy new version
          unarchive:
            src: "https://releases.com/myapp-{{ version }}.tar.gz"
            dest: /opt/myapp
            remote_src: yes
        
        - name: Start application
          service:
            name: myapp
            state: started
        
        - name: Health check
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
            msg: "Deployment failed, rolling back..."
        
        - name: Restore from backup
          unarchive:
            src: "/backup/myapp-{{ ansible_date_time.epoch }}.tar.gz"
            dest: /opt/myapp
        
        - name: Start old version
          service:
            name: myapp
            state: started
      
      always:
        - name: Send notification
          slack:
            token: "{{ slack_token }}"
            msg: "Deployment {{ 'succeeded' if health_check.status == 200 else 'failed' }}"
      
      tags:
        - deploy
```

---

### **Week 7-8: Cloud & Containers**

#### **Cloud Automation**
- AWS modules
- Azure modules
- GCP modules
- Cloud inventory

**AWS Example:**
```yaml
---
- name: Provision AWS infrastructure
  hosts: localhost
  collections:
    - amazon.aws
  
  tasks:
    - name: Create VPC
      ec2_vpc_net:
        name: production-vpc
        cidr_block: 10.0.0.0/16
        region: us-east-1
        tags:
          Environment: production
      register: vpc
    
    - name: Create subnet
      ec2_vpc_subnet:
        vpc_id: "{{ vpc.vpc.id }}"
        cidr: 10.0.1.0/24
        az: us-east-1a
        tags:
          Name: production-subnet
      register: subnet
    
    - name: Launch EC2 instances
      ec2_instance:
        name: "web-{{ item }}"
        instance_type: t2.micro
        image_id: ami-12345
        vpc_subnet_id: "{{ subnet.subnet.id }}"
        security_group: default
        key_name: mykey
        count: 3
      loop: "{{ range(1, 4) | list }}"
```

#### **Docker Management**
```yaml
---
- name: Manage Docker containers
  hosts: docker_hosts
  tasks:
    - name: Ensure Docker is installed
      apt:
        name: docker.io
        state: present
    
    - name: Pull image
      docker_image:
        name: nginx
        tag: latest
        source: pull
    
    - name: Run container
      docker_container:
        name: nginx
        image: nginx:latest
        state: started
        restart_policy: always
        ports:
          - "80:80"
        volumes:
          - /data/nginx:/usr/share/nginx/html
```

#### **Kubernetes Integration**
```yaml
---
- name: Deploy to Kubernetes
  hosts: localhost
  collections:
    - kubernetes.core
  
  tasks:
    - name: Create namespace
      k8s:
        name: production
        api_version: v1
        kind: Namespace
        state: present
    
    - name: Deploy application
      k8s:
        state: present
        definition: "{{ lookup('file', 'k8s/deployment.yaml') }}"
    
    - name: Deploy with Helm
      helm:
        name: myapp
        chart_ref: ./myapp-chart
        release_namespace: production
        values:
          replicaCount: 3
```

---

### **Week 9-10: Security & Vault**

#### **Ansible Vault**
```bash
# Create encrypted file
ansible-vault create secrets.yaml

# Edit encrypted file
ansible-vault edit secrets.yaml

# Encrypt existing file
ansible-vault encrypt vars.yaml

# Decrypt file
ansible-vault decrypt vars.yaml

# View encrypted file
ansible-vault view secrets.yaml

# Run playbook with vault
ansible-playbook playbook.yaml --ask-vault-pass

# Use vault password file
ansible-playbook playbook.yaml --vault-password-file ~/.vault_pass
```

**Vault in Playbooks:**
```yaml
---
- name: Deploy with secrets
  hosts: webservers
  vars_files:
    - secrets.yaml  # Encrypted file
  
  tasks:
    - name: Configure database
      template:
        src: db.conf.j2
        dest: /etc/app/db.conf
      vars:
        db_password: "{{ vault_db_password }}"
```

---

### **Week 11-12: CI/CD & Production**

#### **Jenkins Integration**
```groovy
pipeline {
    agent any
    stages {
        stage('Deploy Infrastructure') {
            steps {
                ansiblePlaybook(
                    playbook: 'site.yaml',
                    inventory: 'inventory/production',
                    credentialsId: 'ansible-ssh',
                    extras: '-e env=production',
                    colorized: true
                )
            }
        }
    }
}
```

#### **Production Best Practices**
- Idempotency
- Error handling
- Testing
- Documentation
- Version control
- Secrets management

---

**Complete Ansible mastery in 12 weeks! 🎓⚙️**

For more resources:
- [Quick Reference](ansible-quick-reference.md)
- [Hands-On Exercises](ansible-hands-on-exercises.md)
- [Troubleshooting Guide](ansible-troubleshooting-guide.md)
- [Interview Questions](ansible-interview-questions.md)


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

