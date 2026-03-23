# Claude Code Setup Guide for nuDesk Founders

**Prepared for:** Kenny Salas   
**Date:** March 21, 2026 (v5)   
**Approach:** Start simple, layer complexity as needed   
**Source of Truth:** Boris Cherny's 10 Tips (Jan 31, 2026\) \+ Anthropic documentation \+ Kenny's actual installed setup

---

## Overview

This guide follows Boris Cherny's (Claude Code creator) 10 recommended best practices, adapted for a non-technical founder who's building AI agents and automations — not shipping production software. Every tip from Boris is covered here, mapped to your context.

**The guide is structured in two parts:**

- **Part 1: Setup** — Configuration you do once (workspace structure, .claude directory, CLAUDE.md, permissions, plugins, commands, skills, hooks, GitHub, MCP)  
- **Part 2: Working Patterns** — Techniques to use daily (plan mode, prompting, verification, subagents, bug fixing, data analysis, learning mode)

---

## Model Selection: Use Opus 4.6

Boris is emphatic: **Opus 4.6 with Thinking is the model to use for Claude Code.** Despite being larger and seemingly slower, it requires less steering, is better at tool use, and actually ends up faster and cheaper in total tokens because it gets things right the first time.

If you're on a Claude Max or Team plan, make sure Opus 4.6 is your default model in Claude Code. The reduced back-and-forth alone makes it worth it — especially for a non-technical founder who benefits from the model needing fewer corrections.

---

# Part 1: Setup

---

## 0\. Workspace Structure

Before configuring anything, set up your folder structure. This matters because **each top-level folder \= one git repo \= one Claude Code context.** When you `cd` into a folder and type `claude`, it reads that folder's CLAUDE.md and `.claude/` config. Your folder structure *is* your context architecture.

### The Layout

```
~/claude-code/                              ← Master workspace (not a git repo itself)
│
├── nudesk-internal/                 ← nuDesk ops, internal tools, corporate
│   ├── CLAUDE.md                    ← "This project is about nuDesk internal operations..."
│   ├── .claude/
│   │   ├── commands/                ← Project-specific slash commands
│   │   ├── skills/                  ← Project-specific skills
│   │   └── mcp.json                ← MCP overrides (if different from global)
│   ├── .env                         ← Secrets (gitignored)
│   ├── agents/                      ← AI agents you build for internal use
│   ├── scripts/                     ← One-off automations, data scripts
│   ├── docs/                        ← Brand guidelines, SOPs, playbooks
│   └── README.md
│
├── champions/                       ← Champions Funding client work
│   ├── CLAUDE.md                    ← "This project supports Champions Funding..."
│   ├── .claude/
│   ├── .env
│   ├── agents/                      ← AM workflow tools, Loan Expert, etc.
│   ├── scripts/
│   ├── docs/                        ← Champions-specific SOPs, guidelines
│   └── README.md
│
├── prime-nexus/                     ← Prime Nexus / Live Oak
│   ├── CLAUDE.md                    ← "This project is the Prime Nexus platform..."
│   ├── .claude/
│   ├── .env
│   ├── hubspot/                     ← HubSpot integration code
│   ├── agents/                      ← SDR agent, PreQual agent, etc.
│   ├── docs/
│   └── README.md
│
├── training/                        ← Cyborg Academy / AI enablement
│   ├── CLAUDE.md
│   ├── .claude/
│   ├── courses/                     ← Training content, modules
│   ├── scripts/                     ← Content generation tools
│   └── README.md
│
└── sandbox/                         ← Experiments, learning, throwaway
    ├── CLAUDE.md                    ← "This is my experimentation space..."
    ├── .claude/
    └── [whatever you're tinkering with]
```

### Design Principles

**Separate repos by business context, not by technology.** Don't organize by tool (all Python together, all agents together). Organize by *what Claude needs to know.* When you're working on Champions, Claude should have Champions context loaded — not Prime Nexus context cluttering the window.

**The `sandbox/` repo matters.** When learning or experimenting, you don't want to pollute a real project. Give yourself a dedicated space with a CLAUDE.md that says "this is for experimentation, prioritize teaching me over production quality."

**Project-specific vs. global config.** Your global commands (`/ship`, `/review`, `/techdebt`) live in `~/.claude/commands/` and work everywhere. Project-specific commands (like a `/loan-check` that only makes sense for Champions) go in that project's `.claude/commands/`.

**Don't nest git repos.** Each folder under `~/claude-code/` is its own independent repo. The `~/claude-code/` folder itself is just a plain directory — not a repo.

### How to Initialize

```shell
# Create the master workspace
mkdir -p ~/claude-code

# Create your first project
cd ~/claude-code
mkdir -p nudesk-internal/{agents,scripts,docs,.claude/commands,.claude/skills}
cd nudesk-internal
git init
touch .env .gitignore CLAUDE.md README.md

# Add .gitignore essentials
echo ".env
__pycache__/
*.pyc
.DS_Store
.claude/worktrees/" > .gitignore

# Repeat for each project
cd ~/claude-code
mkdir -p champions/{agents,scripts,docs,.claude/commands}
cd champions && git init && touch .env .gitignore CLAUDE.md README.md

cd ~/claude-code
mkdir -p prime-nexus/{hubspot,agents,docs,.claude/commands}
cd prime-nexus && git init && touch .env .gitignore CLAUDE.md README.md

cd ~/claude-code
mkdir -p sandbox/.claude/commands
cd sandbox && git init && touch .env .gitignore CLAUDE.md

cd ~/claude-code
mkdir -p training/{courses,scripts,.claude/commands}
cd training && git init && touch .env .gitignore CLAUDE.md README.md
```

Once this is done, you'll populate each CLAUDE.md with project-specific context (covered in the next section).

---

## 0.5. The `~/.claude/` Directory — Your Control Center

Before you start configuring Claude Code, it helps to understand **where everything lives**.

### What is `~/.claude/`?

`~/.claude/` is a **hidden directory** in your home folder. It starts with a dot (`.`), which means your Mac hides it from normal file views — you won't see it in Finder unless you press `Cmd+Shift+.` to toggle hidden files. You also won't see it with a plain `ls` — use `ls -la` to reveal it.

This directory is **Claude Code's personal configuration folder.** It stores everything that makes your Claude Code setup yours:

| Folder | What It Contains |
| :---- | :---- |
| `~/.claude/commands/` | Your global slash commands (`/review`, `/ship`, `/daily-plan`, etc.) |
| `~/.claude/skills/` | Your global skills (nudesk-branding, hubspot-ops, etc.) |
| `~/.claude/plugins/` | Installed plugins and their cached code |
| `~/.claude/agents/` | Custom subagent definitions (code-reviewer, security-reviewer) |
| `~/.claude/memory/` | Claude's persistent memory files (glossary, people, projects) |
| `~/.claude/settings.json` | Permissions, hooks, allowed tools, plugin toggles |
| `~/.claude/CLAUDE.md` | Your global context file (loaded in every session) |
| `~/.claude/mcp.json` | Global MCP server configuration (optional — MCPs can also be cloud-hosted) |

### Local-Only — Not Pushed to Git

