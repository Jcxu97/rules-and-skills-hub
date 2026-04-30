---
name: safe-patch-compact-dbx
description: Local working skill snapshot. Conditional patch-first editing, activation-level scaling, context ledger, and Databricks 429-safe retries.
---

# Safe Patch + Compact + Databricks Guard (Snapshot)

This source snapshot is retained for provenance. The effective merged behavior is in the repository root `SKILL.md`.

Key constraints retained in merged skill:

1. Activation Levels 0–3: scale the ceremony to the actual risk (trivial / normal / risky / recovery) — do not apply full workflow to one-line edits.
2. Editing order: Edit → sequential Edits (top-down) → Write last. Chunk cohesive regions; never bundle unrelated changes.
3. Chunk sizing: safe under ~50 lines, split above ~100.
4. Context ledger (pre-`/compact`): facts over narrative, current state over history, one next step.
5. `/compact` is a handoff, not a habit — not for short tasks or after every tiny edit.
6. Databricks 429: stop, shrink request and `max_tokens`, exponential backoff with jitter, respect `Retry-After`, serialize — never burst-retry the same payload.
7. Budget baseline: QPM 200, TPM 1,000,000; endpoint "No limit" does not override user-level caps.
