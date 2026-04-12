---
name: evidence-collector
description: >
  Background evidence processing skill. Processes the evidence buffer
  (git commits, deployments, security scans) into Asana Production Change Log
  entries and optionally uploads to Vanta. Referenced by session-closeout,
  evidence-collect, and deploy workflows.
user_invocable: false
version: 1.0.0
---

# Evidence Collector

> Not user-invocable. Called by `/session-closeout`, `/evidence-collect`, and deploy workflows.

## Purpose

Process the evidence buffer at `~/.claude/memory/context/evidence-buffer.md` into structured compliance artifacts:
1. Create/update Asana Production Change Log tasks with evidence attachments
2. Map evidence to Control IDs from the 91-control matrix
3. Upload evidence to Vanta (when API available) or save locally for manual upload

## Evidence Buffer Format

The evidence buffer is a flat file with entries appended by hooks and session activity:

```markdown
## [timestamp]
- **Type:** commit | deploy | security-scan | config-change
- **Details:** [hash, message, revision ID, etc.]
- **Controls:** [control IDs, if known]
- **Project:** [project path, if applicable]
```

## Processing Logic

### Step 1: Read and Parse

1. Read `~/.claude/memory/context/evidence-buffer.md`
2. Parse entries into structured records
3. Group by type and time window (same-day entries for the same project = one Change Log entry)

### Step 2: Map to Controls

| Evidence Type | Default Controls | Additional Context |
|--------------|-----------------|-------------------|
| commit | SD-01, SD-02 | If PR merged: add OS-01 |
| deploy | OS-01, OS-12, SD-04 | If Cloud Run: add infra evidence |
| security-scan | SD-04, OS-08 | Map specific findings to controls |
| config-change | OS-10, CR-01 | If encryption-related: add CR-01 |

### Step 3: Create Asana Change Log Entries

For each grouped evidence set:

1. Load Production Change Log GID from `~/.claude/memory/compliance-config.md`
2. Check if a Change Log task already exists for this change (search by git hash or deployment revision)
3. **If exists:** Add a comment with new evidence via `asana_create_task_story`
4. **If new:** Create a task via `asana_create_task`:
   - Title: Brief description derived from commit messages or deploy context
   - Change Type: infer from evidence type (commit → Feature/Bugfix, deploy → Infrastructure, etc.)
   - Control ID: mapped control IDs
   - Description: aggregated evidence details
   - completed: `true` (these are log entries for changes that already happened, not action items)
   - Do NOT set `assignee` — Change Log entries are compliance records, not personal tasks
   - Subtasks: change management checklist from compliance config template
5. Attach evidence summary as a comment

### Step 4: Vanta Upload (conditional)

Check `~/.claude/memory/compliance-config.md` → Vanta → API Access.

**If "yes":**
1. Format each evidence group as a document
2. Upload via `POST https://api.vanta.com/v1/evidence`
   - Respect rate limits: sequential uploads, max 50/min
   - Use naming convention: `{date}_{control-id}_{description}.md`
3. Link to relevant controls via `POST /controls/{id}/evidence`
4. Return upload count and any failures

**If "no" (UI-only):**
1. Save formatted evidence docs to `~/.claude/memory/context/evidence-exports/`
2. Use naming convention: `{date}_{control-id}_{description}.md`
3. Return file list for manual Vanta upload

### Step 5: Archive and Clear

1. Append processed entries to `~/.claude/memory/context/evidence-archive.md` with processing timestamp
2. Clear processed entries from `~/.claude/memory/context/evidence-buffer.md`
3. Return summary: entries processed, Change Log tasks created/updated, Vanta uploads (if any)

## Return Format

The skill returns a structured summary to the calling command:

```
Evidence Processing Complete:
- Entries processed: [N]
- Change Log tasks created: [N]
- Change Log tasks updated: [N]
- Vanta evidence uploaded: [N] (or "N/A — UI-only mode")
- Evidence archived: [N] entries
```
