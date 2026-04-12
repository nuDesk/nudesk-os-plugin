# nuDesk OS Plugin

Claude Code plugin providing an executive operating system — daily planning, task execution, weekly reports, security audits, compliance, and institutional memory.

## Skill Distribution

There are two ways to distribute skills. Choosing the wrong one breaks auto-update for users.

### Plugin-Local Skills (default choice)

Place `SKILL.md` directly at `skills/<name>/SKILL.md`. These are read live from the plugin directory and auto-update when users refresh the plugin. They appear as `nudesk-os:<name>`.

**Use this for any skill that is markdown-only (no images, no binary assets).**

Current: asana-agent, evidence-collector, executive-planning, meeting-prep, memory-management, soc2-compliance, vanta-bridge

### Bundled Skills

Place a `.skill` ZIP at `skills/bundles/<name>.skill`. The ZIP must contain `<name>/SKILL.md` (and any assets). Users extract these to `~/.claude/skills/` via `os-setup` Step 6. They appear without a namespace prefix.

**Only use this when the skill includes supplementary assets (images, templates, reference files).**

Current: ai-solution-architect, nudesk-brand-styling, srd-generator

### Common Mistakes

- Do NOT create a `.skill` ZIP for a markdown-only skill — it won't auto-update
- Do NOT nest SKILL.md as `skills/<name>/<name>/SKILL.md` — the plugin loader expects exactly one level: `skills/<name>/SKILL.md`
- Do NOT manually extract plugin-local skills to `~/.claude/skills/` — that creates a shadow copy that won't auto-update

## Commands

Commands live in `commands/<name>.md`. They appear as `/nudesk-os:<name>` and are read live from the plugin directory (same auto-update behavior as plugin-local skills).

## Editing Workflow

Always edit files in this working copy. Use the PR workflow below to get changes to main,
then run `claude plugin update nudesk-os@nudesk-os` to sync the install clone.
Do NOT edit `~/.claude/plugins/marketplaces/nudesk-os/` directly.

## Branching & PR Workflow (SOC 2 SD-02)

`main` is protected via org-level GitHub Ruleset. All changes require a pull request.

1. Create a feature branch: `git checkout -b <type>/<short-description>`
2. Commit with conventional commit messages
3. Push and create PR: `gh pr create --title "..." --body "..."`
   - Or ask Claude: "Create a PR for these changes"
4. Squash-merge: `gh pr merge --squash --delete-branch`
5. Sync plugin: `claude plugin update nudesk-os@nudesk-os`

Full guide with examples: `references/branch-protection-guide.md`

## Version Bumps

Update the version in both:
- `.claude-plugin/plugin.json`
- `README.md` (version badge at top)
