---
name: soc2-compliance
description: >
  Reference document: SOC 2 Type II compliance controls for Claude Code workflows.
  Embedded in security-check, session-closeout, and deploy commands. Auto-triggered
  when working with credentials, PII, or production systems.
version: 1.0.0
---

# SOC 2 Compliance Controls

> This skill is a reference document. It is not invoked directly.
> It codifies nuDesk's SOC 2 compliance controls for Claude Code sessions.
> Other commands (security-check, session-closeout, deploy) reference this as their
> single source of truth for compliance rules.

## Credential Controls

- **Never store or expose PII on the local drive.** For testing workflows that require realistic data, create synthetic versions that simulate PII structure without containing real information.
- **Never hardcode API keys or credentials** anywhere in code. Use `.env` files for local development and Secret Manager for production.
- **`.env` files are blocked by hook.** The PreToolUse hook prevents Claude from editing `.env` files directly. Modify these manually. (See `templates/hooks-settings.json.template` for the hook definition.)
- **Never expose secrets in the chat window.** API keys, tokens, passwords, and credentials must never appear in conversation. Reference by variable name only.
- **Credentials via Secret Manager only** in production. No environment variable injection at deploy time — use GCP Secret Manager, AWS Secrets Manager, or equivalent.

## Access Controls

- **Service accounts for production.** All production services authenticate via service accounts with minimum required scopes.
- **OAuth for user-context.** When acting on behalf of a user (Gmail, Calendar, Drive), use OAuth with user consent. Never share OAuth tokens between users.
- **Principle of least privilege.** Request only the API scopes needed for the specific operation. Do not request broad scopes "just in case."
- **MCP server credentials are per-user.** Each team member configures their own OAuth tokens. Plugin templates provide the structure; users supply their own credentials.

## Audit Trail

- **All production changes logged.** Every deployment, configuration change, and data modification must be traceable:
  - Git commits with descriptive messages (conventional commit format)
  - Asana tasks for work tracking
  - Deployment records (Cloud Run revision history)
- **Session closeout captures work.** The `/session-closeout` command ensures tasks, decisions, and learnings are captured in Asana and memory.
- **Weekly reports provide accountability.** The `/weekly-report` command creates a weekly audit trail of completed work, posted to Asana.

## Data Handling

- **Synthetic data for testing.** When testing workflows that involve PII, generate synthetic data that matches the structure and format of real data without containing actual personal information.
- **PII only from secure environments.** Real PII may be accessed from GitHub private repos or Google Drive with appropriate permissions — but never pulled into local testing environments.
- **No PII in logs or error messages.** Ensure logging frameworks redact sensitive fields. Error responses should never include PII or internal system details.

## Incident Response

- **Credential exposure protocol:** See `~/Projects/system_docs/config/SECURITY_REVIEW_GUIDE.md` for the full incident response procedure.
- **Severity levels:**
  - CRITICAL: Exposed in public repo, cloud logs, or public system — rotate within 24h
  - HIGH: Exposed in session logs with external access — rotate within 1 week
  - MEDIUM: Exposed only in local terminal on secure machine — rotate when convenient
  - LOW: Non-secret identifiers (Sheet IDs, service account emails) — document only

## Compliance Checklists

### Pre-Commit

- [ ] No hardcoded credentials in staged files
- [ ] `.env` files are not staged
- [ ] No PII in test fixtures or sample data
- [ ] Credential files are gitignored

### Pre-Deploy

- [ ] All secrets sourced from Secret Manager (not environment variables)
- [ ] Service account has minimum required permissions
- [ ] No debug/verbose logging enabled in production config
- [ ] CORS settings are appropriately restrictive
- [ ] Rate limiting is configured on public endpoints

### Session Closeout

- [ ] No credentials were exposed during the session
- [ ] Work is tracked in Asana (tasks created or updated)
- [ ] Any new security findings are documented
- [ ] CLAUDE.md or project docs updated if new compliance patterns discovered

## Third-Party Review

- **Review all third-party MCPs and open-source tools** for security compliance before production use. nuDesk is SOC 2 certified — unvetted dependencies are a compliance risk.
- **Check for known CVEs** in pinned dependency versions before upgrading or adding packages.
- **Prefer well-maintained packages** with active security disclosure programs.
