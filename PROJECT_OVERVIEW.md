# N8n Multi-Server Audit Workflow - Project Overview

A production-grade, automated server audit solution for N8n that monitors both Linux and Windows servers with AI-powered analysis and real-time alerts.

## Quick Links

- 📖 [README.md](README.md) - Complete documentation
- 🚀 [QUICKSTART.md](QUICKSTART.md) - 10-minute setup guide
- 🐧 [LINUX_SETUP.md](LINUX_SETUP.md) - Linux server configuration
- 🪟 [WINDOWS_SETUP.md](WINDOWS_SETUP.md) - Windows server configuration
- 🔧 [WORKFLOW_CUSTOMIZATION.md](WORKFLOW_CUSTOMIZATION.md) - Customization examples
- 🏗️ [WORKFLOW_ARCHITECTURE.md](WORKFLOW_ARCHITECTURE.md) - Workflow architecture
- 📝 [CHANGELOG.md](CHANGELOG.md) - Version history
- 🤝 [CONTRIBUTING.md](CONTRIBUTING.md) - Contribution guidelines

## Project Structure

```
N8n_Agentic_SystemAudit_Workflow/
├── workflows/
│   └── multi-server-audit-workflow.json    # Main N8n workflow
├── README.md                               # Complete documentation (480+ lines)
├── QUICKSTART.md                           # Quick setup guide
├── LINUX_SETUP.md                          # Linux configuration guide
├── WINDOWS_SETUP.md                        # Windows configuration guide
├── WORKFLOW_CUSTOMIZATION.md               # Customization examples
├── WORKFLOW_ARCHITECTURE.md               # Architecture documentation
├── CHANGELOG.md                           # Version history
├── CONTRIBUTING.md                         # Contribution guidelines
├── .env.example                           # Environment template
├── .gitignore                             # Git ignore rules
└── LICENSE                                # MIT License
```

## Key Features at a Glance

### 🎯 Multi-OS Support
- **Linux**: SSH-based auditing with distro detection
  - Supports: Ubuntu, Debian, CentOS, RHEL, Arch, Alpine, and more
  - Metrics: CPU load, memory, disk, journal logs, services, network, users

- **Windows**: WinRM/PowerShell-based auditing
  - Supports: Windows Server 2012 R2+
  - Metrics: CPU, memory, disk, event logs, services, processes, uptime

- **Auto-Detection**: Automatically identifies OS and selects appropriate protocol

### 🤖 AI-Powered Analysis
- **Supported Models**: Google Gemini (default), OpenAI GPT-4, Anthropic Claude, Azure OpenAI
- **Analysis Type**:
  - Root cause identification
  - Immediate remediation steps (safe, read-only commands)
  - Prevention recommendations
- **Customizable**: Modify prompts for different teams (Security, DevOps, DBA)

### 📱 Real-Time Alerts
- **Telegram**: Formatted alerts with:
  - Server details (OS, hostname, user)
  - Issue summary with emojis
  - AI-generated recommendations
  - Safe remediation commands

- **Extensible**: Easy to add Email, Slack, Discord, Microsoft Teams, PagerDuty, etc.

### 🛡️ Production-Safe
- **Read-Only Commands**: All audit commands are safe for production
  - ✅ System monitoring (free, df, uptime, Get-WmiObject)
  - ✅ Log reading (journalctl, Get-EventLog)
  - ✅ Service status checking (systemctl is-active, Get-Service)
  - ❌ NO harmful commands (rm, delete, format, service modifications)

- **Security Features**:
  - SSH key-based authentication support
  - WinRM HTTPS with certificate authentication
  - Least privilege service accounts
  - Secure credential management

### 📊 Comprehensive Metrics

| Metric | Linux | Windows |
|--------|--------|---------|
| CPU Usage | ✅ Load average | ✅ Percentage |
| Memory Usage | ✅ Total/Used/Free % | ✅ Percentage |
| Disk Space | ✅ Usage % | ✅ Usage % |
| Disk I/O | ✅ Utilization % | ❌ (PowerShell available) |
| System Logs | ✅ Journal errors | ✅ Event errors |
| Services | ✅ Status checks | ✅ Stopped services |
| Processes | ✅ Count | ✅ Count |
| Network | ✅ Connections | ❌ (PowerShell available) |
| Users | ✅ Logged-in | ❌ (PowerShell available) |
| Uptime | ✅ Human readable | ✅ Human readable |

