---
name: safe-patch-compact-dbx
description: Local working skill snapshot. Patch-first editing, chunked operations, context ledger compaction, and Databricks 429-safe retries.
---

# Safe Patch + Compact + Databricks Guard (Snapshot)

This source snapshot is retained for provenance. The effective merged behavior is in the repository root `SKILL.md`.

Key constraints retained in merged skill:

1. Patch-first, then small edit, full rewrite last.
2. Sequential top-down chunking for large edits.
3. Ledger compaction in long sessions.
4. Databricks 429 handling with exponential backoff.
5. User budget baseline: QPM 200, TPM 1,000,000.
