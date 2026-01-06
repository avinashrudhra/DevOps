# Ansible Troubleshooting Guide

Comprehensive solutions to common Ansible issues in production environments.

---

## Table of Contents
1. [Connection Issues](#connection-issues)
2. [Authentication Problems](#authentication-problems)
3. [Playbook Errors](#playbook-errors)
4. [Module Failures](#module-failures)
5. [Performance Issues](#performance-issues)
6. [Inventory Problems](#inventory-problems)
7. [Variable Issues](#variable-issues)
8. [Template Errors](#template-errors)
9. [Privilege Escalation](#privilege-escalation)
10. [Production Incidents](#production-incidents)

---

## Connection Issues

### Issue 1: SSH Connection Refused

**Symptoms:**
```
UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: ssh: connect to host 192.168.1.10 port 22: Connection refused"}
```

**Diagnosis:**
```bash
# Test SSH manually
ssh admin@192.168.1.10

# Check SSH service on target
systemctl status sshd

# Check port
netstat -tuln | grep 22
```

**Solutions:**

**1. Verify SSH service:**
```bash
# On target host
sudo systemctl start sshd
sudo systemctl enable sshd
```

**2. Check firewall:**
```bash
# Allow SSH
sudo ufw allow 22/tcp
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --reload
```

**3. Verify inventory:**
```ini
# inventory.ini
[webservers]
web1 ansible_host=192.168.1.10 ansible_port=22 ansible_user=admin
```

---

### Issue 2: Host Key Verification Failed

**Symptoms:**
```
fatal: [web1]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: Host key verification failed."}
```

**Solutions:**

**1. Disable host key checking (not recommended for production):**
```ini
# ansible.cfg
[defaults]
host_key_checking = False
```

**2. Add host keys:**
```bash
# Add known hosts
ssh-keyscan -H 192.168.1.10 >> ~/.ssh/known_hosts

# Or connect manually first
ssh admin@192.168.1.10
```

**3. Use ANSIBLE_HOST_KEY_CHECKING:**
```bash
export ANSIBLE_HOST_KEY_CHECKING=False
ansible-playbook playbook.yaml
```

---

### Issue 3: Connection Timeout

**Symptoms:**
```
fatal: [web1]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: ssh: connect to host web1 port 22: Operation timed out"}
```

**Solutions:**

**1. Increase timeout:**
```ini
# ansible.cfg
[defaults]
timeout = 30
```

**2. Check network connectivity:**
```bash
ping web1
traceroute web1
telnet web1 22
```

**3. Use jump host:**
```ini
# inventory.ini
[webservers]
web1 ansible_host=192.168.1.10 ansible_ssh_common_args='-o ProxyCommand="ssh -W %h:%p jumphost.example.com"'
```

---

## Authentication Problems

### Issue 4: Permission Denied (publickey)

**Symptoms:**
```
fatal: [web1]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: admin@192.168.1.10: Permission denied (publickey,password)."}
```

**Solutions:**

**1. Verify SSH key:**
```bash
# Check SSH key
ssh -i ~/.ssh/id_rsa admin@192.168.1.10

# Specify key in inventory
[webservers]
web1 ansible_ssh_private_key_file=~/.ssh/id_rsa ansible_user=admin
```

**2. Use password authentication:**
```bash
# Install sshpass
sudo apt install sshpass

# Use --ask-pass
ansible-playbook playbook.yaml --ask-pass
```

**3. Add key to authorized_keys:**
```bash
# Copy public key
ssh-copy-id -i ~/.ssh/id_rsa.pub admin@192.168.1.10
```

---

### Issue 5: Sudo Password Required

**Symptoms:**
```
fatal: [web1]: FAILED! => {"msg": "Missing sudo password"}
```

**Solutions:**

**1. Provide sudo password:**
```bash
ansible-playbook playbook.yaml --ask-become-pass
```

**2. Configure passwordless sudo:**
```bash
# On target host
sudo visudo
# Add line:
admin ALL=(ALL) NOPASSWD: ALL
```

**3. In playbook:**
```yaml
---
- hosts: webservers
  become: yes
  vars:
    ansible_become_pass: "{{ sudo_password }}"
```

---

## Playbook Errors

### Issue 6: YAML Syntax Error

**Symptoms:**
```
ERROR! We were unable to read either as JSON nor YAML, these are the errors we got from each:
JSON: Expecting value: line 1 column 1 (char 0)

Syntax Error while loading YAML.
  mapping values are not allowed here
```

**Diagnosis:**
```bash
# Validate YAML
ansible-playbook playbook.yaml --syntax-check

# Use linter
ansible-lint playbook.yaml

# Check with yamllint
yamllint playbook.yaml
```

**Common Issues:**

**1. Indentation:**
```yaml
# ❌ Wrong
tasks:
- name: Task 1
  command: echo hello

# ✅ Correct
tasks:
  - name: Task 1
    command: echo hello
```

**2. Quotes:**
```yaml
# ❌ Wrong
name: This has a: colon

# ✅ Correct
name: "This has a: colon"
```

**3. Lists:**
```yaml
# ❌ Wrong
loop:
- item1, item2, item3

# ✅ Correct
loop:
  - item1
  - item2
  - item3
```

---

### Issue 7: Undefined Variable

**Symptoms:**
```
fatal: [web1]: FAILED! => {"msg": "The task includes an option with an undefined variable. The error was: 'app_port' is undefined"}
```

**Solutions:**

**1. Define variable:**
```yaml
# In playbook
vars:
  app_port: 8080

# In group_vars/webservers.yaml
app_port: 8080

# In inventory
[webservers:vars]
app_port=8080
```

**2. Use default:**
```yaml
tasks:
  - name: Configure app
    template:
      src: app.conf.j2
      dest: /etc/app.conf
    vars:
      app_port: "{{ app_port | default(8080) }}"
```

**3. Check variable:**
```yaml
- name: Debug variable
  debug:
    var: app_port

- name: Only when defined
  command: echo {{ app_port }}
  when: app_port is defined
```

---

## Module Failures

### Issue 8: Module Not Found

**Symptoms:**
```
fatal: [web1]: FAILED! => {"msg": "The module docker_container was not found in configured module paths"}
```

**Solutions:**

**1. Install collection:**
```bash
# Install required collection
ansible-galaxy collection install community.docker

# List collections
ansible-galaxy collection list
```

**2. Use FQCN (Fully Qualified Collection Name):**
```yaml
# Instead of
- docker_container:

# Use
- community.docker.docker_container:
```

**3. Install Python module:**
```bash
# On control node and managed hosts
pip install docker

# Or in playbook
- name: Install docker Python library
  pip:
    name: docker
    state: present
```

---

### Issue 9: Command Failed

**Symptoms:**
```
fatal: [web1]: FAILED! => {"changed": true, "cmd": ["systemctl", "start", "myapp"], "msg": "non-zero return code", "rc": 1}
```

**Diagnosis:**
```yaml
- name: Run command and register result
  command: systemctl start myapp
  register: result
  ignore_errors: yes

- name: Debug result
  debug:
    var: result

- name: Show stdout
  debug:
    var: result.stdout_lines

- name: Show stderr
  debug:
    var: result.stderr_lines
```

**Solutions:**

**1. Check for errors:**
```yaml
- name: Check service exists
  stat:
    path: /etc/systemd/system/myapp.service
  register: service_file

- name: Start service only if exists
  systemd:
    name: myapp
    state: started
  when: service_file.stat.exists
```

**2. Handle return codes:**
```yaml
- name: Command with acceptable return codes
  command: /opt/app/check.sh
  register: result
  failed_when:
    - result.rc != 0
    - result.rc != 2  # Also accept rc=2
```

---

## Performance Issues

### Issue 10: Slow Playbook Execution

**Symptoms:**
- Playbooks take very long time
- High CPU on control node
- Gathering facts takes minutes

**Solutions:**

**1. Enable pipelining:**
```ini
# ansible.cfg
[ssh_connection]
pipelining = True
```

**2. Disable gathering facts:**
```yaml
---
- hosts: webservers
  gather_facts: no  # Disable if not needed
```

**3. Smart fact gathering:**
```ini
# ansible.cfg
[defaults]
gathering = smart
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_facts
fact_caching_timeout = 86400
```

**4. Parallel execution:**
```ini
# ansible.cfg
[defaults]
forks = 20  # Increase from default 5
```

**5. Use async:**
```yaml
- name: Long running task
  command: /opt/long-task.sh
  async: 3600  # Run for up to 1 hour
  poll: 0      # Fire and forget

- name: Check on task later
  async_status:
    jid: "{{ long_task.ansible_job_id }}"
  register: job_result
  until: job_result.finished
  retries: 30
  delay: 10
```

---

## Inventory Problems

### Issue 11: Host Not Found

**Symptoms:**
```
[WARNING]: Could not match supplied host pattern, ignoring: webservers
```

**Solutions:**

**1. Verify inventory:**
```bash
# List hosts
ansible-inventory -i inventory.ini --list

# List specific group
ansible webservers -i inventory.ini --list-hosts

# Graph inventory
ansible-inventory -i inventory.ini --graph
```

**2. Check inventory file:**
```ini
# inventory.ini
[webservers]
web1.example.com
web2.example.com

[all:vars]
ansible_user=admin
```

**3. Multiple inventory sources:**
```bash
# Use multiple inventories
ansible-playbook -i inventory1.ini -i inventory2.ini playbook.yaml
```

---

## Variable Issues

### Issue 12: Variable Precedence Problems

**Diagnosis:**
```yaml
- name: Show variable value and source
  debug:
    msg: "app_port is {{ app_port }}"

- name: Show all variables for host
  debug:
    var: hostvars[inventory_hostname]
```

**Variable Precedence (lowest to highest):**
1. role defaults
2. inventory vars
3. inventory group_vars
4. inventory host_vars
5. playbook group_vars
6. playbook host_vars
7. host facts
8. play vars
9. play vars_files
10. task vars
11. extra vars (-e)

**Solution - Use extra vars for overrides:**
```bash
ansible-playbook playbook.yaml -e "app_port=9000"
```

---

## Template Errors

### Issue 13: Jinja2 Template Error

**Symptoms:**
```
fatal: [web1]: FAILED! => {"msg": "AnsibleUndefinedVariable: 'dict object' has no attribute 'nonexistent'"}
```

**Solutions:**

**1. Safe access:**
```jinja2
{# Bad #}
{{ my_dict.key }}

{# Good - with default #}
{{ my_dict.key | default('default_value') }}

{# Good - check if defined #}
{% if my_dict.key is defined %}
{{ my_dict.key }}
{% endif %}
```

**2. Debug template:**
```yaml
- name: Template debug
  template:
    src: config.j2
    dest: /tmp/config-debug.conf
  check_mode: yes
  diff: yes
```

---

## Production Incidents

### Issue 14: Playbook Fails Mid-Execution

**Recovery:**

**1. Use --start-at-task:**
```bash
# Resume from specific task
ansible-playbook playbook.yaml --start-at-task="Install application"
```

**2. Use tags:**
```yaml
tasks:
  - name: Install packages
    apt:
      name: nginx
    tags: install

  - name: Configure
    template:
      src: config.j2
    tags: configure

  - name: Start service
    service:
      name: nginx
    tags: start
```

```bash
# Run only specific tags
ansible-playbook playbook.yaml --tags "configure,start"

# Skip tags
ansible-playbook playbook.yaml --skip-tags "install"
```

**3. Implement idempotency:**
```yaml
- name: Ensure nginx is installed
  apt:
    name: nginx
    state: present  # Idempotent

- name: Ensure config is present
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  # Safe to run multiple times
```

---

## Debugging Commands

```bash
# Verbose output
ansible-playbook playbook.yaml -v    # verbose
ansible-playbook playbook.yaml -vv   # more verbose
ansible-playbook playbook.yaml -vvv  # very verbose
ansible-playbook playbook.yaml -vvvv # connection debug

# Syntax check
ansible-playbook playbook.yaml --syntax-check

# Dry run
ansible-playbook playbook.yaml --check

# Dry run with diff
ansible-playbook playbook.yaml --check --diff

# List tasks
ansible-playbook playbook.yaml --list-tasks

# List hosts
ansible-playbook playbook.yaml --list-hosts

# Step through
ansible-playbook playbook.yaml --step

# Start at task
ansible-playbook playbook.yaml --start-at-task="Task name"
```

---

**Complete Ansible troubleshooting expertise! 🔧⚙️**

For more resources:
- [Learning Roadmap](ansible-learning-roadmap.md)
- [Quick Reference](ansible-quick-reference.md)
- [Hands-On Exercises](ansible-hands-on-exercises.md)
- [Interview Questions](ansible-interview-questions.md)


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