## Getting Started

### Prerequisites
- N8n instance (self-hosted or cloud)
- At least one Linux or Windows server
- Telegram account for alerts
- AI API key (Google Gemini, OpenAI, or Anthropic)

### Quick Setup (5 steps)

1. **Import Workflow** (1 min)
   - N8n → Workflows → Import
   - Upload `workflows/multi-server-audit-workflow.json`

2. **Setup Telegram** (2 min)
   - Create bot via @BotFather
   - Get Chat ID
   - Add credentials to N8n

3. **Setup AI Model** (2 min)
   - Get Google Gemini API key
   - Add credentials to N8n

4. **Configure Servers** (2 min)
   - Edit Config node
   - Add servers: `host|user|pass|os_type|port|protocol`

5. **Activate** (10 sec)
   - Click Active toggle
   - Workflow runs every 30 minutes

### Detailed Setup

For comprehensive instructions, see:
- [Quick Start Guide](QUICKSTART.md) - 10-minute walkthrough
- [README.md](README.md) - Complete documentation
- [Linux Setup Guide](LINUX_SETUP.md) - Linux server configuration
- [Windows Setup Guide](WINDOWS_SETUP.md) - Windows server configuration

## Server Configuration

### Format
```
host|username|password|os_type|port|protocol
```

### Parameters
- `host`: Server IP or hostname
- `username`: SSH/WinRM username
- `password`: Password (or blank for SSH keys)
- `os_type`: `auto` (recommended), `linux`, `windows`
- `port`: `auto` or specific port (22 for SSH, 5986 for WinRM)
- `protocol`: `auto` (recommended), `ssh`, `winrm`

### Examples

```bash
# Linux with auto-detection
192.168.1.10|admin|password123|auto|auto|auto

# Windows with auto-detection
192.168.1.20|administrator|winpass|auto|auto|auto

# Linux with SSH key (password empty)
192.168.1.30|n8n_audit||linux|22|ssh

# Windows with WinRM HTTPS
192.168.1.40|svcaccount|svcpass|windows|5986|winrm

# Mixed environment with comments
# Production Linux servers
192.168.1.10|admin|pass1|linux|22|ssh
192.168.1.11|admin|pass2|linux|22|ssh

# Production Windows servers
192.168.1.20|admin|pass3|windows|5986|winrm

# Development servers (auto-detect)
192.168.1.100|dev|devpass|auto|auto|auto
```

## Alert Thresholds

Configurable in the **Config** node:

| Threshold | Default | Description |
|----------|---------|-------------|
| `cpu_critical_pct` | 90 | CPU usage % to trigger alert |
| `mem_critical_pct` | 85 | Memory usage % to trigger alert |
| `disk_critical_pct` | 85 | Disk usage % to trigger alert |
| `time_window_min` | 15 | Log analysis time window (minutes) |
| `max_error_lines` | 50 | Maximum error lines to analyze |

## Customization

### Add Custom Audit Checks
See [WORKFLOW_CUSTOMIZATION.md](WORKFLOW_CUSTOMIZATION.md) for examples:
- Docker container monitoring
- SSL certificate expiry checks
- Database connection monitoring
- Application health checks
- Backup status monitoring

### Add Custom Alert Channels
Easy integration with:
- Email (SMTP)
- Slack (Webhooks)
- Discord (Bot API)
- Microsoft Teams (Webhooks)
- PagerDuty (REST API)
- Opsgenie (REST API)

### Customize AI Analysis
Modify AI prompts for different scenarios:
- Security incident response
- DevOps capacity planning
- Database performance tuning
- Application monitoring

## Workflow Architecture

```
Schedule Trigger → Config → Parse Servers → Split Servers
    → Detect OS → Route → [Linux/Windows/Error]
    → Audit → Analyze → Status Router → [Critical/OK]
    → [Batch Critical → AI Analysis → Prepare Telegram → Send Alert]
    → [Log OK]
```

Detailed architecture: [WORKFLOW_ARCHITECTURE.md](WORKFLOW_ARCHITECTURE.md)

## Performance

| Scale | Servers | Schedule | Avg. Execution Time |
|-------|----------|-----------|-------------------|
| Small | < 10 | 30 min | 2-3 min |
| Medium | 10-50 | 60 min | 8-12 min |
| Large | 50+ | 2-4 hours | 20-40 min |

