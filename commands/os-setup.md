---
description: Guided setup wizard for nuDesk OS — configure CLAUDE.md, Asana, memory, hooks, and MCP servers
allowed-tools: Read, Write, Edit, Bash, Grep, Glob, mcp__plugin_asana_asana__asana_list_workspaces, mcp__plugin_asana_asana__asana_get_user, mcp__plugin_asana_asana__asana_get_teams_for_workspace, mcp__plugin_asana_asana__asana_get_projects_for_team, mcp__plugin_asana_asana__asana_get_project, mcp__plugin_asana_asana__asana_search_tasks, mcp__plugin_asana_asana__asana_get_task
---

Guided, interactive setup for nuDesk OS. Detects what's already configured, walks through what's missing, and auto-discovers Asana GIDs via MCP.

Safe to run multiple times — every step checks existing state first and skips what's already done.

## EXECUTION RULES — READ BEFORE ANYTHING ELSE

This command is a **strict state machine**. You execute ONE step per turn.

1. **ONE STEP PER TURN.** After completing a step's action and calling `AskUserQuestion`, your turn is DONE. Output nothing after the `AskUserQuestion` call. Do not preview the next step. Do not explain what comes next.
2. **NEVER call Bash and AskUserQuestion in the same turn.** If a step requires a Bash call followed by a question, call Bash first, then in the SAME turn call AskUserQuestion with the results. But do NOT proceed to the next step.
3. **NEVER read ahead.** Do not reference, summarize, or execute any step beyond the current one.
4. **Wait for the user's answer** before moving to the next step. The user's response to `AskUserQuestion` is your signal to advance.
5. **Skip completed steps.** If a step's detection check shows the item is already configured, show a green status line and move to the next step in the SAME turn (no need to ask).

---

## Step 0: Configuration Scan

Run all of these checks in parallel:

### 0a. CLAUDE.md
- Check if `~/.claude/CLAUDE.md` exists
- If it exists, read it and check for placeholder markers like `[Your Name]`, `[Your Role]`, `[your-email@company.com]`
- Status: **Complete** (exists with real values), **Partial** (exists but has placeholders), or **Missing**

### 0b. Memory Directories
- Check if these directories exist: `~/.claude/memory/`, `~/.claude/memory/people/`, `~/.claude/memory/projects/`, `~/.claude/memory/context/`
- Status: **Complete** (all exist), **Partial** (some exist), or **Missing**

### 0c. Asana Config
- Check if `~/.claude/memory/asana-config.md` exists
- If it exists, check for placeholder markers like `[workspace-gid]`, `[gid]`, `[your-user-gid]`
- Status: **Complete** (exists with real GIDs), **Partial** (exists but has placeholders), or **Missing**

### 0d. MCP Servers
- Read `~/.claude.json` (global config) — check for `mcpServers` keys containing: `n8n-mcp`, `figma`, `context7`, `playwright`
- Check for `gws` CLI: run `gws auth status` via Bash to verify Google Workspace access
- Also check `~/.claude/settings.json` and any workspace-level config
- For each, report: **Configured** or **Not configured**

### 0e. Compliance Hooks
- Read `~/.claude/settings.json` (if it exists)
- Check for a `PreToolUse` hook with `Edit|Write` matcher that blocks `.env` file edits
- Status: **Installed** or **Not installed**

### 0f. Recommended Plugins
- Read `~/.claude/plugins/installed_plugins.json` (if it exists)
- Check for `pr-review-toolkit`
- Status: **Installed** or **Not installed**

### Present the Dashboard

```
EXECUTIVE OS SETUP — Configuration Scan

  CLAUDE.md                 — [Complete / Partial / Missing]
  Memory directories        — [Complete / Partial / Missing]
  Asana config              — [Complete / Partial / Missing]
  MCP: Asana                — [Configured / Not configured] (required)
  gws CLI                   — [Authed / Not authed / Not installed] (recommended)
  MCP: Fireflies            — [Configured / Not configured] (optional)
  MCP: HubSpot              — [Configured / Not configured] (optional)
  Compliance hooks          — [Installed / Not installed]
  Plugin: pr-review-toolkit — [Installed / Not installed] (optional)
```

Then call `AskUserQuestion`: "Proceed with all incomplete steps, or would you like to select specific steps?"

**→ Call `AskUserQuestion`. Your turn ends here.**

---

## Step 1: CLAUDE.md Setup

**If Complete:** Show status and skip to Step 2.

**If Missing or Partial:**

1. Read the template from `~/Projects/nudesk-os-plugin/templates/CLAUDE.md.template`.

