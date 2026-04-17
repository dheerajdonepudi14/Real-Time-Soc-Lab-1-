# Splunk Enterprise Installation on Ubuntu - Complete Guide

##  Overview

This guide covers **complete step-by-step Splunk Enterprise installation** on Ubuntu 20.04/22.04 LTS. Ubuntu is ideal for production and stable lab environments.

---

##  Prerequisites

### System Requirements for Ubuntu

```
Minimum for Lab:
- RAM: 4 GB (8 GB recommended)
- Disk Space: 20 GB free
- CPU: 2 cores minimum
- Network: Bridged adapter mode
- OS: Ubuntu 18.04 LTS or later

Recommended for Production Lab:
- RAM: 8-16 GB
- Disk Space: 50-100 GB
- CPU: 4+ cores
- Network: Static IP configured
- OS: Ubuntu 20.04 or 22.04 LTS
```

### Check Current System

```bash
# Check Ubuntu version
cat /etc/os-release
# Expected: Ubuntu 20.04 LTS or 22.04 LTS

# Check architecture
uname -m
# Expected: x86_64 (64-bit)

# Check available RAM
free -h
# Should show: 4 GB or more

# Check disk space
df -h /
# Should show: 20 GB or more
```

### Network Setup

```bash
# Check network configuration
ip addr show
# Look for inet (not 127.0.0.1)

# Example good output:
# inet 10.0.0.250/24 brd 10.0.0.255 scope global ens33

# If using bridged adapter:
# VirtualBox > VM Settings > Network > Attached to: Bridged Adapter
```

---

##  Step 1: System Preparation (5-10 minutes)

### 1.1 Update System

```bash
# Update package lists
sudo apt update

# Upgrade packages
sudo apt upgrade -y

# Install essential packages
sudo apt install -y \
  curl \
  wget \
  git \
  nano \
  vim \
  htop \
  net-tools \
  openssh-server

# Verify installations
curl --version
wget --version
```

### 1.2 Verify System Requirements

```bash
# Check memory
free -h
# Output example:
# total: 7.8 Gi
# available: 6.2 Gi

# Check disk space
df -h /
# Output example:
# Size: 50G
# Available: 45G

# Check internet
ping google.com -c 1
# Should get: "1 packets transmitted, 1 received"

# Check processor
nproc
# Should show: 2 or more cores
```

### 1.3 Create Working Directory

```bash
# Create directory for installation files
mkdir -p ~/splunk-install
cd ~/splunk-install

# Verify location
pwd
# Output: /home/username/splunk-install
```

### 1.4 Create Splunk User (Optional but Recommended)

```bash
# Create dedicated splunk user
sudo useradd -m -s /bin/bash splunk

# Or create with home directory
sudo useradd -d /opt/splunk -m -s /bin/false splunk

# Add to sudo group (optional)
sudo usermod -aG sudo splunk

# Verify user created
id splunk
```

---

##  Step 2: Download Splunk Enterprise (10-20 minutes)

### 2.1 Open Web Browser

```bash
# On Ubuntu with GUI
firefox &

# Or use curl/wget from terminal (see below)
```

### 2.2 Navigate to Splunk Website

```
Manual Steps:
1. Open: https://www.splunk.com
2. Click "Platform" → "Splunk Enterprise"
3. Click "Trials & Downloads"
4. Click "Start Download"
5. Sign up or log in to account
```

### 2.3 Create/Login to Splunk Account

```
Account Setup:
1. If new: Click "Sign up"
   - Enter email (use real email)
   - Create password
   - Verify email
   
2. If existing: Click "Sign in"
   - Enter email
   - Enter password
   - Click "Sign in"

Save credentials:
- Email: ___________________
- Password: ___________________
```

### 2.4 Select Download

```
After login:
1. Click "Start Download"
2. Select Operating System: "Linux"
3. Select Architecture: "Linux x86_64"
4. Select File Type: ".deb" (Debian)
5. Copy wget link (or click download)
```

### 2.5 Download Using wget (Recommended)

