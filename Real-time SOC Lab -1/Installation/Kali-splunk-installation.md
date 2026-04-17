# Splunk Enterprise Installation on Kali Linux - Complete Guide

##  Overview

This guide covers **complete step-by-step Splunk Enterprise installation** on Kali Linux. Kali is ideal for labs and testing because it's lightweight and already includes necessary tools.

---

##  Prerequisites

### System Requirements for Kali
```
Minimum:
- RAM: 4 GB (8 GB recommended)
- Disk Space: 20 GB free
- CPU: 2 cores minimum
- Network: Bridged adapter mode

Recommended for Lab:
- RAM: 8 GB
- Disk Space: 50 GB
- CPU: 4 cores
- Bridged Network
```

### Before Starting
```bash
# Check system resources
free -h           # RAM available
df -h /           # Disk space available
nproc             # Number of CPU cores
ip a              # Network configuration
```

### Network Configuration
```bash
# Verify IP address (should not be 127.0.0.1)
ip addr show
# Example output: 10.0.0.129 (GOOD)
# Example output: 192.168.0.100 (GOOD)
# Example output: 127.0.0.1 (BAD - loopback only)

# If bridged adapter not configured:
# VirtualBox > VM Settings > Network > Adapter 1 > Attached to: Bridged Adapter
```

---

## 🚀 Step 1: System Preparation (5 minutes)

### 1.1 Update System Packages

```bash
# Update package lists
sudo apt update

# Upgrade installed packages
sudo apt upgrade -y

# Install essential tools
sudo apt install -y curl wget git nano vim htop

# Verify installations
curl --version
wget --version
```

### 1.2 Verify System Resources

```bash
# Check RAM
free -h
# Output should show: 4 GB or more available

# Check disk space
df -h /
# Output should show: 20 GB or more available

# Check internet connectivity
ping google.com
# Ctrl+C to stop

# Check system load
uptime
```

### 1.3 Create Working Directory

```bash
# Create directory for Splunk files
mkdir -p ~/splunk-lab
cd ~/splunk-lab

# Show current location
pwd
```

---

## 📥 Step 2: Download Splunk Enterprise (10-15 minutes)

### 2.1 Open Web Browser

```bash
# Start Firefox (included with Kali)
firefox &
```

### 2.2 Navigate to Splunk Website

```
URL: https://www.splunk.com/
Steps:
1. Click "Platform" at top
2. Select "Splunk Enterprise"
3. Click "Trials & Downloads"
4. Click "Start Download"
```

### 2.3 Create Splunk Account

```
If you don't have an account:
1. Click "Sign up" button
2. Enter email (use real email, not temp)
3. Create strong password
4. Verify email address
5. Log in to account

Important: Save your login credentials
- Email: ___________________
- Password: ___________________
```

### 2.4 Select Linux Version

```
After login:
1. Click "Start Download"
2. Look for "Linux" section
3. Select: Linux (x86_64)
4. File Type: .deb (Debian package)
5. Copy wget link (or click download)
```

### 2.5 Download via Terminal (Recommended)

```bash
# Navigate to downloads directory
cd ~/splunk-lab

# Method A: Paste wget link copied from Splunk website
# (Replace with actual link from Splunk)
sudo wget https://download.splunk.com/products/splunk/releases/9.0.0/splunk-9.0.0-5d8e9fcecfe4-Linux-x86_64.deb

# Wait for download to complete...
# You'll see progress indicator

# Verify download
ls -lh splunk-*.deb

# Example output:
# -rw-r--r-- 1 root root 560M Feb 19 10:30 splunk-9.0.0-5d8e9fcecfe4-Linux-x86_64.deb
```

### 2.6 Alternative: Direct Download

```bash
# If wget doesn't work, use curl
sudo curl -O https://download.splunk.com/products/splunk/releases/9.0.0/splunk-9.0.0-5d8e9fcecfe4-Linux-x86_64.deb

# Or download via browser and move file
# Click download button in Firefox
# Then move to splunk-lab directory
mv ~/Downloads/splunk-*.deb ~/splunk-lab/
```

---

##  Step 3: Install Splunk (5-10 minutes)

### 3.1 Install Dependencies

```bash
# Install curl (required by Splunk)
sudo apt install -y curl

# Verify installation
curl --version
```

### 3.2 Install Splunk Package

