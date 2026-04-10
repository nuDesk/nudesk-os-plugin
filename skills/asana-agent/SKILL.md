---
name: asana-agent
description: >
  Autonomous Asana task executor. Use when asked to "run my tasks",
  "check my Asana queue", "process today's tasks", "work through my agent queue", or
  any request to execute tasks from Asana. Retrieves tasks where
  Task Progress = "Agent Queue", processes each autonomously by matching to relevant
  skills, completes deliverables, marks tasks done, and moves to the next.
version: 2.0.0
---

# Asana Agent — Autonomous Task Executor

Autonomous workflow for processing Asana tasks. This agent retrieves queued tasks, works through them independently, and marks them complete. It only pauses to ask the user when genuinely blocked.

## Core Principles

1. **Understand before acting** — Gather full context from the task, comments, and attachments before executing.
2. **Operate autonomously** — This is a self-driving workflow. Proceed through tasks without waiting for approval unless you are genuinely blocked and cannot make a reasonable judgment call.
3. **Draft, never send** — All external communications (emails, Slack messages, Google Chat messages) must be prepared as **drafts only**. Never send on behalf of the user unless explicitly instructed otherwise for a specific task.
4. **Use available skills** — Match tasks to the right skill for best results.

---

## Workflow

### 1. Retrieve Tasks

Load Asana GIDs from `~/.claude/memory/asana-config.md` for workspace, user, project, and custom field GIDs.

Search Asana for tasks matching ALL criteria:
- Assigned to: me
- Custom field "Task Progress" = "Agent Queue"

Use `asana_search_tasks` with the workspace GID from config. Pass these parameters:
- `workspace`: workspace GID from config
- `assignee_any`: `"me"`
- `custom_fields`: Task Progress field GID set to Agent Queue option GID (both from config)

If no tasks found, report the empty queue and stop.

Present the task queue with a brief summary of each task, then begin processing.

---

### 2. Process Each Task

For each task:

#### a) Comprehensive Task Review

Before doing anything else, gather ALL available context:

- Fetch full task with `asana_get_task` including custom fields
- Read: name, description (notes/html_notes)
- Retrieve comments/history via `asana_get_stories_for_task`
- Check for attachments via `asana_get_attachments_for_object`
- If attachments exist, review their contents
- Look for related context: parent tasks, subtasks, project context

**Important:** Do not limit analysis to just the task title and description. Many tasks are created as quick captures and may lack full context. Attachments, comments, and related items often contain the actual requirements.

#### b) Handle Attachments

- If attachments exist, attempt to access and review their contents
- If attachments are not accessible, note this in the task closeout comment and proceed with available information

#### c) Clarification (only if blocked)

**Only ask the user questions if missing information would prevent you from completing the task.** If the task is clear enough to make a reasonable attempt, proceed without asking.

Ask when:
- The task is ambiguous in a way that could lead to wasted effort (e.g., two very different possible interpretations)
- Critical information is missing and cannot be inferred from context
- The task requires a decision only the user can make (e.g., which vendor to choose)

Do NOT ask when:
- You can infer the answer from task context, comments, or memory
- The question is about preferences you can make a reasonable default choice on
- You're asking "just to be safe" — bias toward action

---

### 3. Identify Relevant Skills

Dynamically evaluate available capabilities:

1. **Scan available skills** — Check `~/.claude/skills/` and any plugin skills for relevant matches
2. **Review skill descriptions** to match task requirements to the most appropriate skill(s)
3. **Consider combining skills** when a task spans multiple domains

If a relevant skill is identified:
- Read the SKILL.md file to understand its workflow and requirements
- Follow the skill's prescribed process

If no specific skill matches:
- Complete the task using general capabilities based on task description and context

---

### 4. Execute Task

Complete the deliverable based on the task requirements and chosen approach. Save any output files as appropriate for the project context.

**External communications rule:** If the task involves sending emails, Slack messages, Google Chat messages, or any outbound communication:
- Prepare the content as a **draft** (e.g., Gmail draft via `gws gmail +draft`)
- Do NOT send unless the task explicitly says "send" and the user has pre-authorized sending for this task
- Note all drafts created in the task closeout comment so the user knows to review and send them

---

### 5. Close Out Task

After completing each task:

#### a) Add Comment
Use `asana_create_task_story` to add a plain-text summary:
- What was completed
- Files/deliverables created (with names)
- Drafts prepared (if any) — specify where to find them (e.g., "Gmail draft: [subject]")
- Any notes, caveats, or recommended follow-ups

#### b) Mark Complete
Use `asana_update_task` to:
- Set `completed: true`
- Update custom field "Task Progress" to "Done"

#### c) Handle Failures
If a task fails during execution (tool error, missing access, unresolvable ambiguity):
- Add a comment to the task explaining what failed and why
- Leave the task status as "Agent Queue" (do NOT mark complete)
- Surface the failure in the session wrap-up
- Move to the next task

#### d) Move to Next Task
Proceed to the next task in the queue immediately. No user sign-off required.

---

### 6. Session Wrap-Up

After all tasks are processed:

1. **Final Summary** — Consolidated summary of all tasks completed, any that failed, and outstanding follow-ups
2. **Communications Audit** — Enumerate every external communication touched during the session and its disposition (draft created, skipped, or flagged). This confirms the draft-only guardrail was respected.
3. **Follow-up tasks** — If execution surfaced new action items, create them in Asana or suggest them to the user
