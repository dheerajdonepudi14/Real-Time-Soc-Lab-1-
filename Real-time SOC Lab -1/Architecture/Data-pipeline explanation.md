┌─────────────────────────────────────────────────────────────────┐
│                    SPLUNK DATA PIPELINE                          │
└─────────────────────────────────────────────────────────────────┘

[1] DATA COLLECTION
     ↓
   Logs from:
   - Servers (syslogs)
   - Applications (Apache, databases)
   - Security devices (firewalls)
   - Devices (Linux, Windows)
     ↓
[2] PARSING
     ↓
   Extract fields:
   - timestamp
   - host
   - source
   - sourcetype
   - Custom fields
     ↓
[3] PREPROCESSING
     ↓
   - Remove junk
   - Normalize data
   - Apply transformations
   - Validate quality
     ↓
[4] INDEXING
     ↓
   Store in Index:
   - main
   - logs
   - ub_logs
   - custom_index
     ↓
[5] SEARCH & ANALYSIS
     ↓
   SPL Queries:
   - index=main
   - | stats count by user
   - | where count > 10
     ↓
[6] VISUALIZATION
     ↓
   Charts, Graphs:
   - Tables
   - Line graphs
   - Pie charts
   - Heatmaps
     ↓
[7] REPORTING
     ↓
   Generate Reports:
   - Daily summaries
   - PDF exports
   - Email distribution
   - Automated schedules
     ↓
[8] MONITORING & ALERTING
     ↓
   Real-time Response:
   - Alert triggered
   - Action executed
   - Incident created
   - Response initiated