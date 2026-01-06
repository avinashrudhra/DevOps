# Linux - The Foundation of DevOps

Complete guide to mastering Linux for DevOps, system administration, and infrastructure management.

---

## 🐧 What is Linux?

**Linux** is an open-source Unix-like operating system kernel that powers:
- 96.3% of the world's top 1 million servers
- 100% of the top 500 supercomputers
- Android (3 billion devices)
- Cloud infrastructure (AWS, Azure, GCP)
- Containers (Docker, Kubernetes)
- DevOps tools (Jenkins, GitLab, etc.)

**Why Linux for DevOps?**
- ✅ **Open Source**: Free, customizable, community-driven
- ✅ **Stability**: Runs for years without reboot
- ✅ **Security**: Robust permission system, SELinux
- ✅ **Performance**: Lightweight, efficient
- ✅ **Automation**: Powerful shell scripting
- ✅ **Package Management**: Easy software installation
- ✅ **Remote Management**: SSH, systemd
- ✅ **Container Native**: Docker, Kubernetes foundation

---

## 🎯 Linux in DevOps Ecosystem

```
┌─────────────────────────────────────────────┐
│            Linux Operating System            │
│         (The Foundation Layer)               │
└─────────────┬───────────────────────────────┘
              │ Runs Everything
    ┌─────────┴─────────┬─────────────┐
    │                   │             │
    ▼                   ▼             ▼
┌─────────┐      ┌─────────┐    ┌─────────┐
│  Docker │      │ Jenkins │    │ Ansible │
│Container│      │  CI/CD  │    │ Config  │
└─────────┘      └─────────┘    └─────────┘
    │                   │             │
    └─────────┬─────────┴─────────────┘
              ▼
    ┌──────────────────┐
    │   Kubernetes     │
    │  Orchestration   │
    └──────────────────┘
```

---

## 🚀 Quick Start

### **Essential Commands**

```bash
# System Information
uname -a                    # System information
hostnamectl                 # System details
lsb_release -a             # Distribution info
cat /etc/os-release        # OS release info

# File Operations
ls -la                     # List files
cd /path/to/directory      # Change directory
pwd                        # Print working directory
cp source dest             # Copy
mv source dest             # Move/rename
rm file                    # Remove file
mkdir directory            # Create directory
touch file                 # Create empty file

# File Viewing
cat file                   # View file
less file                  # Page through file
head file                  # First 10 lines
tail -f file              # Follow file (logs)
grep pattern file          # Search in file

# Permissions
chmod 755 file            # Change permissions
chown user:group file     # Change ownership
ls -l                     # View permissions

# Process Management
ps aux                    # List all processes
top                       # Real-time processes
htop                      # Better top
kill PID                  # Kill process
killall name              # Kill by name
systemctl status service  # Check service

# Package Management (Ubuntu/Debian)
apt update                # Update package list
apt upgrade               # Upgrade packages
apt install package       # Install package
apt remove package        # Remove package

# Package Management (RHEL/CentOS)
yum update                # Update packages
yum install package       # Install package
yum remove package        # Remove package

# Networking
ip addr show              # Show IP addresses
ping host                 # Test connectivity
curl url                  # HTTP request
wget url                  # Download file
netstat -tuln            # Show listening ports
ss -tuln                 # Modern netstat

# Disk Usage
df -h                    # Disk space
du -sh directory         # Directory size
free -h                  # Memory usage

# User Management
useradd username         # Add user
usermod -aG group user   # Add to group
passwd username          # Set password
sudo command             # Run as root
su - username            # Switch user
```

---

## 📦 Core Concepts

### **1. File System Hierarchy**

```
/                          # Root directory
├── bin/                   # Essential user binaries
├── boot/                  # Boot loader files
├── dev/                   # Device files
├── etc/                   # System configuration
│   ├── passwd            # User accounts
│   ├── group             # Group information
│   ├── hosts             # Host name mappings
│   └── systemd/          # Systemd configuration
├── home/                  # User home directories
│   └── username/         # User's home
├── lib/                   # System libraries
├── media/                 # Removable media
├── mnt/                   # Temporary mounts
├── opt/                   # Optional software
├── proc/                  # Process information
├── root/                  # Root user home
├── run/                   # Runtime data
├── sbin/                  # System binaries
├── srv/                   # Service data
├── sys/                   # System information
├── tmp/                   # Temporary files
├── usr/                   # User programs
│   ├── bin/              # User binaries
│   ├── lib/              # Libraries
│   ├── local/            # Local software
│   └── share/            # Shared data
└── var/                   # Variable data
    ├── log/              # Log files
    ├── www/              # Web content
    └── tmp/              # Temporary files
```