**This directory is completely local.** It lives on your machine and does NOT get pushed to git or GitHub. It's personal to you. That's why it's in your home folder (`~`), not inside any project.

This is different from a **project-level** `.claude/` directory (e.g., `~/claude-code/nudesk-internal/.claude/`), which lives inside a git repo and CAN be committed to share team config. Here's the distinction:

| Location | Scope | Shared? |
| :---- | :---- | :---- |
| `~/.claude/` | Global — applies to ALL your projects | No — local to your machine |
| `<project>/.claude/` | Project — only applies when working in that directory | Yes — can be committed to git |

### How It Gets Created

Claude Code creates `~/.claude/` automatically the first time you run it. As you add commands, install plugins, and configure settings, the directory grows. You can also create files manually using `mkdir`, `nano`, or Claude Code itself.

---

## 0.6. Master Tip — Start with the Claude Code Setup Plugin

Before manually configuring everything in this guide, install the **claude-code-setup** plugin. This is the recommended first step for any new project — it analyzes your codebase and recommends which automations to set up.

### Install It

```shell
claude plugin install claude-code-setup
```

### Use It

Once installed, open Claude Code in any project and run:

```
/claude-automation-recommender
```

This will scan your codebase and recommend:

- Which **hooks** to add (auto-formatting, file protection)  
- Which **skills** would be useful (based on what the project does)  
- Which **MCP servers** to connect (based on the tools you use)  
- Which **plugins** to install (based on your workflow)

**Think of it as your setup assistant.** It gives you a guided starting point instead of configuring everything manually. Then use the rest of this guide to verify and fine-tune what was recommended.

---

## 1\. CLAUDE.md — Your Persistent Context

*Boris Tip \#3: "Invest in your CLAUDE.md. After every correction, end with: 'Update your CLAUDE.md so you don't make that mistake again.' Ruthlessly edit your CLAUDE.md over time."*

### Global \+ Project-Level

You should set up **both**:

| Level | Location | Purpose | Loaded When |
| :---- | :---- | :---- | :---- |
| **Global** | `~/.claude/CLAUDE.md` | Who you are, how you work, your preferences | Every session, regardless of project |
| **Project** | `CLAUDE.md` in a project's root folder | Project-specific context, tech stack, conventions | Only when working in that project directory |

**Start with the Global file.** It's the highest-leverage move because it follows you everywhere.

### Your Global CLAUDE.md

Create this file at `~/.claude/CLAUDE.md`:

```
# Global Context for Claude Code

## Who I Am
- Kenny Salas, CEO and Co-Founder of nuDesk LLC
- Non-technical founder learning to build with AI — explain technical decisions clearly
- I build AI agents, automations, and internal tools — not production software
- My partner Sean Salas is Co-Founder responsible for Prime Nexus and Advisory

## How I Work
- I use Claude Code for: building AI agents, automating workflows, HubSpot API integrations, creating internal tools, and prototyping
- Primary language: Python (when code is needed)
- I prefer simple, readable solutions over clever ones
- Always explain what you're doing and why before executing
- Ask before making destructive changes (deleting files, overwriting data)

## My Stack
- Claude (primary AI), Gemini (Google Workspace integration)
- HubSpot CRM (API access via HUBSPOT_ACCESS_TOKEN env var)
- Google Workspace (Gmail, Drive, Sheets, Docs)
- Google Cloud Platform (Cloud SQL PostgreSQL, Cloud Run)
- Asana for project management
- GitHub for version control
- n8n for third-party app orchestration layer

## Company Context
- nuDesk LLC — AI-Workforce Agency for Financial Services
- Anchor client: Champions Funding (mortgage lender)
- Key initiative: Prime Nexus (B2B lending marketplace with Live Oak Bank)
- We're SOC 2 Type I certified, pursuing Type II
- All work must be compliant — never handle PII carelessly

## Preferences
- When building scripts, include error handling and clear logging
- Use .env files for secrets — never hardcode API keys
- Write README files for anything I'll need to revisit later
- Commit frequently with descriptive messages
- If a task has multiple approaches, briefly explain tradeoffs before proceeding

## Git Workflow
- Commit and push at the end of every completed task — don't wait for me to ask
- Use descriptive commit messages following conventional commits (e.g., feat(hubspot): add partner export)
- Push to main for routine work; create a branch for anything experimental
- I don't need PRs for solo work — just commit and push directly
- Keep commits focused — one logical change per commit
```

### How to Create It

```shell
# Create the directory if it doesn't exist
mkdir -p ~/.claude

# Open the file for editing
nano ~/.claude/CLAUDE.md
```

Paste the content above, customize as needed, then save (`Ctrl+X`, then `Y`, then `Enter` in nano).

### Growing It Organically

Boris's \#1 CLAUDE.md insight: **use the `#` key inside Claude Code** to add things you find yourself repeating. Claude will prompt you to save it to your CLAUDE.md. Over time, this becomes your richest context file.

After every correction, tell Claude: **"Update your CLAUDE.md so you don't make that mistake again."** Boris says Claude is eerily good at writing rules for itself.

**Advanced pattern (for later):** One engineer tells Claude to maintain a `/notes` directory for every task/project, updated after every PR, and points CLAUDE.md at it. Consider this once you have multiple active projects.

### Project-Level CLAUDE.md Example

When you start a new project, create a `CLAUDE.md` in the project's root. You can also run `/init` and Claude Code will generate a starter file you can refine.

```
# HubSpot Prime Nexus Integration

## What This Project Does
Automates HubSpot CRM operations for Prime Nexus pipeline management.

## Setup
- Python 3.11+
- Dependencies: `pip install -r requirements.txt`
- Environment: Copy `.env.example` to `.env` and add your HUBSPOT_ACCESS_TOKEN

## Key Files
- `config.py` — HubSpot API configuration and pipeline IDs
- `partners.py` — Partner management functions
- `loans.py` — Loan pipeline operations

## How to Run
- `python main.py` — Run the main workflow
- `python test_connection.py` — Verify HubSpot API connection

## Important Notes
- Prime Nexus Partners pipeline ID: [fill in after setup]
- Prime Nexus Loans pipeline ID: [fill in after setup]
- Never delete pipeline stages that contain active deals
```

**Key principle:** Keep CLAUDE.md concise. Every line competes for attention in the context window. Research suggests LLMs can reliably follow \~150-200 instructions. Claude Code's system prompt already uses \~50, so your CLAUDE.md should focus on what's universally important. Don't try to replace linters with instructions — use deterministic tools for formatting and style enforcement.

**Team pattern:** For shared projects (anything you and Sean both touch), the project-level CLAUDE.md should be **checked into git**. This creates compounding memory — when anyone corrects Claude, the rule gets added to the file and benefits everyone. Boris calls this a "shared knowledge base that prevents the same feedback from being given twice."

---

## 1.5. Permissions Configuration

By default, Claude Code asks for your approval on every bash command and MCP action. This gets old fast — especially when you're telling Claude to do things you've already authorized. **Configuring permissions is essential to avoid approval fatigue.**

### Permission Modes

Claude Code has three permission modes you can cycle through with **Shift+Tab**:

