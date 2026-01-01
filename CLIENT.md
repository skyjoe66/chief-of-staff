# cos-client

Markdown file templates and CLAUDE.md memory scaffold for Windows Claude Code integration.

**Status:** Stable

## What This Is

This repository contains:
- **Markdown templates** for the Chief of Staff productivity system
- **CLAUDE.md** - A memory scaffold that gives Claude persistent context about you
- **Utility scripts** for financial analysis and manual sync operations

The client component lives in your OneDrive folder and syncs automatically to both Windows and Linux.

## How It Works

```
┌─────────────────────────────────────┐
│         Your Windows PC             │
│                                     │
│  OneDrive\chief-of-staff\           │
│  ├── INBOX.md                       │
│  ├── PROJECTS.md        ◀──────────────── You edit files here
│  ├── WAITING_FOR.md                 │     (or via Claude Code)
│  ├── DECISIONS.md                   │
│  └── CLAUDE.md                      │
│                                     │
│  Claude Code reads CLAUDE.md        │
│  for persistent context             │
└──────────────┬──────────────────────┘
               │
               │ OneDrive automatic sync
               ▼
┌─────────────────────────────────────┐
│         OneDrive Cloud              │
│      (Source of Truth)              │
└──────────────┬──────────────────────┘
               │
               │ cos-backend sync service
               ▼
┌─────────────────────────────────────┐
│       Linux Server (dev.local)      │
│                                     │
│  ~/chief-of-staff/                  │
│  (synced copy of your files)        │
└─────────────────────────────────────┘
```

## Prerequisites

