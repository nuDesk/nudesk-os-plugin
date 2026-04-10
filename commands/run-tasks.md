---
description: Execute Asana agent queue tasks autonomously via Ralph Loop
allowed-tools: Read, Write, Edit, Bash, Grep, Glob
---

Execute the Asana agent queue autonomously. This command starts a Ralph Loop that drives the `asana-agent` skill through the full task queue until all tasks are completed.

## Workflow

1. **Start a Ralph Loop** to process the queue:

```
/ralph-loop Process all Asana Agent Queue tasks using the asana-agent skill. Load GIDs from ~/.claude/memory/asana-config.md. Retrieve tasks assigned to me with Task Progress = Agent Queue. For each task: gather full context, match to relevant skills, execute the deliverable, add a summary comment, mark the task completed in Asana, then move to the next. Draft all external communications (emails, messages) — never send. After all tasks are done, print a final summary of completed tasks and any drafts awaiting review. --completion-promise 'All Agent Queue tasks have been processed and completed'
```

2. **The Ralph Loop will autonomously:**
   - Load the asana-agent skill and follow its workflow
   - Process each task: review, skill-match, execute, close out, mark complete
   - Only pause to ask clarifying questions when genuinely blocked
   - Draft (never send) any external communications
   - Iterate until all Agent Queue tasks are done, then exit via completion promise

> **Platform constraint check:** Before writing or modifying code for a managed platform
> (n8n, Cloud Run, Google APIs, HubSpot, Apollo), check the relevant reference doc at
> `~/Projects/nudesk-os-plugin/references/platform-references/`. See CLAUDE.md for the full table.

3. **After the loop completes**, run session closeout:
   - Summarize all tasks completed and drafts awaiting review
   - Create any follow-up tasks in Asana that surfaced during execution
   - Note any drafts (emails, messages) that need the user's review and send
