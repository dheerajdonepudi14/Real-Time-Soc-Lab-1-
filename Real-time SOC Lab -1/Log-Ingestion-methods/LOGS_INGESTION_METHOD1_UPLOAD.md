# Splunk Log Ingestion - METHOD 1: UPLOAD METHOD

## 📥 Overview

The **Upload Method** is the simplest way to ingest logs into Splunk. You upload log files directly through the Splunk web interface. Perfect for:
- Testing and learning
- One-time log imports
- Forensic analysis
- Lab environments

---

## 📋 PREREQUISITES

### Check System Resources
```bash
# Check available space
df -h /

# Check Splunk running
ps aux | grep splunk

# Check Splunk accessible
curl http://localhost:8000
```

### Access Splunk
```
Open browser: http://localhost:8000
Login: admin / [your password]
```

---

## PART 1: LOCATE & COPY LOG FILES FROM LINUX

### Step 1.1: Find Available Log Files

#### On Kali Linux or Ubuntu
```bash
# Navigate to log directory
cd /var/log

# List all logs
ls -la

# List by size
ls -lh | sort -k5 -hr
```

#### Common Log Locations

| Log File | Location | Contains |
|----------|----------|----------|
| **auth.log** | `/var/log/auth.log` | SSH, su, sudo attempts |
| **syslog** | `/var/log/syslog` | System messages |
| **kern.log** | `/var/log/kern.log` | Kernel messages |
| **apache2 access** | `/var/log/apache2/access.log` | Web requests |
| **apache2 error** | `/var/log/apache2/error.log` | Web errors |
| **nginx access** | `/var/log/nginx/access.log` | Nginx requests |
| **nginx error** | `/var/log/nginx/error.log` | Nginx errors |
| **mysql error** | `/var/log/mysql/error.log` | MySQL errors |

### Step 1.2: View Log Content

```bash
# View auth.log
cat /var/log/auth.log | head -20

# Expected output:
# Feb 20 10:15:23 ubuntu sshd[1234]: Failed password for root from 192.168.1.100
# Feb 20 10:15:45 ubuntu sshd[1235]: Accepted publickey for user from 192.168.1.100
# Feb 20 10:16:02 ubuntu sudo: user : TTY=pts/0 ; PWD=/home/user ; USER=root ; COMMAND=/bin/ls

# View syslog
cat /var/log/syslog | head -20

# View apache access log
cat /var/log/apache2/access.log | head -10

# Expected output:
# 192.168.1.100 - - [20/Feb/2026:10:15:23 +0000] "GET /admin HTTP/1.1" 404 256 "-" "Mozilla/5.0"
```

### Step 1.3: Check Log File Sizes

```bash
# Check auth.log size
ls -lh /var/log/auth.log

# Check multiple logs
ls -lh /var/log/*.log

# Check if log is empty
wc -l /var/log/auth.log

# Example output: 5234 /var/log/auth.log (5234 lines)
```

### Step 1.4: Copy Log Files to Desktop (For Easy Access)

#### Option A: Copy to Home Desktop

```bash
# Create logs directory on desktop
mkdir -p ~/Desktop/splunk-logs

# Copy auth log
cp /var/log/auth.log ~/Desktop/splunk-logs/auth.log

# Copy syslog
cp /var/log/syslog ~/Desktop/splunk-logs/syslog.log

# Copy apache logs (if exists)
cp /var/log/apache2/access.log ~/Desktop/splunk-logs/apache-access.log

# Verify copies
ls -la ~/Desktop/splunk-logs/
```

#### Option B: Copy to Temporary Location

```bash
# Copy to temp directory
cp /var/log/auth.log /tmp/auth.log
cp /var/log/syslog /tmp/syslog.log

# Verify
ls -lh /tmp/*.log
```

### Step 1.5: Fix File Permissions (If Needed)

```bash
# Check current permissions
ls -la ~/Desktop/splunk-logs/auth.log

# Make readable by all
chmod 644 ~/Desktop/splunk-logs/auth.log

# Make readable and writable
chmod 777 ~/Desktop/splunk-logs/auth.log

# Verify permissions changed
ls -la ~/Desktop/splunk-logs/auth.log
```

---

## PART 2: UPLOAD LOGS TO SPLUNK

