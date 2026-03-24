# Executive-Level Claude Code Permissions Standard

**Scope:** nuDesk executive users (Sean Salas, Kenny Salas)
**Last updated:** 2026-03-21
**Classification:** Internal reference — not a live config file

> These permissions are applied via `settings.local.json` (per-project, per-machine, gitignored).
> They are NOT committed to repos and do NOT sync across machines.
> This document is the canonical reference for what executive-level users should have.

---

## How Claude Code Permissions Work

| Layer | File | Scope | Committed? |
|-------|------|-------|-----------|
| **Global** | `~/.claude/settings.json` | All projects on this machine | No (user home) |
| **Workspace shared** | `<project>/.claude/settings.json` | Anyone who clones the repo | Yes |
| **Workspace local** | `<project>/.claude/settings.local.json` | Only this machine | No (gitignored) |

Precedence: project-local > project-shared > global. Deny lists always win over allow lists.

---

## Global Permissions (Executive Tier)

These are set in `~/.claude/settings.json` and apply to every project unless overridden.

### Core Tools — Always Allowed

```
Read, Edit, Write, Glob, Grep, Search, WebFetch, WebSearch, Bash(*)
```

**Rationale:** Executive users own the full development lifecycle. Unrestricted bash and file access eliminates friction for build/deploy/debug workflows. Security is enforced via deny lists and hooks, not by restricting core tools.

### MCP Integrations — Full Suite

| MCP Server | Tools Allowed | Rationale |
|------------|--------------|-----------|
| **Asana** (plugin) | All 35 tools (tasks, projects, goals, portfolios, sections, teams, users, attachments, status updates, time periods) | Executive OS daily planning, task execution, strategic reviews |
| **Fireflies** | All 8 tools (search, fetch, transcripts, summaries, users, contacts, groups) | Meeting context for daily plans and follow-ups |
| **HubSpot** | All 8 tools (CRM search, objects, properties, owners, manage, feedback) | Pipeline visibility, deal tracking, campaign management |
| **Google Workspace** | All 23 tools (Gmail search/read/draft/filter/labels, Calendar events/freebusy, Chat spaces/messages/reactions) | Cross-system briefings, email drafts, calendar management |
| **Google Drive** | All 12 read tools (search, listFolder, doc/sheet/slides content, comments, permissions, shared drives) | Document access for context gathering |
| **Gamma** | All 4 tools (presentations, status, themes, folders) | Deck generation for client deliverables |
| **Context7** | Both tools (resolve-library-id, get-library-docs) | Live documentation lookup during development |
| **Playwright** | browser_take_screenshot, browser_click, browser_type, browser_evaluate, browser_fill_form, browser_wait_for, browser_run_code | Browser automation for UI testing, design review, and debugging |
| **Vanta** (conditional) | All MCP tools if Core+ plan: vanta_list_frameworks, vanta_get_controls, vanta_get_tests, vanta_get_vulnerabilities, vanta_get_evidence, vanta_get_people | Compliance dashboard, control status, evidence tracking (read-only via MCP; writes via REST API) |

### Global Deny List — Hard Guardrails

These are **never** overridden, regardless of project:

```
Bash(git push --force*)     # Protects shared branch history
Bash(git push -f*)          # Same — short flag variant
Bash(git reset --hard*)     # Protects uncommitted work
Bash(rm -rf /*)             # Prevents system-level destruction
Bash(rm -rf ~*)             # Prevents home directory destruction
Bash(git clean -f*)         # Protects untracked files
Bash(git branch -D *)       # Protects branch references
```

### Global Hooks — Automated Guardrails

| Hook | Trigger | Action |
|------|---------|--------|
| PreToolUse: `.env` blocker | Edit or Write to `*.env` / `*.env.*` | Block (except `.env.example`) |

---

## Project-Level Permission Patterns

Projects add **targeted** permissions on top of global. These patterns document the standard approach.

### Pattern: Development Project (Portal, MDF backends)

Additional project-level permissions typically include:
- Deployment tools: `gcloud run deploy`, `docker build`, CI/CD commands
- Test runners: `pytest`, `npm test`, `curl` for local API testing
- Package management: `pip3 install`, `npm install`
- Project-specific MCP tools (n8n workflow management, etc.)

Project-level deny lists add:
```
Read(.env), Read(.env.*), Read(**/.env), Read(**/.env.*)   # Block .env reads
Bash(rm *), Bash(rm -*), Bash(sudo *), Bash(rmdir *)       # Block destructive ops
```

### Pattern: Workflow/Automation Project (n8n definitions, nudesk-os-plugin)

Minimal permissions — typically just:
- Asana MCP tools needed for that project's scope
- Python/Node for local testing
- Git operations

### Pattern: Content/Research Project (LinkedIn, Business Plans)

Minimal — usually just:
- Specific skills (e.g., `Skill(writing-style)`)
- Asana for task logging
- No deployment or destructive bash access

---

## SOC 2 Compliance Rules

These rules apply to **all** permission configurations:

1. **No secrets in settings files.** API keys, tokens, and credentials must never appear in `settings.json` or `settings.local.json`. Use `.env` files and reference by variable name only.

2. **No secrets in chat.** The `.env` hook blocks direct editing. Credentials are managed manually or via secret managers.

3. **Deny lists are non-negotiable.** The global deny list (force push, hard reset, rm -rf, etc.) is never overridden at the project level.

4. **Read-only defaults for shared systems.** MCP tools that modify shared state (HubSpot `manage_crm_objects`, Asana `delete_task`, Google Workspace write tools) are allowed at the global level for executive users but should prompt for confirmation via Claude's built-in safety checks on risky actions.

5. **Audit trail.** All Claude Code sessions that touch production systems should use `/session-closeout` to capture actions taken.

---

## Permission Inventory — Current State (2026-03-18)

| Project | MCP Tools | Bash Scope | Deny List | Risk Level |
|---------|-----------|-----------|-----------|-----------|
| **Global** | Full Asana, Fireflies, HubSpot, GWS, Drive, Gamma, Context7, Playwright (7 tools) | `Bash(*)` | Force push, hard reset, rm -rf, clean, branch -D | Controlled |
| **Prime Nexus/Portal** | HubSpot (4 tools) | File-path-specific whitelist | + rm, sudo, .env reads | Restrictive |
| **Prime Nexus/AI_SDR_Campaigns** | HubSpot (6), Drive (6), n8n (2), Asana (1) | Broad bash | + rm, sudo, unlink, shred | Medium |
| **MDF/mdf-notice-agent** | n8n (7) | GCP deploy + LibreOffice | Standard | Medium |
| **MDF/mdf-investor-statement-agent** | n8n (9), Drive (1), Asana (2) | GCP deploy + node | Standard | Medium |
| **nudesk-os-plugin** | Asana (2) | Global only | Global only | Low |
| **swing_local** | Figma (1) | Global only | Global only | Low |
| **SS_LinkedIn** | Asana (1) | Global only | Global only | Low |

---

## Updating This Document

When adding new MCP servers, tools, or changing the permission model:
1. Update the relevant `settings.local.json` or `settings.json`
2. Update this reference doc to reflect the change
3. If a new MCP server is added globally, add it to the MCP Integrations table above
4. If a new deny rule is added, add it to the Global Deny List

---

## Non-Executive Users

This standard is for **executive-level users only** (founders with full system access). Future team members or contractors should receive scoped permissions:

- Read-only MCP access unless their role requires writes
- Project-specific bash whitelists (no `Bash(*)`)
- No access to production deployment commands
- Separate deny lists per role

A team-level permissions template should be created when the first non-executive user is onboarded.
