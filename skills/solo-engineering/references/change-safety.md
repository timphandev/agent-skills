# Change safety

Shipping a change to a system that is already running. The failure mode is specific — old code and new schema coexisting for an interval — and it only happens in production, so it is never observed while writing the change.

Read this **before planning**, not while coding. Every rule here decides the *shape* of the change: how many deploys it takes, what order they run in, whether it can be undone. Discovering any of it with the migration already written means throwing that migration away.

**Assume old code and new code run at the same time.** Any deploy that is not a full stop has an interval where both are live, and a rolling deploy or more than one instance guarantees it. A change that is only correct after every process restarts is a change that is wrong during its own deploy.

**Schema changes go expand → migrate → contract, as separate deploys.** Add the new column nullable and stop there; backfill it; start writing and reading it; and only in a later change drop the old one. Combining any two of those steps means the currently-running code meets a schema it was not written for: a `NOT NULL` column added in the same deploy as the code that populates it fails every insert from the old process still serving traffic, and a column dropped in the same deploy as the code that stopped reading it fails every select from the old one.

**A migration has a stated rollback path before it runs.** Write down what undoes it, or state plainly that it is one-way. Additive changes are trivially reversible; a dropped column or a lossy type conversion is not reversible at all, and that is exactly the thing to know before running it rather than after. A one-way migration against real data belongs under "Needs your call".

**Renaming is two changes, never one.** A column, an API field, a queue name, an event type: add the new name, write to both, migrate readers, then remove the old. The single-step rename works locally, where nothing else is running.

**A change that is risky and reversible should be reversible at runtime, not by deploy.** Put it behind a flag or a configuration value the operator can flip. Rolling back a deploy takes a pipeline run and takes everything else in it back too; flipping a flag takes seconds and takes only the risky thing.

**Say what the deploy order is when it matters.** If the change requires the migration to run before the code, or a worker to be restarted before the API, or a config value to exist first, that ordering is part of the change and belongs in the report. It lives nowhere in the diff, and a solo maintainer deploying on a Friday will not reconstruct it.

**A backfill is a change in its own right, not a step inside a migration.** Updating every existing row is unbounded work against a table of unknown size, and running it inside the migration holds a lock for the duration while the deploy waits behind it. Run it in batches, from a script or job that can be stopped and resumed, with a way to tell how far it got. A backfill that must finish before the next deploy is a dependency for the report; one that cannot be interrupted safely is a design to fix before running rather than a risk to accept.

**A rollback takes the code back and leaves the schema where it is.** Expand → migrate → contract survives that by construction: old code meets a schema carrying extra columns it ignores. What does not survive is a change that made the new shape mandatory — a `NOT NULL` column the old code never populates, a constraint the old writes violate, a renamed table. Which deploys can still be rolled back, and which cannot, belongs in the report. The one that cannot is what the operator needs at the moment they are least able to work it out.

**A flag has a lifecycle, and it is decided when the flag goes in.** Every flag doubles the states the system can occupy, and the combination nobody tests is the one that breaks. Record the removal condition alongside it — after the rollout completes, after a fixed window, once a metric holds. A flag with no removal condition is permanent, and the code on its dead side rots unobserved until something flips it.

**Data written under a flag outlives the flag.** Turning a feature off stops new writes; it removes nothing already written, and teaches the old code nothing about how to read it. Where a flagged path writes a new shape, the off state still has to read it — otherwise the flag is not a rollback but a switch that strands data.
