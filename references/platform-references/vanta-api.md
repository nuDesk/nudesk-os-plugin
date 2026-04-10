# Vanta API — Platform Reference

**Last updated:** 2026-04-10
**Status:** API access verified (2026-03-23) — "Manage Vanta" app type, OAuth client credentials flow working

---

## Tier & Plan

**nuDesk is on the Core+ plan.** Full REST API access and MCP server eligibility confirmed.

| Feature | Basic | Core | Core+ / Enterprise |
|---------|-------|------|--------------------|
| Web UI (controls, tests, evidence, reviews) | Yes | Yes | Yes |
| REST API (`https://api.vanta.com`) | No | Yes | Yes |
| MCP Server (public preview, read-only) | No | No | Yes |
| Custom integrations (webhooks, SCIM) | No | No | Enterprise only |

---

## Authentication

- **OAuth 2.0 Client Credentials** flow
- Token endpoint: `POST https://api.vanta.com/oauth/token`
- Grant type: `client_credentials`
- Scopes: `vanta-api.all:read`, `vanta-api.all:write` (request both, server grants what your app is authorized for)
- **Single active token per application** — issuing a new token immediately revokes the previous one (in-flight requests with old token will get 401)
- Token expiry: **1 hour** — tokens cannot be refreshed; request a new one when expired
- Store credentials in `~/.env` as `VANTA_CLIENT_ID` and `VANTA_CLIENT_SECRET`

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
| OAuth / Auth endpoints | 5 requests / minute |
| Integration API | 20 requests / minute |
| Management API | 50 requests / minute |

Rate limit headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`

---

## REST API — Verified Endpoints

Base URL: `https://api.vanta.com/v1`

### Read Operations

| Endpoint | Description | Use Case |
|----------|-------------|----------|
| `GET /controls` | List all controls with pass/fail status | `/compliance-status` dashboard |
| `GET /controls/{id}` | Single control detail | Deep-dive on specific control |
| `GET /tests` | Automated test results with pass/fail | Identify failing tests |
| `GET /tests/{testId}/entities` | Individual entities within a test | Find which resources are failing |
| `GET /vulnerabilities` | Open vulnerabilities | `/compliance-status` vuln section |
| `GET /documents` | Evidence documents inventory | Gap analysis, staleness check |
| `GET /people` | Personnel with compliance status | Access review completion |
| `GET /frameworks` | Framework mapping (SOC 2, etc.) | Report generation |
| `GET /vendors` | Vendor inventory | Third-party risk management |

### Write Operations (Verified)

| Endpoint | Method | Description | Use Case |
|----------|--------|-------------|----------|
| `POST /tests/{testId}/entities/{entityId}/deactivate` | POST | Suppress a test entity (mark as not applicable) | Decommissioned resources still tracked by Vanta |
| `PATCH /vendors/{vendorId}` | PATCH | Update vendor risk metadata | Set risk level, auth method, data access details |
| `POST /documents/{documentId}/uploads` | POST | Create upload URL for evidence | Step 2 of 3-step evidence upload |
| `POST /documents/{documentId}/submit` | POST | Submit uploaded evidence for review | Step 3 of 3-step evidence upload |

### What the API CANNOT Do

- **Mark tests as passing.** Vanta runs its own automated scans against live infrastructure. You cannot override a test result — you must fix the underlying infrastructure issue and wait for Vanta to re-scan.
- **Create new tests or controls.** These are defined by Vanta's framework mappings.
- **Delete evidence.** Documents can be uploaded and submitted but not removed via API.

---

## Evidence Upload Workflow (3-Step)

Evidence upload is a 3-step process — not a single POST:

1. **`GET /documents`** — List existing evidence documents to find the target document ID
2. **`POST /documents/{documentId}/uploads`** — Request a pre-signed upload URL. Returns `{ uploadUrl, fields }` for the actual file upload
3. **`POST /documents/{documentId}/submit`** — After uploading the file to the pre-signed URL, submit the document for auditor review

**Evidence naming convention:** `{date}_{control-id}_{description}.{ext}` (e.g., `2026-04-10_OS-01_change-log-audit.pdf`)

---

## Vendor Risk Management

`PATCH /vendors/{vendorId}` supports these fields:

| Field | Type | Example |
|-------|------|---------|
| `inherentRiskLevel` | enum | `LOW`, `MEDIUM`, `HIGH`, `CRITICAL` |
| `authDetails.method` | enum | `SSO`, `MFA`, `PASSWORD`, `API_KEY` |
| `dataAccess.hasCustomerData` | boolean | `true` |
| `dataAccess.dataTypes` | array | `["PII", "FINANCIAL"]` |
| `owner` | string | Email of vendor owner |

---

## Test Entity Management

When a test fails because it tracks a decommissioned or irrelevant resource:

```
POST /v1/tests/{testId}/entities/{entityId}/deactivate
```

This suppresses the entity so it no longer contributes to test failures. Use for resources that have been legitimately decommissioned (e.g., shut-down CloudSQL instances, deleted VMs).

**Important:** Deactivating an entity does NOT fix the test — it removes one failing item. If all entities in a test are deactivated or passing, the test passes.

---

## MCP Server (Core+ Plans)

- **Package:** Official Vanta MCP server (public preview)
- **Capabilities:** Read-only — frameworks, controls, tests, vulnerabilities, evidence, people
- **Install:** `claude mcp add vanta -- npx -y @anthropic/vanta-mcp-server`
- **Auth:** Same OAuth 2.0 client credentials

### Available MCP Tools

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
3. **Rate limit awareness.** 20 req/min for integration endpoints is the practical limit for automation — cache responses aggressively
4. **MCP is read-only.** Any write operations (evidence upload, vendor updates, entity deactivation) must go through REST API
5. **Tests cannot be force-passed.** Fix the infrastructure; Vanta re-scans automatically
6. **Evidence naming convention.** Use `{date}_{control-id}_{description}.{ext}` for uploaded evidence
7. **OAuth scope changes require new app.** If you need different scopes, create a new application in Developer Console
8. **Evidence upload is 3-step.** GET document → POST upload URL → POST submit. Skipping step 3 leaves evidence in draft state.
