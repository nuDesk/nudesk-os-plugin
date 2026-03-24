---
name: vanta-bridge
description: >
  Asana-to-Vanta sync skill. Pushes Production Change Log tasks, incident records,
  and evidence to Vanta via REST API. Only active when Vanta API access is confirmed.
  Designed for periodic execution (weekly) or manual trigger.
user_invocable: false
version: 1.0.0
---

# Vanta Bridge

> Not user-invocable. Called by scheduled tasks (weekly sync) and `/evidence-collect`.
> Only active when Vanta API access is confirmed in compliance-config.md.

## Prerequisites

- `~/.claude/memory/compliance-config.md` → Vanta → API Access = "yes"
- Vanta OAuth client credentials configured (client ID in config, secret in `~/.env`)
- Valid token (auto-refreshes if expired)

**If Vanta API is not available:** This skill generates a formatted export for manual upload instead of making API calls.

## Capabilities

### 1. Task Sync: Asana → Vanta

Push Production Change Log tasks as Vanta `security_task` resources.

**Source:** Asana Production Change Log project (tasks completed since last sync)
**Target:** Vanta `POST /tasks` endpoint

Per task:
```json
{
  "taskId": "{asana-task-gid}",
  "title": "{task title}",
  "state": "COMPLETED",
  "priority": "{mapped from Change Type}",
  "assignees": ["{user email}"],
  "externalUrl": "https://app.asana.com/0/{project-gid}/{task-gid}",
  "description": "{task description — truncated to 2000 chars}",
  "completedAt": "{completed_at timestamp}"
}
```

Priority mapping:
| Asana Change Type | Vanta Priority |
|-------------------|----------------|
| Hotfix | HIGH |
| Bugfix | MEDIUM |
| Feature | MEDIUM |
| Config | LOW |
| Infrastructure | MEDIUM |

### 2. Incident Sync: Asana → Vanta

Push Incident Response Log tasks to Vanta.

**Source:** Asana Incident Response Log project
**Target:** Vanta `POST /tasks` endpoint

Per incident:
```json
{
  "taskId": "{asana-task-gid}",
  "title": "{incident title}",
  "state": "{OPEN or COMPLETED}",
  "priority": "{mapped from Severity}",
  "assignees": ["{assignee email}"],
  "externalUrl": "https://app.asana.com/0/{project-gid}/{task-gid}",
  "description": "{incident description}"
}
```

Severity mapping:
| Asana Severity | Vanta Priority |
|---------------|----------------|
| P0 | CRITICAL |
| P1 | HIGH |
| P2 | MEDIUM |
| P3 | LOW |

### 3. Evidence Upload

Upload evidence documents collected by the evidence-collector skill.

**Source:** `~/.claude/memory/context/evidence-exports/` (files from evidence-collector)
**Target:** Vanta `POST /evidence` endpoint

Workflow:
1. List pending evidence files in exports directory
2. For each file:
   a. Upload to Vanta via `POST /evidence` (multipart/form-data)
   b. Link to relevant controls via `POST /controls/{id}/evidence`
   c. Move to `evidence-exports/uploaded/` subdirectory
3. Respect rate limits: sequential uploads, pause if approaching 50 req/min

## Execution Flow

### When API is Available

1. Authenticate: request OAuth token via client credentials
2. Sync tasks: query Asana for new/updated Change Log tasks since last sync
3. Sync incidents: query Asana for new/updated incidents since last sync
4. Upload evidence: process pending evidence files
5. Record sync timestamp in `~/.claude/memory/context/vanta-last-sync.md`
6. Return summary

### When API is Unavailable (Export Mode)

1. Query Asana for the same data as above
2. Generate formatted export files:
   - `{date}_change-log-export.md` — Change Log summary for manual entry
   - `{date}_incident-export.md` — Incident summary for manual entry
   - Evidence files already in exports directory
3. Save all exports to `~/.claude/memory/context/vanta-exports/`
4. Return: "Export files generated for manual Vanta upload: [file list]"

## Rate Limit Handling

- Track request count per minute
- If approaching 50 req/min: pause for remaining time in the minute
- Auth endpoint limit is 5 req/min — cache tokens for full 1-hour lifetime
- If rate limited (429 response): wait for `X-RateLimit-Reset` seconds and retry once

## Sync State

Track last sync in `~/.claude/memory/context/vanta-last-sync.md`:

```markdown
# Vanta Sync State

| Sync Type | Last Run | Items Synced | Status |
|-----------|----------|-------------|--------|
| Task Sync | [timestamp] | [N] | Success / Failed |
| Incident Sync | [timestamp] | [N] | Success / Failed |
| Evidence Upload | [timestamp] | [N] | Success / Failed |
```

## Return Format

```
Vanta Bridge Sync Complete:
- Tasks synced: [N] (new: [N], updated: [N])
- Incidents synced: [N]
- Evidence uploaded: [N]
- Errors: [N] (details if any)
- Next scheduled sync: [date]
```
