# Claude Code — Platform Reference

> Constraint doc for Claude Code CLI — the primary execution environment for all nuDesk development.

## Core Architecture

Claude Code is a CLI-based coding assistant that uses **tool use** to interact with the local filesystem, run commands, and integrate with external systems. The language model itself only processes text — all actions (file reads, writes, command execution) happen through a tool-use protocol where Claude requests an action, the CLI executes it, and results are fed back.

### Tool Categories

| Category | Tools | Notes |
|----------|-------|-------|
| **File I/O** | Read, Edit, Write, Glob, Grep | Prefer dedicated tools over Bash equivalents |
| **Execution** | Bash | Shell commands, git, build tools, deployments |
| **Search** | Glob (file patterns), Grep (content), WebSearch, WebFetch | Use Glob/Grep before Bash grep/find |
| **MCP Tools** | Any `mcp__*` tool | External integrations (Asana, HubSpot, Figma, etc.) |
| **Agent/Task** | Agent (subagents), Task (background work) | Parallel execution for independent work |

## Context Management

### CLAUDE.md Hierarchy (loaded every request)

1. **Global** — `~/.claude/CLAUDE.md` — personal instructions for all projects
2. **Project** — `<project>/.claude/CLAUDE.md` — shared team instructions, committed to git
3. **Local** — `<project>/.claude/settings.local.json` — personal overrides, not committed

### Context Controls

| Control | Purpose |
|---------|---------|
| `@file` mention | Include specific file contents in request context |
| `/init` | Auto-generate project CLAUDE.md from codebase analysis |
| `#` (memory mode) | Edit CLAUDE.md with natural language |
| Escape (1x) | Stop current response, redirect |
| Escape (2x) | Rewind conversation to earlier point |
| `/compact` | Summarize conversation, preserve learned context |
| `/clear` | Full reset for unrelated tasks |

### Context Best Practices

- Too much irrelevant context **decreases** performance — be selective
- Reference critical files (schemas, configs) in CLAUDE.md so they're always available
- Use `@` mentions for targeted context instead of letting Claude search broadly
- Use `/compact` when conversation is cluttered but Claude has useful learned state

## Performance Modes

| Mode | Trigger | Use Case |
|------|---------|----------|
| **Plan Mode** | Shift+Tab (2x) | Multi-step tasks, wide codebase understanding — handles **breadth** |
| **Thinking Mode** | "Ultra think" / "think harder" | Complex logic, tricky debugging — handles **depth** |
| **Combined** | Both | Complex multi-step tasks with hard logic |

Both consume additional tokens. Use judiciously on hard problems, not routine tasks.

## Hooks

Hooks are shell commands that run before/after Claude executes specific tools.

### Hook Types

| Type | Timing | Can Block? | Use Case |
|------|--------|------------|----------|
| **PreToolUse** | Before tool executes | Yes (exit 2) | Block dangerous operations, validate inputs |
| **PostToolUse** | After tool executes | No | Auto-format, run linters, type-check, feedback |
| **TeammateIdle** | Agent Teams: teammate has no tasks | Yes (exit 2) | Reassign work, prevent idle billing |
| **TaskCreated** | Agent Teams: new task created | Yes (exit 2) | Validate task scope, enforce naming |
| **TaskCompleted** | Agent Teams: task marked done | Yes (exit 2) | Quality gate — reject incomplete work |

### Hook Configuration

