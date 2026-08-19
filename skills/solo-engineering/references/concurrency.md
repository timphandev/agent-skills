# Concurrency

Bugs that need two things to happen at the same moment, which a single test run rarely arranges. They do not reproduce on the machine you wrote them on: they are found by reasoning while you write the handler, or by an incident afterwards. There is no third way.

Read this while writing the code. Every rule here is about the shape of a handler or a job, so applying it later means rewriting one.

Assume every handler you write runs twice at the same moment, on different machines, against the same row. That is not a rare case: a double-clicked button, a client retry, two tabs, a webhook redelivery, and two workers pulling the same queue all produce it routinely.

**Read-modify-write across two statements is a lost update.** `SELECT balance` then `UPDATE ... SET balance = $computed` loses one of two concurrent increments, and it loses them silently — both requests return success, and the total is simply wrong later. Push the computation into the database (`SET balance = balance + $delta`), or take a row lock for the duration (`SELECT ... FOR UPDATE` inside the transaction that writes), or carry a version. The rule is not "use a transaction": a transaction at the default isolation level does not stop this on its own.

**Anything a user can edit from two places needs an optimistic-concurrency check.** A version column or updated-at timestamp sent back with the write, and a `WHERE version = $seen` on the update: zero rows affected means someone else changed it, which is a conflict to report, not a retry to swallow. Without it, the second save silently discards the first person's edit, and neither of them ever learns.

**Uniqueness is enforced by a constraint, never by a check-then-insert.** "Look up whether the email exists, then insert" is a race with a window, and the window is exactly as wide as your database round trip. Put a unique index on it and handle the violation as the expected path — the check-first version is not wrong because it is slow, it is wrong because it does not work.

**Anything retriable needs an idempotency key.** Payment captures, outbound webhooks, queue consumers, any handler a client will retry on timeout. The client supplies a key, or you derive one from the request, and you store it with the result: a repeat returns the first result rather than performing the work twice. A timeout does not tell the caller whether the work happened, so every timeout becomes a retry, so every non-idempotent handler eventually double-charges someone.

**A recurring job needs a lease, not just a schedule.** [`performance.md`](performance.md) covers the tick that grows until it overlaps the next one; the overlap itself is the second bug, because two copies of a sweep will process the same rows twice. Take a lock the job holds for its duration — an advisory lock, a lease row with an expiry, or a claim on each batch (`UPDATE ... WHERE status = 'pending' ... RETURNING`) so a row is owned before it is worked. The lease must expire, or a job crashing mid-run blocks every later tick forever.

**Do not hold a transaction open across a network call.** An HTTP request or a queue publish inside a transaction pins a connection and its locks for as long as the remote takes to answer, which is unbounded. Commit first and publish after, accepting that the publish can fail, or use an outbox table the transaction writes and a separate process drains.

**Write to shared resources in a consistent order.** Two transactions locking rows A then B, and B then A, deadlock. A fixed ordering — sort the identifiers before locking — makes that impossible rather than rare.

**Every wait takes a timeout, and every timeout needs a caller that survives it.** An outbound call, a lock acquisition, a queue receive: unbounded, one slow dependency becomes every worker blocked on it, which is how a partial outage turns total. The bound is half the work — what happens when it fires is the other half. An immediate retry against an overloaded dependency is what turns a slow service into a dead one, so back off and cap the attempts.

**Propagate cancellation rather than ignoring it.** Once the caller is gone — request abandoned, deadline passed, process shutting down — work still running spends resources on a result nobody reads, and often keeps writing. Pass the signal down to whatever blocks, and check it between the units of a long loop. Work that genuinely cannot stop halfway without leaving state inconsistent belongs in a transaction or behind an idempotency key, rather than becoming uninterruptible by accident.

**Shutdown is a state to design, not an event to survive.** A process being replaced holds in-flight requests, a claimed message, a lease, a half-written batch. Stop accepting new work, let what is running finish inside a bounded grace period, and release leases explicitly instead of waiting for expiry. Whatever does not finish must be safe for the next process to retry — the idempotency rule again, arriving from the other side. A worker killed mid-batch with no lease expiry blocks that work until someone intervenes by hand.

**In-process state does not survive a second instance.** An in-memory cache, a rate-limit counter, a scheduled tick, a deduplication set: each is correct on one process and quietly wrong on two. Nothing announces it — the limit is simply twice what it claims, the job runs twice, the cache serves two answers. Whether the state can be per-instance is a decision to make before putting it there; where it cannot, it belongs somewhere shared.
