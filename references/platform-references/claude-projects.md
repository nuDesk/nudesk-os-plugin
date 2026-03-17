# Claude Projects & Skills — Platform Reference

Reference doc for building Claude Projects (claude.ai Team/Enterprise) with Skills, Knowledge files, and Project Instructions. Derived from the MDF Biz Case Co-Pilot build (March 2026).

---

## Architecture

Claude Projects use a 3-layer model:

| Layer | What it is | Where it lives |
|-------|-----------|----------------|
| **Project Instructions** | System prompt — orchestrator, routing, conventions | Project settings (single text block) |
| **Skills** | `.md` files with YAML frontmatter — activated on context match | Uploaded as Skills in the project |
| **Knowledge** | `.md` files with domain data — always available in context | Uploaded as Knowledge in the project |

**Key constraint:** There is no code. Everything is prompt engineering. Skills are instruction sets, not executable code.

---

## Model Selection

**Use Sonnet over Opus for projects with detailed skill instructions.**

| Model | Best for | Risk |
|-------|----------|------|
| Sonnet 4.6 | Structured execution — following detailed instructions literally and quickly | May miss nuance in open-ended analysis |
| Opus 4.6 | Open-ended reasoning, complex judgment calls | Overthinks detailed instructions, slower, buggier (embeds artifacts inline, presents markdown instead of generating files) |

**Rule of thumb:** The more prescriptive your skill instructions are, the less model intelligence you need. Sonnet follows the playbook; Opus rewrites it.

---

## Skills — Authoring Best Practices

### Front-load the output requirement

Claude decides what to do based on early instructions, not buried final steps. If your skill produces a specific output (Word doc, Excel file, React artifact), state it at the very top — before any execution steps.

**Pattern:**
```markdown
## Output Requirement — READ THIS FIRST

**This skill produces a [output type]. This is non-negotiable.**

**FAILURE MODE TO AVOID:** [describe the exact wrong behavior]
```

This was the single most impactful fix in the MDF build. Without it, Claude consistently presented research as markdown and stopped before generating the file.

### Describe the failure mode explicitly

Claude responds better to "don't do X" when X is described in specific, recognizable terms — not abstract instructions.

**Bad:** "Always generate the Word document."
**Good:** "Do not write markdown with headers like '## Company Overview' in chat and treat that as the output. If you find yourself formatting research as a readable chat response, STOP — that content belongs in the .docx."

### Broaden trigger language

Users won't use the exact trigger phrases you list. Add a catch-all rule:

```markdown
**Broad language rule:** If the user mentions [intent keywords] in any
combination, trigger this skill. Don't require exact phrasing — match on intent.
```

Also add to project instructions: "Match on intent, not exact wording. When in doubt, name the skill you're about to run so the user can confirm."

### Two-part document structure: Executive Summary + Detail

For any document that a decision-maker reads, lead with the answer:

- **Page 1: Executive Summary** — who, what, verdict, key risks, next steps
- **Remaining pages: Detail** — full analysis, tables, appendices

This pattern applies to both pre-call briefings and formal business cases. Decision-makers scan page 1 and go deeper only if interested.

### Tick-and-tie across skills

When multiple skills produce outputs in the same chat, numbers WILL drift unless you explicitly prevent it:

```markdown
## Score Consistency — Tick and Tie

Use exact scores from the most recent scoring dashboard. Do not re-derive
composite scores. If new data arrived after scoring, flag it as an analyst
note but still use the existing scores. The user decides whether to re-run.
```

**Rules:**
- Use the **latest version** of each data point
- Same number must be identical everywhere it appears
- If scores are stale after new inputs, flag with a note — don't silently re-score
- The user (not Claude) decides when to re-run upstream skills

---

## Artifacts (React)

### Use `application/vnd.ant.react` type

This renders in the artifact panel (right side), not inline in chat. Claude sometimes ignores this and embeds inline — use the same front-loaded mandate pattern as file generation.

### Sandbox limitations (confirmed)

| Capability | Works? | Notes |
|-----------|--------|-------|
| Recharts (radar, bar, pie) | Yes | Import from `recharts` |
| Tailwind CSS | Yes | Utility classes work |
| State management (useState) | Yes | Full React hooks |
| Tab navigation | Yes | Clickable, instant |
| `navigator.clipboard.writeText()` | **No** | Blocked by sandbox |
| Blob / file download | **No** | `allow-downloads` not set |
| `window.print()` | **No** | `allow-modals` not set |
| `localStorage` | **No** | Blocked |
| `fetch()` / external API calls | **No** | Blocked |