### **2. File Permissions**

```bash
# Permission format: rwxrwxrwx (user group others)
# r = read (4), w = write (2), x = execute (1)

-rw-r--r--  1 user group  1234 Jan 1 12:00 file.txt
│││││││││
│││││││││
│││││││└└─── Others: r-- (read only) = 4
││││││└───── Group: r-- (read only) = 4
│││││└────── User: rw- (read/write) = 6
││││└─────── Number of hard links
│││└──────── Owner
││└───────── Group
│└────────── Size in bytes
└─────────── File type (- = file, d = directory, l = link)

# Common permissions:
chmod 644 file    # rw-r--r-- (files)
chmod 755 file    # rwxr-xr-x (executables)
chmod 700 file    # rwx------ (private)
chmod 777 file    # rwxrwxrwx (all access - avoid!)

# Symbolic method:
chmod u+x file    # Add execute for user
chmod g-w file    # Remove write for group
chmod o=r file    # Set others to read only
chmod a+x file    # Add execute for all
```

### **3. Process Management**

```bash
# View processes
ps aux | grep nginx
pgrep nginx
pidof nginx

# Process states
# R = Running
# S = Sleeping
# D = Uninterruptible sleep
# Z = Zombie
# T = Stopped

# Process priority
nice -n 10 command          # Start with lower priority
renice -n 5 -p PID          # Change priority

# Background jobs
command &                    # Run in background
jobs                        # List jobs
fg %1                       # Bring to foreground
bg %1                       # Send to background
Ctrl+Z                      # Suspend current job

# Kill signals
kill -15 PID               # SIGTERM (graceful)
kill -9 PID                # SIGKILL (force)
kill -HUP PID              # SIGHUP (reload config)
pkill -9 nginx             # Kill by name
```

### **4. Systemd Services**

```bash
# Service management
systemctl start nginx       # Start service
systemctl stop nginx        # Stop service
systemctl restart nginx     # Restart service
systemctl reload nginx      # Reload configuration
systemctl status nginx      # Check status
systemctl enable nginx      # Enable at boot
systemctl disable nginx     # Disable at boot

# Service inspection
systemctl list-units --type=service
systemctl list-unit-files --type=service
systemctl cat nginx         # Show service file
journalctl -u nginx         # View service logs
journalctl -u nginx -f      # Follow logs

# System management
systemctl reboot            # Reboot system
systemctl poweroff          # Shutdown
systemctl suspend           # Suspend
```

---

## 🔧 Essential Skills

### **1. Text Processing**

```bash
# grep - search text
grep "error" /var/log/syslog
grep -i "error" file              # Case insensitive
grep -r "pattern" /path           # Recursive
grep -v "pattern" file            # Invert match
grep -E "pattern1|pattern2" file  # Extended regex

# sed - stream editor
sed 's/old/new/' file             # Replace first occurrence
sed 's/old/new/g' file            # Replace all
sed -i 's/old/new/g' file         # Edit in place
sed '/pattern/d' file             # Delete lines matching

# awk - text processing
awk '{print $1}' file             # Print first column
awk -F: '{print $1}' /etc/passwd  # Custom delimiter
awk '$3 > 1000' file              # Conditional
awk '{sum+=$1} END {print sum}'   # Sum column

# cut - cut columns
cut -d: -f1 /etc/passwd           # Get usernames
cut -c1-10 file                   # First 10 characters

# sort & uniq
sort file                         # Sort lines
sort -n file                      # Numeric sort
sort -r file                      # Reverse sort
sort file | uniq                  # Remove duplicates
sort file | uniq -c               # Count occurrences

# tr - translate characters
echo "hello" | tr 'a-z' 'A-Z'     # Uppercase
tr -d '\r' < file                 # Delete characters
```

### **2. Shell Scripting**