| Mode | What Happens |
| :---- | :---- |
| **Plan only** | Claude plans but never executes (read-only) |
| **Default** | Claude asks before every tool use |
| **Accept Edits** | Claude executes file reads/writes without asking, but still asks for bash/MCP |

You can set the **default mode** in `~/.claude/settings.json` so you don't have to toggle it every session. Kenny's setup uses `"defaultMode": "acceptEdits"` — Claude can freely read and edit files, but still asks before running shell commands or MCP tools unless they're in the allow list.

### The `allowedTools` Config

The real power is in `~/.claude/settings.json` → `permissions.allow`. This is a list of tool patterns that Claude can execute **without asking for approval**. Here's a reference template based on Kenny's actual config:

```json
{
  "permissions": {
    "allow": [
      "Read",
      "Edit",
      "Write",
      "Glob",
      "Grep",
      "Search",
      "WebFetch",
      "WebSearch",
      "Bash(*)",

      "Bash(gws*)",          // Google Workspace — all gws CLI commands (replaces 85 old MCP tool entries)
       

      "mcp__claude_ai_Asana_2__asana_get_task",
      "mcp__claude_ai_Asana_2__asana_get_tasks",
      "mcp__claude_ai_Asana_2__asana_get_project",
      "mcp__claude_ai_Asana_2__asana_get_projects",
      "mcp__claude_ai_Asana_2__asana_get_projects_for_workspace",
      "mcp__claude_ai_Asana_2__asana_get_projects_for_team",
      "mcp__claude_ai_Asana_2__asana_get_project_sections",
      "mcp__claude_ai_Asana_2__asana_get_project_task_counts",
      "mcp__claude_ai_Asana_2__asana_search_tasks",
      "mcp__claude_ai_Asana_2__asana_get_stories_for_task",
      "mcp__claude_ai_Asana_2__asana_get_user",
      "mcp__claude_ai_Asana_2__asana_get_teams_for_workspace",
      "mcp__claude_ai_Asana_2__asana_get_teams_for_user",
      "mcp__claude_ai_Asana_2__asana_get_team_users",
      "mcp__claude_ai_Asana_2__asana_get_workspace_users",
      "mcp__claude_ai_Asana_2__asana_list_workspaces",
      "mcp__claude_ai_Asana_2__asana_get_goals",
      "mcp__claude_ai_Asana_2__asana_get_goal",
      "mcp__claude_ai_Asana_2__asana_get_parent_goals_for_goal",
      "mcp__claude_ai_Asana_2__asana_get_portfolios",
      "mcp__claude_ai_Asana_2__asana_get_portfolio",
      "mcp__claude_ai_Asana_2__asana_get_items_for_portfolio",
      "mcp__claude_ai_Asana_2__asana_get_time_periods",
      "mcp__claude_ai_Asana_2__asana_get_time_period",
      "mcp__claude_ai_Asana_2__asana_typeahead_search",
      "mcp__claude_ai_Asana_2__asana_get_allocations",
      "mcp__claude_ai_Asana_2__asana_get_project_statuses",
      "mcp__claude_ai_Asana_2__asana_get_project_status",
      "mcp__claude_ai_Asana_2__asana_get_attachments_for_object",
      "mcp__claude_ai_Asana_2__asana_get_attachment",
      "mcp__claude_ai_Asana_2__asana_update_task",
      "mcp__claude_ai_Asana_2__asana_create_task",
      "mcp__claude_ai_Asana_2__asana_create_task_story",
      "mcp__claude_ai_Asana_2__asana_set_parent_for_task",
      "mcp__claude_ai_Asana_2__asana_add_task_followers",
      "mcp__claude_ai_Asana_2__asana_remove_task_followers",
      "mcp__claude_ai_Asana_2__asana_set_task_dependencies",
      "mcp__claude_ai_Asana_2__asana_set_task_dependents",
      "mcp__claude_ai_Asana_2__asana_create_project",
      "mcp__claude_ai_Asana_2__asana_create_project_status",
      "mcp__claude_ai_Asana_2__asana_create_goal",
      "mcp__claude_ai_Asana_2__asana_update_goal",
      "mcp__claude_ai_Asana_2__asana_update_goal_metric",

      "mcp__claude_ai_Fireflies__fireflies_get_transcripts",
      "mcp__claude_ai_Fireflies__fireflies_get_transcript",
      "mcp__claude_ai_Fireflies__fireflies_get_summary",
      "mcp__claude_ai_Fireflies__fireflies_get_user",
      "mcp__claude_ai_Fireflies__fireflies_get_user_contacts",
      "mcp__claude_ai_Fireflies__fireflies_get_usergroups",
      "mcp__claude_ai_Fireflies__fireflies_search",
      "mcp__claude_ai_Fireflies__fireflies_fetch",

      "mcp__claude_ai_HubSpot__get_crm_objects",
      "mcp__claude_ai_HubSpot__search_crm_objects",
      "mcp__claude_ai_HubSpot__get_properties",
      "mcp__claude_ai_HubSpot__search_properties",
      "mcp__claude_ai_HubSpot__search_owners",
      "mcp__claude_ai_HubSpot__get_user_details",
      "mcp__claude_ai_HubSpot__manage_crm_objects",
      "mcp__claude_ai_HubSpot__submit_feedback",

      "mcp__claude_ai_Gamma__generate",
      "mcp__claude_ai_Gamma__get_generation_status",
      "mcp__claude_ai_Gamma__get_themes",
      "mcp__claude_ai_Gamma__get_folders",

      "mcp__context7__resolve-library-id",
      "mcp__context7__query-docs"
    ],
    "deny": [
      "Bash(rm -rf /*)",
      "Bash(rm -rf ~*)",
      "Bash(git push --force*)",
      "Bash(git push -f *)",
      "Bash(git reset --hard*)",
      "Bash(git checkout -- .)",
      "Bash(git clean -f*)",
      "Bash(git branch -D *)"
    ],
    "defaultMode": "acceptEdits"
  }
}
```

**What this does:**

- `"Read"`, `"Edit"`, `"Write"`, `"Glob"`, `"Grep"`, `"Search"`, `"WebFetch"`, `"WebSearch"` — Claude can freely work with files and search the web  
- `"Bash(*)"` — Claude can run any bash command (with deny-list exceptions for dangerous commands)  
- **Google Workspace (gws CLI) — All Google services via Agent Skills: Gmail, Calendar, Chat, Drive, Docs, Sheets, Slides, and more. Run as bash commands (e.g. gws gmail users messages list). Zero MCP token overhead — 42 skills loaded on-demand from \~/.claude/skills/gws-\***  
-   
- **Asana (35 tools)** — Full project management: tasks, projects, goals, portfolios, teams, users, time periods, statuses, attachments, dependencies, and followers  
- **Fireflies (8 tools)** — All transcript operations: search, fetch, summaries, users, contacts, and groups  
- **HubSpot (8 tools)** — CRM objects, properties, owners, search, management, and feedback  
- **Gamma (4 tools)** — Generate presentations, check status, browse themes and folders  
- **Context7 (2 tools)** — Live SDK documentation lookup  
- `"deny"` — Explicitly block dangerous commands (force push, hard reset, recursive delete)  
- `"defaultMode": "acceptEdits"` — Start every session in accept-edits mode

