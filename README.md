# N8n Multi-Server Audit Workflow with AI & Telegram Alerts

A production-grade, automated server audit workflow for N8n that monitors both Linux and Windows servers, automatically detects the operating system, performs safe read-only audits, analyzes issues with AI, and sends critical alerts via Telegram.

## 🎯 Features

- **🤖 Multi-OS Support**: Automatically detects and audits both Linux and Windows servers
- **🔍 Auto-Detection**: Identifies OS type and Linux distribution automatically
- **🛡️ Production-Safe**: All audit commands are read-only - no harmful commands executed
- **📊 Comprehensive Auditing**: Monitors CPU, memory, disk, services, logs, and more
- **🤖 AI-Powered Analysis**: Uses AI agents to analyze issues and provide remediation steps
- **📱 Telegram Alerts**: Real-time notifications with AI recommendations for critical issues
- **⚙️ Configurable Thresholds**: Customize alert thresholds for your environment
- **🔐 Secure**: Supports SSH for Linux and WinRM for Windows with credential management

## 📋 Requirements

### N8n Setup

- N8n instance (self-hosted or cloud)
- Required N8n nodes:
  - `@n8n/n8n-nodes-langchain.agent` (AI Agent)
  - `@n8n/n8n-nodes-langchain.lmChatGoogleGemini` (or other LLM)
  - `n8n-nodes-base.scheduleTrigger`
  - `n8n-nodes-base.telegram`
  - `n8n-nodes-base.code`
  - `n8n-nodes-base.switch`
  - `n8n-nodes-base.set`
  - `n8n-nodes-base.splitOut`

### Linux Server Requirements

- SSH server enabled
- `sshpass` installed on the N8n host (for password authentication)
  ```bash
  sudo apt-get install sshpass  # Ubuntu/Debian
  sudo yum install sshpass      # CentOS/RHEL
  ```
- Required system tools:
  - `free` - Memory info
  - `df` - Disk usage
  - `uptime` - System load
  - `journalctl` - System logs
  - `iostat` (optional) - Disk I/O statistics
  - `systemctl` - Service management

### Windows Server Requirements

- WinRM enabled and configured
- PowerShell 3.0 or higher
- Network connectivity from N8n host to Windows server
- Recommended: HTTPS with certificate authentication

**Enable WinRM on Windows:**
```powershell
# Enable WinRM over HTTPS (recommended for production)
winrm quickconfig -transport:https
winrm set winrm/config/client '@{TrustedHosts="*"}'

# Enable WinRM over HTTP (for testing only)
winrm quickconfig
winrm set winrm/config/client '@{TrustedHosts="*"}'
```

### External Services

