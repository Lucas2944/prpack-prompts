# Architecture review prompt

Append this to a packed PR context (or any diff + file content you've given the model) and ask for a review.

---

You are conducting an architecture review of the pull request above. Pull back from line-level concerns. Look at the *shape* of the change: where the boundary was drawn, what coupling is introduced or removed, and whether the change holds up over the next year of growth.

Address these questions in order, briefly. Use bullets and concrete file references. A senior reviewer skims this; paragraphs lose them.

1. **What is the change actually doing?** Summarize the structural intent in two sentences.

2. **Where did the seams move?** What modules now know about each other that didn't before? What dependencies were inverted, added, or removed?

3. **Coupling check.** Is any new coupling load-bearing? In particular:
    - Are layers leaking (UI reaching into DB, domain types importing transport types)?
    - Are utilities now importing from feature modules?
    - Is shared mutable state being introduced?

4. **Premature or speculative abstraction.** Does the PR introduce interfaces, factories, or config layers that have only one implementation? Flag them — they are debt.

5. **Scalability.** If this code path goes 10x in volume or scope, where does it break first?

6. **Reversibility.** If this turns out to be wrong in a month, how much work is it to roll back? One-way doors should be called out.

7. **Naming.** Are any new types/functions named in a way that will read poorly in six months — names that describe the implementation (`UserManagerImplV2`) instead of the role (`UserDirectory`)?

End with one line:

```
Architecturally sound | needs trim | re-think before merging
```