**The deny list is your safety net.** Even with `Bash(*)` allowed, these destructive patterns are always blocked.

### How to Edit It

```shell
nano ~/.claude/settings.json
```

Or tell Claude Code: "Add \[tool name\] to my allowed tools in settings.json" — it knows how to update its own config.

**Start conservative, expand over time.** Begin with file tools \+ a few MCP read operations. As you build trust, add more. The deny list protects you from the truly dangerous stuff regardless.

---

## 2\. Skills, Commands & Plugins

*Boris Tip \#4: "Create your own skills and commit them to git. Reuse across every project. If you do something more than once a day, turn it into a skill or command."*

### Slash Commands

Slash commands are single-file prompts you invoke with `/command-name`. They live in `~/.claude/commands/` (global) or `.claude/commands/` (project-level).

```shell
mkdir -p ~/.claude/commands
```

#### `Installed Commands`

Kenny's setup includes 10 global commands. Here's what each one does:

**Everyday Workflow:**

| Command | Purpose |
| :---- | :---- |
| `/review` | Check your work — reads git diff, flags security issues, verifies error handling |
| `/ship` | Commit and push — reviews diff, writes commit message, pushes to branch |
| `/techdebt` | End-of-session cleanup — finds TODOs, FIXMEs, dead code, uncommitted changes |

**Executive OS (CEO daily operations):**

| Command | Purpose |
| :---- | :---- |
| `/daily-plan` | Generate a prioritized daily plan from Asana, Calendar, Gmail, Fireflies, and HubSpot |
| `/session-closeout` | End-of-session wrap-up — capture tasks, update memory, note learnings |
| `/log-task` | Create an Asana task with smart project routing |
| `/run-tasks` | Execute today's Asana agent queue tasks |
| `/weekly-report` | Generate the KKS weekly executive progress report |

**From Plugins:**

| Command | Purpose |
| :---- | :---- |
| `/commit` | Create a git commit (from commit-commands plugin) |
| `/commit-push-pr` | Commit, push, and open a PR (from commit-commands plugin) |
| `/clean_gone` | Clean up local git branches deleted on remote (from commit-commands plugin) |

#### `Creating Your Own Commands`

Create a markdown file in `~/.claude/commands/`. Example:

```shell
# Create the /context-sync command
nano ~/.claude/commands/context-sync.md
```

```
Gather context from the last 7 days across my systems:

1. Check recent git commits and open PRs in the current repo
2. If Google Workspace MCP is connected: summarize relevant Chat and Gmail activity
3. If Asana MCP is connected: list my recent tasks and updates
4. Summarize the current state of the project

Output a concise "state of the world" briefing I can use to pick up where I left off.

Focus area (optional): $ARGUMENTS
```

**Usage:** `/context-sync` or `/context-sync Prime Nexus pipeline`

### Skills

Skills are folders with a `SKILL.md` file plus supporting resources. They live in `~/.claude/skills/` (personal) or `.claude/skills/` (project).

Boris's rule: **if you do something more than once a day, turn it into a skill or command.** Don't create skills preemptively — wait until you see the pattern.

#### `Installed Skills`

Kenny's setup includes 10 global skills:

**Active in Phase 1 (installed and recommended):**

| Skill | Trigger | What It Does |
| :---- | :---- | :---- |
| `nudesk-branding` | Mentions of branding, presentations, client materials | Applies nuDesk brand colors, typography, voice |
| `hubspot-ops` | Auto — triggers when writing HubSpot code | HubSpot API patterns and conventions |
| `new-script` | When scaffolding new Python scripts | Standard nuDesk patterns: logging, dotenv, error handling |
| `executive-planning` | "Plan my day", "help me prioritize" | CEO-level prioritization from Asana, Calendar, Gmail, etc. |
| `asana-agent` | "Run my tasks", "check my Asana queue" | Processes today's Asana tasks assigned as "Agent Queue" |

**Available (Phase 2 — add when needed):**

| Skill | What It Does |
| :---- | :---- |
| `memory-management` | Two-tier memory system — decodes shorthand, acronyms, internal language |
| `claude-md-improver` | Audits and improves CLAUDE.md files across repos |
| `gcp-deploy` | Cloud Run deployment automation |
| `keybindings-help` | Customize keyboard shortcuts and keybindings |
| `session-closeout` | End-of-session wrap-up (also available as a command) |

#### `Creating a Skill`

```shell
mkdir -p ~/.claude/skills/my-skill-name
nano ~/.claude/skills/my-skill-name/SKILL.md
```

Skills use YAML frontmatter for metadata:

```
---
name: my-skill-name
description: One sentence describing when to use this skill.
---

# Skill Title

Instructions for Claude when this skill activates...
```

**Commit skills to git.** This lets you reuse them across machines and share with Sean.

### Plugins — Extending Claude Code

Plugins are community and Anthropic-maintained extensions that add commands, skills, and behaviors to Claude Code. Think of them as **pre-built packages** you can install instead of writing from scratch.

#### `How to Explore the Plugin Marketplace`

```shell
# List all available plugins
claude plugin list

# Search for a specific type
claude plugin search "commit"

# Install a plugin
claude plugin install <plugin-name>
```

#### `Installed Plugins`

Kenny's setup includes 6 plugins:

| Plugin | What It Adds |
| :---- | :---- |
| `claude-code-setup` | `/claude-automation-recommender` — analyzes codebases and recommends automations |
| `commit-commands` | `/commit`, `/commit-push-pr`, `/clean_gone` — git workflow commands |
| `security-guidance` | Security best practices and guidance during development |
| `asana` | Enhanced Asana integration patterns |
| `skill-creator` | Tools to create, modify, test, and benchmark custom skills |
| `ralph-loop` | Infinite recursion protection — prevents Claude from looping endlessly |

#### `How to Install`

```shell
# Install all recommended plugins
claude plugin install claude-code-setup
claude plugin install commit-commands
claude plugin install security-guidance
claude plugin install asana

# Verify installed plugins
claude plugin list --installed
```

**Start with `claude-code-setup`, `commit-commands`, and `security-guidance`.** These three give you the most value immediately. Add others as your workflow grows.

---

## 3\. Hooks — Automated Guardrails

Hooks are shell commands that run automatically before or after Claude uses a tool. They're your **automated guardrails** — enforcing rules without you having to remember them.

### How Hooks Work

| Hook Type | When It Runs | Use Case |
| :---- | :---- | :---- |
| `PreToolUse` | Before Claude executes a tool | Block dangerous actions, validate inputs |
| `PostToolUse` | After Claude executes a tool | Auto-format code, run linters, log activity |

Hooks are configured in `~/.claude/settings.json` under the `"hooks"` key.

### Installed Hooks

**Hook 1: .env File Blocker (PreToolUse)**

Prevents Claude from editing any file that starts with `.env` — protecting your secrets from accidental modification.

```json
{
  "matcher": "Edit|Write",
  "hooks": [
    {
      "type": "command",
      "command": "echo \"$CLAUDE_TOOL_INPUT\" | python3 -c \"import sys,json,os; f=json.load(sys.stdin).get('file_path',''); name=os.path.basename(f); sys.exit(2) if name.startswith('.env') else sys.exit(0)\"",
      "statusMessage": "Checking for protected file edits..."
    }
  ]
}
```

