# prpack-prompts

Four review prompts for LLM code review. Each one is a single markdown file you append to a packed pull request context (or any other code context) when asking Claude / Cursor / GPT to review.

Each prompt:

- **Picks one concern** — security, performance, tests, or architecture — and tells the model to ignore the other three.
- **Demands a structured finding format** — severity, file:line, why it matters, suggested fix.
- **Forces a one-line verdict** at the end (`ship / fix-before-ship / hold`) so you get a usable signal instead of "looks good but you might want to consider..."
- **Tells the model to skip irrelevant sections** instead of padding with platitudes.

## The prompts

| File | When to use |
|---|---|
| [`security.md`](./security.md) | The PR touches auth, input parsing, file paths, subprocess calls, network egress, dependencies, or anything user-facing. |
| [`performance.md`](./performance.md) | The PR touches a request handler, query, loop over collections, or anything that runs on every request. |
| [`tests.md`](./tests.md) | Always — but especially when you suspect the author wrote tests after the implementation. |
| [`architecture.md`](./architecture.md) | The PR is medium-large (>5 files), introduces a new module, or restructures an existing one. |

Pick one or two based on the PR's risk shape. Don't run all four every time.

## Usage

You can use these with any tool that gives an LLM your code, but they're designed to pair with [prpack](https://github.com/Lucas2944/prpack) — a CLI that packs a PR into a single markdown file containing the diff plus the full post-change content of every touched file.

### With prpack

```sh
npx github:Lucas2944/prpack --out ctx.md
cat ctx.md security.md | pbcopy
# paste into Claude / Cursor / your model
```

Or pass the prompt as a `reviewPrompt:` in a `.prpack.yml` config — it'll be appended automatically.

### Without prpack

Paste your diff (and ideally the full file contents) into a model, then paste the chosen prompt below it. Same effect.

### In Cursor

Save one of these files as `.cursorrules` (renamed) and Cursor's chat will adopt the review persona for the session. There's a Cursor-flavored version of these same prompts at [Lucas2944/cursor-review-rules](https://github.com/Lucas2944/cursor-review-rules).

## Why these prompts catch more bugs than "review this code"

A generic "review this code" prompt asks the model to do four jobs at once: find security issues, find perf issues, find missing tests, and flag architectural problems. Models do all four poorly. Asking for one concern at a time, with a strict output format and an explicit "skip irrelevant sections" instruction, gives you actually-useful feedback.

For a reproducible side-by-side demo of the underlying technique (full file context vs raw diff), see [prpack/examples/invoice-refactor](https://github.com/Lucas2944/prpack/tree/main/examples/invoice-refactor) — paste each context into your model and watch the diff-only review miss a null-deref that the full-context review catches.

**prpack v0.2.0** now inlines these four prompts natively. Run `prpack --review security` (or `performance` / `tests` / `architecture` / `general`) with `ANTHROPIC_API_KEY` set and prpack will pack the PR, append the matching prompt, call Anthropic, and stream the review. See [prpack v0.2.0 release](https://github.com/Lucas2944/prpack/releases/tag/v0.2.0).

I wrote up the longer story behind this here: [Your LLM code reviewer is reading half the file](https://scottthurman.hashnode.dev/your-llm-code-reviewer-is-reading-half-the-file).

## Related

- [prpack](https://github.com/Lucas2944/prpack) — the CLI that packs PRs for LLM review.
- [prpack-action](https://github.com/Lucas2944/prpack-action) — GitHub Action wrapper.
- [prpack-demo](https://lucas2944.github.io/prpack-demo/) — browser demo, no install.
- [cursor-review-rules](https://github.com/Lucas2944/cursor-review-rules) — these same prompts shaped as `.cursorrules` for Cursor's chat.
- [Pro Pack](https://scottthurman89.itch.io/prpack) — the `.prpack.yml` flavor of these prompts plus a workflow guide, free-or-pay-what-you-want.

## License

MIT. Yours to use, modify, and embed in your projects.