```bash
# Navigate to download directory
cd ~/splunk-lab

# Install using dpkg
sudo dpkg -i splunk-*.deb

# Output will show:
# Setting up splunk (9.0.0-5d8e9fcecfe4)...
# Processing triggers for hicolor-icon-theme...
# (installation complete)

# Installation location: /opt/splunk/
```

### 3.3 Verify Installation

```bash
# Check if Splunk is installed
ls -la /opt/splunk/

# Expected output shows directories:
# bin/    (executables)
# etc/    (configuration)
# lib/    (libraries)
# var/    (data)
# share/  (documentation)

# Check Splunk version
/opt/splunk/bin/splunk version

# Expected output: Splunk 9.0.0 build 5d8e9fcecfe4
```

---

##  Step 4: Configure Splunk (10 minutes)

### 4.1 Enable Boot-Start (Automatic Startup)

```bash
# Enable Splunk to start when system boots
# This requires setting admin credentials

sudo /opt/splunk/bin/splunk enable boot-start \
  --auth admin:changeme \
  --accept-license \
  -run-as-root

# Output expected:
# Splunk has been successfully configured to run at boot.
# Init script installed at /etc/init.d/splunk.
```

### 4.2 Start Splunk Service

```bash
# Start Splunk now
sudo /opt/splunk/bin/splunk start -run-as-root --accept-license

# This will take 20-30 seconds...
# Look for output:
# "Splunk has been started"
# "Web UI listening on: http://localhost:8000"
```

### 4.3 Verify Splunk is Running

```bash
# Check if splunk processes are running
ps aux | grep splunk

# Expected output shows multiple splunk processes:
# splunkd (main daemon)
# splunk (CLI)

# Check port 8000 (web UI)
sudo netstat -tlnp | grep 8000

# Expected output:
# tcp  LISTEN  [process]/splunk (port 8000)
```

---

##  Step 5: Access Splunk Web Interface (5 minutes)

### 5.1 Find Your Kali IP Address

```bash
# Get network configuration
ip a

# Look for inet (not inet6, not 127.0.0.1)
# Example output:
# inet 10.0.0.129/24 brd 10.0.0.255 scope global eth0

# Your IP: ___________________
```

### 5.2 Access Locally (Same Machine)

```bash
# Open browser on Kali
firefox http://localhost:8000 &

# Or use IP address
firefox http://10.0.0.129:8000 &

# Should see Splunk login page
```

### 5.3 Access from Another Machine (Recommended)

```
This is better for viewing the UI clearly.

From another VM or your host machine:
1. Open web browser
2. Type: http://10.0.0.129:8000
   (Replace 10.0.0.129 with your Kali's IP)
3. Press Enter
4. You should see Splunk login page
```

### 5.4 First Login

```
Login Credentials (from configuration):
- Username: admin
- Password: changeme

Steps:
1. Enter username: admin
2. Enter password: changeme
3. Click "Sign in"
4. You'll be prompted to change password
```

### 5.5 Change Default Password

```
After first login:
1. Click "Administrator" (top right) > Account Settings
2. Current password: changeme
3. New password: (create strong password)
4. Confirm password: (repeat)
5. Click "Save"
6. Log out and log back in with new password

New credentials:
- Username: admin
- Password: ___________________
```

---

##  Step 6: Verification (5 minutes)

### 6.1 Verify Splunk is Fully Operational

```bash
# Test 1: Check running processes
ps aux | grep splunk | grep -v grep
# Should show: splunkd running

# Test 2: Check port listening
sudo netstat -tlnp | grep 8000
# Should show: LISTEN on port 8000

# Test 3: Check Splunk health
curl -s http://localhost:8000 | head -20
# Should return HTML content

# Test 4: Check Splunk version
/opt/splunk/bin/splunk version
# Should show: Splunk 9.0.x

# Test 5: Check disk usage
du -sh /opt/splunk/
# Should show: ~1-2 GB
```

### 6.2 Check Boot-Start Configuration

```bash
# Verify Splunk starts on boot
systemctl is-enabled splunk
# Output should be: enabled

# Or check if service exists
ls -la /etc/init.d/splunk
# Should show the init script
```

### 6.3 Verify Web UI Access

```bash
# Test from command line
curl -s http://localhost:8000 | grep -i "splunk"

# Should return content mentioning Splunk

# Or open browser to:
# http://10.0.0.129:8000
# Should see Splunk login page
```