**How it works:** Before any Edit or Write, this hook checks if the target file starts with `.env`. If so, it blocks the action (exit code 2). This means Claude can never accidentally overwrite your API keys or secrets.

**Hook 2: Ruff Auto-Formatter (PostToolUse)**

Automatically formats Python files after every edit using Ruff (a fast Python formatter/linter).

```json
{
  "matcher": "Edit|Write",
  "hooks": [
    {
      "type": "command",
      "command": "echo \"$CLAUDE_TOOL_INPUT\" | python3 -c \"import sys,json,subprocess; f=json.load(sys.stdin).get('file_path',''); [subprocess.run(['python3','-m','ruff',cmd]+([f] if cmd=='format' else ['--fix',f]),capture_output=True) for cmd in ['format','check']] if f.endswith('.py') else None\"",
      "statusMessage": "Formatting Python file with Ruff..."
    }
  ]
}
```

**How it works:** After any Edit or Write to a `.py` file, this hook runs `ruff format` and `ruff check --fix` on that file. Your Python code stays consistently formatted without you thinking about it.

**Prerequisite:** Install Ruff first: `pip install ruff`

### Gotchas with Hooks

These are hard-won lessons — save yourself the debugging:

1. **In hook commands, use `echo "$CLAUDE_TOOL_INPUT" | python3`** — NOT the `<<<` heredoc syntax. The heredoc approach fails silently in Claude Code's hook execution environment.  
     
2. **A broken PreToolUse hook on Edit|Write blocks its own fix.** If you write a buggy hook that matches Edit or Write, Claude can't fix the settings.json file because the hook blocks the edit. The escape hatch: use Bash to rewrite settings.json directly (`echo '...' > ~/.claude/settings.json`).  
     
3. **For .env protection, check `os.path.basename()` only** — don't try to match `.env` anywhere in the file content, or you'll get false positives when editing files that mention environment variables.

---

## 4\. GitHub Setup

This is straightforward and high-value.

### Install the GitHub App

Inside Claude Code, run:

```
/install-github-app
```

This enables:

- Mentioning `@claude` on issues and PRs  
- Claude Code can create branches, commits, and PRs for you  
- Full git history access for context

### Core Git Workflow

Tell Claude Code:

"Create a new branch, make the changes, commit, and open a PR"

Claude Code understands git natively. The `/ship` command wraps this up neatly.

### Automated Feedback via @Claude

Once the GitHub app is installed, you can mention `@claude` directly on PRs and issues to trigger automated fixes. Boris's workflow:

- A PR has a small issue → comment `@claude fix the error handling here` → Claude opens a follow-up commit  
- Notice a recurring mistake → comment `@claude update CLAUDE.md to prevent this pattern` → Claude adds the rule itself

This creates a continuous improvement loop: every code review finding becomes a permanent rule in your CLAUDE.md, so Claude never makes the same mistake twice. Boris says this alone eliminates most repeated feedback.

### Work Locally, Let Claude Handle Git

**Do all your work on your local Mac.** Claude Code runs in your terminal, on your local filesystem — that's where it's fastest. GitHub is your backup and version history, not your workspace.

The CLAUDE.md template above includes a `Git Workflow` section that tells Claude to commit and push at the end of every completed task automatically. This means you don't think about git at all during normal work. You say "build me X," Claude builds it, and when it's done it commits and pushes without you asking.

**Three levels of git automation:**

| Level | How It Works | When to Use |
| :---- | :---- | :---- |
| **Auto (recommended)** | CLAUDE.md instruction: "Commit and push at the end of every completed task" | Default for all work |
| **Manual checkpoint** | Type `/ship` when you want to commit mid-session | When iterating and you want a save point |
| **Auto-commit hook** | Claude Code hook that commits after every tool execution | Overkill — creates noisy history. Skip this. |

One habit to keep: `git push` before closing your laptop for the day, just as a backup. But if Claude is pushing at the end of every task, you're covered 95% of the time.

---

## 5\. Parallel Work

*Boris Tip \#1: "Spin up 3-5 git worktrees at once, each running its own Claude session in parallel. It's the single biggest productivity unlock, and the top tip from the team."*

### Level 1: Multiple Terminal Tabs (Start Here)

The easiest approach — no special tools needed:

1. Open 2-3 terminal tabs (Cmd+T on Mac Terminal)  
2. Each tab runs a separate `claude` session in a different project directory  
3. Each session is completely independent — its own context window (\~200K tokens), its own CLAUDE.md

**Example parallel workflow:**

- Tab 1: Claude Code working on HubSpot integration  
- Tab 2: Claude Code building a training document  
- Tab 3: Claude Code analyzing data

### Level 2: Git Worktrees (Boris's Top Recommendation)

Git worktrees let you have multiple copies of the same repo checked out at once, each on a different branch. This is how Boris's team works — and he says it's the **\#1 productivity tip**.

```shell
# Create a worktree from your main repo
$ git worktree add .claude/worktrees/feature-a origin/main

# Navigate to it and start Claude Code
$ cd .claude/worktrees/feature-a && claude
```

**Why this matters:** You can have one Claude session building a new feature while another fixes a bug — both in the same repo, on separate branches, without conflicts.

**Pro tips from Boris's team:**

- Name your worktrees descriptively so you can find them easily  
- Set up shell aliases (`za`, `zb`, `zc`) to hop between worktrees in one keystroke  
- Some people have a dedicated "analysis" worktree that's only for reading logs and querying data

**When to start using this:** Once you find yourself wanting to work on two things in the same project at the same time. Don't force it on Day 1 — tabs are fine until then.

### Terminal Tips

*Boris Tip \#7: "The team loves Ghostty\! Use /statusline to customize your status bar to always show context usage and current git branch."*

- **Ghostty** is the terminal Boris's team recommends (synchronized rendering, 24-bit color, proper unicode). See *"Choosing Your Environment"* below for when to adopt it — it's a quality-of-life upgrade to layer in after your first month, not a Day 1 requirement.  
- Use `/statusline` to show context usage and git branch at all times.  
- Color-code your terminal tabs per task — one color per worktree/project.  
- Use `Ctrl+B` to send subagent tasks to background (more on subagents below).

### Voice Input

Boris specifically recommends this: **"You speak 3x faster than you type, and your prompts get way more detailed as a result."**

- On macOS: hit `fn` twice to toggle dictation  
- Install **Wispr Flow** for more natural voice input  
- Especially useful when explaining complex business requirements

### Mobile App: Kick Off Sessions On the Go

Boris starts multiple Claude Code sessions from his phone in the mornings — kicks off tasks, then checks back later from his desk. This is a natural fit for your workflow: queue up 2-3 tasks on your commute or between meetings, then review results when you're ready. Use it for tasks that don't need real-time attention (data pulls, report generation, code refactoring).

---

## 6\. MCP Servers & External Tools

MCP (Model Context Protocol) lets Claude Code connect to external services — reading your email, checking your calendar, managing Asana tasks, querying HubSpot, and more.

