# Ansible Interview Questions

80+ comprehensive interview questions for experienced professionals (5-7+ years) focusing on production Ansible automation.

---

## Table of Contents
1. [Fundamentals](#fundamentals)
2. [Architecture & Design](#architecture--design)
3. [Playbooks & Tasks](#playbooks--tasks)
4. [Inventory Management](#inventory-management)
5. [Variables & Facts](#variables--facts)
6. [Roles & Collections](#roles--collections)
7. [Templates & Files](#templates--files)
8. [Security & Vault](#security--vault)
9. [Performance & Optimization](#performance--optimization)
10. [Production Scenarios](#production-scenarios)

---

## Fundamentals

### Q1: What is Ansible and why would you choose it over other configuration management tools?

**Answer:**

**Ansible** is an agentless, open-source IT automation tool for configuration management, application deployment, and orchestration.

**Key Characteristics:**

**1. Agentless Architecture:**
```
Traditional (Puppet/Chef)    Ansible
┌──────────────┐            ┌──────────────┐
│ Control Node │            │ Control Node │
│   (Server)   │            │     SSH      │
└──────┬───────┘            └──────┬───────┘
       │                           │
       ▼                           ▼
┌──────────────┐            ┌──────────────┐
│ Agent Running│            │  No Agent    │
│  on Target   │            │  Required    │
└──────────────┘            └──────────────┘
```

**2. Simple Syntax:**
```yaml
# Ansible (YAML)
---
- name: Install Nginx
  apt:
    name: nginx
    state: present

# vs Puppet (Ruby DSL)
package { 'nginx':
  ensure => 'present',
}

# vs Chef (Ruby)
package 'nginx' do
  action :install
end
```

**Comparison:**

| Feature | Ansible | Puppet | Chef | SaltStack |
|---------|---------|--------|------|-----------|
| **Agent** | No | Yes | Yes | Yes (optional) |
| **Language** | YAML | Ruby DSL | Ruby | YAML/Python |
| **Learning Curve** | Easy | Moderate | Hard | Moderate |
| **Push/Pull** | Push | Pull | Pull | Both |
| **Setup Complexity** | Low | High | High | Moderate |
| **Execution Model** | Sequential | Declarative | Procedural | Event-driven |

**When to Choose Ansible:**

✅ **Advantages:**
- **No agent installation**: Uses SSH
- **Simple to learn**: YAML syntax
- **Quick setup**: Minutes to start
- **Agentless**: Lower overhead
- **Idempotent**: Safe to run multiple times
- **Large module library**: 3000+ modules
- **Strong community**: Red Hat backed

❌ **Limitations:**
- **SSH overhead**: Can be slower than agent-based
- **Limited Windows support**: PowerShell remoting required
- **No central management**: Unless using Tower/AWX
- **Sequential execution**: By default

**Production Use Cases:**
```yaml
# Configuration Management
- name: Ensure standardized server configuration
  hosts: all
  roles:
    - baseline-security
    - monitoring
    - log-aggregation

# Application Deployment
- name: Deploy microservices
  hosts: app_servers
  serial: 2  # Rolling deployment
  roles:
    - { role: deploy-app, version: "1.2.0" }

# Infrastructure Provisioning
- name: Provision cloud resources
  hosts: localhost
  tasks:
    - amazon.aws.ec2_instance:
        name: web-{{ item }}
        instance_type: t2.micro
      loop: "{{ range(1, 11) | list }}"
```

---

### Q2: Explain Ansible's architecture and execution model

**Answer:**

**Architecture Components:**

```
┌─────────────────────────────────────────────┐
│         Control Node (Ansible Host)          │
│                                              │
│  ┌──────────────────────────────────────┐  │
│  │  Ansible Core                         │  │
│  │  - Inventory                          │  │
│  │  - Playbooks                          │  │
│  │  - Modules                            │  │
│  │  - Plugins                            │  │
│  └──────────────┬───────────────────────┘  │
│                 │                            │
└─────────────────┼────────────────────────────┘
                  │ SSH/WinRM
                  │
        ┌─────────┴──────────┬────────────┐
        │                    │            │
        ▼                    ▼            ▼
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│ Managed Node1 │   │ Managed Node2 │   │ Managed Node3 │
│  (No Agent)   │   │  (No Agent)   │   │  (No Agent)   │
└───────────────┘   └───────────────┘   └───────────────┘
```

**Execution Flow:**

**1. Inventory Loading:**
```bash
# Ansible reads inventory
/etc/ansible/hosts
./inventory.ini
./inventory/production/hosts
dynamic_inventory.py
```

**2. Playbook Parsing:**
```yaml
# Ansible parses YAML playbook
---
- name: Configure web servers
  hosts: webservers
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
```

**3. Connection Establishment:**
```
Control Node ────SSH───► Managed Node
              (Port 22)
```

**4. Module Transfer:**
```
1. Ansible generates Python module code
2. Transfers to /tmp/.ansible/tmp/ on managed node
3. Executes module
4. Returns JSON result
5. Cleans up temporary files
```

**5. Execution Models:**

**Sequential (Default):**
```yaml
hosts: webservers  # web1, web2, web3
tasks:
  - task1  # Runs on web1, then web2, then web3
  - task2  # After task1 completes on all
```

**Parallel (with forks):**
```ini
# ansible.cfg
[defaults]
forks = 10  # Execute on 10 hosts simultaneously
```

**Serial (Rolling Updates):**
```yaml
---
- hosts: webservers
  serial: 2  # 2 hosts at a time
  tasks:
    - name: Deploy application
      # Deploys to 2 servers, then next 2, etc.
```

**Batch Processing:**
```yaml
---
- hosts: webservers
  serial:
    - 1      # First host alone
    - 25%    # Then 25% of remaining
    - 50%    # Then 50% of remaining
```

**Key Concepts:**

**1. Idempotency:**
```yaml
# Safe to run multiple times - same result
- name: Ensure Nginx is installed
  apt:
    name: nginx
    state: present  # Install if not present, skip if present

# vs imperative (not idempotent)
- name: Install Nginx
  command: apt-get install nginx  # Fails if already installed
```

**2. Check Mode (Dry Run):**
```bash
ansible-playbook playbook.yaml --check
# Shows what would change without making changes
```

**3. Diff Mode:**
```bash
ansible-playbook playbook.yaml --check --diff
# Shows exact file changes
```

**4. Module Return Values:**
```json
{
    "changed": true,
    "failed": false,
    "msg": "Package installed successfully",
    "rc": 0,
    "stdout": "nginx installed",
    "stderr": ""
}
```

**Performance Optimizations:**

**1. SSH Pipelining:**
```ini
# ansible.cfg - Reduces SSH connections
[ssh_connection]
pipelining = True  # Requires requiretty=false in /etc/sudoers
```

**2. ControlPersist:**
```ini
[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
# Reuses SSH connections
```

**3. Fact Caching:**
```ini
[defaults]
gathering = smart
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_facts
fact_caching_timeout = 86400  # 24 hours
```

---

### Q3: What are Ansible modules and how do they work?

**Answer:**

**Modules** are discrete units of code that Ansible executes. They are the "tools" in Ansible's toolbox.

**Module Categories:**

**1. System Modules:**
```yaml
# Package management
- apt: name=nginx state=present              # Debian/Ubuntu
- yum: name=httpd state=present              # RHEL/CentOS
- package: name=vim state=present            # OS-agnostic

# Service management
- service: name=nginx state=started enabled=yes
- systemd: name=docker state=restarted daemon_reload=yes

# User management
- user: name=deploy state=present shell=/bin/bash groups=sudo
- group: name=developers state=present

# File operations
- file: path=/var/log/app state=directory mode=0755
- copy: src=file.txt dest=/tmp/file.txt
- template: src=config.j2 dest=/etc/app/config.conf
```

**2. Command Modules:**
```yaml
# command (no shell processing)
- command: /usr/bin/make install
  args:
    chdir: /opt/app
    creates: /opt/app/installed

# shell (with shell processing)
- shell: echo $HOME > /tmp/home.txt

# script (transfer and execute)
- script: ./setup.sh

# raw (no Python required)
- raw: apt-get install python3 -y
```

**3. Cloud Modules:**
```yaml
# AWS
- amazon.aws.ec2_instance:
    name: web-server
    instance_type: t2.micro
    image_id: ami-12345

# Azure
- azure.azcollection.azure_rm_virtualmachine:
    resource_group: myResourceGroup
    name: myVM

# GCP
- google.cloud.gcp_compute_instance:
    name: instance-1
    zone: us-central1-a
```

**4. Container Modules:**
```yaml
# Docker
- docker_container:
    name: nginx
    image: nginx:latest
    state: started
    ports:
      - "80:80"

# Kubernetes
- kubernetes.core.k8s:
    state: present
    definition: "{{ lookup('file', 'deployment.yaml') }}"
```

**Module Execution Workflow:**

```
1. Module Invocation (on control node)
   ↓
2. Ansible generates Python code for module
   ↓
3. Code transferred to managed node via SSH
   ↓
4. Python interpreter executes module
   ↓
5. Module returns JSON output
   ↓
6. Ansible receives and processes result
   ↓
7. Temporary files cleaned up
   ↓
8. Result displayed to user
```

**Module Return Values:**
```yaml
- name: Check service status
  service:
    name: nginx
    state: started
  register: nginx_status

- name: Display result
  debug:
    var: nginx_status
  # Output:
  # {
  #   "changed": true,
  #   "enabled": true,
  #   "name": "nginx",
  #   "state": "started",
  #   "status": {...}
  # }
```

**Custom Modules:**

**Location:**
```
library/
  my_custom_module.py
```

**Basic Structure:**
```python
#!/usr/bin/python

from ansible.module_utils.basic import AnsibleModule

def main():
    module = AnsibleModule(
        argument_spec=dict(
            name=dict(type='str', required=True),
            state=dict(type='str', default='present', choices=['present', 'absent'])
        )
    )
    
    name = module.params['name']
    state = module.params['state']
    
    # Module logic here
    changed = False
    if state == 'present':
        # Do something
        changed = True
    
    # Return result
    module.exit_json(changed=changed, msg=f"Processed {name}")

if __name__ == '__main__':
    main()
```

**Use Custom Module:**
```yaml
- name: Use custom module
  my_custom_module:
    name: example
    state: present
```

**Best Practices:**

1. **Check Mode Support:** Make modules support `--check`
2. **Idempotency:** Ensure safe to run multiple times
3. **Return Values:** Always return changed status
4. **Error Handling:** Proper exception handling
5. **Documentation:** Include module documentation

---

## Production Scenarios

### Q4: Design a complete infrastructure automation solution using Ansible

**Answer:**

**Production Ansible Architecture:**

**Directory Structure:**
```
ansible-infrastructure/
├── ansible.cfg
├── inventory/
│   ├── production/
│   │   ├── hosts
│   │   ├── group_vars/
│   │   │   ├── all.yaml
│   │   │   ├── webservers.yaml
│   │   │   └── databases.yaml
│   │   └── host_vars/
│   │       └── web1.example.com.yaml
│   ├── staging/
│   └── development/
├── playbooks/
│   ├── site.yaml                    # Master playbook
│   ├── webservers.yaml
│   ├── databases.yaml
│   └── monitoring.yaml
├── roles/
│   ├── common/                      # Base configuration
│   ├── nginx/                       # Web server
│   ├── postgresql/                  # Database
│   ├── docker/                      # Container runtime
│   └── monitoring/                  # Prometheus + Grafana
├── group_vars/
│   └── all.yaml                     # Global variables
├── library/                         # Custom modules
├── filter_plugins/                  # Custom filters
├── collections/
│   └── requirements.yaml
└── README.md
```

**Complete Implementation:**

**1. ansible.cfg:**
```ini
[defaults]
inventory = ./inventory
remote_user = ansible
host_key_checking = False
retry_files_enabled = False
gathering = smart
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_facts
fact_caching_timeout = 86400
roles_path = ./roles
collections_path = ./collections
forks = 20
timeout = 30

[privilege_escalation]
become = True
become_method = sudo
become_user = root

[ssh_connection]
pipelining = True
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
control_path = /tmp/ansible-ssh-%%h-%%p-%%r
```

**2. Master Playbook (site.yaml):**
```yaml
---
# Site-wide playbook
- name: Configure all servers with baseline
  hosts: all
  roles:
    - common

- name: Configure web servers
  hosts: webservers
  roles:
    - nginx
    - application

- name: Configure databases
  hosts: databases
  roles:
    - postgresql
    - backup

- name: Configure monitoring
  hosts: monitoring
  roles:
    - prometheus
    - grafana

- name: Configure load balancers
  hosts: loadbalancers
  roles:
    - haproxy
```

**3. Common Role (roles/common/tasks/main.yaml):**
```yaml
---
- name: Update system packages
  apt:
    upgrade: dist
    update_cache: yes
    cache_valid_time: 3600

- name: Install base packages
  apt:
    name: "{{ item }}"
    state: present
  loop:
    - vim
    - git
    - htop
    - curl
    - wget
    - unzip
    - python3-pip

- name: Configure timezone
  timezone:
    name: "{{ timezone | default('UTC') }}"

- name: Configure NTP
  apt:
    name: chrony
    state: present

- name: Start chronyd
  service:
    name: chronyd
    state: started
    enabled: yes

- name: Create ansible user
  user:
    name: ansible
    groups: sudo
    append: yes
    shell: /bin/bash

- name: Set up SSH key
  authorized_key:
    user: ansible
    key: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"

- name: Configure sudo without password
  lineinfile:
    path: /etc/sudoers.d/ansible
    line: 'ansible ALL=(ALL) NOPASSWD: ALL'
    create: yes
    validate: 'visudo -cf %s'

- name: Configure firewall
  ufw:
    rule: allow
    port: "{{ item }}"
    proto: tcp
  loop:
    - "22"    # SSH
    - "80"    # HTTP
    - "443"   # HTTPS

- name: Enable UFW
  ufw:
    state: enabled
```

**4. Application Deployment Role:**
```yaml
# roles/application/tasks/main.yaml
---
- name: Create application user
  user:
    name: "{{ app_user }}"
    home: "/opt/{{ app_name }}"
    shell: /bin/bash

- name: Create application directories
  file:
    path: "{{ item }}"
    state: directory
    owner: "{{ app_user }}"
    mode: '0755'
  loop:
    - "/opt/{{ app_name }}"
    - "/opt/{{ app_name }}/releases"
    - "/opt/{{ app_name }}/shared"
    - "/var/log/{{ app_name }}"

- name: Download application
  get_url:
    url: "{{ app_download_url }}/{{ app_name }}-{{ app_version }}.tar.gz"
    dest: "/tmp/{{ app_name }}-{{ app_version }}.tar.gz"

- name: Extract application
  unarchive:
    src: "/tmp/{{ app_name }}-{{ app_version }}.tar.gz"
    dest: "/opt/{{ app_name }}/releases/"
    remote_src: yes
    creates: "/opt/{{ app_name }}/releases/{{ app_version }}"

- name: Create symlink to current release
  file:
    src: "/opt/{{ app_name }}/releases/{{ app_version }}"
    dest: "/opt/{{ app_name }}/current"
    state: link

- name: Deploy configuration
  template:
    src: app.conf.j2
    dest: "/opt/{{ app_name }}/current/config.conf"
    owner: "{{ app_user }}"
  notify: Restart application

- name: Install systemd service
  template:
    src: app.service.j2
    dest: "/etc/systemd/system/{{ app_name }}.service"
  notify: Restart application

- name: Start application
  systemd:
    name: "{{ app_name }}"
    state: started
    enabled: yes
    daemon_reload: yes
```

**5. CI/CD Integration (Jenkins):**
```groovy
pipeline {
    agent any
    
    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'production'])
        string(name: 'APP_VERSION', defaultValue: 'latest')
        booleanParam(name: 'DEPLOY', defaultValue: false)
    }
    
    stages {
        stage('Lint') {
            steps {
                sh 'ansible-lint playbooks/*.yaml'
            }
        }
        
        stage('Syntax Check') {
            steps {
                sh """
                    ansible-playbook \
                        -i inventory/${params.ENVIRONMENT} \
                        playbooks/site.yaml \
                        --syntax-check
                """
            }
        }
        
        stage('Dry Run') {
            steps {
                sh """
                    ansible-playbook \
                        -i inventory/${params.ENVIRONMENT} \
                        playbooks/site.yaml \
                        -e "app_version=${params.APP_VERSION}" \
                        --check --diff
                """
            }
        }
        
        stage('Deploy') {
            when {
                expression { params.DEPLOY == true }
            }
            steps {
                sh """
                    ansible-playbook \
                        -i inventory/${params.ENVIRONMENT} \
                        playbooks/site.yaml \
                        -e "app_version=${params.APP_VERSION}"
                """
            }
        }
        
        stage('Verify') {
            steps {
                sh """
                    ansible \
                        -i inventory/${params.ENVIRONMENT} \
                        webservers \
                        -m uri \
                        -a "url=http://localhost/health status_code=200"
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

This architecture provides:
- ✅ **Standardization**: Consistent server configuration
- ✅ **Scalability**: Easy to add new servers
- ✅ **Security**: Hardened baseline, firewall rules
- ✅ **Automation**: Full CI/CD integration
- ✅ **Monitoring**: Built-in observability
- ✅ **Disaster Recovery**: Reproducible infrastructure
- ✅ **Multi-Environment**: Dev, staging, production

---

**🎉 Complete Ansible interview preparation! You're ready for senior infrastructure automation roles!** 🚀⚙️

For more resources:
- [Learning Roadmap](ansible-learning-roadmap.md)
- [Quick Reference](ansible-quick-reference.md)
- [Hands-On Exercises](ansible-hands-on-exercises.md)
- [Troubleshooting Guide](ansible-troubleshooting-guide.md)


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

