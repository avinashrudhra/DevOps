# Linux Quick Reference

Essential commands and patterns for daily Linux operations.

---

## 📋 File Operations

```bash
# Navigation
cd /path/to/dir          # Change directory
cd ~                     # Home directory
cd -                     # Previous directory
pwd                      # Print working directory
ls                       # List files
ls -la                   # List all with details
ls -lh                   # Human-readable sizes
ls -lt                   # Sort by time
tree                     # Directory tree

# File manipulation
cp source dest           # Copy
cp -r dir dest           # Copy recursively
mv source dest           # Move/rename
rm file                  # Remove file
rm -rf directory         # Remove directory forcefully
mkdir directory          # Create directory
mkdir -p path/to/dir     # Create nested directories
touch file               # Create empty file
ln -s target link        # Create symbolic link

# File viewing
cat file                 # Display file
less file                # Page through file
more file                # Page through file
head file                # First 10 lines
head -n 20 file          # First 20 lines
tail file                # Last 10 lines
tail -f file             # Follow file (logs)
tail -f file | grep ERROR
wc -l file               # Count lines
```

---

## 🔍 Search & Find

```bash
# Find files
find /path -name "*.txt"
find /path -type f -name "file"
find /path -type d -name "dir"
find /path -mtime -7            # Modified in last 7 days
find /path -size +100M          # Files larger than 100MB
find /path -user username
find /path -perm 777

# Locate (faster, uses database)
updatedb                        # Update database
locate filename

# Which/whereis
which command                   # Show command path
whereis command                 # Show binary, source, manual

# Grep (search in files)
grep "pattern" file
grep -i "pattern" file          # Case insensitive
grep -r "pattern" /path         # Recursive
grep -n "pattern" file          # Show line numbers
grep -v "pattern" file          # Invert match
grep -E "pat1|pat2" file        # Extended regex
grep -A 3 "pattern" file        # 3 lines after
grep -B 3 "pattern" file        # 3 lines before
grep -C 3 "pattern" file        # 3 lines context
```

---

## 📝 Text Processing

```bash
# sed - Stream editor
sed 's/old/new/' file           # Replace first occurrence
sed 's/old/new/g' file          # Replace all
sed -i 's/old/new/g' file       # Edit in-place
sed '/pattern/d' file           # Delete matching lines
sed -n '10,20p' file            # Print lines 10-20

# awk - Pattern scanning
awk '{print $1}' file           # Print first column
awk -F: '{print $1}' /etc/passwd
awk '$3 > 1000' file            # Conditional
awk '{sum+=$1} END {print sum}' # Sum column

# cut - Cut columns
cut -d: -f1 /etc/passwd         # First field
cut -c1-10 file                 # Characters 1-10

# sort - Sort lines
sort file                       # Alphabetical
sort -n file                    # Numeric
sort -r file                    # Reverse
sort -k2 file                   # Sort by column 2
sort -u file                    # Unique lines

# uniq - Remove duplicates
sort file | uniq
sort file | uniq -c             # Count occurrences
sort file | uniq -d             # Only duplicates

# tr - Translate characters
echo "hello" | tr 'a-z' 'A-Z'   # Uppercase
tr -d '\r' < file               # Delete characters
```

---

## 👤 User & Permission Management

```bash
# Users
useradd username                # Add user
useradd -m -s /bin/bash user    # Add with home & shell
usermod -aG sudo username       # Add to group
userdel username                # Delete user
userdel -r username             # Delete with home
passwd username                 # Set password
id username                     # User info
who                             # Logged in users
w                               # Who and what they're doing
last                            # Login history

# Groups
groupadd groupname              # Add group
groupdel groupname              # Delete group
groups username                 # Show user's groups
cat /etc/group                  # All groups

# Permissions
chmod 755 file                  # rwxr-xr-x
chmod 644 file                  # rw-r--r--
chmod u+x file                  # Add execute for user
chmod g-w file                  # Remove write for group
chmod o=r file                  # Set others to read
chown user:group file           # Change ownership
chown -R user:group dir         # Recursive

# Special permissions
chmod +t directory              # Sticky bit
chmod u+s file                  # SUID
chmod g+s directory             # SGID
find / -perm -4000              # Find SUID files
```

---

## ⚙️ Process Management

```bash
# View processes
ps aux                          # All processes
ps -ef                          # Full format
ps aux | grep nginx             # Filter processes
pgrep nginx                     # Find process by name
pidof nginx                     # PIDs by name
pstree                          # Process tree

# Top commands
top                             # Real-time processes
htop                            # Better top
atop                            # Advanced monitoring

# Kill processes
kill PID                        # SIGTERM (15)
kill -9 PID                     # SIGKILL (force)
kill -HUP PID                   # SIGHUP (reload)
killall process_name            # Kill by name
pkill -9 pattern                # Kill by pattern

# Background jobs
command &                       # Run in background
jobs                            # List jobs
fg %1                           # Foreground job 1
bg %1                           # Background job 1
Ctrl+Z                          # Suspend current job
nohup command &                 # Run immune to hangups
disown %1                       # Detach job from shell

# Process priority
nice -n 10 command              # Start with priority
renice -n 5 -p PID              # Change priority
```