*Boris Tip \#5: "Enable MCP servers, then paste a bug thread into Claude and just say 'fix.' Zero context switching."*

### Installed MCP Servers

Kenny's setup includes 5 MCP servers. Google Workspace now uses the gws CLI as Agent Skills — not an MCP server. Here's what's available:

| Server | What It Connects | Example Use |
| :---- | :---- | :---- |
| **Google Workspace** | Gmail, Calendar, Google Chat | "Check my email for anything from Evan today" |
| **Google Drive** | Docs, Sheets, Slides, file management | "Read the Champions SOP from Google Drive" |
| **Asana** | Tasks, projects, goals, portfolios | "What's overdue in my Asana this week?" |
| **HubSpot** | CRM objects, deals, contacts, properties | "Pull all Prime Nexus deals created this month" |
| **Fireflies** | Meeting transcripts and summaries | "Summarize my call with Kristie from yesterday" |
| **Gamma** | AI-generated presentations, documents | "Create a deck about nuDesk's Q1 results" |
| **context7** | Live SDK/library documentation | "Look up the latest HubSpot API docs for deals" |

### How MCP Servers Are Configured

There are two ways to connect MCP servers:

**Cloud-hosted MCPs (recommended for most):** These are managed by Anthropic and the service providers. You authenticate once through Claude Code and they just work. Asana, HubSpot, Fireflies, and Gamma all use this approach — no local config needed.

