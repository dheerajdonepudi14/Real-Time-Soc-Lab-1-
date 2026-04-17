 Splunk SOC Lab – End-to-End SIEM Implementation
 Project Overview

This project demonstrates a complete Splunk Enterprise SIEM lab setup, covering installation, architecture understanding, log ingestion, real-time monitoring, and alerting.

Built to simulate real-world Security Operations Center (SOC) workflows.

 Splunk Architecture (How It Works)

Splunk processes data using a 6-stage pipeline:

Collection → Parsing → Preprocessing → Indexing → Search → Alerting

🔹 Collection

Logs collected from:

/var/log/auth.log

/var/log/syslog

Web server logs

Custom application logs

🔹 Parsing

Splunk extracts fields like:

_time

host

source

sourcetype

user

src_ip

action

🔹 Indexing

Data stored in organized indexes:

main

auth_logs

web_logs

security

Each index uses:

Hot bucket (new data)

Warm bucket

Cold bucket

Frozen (retention-based deletion)

🔹 Search & Analysis

Example SPL queries:

index=auth_logs
index=auth_logs "Failed password"
index=auth_logs | stats count by src_ip
index=auth_logs | timechart count

🔹 Alerting

Alerts trigger when thresholds are met:

index=auth_logs "Failed password"
| stats count by src_ip
| where count > 10


Triggers:

Email notification

Webhook

Script execution

Incident creation

 Installation (Kali / Ubuntu)
🔹 Requirements

4–8 GB RAM

20+ GB disk

2+ CPU cores

Port 8000 accessible

🔹 Install Commands
sudo apt update
sudo apt install curl wget -y
sudo dpkg -i splunk-*.deb
sudo /opt/splunk/bin/splunk enable boot-start --accept-license -run-as-root
sudo /opt/splunk/bin/splunk start --accept-license -run-as-root


Access UI:

http://localhost:8000


Verify:

/opt/splunk/bin/splunk version
sudo systemctl status splunk

 Log Ingestion Methods
🔹 Method 1 – Upload (Beginner)

Upload log manually:

Settings → Add Data → Upload


Best for:

Learning

Testing queries

One-time forensic analysis

🔹 Method 2 – Monitor (Real-Time)

Monitor file continuously:

Settings → Add Data → Monitor
Path: /var/log/auth.log
Index: auth_logs


Logs appear automatically.

Best for:

Live SOC monitoring

Production use

🔹 Method 3 – Forwarder (Enterprise)

Send logs from remote machine:

sudo dpkg -i splunk-universal-forwarder-*.deb
sudo /opt/splunkforwarder/bin/splunk add forward-server <IP>:1818
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/auth.log


Best for:

Multi-server environments

Centralized monitoring

🔹 Method 4 – Attack Simulation

Generate security logs:

for i in {1..10}; do
  sshpass -p wrong ssh root@localhost exit 2>/dev/null
done


Used to:

Test brute-force detection

Validate alerts

Practice SOC response

 Monitoring & Alerting
🔹 Monitoring vs Searching

Searching → After incident

Monitoring → Real-time detection

 Example Alert – Brute Force Detection
index=auth_logs "Failed password"
| stats count by src_ip
| where count > 10


Alert Settings:

Trigger: Every 5 minutes

Condition: count > 10

Actions:

Email SOC team

Block IP

Create ticket

 Example Monitoring Dashboard Panels
Failed Logins
index=auth_logs "Failed password"
| timechart count by src_ip

Web Errors
index=web_logs status >= 400
| stats count by status

Alert Summary
index=_audit action=alert
| stats count by alert_name

 Incident Response Workflow
Alert → Acknowledge → Investigate → Respond → Recover → Document


Example timeline:

10:15 Alert fired
10:16 Analyst reviews
10:17 IP blocked
10:18 Attack stopped

 Monitoring KPIs

True Positive Rate

False Positive Rate

Response Time

Detection Time

Alert Volume per Day

Goal:

Detect < 1 minute

Respond < 5 minutes

 What This Project Demonstrates

Splunk Enterprise deployment
SIEM architecture understanding
Custom index creation
 Multi-method log ingestion
 SPL query development
Brute-force detection use case
Real-time alert configuration
SOC monitoring dashboard
Incident response workflow

 Tools Used

Splunk Enterprise

Kali Linux / Ubuntu

Linux system logs

SPL (Search Processing Language)

Splunk Universal Forwarder

 Resume-Ready Description

Built an end-to-end Splunk SIEM lab including installation, log ingestion (upload, monitor, forwarder), custom indexing, SPL-based detection queries, brute-force attack detection, real-time alerting, dashboard creation, and incident response workflow simulation.

Future Enhancements

Threat intelligence integration

SOAR automation

Multi-index correlation

Log retention policy tuning

Distributed Splunk deployment

