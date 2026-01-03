# Quick Start Guide

Get the Multi-Server Audit Workflow running in 10 minutes!

## Prerequisites Checklist

- [ ] N8n instance running (self-hosted or cloud)
- [ ] At least one Linux or Windows server to audit
- [ ] SSH access to Linux servers
- [ ] WinRM access to Windows servers
- [ ] Telegram account

## Step 1: Setup Telegram Bot (2 minutes)

1. Open Telegram and search for **@BotFather**
2. Send `/newbot`
3. Choose a name (e.g., "ServerAuditBot")
4. Choose a username (e.g., "my_server_audit_bot")
5. **Save the API token** - it looks like `123456789:ABCdefGHIjklMNOpqrsTUVwxyz`

6. Get your Chat ID:
   - Open this URL in your browser: `https://api.telegram.org/bot<YOUR_API_TOKEN>/getUpdates`
   - Send `/start` to your bot in Telegram
   - Refresh the browser page
   - Find `"chat":{"id":123456789}` - that's your Chat ID

## Step 2: Import Workflow (1 minute)

1. Log in to your N8n instance
2. Click **Workflows** → **Import from File**
3. Upload `workflows/multi-server-audit-workflow.json`
4. Click **Import**

## Step 3: Setup Telegram Credentials (1 minute)

1. In N8n, go to **Credentials** → **Add Credential**
2. Search for **Telegram API**
3. Enter your bot token from Step 1
4. Name it "Telegram Bot"
5. Click **Save**

## Step 4: Configure Servers (2 minutes)

1. Open the imported workflow
2. Click on the **Config** node
3. Edit the **servers** field:

```
# Add your Linux server
192.168.1.10|username|password|auto|22|ssh

# Add your Windows server (if you have one)
192.168.1.20|administrator|password|auto|5986|winrm
```

**Replace with your actual server details:**
- `192.168.1.10` → Your server IP
- `username` → SSH/WinRM username
- `password` → SSH/WinRM password
- `auto` → Auto-detect OS (recommended)
- `22` / `5986` → Default port, or `auto`
- `ssh` / `winrm` → Protocol, or `auto`

4. Edit the **telegram_chat_id** field:
   - Replace `YOUR_TELEGRAM_CHAT_ID` with your Chat ID from Step 1

5. Click **Save**

## Step 5: Setup AI Model (2 minutes)

The workflow supports Google Gemini (recommended), OpenAI, or Anthropic.

### Option A: Google Gemini (Recommended - Free Tier Available)

1. Go to [Google AI Studio](https://aistudio.google.com/app/apikey)
2. Sign in with your Google account
3. Click **Create API Key**
4. Copy the API key

In N8n:
1. Go to **Credentials** → **Add Credential**
2. Search for **Google Gemini (PaLM) API**
3. Enter your API key
4. Name it "Google Gemini API"
5. Click **Save**

### Option B: OpenAI

1. Get an API key from [OpenAI Platform](https://platform.openai.com/api-keys)
2. In the workflow, delete the "Gemini Model" node
3. Add "OpenAI Chat Model" node
4. Enter your API key
5. Connect it to "AI Agent Analysis"

## Step 6: Test the Workflow (2 minutes)

1. Click the **Execute Workflow** button (play icon)
2. Watch the execution flow
3. Check the output of each node:
   - **Detect OS & Connect**: Should show connected=true
   - **Linux/Windows Audit**: Should return metrics
   - **Analyze Results**: Should analyze the metrics
   - **Status Router**: Should route to OK or CRITICAL
4. Check your Telegram - you should receive an alert if there are critical issues!

## Step 7: Activate (10 seconds)

1. Click the **Active** toggle in the top-right corner
2. The workflow will now run automatically every 30 minutes

## Troubleshooting Quick Fixes

### Issue: "SSH connection failed"

**Solution:**
```bash
# Test SSH connection from N8n host
ssh username@192.168.1.10

# If using password auth, install sshpass:
sudo apt-get install sshpass  # Ubuntu/Debian
sudo yum install sshpass      # CentOS/RHEL
```

### Issue: "WinRM connection failed"

**Solution:** On your Windows server, run PowerShell as Administrator:
```powershell
winrm quickconfig
winrm set winrm/config/client '@{TrustedHosts="*"}'
```

### Issue: No Telegram alerts

**Solution:**
1. Send `/start` to your bot in Telegram
2. Verify your Chat ID is correct
3. Check N8n execution logs for errors

### Issue: AI analysis returns empty

**Solution:**
1. Verify your API key is valid
2. Check if you've reached the API quota
3. Try a different AI model

## Next Steps

- 📖 Read the full [README.md](README.md) for detailed configuration
- 🔐 Follow [Security Best Practices](README.md#-security-best-practices) for production
- 📊 Customize [Audit Thresholds](README.md#configure-thresholds) for your needs
- 🚀 Consider using [SSH keys instead of passwords](README.md#use-ssh-keys-instead-of-passwords)

## Support

If you need help:
1. Check the [Troubleshooting](README.md#-troubleshooting) section in README.md
2. Review N8n execution logs
3. Verify server connectivity and credentials

---

**Estimated Setup Time:** 10 minutes  
**Difficulty:** Beginner-friendly  
**Prerequisites:** Basic Linux/Windows server knowledge

**Maintained By:** Abubakkar Khan Fazla Rabbi — System Enngineer | Ethical Hacker