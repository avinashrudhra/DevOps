# Linux Learning Roadmap

Complete 12-week curriculum to master Linux for DevOps and system administration.

---

## 🎯 Learning Objectives

By completing this roadmap, you will:
- ✅ Master Linux command line and shell
- ✅ Understand system administration
- ✅ Write efficient shell scripts
- ✅ Manage services and processes
- ✅ Configure networking
- ✅ Implement security best practices
- ✅ Optimize system performance
- ✅ Troubleshoot production issues

**Target Audience:** DevOps engineers, SysAdmins, Software engineers (3-7+ years experience)

---

## Week 1: Linux Fundamentals

### Day 1-2: Introduction & Installation
- Linux history and distributions
- Choosing a distribution (Ubuntu, CentOS, Debian)
- Installation methods (VirtualBox, WSL2, Cloud)
- Terminal basics

**Practice:**
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install build-essential

# CentOS/RHEL
sudo yum groupinstall "Development Tools"

# System info
uname -a
cat /etc/os-release
hostnamectl
```

### Day 3-4: File System & Navigation
- File system hierarchy
- Navigation commands
- File operations
- Working with directories

**Commands:**
```bash
ls -la                  # List all files
cd /path               # Change directory
pwd                    # Print working directory
tree /etc              # Directory tree

cp -r source dest      # Copy recursively
mv old new            # Move/rename
rm -rf directory      # Remove forcefully
mkdir -p path/to/dir  # Create nested dirs
```

### Day 5-7: File Viewing & Editing
- cat, less, more, head, tail
- vi/vim basics
- nano editor
- File searching

**Vi/Vim Essentials:**
```
# Command mode
i           # Insert mode
ESC         # Back to command mode
:w          # Save
:q          # Quit
:wq or :x   # Save and quit
:q!         # Quit without saving
/pattern    # Search
dd          # Delete line
yy          # Copy line
p           # Paste

# Navigation
h j k l     # Left down up right
0           # Start of line
$           # End of line
gg          # Start of file
G           # End of file
```

---

## Week 2: Users, Permissions & Processes

### Day 8-10: User Management
```bash
# User operations
useradd username
usermod -aG sudo username
passwd username
userdel username

# View users
cat /etc/passwd
id username
who
w
last

# Groups
groupadd developers
usermod -aG developers john
groups username
```

### Day 11-12: File Permissions
```bash
# Permission structure
-rw-r--r--  # user:rw- group:r-- others:r--

# Numeric method
chmod 644 file   # rw-r--r--
chmod 755 file   # rwxr-xr-x
chmod 700 file   # rwx------

# Symbolic method
chmod u+x file   # Add execute for user
chmod g-w file   # Remove write for group
chmod o=r file   # Set others to read only

# Ownership
chown user:group file
chown -R user:group directory

# Special permissions
chmod +t directory     # Sticky bit
chmod u+s file         # SUID
chmod g+s directory    # SGID
```

### Day 13-14: Process Management
```bash
# View processes
ps aux
ps -ef
pstree
top
htop

# Process control
kill PID
kill -9 PID
killall process_name
pkill pattern

# Background jobs
command &
jobs
fg %1
bg %1
nohup command &

# Process priority
nice -n 10 command
renice -n 5 -p PID
```

---

## Week 3: Package Management & Services

### Day 15-17: Package Management

**Ubuntu/Debian (APT):**
```bash
apt update
apt upgrade
apt dist-upgrade
apt install package
apt remove package
apt purge package
apt autoremove
apt search package
apt show package
apt list --installed
```

**RHEL/CentOS (YUM/DNF):**
```bash
yum update
yum install package
yum remove package
yum search package
yum info package
yum list installed

# DNF (Fedora, RHEL 8+)
dnf update
dnf install package
```

### Day 18-20: Systemd Services
```bash
# Service management
systemctl start service
systemctl stop service
systemctl restart service
systemctl reload service
systemctl status service
systemctl enable service
systemctl disable service

# Service inspection
systemctl list-units --type=service
systemctl list-unit-files
systemctl cat service
systemctl show service

# Logs
journalctl -u service
journalctl -u service -f
journalctl -u service --since "1 hour ago"
journalctl -p err
journalctl --disk-usage

# System targets
systemctl get-default
systemctl set-default multi-user.target
systemctl isolate graphical.target
```

### Day 21: Creating Services
```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Application
After=network.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/start.sh
ExecStop=/opt/myapp/stop.sh
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload
systemctl enable myapp
systemctl start myapp
```

---

## Week 4: Shell Scripting

### Day 22-24: Bash Basics
```bash
#!/bin/bash

# Variables
NAME="DevOps"
VERSION=1.0
READONLY=readonly_value

# Command substitution
CURRENT_DATE=$(date +%Y-%m-%d)
HOSTNAME=`hostname`

# Environment variables
export PATH=$PATH:/custom/path
echo $HOME
echo $USER

# User input
read -p "Enter name: " NAME
read -s -p "Enter password: " PASS

# Arguments
echo "Script: $0"
echo "Args: $@"
echo "Count: $#"
echo "First: $1"
```

### Day 25-26: Control Structures
```bash
# If statements
if [ "$1" == "start" ]; then
    echo "Starting..."
elif [ "$1" == "stop" ]; then
    echo "Stopping..."
else
    echo "Unknown command"
fi

# Test operators
# String: =, !=, -z (empty), -n (not empty)
# Numeric: -eq, -ne, -lt, -le, -gt, -ge
# File: -f (file), -d (directory), -e (exists), -r (readable)

