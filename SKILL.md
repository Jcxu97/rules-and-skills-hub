---
name: unified-rules-skills-harness
description: Unified Cursor/Claude operating skill that merges spec-delegate-review discipline, patch-first chunked editing, response style constraints, context compaction, and Databricks 429-safe execution. Use for any non-trivial coding, writing, refactor, or long-session task.
---

# Unified Rules + Skills Harness

## Scope

Use this as the single default skill for substantive tasks. It merges:

1. Multi-agent spec-delegate-review workflow.
2. Patch-first, chunked editing discipline.
3. Response-style constraints (heading-first prose, no unordered bullets).
4. Context compaction ledger for long sessions.
5. Databricks AI Gateway rate-limit-safe behavior.

## Operating Order

For non-trivial tasks, run this loop:

1. Define a micro-spec (goal, non-goal, done condition).
2. Explore relevant files only; avoid broad unrelated edits.
3. Apply minimal changes with patch-first strategy.
4. Validate with the cheapest meaningful check.
5. Compact context when session grows noisy.

## Editing Safety Rules

Always prefer:

1. Patch/diff edits.
2. Small targeted edits.
3. Full-file rewrite only as last resort.

Chunking rules:

1. Target 50 lines or less per edit when possible.
2. Avoid edits above about 100 lines in one operation.
3. Split large changes into sequential top-down chunks.
4. Re-read changed region after each chunk before continuing.
5. Never retry a failed large edit with identical payload.

## Output Style Rules

Normal replies must follow:

1. Markdown headings with clear hierarchy.
2. Natural prose paragraphs, not unordered bullet walls.
3. Numbered lists only for ordered steps.
4. Use tables for 4+ equal parallel items.
5. Keep tone concise and direct.
6. Match the user's language.

For short replies under ~100 words, headings are optional.

## Context Compaction Ledger

When conversations become long, compress to this active ledger:

Current objective:
<one sentence>

Active files:
<only active paths>

Editing mode:
patch-only / small-edit / direct-edit

Completed chunks:
<short factual lines>

Open issues:
<short factual lines>

Environment constraints:
<only still-relevant constraints>

Next step:
<one sentence>

Validation:
not run / passed / failed (+ active result)

Compact after every 2-4 successful chunks or phase changes.

## Databricks Guardrails

Assume AI Gateway requests can be constrained by QPM, ITPM, OTPM, and pre-reserved output capacity from max_tokens.

Mandatory on 429 REQUEST_LIMIT_EXCEEDED:

1. Stop immediate retries.
2. Reduce scope and expected output.
3. Lower max_tokens.
4. Retry with exponential backoff (2s, 4s, 8s, 16s, optional jitter).
5. Respect Retry-After if provided.

Current working defaults for this environment:

1. User (Default) QPM: 200.
2. User (Default) TPM: 1,000,000.
3. Endpoint-level No limit does not bypass user-level limits.

## Source and Verification Discipline

When external facts are uncertain:

1. Prefer official docs and primary sources.
2. For research claims, prefer DOI-backed papers.
3. For engineering implementation, prefer official/high-signal repos.
4. Validate critical assumptions before coding.

## Forbidden Behaviors

Do not:

1. Rewrite whole files when patching is possible.
2. Mix unrelated edits in one change batch.
3. Burst retry after rate-limit or overload failures.
4. Continue from stale context without compaction when session is bloated.
5. Produce giant unchanged code dumps.

## Trigger Phrases

Activate this skill when requests imply any of:
patch-first, chunked edit, avoid large write, long session, compact context, Databricks 429, REQUEST_LIMIT_EXCEEDED, socket reset, reduce output, safe continuation.
