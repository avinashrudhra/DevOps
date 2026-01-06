# Ansible Quick Reference

Essential commands, modules, and patterns for Ansible automation.

---

## 📋 Installation

```bash
# macOS
brew install ansible

# Ubuntu/Debian
sudo apt update && sudo apt install ansible

# RHEL/CentOS
sudo yum install ansible

# Python pip
pip install ansible

# Verify
ansible --version
```

---

## 🗂️ Configuration

### **ansible.cfg**
```ini
[defaults]
inventory = ./inventory
remote_user = admin
host_key_checking = False
retry_files_enabled = False
gathering = smart
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_facts
fact_caching_timeout = 86400
roles_path = ./roles:~/.ansible/roles
collections_path = ./collections:~/.ansible/collections

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False

[ssh_connection]
pipelining = True
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
```

---

## 📦 Inventory

### **INI Format**
```ini
# inventory.ini
[webservers]
web1.example.com
web2.example.com ansible_host=192.168.1.10

[databases]
db1.example.com ansible_host=192.168.1.20
db2.example.com ansible_host=192.168.1.21

[loadbalancers]
lb1.example.com

# Group of groups
[production:children]
webservers
databases
loadbalancers

# Group variables
[webservers:vars]
ansible_user=webadmin
http_port=80

[databases:vars]
ansible_user=dbadmin
db_port=5432

# All hosts variables
[all:vars]
ansible_port=22
ansible_python_interpreter=/usr/bin/python3
```

### **YAML Format**
```yaml
# inventory.yaml
all:
  children:
    webservers:
      hosts:
        web1.example.com:
        web2.example.com:
          ansible_host: 192.168.1.10
      vars:
        http_port: 80
    databases:
      hosts:
        db1.example.com:
          ansible_host: 192.168.1.20
        db2.example.com:
          ansible_host: 192.168.1.21
      vars:
        db_port: 5432
    production:
      children:
        webservers:
        databases:
  vars:
    ansible_user: admin
```

---

## ⚡ Ad-Hoc Commands

```bash
# Ping all hosts
ansible all -m ping

# Run command
ansible webservers -m command -a "uptime"

# Run shell command
ansible webservers -m shell -a "ps aux | grep nginx"

# Copy file
ansible webservers -m copy -a "src=/tmp/file dest=/tmp/file"

# Install package
ansible webservers -m apt -a "name=nginx state=present" --become

# Start service
ansible webservers -m service -a "name=nginx state=started" --become

# Gather facts
ansible web1.example.com -m setup

# Gather subset of facts
ansible all -m setup -a "filter=ansible_distribution*"

# Create user
ansible webservers -m user -a "name=deploy state=present" --become

# Synchronize files
ansible webservers -m synchronize -a "src=/local/path dest=/remote/path"
```

---

## 📝 Playbook Syntax

### **Basic Playbook**
```yaml
---
- name: Configure web servers
  hosts: webservers
  become: yes
  vars:
    http_port: 80
  
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
```

### **Multiple Plays**
```yaml
---
- name: Configure web servers
  hosts: webservers
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present

- name: Configure databases
  hosts: databases
  tasks:
    - name: Install PostgreSQL
      apt:
        name: postgresql
        state: present
```

---

## 🔧 Common Modules

### **Package Management**
```yaml
# APT (Debian/Ubuntu)
- apt:
    name: nginx
    state: present
    update_cache: yes

# YUM (RHEL/CentOS)
- yum:
    name: httpd
    state: present

# DNF (Fedora)
- dnf:
    name: nginx
    state: latest

# Package (OS agnostic)
- package:
    name: vim
    state: present
```

### **Service Management**
```yaml
# Service module
- service:
    name: nginx
    state: started
    enabled: yes

# Systemd module
- systemd:
    name: docker
    state: restarted
    daemon_reload: yes
    enabled: yes
```

### **File Operations**
```yaml
# Copy file
- copy:
    src: /local/file.txt
    dest: /remote/file.txt
    owner: root
    group: root
    mode: '0644'

# Template file
- template:
    src: templates/config.j2
    dest: /etc/app/config.conf
    backup: yes

# Create file/directory
- file:
    path: /var/log/myapp
    state: directory
    mode: '0755'

# Symbolic link
- file:
    src: /opt/app/current
    dest: /opt/app/release
    state: link

# Remove file
- file:
    path: /tmp/tempfile
    state: absent

# Line in file
- lineinfile:
    path: /etc/hosts
    line: "192.168.1.10 web1.local"
    create: yes

# Block in file
- blockinfile:
    path: /etc/hosts
    block: |
      192.168.1.10 web1.local
      192.168.1.11 web2.local
```

