---
description: Run a security review following the safe credential checklist
allowed-tools: Read, Grep, Glob, Bash
---

Run a read-only security audit of the current project. This command checks for common vulnerabilities without exposing credential contents.

**CRITICAL: Do NOT read credential file contents. Check existence and gitignore status only.**

For the full review guide, see: `~/Projects/system_docs/config/SECURITY_REVIEW_GUIDE.md`

## Safe Commands Only

Use ONLY these patterns — never `cat`, `grep`, or `jq` on credential files:

```bash
# Check if credential files exist (without reading them)
test -f <path> && echo "EXISTS" || echo "NOT FOUND"
ls -1 credentials/ 2>/dev/null

# Verify files are gitignored
git check-ignore -v <path>
git ls-files credentials/ | wc -l  # Should return 0

# Search for credential references (not values)
grep -r "API_KEY\|SECRET\|PASSWORD" --include="*.py" src/ | grep -v "credentials/"
grep -r "os.getenv\|os.environ" --include="*.py"
```

## Security Review Checklist

### 1. Repository Security

- [ ] **Verify .gitignore exists and covers credential patterns**
  ```bash
  cat .gitignore | grep -E "credentials/|\.env|\.pickle|*key*.json"
  ```

- [ ] **Confirm credential files are NOT tracked by git**
  ```bash
  git ls-files credentials/ | wc -l  # Should be 0
  git ls-files | grep -E "\.env|\.pickle|*key*.json"  # Should be empty
  ```

- [ ] **Check git history for accidentally committed secrets**
  ```bash
  git log --all --oneline -- credentials/
  git log --all --oneline | grep -i "secret\|credential\|key" | head -20
  ```

- [ ] **Verify credential files exist locally (without reading them)**
  ```bash
  ls -1 credentials/ 2>/dev/null
  ```

### 2. Code Security

- [ ] **Search for hardcoded API keys (exclude credential directories)**
  ```bash
  grep -r "api.*key.*=.*['\"]" --include="*.py" src/ | grep -v "os.getenv"
  ```

- [ ] **Search for hardcoded passwords**
  ```bash
  grep -r "password.*=.*['\"]" --include="*.py" src/ | grep -v "os.getenv"
  ```

- [ ] **Verify environment variables are used for secrets**
  ```bash
  grep -r "os.getenv.*API_KEY\|os.environ" --include="*.py"
  ```

- [ ] **Check for TODO/FIXME related to security**
  ```bash
  grep -r "TODO.*secret\|FIXME.*credential\|TODO.*security" --include="*.py"
  ```

### 3. Configuration Security

- [ ] **Check for .env.example (template without secrets)**
  ```bash
  test -f .env.example && echo "Template exists" || echo "Consider creating .env.example"
  ```

- [ ] **Check Docker/deployment configs for secret management**
  ```bash
  grep -r "env_file\|secrets:" docker-compose.yml Dockerfile 2>/dev/null
  ```

### 4. Documentation Security

- [ ] **Verify documentation doesn't include actual secrets**
  ```bash
  grep -r "sk-\|GOCSPX-\|BEGIN PRIVATE KEY" docs/ README.md 2>/dev/null
  ```

### 5. Test Coverage for Security-Sensitive Code

- [ ] **Check that modified source files have corresponding test files**
  ```bash
  # For Python projects: check if tests/ mirrors src/ structure
  for f in $(git diff --name-only --diff-filter=AM HEAD~5 -- '*.py' | grep -v test); do
    test_file="tests/test_$(basename $f)"
    test -f "$test_file" && echo "COVERED: $f -> $test_file" || echo "MISSING: $f has no test at $test_file"
  done
  ```

- [ ] **Check that auth/permission code has dedicated tests**
  ```bash
  grep -rl "auth\|permission\|role\|token" --include="*.py" src/ | while read f; do
    test_file="tests/test_$(basename $f)"
    test -f "$test_file" && echo "COVERED: $f" || echo "WARNING: auth-related file $f has no test"
  done
  ```

- [ ] **Verify no real credentials in test fixtures**
  ```bash
  grep -r "sk-\|GOCSPX-\|BEGIN PRIVATE KEY\|password.*=.*['\"][^'\"]*[a-zA-Z0-9]" tests/ fixtures/ test_data/ 2>/dev/null
  ```

## Output Format

Present findings as a markdown table:

| Check | Status | Notes |
|-------|--------|-------|
| .gitignore coverage | PASS/FAIL | Details |
| Credentials not tracked | PASS/FAIL | Details |
| No hardcoded secrets | PASS/FAIL | Details |
| ... | ... | ... |

**Summary:** X passed, Y failed, Z warnings

If any FAIL items are found, recommend specific remediations.
