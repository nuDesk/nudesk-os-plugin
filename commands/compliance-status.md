---
description: Compliance dashboard — single view across Vanta and Asana compliance projects
allowed-tools: Read, Bash, Glob, Grep, mcp__plugin_asana_asana__asana_search_tasks, mcp__plugin_asana_asana__asana_get_task, mcp__plugin_asana_asana__asana_get_project, mcp__plugin_asana_asana__asana_get_project_task_counts
---

Single-pane compliance dashboard. Pulls data from Asana compliance projects and Vanta (when API available), cross-references with the 91-control matrix, and surfaces gaps.

## Step 1: Load Compliance Config

Read `~/.claude/memory/compliance-config.md` to load:
- Vanta plan and API access status
- Asana compliance project GIDs (Production Change Log, Incident Response Log, Risk Register)
- Custom field GIDs
- Scheduled review dates

If the file doesn't exist or has only placeholders: stop and tell the user to run `/os-setup` first.

## Step 2: Query Asana Compliance Projects

Run these queries in parallel using Asana MCP:

### 2a. Production Change Log

Use `asana_search_tasks` against the Production Change Log project GID:
- **Recent changes (last 30 days):** `completed_at_after` = 30 days ago, `completed` = true
- **Open changes (in progress):** `completed` = false
- For each, request `opt_fields`: name, completed, completed_at, custom_fields, subtasks

Analyze:
- Total changes logged in last 30 days
- Changes with **incomplete subtask checklists** (change management checklist not fully checked)
- Changes **missing Control ID** custom field
- Changes still open (in-progress deploys)

### 2b. Incident Response Log

Use `asana_search_tasks` against the Incident Response Log project GID:
- **Open incidents:** `completed` = false
- Request `opt_fields`: name, custom_fields, created_at, subtasks

Categorize by Severity custom field: P0, P1, P2, P3.

### 2c. Risk Register

Use `asana_search_tasks` against the Risk Register project GID:
- **Active risks:** `completed` = false
- Request `opt_fields`: name, custom_fields, notes

Categorize by Risk Level custom field: Critical, High, Medium, Low.

## Step 3: Query Vanta (if API available)

Check compliance-config.md → Vanta → API Access.

**If "yes":** Query Vanta REST API for:
1. `GET /controls` — overall control pass/fail percentage
2. `GET /tests` — failing automated tests (filter to failed status)
3. `GET /evidence` — evidence docs older than 90 days (staleness check)
4. `GET /people` — access review completion percentage
5. `GET /vulnerabilities` — open vulns by severity

**If "no" or MCP/API unavailable:** Skip. Note "Vanta data unavailable — operating in export mode" in the output.

## Step 4: Check Scheduled Reviews

Read the Scheduled Reviews table from compliance-config.md.

For each review:
- Calculate days until next due date
- Flag as **OVERDUE** if past due
- Flag as **DUE SOON** if within 14 days
- Note which system owns the review (Vanta or Asana)

## Step 5: Present Dashboard

```
COMPLIANCE STATUS — [Today's Date]

PRODUCTION CHANGE LOG
  Changes logged (30 days):    [N]
  Incomplete checklists:       [N] ⚠️ (if > 0)
  Missing Control IDs:         [N] ⚠️ (if > 0)
  In-progress changes:         [N]

INCIDENT RESPONSE
  Open incidents:              [N]
    P0 (Critical):             [N] 🔴 (if > 0)
    P1 (High):                 [N] 🟠 (if > 0)
    P2 (Medium):               [N]
    P3 (Low):                  [N]

RISK REGISTER
  Active risks:                [N]
    Critical:                  [N] 🔴 (if > 0)
    High:                      [N] 🟠 (if > 0)
    Medium:                    [N]
    Low:                       [N]

VANTA (if available)
  Controls passing:            [N]% ([passing]/[total])
  Failing tests:               [N]
  Stale evidence (>90 days):   [N]
  Access review completion:    [N]%
  Open vulnerabilities:        [N] (Critical: [N], High: [N])

— OR —

VANTA
  Status: UI-only — data not available via API
  Recommendation: Check Vanta dashboard manually for control test results

SCHEDULED REVIEWS
  🔴 OVERDUE:     [Review name] — due [date] ([N] days overdue) — [System]
  🟡 DUE SOON:    [Review name] — due [date] ([N] days) — [System]
  ✅ ON TRACK:     [Review name] — due [date] ([N] days) — [System]
```

## Step 6: Recommended Actions

Based on the dashboard data, suggest up to 5 prioritized actions:

1. **Critical items first:** Open P0/P1 incidents, overdue reviews, Critical/High risks
2. **Compliance gaps:** Incomplete change checklists, missing Control IDs
3. **Evidence staleness:** Items needing refresh
4. **Upcoming deadlines:** Reviews due within 14 days

For each action, offer:
- "Create Asana task for this?" (if actionable)
- "Run `/evidence-collect` to gather evidence?" (if evidence-related)
- "Run `/incident-log` to document?" (if incident-related)
- "Generate report for Vanta upload?" (if Vanta is UI-only)