*Times vary based on network latency and AI service response time*

## Security Best Practices

### For Production
1. ✅ Use SSH keys instead of passwords
2. ✅ Use WinRM HTTPS with certificates for Windows
3. ✅ Create dedicated service accounts with minimal permissions
4. ✅ Implement firewall rules
5. ✅ Use VPN or private networks
6. ✅ Rotate credentials regularly
7. ✅ Never commit credentials to version control
8. ✅ Review all commands before deployment

### Command Safety
All audit commands are **read-only**:
- ✅ Monitoring commands (free, df, uptime, Get-WmiObject)
- ✅ Log reading (journalctl, Get-EventLog)
- ✅ Service status (systemctl is-active, Get-Service)

**NO** harmful commands:
- ❌ No rm, delete, format commands
- ❌ No service restarts/stops
- ❌ No configuration modifications
- ❌ No package installations

## Troubleshooting

### Common Issues
- SSH connection failed → Check SSH server, firewall, credentials
- WinRM connection failed → Enable WinRM, configure firewall
- No Telegram alerts → Verify bot, Chat ID, check logs
- AI analysis empty → Check API key, quota, network
- Permission denied → Check user permissions, sudo config

For detailed troubleshooting:
- [README.md - Troubleshooting Section](README.md#-troubleshooting)
- [Linux Setup Guide - Troubleshooting](LINUX_SETUP.md#troubleshooting)
- [Windows Setup Guide - Troubleshooting](WINDOWS_SETUP.md#troubleshooting)

## Documentation Files

| File | Purpose | Lines |
|------|---------|-------|
| [README.md](README.md) | Complete documentation | 480+ |
| [QUICKSTART.md](QUICKSTART.md) | 10-minute setup guide | 150+ |
| [LINUX_SETUP.md](LINUX_SETUP.md) | Linux server configuration | 350+ |
| [WINDOWS_SETUP.md](WINDOWS_SETUP.md) | Windows server configuration | 300+ |
| [WORKFLOW_CUSTOMIZATION.md](WORKFLOW_CUSTOMIZATION.md) | Customization examples | 450+ |
| [WORKFLOW_ARCHITECTURE.md](WORKFLOW_ARCHITECTURE.md) | Workflow architecture | 400+ |
| [CHANGELOG.md](CHANGELOG.md) | Version history | 200+ |
| [CONTRIBUTING.md](CONTRIBUTING.md) | Contribution guidelines | 300+ |

## Version

**Current Version:** 1.0.0
**Release Date:** 2024-01-15
**N8n Version Required:** 1.0.0+
**Required Nodes:** LangChain nodes must be enabled

## Roadmap

### Version 1.1.0 (Planned)
- Web dashboard for audit history
- Historical trend analysis
- Multi-environment workflow templates
- macOS server support
- BSD server support

### Version 1.2.0 (Planned)
- Custom alert rules engine
- Dependency graph visualization
- Automated remediation workflows
- Integration with Prometheus/Grafana
- Mobile app for alerts

### Version 2.0.0 (Planned)
- Machine learning for anomaly detection
- Predictive maintenance alerts
- Multi-tenant support
- RBAC for workflow management
- Distributed execution mode

## License

MIT License - See [LICENSE](LICENSE) file for details.

## Support & Community

### Getting Help
1. Check documentation (README.md, setup guides)
2. Review WORKFLOW_CUSTOMIZATION.md for examples
3. Check existing GitHub issues
4. Create a new issue with details

### Contributing
We welcome contributions! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

### Areas for Contribution
- New OS support (macOS, BSD)
- Additional audit metrics
- New alert channels
- Enhanced AI analysis
- Documentation improvements
- Bug fixes and optimizations

## Credits

- **Workflow**: Multi-Server Audit with AI + Telegram
- **AI Integration**: Google Gemini, OpenAI, Anthropic
- **Notifications**: Telegram Bot API
- **Inspiration**: Based on production server monitoring needs

## Acknowledgments

- N8n team for the excellent automation platform
- Google for Gemini AI API
- OpenAI for GPT models
- Anthropic for Claude models
- All contributors and testers

---

**Status:** ✅ Production Ready
**Last Updated:** 2024-01-15
**Maintained By:** Community Contributors

For the complete experience, start with [QUICKSTART.md](QUICKSTART.md) to get up and running in 10 minutes!