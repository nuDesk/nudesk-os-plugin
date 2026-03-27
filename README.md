# nuDesk OS — Claude Code Plugin

> **Version 3.1.1**

An executive operating system for Claude Code. Turns Claude into a daily operator that plans your day, executes tasks, generates weekly reports, runs security audits, and manages institutional memory — all powered by your existing tools.

## What It Does

### Commands

| Command | Cadence | Description |
|---------|---------|-------------|
| `/nudesk-os:daily-plan` | Daily | Morning planning — pulls from Asana, Calendar, Gmail, Fireflies, and HubSpot to generate a prioritized daily action list |
| `/nudesk-os:strategic-review` | Weekly/bi-weekly | Strategic planning — 7-30 day task landscape, undated task audit, calendar density, pipeline health, and strategic themes |
| `/nudesk-os:weekly-report` | Weekly | Generates a weekly executive progress report from all data sources and posts it to Asana |
| `/nudesk-os:run-tasks` | As needed | Executes today's Asana "Agent Queue" tasks sequentially with skill matching and user sign-off |
| `/nudesk-os:log-task` | As needed | Quick-capture an Asana task with smart project routing from a one-line description |
| `/nudesk-os:add-scheduled-task` | As needed | Add a new scheduled Asana task template — discovers templates, updates config, creates Cloud Scheduler job |
| `/nudesk-os:manage-schedules` | As needed | List, edit, pause/resume, or delete Cloud Scheduler jobs for Asana task automation |
| `/nudesk-os:session-closeout` | End of session | End-of-session wrap-up — captures tasks, updates memory, logs learnings |
| `/nudesk-os:context-sync` | As needed | Gather a 7-day cross-system briefing — git, Asana, Calendar, Gmail, Chat |
| `/nudesk-os:security-check` | As needed | Run a read-only security audit following the safe credential checklist |
| `/nudesk-os:os-setup` | First run | Guided setup wizard — configure CLAUDE.md, Asana, memory, hooks, and MCP servers with auto-discovery |
| `/nudesk-os:compliance-status` | As needed | Compliance dashboard — single view across Vanta and Asana compliance projects |
| `/nudesk-os:evidence-collect` | As needed | Collect compliance evidence from git, deployments, infrastructure — prepare for Asana and Vanta |
| `/nudesk-os:incident-log` | As needed | Log and track a security incident with severity classification and 6-phase response |
| `/nudesk-os:compliance-report` | Quarterly | Generate audit-ready compliance report — all 91 controls with status and evidence |
| `/nudesk-os:os-audit` | Monthly | Audit nuDesk OS installation — check config, plugins, compliance controls, and propose updates |

### Skills

| Skill | Type | Description |
|-------|------|-------------|
| **executive-planning** | Reference | Prioritization framework (embedded in commands, not invoked directly) |
| **asana-agent** | Auto-triggered | "run my tasks", "check my Asana queue", "process today's tasks" |
| **memory-management** | Auto-triggered | "remember this", "who is X", "what does X mean" |
| **soc2-compliance** | Active + Reference | SOC 2 compliance enforcement — queries Asana and Vanta for live control status against 91-control matrix |
| **evidence-collector** | Background | Processes evidence buffer into Asana Change Log entries and Vanta uploads |
| **vanta-bridge** | Background | Syncs Asana compliance data (change log, incidents) to Vanta via REST API |
| **srd-generator** | On-demand | Generates Solution Requirements Documents for AI agent consumption — "PRD", "requirements document", "build brief", "spec out", "technical requirements" |
| **ai-solution-architect** | On-demand | Technical strategy partner for AI solution design and build vs. buy decisions — "help me design", "I need to build", "architecture review", "solution design" |
| **nudesk-brand-styling** | Auto-triggered | Applies nuDesk brand colors, typography, and design standards to presentations, reports, and client-facing materials |

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
| `compliance-config.md.template` | Vanta status, Asana compliance project GIDs, custom fields, review schedule |

### References

Institutional knowledge docs bundled with the plugin — platform constraints, security procedures, MCP setup guides, and brand guides. These are generic best-practice docs (org-specific IDs and credentials scrubbed).

| Directory | Contents |
|-----------|----------|
| `references/platform-references/` | Verified patterns and anti-patterns for managed platforms (n8n Cloud, GCP Cloud Run, Google APIs, HubSpot, Apollo, Lovable, Vanta) |
| `references/security/` | Security review guide, control-action map (91 controls → enforcement mechanisms), executive permissions standard |
| `references/setup/` | Setup guide for the `gws` CLI (Google Workspace via Bash) |
| `references/mcp-setup/` | Step-by-step setup guides for Google Drive and Google Workspace MCP servers |
| `references/brand-guides/` | Visual identity and brand voice rules for client-facing deliverables |

## Compliance

nuDesk OS includes a SOC 2 compliance operating system that bridges Vanta (primary compliance platform) and Asana (operational execution layer).

### Architecture

```
VANTA (System of Record)
├── Access Reviews, Vendor Assessments, Policy Acknowledgments
├── Automated compliance tests and evidence collection
└── Framework tracking (SOC 2 TSC mapping)

ASANA (Operational Execution — workflows Vanta doesn't natively support)
├── Production Change Log — all prod changes with compliance checklists
├── Incident Response Log — structured 6-phase incident tracking
└── Risk Register — active risks with treatment plans

EXECUTIVE OS (Bridge + Enforcement)
├── Hooks: .env blocker, pre-deploy gate, PII scan, evidence buffer
├── Commands: /compliance-status, /evidence-collect, /incident-log, /compliance-report
├── Skills: soc2-compliance (active), evidence-collector, vanta-bridge
└── Enhanced: /security-check, /session-closeout, /weekly-report, /daily-plan
```

