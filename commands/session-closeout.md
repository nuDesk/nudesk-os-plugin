---
description: End-of-session wrap-up — capture tasks, update memory, note learnings
allowed-tools: Read, Write, Edit, Grep, Glob, mcp__plugin_asana_asana__asana_search_tasks, mcp__plugin_asana_asana__asana_create_task, mcp__plugin_asana_asana__asana_update_task, mcp__plugin_asana_asana__asana_create_task_story
---

Run end-of-session closeout. This captures open items as Asana tasks, updates memory with new context, and logs Claude Code learnings.

## Step 1: Session Review

Review the current conversation to identify:

**Potential Asana tasks:**
- Action items that were discussed but not yet tracked
- Follow-ups committed to during the session
- Decisions that require next steps
- Items flagged for future attention

**New context for memory:**
- New people mentioned (names, roles, companies)
- New terms or acronyms used
- Project updates or status changes
- Updated priorities or strategic shifts
- New client information or relationship updates

**Test coverage check:**
- Were tests written or updated for code changes made this session?
- If not, is there a documented reason (config-only change, documentation, planning session)?
- Note: this is a prompt, not a hard gate. Capture the answer in the closeout record.

**Claude Code learnings:**
- Bash commands that were used or discovered
- Code style patterns followed
- Testing approaches that worked
- Environment/configuration quirks
- Warnings or gotchas encountered
- Useful patterns worth remembering

## Step 2: Search Asana, Then Suggest

### Step 2a: Search for Existing Tasks First

Before suggesting anything new, search Asana for tasks related to the session's work. Use `asana_search_tasks` with relevant keywords (project names, initiative names, people involved).

**Prefer updating existing tasks over creating new ones.** Most sessions advance an existing initiative, not start a new one. If a related task exists, suggest adding a comment (via `asana_create_task_story`) summarizing what was accomplished, rather than creating a duplicate.

### Step 2b: Granularity Rule — One Task Per Session

A typical session produces **1 task at most**. Multiple tasks only make sense for daily-plan sessions that complete several queued items.

Suggest at the **initiative or workstream level**, NOT the subtask level. If the session involved 5 small steps toward one goal, that's ONE task (or one update to an existing task).

**DO:** "Update existing task with progress comment" or "Finalize BDR Platform MVP for Gil"
**DON'T:** "Update sidebar CSS", "Fix API endpoint", "Add export button", "Write tests"

### Format

```
ASANA SEARCH RESULTS:
- [Existing task name] (GID: xxx) — [status] — Matches because: [reason]

RECOMMENDED ACTION:
1. Update "[Existing task name]" with comment summarizing session progress
   Comment: [brief summary of what was done]

— OR (if no related task found) —

1. Create: [High-level task name] -> [Suggested project] — Due: [date]
   Description: [brief bullets of what was done]

Proceed / modify / skip?
```

Apply smart routing logic from `~/.claude/memory/asana-config.md` (only for new tasks). The config file contains a routing table that maps context clues to Asana project GIDs. Use project GIDs directly — no typeahead searches needed.

Only suggest tasks that are genuinely material — skip trivial or already-tracked items.

**Config:** Load Asana GIDs from `~/.claude/memory/asana-config.md` for workspace, user, project, and custom field GIDs.

**IMPORTANT: STOP HERE.** Present the suggested tasks and wait for the user to respond (create all / select numbers / skip / modify). Do NOT proceed to Step 3 until the user has confirmed their task choices and tasks have been created (or skipped). This is a hard gate — Steps 3-5 only run after the user responds to Step 2.

When the user responds, create only approved tasks using Asana MCP tools. Include the subtask bullets in the Asana task description (in `html_notes` format). If the user provides modifications (different deadlines, project routing, etc.), apply those before creating.

## Step 3: Update Memory

If new context was identified:

1. **CLAUDE.md updates** — Update the Working Memory section if:
   - Priorities have shifted
   - New frequently-referenced people should be added
   - New terms are being used regularly
   - Project status has changed

2. **~/.claude/memory/ updates** — Add or update files for:
   - New people -> `~/.claude/memory/people/` or `~/.claude/memory/glossary.md`
   - New terms -> `~/.claude/memory/glossary.md`
   - Project updates -> `~/.claude/memory/projects/`
   - Company context -> `~/.claude/memory/context/`

Present proposed memory updates before writing:
```
MEMORY UPDATES:

- ~/.claude/CLAUDE.md: [what will change]
- ~/.claude/memory/glossary.md: [additions]
- ~/.claude/memory/people/[name].md: [new file or updates]

Apply these updates? [yes/no]
```

Follow the memory-management skill's conventions for formatting and hot cache vs. deep memory.

## Step 4: Claude Code Learnings

Check if there are learnings worth persisting:

- Useful bash commands or patterns discovered
- Configuration quirks or gotchas
- Code patterns specific to this codebase
- Environment setup notes

If learnings exist, propose additions to the relevant CLAUDE.md (project-level or global). Keep entries concise — one line per concept.

## Step 5: Confirm Closeout

Summarize what was captured:
- Tasks created (with Asana links if available)
- Memory updated (what changed)
- Learnings captured (what was added)
- Any items intentionally skipped

End with: "Session wrapped. Anything else before we close out?"
