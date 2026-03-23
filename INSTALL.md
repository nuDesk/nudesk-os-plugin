# nuDesk OS — Installation Guide

Complete setup for a new nuDesk team member. Takes about 10-15 minutes.

---

## Prerequisites

Confirm both are installed before starting:

```bash
brew --version    # Should return a version number
claude --version  # Should return a version number
```

**If Homebrew is not installed:**
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

**If Claude Code CLI is not installed:**
```bash
npm install -g @anthropic-ai/claude-code
```

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
4. **gws CLI** — Install and authenticate Google Workspace access (Gmail, Calendar, Drive, Chat)
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
| `gws` auth fails during setup | Run `gws auth setup` manually, sign in with your `@nudesk.ai` account |
| Asana MCP not connecting | Confirm the `asana` plugin is installed: `claude plugin list` |
| Missing skills (srd-generator, etc.) | Run `/nudesk-os:os-setup` Step 6 — it checks and reinstalls if missing |
| Need to start over | Run `/nudesk-os:os-setup` — it's safe to re-run, skips completed steps |
