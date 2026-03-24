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

`gcloud` is required to configure the OAuth client used by the `gws` CLI for Google Workspace access.

Before proceeding, make sure you have received the **OAuth Client ID and Client Secret** from your nuDesk admin — they will be shared with you securely. You will be prompted to paste them in during the authentication step.

---

## Step 1: Install nuDesk OS

Open Claude Code and type `/plugin`, then press Enter.

1. Navigate to **Marketplaces** and select **Add Marketplace**
2. Paste in the nuDesk plugin repository URL:
   ```
   https://github.com/nuDesk/nudesk-os-plugin
   ```
3. Once added, go to **Discover**, find **nuDesk OS**, and install it

---

## Step 2: Install Required Plugins

Open Claude Code and type `/plugin`, then press Enter. Use the plugin browser to search for and install each of the following one at a time. Always select **"Install for you (user scope)"** so plugins are available across all your projects.

**Required plugins — install this one:**

| Plugin | Search for |
|--------|------------|
| Skill Creator | `skill` |

**Role-specific — install based on your role:**

| Plugin | Search for | When to install |
|--------|------------|-----------------|
| Asana | `asana` | Task and project management |
| Commit Commands | `commit` | Git workflow automation |
| Ralph Loop | `ralph` | Recurring agent tasks |
| Security Guidance | `security` | Security reviews and compliance |
| Superpowers | `superpowers` | Advanced Claude Code workflows |
| Frontend Design | `frontend` | Building UI |
| Playwright | `playwright` | Browser automation |

Once all plugins are installed, run `/reload-plugins` to activate them in your current session.

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
4. **gws CLI** — Install and authenticate Google Workspace access. Run `gws auth setup`, paste in the Client ID and Secret when prompted, then select **"Recommended core consumer scopes"** and also manually check **"Chat messages"** (not included by default). See `references/setup/gws-cli-setup.md` for full details.
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
| `gws` auth fails — "No OAuth client configured" | You ran `gws auth login` before configuring credentials. Run `gws auth setup` instead and paste in the Client ID and Secret when prompted. |
| Asana MCP not connecting | Confirm the `asana` plugin is installed: `claude plugin list` |
| Missing skills (srd-generator, etc.) | Run `/nudesk-os:os-setup` Step 6 — it checks and reinstalls if missing |
| Need to start over | Run `/nudesk-os:os-setup` — it's safe to re-run, skips completed steps |
| Windows: `claude` not found after install | Close and reopen PowerShell, or run `npm install -g @anthropic-ai/claude-code` again |
| Windows: `gcloud` not found after install | Close and reopen PowerShell; if still missing, re-run `winget install Google.CloudSDK` |