2. Call `AskUserQuestion` with:

> To generate your CLAUDE.md, I need a few details:
>
> 1. **Full name and role** (e.g., "Jane Smith, VP Engineering at Acme Corp")
> 2. **Email address**
> 3. **Company name and one-line description** (e.g., "Acme Corp — AI-powered logistics")
> 4. **Primary programming language** (e.g., Python, TypeScript)
> 5. **Key tools and platforms** you use daily (e.g., GCP, n8n, HubSpot, Slack)
> 6. **How should Claude communicate with you?** (e.g., "concise and direct", "explain trade-offs before acting")

**→ Call `AskUserQuestion`. Your turn ends here.**

### On user response:

3. Generate a CLAUDE.md from the template, substituting the user's answers into:
   - `Who I Am` section (name, role, company)
   - `How I Work` section (language, communication style)
   - `My Stack` section (tools and platforms)
   - `Company Context` section (company name, description)
   - `Integration Config` → Email field

4. Leave these sections as template placeholders with their guidance comments intact — the user will fill these in over time:
   - Strategic Goals
   - Priorities (set `last_updated` to today's date)
   - Working Memory (People, Terms, Clients tables)
   - Integration Config (HubSpot Owner ID, Report Recipient, Report Task Name)

5. Present the generated CLAUDE.md to the user for review.

6. Call `AskUserQuestion`: "Here's your starter CLAUDE.md. Want me to write it to `~/.claude/CLAUDE.md`? You can refine the Strategic Goals, Priorities, and Working Memory sections later."

**→ Call `AskUserQuestion`. Your turn ends here.**

### On confirmation:

7. Write the file to `~/.claude/CLAUDE.md`.
8. Proceed to Step 2.

---

## Step 2: Memory Directory Setup

**If Complete:** Show status and skip to Step 3.

**If Missing or Partial:**

1. Show which directories exist vs. missing.
2. Create the missing directories:
   ```bash
   mkdir -p ~/.claude/memory/people ~/.claude/memory/projects ~/.claude/memory/context
   ```
3. Read `~/Projects/nudesk-os-plugin/templates/memory-scaffold.md` and create a starter `~/.claude/memory/glossary.md` with the glossary template:
   ```markdown
   # Glossary

   | Term | Meaning |
   |------|---------|
   ```
4. Report what was created and proceed to Step 3.

This step requires no user interaction unless all directories already exist. If directories were created, briefly confirm and move on.

---

## Step 3: MCP Server Verification

### 3a. Asana (Required)

**If not configured:**
- Tell the user Asana MCP is required for nuDesk OS and provide the install command:
  ```
  claude mcp add asana -- npx -y @anthropic/asana-mcp-server
  ```
- Explain they'll need to re-run `/nudesk-os:os-setup` after installing.
- Call `AskUserQuestion`: "Asana MCP is not configured. Would you like to install it now, or skip and come back later?"
- **→ Your turn ends here.**

**If configured:**
- Smoke test: call `asana_list_workspaces`
- If successful, show the workspace name(s) and mark as verified
- If it errors, report the error and suggest troubleshooting (re-auth, check token)

### 3b. Google Workspace via `gws` CLI (Recommended)

The `gws` CLI provides access to Gmail, Calendar, Drive, Chat, Docs, Sheets, Slides, and People — all via Bash commands (not an MCP server). Each user authenticates with their own nuDesk Google account.

#### 3b-i. Check if `gws` is installed

```bash
which gws
```

**If not found:** Provide these install steps and wait for the user to complete them before continuing:

```bash
brew install google/gws/gws
npx skills add --yes --global https://github.com/googleworkspace/cli
```

Explain: "This installs the Google Workspace CLI and 42 Agent Skills that let Claude access your Gmail, Calendar, Drive, Chat, Docs, and Sheets."

After the user confirms, proceed to 3b-ii.

**If found:** Proceed directly to 3b-ii.

#### 3b-ii. Check auth status

```bash
gws auth status
```

Parse the output:

**If `token_valid` is not `true`:**
- Instruct: "Run `gws auth setup` to authenticate with your nuDesk Google account. A browser window will open — sign in with your nuDesk email (@nudesk.ai)."
- Call `AskUserQuestion`: "Run `gws auth setup` now and let me know when you've completed the browser sign-in."
- **→ Your turn ends here.** After they confirm, run `gws auth status` again to verify.
- If it still fails: point to `~/Projects/nudesk-os-plugin/references/setup/gws-cli-setup.md` for detailed troubleshooting.

**If `token_valid` is `true`:** Proceed to 3b-iii.

#### 3b-iii. Smoke test

```bash
gws gmail users messages list --params '{"userId":"me","maxResults":1}'
```

**If successful:** Show green status — "gws CLI: Connected and authenticated."

**If failed:** Diagnose:
- Wrong account: "Are you authenticated with your nuDesk email? Run `gws auth status` to check which account is active."
- Missing Gmail scope: "Run `gws auth login -s gmail` to add Gmail access."
- Other error: Reference `~/Projects/nudesk-os-plugin/references/setup/gws-cli-setup.md` troubleshooting section.

#### 3b-iv. Scope check

Run `gws auth status` and verify these scopes are present in the output:
- `gmail.readonly` (or broader scope like `gmail.modify` or `gmail`)
- `calendar.readonly` (or broader)
- `drive.readonly` (or broader)

**If any are missing:** Run:
```bash
gws auth login -s gmail,calendar,drive
```
Then verify again with `gws auth status`.

### 3c. Optional MCP Servers (Fireflies, HubSpot)

For each claude.ai connector: note whether configured. If not, briefly explain what it enables. Do not push for installation.

### Present MCP Summary

Show a table of all MCP servers with their status (Verified / Configured but untested / Not configured).

Call `AskUserQuestion`: "MCP server status shown above. Ready to continue to Asana config setup?"

**→ Call `AskUserQuestion`. Your turn ends here.**

---

## Step 4: Asana Config Auto-Discovery

**If Asana MCP is not available (not configured or failed smoke test):** Skip this step. Tell the user to install Asana MCP first, then re-run os-setup.

**If asana-config.md is Complete (exists with real GIDs, no placeholders):** Show status and skip to Step 5.

### Sub-step 4a: Workspace and User

1. Call `asana_list_workspaces`.
   - If one workspace: confirm it with the user.
   - If multiple: ask the user to choose.
2. Call `asana_get_user` with `user_gid: "me"` to get the user's GID and name.
3. Present: "Found workspace **[name]** (`[gid]`) and user **[name]** (`[gid]`). Correct?"

Call `AskUserQuestion` to confirm.

**→ Call `AskUserQuestion`. Your turn ends here.**

### Sub-step 4b: Teams and Projects

1. Call `asana_get_teams_for_workspace` with the confirmed workspace GID.
2. For each team, call `asana_get_projects_for_team` to get the project list.
3. Present all projects organized by team:
   ```
   Team: [Team Name]
     - [Project Name] (GID: [gid])
     - [Project Name] (GID: [gid])

   Team: [Team Name]
     - [Project Name] (GID: [gid])
   ```
4. Call `AskUserQuestion`: "Here are your Asana projects. For each project, I'll need routing keywords — words that should trigger task creation in that project. Which projects are client-facing vs. internal? Any you want to exclude from routing?"

**→ Call `AskUserQuestion`. Your turn ends here.**

### Sub-step 4c: Custom Field Discovery

1. Use `asana_search_tasks` to find 3-5 recent tasks in the workspace that are assigned to the user. Request `opt_fields: custom_fields`.
2. For the first task that has custom fields, call `asana_get_task` with:
   ```
   opt_fields: custom_fields,custom_fields.name,custom_fields.enum_options,custom_fields.enum_options.name,custom_fields.gid,custom_fields.enum_options.gid
   ```
3. From the response, look for fields named (case-insensitive):
   - **Task Progress** (or similar: "Status", "Progress") — extract field GID and all enum option GIDs with names
   - **Type** (or similar: "Task Type", "Category") — same
   - **Priority** — same
4. If not all three fields are found on the first task, try 2-3 more tasks from different projects.
5. Present what was found:
   ```
   Custom Fields Discovered:

   Task Progress (GID: [gid])
     Not Started: [gid]
     Agent Queue: [gid]
     In Progress: [gid]
     ...

   Type (GID: [gid])
     [Type 1]: [gid]
     ...

   Priority (GID: [gid])
     High: [gid]
     Medium: [gid]
     Low: [gid]
   ```
   If any field was NOT found, note it and explain the user can fill it in manually later (reference the "How to Find Your GIDs" section in the template).

Call `AskUserQuestion`: "Here are the custom fields I discovered. Do these look correct? Any fields missing that you'd like me to search for?"

**→ Call `AskUserQuestion`. Your turn ends here.**

### Sub-step 4d: Generate and Confirm

1. Read the template from `~/Projects/nudesk-os-plugin/templates/asana-config.md.template`.
2. Assemble the complete `asana-config.md` by filling in all discovered values:
   - Workspace name and GID
   - User name and GID
   - Custom field GIDs and enum option GIDs
   - Projects organized into Client Projects and Internal Projects with routing keywords
   - Set the verification date to today
3. Leave these sections as templates (user fills in later):
   - Additional users (Report Recipient)
   - Scheduled Task Automation paths
4. If `~/.claude/memory/asana-config.md` already exists with manual customizations (e.g., Scheduled Task Automation paths filled in), preserve those sections.
5. Present the full generated config.

Call `AskUserQuestion`: "Here's your generated Asana config. Want me to write it to `~/.claude/memory/asana-config.md`?"

**→ Call `AskUserQuestion`. Your turn ends here.**

### On confirmation:

6. Write the file to `~/.claude/memory/asana-config.md`.
7. Proceed to Step 5.

---

## Step 5: Compliance Hooks

**If already installed:** Show status and skip to Step 6.

**If not installed:**

1. Read the hook template from `~/Projects/nudesk-os-plugin/templates/hooks-settings.json.template`.
   This hook is defined by the **soc2-compliance** skill (`~/Projects/nudesk-os-plugin/skills/soc2-compliance/SKILL.md`) — it is a core SOC 2 Type II control for Claude Code workflows.
2. Check if `~/.claude/settings.json` exists.
   - **If it exists:** Read it. Show the user what will be added (the PreToolUse hook). Merge the hook into the existing settings — preserve all other settings and hooks.
   - **If it does not exist:** Show the full settings.json content that will be created.
3. Call `AskUserQuestion`: "This hook prevents Claude from directly editing .env files — credentials must be modified manually. Install it to `~/.claude/settings.json`?"

**→ Call `AskUserQuestion`. Your turn ends here.**

### On confirmation:

4. Write or edit `~/.claude/settings.json` with the hook merged in.
5. Proceed to Step 6.

---

## Step 6: Recommended Plugins and Skills

**If pr-review-toolkit is installed:** Show status.

**If not installed:**

Inform the user:

> **pr-review-toolkit** provides 6 specialized review agents (code-reviewer, pr-test-analyzer, silent-failure-hunter, code-simplifier, comment-analyzer, type-design-analyzer) and a `/pr-review-toolkit:review-pr` command for comprehensive PR reviews. Optional — primarily useful for teams doing formal code review workflows.
>
> Install with:
> ```
> claude plugin add pr-review-toolkit@claude-plugins-official
> ```

This is informational only — do not auto-install. The user can run the command themselves if interested.

### Recommended Skills

The following skills are bundled with nuDesk OS and should already be installed at `~/.claude/skills/`. Check that these three are present:

```bash
ls ~/.claude/skills/srd-generator/SKILL.md
ls ~/.claude/skills/ai-solution-architect/SKILL.md
ls ~/.claude/skills/nudesk-brand-styling/SKILL.md
```

If any are missing, run from the plugin directory:
```bash
cd ~/Projects/nudesk-os-plugin/skills/bundles
unzip -o srd-generator.skill -d ~/.claude/skills/
unzip -o ai-solution-architect.skill -d ~/.claude/skills/
unzip -o nudesk-brand-styling.skill -d ~/.claude/skills/
```

**Skill descriptions:**

| Skill | Trigger | What it does |
|-------|---------|--------------|
| **srd-generator** | "PRD", "requirements document", "build brief", "spec out", "technical requirements" | Generates structured Solution Requirements Documents optimized for AI coding agents (Claude Code, Lovable, n8n) |
| **ai-solution-architect** | "help me design", "I need to build", "architecture review", "solution design", "how should we build", "evaluate options for" | Technical strategy partner — explores architecture options, build vs. buy decisions, and designs AI-powered solutions |
| **nudesk-brand-styling** | "nuDesk brand", "brand guidelines", "apply nuDesk styling", "make it on-brand", "nuDesk presentation", "nuDesk colors", "client-facing materials" | Applies nuDesk's official brand colors, typography, and design standards to any deliverable |

Proceed to Step 7.

---

## Step 7: Verification

Re-run the Step 0 checks (all in parallel). Present a final status report:

```
EXECUTIVE OS SETUP COMPLETE — [Date]

PASSED
- [Item]: [Brief note]
- ...

OPTIONAL (not configured)
- [Item]: [What it enables]
- ...

NEXT STEPS
1. Run /nudesk-os:daily-plan to test your full setup end-to-end
2. Fill in Strategic Goals, Priorities, and Working Memory in ~/.claude/CLAUDE.md as you work
3. Memory will grow automatically via /nudesk-os:session-closeout
```

Done.
