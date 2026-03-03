---
description: Add a new scheduled Asana task template to the automation service
argument-hint: [optional template name hint]
allowed-tools: Read, Grep, Glob, Bash, Edit, Write, AskUserQuestion
---

Walk through adding a new scheduled task template to the Asana Task Automation service. This command discovers templates from Asana, updates the config, and creates a Cloud Scheduler job.

## Prerequisites

The Asana Task Automation service must be deployed to Cloud Run. The project lives at `~/Documents/claude-code/nudesk-internal/asana-task-automation/`.

## Config

Load Asana GIDs from `~/.claude/memory/asana-config.md` for project routing.
Read `~/Documents/claude-code/nudesk-internal/asana-task-automation/config/templates.yaml` for current template config.

## Step 1: Choose Project

Ask the user which Asana project has the template. Present options from the routing table in asana-config.md:

- Invoicing & Cash Management (1212248321380340) — invoicing, billing, payroll
- Champions Funding (1211943050514909) — CHP operations
- People & Ops Management (1211903626313596) — people ops, HR
- Executive Management (1211894375322111) — executive tasks
- Other (let user specify)

If `$ARGUMENTS` contains a hint, pre-select the most likely project.

## Step 2: Discover Templates

Call the Asana API to list task templates in the selected project:

```bash
cd ~/Documents/claude-code/nudesk-internal/asana-task-automation && python3 -m app.services.asana_service list-templates <project_gid>
```

Display the results and ask which template to schedule. If no templates found, tell the user they need to create a task template in Asana first.

## Step 3: Configure Schedule

Ask the user three questions:

1. **Recurrence pattern:**
   - Monthly on a specific day (e.g., 1st, 15th)
   - Twice monthly on two days (e.g., 1st and 15th)
   - Weekly on a specific day (e.g., Monday)

2. **Trigger time:** Default 8:00 AM. Let user override.

3. **Timezone:** Default America/Phoenix. Let user override.

Build the cron expression:
- Monthly day 15 at 8am: `0 8 15 * *`
- Twice monthly (1st and 15th): creates TWO scheduler jobs
- Weekly Monday at 8am: `0 8 * * 1`

## Step 4: Generate Template ID

Create a slug from the template name:
- Lowercase, hyphens for spaces
- Remove special characters
- Example: "Champions Pre-Payroll - 15th Cycle" → `champions-pre-payroll-15th-cycle`

Show the generated ID and let user override.

## Step 5: Update Config

Add the new entry to `~/Documents/claude-code/nudesk-internal/asana-task-automation/config/templates.yaml`:

```yaml
<template-id>:
  template_gid: "<discovered_gid>"
  name: "<template name>"
  description: "<user-provided or inferred description>"
  project_gid: "<project_gid>"
```

## Step 6: Create Cloud Scheduler Job

Read the Cloud Run service URL. Check these locations in order:
1. `~/Documents/claude-code/nudesk-internal/asana-task-automation/.env` for `CLOUD_RUN_URL`
2. Ask user to provide it, or fetch via:
   ```bash
   gcloud run services describe asana-task-automation --region us-west1 --format 'value(status.url)'
   ```

Create the scheduler job:
```bash
gcloud scheduler jobs create http <template-id>-schedule \
  --location us-west1 \
  --schedule "<cron_expression>" \
  --time-zone "<timezone>" \
  --uri "<CLOUD_RUN_URL>/instantiate" \
  --http-method POST \
  --headers "Content-Type=application/json" \
  --message-body '{"template_id":"<template-id>"}' \
  --oidc-service-account-email <SERVICE_ACCOUNT> \
  --oidc-token-audience "<CLOUD_RUN_URL>"
```

If the service account is not known, ask the user or attempt to discover it from the existing Cloud Run service config.

## Step 7: Commit and Confirm

Commit the templates.yaml change:
```bash
cd ~/Documents/claude-code/nudesk-internal/asana-task-automation && git add config/templates.yaml && git commit -m "feat: add scheduled template <template-id>"
```

Confirm to user:
```
Scheduled: <template name>
Template ID: <template-id>
Schedule: <human-readable schedule>
Cloud Scheduler job: <job-name>
Config updated: config/templates.yaml
```

If this is a "twice monthly" schedule, note that two separate Cloud Scheduler jobs were created.
