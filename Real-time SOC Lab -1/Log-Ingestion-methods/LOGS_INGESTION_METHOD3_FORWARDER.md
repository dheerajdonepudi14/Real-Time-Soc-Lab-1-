# Splunk Log Ingestion - METHOD 3: UNIVERSAL FORWARDER METHOD

## 🚀 Overview

The **Universal Forwarder** allows you to send logs from remote machines to a central Splunk instance. Perfect for:
- Multi-machine monitoring
- Centralized logging
- Production deployments
- Distributed security operations

**Architecture:**
```
Ubuntu Machine (Forwarder) ──→ Kali Machine (Splunk Instance)
    ├─ /var/log/auth.log  →  Port 1818
    ├─ /var/log/syslog    →
    └─ /var/log/apache2/  →
```

---

## PART 1: INSTALL SPLUNK UNIVERSAL FORWARDER ON UBUNTU

### Step 1.1: Download Splunk Forwarder

**On Ubuntu Machine:**

```bash
# Create directory for download
mkdir -p ~/splunk-forwarder
cd ~/splunk-forwarder

# Download Splunk Universal Forwarder
# Go to: https://www.splunk.com/en_us/download/universal-forwarder.html

# Or use wget (paste link from website)
sudo wget https://download.splunk.com/products/universalforwarder/releases/9.0.0/splunk-universal-forwarder-9.0.0-5d8e9fcecfe4-Linux-x86_64.deb

# Wait for download (takes 5-10 minutes, ~100MB file)
# Verify download
ls -lh splunk-universal-forwarder-*.deb

# Expected: ~100MB file
```

### Step 1.2: Install Forwarder

```bash
# Navigate to download directory
cd ~/splunk-forwarder

# Install using dpkg
sudo dpkg -i splunk-universal-forwarder-*.deb

# Expected output:
# Setting up splunk-universal-forwarder (9.0.0) ...
# Forwarder installed at: /opt/splunkforwarder/

# Verify installation
ls -la /opt/splunkforwarder/

# Should show:
# bin/    (executables)
# etc/    (configuration)
# lib/    (libraries)
# var/    (data)
```

### Step 1.3: Check Forwarder Version

```bash
/opt/splunkforwarder/bin/splunk version

# Expected output:
# splunk-universal-forwarder 9.0.0 build 5d8e9fcecfe4
```

---

## PART 2: CONFIGURE FORWARDER TO SEND LOGS

### Step 2.1: Start Forwarder Service

```bash
# Enable boot-start
sudo /opt/splunkforwarder/bin/splunk enable boot-start \
  --auth admin:changeme \
  --accept-license \
  -run-as-root

# Start forwarder
sudo /opt/splunkforwarder/bin/splunk start -run-as-root --accept-license

# Wait for startup (takes 20-30 seconds)
# Look for: "Forwarder has been started"
```

### Step 2.2: Verify Forwarder Running

```bash
# Check processes
ps aux | grep splunk | grep -v grep

# Should show:
# /opt/splunkforwarder/bin/splunkd -p 8089

# Check port (8089 is forwarder communication port)
sudo netstat -tlnp | grep 8089

# Should show:
# tcp  LISTEN  [process]/splunkd port 8089
```

### Step 2.3: Configure Where to Send Logs

**Goal:** Tell forwarder to send logs to your Splunk instance (Kali machine)

**Find Kali Machine IP:**

On Kali machine:
```bash
ip addr show

# Look for inet (not 127.0.0.1)
# Example: 10.0.0.129
```

**On Ubuntu, Configure Forwarder:**

```bash
# Add forward server (where to send logs)
# Format: /opt/splunkforwarder/bin/splunk add forward-server [IP]:[PORT]

sudo /opt/splunkforwarder/bin/splunk add forward-server 10.0.0.129:1818 \
  -auth admin:changeme

# Replace 10.0.0.129 with your actual Kali IP
# Port 1818 is Splunk's default forwarder listening port

# Expected output:
# Forward server configured successfully
```

### Step 2.4: Verify Forward Server Configuration

```bash
# List configured forward servers
sudo /opt/splunkforwarder/bin/splunk list forward-server \
  -auth admin:changeme

# Expected output:
# Active forward server: 10.0.0.129:1818
```

---

## PART 3: CONFIGURE WHAT LOGS TO SEND

### Step 3.1: Tell Forwarder Which Files to Monitor

```bash
# Add monitor for auth.log
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/auth.log \
  -index ub_logs \
  -auth admin:changeme

# Add monitor for syslog
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/syslog \
  -index ub_logs \
  -auth admin:changeme

# Add monitor for apache (if installed)
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/apache2/ \
  -index ub_logs \
  -auth admin:changeme

# Expected output for each:
# Monitor configured successfully
```