- Windows 10/11 with OneDrive installed and signed in
- Claude Code CLI installed
- [cos-backend](https://github.com/<your-username>/cos-backend) set up on Linux (for sync)

## Installation

### 1. Clone to OneDrive

```powershell
cd "$env:USERPROFILE\OneDrive"
git clone https://github.com/<your-username>/cos-client.git chief-of-staff
cd chief-of-staff
```

**Important:** Clone directly into OneDrive so files sync automatically.

### 2. Personalize CLAUDE.md

Open `CLAUDE.md` and replace all placeholders:

```markdown
## Identity & Background

### Who I Am
- **Name:** <YOUR_NAME>
- **Role:** <YOUR_ROLE>
- **Company:** <YOUR_COMPANY>
- **Email:** <YOUR_EMAIL>
- **Location:** <YOUR_LOCATION>
```

This file is your "memory scaffold" - it gives Claude persistent context about you across all conversations.

### 3. Initialize Your Markdown Files

The templates are ready to use. Customize the structure if needed:

- `INBOX.md` - Quick capture for thoughts and tasks
- `PROJECTS.md` - Active projects with status tracking
- `WAITING_FOR.md` - Items delegated to others
- `DECISIONS.md` - Log of decisions for reference

### 4. Set Up MCP Servers

MCP servers give Claude Code access to external services. Install them locally (not in OneDrive) to avoid syncing large `node_modules` folders.

```powershell
# Create local MCP directory (not synced)
mkdir "$env:USERPROFILE\chief-of-staff\mcps"
```

**To configure MCPs**, open Claude Code and ask:
> "Help me set up the google-workspace MCP server"

Or:
> "Help me set up the onenote MCP server"

Claude Code will guide you through installation and OAuth flows.

#### Recommended MCP Servers

| Server | Purpose | Installation |
|--------|---------|--------------|
| `google-workspace` | Gmail, Calendar, Drive, Tasks | Ask Claude Code |
| `google-tasks` | Task management | Ask Claude Code |
| `microsoft-mcp` | OneDrive, Outlook | Ask Claude Code |
| `onenote` | OneNote notebooks | Ask Claude Code |

### 5. Verify Setup

Open Claude Code in the chief-of-staff folder:

```powershell
cd "$env:USERPROFILE\OneDrive\chief-of-staff"
claude
```

Ask Claude:
> "What do you know about me from CLAUDE.md?"

Claude should respond with details from your memory scaffold.

## Usage

### Daily Workflow

1. **Capture** - Dump thoughts into INBOX.md throughout the day
2. **Process** - During review, move items to appropriate files
3. **Work** - Use PROJECTS.md to track active work
4. **Delegate** - Track delegated items in WAITING_FOR.md
5. **Decide** - Log decisions in DECISIONS.md

### Working with Claude Code

Open Claude Code in the chief-of-staff folder:

```powershell
cd "$env:USERPROFILE\OneDrive\chief-of-staff"
claude
```

Claude automatically reads CLAUDE.md and has context about:
- Your identity and background
- Your working style preferences
- Your communication preferences
- What Claude can/cannot do on your behalf
- Your standard operating procedures

### Example Interactions

> "Add 'Review Q1 report' to my inbox"

> "What's the status of my active projects?"

> "Draft an email to [client] about the project timeline"

> "What decisions have I made this month?"

### Accessing Files from Linux

SSH to your Linux server from Windows:

```powershell
ssh dev.local
```

Files are available at `~/chief-of-staff/` with ~30 second sync latency.

## File Structure

```
OneDrive\chief-of-staff\          # Synced via OneDrive
├── INBOX.md                       # Quick capture
├── PROJECTS.md                    # Active projects
├── WAITING_FOR.md                 # Delegated items
├── DECISIONS.md                   # Decision log
├── CLAUDE.md                      # Memory scaffold (personalize this!)
├── scripts/
│   ├── sync.bat                   # Manual sync utility
│   └── ...                        # Other utilities
├── financial/                     # Optional: QIF exports
│   ├── split_qif_by_month.py
│   └── monthly/
├── work-orders/                   # Complex task specs
└── templates/                     # Reusable templates

%USERPROFILE%\chief-of-staff\     # Local only (NOT synced)
└── mcps/                          # MCP servers with node_modules
    └── onenote-mcp/
```

## Customizing CLAUDE.md

The memory scaffold has several sections to personalize:

### Identity Section
Your basic info - name, role, company, contact details.

### Working Style
How you prefer to work - energy patterns, batch processing preferences, weekly rhythm.

### Authority Boundaries
What Claude can do autonomously vs. what requires your approval. Customize based on your comfort level.

### Standard Operating Procedures
Your workflows for common tasks - onboarding clients, writing proposals, weekly reviews.

### Voice & Tone
How Claude should write on your behalf - tone, length, signature style.

## Troubleshooting

### Files not syncing to Linux

1. Check OneDrive is running and synced
2. Verify cos-backend service is running on Linux:
   ```bash
   ssh dev.local "systemctl --user status cos-file-watcher"
   ```
3. Check sync logs:
   ```bash
   ssh dev.local "tail -20 ~/chief-of-staff/logs/watcher.log"
   ```

### Claude doesn't know my context

1. Verify CLAUDE.md exists and has content
2. Make sure you're running Claude Code from the chief-of-staff directory
3. Check that CLAUDE.md is being read:
   ```
   claude
   > "Read CLAUDE.md and tell me what you see"
   ```

### MCP servers not working

1. Check server is registered:
   ```powershell
   claude mcp list
   ```
2. Re-add if needed - ask Claude Code for help
3. Check for authentication issues - may need to re-authenticate

### OneDrive conflicts

If OneDrive shows sync conflicts:
1. Check `.sync_backups/` on Linux for original versions
2. Manually reconcile the conflicted files
3. Delete the conflict copies once resolved

### Git issues in OneDrive

OneDrive and Git can conflict. If you see issues:
1. Add to `.gitignore`: `.git` folder conflicts
2. Consider using Git only for initial clone, then let OneDrive handle sync

## New Windows PC Setup

When setting up on a new Windows machine:

### 1. Sign into OneDrive
Files sync automatically once OneDrive is set up.

### 2. Install Claude Code CLI
Download from Anthropic and add to PATH.

### 3. Install MCP Servers Locally
MCP servers aren't synced. Reinstall on each machine:

```powershell
mkdir "$env:USERPROFILE\chief-of-staff\mcps"
cd "$env:USERPROFILE\chief-of-staff\mcps"

# Then ask Claude Code to help set up each MCP
claude
> "Help me set up the onenote MCP server"
```

### 4. Authenticate MCPs
Each MCP needs authentication on the new machine. Claude Code will guide you through OAuth flows.

---

*See also: [cos-backend](../cos-backend) | [cos-frontend](../cos-frontend)*
