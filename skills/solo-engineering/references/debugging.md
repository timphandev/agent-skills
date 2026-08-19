# Debugging

Finding the cause of a defect, as distinct from changing code once the cause is known. The two are different work, and conflating them is expensive in a specific way: a fix aimed at a guessed cause is indistinguishable from one aimed at a known cause, right up until the symptom returns.

Read this when something is already wrong — a failing test, a reported symptom, a stack trace, behavior that contradicts what the code appears to say. Read it **before forming a theory**, because the first theory is the one every later observation gets bent to fit.

**The output is a cause, not a change.** Diagnosis ends when the wrong behavior can be traced to a line and explained in a sentence, with evidence to point at. Only then does the change route apply — a plan if the fix spans files, a regression test either way. Editing before that sentence exists is experimenting on the repository, not fixing it.

---

## Reproduce before anything else

**A defect that cannot be reproduced cannot be verified as fixed.** Reproduction comes first for that reason alone, and confidence does not substitute for it: the common way a fix ships wrong is that it corrected a real bug which was not *this* bug, and nothing downstream catches the difference.

Get to the smallest thing that fails reliably. A failing test is ideal, and writing one costs less than it appears to — `testing.md` requires a regression test for the fix regardless, so a reproduction written now is that test arriving early. Where a test cannot reach it, a command or a script will do: something re-runnable on demand.

**Run it enough times to know whether it is deterministic.** A failure that appears once in five is a different defect from one that appears every time, and treating the first as the second wastes the investigation — the "fix" lands, the suite goes green, and nothing has been learned. Intermittency is itself evidence: it points at time, ordering, concurrency, shared state, or data left behind by another test, and `concurrency.md` covers those.

**An unreproducible defect is reported, not guessed at.** Say what was tried and what would settle it — a log line, an input, an environment detail, access to the failing instance. A fix for a defect nobody can reproduce is a fix nobody can verify, which is worse than the open bug: the bug is now believed closed.

---

## Read the actual evidence

**Read the whole error before theorising.** Stack traces get skimmed to the first familiar frame and abandoned there, which is how the wrong layer takes the blame. The frame that matters is often several down, and the message often names the value that was wrong. A wrapped chain reads as a path from what the caller wanted to what actually failed: the innermost cause is the fact, the outer layers say where it surfaced.

**Confirm that what is running is what is being read.** Astonishing behavior rarely is: a stale build, an old container, an unapplied migration, a dependency at a version the lockfile does not describe, a second config shadowing the one that was found. Checking costs a minute and explains the symptom more often than the code turns out to be mysterious.

**Observe rather than infer wherever observing is possible.** Print the value, log the query, read the row, run the request. Concluding from source what the code must do is how a wrong model of the system survives an entire investigation — nothing ever contradicts it. The skill's rule against asserting unobserved behavior applies here most sharply.

**Look at what changed.** Where something worked before, the diff between then and now contains the cause. `git log` and `git bisect` answer that faster than reading does, and bisect is the underused half: it needs only a reliable reproduction, and it returns a commit rather than a theory.

---

## Narrow before fixing

**Halve the search space rather than inspecting it.** Establish which side of a boundary the failure sits on — this layer or the one below, this input or that one, with the cache or without — and discard the other half. A few halvings beat any amount of reading, and each one yields a fact rather than a suspicion.

**Change one thing at a time.** Two simultaneous edits followed by a green run say nothing about which mattered, and both then stay in the diff, since neither can be removed safely.

**Revert every probe.** Debug prints, loosened conditions, commented-out calls, a hard-coded value that made the failure arrive faster — these are among the most commonly committed accidents, and they read as deliberate to whoever arrives later. The finished diff holds the fix and nothing that was scaffolding for finding it.

**Distrust a theory that explains only part of the evidence.** A cause that accounts for the failure but not for why it started on Tuesday, or not for why it affects one tenant only, is a contributing factor with the real cause still upstream. The explanation is finished when it covers everything observed, not when it covers enough to justify an edit.

---

## Fix the cause, at the right level

**Suppressing the symptom is not fixing the defect**, and the tells are specific: a null check where the null should never have arrived, a retry around an operation that fails deterministically, a widened type that silences a compiler, a test loosened until it passes, a `try` around a failure that is not actually acceptable. Each stops the observation without touching what produced it, so the defect remains and resurfaces somewhere less obvious.

Containment before diagnosis is sometimes correct — a production incident, a blocked release — and it is the maintainer's decision rather than one to make quietly. Ship it, name it as containment under **Needs your call**, and say what the real cause is and what fixing it would take. Containment reported as a fix is the failure this rule exists to prevent.

**Ask how far upstream the cause goes.** The line that crashed is where the bad value arrived, not necessarily where it was created. A fix at the crash site leaves the source producing bad values for every other consumer. The source is usually the right level; a fix somewhere in between usually means the boundaries are unclear, which is worth reporting.

**Ask whether the same mistake exists elsewhere.** One defective handler is frequently a pattern copied into four, and searching for the shape is cheap. Fixing the others unasked is out of scope — the diff holds what the task named — so they belong under **Left alone**, with paths and with what makes them look like the same defect.

---

## What to carry into the report

Diagnosis produces knowledge the diff does not contain. The fix is often one line; what took the time was establishing which line.

- **The cause, stated plainly** — one sentence, in **Result** or **Changed**. "The sweep query filtered on `status` without matching `NULL`, so rows never assigned a status were invisible to it" carries more than the diff does.
- **What proved it** — the reproduction, failing before the fix and passing after. This is the regression test `testing.md` already requires, so one artifact serves both.
- **What was ruled out**, where a plausible suspect turned out innocent. That is what saves the next reader an afternoon.
- **The same defect found elsewhere**, under **Left alone**, with paths.
- **Containment shipped in place of a fix**, under **Needs your call**, always.
