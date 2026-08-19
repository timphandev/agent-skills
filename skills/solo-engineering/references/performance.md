# Performance

Assume every list runs against a table with hundreds of thousands of rows today and more later. The cost of designing for that up front is a bound and an index; the cost of retrofitting it is a rewrite of every caller.

## Data access

**No query without a bound.** Every list endpoint paginates, through one shared parameter-parsing helper rather than hand-rolled page and sort handling per handler — otherwise each handler grows its own subtly different rules for what `sort=` accepts, and one of them will interpolate it into SQL.

The one query shape that legitimately resists pagination is a tree, which a client cannot reassemble when children land on a different page than their parents. That still takes a hard cap, just not a page parameter.

**Never accept a client-supplied sort or filter column without resolving it against an allowlist.** It is an injection vector and an unindexed-scan vector at the same time.

**No N+1.** A handler returning a list resolves related data with a join or one batched query, never a lookup inside a loop. This includes the loops that do not look like loops: a permission check per row, an enrichment call per item, a decrypt per record. Verify with query logging rather than by reading the code — the loop is often two call frames away.

**Every foreign key, and every column a list query filters or sorts on, needs index coverage before that query ships.** Coverage is the goal, not an index per predicate: one composite index usually serves a filter-plus-sort pair that would otherwise attract two single-column indexes, each taxing every write. What decides whether a column earns its own is skew, not cardinality — an evenly-split boolean is worth nothing to the planner, while a status column that is 99% `done` and 1% `pending` justifies a partial index on the 1%, which is exactly the shape a work queue wants. Read the query plan to confirm an index is used rather than inferring it from the index existing.

**Building an index on a table that already holds rows needs the engine's online path, and that path constrains where the migration can live.** A default `CREATE INDEX` blocks writes for the duration. Engines differ enough that this must be checked rather than assumed: Postgres offers `CREATE INDEX CONCURRENTLY`, which cannot run inside a transaction — so a migration tool that wraps each migration in one by default has to be told not to, and that is the failure that shows up at deploy rather than in review. MySQL/InnoDB builds online by default. SQLite has no concurrent build at all. Confirm what your engine and your migration runner actually do, and ship the index separately from the migration that introduces the column.

**Read the query plan for anything touching a large table.** `EXPLAIN` is cheaper than the incident.

## Background work

**A recurring job must be O(work), not O(table).** A sweep that scans every row to find the few that are due gets slower every day the system succeeds, and it does so silently — nothing fails, the tick just takes longer until it overlaps the next one. That overlap is a correctness problem as well as a performance one, and [`concurrency.md`](concurrency.md) covers the lease that prevents it.

The shape that holds: a cursor column — next-due timestamp, watermark, version — that is indexed, seeded when the row is created, and advanced when the work is done. Query by the cursor with a batch cap. Throughput is then bounded by how much work exists, not by how much history does.

**Anything append-only needs a retention *decision* in the same change that starts writing it** — recorded, not necessarily implemented. History tables, delivery queues, event streams and audit logs all grow without bound otherwise.

**Do not pick the retention period yourself.** How long records are kept is the operator's call and is frequently fixed by law or contract — audit, access and financial logs especially. Raise it under "Needs your call" with the growth rate you measured. When a purge is implemented, delete in batches: a single unbounded delete holds a write lock for as long as it runs, which on a single-writer database means the application stops.

**Tunable configuration must be changeable without a restart.** Configuration pinned at startup is decorative, because nobody will restart a production service to change an interval. Read it through a cached value with a bounded refresh or an invalidation signal so the tick reads memory: one small query on a slow tick costs nothing, but the same read inside the per-item loop is the N+1 above, and a multi-field config read field-by-field can also observe a half-applied change.

## Caching

**Cache only what you measured.** A cache is a second source of truth, and every one of them can serve something stale. That is a real cost, paid forever, so it needs a real query to justify it — read the plan, time the call. Fixing the query or adding the index is the better outcome where it is available, because it has no staleness at all.

**Decide the invalidation before writing the cache.** Every entry needs a stated answer to "what makes this wrong, and what removes it then" — a TTL, an event on write, a key that changes with the data. An entry whose only answer is a long TTL is a decision to serve stale data for that long, which is sometimes correct and must be deliberate rather than inherited from a default.

**Prefer a key that changes over a deletion that must fire.** Building the version, updated-at, or content hash into the key means a stale entry is never read and simply ages out, while explicit invalidation has to reach every place that caches — and the one path that forgets is silent. Where deletion is genuinely needed, it belongs with the write, in the same commit and the same code path.

**Never cache across a permission boundary.** A key omitting the tenant, user, or role serves one caller's data to another, which is an authorization bug that no route-level check catches because the check passed for whoever populated the entry. Include the scope in the key, or do not cache the value at all.

**A cache miss must not stampede.** When a hot entry expires, every concurrent request recomputes it at once and the recomputation is the expensive thing you were avoiding. Collapse concurrent misses onto one computation, or stagger expiry, so a miss costs one recompute rather than however many callers arrived that second.

## Frontend

- **Any list that can exceed a hundred items virtualizes**, and the threshold is what the list *can* hold, not what it holds in development. Not virtualizing is a decision to state rather than to default into: virtualization costs in-page find, printing, and the item count a screen reader announces, so a list that must stay fully searchable is a legitimate exception with a legitimate reason.
- **No data waterfalls.** Fetch in parallel; chain only when the second request genuinely needs the first's result. A chain of three 200ms requests is a 600ms blank screen that did not have to exist.
- **Choose a cache policy per query type explicitly, never the global default.** Reference data that changes at setup, operator data that changes hourly, and machine-written state that changes every few seconds want completely different staleness — one default is wrong for at least two of them.
- **Check bundle impact before adding a dependency.** Build and read the output. Prefer libraries that tree-shake; a utility library imported for one function should cost one function.
