# Ansible - Infrastructure as Code & Automation

Complete guide to mastering Ansible for configuration management, automation, and infrastructure orchestration.

---

## 📚 What is Ansible?

**Ansible** is an open-source automation tool for configuration management, application deployment, and infrastructure orchestration. It's known for:

- **Agentless**: No agent installation required (uses SSH)
- **Simple**: Human-readable YAML syntax
- **Powerful**: Automate complex multi-tier deployments
- **Idempotent**: Safe to run multiple times
- **Extensible**: 3000+ modules available
- **Open Source**: Free and community-driven

---

## 🎯 Why Ansible?

### **Without Ansible:**
```bash
# SSH to each server
ssh server1
  apt update && apt install nginx
  systemctl start nginx
  
ssh server2
  apt update && apt install nginx
  systemctl start nginx

# Repeat for 100 servers...
```

### **With Ansible:**
```yaml
# playbook.yaml
- hosts: webservers
  tasks:
    - name: Install and start Nginx
      apt:
        name: nginx
        state: present
      notify: Start Nginx
      
# Run once for all servers
ansible-playbook playbook.yaml
```

### **Key Benefits:**
- ✅ **Agentless**: Uses SSH, no agent required
- ✅ **Simple Syntax**: YAML-based, easy to learn
- ✅ **Idempotent**: Safe to run repeatedly
- ✅ **Declarative**: Describe desired state
- ✅ **Cross-Platform**: Linux, Windows, macOS, cloud
- ✅ **Large Module Library**: 3000+ built-in modules
- ✅ **Integration**: Git, Jenkins, Docker, Kubernetes, Cloud providers

---

## 🚀 Quick Start

### **Installation**

```bash
# macOS
brew install ansible

# Ubuntu/Debian
sudo apt update
sudo apt install ansible

# RHEL/CentOS
sudo yum install ansible

# Python pip
pip install ansible

# Verify installation
ansible --version
```

### **First Playbook**

**1. Create Inventory:**
```ini
# inventory.ini
[webservers]
web1.example.com
web2.example.com

[databases]
db1.example.com
```

**2. Create Playbook:**
```yaml
# nginx.yaml
---
- name: Install and configure Nginx
  hosts: webservers
  become: yes
  
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
        update_cache: yes
    
    - name: Start Nginx service
      service:
        name: nginx
        state: started
        enabled: yes
    
    - name: Deploy index.html
      copy:
        content: "<h1>Hello from Ansible!</h1>"
        dest: /var/www/html/index.html
```

**3. Run Playbook:**
```bash
# Test connectivity
ansible webservers -i inventory.ini -m ping

# Run playbook
ansible-playbook -i inventory.ini nginx.yaml

# Check syntax
ansible-playbook nginx.yaml --syntax-check

# Dry run
ansible-playbook nginx.yaml --check
```

---

## 📦 Core Concepts

### **1. Inventory**
Define your hosts and groups:

```ini
# inventory/production
[webservers]
web[1:3].example.com

[databases]
db1.example.com
db2.example.com

[loadbalancers]
lb1.example.com

[production:children]
webservers
databases
loadbalancers

[production:vars]
ansible_user=admin
ansible_port=22
```

**Dynamic Inventory:**
```bash
# AWS EC2 inventory
ansible-inventory -i aws_ec2.yaml --list

# Get from script
./inventory.py --list
```

### **2. Playbooks**
YAML files defining automation:

```yaml
---
- name: Configure web application
  hosts: webservers
  become: yes
  vars:
    app_port: 8080
    app_version: "1.0.0"
  
  tasks:
    - name: Install dependencies
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - python3
        - python3-pip
        - nginx
    
    - name: Deploy application
      git:
        repo: https://github.com/user/app.git
        dest: /opt/myapp
        version: "{{ app_version }}"
    
    - name: Install Python packages
      pip:
        requirements: /opt/myapp/requirements.txt
    
    - name: Start application
      systemd:
        name: myapp
        state: started
        enabled: yes
```

### **3. Modules**
Built-in units of work:

```yaml
# Package management
- apt: name=nginx state=present           # Debian/Ubuntu
- yum: name=httpd state=present           # RHEL/CentOS
- package: name=vim state=present         # Platform agnostic

# Files
- copy: src=file.txt dest=/tmp/file.txt
- template: src=config.j2 dest=/etc/app/config.conf
- file: path=/tmp/dir state=directory mode=0755

# Services
- service: name=nginx state=started enabled=yes
- systemd: name=docker state=restarted daemon_reload=yes

# Commands
- command: /usr/bin/make install
- shell: echo $HOME
- script: ./setup.sh

# Cloud
- ec2: instance_type=t2.micro image=ami-12345
- azure_rm_virtualmachine: name=myvm location=eastus
- gcp_compute_instance: name=instance-1 zone=us-central1-a

# Containers
- docker_container: name=nginx image=nginx:latest
- kubernetes.core.k8s: state=present definition="{{ lookup('file', 'deployment.yaml') }}"
```

### **4. Roles**
Reusable components:

```
roles/
└── nginx/
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

**Using Roles:**
```yaml
---
- name: Configure web servers
  hosts: webservers
  roles:
    - common
    - nginx
    - application
```

### **5. Variables**
Dynamic values:

```yaml
# Playbook variables
vars:
  app_name: myapp
  app_port: 8080

# Inventory variables
[webservers:vars]
http_port=80
domain=example.com

# Group variables (group_vars/webservers.yaml)
---
nginx_port: 80
php_version: "7.4"

