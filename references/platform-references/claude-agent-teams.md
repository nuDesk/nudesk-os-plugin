# Claude Agent Teams — Platform Reference

> Constraint doc for Claude Code Agent Teams — coordinated multi-agent parallel execution within Claude Code sessions.

## Prerequisites

- **Feature flag:** `enableAgentTeams: true` in `~/.claude/settings.json`
- **Claude Code version:** 2.1.32 or later (current: 2.1.76)
- **Agent definitions:** At least one agent defined in `~/.claude/agents/` or plugin `agents/` directory

## Display Modes

| Mode | Setup | Navigation | Our Standard |
|------|-------|------------|--------------|
| **In-process** (default) | No extra setup | Shift+Down to cycle teammates | **Yes** |
| **tmux split-pane** | Requires tmux installed + `teammateMode: "tmux"` | Each teammate in its own pane | No — available but not part of standard setup |

In-process mode works in any terminal. All teammates share the same terminal window; use **Shift+Down** to cycle through teammate views.

## Core Concepts

- **Lead session:** Your main Claude Code session that creates and directs the team
- **Teammates:** Independent Claude Code sessions spawned by the lead, each with their own context
- **Shared task list:** All teammates see and update the same task list — this is the coordination mechanism
- **Inter-agent messaging:** Teammates can message each other directly via `SendMessage({to: "teammate-name"})`, not just report to the lead

## How It Works

1. Lead creates a team via `TeamCreate` with teammate definitions
2. Each teammate gets a name, model, and initial prompt
3. Teammates work independently, updating shared tasks as they progress
4. Lead monitors progress and can send messages to redirect work
5. When all tasks complete, lead synthesizes results

## Constraints

| Constraint | Detail |
|------------|--------|
| **One team per session** | Cannot create nested teams or teams within teams |
| **No session resumption** | In-process teams cannot be resumed after session ends — work must complete in one session |
| **No `skills`/`mcpServers` frontmatter** | Teammates do NOT inherit skills or MCP server access from agent definitions. Only `tools` allowlists are applied. |
| **Teammate context** | Each teammate starts fresh — they don't share the lead's conversation history |
| **File conflicts** | Two teammates editing the same file WILL overwrite each other — assign clear file ownership |
| **Cost scaling** | Each teammate consumes tokens independently — cost scales linearly with team size |
| **Task list is the contract** | Teammates coordinate via tasks, not shared memory or files |

## Hook Events

Agent Teams introduces three hook events in `settings.json`:

| Event | Fires When | Can Block? | Use Case |
|-------|-----------|------------|----------|
| **TeammateIdle** | A teammate has no more tasks to work on | Yes (exit 2) | Reassign work, prevent idle billing |
| **TaskCreated** | Any teammate creates a new task | Yes (exit 2) | Validate task scope, enforce naming conventions |
| **TaskCompleted** | A teammate marks a task as done | Yes (exit 2) | Quality gate — reject incomplete work |

Hook configuration follows the same `settings.json` pattern as `PreToolUse`/`PostToolUse`:

```json
{
  "hooks": {
    "TeammateIdle": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "your-command-here"
          }
        ]
      }
    ]
  }
}
```

## Best Practices

### Team Composition
- **3–5 teammates max** — diminishing returns beyond that
- **5–6 tasks per teammate** — enough to stay busy, not so many they lose focus
- **Sonnet for teammates** — cost-effective for execution work; reserve Opus for the lead
- **Clear file ownership** — assign each teammate specific files/directories to prevent conflicts

### When to Use Teams vs Subagents

| Use Case | Tool |
|----------|------|
| Quick research lookup | Subagent (`Agent` tool) |
| Single-file edit | Subagent |
| Multi-module feature (frontend + backend + tests) | Agent Team |
| Parallel investigation (competing hypotheses) | Agent Team |
| Full PR workflow (implement + test + review) | Agent Team |
| Security audit across multiple layers | Agent Team |

### Task Design
- Write clear, self-contained task descriptions — teammates don't share your context
- Include file paths and expected outputs in task descriptions
- Use task dependencies to enforce ordering when needed
- Keep tasks atomic — one deliverable per task

### Safety
- Start with **read-only tasks** (research, review, audit) before using teams for code generation
- Always confirm team structure with the user before spawning — teams consume significant tokens
- Monitor teammate progress via the shared task list
- Use `TeamDelete` to clean up if a team goes off track

## Known Limitations

1. **No MCP access for teammates** — teammates cannot use MCP servers (Asana, HubSpot, Figma, etc.). Design tasks so the lead handles MCP calls.
2. **No skill inheritance** — teammates don't get skills from agent definitions or plugins. They have base Claude Code tools only (plus any `tools` allowlist).
3. **Session-bound** — in-process teams end when the session ends. No resume capability.
4. **No nested teams** — a teammate cannot spawn its own team.
5. **Worktree isolation optional** — teammates can use `isolation: "worktree"` for git-safe parallel work, but this adds setup overhead.

## Integration with nuDesk OS

- **Agent definitions** (`~/.claude/agents/` and `nudesk-os-plugin/agents/`) define available teammate roles
- **`/os-setup`** Step 5c verifies Agent Teams prerequisites (feature flag, version, agents)
- **`/os-audit`** Section 6 checks Agent Teams health
- Teams are most valuable for multi-layer work: one teammate on frontend, one on backend, one writing tests, one reviewing
