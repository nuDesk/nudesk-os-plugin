# Security Review Best Practices

**Purpose:** Guide for conducting security reviews without exposing sensitive credentials.

## What to Check

### 1. Credential Files
- Service account keys (`.json`, `.pem`)
- OAuth credentials
- API keys and tokens
- Environment files (`.env`, `.env.local`)
- Token/session files (`.pickle`, `.token`)

### 2. Hardcoded Secrets
- API keys in code
- Passwords in configuration
- Database connection strings
- Private keys embedded in code

### 3. Git Repository Status
- Verify credentials are gitignored
- Check no secrets are tracked by git
- Verify no secrets in git history

### 4. Public Identifiers (Low Risk)
- Google Sheet IDs (not secrets, but environment-specific)
- Service account emails (public, but environment-specific)
- Project IDs (public)

## Safe Commands

### Check if Credential Files Exist (Without Reading Them)

```bash
# Check if files exist (returns exit code, doesn't print content)
test -f credentials/service-account-key.json && echo "EXISTS" || echo "NOT FOUND"

# List credential filenames only (not contents)
ls -1 credentials/ 2>/dev/null

# Count credential files
ls credentials/*.json 2>/dev/null | wc -l
```

### Verify Files Are Gitignored

```bash
# Check if file is ignored by git (doesn't read content)
git check-ignore -v credentials/service-account-key.json

# Check if file is tracked by git
git ls-files credentials/ | wc -l  # Should return 0

# Verify directory is in .gitignore
grep "credentials/" .gitignore
```

### Search for Credential References (Not Values)

```bash
# Search for variable names, not actual secrets
grep -r "API_KEY\|SECRET\|PASSWORD" --include="*.py" | grep -v "credentials/"

# Search for import statements
grep -r "from.*credentials import\|load.*credentials" --include="*.py"

# Search for environment variable usage
grep -r "os.getenv\|os.environ" --include="*.py"
```

### Check Git History (Safe)

```bash
# Check if credentials directory was ever committed
git log --all --oneline -- credentials/

# Check if specific patterns appear in history (file names only)
git log --all --oneline --name-only | grep -i "key\|secret\|credential"
```

## Dangerous Commands to Avoid

### NEVER Read Credential Files Directly

```bash
# DON'T - Exposes secrets in terminal
cat credentials/service-account-key.json
grep -r "private_key\|client_secret" credentials/
grep -r "BEGIN PRIVATE KEY" .
jq . credentials/oauth_credentials.json
```

### NEVER Use Wildcards That Might Match Credentials

```bash
# DON'T - Might read credential files
grep -r "project_id" *.json
find . -name "*.json" -exec cat {} \;
```

### NEVER Print Environment Variables

```bash
# DON'T - Exposes secrets in env vars
env | grep -i "key\|secret\|password"
printenv
```

## Security Review Checklist

### Pre-Review Setup
- [ ] Understand what credentials the project uses
- [ ] Know where credentials should be located
- [ ] Have .gitignore patterns documented
- [ ] Prepare safe commands in advance

### Repository Security Check
- [ ] Verify .gitignore exists and includes credential patterns
- [ ] Confirm credential files are NOT tracked by git
- [ ] Check git history for accidentally committed secrets
- [ ] Verify credential files exist locally (without reading them)

### Code Security Check
- [ ] Search for hardcoded API keys (exclude credential directories)
- [ ] Search for hardcoded passwords
- [ ] Verify environment variables are used for secrets
- [ ] Check for TODO/FIXME related to security

### Configuration Security Check
- [ ] Check for .env.example (template without secrets)
- [ ] Verify workflow configs don't embed credentials
- [ ] Check Docker/deployment configs for secret management

### Documentation Security Check
- [ ] Verify documentation doesn't include actual secrets
- [ ] Check for example credentials in docs

### Post-Review Actions
- [ ] Document any hardcoded identifiers that need updating
- [ ] Create list of credentials that need rotation during handoff
- [ ] Update .gitignore if new credential patterns found
- [ ] Add security notes to CLAUDE.md or project README

## Incident Response

### If Credentials Are Exposed in Terminal/Logs

1. **Assess Exposure Level:**
   - Local terminal only? (Low risk)
   - Committed to git? (High risk)
   - Uploaded to cloud/public? (Critical risk)

2. **Local Terminal Exposure:**
   ```bash
   # Clear terminal scrollback
   clear && printf '\033[2J\033[3J\033[1;1H'
   # Close terminal to clear history
   exit
   ```

3. **Git Repository Exposure:**
   ```bash
   # If credentials were committed, rotate immediately
   # Then remove from git history
   git filter-branch --force --index-filter \
     "git rm --cached --ignore-unmatch credentials/secret-file.json" \
     --prune-empty --tag-name-filter cat -- --all
   # Force push to remove from remote
   git push origin --force --all
   ```

4. **Credential Rotation Priority:**
   - **CRITICAL (Rotate within 24h):** Exposed in public repo, cloud logs, or public system
   - **HIGH (Rotate within 1 week):** Exposed in session logs with external access
   - **MEDIUM (Rotate when convenient):** Exposed only in local terminal on secure machine
   - **LOW (Document only):** Non-secret identifiers (Sheet IDs, service account emails)

## Quick Reference: Safe Security Check

```bash
# 1. Verify .gitignore
cat .gitignore | grep -E "credentials/|\.env|\.pickle"

# 2. Check nothing tracked in credentials/
git ls-files credentials/ | wc -l  # Should output: 0

# 3. Verify credential files exist locally
ls -1 credentials/ 2>/dev/null

# 4. Check for hardcoded secrets in code (safe search)
grep -r "api.*key.*=.*['\"]" --include="*.py" src/ | grep -v "os.getenv" | head -10

# 5. Verify environment variable usage
grep -r "os.getenv\|os.environ" --include="*.py" | grep -i "key\|secret" | head -10

# Done! No credentials exposed.
```
