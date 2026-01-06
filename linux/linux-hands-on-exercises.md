# Linux Hands-On Exercises

40+ practical exercises to master Linux from beginner to expert level.

---

## Beginner Level Exercises

### Exercise 1: Linux Installation & Basic Navigation
**Objective**: Install Linux and navigate the file system

**Tasks:**
1. Install Ubuntu or CentOS (VirtualBox/WSL2)
2. Open terminal
3. Navigate file system
4. Create directories and files

<details>
<summary>Solution</summary>

```bash
# Check system info
uname -a
cat /etc/os-release
hostnamectl

# Navigate
cd /
ls -la
cd /home
pwd
cd ~
pwd

# Create structure
mkdir -p ~/projects/devops/scripts
cd ~/projects/devops
touch README.md
ls -l

# Create multiple files
touch file1.txt file2.txt file3.txt
mkdir logs configs data

# Tree structure
tree ~/projects
```

**Verification:**
```bash
ls -la ~/projects/devops
```
</details>

---

### Exercise 2: File Operations & Permissions
**Objective**: Master file operations and permission management

<details>
<summary>Solution</summary>

```bash
# Create test files
mkdir ~/permission-test
cd ~/permission-test
touch file1.txt file2.txt
echo "Hello Linux" > file1.txt

# Check permissions
ls -l
# -rw-rw-r-- 1 user user 12 Jan 1 12:00 file1.txt

# Modify permissions
chmod 644 file1.txt    # rw-r--r--
chmod 755 file2.txt    # rwxr-xr-x
chmod u+x file1.txt    # Add execute for user
chmod g-w file1.txt    # Remove write for group

# Change ownership
sudo chown root:root file1.txt
sudo chown $USER:$USER file1.txt

# Create with specific permissions
umask 022
touch newfile.txt      # Created with 644

# Special permissions
mkdir shared
chmod +t shared        # Sticky bit
ls -ld shared

# Find files by permission
find ~ -perm 777
find ~ -type f -perm -u+x
```
</details>

---

### Exercise 3: Text Processing
**Objective**: Use grep, sed, awk for text manipulation

<details>
<summary>Solution</summary>

```bash
# Create sample file
cat > users.txt <<EOF
john:developer:1001
jane:manager:1002
bob:developer:1003
alice:admin:1004
charlie:developer:1005
EOF

# Grep examples
grep "developer" users.txt
grep -i "DEVELOPER" users.txt
grep -v "developer" users.txt
grep -c "developer" users.txt
grep -n "manager" users.txt

# Sed examples
sed 's/developer/engineer/' users.txt
sed 's/developer/engineer/g' users.txt
sed '/bob/d' users.txt
sed -n '2,4p' users.txt
sed -i.bak 's/admin/administrator/g' users.txt

# Awk examples
awk -F: '{print $1}' users.txt
awk -F: '{print $1, $2}' users.txt
awk -F: '$2=="developer" {print $1}' users.txt
awk -F: '{print NR, $1}' users.txt
awk -F: '$3 > 1002 {print $1, $3}' users.txt

# Combined
cat users.txt | grep developer | awk -F: '{print $1}' | sort

# Log parsing
cat /var/log/syslog | grep ERROR | tail -20
cat /var/log/auth.log | grep "Failed password" | awk '{print $1, $2, $3, $11}' | sort | uniq -c
```
</details>

---

## Intermediate Level Exercises

### Exercise 4: Process Management
**Objective**: Manage processes, jobs, and resource usage

<details>
<summary>Solution</summary>

```bash
# Background processes
sleep 100 &
jobs
fg %1

# Multiple jobs
sleep 200 &
sleep 300 &
jobs -l
kill %2

# Process monitoring
ps aux | grep sleep
pgrep sleep
top
htop

# Kill processes
killall sleep
pkill -9 sleep

# Process priority
nice -n 10 sleep 1000 &
ps -l | grep sleep
renice -n 5 -p $(pgrep sleep)

# Process information
ps aux --sort=-%mem | head -10
ps aux --sort=-%cpu | head -10

# Nohup
nohup ./long-running-script.sh &
tail -f nohup.out
```

**Create test script:**
```bash
#!/bin/bash
# resource-test.sh
while true; do
    echo "Running..."
    sleep 5
done
```

```bash
chmod +x resource-test.sh
./resource-test.sh &
ps aux | grep resource-test
kill $(pgrep -f resource-test)
```
</details>

---

### Exercise 5: Shell Scripting
**Objective**: Write production-ready shell scripts

<details>
<summary>Solution</summary>

