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

## Step 6: Chief of Staff Analysis

Act as Kenny's chief of staff — not a reporter. Analyze all gathered data (Asana, Fireflies, prior week goals from Step 3) and surface the 1-2 questions the *data* actually demands. Do not ask a fixed set of questions. Do not ask for information already visible in the data.

**Always ask (mandatory):** "For Champions Funding, what are your MTD Fundings and Submissions vs. budget?" — even if no other questions are needed.

**Then look for and ask about the 1-2 most important gaps:**
- Gaps between last week's stated goals (Step 3) and what actually got done
- Uncaptured commitments from Fireflies not reflected in Asana
- Decisions or pivots made that aren't obvious from task names
- Things conspicuously absent, stuck, or at risk
- Context that would materially change how the report reads

Ask one question at a time. Wait for each response before asking the next.

## Step 7: Synthesize Upcoming Week Priorities

Instead of asking a blank question, synthesize a data-driven priority suggestion from everything gathered so far (Asana tasks, Fireflies action items, user input from Step 6) plus forward-looking signals. This follows the same pattern as `/daily-plan` Step 3 but scoped to the upcoming week.

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

### Step 7d: Task Rescheduling

After priorities are confirmed in Step 7c, help spread next week's tasks across the week based on priorities, effort, and calendar load. This prevents pile-ups on Monday or Friday.

**Using data already gathered (Step 4 upcoming tasks + Step 7a calendar):**

1. Map each open task due in the next 7 days against the calendar:
   - Identify overloaded days: many tasks due AND heavy meeting load (>3h meetings)
   - Identify under-utilized days: light calendar AND few tasks due
   - Flag tasks specifically clustered on Monday or Friday

2. Cross-reference against confirmed priorities from Step 7c:
   - Higher-priority items should land on days with available deep-work capacity
   - Effort-intensive tasks (multi-day) should avoid meeting-heavy days

3. Present a table — only include tasks that would benefit from moving (cap at 5-7 suggestions):

```
TASK SCHEDULE REVIEW — [Next Week Date Range]

Task                     | Current Due | Suggested Due | Reason
-------------------------|-------------|---------------|---------------------------
[Task name]              | Monday      | Wednesday     | Monday heavy (3 meetings)
[Task name]              | Friday      | Tuesday       | High priority, clear slot
```

4. Ask: "Approve all, review one by one, or skip?"

5. Execute approved changes via `asana_update_task` (set `due_on`). Confirm each change.

**STOP — wait for user response before proceeding to Step 8.**

---

## Step 8: Draft Report

Generate the report using this template. Target **500-800 words**.

### Report Structure

```
🗓 Week of [Start Date – End Date] – KKS Weekly Update

Summary: [One paragraph, 3-5 sentences. Cover the week at a high level across all active initiatives. Be candid — name wins and misses. Do not list tasks; synthesize themes and outcomes.]

[Emoji] [Client or Initiative Name]
Status: [Short health descriptor — e.g., "Active — strong progress", "Behind — competing priorities", "On track"]
Progress:
- [Narrative bullet: explain what happened AND why, with context and rationale]
- [Include data points or metrics inline where available]
- [Flag decisions made, pivots taken, or risks identified]
Next Steps:
- [Specific forward action for next week]
- [Continue as needed]

[Repeat for each active initiative]

[Optional — include only if relevant:]
⚠️ Flag for review: [One item needing attention or a decision]

🎯 Key Milestones Going Into Next Week
[Emoji] [Initiative]: [One-line directional milestone]
[Repeat for each active initiative]
```

### Organization Rules

**Always use this fixed section order:**
1. 🏢 Champions Funding
2. 🤖 AI Development & Enablement
3. ⚙️ nuDesk Management

**Champions Funding always includes a metrics block immediately after the Status line:**
```
🏢 Champions Funding
Status: [descriptor]
MTD Fundings: $[actual] / $[budget] ([X]% to budget)
MTD Submissions: [actual] / [budget] ([X]% to budget)
Progress:
- [narrative bullet]
Next Steps:
- [forward action]
```
- MTD values come from user input in Step 6
- If user didn't provide them, write: "MTD metrics not provided this week"

**AI Development & Enablement** — agent builds, training programs, internal tools, AI rollouts across clients

**nuDesk Management** — Prime Nexus, operations, finance, compliance (SOC 2), team/admin, business development

- Omit a bucket entirely if there was no activity — don't force empty sections
- Each bucket follows: Status line + Progress bullets + Next Steps
- The closing milestone list mirrors all three buckets — one line each, forward-looking

### Progress Bullet Style
- Narrative, not a task list — explain context and rationale, not just what was done
- Include data where available (pipeline numbers, percentages, deal sizes)
- Be candid about misses and delays — include the reason and any decision made
- Each bullet should stand alone as meaningful to an executive reader

### Formatting Rules
- **Plain text only** — No markdown symbols (**, ##, etc.) in the final Asana comment
- **Asana-native formatting** — Use line breaks, dashes for bullets, colons for emphasis
- **Summary is prose** — Not bullets
- **Omit empty sections** — Skip any initiative with no activity this week

### Tone
- Strategic and narrative — not a task log
- Candid about misses — name them with rationale, not just omit them
- Outcome-focused — what moved, what was decided, what changed
- Forward-looking — each initiative ends with clear next steps

## Step 9: Post to Asana

After user approves the draft:

1. **Create a new Asana task** using `asana_create_task`:
   - `name`: `KKS Weekly Milestones & Report — Week of [Start Date]`
   - `project_id`: `1211894375322111` (Executive Management)
   - `assignee`: `me`
   - `due_on`: next Monday from today (ISO 8601, YYYY-MM-DD)
   - `custom_fields` (JSON string):
     - Task Progress = In Progress: `{"1211903626313619": "1211903626313621"}`
     - Type = Reporting: `{"1211907135805437": "1211907135805442"}`
     - Priority = High: `{"1211907466745700": "1211907466745701"}`

2. **Post the report as a comment** using `asana_create_task_story` on the newly created task:
   - Full report text
   - End with a mention of Sean Salas (GID: `1211913018942721`)

3. Confirm: "Report posted to Asana. Task created — In Progress, due [next Monday date]."

## Step 10: Update CLAUDE.md Priorities

After posting, check if the "Upcoming Week Priorities" from the report differ from the current Priorities section in `~/.claude/CLAUDE.md`.

If they differ:
- Update the Priorities section with the new priorities from the report
- Update the `last_updated` date
- Confirm: "CLAUDE.md priorities updated to reflect next week's focus."
