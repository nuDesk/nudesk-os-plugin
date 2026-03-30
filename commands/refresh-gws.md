---
description: Refresh Google Workspace CLI and skills to the latest version
allowed-tools: Bash, Read, Glob
---

Update the Google Workspace CLI binary and refresh all GWS Agent Skills from the upstream repository.

## Step 1: Check Current State

Run these in parallel:

1. Get the currently installed CLI version:
   ```bash
   gws --version 2>/dev/null || echo "gws CLI not installed"
   ```

2. Count currently installed GWS skills:
   ```bash
   ls -d ~/.claude/skills/gws-* 2>/dev/null | wc -l
   ```

3. Check the latest available version from npm:
   ```bash
   npm view @googleworkspace/cli version 2>/dev/null || echo "Unable to check latest version"
   ```

Report the current version, latest available version, and number of installed skills.

## Step 2: Update CLI

Install or update the GWS CLI globally:

```bash
npm install -g @googleworkspace/cli
```

Confirm the new version after install:

```bash
gws --version
```

## Step 3: Refresh All Skills

Re-install all skills from the upstream repository. This overwrites existing skill files with the latest versions:

```bash
npx skills add --yes --global https://github.com/googleworkspace/cli
```

After the install completes, count the refreshed skills:

```bash
ls -d ~/.claude/skills/gws-* 2>/dev/null | wc -l
```

## Step 4: Verify Authentication

Check that auth credentials are still valid after the update:

```bash
gws auth status
```

If auth fails (encryption_valid: false, or token_valid: false), inform the user and suggest:

```bash
gws auth login --services gmail,calendar,drive,sheets,docs,slides,chat,people
```

Notes:
- Use `--services` to limit OAuth scopes to services actually in use. Requesting all scopes causes `invalid_scope` errors if the OAuth consent screen doesn't have them approved.
- If the OAuth client itself needs reconfiguring, use `gws auth setup` instead (NOT `gws auth login`)
- On macOS: v0.22.3+ uses native Keychain. Old `.encryption_key` fallback files from earlier versions are auto-removed.

## Step 5: Summary

Report a table with:

| Item | Before | After |
|------|--------|-------|
| CLI version | (from Step 1) | (from Step 2) |
| Skills installed | (from Step 1) | (from Step 3) |
| Auth status | -- | (from Step 4) |

If any new skills were added (count increased), list the new skill names.
If any skills were removed (count decreased), list the removed skill names.
