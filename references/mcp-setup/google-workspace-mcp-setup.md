# Google Workspace MCP Setup for Claude Code

**Purpose:** Connect Gmail, Google Calendar, and Google Chat to Claude Code so you can read emails, check your schedule, and search Chat messages directly from your terminal.

**Package:** `workspace-mcp` (PyPI)

## Prerequisites

- Claude Code installed and working
- A Google Workspace account
- Python 3.10+ installed (`python3 --version` to check)
- `uv` installed (`brew install uv` if not already)

## Step 1: Get the OAuth Credentials

You need a Google Cloud OAuth app with the following APIs enabled:
- Gmail API
- Google Calendar API
- Google Chat API

You'll need two values:
- `GOOGLE_OAUTH_CLIENT_ID` — looks like `1039881044029-xxxxx.apps.googleusercontent.com`
- `GOOGLE_OAUTH_CLIENT_SECRET` — looks like `GOCSPX-xxxxx`

> If your organization already has a shared OAuth app, ask your admin for the Client ID and Client Secret. Otherwise, see "Creating Your Own OAuth App" at the bottom of this doc.

## Step 2: Add the MCP Server to Claude Code

Using the CLI:

```bash
claude mcp add --scope user google-workspace \
  -e GOOGLE_OAUTH_CLIENT_ID="PASTE_CLIENT_ID_HERE" \
  -e GOOGLE_OAUTH_CLIENT_SECRET="PASTE_CLIENT_SECRET_HERE" \
  -- uvx workspace-mcp --tools gmail calendar chat
```

Or manually add to `~/.claude/mcp.json`:

```json
{
  "mcpServers": {
    "google-workspace": {
      "type": "stdio",
      "command": "uvx",
      "args": [
        "workspace-mcp",
        "--tools",
        "gmail",
        "calendar",
        "chat"
      ],
      "env": {
        "GOOGLE_OAUTH_CLIENT_ID": "PASTE_CLIENT_ID_HERE",
        "GOOGLE_OAUTH_CLIENT_SECRET": "PASTE_CLIENT_SECRET_HERE"
      }
    }
  }
}
```

Replace the placeholder values with your actual credentials.

## Step 3: Authenticate with Google

1. Quit Claude Code (`/exit`) and reopen it (`claude`)
2. Run `/mcp` to verify `google-workspace` appears in the server list
3. The first time you use a Google Workspace tool, it will open your browser for authentication:
   - Sign in with your Google Workspace account
   - Authorize the requested permissions (Gmail, Calendar, Chat)
   - Complete the redirect flow
4. Your OAuth token gets saved locally at `~/.google_workspace_mcp/credentials/`

## Step 4: Verify It Works

In Claude Code, try each service:

- **Gmail:** "Search my Gmail for the most recent message from [name]"
- **Calendar:** "What meetings do I have tomorrow?"
- **Google Chat:** "Search Google Chat for recent messages mentioning [topic]"

## Step 5: Pre-Approve MCP Tools (Optional)

By default, Claude Code asks for approval every time it uses an MCP tool. To avoid this, add trusted tools to your `~/.claude/settings.json` allow list:

```json
{
  "permissions": {
    "allow": [
      "mcp__google-workspace__search_gmail_messages",
      "mcp__google-workspace__get_gmail_message_content",
      "mcp__google-workspace__get_gmail_messages_content_batch",
      "mcp__google-workspace__get_gmail_thread_content",
      "mcp__google-workspace__get_gmail_threads_content_batch",
      "mcp__google-workspace__get_gmail_attachment_content",
      "mcp__google-workspace__list_gmail_labels",
      "mcp__google-workspace__list_gmail_filters",
      "mcp__google-workspace__get_events",
      "mcp__google-workspace__list_calendars",
      "mcp__google-workspace__query_freebusy",
      "mcp__google-workspace__list_spaces",
      "mcp__google-workspace__get_messages",
      "mcp__google-workspace__search_messages",
      "mcp__google-workspace__draft_gmail_message",
      "mcp__google-workspace__manage_event",
      "mcp__google-workspace__manage_gmail_label",
      "mcp__google-workspace__modify_gmail_message_labels",
      "mcp__google-workspace__batch_modify_gmail_message_labels",
      "mcp__google-workspace__create_reaction"
    ]
  }
}
```

Start with just the read-only tools if you want to be conservative, then add write tools as you build trust.

## Troubleshooting

**Server shows "failed" in `/mcp`:**
- Check that `uv` is installed: `uvx --version`
- If `uvx` is not found, install it: `brew install uv`
- Run `claude --debug` in a new session to see the error output

**Authentication errors:**
- Make sure you're signing in with the correct Google Workspace account
- Try deleting your saved token and re-authenticating: `rm -rf ~/.google_workspace_mcp/credentials/`
- Then restart Claude Code and try again

**"Permission denied" on Gmail/Calendar/Chat:**
- The OAuth app needs the correct API scopes
- Verify Gmail API, Calendar API, and Google Chat API are enabled in Google Cloud Console

**Only some tools work:**
- The `--tools gmail calendar chat` flag controls which services are enabled
- Google Drive and Docs are better served by the separate `google-drive` MCP — see `google-drive-mcp-setup.md`

## What You Can Do With It

- **Gmail:** "Search my inbox for unread messages from this week" / "Draft a reply to [name]'s email"
- **Calendar:** "What's on my calendar today?" / "Find a free slot for a 30-min meeting tomorrow"
- **Google Chat:** "Search Chat for messages about [topic]" / "What did [name] post in the [space] space?"

**Power Combos (with other MCP servers):**
- "Check my email and Asana for anything urgent, then plan my day" (Gmail + Asana)
- "Summarize yesterday's meeting and create follow-up tasks" (Fireflies + Asana)
- "Pull this month's CRM deals and email me a summary" (HubSpot + Gmail)

## Creating Your Own OAuth App

If you need to set up your own Google Cloud OAuth credentials:

1. Go to https://console.cloud.google.com
2. Select your project (or create one)
3. Navigate to **APIs & Services > Enabled APIs & services**
4. Enable these APIs:
   - Gmail API
   - Google Calendar API
   - Google Chat API
5. Navigate to **APIs & Services > Credentials**
6. Click **+ CREATE CREDENTIALS > OAuth client ID**
   - Application type: **Desktop app**
   - Name: `Claude Code Workspace MCP`
7. Click Create
8. Copy the Client ID and Client Secret
9. Use these values in Step 2 above

> If prompted to configure the OAuth consent screen, set User type to "Internal" (for Workspace accounts) and fill in the required fields.

## Security Notes

- Your OAuth token is stored locally at `~/.google_workspace_mcp/credentials/your-email.json`
- The token file should never be committed to git or shared
- The MCP server runs locally on your machine — no data passes through third-party servers
- The OAuth Client ID and Secret identify the app, not your personal account — sharing them with team members is fine
- Each person authenticates with their own Google account and gets their own token
