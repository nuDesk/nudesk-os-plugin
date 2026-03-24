---
description: Log and track a security incident with structured severity classification and 6-phase response
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, mcp__plugin_asana_asana__asana_create_task, mcp__plugin_asana_asana__asana_update_task, mcp__plugin_asana_asana__asana_create_task_story, mcp__plugin_asana_asana__asana_search_tasks, mcp__plugin_asana_asana__asana_get_task
---

Structured incident logging addressing controls IR-01 through IR-05. Creates a properly classified incident task in the Asana Incident Response Log with 6-phase subtask checklist.

## EXECUTION RULES

This command is interactive. Each step gathers information from the user and builds the incident record incrementally. Do not skip ahead.

---

## Step 1: Load Config

Read `~/.claude/memory/compliance-config.md` to load:
- Incident Response Log project GID
- Severity custom field GID and enum option GIDs
- Control ID custom field GID
- Incident response subtask template

Read `~/.claude/memory/asana-config.md` for workspace and user GIDs.

If the Incident Response Log is not configured: stop and direct user to `/os-setup`.

## Step 2: Incident Classification

Ask the user to describe the incident:

Call `AskUserQuestion`: "Describe the security incident:\n\n1. **What happened?** (brief description)\n2. **When was it discovered?** (date/time)\n3. **What systems/data are affected?**\n4. **Is this ongoing or contained?**\n5. **Does this involve PHI or PII?** (triggers HIPAA notification requirements under IR-04)"

**→ Call `AskUserQuestion`. Your turn ends here.**

## Step 3: Classify Severity

Based on the user's description, recommend a severity classification per nuDesk's incident response policy:

| Severity | Criteria | Escalation | SLA |
|----------|----------|------------|-----|
| **P0 — Critical** | Active data breach, production system compromise, credential exposure in public system | Immediate notification to IT/Engineering leadership + CEO | Triage within 1 hour |
| **P1 — High** | Significant vulnerability discovered, unauthorized access attempt detected, credential exposure with limited access | Ticket + manager notification | Triage within 4 hours |
| **P2 — Medium** | Policy violation, minor vulnerability, credential exposure on secure local machine | Standard ticket | Triage within 24 hours |
| **P3 — Low** | Non-secret identifier exposure, documentation gap, minor config issue | Document only | Triage within 1 week |

Present recommendation:

```
SEVERITY CLASSIFICATION

Recommended: [P0/P1/P2/P3] — [Criteria match]
Rationale: [Why this severity level]

Escalation path: [Per severity level]
HIPAA notification required: [Yes/No] (IR-04)
```

Call `AskUserQuestion`: "Agree with this severity classification? Adjust if needed."

**→ Call `AskUserQuestion`. Your turn ends here.**

## Step 4: Create Incident Task

Create the incident task in Asana:

1. **Create task** via `asana_create_task`:
   - **Project:** Incident Response Log GID
   - **Title:** `[P{severity}] {brief description}` (e.g., "[P2] API key exposed in session log")
   - **Severity** custom field: set to confirmed severity
   - **Control ID** custom field: IR-01 (initial report)
   - **Description (html_notes):**
     ```
     INCIDENT REPORT

     Reported: {date/time}
     Reported by: {user name}
     Severity: P{N} — {severity name}
     Status: Phase 1 — Reported

     DESCRIPTION
     {What happened — from user's input}

     AFFECTED SYSTEMS/DATA
     {Systems and data affected}

     HIPAA/PHI INVOLVED: {Yes/No}
     {If yes: "HIPAA notification timeline: 60 days from discovery per IR-04"}

     ESCALATION PATH
     {Per severity level}

     TIMELINE
     - {discovery date}: Incident discovered
     - {report date}: Incident reported via /incident-log
     ```

2. **Add 6-phase subtasks** from the compliance config template:
   - Phase 1: Report — incident reported, initial details captured
   - Phase 2: Triage — severity classified, escalation path determined
   - Phase 3: Investigate — root cause analysis underway
   - Phase 4: Contain — immediate containment actions taken
   - Phase 5: Recover — systems restored to normal operation
   - Phase 6: Harden — preventive measures implemented, lessons documented

3. **Mark Phase 1 subtask as complete** (we just reported it).

## Step 5: Initial Triage Walkthrough

Walk through the initial triage checklist:

```
TRIAGE CHECKLIST

1. [ ] Is the incident still active? → If yes, immediate containment is priority
2. [ ] Who else needs to be notified? → Per escalation path for {severity}
3. [ ] What evidence should be preserved? → Logs, screenshots, git history
4. [ ] Are other systems at risk? → Assess blast radius
5. [ ] What's the immediate containment action? → Rotate credentials, disable access, etc.
```

Call `AskUserQuestion`: "Let's work through the triage checklist. For each item, provide your assessment or action taken."

**→ Call `AskUserQuestion`. Your turn ends here.**

## Step 6: Update Incident Record

Based on triage answers:

1. Update the task description with triage findings via `asana_update_task`
2. Add a comment via `asana_create_task_story` with the triage summary
3. Mark Phase 2 (Triage) subtask as complete
4. If containment actions were taken, note them in the timeline

## Step 7: HIPAA Check (if PHI involved)

If the incident involves PHI (per Step 2):

```
⚠️  HIPAA NOTIFICATION REQUIRED (IR-04)

Per nuDesk policy, HIPAA breach notification must occur within 60 days of discovery.

Discovery date: {date}
Notification deadline: {date + 60 days}

Required actions:
1. Legal review of breach scope
2. Executive approval of notification content (IR-05)
3. Affected individual notification
4. HHS notification (if ≥500 individuals)

Track these as subtasks on this incident.
```

Add HIPAA-specific subtasks to the incident task.

## Step 8: Summary and Next Steps

Present the incident summary:

```
INCIDENT LOGGED — [Asana task link]

Severity: P{N}
Status: Phase 2 complete (Triage)
Next phase: Investigation

Immediate actions:
- {List any immediate actions identified during triage}

Notification requirements:
- {Who needs to be notified per escalation path}
- {HIPAA notification if applicable}

Next steps:
1. Complete investigation (Phase 3) and update the Asana task
2. Document containment actions (Phase 4) as they're taken
3. Run /incident-log again to update progress, or update the Asana task directly
```

If Vanta API is available (check compliance-config.md):
- Offer to sync the incident to Vanta: "Sync this incident to Vanta?"
