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

Always edit files in this working copy, commit and push to GitHub, then run `claude plugin update nudesk-os@marketplace` to sync the install clone. Do NOT edit `~/.claude/plugins/marketplaces/nudesk-os/` directly.

## Version Bumps

Update the version in both:
- `.claude-plugin/plugin.json`
- `README.md` (version badge at top)
