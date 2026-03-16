---
description: Strategic planning review — 7-30 day horizon with undated task audit
allowed-tools: Read, Write, Edit, Bash, Grep, Glob
---

Run the user's strategic review. This is the weekly/bi-weekly command for strategic planning — covers the 7-30 day horizon, audits undated tasks, and synthesizes strategic themes. Complements /daily-plan (today) and /weekly-report (retrospective).

## Step 1: Load Strategic Context

Read `~/.claude/CLAUDE.md` to load:
- **User profile** — name, email, role from the "Who I Am" section
- **Strategic Goals** — from the goals section
- **Current Priorities** (with `last_updated` date)
- **Working Memory** — people, terms, clients
- **Integration config** — HubSpot owner ID, email addresses

Read `~/.claude/memory/asana-config.md` for all workspace, user, custom field, and project GIDs.

Confirm with the user: "Running strategic review for the next 30 days ([today] - [today+30]). Adjust the window?"

Wait for confirmation before proceeding.

## Step 2: Upcoming Task Landscape (7-30 days)

Pull tasks due in the next 30 days using `asana_search_tasks`:
- `assignee_any`: "me"
- `completed`: false
- `due_on_after`: today
- `due_on_before`: today + 30 days
- `workspace`: workspace GID from config
- `opt_fields`: name, due_on, projects.name, custom_fields, resource_subtype, notes

### Milestone Tasks

Run a second query with `resource_subtype=milestone` to separately surface Asana milestones:
- Same filters as above but add `resource_subtype`: milestone

Present milestones prominently at the top as "Strategic Markers" before the weekly breakdown.

### Weekly Breakdown

Group remaining tasks by week:
- **Week 1** (days 1-7): [count] tasks — [key items]
- **Week 2** (days 8-14): [count] tasks — [key items]
- **Week 3** (days 15-21): [count] tasks — [key items]
- **Week 4** (days 22-30): [count] tasks — [key items]

### Strategic Alignment Check

Cross-reference against CLAUDE.md priorities:
- Flag tasks that don't align with any current priority (potential distractions)
- Flag priority areas with zero tasks in the 30-day window (gaps in execution)

## Step 3: Undated Task Audit

This is the key differentiator of strategic-review — surfacing tasks that accumulate without dates.

### Query

Use `asana_search_tasks`:
- `assignee_any`: "me"
- `completed`: false
- `workspace`: workspace GID from config
- `opt_fields`: name, due_on, modified_at, projects.name, custom_fields, notes

Filter client-side to tasks where `due_on` is null. Cap at 100 tasks for manageability.

### Categorize Each Task

| Category | Criteria | Suggested Action |
|----------|----------|-----------------|
| **Relevant — Add Date** | Modified <30 days ago AND aligns with a CLAUDE.md priority | Suggest due date based on priority tier: P1 = +7 days, P2-3 = +14 days, P4-6 = +21-30 days |
| **Stale — Archive** | Modified >30 days ago AND no priority alignment | Set Task Progress to "Deferred" (use GID from asana-config.md), add comment explaining why |
| **Misaligned — Delegate** | Belongs to another person's scope (check CLAUDE.md Working Memory for team) | Recommend reassign or defer |
| **Unclear** | Can't determine relevance from name/notes | Ask user for guidance |

### Present Recommendations

Group by category and present:

```
UNDATED TASK AUDIT — [X] tasks found

RELEVANT — ADD DATE ([N] tasks)
- [Task name] → Suggested: [date] — Reason: [priority alignment]
- ...

STALE — ARCHIVE ([N] tasks)
- [Task name] — Last modified: [date] — Action: Set to Deferred
- ...

MISALIGNED — DELEGATE ([N] tasks)
- [Task name] → Recommend: [reassign to X / defer]
- ...

UNCLEAR ([N] tasks)
- [Task name] — Need your input
- ...
```

Then ask: "Approve all recommendations, by category, or review one by one?"

### Execute Approved Changes

For approved tasks:
- **Add Date**: Use `asana_update_task` to set `due_on`
- **Archive**: Use `asana_update_task` to set Task Progress custom field to "Deferred" (GID from asana-config.md), then `asana_create_task_story` to add a comment: "Deferred during strategic review [date] — [reason]"
- **Delegate**: Use `asana_update_task` to reassign or defer as directed

Track changes made for the dashboard summary.

## Step 4: Calendar Density (next 2-4 weeks)

Use the **`gws` CLI** via Bash. Run:
```
gws calendar events list --params '{"calendarId": "primary", "timeMin": "<today>T00:00:00Z", "timeMax": "<today+28d>T23:59:59Z", "maxResults": 100, "singleEvents": true, "orderBy": "startTime"}'
```

### Analysis by Week

For each of the next 4 weeks, calculate:
- **Total meeting hours**
- **Heavy days** (>4 hours of meetings)
- **Deep work days** (<2 hours of meetings)

### Deep Work Windows

Identify the best 2-3 deep work blocks in the next 2 weeks — these are half-day or full-day slots with minimal meetings, ideal for Tier 2 strategic work.

