# Workflow Architecture Overview

Visual representation of the N8n Multi-Server Audit Workflow.

## Workflow Flowchart

```
┌─────────────────────────────────────────────────────────────────┐
│                        SCHEDULE TRIGGER                           │
│                    (Every 30 minutes by default)                 │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                            CONFIG NODE                          │
│  - Server credentials (host|user|pass|os_type|port|protocol)  │
│  - Alert thresholds (CPU, Memory, Disk)                        │
│  - Telegram Chat ID                                            │
│  - Time windows and other settings                              │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                         PARSE SERVERS                           │
│              Convert config string to server objects             │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                        SPLIT SERVERS                            │
│                    Split into individual items                  │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                    DETECT OS & CONNECT                          │
│          - Try SSH connection (Linux)                             │
│          - Try WinRM connection (Windows)                        │
│          - Detect Linux distribution                             │
│          - Set connection status                                 │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                        ROUTE BY OS                              │
│              Set routing key based on OS detection              │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
                    ┌────────┴────────┐
                    │  OS ROUTER      │
                    │  (Switch Node)  │
                    └───────┬────────┘
            ┌───────────────┼───────────────┐
            │               │               │
            ▼               ▼               ▼
    ┌───────────┐   ┌───────────┐   ┌───────────┐
    │   LINUX   │   │  WINDOWS  │   │   ERROR   │
    │  (SSH)    │   │  (WinRM)  │   │  Handler  │
    └─────┬─────┘   └─────┬─────┘   └─────┬─────┘
          │               │               │
          └───────────────┴───────────────┘
                          │
                          ▼
          ┌───────────────────────────────┐
          │      ANALYZE RESULTS          │
          │  - Evaluate thresholds         │
          │  - Check for errors           │
          │  - Build report               │
          │  - Set critical flag          │
          └───────────────┬───────────────┘
                          │
                          ▼
          ┌───────────────────────────────┐
          │      STATUS ROUTER           │
          │   (Switch Node)              │
          └───────┬───────────┬─────────┘
                  │           │
         CRITICAL │           │ OK
                  │           │
                  ▼           ▼
    ┌─────────────────────┐  ┌─────────────────────┐
    │ BATCH CRITICAL      │  │  LOG OK SERVERS     │
    │    SERVERS          │  │                     │
    │ - Consolidate       │  │ - Log healthy       │
    │   critical servers  │  │   audits           │
    │ - Prepare for AI    │  │ - No alert sent    │
    └─────────┬───────────┘  └─────────────────────┘
              │
              ▼
    ┌─────────────────────┐
    │   AI AGENT         │
    │   ANALYSIS         │
    │ - Analyze issues   │
    │ - Identify causes  │
    │ - Provide fixes    │
    │ - Suggest          │
    │   prevention       │
    └─────────┬─────────┘
              │
              ▼
    ┌─────────────────────┐
    │   PREPARE          │
    │   TELEGRAM         │
    │ - Format message   │
    │ - Add AI analysis │
    │ - Split for       │
    │   each server     │
    └─────────┬─────────┘
              │
              ▼
    ┌─────────────────────┐
    │ SEND TELEGRAM       │
    │ ALERT              │
    │ - Send formatted   │
    │   alert            │
    │ - Include AI       │
    │   recommendations  │
    └─────────────────────┘
```

## Node Sequence

1. **Schedule Trigger** - Initiates workflow on schedule
2. **Config** - Provides configuration data
3. **Parse Servers** - Parses server configuration string
4. **Split Servers** - Creates individual items for each server
5. **Detect OS & Connect** - Detects OS and tests connection
6. **Route by OS** - Sets routing information
7. **OS Router** - Routes to appropriate audit path
8. **Linux Audit** OR **Windows Audit** OR **Handle Error** - Performs OS-specific audit
9. **Analyze Results** - Evaluates audit data and detects issues
10. **Status Router** - Routes based on critical/OK status
11. **Batch Critical Servers** (critical only) - Consolidates critical servers
12. **AI Agent Analysis** (critical only) - Analyzes issues with AI
13. **Prepare Telegram** (critical only) - Formats alert messages
14. **Send Telegram Alert** (critical only) - Sends notifications
15. **Log OK Servers** (OK only) - Logs successful audits

## Data Flow

```
Config String
    ↓
Server Objects Array
    ↓
Individual Server Items (one per server)
    ↓
Server with Detected OS + Connection Status
    ↓
Server with Audit Data (metrics, logs, status)
    ↓
Server with Analysis (critical issues detected)
    ↓
[Split] Critical Path → AI Analysis → Telegram Alert
         OK Path → Logging
```

## Key Data Structures

### Server Configuration (Config Node)
```javascript
{
  servers: "192.168.1.10|user|pass|auto|22|ssh\n192.168.1.20|admin|pass|auto|5986|winrm",
  time_window_min: 15,
  mem_critical_pct: 85,
  disk_critical_pct: 85,
  cpu_critical_pct: 90,
  telegram_chat_id: "123456789",
  max_error_lines: 50
}
```

### Parsed Server Object
```javascript
{
  host: "192.168.1.10",
  user: "admin",
  pass: "password123",
  os_type: "auto",
  port: "22",
  protocol: "ssh"
}
```