### **Command Execution**
```yaml
# Command (no shell processing)
- command: /usr/bin/make install
  args:
    chdir: /opt/app
    creates: /opt/app/installed.txt

# Shell (with shell processing)
- shell: echo $HOME > /tmp/home.txt

# Script execution
- script: ./setup.sh

# Raw (no python required)
- raw: apt-get install python3 -y
```

### **User & Group**
```yaml
# Create user
- user:
    name: deploy
    state: present
    shell: /bin/bash
    groups: sudo
    append: yes
    create_home: yes

# Create group
- group:
    name: developers
    state: present

# Set authorized key
- authorized_key:
    user: deploy
    state: present
    key: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
```

### **Git**
```yaml
- git:
    repo: https://github.com/user/repo.git
    dest: /opt/myapp
    version: main
    force: yes
```

### **Docker**
```yaml
# Manage image
- docker_image:
    name: nginx
    tag: latest
    source: pull

# Manage container
- docker_container:
    name: nginx
    image: nginx:latest
    state: started
    restart_policy: always
    ports:
      - "80:80"
    volumes:
      - /data:/usr/share/nginx/html
    env:
      NGINX_HOST: example.com
```

---

## 🎯 Variables

### **Defining Variables**
```yaml
# In playbook
vars:
  http_port: 80
  domain: example.com

# From file
vars_files:
  - vars/common.yaml
  - "vars/{{ env }}.yaml"

# In inventory
[webservers:vars]
http_port=80

# In group_vars/webservers.yaml
---
http_port: 80
domain: example.com

# In host_vars/web1.yaml
---
server_id: 1
```

### **Using Variables**
```yaml
- name: Configure port
  template:
    src: app.conf.j2
    dest: "/etc/app/config.conf"
  vars:
    app_port: "{{ http_port | default(8080) }}"
```

### **Magic Variables**
```yaml
# Host information
{{ inventory_hostname }}
{{ ansible_hostname }}
{{ ansible_host }}

# Group information
{{ groups['webservers'] }}
{{ groups['all'] }}

# Host variables
{{ hostvars['web1']['ansible_host'] }}

# Playbook information
{{ playbook_dir }}
{{ ansible_play_hosts }}
```

---

## 🎨 Jinja2 Templates

### **Basic Syntax**
```jinja2
{# Comment #}
{{ variable }}
{% if condition %}...{% endif %}
{% for item in list %}...{% endfor %}

{# Example #}
server {
    listen {{ http_port }};
    server_name {{ domain }};
    
    {% if ssl_enabled %}
    listen 443 ssl;
    ssl_certificate {{ ssl_cert }};
    {% endif %}
    
    location / {
        proxy_pass http://{{ backend_host }}:{{ backend_port }};
    }
}
```

### **Filters**
```jinja2
{{ name | upper }}
{{ name | lower }}
{{ path | basename }}
{{ path | dirname }}
{{ list | join(', ') }}
{{ dict | to_json }}
{{ dict | to_yaml }}
{{ string | b64encode }}
{{ string | b64decode }}
{{ number | int }}
{{ number | float }}
{{ value | default('default_value') }}
{{ list | length }}
{{ list | first }}
{{ list | last }}
```

---

## 🔁 Loops

```yaml
# Simple loop
- name: Install packages
  apt:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - git
    - vim

# Loop with dict
- name: Create users
  user:
    name: "{{ item.name }}"
    groups: "{{ item.groups }}"
  loop:
    - { name: 'alice', groups: 'sudo' }
    - { name: 'bob', groups: 'developers' }

# Loop with index
- name: Create files
  file:
    path: "/tmp/file{{ item.0 }}.txt"
    state: touch
  loop: "{{ range(1, 6) | list }}"

# Loop with dict items
- name: Set environment variables
  lineinfile:
    path: /etc/environment
    line: "{{ item.key }}={{ item.value }}"
  loop: "{{ env_vars | dict2items }}"
```

---

## 🔀 Conditionals

```yaml
# When condition
- name: Install on Ubuntu
  apt:
    name: nginx
  when: ansible_distribution == "Ubuntu"

# Multiple conditions (AND)
- name: Task
  debug:
    msg: "Runs on Ubuntu 20.04"
  when:
    - ansible_distribution == "Ubuntu"
    - ansible_distribution_version == "20.04"

# OR condition
- name: Task
  debug:
    msg: "Runs on Ubuntu or Debian"
  when: ansible_distribution == "Ubuntu" or ansible_distribution == "Debian"

# Check variable defined
- name: Task
  debug:
    msg: "Variable is defined"
  when: my_var is defined

# Check command result
- name: Check if file exists
  stat:
    path: /tmp/file
  register: file_stat

- name: Create if not exists
  file:
    path: /tmp/file
    state: touch
  when: not file_stat.stat.exists
```

