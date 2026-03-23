# Memory Directory Setup

nuDesk OS uses a two-tier memory system. Create this directory structure under `~/.claude/`:

```
~/.claude/
├── CLAUDE.md                    <- Your main config (use CLAUDE.md.template)
└── memory/
    ├── asana-config.md          <- Asana GIDs (use asana-config.md.template)
    ├── glossary.md              <- Full decoder ring for terms and acronyms
    ├── people/                  <- Individual person profiles
    │   └── [person-name].md
    ├── projects/                <- Project detail files
    │   └── [project-name].md
    └── context/                 <- Company and team context
        └── [topic].md
```

## Quick Start

1. Copy `CLAUDE.md.template` to `~/.claude/CLAUDE.md` and fill in your details
2. Copy `asana-config.md.template` to `~/.claude/memory/asana-config.md` and fill in your Asana GIDs
3. Create `~/.claude/memory/glossary.md` with your team's acronyms and terms
4. The `people/`, `projects/`, and `context/` directories will populate over time as you use the session-closeout command

## glossary.md Starter

```markdown
# Glossary

| Term | Meaning |
|------|---------|
| **[ACRONYM]** | [Full meaning] |
```

Memory grows organically — the session-closeout command will suggest additions after each session.