```bash
# Navigate to working directory
cd ~/splunk-install

# Method 1: Using wget (paste link from Splunk)
sudo wget https://download.splunk.com/products/splunk/releases/9.0.0/splunk-9.0.0-5d8e9fcecfe4-Linux-x86_64.deb

# This takes 5-15 minutes depending on internet speed
# You'll see progress bar

# Verify download completed
ls -lh splunk-*.deb

# Example output:
# -rw-r--r-- 1 root root 560M Feb 19 10:30 splunk-9.0.0-5d8e9fcecfe4-Linux-x86_64.deb
```

### 2.6 Alternative Download Methods

```bash
# Method 2: Using curl
sudo curl -O https://download.splunk.com/products/splunk/releases/9.0.0/splunk-9.0.0-5d8e9fcecfe4-Linux-x86_64.deb

# Method 3: Download via browser
# 1. Click download button in Firefox
# 2. File goes to ~/Downloads/
# 3. Copy to working directory:
cp ~/Downloads/splunk-*.deb ~/splunk-install/

# Method 4: Using aria2 (for faster downloads)
sudo apt install -y aria2
sudo aria2c https://download.splunk.com/products/splunk/releases/9.0.0/splunk-9.0.0-5d8e9fcecfe4-Linux-x86_64.deb
```

### 2.7 Verify Download

```bash
# Check file size (should be 500MB+)
ls -lh splunk-*.deb

# Check file integrity
file splunk-*.deb
# Output: PE32+ executable (Windows/Linux Installer)

# Or check MD5 hash (if provided by Splunk)
md5sum splunk-*.deb
# Compare with value from Splunk website
```

---

##  Step 3: Install Splunk (5-15 minutes)

### 3.1 Install Dependencies

```bash
# Install curl (required by Splunk)
sudo apt install -y curl

# Install other useful tools
sudo apt install -y wget git openssl

# Verify curl installed
curl --version
```

### 3.2 Install Splunk Package

```bash
# Navigate to installation directory
cd ~/splunk-install

# Install using dpkg
sudo dpkg -i splunk-*.deb

# The installation will take 5-15 minutes
# You'll see output like:
# Setting up splunk (9.0.0-5d8e9fcecfe4) ...
# Done processing triggers

# If there are warnings, they're usually safe to ignore
```

### 3.3 Verify Installation

```bash
# Check Splunk installation location
ls -la /opt/splunk/

# Expected directories:
# bin/    (executables)
# etc/    (configuration)
# lib/    (libraries)
# var/    (data)
# share/  (documentation)
# fuse_packages/ (packages)

# Check Splunk version
/opt/splunk/bin/splunk version

# Expected output:
# Splunk 9.0.0 build 5d8e9fcecfe4
```

### 3.4 Fix Permissions (If Needed)

```bash
# Check ownership
ls -ld /opt/splunk/

# If owned by root (it is by default)
sudo chown -R splunk:splunk /opt/splunk/

# Or keep as root (fine for labs)
# Skip this step if you want root ownership
```

---

## ⚙️ Step 4: Configure Splunk (10-15 minutes)

### 4.1 Enable Boot-Start (Automatic Startup)

```bash
# Enable Splunk to start automatically on boot
# This sets up admin credentials

sudo /opt/splunk/bin/splunk enable boot-start \
  --auth admin:changeme \
  --accept-license \
  -run-as-root

# Expected output:
# Init script installed at /etc/init.d/splunk
# Splunk has been successfully configured to run at boot.
# To start Splunk manually, run the following command:
# /opt/splunk/bin/splunk start -run-as-root
```

### 4.2 Start Splunk Service

```bash
# Start Splunk immediately
sudo /opt/splunk/bin/splunk start -run-as-root --accept-license

# This takes 20-30 seconds...
# Wait for output:
# "Splunk has been started with the following configuration..."
# "Web UI listening on: http://localhost:8000"

# Watch the startup (optional)
tail -f /opt/splunk/var/log/splunk/splunkd.log
# Press Ctrl+C to stop watching
```

### 4.3 Verify Splunk Running