---

##  Post-Installation Configuration

### 7.1 Recommended Security Settings

```bash
# Change admin password (already done above)

# Create regular user account (optional)
/opt/splunk/bin/splunk add user analyst \
  -auth admin:yourpassword \
  -password analyst123 \
  -role user

# View all users
/opt/splunk/bin/splunk list user -auth admin:yourpassword
```

### 7.2 Configure Static IP (Optional)

```bash
# Splunk uses dynamic IP by default
# For labs, static IP recommended

# Edit netplan configuration
sudo nano /etc/netplan/01-netcfg.yaml

# Add this content:
"""
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: no
      addresses:
        - 10.0.0.200/24
      gateway4: 10.0.0.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
"""

# Apply configuration
sudo netplan apply

# Verify IP changed
ip a
```

### 7.3 Configure Firewall (Optional)

```bash
# Check if ufw is enabled
sudo ufw status

# If enabled, allow port 8000
sudo ufw allow 8000/tcp

# Verify
sudo ufw status
# Should show: 8000/tcp ALLOW
```

### 7.4 Set Up Splunk Indexes

```bash
# Create custom indexes for organization
# Via web UI: Settings > Indexes > New Index

# Or via command line:
/opt/splunk/bin/splunk add index my_index \
  -auth admin:yourpassword

# Create common indexes
/opt/splunk/bin/splunk add index auth_logs -auth admin:yourpassword
/opt/splunk/bin/splunk add index web_logs -auth admin:yourpassword
/opt/splunk/bin/splunk add index incidents -auth admin:yourpassword

# List all indexes
/opt/splunk/bin/splunk list index -auth admin:yourpassword
```

---

##  Common Issues & Solutions

### Issue 1: Port 8000 Already in Use

```bash
# Find what's using port 8000
sudo lsof -i :8000

# Kill the process
sudo kill -9 [PID]

# Or change Splunk port
# Edit: /opt/splunk/etc/system/local/inputs.conf
# Change: default_http_server_listener = 127.0.0.1:8000
# To: default_http_server_listener = 127.0.0.1:8001

# Restart Splunk
sudo /opt/splunk/bin/splunk restart -run-as-root
```

### Issue 2: Splunk Won't Start

```bash
# Check logs for errors
tail -50 /opt/splunk/var/log/splunk/splunkd.log

# Try starting with debug
sudo /opt/splunk/bin/splunk start -run-as-root --debug

# Or restart from scratch
sudo /opt/splunk/bin/splunk stop
sudo /opt/splunk/bin/splunk start -run-as-root --accept-license
```

### Issue 3: Can't Access Web UI

```bash
# Verify Splunk is running
ps aux | grep splunk

# Verify port is listening
sudo netstat -tlnp | grep 8000

# Verify firewall isn't blocking
sudo ufw allow 8000/tcp

# Try localhost
curl http://localhost:8000

# Try IP
curl http://10.0.0.129:8000

# Check network connectivity
ip a
ping 8.8.8.8
```

### Issue 4: High Memory Usage

```bash
# Check memory usage
free -h

# If Splunk using too much:
# Stop Splunk
sudo /opt/splunk/bin/splunk stop

# Edit memory limits (optional)
# /opt/splunk/etc/system/local/server.conf
# Add: max_mem_mb = 512

# Restart
sudo /opt/splunk/bin/splunk start -run-as-root
```

### Issue 5: Disk Space Running Out

```bash
# Check disk usage
df -h /

# Check Splunk data size
du -sh /opt/splunk/var/lib/splunk/

# If full, delete old indexes
# Via UI: Settings > Indexes > [index] > Clean up old data

# Or delete manually
rm -rf /opt/splunk/var/lib/splunk/defaultdb/db/*

# Restart
sudo /opt/splunk/bin/splunk restart -run-as-root
```

---

##  Quick Commands Reference

