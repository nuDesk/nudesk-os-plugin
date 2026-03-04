---
description: Add a new scheduled Asana task template to the automation service
argument-hint: [optional template name hint]
allowed-tools: Read, Grep, Glob, Bash, Edit, Write
---

Add a new scheduled task template to the Asana Task Automation service. Discovers templates from Asana, updates config, and creates a Cloud Scheduler job.

## EXECUTION RULES — READ BEFORE ANYTHING ELSE

This command is a **strict state machine**. You execute ONE step per turn.

1. **ONE STEP PER TURN.** After completing a step's action and calling `AskUserQuestion`, your turn is DONE. Output nothing after the `AskUserQuestion` call. Do not preview the next step. Do not explain what comes next.
2. **NEVER call Bash and AskUserQuestion in the same turn.** If a step requires a Bash call followed by a question, call Bash first, then in the SAME turn call AskUserQuestion with the results. But do NOT proceed to the next step.
3. **NEVER read ahead.** Do not reference, summarize, or execute any step beyond the current one.
4. **Wait for the user's answer** before moving to the next step. The user's response to `AskUserQuestion` is your signal to advance.

## Config — Hardcoded Values (use these exactly, never discover or override)

- **GCP project:** `nudesk-agent-builder`
- **Region:** `us-west1`
- **Service URL:** `https://asana-task-automation-1039881044029.us-west1.run.app`
- **Service account:** `cloud-scheduler-invoker@nudesk-agent-builder.iam.gserviceaccount.com`
- **Templates config:** `~/Documents/claude-code/nudesk-internal/asana-task-automation/config/templates.yaml`
- **Service repo:** `~/Documents/claude-code/nudesk-internal/asana-task-automation/`

---

## Step 1: Choose Project

Load Asana GIDs from `~/.claude/memory/asana-config.md`.

Call `AskUserQuestion` with these project options:
- Invoicing & Cash Management (1212248321380340) — invoicing, billing, payroll
- Champions Funding (1211943050514909) — CHP operations
- People & Ops Management (1211903626313596) — people ops, HR
- Executive Management (1211894375322111) — executive tasks

If `$ARGUMENTS` contains a hint, pre-select the most likely project.

**→ Call `AskUserQuestion`. Your turn ends here.**

---

## Step 2: Discover Templates

Run this command to list templates in the project the user chose:

```bash
cd ~/Documents/claude-code/nudesk-internal/asana-task-automation && python3 -m app.services.asana_service list-templates <project_gid>
```

Then present the discovered templates to the user via `AskUserQuestion` — ask which template to schedule. If no templates found, tell the user to create one in Asana first.

**→ Call Bash, then call `AskUserQuestion` with the results. Your turn ends here.**

---

## Step 3: Configure Schedule

Call `AskUserQuestion` with these questions (can be a single multi-question call):

1. **Recurrence pattern:** Monthly on a specific day / Twice monthly / Weekly on a specific day
2. **Trigger time:** Default 8:00 AM
3. **Timezone:** Default America/Phoenix

**→ Call `AskUserQuestion`. Your turn ends here.**

---

## Step 4: Confirm Template ID

Build the cron expression from the user's schedule answers:
- Monthly day 15 at 8am: `0 8 15 * *`
- Twice monthly (1st and 15th): creates TWO scheduler jobs
- Weekly Monday at 8am: `0 8 * * 1`

Generate a slug from the template name (lowercase, hyphens, no special chars).

Call `AskUserQuestion` showing the generated template ID and cron expression. Ask user to confirm or override.

**→ Call `AskUserQuestion`. Your turn ends here.**

---

## Step 5: Update Config

Read then edit `~/Documents/claude-code/nudesk-internal/asana-task-automation/config/templates.yaml`.

Add the new entry:
```yaml
<template-id>:
  template_gid: "<discovered_gid>"
  name: "<template name>"
  description: "<user-provided or inferred description>"
  project_gid: "<project_gid>"
```

Then proceed directly to Step 6 (no user input needed here).

---

## Step 6: Create Cloud Scheduler Job

Do NOT run gcloud describe, check .env files, or attempt to discover infrastructure values. Use the hardcoded config above.

```bash
gcloud scheduler jobs create http <template-id>-schedule \
  --project nudesk-agent-builder \
  --location us-west1 \
  --schedule "<cron_expression>" \
  --time-zone "<timezone>" \
  --uri "https://asana-task-automation-1039881044029.us-west1.run.app/instantiate" \
  --http-method POST \
  --headers "Content-Type=application/json" \
  --message-body '{"template_id":"<template-id>"}' \
  --oidc-service-account-email cloud-scheduler-invoker@nudesk-agent-builder.iam.gserviceaccount.com \
  --oidc-token-audience "https://asana-task-automation-1039881044029.us-west1.run.app"
```

If twice monthly, create two jobs with suffixes `-1st-schedule` and `-15th-schedule`.

Then proceed directly to Step 7.

---

## Step 7: Commit and Confirm

Commit the templates.yaml change:
```bash
cd ~/Documents/claude-code/nudesk-internal/asana-task-automation && git add config/templates.yaml && git commit -m "feat: add scheduled template <template-id>"
```

Output a summary:
```
Scheduled: <template name>
Template ID: <template-id>
Schedule: <human-readable schedule>
Cloud Scheduler job: <job-name>
Config updated: config/templates.yaml
```

Done.