**If auth is needed:** Present the authorization URL and note "Calendar data unavailable — Google Workspace MCP needs re-auth." Continue without this section.

## Step 5: Pipeline Health (HubSpot)

**IMPORTANT: Only surface deals and tasks owned by the user.** Use the `hubspot_owner_id` from CLAUDE.md to filter all queries.

### Deals

Use `search_crm_objects` with:
- `objectType`: deals
- `filterGroups`: filter on `hubspot_owner_id` = user's owner ID AND `dealstage` NOT IN closedwon, closedlost
- `properties`: dealname, dealstage, closedate, amount, notes_last_updated

Flag:
- **Closing soon**: Deals with close date in the next 30 days
- **Stale**: Deals where `notes_last_updated` is >14 days ago
- **High value**: Deals with amount >$50K

### Tasks

Use `search_crm_objects` with:
- `objectType`: tasks
- `filterGroups`: filter on `hubspot_owner_id` = user's owner ID AND `hs_task_status` != COMPLETED
- `properties`: hs_task_subject, hs_task_status, hs_timestamp, hs_task_priority

Flag overdue tasks (where `hs_timestamp` is in the past).

**If HubSpot MCP is not available:** Note "HubSpot data unavailable" and continue.

## Step 6: Commitment Tracking (Fireflies)

Use `fireflies_get_transcripts` for the past 7 days. Filter to meetings where the user participated (match against user's email addresses from CLAUDE.md).

### Extract

For each relevant meeting:
- Meeting title and date
- Action items assigned to the user
- Decisions made that may need follow-through

### Cross-Reference

Compare Fireflies action items against:
- Asana tasks due in next 30 days (from Step 2)
- Undated tasks (from Step 3)

Flag **uncaptured commitments** — action items from meetings that don't have a corresponding Asana task. Offer to create tasks for each:

"Found [N] commitments from meetings without matching Asana tasks. Want me to create tasks for these?"

If yes: Use the same task creation pattern as `/log-task` (smart routing, custom fields, etc.).

**If Fireflies MCP is not available:** Note "Fireflies data unavailable" and continue.

## Step 7: Strategic Theme Synthesis

Synthesize everything gathered (Steps 2-6) into 3-5 strategic focus themes for the next 2-4 weeks.

### For Each Theme

- **Theme name**: Clear, action-oriented label
- **Connected goal**: Which strategic goal does this serve (from CLAUDE.md)
- **Key actions**: 2-4 specific next steps
- **Effort tag**: Quick / Half-day / Multi-day / Ongoing
- **Source attribution**: Where this theme emerged from (Asana, HubSpot, Fireflies, etc.)
- **Suggested deep work window**: From calendar density analysis (Step 4)

### Anti-Patterns

- **Max 5 themes** — Executive time is the scarcest resource. Ruthlessly prioritize.
- **Each theme must connect to a strategic goal** — if it doesn't, it goes to a parking lot.
- **Don't duplicate daily-plan scope** — themes should be about the 7-30 day horizon, not today's tasks.

## Step 8: Present Complete Dashboard

Output format — clean, scannable, strategic:

```
STRATEGIC REVIEW — [Start Date] to [End Date]

STRATEGIC MARKERS (Milestones)
- [Milestone]: [Due date] — [Project]

1. TASK LANDSCAPE (30-day view)
   Week 1: [count] tasks | Week 2: [count] | Week 3: [count] | Week 4: [count]
   Alignment gaps: [priorities with no tasks]
   Distractions: [tasks not matching any priority]

2. UNDATED TASK AUDIT
   [N] total undated tasks found
   - [N] to date | [N] to archive | [N] to delegate | [N] unclear
   Status: [awaiting approval / X changes made]

3. CALENDAR DENSITY
   [Week-by-week meeting load summary]
   Best deep work windows: [dates/times]

4. PIPELINE HEALTH
   [N] active deals | [N] closing in 30 days | [N] stale
   [N] overdue HubSpot tasks

5. UNCAPTURED COMMITMENTS
   [N] action items from meetings without Asana tasks
   Status: [awaiting action / X tasks created]

6. STRATEGIC THEMES
   Theme 1: [Name] — [Goal] — [Effort] — [Source]
     Actions: [key next steps]
     Window: [suggested deep work slot]
   Theme 2: ...
   ...
```

## Step 9: Interactive Execution

After presenting the dashboard, offer to:

1. **Execute undated task recommendations** — process the approved changes from Step 3
2. **Create tasks for uncaptured commitments** — from Step 6
3. **Dive deeper into any theme** — expand on strategic actions, draft communications, etc.

Ask: "What would you like to act on first?"

### Scope Boundaries

This command DOES:
- Modify individual Asana tasks (dates, archive, reassign) with user approval
- Create new Asana tasks for uncaptured commitments
- Provide strategic recommendations

This command does NOT:
- Post reports to Asana (that's /weekly-report's job)
- Update CLAUDE.md priorities (that's /weekly-report's job)
- Handle today's operational tasks (that's /daily-plan's job)
