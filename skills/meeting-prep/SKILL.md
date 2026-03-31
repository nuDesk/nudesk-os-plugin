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
- **Brief output** — deliver as markdown first, then create a Google Doc after user sign-off

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

## Step 5: Generate Meeting Prep Brief (Markdown Draft)

Generate the brief as markdown and present it to the user. This is the review draft before creating the Google Doc.

Adapt format based on relationship type:

### For Prospects

```
## Meeting Prep Brief: [Company Name]

**Date:** [Day of week], [Month] [Day], [Year] | [Start time] - [End time] [TZ]
**Type:** [Meeting type — e.g., First consultation call (new prospect)]
**Meeting Objective:** [From discovery or inferred from context]

## Research & Engagement History

### Engagement History
- [Summary of prior interactions from email, HubSpot, Fireflies — or note if new relationship]

### Company Overview
[Company name] -- [One-line description, location, identifiers]
[Niche/focus areas]
[Scale/metrics]
[Differentiator]

### Participant Bios
[Name], [Title] -- [Key background, achievements, relevant context]

## Meeting Preparation

### Recommended Agenda
[Topic] ([time]) -- [Description]

### Key Talking Points
[Topic] -- [Explanation of why this matters and how to position it]

### What We've Built (Capabilities to Highlight)
[Capability name] -- [Description of what it does and why it's relevant]

### Discovery Questions
[Question]

### Expected Questions and nuDesk Responses
Q: [Expected question]
A: [Suggested response]
```

### For Clients

Same structure but replace "Research & Engagement History" focus with current engagement status, recent deliverables, and open items. Replace "Discovery Questions" with "Questions for Client." Skip "What We've Built" unless demoing new capabilities.

### For Network/Other

Simplify to: Date, Type, Meeting Objective, Engagement History, Participant Bios, Talking Points, Questions to Ask.

After presenting the markdown brief, ask the user: **"Ready to create the Google Doc?"**

Do not proceed to Step 6 until the user signs off. They may request changes to the brief content first.

## Step 6: Create Google Doc

After user sign-off, create a formatted Google Doc in the Meeting Notes shared drive folder.

### 6a. Create the document

```bash
gws docs documents create --json '{"title":"Meeting Prep Brief: [Company Name]"}' --format json
```

Extract the `documentId` from the response.

### 6b. Move to Meeting Notes folder

The doc must be moved to the **Meeting Notes (nuDesk)** shared drive folder:
- **Folder ID:** `1dIPBQ--7uI9YEKn8DAq7ityBdcOa21cM`
- **Drive ID:** `0APOefP_6ePhHUk9PVA`

```bash
gws drive files update --params '{"fileId":"[DOC_ID]","addParents":"1dIPBQ--7uI9YEKn8DAq7ityBdcOa21cM","removeParents":"[CURRENT_PARENT_ID]","supportsAllDrives":true}' --format json
```

To get the current parent, read the file metadata first:
```bash
gws drive files get --params '{"fileId":"[DOC_ID]","fields":"id,parents","supportsAllDrives":true}' --format json
```

### 6c. Format the document with batchUpdate

Use `gws docs documents batchUpdate` to insert and format all content. The document must match this style structure (based on the established template):

**Document formatting rules:**

| Content | Google Docs Style |
|---------|-------------------|
| Title (e.g., "Meeting Prep Brief: Convoy Home Loans") | `TITLE` (set by `namedStyleType`) |
| Metadata lines (Date, Type, Meeting Objective) | `NORMAL_TEXT` with bold label + normal value |
| Major sections (Research & Engagement History, Meeting Preparation) | `HEADING_2` |
| Subsections (Engagement History, Company Overview, Participant Bios, etc.) | `HEADING_3` |
| Bullet points | `NORMAL_TEXT` starting with `- ` |
| Agenda items | `NORMAL_TEXT` — "[Topic] ([time]) -- [Description]" |
| Q&A pairs | `NORMAL_TEXT` — separate `Q:` and `A:` lines |

**Important formatting notes:**
- Do NOT use markdown tables in the Google Doc. Convert "Expected Questions" to Q:/A: line pairs.
- Bold labels in metadata lines (e.g., **Date:**, **Type:**, **Meeting Objective:**) use `updateTextStyle` with `bold: true`.
- Insert blank lines between sections for readability.
- The title is set via `insertText` at index 1 + `updateParagraphStyle` with `namedStyleType: TITLE`.

**batchUpdate request structure:**

Build the requests array by inserting text bottom-up (insert from end to start so character indices don't shift). Alternatively, insert sequentially and track the running index.

```bash
gws docs documents batchUpdate --params '{"documentId":"[DOC_ID]"}' --json '{"requests":[...]}' --format json
```

Each request in the array is one of:
- `insertText` — `{"insertText":{"location":{"index":N},"text":"content\n"}}`
- `updateParagraphStyle` — `{"updateParagraphStyle":{"range":{"startIndex":N,"endIndex":M},"paragraphStyle":{"namedStyleType":"HEADING_2"},"fields":"namedStyleType"}}`
- `updateTextStyle` — `{"updateTextStyle":{"range":{"startIndex":N,"endIndex":M},"textStyle":{"bold":true},"fields":"bold"}}`

### 6d. Share the link

After creating the doc, share the Google Doc link with the user:

```
Doc created: https://docs.google.com/document/d/[DOC_ID]/edit
```

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
| `gws docs documents create` | Step 6a | Create blank Google Doc |
| `gws drive files update` | Step 6b | Move doc to shared drive folder |
| `gws docs documents batchUpdate` | Step 6c | Insert and format doc content |

## Gmail Base Filter

All Gmail searches in this skill MUST prepend this filter:

```
(to:kenny@nudesk.ai OR cc:kenny@nudesk.ai OR to:kenny@cintargroup.com OR cc:kenny@cintargroup.com) is:important
```

Then append attendee-specific terms (name, domain, company).
