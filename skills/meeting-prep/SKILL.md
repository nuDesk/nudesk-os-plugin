---
name: meeting-prep
description: Prepare nuDesk co-founders for business development, client, and network meetings. Use when asked to prepare for a meeting, create a meeting brief, help with meeting preparation, or when phrases like "I have a meeting with...", "meeting prep", "prepare me for...", or "upcoming meeting" appear. Autonomous by default — auto-selects meetings, runs parallel research, and generates briefs with minimal interruption.
---

# Meeting Prep Skill

Prepare nuDesk co-founders for business meetings through calendar integration, parallel engagement analysis, and adaptive discovery.

## Autonomy Model

This skill minimizes round trips. Most preps complete with 0-1 questions.

- **1 meeting in next 24h** — auto-select it, announce which one
- **Multiple meetings** — present list, ask user to pick (only required stop)
- **User names a person or company** (e.g., "prep for my call with Evan") — match by attendee name or title, skip the list
- **Discovery questions** — only asked when engagement history is thin (see Step 3)
- **Brief output** — deliver directly, no approval gate

## Step 1: Calendar Check

Identify the meeting to prepare for.

**Tools:**
```bash
gws calendar +agenda --days 7
```

**Logic:**
1. If user specified a person/company, scan the agenda for a matching meeting (attendee name, title, or description). Auto-select on match.
2. If only 1 meeting in the next 24 hours, auto-select and announce: "Preparing for [title] with [attendees] at [time]."
3. If multiple meetings and no user hint, present the list and ask which one.
4. Once selected, get full meeting details:
```bash
gws workflow +meeting-prep
```

Extract: attendee names, email domains, meeting title, description, linked docs.

## Step 2: Parallel Engagement Research

After meeting selection, launch **all three tracks simultaneously** as parallel operations. Do not run these sequentially.

### Track A: Email History
```bash
gws gmail users messages list --params 'q=(to:kenny@nudesk.ai OR cc:kenny@nudesk.ai OR to:kenny@cintargroup.com OR cc:kenny@cintargroup.com) is:important {attendee name OR domain}'
```

Always include the base Gmail filter above. Append attendee names and/or company domain as additional search terms.

Look for: recent threads, open questions, commitments made, tone of relationship.

Read the top 3-5 most recent relevant messages for context.

### Track B: HubSpot CRM
Use the HubSpot MCP tools:
- `mcp__claude_ai_HubSpot__search_crm_objects` — search contacts by attendee name/email, search deals by company name
- `mcp__claude_ai_HubSpot__get_crm_objects` — pull full contact or deal records for matches

Look for: deal stage, lifecycle stage, last interaction date, recorded notes, associated deals, owner.

### Track C: Fireflies Meeting Transcripts
Use the Fireflies MCP tools:
- `mcp__claude_ai_Fireflies__fireflies_search` — search by attendee name or company
- `mcp__claude_ai_Fireflies__fireflies_get_summary` — get AI summary for matching meetings

Look for: past meeting topics, action items, decisions, follow-ups, relationship dynamics.

### Synthesize Findings

After all three tracks complete, classify the relationship:
- **New contact** — no hits across email, HubSpot, or Fireflies
- **Warm lead / prospect** — some engagement but no active deal or client relationship
- **Active client** — ongoing engagement, active deals or projects
- **Internal / partner** — nuDesk team or close partner

## Step 3: Conditional Discovery

Based on engagement depth, decide whether to ask questions:

**Rich history** (hits in 2+ tracks) — Skip questions. Generate brief from data.

**Partial history** (hits in 1 track, gaps in others) — Ask **1 targeted question** to fill the gap:
- Have HubSpot deal but no recent emails: "What's the latest on [deal name]?"
- Have emails but no clear meeting objective: "What's your goal for this meeting?"
- Have Fireflies but nothing recent: "Has anything changed since your last conversation on [date]?"

**No history** (new relationship) — Ask **up to 2 questions:**
1. How did this meeting come about?
2. What outcome are you aiming for?

**User provided context upfront** — Extract what you can from their message. Only ask if something critical is missing (e.g., meeting objective is completely unclear).

**Max questions in any scenario: 2.**

## Step 4: Additional Research (Parallel, Conditional)

