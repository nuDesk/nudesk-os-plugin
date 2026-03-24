# Vanta API — Platform Reference

**Last updated:** 2026-03-23
**Status:** API access verified (2026-03-23) — "Manage Vanta" app type, OAuth client credentials flow working

---

## Tier Availability

| Feature | Basic | Core | Enterprise |
|---------|-------|------|------------|
| Web UI (controls, tests, evidence, reviews) | Yes | Yes | Yes |
| REST API (`https://api.vanta.com`) | Verify | Yes | Yes |
| MCP Server (public preview, read-only) | No | Yes | Yes |
| Custom integrations (webhooks, SCIM) | No | No | Yes |

> **nuDesk is on the Basic plan.** Check Developer Console to verify API access — some Basic plans have API enabled.

---

## Authentication

- **OAuth 2.0 Client Credentials** flow
- Token endpoint: `POST https://api.vanta.com/oauth/token`
- Grant type: `client_credentials`
- Scopes: `vanta-api.all:read`, `vanta-api.all:write` (request both, server grants what your app is authorized for)
- **Single active token per application** — issuing a new token immediately revokes the previous one (in-flight requests with old token will get 401)
- Token expiry: **1 hour** — tokens cannot be refreshed; request a new one when expired
- Store client secret in `~/.env` as `VANTA_CLIENT_SECRET`

### Token Request

```bash
curl -X POST https://api.vanta.com/oauth/token \
  -H "Content-Type: application/json" \
  -d '{
    "client_id": "<CLIENT_ID>",
    "client_secret": "<CLIENT_SECRET>",
    "grant_type": "client_credentials",
    "scope": "vanta-api.all:read vanta-api.all:write"
  }'
```

---

## Rate Limits

| Endpoint Type | Limit |
|---------------|-------|
| Management API | 50 requests / minute |
| Auth endpoints | 5 requests / minute |

Rate limit headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`

---

## REST API Capabilities

Base URL: `https://api.vanta.com/v1`

### Read Operations

| Endpoint | Description | Use Case |
|----------|-------------|----------|
| `GET /controls` | List all controls with pass/fail status | `/compliance-status` dashboard |
| `GET /controls/{id}` | Single control detail | Deep-dive on specific control |
| `GET /tests` | Automated test results | Evidence freshness checks |
| `GET /vulnerabilities` | Open vulnerabilities | `/compliance-status` vuln section |
| `GET /evidence` | Evidence documents inventory | Gap analysis |
| `GET /people` | Personnel with compliance status | Access review completion |
| `GET /frameworks` | Framework mapping (SOC 2, etc.) | Report generation |

### Write Operations

| Endpoint | Description | Use Case |
|----------|-------------|----------|
| `POST /evidence` | Upload evidence document | `/evidence-collect` → Vanta |
| `POST /tasks` | Create security task | Asana → Vanta sync |
| `PUT /tasks/{id}` | Update task status | Bridge sync updates |
| `POST /controls/{id}/evidence` | Link evidence to control | Evidence mapping |

### Task Sync (Asana → Vanta)

The `security_task` resource supports pushing Asana Change Log entries to Vanta:

```json
{
  "taskId": "asana-task-gid",
  "title": "Deploy Portal v2.3.1",
  "state": "COMPLETED",
  "priority": "MEDIUM",
  "assignees": ["sean@nudesk.ai"],
  "externalUrl": "https://app.asana.com/0/project-gid/task-gid"
}
```

### Evidence Upload Workflow

1. `GET /evidence` — list existing evidence docs
2. `POST /evidence` — upload new document (multipart/form-data)
3. `POST /controls/{id}/evidence` — link document to specific control
4. Evidence appears in Vanta UI for auditor review

---

## MCP Server (Core+ Plans Only)

- **Package:** Official Vanta MCP server (public preview)
- **Capabilities:** Read-only — frameworks, controls, tests, vulnerabilities, evidence, people
- **Install:** `claude mcp add vanta -- npx -y @anthropic/vanta-mcp-server`
- **Auth:** Same OAuth 2.0 client credentials

### Available MCP Tools (Core+)

| Tool | Description |
|------|-------------|
| `vanta_list_frameworks` | List compliance frameworks |
| `vanta_get_controls` | Get controls with status |
| `vanta_get_tests` | Get automated test results |
| `vanta_get_vulnerabilities` | Get open vulnerabilities |
| `vanta_get_evidence` | Get evidence inventory |
| `vanta_get_people` | Get personnel compliance status |

---

## Graceful Degradation Strategy

The Executive OS works at all Vanta tiers:

| Tier | Integration Mode | How It Works |
|------|-----------------|--------------|
| **Basic (UI-only)** | Export mode | `/evidence-collect` saves files locally; `/compliance-report` generates docs for manual Vanta upload |
| **Core (API)** | REST sync | `vanta-bridge` skill pushes evidence and tasks via REST API |
| **Core+ (API + MCP)** | Full integration | Real-time queries via MCP + REST writes for sync |

---

## Key Constraints

1. **Single active token.** Never run concurrent API sessions — one token revokes the previous
2. **Read-heavy, write-light.** Batch evidence uploads; don't call write endpoints in tight loops
3. **Rate limit awareness.** 50 req/min is generous for batch operations but tight for real-time dashboards — cache responses
4. **MCP is read-only.** Any write operations (evidence upload, task sync) must go through REST API
5. **Evidence naming convention.** Use `{date}_{control-id}_{description}.{ext}` for uploaded evidence (e.g., `2026-03-23_OS-01_change-log-audit.pdf`)
6. **OAuth scope changes require new app.** If you need different scopes, create a new application in Developer Console

---

## Verification Checklist (Phase 1.1)

- [ ] Navigate to Vanta Settings → Developer Console
- [ ] Can you create an Application? → Records API access status
- [ ] If yes: create OAuth app, record client ID, store secret in `~/.env`
- [ ] Smoke test: request token, call `GET /controls`
- [ ] Record plan tier and API access in `compliance-config.md`
- [ ] If no Developer Console: record "UI-only" in config, proceed with export mode
