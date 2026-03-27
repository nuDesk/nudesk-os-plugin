---
description: Generate a prioritized daily plan from Asana, Calendar, Fireflies, and HubSpot
allowed-tools: Read, Grep, Glob, Bash, Write, Edit
---

Generate the user's prioritized daily plan. This is the morning command -- scan all sources, synthesize, and deliver a ranked action list with effort tags.

The command runs in two phases: first surface calendar and uncaptured commitments (pause for input), then present the full prioritized action list.

## Step 1: Load Context

Read `~/.claude/CLAUDE.md` to load:
- **User profile** -- name, email, role from the "Who I Am" section
- **Current priorities** -- from the "Priorities" section (strategic lens for ranking)
- **Working Memory** -- people, terms, clients from the "Working Memory" section
- **Integration config** -- email address, HubSpot owner ID, and other service identifiers

Pay special attention to the "Priorities" section -- these are the strategic lens for ranking everything.

## Step 2: Pull Data from All Sources (in parallel where possible)

### Asana -- Today's Tasks + Overdue
Load Asana GIDs from `~/.claude/memory/asana-config.md` for all workspace, user, and custom field references.

Search for tasks assigned to me:
- Due today or overdue (`due_on_before`: today, `completed`: false)
- Use `asana_search_tasks` with `assignee_any`: "me", workspace GID from config
- Include opt_fields: name, due_on, projects.name, custom_fields

Also check for tasks in "Agent Queue" status (these should be called out separately as automatable).

### Google Calendar -- Today's Schedule
Use the **`gws` CLI** via Bash (NOT claude.ai connectors). Run:
```
gws calendar events list --params '{"calendarId": "primary", "timeMin": "<today>T00:00:00Z", "timeMax": "<today>T23:59:59Z", "maxResults": 25, "singleEvents": true, "orderBy": "startTime"}'
```

Map committed time blocks and note gaps available for focused work.

**If auth fails:** Note "Calendar data unavailable -- run `gws auth login -s calendar` to re-auth."

### Fireflies -- Recent Meeting Action Items
Use `fireflies_get_transcripts` for the past 2 days. Extract action items assigned to the user from meeting summaries. These represent commitments that need follow-through.

### Commitment Cross-Reference (Fireflies vs Asana)

After gathering both Fireflies action items and Asana tasks, compare them:

1. For each Fireflies action item, check if a matching Asana task already exists (match by keyword/topic -- exact title match is not required)
2. Separate into two lists:
   - **Tracked**: Action items that have a corresponding Asana task (include in priority ranking as-is)
   - **Uncaptured**: Action items with no matching Asana task (these are commitments at risk of being dropped)

**If Fireflies MCP is not available:** Skip this cross-reference and continue.

### HubSpot -- My Deals & Tasks Only
Use HubSpot MCP tools filtered to the user's HubSpot owner ID (from CLAUDE.md). Do NOT show deals or tasks owned by other team members.

**Deals:** Search for open deals where `hubspot_owner_id` = user's owner ID and dealstage is not closedwon/closedlost. Include properties: dealname, dealstage, closedate, amount, notes_last_updated. Sort by closedate ascending.

**Tasks:** Search for HubSpot tasks (objectType: `tasks`) where `hubspot_owner_id` = user's owner ID and `hs_task_status` != `COMPLETED`. Include properties: hs_task_subject, hs_task_status, hs_timestamp, hs_task_priority.

Keep this lightweight -- only surface items needing attention.

### Compliance Quick Check

After pulling all data sources, run a lightweight compliance check (2-3 Asana queries max).

Load `~/.claude/memory/compliance-config.md`. If it exists with real GIDs:

1. **Scheduled reviews due within 7 days?** Read the Scheduled Reviews table and compare dates to today.
2. **Incomplete production change records?** `asana_search_tasks` against Production Change Log -- count tasks with `completed` = false (open changes without completed checklists).
3. **Open incidents?** `asana_search_tasks` against Incident Response Log -- count/severity of open incidents.

If compliance-config.md is not configured: skip this check entirely. Do not mention it.

---

## Step 3: Synthesize and Rank

Apply the 4-tier prioritization framework below. Cross-reference all sources against CLAUDE.md priorities.

### Tier 1: Non-Negotiable (do first)
- Client deliverable deadlines
- Commitments made in recent meetings (Fireflies action items)
- Anything blocking someone else's work

### Tier 2: Strategic (do today)
- Items aligned with CLAUDE.md top priorities
- Revenue-impacting activities (pipeline, BD, client relationships)
- Tasks that have been overdue >2 days

### Tier 3: Important (schedule this week)
- Operational tasks (invoicing, compliance, admin)
- Internal process improvements
- Content and marketing activities