Run these in parallel where triggered. Skip what's redundant with Step 2 findings.

### Contact Directory Check (always run)
```bash
gws people people searchDirectoryPeople --params 'query={attendee name}' --params 'readMask=names,emailAddresses,organizations,phoneNumbers' --params 'sources=DIRECTORY_SOURCE_TYPE_DOMAIN_PROFILE'
```
Determines if attendees are internal (nuDesk/Cintar) vs external.

### Company Research (new relationships and prospects only)
Use `WebSearch` for: company background, recent news, funding, key metrics, industry position.

### Participant Bios (new contacts only)
Use `WebSearch` for: attendee backgrounds, roles, career history. Skip for people already in Kenny's working memory (see CLAUDE.md People table).

## Step 5: Generate Meeting Prep Brief

Adapt format based on relationship type. Deliver directly — no approval gate.

### For Prospects

```
## Meeting Prep Brief: [Company Name]

**Date:** [From calendar]
**Meeting Objective:** [From discovery or inferred from context]

### Engagement History
[Summary of prior interactions from email, HubSpot, Fireflies — or note if new relationship]

### Company Overview
[Brief company background, recent news, relevant metrics]

### Participant Bios
- **[Name], [Title]** — [Key background points]

### Recommended Agenda
1. Opening / rapport (5 min)
2. [Agenda item based on context]
3. [Agenda item based on context]
4. Next steps (5 min)

### Key Talking Points
- [Point aligned to nuDesk relevance and prior context]

### Discovery Questions
- [Question based on meeting goal and engagement history]

### Expected Questions & nuDesk Responses

| Expected Question | Suggested Response |
|-------------------|-------------------|
| Can nuDesk work with our existing LOS or core? | Yes, we specialize in integrating into legacy and modern ecosystems. |
| How is your platform different from internal automation or RPA tools? | nuDesk focuses on intelligence and orchestration across business rules, not just task-level automation. |
| What's the typical time to value? | Many clients see measurable outcomes within 30-60 days of deployment. |
| Is it configurable to our specific credit products or channels? | Yes, we support tailored rules, routes, and validations. |
| How do you handle data security and compliance? | SOC 2-compliant, with full audit logging and encryption in motion and at rest. |
```

### For Clients

```
## Meeting Prep Brief: [Client Name]

**Date:** [From calendar]
**Meeting Objective:** [From discovery or inferred from context]

### Engagement History
[Current engagement status, recent deliverables, relationship context from email/HubSpot/Fireflies]

### Participant Bios
- **[Name], [Title]** — [Key background points]

### Recommended Agenda
1. Check-in / rapport (5 min)
2. [Agenda item based on context]
3. Next steps (5 min)

### Key Talking Points
- [Point aligned to client priorities and recent engagement]

### Questions for Client
- [Question based on meeting goal]

### Expected Questions & nuDesk Responses

| Expected Question | Suggested Response |
|-------------------|-------------------|
| [Context-specific question from history] | [Tailored response] |
```

### For Network/Other

Simplify to: Meeting Objective, Engagement History, Participant Bios, Talking Points, Questions to Ask.

## Tool Reference

| Tool | Used In | Purpose |
|------|---------|---------|
| `gws calendar +agenda` | Step 1 | List upcoming meetings |
| `gws workflow +meeting-prep` | Step 1 | Full meeting details with attendees |
| `gws gmail users messages list` | Step 2A | Email search (always include base filter) |
| HubSpot MCP `search_crm_objects` | Step 2B | CRM contact and deal search |
| HubSpot MCP `get_crm_objects` | Step 2B | Full CRM record details |
| Fireflies MCP `fireflies_search` | Step 2C | Past meeting transcript search |
| Fireflies MCP `fireflies_get_summary` | Step 2C | AI meeting summaries |
| `gws people people searchDirectoryPeople` | Step 4 | Internal vs external contact check |
| `WebSearch` | Step 4 | Company and participant research |

## Gmail Base Filter

All Gmail searches in this skill MUST prepend this filter:

```
(to:kenny@nudesk.ai OR cc:kenny@nudesk.ai OR to:kenny@cintargroup.com OR cc:kenny@cintargroup.com) is:important
```

Then append attendee-specific terms (name, domain, company).
