---
description: Generate a weekly executive progress report
allowed-tools: Read, Write, Edit, Bash, Grep, Glob
---

Generate the user's weekly executive progress report. Full inline workflow — gather data, draft, and post to Asana.

## Step 1: Load User Config

Read `~/.claude/CLAUDE.md` to load:
- **User name and email** from "Who I Am"
- **Report recipient** — the person tagged on weekly reports (defined in Working Memory or Preferences)
- **Strategic goals** — from the goals/priorities sections
- **Working Memory** — people, terms, clients for context

Read `~/.claude/memory/asana-config.md` to load all Asana GIDs.

## Step 2: Determine Reporting Period

Calculate the reporting period based on the current day:

**If today is Sunday:**
- Report covers: Monday of current week through today (Sunday)

**If today is Monday:**
- Report covers: Previous Monday through previous Sunday

**Other days:**
- Report covers: Most recent Monday through most recent Sunday

Confirm with user: "Preparing report for [Start Date] - [End Date]. Correct?"

Wait for confirmation before proceeding.

## Step 3: Gather Prior Week Goals

Check if previous report goals are available:
- Search Asana for the most recent weekly executive report task (search by the report task name pattern from user's config)
- Look for the "Upcoming Week Priorities" section from that report

Present to user:
> "Last week you planned to focus on:
> - [Priority 1]
> - [Priority 2]
> - [Priority 3]
>
> Did these hold, or did priorities shift?"

**Fallback:** If no prior report is found, ask: "What were your main goals this week?"

Wait for response before proceeding.

## Step 4: Pull Asana Data

### Completed Tasks
Use `asana_search_tasks` with:
- `completed`: true
- `completed_at_after`: Start of reporting period (ISO 8601)
- `completed_at_before`: End of reporting period (ISO 8601)
- `assignee_any`: "me"
- `workspace`: workspace GID from config
- `opt_fields`: name, completed_at, notes, projects.name

### Upcoming Tasks (Next 7 Days)
Use `asana_search_tasks` with:
- `completed`: false
- `due_on_before`: 7 days from today (ISO 8601)
- `assignee_any`: "me"
- `workspace`: workspace GID from config
- `opt_fields`: name, due_on, projects.name

**Flag concerns:**
- Tasks overdue or at risk of slipping
- Capacity concerns if >15 tasks due in next 7 days

## Step 5: Pull Fireflies Data

Use `fireflies_get_transcripts` with:
- `fromDate`: Start of reporting period
- `toDate`: End of reporting period

Filter results to meetings where the user participated (match against user's email addresses from CLAUDE.md).

Extract from each relevant meeting:
- Meeting title
- Brief summary
- Action items assigned to the user

## Step 6: Ask Clarifying Questions

After gathering automated data, ask **up to 5 clarifying questions, one at a time**:

1. **Key outcomes or decisions** — What got resolved or decided this week?
2. **Strategic shifts or pivots** — Any changes in direction or priorities?
3. **Client relationship updates** — Wins, escalations, or notable interactions?
4. **Blockers or items to flag** — What's stuck or needs attention?
5. **Context gaps** — Anything not obvious from task names or meeting summaries?

Ask questions selectively based on what the automated data doesn't cover.

## Step 7: Synthesize Upcoming Week Priorities

Instead of asking a blank question, synthesize a data-driven priority suggestion from everything gathered so far (Asana tasks, Fireflies action items, user input from Step 6) plus forward-looking signals. This follows the same pattern as `/daily-plan` Step 3 but scoped to the upcoming week.

### Step 7-pre: Check for Recent Strategic Review

Ask: "Did you run a /strategic-review this week?"

**If yes:** Skip Step 7a entirely (Calendar, HubSpot, Gmail pulls are redundant — strategic-review already gathered this data). Use strategic themes from the review as the basis for upcoming week priorities. Proceed directly to Step 7b.

**If no:** Proceed with Step 7a as written (self-sufficient fallback).

### Step 7a: Pull Forward-Looking Signals

Pull three additional data sources to inform next-week priorities. Each has a graceful fallback — never block the report on a missing connector.

#### HubSpot — Upcoming Pipeline Activity
Filter all HubSpot queries to the user's deals/tasks only (use `hubspot_owner_id` from CLAUDE.md).

Use HubSpot MCP tools to check for:
- Deals with close dates in the next 14 days
- Deals with recent activity (last 7 days) that may need follow-up
- Any contacts or companies with tasks due in the next 7 days

Keep this lightweight — only surface items needing CEO attention.

**If HubSpot MCP is not available:** Note "HubSpot data unavailable" and continue.

#### Google Calendar — Next Week's Committed Time
Use the **`gws` CLI** via Bash. Run:
```
gws calendar events list --params '{"calendarId": "primary", "timeMin": "<next-monday>T00:00:00Z", "timeMax": "<next-friday>T23:59:59Z", "maxResults": 100, "singleEvents": true, "orderBy": "startTime"}'
```
Map committed time blocks and identify:
- Meetings requiring prep
- Days with heavy meeting load (limited deep-work capacity)
- Open blocks available for strategic work

**If auth fails:** Note "Calendar data unavailable — run `gws auth login -s calendar` to re-auth." and continue.

#### Gmail — Important Unread/Unanswered
Use the **`gws` CLI** via Bash. Run:
```
gws gmail users messages list --params '{"userId": "me", "q": "is:important newer_than:7d", "maxResults": 20}'
```
Then fetch individual message details with:
```
gws gmail users messages get --params '{"userId": "me", "id": "<messageId>"}'
```

Focus on threads >24hrs old without a response. Prioritize messages from known contacts in CLAUDE.md Working Memory.

**If `gws` auth fails:** Note "Gmail data unavailable" and continue.

### Step 7b: Synthesize Using Executive Planning Framework

Apply the 4-tier prioritization framework below, adapted for weekly scope. Cross-reference all gathered data against CLAUDE.md priorities and strategic goals.

#### Tier 1: Carry-Forward Commitments (must do)
- Unfulfilled Fireflies action items from this week's meetings
- Overdue Asana tasks carrying into next week
- Client promises or deliverable deadlines
- Anything blocking someone else's work

#### Tier 2: Strategic Priorities (should do)
- Items aligned with CLAUDE.md priorities and strategic goals
- Revenue-impacting work (HubSpot deals, BD opportunities)
- Momentum projects that risk stalling without attention
- Key relationship touchpoints (follow-ups with stakeholders)

#### Tier 3: Operational Necessities (schedule it)
- Admin, compliance, invoicing, routine follow-ups
- Internal process improvements
- Operational tasks that have a deadline next week

#### Tier 4: Delegate or Park (assign or defer)
- Items team members should own
- Low-urgency deferrals that can wait another week
- Nice-to-have improvements with no deadline pressure

#### Effort Tags (weekly scope)
Assign one tag to each priority theme:
| Tag | Duration | Example |
|-----|----------|---------|
| Quick | Under 1 hour | Reply, approval, short follow-up |
| Half-day | 2-4 hours | Focused deliverable, strategy session |
| Multi-day | Spans multiple days | Major project milestone, complex creation |
| Ongoing | Continuous through the week | Client management, recurring process |

#### Constraints
- Cap the suggestion at **5-7 strategic priority themes** (not individual tasks) — executive time is the scarcest resource
- Each theme should roll up related tasks/actions under one heading
- Include **source attribution** on each suggestion (e.g., "Asana", "Fireflies action item", "HubSpot deal", "User input")

### Step 7c: Present and Confirm

Present the tiered priority suggestion list to the user in a clean, scannable format:

```
SUGGESTED PRIORITIES — [Next Week Date Range]

TIER 1: CARRY-FORWARD COMMITMENTS
- [Theme]: [Key actions] — [source] — [effort tag]

TIER 2: STRATEGIC PRIORITIES
- [Theme]: [Key actions] — [source] — [effort tag]

TIER 3: OPERATIONAL
- [Theme]: [Key actions] — [source] — [effort tag]

TIER 4: DELEGATE OR PARK
- [Item] → [Delegate to whom / Park reason]
```

Then ask: "These are the suggested priorities for next week based on this week's data. Would you like to approve as-is, adjust the order, add/remove items, or reassign anything?"

Wait for the user to confirm or adjust before proceeding to the report draft.

## Step 8: Draft Report

Generate the report using this template. Target **200-500 words**.

The report structure should include:
- Header with report name, date range, and key metrics (worked hours, tasks completed, meetings attended)
- Executive Summary (1 paragraph, 3-5 sentences, outcome-focused)
- This Week's Focus (prior goals + outcomes)
- Deeper Dive sections organized by the user's work categories (e.g., Account Management, Development, General Management, Business Development)
- Upcoming Week Priorities (from Step 7)
- Tag the report recipient at the end

### Formatting Rules
- **Plain text only** — No markdown symbols (**, ##, etc.) in the final Asana comment
- **Asana-native formatting** — Use line breaks, dashes for bullets, colons for emphasis
- **Scannable** — Bullet points within sections (dashes, not asterisks)
- **Executive Summary is prose** — Not bullets
- **Omit empty sections** — Skip any section with no activity

### Tone
- Professional and concise — executive-friendly
- Outcome-focused — highlight what got done and decided
- Constructive on risks — flag concerns without being alarmist
- Forward-looking — connect this week's work to next week's priorities

## Step 9: Post to Asana

After user approves the draft:

1. **Find the recurring task** using `asana_typeahead_search`:
   - Query: the report task name (from user's CLAUDE.md or convention)
   - Resource type: task
   - Workspace: workspace GID from asana-config.md

2. **Get the report recipient's user GID** from asana-config.md

3. **Update the task title** using `asana_update_task`:
   - Add reporting period: `[Report Name] ([Start Date] - [End Date])`

4. **Post the report as a comment** using `asana_create_task_story`:
   - Include the full report text
   - End by mentioning the report recipient

5. Confirm: "Report posted to Asana."

## Step 10: Update CLAUDE.md Priorities

After posting, check if the "Upcoming Week Priorities" from the report differ from the current Priorities section in `~/.claude/CLAUDE.md`.

If they differ:
- Update the Priorities section with the new priorities from the report
- Update the `last_updated` date
- Confirm: "CLAUDE.md priorities updated to reflect next week's focus."