- **Telegram Bot**: Create via [@BotFather](https://t.me/BotFather)
- **AI Model**: Google Gemini API key (or OpenAI/Anthropic alternative)

## 🚀 Installation

### 1. Import the Workflow

1. Navigate to your N8n instance
2. Go to **Workflows** → **Import**
3. Upload `workflows/multi-server-audit-workflow.json`
4. The workflow will be imported but deactivated (safe by default)

### 2. Configure Credentials

#### Telegram Bot Credentials

1. Create a Telegram bot via [@BotFather](https://t.me/BotFather)
   - Send `/newbot`
   - Follow the prompts to create your bot
   - Save the API token provided

2. Get your Chat ID:
   ```bash
   # Use this URL in your browser (replace YOUR_BOT_TOKEN)
   https://api.telegram.org/botYOUR_BOT_TOKEN/getUpdates
   ```
   - Send a message to your bot, then check the JSON response
   - Look for `"chat":{"id":123456789}` - that's your Chat ID

3. Add credentials in N8n:
   - Go to **Credentials** → **Add Credential**
   - Select **Telegram API**
   - Enter your bot token
   - Name it something like "Telegram Bot"

#### AI Model Credentials (Google Gemini)

1. Get Google AI API key:
   - Go to [Google AI Studio](https://aistudio.google.com/app/apikey)
   - Create an API key
   - Copy the API key

2. Add credentials in N8n:
   - Go to **Credentials** → **Add Credential**
   - Select **Google Gemini (PaLM) API**
   - Enter your API key
   - Name it "Google Gemini API"

#### Alternative AI Models

The workflow supports these alternative AI models:

- **OpenAI GPT-4**: Use `@n8n/n8n-nodes-langchain.lmChatOpenAi` node
- **Anthropic Claude**: Use `@n8n/n8n-nodes-langchain.lmChatAnthropic` node
- **Azure OpenAI**: Use `@n8n/n8n-nodes-langchain.lmChatAzureOpenAi` node

To switch models:
1. Delete the "Gemini Model" node
2. Add your preferred model node
3. Connect it to the "AI Agent Analysis" node via the `ai_languageModel` output

### 3. Configure the Workflow

#### Edit the Config Node

1. Click on the **Config** node
2. Update the **servers** field with your server information:

**Format:**
```
host|username|password|os_type|port|protocol
```

**Parameters:**
- `host`: Server IP address or hostname
- `username`: SSH/WinRM username
- `password`: Password for authentication (or leave blank if using keys)
- `os_type`: Operating system type
  - `auto`: Auto-detect OS (recommended)
  - `linux`: Linux server
  - `windows`: Windows server
- `port`: Port number or `auto`
  - `auto`: Uses default (22 for SSH, 5986 for WinRM HTTPS)
  - Custom port number
- `protocol`: Connection protocol or `auto`
  - `auto`: Auto-detect protocol
  - `ssh`: SSH connection (Linux)
  - `winrm`: WinRM connection (Windows)

**Example Configuration:**
```
# Linux servers
192.168.1.10|admin|secure_password|auto|22|ssh
192.168.1.11|devops|another_password|linux|22|ssh

# Windows servers
192.168.1.20|administrator|win_password|auto|5986|winrm
192.168.1.21|svcaccount|svc_password|windows|5986|winrm

# Auto-detect mixed servers
192.168.1.30|admin|password123|auto|auto|auto
192.168.1.31|admin|password123|auto|auto|auto
```

#### Configure Thresholds

Update these values in the Config node:

- **time_window_min**: Time window in minutes for log analysis (default: 15)
- **mem_critical_pct**: Memory usage % to trigger critical alert (default: 85)
- **disk_critical_pct**: Disk usage % to trigger critical alert (default: 85)
- **cpu_critical_pct**: CPU usage % to trigger critical alert (default: 90)
- **max_error_lines**: Maximum error lines to analyze (default: 50)
- **telegram_chat_id**: Your Telegram Chat ID for alerts

#### Configure Telegram Node

1. Click on the **Send Telegram Alert** node
2. Select your Telegram credentials from the dropdown
3. Verify the Chat ID is correctly set

#### Configure AI Model Node

1. Click on the **Gemini Model** node
2. Select your Google Gemini credentials
3. Adjust model settings if needed:
   - Model: `models/gemini-2.0-flash-exp` (fast, efficient)
   - Max Output Tokens: 512
   - Temperature: 0.3 (lower = more deterministic)

### 4. Activate the Workflow

1. Review all configurations
2. Click the **Active** toggle in the top-right corner
3. The workflow will run according to the Schedule Trigger (default: every 30 minutes)

## 📊 Audit Metrics

### Linux Servers

| Metric | Description | Command Used |
|--------|-------------|--------------|
| **CPU Load** | System load average (1, 5, 15 min) | `uptime` |
| **Memory** | Total, used, free memory in MB | `free -m` |
| **Disk Space** | Disk usage percentage | `df -h` |
| **Disk I/O** | Disk utilization percentage | `iostat` |
| **Journal Errors** | Recent error logs | `journalctl -p err` |
| **Services** | Status of key services | `systemctl` |
| **Processes** | Number of running processes | `ps aux \| wc -l` |
| **Network** | Active network connections | `netstat` / `ss` |
| **Users** | Logged-in users | `who` |
| **Uptime** | System uptime | `uptime -p` |

### Windows Servers

| Metric | Description | PowerShell Command |
|--------|-------------|-------------------|
| **CPU** | CPU usage percentage | `Get-WmiObject Win32_Processor` |
| **Memory** | Memory usage percentage | `Get-WmiObject Win32_OperatingSystem` |
| **Disk** | Disk usage percentage | `Get-WmiObject Win32_LogicalDisk` |
| **Event Errors** | Recent error logs | `Get-EventLog` |
| **Services** | Stopped auto-start services | `Get-Service` |
| **Processes** | Number of running processes | `(Get-Process).Count` |
| **Uptime** | System uptime | `Get-WmiObject Win32_OperatingSystem` |

## 🔐 Security Best Practices

### For Production Environments

1. **Use SSH Keys Instead of Passwords**
   ```bash
   # Generate SSH key on N8n host
   ssh-keygen -t ed25519 -f ~/.ssh/n8n_audit

   # Copy public key to Linux servers
   ssh-copy-id -i ~/.ssh/n8n_audit.pub user@server
   ```

2. **Use WinRM Certificate Authentication**
   - Configure WinRM with HTTPS and certificates
   - Avoid basic authentication in production

3. **Network Security**
   - Use VPN or private network connections
   - Restrict WinRM and SSH access by firewall
   - Implement IP whitelisting

4. **Credential Management**
   - Use N8n's credential manager (don't hardcode passwords)
   - Rotate credentials regularly
   - Use different credentials for different environments

5. **Least Privilege**
   - Use dedicated service accounts with minimal permissions
   - Grant only read-only access for audit commands

### Audit Commands Safety

All commands used in this workflow are **read-only** and safe for production:

- ✅ `free`, `df`, `uptime` - System monitoring (read-only)
- ✅ `journalctl` - Log reading (read-only)
- ✅ `systemctl is-active` - Service status checking (read-only)
- ✅ `iostat` - Disk I/O statistics (read-only)
- ✅ `ps`, `who`, `netstat` - Process/user/network info (read-only)
- ✅ PowerShell `Get-*` cmdlets - Read-only information retrieval

**NO** harmful commands:
- ❌ No `rm`, `delete`, `format` commands
- ❌ No service restarts/stops
- ❌ No configuration modifications
- ❌ No package installations

## 📱 Telegram Alert Format

Critical alerts include:

```
🚨 CRITICAL ALERT

🐧 Server: 192.168.1.10
OS: LINUX (ubuntu)
User: admin

Issues Detected:
⚠️ CONNECTION FAILED

---

AI Analysis & Recommendations:

Root Cause Analysis:
• SSH connection timeout
• Network connectivity issue

Immediate Actions:
• Test connectivity: ping 192.168.1.10
• Check firewall: sudo ufw status
• Verify SSH: systemctl status ssh

Prevention:
• Implement network monitoring
• Set up failover SSH access

---

🕐 2024-01-15T10:30:00.000Z
Multi-Server Audit Workflow
```

## 🛠️ Troubleshooting

### Common Issues

#### 1. SSH Connection Failed

**Problem:** `SSH connection failed: Connection refused`

**Solutions:**
- Verify SSH server is running: `systemctl status ssh`
- Check firewall: `sudo ufw status`
- Verify port: `telnet <server> 22`
- Check credentials in Config node

#### 2. WinRM Connection Failed

**Problem:** WinRM connection not working

**Solutions:**
```powershell
# On Windows server, check WinRM status
winrm quickconfig
winrm get winrm/config/listener

# Test from N8n host
curl -k https://<windows-server>:5986/wsman
```

#### 3. No Telegram Alerts Received

**Problem:** Workflow runs but no alerts

**Solutions:**
- Verify chat_id is correct
- Check bot has permission to send messages
- Test bot: Send `/start` to your bot first
- Check N8n execution logs for errors

#### 4. AI Analysis Not Working

**Problem:** AI Agent returns empty or error

**Solutions:**
- Verify API key is valid
- Check API quota/limits
- Try a different AI model
- Check network connectivity to AI service

#### 5. Permission Denied Errors

**Problem:** Commands return "Permission denied"

**Solutions:**
- Use `sudo` (configure passwordless sudo for specific commands)
- Check user permissions on target server
- Verify command is safe and read-only

### Debug Mode

To enable detailed logging:

1. Add a new **Set** node after each audit node
2. Enable "Always Output Data" in node settings
3. Check N8n execution logs for detailed output

## 📈 Performance Considerations

### Large Server Lists

If auditing many servers:

1. **Increase Schedule Interval**: Change to 60+ minutes
2. **Implement Batch Processing**: Process servers in groups
3. **Add Delays**: Increase delay in "Batch Critical Servers" node
4. **Parallel Execution**: Consider splitting into multiple workflows

### Network Latency

For servers in different regions:

- Increase SSH/WinRM timeouts in code nodes (default: 10-20 seconds)
- Consider regional N8n instances
- Use CDN or load balancers

## 🔄 Customization

### Adding Custom Audit Checks

To add custom checks for Linux:

1. Edit the **Linux Audit** node
2. Add new SSH commands in the `try` block
3. Store results in the `auditData` object
4. Update **Analyze Results** node to check for issues

Example:
```javascript
// Custom check: Docker container status
try {
  const dockerStatus = execSync(
    `${sshBase} "docker ps -a --format '{{.Names}}|{{.Status}}'"`,
    { encoding: 'utf8', timeout: 10000 }
  );
  auditData.docker_containers = dockerStatus.trim();
} catch (e) {
  auditData.docker_error = 'Docker not installed or inaccessible';
}
```

### Custom Alert Channels

To add more alert channels (email, Slack, etc.):

1. Add new nodes after **Prepare Telegram**
2. Duplicate the flow for each channel
3. Configure channel-specific credentials
4. Use different branches from Status Router

### Adjusting AI Prompts

To customize AI analysis:

1. Edit the **AI Agent Analysis** node
2. Modify the system message or prompt
3. Adjust output format requirements
4. Tune temperature (0.1-0.5 for more deterministic)

## 📚 Additional Resources

- [N8n Documentation](https://docs.n8n.io)
- [SSH Key Authentication](https://www.ssh.com/academy/ssh/key)
- [WinRM Configuration](https://docs.microsoft.com/en-us/windows/win32/winrm/overview)
- [Telegram Bot API](https://core.telegram.org/bots/api)
- [Google AI Studio](https://aistudio.google.com)

## 🤝 Contributing

Contributions are welcome! Areas for improvement:

- Additional OS support (macOS, BSD)
- More audit metrics
- Enhanced AI analysis
- Additional alert channels
- Web dashboard integration
- Historical reporting

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## ⚠️ Disclaimer

This workflow is provided as-is for monitoring and auditing purposes. Always:
- Test in non-production environments first
- Review and understand all commands
- Implement proper security measures
- Keep credentials secure
- Monitor workflow performance

The authors are not responsible for any issues arising from the use of this workflow.

---

**Version:** 1.0.0  
**Last Updated:** 2024-01-15  
**Supported N8n Version:** 1.0.0+  
**Requires:** N8n with LangChain nodes enabled