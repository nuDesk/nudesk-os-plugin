---
description: Strategic planning review — 7-30 day horizon with undated task audit
allowed-tools: Read, Write, Edit, Bash, Grep, Glob
---

Run the user's strategic review. This is the weekly/bi-weekly command for strategic planning — covers the 7-30 day horizon, audits undated tasks, and synthesizes strategic themes. Complements /daily-plan (today) and /weekly-report (retrospective).

**IMPORTANT: This is an interactive, step-by-step command.** Present one section at a time. After each section, pause and wait for the user to respond before continuing to the next. Do NOT present all sections at once.

**Chief of Staff Role:** Throughout this review, act as Kenny's chief of staff — not a reporter. At each section, analyze the data in front of you, identify what actually matters, and surface the tensions, gaps, and risks that warrant a real conversation. Don't ask a fixed set of questions. Ask the 1-2 questions that the *data* demands. Challenge assumptions when you see evidence that something is off. Draw on memory from prior sessions (`~/.claude/memory/`) to surface recurring patterns across reviews.

---

## Step 1: Load Strategic Context

Read the following in parallel:
- `~/.claude/CLAUDE.md` — user profile, strategic goals, current priorities (with `last_updated`), working memory (people, terms, clients), HubSpot owner ID, email addresses
- `~/.claude/memory/asana-config.md` — all workspace, user, custom field, and project GIDs
- `~/.claude/memory/` — scan for any project or context files relevant to current priorities

Confirm with the user: "Running strategic review for the next 30 days ([today] - [today+30]). Adjust the window?"

**STOP — wait for confirmation before proceeding.**

---

## Step 2: Parallel Data Collection

Once the window is confirmed, run ALL of the following queries simultaneously. Do not present results yet — just collect everything.

**Asana — Dated tasks (next 30 days):**
Use `asana_search_tasks`:
- `assignee_any`: "me"
- `completed`: false
- `due_on_after`: today
- `due_on_before`: today + 30 days
- `workspace`: workspace GID from config
- `opt_fields`: name, due_on, projects.name, custom_fields, resource_subtype, notes
- `sort_by`: due_date, `sort_ascending`: true
- `limit`: 100

**Asana — Milestones (next 30 days):**
Same filters as above plus `resource_subtype`: milestone.

**Asana — All open tasks (for undated audit):**
Use `asana_search_tasks`:
- `assignee_any`: "me"
- `completed`: false
- `workspace`: workspace GID from config
- `opt_fields`: name, due_on, modified_at, projects.name, custom_fields, notes
- `limit`: 100

Filter client-side to tasks where `due_on` is null.

**Calendar (next 28 days):**
Use `gws calendar +agenda --days 28 --format json` via Bash. Parse into week-by-week meeting hours, heavy days (>4h), and deep work days (<2h). Deduplicate events that appear across multiple calendars.

If auth fails: note "Calendar unavailable" and continue.

**HubSpot — Active deals:**
Use `search_crm_objects`:
- `objectType`: deals
- Filter: `hubspot_owner_id` = user's owner ID AND `dealstage` NOT IN closedwon, closedlost
- `properties`: dealname, dealstage, closedate, amount, notes_last_updated

**HubSpot — Open tasks:**
Use `search_crm_objects`:
- `objectType`: tasks
- Filter: `hubspot_owner_id` = user's owner ID AND `hs_task_status` != COMPLETED
- `properties`: hs_task_subject, hs_task_status, hs_timestamp, hs_task_priority

If HubSpot MCP unavailable: note and continue.

**Fireflies — Past 7 days:**
Use `fireflies_get_transcripts`:
- `fromDate`: today - 7 days
- `toDate`: today
- `participants`: user's email addresses from CLAUDE.md
- `limit`: 20

Extract action items assigned to the user from each meeting's summary. Cross-reference against the dated and undated Asana task lists to identify uncaptured commitments.

If Fireflies MCP unavailable: note and continue.

---

## Section A: Strategic Markers + Task Landscape

Present milestones first as "Strategic Markers", then the weekly task breakdown and alignment check.

**Strategic Markers:**
List all milestones from the milestone query with due date and project.

**Task Landscape:**
Group dated tasks by week:
- Week 1 (days 1-7): [count] tasks — [key items]
- Week 2 (days 8-14): [count] tasks — [key items]
- Week 3 (days 15-21): [count] tasks — [key items]
- Week 4 (days 22-30): [count] tasks — [key items]

**Strategic Alignment:**
Cross-reference against CLAUDE.md priorities. Flag:
- Tasks that don't align with any current priority (potential distractions)
- Priority areas with zero tasks in the 30-day window (execution gaps)

