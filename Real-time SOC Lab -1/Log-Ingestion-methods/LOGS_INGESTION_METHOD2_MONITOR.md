# Splunk Log Ingestion - METHOD 2: MONITOR METHOD

## 📡 Overview

The **Monitor Method** allows Splunk to continuously monitor and ingest logs in real-time from files, folders, or network ports. Perfect for:
- Real-time log monitoring
- Continuous security operations
- Production environments
- Live threat detection

---

## 🎯 MONITOR METHOD OPTIONS

There are three types of monitoring:

```
Monitor Method
├─ Monitor Files/Folders (most common)
├─ Monitor Network Ports (TCP/UDP)
└─ Monitor HTTP endpoints (advanced)
```

---

## PART 1: MONITOR FILES AND FOLDERS

### Step 1.1: Access Splunk Monitor Setup

1. **Open Splunk:** `http://localhost:8000`
2. **Login:** admin / [your password]
3. **Go to Settings:**
   - Click gear icon (top right)
   - Select "Settings"
4. **Click "Add Data"**
5. **Click "Monitor"**
6. **Select "Files & Directories"**

### Step 1.2: Choose What to Monitor

**UI Shows Options:**
```
Monitor
├─ Files & Directories (Local files/folders)
└─ Network Ports (TCP/UDP data)
```

**Select:** "Files & Directories"

### Step 1.3: Enter Path to Monitor

**Input Field: "Path"**

#### Option A: Monitor Single Log File

```
Path: /var/log/auth.log
```

**Example Paths:**
```
/var/log/auth.log         (Linux auth logs)
/var/log/syslog           (System logs)
/var/log/apache2/access.log (Web access logs)
/var/log/apache2/error.log  (Web error logs)
```

#### Option B: Monitor Entire Folder

```
Path: /var/log/
```

