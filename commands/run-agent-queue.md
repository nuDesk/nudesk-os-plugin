---
description: Autonomously process Agent Queue tasks overnight with iterative QA — no approval gates
allowed-tools: Read, Write, Edit, Bash, Grep, Glob
---

# Autonomous Agent Queue Processor

Process ALL Asana tasks flagged with the Agent Queue custom field autonomously. No approval gates, no interactive clarification. Iterative build-review-test loops for quality. Per-task Asana comments summarizing results.

> **This is NOT `/run-tasks`.** `/run-tasks` delegates to the interactive `asana-agent` skill with mandatory approval gates. This command is fully autonomous, batch-oriented, and designed for unattended overnight execution.

---

## Phase 1 — Initialization & Queue Fetch

1. Load Asana GIDs from `~/.claude/memory/asana-config.md`
2. Record queue start time: `START_TIME=$(date -u +%Y-%m-%dT%H:%M:%SZ)`
3. Create log directory and file:
   ```
   mkdir -p ~/Projects/SS-Agent/logs
   LOG_FILE=~/Projects/SS-Agent/logs/agent-queue-$(date +%Y-%m-%d).log
   ```
4. Search for tasks via `asana_search_tasks`:
   - `workspace: 1211894317926172`
   - `assignee_any: "1211913018942721"` (Sean)
   - `completed: false`
   - **No date filter** — process ALL queued tasks to clear the backlog
   - **Do NOT use `custom_fields` filter** — Asana's search API rejects or silently ignores custom field filters on the Agent Queue Flag field. Instead, fetch all incomplete tasks assigned to Sean and filter client-side.
5. **Client-side filter:** For each returned task, check `custom_fields` array for GID `1211903626313619` (Task Progress) with `display_value: "Agent Queue"`. Only tasks matching this filter enter the queue. Discard the rest.
6. For each queued task, fetch full details with `asana_get_task`:
   - Fields: `name, notes, custom_fields, due_on, projects.name, memberships`
   - `include_subtasks: true` — subtasks define execution steps
   - `include_comments: true` — comments may contain additional context
7. Check for attachments via `asana_get_attachments_for_object`
8. Read the "Project Folder" custom field (GID: `1214000778857113`) to determine working directory
9. If Project Folder is empty, default working directory to `~/Projects/SS-Agent/`

**Pre-flight:** Before fetching tasks, run the `/refresh-gws` check. If gws auth is invalid and any queued tasks may need Google Workspace access, print a warning but continue — tasks that don't need gws will still process. Log the auth status.

**If zero tasks found:** Print "AGENT QUEUE: Empty — no tasks to process" and exit.

---

## Phase 2 — Prioritization & Sorting

Sort the task list by:
1. **Priority:** Highest > High > Medium > Low > unset
2. **Due date:** overdue first, then soonest due
3. **Creation date:** oldest first (tiebreaker)

Priority GID mapping (from asana-config):
- `1211963278594385` = Highest
- `1211963278594386` = High
- `1211963278594387` = Medium
- `1211963278594388` = Low

Print queue inventory to console:
```
AGENT QUEUE: [N] tasks
──────────────────────────────────────────
1. [Task Name] — [Priority] — [Type] — [Project Folder] — [N subtasks]
2. [Task Name] — [Priority] — [Type] — [Project Folder] — [N subtasks]
...
──────────────────────────────────────────
```

Log the same inventory to `$LOG_FILE`.

---

## Phase 3 — Execution Strategy Classification

For each task, determine the execution strategy based on these input signals (in priority order):

### Input Signals

1. **Subtasks** — If the task has subtasks, they define the execution plan. Each subtask = a discrete step.
2. **Description** — The task's `notes` field is the prompt with role, context, instructions, and output format.
3. **Type field** — Secondary signal for classification.

### Strategy Decision Logic

| Strategy | When | Indicators |
|----------|------|------------|
| **Direct** | Single-output deliverables | Type = Strategy & Planning, Analysis, Reporting, Account Mgmt, Business Dev; description requests a single document/artifact; no subtasks OR subtasks are sequential steps for one deliverable |
| **Subagent** | Quick research or focused lookups | Type = Learning, Activity; focused question; estimated < 15 min; no subtasks |
| **Agent Team** | Multi-file development or multi-module work | Type = Development, Security; subtasks reference different files/modules/layers; description spans multiple components |

### Tool Access for Agent Teams

| Tool | Lead | Teammates | Notes |
|------|------|-----------|-------|
| **MCP tools** (Asana, HubSpot, Fireflies, n8n, Figma) | Yes | **No** | Lead must pre-fetch MCP data and write to `{project}/.agent-context/{task-gid}-data.md` for teammates |
| **`gws` CLI** (Google Workspace — Drive, Docs, Sheets, Gmail, Calendar) | Yes | **Yes** (via Bash) | Teammates CAN use `gws` directly — no pre-fetch needed |
| **Bash, Read, Write, Edit, Grep, Glob** | Yes | Yes | Standard file/system tools available to all |