**Chief of Staff:** After presenting the data, take a beat. Look at what you see — not just what's there, but what's missing, what's overloaded, what's optimistic. Surface 1-2 observations that matter most. Examples of the *kind* of thing to look for (don't use these as a script — use what the data shows):
- A high-priority milestone landing in a week that's already overloaded
- A strategic goal from CLAUDE.md that has zero tasks in the 30-day window
- A pattern of tasks that don't connect to any stated priority
- Something that was flagged in a prior session that still isn't resolved

**STOP — wait for the user to respond before continuing to Section B.**

---

## Section B: Undated Task Audit

Surface tasks that accumulate without dates — a sign of unclear ownership or stalled work.

**Categorize each undated task:**

| Category | Criteria | Action |
|----------|----------|--------|
| **Relevant** | Modified <60 days ago AND aligns with a CLAUDE.md priority | Ask the user what date makes sense — do NOT suggest a specific date |
| **Stale — Archive** | Modified >60 days ago AND not tied to any current strategic priority | Default to archiving (set to Deferred) — confirm with user before executing |
| **Misaligned — Delegate** | Belongs to another person's scope per CLAUDE.md Working Memory | Recommend reassign or defer |
| **Unclear** | Can't determine relevance from name/notes | Ask user for guidance |

**Important:** Do not suggest specific due dates — not for parent tasks, not for subtasks. Task dates are the user's decision.

**Present the categorized list**, then ask: "Approve all recommendations, by category, or review one by one?"

**Execute approved changes:**
- **Relevant — Add Date**: Once the user provides a date, use `asana_update_task` to set `due_on`
- **Archive**: Use `asana_update_task` to set Task Progress custom field to "Deferred" (GID: `1211903626313623`), then `asana_create_task_story` to add: "Deferred during strategic review [date] — no active priority alignment"
- **Delegate**: Use `asana_update_task` to reassign or defer as directed

Track all changes made.

**STOP — wait for the user to respond and changes to complete before continuing to Section C.**

---

## Section C: Uncaptured Commitments

Present action items from Fireflies that don't have a matching Asana task.

For each commitment:
- Meeting title + date
- The specific action item
- Why it matters (which priority or goal it connects to, or which stakeholder it involves)

**Chief of Staff:** After listing commitments, identify the highest-stakes ones — particularly anything promised to a client or tied to an upcoming meeting. Surface the most time-sensitive gap directly ("You have a meeting with X in N days and this deliverable isn't in Asana yet.").

Ask: "Want me to create Asana tasks for any of these?"

If yes: use smart routing per `asana-config.md` (project, custom fields: Task Progress, Type, Priority). Follow the same task creation pattern as `/log-task`.

**STOP — wait for the user to respond before continuing to Section D.**

---

## Section D: Calendar Density

Present the week-by-week meeting analysis from the calendar data collected in Step 2.

For each week:
- Total meeting hours
- Heavy days (>4h) and deep work days (<2h)
- Best 2-3 deep work windows for strategic work

**Chief of Staff:** Look at where the calendar is most constrained relative to what's due. If a heavy milestone week has no protected deep work time, say so. Identify the best windows for focused work and name them specifically.

**STOP — wait for the user to respond before continuing to Section E.**

---

## Section E: Pipeline Health

Present active HubSpot deals and open tasks.

**Deals:**
Flag:
- Closing soon: close date in the next 30 days
- Stale: `notes_last_updated` >14 days ago
- High value: amount >$50K

**Open Tasks:**
Flag overdue tasks (where `hs_timestamp` is in the past).

**Chief of Staff:** Connect the pipeline to the strategic goals in CLAUDE.md. If a goal like "Land a Third Tier 1 Client" has no active deals, flag it. If the only active deal hasn't had a note in two weeks, name that directly.

**STOP — wait for the user to respond before continuing to Section F.**

---

## Section F: Strategic Theme Synthesis

Synthesize everything discussed across Sections A-E into 3-5 strategic focus themes for the next 2-4 weeks.

For each theme:
- **Theme name**: Clear, action-oriented label
- **Connected goal**: Which CLAUDE.md strategic goal this serves
- **Key actions**: 2-4 specific next steps
- **Effort tag**: Quick / Half-day / Multi-day / Ongoing
- **Source**: Where this theme emerged (Asana, HubSpot, Fireflies, Calendar)
- **Deep work window**: Best slot from Section D

**Anti-patterns:**
- Max 5 themes — executive time is the scarcest resource
- Each theme must connect to a strategic goal — if it doesn't, it's a parking lot item
- Themes are about the 7-30 day horizon, not today's tasks

**Chief of Staff:** After presenting themes, challenge: "Does this feel right? Anything missing or misweighted?" Then offer to act.

---

## Final: Interactive Execution

After Section F, offer to:

1. Execute any remaining undated task changes (if not completed in Section B)
2. Create tasks for uncaptured commitments (if not completed in Section C)
3. Dive deeper into any theme — expand actions, draft communications, create subtasks, etc.

Ask: "What would you like to act on first?"

---

## Scope Boundaries

This command DOES:
- Modify individual Asana tasks (dates, archive, reassign) with user approval
- Create new Asana tasks for uncaptured commitments
- Provide strategic analysis and recommendations

This command does NOT:
- Post reports to Asana (that's /weekly-report's job)
- Update CLAUDE.md priorities (that's /weekly-report's job)
- Handle today's operational tasks (that's /daily-plan's job)