**System Monitor Script:**
```bash
#!/bin/bash
# system-monitor.sh

# Colors
RED='\033[0:31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

echo "=== System Monitor ==="
echo

# CPU Usage
echo -e "${GREEN}CPU Usage:${NC}"
top -bn1 | grep "Cpu(s)" | awk '{print "  "$2}'

# Memory Usage
echo -e "${GREEN}Memory Usage:${NC}"
free -h | awk 'NR==2{printf "  Used: %s/%s (%.2f%%)\n", $3,$2,$3*100/$2}'

# Disk Usage
echo -e "${GREEN}Disk Usage:${NC}"
df -h / | awk 'NR==2{printf "  Used: %s/%s (%s)\n", $3,$2,$5}'

# Top 5 CPU processes
echo -e "${GREEN}Top 5 CPU Processes:${NC}"
ps aux --sort=-%cpu | head -6 | tail -5 | awk '{printf "  %-20s %s%%\n", $11, $3}'

# Check critical services
echo -e "${GREEN}Service Status:${NC}"
for service in sshd nginx docker; do
    if systemctl is-active --quiet $service; then
        echo -e "  $service: ${GREEN}Running${NC}"
    else
        echo -e "  $service: ${RED}Stopped${NC}"
    fi
done

# Check disk space warning
DISK_USAGE=$(df -h / | awk 'NR==2{print $5}' | sed 's/%//')
if [ $DISK_USAGE -gt 80 ]; then
    echo -e "${RED}Warning: Disk usage is above 80%!${NC}"
fi
```

**Backup Script:**
```bash
#!/bin/bash
# backup.sh

SOURCE="/opt/myapp"
BACKUP_DIR="/backup"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/backup_$DATE.tar.gz"

# Check if source exists
if [ ! -d "$SOURCE" ]; then
    echo "Error: Source directory does not exist"
    exit 1
fi

# Create backup directory
mkdir -p $BACKUP_DIR

# Create backup
tar -czf $BACKUP_FILE $SOURCE 2>/dev/null

if [ $? -eq 0 ]; then
    echo "Backup successful: $BACKUP_FILE"
    echo "Size: $(du -h $BACKUP_FILE | awk '{print $1}')"
else
    echo "Backup failed!"
    exit 1
fi

# Keep only last 7 backups
cd $BACKUP_DIR
ls -t backup_*.tar.gz | tail -n +8 | xargs rm -f

# Send notification
echo "Backup completed at $(date)" | mail -s "Backup Status" admin@example.com
```
</details>

---

## Advanced Level Exercises

### Exercise 6: Systemd Service Creation
**Objective**: Create and manage systemd services

<details>
<summary>Solution</summary>

**Create Application:**
```bash
# Create application script
sudo mkdir -p /opt/myapp
sudo cat > /opt/myapp/app.sh <<'EOF'
#!/bin/bash
while true; do
    echo "$(date): Application running" >> /var/log/myapp.log
    sleep 60
done
EOF

sudo chmod +x /opt/myapp/app.sh
```

**Create Systemd Service:**
```bash
sudo cat > /etc/systemd/system/myapp.service <<'EOF'
[Unit]
Description=My Application
After=network.target
Wants=network-online.target

[Service]
Type=simple
User=nobody
Group=nogroup
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/app.sh
ExecStop=/bin/kill -SIGTERM $MAINPID
Restart=on-failure
RestartSec=10
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
EOF
```

**Manage Service:**
```bash
# Reload systemd
sudo systemctl daemon-reload

# Start service
sudo systemctl start myapp

# Check status
sudo systemctl status myapp

# Enable at boot
sudo systemctl enable myapp

# View logs
sudo journalctl -u myapp -f

# Stop service
sudo systemctl stop myapp
```
</details>

---

### Exercise 7: Network Configuration & Troubleshooting
**Objective**: Configure networking and diagnose issues

<details>
<summary>Solution</summary>

```bash
# Network information
ip addr show
ip route show
ip link show

# Configure static IP (Ubuntu 20.04+ netplan)
sudo cat > /etc/netplan/01-netcfg.yaml <<EOF
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
EOF

sudo netplan apply

# DNS testing
dig google.com
nslookup google.com
host google.com

# Connectivity testing
ping -c 4 8.8.8.8
traceroute google.com
mtr --report google.com

# Port scanning
nmap localhost
nc -zv localhost 22
telnet localhost 80

# Check listening ports
ss -tuln
netstat -tuln
lsof -i -P

# Find process on port
lsof -i :80
ss -tulpn | grep :80

# Firewall
sudo ufw status
sudo ufw allow 80/tcp
sudo ufw deny 3306/tcp
sudo ufw enable

# Network statistics
ss -s
netstat -s
ifconfig eth0 | grep "RX packets"

# Bandwidth monitoring
iftop
nethogs
```
</details>

---

### Exercise 8: Performance Tuning
**Objective**: Monitor and optimize system performance

<details>
<summary>Solution</summary>

