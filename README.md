# rules-and-skills-hub

Unified repository for your Cursor/Claude rules and skills.

## What this repo contains

This repository consolidates:

1. `cursor-style-rule` (response style rule).
2. `claude-code-multi-agent-harness` (spec-delegate-review harness).
3. Local working skills (`safe-patch-compact-dbx`, `chunked-edit`) merged into one top-level skill.

## Single skill entrypoint

Use the merged skill at:

```text
SKILL.md
```

Recommended install path:

```text
~/.cursor/skills/unified-rules-skills-harness/SKILL.md
```

or for Claude Code:

```text
~/.claude/skills/unified-rules-skills-harness/SKILL.md
```

## Source snapshots

Original repositories are preserved under:

```text
sources/cursor-style-rule/
sources/claude-code-multi-agent-harness/
```

This keeps upstream context intact while allowing the merged one-file workflow.