**What This Does:**
- `/var/log/auth.log`: Monitor authentication logs
- `/var/log/syslog`: Monitor system logs
- `/var/log/apache2/`: Monitor web server logs
- `-index ub_logs`: Create index called "ub_logs" on Splunk
- `-auth admin:changeme`: Use these credentials

### Step 3.2: Verify Monitors Configured

```bash
# List all monitors
sudo /opt/splunkforwarder/bin/splunk list monitor \
  -auth admin:changeme

# Expected output:
# /var/log/auth.log  [Index: ub_logs]
# /var/log/syslog    [Index: ub_logs]
# /var/log/apache2/  [Index: ub_logs]
```

### Step 3.3: Restart Forwarder to Apply Changes

```bash
# Restart forwarder
sudo /opt/splunkforwarder/bin/splunk restart -run-as-root

# Wait for restart (takes 20-30 seconds)
# Watch for: "Forwarder has been restarted"
```

---

## PART 4: CONFIGURE SPLUNK TO RECEIVE FORWARDER DATA

### Step 4.1: Enable Receiving on Splunk Instance (Kali)

On Kali machine with Splunk:

```bash
# Configure Splunk to listen on port 1818
sudo /opt/splunk/bin/splunk add forward-server :1818 \
  -auth admin:changeme

# Wait, that's wrong. Use this instead:

# Configure receiving (listening) port
sudo /opt/splunk/bin/splunk add receive-server :1818 \
  -auth admin:changeme

# Expected output:
# Receiving port 1818 configured successfully
```

**Alternative (via Splunk UI):**

1. Open Splunk: `http://localhost:8000`
2. Settings > Forwarding and Receiving
3. Configure Receiving > Add Port
4. Enter Port: 1818
5. Save

### Step 4.2: Create Index on Splunk

On Kali Splunk instance:

```bash
# Create index to receive forwarder data
sudo /opt/splunk/bin/splunk add index ub_logs \
  -auth admin:changeme

# Expected output:
# Index ub_logs created successfully
```

**Alternative (via Splunk UI):**

1. Settings > Indexes
2. New Index
3. Name: ub_logs
4. Click Save

### Step 4.3: Restart Splunk

```bash
# Restart to apply receiving port configuration
sudo /opt/splunk/bin/splunk restart -run-as-root

# Wait 30 seconds for full restart
```

### Step 4.4: Verify Splunk Listening

```bash
# Check port 1818 listening
sudo netstat -tlnp | grep 1818

# Should show:
# tcp  LISTEN  [process]/splunk  port 1818
```

---

## PART 5: TEST FORWARDER COMMUNICATION

### Step 5.1: Generate Activity on Ubuntu

On Ubuntu machine:

```bash
# Generate login attempts
ssh root@localhost  # fails
ssh admin@localhost # fails
ssh user@localhost  # fails

# These create entries in auth.log
# Forwarder will pick them up and send to Splunk
```

### Step 5.2: Check Forwarder Logs

On Ubuntu:

```bash
# View forwarder activity
tail -20 /opt/splunkforwarder/var/log/splunk/splunkd.log

# Look for messages about:
# "forwarding to 10.0.0.129:1818"
# "events processed"
# "connection successful"

# Example output:
# ... forward connection established
# ... forwarding 15 events
# ... connection to 10.0.0.129:1818 active
```

### Step 5.3: Search on Splunk

On Kali Splunk instance:

1. Open Splunk: `http://localhost:8000`
2. Go to Search and Reporting app
3. Enter search: `index=ub_logs`
4. Click Search

**Expected Results:**
```
Results: 15 events
Time Range: Last 24 hours

Event 1:
Feb 20 10:15:23 ubuntu sshd[3000]: Failed password for root from 127.0.0.1

Event 2:
Feb 20 10:15:31 ubuntu sshd[3001]: Failed password for admin from 127.0.0.1

Event 3:
Feb 20 10:15:39 ubuntu sshd[3002]: Failed password for user from 127.0.0.1
...
```

**Congratulations!** Logs are being forwarded successfully! ✅

---

## PART 6: ADVANCED FORWARDER CONFIGURATION

### Option 1: Forward to Multiple Destinations

On Ubuntu forwarder:

```bash
# Add second Splunk instance
sudo /opt/splunkforwarder/bin/splunk add forward-server 192.168.1.200:1818 \
  -auth admin:changeme

# Now forwards to:
# 1. 10.0.0.129:1818 (original)
# 2. 192.168.1.200:1818 (second instance)

# Logs sent to both destinations
```

### Option 2: Forward Specific Source Types Only

