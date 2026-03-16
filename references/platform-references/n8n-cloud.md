# n8n Cloud — Platform Reference

Last updated: 2026-03-04

## Hard Constraints

| Constraint | Limit | Notes |
|-----------|-------|-------|
| **Code node execution** | **60 seconds** | Hard limit on n8n Cloud. NOT configurable by users. Task runner timeout — applies to both v1.x and v2.x on Cloud. Self-hosted can increase via `N8N_RUNNERS_TASK_TIMEOUT`. |
| **Gemini output tokens** | 8,192 tokens | Gemini 1.5 Flash default. Exceeding crashes LangChain parser. Design tool calls to pass minimal data. |
| **Sandbox — no `fetch()`** | Blocked | `fetch()` is NOT available in Code node sandbox on n8n Cloud. n8n 2.x makes it worse (tighter task runners). |
| **Sandbox — no `require()`** | Blocked | Cannot import Node.js modules in Code nodes. |

## Verified Patterns

### HTTP calls from Code nodes
Use `this.helpers.httpRequest()` — the n8n built-in HTTP helper that works in the Cloud sandbox.

```javascript
const result = await this.helpers.httpRequest({
  method: 'POST',
  url: 'https://your-api.com/endpoint',
  body: payload,
  json: true,
  timeout: 55000  // Stay under the 60s Code node limit
});
```

Do NOT use `fetch()`, `require('http')`, or `$http`. They are not available.

### Code Tool (toolCode) input
AI input comes through the `query` variable, not top-level schema variables.

```javascript
const args = typeof query === 'string' ? JSON.parse(query) : query;
```

### Workflow static data
`$getWorkflowStaticData('global')` persists across chat turns within the same session. Useful for storing data on upload turn and reading it on tool-call turn.

Note: Each chat message triggers a new workflow execution. Upstream nodes (CSV parse, normalize) only run on their trigger turn, not on subsequent chat turns.

### LangChain / Gemini crash prevention
n8n 1.x bundles `@langchain/google-genai@2.0.0` which has a known `parts.reduce()` crash on undefined `candidateContent.parts`.

**Required settings:**
- **Gemini model node: Pin `modelName` explicitly** — n8n's default can change server-side. Use `models/gemini-2.0-flash` (Gemini 1.5 models are retired/404). Never leave modelName unset.
- Agent node: `enableStreaming: false` — routes through `executor.invoke()` instead of the crash-prone `executor.streamEvents()` path
- Gemini model node: Set all 4 safety categories to `BLOCK_NONE` — prevents empty content responses that trigger the parser crash:
  - `HARM_CATEGORY_HARASSMENT`
  - `HARM_CATEGORY_HATE_SPEECH`
  - `HARM_CATEGORY_SEXUALLY_EXPLICIT`
  - `HARM_CATEGORY_DANGEROUS_CONTENT`

**Why this matters:** When `modelName` is not explicitly set, n8n uses the node definition's default, which updates server-side. A previously working workflow will silently break when the default model changes. Always pin model versions.

### Numeric data handling
Never convert CSV string values to numbers in Normalize Data nodes. `"0.00%"` becoming numeric `0` creates many identical zero values that trigger the LangChain parser crash.

Keep all values as strings — the backend should handle parsing.

## Anti-Patterns (Don't Do These)

| Anti-Pattern | Why It Fails | Do This Instead |
|-------------|-------------|-----------------|
| `fetch()` in Code nodes | Not available in Cloud sandbox | `this.helpers.httpRequest()` |
| HTTP Request Tool with array body params | Stringifies arrays as `"[object Object]"` → 422 errors | Code Tool with `this.helpers.httpRequest()` |
| Passing large data through AI tool calls | Hits Gemini output token limit (8,192) | Store in `$getWorkflowStaticData`, read in Code Tool |
| Backend API calls > 55 seconds | Code node killed at 60s | Optimize backend (batch operations, parallel I/O) |
| Converting CSV values to numbers | Triggers LangChain crash with zero values | Keep as strings; backend parses |
| Setting workflow `executionTimeout` to fix Code node timeout | They're different timeouts | Reduce actual Code node execution time |
| Leaving Gemini `modelName` unset | Default changes server-side — breaks parser | Always pin model explicitly |
| Updating Code node via API and expecting immediate effect | n8n Cloud caches active workflows in memory. API/MCP updates save to DB but the running webhook listener keeps the old code. | After updating Code node via API/MCP, **manually toggle the workflow active switch off/on** to force a reload. |

## n8n MCP Tools

The n8n MCP (`n8n-mcp`) provides workflow management. Key tools:

| Tool | Use For |
|------|---------|
| `n8n_get_workflow` | Read workflow (modes: full, structure, details, minimal) |
| `n8n_update_partial_workflow` | Incremental updates (updateNode, updateSettings, addConnection) |
| `get_node` | Node schema, properties, version info |
| `validate_node` | Check node configuration |
| `n8n_executions` | View execution history and errors |
| `tools_documentation` | MCP tool reference docs |

### MCP gotchas
- `n8n_update_partial_workflow` connections use `source`/`target` syntax (not `from`/`to`)
- Node updates require `nodeId` (UUID), not node `name`
- `updateSettings` is the operation type for workflow-level settings (executionTimeout, etc.)

## Version Considerations

| Topic | Detail |
|-------|--------|
| v1.x → v2.x upgrade | Breaking changes: tighter sandbox, Save/Publish separation, env var blocking |
| `$getWorkflowStaticData` | May not work in task runner sandbox (v2.x moves Code nodes to isolated task runners) |
| Save/Publish separation | v2.x separates saving from publishing — active workflows require explicit publish step |
| Environment variable blocking | v2.x blocks `process.env` access by default — Code nodes reading env vars will break silently |

**Before any upgrade:** Run Migration Report (Settings > Migration Report) and verify `$getWorkflowStaticData` works in the new sandbox.

## Community Resources

- n8n Community Forum: community.n8n.io (search here FIRST for platform issues)
- Code node timeout threads: search "task request timed out 60 seconds"