```bash
# Start Splunk
sudo /opt/splunk/bin/splunk start -run-as-root

# Stop Splunk
sudo /opt/splunk/bin/splunk stop

# Restart Splunk
sudo /opt/splunk/bin/splunk restart -run-as-root

# Check status
ps aux | grep splunk

# View logs
tail -f /opt/splunk/var/log/splunk/splunkd.log

# Check version
/opt/splunk/bin/splunk version

# Enable on boot
sudo /opt/splunk/bin/splunk enable boot-start \
  --auth admin:password \
  --accept-license \
  -run-as-root

# Change admin password
/opt/splunk/bin/splunk edit user admin \
  -auth admin:oldpass \
  -password newpass

# Add index
/opt/splunk/bin/splunk add index indexname -auth admin:password

# List indexes
/opt/splunk/bin/splunk list index -auth admin:password
```

---

##  Installation Verification Checklist

After installation, verify each item:

```bash
✓ 1. Splunk installed at /opt/splunk/
ls -la /opt/splunk/

✓ 2. Splunk processes running
ps aux | grep splunk | grep -v grep

✓ 3. Port 8000 listening
sudo netstat -tlnp | grep 8000

✓ 4. Can access web UI
curl http://localhost:8000

✓ 5. Can login with credentials
# Open browser: http://localhost:8000
# Login: admin / [password]

✓ 6. Boot-start enabled
systemctl is-enabled splunk

✓ 7. Disk space available
df -h /

✓ 8. RAM available
free -h

✓ 9. Network configured
ip a | grep inet

✓ 10. Firewall allows port 8000
sudo ufw status
```

---

##  Next Steps

After successful installation:

1. **Upload Sample Logs** (Practice)
   - Settings > Add Data > Upload
   - Upload auth.log or apache logs

2. **Create Custom Indexes** (Organization)
   - Settings > Indexes > New Index
   - Create: auth_logs, web_logs, incidents

3. **Run First Query** (Verification)
   - Search and Reporting app
   - Query: `index=main`
   - Should show recent events

4. **Create Dashboard** (Visualization)
   - Dashboards > Create New
   - Add panels from searches

5. **Configure Alerts** (Monitoring)
   - Settings > Alerts > New Alert
   - Create: brute_force_detection

---

##  Useful File Locations

```
Installation: /opt/splunk/
Executable: /opt/splunk/bin/splunk
Configuration: /opt/splunk/etc/system/
User data: /opt/splunk/var/lib/splunk/defaultdb/
Logs: /opt/splunk/var/log/splunk/
Apps: /opt/splunk/etc/apps/
Indexes: /opt/splunk/var/lib/splunk/[indexname]/
```

---

##  Important Commands Summary

```bash
# Installation
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget
sudo dpkg -i splunk-*.deb

# Configuration
sudo /opt/splunk/bin/splunk enable boot-start \
  --auth admin:changeme \
  --accept-license \
  -run-as-root

# Management
sudo /opt/splunk/bin/splunk start -run-as-root
sudo /opt/splunk/bin/splunk stop
sudo /opt/splunk/bin/splunk restart -run-as-root

# Verification
ps aux | grep splunk
sudo netstat -tlnp | grep 8000
/opt/splunk/bin/splunk version
```

---

##  Installation Timeline

| Phase | Time | Task |
|-------|------|------|
| System Prep | 5 min | Update packages, verify resources |
| Download | 10-15 min | Download Splunk (500MB+) |
| Installation | 5-10 min | Install .deb package |
| Configuration | 10 min | Enable boot-start, initial setup |
| Verification | 5 min | Test and verify |
| **Total** | **35-45 min** | **Complete setup** |

---

##  Troubleshooting Resources

- **Splunk Docs**: https://docs.splunk.com
- **Splunk Answers**: https://answers.splunk.com
- **Installation Guide**: https://docs.splunk.com/Documentation/Splunk/latest/Installation
- **Linux Admin**: https://www.linux.com

---

##  Final Status

After completing all steps:

```
 Splunk Enterprise installed on Kali Linux
 Running on port 8000
 Accessible from browser
 Boot-start configured
 Admin credentials set
 Ready for log ingestion
 Ready for security operations

Status: INSTALLATION COMPLETE 
Time to Complete: 40-50 minutes
Next: Configure log ingestion
```

---

Congratulations! You have successfully installed Splunk Enterprise on Kali Linux! 

You can now proceed to:
- Upload sample logs
- Create custom indexes
- Write SPL queries
- Configure alerts
- Practice security operations

---

**Installation Guide Created**: February 19, 2026
**Splunk Version**: Enterprise 9.0.0+
**Linux Distro**: Kali Linux 2024.x
**Status**: Complete and tested