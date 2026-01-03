# Windows Server Setup Guide

Detailed guide for configuring Windows servers for audit with the N8n Multi-Server Audit Workflow.

## Prerequisites

- Windows Server 2012 R2 or later
- PowerShell 3.0 or higher
- Administrator access

## Quick Setup (5 minutes)

Run these commands in PowerShell **as Administrator**:

```powershell
# 1. Enable WinRM
winrm quickconfig -transport:https

# 2. Allow connections from N8n host (replace with N8n server IP)
winrm set winrm/config/client '@{TrustedHosts="N8N_SERVER_IP"}'

# 3. Verify WinRM is running
winrm get winrm/config/listener

# 4. Test WinRM (from N8n host)
# Test from another machine: Test-WSMan -ComputerName YOUR_WINDOWS_SERVER_IP
```

## Detailed Configuration

### 1. Enable WinRM

WinRM (Windows Remote Management) is required for remote PowerShell execution.

**Option A: Quick Setup (HTTP - for testing)**
```powershell
winrm quickconfig
winrm set winrm/config/client '@{TrustedHosts="*"}'
```

**Option B: Secure Setup (HTTPS - for production)**
```powershell
# Enable WinRM over HTTPS
winrm quickconfig -transport:https

# Create a self-signed certificate
$cert = New-SelfSignedCertificate -DnsName "YOUR_SERVER_FQDN" -CertStoreLocation "Cert:\LocalMachine\My"

# Get certificate thumbprint
$certThumbprint = $cert.Thumbprint

# Create HTTPS listener
New-Item -Path WSMan:\localhost\Listener -Transport HTTPS -Address * -CertificateThumbprint $certThumbprint -Force

# Allow connections from N8n host
winrm set winrm/config/client '@{TrustedHosts="N8N_SERVER_IP"}'
```

### 2. Configure Firewall

Allow WinRM traffic through Windows Firewall:

```powershell
# Allow WinRM HTTP (port 5985)
New-NetFirewallRule -DisplayName "Windows Remote Management (HTTP-In)" -Direction Inbound -LocalPort 5985 -Protocol TCP -Action Allow

# Allow WinRM HTTPS (port 5986) - recommended for production
New-NetFirewallRule -DisplayName "Windows Remote Management (HTTPS-In)" -Direction Inbound -LocalPort 5986 -Protocol TCP -Action Allow

# Verify rules
Get-NetFirewallRule -DisplayName "*WinRM*"
```

### 3. Configure WinRM Settings

Optimize WinRM for auditing:

```powershell
# Increase max memory per shell (for larger data)
winrm set winrm/config/winrs '@{MaxMemoryPerShellMB=512}'

# Increase max timeout (seconds)
winrm set winrm/config '@{MaxTimeoutms=60000}'

# Allow unlimited data size
winrm set winrm/config '@{MaxEnvelopeSizeKB=500}'

# Enable basic authentication (if needed - not recommended for production)
winrm set winrm/config/service/auth '@{Basic=true}'
winrm set winrm/config/service '@{AllowUnencrypted=false}'
```

### 4. Create Dedicated Service Account

**Best Practice:** Use a dedicated service account with minimal permissions.

```powershell
# Create service account
$Password = ConvertTo-SecureString "SecurePassword123!" -AsPlainText -Force
New-LocalUser -Name "n8n_audit" -Password $Password -Description "N8n Audit Service Account"

# Add to Remote Management Users group
Add-LocalGroupMember -Group "Remote Management Users" -Member "n8n_audit"

# Grant necessary permissions (read-only)
# This user needs permission to run read-only PowerShell commands
```

### 5. Grant Required Permissions

The service account needs these permissions:

- **Remote Management Users group**: Required for WinRM access
- **Event Log Readers group**: To read event logs
- **Performance Monitor Users group**: To read performance counters

```powershell
# Add to additional groups
Add-LocalGroupMember -Group "Event Log Readers" -Member "n8n_audit"
Add-LocalGroupMember -Group "Performance Monitor Users" -Member "n8n_audit"

# Verify group membership
Get-LocalGroupMember -Group "Remote Management Users"
Get-LocalGroupMember -Group "Event Log Readers"
```

### 6. Test WinRM Connection

Test from your N8n host:

**Using PowerShell (from N8n host):**
```powershell
# Test connection
Test-WSMan -ComputerName YOUR_WINDOWS_SERVER_IP -Port 5986 -UseSSL

# Test remote command
Invoke-Command -ComputerName YOUR_WINDOWS_SERVER_IP -Port 5986 -UseSSL -Credential (Get-Credential) -ScriptBlock { Get-Process }
```

**Using curl (from Linux N8n host):**
```bash
# Test HTTPS connection
curl -k -u username:password https://YOUR_WINDOWS_SERVER_IP:5986/wsman

# Test HTTP connection
curl -u username:password http://YOUR_WINDOWS_SERVER_IP:5985/wsman
```

## Security Hardening

### 1. Use HTTPS with Certificates

Generate and use proper SSL certificates instead of self-signed:

```powershell
# Import your organization's certificate
Import-Certificate -FilePath "C:\certs\server.crt" -CertStoreLocation Cert:\LocalMachine\My

# Update WinRM listener
Set-Item -Path WSMan:\localhost\Listener\Listener_*\CertificateThumbprint -Value "YOUR_CERT_THUMBPRINT"
```