---

## 📦 Package Management

### Ubuntu/Debian (APT)
```bash
apt update                      # Update package list
apt upgrade                     # Upgrade packages
apt dist-upgrade                # Dist upgrade
apt install package             # Install
apt remove package              # Remove
apt purge package               # Remove with config
apt autoremove                  # Remove unused
apt search package              # Search
apt show package                # Show info
apt list --installed            # List installed
apt list --upgradable           # List upgradable

# dpkg
dpkg -i package.deb             # Install .deb
dpkg -r package                 # Remove
dpkg -l                         # List installed
dpkg -L package                 # List files in package
dpkg -S /path/to/file           # Find package owning file
```

### RHEL/CentOS (YUM/DNF)
```bash
yum update                      # Update
yum install package             # Install
yum remove package              # Remove
yum search package              # Search
yum info package                # Info
yum list installed              # List installed

# DNF (Fedora, RHEL 8+)
dnf update
dnf install package
dnf remove package

# rpm
rpm -ivh package.rpm            # Install
rpm -Uvh package.rpm            # Upgrade
rpm -qa                         # List all
rpm -ql package                 # List files
rpm -qf /path/to/file           # Find package
```

---

## 🔧 System Services (systemd)

```bash
# Service management
systemctl start service         # Start
systemctl stop service          # Stop
systemctl restart service       # Restart
systemctl reload service        # Reload config
systemctl status service        # Status
systemctl enable service        # Enable at boot
systemctl disable service       # Disable at boot
systemctl is-enabled service    # Check if enabled
systemctl is-active service     # Check if active

# List services
systemctl list-units --type=service
systemctl list-unit-files --type=service
systemctl list-units --failed

# Service inspection
systemctl cat service           # Show service file
systemctl show service          # Show properties
systemctl edit service          # Edit override

# Logs (journalctl)
journalctl -u service           # Service logs
journalctl -u service -f        # Follow logs
journalctl -u service --since "1 hour ago"
journalctl -u service --since today
journalctl -p err               # Error level
journalctl -b                   # Current boot
journalctl --disk-usage         # Disk usage
journalctl --vacuum-time=2d     # Clean old logs

# System control
systemctl reboot                # Reboot
systemctl poweroff              # Shutdown
systemctl suspend               # Suspend
systemctl hibernate             # Hibernate
```

---

## 🌐 Networking

```bash
# Network interfaces
ip addr show                    # Show IP addresses
ip addr add 192.168.1.10/24 dev eth0
ip link show                    # Show interfaces
ip link set eth0 up             # Enable interface
ip link set eth0 down           # Disable interface
ip route show                   # Show routes
ip route add default via 192.168.1.1

# Legacy commands (still work)
ifconfig                        # Show interfaces
ifconfig eth0 up                # Enable
route -n                        # Show routes
route add default gw 192.168.1.1

# DNS
dig example.com                 # DNS lookup
dig +short example.com          # Short output
nslookup example.com            # NS lookup
host example.com                # Host lookup
cat /etc/resolv.conf            # DNS servers

# Connectivity
ping -c 4 example.com           # Ping 4 times
traceroute example.com          # Trace route
mtr example.com                 # Network diagnostic
nc -zv host 80                  # Port scan

# Ports & connections
netstat -tuln                   # Listening ports
netstat -an                     # All connections
ss -tuln                        # Modern netstat
ss -s                           # Statistics
lsof -i :80                     # Process on port
lsof -i -P                      # All network files

# Download
curl -O url                     # Download file
curl -I url                     # Headers only
curl -X POST -d "data" url      # POST request
wget url                        # Download
wget -c url                     # Resume download

# Firewall (UFW - Ubuntu)
ufw status                      # Check status
ufw allow 22/tcp                # Allow SSH
ufw deny 80/tcp                 # Deny HTTP
ufw delete allow 22             # Remove rule
ufw enable                      # Enable firewall
ufw disable                     # Disable firewall

# Firewall (firewalld - RHEL/CentOS)
firewall-cmd --list-all
firewall-cmd --add-port=80/tcp --permanent
firewall-cmd --remove-port=80/tcp --permanent
firewall-cmd --reload
firewall-cmd --list-ports
```

---

## 💾 Disk Management

```bash
# Disk usage
df -h                           # Disk space
df -i                           # Inodes
du -sh directory                # Directory size
du -h --max-depth=1             # Size of subdirs
ncdu                            # Interactive disk usage

# Disk info
fdisk -l                        # List disks
lsblk                           # Block devices
blkid                           # UUIDs
parted -l                       # Partition info

# Mount
mount                           # Show mounts
mount /dev/sdb1 /mnt            # Mount
umount /mnt                     # Unmount
mount -a                        # Mount all (fstab)

# /etc/fstab format
# <device> <mount> <type> <options> <dump> <pass>
/dev/sdb1  /data  ext4  defaults  0  2
UUID=xxx   /data  ext4  defaults  0  2
```