```bash
# Check running processes
ps aux | grep splunk | grep -v grep

# Expected output shows:
# /opt/splunk/bin/splunkd -p 8089 ...
# /opt/splunk/bin/python -m splunk.appserver ...

# Check port 8000 (web UI)
sudo netstat -tlnp | grep 8000

# Expected output:
# tcp  0  0 0.0.0.0:8000  0.0.0.0:*  LISTEN  12345/splunk

# Or use ss command (newer)
sudo ss -tlnp | grep 8000
```

### 4.4 Check Splunk Logs

```bash
# View recent log entries
tail -20 /opt/splunk/var/log/splunk/splunkd.log

# Watch logs in real-time
tail -f /opt/splunk/var/log/splunk/splunkd.log
# Press Ctrl+C to stop

# Check for errors
grep ERROR /opt/splunk/var/log/splunk/splunkd.log
```

---

##  Step 5: Access Splunk Web Interface (5-10 minutes)

### 5.1 Find Your Ubuntu IP Address

```bash
# List network interfaces
ip addr show

# Look for inet (not inet6)
# Example output:
# inet 10.0.0.250/24 brd 10.0.0.255 scope global ens33
# ↑ This is your IP (10.0.0.250)

# Quick command to get IP
hostname -I
# Output: 10.0.0.250

# Or check specific interface
ip addr show ens33 | grep "inet "
```

### 5.2 Access Splunk Locally (Same Machine)

```bash
# Method 1: Using localhost
firefox http://localhost:8000 &

# Method 2: Using IP address
firefox http://10.0.0.250:8000 &

# Method 3: Using curl to test
curl -s http://localhost:8000 | head -20
# Should return HTML content
```

### 5.3 Access from Another Machine (Recommended for Better UI)

```
From another VM or your host machine:

Steps:
1. Open web browser (Chrome, Firefox, Edge)
2. Type in address bar: http://10.0.0.250:8000
   (Replace 10.0.0.250 with your Ubuntu IP)
3. Press Enter
4. You should see Splunk login page

Or use curl:
curl http://10.0.0.250:8000 | grep -i splunk
```

### 5.4 First Login

```
Login page will appear

Credentials:
- Username: admin
- Password: changeme

Steps:
1. Click in "Username" field
2. Type: admin
3. Click in "Password" field
4. Type: changeme
5. Click "Sign in"
```

### 5.5 Change Default Password

```
After successful login:

1. Click "Administrator" (top right corner)
2. Select "Account Settings"
3. Change password:
   - Current password: changeme
   - New password: [create strong password]
   - Confirm new password: [repeat]
4. Click "Save"
5. System will log you out
6. Log back in with new password

Save new credentials:
- Username: admin
- New Password: ___________________
```

---

##  Step 6: Post-Installation Setup (10-15 minutes)

### 6.1 Configure Static IP (Recommended)

```bash
# Find current network interface
ip link show
# Usually: eth0, ens33, ens0, enp0s3, etc.

# Current IP and gateway
ip route show
# Look for: default via X.X.X.X

# Get nameservers
cat /etc/resolv.conf

# For Ubuntu 20.04/22.04 using Netplan:
sudo nano /etc/netplan/00-installer-config.yaml

# Or edit primary config:
sudo nano /etc/netplan/01-netcfg.yaml

# Add this content (example):
"""
network:
  version: 2
  renderer: networkd
  ethernets:
    ens33:
      dhcp4: no
      addresses:
        - 10.0.0.250/24
      gateway4: 10.0.0.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
"""

# Apply configuration
sudo netplan apply

# Verify IP changed
ip addr show
# Should show: inet 10.0.0.250/24
```

### 6.2 Configure Firewall

```bash
# Check firewall status
sudo ufw status

# If disabled, skip this
# If enabled, allow Splunk ports

# Allow port 8000 (web UI)
sudo ufw allow 8000/tcp

# Allow port 8089 (forwarder communication)
sudo ufw allow 8089/tcp

# For lab, allow all traffic (NOT for production)
# sudo ufw allow from 192.168.0.0/16

# Verify rules
sudo ufw status

# Expected output:
# 8000/tcp  ALLOW
# 8089/tcp  ALLOW
```

### 6.3 Create Custom Indexes

