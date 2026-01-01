# cos-frontend

Web interface for viewing and editing Chief of Staff markdown files with integrated Claude chat.

**Status:** ⚠️ Early Draft - Basic functionality works, but expect rough edges and limited features.

## What This Does

- View rendered markdown files in a dark-themed web UI
- Edit files directly in the browser
- Chat with Claude about your files (Claude sees all file contents as context)
- Keyboard shortcuts for efficient workflow

## Prerequisites

### System Requirements
- Linux server (same machine as cos-backend recommended)
- Python 3.12+
- uv (recommended) or pip

### Dependencies
- [cos-backend](https://github.com/<your-username>/cos-backend) installed and running
- Claude Code CLI installed and authenticated

### Claude CLI Authentication

The frontend's chat feature calls Claude via the CLI (`claude -p`). This requires proper authentication on the server.

#### Option 1: Claude Pro/Team Subscription (Recommended)

If you have a Claude Pro or Team subscription at claude.ai:

```bash
# On your Linux server
claude login
```

This opens a browser for OAuth. After authenticating, the CLI uses your subscription - no API costs.

**Note:** If running headless (no browser), use:
```bash
claude login --method device-code
```

#### Option 2: API Key

If you prefer to use the Anthropic API directly:

```bash
# Set your API key
export ANTHROPIC_API_KEY=sk-ant-...

# Or add to your shell profile
echo 'export ANTHROPIC_API_KEY=sk-ant-...' >> ~/.bashrc
```

**Warning:** API usage incurs costs. The chat feature sends all four markdown files as context with every message, which can add up.

#### Verify Authentication

```bash
# Test that Claude CLI works
claude -p "Say OK if you can read this"
```

If this returns a response, the frontend chat will work.

## Installation

### 1. Clone the Repository

```bash
cd ~
git clone https://github.com/<your-username>/cos-frontend.git
cd cos-frontend
```

### 2. Set Up Python Environment

With uv (recommended):
```bash
uv venv
source .venv/bin/activate
uv sync
```

With pip:
```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### 3. Create Environment File

```bash
cp .env.example .env
chmod 600 .env
```

Edit `.env`:

```bash
# Generate a secret key
SECRET_KEY=$(python -c "import secrets; print(secrets.token_hex(32))")

# Path to your markdown files (must match backend location)
COS_DIR=/home/<username>/chief-of-staff
```

### 4. Verify Configuration

```bash
# Check that COS_DIR points to valid files
ls -la $(grep COS_DIR .env | cut -d'=' -f2)/*.md
```

You should see INBOX.md, PROJECTS.md, WAITING_FOR.md, DECISIONS.md.

## Usage

### Start the Server

Development mode:
```bash
cd ~/cos-frontend
source .venv/bin/activate
python app.py
```

With uv:
```bash
cd ~/cos-frontend
uv run python app.py
```

Server runs at `http://0.0.0.0:5006`

### Access the Interface

From any machine on your local network:
- `http://dev.local:5006`
- `http://<server-ip>:5006`

### Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl+S` | Save file (in edit mode) |
| `Escape` | Cancel editing |
| `Ctrl+Enter` | Send chat message |

### File Navigation

Click any file in the sidebar to view it:
- **Inbox** → INBOX.md
- **Projects** → PROJECTS.md
- **Waiting For** → WAITING_FOR.md
- **Decisions** → DECISIONS.md

### Editing

1. Click **Edit** button
2. Modify content in the text editor
3. Press **Ctrl+S** or click **Save**
4. Changes are written immediately to disk
5. Backend sync picks them up within 30 seconds

### Claude Chat

1. Type your question in the chat box
2. Click **Send** or press **Ctrl+Enter**
3. Claude receives all four markdown files as context
4. Response appears below the input

**Timeout:** 120 seconds. Complex questions may take a while.

## Running as a Service (Optional)

For persistent operation:

```bash
mkdir -p ~/.config/systemd/user
cp systemd/cos-frontend.service ~/.config/systemd/user/

# Edit paths in the service file
nano ~/.config/systemd/user/cos-frontend.service

systemctl --user daemon-reload
systemctl --user enable cos-frontend
systemctl --user start cos-frontend
```

## Configuration

### Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `SECRET_KEY` | Yes | Flask session secret (generate randomly) |
| `COS_DIR` | Yes | Path to markdown files directory |

### Port Configuration

Default port is 5006. To change, edit `app.py` or set:
```bash
export FLASK_RUN_PORT=8080
```

## File Structure

```
cos-frontend/
├── app.py                  # Flask application
├── main.py                 # Entry point
├── pyproject.toml          # Dependencies (uv/pip)
├── requirements.txt        # Dependencies (pip fallback)
├── .env                    # Configuration (DO NOT COMMIT)
├── .env.example            # Template
├── .venv/                  # Python environment
├── static/
│   ├── css/
│   │   └── style.css       # Dark theme
│   └── js/
│       └── app.js          # Frontend logic
├── templates/
│   ├── base.html
│   └── index.html
└── systemd/
    └── cos-frontend.service
```

## Limitations (Early Draft)

This is an early draft with known limitations:

- **No authentication** - Anyone on your network can access/edit files
- **No HTTPS** - Traffic is unencrypted (fine for local network)
- **No real-time sync** - Must refresh to see changes made elsewhere
- **Basic error handling** - Errors may not display clearly
- **Single user** - No multi-user support
- **No mobile optimization** - Desktop-focused design

## Troubleshooting

### Page won't load

```bash
# Check if server is running
curl http://localhost:5006

# Check for Python errors
cd ~/cos-frontend
source .venv/bin/activate
python app.py
```

### Files not found / 404 errors

```bash
# Verify COS_DIR is correct
source .env
ls -la $COS_DIR/*.md

# Should list all four markdown files
```

### Claude chat returns error

```bash
# Test Claude CLI directly
claude -p "Say OK"

# If this fails, re-authenticate
claude login
```

### Chat timeout

The default timeout is 120 seconds. If Claude is taking too long:
- Simplify your question
- Check if Claude CLI works directly in terminal
- Your files might be very large (context limit)

### "Command not found: claude"

Claude CLI isn't in PATH. Either:
```bash
# Add to PATH
export PATH="$PATH:/path/to/claude"

# Or specify full path in app.py
```

## Integration

### With Backend

Both services read/write the same files:

```
cos-backend syncs:  OneDrive ←→ /home/<user>/chief-of-staff/
cos-frontend reads: /home/<user>/chief-of-staff/ (via COS_DIR)
```

Changes made in the frontend are immediately available to the sync service.

### Network Access

The frontend binds to `0.0.0.0:5006`, making it accessible from:
- `localhost:5006` (on the server)
- `<server-ip>:5006` (from other machines)
- `dev.local:5006` (if you have DNS/hosts configured)

**Security note:** No authentication means anyone on your network can access it. Only run on trusted networks.

---

*See also: [cos-backend](../cos-backend) | [cos-client](../cos-client)*
