# nuDesk OS — Installation Guide

Complete setup for a new nuDesk team member. Takes about 10-15 minutes.

---

## Prerequisites — Part 1: Install Required Software

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

## Prerequisites — Part 2: Launch Claude and Authenticate GWS

### Launch Claude Code

1. Open a terminal (macOS: Terminal app / Windows: PowerShell)
2. Navigate to your working folder. The easiest way is to type `cd` followed by a space, then drag your working folder into the terminal window — it will fill in the path automatically. Hit Enter.
3. Type `claude` and hit Enter to launch Claude Code

If prompted to log in, type `/login` and follow the browser authorization flow. Sign in with your nuDesk Google account (`@nudesk.ai`) and click **Allow**.

---

### Credentials you will need

Before authenticating GWS, make sure your nuDesk admin has shared the following with you securely:

- **OAuth Client ID**
- **OAuth Client Secret**

Have these ready to paste in — you will be prompted for them in the next step.

---

### Authenticate GWS CLI

The GWS CLI gives Claude Code access to Gmail, Calendar, Drive, Chat, Docs, and Sheets on your behalf.

1. Open a **new terminal tab** (do not use the Claude Code terminal for this)
2. Navigate to your working folder the same way as above (`cd` + drag + Enter)
3. Run:

```bash
gws auth setup
```

4. When prompted, paste in your **OAuth Client ID**, then your **OAuth Client Secret**

5. An interactive scope selection screen will appear. Follow these steps:
   - Scroll to **"Recommended core consumer scopes"** and press `Space` to select it
   - Scroll up in the list to find **"Chat messages"** (located just above Drive) and press `Space` to select it — this is not included in the defaults and is required for Google Chat
   - Press `Enter` to continue

   > Do not select any "read-only" scope variants — they will limit what Claude can do on your behalf.

6. Copy the OAuth URL that appears and open it in your browser
7. Sign in with your **nuDesk Google account** (`@nudesk.ai`) and click **Allow**
8. Return to the terminal — you should see a confirmation that authentication was successful

Verify:
```bash
gws auth status
```

Confirm `token_valid: true` and that the email matches your nuDesk address.

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

Type `/plugin` in Claude Code and press Enter. Search for and install each plugin one at a time. Always select **"Install for you (user scope)"** so plugins are available across all your projects.

**Required — install this one:**

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

In Claude Code, run:

```
/nudesk-os:os-setup
```

The wizard will walk you through:

1. **CLAUDE.md** — Generate your personal Claude config (name, role, stack, priorities)
2. **Memory directories** — Create `~/.claude/memory/` structure
3. **Asana config** — Auto-discover your workspace, user GID, projects, and custom fields via MCP
4. **gws CLI** — Confirm GWS authentication from Part 2 above
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

When a new version is released, type `/plugin` in Claude Code, navigate to **Installed**, find **nuDesk OS**, and select **Update**. Then run `/reload-plugins` to apply the changes.

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| Commands not appearing after install | Run `/reload-plugins` in Claude Code |
| `gws` auth fails — "No OAuth client configured" | You ran `gws auth login` before configuring credentials. Run `gws auth setup` instead and paste in the Client ID and Secret when prompted. |
| Asana MCP not connecting | Confirm the `asana` plugin is installed via `/plugin` |
| Missing skills (srd-generator, etc.) | Run `/nudesk-os:os-setup` Step 6 — it checks and reinstalls if missing |
| Need to start over | Run `/nudesk-os:os-setup` — it's safe to re-run, skips completed steps |
| Windows: `claude` not found after install | Close and reopen PowerShell, or run `npm install -g @anthropic-ai/claude-code` again |
| Windows: `gcloud` not found after install | Close and reopen PowerShell; if still missing, re-run `winget install Google.CloudSDK` |
