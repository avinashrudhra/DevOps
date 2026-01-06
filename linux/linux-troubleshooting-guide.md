# Linux Troubleshooting Guide

Comprehensive solutions to common Linux issues in production environments.

---

## Table of Contents
1. [Boot Issues](#boot-issues)
2. [Disk Space Problems](#disk-space-problems)
3. [Performance Issues](#performance-issues)
4. [Network Problems](#network-problems)
5. [Permission Denied Errors](#permission-denied-errors)
6. [Service Failures](#service-failures)
7. [Memory Issues](#memory-issues)
8. [Process Problems](#process-problems)
9. [SSH Connection Issues](#ssh-connection-issues)
10. [Production Incidents](#production-incidents)

---

## Boot Issues

### Issue 1: System Won't Boot

**Symptoms:**
- Blank screen after GRUB
- Kernel panic
- Dropped to emergency shell

**Diagnosis:**
```bash
# From rescue/live CD
mount /dev/sda1 /mnt
chroot /mnt

# Check logs
journalctl -xb
dmesg | less

# Check disk
fsck -y /dev/sda1
```

**Solutions:**

**1. Repair GRUB:**
```bash
# From live CD
sudo mount /dev/sda1 /mnt
sudo mount --bind /dev /mnt/dev
sudo mount --bind /proc /mnt/proc
sudo mount --bind /sys /mnt/sys
sudo chroot /mnt

# Reinstall GRUB
grub-install /dev/sda
update-grub

# Exit and reboot
exit
sudo reboot
```

**2. Fix fstab:**
```bash
# Boot to recovery mode
nano /etc/fstab
# Comment out problematic entries with #
```

---

## Disk Space Problems

### Issue 2: No Space Left on Device

**Symptoms:**
```
-bash: cannot create temp file: No space left on device
df: /dev/sda1: No space left on device
```

**Diagnosis:**
```bash
# Check disk space
df -h

# Check inodes
df -i

# Find large directories
du -sh /* | sort -hr | head -10
du -h --max-depth=1 / | sort -hr

# Find large files
find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null
```

**Solutions:**

**1. Clean Package Cache:**
```bash
# Ubuntu/Debian
sudo apt clean
sudo apt autoclean
sudo apt autoremove

# RHEL/CentOS
sudo yum clean all
sudo dnf clean all
```

**2. Clean Logs:**
```bash
# Truncate large log files
sudo truncate -s 0 /var/log/large-log.log

# Delete old logs
sudo find /var/log -name "*.log.*" -mtime +30 -delete

# Clean journal logs
sudo journalctl --vacuum-time=7d
sudo journalctl --vacuum-size=500M
```

**3. Find and Remove:**
```bash
# Find deleted but open files
sudo lsof | grep deleted
# Kill the process holding the file

# Find core dumps
find / -name "core.*" -delete

# Clean temp directories
sudo rm -rf /tmp/*
sudo rm -rf /var/tmp/*
```

**4. Check Inodes:**
```bash
# If inodes full
df -i

# Find directories with many files
for dir in /*; do echo "$dir"; find "$dir" -type f | wc -l; done

# Common culprits
find /var/spool/postfix -type f | wc -l
find /tmp -type f | wc -l
```

---

## Performance Issues

### Issue 3: High CPU Usage

**Diagnosis:**
```bash
# Check CPU usage
top
htop
mpstat 1 5

# Find CPU-heavy processes
ps aux --sort=-%cpu | head -10

# Check load average
uptime
w
cat /proc/loadavg
```

**Solutions:**

**1. Identify Culprit:**
```bash
# Find process
ps aux | grep -v grep | sort -k3 -r | head -5

# Kill if necessary
kill -15 PID  # Graceful
kill -9 PID   # Force

# Check cron jobs
crontab -l
sudo cat /etc/cron*/*
```

**2. Limit Process:**
```bash
# Set CPU limit
cpulimit -p PID -l 50  # 50% CPU

# Change priority
renice -n 10 -p PID
```

---

### Issue 4: High Memory Usage

**Diagnosis:**
```bash
# Check memory
free -h
vmstat 1 5
cat /proc/meminfo

# Find memory-heavy processes
ps aux --sort=-%mem | head -10

# Check swap usage
swapon --show
cat /proc/swaps
```

**Solutions:**

**1. Clear Cache:**
```bash
# Drop caches (safe)
sync
echo 3 | sudo tee /proc/sys/vm/drop_caches
```

**2. Kill Memory Leaks:**
```bash
# Find memory leak
ps aux --sort=-%mem | head -10
kill -9 PID
```

**3. Adjust Swappiness:**
```bash
# Check current
cat /proc/sys/vm/swappiness

# Set temporarily
sudo sysctl vm.swappiness=10

# Permanent
echo "vm.swappiness=10" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

---

## Network Problems

### Issue 5: No Network Connectivity

**Diagnosis:**
```bash
# Check interface status
ip addr show
ip link show
ifconfig

# Check route
ip route show
route -n

# Check DNS
cat /etc/resolv.conf
dig google.com
nslookup google.com

# Test connectivity
ping 8.8.8.8
ping google.com
traceroute 8.8.8.8
```

**Solutions:**

**1. Restart Network:**
```bash
# Ubuntu/Debian
sudo systemctl restart networking
sudo systemctl restart NetworkManager

# RHEL/CentOS
sudo systemctl restart network
sudo nmcli networking off
sudo nmcli networking on
```

**2. Fix DNS:**
```bash
# Edit resolv.conf
sudo nano /etc/resolv.conf
# Add:
nameserver 8.8.8.8
nameserver 8.8.4.4

# Netplan (Ubuntu 20.04+)
sudo nano /etc/netplan/01-netcfg.yaml
# Add DNS
sudo netplan apply
```

**3. Check Firewall:**
```bash
# UFW
sudo ufw status
sudo ufw disable  # Temporary

# firewalld
sudo firewall-cmd --list-all
sudo systemctl stop firewalld  # Temporary
```

---

### Issue 6: Cannot Connect to Port

**Diagnosis:**
```bash
# Check if port is listening
ss -tuln | grep :80
netstat -tuln | grep :80
lsof -i :80

# Check firewall
sudo ufw status numbered
sudo iptables -L -n

# Test locally
telnet localhost 80
curl localhost:80
nc -zv localhost 80
```

**Solutions:**

**1. Start Service:**
```bash
sudo systemctl start nginx
sudo systemctl status nginx
```

**2. Open Port:**
```bash
# UFW
sudo ufw allow 80/tcp

# firewalld
sudo firewall-cmd --add-port=80/tcp --permanent
sudo firewall-cmd --reload

# iptables
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
```

---

## Permission Denied Errors

### Issue 7: Permission Denied

**Diagnosis:**
```bash
# Check file permissions
ls -l filename

# Check directory permissions
ls -ld directory

# Check ownership
stat filename

# Check your user/groups
id
groups
```

**Solutions:**

**1. Fix Permissions:**
```bash
# Files
chmod 644 file        # rw-r--r--
chmod 755 file        # rwxr-xr-x

# Directories
chmod 755 directory   # rwxr-xr-x

# Recursive
chmod -R 755 directory
```

**2. Fix Ownership:**
```bash
sudo chown user:group file
sudo chown -R user:group directory
```

**3. Add User to Group:**
```bash
sudo usermod -aG groupname username
# Log out and back in
```

---

## Service Failures

### Issue 8: Service Won't Start

**Diagnosis:**
```bash
# Check status
sudo systemctl status servicename

# View logs
sudo journalctl -u servicename -xe
sudo journalctl -u servicename --since "5 minutes ago"

# Check configuration
sudo systemctl cat servicename
sudo servicename -t  # Test config (nginx, apache)
```

**Solutions:**

**1. Fix Configuration:**
```bash
# Test config
sudo nginx -t
sudo apache2ctl configtest

# Fix errors and restart
sudo systemctl restart servicename
```

**2. Check Dependencies:**
```bash
# View dependencies
systemctl list-dependencies servicename

# Check if dependency running
systemctl status dependency
```

**3. Reset Failed State:**
```bash
sudo systemctl reset-failed servicename
sudo systemctl start servicename
```

---

## SSH Connection Issues

### Issue 9: Cannot SSH to Server

**Diagnosis:**
```bash
# Test connectivity
ping server
telnet server 22
nc -zv server 22

# Check SSH service
sudo systemctl status sshd

# Check SSH config
sudo sshd -t
sudo cat /etc/ssh/sshd_config

# Check logs
sudo tail -f /var/log/auth.log
sudo journalctl -u sshd -f
```

**Solutions:**

**1. Start SSH Service:**
```bash
sudo systemctl start sshd
sudo systemctl enable sshd
```

**2. Fix SSH Config:**
```bash
# Common issues in /etc/ssh/sshd_config
Port 22
PermitRootLogin no
PasswordAuthentication yes
PubkeyAuthentication yes

sudo systemctl restart sshd
```

**3. Fix Permissions:**
```bash
# Fix SSH key permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
chmod 644 ~/.ssh/authorized_keys
```

**4. Firewall:**
```bash
sudo ufw allow 22/tcp
sudo firewall-cmd --add-service=ssh --permanent
sudo firewall-cmd --reload
```

---

## Production Incidents

### Issue 10: Server Unresponsive

**Emergency Response:**

**1. Check if Alive:**
```bash
ping server
ssh server
```

**2. Console Access (if available):**
```bash
# Check load
uptime
w

# Check processes
top
ps aux

# Check disk
df -h

# Check memory
free -h

# Check logs
dmesg | tail -50
journalctl -xe
```

**3. Emergency Actions:**
```bash
# Kill runaway process
kill -9 PID

# Clear cache
sync; echo 3 > /proc/sys/vm/drop_caches

# Emergency reboot (if no choice)
sync
reboot
# Or magic SysRq
echo b > /proc/sysrq-trigger
```

---

### Issue 11: Root Filesystem Full

**Emergency Fix:**
```bash
# Find and delete immediately
du -sh /* | sort -hr | head -5

# Clean logs aggressively
sudo truncate -s 0 /var/log/*.log

# Remove old kernels
dpkg --list | grep linux-image
sudo apt remove linux-image-X.X.X-X-generic

# Clean packages
sudo apt clean
sudo apt autoclean
```

---

## Diagnostic Commands Summary

```bash
# System Overview
uptime
dmesg
journalctl -xe

# CPU
top, htop
mpstat 1 5
sar -u 1 10

# Memory
free -h
vmstat 1 5
cat /proc/meminfo

# Disk
df -h
df -i
iotop
iostat -x 1 5

# Network
ss -tuln
netstat -tuln
iftop
nethogs

# Processes
ps aux
pstree
lsof

# Logs
tail -f /var/log/syslog
journalctl -f
dmesg -T
```

---

**Complete Linux troubleshooting expertise! 🔧🐧**

For more resources:
- [Learning Roadmap](linux-learning-roadmap.md)
- [Quick Reference](linux-quick-reference.md)
- [Hands-On Exercises](linux-hands-on-exercises.md)
- [Interview Questions](linux-interview-questions.md)

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

