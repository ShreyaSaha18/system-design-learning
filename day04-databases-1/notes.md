# Day 4 — Databases I: SQL vs NoSQL, Indexing, Replication

## 1. SQL (relational) vs NoSQL

| | SQL (Postgres, MySQL) | NoSQL (varies by type) |
|---|---|---|
| Schema | Fixed, defined upfront | Flexible/schemaless |
| Relationships | Joins are native and efficient | Joins are avoided/denormalized instead |
| Transactions | Strong ACID guarantees | Often weaker (BASE — eventually consistent) |
| Scaling | Vertical is easy; horizontal (sharding) is hard, manual | Many are built to shard/scale horizontally out of the box |
| Best for | Data with real relationships, need for strong consistency (money, inventory, orders) | Massive scale, simple access patterns, flexible/evolving data (activity feeds, sessions, logs) |

**NoSQL isn't one thing** — it's a family, pick based on access pattern:
- **Key-value** (Redis, DynamoDB): O(1) lookup by key. Great for caching, sessions.
- **Document** (MongoDB): JSON-like documents, flexible schema, good for content that varies in shape (user profiles, product catalogs).
- **Wide-column** (Cassandra, HBase): built for massive write throughput and horizontal scale, good for time-series/logs/activity data.
- **Graph** (Neo4j): optimized for relationship-heavy queries (social graphs, recommendation engines).

**Interview habit**: don't say "NoSQL is more scalable" as a blanket claim — say *why*: many NoSQL systems avoid multi-row joins/transactions, which is precisely what makes horizontal partitioning tractable. SQL databases *can* scale horizontally (sharding) too — it's just significantly harder to do correctly because joins and transactions across shards are expensive/complex.

## 2. Indexing

A database index (typically a B-tree) is a separate, sorted data structure that lets the DB find rows without scanning the whole table.

- Without an index: `WHERE user_id = 5` scans every row (O(n)).
- With an index on `user_id`: it's closer to O(log n).

**Trade-off**: indexes speed up reads but slow down writes (every INSERT/UPDATE must also update every index on that table), and they cost storage. So you index columns you frequently filter/sort/join on — not everything.

**Composite indexes**: an index on `(user_id, created_at)` speeds up queries filtering on `user_id` alone, or on `user_id` AND `created_at` together — but not on `created_at` alone (order matters).

## 3. Replication (this directly answers yesterday's exercise)

**Leader-follower (primary-replica) replication**:
- All **writes** go to a single **leader/primary**.
- The leader streams changes to one or more **followers/replicas**.
- **Reads** can be served from any replica (or the leader).

This is exactly how you handle a lopsided read:write ratio like the URL shortener (1:100): one primary absorbs the (small) write volume, and you add as many read replicas as needed to absorb the (huge) read volume — scaling reads and writes **independently**.

**Sync vs Async replication**:
- **Synchronous**: leader waits for replica(s) to confirm before acknowledging the write. Safer (no data loss if leader dies), but slower (write latency depends on the slowest replica).
- **Asynchronous**: leader acknowledges immediately, replicates in the background. Faster writes, but there's **replication lag** — a replica might briefly serve stale data (or even lose the last few writes if the leader crashes before replicating).

**This connects straight back to CAP theorem (Day 1 preview)**: async replication is choosing availability/latency over strict consistency. For the URL shortener, this means: if someone creates a short link and immediately shares it, a reader hitting a lagging replica might briefly get a 404 before the link propagates. Whether that's acceptable is a **product decision**, not just a technical one — worth stating out loud in an interview.

**Multi-leader / leaderless replication** exist too (writes accepted at multiple nodes) — used when you need write availability across regions, but they introduce conflict resolution problems. We'll touch this again in the CAP/partitioning deep dive.

---

## Exercise (before Day 5)

Back to the URL shortener:
1. Would you store the short-code → long-URL mapping in SQL or NoSQL? Justify with the actual access pattern (simple key lookup, no relationships, huge scale).
2. Would you index the short-code column, the long-URL column, both, or neither? Why?
3. You now have 1 primary + 5 read replicas. A user creates a link and immediately sends it to a friend, who clicks it 200ms later. What could go wrong, and what would you tell your team to do about it?

---

## Where we go next (Day 5)
Sharding & the CAP theorem, in depth — what happens when replication alone isn't enough and you need to *partition* data across multiple primaries.