---

## 🎭 Handlers

```yaml
tasks:
  - name: Update nginx config
    template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    notify:
      - Restart Nginx
      - Check Nginx

handlers:
  - name: Restart Nginx
    service:
      name: nginx
      state: restarted
  
  - name: Check Nginx
    command: nginx -t
```

---

## 📦 Roles

### **Using Roles**
```yaml
---
- name: Configure servers
  hosts: webservers
  roles:
    - common
    - { role: nginx, nginx_port: 8080 }
    - role: application
      vars:
        app_version: "1.0.0"
```

### **Creating Role**
```bash
ansible-galaxy init myrole

# Structure
myrole/
├── defaults/main.yaml       # Default variables
├── vars/main.yaml           # Role variables
├── tasks/main.yaml          # Tasks
├── handlers/main.yaml       # Handlers
├── templates/               # Jinja2 templates
├── files/                   # Static files
├── meta/main.yaml           # Role metadata
└── README.md
```

---

## 🚀 Running Playbooks

```bash
# Basic execution
ansible-playbook playbook.yaml

# Specify inventory
ansible-playbook -i inventory.ini playbook.yaml

# Check syntax
ansible-playbook playbook.yaml --syntax-check

# Dry run
ansible-playbook playbook.yaml --check

# Diff mode
ansible-playbook playbook.yaml --check --diff

# Limit to hosts
ansible-playbook playbook.yaml --limit webservers
ansible-playbook playbook.yaml --limit web1.example.com

# Tags
ansible-playbook playbook.yaml --tags "deploy"
ansible-playbook playbook.yaml --skip-tags "deploy"

# Extra variables
ansible-playbook playbook.yaml -e "env=production version=1.0.0"
ansible-playbook playbook.yaml -e @vars/production.yaml

# Verbose output
ansible-playbook playbook.yaml -v    # verbose
ansible-playbook playbook.yaml -vv   # more verbose
ansible-playbook playbook.yaml -vvv  # very verbose
ansible-playbook playbook.yaml -vvvv # connection debugging

# Start at task
ansible-playbook playbook.yaml --start-at-task="Install Nginx"

# List tasks
ansible-playbook playbook.yaml --list-tasks

# List hosts
ansible-playbook playbook.yaml --list-hosts
```

---

## 🔐 Ansible Vault

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

# Rekey (change password)
ansible-vault rekey secrets.yaml

# Run with vault
ansible-playbook playbook.yaml --ask-vault-pass

# Use password file
ansible-playbook playbook.yaml --vault-password-file ~/.vault_pass

# Multiple vault passwords
ansible-playbook playbook.yaml --vault-id dev@~/.vault_dev --vault-id prod@~/.vault_prod
```

---

## 🌐 Cloud Modules

### **AWS**
```yaml
# EC2 instance
- ec2_instance:
    name: web-server
    instance_type: t2.micro
    image_id: ami-12345
    region: us-east-1
    key_name: mykey
    vpc_subnet_id: subnet-12345
    security_group: default

# S3 bucket
- aws_s3:
    bucket: my-bucket
    object: /files/data.txt
    src: /local/data.txt
    mode: put
```

### **Azure**
```yaml
- azure_rm_virtualmachine:
    resource_group: myResourceGroup
    name: myVM
    vm_size: Standard_DS1_v2
    admin_username: azureuser
    ssh_password_enabled: false
    image:
      offer: UbuntuServer
      publisher: Canonical
      sku: 18.04-LTS
      version: latest
```

---

## 🐛 Debugging

```yaml
# Debug module
- name: Print variable
  debug:
    var: my_variable

- name: Print message
  debug:
    msg: "Value is {{ my_var }}"

# Register and debug
- name: Run command
  command: ls /tmp
  register: result

- name: Show result
  debug:
    var: result.stdout_lines

# Assert
- name: Verify condition
  assert:
    that:
      - ansible_distribution == "Ubuntu"
      - ansible_distribution_version >= "20.04"
    fail_msg: "Unsupported OS"
    success_msg: "OS is supported"
```

---

**Quick reference complete! ⚙️✨**

For more resources:
- [Learning Roadmap](ansible-learning-roadmap.md)
- [Hands-On Exercises](ansible-hands-on-exercises.md)
- [Troubleshooting Guide](ansible-troubleshooting-guide.md)
- [Interview Questions](ansible-interview-questions.md)


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