### Server with OS Detection
```javascript
{
  host: "192.168.1.10",
  user: "admin",
  detected_os: "linux",
  distro: "ubuntu",
  connected: true,
  port: "22",
  protocol: "ssh",
  error: null
}
```

### Audit Data (Linux)
```javascript
{
  hostname: "192.168.1.10",
  os: "linux",
  distro: "ubuntu",
  username: "admin",
  audit_timestamp: "2024-01-15T10:30:00.000Z",
  ssh_connected: true,
  cpu_load: "0.50, 0.60, 0.55",
  mem_total: 8192,
  mem_used: 6144,
  mem_free: 2048,
  mem_pct: 75.0,
  disk_used_pct: 65,
  disk_used: "125G",
  disk_total: "200G",
  disk_name: "sda",
  disk_util_pct: 45.2,
  uptime: "up 30 days",
  journal_errors: "...",
  service_status: "ssh:active, nginx:active",
  process_count: 156,
  active_connections: 42,
  logged_users: "admin, devuser"
}
```

### Audit Data (Windows)
```javascript
{
  hostname: "192.168.1.20",
  os: "windows",
  distro: "windows",
  username: "administrator",
  audit_timestamp: "2024-01-15T10:30:00.000Z",
  winrm_connected: true,
  cpu_pct: 65.5,
  mem_pct: 72.3,
  disk_used_pct: 58.0,
  process_count: 245,
  uptime: "14 days, 6 hours",
  stopped_services: "...",
  event_errors: "..."
}
```

### Analysis Result
```javascript
{
  hostname: "192.168.1.10",
  os: "linux",
  distro: "ubuntu",
  username: "admin",
  report: "...formatted text report...",
  audit_data: {...},
  conn_failed: false,
  cpu_critical: false,
  mem_critical: true,
  disk_critical: false,
  has_journal_errors: false,
  has_event_errors: false,
  has_service_issues: false,
  has_critical: true,
  cpu_pct: 50,
  mem_pct: 85,
  disk_used_pct: 65,
  disk_util_pct: 45
}
```

## Alert Format

```
🚨 CRITICAL ALERT

🐧 Server: 192.168.1.10
OS: LINUX (ubuntu)
User: admin

Issues Detected:
🔴 Memory: 85.0%

---

AI Analysis & Recommendations:

Root Cause Analysis:
• High memory usage detected
• Possible memory leak in application

Immediate Actions:
• Check top processes: top -o %MEM
• Analyze memory: free -h
• Check services: systemctl status nginx

Prevention:
• Implement memory monitoring alerts
• Consider increasing available memory

---

🕐 2024-01-15T10:30:00.000Z
Multi-Server Audit Workflow
```

## Execution Time Estimates

| Step | Time per Server | Notes |
|------|-----------------|-------|
| OS Detection | 5-10 seconds | SSH connection + OS check |
| Linux Audit | 10-20 seconds | Multiple SSH commands |
| Windows Audit | 15-25 seconds | WinRM + PowerShell |
| Analysis | < 1 second | Threshold evaluation |
| AI Analysis | 5-15 seconds | Depends on AI service |
| Total (per server) | 30-50 seconds | Excludes AI |

**Example execution times:**
- 5 servers, all OK: ~2-3 minutes
- 5 servers, 2 critical: ~3-4 minutes (with AI analysis)
- 20 servers, all OK: ~8-12 minutes
- 20 servers, 5 critical: ~12-20 minutes (with AI analysis)

## Parallel Processing

Currently processes servers **sequentially** (one at a time).

To enable parallel processing:
1. Use **Loop Over Items** node instead of split
2. Configure concurrency (e.g., 5 concurrent servers)
3. Add rate limiting to avoid overwhelming servers

## Scaling Considerations

### Small Scale (< 10 servers)
- Single workflow
- Sequential processing
- 30-minute schedule
- No rate limiting needed

### Medium Scale (10-50 servers)
- Single workflow
- Consider batch processing
- 60-minute schedule
- Add delays between servers

### Large Scale (50+ servers)
- Multiple workflows (by environment/region)
- Parallel processing with concurrency limit
- 2-4 hour schedule
- Implement queue system
- Consider distributed N8n instances

## Customization Points

1. **Schedule Trigger** - Change execution frequency
2. **Config Node** - Add/modify thresholds and settings
3. **OS Detection** - Add support for new OS types
4. **Audit Nodes** - Add custom metrics and checks
5. **Analysis Node** - Modify threshold logic
6. **AI Agent** - Customize analysis prompts
7. **Telegram Node** - Adjust alert format and content
8. **Add New Alert Channels** - Email, Slack, Discord, etc.

## Error Handling

| Error Type | Handling | User Notification |
|------------|----------|-------------------|
| Connection Failed | Retry once, then fail | ✅ Telegram alert |
| SSH Timeout | Increase timeout, retry | ✅ Telegram alert |
| Permission Denied | Log error, mark as critical | ✅ Telegram alert |
| Command Failed | Continue with available data | ⚠️ Partial alert |
| AI Error | Send alert without AI analysis | ⚠️ Alert with error note |

---

For detailed configuration instructions, see [README.md](README.md)
For quick setup, see [QUICKSTART.md](QUICKSTART.md)