### Tier 4: Delegate or Defer
- Tasks assignable to team members (check CLAUDE.md Working Memory for team)
- Low-urgency follow-ups
- Nice-to-have improvements

### Effort Tags

Assign one tag to each action item:

| Tag | Duration | Type |
|-----|----------|------|
| 15m | Under 15 minutes | Quick reply, approval, review |
| 30m | 15-30 minutes | Short task, email draft, brief call |
| 1hr | 30-60 minutes | Focused task, document review, planning |
| 2hr | 1-2 hours | Deep deliverable, analysis, strategy |
| Deep | 2+ hours | Major project work, complex creation |

### Context Awareness

When synthesizing, account for:
- **Calendar density** -- Heavy meeting days mean only quick tasks fit in gaps
- **Energy management** -- Front-load strategic/creative work, back-load admin
- **Stakeholder urgency** -- Requests from key contacts (listed in CLAUDE.md) get elevated; internal admin gets deferred
- **Momentum** -- If a project has been stalled, consider prioritizing to unblock
- **Capacity** -- If total effort of Tier 1 + Tier 2 items exceeds available non-meeting hours, recommend deferring Tier 3/4 items to specific later days this week

### Synthesis Order
1. **Commitments with deadlines** -- deliverables due today
2. **Strategic priority items** -- tasks aligned with top CLAUDE.md priorities
3. **Pipeline items** -- BD or HubSpot items needing movement
4. **Operational tasks** -- admin, routine items

---

## Step 4: Present Phase A -- Calendar + Uncaptured Commitments

Present this first and **stop to wait for user input** before continuing to Phase B.

Output format:

```
DAILY PLAN -- [Today's Date]

CALENDAR
[List today's meetings with times and attendees. No prep tasks -- just the schedule. Or "No calendar data available"]

UNCAPTURED COMMITMENTS (from meetings -- no Asana task found)
1. [Action item] -- [Meeting name, date]
2. ...
[Or "None found"]
```

If uncaptured commitments exist, prompt:

"Found [N] meeting commitments without Asana tasks. Want me to create tasks for these? (all / select numbers / skip)"

If approved: Create tasks using the same pattern as `/log-task` -- smart project routing from `asana-config.md`, set Task Progress to "Not Started", Type and Priority based on context. Set due dates based on urgency (commitments from today/yesterday = due today, older = due tomorrow).

**Wait for the user's response before proceeding to Step 5.**

## Step 5: Present Phase B -- Priority Actions

After the user responds to uncaptured commitments, merge all sources into one ranked list: Asana tasks, newly created commitment tasks, HubSpot items, and CLAUDE.md priorities.

Output format:

```
PRIORITY ACTIONS (ranked)
1. [Task/action] -- [source] -- [effort: 15m/30m/1hr/2hr/deep]
2. [Task/action] -- [source] -- [effort tag]
3. ...

AGENT QUEUE (automatable)
- [Tasks that can be run via /run-tasks]
[Or omit section if none]

COMPLIANCE
- Reviews due within 7 days: [list or "None"]
- Incomplete change records: [N]
- Open incidents: [count by severity or "None"]
[Or omit section if compliance-config.md not configured]
```

Keep the total list to **no more than 10 priority actions**. If there are more than 10, drop the lowest-ranked items entirely (do not create a separate overflow section).

If the day looks overloaded (total effort exceeds available non-meeting hours), explicitly recommend which Tier 3/4 items to push later in the week and suggest specific days.

If priority is unclear between items, use AskUserQuestion to clarify before finalizing the list.

## Step 6: Confirm & Load Task List

1. After presenting the Priority Actions table, prompt:

   "Any edits to the priority list? (reorder / add / remove / change effort tags / looks good)"

   Wait for the user's response. If they request changes, update the table and re-present it. Repeat until the user confirms the list is ready (e.g., "looks good", "confirmed", "good to go", etc.).

   **Moving tasks to other days:** If the user moves a task to a different day (e.g., "push #5 to Monday", "do #3 tomorrow"), remove it from today's table entirely -- do not keep it with an updated date. Update the Asana due date if the task came from Asana, then re-present the shortened table.

2. Once confirmed, create a TaskCreate entry for each priority action in the confirmed table. Use the task name and effort tag from the table (e.g., "Draft Champions status email [30m]"). Preserve the ranked order.

3. If Agent Queue tasks exist, prompt:

   "You have [N] tasks in Agent Queue. Want me to run /run-tasks as a background agent while you work on your priority items?"

   If yes: Launch /run-tasks via the Agent tool with run_in_background=true, then continue the interactive session with the user on their priority actions.

4. Ask: "Which task would you like to start with?"

This allows Claude to help work through tasks conversationally -- whether that's drafting an email, researching a topic, or executing an Asana task.