---

## 📊 System Monitoring

```bash
# CPU & Memory
top                             # Real-time
htop                            # Better top
free -h                         # Memory usage
vmstat 1                        # Virtual memory stats
mpstat                          # CPU stats
sar -u 1 10                     # CPU usage
lscpu                           # CPU info
cat /proc/cpuinfo               # Detailed CPU info

# Memory
free -m                         # MB
free -g                         # GB
cat /proc/meminfo               # Memory info

# Disk I/O
iotop                           # I/O by process
iostat                          # I/O statistics
sar -d 1 10                     # Disk stats

# Network monitoring
iftop                           # Network bandwidth
nethogs                         # Net usage by process
bmon                            # Bandwidth monitor
iptraf-ng                       # IP traffic monitor

# System load
uptime                          # System uptime
w                               # Load average
cat /proc/loadavg               # Load info

# Logs
tail -f /var/log/syslog         # System log
tail -f /var/log/auth.log       # Auth log
dmesg                           # Kernel messages
dmesg -T                        # Human-readable time
```

---

## 🔐 Security

```bash
# SSH
ssh user@host                   # Connect
ssh -i key user@host            # With key
ssh-keygen -t rsa -b 4096       # Generate key
ssh-copy-id user@host           # Copy public key
scp file user@host:/path        # Copy file
scp -r dir user@host:/path      # Copy directory

# Firewall
ufw status numbered             # Show rules
ufw allow from 192.168.1.0/24   # Allow subnet
ufw limit ssh                   # Rate limit

# Security scanning
nmap localhost                  # Port scan
lynis audit system              # Security audit
rkhunter --check                # Rootkit check
chkrootkit                      # Rootkit check

# File integrity
md5sum file                     # MD5 hash
sha256sum file                  # SHA256 hash
find / -perm -4000              # SUID files
find / -perm -2000              # SGID files
```

---

## ⏰ Scheduling (Cron)

```bash
# Cron syntax
# * * * * * command
# │ │ │ │ │
# │ │ │ │ └─── Day of week (0-7)
# │ │ │ └───── Month (1-12)
# │ │ └─────── Day of month (1-31)
# │ └───────── Hour (0-23)
# └─────────── Minute (0-59)

# Edit crontab
crontab -e                      # Edit
crontab -l                      # List
crontab -r                      # Remove

# Examples
*/5 * * * * /script.sh          # Every 5 minutes
0 2 * * * /backup.sh            # Daily at 2am
0 */6 * * * /check.sh           # Every 6 hours
0 0 * * 0 /weekly.sh            # Weekly on Sunday
0 0 1 * * /monthly.sh           # Monthly on 1st

# System cron
ls /etc/cron.{daily,weekly,monthly}
```

---

## 🔄 Archive & Compression

```bash
# tar
tar -czf archive.tar.gz files   # Create gzip
tar -cjf archive.tar.bz2 files  # Create bzip2
tar -xzf archive.tar.gz         # Extract gzip
tar -xjf archive.tar.bz2        # Extract bzip2
tar -xzf archive.tar.gz -C /path # Extract to path
tar -tzf archive.tar.gz         # List contents

# gzip/gunzip
gzip file                       # Compress
gunzip file.gz                  # Decompress
gzip -d file.gz                 # Decompress

# zip/unzip
zip archive.zip files           # Create
unzip archive.zip               # Extract
unzip -l archive.zip            # List contents
```

---

## 💡 Useful One-Liners

```bash
# Find and kill process
ps aux | grep process | grep -v grep | awk '{print $2}' | xargs kill -9

# Find large files
find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null

# Disk usage sorted
du -sh /* | sort -hr | head -10

# Count files in directory
find /path -type f | wc -l

# Find recently modified files
find /path -mtime -1 -type f

# Replace string in multiple files
find . -type f -name "*.txt" -exec sed -i 's/old/new/g' {} \;

# Check which process uses port
lsof -i :8080
netstat -tulpn | grep :8080

# Monitor log file
tail -f /var/log/syslog | grep ERROR

# Directory sizes sorted
du -h --max-depth=1 | sort -hr

# CPU usage by process
ps aux --sort=-%cpu | head -10

# Memory usage by process
ps aux --sort=-%mem | head -10

# Find zombie processes
ps aux | awk '$8=="Z"'

# Network connections by state
netstat -an | awk '{print $6}' | sort | uniq -c
```

---

**Quick reference complete! 🐧✨**

For more resources:
- [Learning Roadmap](linux-learning-roadmap.md)
- [Hands-On Exercises](linux-hands-on-exercises.md)
- [Troubleshooting Guide](linux-troubleshooting-guide.md)
- [Interview Questions](linux-interview-questions.md)

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

