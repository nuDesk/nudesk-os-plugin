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
- `pr-review-toolkit@claude-plugins-official` — PR review suite with 6 agents: code-reviewer, pr-test-analyzer, silent-failure-hunter, code-simplifier, comment-analyzer, type-design-analyzer. Provides `/pr-review-toolkit:review-pr` command.

### Required Skills
Check that these skills are installed at `~/.claude/skills/`:
- [ ] `srd-generator` — `~/.claude/skills/srd-generator/SKILL.md`
- [ ] `ai-solution-architect` — `~/.claude/skills/ai-solution-architect/SKILL.md`
- [ ] `nudesk-brand-styling` — `~/.claude/skills/nudesk-brand-styling/SKILL.md`

Run: `ls ~/.claude/skills/srd-generator/SKILL.md ~/.claude/skills/ai-solution-architect/SKILL.md ~/.claude/skills/nudesk-brand-styling/SKILL.md`

If any are missing, reinstall from the plugin:
```bash
cd ~/Projects/executive-os-plugin/skills/bundles
unzip -o srd-generator.skill -d ~/.claude/skills/
unzip -o ai-solution-architect.skill -d ~/.claude/skills/
unzip -o nudesk-brand-styling.skill -d ~/.claude/skills/
```

### MCP Servers
Check `~/.claude.json` and workspace `settings.json` for configured MCP servers:
- [ ] **Asana** — Required
- [ ] **Fireflies** — Optional
- [ ] **HubSpot** — Optional

### gws CLI (Google Workspace)
Run `gws auth status` via Bash — check for `"token_valid": true`. Google Workspace (Gmail, Calendar, Drive, Chat, Docs, Sheets) is accessed via the `gws` CLI, not an MCP server.

Smoke test: `gws gmail users messages list --params '{"userId":"me","maxResults":1}'`

Note which are configured vs. missing. Reference `~/Projects/executive-os-plugin/references/setup/gws-cli-setup.md` for setup instructions.

Note: The **security-reviewer** agent is bundled at `~/Projects/executive-os-plugin/agents/security-reviewer.md` — it is available for security audits via the Agent tool and does not require MCP configuration.

## 3. Compliance Controls

### .env Blocker Hook
Read `~/.claude/settings.json`:
- [ ] PreToolUse hook exists with `Edit|Write` matcher
- [ ] Hook blocks `.env` file edits (but allows `.env.example`)
- [ ] Hook uses the **bash pattern** (correct): `file="$CLAUDE_FILE_PATH"; if [[ "$file" == *.env ...`
- [ ] Hook does NOT use the old Python pattern (broken): `echo "$CLAUDE_TOOL_INPUT" | python3 -c ...`

If the Python pattern is present, replace it with the bash one-liner from `templates/hooks-settings.json.template`.

If missing entirely, provide the hook JSON from `templates/hooks-settings.json.template`.

### .gitignore Coverage
In the current project directory, check `.gitignore` for:
- [ ] `credentials/` pattern
- [ ] `.env` pattern
- [ ] `*.pickle` pattern
- [ ] `*.key` or `*.pem` pattern

Reference the SOC 2 compliance skill for the full checklist.

## 4. Platform References

Check `~/Projects/executive-os-plugin/references/platform-references/` exists and list what's present:

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
- [ ] `~/Projects/executive-os-plugin/references/` exists with expected subdirectories
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
