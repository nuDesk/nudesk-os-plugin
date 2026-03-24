---
name: soc2-compliance
description: >
  Active SOC 2 compliance enforcement skill. Queries Asana (Production Change Log,
  Incident Response, Risk Register) and Vanta (when API available) to assess control
  status against nuDesk's 91 discrete controls across 16 policies.
  Callable by other commands or directly by users.
user_invocable: true
version: 2.0.0
---

# SOC 2 Compliance — Active Enforcement

> This skill is both a reference and an active query engine. It queries Asana and Vanta
> for live compliance data, cross-references against nuDesk's 91-control matrix, and
> identifies gaps with recommended actions.

## How to Use

- **Direct invocation:** User runs the skill for a compliance check
- **Embedded:** Called by `/compliance-status`, `/security-check`, `/session-closeout`, `/evidence-collect`
- **Auto-triggered:** When working with credentials, PII, or production systems

## Data Sources

### Primary: Asana (always available)

Load compliance project GIDs from `~/.claude/memory/compliance-config.md`.

Query these Asana projects via MCP:
1. **Production Change Log** — recent changes, incomplete checklists, missing Control IDs
2. **Incident Response Log** — open incidents by severity, overdue phases
3. **Risk Register** — active risks by level, treatment plan status

### Secondary: Vanta (when API access confirmed)

Check `~/.claude/memory/compliance-config.md` → Vanta → API Access.

If "yes": query Vanta REST API or MCP (if Core+ plan) for:
- Overall control pass/fail percentage
- Failing automated tests
- Evidence freshness (docs older than 90 days)
- Review completion status (access reviews, vendor assessments, policy acknowledgments)

**If Vanta is unavailable or UI-only:** Skip Vanta queries gracefully. Report Asana data only and note: "Vanta data unavailable — operating in export mode."

## 91-Control Reference

The full control table lives in `~/Projects/nudesk-os-plugin/knowledge-base/Policies/_Summary of nuDesk Sec Policies.md` (Tier 1 — Controls Quick Reference).

### Control Categories

| Category | Count | Enforcement |
|----------|-------|-------------|
| **A — Automatable** | 28 | Hooks, scheduled tasks, auto-evidence via Asana |
| **B — Semi-Automated** | 31 | Human-triggered commands with Asana workflow execution |
| **C — Policy-Only** | 32 | Scheduled review reminders + acknowledgment tracking |

### Key Automatable Controls (Category A)

| Control | Statement | Enforcement Mechanism |
|---------|-----------|----------------------|
| OS-01 | Changes tested, reviewed, approved before deploy | Pre-deploy hook + Change Log subtask checklist |
| SD-01 | Code changes require review before merge | PR review + Change Log evidence |
| SD-02 | Mandatory version control | Git hook evidence (commit logs) |
| SD-04 | Code scanned before deployment | `/security-check` + evidence buffer |
| OS-04 | No confidential data in non-prod | PII pattern scan hook |
| AC-13 | Source code access restricted and logged | Git access logs |
| DM-02 | Confidential data encrypted at rest/transit | Config scan evidence |
| CR-01 | AES-256 at rest, TLS in transit | Infrastructure config checks |
| AI-04 | PII anonymized before AI processing | PII scan hook on Write/Edit |

### Key Semi-Automated Controls (Category B)

| Control | Statement | Enforcement Mechanism |
|---------|-----------|----------------------|
| IR-01–05 | Incident response lifecycle | `/incident-log` command |
| RM-01–03 | Risk assessment and register | Risk Register project + scheduled reviews |
| OS-08 | Quarterly vulnerability scans | Scheduled task + evidence collection |
| BC-01 | Annual DR test | Scheduled task + evidence collection |

## Credential Controls

- **Never store or expose PII on the local drive.** Create synthetic test data.
- **Never hardcode API keys or credentials.** Use `.env` files locally, Secret Manager in production.
- **`.env` files are blocked by hook.** Modify manually.
- **Never expose secrets in chat.** Reference by variable name only.
- **Credentials via Secret Manager only** in production.

## Access Controls

- **Service accounts for production.** Minimum required scopes.
- **OAuth for user-context.** Per-user tokens, no sharing.
- **Principle of least privilege.** Only request needed scopes.
- **MCP credentials are per-user.** Templates provide structure; users supply credentials.

## Audit Trail

- **All production changes logged** to Asana Production Change Log with subtask checklist.
- **Session closeout** captures work via `/session-closeout` → evidence buffer.
- **Weekly reports** provide accountability via `/weekly-report`.
- **Evidence buffer** at `~/.claude/memory/context/evidence-buffer.md` — batch-processed by `/evidence-collect`.

## Data Handling

- **Synthetic data for testing.** Match structure without real PII.
- **PII only from secure environments.** Never in local testing.
- **No PII in logs.** Redact sensitive fields in error responses.

## Incident Response

- **Severity levels:** P0 (Critical), P1 (High), P2 (Medium), P3 (Low)
- **P0:** Immediate notification to IT/Engineering leadership; rotate exposed credentials within 24h
- **P1:** Ticket + manager notification; rotate within 1 week
- **P2:** Standard ticket; rotate when convenient
- **P3:** Document only; non-secret identifiers
- **HIPAA:** If PHI involved, follow IR-04 notification procedures (60-day window)
- **Use `/incident-log`** to create properly structured incident tasks

## Compliance Checklists

### Pre-Commit (SD-01, SD-02)
- [ ] No hardcoded credentials in staged files
- [ ] `.env` files are not staged
- [ ] No PII in test fixtures or sample data
- [ ] Credential files are gitignored

### Pre-Deploy (OS-01, SD-04, CR-01)
- [ ] All secrets sourced from Secret Manager
- [ ] Service account has minimum required permissions
- [ ] No debug/verbose logging in production config
- [ ] CORS settings appropriately restrictive
- [ ] Rate limiting configured on public endpoints
- [ ] `/security-check` has been run this session
- [ ] Production Change Log entry created with subtask checklist

### Session Closeout
- [ ] No credentials exposed during session
- [ ] Work tracked in Asana
- [ ] Security findings documented
- [ ] Evidence buffer updated with session commits
- [ ] CLAUDE.md updated if new compliance patterns discovered

## Test Compliance Matrix

| Change Type | Required Tests | Gate | Enforcement |
|-------------|---------------|------|-------------|
| API endpoint | Unit + integration test | Pre-commit | `pr-test-analyzer` |
| Auth/permissions | Security test + integration | Pre-deploy | `security-reviewer` + `pr-test-analyzer` |
| Database schema | Migration + rollback test | Pre-deploy | Manual dry-run |
| Frontend component | Behavioral component test | Pre-commit | `pr-test-analyzer` |
| Credential handling | Security review | Pre-deploy | `security-reviewer` |
| n8n workflow | E2E test with synthetic payload | Pre-publish | Manual |
| Infrastructure | Dry-run + smoke test | Pre-deploy | Local build + health check |

## Test Data Handling

- **Synthetic data for all fixtures.** Use `faker` (Python) or `@faker-js/faker` (JS).
- **No real GIDs/IDs/tokens in committed fixtures.** Use `"gid_test_001"` patterns.
- **No real emails/phones.** Use `@example.com` and `555-0100` range.
- **Test fixtures gitignored if environment-specific.**
- **Tests must not hit production APIs.**
- **Dedicated test credentials in `.env.test` (gitignored).**

## Third-Party Review

- Review all third-party MCPs and open-source tools before production use.
- Check for known CVEs in dependency versions.
- Prefer well-maintained packages with security disclosure programs.
