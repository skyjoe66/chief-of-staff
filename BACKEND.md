# cos-backend

Bidirectional OneDrive sync service and morning briefing automation for the Chief of Staff system.

**Status:** Stable

## What This Does

- Syncs markdown files between Linux and OneDrive every 30 seconds
- Sends daily morning briefing emails with calendar, tasks, and project summaries
- Provides smart merge logic to prevent sync conflicts
- Runs as a systemd user service

## Prerequisites

### System Requirements
- Ubuntu 22.04+ (or similar Linux distribution)
- Python 3.12+
- systemd with user services enabled

### Accounts & Services
- Microsoft 365 account with OneDrive
- SMTP credentials (Gmail, SendGrid, or any SMTP provider)
- Google account (optional, for calendar/tasks in morning briefing)

### Tools
- Claude Code CLI (for MCP setup assistance)
- Git

## Installation

### 1. Clone the Repository

```bash
cd ~
git clone https://github.com/<your-username>/cos-backend.git chief-of-staff
cd chief-of-staff
```

### 2. Set Up Python Environment

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### 3. Create Environment File

```bash
cp .env.example .env
chmod 600 .env
```

Edit `.env` with your values:

```bash
# SMTP Configuration (required for morning briefing)
MAIL_SERVER=<smtp.your-provider.com>
MAIL_PORT=587
MAIL_USERNAME=<your-smtp-username>
MAIL_PASSWORD=<your-smtp-password>
MAIL_DEFAULT_SENDER=<your-email@example.com>
CONTACT_EMAIL=<your-email@example.com>

# Paths
COS_PATH=/home/<username>/chief-of-staff

# Feature Flags
USE_ONEDRIVE=true
```

### 4. Initialize Markdown Files

```bash
# Create from templates (if not already present)
cp templates/INBOX.template.md INBOX.md
cp templates/PROJECTS.template.md PROJECTS.md
cp templates/WAITING_FOR.template.md WAITING_FOR.md
cp templates/DECISIONS.template.md DECISIONS.md
```

