# Writing a project's agent instructions

Read this **when the user has asked you** to set up or restructure a project's agent instructions file — `CLAUDE.md`, `AGENTS.md`, or whatever the agent in use reads.

Everything below describes how to write that file well. None of it is licence to rewrite one you were not asked to touch. That file is what constrains you; deciding on your own that it has "grown unwieldy" and reshaping it is you editing your own instructions, which is a stop-and-ask no matter how much better the result would be. Noticing that it is stale or self-contradictory is worth reporting under **Left alone** — acting on that unasked is not.

## What belongs where

`solo-engineering` carries what is true across every project: the stance, the route a task runs — diagnosing, orienting, shaping, planning, acting, proving, reporting — the blast-radius limits, and the floors for testing, planning, debugging, performance, security, concurrency, change safety and prose. The project's file carries only what is true about **that** project. Duplicating a rule into both is worse than either alone — they will disagree eventually, and nobody will notice which one is stale.

A project file earns its length with these, and roughly in this order:

1. **What the project is** — one or two concrete sentences. Someone must be able to tell whether a domain rule below makes sense.
2. **Anything about this project that contradicts or sharpens the general stance** — a stated scale target, a known ceiling, a release status that changes the breaking-change calculus.
3. **Skills to load, by trigger** — a table of "when the work touches X, load Y". State the trigger, not just the requirement: "use skill X" leaves the agent guessing when.
4. **Commands** — the exact entry points for dev, build, test, lint, codegen, and the one that runs everything. Verify each one runs before writing it down.
5. **Architecture** — the layers or module boundaries and, critically, the import rules between them. This is the part an agent cannot infer from a few files.
6. **Domain rules** — the invariants, traps and rejected alternatives that live nowhere in the code. This is usually the longest section and the highest-value one.
7. **Data and persistence conventions** — migration policy, timestamp handling, indexing rules, anything the storage engine gets wrong by default.
8. **How the project ships** — whether a deploy is rolling or a full stop, whether more than one instance runs, how migrations are applied relative to the code, and what rolling back actually involves. The general floor assumes old and new code overlap; a project where they cannot should say so, and a project where they do should say what the operator has to do by hand.
9. **Project-specific security, performance or testing rules** that go beyond the general floors.

## How to write the rules

**Write rules, not preferences.** "Prefer X where it makes sense" gives an agent nothing to check against. "Use X; Y is acceptable only when Z" does.

**Give the reason, always.** An agent that knows *why* a rule exists applies it correctly to a case the rule did not anticipate; one following a bare directive fails at the first edge. This is the single highest-leverage property of the file. Every "must" earns an "— otherwise".

**Record the rejected alternative.** The highest-value line in a project file is the one saying what was tried and why it was abandoned. Nobody reconstructs that from source, and without it the next agent "simplifies" straight back into the bug.

**Make compliance observable.** Pair a rule with something checkable — a command that must pass, a test that pins the behavior, a section of the report that must name what was applied. A rule nobody can check erodes.

**State the trap, not just the rule.** "The unchecked filter must match `NULL` as well as the literal, or every unchecked row vanishes from the one filter meant to find them" teaches the shape of the bug. "Handle `NULL` correctly" does not.

## What to keep out

- **Backstory.** "We used to X, now we Y", migration notes after the migration, alternatives considered during planning that are not live constraints. Commit messages already hold these, timestamped. When you are rewriting a section you were asked to rewrite, a sentence that loses only history and not a constraint can go — but the burden of proof runs the other way: a sentence you do not understand the purpose of is a constraint you have not identified yet, not backstory. Leave it and say you left it.
- **Roadmap mixed into current behavior.** A reader cannot tell which sentences describe reality. If a planned change is a live constraint — "do not switch this to cursor pagination yet" — state it as a constraint in one line.
- **Anything the repository already records** — the file tree, what a function does, what a past fix was. The file is for what code cannot say about itself.
- **Restating this skill.** See above.

## Keeping it honest

**A stale line here misleads more often than a stale line anywhere else in the repository**, because this file is consulted on every task and nothing compiles it. When a change alters what the file describes, update it in the same change.

**Verify every reference before writing it.** File paths, symbol names, command names, config keys. A confidently-wrong path sends every future agent to a file that does not exist.

**Rewrite sections whole rather than appending.** A project file degrades by accretion: a heading that no longer matches what sits under it, a list whose later entries were added in a different register than its first three, a rule stated twice with different reasons. When a section needs changing, rewrite it against current reality and then re-read it cold — does it repeat itself? Is any sentence meaningful only in light of what used to be there? Tighten until no.
