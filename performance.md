# Performance review prompt

Append this to a packed PR context (or any diff + file content you've given the model) and ask for a review.

---

You are conducting a performance review of the pull request above. Skip line-level style nits — focus on how this code behaves under real load.

For every finding, output:

```
[IMPACT] file:line — one-sentence summary
Cost shape: constant | linear | quadratic | network | blocking
Why it matters: <one or two sentences>
Suggested fix: <one or two sentences, prefer concrete code>
```

Impact levels:

- **HIGH** — changes asymptotic complexity or adds RTT to a hot path
- **MEDIUM** — per-request waste on a non-trivial endpoint
- **LOW** — cold path or one-off

Specifically check for:

1. Loops that issue queries / network calls per iteration (N+1).
2. Newly-added `await` inside a tight loop where `Promise.all` would unblock work.
3. Re-allocations inside hot paths: new buffers, regex compilation, JSON.parse, deep clones — flag if it looks per-request.
4. Blocking I/O on async paths: sync FS, sync crypto, sync compress.
5. Datastructure choices: `Array.includes` for membership tests on hot lookups, `Object.keys` length checks on large dicts, repeated sorts of the same data.
6. Database changes: missing indexes implied by new query patterns, `SELECT *` additions, transactions held open across awaits.
7. Cache invalidation: did this PR widen a cache key, miss a key, or extend TTL inappropriately?
8. Bundle size impact (frontend): new heavy deps, accidental re-export of an entire library, dynamic-import that became static.

If a section of the diff is irrelevant to performance, say so once and move on.

End with one line:

```
Bottom line: ship | measure-before-ship | regression-likely
```