```bash
# Via command line
# Format: splunk add index [name] -auth [user:pass]

/opt/splunk/bin/splunk add index auth_logs \
  -auth admin:yourpassword

/opt/splunk/bin/splunk add index web_logs \
  -auth admin:yourpassword

/opt/splunk/bin/splunk add index incidents \
  -auth admin:yourpassword

/opt/splunk/bin/splunk add index network \
  -auth admin:yourpassword

# Verify indexes created
/opt/splunk/bin/splunk list index -auth admin:yourpassword

# Or create via web UI:
# Settings > Indexes > New Index
```

### 6.4 Create Additional Users

```bash
# Add analyst user
/opt/splunk/bin/splunk add user analyst \
  -auth admin:yourpassword \
  -password analyst123 \
  -role user

# Add power_user
/opt/splunk/bin/splunk add user power_user \
  -auth admin:yourpassword \
  -password power123 \
  -role power

# List all users
/opt/splunk/bin/splunk list user -auth admin:yourpassword

# Delete user
/opt/splunk/bin/splunk remove user analyst \
  -auth admin:yourpassword
```

---

##  Verification Checklist

After installation, verify everything:

```bash
# ✓ 1. System resources
free -h          # RAM check
df -h /          # Disk check
nproc            # CPU check

# ✓ 2. Installation location
ls -la /opt/splunk/

# ✓ 3. Splunk version
/opt/splunk/bin/splunk version

# ✓ 4. Process running
ps aux | grep splunk | grep -v grep

# ✓ 5. Port listening
sudo netstat -tlnp | grep 8000
sudo ss -tlnp | grep 8000

# ✓ 6. Boot-start enabled
systemctl is-enabled splunk
sudo systemctl status splunk

# ✓ 7. Web UI accessible
curl -s http://localhost:8000 | head -5

# ✓ 8. Network configuration
ip addr show
hostname -I

# ✓ 9. Firewall allows traffic
sudo ufw status

# ✓ 10. Can login
# Open browser: http://localhost:8000
# Login: admin / password
```

---

##  Common Issues & Solutions

### Issue 1: dpkg Installation Fails

```bash
# Error: dependency not satisfied

# Solution 1: Install dependencies
sudo apt install -y libssl-dev libffi-dev

# Solution 2: Fix broken packages
sudo apt --fix-broken install

# Solution 3: Force installation
sudo dpkg --force-all -i splunk-*.deb
```

### Issue 2: Port 8000 Already in Use

```bash
# Check what's using port 8000
sudo lsof -i :8000

# Kill the process
sudo kill -9 [PID]

# Or change Splunk port to 8001
# Edit: /opt/splunk/etc/system/local/inputs.conf
# Change: default_http_server_listener = 127.0.0.1:8000
# To: default_http_server_listener = 127.0.0.1:8001

# Restart
sudo systemctl restart splunk
```

### Issue 3: Can't Access Web UI

```bash
# Check if Splunk is running
ps aux | grep splunk

# If not running:
sudo systemctl start splunk

# Check port
sudo netstat -tlnp | grep 8000

# Test localhost
curl http://localhost:8000

# Check firewall
sudo ufw allow 8000/tcp

# Restart Splunk
sudo systemctl restart splunk
```

### Issue 4: Out of Memory

```bash
# Check available RAM
free -h

# If memory critical:
sudo systemctl stop splunk

# Wait a minute for cleanup
sleep 60

# Restart with limited memory
sudo /opt/splunk/bin/splunk start -run-as-root
```

### Issue 5: Disk Space Full

```bash
# Check disk usage
df -h /

# Check Splunk size
du -sh /opt/splunk/

# Clean old indexes
# Via UI: Settings > Indexes > [index] > Clean

# Or delete manually
sudo rm -rf /opt/splunk/var/lib/splunk/defaultdb/db/*old*

# Restart
sudo systemctl restart splunk
```

---

##  Management Commands

```bash
# Start Splunk
sudo systemctl start splunk
# Or: sudo /opt/splunk/bin/splunk start -run-as-root

# Stop Splunk
sudo systemctl stop splunk
# Or: sudo /opt/splunk/bin/splunk stop

# Restart Splunk
sudo systemctl restart splunk
# Or: sudo /opt/splunk/bin/splunk restart -run-as-root

# Check status
sudo systemctl status splunk

# Enable auto-start
sudo systemctl enable splunk

# Disable auto-start
sudo systemctl disable splunk

# View logs
sudo tail -f /opt/splunk/var/log/splunk/splunkd.log

# Check version
/opt/splunk/bin/splunk version

# Change password
/opt/splunk/bin/splunk edit user admin \
  -auth admin:oldpass \
  -password newpass
```

