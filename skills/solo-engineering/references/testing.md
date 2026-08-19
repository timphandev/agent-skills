# Testing

A change without tests is not done. What "done" means depends on what changed, not on a coverage percentage.

## What to test

**Always** — anything with a branch. Domain rules and validators, authorization and permission checks, use cases, data-access code containing conditions, joins or transactions, request handlers (status codes, binding, error mapping), and every user-visible behavior of a page: what renders for a given state, what a submit sends, what an error shows. This is where real bugs live and where a regression actually costs something.

**Don't** — code with no behavior to assert against:

- plain type, constant and interface declarations with no methods
- dependency-injection and container wiring
- the application entry point
- generated code (API clients, schema types, docs)
- vendored components pulled from a registry and re-fetched rather than edited

A test over these measures the compiler, the framework, or an upstream project — not this codebase. Worse, they inflate the coverage number, which then hides the untested branch that mattered.

## Unconditional rules

**Every bug fix ships a regression test that fails before the fix and passes after.** No exceptions by package, file, or size of fix. Write the test first and watch it fail — a test written after the fix, never having failed, proves nothing about whether it would catch the bug returning.

**Confirm every new test actually ran, and that it can fail.** These are two distinct failures and both are silent. A test that never executed — a filename the runner does not match, a suite excluded by config, a file in a directory nothing collects from, a `describe` block left focused elsewhere — reports as a green suite, which is indistinguishable from a passing one. Read the run's output for the test's own name rather than the summary line. Then check it can fail at all: a test asserting nothing, awaiting nothing, or asserting a tautology passes against any implementation. Break the code, or the expected value, and watch it go red once. A suite that stays green while the thing it covers is broken is worse than no suite, because it is the reason nobody looks.

**Revert only the defect, not the whole file, when checking that.** Undoing an entire change to see the test fail also undoes the seams the test needs — an injected batch size, a threaded clock, a parameter that made the path reachable at all — so the run takes a different route, never reaches the broken line, and passes. That green is the most convincing false negative available: it looks like proof the test is sound, and it arrives at the exact moment you were trying to be rigorous. Put back the one condition, the one operator, the one comparison, and leave everything else in place.

**Every exported function with two or more branches gets a test per branch.** Judged by reading the diff, not by a percentage. For a validation schema that means one case per rule it can reject plus one that passes: a schema tested only with valid input asserts nothing about the rules that are its entire reason to exist.

**Use the real dependency where you can run it — the same engine you deploy**, in-process or in a container. Exercising real SQL catches what a mock repository never sees, and the SQL is where the bug is. Substituting a *different* engine for speed gives that back: an in-memory stand-in for the database you actually ship is a mock wearing SQL, and it diverges exactly where bugs live — upsert syntax, `RETURNING`, JSON operators, isolation levels, case sensitivity, how `NULL` behaves in a unique constraint. That trade is only sound when the substitute *is* the production engine. Reserve fakes for what you genuinely cannot run — a clock, an outbound network call, a payment provider.

**Inject the clock; never sleep.** A test that sleeps is slow and flaky in proportion to how loaded the machine is. A service or background process taking a clock dependency instead of calling the system clock is what makes its timing testable at all.

## Coverage is observed, not gated

Run the coverage report to find where untested logic hides, then read the **branch** column rather than the total. A file at 100% statements with an untested error path is exactly the gap a high overall number conceals.

A package below some arbitrary percentage is not by itself a defect. Look at what is uncovered: if it is logic, fix it; if it is one of the excluded categories above, leave it.

**Exclusions live in the coverage config with a reason on each entry, and nowhere else.** Never silence a file with an inline ignore comment — it hides the exclusion from the one place a reader looks to audit what is not being measured, so the list they read is quietly incomplete.

## File structure

**One test file per source file**, named after it. Co-locate where the language does that — `foo.go` → `foo_test.go`, `Bar.tsx` → `Bar.test.tsx` — and where the ecosystem puts tests in their own tree instead, mirror the source path inside it: `src/billing/invoice.py` → `tests/billing/test_invoice.py`. Follow whichever layout the repository already uses; introducing a second one is the "two ways to do the same thing" problem from the skill. What does not vary is the one-to-one mapping. Never name a test file after a function or a single concern — a source file declares several functions, and splitting their tests across files breaks the file-to-file correspondence that test runners, coverage output and IDE navigation all rely on.

**Order test functions to match their subjects' order in the source file**, so a reader scrolling both side by side finds the matching test by position instead of searching.

**Scenarios go in named subtests, not in separate top-level functions with the case glued onto the name.** `TestUpdate` with subtests reads as one subject with cases; `TestUpdateWhenSystemRoleAndNameChanged` reads as an unrelated function and sorts away from its siblings.

**Every test that can run independently runs in parallel.** Serial tests that did not need to be serial are the slowest part of most suites. "Can run independently" is doing real work in that sentence, and two cases fail it routinely: a test mutating process-wide state (Go's `t.Setenv` panics outright in a parallel test), and tests sharing one database instance without isolating per test — the leading source of flakes that only appear under load.

## Test behavior, not implementation

Assert what a user sees or what a function returns — never which internal functions were called. A test coupled to internals fails on every refactor that changes nothing observable, which trains everyone to update tests without reading them.

On a UI, drive interactions the way a user does and query by role or accessible name. A test id is for a probe component the test file itself renders, never for reaching into a real one: an element a test can only find by test id is an accessibility finding, not a query problem.