```bash
#!/bin/bash
# Basic script template

# Variables
NAME="DevOps"
VERSION=1.0

# Functions
function deploy() {
    echo "Deploying $1"
    # Deployment logic
}

# Conditionals
if [ "$VERSION" -gt "0.9" ]; then
    echo "Version OK"
elif [ "$VERSION" -eq "0.9" ]; then
    echo "Version acceptable"
else
    echo "Version too old"
fi

# Loops
for server in web1 web2 web3; do
    echo "Configuring $server"
done

while read line; do
    echo "Processing: $line"
done < file.txt

# Arrays
SERVERS=("web1" "web2" "web3")
for server in "${SERVERS[@]}"; do
    ssh $server "uptime"
done

# Error handling
set -e  # Exit on error
set -u  # Exit on undefined variable
set -o pipefail  # Exit on pipe failure

# Command line arguments
echo "Script: $0"
echo "First arg: $1"
echo "All args: $@"
echo "Arg count: $#"
```

### **3. Networking**

```bash
# Network configuration
ip addr show                      # Show IP addresses
ip route show                     # Show routes
ip link show                      # Show interfaces

# DNS
dig example.com                   # DNS lookup
nslookup example.com              # DNS query
host example.com                  # Simple DNS lookup

# Connectivity
ping -c 4 example.com            # Ping 4 times
traceroute example.com           # Trace route
mtr example.com                  # Network diagnostics

# Ports and connections
netstat -tuln                    # Listening ports
ss -tuln                         # Modern netstat
lsof -i :80                      # Process on port 80
nc -zv host 80                   # Port scan

# HTTP
curl -I https://example.com      # Headers only
curl -X POST -d "data" url       # POST request
wget https://example.com/file    # Download file

# Firewall (UFW)
ufw status                       # Check status
ufw allow 22/tcp                 # Allow SSH
ufw deny 80/tcp                  # Deny HTTP
ufw enable                       # Enable firewall

# Firewall (firewalld)
firewall-cmd --list-all
firewall-cmd --add-port=80/tcp --permanent
firewall-cmd --reload
```

---

## 🔐 Security Best Practices

```bash
# User security
passwd                           # Change password
passwd -l username               # Lock account
chmod 600 ~/.ssh/id_rsa         # Secure private key
chmod 644 ~/.ssh/id_rsa.pub     # Public key permissions

# SSH hardening (/etc/ssh/sshd_config)
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
Port 2222

# Sudo configuration (/etc/sudoers)
# Use visudo to edit
username ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx

# File system security
find / -perm -4000              # Find SUID files
find / -perm -2000              # Find SGID files
find / -type f -perm 777        # World-writable files

# SELinux
getenforce                      # Check status
setenforce 0                    # Disable temporarily
sestatus                        # Detailed status
```

---

## 📚 Learning Resources

### **📖 Documentation**
1. [Linux Learning Roadmap](linux-learning-roadmap.md) - 12-week structured curriculum
2. [Linux Quick Reference](linux-quick-reference.md) - Commands and patterns
3. [Hands-On Exercises](linux-hands-on-exercises.md) - 40+ practical labs
4. [Troubleshooting Guide](linux-troubleshooting-guide.md) - Common issues
5. [Interview Questions](linux-interview-questions.md) - 100+ questions

### **🔗 Official Resources**
- [Linux Foundation](https://www.linuxfoundation.org/)
- [The Linux Documentation Project](https://tldp.org/)
- [Red Hat Documentation](https://access.redhat.com/documentation/)
- [Ubuntu Documentation](https://help.ubuntu.com/)
- [Arch Wiki](https://wiki.archlinux.org/) - Excellent for all distros

---

## 🎯 Linux for DevOps Tools

### **All DevOps Tools Run on Linux:**

```bash
# Git on Linux
sudo apt install git
git config --global user.name "Name"

# Docker on Linux
curl -fsSL https://get.docker.com | sh
systemctl start docker

# Kubernetes on Linux
kubectl get nodes
kubeadm init

# Jenkins on Linux
java -jar jenkins.war
systemctl start jenkins

# Ansible on Linux
apt install ansible
ansible-playbook playbook.yaml

# Maven on Linux
apt install maven
mvn clean install
```

---

## 🎓 Next Steps

1. **Start Learning**: Follow the [Learning Roadmap](linux-learning-roadmap.md)
2. **Practice**: Complete [Hands-On Exercises](linux-hands-on-exercises.md)
3. **Reference**: Use [Quick Reference](linux-quick-reference.md) guide
4. **Troubleshoot**: Check [Troubleshooting Guide](linux-troubleshooting-guide.md)
5. **Interview Prep**: Study [Interview Questions](linux-interview-questions.md)

---

**Master Linux - The Foundation of DevOps! 🐧💪🚀**

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