**Local MCP config: For self-hosted or custom MCPs, you add them to \~/.claude/mcp.json. context7 uses this approach:**

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"]
    },
    // Google Drive removed — use gws CLI instead (see below)
      "command": "npx",
      
    }
  }
}
```

### Setup Order

1. **gws CLI (Google Workspace) —** Set up first. Install gws and its Agent Skills to give Claude access to all Google services: Gmail, Calendar, Chat, Drive, Docs, Sheets, and more. No MCP server needed — zero token overhead. Installation steps:  
   - **brew install google/gws/gws**  
   - **gws auth setup**  
   - **npx skills add \--yes \--global [https://github.com/googleworkspace/cli](https://github.com/googleworkspace/cli)**  
   - **This installs 42 Agent Skills into \~/.claude/skills/gws-\* that Claude invokes on-demand using bash commands like gws gmail users messages list.**  
2. **Asana** — Connect second. Essential for the Executive OS commands (`/daily-plan`, `/run-tasks`).  
3. **HubSpot** — Connect third. Required for any CRM operations.  
4. **Fireflies — Connect fourth. Meeting transcript access.**  
5. **context7 — Connect fifth. Live documentation lookup (requires Node.js: brew install node).**  
6. **Gamma** — Connect last. Presentation generation — nice to have.

### Important Note on Context Window

**Don't enable all MCPs at once. Each MCP server adds tool definitions to your context window. Your 200K context window can shrink significantly with too many tools enabled. Note: Google Workspace avoids this entirely by using Agent Skills (loaded on-demand) instead of an MCP server — a key advantage of the gws CLI approach. For other MCPs, use disabledMcpServers in a project's .claude/mcp.json to turn off unused ones per project.**

### Web Fetch (Built-In)

No setup required. Claude Code can fetch web content directly:

"Fetch the HubSpot API documentation for deals endpoints and tell me how to create custom properties"

---

# Part 2: Working Patterns

These aren't things you configure — they're techniques to use every day.

---

## 7\. Plan Mode

*Boris Tip \#2: "Start every complex task in plan mode. Pour your energy into the plan so Claude can 1-shot the implementation."*

### The Built-In Plan Mode Toggle

Claude Code has a **built-in plan mode** you should use constantly. Toggle it with:

**Shift+Tab** to cycle between modes

When plan mode is on, Claude Code will think through the approach *before* writing any code. This is different from the `/plan` slash command — it's a native feature that changes how Claude processes your request.

### Boris's Plan Mode Patterns

**Pattern 1: Plan → Review → Build** One of Boris's team members has Claude write the plan, then spins up a *second* Claude session to review it as a "staff engineer" before approving. For you, a simpler version: read the plan yourself, push back on anything that seems overcomplicated, *then* approve.

**Pattern 2: Re-Plan When Things Go Sideways** The moment something goes wrong, **don't keep pushing.** Switch back to plan mode and re-plan from the current state. Boris says this is a critical habit — the instinct is to keep debugging, but starting from a fresh plan is usually faster.

**Pattern 3: Plan for Verification Too** Tell Claude to enter plan mode not just for the build, but for the *verification step* — how will we know this actually works?

### The Payoff: Auto-Accept Mode

This is the full workflow Boris describes:

1. **Toggle plan mode on** (Shift+Tab)  
2. **Go back and forth** with Claude until the plan is solid — challenge it, ask questions, refine  
3. **Once the plan is right**, toggle to **auto-accept edits mode** — Claude executes without asking for confirmation on each change

Boris says with Opus 4.6, once the plan is good, the model executes "almost perfectly." The planning phase is where you invest your energy. Execution becomes hands-off.

**To toggle auto-accept:** Press Shift+Tab to cycle through modes. The modes are: plan only → plan \+ auto-accept → normal (confirm each edit).

### When to Use Plan Mode

- Any task that touches more than 1-2 files  
- When you're not sure about the right approach  
- Before any refactoring  
- Before anything that modifies production data or APIs

---

## 8\. Prompting Techniques

*Boris Tip \#6: "Level up your prompting. Challenge Claude. Say 'Grill me on these changes and don't make a PR until I pass your test.'"*

These are specific phrases Boris recommends. Bookmark them.

### Challenge Prompts

**When Claude gives you a mediocre solution:**

"Knowing everything you know now, scrap this and implement the elegant solution"

This is a game-changer. Instead of iterating on a bad foundation, it tells Claude to start fresh with everything it learned from the first attempt.

**When you want Claude to be your reviewer:**

"Grill me on these changes and don't make a PR until I pass your test"

This flips the dynamic — Claude becomes the quality gatekeeper.

**When you want verification:**

"Prove to me this works"

Claude will diff the main branch against the feature branch and walk through the evidence.

### Writing Better Specs

Boris's \#1 prompting tip beyond challenge phrases: **write detailed specs and reduce ambiguity before handing work off.** The more specific you are, the better the output. This is where voice dictation shines — speak your full business context naturally instead of typing terse instructions.

### For Your Context

As a non-technical founder, the most useful prompt patterns are:

- **Before building:** "Explain what you're about to do in plain English, then wait for my approval"  
- **After building:** "Walk me through what you built and why you made each decision"  
- **When stuck:** "What are 3 different approaches to solve this? List tradeoffs for each"  
- **When reviewing:** "Assume I don't understand the code. Explain what this does and flag anything risky"

---

## 9\. Give Claude a Way to Verify Its Output

*Boris: "This dramatically improves results. It's like giving a painter their eyesight back."*

This is arguably Boris's most important quality tip. Whenever possible, don't just tell Claude to build something — **give it a way to check its own work.** The self-correction loop is where output quality jumps from "okay" to "reliable."

### Verification Methods

**Run tests:**

"Write the function, then write a test for it and run the test. Keep fixing until the test passes."

**Start a server:**

"Start the dev server and verify the endpoint returns the expected response."

**Check output visually:**

"Generate the report, then open it and confirm the formatting looks right."

**Cross-reference data:**

"Pull the HubSpot deals, then verify the total matches what I see in the dashboard."

### The Principle

If Claude can *see* the result of its work — through tests, logs, server output, or any kind of feedback — it will catch and fix its own mistakes. Without verification, you're relying on it getting things right in one shot. With verification, you're giving it a correction loop.

For your context as a non-technical founder: always ask Claude to **prove it works**, not just tell you it works. "Run it and show me the output" is a better habit than "looks good, ship it."

---

## 10\. Subagents

*Boris Tip \#8: "Append 'use subagents' to any request where you want Claude to throw more compute at the problem."*

This was in "Phase 2" in the previous guide — but Boris makes it clear this is a **zero-effort, immediate technique**. You don't configure anything. You just say it.

### How to Use Subagents

Simply add "use subagents" to your prompt:

"Explore this codebase and explain how it's structured. Use subagents."

Claude will launch multiple parallel agents — each investigating a different aspect (entry points, components, tools, state management, tests). Results get combined into a single answer.

**Use `Ctrl+O` to expand/collapse subagent details.** **Use `Ctrl+B` to send subagent work to the background.**

### When Subagents Help Most

- **Exploring unfamiliar code:** "Use 5 subagents to explore this codebase"  
- **Research tasks:** "Research these 3 API options in parallel. Use subagents."  
- **Multi-file changes:** "Update all files that reference the old pipeline ID. Use subagents."  
- **Complex debugging:** "Trace this error through the codebase. Use subagents."

### When to Skip Subagents

- Simple single-file edits  
- Questions that only need one piece of information  
- When you're almost out of context window (each subagent uses tokens)

---

## 11\. Bug Fixing Workflow

*Boris Tip \#5: "Claude fixes most bugs by itself. Paste a bug thread into Claude and just say 'fix.' Zero context switching."*

### The Pattern

1. **Ensure gws CLI skills are installed (see Section 6 above) — nuDesk uses Google Chat, not Slack**  
2. When someone reports a bug in Chat, copy the relevant message  
3. Paste it into Claude Code and say:

"fix this bug: \[paste the message/error\]"

Claude will understand the bug, trace the issue in your codebase, and implement a fix. Zero manual context transfer.

### Other Bug Fixing Patterns

**From error messages:**

"Here's the error: \[paste error\]. Fix it."

**From CI/CD:**

"Go fix the failing CI tests."

Don't micromanage *how*. Just point Claude at the problem and let it work.

**From logs:**

Point Claude at docker logs to troubleshoot distributed systems — Boris says it's "surprisingly capable at this."

---

## 12\. Data & Analytics

*Boris Tip \#9: "Ask Claude Code to use the 'bq' CLI to pull and analyze metrics on the fly. This works for any database that has a CLI, MCP, or API."*

Boris says he hasn't written a line of SQL in 6+ months. The same principle applies to your stack.

### Your Data Sources

Instead of BigQuery, your equivalent tools are:

**HubSpot (via MCP):**

"Pull all Prime Nexus deals from HubSpot that were created this month. Show me conversion rate by stage."

"How many partners have submitted deals in the last 30 days? Break it down by week."

**Google Sheets (via gws CLI):**

"Analyze this spreadsheet and tell me which account managers have the highest close rate"

**Asana (via MCP):**

"How many tasks are overdue across all nuDesk projects? Who owns them?"

**Fireflies (via MCP):**

"Pull my meeting transcripts from this week and summarize the key decisions made"

### The Key Insight

Any system with a CLI, API, or MCP connection can be queried through Claude Code. You don't need to learn SQL, HubSpot's query syntax, or Sheets formulas. Just describe what you want in plain English. Claude Code handles the translation.

---

## 13\. Learning Mode

*Boris Tip \#10: "A few tips from the team to use Claude Code for learning: Enable the 'Explanatory' or 'Learning' output style in /config to have Claude explain the why behind its changes."*

**This is high-priority for you as a non-technical founder.** It transforms Claude from "tool that does things" to "mentor that teaches while doing things."

### Enable Learning Output Style

Run `/config` in Claude Code and set the output style to **"Explanatory"** or **"Learning"**. This makes Claude explain the *reasoning* behind every change, not just make the change silently.

### Learning Techniques from Boris's Team

**Visual HTML presentations:**

"Generate a visual HTML presentation explaining how this HubSpot integration works"

Boris says Claude makes surprisingly good slides for explaining unfamiliar code or architectures.

**ASCII diagrams:**

"Draw an ASCII diagram showing how data flows from Prime Nexus partners through the HubSpot pipeline to Live Oak"

Great for understanding system architecture without reading code.

**Spaced-repetition learning:**

"Build a spaced-repetition learning exercise: I'll explain how \[concept\] works, you ask follow-ups to fill gaps, then store the result"

This turns Claude into an active tutor instead of a passive tool.

### For Your Daily Learning

Make it a habit: after Claude builds something, ask:

- "Why did you structure it that way?"  
- "What would break if I changed X?"  
- "What's the most important thing I should understand about this?"

This compounds. In 6 months, your technical understanding will be dramatically sharper.

---

# Quick Start Checklist

Your implementation order, from highest to lowest impact:

| \# | Action | Time | Impact |
| :---- | :---- | :---- | :---- |
| 1 | Set up `~/claude-code/` workspace with project folders | 10 min | High |
| 2 | Create `~/.claude/CLAUDE.md` (global) | 15 min | High |
| 3 | Set Opus 4.6 as default model | 1 min | High |
| 4 | Install `claude-code-setup` plugin and run `/claude-automation-recommender` | 5 min | High |
| 5 | Configure permissions in `~/.claude/settings.json` (allowedTools \+ defaultMode) | 10 min | High |
| 6 | Install core plugins: `commit-commands`, `security-guidance` | 5 min | High |
| 7 | Enable Learning output style (`/config`) | 2 min | High |
| 8 | Install gws CLI \+ Agent Skills (Google Workspace); connect Asana and HubSpot MCP servers | 15 min | High |
| 9 | Create `/review` and `/ship` slash commands (or use the ones from commit-commands) | 10 min | Medium |
| 10 | Run `/install-github-app` in Claude Code | 5 min | Medium |
| 11 | Learn plan mode → auto-accept flow (Shift+Tab) | 2 min | Medium |
| 12 | Adopt verification habit ("prove it works") | 0 min | Medium |
| 13 | Set up Wispr Flow for voice prompts | 10 min | Medium |
| 14 | Add MCP servers: Fireflies, context7, Gamma | 10 min | Medium |
| 15 | Create project-level CLAUDE.md for first project | 10 min | Medium |
| 16 | Try subagents on your next complex task | 0 min | Medium |
| 17 | Try parallel sessions (2 tabs or mobile) | 5 min | Low |
| 18 | Add Executive OS commands (`/daily-plan`, `/session-closeout`, etc.) | 15 min | Low |
| 19 | Add nuDesk branding skill | 15 min | Low |

**Core setup time: \~50 minutes** (items 1-8) **Full setup time: \~145 minutes** (all items)

---

# Boris's Tips — Quick Reference Card

*Combined from X post (10 Tips) and interview. Principles, not an exhaustive list.*

| \# | Principle | One-Liner for Daily Use |
| :---- | :---- | :---- |
| 1 | **Do more in parallel** | Open 2-3 tabs/worktrees \+ kick off sessions from mobile. Check back later. |
| 2 | **Plan → Auto-accept** | Shift+Tab for plan mode. Invest in the plan. Once solid, toggle auto-accept and let Claude execute. |
| 3 | **Invest in CLAUDE.md** | Use `#` to add rules. After corrections: "Update CLAUDE.md so you don't make that mistake again." Check into git for team sharing. |
| 4 | **Create skills & commands** | If you do it more than once a day, make it a slash command or skill. Commit to git. |
| 5 | **Claude fixes bugs** | Paste a bug thread \+ say "fix." Or: "Go fix the failing tests." |
| 6 | **Level up prompting** | "Scrap this and implement the elegant solution." "Grill me on these changes." |
| 7 | **Give Claude verification** | Always let Claude run tests, start servers, or check output. Self-correction \> one-shot. |
| 8 | **Use Opus 4.6** | Less steering, better tool use, actually cheaper in total tokens. Set as default. |
| 9 | **Use subagents** | Append "use subagents" to any prompt for more compute / parallel exploration. |
| 10 | **Data & analytics** | Query HubSpot, Sheets, Asana in plain English via MCP/API — no SQL needed. |
| 11 | **Learning mode** | `/config` → Explanatory output. Ask for HTML presentations, ASCII diagrams. |
| 12 | **Terminal setup** | Voice dictation (fn+fn), `/statusline`, color-code tabs, try Ghostty. |

