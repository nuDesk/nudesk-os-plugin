# gws CLI Setup Guide

**Purpose:** Access Gmail, Calendar, Drive, Chat, Docs, Sheets, Slides, and People from Claude Code via the official Google Workspace CLI.

**Access method:** Bash tool — `gws` runs as a CLI command, not an MCP server.

---

## 1. Prerequisites

### macOS

Homebrew must be installed:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Verify: `brew --version`

Install Google Cloud SDK:

```bash
brew install --cask google-cloud-sdk
```

Verify: `gcloud --version`

---

### Windows

Open **PowerShell as Administrator**.

Install Google Cloud SDK:

```powershell
winget install Google.CloudSDK
```

Close and reopen PowerShell after installing, then initialize:

```powershell
gcloud init
```

Verify: `gcloud --version`

---

### Both platforms — credentials setup

The `gws` CLI requires an OAuth client to authenticate. This is configured via a `client_secret.json` file tied to a GCP project.

Retrieve the `client_secret.json` file from the **nuDesk NordPass vault** and save it to the correct location for your OS:

| OS | Path |
|----|------|
| macOS | `~/.config/gws/client_secret.json` |
| Windows | `%USERPROFILE%\.config\gws\client_secret.json` (e.g. `C:\Users\yourname\.config\gws\client_secret.json`) |

Create the folder if it doesn't exist:

**macOS:**
```bash
mkdir -p ~/.config/gws
```

**Windows (PowerShell):**
```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.config\gws"
```

> **Note:** We are evaluating a simplified credential path that would remove the gcloud dependency. Until then, the shared credentials file is required.

---

## 2. Install gws CLI

### macOS

```bash
brew install google/gws/gws
```

Verify: `gws --version`

---

### Windows

Download the latest Windows binary from the [gws CLI releases page](https://github.com/googleworkspace/cli/releases). Download `gws_windows_amd64.exe`, rename it to `gws.exe`, and move it to a folder on your PATH (e.g. `C:\Program Files\gws\`).

Add it to your PATH if needed (PowerShell as Administrator):

```powershell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\gws", "Machine")
```

Close and reopen PowerShell, then verify: `gws --version`

---

## 3. Install Agent Skills

This installs 42 Claude Agent Skills that wrap `gws` commands for use in Claude Code sessions:

```bash
npx skills add --yes --global https://github.com/googleworkspace/cli
```

These skills cover Gmail, Calendar, Drive, Chat, Docs, Sheets, Slides, Meet, Forms, Keep, Tasks, People, Classroom, and Admin.

---

## 4. Authenticate

Confirm `client_secret.json` is in place (from NordPass, at the path for your OS listed above), then run:

```bash
gws auth login
```

This opens a browser for OAuth consent. Sign in with your **nuDesk Google account** (`@nudesk.ai`). Select **Full Access (All Scopes)** when prompted on the consent screen.

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
| `which gws` returns nothing (macOS) | Run `brew install google/gws/gws` |
| `gws` not found (Windows) | Ensure `gws.exe` is in a folder on your PATH; close and reopen PowerShell |
| `token_valid: false` | Run `gws auth login` |
| "No OAuth client configured" error | Ensure `client_secret.json` is saved to the correct path for your OS (see Prerequisites above) |
| Wrong Google account authenticated | Run `gws auth login` again and sign in with your `@nudesk.ai` account |
| Token expired | Run `gws auth login` (refresh token auto-handles most cases) |
| Missing scope (403 error) | Run `gws auth login -s <scope>` for the missing service |
| `validationError` on service name | Run `gws --help` for valid service names |
| Scopes not showing in `gws auth status` | Run `gws auth login -s gmail,calendar,drive` to re-grant |

---

## 8. SOC 2 Note

OAuth tokens are stored locally in encrypted storage (`~/.config/gws/credentials.enc` on macOS, `%USERPROFILE%\.config\gws\credentials.enc` on Windows) — encrypted at rest, never shared. Each team member authenticates as themselves using their own nuDesk Google account. No shared service account credentials. This satisfies nuDesk's SOC 2 access control requirement: **OAuth for user-context operations with user consent.**

---

## Superseded Guides

`google-workspace-mcp-setup.md` and `google-drive-mcp-setup.md` have been removed. This guide replaces both.
