---
name: chunked-edit
description: Local working skill snapshot. Edit long files in small sequential chunks to avoid freeze/timeouts.
---

# Chunked Edit (Snapshot)

This source snapshot is retained for provenance. The chunked-edit behavior is merged into the root `SKILL.md`.

Key retained behavior:

1. Prefer chunks around 50 lines.
2. Avoid chunks above about 100 lines.
3. Edit top-down and verify each chunk before the next.
4. Do not retry the same failing large edit payload.
