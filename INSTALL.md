# nuDesk OS — Installation Guide

Complete setup for a new nuDesk team member. Takes about 10-15 minutes.

---

## Prerequisites

### macOS

**1. Install Homebrew**

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Verify:
```bash
brew --version
```

---

**2. Install Claude Code CLI**

```bash
npm install -g @anthropic-ai/claude-code
```

Verify:
```bash
claude --version
```

---

**3. Install Google Cloud SDK**

```bash
brew install --cask google-cloud-sdk
```

Verify:
```bash
gcloud --version
```

---

### Windows

Open **PowerShell as Administrator** for all steps below.

**1. Install Node.js** (required for Claude Code)

```powershell
winget install OpenJS.NodeJS
```

Close and reopen PowerShell, then verify:
```powershell
node --version
```

---

**2. Install Claude Code CLI**

```powershell
npm install -g @anthropic-ai/claude-code
```

Verify:
```powershell
claude --version
```

---

**3. Install Google Cloud SDK**

```powershell
winget install Google.CloudSDK
```

Close and reopen PowerShell, then run the setup wizard:
```powershell
gcloud init
```

Verify:
```powershell
gcloud --version
```

---

### Both platforms — credentials setup

`gcloud` is required to configure the OAuth client used by the `gws` CLI for Google Workspace access. You will also need the `client_secret.json` file — retrieve it from the **nuDesk NordPass vault** before proceeding.

Save it to the correct location for your OS:

| OS | Path |
|----|------|
| macOS | `~/.config/gws/client_secret.json` |
| Windows | `%USERPROFILE%\.config\gws\client_secret.json` (e.g. `C:\Users\yourname\.config\gws\client_secret.json`) |

> **Note:** We are evaluating a simplified credential alternative that would remove this requirement. Until then, `gcloud` + the shared credentials file is the required path.

---

## Step 1: Install nuDesk OS

Register the nuDesk plugin marketplace and install:

```bash
claude plugin marketplace add https://github.com/nuDesk/nudesk-os-plugin
claude plugin install nudesk-os@nudesk-os
```

---

## Step 2: Install Required Plugins

```bash
claude plugin add asana@claude-plugins-official
claude plugin add commit-commands@claude-plugins-official
claude plugin add ralph-loop@claude-plugins-official
claude plugin add security-guidance@claude-plugins-official
claude plugin add superpowers@claude-plugins-official
claude plugin add skill-creator@claude-plugins-official
```

**Role-specific (install if relevant to your work):**
```bash
claude plugin add frontend-design@claude-plugins-official  # If building UI
claude plugin add playwright@claude-plugins-official       # If doing browser automation
```

---

## Step 3: Run the Setup Wizard

Open Claude Code and run:

```
/nudesk-os:os-setup
```

The wizard will walk you through:

1. **CLAUDE.md** — Generate your personal Claude config (name, role, stack, priorities)
2. **Memory directories** — Create `~/.claude/memory/` structure
3. **Asana config** — Auto-discover your workspace, user GID, projects, and custom fields via MCP
4. **gws CLI** — Install and authenticate Google Workspace access. When prompted for scopes, select **"Recommended core consumer scopes"** and also manually check **"Chat messages"** (it is not included by default). See `references/setup/gws-cli-setup.md` for full details.
5. **Compliance hooks** — Install the SOC 2 `.env` blocker hook
6. **Skills** — Verify nuDesk OS skills are installed (srd-generator, ai-solution-architect, nudesk-brand-styling)

---

## Step 4: Confirm Everything Works

```
/nudesk-os:daily-plan
```

This touches every live data source (Asana, Calendar, Gmail, Fireflies, HubSpot). If it runs without errors, you're fully set up.

For a detailed health report:
```
/nudesk-os:os-audit
```

---

## Updating nuDesk OS

When a new version is released:

```bash
claude plugin update nudesk-os@nudesk-os
```

Then reload plugins in Claude Code with `/reload-plugins`.

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| Commands not appearing after install | Run `/reload-plugins` in Claude Code |
| `gws` auth fails — "No OAuth client configured" | Ensure `client_secret.json` is in the correct path for your OS (see Prerequisites above), then run `gws auth login` |
| Asana MCP not connecting | Confirm the `asana` plugin is installed: `claude plugin list` |
| Missing skills (srd-generator, etc.) | Run `/nudesk-os:os-setup` Step 6 — it checks and reinstalls if missing |
| Need to start over | Run `/nudesk-os:os-setup` — it's safe to re-run, skips completed steps |
| Windows: `claude` not found after install | Close and reopen PowerShell, or run `npm install -g @anthropic-ai/claude-code` again |
| Windows: `gcloud` not found after install | Close and reopen PowerShell; if still missing, re-run `winget install Google.CloudSDK` |