### Step 2.1: Access Splunk Upload Interface

1. **Open browser:** `http://localhost:8000`
2. **Login:** admin / [your password]
3. **Go to Settings:**
   - Click gear icon (top right)
   - Select "Settings"
4. **Click "Add Data"**
5. **Select "Upload"**

### Step 2.2: Select Log File to Upload

**UI Steps:**
1. Click **"Select File"** button
2. Navigate to `~/Desktop/splunk-logs/`
3. Select `auth.log`
4. Click **"Open"**

**Expected Result:**
```
File selected: auth.log
File size: ~500 KB
Preview showing log entries
```

### Step 2.3: Preview File Content

**Screen Shows:**
```
Preview of auth.log

Feb 20 10:15:23 ubuntu sshd[1234]: Failed password for root from 192.168.1.100
Feb 20 10:15:45 ubuntu sshd[1235]: Accepted publickey for user from 192.168.1.100
Feb 20 10:16:02 ubuntu sudo: user : TTY=pts/0 ; PWD=/home/user ; USER=root ; COMMAND=/bin/ls
Feb 20 10:16:23 ubuntu sshd[1236]: Connection closed by 192.168.1.100 port 22222
...
(more entries)
```

**Click Next** if preview looks good.

### Step 2.4: Set Source Name

**Input Field:**
- Label: "Source Name"
- Current: `auth.log` (auto-filled)
- Change to: `auth.log` (or custom name)

**Examples:**
```
auth.log
authentication_logs
server1_auth
ubuntu_auth_logs
```

**Why:** Identifies where the logs came from

**Click Next** to proceed.

### Step 2.5: Select Source Type

**What is Source Type?**
- Tells Splunk how to parse the log
- Automatic format recognition
- Determines field extraction

**UI Shows Dropdown:**
```
Source type selection:
├─ syslog (System logs)
├─ auth (Authentication)
├─ apache (Web server)
├─ access_combined (Web access)
├─ mysql (MySQL logs)
├─ linux_secure (Linux auth)
└─ Other options...
```

**For auth.log, Select:**
- **syslog** (if auto-detected correctly)
- Or manually select **auth**

**If Wrong Type Selected:**
1. Click dropdown
2. Search for correct type
3. Type: `auth`
4. Select from list

**Click Next** to proceed.

### Step 2.6: Set Input Type

**Shows:**
```
Input Type: upload
Input Path: auth.log
Host: [auto-detected]
Sourcetype: auth
Index: main (default)
```

**Review:**
- ✓ Input type: upload
- ✓ File: auth.log
- ✓ Source type: auth or syslog
- ✓ Host: [your machine name]

**Click Next** to select index.

### Step 2.7: Select or Create Index

**Default Option:**
```
Index: main
(Default index for all data)
```

**Or Create Custom Index:**
1. Click **"Create new index"**
2. Enter name: `auth_logs`
3. Click **"Save"**

**Index Details Form:**
```
Index Name: auth_logs
Description: Linux authentication logs
Max size (MB): 500
Retention (days): 365 (or 0 for forever)
```

**Select Your Index:**
```
☑ auth_logs (if created)
OR
☑ main (default)
```

**Click Next** to review.

### Step 2.8: Final Review

**Confirmation Screen Shows:**
```
Input Type: upload
File Name: auth.log
Source Name: auth.log
Source Type: auth
Host: ubuntu (or your hostname)
Index: auth_logs
```

**Verify All Settings Correct:**
- ✓ File name
- ✓ Source type
- ✓ Host name
- ✓ Index name

**Click "Submit"** to complete upload.

---

## PART 3: VERIFY UPLOAD & SEARCH LOGS

### Step 3.1: See Upload Confirmation

**Success Message:**
```
✅ Upload successful!
298 events uploaded successfully

Event count: 298
Index: auth_logs
Host: ubuntu
Sourcetype: auth
```

### Step 3.2: Start Searching

**Option 1: Immediate Search**
- Click **"Start Searching"** button
- Automatically shows search results
- Filter: `index=auth_logs`

**Option 2: Manual Search**
1. Go to **"Search and Reporting"** app
2. Enter search: `index=auth_logs`
3. Click **"Search"**

