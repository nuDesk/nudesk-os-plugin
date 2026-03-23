# gws CLI Setup Guide

**Purpose:** Access Gmail, Calendar, Drive, Chat, Docs, Sheets, Slides, and People from Claude Code via the official Google Workspace CLI.

**Access method:** Bash tool — `gws` runs as a CLI command, not an MCP server.

---

## 1. Prerequisites

Homebrew must be installed on macOS:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Verify: `brew --version`

---

## 2. Install gws CLI

```bash
brew install google/gws/gws
```

Verify: `gws --version`

---

## 3. Install Agent Skills

This installs 42 Claude Agent Skills that wrap `gws` commands for use in Claude Code sessions:

```bash
npx skills add --yes --global https://github.com/googleworkspace/cli
```

These skills cover Gmail, Calendar, Drive, Chat, Docs, Sheets, Slides, Meet, Forms, Keep, Tasks, People, Classroom, and Admin.

---

## 4. Authenticate

Run the auth wizard — it opens a browser for OAuth consent:

```bash
gws auth setup
```

Sign in with your **nuDesk Google account** (`@nudesk.ai`). Select **Full Access (All Scopes)** when prompted on the consent screen.

---

## 5. Verify

```bash
gws auth status
```

Confirm these values in the output:
- `token_valid: true`
- `auth_method: oauth2`
- Account email matches your nuDesk address

Quick smoke test:

```bash
gws gmail users messages list --params '{"userId": "me", "maxResults": 1}'
```

---

## 6. Scope Reference

| Scope | Capability |
|-------|------------|
| `gmail` | Read, send, label, and manage Gmail messages |
| `calendar` | Read and write Google Calendar events |
| `drive` | Read and write Google Drive files and folders |
| `chat` | Send and read Google Chat messages and spaces |
| `docs` | Read and write Google Docs |
| `sheets` | Read and write Google Sheets |
| `slides` | Read and write Google Slides |
| `people` | Read contacts and directory |

To add missing scopes:

```bash
gws auth login -s gmail,calendar,drive,chat,docs,sheets,slides,people
```

---

## 7. Troubleshooting

| Issue | Fix |
|-------|-----|
| `which gws` returns nothing | Run `brew install google/gws/gws` |
| `token_valid: false` | Run `gws auth login` |
| Wrong Google account authenticated | Run `gws auth setup` and sign in with your `@nudesk.ai` account |
| Token expired | Run `gws auth login` (refresh token auto-handles most cases) |
| Missing scope (403 error) | Run `gws auth login -s <scope>` for the missing service |
| `validationError` on service name | Run `gws --help` for valid service names |
| Scopes not showing in `gws auth status` | Run `gws auth login -s gmail,calendar,drive` to re-grant |

---

## 8. SOC 2 Note

OAuth tokens are stored locally in macOS Keychain (`~/.config/gws/credentials.enc`) — encrypted at rest, never shared. Each team member authenticates as themselves using their own nuDesk Google account. No shared service account credentials. This satisfies nuDesk's SOC 2 access control requirement: **OAuth for user-context operations with user consent.**

---

## Superseded Guides

`google-workspace-mcp-setup.md` and `google-drive-mcp-setup.md` have been removed. This guide replaces both.