---

##  Important File Locations

```
Installation Root: /opt/splunk/
Executable: /opt/splunk/bin/splunk
Configuration: /opt/splunk/etc/system/
User Config: /opt/splunk/etc/system/local/
Data/Indexes: /opt/splunk/var/lib/splunk/
Logs: /opt/splunk/var/log/splunk/
Apps: /opt/splunk/etc/apps/
Backup: /opt/splunk/var/lib/splunk/backup/
Init Script: /etc/init.d/splunk
Systemd Service: /etc/systemd/system/splunk.service
```

---

##  Installation Timeline

| Phase | Time | Tasks |
|-------|------|-------|
| System Prep | 5-10 min | Update, check resources |
| Download | 10-20 min | Download Splunk (500MB+) |
| Installation | 5-15 min | Install .deb package |
| Configuration | 10-15 min | Boot-start, indexes |
| Verification | 5-10 min | Test all components |
| **TOTAL** | **35-70 min** | **Full setup** |

---

##  Next Steps After Installation

1. **Ingest Sample Logs** (20-30 minutes)
   ```
   Settings > Add Data > Upload
   Upload: auth.log, apache logs, etc.
   ```

2. **Create Dashboards** (30-45 minutes)
   ```
   Dashboards > Create New Dashboard
   Add panels from searches
   ```

3. **Write SPL Queries** (1-2 hours)
   ```
   Search and Reporting > Learn SPL
   Create: 30+ practice queries
   ```

4. **Configure Alerts** (30-45 minutes)
   ```
   Settings > Alerts > New Alert
   Create: 5+ detection alerts
   ```

5. **Set Up Forwarder** (1-2 hours)
   ```
   Install forwarder on another system
   Configure log forwarding
   ```

---

##  Quick Start After Installation

```bash
# 1. Verify Splunk is running
sudo systemctl status splunk

# 2. Access web UI
# Open: http://localhost:8000
# Login: admin / yourpassword

# 3. Upload sample logs
# Settings > Add Data > Upload > Select file

# 4. Create custom index
# Settings > Indexes > New Index > auth_logs

# 5. Run first query
# Search and Reporting > index=main

# 6. Create alert
# Settings > Alerts > New Alert
```

---

##  Troubleshooting Resources

- **Splunk Installation Docs**: https://docs.splunk.com/Documentation/Splunk/latest/Installation
- **Splunk Linux Admin**: https://docs.splunk.com/Documentation/Splunk/latest/Installation/InstallonLinux
- **Splunk Answers**: https://answers.splunk.com
- **Ubuntu Docs**: https://ubuntu.com/server/docs

---

##  Final Checklist

```
Before using Splunk in production:

☑ Splunk installed at /opt/splunk/
☑ Processes running (splunkd, python)
☑ Port 8000 listening
☑ Can login with admin credentials
☑ Custom indexes created
☑ Static IP configured
☑ Firewall rules set
☑ Boot-start enabled
☑ Backup strategy planned
☑ Monitoring configured
```

---

##  Installation Complete!

**Congratulations! Splunk is now installed and running on Ubuntu!**

### Your Splunk Details:
- **Installation Path**: /opt/splunk/
- **Web UI**: http://localhost:8000
- **Admin User**: admin
- **Port**: 8000 (web), 8089 (forwarder)
- **Status**: ✅ Running
- **Boot-start**: ✅ Enabled

### Next Actions:
1. Upload sample logs
2. Create custom indexes
3. Learn SPL queries
4. Configure alerts
5. Set up log forwarding

---

**Installation Guide Completed**: February 19, 2026
**Splunk Version**: Enterprise 9.0.0+
**Ubuntu Versions**: 20.04 LTS, 22.04 LTS
**Status**: Production-ready
**Time to Complete**: 35-70 minutes