---
name: solo-start
description: "Load this first on any task that will read or change a repository, before deciding anything else about how to proceed — including tasks that look too small to need routing, since deciding a task is small is itself the judgment this file exists to check. It is the entry point for the solo-* working agreement, active on any coding, review, planning, setup or debugging task in a codebase maintained by one person or a small team. It reads what the task actually is and names which solo-* skills to load: a repository with no CLAUDE.md loads solo-project-setup before anything else, and essentially everything else that touches a repository loads solo-engineering. It also settles which skill owns a job when two look like they overlap, and can write the always-load directive into a project's CLAUDE.md or AGENTS.md so the set triggers reliably on every later task."
user-invocable: true
license: MIT
metadata:
  author: timphandev
  version: "1.0.0"
---

# Solo start

The entry point for the `solo-*` set: a working agreement for codebases maintained by one person or a handful, where the maintainer reviews finished changes rather than answering questions mid-task.

This file routes. It does not carry the rules themselves — each skill below does that, and duplicating them here would produce two copies that disagree.

**Load the skills this table names, and load them now** — before planning, before reading much of the codebase, and all at once rather than one at a time. A skill loaded after the plan is written cannot change the plan, which is the moment it would have been worth most.

---

## Which skills to load

Almost every task in a repository loads `solo-engineering`, so the routing question is really two narrower ones: **does anything need to come first, and is this a task at all?**

| The task | Load |
|---|---|
| A repository with no `CLAUDE.md` / `AGENTS.md` — empty folder or fresh scaffold | `solo-project-setup` **first**, then `solo-engineering` for the scaffolding that follows |
| A question answerable by reading, with nothing to change | Neither. Answer it. |
| Everything else that reads or changes a repository | `solo-engineering` |

That third row is deliberately wide. A one-line fix, a bug with no known cause, a refactor, a dependency bump, a diff to review, a plan to write — all load the same skill, because verification and the closing report are not what varies between them. What varies is which of *its* references apply, and it decides that itself at Orient rather than needing a router to guess from the request.

The set is small on purpose and will grow. When a skill is added, it earns a row here — a skill nothing routes to is a skill that never triggers.

---

## Where the boundaries fall

Two pairs look like they overlap. They do not, and getting them wrong is expensive in opposite directions.

**`solo-project-setup` versus `solo-engineering`.** The split is not "new project versus old project" — it is **which kind of decision the work makes**.

Setup settles what cannot be inferred: the target scale, the release stance, how much autonomy you want, where plans live, what the commands are. Nothing in the codebase can answer these, which is why they are asked rather than derived.

Engineering makes decisions that *can* be inferred — it follows the patterns already there, and where the project's instructions have spoken, it obeys them. Its whole discipline rests on there being something to infer from.

So: a repository with no instructions file has no ground for engineering to stand on, and setup runs first. A repository that has one does not need setup again, even if the change is large.

**Setup versus scaffolding.** Setup stops at the agent-facing configuration. Choosing a stack, creating directories, installing dependencies and initialising git are ordinary changes: they have trade-offs, they can fail, and they deserve a plan, a verification step and a report. They belong to `solo-engineering`.

The order that works is **configure → plan → scaffold**. A scaffold laid down before the conventions exist is one the conventions then have to accommodate.

---

## What the set assumes

Worth knowing before loading anything, because it determines whether these skills fit at all:

- **One maintainer, or a handful**, who would rather review a finished change than be interrupted. On a larger team, several rules need narrowing — each is marked `[Calibrate]` in `solo-engineering`, and the project's own instructions are where they get overridden.
- **The project's own instructions outrank these skills.** Precedence runs: your own safety rules, then the user, then the project's file, then the skill. These are a baseline for where a project is silent, never an override of what it says.
- **The repository is long-lived**, so the trade-off defaults to correct-over-quick, and reversibility is a design constraint rather than an afterthought.

---

## Making the set trigger reliably

A skill cannot force another skill to load. This file can name what to load, but the model still has to act on it — so on a repository where you want the agreement to hold on *every* task, put the instruction where it is read unconditionally: the project's own `CLAUDE.md` or `AGENTS.md`.

**That file is a stop-and-ask.** It is what constrains the agent, so writing to it is not something to do because this section describes how — only because the user asked for it. Where they have not, noticing that the set would trigger more reliably with such a block is a line in the report, not an edit.

Once they have asked, add a block like this near the top of that file, adjusted to what the project actually uses:

```markdown
## Working agreement

Load `solo-start` at the beginning of every task and follow the skills it names.
This is not optional and applies to one-line changes as much as to features.
```

Where the project has more specific routing of its own — a skill for its framework, a house testing skill — list those in the same block with their triggers, so one place answers "what do I load for this?".

`solo-project-setup` covers how to write that file properly, and owns it.