Location: `settings.json` (global or project-level) under `"hooks"` key.

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "your-command-here"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "your-post-command-here"
          }
        ]
      }
    ]
  }
}
```

### Hook Data Protocol

- Hooks receive tool call data as **JSON via stdin**: `{ session_id, tool_name, input: { ... } }`
- **Exit 0** = allow tool call to proceed
- **Exit 2** = block tool call (PreToolUse only)
- **stderr output** = feedback message sent back to Claude
- Must restart Claude Code after hook changes

### Proven Hook Patterns

| Hook | Type | Matcher | Purpose |
|------|------|---------|---------|
| **.env blocker** | PreToolUse | `Edit\|Write` | Block edits to `.env` files (allow `.env.example`) |
| **TypeScript type checker** | PostToolUse | `Edit\|Write` (on `.ts`/`.tsx`) | Run `tsc --no-emit` after edits, feed errors back |
| **ESLint auto-fix** | PostToolUse | `Edit\|Write` (on `.ts`/`.tsx`) | Run `eslint --fix` after edits |
| **Duplicate code detector** | PostToolUse | `Edit\|Write` (on watched dirs) | Launch secondary Claude to check for duplication |
| **Test runner** | PostToolUse | `Edit\|Write` (on `src/`) | Run relevant tests after source file changes |

## Agent Teams

Agent Teams coordinates multiple independent Claude Code sessions working in parallel — shared task lists, inter-agent messaging, and centralized management from a lead session.

**Key facts:**
- Requires `enableAgentTeams: true` in settings and Claude Code >= 2.1.32
- Default display mode is **in-process** (Shift+Down to cycle teammates)
- Teammates do NOT get `skills` or `mcpServers` from agent frontmatter — only `tools` allowlists
- One team per session, no nested teams, no session resumption for in-process mode
- Teammates can message each other directly via `SendMessage`

For full constraints, best practices, and hook event details, see the dedicated reference doc:
`~/Projects/nudesk-os-plugin/references/platform-references/claude-agent-teams.md`

## Custom Commands

- Location: `.claude/commands/` in project directory
- File naming: `filename.md` → `/filename` command
- Arguments: `$arguments` placeholder in markdown for runtime parameters
- Restart Claude Code after creating new command files

## MCP Servers

MCP servers extend Claude Code with external tool capabilities.

### Installation

```bash
claude mcp add <name> <start-command>
```

### Auto-Approve

Add `"mcp__<servername>__*"` patterns to `settings.json` → `permissions.allow` array.

### Key Constraints

- Each MCP tool requires individual permission listing (no wildcards for tool names in GitHub Actions)
- MCP servers run locally or remotely — verify security posture before production use
- Large API surfaces (e.g., Google Workspace) can overflow context — prefer CLI wrappers when available

## Claude Code SDK

Programmatic interface for embedding Claude Code in pipelines, scripts, and hooks.

- **Libraries**: CLI, TypeScript, Python
- **Default permissions**: Read-only (files, dirs, grep)
- **Write permissions**: Must be explicitly enabled via `options.allowTools`
- **Best for**: Helper commands, CI/CD scripts, hooks that need Claude intelligence

## GitHub Actions Integration

- Install via `/install GitHub app` command
- Two default actions: `@Claude` mention support + automatic PR review
- Custom instructions via config files in `.github/workflows/`
- MCP server tools require individual permission listing in Actions config
- Claude can run browser tests via Playwright MCP in CI

## Permission Model

### Permission Modes

| Mode | Behavior |
|------|----------|
| **acceptEdits** | Auto-approve reads, prompt for writes |
| **plan** | Research only, no modifications |

### Allow/Deny Lists

```json
{
  "permissions": {
    "allow": ["Read", "Edit", "Bash(git *)"],
    "deny": ["Bash(git push --force*)"]
  }
}
```

## Key Constraints & Gotchas

1. **Tool use quality = assistant quality** — Claude's tool use is its primary differentiator
2. **No persistent shell state** — each Bash call is a fresh shell (working directory persists)
3. **Context window limits** — conversation auto-compresses as it approaches limits; use `/compact` proactively
4. **Hooks require restart** — changes to hook config don't take effect until Claude Code restarts
5. **MCP auth caching** — first tool use may require approval; auto-approve via settings for trusted servers
6. **Screenshot paste** — use Ctrl+V (not Cmd+V on macOS) to paste screenshots
7. **Git integration** — Claude can stage/commit but should never push without explicit user approval
