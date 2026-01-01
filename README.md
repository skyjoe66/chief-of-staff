# Chief of Staff - Personal AI Productivity System

A multi-component personal productivity system that combines markdown-based task management with AI assistance. Syncs between Windows and Linux via OneDrive, provides a web UI for file management, and integrates Claude for intelligent assistance.

## Architecture

```
┌────────────────────────┐      ┌─────────────────────┐      ┌────────────────────────┐
│    Windows Client      │◀────▶│      OneDrive       │◀────▶│   Linux Server         │
│    (Interactive)       │      │  (Source of Truth)  │      │   (dev.local)          │
│                        │      │                     │      │                        │
│  • Claude Code + MCPs  │      │  • INBOX.md         │      │  ┌──────────────────┐  │
│  • Direct file editing │      │  • PROJECTS.md      │      │  │    Frontend      │  │
│  • OneDrive native     │      │  • WAITING_FOR.md   │      │  │  Flask UI :5006  │  │
│    sync                │      │  • DECISIONS.md     │      │  └──────────────────┘  │
│                        │      │  • CLAUDE.md        │      │                        │
└────────────────────────┘      └─────────────────────┘      │  ┌──────────────────┐  │
                                                             │  │    Backend       │  │
                                                             │  │  Sync + Cron     │  │
                                                             │  └──────────────────┘  │
                                                             └────────────────────────┘
```

## Repositories

| Repo | Purpose | Status |
|------|---------|--------|
| **[cos-backend](https://github.com/<your-username>/cos-backend)** | Bidirectional OneDrive sync, morning briefing, background automation | Stable |
| **[cos-frontend](https://github.com/<your-username>/cos-frontend)** | Web UI for viewing/editing markdown files with Claude chat | ⚠️ Early Draft |
| **[cos-client](https://github.com/<your-username>/cos-client)** | Markdown templates and CLAUDE.md memory scaffold | Stable |

## Quick Start

### Prerequisites

- Linux server (Ubuntu 22.04+ recommended) with Python 3.12+
- Windows PC with OneDrive installed
- Microsoft 365 account (for OneDrive sync)
- Claude Code CLI installed on both machines
- SMTP credentials for morning briefings (optional)

### Installation Order

1. **[cos-backend](docs/BACKEND.md)** - Set up sync service and morning briefing on Linux
2. **[cos-frontend](docs/FRONTEND.md)** - Deploy web UI on Linux (optional)
3. **[cos-client](docs/CLIENT.md)** - Configure Windows with templates and CLAUDE.md

## Core Files

These markdown files are the heart of the system:

| File | Purpose |
|------|---------|
| `INBOX.md` | Quick capture - dump thoughts here for later processing |
| `PROJECTS.md` | Active projects with next actions and status |
| `WAITING_FOR.md` | Delegated items you're waiting on |
| `DECISIONS.md` | Decision log for reference |
| `CLAUDE.md` | Memory scaffold - gives Claude persistent context about you |

## Key Features

### Bidirectional Sync
- Runs every 30 seconds via systemd service
- Smart merge logic prevents conflicts
- Backups created before any merge operation

### Morning Briefing
- Daily email at 9 AM (configurable)
- Pulls from calendar, tasks, and all markdown files
- Sent from Linux server via SMTP

### Web Interface
- View and edit markdown files in browser
- Dark theme, keyboard shortcuts (Ctrl+S, Escape)
- Integrated Claude chat with full file context

### Claude Integration
- CLAUDE.md provides persistent context across conversations
- MCP servers connect Claude to Google Workspace, OneDrive, OneNote
- Works with Claude Code on both Windows and Linux

## MCP Servers

This system uses several MCP (Model Context Protocol) servers. Setup varies by server - **use Claude Code to get current installation instructions**.

| Server | Purpose | Environments |
|--------|---------|--------------|
| `google-workspace` | Gmail, Calendar, Drive, Tasks | Linux, Windows |
| `google-tasks` | Task management | Linux, Windows |
| `microsoft-mcp` | OneDrive, Outlook | Linux, Windows |
| `onenote` | OneNote notebooks | Windows only |

**To configure MCPs:** Open Claude Code and ask:
> "Help me set up the google-workspace MCP server for Gmail, Calendar, Drive, and Tasks"

Claude Code will provide current installation steps and handle OAuth flows interactively.

## Configuration Reference

### Backend Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `MAIL_SERVER` | Yes | SMTP server hostname |
| `MAIL_PORT` | Yes | SMTP port (usually 587) |
| `MAIL_USERNAME` | Yes | SMTP username |
| `MAIL_PASSWORD` | Yes | SMTP password |
| `MAIL_DEFAULT_SENDER` | Yes | From address for emails |
| `CONTACT_EMAIL` | Yes | Recipient for briefing |
| `COS_PATH` | Yes | Path to chief-of-staff directory |
| `USE_ONEDRIVE` | No | Enable OneDrive sync (default: true) |

### Frontend Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `SECRET_KEY` | Yes | Flask session secret key |
| `COS_DIR` | Yes | Path to markdown files directory |

## Security Notes

**Never commit these files:**
- `.env` (contains SMTP credentials)
- `*token*.json` (OAuth tokens)
- `*.local.json` (local settings)
- `.mcp.json` (may contain OAuth secrets)

## Troubleshooting

See component-specific READMEs for detailed troubleshooting:
- [Backend Troubleshooting](docs/BACKEND.md#troubleshooting)
- [Frontend Troubleshooting](docs/FRONTEND.md#troubleshooting)
- [Client Troubleshooting](docs/CLIENT.md#troubleshooting)

## License

MIT License - See LICENSE file in each repository.

---

*For detailed setup instructions, see the component-specific documentation linked above.*
