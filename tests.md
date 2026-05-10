# Test-coverage review prompt

Append this to a packed PR context (or any diff + file content you've given the model) and ask for a review.

---

You are reviewing the test coverage of the pull request above. Look at both the changed source and the changed/adjacent tests, then ask: would a real failure in this code be caught?

For every finding, output:

```
[GAP] file:line — what behavior is untested
Why it matters: <what could break in production unnoticed>
Suggested test: <a one-paragraph description of a specific test case, with concrete inputs and expected outputs>
```

Specifically check for:

1. New code paths with no test that exercises them.
2. Tests that assert on incidental output (full JSON equality, field order) where the intent is narrower — these are brittle and will pass even when wrong.
3. Tests that mock the boundary so heavily that the real failure mode (DB error, network timeout, malformed payload) is untested.
4. Edge cases that matter for THIS change: empty input, null, undefined, max-length, negative numbers, unicode, concurrent access. Don't list every theoretical case — pick the ones the implementation actually handles.
5. Tests that pass today but would also pass if the implementation were wrong. ("Tests the spec" vs. "tests what the function happens to return".)
6. Removed tests — was their behavior moved elsewhere, or just dropped?

Order findings by severity. Do not pad.

End with one line:

```
Coverage verdict: adequate | has-gaps | not-meaningfully-tested
```