---

# Choosing Your Environment

A common question: should you use Claude Code in a plain terminal, an IDE like Cursor or VSCode, or a tool like Conductor? Here's the decision framework.

### Where to Run Claude Code

| Environment | What It Is | Best For | Recommendation |
| :---- | :---- | :---- | :---- |
| **Terminal.app / iTerm2** | Mac's built-in terminal or popular alternative | Direct, simple, no extra tools | **Start here** |
| **Ghostty** | GPU-accelerated terminal by HashiCorp founder | Heavy terminal users who want speed \+ visual polish | Layer in after 1-2 months |
| **VSCode \+ Claude Extension** | Code editor with visual file browser \+ Claude Code in integrated terminal | People who want to visually browse files while working | Try if terminal feels disorienting |
| **Cursor** | AI-powered IDE with its own built-in AI | Professional developers writing code line-by-line | **Skip** — different workflow than yours |
| **Conductor** | Visual dashboard managing parallel Claude Code agents in git worktrees | Developers running 3-5+ agents on the same codebase simultaneously | Layer in after 2-3 months, if ever |

**Why terminal over an IDE:** You're not sitting in an editor writing code line by line. You're giving Claude high-level instructions and letting it build. A plain terminal is the cleanest interface for that interaction. It's what Boris and his team use, and what this guide is built around.

**Why not Cursor:** Cursor's AI competes with Claude Code rather than complementing it. You'd pay $20/mo extra for an AI assistant you don't need alongside the one you already have. Cursor is excellent for developers who want inline code completion as they type — but that's not your workflow.

### When to Adopt Each Tool

**Ghostty** — Adopt when:

- You're spending 1+ hours daily in the terminal  
- You notice rendering lag or poor font display  
- You want split panes and visual customization  
- Boris's team loves it, but they're in terminal 8+ hours/day. It's a quality-of-life upgrade, not a capability unlock.

**VSCode \+ Claude Extension** — Try if:

- You find yourself constantly running `ls` and `cat` just to understand your project structure  
- You want to click through folders visually while Claude works in a panel beside them  
- It's still Claude Code under the hood — same commands, same CLAUDE.md, same everything

**Conductor** — Adopt when:

- You're routinely wishing you could work on 3+ things in the same repo simultaneously  
- The manual git worktree setup feels too tedious  
- You want a visual dashboard showing all active agents, diffs, and test results  
- **Note:** No SOC 2 certification. Pilot on internal projects only. Requires granting GitHub access.

**General rule:** Don't pre-optimize your toolchain. Start with the simplest setup (plain terminal). Layer in tools only when you feel specific friction that the tool solves. You'll know because you'll feel the pain.

---

# Phase 2: Add When Ready

Once you're comfortable with Part 1, these are the next layer of customizations to adopt gradually:

### Memory System

A two-tier architecture: CLAUDE.md acts as your "hot cache" (loaded every session), while `~/.claude/memory/` stores detailed reference files (glossary, people, project context) that Claude reads on demand. This lets Claude understand your shorthand, acronyms, and internal language — like a colleague who's been at the company for months.

### Custom Subagents

Define specialized reviewers in `~/.claude/agents/` that run on cheaper/faster models. Kenny's setup includes:

- **code-reviewer** — Reviews Python code for quality, error handling, and nuDesk conventions (runs on Sonnet)  
- **security-reviewer** — Scans for security vulnerabilities, PII exposure, and SOC 2 compliance issues (runs on Sonnet)

These get invoked automatically when you run `/review` or can be triggered by Claude during PR workflows.

### Additional Plugins

- **skill-creator** — Create, modify, test, and benchmark custom skills with evals  
- **ralph-loop** — Infinite recursion protection (prevents Claude from looping endlessly on failed tasks)

### Additional Skills

- **gcp-deploy** — Cloud Run deployment automation  
- **claude-md-improver** — Audit and improve CLAUDE.md files across repos  
- **keybindings-help** — Customize keyboard shortcuts and keybindings  
- **memory-management** — Full memory system with shorthand decoding

### Git Worktrees

Graduate from tabs to worktrees when you're working on 2+ things in the same repo regularly. See Section 5 for details.

### Custom MCP Servers

Build your own connectors for nuDesk-specific systems that don't have pre-built MCPs.

### CI/CD Integration

Run Claude Code in automated pipelines via `claude -p` for headless execution. Useful for automated code reviews, test generation, and deployment workflows.

### Analytics-Engineer Agents

Build agents that write queries, review data, and test changes in dev (Boris's team tip).

---

## Resources

- [Anthropic: Claude Code Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices)  
- [Anthropic: Using CLAUDE.md Files](https://claude.com/blog/using-claude-md-files)  
- [Claude Code Skills Documentation](https://code.claude.com/docs/en/skills)  
- [Claude Code Hooks](https://code.claude.com/docs/en/hooks)  
- [Claude Code Common Workflows](https://code.claude.com/docs/en/common-workflows)  
- [Boris's 30-Minute Tutorial](https://www.youtube.com/live/6eBSHbLKuN0)  
- [Boris's 10 Tips Thread](https://x.com/bcherny/status/1885428510880551197) (Jan 31, 2026\)  
- [HumanLayer: Writing Good CLAUDE.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md)

---

*This guide is designed to evolve. As you use Claude Code more, update your CLAUDE.md files and add commands/skills based on patterns you discover. Remember: "Update your CLAUDE.md so you don't make that mistake again."*  
