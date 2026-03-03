# Executive OS — Claude Code Plugin

An executive operating system for Claude Code. Turns Claude into a daily operator that plans your day, executes tasks, generates weekly reports, and manages institutional memory — all powered by your existing tools.

## What It Does

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

## Skills

| Skill | Type | Description |
|-------|------|-------------|
| **executive-planning** | Reference | Prioritization framework (embedded in commands, not invoked directly) |
| **asana-agent** | Auto-triggered | "run my tasks", "check my Asana queue", "process today's tasks" |
| **memory-management** | Auto-triggered | "remember this", "who is X", "what does X mean" |

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

### 1. Install the Plugin

```bash
claude plugin add --url https://github.com/nuDesk/executive-os-plugin
```

### 2. Configure Your CLAUDE.md

Copy `templates/CLAUDE.md.template` to `~/.claude/CLAUDE.md` and fill in your details:

- Your name, role, email
- Your strategic goals and current priorities
- Working memory (key people, terms, clients)
- Integration config (email, HubSpot owner ID)

### 3. Configure Asana

Copy `templates/asana-config.md.template` to `~/.claude/memory/asana-config.md` and fill in your Asana GIDs:

- Workspace and user GIDs
- Custom field GIDs and enum option GIDs
- Project routing table (keywords → project GIDs)

See the template's "How to Find Your GIDs" section for help.

### 4. Set Up Memory Directory

Create the memory directory structure:

```bash
mkdir -p ~/.claude/memory/{people,projects,context}
```

See `templates/memory-scaffold.md` for the full structure and a glossary starter.

### 5. Connect MCP Servers

Install and configure the MCP servers you'll use. At minimum, you need Asana:

```bash
claude plugin install asana@claude-plugins-official
```

## How It Works

Executive OS is built on a simple principle: **your CLAUDE.md is your config file**.

Every command reads from `~/.claude/CLAUDE.md` for user identity, priorities, and working memory. Every Asana interaction reads from `~/.claude/memory/asana-config.md` for GIDs and routing rules.

This means:
- No hardcoded user data in the plugin itself
- Each team member configures their own CLAUDE.md
- The plugin adapts to your role, your clients, your priorities
- Memory grows organically as you work — the session-closeout command suggests updates

## Recommended Workflow

### Daily
1. **Morning:** `/executive-os:daily-plan` — Get your prioritized action list
2. **During the day:** `/executive-os:log-task` — Quick-capture tasks as they come up
3. **Task execution:** `/executive-os:run-tasks` — Process your agent queue
4. **End of session:** `/executive-os:session-closeout` — Capture tasks, update memory

### Weekly
5. **Mid-week or as needed:** `/executive-os:strategic-review` — 7-30 day strategic planning + undated task hygiene
6. **End of week:** `/executive-os:weekly-report` — Generate and post your report (skips redundant data pulls if strategic-review already ran)

## License

MIT

## Author

Built by [nuDesk LLC](https://nudesk.ai)