**This monitors ALL files in /var/log/**

**Result:** Will ingest auth.log, syslog, kern.log, apache2 folder, mysql folder, etc.

#### Option C: Monitor Specific Pattern

```
Path: /var/log/auth.log*
```

**This monitors:**
- auth.log
- auth.log.1
- auth.log.2
- etc. (rotated files)

### Step 1.4: Review Settings

**Configuration Shows:**
```
Path: /var/log/auth.log
Input name: (auto-filled or custom)
Source type: (auto-detected or select)
Index: main (or select different)
Host: (auto-detected)
Recursive: Yes/No (for folders)
```

**Modify if Needed:**

#### Source Type Selection

```
For auth.log:
├─ syslog (recommended)
├─ auth
├─ linux_secure
└─ other
```

**Select:** syslog or auth

#### Index Selection

```
Default: main
Or select: auth_logs (if created)
```

#### Recursive Option (For Folders)

```
If monitoring /var/log/:
Recursive: Yes (monitors subfolders)
Recursive: No (only /var/log/ top level)
```

### Step 1.5: Submit Monitoring Configuration

**Click "Save"**

**Confirmation:**
```
✅ Monitor configured successfully!
Path: /var/log/auth.log
Source Type: auth
Index: main
Status: Active
```

---

## PART 2: MONITOR NETWORK PORTS

### Step 2.1: Access Network Port Monitor

1. **Settings > Add Data > Monitor**
2. **Select "Network Port"**
3. **Choose protocol:**
   - TCP (more reliable)
   - UDP (faster)

### Step 2.2: Configure Port Monitoring

**Form Shows:**
```
Protocol: TCP
Port: [1-65535]
Source type: (auto-detect)
Index: (select)
Host: (auto-detected)
```

### Step 2.3: Enter Port Number

**Example Setup:**
```
Protocol: TCP
Port: 1212
Source Type: syslog (or other)
Index: ports
```

**Why Port 1212?**
- Unused port
- High number (>1024, no special permissions)
- Easy to remember for testing

### Step 2.4: Test Port Monitoring

**On Local Machine (Kali/Ubuntu):**

Open terminal 1 (start listening):
```bash
# Start monitoring port with Splunk (already done via Splunk UI)
# Or manually test with netcat
nc -l -p 1212
```

Open terminal 2 (send test data):
```bash
# Send data to the port
echo "Test message from nc" | nc localhost 1212
echo "Another test message" | nc localhost 1212
```

**Expected Behavior:**
- Data sent on terminal 2
- Appears in Splunk in real-time
- Shows as new events

### Step 2.5: Verify Port Monitoring

**In Splunk Search:**
```spl
index=ports

# OR (if monitoring port 1212)
sourcetype=tcp:1212
```

**Results Show:**
```
Test message from nc
Another test message
```

---

## PART 3: REAL-TIME MONITORING SETUP (auth.log Example)

### Complete Monitoring Workflow

#### Step 3.1: Configure auth.log Monitoring

1. **Settings > Add Data > Monitor**
2. **Select: Files & Directories**
3. **Path:** `/var/log/auth.log`
4. **Source Type:** `auth` or `syslog`
5. **Index:** `auth_logs`
6. **Save**

#### Step 3.2: Generate Activity to Test

**Terminal 1 (Watch real-time monitoring):**
```bash
# Search in Splunk for new events
# Search and Reporting app
# Query: index=auth_logs earliest=-5m

# This shows logs from last 5 minutes
# Refresh every few seconds
```

**Terminal 2 (Generate activity):**
```bash
# Try SSH login attempts
ssh root@localhost
# (wrong password - generates log)

ssh admin@localhost
# (wrong password - generates log)

sudo ls /root
# (generates sudo log entry)

su - user1
# (generates su attempt)
```

#### Step 3.3: Watch Real-Time Updates

**In Splunk Search Results:**
- New entries appear automatically
- See login attempts in real-time
- Timestamp updates instantly
- No refresh needed (auto-updates)

**Example Real-Time Results:**
```
Feb 20 10:30:15 ubuntu sshd[3000]: Failed password for root from 127.0.0.1 port 55555 ssh2
Feb 20 10:30:23 ubuntu sshd[3001]: Failed password for admin from 127.0.0.1 port 55556 ssh2
Feb 20 10:30:31 ubuntu sudo: user : TTY=pts/0 ; PWD=/home/user ; USER=root ; COMMAND=/bin/ls
Feb 20 10:30:45 ubuntu su: Failed password for user1
```

---

## PART 4: MULTI-FILE MONITORING

### Monitor Multiple Log Files

#### Setup Multiple Monitors

**Option 1: Monitor Entire Log Directory**

1. **Settings > Add Data > Monitor**
2. **Path:** `/var/log/`
3. **Recursive:** Yes
4. **Source Type:** (set to default)
5. **Index:** `system_logs`
6. **Save**

**Result:** Monitors ALL logs in /var/log/ and subfolders:
```
/var/log/auth.log ✓
/var/log/syslog ✓
/var/log/kern.log ✓
/var/log/apache2/access.log ✓
/var/log/apache2/error.log ✓
/var/log/mysql/error.log ✓
etc.
```

**Search All in One Index:**
```spl
index=system_logs
| stats count by sourcetype
```

**Results Show:**
```
sourcetype           count
auth                 5234
syslog               8234
apache               3456
mysql                234
```

#### Option 2: Monitor Specific Files Separately

Create separate monitors for different types:

**Monitor 1: Authentication Logs**
```
Path: /var/log/auth.log
Source Type: auth
Index: auth_logs
```

**Monitor 2: Web Server Logs**
```
Path: /var/log/apache2/
Source Type: apache
Index: web_logs
```

**Monitor 3: System Logs**
```
Path: /var/log/syslog
Source Type: syslog
Index: system_logs
```

**Search Each Index:**
```spl
# Failed SSH attempts
index=auth_logs "Failed password"

# 404 errors
index=web_logs status=404

# System errors
index=system_logs "ERROR"
```

---

## PART 5: PRACTICAL MONITORING SCENARIOS

### Scenario 1: Real-Time Security Monitoring

```
Goal: Detect failed login attempts immediately
```

**Setup Monitor:**
```
Path: /var/log/auth.log
Source Type: auth
Index: security
```

**Create Real-Time Search:**
```spl
index=security "Failed password"
| stats count by src_ip
| where count > 5
```

**Set Alert:**
```
Alert Name: Brute Force Detection
Condition: count > 5
Timeframe: Last 5 minutes
Action: Email security team
```

**Test Alert:**
```bash
# Generate failed logins
ssh root@localhost  # wrong password
ssh root@localhost  # wrong password
ssh root@localhost  # wrong password
ssh root@localhost  # wrong password
ssh root@localhost  # wrong password
ssh root@localhost  # wrong password (6 attempts)

# Alert fires! Email sent immediately
```

### Scenario 2: Monitor Web Server in Real-Time

```
Goal: See every web request immediately
```

**Setup Monitor:**
```
Path: /var/log/apache2/access.log
Source Type: apache
Index: web_logs
```

**Real-Time Dashboard:**
```spl
index=web_logs
| timechart count by status

# Shows HTTP status codes over time
```

**Generate Web Traffic:**
```bash
# Terminal 1: Start Apache
sudo service apache2 start

# Terminal 2: Generate requests
curl http://localhost/
curl http://localhost/admin  # 404
curl http://localhost/api

# Terminal 3: Search in Splunk
# Logs appear immediately in dashboard
```

### Scenario 3: System Activity Monitoring

```
Goal: Monitor all system changes
```

**Setup Monitor:**
```
Path: /var/log/syslog
Source Type: syslog
Index: system
```

**Monitor Specific Activity:**
```spl
# User additions
index=system "useradd"

# Package installations
index=system "apt" "installed"

# Sudo commands executed
index=system "sudo"

# SSH connections
index=system "sshd"
```

**Real-Time Alert for New Users:**
```spl
index=system "useradd"
```

**Test:**
```bash
# Create new user
sudo useradd testuser

# Check Splunk
# Search: index=system "useradd"
# Result appears instantly
```

---

## PART 6: MONITORING BEST PRACTICES

### 1. Use Specific Paths

```
✓ GOOD:
/var/log/auth.log
/var/log/apache2/access.log
/var/log/syslog

✗ BAD:
/var/log/  (too broad, includes everything)
/ (monitors entire system!)
```

### 2. Set Appropriate Source Types

```
/var/log/auth.log → auth or syslog
/var/log/apache2/access.log → apache
/var/log/mysql/error.log → mysql
/var/log/nginx/access.log → nginx
```

### 3. Organize by Index

```
Create custom indexes:
- auth_logs (authentication events)
- web_logs (web server logs)
- system_logs (system messages)
- app_logs (application logs)
```

### 4. Monitor Log Rotation

When logs rotate (auth.log → auth.log.1):

```
✓ GOOD (handles rotation):
Path: /var/log/auth.log*
Includes: auth.log, auth.log.1, auth.log.2...

✗ BAD (misses rotated logs):
Path: /var/log/auth.log
Only monitors original file
```

### 5. Verify Permissions

```bash
# Check if Splunk can read files
ls -la /var/log/auth.log

# Should show readable by all or group
-rw-r----- 1 root adm 234567 Feb 20 10:30 /var/log/auth.log

# If not readable:
sudo chmod 644 /var/log/auth.log
```

---

## PART 7: MONITOR VS UPLOAD COMPARISON

| Feature | Monitor | Upload |
|---------|---------|--------|
| **Real-time** | Yes ✓ | No (one-time) |
| **Continuous** | Yes ✓ | No |
| **Setup** | Once | Each time |
| **Live alerts** | Yes ✓ | No |
| **Production** | Yes ✓ | No |
| **Testing** | Possible | Better |
| **Performance** | Lighter | Heavier |
| **Best for** | Ongoing ops | Learning |

---

## PART 8: TROUBLESHOOTING

### Problem: Monitor Not Ingesting Logs

**Check 1: Monitor Status**
```
Settings > Data Inputs > Files & Directories
Should show: Active status
```

**Check 2: File Exists**
```bash
ls -la /var/log/auth.log
# Should show file exists
```

**Check 3: Permissions**
```bash
ls -la /var/log/auth.log
# Should be readable (r permission)
```

**Check 4: Log Has New Data**
```bash
tail -f /var/log/auth.log
# New entries appearing?
```

**Check 5: Search Results**
```spl
index=auth_logs | stats count
# Should show count > 0
```

### Problem: Monitoring Stops After System Restart

**Cause:** Monitor not set to auto-start

**Solution:**
```
Monitor is persistent and auto-starts
If not working:
1. Verify monitor still configured
2. Restart Splunk: sudo systemctl restart splunk
3. Check if file path still exists
```

### Problem: File Permissions Changed

**Cause:** Log file permissions too restrictive

**Solution:**
```bash
# Check current permissions
ls -la /var/log/auth.log

# Make readable
sudo chmod 644 /var/log/auth.log

# Verify
sudo usermod -aG adm splunk
# (add splunk user to log readers group)

# Restart Splunk
sudo systemctl restart splunk
```

### Problem: Too Much Data (Disk Full)

**Cause:** Monitoring generates too much data

**Solution:**
1. **Limit what you monitor:**
   - Monitor specific files, not entire /var/log/
   - Use source type filters

2. **Configure index retention:**
   - Settings > Indexes > [index]
   - Set max size and retention

3. **Create index with limits:**
   - Settings > Indexes > New Index
   - Max size: 10 GB (or your preference)
   - Retention: 30 days (or your preference)

---

## PART 9: MULTIPLE MONITOR SETUP

### Complete Production Monitoring Setup

**Monitor 1: Authentication**
```
Path: /var/log/auth.log
Source Type: auth
Index: auth_logs
Purpose: Track login attempts, sudo usage, su attempts
```

**Monitor 2: System Events**
```
Path: /var/log/syslog
Source Type: syslog
Index: system_logs
Purpose: System messages, service starts, errors
```

**Monitor 3: Web Server**
```
Path: /var/log/apache2/access.log
Source Type: apache
Index: web_logs
Purpose: HTTP requests, status codes, client IPs
```

**Monitor 4: Web Errors**
```
Path: /var/log/apache2/error.log
Source Type: apache_error
Index: web_logs
Purpose: Web server errors, crashes, issues
```

**Monitor 5: Security Port**
```
Protocol: TCP
Port: 9999
Source Type: syslog
Index: syslog_receive
Purpose: Receive syslog from remote machines
```

### Search All Monitors

```spl
# Count events by monitor
index=auth_logs OR index=system_logs OR index=web_logs
| stats count by index

# Results:
index            count
auth_logs        5234
system_logs      8234
web_logs         12456
```

---

## QUICK REFERENCE

```
MONITOR SETUP CHECKLIST:

☑ Open Splunk Settings
☑ Click Add Data > Monitor
☑ Choose: Files & Directories or Network Port
☑ Enter path or port number
☑ Select source type
☑ Select index
☑ Click Save
☑ Verify in search (index=[your_index])
☑ Generate test activity
☑ Watch logs appear in real-time
☑ Create alert (optional)
```

---

## NEXT STEPS

After mastering Monitor Method:
1. Learn **Forward Method** (remote log sending)
2. Set up multi-machine monitoring
3. Create automated alerts
4. Build real-time dashboards
5. Scale to production

---

**Monitor Method is Complete!** ✅

You now know how to:
- Set up file monitoring
- Monitor network ports
- Track logs in real-time
- Generate test activities
- Create real-time alerts
- Troubleshoot monitoring issues

**Next:** Learn the Forward Method for centralizing logs from multiple machines!