**Key implication:** `gws` is a Bash CLI tool, not MCP. Teammates have full access to Google Workspace. Only MCP-dependent data (HubSpot CRM, Asana, Fireflies transcripts) needs to be pre-fetched by the lead.

**`gws` auth note:** The `gws` CLI uses OAuth2 tokens stored in macOS Keychain (`~/.config/gws/`). Auth is session-persistent — teammates inherit it. If auth expires mid-run, log the error and continue to the next task (auth refresh requires interactive browser flow).

---

## Phase 4 — Sequential Task Processing (with Iterative QA)

Process tasks one at a time. For each task:

### 4a. Setup

1. Log task start: `echo "[$(date -u +%Y-%m-%dT%H:%M:%SZ)] START: {task_name} (GID: {task_gid})" >> $LOG_FILE`
2. Validate Project Folder exists at `~/Projects/{value}`:
   - If the path doesn't exist, log warning and skip task (set to "Waiting" with comment)
3. Read the project's `CLAUDE.md` if it exists (for project-specific instructions, build commands, test commands)
4. **Update Task Progress to "In Progress"** via `asana_update_task`:
   - `custom_fields: {"1211903626313619": "1211903626313621"}`
   - This moves the task out of "Agent Queue" status, preventing re-processing on the next run.
   - **If this fails** (field not on the task's project), log a warning and continue — do NOT treat this as a task failure. Also try clearing the deprecated Agent Queue Flag field if present: `custom_fields: {"1213997640085002": null}`.
5. Download any text-based attachments and include content in execution context
6. If subtasks exist, load them as the execution plan (ordered by position)
7. **For document/memo tasks:** Load the correct brand guide based on the task's Asana project. Apply brand colors, typography, and voice guidelines to all generated documents.

   **Brand Guide Routing (by Asana project):**

   | Asana Project | Brand | Guide Path |
   |---------------|-------|------------|
   | Prime Nexus, Live Oak Bank, SwingLocal | PRIME Nexus | `~/Projects/nudesk-os-plugin/references/brand-guides/prime-nexus-brand-guide.md` |
   | Executive Management, AI Development, IT & Vendor Review, Business Development, Compliance & Data Security | nuDesk | `~/Projects/nudesk-os-plugin/references/brand-guides/nudesk-brand-guide.md` |
   | Tasks mentioning Cintar, investor relations, or holding company matters | Cintar Group | `~/Projects/nudesk-os-plugin/references/brand-guides/cintar-group-brand-guide.md` |
   | All others / fallback | nuDesk | `~/Projects/nudesk-os-plugin/references/brand-guides/nudesk-brand-guide.md` |

   If the referenced brand guide file does not exist, log a warning and fall back to a clean professional document with the nuDesk footer. Do NOT generate unbranded documents.

### 4b. Execute Based on Strategy

#### Direct Strategy
- The task description IS the prompt. Execute it directly.
- If subtasks exist, execute them sequentially as steps toward the deliverable.
- Write output where the description specifies.
- **Memo / report / strategy deliverables default to `.docx` format:**
  - Generate via `python-docx` with `Document()` (blank document — do NOT depend on templates in `/tmp/` or other ephemeral paths)
  - Apply brand colors and typography from the brand guide loaded in step 4a.8. Each brand defines its own palette — read the guide and extract: primary dark, primary accent, secondary accent, heading font, body font.
  - Standard document structure (applies to all brands):
    - Branded heading colors and sizes (H1 > H2 > H3 hierarchy)
    - Table headers: primary dark background with white text, alternating row shading
    - Callouts: primary accent left border for key recommendations
    - Dividers: primary accent color
    - Footer: centered entity name + tagline (e.g., "nuDesk LLC | AI-Workforce Agency for Financial Services")
  - Write a rerunnable Python generation script to the project's `scripts/` directory
  - Output the `.docx` to the project's `reports/` directory or `~/Projects/SS-Agent/deliverables/`
  - Bullet formatting: simple left indent (`Inches(0.25)`), NO hanging indent (`first_line_indent`)
- Other deliverables (code, configs, data files) use the appropriate file extension.
- Fallback default location: `~/Projects/SS-Agent/deliverables/{task-name-slugified}.docx`

#### Subagent Strategy
- Spawn via the Agent tool with:
  - The task description as the prompt
  - The project's CLAUDE.md content as additional context
  - `subagent_type: "general-purpose"`
- Capture the agent's output as the deliverable.

#### Agent Team Strategy
1. Analyze subtasks/description for distinct work streams (different files, modules, or layers)
2. **Pre-fetch all MCP data** the teammates will need:
   - Query HubSpot, Asana, Fireflies, etc. for relevant data
   - Write results to `{project_folder}/.agent-context/{task-gid}-data.md`
3. Create team via `TeamCreate`:
   - 2-4 Sonnet teammates max
   - Each teammate gets clear file ownership (no overlapping files)
   - Lead coordinates and handles MCP-only operations
4. Monitor team progress and collect results
5. Clean up `.agent-context/` directory after completion

### 4c. Iterative QA Loop (Ralph Loop Pattern)

After initial execution, enter an iterative self-review cycle. Max **3 iterations** to prevent infinite loops.

#### For Document / Memo / Strategy Tasks:
1. **Requirements check:** Does the output address ALL sections/points requested in the description?
2. **Tone check:** Is the tone appropriate for the audience (executive, technical, client-facing)?
3. **Data accuracy:** Are referenced data points, names, dates correct?
4. **Formatting standards:** Professional headers, consistent formatting, appropriate use of tables vs. prose, branded `.docx` structure (not markdown)
5. **Brand compliance:** Verify the document applies the correct brand palette per the guide loaded in 4a.8. Check: heading colors match the brand's primary dark, accent colors match the brand's primary accent, table headers are branded, footer includes the correct entity name. If colors or formatting are missing, regenerate the python-docx script with brand helpers.
6. If any check fails, revise and re-check (up to 3 iterations)

#### For Development Tasks:
1. **Run tests:** Execute applicable test commands from the project's CLAUDE.md:
   - Python: `pytest` or project-specific test command
   - Frontend: `vitest`, `npm test`, or project-specific
2. **Run linters/type checkers:** If configured in the project (eslint, mypy, ruff, etc.)
3. **Requirements validation:** Check output against the task description and subtask requirements
4. **If tests/linters fail:** Fix issues and re-run (up to 3 iterations)

#### For All Tasks:
- Each iteration logs what was changed and why to `$LOG_FILE`
- If still not passing after 3 iterations:
  - Note the remaining issues in the Asana comment
  - Set task to **"Waiting"** instead of "Pending Review"

### 4d. Security & Compliance (SOC 2)

**Authorization model:**
- The task assignee is the **authorizer**. If the queue runs under Sean's tasks, Sean authorized the work by placing it in Agent Queue. Same applies to Kenny when he has access.
- For **development tasks**, authorization to execute is implicit (assignee queued it), but **deployment approval is separate** — the task stays in "Pending Review" until the assignee reviews and approves (see 4e).

Apply these checks based on task type:

- **For development tasks:**
  - Check for OWASP top 10 vulnerabilities in any code written (injection, XSS, etc.)
  - Verify no hardcoded credentials or API keys in output
  - Check for insecure dependencies if new packages were added
  - **Do NOT push to remote or deploy.** Code stays local. The assignee reviews and pushes/deploys after human review.
  - **Branch protection (all nuDesk repos):** If the task produces code changes,
    create a feature branch (`agent-queue/YYYY-MM-DD-{task-slug}`) and commit there.
    After all tasks complete, open a PR per repo via `gh pr create` with a summary.
    Leave for human review and merge the next morning.
- **For tasks touching production systems:**
  - Log the activity as a change record (per SOC 2 Change Management controls)
- **For tasks involving data:**
  - Verify no PII exposure in output files
  - Verify no secrets in output files
- **Activity logging:** For EVERY task, log to `$LOG_FILE`:
  - Task GID, task name
  - Start/end timestamps
  - Strategy used (Direct/Subagent/Agent Team)
  - Files created or modified (list paths)
  - Compliance checks performed
  - Result (success/fail/partial)
  - Authorizer (task assignee name)

### 4e. Post-Execution — Asana Updates

**Add completion comment** via `asana_create_task_story`:

```
Agent Queue — Task Complete

Summary: [1-2 sentence description of what was done]

Deliverables:
- [file path or description of output]

QA: [Passed/N iterations] — [brief note on what was validated]
Compliance: [Any SOC 2 checks performed, or "N/A"]
Duration: [time elapsed for this task]

Follow-ups: [Any recommended next steps, or "None"]
```

**Update Task Progress based on task type:**

- **Development / Security tasks (code was written or modified):**
  - Set Task Progress to **"Pending Review"** (`1213004223487118`)
  - Do NOT mark the task complete — code requires human review before production deployment
  - Note in the Asana comment: "Set to Pending Review — code changes require human review before merge/deploy."
  - **SOC 2 rationale:** Change Management (CC8.1) requires review and approval of changes before production deployment. The task assignee (Sean, or Kenny when he has access) serves as the reviewer.

- **All other tasks (documents, memos, reports, analysis, research):**
  - Set Task Progress to **"Done"** (`1211903626313624`)
  - Mark the task complete via `asana_update_task` with `completed: true`

If the Task Progress update fails (field not on project), log and continue — the Asana comment is the primary completion signal.

**If subtasks exist**, mark each completed subtask as done:
- `asana_update_task` with `completed: true` for each subtask that was successfully executed

---

## Phase 5 — Error Handling

### Per-Task Timeouts
- **Direct/Subagent:** 30 minutes max per task
- **Agent Team:** 45 minutes max per task

### On Task Failure or Timeout:
1. Add Asana comment with error details:
   ```
   Agent Queue — Task Failed

   Error: [error message or timeout]
   Partial progress: [what was completed before failure]
   Attempted: [strategy used, iterations completed]

   This task has been set to "Waiting" for manual review.
   ```
2. **Try to set Task Progress to "Waiting"** (`custom_fields: {"1211903626313619": "1211903626313622"}`) — NOT back to Agent Queue (prevents infinite retry loops). If the field is not on the task, skip — the Agent Queue flag was already cleared in 4a.4, and the error comment is the signal.
3. Log failure details to `$LOG_FILE`
4. **Continue to next task** — never let one failure stop the queue

### Critical Errors (Stop Entire Run):
- Asana API unreachable (cannot fetch or update tasks)
- Filesystem read-only or disk full
- Auth failures on MCP tools that prevent all task processing

On critical error, log to `$LOG_FILE` and print error to console before exiting.

---

## Phase 6 — Cleanup & Final Summary

1. **Clean up temp directories:** Remove any `.agent-context/` directories created during execution
2. **Verify no Agent Teams left running:** Check and clean up any orphaned team processes
3. **Create PRs for code changes:** If any tasks produced commits on feature branches,
   open a PR per repo via `gh pr create`. Include task GIDs and names in the PR body.
   Format: `gh pr create --title "agent-queue: YYYY-MM-DD batch" --body "Tasks processed: ..."`
4. **Log completion:** Record end time and summary in `$LOG_FILE`
5. **Print final summary to console:**

```
AGENT QUEUE COMPLETE
──────────────────────────────────────────
Tasks: [N] processed | [N] succeeded | [N] failed | [N] skipped
Duration: [total time]
Log: ~/Projects/SS-Agent/logs/agent-queue-YYYY-MM-DD.log
──────────────────────────────────────────

Succeeded:
  - [Task Name] → [deliverable path or "Asana comment posted"]
  - ...

Failed:
  - [Task Name] → [brief reason]
  - ...
```

---

## Important Reminders

- **No approval gates.** Execute every task autonomously. Do not pause for user confirmation.
- **No interactive clarification.** If a task description is ambiguous, make a reasonable interpretation, execute, and note the interpretation in the Asana comment.
- **Platform constraints.** Before writing code for any managed platform, check `~/Projects/nudesk-os-plugin/references/platform-references/` for constraints.
- **Project CLAUDE.md.** Always read and follow the project-specific CLAUDE.md if one exists in the working directory.
- **Deliverables directory.** Ensure `~/Projects/SS-Agent/deliverables/` exists before writing default deliverables.
- **SOC 2.** Log everything. No exceptions.
- **Brand guide.** Always load and apply brand guidelines for document deliverables. Never generate unbranded memos or reports.

---

## Asana API Quirks (Learned from Production)

These are verified behaviors discovered during live runs. Do not attempt the failing patterns — go directly to the working patterns.

| Quirk | Failing Pattern | Working Pattern |
|-------|----------------|-----------------|
| **Custom field search** | `custom_fields_any.{gid}: ["{value}"]` in `asana_search_tasks` — rejected or returns empty | Broad search (assignee + incomplete), then filter client-side by checking `custom_fields` array for `display_value` match |
| **Agent Queue is a Task Progress option** | The separate Agent Queue Flag field (GID `1213997640085002`) was a temporary workaround | Use Task Progress = "Agent Queue" (GID `1213004221997428` on field `1211903626313619`). Deprecated flag field should be cleared with `null` if encountered. |
| **Task Progress field not on all projects** | `custom_fields: {"1211903626313619": "..."}` — fails with "Custom field with ID is not on given object" | Always wrap Task Progress updates in try/catch. The field is project-dependent — not all projects have it. |
| **Google Drive Shared Drives** | `gws drive files create` without `supportsAllDrives` — 404 on Shared Drive folders | Always include `"supportsAllDrives": true` in params for any Drive operation |
| **python-docx template paths** | Referencing templates in `/tmp/` or other ephemeral locations | Use `Document()` (blank) with brand helper functions, or reference templates from stable project paths (`reports/templates/`) |