**System Monitoring:**
```bash
# CPU monitoring
mpstat 1 5
sar -u 1 10
top -bn1 | grep "Cpu(s)"

# Memory monitoring
free -h
vmstat 1 5
cat /proc/meminfo

# Disk I/O
iostat -x 1 5
iotop

# Network I/O
iftop
nethogs
```

**Performance Tuning:**
```bash
# Kernel parameters
sudo sysctl -a | grep -i tcp
sudo sysctl -w net.ipv4.tcp_fin_timeout=30
sudo sysctl -w net.core.somaxconn=1024

# Permanent changes
sudo cat >> /etc/sysctl.conf <<EOF
net.ipv4.tcp_fin_timeout = 30
net.core.somaxconn = 1024
vm.swappiness = 10
fs.file-max = 100000
EOF

sudo sysctl -p

# Process limits
ulimit -a
ulimit -n 65535

# Permanent limits
sudo cat >> /etc/security/limits.conf <<EOF
*  soft  nofile  65535
*  hard  nofile  65535
*  soft  nproc   65535
*  hard  nproc   65535
EOF

# Swap management
free -h
sudo swapoff -a
sudo dd if=/dev/zero of=/swapfile bs=1G count=4
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```
</details>

---

### Exercise 9: Log Analysis & Monitoring
**Objective**: Analyze logs and set up monitoring

<details>
<summary>Solution</summary>

**Log Analysis:**
```bash
# System logs
tail -f /var/log/syslog
journalctl -f

# Auth logs
tail -f /var/log/auth.log
last
lastb  # Failed logins

# Failed SSH attempts
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -nr

# Successful SSH logins
grep "Accepted password" /var/log/auth.log

# Kernel messages
dmesg -T
dmesg | grep -i error

# Service-specific logs
journalctl -u nginx -f
journalctl -u nginx --since "1 hour ago"
journalctl -u nginx --until "10 minutes ago"

# Disk usage by logs
du -sh /var/log/*
journalctl --disk-usage

# Clean old logs
sudo journalctl --vacuum-time=7d
sudo find /var/log -name "*.log.*" -mtime +30 -delete
```

**Log Monitoring Script:**
```bash
#!/bin/bash
# log-monitor.sh

LOG_FILE="/var/log/auth.log"
ALERT_EMAIL="admin@example.com"
FAILED_THRESHOLD=5

# Count failed login attempts
FAILED_COUNT=$(grep "Failed password" $LOG_FILE | grep "$(date '+%b %d')" | wc -l)

if [ $FAILED_COUNT -gt $FAILED_THRESHOLD ]; then
    echo "Alert: $FAILED_COUNT failed login attempts today" | \
        mail -s "Security Alert" $ALERT_EMAIL
    
    # Show top IPs
    grep "Failed password" $LOG_FILE | grep "$(date '+%b %d')" | \
        awk '{print $11}' | sort | uniq -c | sort -nr | head -5
fi
```
</details>

---

### Exercise 10: Complete DevOps Environment Setup
**Objective**: Set up complete server environment for DevOps

<details>
<summary>Solution</summary>

**Setup Script:**
```bash
#!/bin/bash
# devops-setup.sh

set -e

echo "=== DevOps Environment Setup ==="

# Update system
echo "Updating system..."
sudo apt update && sudo apt upgrade -y

# Install essential tools
echo "Installing essential tools..."
sudo apt install -y \
    git curl wget vim htop tree \
    build-essential software-properties-common \
    apt-transport-https ca-certificates gnupg

# Install Docker
echo "Installing Docker..."
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER

# Install Docker Compose
echo "Installing Docker Compose..."
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" \
    -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Install Kubernetes tools
echo "Installing kubectl..."
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Install Ansible
echo "Installing Ansible..."
sudo apt-add-repository ppa:ansible/ansible -y
sudo apt update
sudo apt install ansible -y

# Install monitoring tools
echo "Installing monitoring tools..."
sudo apt install -y \
    sysstat iotop iftop nethogs \
    nmap tcpdump net-tools

# Configure firewall
echo "Configuring firewall..."
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw --force enable

# Create project structure
echo "Creating project structure..."
mkdir -p ~/projects/{ansible,docker,kubernetes,scripts}

# Done
echo "=== Setup Complete ==="
echo "Please log out and back in for group changes to take effect"
```

```bash
chmod +x devops-setup.sh
./devops-setup.sh
```
</details>

---

**Complete all exercises to master Linux! 🐧🚀**

For more resources:
- [Learning Roadmap](linux-learning-roadmap.md)
- [Quick Reference](linux-quick-reference.md)
- [Troubleshooting Guide](linux-troubleshooting-guide.md)
- [Interview Questions](linux-interview-questions.md)

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