# Case statement
case "$1" in
    start)
        start_service
        ;;
    stop)
        stop_service
        ;;
    restart)
        stop_service
        start_service
        ;;
    *)
        echo "Usage: $0 {start|stop|restart}"
        exit 1
        ;;
esac

# Loops
for i in {1..10}; do
    echo "Number: $i"
done

for file in *.txt; do
    echo "Processing $file"
done

while read line; do
    echo "Line: $line"
done < file.txt

counter=0
while [ $counter -lt 10 ]; do
    echo $counter
    ((counter++))
done
```

### Day 27-28: Functions & Advanced
```bash
# Functions
function backup_db() {
    local DB_NAME=$1
    local BACKUP_DIR=$2
    echo "Backing up $DB_NAME to $BACKUP_DIR"
    # Backup logic
    return 0
}

backup_db "mydb" "/backup"

# Arrays
SERVERS=("web1" "web2" "web3")
echo ${SERVERS[0]}
echo ${SERVERS[@]}
echo ${#SERVERS[@]}

for server in "${SERVERS[@]}"; do
    echo "Server: $server"
done

# Error handling
set -e  # Exit on error
set -u  # Exit on undefined variable
set -o pipefail

trap 'echo "Error on line $LINENO"' ERR
trap 'cleanup' EXIT

# Debugging
set -x  # Print commands
set -v  # Print input lines
```

---

## Week 5-6: Networking & Security

### Networking Fundamentals
```bash
# Network configuration
ip addr show
ip route show
ip link show

# DNS
dig example.com
nslookup example.com
cat /etc/resolv.conf

# Connectivity
ping -c 4 example.com
traceroute example.com
mtr example.com

# Ports
netstat -tuln
ss -tuln
lsof -i :80
nmap localhost

# Firewall (UFW)
ufw status
ufw allow 22/tcp
ufw deny 80/tcp
ufw enable

# Firewall (firewalld)
firewall-cmd --list-all
firewall-cmd --add-port=80/tcp --permanent
firewall-cmd --reload
```

### Security Hardening
```bash
# SSH hardening
# /etc/ssh/sshd_config
PermitRootLogin no
PasswordAuthentication no
Port 2222
AllowUsers user1 user2

# Fail2ban
apt install fail2ban
systemctl enable fail2ban

# Update system
apt update && apt upgrade
yum update

# Remove unnecessary packages
apt autoremove

# Check open ports
ss -tuln
netstat -tuln

# SELinux
getenforce
sestatus
semanage port -l
```

---

## Week 7-8: System Monitoring & Performance

### Monitoring Commands
```bash
# System resources
top
htop
atop
glances

# Memory
free -h
vmstat 1
cat /proc/meminfo

# Disk
df -h
du -sh /*
iotop
iostat

# CPU
mpstat
sar -u 1 10
lscpu

# Network
iftop
nethogs
bmon

# Logs
tail -f /var/log/syslog
journalctl -f
dmesg -T
```

### Performance Tuning
```bash
# Kernel parameters
sysctl -a
sysctl -w net.ipv4.ip_forward=1

# /etc/sysctl.conf
net.ipv4.tcp_fin_timeout = 30
net.core.somaxconn = 1024
vm.swappiness = 10

# Apply
sysctl -p

# Limits
# /etc/security/limits.conf
* soft nofile 65535
* hard nofile 65535

ulimit -n
```

---

## Week 9-10: Advanced Topics

### Disk Management
```bash
# Partitions
fdisk -l
parted /dev/sda print
lsblk

# LVM
pvcreate /dev/sdb
vgcreate vg01 /dev/sdb
lvcreate -L 10G -n lv01 vg01
mkfs.ext4 /dev/vg01/lv01

# Mount
mount /dev/sdb1 /mnt
umount /mnt

# /etc/fstab
/dev/sdb1  /data  ext4  defaults  0  2
UUID=xxx   /data  ext4  defaults  0  2

# RAID
mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
```

### Automation & Cron
```bash
# Cron syntax
# * * * * * command
# │ │ │ │ │
# │ │ │ │ └─── Day of week (0-7)
# │ │ │ └───── Month (1-12)
# │ │ └─────── Day of month (1-31)
# │ └───────── Hour (0-23)
# └─────────── Minute (0-59)

# Examples
0 0 * * * /backup.sh           # Daily at midnight
0 */6 * * * /check.sh          # Every 6 hours
*/15 * * * * /monitor.sh       # Every 15 minutes
0 9 * * 1-5 /workday.sh        # Weekdays at 9am

# Edit crontab
crontab -e
crontab -l
```

---

## Week 11-12: Production Ready

### High Availability
- Load balancing
- Failover
- Clustering
- Backup strategies

### Containerization
```bash
# Docker on Linux
systemctl start docker
docker run -d nginx
docker ps
```

### Configuration Management
```bash
# Ansible
ansible-playbook -i hosts site.yaml
```

### Monitoring Stack
- Prometheus + Grafana
- ELK Stack
- Nagios/Zabbix

---

**Complete Linux mastery in 12 weeks! 🐧🎓**

For more resources:
- [Quick Reference](linux-quick-reference.md)
- [Hands-On Exercises](linux-hands-on-exercises.md)
- [Troubleshooting Guide](linux-troubleshooting-guide.md)
- [Interview Questions](linux-interview-questions.md)

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