### 91 Controls Across 16 Policies

All controls are mapped in `references/security/control-action-map.md`:
- **28 Automatable** — enforced via hooks, scheduled tasks, auto-evidence
- **31 Semi-Automated** — human-triggered commands with Asana workflows
- **32 Policy-Only** — scheduled review reminders and acknowledgment tracking

### Vanta Integration

Works at all Vanta tiers:
- **Basic (UI-only):** Evidence and reports generated locally for manual Vanta upload
- **Core (API):** REST API sync via vanta-bridge skill
- **Core+ (API + MCP):** Real-time queries + REST writes

### Getting Started with Compliance

1. Run `/nudesk-os:os-setup` — the wizard now includes compliance infrastructure setup (Step 5b)
2. Run `/nudesk-os:compliance-status` for your first dashboard view
3. Run `/nudesk-os:os-audit` to verify compliance config health

### Evidence Collection Flow

```
Session Work → Hooks append to evidence buffer → /session-closeout processes buffer
    → evidence-collector creates Asana Change Log entries
    → vanta-bridge syncs to Vanta (if API available)
```

## Prerequisites

### MCP Servers

nuDesk OS works best with these MCP servers connected:

| Server | Used For | Required? |
|--------|----------|-----------|
| **Asana** | Task management, agent queue, weekly reports | Yes |
| **Fireflies** | Meeting transcripts and action items | Optional |
| **HubSpot** | CRM deals and tasks | Optional |

Commands gracefully degrade when optional servers aren't available — they'll note what's missing and continue.

### gws CLI (Google Workspace)

Gmail, Calendar, Drive, Chat, Docs, Sheets, and Slides are accessed via the **`gws` CLI** — not an MCP server. Each user authenticates with their own Google account.

Install:
```bash
brew install google/gws/gws
npx skills add --yes --global https://github.com/googleworkspace/cli
gws auth setup
```

See `references/setup/gws-cli-setup.md` for the full setup guide.

### Asana Custom Fields

The agent queue system requires these custom fields on your Asana tasks:

- **Task Progress** — with at least these enum values: Not Started, Agent Queue, In Progress, Pending Review, Done
- **Type** — categorizes tasks (e.g., Account Mgmt., Admin, Development)
- **Priority** — High, Medium, Low

## Setup

### Quick Start (Recommended)

Install the plugin, then run the guided setup wizard:

```bash
claude plugin add --url https://github.com/nuDesk/nudesk-os-plugin
```

```
/nudesk-os:os-setup
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
6. Run `/nudesk-os:os-audit` to verify

## How It Works

nuDesk OS is built on a simple principle: **your CLAUDE.md is your config file**.

Every command reads from `~/.claude/CLAUDE.md` for user identity, priorities, and working memory. Every Asana interaction reads from `~/.claude/memory/asana-config.md` for GIDs and routing rules.

This means:
- No hardcoded user data in the plugin itself
- Each team member configures their own CLAUDE.md
- The plugin adapts to your role, your clients, your priorities
- Memory grows organically as you work — the session-closeout command suggests updates

### Platform References

Commands that write or modify code check the bundled `references/platform-references/` directory for platform constraints before executing. This prevents repeat debugging across team members when working with managed platforms (n8n, Cloud Run, Google APIs, HubSpot, Apollo, Lovable).

After installing the plugin, reference docs are available at `~/Projects/nudesk-os-plugin/references/`. The CLAUDE.md template points to these paths by default.

## Recommended Workflow

### Daily
1. **Morning:** `/nudesk-os:daily-plan` — Get your prioritized action list
2. **During the day:** `/nudesk-os:log-task` — Quick-capture tasks as they come up
3. **Task execution:** `/nudesk-os:run-tasks` — Process your agent queue
4. **End of session:** `/nudesk-os:session-closeout` — Capture tasks, update memory

### Weekly
5. **Mid-week or as needed:** `/nudesk-os:strategic-review` — 7-30 day strategic planning + undated task hygiene
6. **End of week:** `/nudesk-os:weekly-report` — Generate and post your report (skips redundant data pulls if strategic-review already ran)

### Periodic
7. **Monthly:** `/nudesk-os:os-audit` — Check installation health and plugin currency
8. **As needed:** `/nudesk-os:security-check` — Security audit before deployments or reviews
9. **As needed:** `/nudesk-os:context-sync` — Catch up on a project after time away

## Onboarding (New Team Members)

See **[INSTALL.md](./INSTALL.md)** for the full step-by-step guide.

**Quick start:**

```bash
# Prerequisites: Homebrew + Claude Code CLI must be installed first

# 1. Install nuDesk OS
claude plugin marketplace add https://github.com/nuDesk/nudesk-os-plugin
claude plugin install nudesk-os@nudesk-os

# 2. Install required plugins
claude plugin add asana@claude-plugins-official
claude plugin add commit-commands@claude-plugins-official
claude plugin add ralph-loop@claude-plugins-official
claude plugin add security-guidance@claude-plugins-official
claude plugin add superpowers@claude-plugins-official
claude plugin add skill-creator@claude-plugins-official

# 3. Run setup wizard — handles everything else
# (Open Claude Code and run:)
# /nudesk-os:os-setup
```

## Where Things Live

| Location | Purpose | Examples |
|----------|---------|---------|
| `~/Projects/nudesk-os-plugin/` | Plugin source code, docs, and references | commands, skills, agents, templates, references, README |
| `~/.claude/` | Runtime config (hidden, auto-managed) | CLAUDE.md, memory/, plugins/, settings |
| `~/Projects/.claude/` | Workspace-scoped overrides | project-specific skills, hooks, agents |

## License

MIT

## Author

Built by [nuDesk LLC](https://nudesk.ai)
