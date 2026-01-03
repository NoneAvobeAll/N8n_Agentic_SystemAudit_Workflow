# Changelog

All notable changes to the N8n Multi-Server Audit Workflow project.

## [1.0.0] - 2024-01-15

### Added
- Initial release of production-grade Multi-Server Audit Workflow
- **Multi-OS Support**: Automatic detection and auditing of Linux and Windows servers
- **Linux Distro Detection**: Automatically identifies Ubuntu, Debian, CentOS, RHEL, and other distributions
- **Production-Safe Auditing**: All commands are read-only - no harmful commands executed
- **Comprehensive Metrics**:
  - Linux: CPU load, memory, disk space & I/O, journal errors, service status, network connections, logged users, uptime
  - Windows: CPU usage, memory usage, disk usage, event log errors, stopped services, process count, uptime
- **AI-Powered Analysis**: Integrated AI agent (Google Gemini) for issue analysis and remediation recommendations
- **Real-Time Alerts**: Telegram notifications with AI-generated recommendations
- **Configurable Thresholds**: Customizable CPU, memory, and disk usage thresholds
- **Configuration Management**: Centralized Config node for server credentials and settings
- **Auto-Detection**: Automatically detects OS type and selects appropriate audit protocol
- **Multi-Protocol Support**: SSH for Linux, WinRM for Windows
- **Scheduled Execution**: Runs at configurable intervals (default: 30 minutes)
- **Error Handling**: Robust error handling and connection failure detection
- **Batch Processing**: Efficient processing of multiple servers
- **Status Routing**: Separate handling for critical and OK servers

### Documentation
- **README.md**: Comprehensive 480+ line documentation covering installation, configuration, security, and troubleshooting
- **QUICKSTART.md**: 10-minute setup guide for rapid deployment
- **LINUX_SETUP.md**: Detailed Linux server configuration guide with distribution-specific instructions
- **WINDOWS_SETUP.md**: Complete Windows server setup guide with WinRM configuration
- **WORKFLOW_CUSTOMIZATION.md**: Advanced customization examples and templates
- **.env.example**: Environment variable template for configuration
- **.gitignore**: Proper exclusion of sensitive files

### Security Features
- Read-only audit commands only (no rm, delete, format, service modifications)
- SSH key-based authentication support
- WinRM HTTPS with certificate authentication support
- Least privilege service account configuration
- Firewall configuration guides
- Secure credential management recommendations

### Workflow Features
- **Schedule Trigger**: Configurable scheduling (interval, cron)
- **Config Node**: Centralized configuration for servers and thresholds
- **OS Detection**: Automatic OS and Linux distro detection
- **OS Router**: Smart routing to Linux/Windows/Error audit paths
- **Linux Audit Node**: Comprehensive Linux metrics collection via SSH
- **Windows Audit Node**: Windows metrics collection via WinRM/PowerShell
- **Analysis Node**: Threshold evaluation and issue detection
- **Status Router**: Separate flows for critical and OK servers
- **Batch Critical Servers**: Consolidates multiple critical servers for AI analysis
- **AI Agent Analysis**: LLM-powered issue analysis and recommendations
- **Telegram Alert**: Formatted alerts with AI recommendations
- **Log OK Servers**: Logging of healthy server audits

### Server Configuration Format
```
host|username|password|os_type|port|protocol
```
- os_type: auto, linux, windows
- port: auto or specific port number
- protocol: auto, ssh, winrm

### Supported AI Models
- Google Gemini (default, recommended)
- OpenAI GPT-4
- Anthropic Claude
- Azure OpenAI

### Monitoring Thresholds (Configurable)
- CPU critical: 90% (default)
- Memory critical: 85% (default)
- Disk critical: 85% (default)
- Time window for logs: 15 minutes (default)

### Troubleshooting Guides
- SSH connection failures
- WinRM connection issues
- Telegram alert problems
- AI analysis errors
- Permission denied errors

### Customization Examples
- Docker container monitoring
- SSL certificate expiry checks
- Database connection monitoring
- Application health checks
- Backup status monitoring
- Custom alert channels (Email, Slack, Discord)
- Environment-based server grouping
- Data export to databases and spreadsheets

### Performance Considerations
- Batch processing for large server lists
- Network latency handling
- Configurable timeouts
- Rate limiting for external APIs

---

## Future Enhancements (Planned)

### Version 1.1.0 (Proposed)
- [ ] Web dashboard for audit history
- [ ] Historical trend analysis
- [ ] Multi-environment workflow templates
- [ ] macOS server support
- [ ] BSD server support
- [ ] Container orchestration monitoring (Kubernetes, Docker Swarm)

### Version 1.2.0 (Proposed)
- [ ] Custom alert rules engine
- [ ] Dependency graph visualization
- [ ] Automated remediation workflows
- [ ] Integration with popular monitoring tools (Prometheus, Grafana)
- [ ] Mobile app for alerts

### Version 2.0.0 (Proposed)
- [ ] Machine learning for anomaly detection
- [ ] Predictive maintenance alerts
- [ ] Multi-tenant support
- [ ] RBAC for workflow management
- [ ] Distributed execution mode

---

## Migration Notes

### From Original Linux-Only Workflow
The original Linux-only workflow (`Multi-Server Error Monitoring with AI + Telegram - FIXED`) has been completely refactored to support:
- Multi-OS (Linux + Windows)
- Auto-detection capabilities
- Enhanced security measures
- Production-safe audit commands
- More comprehensive metrics
- Better error handling
- Advanced AI analysis

**No backward compatibility** - this is a complete rewrite. Configure servers from scratch using the new format.

---

## Breaking Changes

### Version 1.0.0
- Server configuration format changed from simple `host|user|pass` to `host|user|pass|os_type|port|protocol`
- Workflow structure completely redesigned
- Requires N8n with LangChain nodes enabled
- Requires Google Gemini API or alternative AI model

---

## Security Advisories

### Version 1.0.0 - Security Best Practices
- ⚠️ Never commit server credentials to version control
- ⚠️ Use SSH keys instead of passwords in production
- ⚠️ Enable WinRM HTTPS with certificate authentication for Windows
- ⚠️ Use dedicated service accounts with minimal permissions
- ⚠️ Review all audit commands before deployment
- ⚠️ Test in non-production environments first

---

## License

GNU General Public License v3.0 - See LICENSE file for details.

---

## Support

For issues, questions, or contributions:
1. Check documentation (README.md, QUICKSTART.md, setup guides)
2. Review WORKFLOW_CUSTOMIZATION.md for examples
3. Test connection and commands manually before deployment
4. Check N8n execution logs for detailed errors

---

**Maintained by**: Community contributors
**Last Updated**: 2024-01-15