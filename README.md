# Executive OS — Claude Code Plugin

> **Version 3.0.0**

An executive operating system for Claude Code. Turns Claude into a daily operator that plans your day, executes tasks, generates weekly reports, runs security audits, and manages institutional memory — all powered by your existing tools.

## What It Does

### Commands

| Command | Cadence | Description |
|---------|---------|-------------|
| `/executive-os:daily-plan` | Daily | Morning planning — pulls from Asana, Calendar, Gmail, Fireflies, and HubSpot to generate a prioritized daily action list |
| `/executive-os:strategic-review` | Weekly/bi-weekly | Strategic planning — 7-30 day task landscape, undated task audit, calendar density, pipeline health, and strategic themes |
| `/executive-os:weekly-report` | Weekly | Generates a weekly executive progress report from all data sources and posts it to Asana |
| `/executive-os:run-tasks` | As needed | Executes today's Asana "Agent Queue" tasks sequentially with skill matching and user sign-off |
| `/executive-os:log-task` | As needed | Quick-capture an Asana task with smart project routing from a one-line description |
| `/executive-os:add-scheduled-task` | As needed | Add a new scheduled Asana task template — discovers templates, updates config, creates Cloud Scheduler job |
| `/executive-os:manage-schedules` | As needed | List, edit, pause/resume, or delete Cloud Scheduler jobs for Asana task automation |
| `/executive-os:session-closeout` | End of session | End-of-session wrap-up — captures tasks, updates memory, logs learnings |
| `/executive-os:context-sync` | As needed | Gather a 7-day cross-system briefing — git, Asana, Calendar, Gmail, Chat |
| `/executive-os:security-check` | As needed | Run a read-only security audit following the safe credential checklist |
| `/executive-os:os-setup` | First run | Guided setup wizard — configure CLAUDE.md, Asana, memory, hooks, and MCP servers with auto-discovery |
| `/executive-os:os-audit` | Monthly | Audit Executive OS installation — check config, plugins, compliance controls, and propose updates |

### Skills

| Skill | Type | Description |
|-------|------|-------------|
| **executive-planning** | Reference | Prioritization framework (embedded in commands, not invoked directly) |
| **asana-agent** | Auto-triggered | "run my tasks", "check my Asana queue", "process today's tasks" |
| **memory-management** | Auto-triggered | "remember this", "who is X", "what does X mean" |
| **soc2-compliance** | Reference | SOC 2 Type II compliance controls for Claude Code workflows |

### Agents

| Agent | Description |
|-------|-------------|
| **security-reviewer** | Generalized web application security audit agent — scans for OWASP vulnerabilities, auth issues, secrets exposure |

### Templates

| Template | Purpose |
|----------|---------|
| `CLAUDE.md.template` | Global Claude Code config — identity, stack, priorities, platform references |
| `asana-config.md.template` | Asana GIDs, routing table, and scheduled task automation paths |
| `memory-scaffold.md` | Memory directory structure and glossary starter |
| `hooks-settings.json.template` | SOC 2 .env blocker hook for workspace settings |

### References

Institutional knowledge docs bundled with the plugin — platform constraints, security procedures, MCP setup guides, and brand guides. These are generic best-practice docs (org-specific IDs and credentials scrubbed).

| Directory | Contents |
|-----------|----------|
| `references/platform-references/` | Verified patterns and anti-patterns for managed platforms (n8n Cloud, GCP Cloud Run, Google APIs, HubSpot, Apollo, Lovable) |
| `references/security/` | Security review guide — safe commands, dangerous commands to avoid, incident response |
| `references/mcp-setup/` | Step-by-step setup guides for Google Drive and Google Workspace MCP servers |
| `references/brand-guides/` | Visual identity and brand voice rules for client-facing deliverables |

## Prerequisites

### MCP Servers

Executive OS works best with these MCP servers connected:

| Server | Used For | Required? |
|--------|----------|-----------|
| **Asana** | Task management, agent queue, weekly reports | Yes |
| **Google Workspace** | Gmail, Calendar | Recommended |
| **Fireflies** | Meeting transcripts and action items | Optional |
| **HubSpot** | CRM deals and tasks | Optional |

Commands gracefully degrade when optional servers aren't available — they'll note what's missing and continue.

### Asana Custom Fields

The agent queue system requires these custom fields on your Asana tasks:

