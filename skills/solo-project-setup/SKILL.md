---
name: solo-project-setup
description: "Load this whenever a repository has no CLAUDE.md or AGENTS.md — an empty folder, a fresh scaffold, an inherited project with no agent instructions — and before writing any code there, because every convention invented before that file exists is a convention nobody agreed to. Also use it when asked to write, restructure, or fix a project's own agent instructions file. It interviews for what the project actually is, settles the decisions a solo maintainer's working agreement leaves open — autonomy, target scale, release stance, commit flow, where plans live — establishes and verifies the commands every later change is checked against, and writes the file from those answers. It deliberately stops before scaffolding: choosing a stack, creating directories and installing dependencies are ordinary changes that belong to solo-engineering, which runs them through planning and verification like anything else."
user-invocable: true
license: MIT
metadata:
  author: timphandev
  version: "1.0.0"
---

# Solo project setup

An empty repository has no patterns to infer from. Every rule in `solo-engineering` that says "follow what the codebase already does" has nothing to point at, so the ordinary discipline for making a change does not apply until something exists to be disciplined about.

This skill produces that something. Not code — the **agreement**: what the project is, what its commands are, and which of the open decisions have been settled. One file, written before anything else, that every later task reads.

**What this skill does not do.** It does not choose a framework, create source directories, install dependencies, or initialise git. Those are ordinary changes with real trade-offs and real failure modes, and they deserve a plan, a verification step, and a report — which is what `solo-engineering` provides. The order that works is **configure → plan → scaffold**: a scaffold laid down before the conventions exist is one the conventions then have to accommodate.

---

## Why this comes first

An agent working in an unconfigured repository invents. It picks a test layout, a naming style, an error-handling shape — each individually defensible, none of them agreed. By the time you read the diff, those inventions are load-bearing: the second file follows the first, and changing them later means changing everything written since.

The file this skill writes is what makes the inventions unnecessary. It is also the only place the **[Calibrate]** decisions in `solo-engineering` can live — that skill states defaults and says explicitly that a project should override them. Without this step, those defaults get silently inherited rather than chosen, which is the same as not having decided.

---

## The interview

Ask in the user's language. One question at a time — a list of eight questions gets three answered, and the five skipped are the ones you most needed.

Stop when you can write the file. That is the completion test, not the number of questions asked: if a section below would be guesswork, ask about it; if the answer is already obvious from what exists, do not.

**On a repository that is not empty, read before asking.** A `package.json`, a `go.mod`, a `Cargo.toml`, an existing test directory or CI workflow answers several of these already. Confirm what you inferred rather than asking from scratch — an interview that asks what the code plainly says reads as not having looked.

### What to establish

**What the project is.** One or two concrete sentences, in the user's own words where possible. Everything below is judged against this, and a domain rule cannot be evaluated without it.

**The commands.** Development, build, test, lint, type-check, and the single one that runs everything. These are what every later task verifies against, so an unverified command here poisons every report that follows. On an empty project they may not exist yet — record what they *will* be, and mark the section as pending until the scaffold lands.

**The [Calibrate] decisions.** These are the ones `solo-engineering` leaves open by design. Each already has a default there, so the question is never whether the project gets an answer — it is whether the answer was chosen or inherited. Each has a real cost either way:

| Decision | What to ask | Default if unset | Why it cannot be left at the default silently |
|---|---|---|---|
| Autonomy | Do you want a finished change to review, or to be asked before non-trivial choices? | Proceed on anything recoverable; stop on anything destructive or outward-facing | Determines whether an agent stops mid-task. Wrong either way is expensive: too much stopping wastes your time, too little produces changes you did not agree to. |
| Target scale | How many rows in the largest table in two years? How many concurrent users? | None — the rule has no number to apply | "Design for scale" means nothing without a number. This is the one default that is not a default: unset, every performance decision is a guess. |
| Release stance | Is this released, or can interfaces still change freely? | Released; compatibility wins | Pre-release makes breaking changes cheap and a compatibility plan wasteful. Released makes the opposite true, and inheriting it costs a project that could have moved faster. |
| Commit flow | Branch and merge, or commit directly to the default branch? | A branch | Cannot be inferred: squash- and rebase-merge produce the same flat history a direct-commit repository has. |
| Where plans live | `docs/plans/`, an issue tracker, somewhere else? | `docs/plans/<short-name>.md` | Multi-step work needs a durable plan somewhere. Which directory matters less than that it is under version control — but a project already using a tracker gets two homes for plans if nobody says so. |

Ask about the ones whose default would be wrong here. A project whose maintainer is happy with every default needs no section for this — record that they were reviewed, so the next agent can tell a considered default from an unexamined one.

**Architecture and boundaries**, if any are already decided — layers, module boundaries, the import rules between them. An agent cannot infer an import rule from a few files, and it is the constraint most often violated first.

**Domain rules and traps.** The invariants that live nowhere in the code, and the alternatives already rejected. This section is usually the longest and the highest-value one, and it is worth asking directly: *what has bitten you here before?* On a genuinely new project the answer may be "nothing yet" — leave the section out rather than inventing content for it.

**Skills to load, by trigger.** Which skills apply to which kind of work in this project. State the trigger, not just the name: "when the work touches a query" is actionable, "use the performance skill" is not.

---

## Writing the file

`references/project-file.md` carries how to write it well — what belongs in it, in what order, how to phrase a rule so it survives a case it did not anticipate, and what to keep out. Read it before writing rather than after; it is short, and the failure it prevents is a file that reads well and constrains nothing.

Two things it says that matter most here, because they are hardest to fix later:

**Give the reason, always.** A rule with its reason attached applies correctly to a case it never anticipated. A bare directive fails at the first edge, and every project is mostly edges.

**Do not restate `solo-engineering`.** That skill carries what is true across every project. This file carries what is true about *this* one. A rule duplicated into both will disagree eventually, and nobody will know which copy is stale.

**Verify every command before writing it down.** Run it. A confidently-wrong command in this file sends every future task to a failure that looks like the task's fault.

Choose the filename the user's agent actually reads — `CLAUDE.md` for Claude Code, `AGENTS.md` for several others. Where both apply, write one and have the other point at it rather than maintaining two.

---

## Handing off

Setup ends when the file exists and its commands have been checked. Say so plainly, and say what comes next: the project is now configured, and scaffolding it — choosing the stack, creating the structure, installing dependencies — runs through `solo-engineering` as an ordinary change, with a plan, verification and a report.

Close with the report structure `solo-engineering` defines, in the user's language:

```
### Result
<one line: the project is configured, and what remains>

### Needs your call
- <anything the interview could not settle, with options and a recommendation>

### Changed
- <the file written, and what it establishes>

### Verified
- <each command run> — pass/fail
- <or: pending — no scaffold yet, commands recorded but unverifiable>

### Left alone
- <a section deliberately omitted, and why>
```

**Verified is where this skill is most often dishonest.** On an empty project most commands cannot run yet, and the honest report says that rather than implying they passed. A recorded-but-unverified command is fine; a recorded command presented as checked is how the next task starts from a false premise.
