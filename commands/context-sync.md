---
description: Gather a 7-day cross-system briefing — git, Asana, Calendar, Gmail, Chat
argument-hint: [optional focus area]
allowed-tools: Read, Grep, Glob, Bash
---

Gather context from the last 7 days across my systems to produce a "state of the world" briefing for the current project.

Focus area (optional): $ARGUMENTS

## Steps

### 1. Git Activity (last 7 days)
- Run `git log --oneline --since="7 days ago" --all` to see recent commits across all branches
- Run `git branch -a` to see active branches
- Check for open PRs with `gh pr list` (if GitHub CLI available)
- Note any uncommitted changes with `git status`

### 2. Asana Context
Use Asana MCP tools to:
- Search for my tasks updated in the last 7 days (`modified_since`)
- Filter to tasks relevant to this project (match repo name or focus area to Asana project)
- List any tasks due in the next 3 days
- Note tasks in "Agent Queue" status that haven't been processed

### 3. Project File Context
- Read the project's `CLAUDE.md` or `README.md` for current state documentation
- Check for any `DECISIONS.md`, `ARCHITECTURE.md`, or `OPERATIONS.md` files
- Look for TODO comments in recently modified files

### 4. Google Workspace Activity (last 7 days)
Use the **`gws` CLI** via Bash to:
- **Calendar:** `gws calendar events list --params '{"calendarId": "primary", "timeMin": "<7d-ago>T00:00:00Z", "timeMax": "<today>T23:59:59Z", "maxResults": 50, "singleEvents": true, "orderBy": "startTime"}'` — flag meetings relevant to this project or focus area
- **Gmail:** `gws gmail users messages list --params '{"userId": "me", "q": "<project-keywords> newer_than:7d", "maxResults": 20}'` — summarize key decisions, requests, or blockers
- **Chat:** `gws chat spaces messages list --params '{"parent": "spaces/<spaceId>", "filter": "createTime > \"<7d-ago>T00:00:00Z\""}'` — check relevant spaces for project-related messages

Summarize only what's relevant to this project — skip unrelated meetings and emails.

### 5. Synthesize Briefing

Output a concise briefing in this format:

```
## State of the World: [Project Name]
Generated: [date]
Focus: [focus area if provided, otherwise "General"]

### Recent Activity (last 7 days)
- [X] commits across [Y] branches
- Key changes: [bullet summary of significant commits]
- Open PRs: [list or "none"]
- Uncommitted work: [status or "clean"]

### Asana Status
- Active tasks: [count and summary]
- Due soon: [tasks due in next 3 days]
- Agent queue: [pending items]

### Comms & Meetings
- Key meetings: [relevant meetings from last 7 days]
- Email threads: [important Gmail threads related to this project]
- Chat highlights: [notable Google Chat messages, if any]

### Current State
[2-3 sentences on where the project stands based on all signals above]

### Suggested Next Steps
[3-5 actionable items based on what you found, ordered by priority]
```

If a focus area was provided via $ARGUMENTS, weight the briefing toward that topic — filter commits, tasks, and suggestions to what's relevant.

Usage: /context-sync or /context-sync Prime Nexus pipeline
