---
description: Audit Executive OS installation — check config, skills, plugins, and propose updates
allowed-tools: Read, Grep, Glob, Bash
---

Scan the Executive OS installation and produce a health report with actionable recommendations.

## 1. Configuration Health

### CLAUDE.md
Read `~/.claude/CLAUDE.md` and check for these required sections:
- [ ] Who I Am (name, role, email)
- [ ] My Stack (tools and platforms)
- [ ] Platform Reference Docs (table of constraint docs)
- [ ] Working Memory (People, Terms, Clients tables)
- [ ] Priorities (with `last_updated` date)
- [ ] Integration Config (email, HubSpot owner ID)

For each missing section, provide a template snippet the user can add.

If Priorities `last_updated` is more than 7 days ago, flag it as a warning.

### Asana Config
Check if `~/.claude/memory/asana-config.md` exists and has:
- [ ] Workspace GID populated (not placeholder)
- [ ] At least one user GID populated
- [ ] Custom field GIDs populated (Task Progress, Type, Priority)
- [ ] At least one project in the routing table
- [ ] Scheduled Task Automation section with paths filled in

### Memory Directory
Verify these paths exist:
```
~/.claude/memory/
~/.claude/memory/people/
~/.claude/memory/projects/
~/.claude/memory/context/
```

If missing, provide the `mkdir -p` command to create them.

## 2. Plugin & Skill Currency

### Installed Plugins
Read `~/.claude/plugins/installed_plugins.json` and check:
- [ ] `executive-os` plugin is installed
- [ ] Version matches the latest in `.claude-plugin/plugin.json` from the repo at `~/Projects/executive-os-plugin/`

### Recommended Plugins
Check if these optional but recommended plugins are installed:
- `pr-review-toolkit` — test quality analysis
- `test-generator` — automated test generation (if evaluated and adopted)

### MCP Servers
Check `~/.claude.json` and workspace `settings.json` for configured MCP servers:
- [ ] **Asana** — Required
- [ ] **Google Workspace** — Recommended (Gmail, Calendar)
- [ ] **Google Drive** — Optional
- [ ] **Fireflies** — Optional
- [ ] **HubSpot** — Optional

Note which are configured vs. missing.

## 3. Compliance Controls

### .env Blocker Hook
Read the workspace `settings.json` (check both `~/.claude/settings.json` and the project-level `.claude/settings.json`):
- [ ] PreToolUse hook exists with `Edit|Write` matcher
- [ ] Hook blocks `.env` file edits (but allows `.env.example`)

If missing, provide the hook JSON from `templates/hooks-settings.json.template`.

### .gitignore Coverage
In the current project directory, check `.gitignore` for:
- [ ] `credentials/` pattern
- [ ] `.env` pattern
- [ ] `*.pickle` pattern
- [ ] `*.key` or `*.pem` pattern

Reference the SOC 2 compliance skill for the full checklist.

## 4. Platform References

Check `~/Projects/system_docs/platform_references/` exists and list what's present:

**Expected docs:**
| Platform | File | Status |
|----------|------|--------|
| n8n Cloud | `n8n-cloud.md` | ? |
| GCP Cloud Run | `gcp-cloud-run.md` | ? |
| Google APIs | `google-apis.md` | ? |
| HubSpot API | `hubspot-api.md` | ? |
| HubSpot Conventions | `hubspot-conventions.md` | ? |
| Apollo API | `apollo-api.md` | ? |

Flag any platforms referenced in CLAUDE.md "My Stack" section that don't have a corresponding reference doc.

## 5. File Organization

- [ ] Plugin repo cloned at `~/Projects/executive-os-plugin/`
- [ ] `~/Projects/system_docs/` exists with expected subdirectories
- [ ] No orphaned config files (e.g., old `.claude/commands/` files that duplicate plugin commands)

## Output

Present results as a health report:

```
EXECUTIVE OS HEALTH CHECK — [Date]

PASSED
- [Item]: [Brief note]
- ...

WARNINGS
- [Item]: [What's wrong] — [How to fix]
- ...

ACTION ITEMS
- [ ] [Specific fix with command or template snippet]
- [ ] ...
```

Keep the report scannable. Group by severity. Include specific fix commands for every action item — the user should be able to copy-paste to resolve.
