# agent-skills

[![skills.sh](https://skills.sh/b/timphandev/agent-skills)](https://skills.sh/timphandev/agent-skills)

Skills for AI coding agents, focused on how an agent should *conduct itself* on a task rather than on a specific framework or language.

```bash
npx skills add timphandev/agent-skills
```

## The `solo-*` set

Three skills that layer, plus room to grow. Start with **`solo-start`** — it reads the task and names which of the others to load.

| Skill | What it is for |
|---|---|
| [`solo-start`](skills/solo-start/SKILL.md) | Entry point. Routes a task to the skills below, settles which one owns a job when two look like they overlap, and can write the always-load directive into your project's `CLAUDE.md`. |
| [`solo-project-setup`](skills/solo-project-setup/SKILL.md) | An empty folder or a repository with no agent instructions. Interviews for what the project is, settles the decisions the agreement leaves open, and writes the `CLAUDE.md` every later task reads. Stops before scaffolding. |
| [`solo-engineering`](skills/solo-engineering/SKILL.md) | Everything else — the route from a request to a shipped change, and the quality floors underneath it. |

The order matters: **configure → plan → scaffold.** Setup produces the agreement; scaffolding a project is then an ordinary change that runs through the engineering route like any other, with a plan, verification and a report.

## Skills

### `solo-engineering`

The working agreement an agent follows on implementation work in a codebase meant to live for years, maintained by one person or a handful.

Most skills teach an agent a technology. This one covers what goes wrong regardless of technology: an agent that guesses instead of checking, that fixes three unrelated things in one diff, that reports "done" on code it never ran, or that runs one short command that happens to be unrecoverable.

It is organised as the route a task actually runs — **Diagnose → Orient → Shape → Plan → Act → Prove → Report** — because most of these rules exist to be applied at one particular moment and are useless applied later. Four stages apply to every change and are carried inline; the three conditional ones live in references the skill opens when the task needs them, so a one-line fix does not pay for the machinery a migration needs.

It covers:

- **Judging the altitude first** — whether the request is a fix you can just make, a change that needs a plan, or a goal that needs shaping with you before either. Planning a typo wastes a round trip; coding a goal produces something nobody asked for.
- **Plans as files, not messages** — written to `docs/plans/` before code, as vertical slices that each ship working and each carry the exact command that proves them. A plan in the conversation is gone when the context is compacted; the plan file plus the diff is the whole handoff across sessions.
- **Recoverability before action** — the question is not "am I sure?" or "how many commands would undo this?" but "does the previous state survive?" `git reset --hard`, `rm -rf` and `DROP TABLE` are one command each and none of them can be taken back. Pushing, remote state, CI config, credential rotation and writes outside the repository are all stop-and-ask.
- **Diagnosis before change** — a symptom is not a task until its cause fits in a sentence. A defect nobody can reproduce is one nobody can verify as fixed, and a null check where the null should never have arrived stops the observation without touching what produced it.
- **Scope discipline** — the diff contains what the task named; anything else found along the way gets reported, not silently fixed.
- **What to do when reality contradicts the plan** — which contradictions are corrections to make quietly, which are decisions to stop for, and why finishing a plan whose premise broke is the most expensive outcome available.
- **Verification** — run the project's own checks, confirm they actually covered the change rather than trusting a green summary, report what ran, redact the output before pasting it, and never assert behavior you did not observe.
- **Knowing how the change comes back out** — a thinly-staffed project has nobody to hand a failing deploy to, so reversibility is a design constraint: additive before destructive, a flag over a redeploy, and an irreversible migration named as such before it runs rather than after.
- **Prose that describes the current state** — comments, error messages, READMEs and docs alike: no changelogs in prose, no ticket numbers, no patched-in corrections that orient a reader against a version they never saw.
- **A report ordered by what the reader needs first** — the outcome in one line, then what needs their decision, then what shipping it requires, then what changed and what proved it. Not the order the work happened in.
- **Quality floors** for [testing](skills/solo-engineering/references/testing.md) (read on every code change) and, loaded on demand, for [planning](skills/solo-engineering/references/planning.md), [debugging](skills/solo-engineering/references/debugging.md), [performance](skills/solo-engineering/references/performance.md), [security](skills/solo-engineering/references/security.md), [concurrency](skills/solo-engineering/references/concurrency.md), [change safety](skills/solo-engineering/references/change-safety.md), and [comments, error messages and documentation](skills/solo-engineering/references/prose-in-the-repo.md).

Precedence runs: your agent's own safety rules, then the user, then the project's instructions, then this skill. It is a baseline for where a project is silent — never an override of what it says, and never an override of what you say.

## Who it is calibrated for

The prefix is the scope. Autonomy here assumes a maintainer who would rather review a finished change than answer questions mid-task, on a repository where a wrong recoverable change costs a revert rather than a coordination problem.

Most of the set is not team-size dependent — the destructive-action limits, prompt-injection resistance, verification, scope and the closing report hold anywhere. A few rules are, and each is marked **[Calibrate]** in the skill at the point it appears. On a larger team, override these in your project's instructions file, which outranks the skill:

| Decision | Default here | Change it when |
|---|---|---|
| Autonomy | Proceed on recoverable choices; stop on everything destructive or outward-facing | More people share the repo, or a wrong change costs coordination — narrow it further |
| Commit flow | Branch by default | The project wants direct commits on the default branch |
| Release stance | Assume released; compatibility wins | Pre-release, where breaking changes are genuinely cheaper now than a compatibility plan forever |
| Scale bar | "Design for the target scale" — no number | Always. Put the real number in your project's instructions |
| Where plans live | `docs/plans/<name>.md` | You use an issue tracker or a different directory — what matters is that the plan is durable somewhere |

Narrowing these does not loosen anything else: the blast-radius limits and the verification rules are not part of the autonomy trade.

Stack, domain rules, commands and team conventions belong in your project's `CLAUDE.md` or `AGENTS.md`, layered on top — `solo-project-setup` exists to produce exactly that file, and carries the guide for writing one that does not duplicate the skills.

## Contents

```
skills.sh.json
skills/
├── solo-start/
│   └── SKILL.md
├── solo-project-setup/
│   ├── SKILL.md
│   └── references/
│       └── project-file.md
└── solo-engineering/
    ├── SKILL.md
    └── references/
        ├── testing.md
        ├── planning.md
        ├── debugging.md
        ├── performance.md
        ├── security.md
        ├── concurrency.md
        ├── change-safety.md
        └── prose-in-the-repo.md
```

Each reference lives in exactly one skill. A skill needing another's routes to it by name
rather than carrying a copy, since a skill cannot reliably reference a file in a sibling's
directory — and two copies of a rule are two copies that eventually disagree.

Everything here is Markdown. No scripts, no executables, no network calls, no dependencies — installing this adds text an agent reads, and nothing that runs. See [SECURITY.md](SECURITY.md) for what to report.

## License

MIT