### Step 3.3: View Search Results

**Results Display:**
```
Results: 298 events
Time Range: Last 24 hours

Event 1:
Feb 20 10:15:23 ubuntu sshd[1234]: Failed password for root from 192.168.1.100

Event 2:
Feb 20 10:15:45 ubuntu sshd[1235]: Accepted publickey for user from 192.168.1.100

Event 3:
Feb 20 10:16:02 ubuntu sudo: user : TTY=pts/0 ; PWD=/home/user ; USER=root ; COMMAND=/bin/ls
...
(more events)
```

### Step 3.4: Analyze Extracted Fields

**Click on Event** to expand:
```
_time: Feb 20 10:15:23
_raw: Full log line
host: ubuntu
source: auth.log
sourcetype: auth
user: root (extracted)
src_ip: 192.168.1.100 (extracted)
action: Failed password (extracted)
```

### Step 3.5: Run Sample Queries

**Query 1: Count Events**
```spl
index=auth_logs | stats count
```
**Result:** 298 events

**Query 2: Failed Logins**
```spl
index=auth_logs "Failed password" | stats count by src_ip
```
**Result:**
```
src_ip            count
192.168.1.100     145
192.168.1.101     89
10.0.0.50         24
```

**Query 3: Successful Logins**
```spl
index=auth_logs "Accepted publickey" | stats count by user
```
**Result:**
```
user         count
root         52
admin        38
user1        23
```

---

## PART 4: UPLOAD ADDITIONAL LOG FILES

### Repeat Process for More Logs

```bash
# Copy more log files
cp /var/log/syslog ~/Desktop/splunk-logs/syslog.log

# If Apache installed
cp /var/log/apache2/access.log ~/Desktop/splunk-logs/apache-access.log
cp /var/log/apache2/error.log ~/Desktop/splunk-logs/apache-error.log

# If MySQL installed
cp /var/log/mysql/error.log ~/Desktop/splunk-logs/mysql-error.log
```

### Upload Each File

For each log file:
1. Settings > Add Data > Upload
2. Select file
3. Set source name
4. Select source type
5. Select/create index
6. Submit
7. Verify

### Create Search Index View

**Search All Uploaded Logs:**
```spl
index=auth_logs OR index=web_logs OR index=system_logs
| stats count by sourcetype
```

---

## PART 5: PRACTICAL EXAMPLES

### Example 1: Analyze Failed SSH Attempts

```bash
# On your Linux machine, try failing to login
ssh root@localhost
# (wrong password)
ssh admin@localhost
# (wrong password)
ssh user@localhost
# (wrong password)
```

```bash
# These create new log entries in auth.log
tail -5 /var/log/auth.log

# Output:
# Feb 20 10:20:15 ubuntu sshd[2000]: Failed password for root from 127.0.0.1
# Feb 20 10:20:23 ubuntu sshd[2001]: Failed password for admin from 127.0.0.1
# Feb 20 10:20:31 ubuntu sshd[2002]: Failed password for user from 127.0.0.1
```

```bash
# Copy updated log
cp /var/log/auth.log ~/Desktop/splunk-logs/auth.log
```

```
Upload to Splunk again
Search: index=auth_logs "Failed password"
Results show 3 new failed attempts
```

### Example 2: Analyze Web Server Logs

```bash
# If Apache running
sudo service apache2 start

# Generate some web traffic
curl http://localhost/
curl http://localhost/admin (404)
curl http://localhost/api/users

# Check access log
tail -5 /var/log/apache2/access.log

# Output:
# 127.0.0.1 - - [20/Feb/2026:10:22:10 +0000] "GET / HTTP/1.1" 200 11370
# 127.0.0.1 - - [20/Feb/2026:10:22:15 +0000] "GET /admin HTTP/1.1" 404 256
# 127.0.0.1 - - [20/Feb/2026:10:22:20 +0000] "GET /api/users HTTP/1.1" 200 1234
```

```bash
# Upload to Splunk
cp /var/log/apache2/access.log ~/Desktop/splunk-logs/apache-access.log
```

```
Upload to Splunk
Search: index=web_logs status=404
Results show 404 errors
```

### Example 3: System Activity Analysis

