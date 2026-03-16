# Google Workspace CLI (`gws`) Setup for Claude Code

**Purpose:** Access Gmail, Calendar, Chat, Drive, Docs, Sheets, Slides, People, Tasks, Forms, Meet, and Admin from Claude Code via the official `@googleworkspace/cli` tool.

**Package:** `@googleworkspace/cli` (npm)
**Auth:** OAuth2 via browser, tokens encrypted in macOS Keychain
**Access method:** Bash tool (not MCP — Google removed MCP mode due to context window overflow from their large API surface)

## Prerequisites

- Node.js 18+
- A GCP project with Workspace APIs enabled
- An OAuth 2.0 client ID (Desktop type) in that GCP project
- `gcloud` CLI installed and authenticated

## Step 1: Install

```bash
npm install -g @googleworkspace/cli
```

## Step 2: Auth Setup

This configures the GCP project and OAuth client:

```bash
gws auth setup --client-id="YOUR_CLIENT_ID" --client-secret="YOUR_CLIENT_SECRET"
```

If running the interactive wizard instead, you'll need:
- Your GCP project ID (e.g., `nudesk-agent-builder`, NOT the project number)
- Select which Workspace APIs to enable (recommended: all core services)

## Step 3: Login

Authenticate with full scopes for all services:

```bash
gws auth login -s drive,gmail,calendar,sheets,docs,slides,chat,people
```

This opens a browser for OAuth consent. Select **Full Access (All Scopes)** if your consent screen is set to Internal (Workspace org only).

## Step 4: Verify

```bash
gws auth status
```

Confirm: `"token_valid": true`, `"auth_method": "oauth2"`, `"storage": "encrypted"`.

Quick functional test:

```bash
gws gmail users messages list --params '{"userId": "me", "maxResults": 1}'
```

## Usage in Claude Code

`gws` is accessed via the Bash tool, not as an MCP server. Usage pattern:

```bash
gws <service> <resource> <method> --params '{"key": "value"}' --json '{"body": "data"}'
```

### Common Commands

```bash
# Gmail — list messages
gws gmail users messages list --params '{"userId": "me", "q": "is:unread newer_than:1d", "maxResults": 10}'

# Gmail — read a message
gws gmail users messages get --params '{"userId": "me", "id": "<messageId>"}'

# Calendar — today's events
gws calendar events list --params '{"calendarId": "primary", "timeMin": "2026-03-16T00:00:00Z", "timeMax": "2026-03-16T23:59:59Z", "singleEvents": true, "orderBy": "startTime"}'

# Drive — list files
gws drive files list --params '{"pageSize": 10}'

# Chat — send a message
gws chat spaces messages create --params '{"parent": "spaces/<spaceId>"}' --json '{"text": "Hello"}'

# People — lookup a contact
gws people people get --params '{"resourceName": "people/<id>", "personFields": "names,emailAddresses"}'
```

### Shell Escaping

- Use `\u0021` for `!` in JSON strings (bash interprets `!` as history expansion)
- Use single quotes for `--params` and `--json` values
- For apostrophes in text, close and reopen the single quote: `'it'\''s working'`

## Troubleshooting

| Issue | Fix |
|-------|-----|
| `auth_method: none` | Run `gws auth login` |
| Token expired | Run `gws auth login` again (refresh token handles most cases automatically) |
| API not enabled | Run `gws auth setup` and enable the missing API |
| `validationError` on service | Check `gws --help` for valid service names |

## Security Notes

- Credentials encrypted at rest, stored in macOS Keychain (`~/.config/gws/credentials.enc`)
- No plain-text credential files on disk (unlike legacy community MCPs)
- OAuth consent screen should be set to **Internal** (Workspace org only)
- Replaces legacy `workspace-mcp` (PyPI) and `@piotr-agier/google-drive-mcp` (npm) community MCPs

## Superseded Setup Guides

The following guides are deprecated and retained only for reference:
- `google-workspace-mcp-setup.md` — replaced by this guide
- `google-drive-mcp-setup.md` — replaced by this guide
