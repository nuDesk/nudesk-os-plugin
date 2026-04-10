# Branch Protection & PR Workflow — nuDesk

> Effective: 2026-04-10 | SOC 2 Control: SD-02 | Applies to: All nuDesk GitHub repos

## Why

SOC 2 requires every production change to go through a documented review process.
Branch protection enforces this by requiring a pull request before code reaches `main`.
Every PR creates an audit trail — who changed what, when, and why.

## The 30-Second Version

**Before (old way):**
```
git add . && git commit -m "feat: thing" && git push origin main
```

**After (new way):**
```
git checkout -b feat/thing
git add . && git commit -m "feat: thing"
git push -u origin feat/thing
gh pr create --title "feat: thing" --body "Description here"
gh pr merge --squash --delete-branch
```

**Or just ask Claude:**
> "Commit these changes and create a PR"

Claude Code handles branching, pushing, and PR creation automatically.

## Workflow by Scenario

### Scenario 1: Quick fix (5 min)
```bash
git checkout -b fix/typo-in-readme
# make your fix
git add README.md
git commit -m "fix: correct typo in README"
git push -u origin fix/typo-in-readme
gh pr create --fill    # auto-fills title+body from commit
gh pr merge --squash --delete-branch
```

### Scenario 2: Feature work (multi-commit)
```bash
git checkout -b feat/new-skill
# work, commit, work, commit...
git push -u origin feat/new-skill
gh pr create --title "feat: add meeting-prep skill" --body "Adds the meeting-prep skill with..."
# Review the PR yourself or wait for the other person
gh pr merge --squash --delete-branch
```

### Scenario 3: Using Claude Code (easiest)
Just tell Claude what you want. Examples:
- "Make this change and create a PR"
- "Commit everything and open a PR with a good description"
- "/commit" — Claude's built-in commit skill (creates PR if on protected branch)

### Scenario 4: Overnight agent queue
The agent creates a feature branch and opens a PR automatically.
Sean reviews and merges the next morning.

### Scenario 5: Emergency (bypass)
Admins can push directly to main. Every bypass is logged in GitHub's audit log.
Use sparingly — the audit trail is permanent.
```bash
git push origin main  # works for admins, logged as bypass
```

## Branch Naming

| Prefix | Use for | Example |
|--------|---------|---------|
| `feat/` | New features or skills | `feat/meeting-prep` |
| `fix/` | Bug fixes | `fix/version-sync` |
| `chore/` | Maintenance, cleanup | `chore/update-deps` |
| `docs/` | Documentation only | `docs/add-brand-guide` |
| `agent-queue/` | Overnight agent work | `agent-queue/2026-04-10` |

## After Merging

For the nudesk-os-plugin repo specifically:
```bash
claude plugin update nudesk-os@marketplace
```

For other repos — no extra step needed. The merge to main IS the deploy trigger
(or deploy manually as before).

## FAQ

**Q: Can I self-merge my own PR?**
A: Yes. Zero approvals required. The PR exists for the audit trail, not as a blocker.

**Q: What if the other person is asleep/unavailable?**
A: Self-merge. The PR + your name on it is the audit evidence.

**Q: Do I need to do this for every tiny change?**
A: Yes — but it's ~30 seconds extra. Claude Code makes it even faster.

**Q: What about repos I rarely touch?**
A: Same rule applies. The org ruleset covers all repos automatically.

**Q: Can I still use `git commit` locally without a branch?**
A: Yes — commits are local. The protection only kicks in when you push to `main`.
Just create the branch before pushing.
