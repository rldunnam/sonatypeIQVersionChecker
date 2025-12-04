# Sonatype IQ Server Version Checker

Automatically monitor the Sonatype IQ Server release notes page for new versions and send instant notifications via Email, Slack, and/or Microsoft Teams when a new version is detected.

## Features

- 🔔 **Multi-Channel Notifications** - Email, Slack, and Microsoft Teams support
- 🔄 **Automatic Version Tracking** - Remembers last version and only alerts on changes
- 🛡️ **Retry Logic** - Automatic retry with exponential backoff for network failures
- 🧪 **Dry Run Mode** - Test configuration without sending notifications
- 🧰 **Force Notify** - Test notifications on demand without version changes
- ⚙️ **Flexible Configuration** - Environment variables, .env file, or command-line arguments
- 📊 **Detailed Logging** - Comprehensive logging with verbose mode for troubleshooting
- 🚀 **Zero Dependencies** - Only requires Python 3.7+ and the `requests` library

## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Notification Setup](#notification-setup)
- [Scheduling](#scheduling)
- [Troubleshooting](#troubleshooting)
- [Examples](#examples)

## Prerequisites

- Python 3.7 or higher
- `requests` library
- (Optional) `python-dotenv` for .env file support

### Install Dependencies

```bash
pip install requests

# Optional: For .env file support
pip install python-dotenv
```

## Installation

1. **Download the script:**
   ```bash
   curl -O https://your-repo/sonatype_version_checker.py
   chmod +x sonatype_version_checker.py
   ```

2. **Create configuration file:**
   ```bash
   cp .env.example .env
   # Edit .env with your settings
   ```

3. **Secure the configuration file:**
   ```bash
   chmod 600 .env
   ```

4. **Test the setup:**
   ```bash
   ./sonatype_version_checker.py --dry-run --enable-email --enable-slack --enable-teams
   ```

## Configuration

### Configuration Priority

The script uses configuration in this priority order (highest to lowest):

1. **Command-line arguments** (e.g., `--enable-email`)
2. **Environment variables** (e.g., `export ENABLE_EMAIL=true`)
3. **.env file** (loads automatically if present)
4. **Default values**

### Quick Start Configurations

#### Email Only
```bash
# .env file
ENABLE_EMAIL=true
EMAIL_USERNAME=monitor@example.com
EMAIL_PASSWORD=your_password
EMAIL_FROM=monitor@example.com
EMAIL_TO=team@example.com
```

#### Slack Only
```bash
# .env file
ENABLE_SLACK=true
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/YOUR/WEBHOOK/URL
```

#### Microsoft Teams Only
```bash
# .env file
ENABLE_TEAMS=true
TEAMS_WEBHOOK_URL=https://your-webhook-url
```

#### All Channels
```bash
# .env file
ENABLE_EMAIL=true
ENABLE_SLACK=true
ENABLE_TEAMS=true

# Email settings
EMAIL_USERNAME=monitor@example.com
EMAIL_PASSWORD=your_password
EMAIL_FROM=monitor@example.com
EMAIL_TO=team@example.com

# Slack settings
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/YOUR/WEBHOOK/URL

# Teams settings
TEAMS_WEBHOOK_URL=https://your-webhook-url
```

### Configuration Parameters

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `ENABLE_EMAIL` | No | `false` | Enable email notifications |
| `ENABLE_SLACK` | No | `false` | Enable Slack notifications |
| `ENABLE_TEAMS` | No | `false` | Enable Microsoft Teams notifications |
| `SMTP_SERVER` | If email enabled | `smtp.office365.com` | SMTP server hostname |
| `SMTP_PORT` | If email enabled | `587` | SMTP server port |
| `EMAIL_USERNAME` | If email enabled | - | Email account username |
| `EMAIL_PASSWORD` | If email enabled | - | Email account password |
| `EMAIL_FROM` | If email enabled | - | From email address |
| `EMAIL_TO` | If email enabled | - | To email address(es) |
| `SLACK_WEBHOOK_URL` | If Slack enabled | - | Slack incoming webhook URL |
| `TEAMS_WEBHOOK_URL` | If Teams enabled | - | Teams webhook or Power Automate URL |
| `VERSION_FILE` | No | `sonatype_last_version.txt` | File to store version history |
| `DRY_RUN` | No | `false` | Test mode without sending notifications |

## Usage

### Command-Line Options

```
Usage: sonatype_version_checker.py [OPTIONS]

Notification Control:
  --enable-email          Enable email notifications
  --enable-slack          Enable Slack notifications
  --enable-teams          Enable Microsoft Teams notifications
  --disable-email         Explicitly disable email
  --disable-slack         Explicitly disable Slack
  --disable-teams         Explicitly disable Teams

Email Configuration:
  --smtp-server HOST      SMTP server hostname
  --smtp-port PORT        SMTP server port
  --email-username USER   Email account username
  --email-password PASS   Email account password
  --email-from ADDR       From email address
  --email-to ADDR         To email address

Slack Configuration:
  --slack-webhook URL     Slack webhook URL

Teams Configuration:
  --teams-webhook URL     Teams webhook URL

General Options:
  --version-file PATH     Version tracking file path
  --dry-run               Test without sending notifications
  --force-notify          Force notification even if version unchanged
  --verbose               Enable verbose logging
  --help                  Show help message
```

### Basic Usage Examples

```bash
# Enable email notifications (using .env for credentials)
./sonatype_version_checker.py --enable-email

# Enable all notification channels
./sonatype_version_checker.py --enable-email --enable-slack --enable-teams

# Test configuration without sending notifications
./sonatype_version_checker.py --dry-run --enable-email --enable-teams

# Force a test notification (useful for testing)
./sonatype_version_checker.py --enable-teams --force-notify

# Override email recipient via command line
./sonatype_version_checker.py --enable-email --email-to admin@example.com

# Verbose mode for troubleshooting
./sonatype_version_checker.py --enable-email --verbose
```

## Notification Setup

### Email Setup

#### Microsoft 365 / Outlook.com

1. Use your Microsoft 365 credentials
2. If using MFA, create an **App Password**:
   - Go to [Microsoft Account Security](https://account.microsoft.com/security)
   - Security → More security options → App passwords
   - Create app password for "Sonatype Monitor"
   - Use this password in `EMAIL_PASSWORD`

**Configuration:**
```bash
SMTP_SERVER=smtp.office365.com
SMTP_PORT=587
EMAIL_USERNAME=your-email@example.com
EMAIL_PASSWORD=your-app-password
```

#### Gmail

1. Enable 2-Step Verification on your Google Account
2. Generate an **App Password**:
   - Go to [Google Account](https://myaccount.google.com/)
   - Security → 2-Step Verification → App passwords
   - Select "Mail" and "Other (Custom name)"
   - Enter "Sonatype Monitor"
   - Copy the 16-character password

**Configuration:**
```bash
SMTP_SERVER=smtp.gmail.com
SMTP_PORT=587
EMAIL_USERNAME=your-email@gmail.com
EMAIL_PASSWORD=your-16-char-app-password
```

### Slack Setup

1. **Create Incoming Webhook:**
   - Go to [Slack API Apps](https://api.slack.com/apps)
   - Click "Create New App" → "From scratch"
   - Name it "Sonatype Monitor" and select your workspace
   - Click "Incoming Webhooks" → Enable
   - Click "Add New Webhook to Workspace"
   - Select the channel for notifications
   - Copy the webhook URL

2. **Configure:**
   ```bash
   SLACK_WEBHOOK_URL=https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXX
   ```

### Microsoft Teams Setup

#### Power Automate Flow (Recommended)

1. **Create Flow:**
   - Go to [Power Automate](https://make.powerautomate.com/)
   - Create → Automated cloud flow
   - Skip trigger selection

2. **Add HTTP Trigger:**
   - Search for "When a HTTP request is received"
   - Add this trigger
   - **Who can trigger:** Select **"Anyone"**
   - **Schema** (optional but recommended):
   ```json
   {
       "type": "object",
       "properties": {
           "title": {"type": "string"},
           "version": {"type": "string"},
           "status": {"type": "string"},
           "description": {"type": "string"},
           "url": {"type": "string"},
           "emoji": {"type": "string"}
       }
   }
   ```

3. **Add Teams Action:**
   - Add action → "Post message in a chat or channel"
   - Post as: Flow bot
   - Team: [Select your team]
   - Channel: [Select your channel]
   - Message:
   ```
   🔒 **New Sonatype IQ Server Version Available**
   
   **Version:** @{triggerBody()?['version']}
   **Status:** @{triggerBody()?['status']}
   
   @{triggerBody()?['description']}
   
   [View Release Notes](@{triggerBody()?['url']})
   ```

4. **Save and Copy URL:**
   - Save the flow
   - Copy the HTTP POST URL from the trigger
   - Use as `TEAMS_WEBHOOK_URL`

## Scheduling

### Cron (Linux/macOS)

Check for updates every 6 hours:

```bash
# Edit crontab
crontab -e

# Add this line (runs at 00:00, 06:00, 12:00, 18:00)
0 */6 * * * cd /path/to/script && /usr/bin/python3 sonatype_version_checker.py >> /var/log/sonatype-checker.log 2>&1
```

Check daily at 9 AM:
```bash
0 9 * * * cd /path/to/script && /usr/bin/python3 sonatype_version_checker.py
```

### Systemd Timer (Linux)

**Create `/etc/systemd/system/sonatype-checker.service`:**
```ini
[Unit]
Description=Sonatype IQ Server Version Checker
After=network-online.target

[Service]
Type=oneshot
User=sonatype-checker
WorkingDirectory=/opt/sonatype-checker
EnvironmentFile=/opt/sonatype-checker/.env
ExecStart=/usr/bin/python3 /opt/sonatype-checker/sonatype_version_checker.py
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

**Create `/etc/systemd/system/sonatype-checker.timer`:**
```ini
[Unit]
Description=Run Sonatype Version Checker every 6 hours

[Timer]
OnBootSec=10min
OnUnitActiveSec=6h
Persistent=true

[Install]
WantedBy=timers.target
```

**Enable and start:**
```bash
sudo systemctl daemon-reload
sudo systemctl enable sonatype-checker.timer
sudo systemctl start sonatype-checker.timer

# Check status
sudo systemctl status sonatype-checker.timer
```

### Docker

**Dockerfile:**
```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY sonatype_version_checker.py .
RUN pip install --no-cache-dir requests

CMD ["python", "sonatype_version_checker.py"]
```

**docker-compose.yml:**
```yaml
version: '3.8'
services:
  sonatype-checker:
    build: .
    environment:
      - ENABLE_TEAMS=true
      - TEAMS_WEBHOOK_URL=${TEAMS_WEBHOOK_URL}
    volumes:
      - ./data:/app/data
    restart: unless-stopped
```

## Troubleshooting

### Common Issues

#### 1. Version Pattern Not Found

**Error:** `No version found in page content`

**Cause:** Sonatype website HTML structure changed

**Solutions:**
- Run with `--verbose` to see page content preview
- Check if URL is still correct: https://help.sonatype.com/en/iq-server-release-notes.html
- The pattern looks for 3-digit version numbers (e.g., "170", "171", "172")
- Update `VERSION_PATTERN` regex in the script if needed:
  ```python
  VERSION_PATTERN = r"\b\d{3}\b"  # Current pattern
  ```

#### 2. Authentication Failed (Email)

**Error:** `Email authentication failed`

**Solutions:**
- Verify credentials are correct
- For Microsoft 365/Gmail with MFA: Use an **App Password**
- Check `SMTP_SERVER` and `SMTP_PORT` are correct

#### 3. Teams Webhook Returns 401

**Error:** `Teams API returned status 401`

**Solution:**
1. Open your Power Automate flow
2. Click "When a HTTP request is received" trigger
3. Change "Who can trigger the flow" to **"Anyone"**
4. Save and copy the NEW webhook URL
5. Update your `.env` file

#### 4. No Notifications on First Run

**This is expected!** The first run records the current version but doesn't send notifications.

**To test:**
```bash
# Use force-notify flag
./sonatype_version_checker.py --enable-teams --force-notify

# Or change the version file
echo "169" > sonatype_last_version.txt
./sonatype_version_checker.py --enable-teams
```

### Debug Mode

Enable verbose logging:

```bash
./sonatype_version_checker.py --verbose --enable-teams --force-notify
```

## Examples

### Example 1: First-Time Setup

```bash
# Create .env file
cat > .env << 'EOF'
ENABLE_TEAMS=true
TEAMS_WEBHOOK_URL=https://prod-00.eastus.logic.azure.com:443/workflows/abc123...
EOF

# Secure it
chmod 600 .env

# Test
./sonatype_version_checker.py --dry-run

# Test notification
./sonatype_version_checker.py --force-notify

# Run
./sonatype_version_checker.py
```

### Example 2: All Notification Channels

```bash
# .env
cat > .env << 'EOF'
ENABLE_EMAIL=true
ENABLE_SLACK=true
ENABLE_TEAMS=true

EMAIL_USERNAME=monitor@example.com
EMAIL_PASSWORD=password123
EMAIL_FROM=monitor@example.com
EMAIL_TO=team@example.com

SLACK_WEBHOOK_URL=https://hooks.slack.com/services/YOUR/WEBHOOK/URL
TEAMS_WEBHOOK_URL=https://your-flow-url
EOF

./sonatype_version_checker.py
```

### Example 3: Monitoring Script

```bash
#!/bin/bash
# sonatype-monitor.sh

LOG_FILE="/var/log/sonatype-checker.log"
ERROR_FILE="/var/log/sonatype-checker-error.log"

echo "[$(date)] Starting Sonatype version check" >> "$LOG_FILE"

if /usr/local/bin/sonatype_version_checker.py >> "$LOG_FILE" 2>> "$ERROR_FILE"; then
    echo "[$(date)] Check completed successfully" >> "$LOG_FILE"
else
    EXIT_CODE=$?
    echo "[$(date)] Check failed with exit code $EXIT_CODE" >> "$ERROR_FILE"
    
    # Send alert
    if command -v mail &> /dev/null; then
        mail -s "Sonatype Checker Failed" admin@example.com < "$ERROR_FILE"
    fi
fi

# Rotate logs (keep last 30 days)
find /var/log -name "sonatype-checker*.log" -mtime +30 -delete
```

## Version Pattern Notes

The script looks for **3-digit version numbers** in the HTML (e.g., "170", "171", "172"). It then takes the highest number found as the current version.

**Current Pattern:**
```python
VERSION_PATTERN = r"\b\d{3}\b"
```

This matches:
- ✅ `170` - matches
- ✅ `171` - matches
- ✅ `172` - matches
- ❌ `17` - too short
- ❌ `1702` - too long

**If Sonatype changes their versioning scheme**, you may need to update the pattern. Alternative patterns:

```python
# For 4-digit versions (e.g., 1702, 1703)
VERSION_PATTERN = r"\b\d{4}\b"

# For versions with dots (e.g., 170.0, 171.0)
VERSION_PATTERN = r"\b\d{3}\.\d+\b"

# For "Version XXX" format
VERSION_PATTERN = r"Version\s+(\d{3})"
```

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success - No new version or notifications sent successfully |
| 1 | Error - Configuration error, network failure, or notification failure |

## Security Best Practices

1. **Protect credentials:**
   ```bash
   chmod 600 .env
   chown sonatype-checker:sonatype-checker .env
   ```

2. **Never commit .env to version control:**
   ```bash
   echo ".env" >> .gitignore
   echo "sonatype_last_version.txt" >> .gitignore
   ```

3. **Use app-specific passwords** for email accounts with MFA

4. **Rotate credentials regularly** (every 90 days recommended)

5. **Use dedicated service accounts** with minimal permissions

6. **For production:**
   - Use secret management systems (AWS Secrets Manager, Azure Key Vault, etc.)
   - Run as non-privileged user
   - Implement log rotation
   - Monitor script execution
   - Set up alerting for script failures

## Power Automate Payload Schema

For the Teams Power Automate integration, use this JSON schema:

```json
{
    "type": "object",
    "properties": {
        "title": {"type": "string"},
        "version": {"type": "string"},
        "status": {"type": "string"},
        "description": {"type": "string"},
        "url": {"type": "string"},
        "emoji": {"type": "string"}
    }
}
```

**Example Payload:**
```json
{
    "title": "New Sonatype IQ Server Version Available",
    "version": "172",
    "status": "New Release",
    "description": "A new version of Sonatype IQ Server has been released: 172",
    "url": "https://help.sonatype.com/en/iq-server-release-notes.html",
    "emoji": "🔒"
}
```

## Contributing

Contributions are welcome! Please:
1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## License

[Your License Here]

## Support

- **Issues:** [GitHub Issues](https://github.com/your-org/sonatype-checker/issues)
- **Documentation:** [Wiki](https://github.com/your-org/sonatype-checker/wiki)
- **Email:** support@your-org.com

## Changelog

### Version 2.0.0 (Current)
- ✨ Added Microsoft Teams support (Power Automate + Traditional webhooks)
- ✨ Multi-channel notification support
- ✨ Added `--force-notify` flag for testing
- 🔄 Improved retry logic with exponential backoff
- 📝 Better error messages and validation
- 🧪 Enhanced dry-run mode
- 📊 Verbose logging mode with detailed debugging
- 🐛 Improved error handling for all notification channels

### Version 1.0.0
- Initial release
- Email and Slack support
- Basic version tracking

---

**Last Updated:** December 2024