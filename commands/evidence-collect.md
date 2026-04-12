---
description: Collect compliance evidence from git, deployments, and infrastructure — prepare for Asana and Vanta
allowed-tools: Read, Bash, Glob, Grep, Write, Edit, mcp__plugin_asana_asana__asana_search_tasks, mcp__plugin_asana_asana__asana_get_task, mcp__plugin_asana_asana__asana_create_task, mcp__plugin_asana_asana__asana_update_task, mcp__plugin_asana_asana__asana_create_task_story
---

Collect technical compliance evidence that Vanta can't see natively. Process the evidence buffer, gather git/deployment/config data, and create Asana Change Log entries. Optionally upload to Vanta when API is available.

## Step 1: Load Config

Read `~/.claude/memory/compliance-config.md` to load:
- Production Change Log project GID
- Custom field GIDs (Control ID, Change Type)
- Vanta API access status
- Change Log subtask template

Read `~/.claude/memory/asana-config.md` for workspace and user GIDs.

If compliance config is missing: stop and direct user to `/os-setup`.

## Step 2: Process Evidence Buffer

Check if `~/.claude/memory/context/evidence-buffer.md` exists.

If it exists, read and parse entries. Each entry should contain:
- Timestamp
- Type (commit, deploy, security-scan, config-change)
- Details (hash, message, revision ID, etc.)
- Associated Control IDs (if tagged)

Group entries by type and time window.

## Step 3: Collect Evidence by Type

Run applicable collectors based on the current project context. Ask the user which types to collect, or collect all if invoked from `/session-closeout`.

### 3a. Git Evidence (SD-01, SD-02, OS-01)

```bash
# Recent commits (last 7 days)
git log --oneline --since="7 days ago" --format="%H|%s|%an|%ai"

# PR merge history (if GitHub)
gh pr list --state merged --limit 20 --json number,title,mergedAt,author

# Branch protection status
gh api repos/{owner}/{repo}/branches/main/protection 2>/dev/null || echo "No branch protection or not a GitHub repo"
```

Produce a summary: number of commits, number of PRs merged, branch protection status.

### 3b. Deployment Evidence (OS-12, SD-04)

```bash
# Cloud Run revision history (if GCP project)
gcloud run revisions list --service=[service-name] --region=[region] --format="table(name,creation_timestamp,status)" --limit=10 2>/dev/null || echo "No Cloud Run service found"
```

Produce a summary: recent deployments, revision IDs, timestamps.

### 3c. Configuration Evidence (OS-10, CR-01)

```bash
# Check for encryption in transit (TLS)
# Check for secrets management
# Check for CORS configuration
# These are project-specific — run applicable checks
```

### 3d. Security Scan Evidence (SD-04)

Check if `/security-check` was run this session. If so, reference those findings.

If not, offer to run it: "No security scan found this session. Run `/security-check` first?"

## Step 4: Present Evidence Summary

```
EVIDENCE COLLECTION — [Today's Date]

EVIDENCE BUFFER
  Entries processed:     [N]
  Commits:               [N]
  Deployments:           [N]
  Security scans:        [N]
  Config changes:        [N]

GIT EVIDENCE (SD-01, SD-02, OS-01)
  Commits (7 days):      [N]
  PRs merged:            [N]
  Branch protection:     [Enabled / Not enabled / N/A]

DEPLOYMENT EVIDENCE (OS-12, SD-04)
  Recent deployments:    [N]
  Latest revision:       [revision-id] ([timestamp])

SECURITY EVIDENCE (SD-04)
  Last scan:             [date] or "Not scanned this session"

PROPOSED ACTIONS:
1. Create Change Log entry: "[Description]" — Controls: [IDs]
2. Create Change Log entry: "[Description]" — Controls: [IDs]
3. ...
```

**IMPORTANT: STOP HERE.** Present the summary and proposed actions. Wait for user confirmation before creating any Asana tasks or uploading evidence.

## Step 5: Create Asana Change Log Entries

For each approved action:

1. Create a task in the Production Change Log project via `asana_create_task`:
   - **Title:** Brief description of the change
   - **Change Type** custom field: appropriate enum value
   - **Control ID** custom field: relevant control ID(s)
   - **Description (html_notes):** What changed and why, with evidence details
   - **completed:** `true` (these are log entries for changes that already happened, not action items)
   - **Do NOT set `assignee`** — Change Log entries are compliance records, not personal tasks. Setting an assignee clutters My Tasks.
   - **Subtasks:** Add the change management checklist from the compliance config template

2. Add evidence as a comment via `asana_create_task_story`:
   - Git log summary
   - Deployment details
   - Security scan results
   - Link to PR (if applicable)

3. Ask the user to review and complete the subtask checklist items that require human verification.

## Step 6: Vanta Upload (if API available)

Check compliance-config.md → Vanta → API Access.

**If "yes":**
1. Format evidence as documents for Vanta upload
2. Upload via `POST /evidence` endpoint
3. Link to relevant controls via `POST /controls/{id}/evidence`
4. Report: "Evidence uploaded to Vanta: [N] documents linked to [N] controls"

**If "no" (UI-only):**
1. Save evidence as local files in `~/.claude/memory/context/evidence-exports/`
2. Use naming convention: `{date}_{control-id}_{description}.md`
3. Report: "Evidence saved locally for manual Vanta upload: [file list]"

## Step 7: Clear Evidence Buffer

After processing, clear the evidence buffer:
- Archive processed entries (append to `~/.claude/memory/context/evidence-archive.md` with processing date)
- Clear `~/.claude/memory/context/evidence-buffer.md`
- Report: "Evidence buffer cleared. [N] entries archived."