**Implication:** Artifacts are visualization-only. For file export, use a separate skill that generates files via code execution.

### Separation of concerns

- **Dashboard artifact** = visualization and interaction (view scores, adjust tiers, see charts)
- **Scoring report skill** = file export (Excel with live formulas)
- **Business case skill** = narrative export (Word document)

Don't try to make the artifact do export — it can't.

---

## Code Execution (File Generation)

### What works

Claude's code execution sandbox (Python) can generate:
- `.docx` files via `python-docx`
- `.xlsx` files via `openpyxl` (with live Excel formulas)
- Images via `matplotlib`

These appear as downloadable files in chat.

### Multi-tool turn conflict

When a skill requires **web search + MCP calls + code execution** in the same turn, Claude may complete the research and "feel done" before reaching file generation. This is an emergent behavior under context pressure, not a hard platform limit.

**Workarounds:**
1. Front-load the file generation mandate (most effective)
2. Accept that the user may need to say "generate the doc" as a follow-up
3. Keep skill instructions concise — reduce context pressure before the generation step

### User preference overrides

Claude may honor user-level preferences (e.g., "prefer markdown over Word") over skill instructions. If the user has personal settings that conflict with skill output requirements, the skill instruction may lose. The user should adjust their preferences, or explicitly request the output format (e.g., "as a Word doc").

---

## Browser Tool Conflicts

### Chrome integration vs. native web search

Claude may use the Chrome browser integration (page-by-page navigation) instead of its built-in web search (fast, direct results). Explicitly block this in skills that need web research:

```markdown
**Do NOT use the Chrome integration, browser automation, Playwright, or any
MCP-based browser tool — use Claude's native web search capability only.**
```

### MCP connector requirements

These require org admin access to enable — they can't be configured from skill files:
- HubSpot MCP connector
- Gmail MCP connector
- Web search toggle (must be ON at org level)

---

## Project Instructions — Key Patterns

### Skill routing with intent matching

```markdown
**Skill matching — be generous with intent:** Match on intent, not exact
wording. When in doubt, name the skill you're about to run so the user
can confirm.
```

### Output conventions — default to files

```markdown
**Word document exports:** The [skill names] produce downloadable files —
not markdown in chat. If in doubt, generate the file — the user can always
ask for markdown instead, but the default is always the downloadable document.
```

### Order-agnostic design

Documents arrive in any order. Never block on missing inputs. When new documents update previously extracted data, note what changed and flag which downstream scores may be affected. Do not auto-re-run — the user confirms.

---

## Common Failure Modes & Fixes

| Failure | Root Cause | Fix |
|---------|-----------|-----|
| Skill presents markdown instead of generating file | "Task complete" instinct after research | Front-load output mandate with explicit failure mode |
| Artifact renders inline instead of in panel | Model preference for inline content | Front-load artifact panel requirement |
| Scores change between skills | Downstream skill re-analyzes independently | Tick-and-tie rules — use exact upstream numbers |
| Skill doesn't trigger on broad language | Trigger examples are too narrow/specific | Add catch-all intent rule + broaden examples |
| Chrome integration used for web research | Model picks browser tool over native search | Explicitly block Chrome/Playwright in skill |
| User preference overrides skill instruction | User-level settings (e.g., prefer markdown) | User adjusts settings, or adds "as a Word doc" to prompt |
| Opus overthinks and is slow/buggy | Model too powerful for structured instructions | Use Sonnet for execution-heavy projects |

---

## Checklist — Before Deploying a Claude Project

- [ ] Project Instructions uploaded (single text block)
- [ ] All Skills uploaded as `.md` files with YAML frontmatter
- [ ] All Knowledge files uploaded
- [ ] **Code execution and file creation** toggled ON (Settings > Capabilities)
- [ ] **Web search** toggled ON (org-level setting)
- [ ] MCP connectors enabled (HubSpot, Gmail, etc.) if skills reference them
- [ ] Model set to Sonnet 4.6 (unless open-ended reasoning is the primary use case)
- [ ] User-level preferences checked for conflicts (markdown vs. file, inline vs. artifact)
- [ ] Test each skill in a fresh chat with realistic data
