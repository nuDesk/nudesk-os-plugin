---
description: List, edit, or delete Cloud Scheduler jobs for Asana task automation
argument-hint: [list | edit | delete]
allowed-tools: Read, Grep, Glob, Bash, Edit, AskUserQuestion
---

Manage the Cloud Scheduler jobs that trigger the Asana Task Automation service. Supports listing, editing, and deleting scheduled templates.

## Config

### Infrastructure (stable — use these exactly)

- **GCP project:** `nudesk-agent-builder`
- **Region:** `us-west1`
- **Service URL:** `https://asana-task-automation-1039881044029.us-west1.run.app`
- **Service account:** `cloud-scheduler-invoker@nudesk-agent-builder.iam.gserviceaccount.com`

### Local Paths (from user config)

Read the **Scheduled Task Automation** section of `~/.claude/memory/asana-config.md` to get:
- **Templates config** — path to `config/templates.yaml`
- **Service repo** — path to the service repo root

If the Scheduled Task Automation section is missing from asana-config.md, ask the user for the paths.

## Step 1: Determine Mode

If `$ARGUMENTS` contains "list", "edit", or "delete", go directly to that mode. Otherwise, ask:

```
What would you like to do?
1. List — View all scheduled jobs with status and next run
2. Edit — Change schedule, time, timezone, or pause/resume a job
3. Delete — Remove a scheduler job and optionally clean up config
```

---

## Mode: List

Run:
```bash
gcloud scheduler jobs list --location us-west1 --format="table(name.basename(), schedule, timeZone, state, status.latestExecTime, scheduleTime)" 2>/dev/null
```

Present the results in a clean table:
```
SCHEDULED JOBS:

| # | Job Name                                  | Schedule    | Timezone        | State   | Last Run          | Next Run          |
|---|-------------------------------------------|-------------|-----------------|---------|-------------------|-------------------|
| 1 | champions-pre-payroll-15th-schedule        | 0 8 1 * *   | America/Phoenix | ENABLED | 2026-03-01 08:00  | 2026-04-01 08:00  |
| 2 | invoice-champions-eom-cycle-schedule       | 0 8 1 * *   | America/Phoenix | ENABLED | —                 | 2026-04-01 08:00  |
...
```

After displaying, ask if the user wants to edit or delete any of them. If not, done.

---

## Mode: Edit

### Step 1: List and select

First run the List mode to show all jobs. Then ask the user which job to edit (by number or name).

### Step 2: Show current config

Run:
```bash
gcloud scheduler jobs describe <job-name> --location us-west1 --format="yaml(schedule, timeZone, state, httpTarget.uri, httpTarget.body)" 2>/dev/null
```

Decode the base64 body to show the template_id:
```bash
echo "<base64_body>" | base64 --decode
```

Present:
```
Current config for <job-name>:
  Schedule:    0 8 1 * *  (1st of every month at 8:00 AM)
  Timezone:    America/Phoenix
  State:       ENABLED
  Template ID: champions-pre-payroll-15th
```

### Step 3: Ask what to change

Present options:
1. **Change schedule** — New cron expression (help the user build it if they describe in plain English)
2. **Change timezone** — Switch timezone
3. **Pause** — Temporarily stop the job (keeps config, just won't fire)
4. **Resume** — Re-enable a paused job
5. **Cancel** — Go back

### Step 4: Apply changes

For schedule changes:
```bash
gcloud scheduler jobs update http <job-name> --location us-west1 --schedule "<new_cron>" --time-zone "<timezone>"
```

For pause:
```bash
gcloud scheduler jobs pause <job-name> --location us-west1
```

For resume:
```bash
gcloud scheduler jobs resume <job-name> --location us-west1
```

Confirm the change was applied by describing the job again.

---

## Mode: Delete

### Step 1: List and select

First run the List mode to show all jobs. Then ask the user which job to delete (by number or name).

### Step 2: Confirm

Show the job details and ask for explicit confirmation:
```
About to delete: <job-name>
Schedule: <cron> (<human-readable>)
Template: <template_id>

This will stop the scheduled task creation. The template config in templates.yaml will remain unless you also want to remove it.

Delete this job? [yes/no]
```

### Step 3: Delete

```bash
gcloud scheduler jobs delete <job-name> --location us-west1 --quiet
```

### Step 4: Optionally clean up config

Ask if the user also wants to remove the template entry from `templates.yaml`. If yes:
- Read `<templates-config-path>`
- Remove the matching template entry
- Commit the change

---

## Cron Helper

When users describe schedules in plain English, translate to cron:

| Description | Cron |
|-------------|------|
| 1st of every month at 8am | `0 8 1 * *` |
| 15th of every month at 8am | `0 8 15 * *` |
| Every Monday at 9am | `0 9 * * 1` |
| Every weekday at 7:30am | `30 7 * * 1-5` |
| 1st and 15th at 8am | Two jobs: `0 8 1 * *` and `0 8 15 * *` |
| Last day of month | Approximate with `0 8 28 * *` and note the limitation |

Always show the cron expression and ask for confirmation before applying.