```bash
# Edit forwarder configuration
sudo nano /opt/splunkforwarder/etc/system/local/inputs.conf

# Add filter (example):
[monitor:///var/log/auth.log]
sourcetype = auth
index = ub_logs
whitelist = .auth
```

### Option 3: Load Balance Between Splunk Instances

```bash
# Configure load balancing
sudo /opt/splunkforwarder/bin/splunk add load-balance \
  10.0.0.129:1818 192.168.1.200:1818 \
  -auth admin:changeme

# Distributes load across two instances
```

### Option 4: Filter/Drop Certain Logs

```bash
# Edit inputs.conf
sudo nano /opt/splunkforwarder/etc/system/local/inputs.conf

# Add filter to drop certain events:
[monitor:///var/log/auth.log]
sourcetype = auth
index = ub_logs
route = hasData
```

---

## PART 7: PRACTICAL FORWARDER SCENARIOS

### Scenario 1: Three-Machine Setup

```
Machine 1 (Ubuntu) ──┐
                      ├──→ Kali (Splunk) ← Central logging
Machine 2 (Ubuntu) ──┘

Configuration:
Ubuntu 1:
- Forward to 10.0.0.129:1818
- Send: /var/log/auth.log, /var/log/syslog

Ubuntu 2:
- Forward to 10.0.0.129:1818
- Send: /var/log/apache2/
```

**Setup Ubuntu 2:**

```bash
# Install forwarder (same as before)
sudo dpkg -i splunk-universal-forwarder-*.deb

# Enable boot-start
sudo /opt/splunkforwarder/bin/splunk enable boot-start \
  --auth admin:changeme \
  --accept-license \
  -run-as-root

# Add forward server
sudo /opt/splunkforwarder/bin/splunk add forward-server 10.0.0.129:1818 \
  -auth admin:changeme

# Add monitors
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/apache2/ \
  -index ub_logs2 \
  -auth admin:changeme

# Restart
sudo /opt/splunkforwarder/bin/splunk restart -run-as-root
```

**Search on Splunk:**

```spl
# Logs from Machine 1
index=ub_logs host=ubuntu-1

# Logs from Machine 2
index=ub_logs2 host=ubuntu-2

# All logs from all machines
index=ub_logs OR index=ub_logs2
| stats count by host
```

### Scenario 2: Real-Time Centralized Monitoring

**Goal:** Monitor 5 Ubuntu servers from one central Splunk instance

**Setup (repeat on each Ubuntu):**

```bash
# Each Ubuntu machine:
# 1. Install forwarder
# 2. Configure to send to central Splunk
# 3. Monitor their local logs
```

**On Central Splunk:**

```spl
# See all servers' logs
index=ub_logs
| stats count by host

# Find failed logins across all servers
index=ub_logs "Failed password"
| stats count by host, src_ip

# Real-time security dashboard
index=ub_logs
| timechart count by host
```

---

## PART 8: TROUBLESHOOTING FORWARDER

### Problem: Forwarder Won't Start

**Symptom:** `Forwarder did not start`

**Check:**
```bash
# Check status
ps aux | grep splunkforwarder | grep -v grep

# Check logs
tail -50 /opt/splunkforwarder/var/log/splunk/splunkd.log

# Try starting manually
sudo /opt/splunkforwarder/bin/splunk start -run-as-root --accept-license
```

**Solution:**
```bash
# Restart from scratch
sudo /opt/splunkforwarder/bin/splunk stop
sleep 10
sudo /opt/splunkforwarder/bin/splunk start -run-as-root --accept-license
```

### Problem: Logs Not Appearing on Splunk

**Check 1: Forwarder has forward server configured**
```bash
sudo /opt/splunkforwarder/bin/splunk list forward-server \
  -auth admin:changeme

# Should show: 10.0.0.129:1818
```

**Check 2: Splunk listening on port 1818**
```bash
# On Kali (Splunk instance)
sudo netstat -tlnp | grep 1818

# Should show port listening
```

**Check 3: Network connectivity**
```bash
# From Ubuntu to Kali
nc -zv 10.0.0.129 1818

# Should show: Connection successful
```

**Check 4: Forwarder monitors configured**
```bash
sudo /opt/splunkforwarder/bin/splunk list monitor \
  -auth admin:changeme

# Should show monitored paths
```

**Check 5: Splunk has receiving port configured**
```bash
# On Kali Splunk
sudo netstat -tlnp | grep 1818

# Should show listening
```

### Problem: High CPU Usage

**Cause:** Forwarder monitoring too much

**Solution:**
```bash
# Monitor specific files, not directories
sudo /opt/splunkforwarder/bin/splunk remove monitor /var/log/ \
  -auth admin:changeme

sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/auth.log \
  -auth admin:changeme

sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/syslog \
  -auth admin:changeme

# Restart
sudo /opt/splunkforwarder/bin/splunk restart -run-as-root
```

