---
description: Execute today's Asana agent queue tasks
allowed-tools: Read, Write, Edit, Bash, Grep, Glob
---

Execute the Asana agent queue. This command delegates to the `asana-agent` skill for task processing.

## Workflow

1. **Load the asana-agent skill** and follow its complete workflow:
   - Load Asana GIDs from `~/.claude/memory/asana-config.md`
   - Retrieve tasks assigned to me, due today, with Task Progress = "Agent Queue"
   - Present the queue for review
   - Process each task sequentially with clarification, skill matching, execution, and closeout

> **Platform constraint check:** Before writing or modifying code for a managed platform
> (n8n, Cloud Run, Google APIs, HubSpot, Apollo), check the relevant reference doc at
> `~/Projects/nudesk-os-plugin/references/platform-references/`. See CLAUDE.md for the full table.

2. **After each task completes**, check if any new Asana tasks should be created as follow-ups. If so, suggest them (see session closeout behavior below).

3. **After all tasks are processed**, run session closeout:
   - Summarize what was completed
   - Suggest any follow-up tasks that should be created in Asana
   - Present suggestions for user confirmation before creating
   - Update memory if new context emerged (new contacts, terms, project updates)

Follow the asana-agent skill's process exactly — it handles clarification, skill matching, execution, and Asana updates.
