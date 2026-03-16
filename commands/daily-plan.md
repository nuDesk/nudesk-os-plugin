---
description: Generate a prioritized daily plan from Asana, Calendar, Gmail, Fireflies, and HubSpot
allowed-tools: Read, Grep, Glob, Bash, Write, Edit
---

Generate the user's prioritized daily plan. This is the morning command — scan all sources, synthesize, and deliver a ranked action list with effort tags.

## Step 1: Load Context

Read `~/.claude/CLAUDE.md` to load:
- **User profile** — name, email, role from the "Who I Am" section
- **Current priorities** — from the "Priorities" section (strategic lens for ranking)
- **Working Memory** — people, terms, clients from the "Working Memory" section
- **Integration config** — email address, HubSpot owner ID, and other service identifiers

Pay special attention to the "Priorities" section — these are the strategic lens for ranking everything.

## Step 2: Pull Data from All Sources (in parallel where possible)

### Asana — Today's Tasks + Overdue
Load Asana GIDs from `~/.claude/memory/asana-config.md` for all workspace, user, and custom field references.

Search for tasks assigned to me:
- Due today or overdue (`due_on_before`: today, `completed`: false)
- Use `asana_search_tasks` with `assignee_any`: "me", workspace GID from config
- Include opt_fields: name, due_on, projects.name, custom_fields

Also check for tasks in "Agent Queue" status (these should be called out separately as automatable).

### Google Calendar — Today's Schedule
Use the **`gws` CLI** via Bash (NOT claude.ai connectors). Run:
```
gws calendar events list --params '{"calendarId": "primary", "timeMin": "<today>T00:00:00Z", "timeMax": "<today>T23:59:59Z", "maxResults": 25, "singleEvents": true, "orderBy": "startTime"}'
```

Map committed time blocks and note gaps available for focused work.

**If auth fails:** Note "Calendar data unavailable — run `gws auth login -s calendar` to re-auth."

### Gmail — Flagged/Important Unread
Use the **`gws` CLI** via Bash (NOT claude.ai connectors). Run:
```
gws gmail users messages list --params '{"userId": "me", "q": "is:important is:unread newer_than:1d", "maxResults": 15}'
```
Then fetch individual message details with:
```
gws gmail users messages get --params '{"userId": "me", "id": "<messageId>"}'
```

Prioritize messages from known contacts listed in CLAUDE.md Working Memory. Flag anything that looks time-sensitive or requires a response.

**If auth fails:** Note "Gmail data unavailable — run `gws auth login -s gmail` to re-auth."

### Fireflies — Recent Meeting Action Items
Use `fireflies_get_transcripts` for the past 2 days. Extract action items assigned to the user from meeting summaries. These represent commitments that need follow-through.

### Commitment Cross-Reference (Fireflies vs Asana)

After gathering both Fireflies action items and Asana tasks, compare them:

1. For each Fireflies action item, check if a matching Asana task already exists (match by keyword/topic — exact title match is not required)
2. Separate into two lists:
   - **Tracked**: Action items that have a corresponding Asana task (include in priority ranking as-is)
   - **Uncaptured**: Action items with no matching Asana task (these are commitments at risk of being dropped)

If uncaptured commitments are found, they will be surfaced in a dedicated section of the daily plan output (see Step 4) and offered for quick task creation in Step 5.

**If Fireflies MCP is not available:** Skip this cross-reference and continue.

### HubSpot — My Deals & Tasks Only
Use HubSpot MCP tools filtered to the user's HubSpot owner ID (from CLAUDE.md). Do NOT show deals or tasks owned by other team members.

**Deals:** Search for open deals where `hubspot_owner_id` = user's owner ID and dealstage is not closedwon/closedlost. Include properties: dealname, dealstage, closedate, amount, notes_last_updated. Sort by closedate ascending.

**Tasks:** Search for HubSpot tasks (objectType: `tasks`) where `hubspot_owner_id` = user's owner ID and `hs_task_status` != `COMPLETED`. Include properties: hs_task_subject, hs_task_status, hs_timestamp, hs_task_priority.

Keep this lightweight — only surface items needing attention.

## Step 3: Synthesize and Rank

Apply the 4-tier prioritization framework below. Cross-reference all sources against CLAUDE.md priorities.

### Tier 1: Non-Negotiable (do first)
- Calendar commitments (meetings, calls)
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
- **Calendar density** — Heavy meeting days mean only quick tasks fit in gaps
- **Energy management** — Front-load strategic/creative work, back-load admin
- **Stakeholder urgency** — Requests from key contacts (listed in CLAUDE.md) get elevated; internal admin gets deferred
- **Momentum** — If a project has been stalled, consider prioritizing to unblock

### Synthesis Order
1. **Commitments with deadlines** — meetings, deliverables due today
2. **Strategic priority items** — tasks aligned with top CLAUDE.md priorities
3. **Relationship maintenance** — emails/follow-ups with key stakeholders
4. **Pipeline items** — BD or HubSpot items needing movement
5. **Operational tasks** — admin, routine items

## Step 4: Present the Daily Plan

Output format — clean, scannable, no fluff:

```
DAILY PLAN — [Today's Date]

CALENDAR
[List today's meetings with times, or "No calendar data available"]

PRIORITY ACTIONS (ranked)
1. [Task/action] — [source] — [effort: 15m/30m/1hr/2hr/deep work]
2. [Task/action] — [source] — [effort tag]
3. ...

EMAILS NEEDING RESPONSE
- [Sender]: [Subject snippet] — [urgency: now/today/this week]

UNCAPTURED COMMITMENTS (from meetings — no Asana task found)
- [Action item] — [Meeting name, date] — Want me to create a task?

AGENT QUEUE (automatable)
- [Tasks that can be run via /run-tasks]

WATCH LIST (not urgent but track)
- [Items to keep on radar]
```

Keep the total list to **no more than 10 priority actions**. If there are more, group lower-priority items under "Watch List."

## Step 5: Interactive Handoff

After presenting the plan:

1. If uncaptured commitments were found, prompt:

   "Found [N] meeting commitments without Asana tasks. Want me to create tasks for these? (all / select numbers / skip)"

   If approved: Create tasks using the same pattern as `/log-task` — smart project routing from `asana-config.md`, set Task Progress to "Not Started", Type and Priority based on context. Set due dates based on urgency (commitments from today/yesterday = due today, older = due tomorrow).

2. If Agent Queue tasks exist, prompt:

   "You have [N] tasks in Agent Queue. Want me to run /run-tasks as a background agent while you work on your priority items?"

   If yes: Launch /run-tasks via the Agent tool with run_in_background=true, then continue the interactive session with the user on their priority actions.

3. Ask: "Which task would you like to start with?"

This allows Claude to help work through tasks conversationally — whether that's drafting an email, researching a topic, or executing an Asana task.
