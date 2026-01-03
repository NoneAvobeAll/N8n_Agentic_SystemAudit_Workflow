# Workflow Customization Examples

This document provides examples and templates for customizing the Multi-Server Audit Workflow for different use cases.

## Table of Contents

- [Adding Custom Audit Checks](#adding-custom-audit-checks)
- [Custom Alert Channels](#custom-alert-channels)
- [Scheduling Variations](#scheduling-variations)
- [Server Group Configuration](#server-group-configuration)
- [Advanced AI Prompting](#advanced-ai-prompting)
- [Data Export & Reporting](#data-export--reporting)
- [Multi-Environment Setup](#multi-environment-setup)

---

## Adding Custom Audit Checks

### Linux: Check Docker Container Status

Edit the **Linux Audit** node and add this code after the existing metrics:

```javascript
// Docker Container Status (safe, read-only)
try {
  const dockerContainers = execSync(
    `${sshBase} "docker ps -a --format '{{.Names}}|{{.Status}}|{{.Ports}}'"`,
    { encoding: 'utf8', timeout: 15000 }
  );
  auditData.docker_containers = dockerContainers.trim();
  
  // Check for stopped containers
  const stoppedContainers = dockerContainers.match(/Exited/gi);
  auditData.docker_stopped_count = stoppedContainers ? stoppedContainers.length : 0;
} catch (e) {
  auditData.docker_error = 'Docker not installed or inaccessible';
}
```

Then update the **Analyze Results** node to check for Docker issues:

```javascript
const dockerCritical = d.docker_stopped_count > 0;
const hasCritical = hasCritical || dockerCritical;
```

### Linux: Check Disk I/O Wait Time

```javascript
// Disk I/O Wait Time (percentage)
try {
  const iowait = execSync(
    `${sshBase} "iostat -x 1 2 | awk 'NR==4{print \\$4}'"`,
    { encoding: 'utf8', timeout: 15000 }
  );
  auditData.iowait_pct = parseFloat(iowait.trim()) || 0;
} catch (e) {
  auditData.iowait_pct = 0;
}
```

### Windows: Check Windows Updates

Edit the **Windows Audit** node:

```javascript
// Windows Update Status
const updatesScript = `
  $updates = Get-WmiObject -Class Win32_QuickFixEngineering -ErrorAction SilentlyContinue
  $latest = $updates | Sort-Object InstalledOn -Descending | Select-Object -First 1
  @{
    HotFixID = $latest.HotFixID
    InstalledOn = $latest.InstalledOn
    Description = $latest.Description
  } | ConvertTo-Json
`;
const updatesResult = executePowerShell(updatesScript);
auditData.latest_update = updatesResult || 'Unknown';
```

### Windows: Check Active Directory Replication

```javascript
// AD Replication Health (for Domain Controllers)
const adScript = `
  Import-Module ActiveDirectory -ErrorAction SilentlyContinue
  repadmin /replsummary * /errorsonly | ConvertTo-Json
`;
const adResult = executePowerShell(adScript);
auditData.ad_replication = adResult || 'N/A';
```

---

## Custom Alert Channels

### Add Email Alerts

1. Add an **Email** node after **Prepare Telegram**
2. Configure SMTP credentials
3. Add this code to route:

```javascript
// In Status Router, add a new output for EMAIL
// Then add an Email node with this code:

const emailSubject = `[CRITICAL] Server Alert: ${$json.hostname}`;
const emailBody = `
Server Audit Critical Alert

Hostname: ${$json.hostname}
OS: ${$json.os.toUpperCase()}
User: ${$json.username}

Issues:
${$json.report}

AI Analysis:
${$json.ai_analysis}

Timestamp: ${new Date().toISOString()}
`;

return [{
  json: {
    subject: emailSubject,
    text: emailBody,
    to: 'your-email@example.com'
  }
}];
```

### Add Slack Alerts

1. Add a **Slack** node
2. Configure Slack webhook
3. Format message:

```javascript
const slackMessage = {
  channel: '#server-alerts',
  username: 'N8n Audit Bot',
  icon_emoji: ':rotating_light:',
  text: `🚨 *Critical Server Alert*`,
  attachments: [
    {
      color: 'danger',
      fields: [
        { title: 'Hostname', value: $json.hostname, short: true },
        { title: 'OS', value: $json.os.toUpperCase(), short: true },
        { title: 'Issues', value: $json.report.substring(0, 500), short: false }
      ]
    }
  ]
};

return [{ json: slackMessage }];
```

### Add Discord Alerts

```javascript
const discordPayload = {
  username: 'N8n Audit Bot',
  avatar_url: 'https://example.com/bot-avatar.png',
  embeds: [
    {
      title: '🚨 Critical Server Alert',
      color: 16711680, // Red
      fields: [
        { name: 'Hostname', value: $json.hostname, inline: true },
        { name: 'OS', value: $json.os.toUpperCase(), inline: true },
        { name: 'Issues', value: $json.report.substring(0, 1000) }
      ],
      timestamp: new Date().toISOString()
    }
  ]
};

return [{ json: discordPayload }];
```

---

## Scheduling Variations

### Run Every Hour

Edit the **Schedule Trigger** node:

```json
{
  "rule": {
    "interval": [
      {
        "field": "hours"
      }
    ]
  }
}
```

### Run at Specific Times (e.g., every 6 hours)

```json
{
  "rule": {
    "interval": [
      {
        "field": "hours",
        "hoursInterval": 6
      }
    ]
  }
}
```

### Run Daily at 2 AM UTC

```json
{
  "rule": {
    "interval": [
      {
        "field": "cronExpression",
        "expression": "0 2 * * *"
      }
    ]
  }
}
```

### Run Weekdays at 9 AM

```json
{
  "rule": {
    "interval": [
      {
        "field": "cronExpression",
        "expression": "0 9 * * 1-5"
      }
    ]
  }
}
```

### Run Every 15 Minutes During Business Hours

```json
{
  "rule": {
    "interval": [
      {
        "field": "cronExpression",
        "expression": "*/15 9-17 * * 1-5"
      }
    ]
  }
}
```

---

## Server Group Configuration

### Organize Servers by Environment

Create multiple Config nodes or use a Set node to filter:

```javascript
// In Parse Servers node, add environment tag
const servers = $json.Servers.split('\n').filter(s => s.trim()).map(line => {
  const parts = line.split('|');
  return {
    host: parts[0]?.trim(),
    user: parts[1]?.trim(),
    pass: parts[2]?.trim(),
    os_type: parts[3]?.trim() || 'auto',
    port: parts[4]?.trim() || 'auto',
    protocol: parts[5]?.trim() || 'auto',
    environment: parts[6]?.trim() || 'production' // New field
  };
});

return servers.map(server => ({ json: server }));
```

Server configuration format:
```
192.168.1.10|admin|pass|auto|auto|auto|production
192.168.1.20|admin|pass|auto|auto|auto|staging
192.168.1.30|admin|pass|auto|auto|auto|development
```

### Filter by Environment

Add a Switch node after parsing:

```javascript
// Route by environment
return {
  json: {
    ...$input.item.json,
    env_route: $json.environment
  }
};
```

Switch node rules:
- **PRODUCTION**: `{{ $json.environment === 'production' }}`
- **STAGING**: `{{ $json.environment === 'staging' }}`
- **DEVELOPMENT**: `{{ $json.environment === 'development' }}`

Use different schedules for each environment:
- Production: Every 30 minutes
- Staging: Every 2 hours
- Development: Daily

---

## Advanced AI Prompting

### Customize for Security Team

Edit the **AI Agent Analysis** node prompt:

```
You are a security incident response expert. Analyze the server audit and provide:

1. **Threat Assessment** (2 bullets)
   - Is there evidence of a security incident?
   - What is the risk level (Low/Medium/High/Critical)?

2. **Immediate Investigation** (specific commands)
   - Commands to gather forensic evidence
   - Commands to check for unauthorized access
   - All commands must be safe and read-only

3. **Containment Actions** (if threat detected)
   - Steps to isolate affected systems
   - Steps to preserve evidence

4. **Post-Incident Recovery** (1-2 bullets)

Format in Telegram Markdown. Include specific log paths to check. Max 400 words.
```

### Customize for DevOps Team

```
You are a senior DevOps engineer. Analyze the server audit and provide:

1. **Capacity Analysis** (2 bullets)
   - Are resources sufficient for current load?
   - Any bottlenecks identified?

2. **Optimization Recommendations** (bash commands)
   - Commands to free up resources
   - Commands to optimize performance
   - Safe commands only (read-only or safe restarts)

3. **Scaling Recommendations** (1-2 bullets)
   - When to scale up or out
   - What resources need scaling

Format in Telegram Markdown. Max 400 words.
```

### Custom Analysis for Database Servers

```
You are a database performance expert. Analyze the server audit and provide:

1. **Database Performance** (2 bullets)
   - Are database performance indicators normal?
   - Any signs of query issues?

2. **Immediate Actions** (SQL queries and commands)
   - Queries to identify slow queries
   - Commands to check connection pool
   - Commands to analyze disk I/O for DB

3. **Prevention** (1-2 bullets)
   - Index optimization tips
   - Query optimization suggestions

Format in Telegram Markdown. Max 400 words.
```

---

## Data Export & Reporting

### Export Audit Results to Google Sheets

1. Add **Google Sheets** node
2. Create a new sheet or append to existing
3. Configure credentials

```javascript
const sheetData = {
  values: [
    ['Timestamp', 'Hostname', 'OS', 'CPU %', 'Mem %', 'Disk %', 'Status'],
    [
      new Date().toISOString(),
      $json.hostname,
      $json.os.toUpperCase(),
      $json.cpu_pct || 0,
      $json.mem_pct || 0,
      $json.disk_used_pct || 0,
      $json.has_critical ? 'CRITICAL' : 'OK'
    ]
  ]
};

return [{ json: sheetData }];
```

### Export to PostgreSQL/MySQL

1. Add **Postgres** or **MySQL** node
2. Configure database connection
3. Insert audit results

```javascript
const query = `
  INSERT INTO server_audit_log 
    (timestamp, hostname, os, cpu_pct, mem_pct, disk_pct, status, audit_data)
  VALUES 
    ($1, $2, $3, $4, $5, $6, $7, $8)
`;

return [{
  json: {
    query: query,
    values: [
      new Date().toISOString(),
      $json.hostname,
      $json.os,
      $json.cpu_pct,
      $json.mem_pct,
      $json.disk_used_pct,
      $json.has_critical ? 'CRITICAL' : 'OK',
      JSON.stringify($json)
    ]
  }
}];
```

### Generate Daily Summary Report

Create a separate workflow that runs daily and generates a summary:

```javascript
// Query database for last 24 hours of audits
// Generate statistics:
// - Number of critical alerts
// - Average resource usage
// - Top problematic servers
// - Trend analysis

const summary = `
## Daily Server Audit Summary

**Date:** ${new Date().toLocaleDateString()}

**Critical Alerts:** ${criticalCount}
**OK Audits:** ${okCount}
**Alert Rate:** ${(criticalCount / (criticalCount + okCount) * 100).toFixed(1)}%

**Average Resource Usage:**
- CPU: ${avgCpu.toFixed(1)}%
- Memory: ${avgMem.toFixed(1)}%
- Disk: ${avgDisk.toFixed(1)}%

**Top Problematic Servers:**
${topServers.map(s => `- ${s.hostname}: ${s.alertCount} alerts`).join('\n')}
`;

return [{ json: { summary } }];
```

---

## Multi-Environment Setup

### Separate Workflows by Environment

Create multiple workflow files:
- `production-audit-workflow.json`
- `staging-audit-workflow.json`
- `development-audit-workflow.json`

Each workflow can have:
- Different server lists
- Different alert thresholds
- Different schedules
- Different notification channels

### Single Workflow with Environment Switching

Add an **Environment Variable** node at the start:

```javascript
const env = $env.AUDIT_ENV || 'production';

const serverConfigs = {
  production: [
    '192.168.1.10|admin|pass|auto|auto|auto',
    '192.168.1.11|admin|pass|auto|auto|auto'
  ],
  staging: [
    '192.168.2.10|admin|pass|auto|auto|auto',
    '192.168.2.11|admin|pass|auto|auto|auto'
  ],
  development: [
    '192.168.3.10|admin|pass|auto|auto|auto',
    '192.168.3.11|admin|pass|auto|auto|auto'
  ]
};

const thresholds = {
  production: { cpu: 90, mem: 85, disk: 85 },
  staging: { cpu: 95, mem: 90, disk: 90 },
  development: { cpu: 99, mem: 95, disk: 95 }
};

return {
  json: {
    environment: env,
    servers: serverConfigs[env].join('\n'),
    thresholds: thresholds[env]
  }
};
```

---

## Advanced Use Cases

### Check SSL Certificate Expiry

```javascript
// Linux: Check SSL certificates
try {
  const certCheck = execSync(
    `${sshBase} "echo | openssl s_client -servername google.com -connect google.com:443 2>/dev/null | openssl x509 -noout -dates"`,
    { encoding: 'utf8', timeout: 10000 }
  );
  auditData.ssl_cert_info = certCheck.trim();
  
  // Parse expiry date
  const expiryMatch = certCheck.match(/notAfter=(.+)/);
  if (expiryMatch) {
    const expiryDate = new Date(expiryMatch[1]);
    const daysUntilExpiry = Math.ceil((expiryDate - new Date()) / (1000 * 60 * 60 * 24));
    auditData.ssl_days_until_expiry = daysUntilExpiry;
  }
} catch (e) {
  auditData.ssl_error = 'SSL check failed';
}
```

### Monitor Database Connections

```javascript
// Linux: Check MySQL connections
try {
  const mysqlConnections = execSync(
    `${sshBase} "mysql -e 'SHOW STATUS LIKE \"Threads_connected\";' -N -s"`,
    { encoding: 'utf8', timeout: 10000 }
  );
  auditData.mysql_connections = parseInt(mysqlConnections.trim()) || 0;
} catch (e) {
  auditData.mysql_error = 'MySQL not available';
}

// Linux: Check PostgreSQL connections
try {
  const pgConnections = execSync(
    `${sshBase} "psql -U postgres -c 'SELECT count(*) FROM pg_stat_activity;' -t"`,
    { encoding: 'utf8', timeout: 10000 }
  );
  auditData.postgres_connections = parseInt(pgConnections.trim()) || 0;
} catch (e) {
  auditData.postgres_error = 'PostgreSQL not available';
}
```

### Check Application Health

```javascript
// Linux: Check HTTP endpoints
try {
  const httpCheck = execSync(
    `${sshBase} "curl -s -o /dev/null -w '%{http_code}' http://localhost:8080/health"`,
    { encoding: 'utf8', timeout: 10000 }
  );
  auditData.http_status_code = parseInt(httpCheck.trim());
  auditData.http_healthy = auditData.http_status_code === 200;
} catch (e) {
  auditData.http_healthy = false;
}
```

### Monitor Backup Status

```javascript
// Linux: Check recent backups
try {
  const backupCheck = execSync(
    `${sshBase} "find /backup -name '*.tar.gz' -mtime -1 | wc -l"`,
    { encoding: 'utf8', timeout: 10000 }
  );
  auditData.recent_backups = parseInt(backupCheck.trim()) || 0;
  auditData.backup_critical = auditData.recent_backups === 0;
} catch (e) {
  auditData.backup_error = 'Backup check failed';
}
```

---

## Performance Optimization

### Batch Process Servers

For large numbers of servers, process in batches:

```javascript
const batchSize = 10;
const allServers = $json.servers.split('\n');
const batches = [];

for (let i = 0; i < allServers.length; i += batchSize) {
  batches.push(allServers.slice(i, i + batchSize).join('\n'));
}

return batches.map(batch => ({
  json: { servers: batch, batch_number: Math.ceil(allServers.length / batchSize) }
}));
```

### Add Progress Tracking

```javascript
// Track which servers have been audited
return [{
  json: {
    total_servers: totalServers,
    audited_servers: auditedCount,
    critical_servers: criticalCount,
    progress: `${auditedCount}/${totalServers}`,
    percentage: ((auditedCount / totalServers) * 100).toFixed(1)
  }
}];
```

---

## Complete Example: Full-Stack Application Monitoring

```javascript
// Complete audit for a web application stack
const appStackAudit = {
  // Database
  database: {
    connections: 0,
    slow_queries: 0,
    replication_lag: 0
  },
  
  // Application Server
  app_server: {
    response_time: 0,
    error_rate: 0,
    active_sessions: 0
  },
  
  // Web Server
  web_server: {
    requests_per_sec: 0,
    active_connections: 0,
    ssl_days_left: 0
  },
  
  // Cache
  cache: {
    hit_rate: 0,
    memory_usage: 0,
    evictions: 0
  }
};

// Implement checks for each component...
// Return consolidated report

return { json: appStackAudit };
```

---

Need more examples? Check the [README.md](README.md) for the complete workflow documentation, or create an issue for specific use cases.