### Problem: Disk Space Running Out

**Cause:** Splunk storing too many events

**Solution:**

On Kali Splunk:
```bash
# Limit index size
sudo /opt/splunk/bin/splunk add index ub_logs maxKBps=1000 \
  -auth admin:changeme

# Delete old data
sudo /opt/splunk/bin/splunk clean eventdata -index ub_logs \
  -auth admin:changeme
```

---

## PART 9: COMPLETE FORWARDER SETUP SCRIPT

### One-Command Setup (Ubuntu)

```bash
#!/bin/bash

# Colors
GREEN='\033[0;32m'
BLUE='\033[0;34m'
NC='\033[0m'

echo -e "${BLUE}Installing Splunk Universal Forwarder...${NC}"

# Download
cd ~/Downloads
wget https://download.splunk.com/products/universalforwarder/releases/9.0.0/splunk-universal-forwarder-9.0.0-5d8e9fcecfe4-Linux-x86_64.deb

# Install
echo -e "${BLUE}Installing package...${NC}"
sudo dpkg -i splunk-universal-forwarder-*.deb

# Enable boot-start
echo -e "${BLUE}Enabling boot-start...${NC}"
sudo /opt/splunkforwarder/bin/splunk enable boot-start \
  --auth admin:changeme \
  --accept-license \
  -run-as-root

# Start
echo -e "${BLUE}Starting forwarder...${NC}"
sudo /opt/splunkforwarder/bin/splunk start -run-as-root --accept-license

# Configure (change IP to your Splunk IP)
SPLUNK_IP="10.0.0.129"
echo -e "${BLUE}Configuring forward server: $SPLUNK_IP${NC}"
sudo /opt/splunkforwarder/bin/splunk add forward-server $SPLUNK_IP:1818 \
  -auth admin:changeme

# Add monitors
echo -e "${BLUE}Adding monitors...${NC}"
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/auth.log \
  -index ub_logs \
  -auth admin:changeme

sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/syslog \
  -index ub_logs \
  -auth admin:changeme

# Restart
echo -e "${BLUE}Restarting forwarder...${NC}"
sudo /opt/splunkforwarder/bin/splunk restart -run-as-root

# Verify
echo -e "${GREEN}✓ Forwarder installation complete!${NC}"
echo -e "${GREEN}✓ Forward server: $SPLUNK_IP:1818${NC}"
echo -e "${GREEN}✓ Monitors: auth.log, syslog${NC}"
echo -e "${BLUE}Search on Splunk: index=ub_logs${NC}"
```

**Save and run:**
```bash
chmod +x setup-forwarder.sh
./setup-forwarder.sh
```

---

## QUICK REFERENCE TABLE

| Task | Command |
|------|---------|
| Download forwarder | `wget https://download.splunk.com/...forwarder...deb` |
| Install | `sudo dpkg -i splunk-universal-forwarder-*.deb` |
| Enable boot-start | `sudo /opt/splunkforwarder/bin/splunk enable boot-start ...` |
| Start forwarder | `sudo /opt/splunkforwarder/bin/splunk start -run-as-root` |
| Add forward server | `sudo /opt/splunkforwarder/bin/splunk add forward-server [IP]:1818` |
| Add monitor | `sudo /opt/splunkforwarder/bin/splunk add monitor /path/to/log` |
| List monitors | `sudo /opt/splunkforwarder/bin/splunk list monitor` |
| Restart forwarder | `sudo /opt/splunkforwarder/bin/splunk restart -run-as-root` |
| View logs | `tail -f /opt/splunkforwarder/var/log/splunk/splunkd.log` |

---

## SUMMARY

**Forwarder Architecture:**
```
Ubuntu Machine (logs)
    ↓
    └─→ Splunk Universal Forwarder
        ├─ Monitor /var/log/auth.log
        ├─ Monitor /var/log/syslog
        └─ Monitor /var/log/apache2/
            ↓
            └─→ Send to Kali (Port 1818)
                ↓
                └─→ Splunk Instance
                    └─ Index: ub_logs
                       └─ Searchable and Monitored
```

**Time to Setup:**
- Download: 5-10 minutes
- Install: 2-3 minutes
- Configure: 5 minutes
- **Total: 12-18 minutes**

---

**Universal Forwarder Method Complete!** ✅

You now know how to:
- Install Splunk Universal Forwarder
- Configure log forwarding
- Set up receiving on Splunk
- Create indexes for forwarder data
- Monitor multiple machines
- Troubleshoot forwarder issues
- Scale to production setups

**Next:** Learn how to generate attack logs for security testing!
