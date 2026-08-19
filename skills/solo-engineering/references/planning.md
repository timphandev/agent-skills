# Planning a change

Turning a request into an ordered set of units you can build and prove one at a time. Read this at Orient, the moment the altitude table says a request spans several files or carries a decision you cannot infer — before reading much of the codebase, and before writing anything.

Reading it later is the failure it exists to prevent: a plan written after the code is a changelog, and a plan read after the shape was chosen cannot change the shape.

Two things arrive here besides the plan itself. **Shape** comes first when the request is a goal rather than a change, because there is nothing to plan until the goal is a statement. **Commits** comes last because it governs what happens to finished work, and its rules are the same rules about not doing unasked what you were not asked to do.

---

## Shape

Only when the request is a goal rather than a change — "make search faster", "add billing". Skip it whenever you can already state what the change is.

**Ask what the finished thing does, not how to build it.** One question at a time — a list of six questions gets three answered, and the three that come back are rarely the three that mattered.

**Stop asking when you can state the change in two sentences and the user does not correct them.** That statement is the scope. Everything the plan contains has to trace back to it, and anything that does not is how a feature quietly becomes two.

---

## Plan

**The plan is a file, not a message.** Write it to `docs/plans/<short-name>.md` before touching code. **[Calibrate]** A plan that lives only in the conversation is gone the moment the context is compacted or the session ends — and multi-step work outlives both routinely. The file is also what a second agent, or you in a week, reads instead of re-deriving every decision from the diff. Where the project names a different location — an issue tracker, another directory — use that; what matters is that the plan is durable and under version control, not which directory holds it.

A plan carries:

- **Goal** — one sentence, what is true when this is done.
- **Constraints** — copied verbatim from what the user said, not paraphrased. Paraphrasing is how a constraint quietly loosens: "must not break existing tokens" becomes "should stay backward compatible" becomes a judgment call you make later at 2am.
- **What exists now** — the files, functions and patterns the change touches, with paths, written after reading them rather than from memory. This is the part that catches a plan built on an assumption.
- **Slices** — the ordered units of work, below.
- **Not doing** — what a reader might reasonably expect and will not get, and why. This is where scope discipline starts. The report's **Left alone** should be predictable from here.

### A slice is vertical and shippable

Each slice is one complete path through the stack — model, handler, test, together — not one horizontal layer. A layer cannot be verified on its own, so a plan built from layers has no verification until the end, which is when problems are most expensive and hardest to attribute.

Each slice states:

- **What it does**, from the outside, in one sentence.
- **Files** — exact paths, created or modified.
- **How it is proven** — the exact command, and what output means it passed. A slice with no verification step is not a slice; it is a hope.
- **Depends on** — earlier slices only. A slice depending on a later one means the order is wrong.

A slice touching more than about five files, or that needs two sentences to describe, is two slices that have not been separated yet. Split it before starting rather than halfway through.

### No placeholders

"TBD", "add error handling", "similar to slice 2", "wire it up" — each reads as a decision that has been made and has not been. Making them is the plan's entire job; a plan that defers them has moved the thinking to the moment you were least able to do it, with code already written against the assumption.

Where a decision genuinely cannot be made until an earlier slice runs, write it as a decision point with the options and what would settle it — not as a gap.

### When reality diverges from the plan

It will. A plan is written from what was readable before starting, and the code will contradict it. What matters is which kind of contradiction:

- **The plan named the wrong file, or a helper already exists.** Fix the plan file and carry on. This is a correction, not a decision.
- **A slice turns out to need a decision the plan assumed away.** Stop and ask, once, with the options and your recommendation.
- **Two slices in, the shape is wrong.** Say so rather than completing a plan you no longer believe in. Finishing a plan whose premise broke is the most expensive outcome available here: it produces a change that is reviewable, verified, and wrong — which is far harder to catch than one that is visibly unfinished.

Do not silently improvise around a blocker. The plan is the record of what was agreed, and an agent that quietly rewrites it in its head has removed the user from a decision they were part of.

**Keep the plan file current.** Mark slices done as they land. When work spans sessions, the plan file plus the diff is the entire handoff — a fresh agent with no memory of the conversation should be able to read both and continue. If it cannot, the plan was underwritten.

### What skipping this sounds like

| The thought | Why it is wrong here |
|---|---|
| "It's faster to just start coding." | It is, until slice three contradicts slice one. The plan costs minutes; that contradiction costs everything built since. |
| "I'll write the plan afterwards, from the diff." | A plan derived from what you built cannot catch what you should have built instead. That is a changelog. |
| "The user is watching, they'll correct me." | They asked for a finished change, not a live review. Needing mid-task correction is the cost this whole file exists to avoid. |
| "This slice is small, I'll verify at the end." | Verification at the end tells you something broke, not which slice broke it. |
| "The plan said X but Y is obviously better." | Then say so. Obvious-to-you is exactly the class of change the user did not agree to. |
| "The plan file is stale now, I'll leave it." | A stale plan is worse than none, because the next reader trusts it. Update it or delete it. |

---

## Commits

- **Commit only when asked**, unless the project says otherwise. Pushing is separate and always needs asking.
- **Default to a branch.** **[Calibrate]** Commit directly to the default branch only where the project's instructions say that is the flow. History shape is a weak signal and reads backwards on modern setups: rebase- and squash-merge policies produce exactly the flat, linear history a solo repository has, so "it looks linear" is not evidence that direct commits are welcome.
- **The message explains why; the diff shows what.** A subject line naming the change, then a body giving the reason it was needed and the constraint that shaped it. Skip the body only when the reason is genuinely self-evident.
- **No ticket numbers, dates, or authorship in code comments** — that is what the commit is for, where version control keeps them with a timestamp and a diff attached. The commit carries the history so the comment can describe the code as it stands.