```bash
# Generate system activity
sudo apt update
sudo apt install htop
sudo useradd testuser
sudo usermod -aG sudo testuser
```

```bash
# Check syslog
tail -20 /var/log/syslog

# Shows all system activities
```

```bash
# Upload syslog
cp /var/log/syslog ~/Desktop/splunk-logs/syslog.log
```

```
Upload to Splunk
Search: index=system_logs "useradd" OR "usermod"
Results show user management activities
```

---

## PART 6: TROUBLESHOOTING

### Problem: Upload Fails with "File Not Found"

**Cause:** File doesn't exist or wrong path

**Solution:**
```bash
# Verify file exists
ls -la ~/Desktop/splunk-logs/auth.log

# If doesn't exist, copy again
cp /var/log/auth.log ~/Desktop/splunk-logs/auth.log
```

### Problem: File Shows Blank Preview

**Cause:** File is empty

**Solution:**
```bash
# Check if file has content
wc -l ~/Desktop/splunk-logs/auth.log

# If 0 lines, generate some activity
# Login attempts, run commands, etc.
```

### Problem: Wrong Source Type Applied

**Cause:** Auto-detection incorrect

**Solution:**
1. Go to: Settings > Field Extractions
2. Create custom extraction
3. Or re-upload with correct type selected manually

### Problem: Can't See Logs in Search

**Cause:** Wrong index or query

**Solution:**
```spl
# Check what indexes have data
| eventcount summarize=false index=* | dedup index

# Search specific index
index=auth_logs

# Or search all
index=*
```

### Problem: Search Returns Too Many Results

**Cause:** Time range too broad

**Solution:**
1. Change time range (top right dropdown)
2. Select: "Last 24 hours" or "Last hour"
3. Or specify in query: `earliest=-24h`

---

## SUMMARY TABLE

| Step | Action | Command/UI |
|------|--------|-----------|
| 1 | Find logs | `ls /var/log/` |
| 2 | Copy logs | `cp /var/log/auth.log ~/Desktop/` |
| 3 | Fix permissions | `chmod 644 auth.log` |
| 4 | Open Splunk | `http://localhost:8000` |
| 5 | Go to upload | Settings > Add Data > Upload |
| 6 | Select file | Click "Select File" |
| 7 | Verify preview | See log entries |
| 8 | Set source name | auth.log |
| 9 | Select source type | auth or syslog |
| 10 | Create index | auth_logs |
| 11 | Submit | Click Submit |
| 12 | Verify | View uploaded events |
| 13 | Search | index=auth_logs |

---

## QUICK REFERENCE COMMANDS

```bash
# All-in-one copy and upload workflow
cd /var/log

# Copy auth logs
cp auth.log ~/Desktop/splunk-logs/auth.log
cp syslog ~/Desktop/splunk-logs/syslog.log

# Make readable
chmod 644 ~/Desktop/splunk-logs/*.log

# Verify
ls -lh ~/Desktop/splunk-logs/

# Then open Splunk and upload
# Browser: http://localhost:8000
# Settings > Add Data > Upload
```

---

## BEST PRACTICES

1. **Always verify log content before upload**
   ```bash
   head -10 auth.log
   ```

2. **Use meaningful names**
   ```
   Good: auth.log, apache-access.log, system-syslog.log
   Bad: log1.txt, data.log, test.log
   ```

3. **Create custom indexes**
   ```
   Organize by type:
   - auth_logs (authentication)
   - web_logs (web server)
   - system_logs (system)
   - app_logs (applications)
   ```

4. **Always verify uploads**
   ```spl
   Search immediately after upload
   index=auth_logs | stats count
   ```

5. **Document your sources**
   ```
   Keep notes of:
   - What log file uploaded
   - When uploaded
   - Source type used
   - Index used
   ```

---

## NEXT STEPS

After mastering the Upload Method:
1. Learn **Monitor Method** (continuous monitoring)
2. Learn **Forward Method** (multi-machine setup)
3. Create automated ingestion workflows
4. Scale to production environments

---

**Upload Method is Complete!** ✅

You now know how to:
- Find log files on Linux
- Copy logs to accessible location
- Upload to Splunk via web UI
- Verify uploaded logs
- Search and analyze logs

**Next:** Learn the Monitor Method for real-time log ingestion!
