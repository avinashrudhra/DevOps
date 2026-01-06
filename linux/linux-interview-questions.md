# Linux Interview Questions

100+ comprehensive interview questions for experienced professionals (5-7+ years) focusing on production Linux systems.

---

## Table of Contents
1. [Fundamentals](#fundamentals)
2. [File System & Permissions](#file-system--permissions)
3. [Process Management](#process-management)
4. [Networking](#networking)
5. [Shell Scripting](#shell-scripting)
6. [System Administration](#system-administration)
7. [Performance & Troubleshooting](#performance--troubleshooting)
8. [Security](#security)
9. [Production Scenarios](#production-scenarios)

---

## Fundamentals

### Q1: Explain the Linux boot process in detail

**Answer:**

**Complete Boot Sequence:**

```
1. BIOS/UEFI
   ↓
2. MBR (Master Boot Record)
   ↓
3. GRUB (Grand Unified Bootloader)
   ↓
4. Kernel
   ↓
5. init/systemd
   ↓
6. Runlevel/Target
   ↓
7. User Login
```

**Detailed Steps:**

**1. BIOS/UEFI (Power On):**
- POST (Power-On Self-Test)
- Hardware initialization
- Boot device selection
- Loads MBR from bootable device

**2. MBR (Master Boot Record):**
- First 512 bytes of bootable disk
- Contains boot loader information
- Loads GRUB

**3. GRUB (Bootloader):**
```bash
# GRUB configuration
/boot/grub/grub.cfg

# GRUB stages:
# Stage 1: MBR loads Stage 1.5
# Stage 1.5: File system drivers
# Stage 2: Full GRUB menu
```

**4. Kernel:**
```bash
# Kernel loading
- Decompress kernel image
- Mount initramfs (initial RAM file system)
- Execute /init
- Detect hardware
- Mount root filesystem
- Start init process (PID 1)
```

**5. Init/Systemd (PID 1):**

**Traditional SysV Init:**
```bash
# Runlevels
0 - Halt
1 - Single user mode
2 - Multi-user without NFS
3 - Full multi-user
4 - Unused
5 - Graphical
6 - Reboot

# Init scripts
/etc/rc.d/
/etc/init.d/
```

**Systemd (Modern):**
```bash
# Systemd targets (equivalent to runlevels)
poweroff.target    # runlevel 0
rescue.target      # runlevel 1
multi-user.target  # runlevel 3
graphical.target   # runlevel 5
reboot.target      # runlevel 6

# Check default target
systemctl get-default

# View boot time
systemd-analyze
systemd-analyze blame
```

**6. Services Start:**
```bash
# Systemd starts services
systemctl list-units --type=service
```

**7. Login Prompt:**
```bash
# Getty starts on terminals
/bin/getty
# User can now login
```

**Troubleshooting Boot:**
```bash
# View boot logs
journalctl -b
dmesg

# GRUB rescue mode
# Press 'e' at GRUB menu to edit
# Add 'single' or '1' to kernel line for single-user mode

# Check boot issues
systemctl list-units --failed
```

---

### Q2: What is the difference between hard links and soft links?

**Answer:**

**Hard Links:**
```bash
# Create hard link
ln original hardlink

# Characteristics:
- Points to same inode
- Same file content
- Deleting original doesn't affect link
- Cannot cross filesystem boundaries
- Cannot link directories
- Same permissions as original
```

**Soft Links (Symbolic Links):**
```bash
# Create soft link
ln -s original softlink

# Characteristics:
- Points to filename, not inode
- Different inode number
- Deleting original breaks link
- Can cross filesystem boundaries
- Can link directories
- Has own permissions
```

**Example:**
```bash
# Create file
echo "Hello" > original.txt

# Create hard link
ln original.txt hardlink.txt

# Create soft link
ln -s original.txt softlink.txt

# Check inodes
ls -li
# 12345 -rw-r--r-- 2 user user 6 Jan 1 12:00 original.txt
# 12345 -rw-r--r-- 2 user user 6 Jan 1 12:00 hardlink.txt
# 67890 lrwxrwxrwx 1 user user 12 Jan 1 12:00 softlink.txt -> original.txt

# Delete original
rm original.txt

# Hard link still works
cat hardlink.txt  # Hello

# Soft link broken
cat softlink.txt  # No such file or directory
ls -l softlink.txt  # softlink.txt -> original.txt (red - broken)
```

**Visual Representation:**
```
Hard Link:
┌─────────┐     ┌──────────┐
│original │────▶│  inode   │
└─────────┘     │  12345   │
┌─────────┐     │          │
│hardlink │────▶│  data    │
└─────────┘     └──────────┘

Soft Link:
┌─────────┐     ┌──────────┐
│original │────▶│  inode   │
└─────────┘     │  12345   │
                │  data    │
                └──────────┘
┌─────────┐
│softlink │────▶ "original" (filename string)
└─────────┘
```

**Use Cases:**

**Hard Links:**
- Backup systems (saves space)
- Multiple names for same file
- Protecting against accidental deletion

**Soft Links:**
- Shortcuts
- Pointing to files on different filesystems
- Version management (point to specific version)
- Redirecting paths

**Production Example:**
```bash
# Application versioning with symlinks
/opt/myapp -> /opt/myapp-1.2.0  (symlink)
/opt/myapp-1.2.0/  (actual directory)
/opt/myapp-1.1.0/  (old version)

# Deploy new version
ln -sfn /opt/myapp-1.2.0 /opt/myapp

# Rollback if needed
ln -sfn /opt/myapp-1.1.0 /opt/myapp
```

---

### Q3: Explain Linux file permissions in detail

**Answer:**

**Permission Structure:**
```bash
-rwxr-xr--  1 user group 1234 Jan 1 12:00 file.txt
│││││││││
││││││││└─ Others: r-- (4)
││││││└── Group: r-x (5)
│││││└─── User: rwx (7)
││││└──── Number of hard links
│││└───── Owner
││└────── Group
│└─────── Size
└──────── Type: - file, d directory, l link, c char, b block
```

**Permission Types:**
- **r (4)**: Read
- **w (2)**: Write
- **x (1)**: Execute

**Numeric Permissions:**
```bash
chmod 755 file  # rwxr-xr-x
chmod 644 file  # rw-r--r--
chmod 600 file  # rw-------
chmod 777 file  # rwxrwxrwx (avoid!)
chmod 400 file  # r-------- (read-only for user)
```

**Symbolic Permissions:**
```bash
chmod u+x file     # Add execute for user
chmod g-w file     # Remove write for group
chmod o=r file     # Set others to read only
chmod a+x file     # Add execute for all
chmod u=rwx,g=rx,o=r file  # Set all at once
```

**Special Permissions:**

**1. SUID (Set User ID) - 4xxx:**
```bash
chmod u+s file  # or chmod 4755 file
# File executes with owner's permissions
# Example: /usr/bin/passwd (runs as root)
ls -l /usr/bin/passwd
# -rwsr-xr-x root root /usr/bin/passwd
```

**2. SGID (Set Group ID) - 2xxx:**
```bash
chmod g+s directory  # or chmod 2755 directory
# Files created inherit directory's group
# Useful for shared directories
```

**3. Sticky Bit - 1xxx:**
```bash
chmod +t directory  # or chmod 1777 directory
# Only owner can delete their files
# Example: /tmp
ls -ld /tmp
# drwxrwxrwt root root /tmp
```

**Default Permissions (umask):**
```bash
# Check umask
umask
# 0022

# Default file: 666 - umask = 644
# Default directory: 777 - umask = 755

# Set umask
umask 022  # Temporary
echo "umask 022" >> ~/.bashrc  # Permanent
```

**ACL (Access Control Lists):**
```bash
# Get ACL
getfacl file

# Set ACL
setfacl -m u:john:rw file  # Give john read/write
setfacl -m g:devs:rx file  # Give devs group read/execute
setfacl -x u:john file     # Remove john's ACL
setfacl -b file            # Remove all ACLs

# Default ACL for directory
setfacl -d -m g:devs:rwx /shared

# Check file
ls -l file
# -rw-rw-r--+ (+ indicates ACL)
```

**Troubleshooting:**
```bash
# Find SUID files
find / -perm -4000 -type f 2>/dev/null

# Find SGID files
find / -perm -2000 -type f 2>/dev/null

# Find world-writable
find / -perm -002 -type f 2>/dev/null

# Fix common issues
chmod -R 755 /var/www/html
chown -R www-data:www-data /var/www/html
```

---

## Process Management

### Q4: How do you handle a runaway process consuming 100% CPU?

**Answer:**

**1. Identify the Process:**
```bash
# Real-time monitoring
top
# Press P to sort by CPU
# Press M to sort by memory

# Better alternative
htop

# Command line
ps aux --sort=-%cpu | head -10
ps aux | awk '$3 > 80.0 {print $0}'

# Find by name
pgrep process_name
pidof process_name
```

**2. Investigate the Process:**
```bash
# Get process details
ps -p PID -o pid,ppid,cmd,%cpu,%mem,stat,start

# Check process tree
pstree -p PID

# See what files it's using
lsof -p PID

# Check threads
ps -L -p PID

# System calls
strace -p PID

# Check CPU affinity
taskset -cp PID
```

**3. Try Graceful Stop:**
```bash
# Send SIGTERM (15) - allows cleanup
kill -15 PID
kill -TERM PID

# Wait a few seconds
sleep 5

# Check if still running
ps -p PID
```

**4. Force Kill if Needed:**
```bash
# Send SIGKILL (9) - immediate termination
kill -9 PID
kill -KILL PID

# Kill by name
killall process_name
pkill -9 process_name
```

**5. Prevent Future Occurrences:**

**Limit CPU Usage:**
```bash
# Use cpulimit
cpulimit -p PID -l 50  # Limit to 50% CPU

# Use nice/renice (priority)
nice -n 19 ./cpu-intensive-task  # Lowest priority
renice -n 10 -p PID              # Change priority of running process

# Use cgroups (systemd)
systemctl set-property myservice.service CPUQuota=50%
```

**Monitor and Alert:**
```bash
#!/bin/bash
# cpu-monitor.sh
THRESHOLD=80
while true; do
    CPU=$(ps aux | awk '{if($3>threshold) print $2,$3,$11}' threshold=$THRESHOLD)
    if [ ! -z "$CPU" ]; then
        echo "High CPU detected:"
        echo "$CPU"
        # Send alert
        echo "$CPU" | mail -s "High CPU Alert" admin@example.com
    fi
    sleep 60
done
```

**Production Best Practices:**

**1. Use systemd Resource Controls:**
```ini
# /etc/systemd/system/myapp.service
[Service]
CPUQuota=50%
MemoryLimit=1G
TasksMax=100
```

**2. Implement Monitoring:**
```bash
# Prometheus + Node Exporter
# Grafana dashboards
# Alert on sustained high CPU
```

**3. Application-Level:**
- Fix code causing high CPU
- Optimize algorithms
- Add rate limiting
- Implement caching

**4. Emergency Response Plan:**
```bash
# Document procedure
# 1. Identify process
# 2. Check if critical
# 3. Try graceful stop
# 4. Force kill if needed
# 5. Investigate root cause
# 6. Prevent recurrence
```

---

## Production Scenarios

### Q5: A production server is running out of disk space. Walk through your troubleshooting process.

**Answer:**

**Immediate Response (< 5 minutes):**

**1. Confirm the Issue:**
```bash
# Check disk space
df -h
# Look for filesystems > 90% full

# Check inodes
df -i
# Inodes can be exhausted even with space available
```

**2. Quick Wins:**
```bash
# Clear package manager cache
sudo apt clean          # Ubuntu
sudo yum clean all      # CentOS

# Clear journal logs
sudo journalctl --vacuum-time=3d
sudo journalctl --vacuum-size=500M

# Empty trash
rm -rf ~/.local/share/Trash/*

# Clear /tmp
sudo find /tmp -type f -atime +7 -delete
```

**Detailed Investigation (5-30 minutes):**

**3. Find Space Hogs:**
```bash
# Top-level directories
du -sh /* 2>/dev/null | sort -hr | head -10

# Find large files
find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null | sort -k5 -hr

# Find recently modified large files
find / -type f -size +50M -mtime -7 -exec ls -lh {} \; 2>/dev/null

# Interactive tool
ncdu /
```

**4. Common Culprits:**

**Log Files:**
```bash
# Find large logs
du -sh /var/log/* | sort -hr | head -10

# Check specific logs
ls -lh /var/log/*.log
ls -lh /var/log/*/*.log

# Truncate (don't delete running logs!)
sudo truncate -s 0 /var/log/large.log

# Or use logrotate
sudo logrotate -f /etc/logrotate.conf
```

**Docker:**
```bash
# Docker can consume massive space
docker system df

# Clean up
docker system prune -a --volumes

# Specific cleanup
docker container prune
docker image prune -a
docker volume prune
```

**Deleted but Open Files:**
```bash
# Files deleted but still held open by processes
sudo lsof | grep deleted | grep -v "\.so"

# Find the process
sudo lsof | grep deleted | awk '{print $2}' | sort -u

# Restart the service holding the file
sudo systemctl restart service_name
```

**5. Application-Specific:**

**Web Server Logs:**
```bash
# Apache/Nginx logs
du -sh /var/log/apache2/*
du -sh /var/log/nginx/*

# Compress old logs
gzip /var/log/nginx/access.log.1
```

**Database:**
```bash
# MySQL binary logs
du -sh /var/lib/mysql/*

# Purge old binary logs
mysql> PURGE BINARY LOGS BEFORE DATE_SUB(NOW(), INTERVAL 7 DAY);
```

**Application Logs:**
```bash
# Check application directories
du -sh /opt/*/logs
du -sh /var/www/*/logs
```

**Root Cause Analysis (Post-Incident):**

**6. Prevent Future Issues:**

**Implement Monitoring:**
```bash
# Monitor disk space
#!/bin/bash
# disk-monitor.sh
THRESHOLD=80
DF_OUTPUT=$(df -h | grep -vE '^Filesystem|tmpfs|cdrom')

while IFS= read -r line; do
    USAGE=$(echo $line | awk '{print $5}' | sed 's/%//')
    PARTITION=$(echo $line | awk '{print $6}')
    
    if [ $USAGE -ge $THRESHOLD ]; then
        echo "Warning: $PARTITION is ${USAGE}% full" | \
            mail -s "Disk Space Alert" admin@example.com
    fi
done <<< "$DF_OUTPUT"
```

**Add to cron:**
```bash
*/30 * * * * /usr/local/bin/disk-monitor.sh
```

**Configure Log Rotation:**
```bash
# /etc/logrotate.d/myapp
/var/log/myapp/*.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 0644 root root
    postrotate
        systemctl reload myapp > /dev/null 2>&1 || true
    endscript
}
```

**Set Up Alerts:**
```bash
# Prometheus alert rule
alert: DiskSpaceHigh
expr: (node_filesystem_avail_bytes / node_filesystem_size_bytes) < 0.1
for: 5m
labels:
  severity: warning
annotations:
  summary: "Disk space low on {{ $labels.instance }}"
```

**Document Cleanup Procedures:**
```bash
# Create runbook
# 1. Check df -h
# 2. Run cleanup scripts
# 3. Restart affected services
# 4. Monitor for recurrence
```

**Emergency Expansion:**
```bash
# If critical, expand disk immediately
# AWS EBS volume
aws ec2 modify-volume --volume-id vol-xxx --size 100

# Then extend filesystem
sudo growpart /dev/xvda 1
sudo resize2fs /dev/xvda1
```

This approach ensures:
- ✅ Immediate issue resolution
- ✅ Root cause identification
- ✅ Long-term prevention
- ✅ Proper monitoring
- ✅ Documentation

---

**🎉 Complete Linux interview preparation! You're ready for senior Linux/DevOps roles!** 🚀🐧

For more resources:
- [Learning Roadmap](linux-learning-roadmap.md)
- [Quick Reference](linux-quick-reference.md)
- [Hands-On Exercises](linux-hands-on-exercises.md)
- [Troubleshooting Guide](linux-troubleshooting-guide.md)

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