# Host variables (host_vars/web1.yaml)
---
server_id: 1
backup_enabled: true

# Using variables
- name: Configure application
  template:
    src: app.conf.j2
    dest: "/etc/{{ app_name }}/config.conf"
```

### **6. Templates (Jinja2)**
Dynamic configuration files:

```jinja2
{# nginx.conf.j2 #}
server {
    listen {{ nginx_port }};
    server_name {{ domain }};
    
    location / {
        proxy_pass http://{{ app_host }}:{{ app_port }};
        proxy_set_header Host $host;
    }
    
    {% if ssl_enabled %}
    ssl_certificate {{ ssl_cert_path }};
    ssl_certificate_key {{ ssl_key_path }};
    {% endif %}
}
```

---

## 🔗 Integration with DevOps Tools

### **With Git**
```bash
# Version control playbooks
git init
git add playbooks/ inventory/ roles/
git commit -m "Add Ansible infrastructure"
git push
```

### **With Jenkins**
```groovy
pipeline {
    agent any
    stages {
        stage('Deploy with Ansible') {
            steps {
                ansiblePlaybook(
                    playbook: 'deploy.yaml',
                    inventory: 'inventory/production',
                    credentialsId: 'ansible-ssh-key',
                    extras: '-e app_version=${BUILD_NUMBER}'
                )
            }
        }
    }
}
```

### **With Docker**
```yaml
- name: Manage Docker containers
  hosts: docker_hosts
  tasks:
    - name: Pull image
      docker_image:
        name: myapp
        tag: "{{ version }}"
        source: pull
    
    - name: Run container
      docker_container:
        name: myapp
        image: "myapp:{{ version }}"
        state: started
        ports:
          - "8080:8080"
```

### **With Kubernetes**
```yaml
- name: Deploy to Kubernetes
  hosts: localhost
  tasks:
    - name: Create namespace
      kubernetes.core.k8s:
        name: production
        api_version: v1
        kind: Namespace
        state: present
    
    - name: Deploy application
      kubernetes.core.k8s:
        state: present
        definition: "{{ lookup('file', 'deployment.yaml') }}"
```

### **With Helm**
```yaml
- name: Deploy Helm chart
  hosts: localhost
  tasks:
    - name: Deploy application
      kubernetes.core.helm:
        name: myapp
        chart_ref: ./myapp-chart
        release_namespace: production
        values:
          replicaCount: 3
          image:
            tag: "{{ app_version }}"
```

---

## 🎯 Common Use Cases

### **1. Server Configuration**
```yaml
- name: Configure servers
  hosts: all
  become: yes
  tasks:
    - name: Update system
      apt:
        upgrade: dist
        update_cache: yes
    
    - name: Install security updates
      apt:
        name: unattended-upgrades
        state: present
    
    - name: Configure firewall
      ufw:
        rule: allow
        port: "{{ item }}"
      loop:
        - "22"
        - "80"
        - "443"
```

### **2. Application Deployment**
```yaml
- name: Deploy application
  hosts: webservers
  vars:
    app_version: "1.0.0"
  tasks:
    - name: Stop application
      systemd:
        name: myapp
        state: stopped
    
    - name: Download new version
      get_url:
        url: "https://releases.example.com/myapp-{{ app_version }}.tar.gz"
        dest: /tmp/
    
    - name: Extract application
      unarchive:
        src: "/tmp/myapp-{{ app_version }}.tar.gz"
        dest: /opt/myapp
        remote_src: yes
    
    - name: Start application
      systemd:
        name: myapp
        state: started
```

### **3. Infrastructure Provisioning**
```yaml
- name: Provision AWS infrastructure
  hosts: localhost
  tasks:
    - name: Create VPC
      amazon.aws.ec2_vpc_net:
        name: production-vpc
        cidr_block: 10.0.0.0/16
        region: us-east-1
    
    - name: Create EC2 instances
      amazon.aws.ec2_instance:
        name: "web-{{ item }}"
        instance_type: t2.micro
        image_id: ami-12345
        count: 3
      loop: "{{ range(1, 4) | list }}"
```

---

## 📚 Learning Resources

### **📖 Documentation**
1. [Ansible Learning Roadmap](ansible-learning-roadmap.md) - 12-week structured curriculum
2. [Ansible Quick Reference](ansible-quick-reference.md) - Commands and patterns
3. [Hands-On Exercises](ansible-hands-on-exercises.md) - 30+ practical labs
4. [Troubleshooting Guide](ansible-troubleshooting-guide.md) - Common issues
5. [Interview Questions](ansible-interview-questions.md) - 80+ questions

### **🔗 Official Resources**
- [Ansible Documentation](https://docs.ansible.com/)
- [Ansible GitHub](https://github.com/ansible/ansible)
- [Ansible Galaxy](https://galaxy.ansible.com/) - Community roles
- [Ansible Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)

---

## 🎓 Next Steps

1. **Start Learning**: Follow the [Learning Roadmap](ansible-learning-roadmap.md)
2. **Practice**: Complete [Hands-On Exercises](ansible-hands-on-exercises.md)
3. **Reference**: Use [Quick Reference](ansible-quick-reference.md) guide
4. **Troubleshoot**: Check [Troubleshooting Guide](ansible-troubleshooting-guide.md)
5. **Interview Prep**: Study [Interview Questions](ansible-interview-questions.md)

---

**Master Infrastructure as Code with Ansible! 🔧⚙️🚀**

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

