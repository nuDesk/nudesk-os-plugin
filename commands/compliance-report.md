---
description: Generate a full audit-ready compliance report — all 91 controls with status, evidence, and enforcement mechanisms
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, mcp__plugin_asana_asana__asana_search_tasks, mcp__plugin_asana_asana__asana_get_task, mcp__plugin_asana_asana__asana_get_project, mcp__plugin_asana_asana__asana_get_project_task_counts
---

Generate a comprehensive SOC 2 compliance report covering all 91 controls across 16 policy areas. Pulls live data from Asana and Vanta (when available), cross-references with the control-action map, and produces an audit-ready document.

## Step 1: Load Configuration

Read these files in parallel:
- `~/.claude/memory/compliance-config.md` — project GIDs, Vanta status, review schedule
- `~/.claude/memory/asana-config.md` — workspace and user GIDs
- `~/Projects/nudesk-os-plugin/references/security/control-action-map.md` — enforcement mechanisms
- `~/Projects/nudesk-os-plugin/knowledge-base/Policies/_Summary of nuDesk Sec Policies.md` — 91-control reference

If compliance config is missing: stop and direct user to `/os-setup`.

## Step 2: Gather Data from Asana

### 2a. Production Change Log

Query the Production Change Log project:
- Total tasks (all time and last 90 days)
- Tasks with completed subtask checklists vs. incomplete
- Tasks by Change Type (Feature, Bugfix, Hotfix, Config, Infrastructure)
- Tasks missing Control ID field

### 2b. Incident Response Log

Query the Incident Response Log project:
- Total incidents (all time and last 90 days)
- Open incidents by severity
- Closed incidents with all 6 phases completed vs. partially completed
- Average time to close by severity (if data available)

### 2c. Risk Register

Query the Risk Register project:
- Active risks by Risk Level
- Risks with treatment plans (description populated) vs. without
- Recently closed risks (last 90 days)

## Step 3: Gather Data from Vanta (if available)

Check compliance-config.md → Vanta → API Access.

**If "yes":**
1. `GET /controls` — all controls with pass/fail status
2. `GET /tests` — automated test results (passing, failing, not configured)
3. `GET /evidence` — evidence inventory with freshness dates
4. `GET /people` — personnel compliance status (training, acknowledgments)
5. `GET /vulnerabilities` — open vulnerabilities by severity

**If "no":** Skip. Note "Vanta data based on last manual check" in report.

## Step 4: Cross-Reference Controls

For each of the 91 controls, determine status:

| Status | Criteria |
|--------|----------|
| **Passing** | Evidence exists, enforcement active, no open findings |
| **Partial** | Some evidence or enforcement, but gaps identified |
| **Failing** | No evidence, enforcement not active, or open findings |
| **Not Applicable** | Control doesn't apply to current operations |
| **Policy-Only** | Category C control — enforcement is organizational, not technical |

Use the control-action map to determine what evidence/enforcement should exist for each control.

## Step 5: Calculate Audit Readiness Score

```
Audit Readiness = (Evidence Score × 0.4) + (Review Score × 0.3) + (Change Documentation Score × 0.3)

Evidence Score = controls with current evidence / total controls requiring evidence
Review Score = on-time reviews completed / total reviews scheduled
Change Documentation Score = changes with complete checklists / total changes logged
```

## Step 6: Draft Report

Generate the report with these sections:

### Executive Summary
- Overall audit readiness score (percentage)
- Controls passing / partial / failing / policy-only
- Key risk areas (top 3-5)
- Recommendation summary

### Control Status by Policy Area

For each of the 16 policy areas:
```
[POLICY AREA NAME] — [N] controls
  Passing: [N]  Partial: [N]  Failing: [N]  Policy-Only: [N]

  Failing/Partial controls:
  - [Control ID]: [Status] — [Issue] — [Recommended action]
```

Policy areas: Access Control, Asset Management, Business Continuity/DR, Code of Conduct, Cryptography, Data Management, Human Resource Security, Incident Response, Information Security, Operations Security, Physical Security, Risk Management, Secure Development, Third-Party Management, AI Governance

### Evidence Inventory

| Control ID | Evidence Type | Last Updated | Source | Freshness |
|-----------|--------------|-------------|--------|-----------|
| OS-01 | Change Log | [date] | Asana | Current / Stale |
| SD-01 | PR history | [date] | GitHub | Current / Stale |
| ... | ... | ... | ... | ... |

Flag evidence older than 90 days as "Stale."

### Review Calendar

| Review | Frequency | Last Completed | Next Due | Status | System |
|--------|-----------|---------------|----------|--------|--------|
| Quarterly Access Review | Quarterly | [date] | [date] | On Track / Due Soon / Overdue | Vanta |
| ... | ... | ... | ... | ... | ... |

### Remediation Backlog

Prioritized list of items needing action:

| Priority | Control | Issue | Recommended Action | Owner |
|----------|---------|-------|-------------------|-------|
| Critical | [ID] | [Issue] | [Action] | [Person/Team] |
| High | [ID] | [Issue] | [Action] | [Person/Team] |
| ... | ... | ... | ... | ... |

### Audit Readiness Score Breakdown

```
AUDIT READINESS: [N]%

  Evidence freshness:          [N]% ([N]/[N] controls with current evidence)
  Review completeness:         [N]% ([N]/[N] reviews on schedule)
  Change documentation:        [N]% ([N]/[N] changes with complete checklists)

  Trend: [Improving / Stable / Declining] (compared to last report if available)
```

## Step 7: Present and Confirm

Present the full report to the user for review.

Call `AskUserQuestion`: "Here's the compliance report. Options:\n1. **Save to Google Drive** (via `gws`) for sharing\n2. **Create Asana remediation tasks** for failing/partial controls\n3. **Export for Vanta upload** (if UI-only mode)\n4. **All of the above**\n5. **Just save locally**"

**→ Call `AskUserQuestion`. Your turn ends here.**

## Step 8: Execute Chosen Actions

Based on user's choice:

### Save to Google Drive
```bash
# Create the report as a Google Doc
gws docs documents create --json '{"title": "nuDesk SOC 2 Compliance Report — [date]"}'
# Insert report content
gws docs documents batchUpdate --params '{"documentId": "[doc-id]"}' --json '{"requests": [...]}'
```

### Create Remediation Tasks
For each failing/partial control, create an Asana task:
- Project: appropriate compliance project (Change Log, Incident Response, or Risk Register)
- Title: `[Control ID] Remediation: [brief description]`
- Description: issue details and recommended action
- Due date: based on priority (Critical: 7 days, High: 30 days, Medium: 90 days)

### Export for Vanta
Save as markdown file: `~/.claude/memory/context/vanta-exports/{date}_compliance-report.md`

### Save Locally
Save to: `~/.claude/memory/context/compliance-reports/{date}_compliance-report.md`