- **Task Progress** — with at least these enum values: Not Started, Agent Queue, In Progress, Pending Review, Done
- **Type** — categorizes tasks (e.g., Account Mgmt., Admin, Development)
- **Priority** — High, Medium, Low

## Setup

### Quick Start (Recommended)

Install the plugin, then run the guided setup wizard:

```bash
claude plugin add --url https://github.com/nuDesk/executive-os-plugin
```

```
/executive-os:os-setup
```

The setup wizard will:
- Generate your `~/.claude/CLAUDE.md` from your answers
- Create memory directories
- Check and smoke-test your MCP servers
- Auto-discover Asana GIDs (workspace, user, projects, custom fields) via MCP
- Install compliance hooks
- Recommend optional plugins

### Manual Setup

If you prefer to configure everything by hand:

1. Copy `templates/CLAUDE.md.template` to `~/.claude/CLAUDE.md` and fill in your details
2. Copy `templates/asana-config.md.template` to `~/.claude/memory/asana-config.md` and fill in your Asana GIDs (see the template's "How to Find Your GIDs" section)
3. Create memory directories: `mkdir -p ~/.claude/memory/{people,projects,context}`
4. Install MCP servers (at minimum, Asana): `claude mcp add asana -- npx -y @anthropic/asana-mcp-server`
5. Install the `.env` blocker hook from `templates/hooks-settings.json.template` into `~/.claude/settings.json`
6. Run `/executive-os:os-audit` to verify

## How It Works

Executive OS is built on a simple principle: **your CLAUDE.md is your config file**.

Every command reads from `~/.claude/CLAUDE.md` for user identity, priorities, and working memory. Every Asana interaction reads from `~/.claude/memory/asana-config.md` for GIDs and routing rules.

This means:
- No hardcoded user data in the plugin itself
- Each team member configures their own CLAUDE.md
- The plugin adapts to your role, your clients, your priorities
- Memory grows organically as you work — the session-closeout command suggests updates

### Platform References

Commands that write or modify code check the bundled `references/platform-references/` directory for platform constraints before executing. This prevents repeat debugging across team members when working with managed platforms (n8n, Cloud Run, Google APIs, HubSpot, Apollo, Lovable).

After installing the plugin, reference docs are available at `~/Projects/executive-os-plugin/references/`. The CLAUDE.md template points to these paths by default.

## Recommended Workflow

### Daily
1. **Morning:** `/executive-os:daily-plan` — Get your prioritized action list
2. **During the day:** `/executive-os:log-task` — Quick-capture tasks as they come up
3. **Task execution:** `/executive-os:run-tasks` — Process your agent queue
4. **End of session:** `/executive-os:session-closeout` — Capture tasks, update memory

### Weekly
5. **Mid-week or as needed:** `/executive-os:strategic-review` — 7-30 day strategic planning + undated task hygiene
6. **End of week:** `/executive-os:weekly-report` — Generate and post your report (skips redundant data pulls if strategic-review already ran)

### Periodic
7. **Monthly:** `/executive-os:os-audit` — Check installation health and plugin currency
8. **As needed:** `/executive-os:security-check` — Security audit before deployments or reviews
9. **As needed:** `/executive-os:context-sync` — Catch up on a project after time away

## Onboarding Checklist (New Team Members)

For new nuDesk team members setting up Executive OS:

1. [ ] Install Claude Code CLI
2. [ ] Clone the plugin repo: `git clone https://github.com/nuDesk/executive-os-plugin.git ~/Projects/executive-os-plugin/`
3. [ ] Install the plugin: `claude plugin add --url https://github.com/nuDesk/executive-os-plugin`
4. [ ] Install Asana MCP: `claude mcp add asana -- npx -y @anthropic/asana-mcp-server`
5. [ ] Run `/executive-os:os-setup` — the wizard handles CLAUDE.md, memory directories, Asana config (auto-discovers GIDs), compliance hooks, and plugin recommendations
6. [ ] Run `/executive-os:daily-plan` to confirm data sources are connected

## Where Things Live

| Location | Purpose | Examples |
|----------|---------|---------|
| `~/Projects/executive-os-plugin/` | Plugin source code, docs, and references | commands, skills, agents, templates, references, README |
| `~/.claude/` | Runtime config (hidden, auto-managed) | CLAUDE.md, memory/, plugins/, settings |
| `~/Projects/.claude/` | Workspace-scoped overrides | project-specific skills, hooks, agents |

## License

MIT

## Author

Built by [nuDesk LLC](https://nudesk.ai)
