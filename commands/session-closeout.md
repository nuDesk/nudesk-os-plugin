---
description: End-of-session wrap-up — capture tasks, update memory, note learnings
allowed-tools: Read, Write, Edit, Grep, Glob
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

**Claude Code learnings:**
- Bash commands that were used or discovered
- Code style patterns followed
- Testing approaches that worked
- Environment/configuration quirks
- Warnings or gotchas encountered
- Useful patterns worth remembering

## Step 2: Suggest Asana Tasks

### Granularity Rule — Think Big, Not Small

Suggest tasks at the **initiative or workstream level**, NOT the subtask level. If the session involved 5 small steps toward one goal, that's ONE task — not five.

**DO:** "Finalize BDR Platform MVP for Gil" (with subtasks listed in the description)
**DON'T:** "Update sidebar CSS", "Fix API endpoint", "Add export button", "Write tests", "Deploy to staging"

Roll up related work into a single parent task. Include the specific subtasks or steps completed as bullet points in the task **description**, not as separate Asana tasks. A typical session should produce 1-3 tasks, rarely more.

### Format

```
SUGGESTED TASKS FROM THIS SESSION:

1. [High-level task name] -> [Suggested project] — Due: [date]
   Subtasks completed: [brief bullets of what was done]

2. [High-level task name] -> [Suggested project] — Due: [date]
   Subtasks completed: [brief bullets of what was done]

Create all / select specific numbers / skip?
```

Apply smart routing logic from `~/.claude/memory/asana-config.md`. The config file contains a routing table that maps context clues (keywords, people, clients) to Asana project GIDs. Use project GIDs directly — no typeahead searches needed.

Only suggest tasks that are genuinely material — skip trivial or already-tracked items.

Wait for user confirmation. Create only approved tasks using Asana MCP tools. When creating, include the subtask bullets in the Asana task description (in `html_notes` format).

**Config:** Load Asana GIDs from `~/.claude/memory/asana-config.md` for workspace, user, project, and custom field GIDs.

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
