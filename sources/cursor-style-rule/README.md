# cursor-style-rule

**One file. Zero bullet points. Dramatically better AI output.**

A Cursor IDE rule (`.mdc`) that forces AI assistants — GPT-4o, Claude, Gemini, or any model behind Cursor — to produce **clean, structured prose** instead of the default wall-of-bullets style. Drop it into any project and the difference is immediate.

## The problem

Every LLM defaults to the same output pattern: a short intro sentence, then an avalanche of `- bullet point` lines, then a one-line summary. This is fine for quick Q&A, but terrible for code reviews, design discussions, architecture docs, and anything you actually need to *read*.

The deeper issue is that bullet points **flatten hierarchy**. A 12-bullet list treats every item as equally important, which means nothing stands out. Prose with headings and selective bold forces the model to **prioritize and structure** its reasoning — and that alone improves answer quality.

## What this rule does

| Dimension | Default LLM behavior | With this rule |
|---|---|---|
| Structure | Flat bullet dump | `#` / `##` / `###` heading hierarchy |
| Body text | Bullet fragments | Natural prose paragraphs |
| Parallel data | More bullets | Markdown **tables** |
| Emphasis | Bold everywhere | One bold span per paragraph max |
| Length | Unbounded | 200–400 words target |
| Language | Always English | Matches user's language automatically |
| Short answers | Still gets a heading | Headings optional under ~100 words |

## Quick start

### Option A — Per-project (recommended)

Copy the rule file into your project:

```
mkdir -p .cursor/rules
cp response-style.mdc .cursor/rules/response-style.mdc
```

Done. Cursor picks it up automatically for this workspace. Commit it so your team gets the same style.

### Option B — Global (all projects)

1. Open Cursor → **Settings** → **General** → **Rules for AI** (or **User Rules** depending on version).
2. Paste the contents of `response-style.mdc` (everything between the `---` fences, plus the body below).
3. Save. This now applies to every workspace on your machine.

## Before / After

**Before** (default GPT-4o style):

```
Here are some suggestions for improving your code:

- Use dependency injection instead of global state
- Add error handling for the API calls
- Consider caching the database queries
- The function names could be more descriptive
- Add type hints to the parameters
- Consider splitting the large function into smaller ones
```

**After** (with this rule):

```
## Architecture

The main bottleneck is global state in the service layer. Extracting
dependencies into constructor parameters would make each class testable
in isolation and remove the implicit coupling between modules.

## Error handling

The three API calls in `sync_pipeline()` currently let exceptions
propagate uncaught. Wrapping each in a try/except with structured
logging would prevent a single upstream timeout from crashing the
entire pipeline.

## Performance

The `get_user_profile` query runs on every request without caching.
A 60-second TTL cache on the user object would cut database load
by roughly 80% for the dashboard page.
```

The second version is **the same information**, but organized so you can actually scan it, share it in a PR, or paste it into a design doc without reformatting.

## Why it works especially well on GPT-style models

GPT-4o and GPT-4o-mini have the **strongest default bias toward bullet lists** among current frontier models. Claude is somewhat better at prose by default, but still over-uses bullets when not constrained. This rule acts as a hard override that redirects the model's formatting habits at the system-prompt level.

The key insight is that **banning bullets forces the model to think harder about structure**. When it can't just dump 10 dashes, it has to decide what deserves a heading, what belongs in a paragraph, and what warrants a table. That decision process produces more thoughtful output — not just prettier output.

## Customization

The rule is a single `.mdc` file. Edit it to taste.

**Want to allow bullets in specific cases?** Change "under any circumstances" to "except inside code review checklists" or whatever your exception is.

**Want longer answers?** Change `200–400 words` to `400–800 words`.

**Want English only?** Remove the "Reply in the same language" paragraph.

## Compatibility

This rule works with any model available through Cursor: GPT-4o, GPT-4o-mini, Claude Opus/Sonnet, Gemini Pro, and local models via Ollama or LM Studio. The `.mdc` format is Cursor-specific, but the text content can be pasted into any system prompt (ChatGPT custom instructions, Claude Projects, etc.) with the same effect.

## License

MIT — use it, fork it, adapt it.

## Contributing

Open an issue or PR if you find a phrasing that works better for a specific model or use case. The goal is a single portable rule that produces clean output across all major LLMs.