### 2. Disable Basic Authentication

Use NTLM or Kerberos authentication instead:

```powershell
# Disable basic authentication
winrm set winrm/config/service/auth '@{Basic=false}'
winrm set winrm/config/service/auth '@{Negotiate=true}'
winrm set winrm/config/service/auth '@{Kerberos=true}'
```

### 3. Limit Trusted Hosts

Only allow specific hosts:

```powershell
# Allow only your N8n server
winrm set winrm/config/client '@{TrustedHosts="N8N_SERVER_IP"}'

# Allow multiple specific hosts
winrm set winrm/config/client '@{TrustedHosts="N8N_SERVER_IP1,N8N_SERVER_IP2"}'
```

### 4. Enable WinRM Logging

```powershell
# Enable WinRM logging
winrm set winrm/config '@{EnableLogging=true}'

# Check logs
Get-WinEvent -LogName Microsoft-Windows-WinRM/Operational -MaxEvents 10
```

## Troubleshooting

### Issue: "Access Denied" Error

**Solution:**
```powershell
# Verify user is in correct groups
Get-LocalGroupMember -Group "Remote Management Users"

# Check WinRM service
Get-Service WinRM | Select-Object Status, StartType

# Restart WinRM service
Restart-Service WinRM
```

### Issue: "Connection Refused"

**Solution:**
```powershell
# Check WinRM listener
winrm enumerate winrm/config/listener

# Check firewall
Get-NetFirewallRule -DisplayName "*WinRM*"

# Test local connection
Test-WSMan -ComputerName localhost
```

### Issue: "SSL Certificate Error"

**Solution:**
```powershell
# Check certificate
Get-ChildItem -Path Cert:\LocalMachine\My

# If using self-signed, export and import on N8n host
# Or use -SkipCACheck parameter (not recommended for production)
```

### Issue: "Command Timeout"

**Solution:**
```powershell
# Increase timeout
winrm set winrm/config '@{MaxTimeoutms=120000}'

# Increase shell timeout
winrm set winrm/config/winrs '@{IdleTimeout=7200000}'
```

## Required PowerShell Commands

The workflow uses these read-only PowerShell commands:

| Command | Purpose | Safety |
|---------|---------|--------|
| `Get-WmiObject Win32_Processor` | CPU usage | ✅ Read-only |
| `Get-WmiObject Win32_OperatingSystem` | Memory & uptime | ✅ Read-only |
| `Get-WmiObject Win32_LogicalDisk` | Disk usage | ✅ Read-only |
| `Get-EventLog` | Event logs | ✅ Read-only |
| `Get-Service` | Service status | ✅ Read-only |
| `(Get-Process).Count` | Process count | ✅ Read-only |

## Alternative: Using SSH on Windows

If WinRM is not an option, you can install OpenSSH Server on Windows:

```powershell
# Install OpenSSH Server
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0

# Start and enable SSH service
Start-Service sshd
Set-Service -Name sshd -StartupType 'Automatic'

# Configure firewall
New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22

# Now you can connect via SSH like Linux servers
# Use os_type: linux in your Config node
```

## Advanced Configuration

### Custom PowerShell Execution Policy

```powershell
# Set to RemoteSigned for flexibility
Set-ExecutionPolicy RemoteSigned -Scope Process

# Or allow specific scripts
Set-ExecutionPolicy Bypass -Scope Process
```

### Configure WinRM Quotas

```powershell
# Max concurrent shells
winrm set winrm/config/winrs '@{MaxShellsPerUser=10}'

# Max concurrent operations per shell
winrm set winrm/config/winrs '@{MaxOperationsPerShell=50}'

# Max processes per shell
winrm set winrm/config/winrs '@{MaxProcessesPerShell=20}'
```

## Testing Checklist

Before adding the server to your N8n workflow:

- [ ] WinRM is enabled and running
- [ ] Firewall allows WinRM traffic
- [ ] Service account can connect remotely
- [ ] Service account has read-only permissions
- [ ] PowerShell commands work remotely
- [ ] Connection works from N8n host
- [ ] HTTPS is configured (for production)
- [ ] Trusted hosts are configured

## Example Configuration for N8n Config Node

```
# Windows server with WinRM HTTPS
192.168.1.20|n8n_audit|SecurePassword123|auto|5986|winrm

# Windows server with SSH (OpenSSH installed)
192.168.1.21|n8n_audit|SecurePassword123|linux|22|ssh

# Auto-detect (tries WinRM first, then SSH)
192.168.1.22|administrator|AdminPass123|auto|auto|auto
```

## Additional Resources

- [WinRM Configuration Reference](https://docs.microsoft.com/en-us/windows/win32/winrm/remote-management-with-winrm)
- [PowerShell Remoting](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/enable-psremoting)
- [Windows Remote Management](https://docs.microsoft.com/en-us/windows/win32/winrm/portal)

## Need Help?

1. Check N8n execution logs for detailed error messages
2. Verify WinRM listener status: `winrm get winrm/config/listener`
3. Test from N8n host using `Test-WSMan` (PowerShell) or `curl` (Linux)
4. Review Windows Event Logs: `Get-WinEvent -LogName Microsoft-Windows-WinRM/Operational`