Or clone the [cos-client](https://github.com/<your-username>/cos-client) repo for pre-built templates.

### 5. Set Up OneDrive Authentication

```bash
source venv/bin/activate
python scripts/onedrive_auth.py
```

This launches a device code flow:
1. Open the URL displayed in terminal
2. Enter the code shown
3. Sign in with your Microsoft account
4. Grant the requested permissions

Token is saved to `.onedrive_token.json`.

### 6. Install Systemd Service

```bash
# Copy service file (edit paths first if username differs)
mkdir -p ~/.config/systemd/user
cp systemd/cos-file-watcher.service ~/.config/systemd/user/

# Edit the service file to match your paths
nano ~/.config/systemd/user/cos-file-watcher.service

# Enable and start
systemctl --user daemon-reload
systemctl --user enable cos-file-watcher
systemctl --user start cos-file-watcher

# Enable lingering (keeps service running after logout)
loginctl enable-linger $USER
```

### 7. Set Up Morning Briefing Cron

```bash
crontab -e
```

Add:
```
0 9 * * * /home/<username>/chief-of-staff/scripts/run_briefing.sh
```

### 8. Set Up MCP Servers (Optional)

For Claude Code integration on this server:

```bash
claude
```

Then ask Claude Code:
> "Help me set up the google-workspace MCP server for Gmail, Calendar, Drive, and Tasks"

Follow the interactive prompts. Claude Code will guide you through OAuth setup.

## Configuration

### Environment Variables

See `.env.example` for all available options.

### Sync Behavior

| Setting | Default | Description |
|---------|---------|-------------|
| Interval | 30 seconds | How often sync runs |
| Files | `*.md` in root | Which files sync |
| Backups | `.sync_backups/` | Pre-merge backup location |

### Smart Merge Logic

When both local and OneDrive have changes to the same file:

| File | Strategy |
|------|----------|
| `INBOX.md` | Union of all items (nothing lost) |
| `PROJECTS.md` | Keeps the longer/more complete version |
| `WAITING_FOR.md` | Union of table rows, deduplicates |
| `DECISIONS.md` | Preserves all entries |

## Usage

### Check Service Status

```bash
systemctl --user status cos-file-watcher
```

### View Logs

```bash
# Real-time sync logs
tail -f ~/chief-of-staff/logs/watcher.log

# Morning briefing logs
tail -f ~/chief-of-staff/logs/briefing.log
```

### Manual Sync Commands

```bash
cd ~/chief-of-staff
source venv/bin/activate

# Bidirectional sync
python scripts/bidirectional_sync.py

# Force download from OneDrive
python scripts/bidirectional_sync.py --download

# Force upload to OneDrive
python scripts/bidirectional_sync.py --upload
```

### Preview Morning Briefing

```bash
cd ~/chief-of-staff
source venv/bin/activate
set -a && source .env && set +a
python scripts/morning_briefing.py --preview
```

Opens `briefing_preview.html` in browser.

### Send Briefing Manually

```bash
cd ~/chief-of-staff
./scripts/run_briefing.sh
```

## File Structure

```
chief-of-staff/
├── INBOX.md                    # Quick capture (synced)
├── PROJECTS.md                 # Active projects (synced)
├── WAITING_FOR.md              # Delegated items (synced)
├── DECISIONS.md                # Decision log (synced)
├── CLAUDE.md                   # Memory scaffold (synced)
├── .env                        # Configuration (DO NOT COMMIT)
├── .env.example                # Template for .env
├── .onedrive_token.json        # OAuth token (DO NOT COMMIT)
├── .sync_state.json            # Sync tracking
├── .sync_backups/              # Pre-merge backups
├── requirements.txt            # Python dependencies
├── logs/
│   ├── watcher.log
│   ├── sync.log
│   └── briefing.log
├── scripts/
│   ├── bidirectional_sync.py
│   ├── watch_and_sync.py
│   ├── morning_briefing.py
│   ├── run_briefing.sh
│   └── onedrive_auth.py
├── systemd/
│   └── cos-file-watcher.service
├── templates/
│   └── *.template.md
└── mcp/                        # MCP servers (if configured)
```

## Troubleshooting

### Service won't start

```bash
# Check for errors
journalctl --user -u cos-file-watcher -n 50

# Verify Python works
~/chief-of-staff/venv/bin/python --version

# Test script directly
cd ~/chief-of-staff
source venv/bin/activate
python scripts/watch_and_sync.py --interval 30
```

### OneDrive token expired

```bash
cd ~/chief-of-staff
source venv/bin/activate
python scripts/onedrive_auth.py
# Complete device code flow again
```

### Sync conflicts

1. Check `.sync_backups/` for original versions
2. Manually review and reconcile
3. The merge logic errs toward preserving all content

### Email not sending

```bash
# Test SMTP connection
cd ~/chief-of-staff
source venv/bin/activate
python -c "
import smtplib
from dotenv import load_dotenv
import os
load_dotenv()
server = smtplib.SMTP(os.getenv('MAIL_SERVER'), int(os.getenv('MAIL_PORT')))
server.starttls()
server.login(os.getenv('MAIL_USERNAME'), os.getenv('MAIL_PASSWORD'))
print('SMTP OK')
server.quit()
"
```

### Permission issues

```bash
chmod 600 .env .onedrive_token.json
chmod +x scripts/*.sh
```

## Integration

### With Frontend

The frontend reads from the same directory. Set `COS_DIR` in the frontend's `.env` to point here:

```
COS_DIR=/home/<username>/chief-of-staff
```

### With Windows Client

OneDrive handles sync to Windows automatically:

```
Windows (OneDrive native) ←→ OneDrive Cloud ←→ Linux (this backend)
```

Files edited on Windows appear here within ~30 seconds (OneDrive sync + backend sync interval).

---

*Next: [cos-frontend](../cos-frontend) or [cos-client](../cos-client)*
