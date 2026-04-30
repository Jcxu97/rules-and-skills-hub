---
name: unified-rules-skills-harness
description: Unified Cursor/Claude operating skill that merges spec-delegate-review discipline, conditional patch-first chunked editing, response style constraints, /compact-ready context ledgers, and Databricks 429-safe execution. Use lightly for normal substantive tasks and fully for long files, large writes, bloated context, socket resets, or rate-limit recovery.
---

# Unified Rules + Skills Harness

## Scope

Use this as the single default skill for substantive tasks, but treat it as a safety belt rather than a ritual. It merges:

1. Multi-agent spec-delegate-review workflow.
2. Patch-first, chunked editing discipline.
3. Response-style constraints (heading-first prose, no unordered bullets).
4. Context compaction ledger for long sessions.
5. Databricks AI Gateway rate-limit-safe behavior.

## Activation Levels

### Level 0 - Trivial edit

Use normal direct editing when the change is under about 20 lines, touches one obvious region, and has no recent tool failures.

### Level 1 - Normal substantive edit

Use patch-first thinking, keep the diff surgical, and run the cheapest relevant validation. A full ledger is not required.

### Level 2 - Risky edit

Use small top-down chunks when the change is over about 50 lines, touches a long file, edits structured config, or changes generated-looking content.

### Level 3 - Recovery mode

Use full ledger + chunked edits + low-output behavior when an edit freezes, a socket resets, context is noisy, output is getting large, or any 429 appears.

## Operating Order

For non-trivial tasks, scale this loop to the activation level:

1. Define a micro-spec (goal, non-goal, done condition).
2. Explore relevant files only; avoid broad unrelated edits.
3. Apply minimal changes with patch-first strategy.
4. Validate with the cheapest meaningful check.
5. Compact context when session grows noisy, using `/compact` as a handoff point when supported.

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

Compact after every 2-4 risky chunks or phase changes.

### Claude Code `/compact` handoff

When Claude Code supports `/compact`, use it as the explicit handoff for long or noisy sessions.

Before recommending `/compact`:

1. Produce the ledger above with only active state.
2. Include one exact next step.
3. State that after `/compact`, work should resume only from that next step.
4. Drop stale debate, rejected approaches, repeated background, and old logs.

Recommend `/compact` when:

1. The same background has been repeated.
2. The task crosses a major phase boundary.
3. There have been 2-4 risky patch rounds and more work remains.
4. Databricks 429 or socket instability suggests context/output pressure.

Do not recommend `/compact` for a fresh short task or after every tiny edit.

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
6. Force full recovery workflow onto trivial one-line edits.
7. Recommend `/compact` so often that it interrupts steady progress.

## Trigger Phrases

Activate this skill when requests imply any of:
patch-first, chunked edit, avoid large write, long session, compact context, Databricks 429, REQUEST_LIMIT_EXCEEDED, socket reset, reduce output, safe continuation.
