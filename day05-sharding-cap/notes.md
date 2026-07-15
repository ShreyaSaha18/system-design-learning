# Day 5 — Sharding & the CAP Theorem, In Depth

## 1. Why replication alone eventually runs out of road

Day 4 covered read replicas: one primary takes writes, replicas absorb reads. That solves *read* scaling. But two problems remain:

- **Writes still all go through one primary.** If your write volume outgrows what a single machine can handle, replicas don't help at all.
- **The whole dataset must still fit on one machine.** Replication copies the *entire* dataset to every replica — it doesn't split it up.

**Sharding (horizontal partitioning)** solves both: instead of copying the whole dataset everywhere, you split it into pieces ("shards"), each living on its own machine (often with its own primary + replicas underneath it). Now you can scale writes *and* total data size by adding more shards.

**Important terminology distinction** (often conflated by beginners): **replication** = copying the *same* data to multiple machines (scales reads, adds redundancy). **Sharding** = splitting *different* data across multiple machines (scales writes and storage). Most large systems do both at once — each shard is itself replicated.

## 2. Choosing a shard key

The **shard key** (or partition key) determines which shard a given row lives on. This is one of the most consequential decisions in a sharded system's design, and it's hard to change later. A good shard key:
- Distributes data (and load) evenly across shards — no shard should get dramatically more data or traffic than others.
- Matches your access patterns — most queries should be answerable from a single shard (cross-shard queries are expensive or impossible, see below).

**Bad shard key example**: sharding a `users` table by signup date. All *new* signups (and all their activity) hit the newest shard — a classic **hot shard** problem, where one shard is overloaded while others sit idle.

## 3. Sharding strategies

- **Range-based sharding**: assign contiguous ranges of the key to each shard (e.g., user IDs 1–1M → shard A, 1M–2M → shard B). Simple, and range queries stay efficient. Downside: risk of hotspots if writes cluster at one end of the range (e.g., auto-incrementing IDs always hit the newest/last shard).
- **Hash-based sharding**: `shard = hash(key) % N`. Distributes load evenly regardless of key patterns. Downside: changing `N` (adding a shard) reshuffles almost *every* key's assignment, since the modulo changes for nearly everything — a painful, high-risk migration. (This exact problem is what **consistent hashing** exists to fix — full day dedicated to it later; for now just know plain hash-mod-N has this weakness.)
- **Directory-based sharding**: a lookup service/table explicitly maps each key (or key range) to a shard. Flexible — you can rebalance by just updating the directory — but that directory becomes a new component to keep available and consistent (and a potential bottleneck/SPOF if not itself replicated).

## 4. The real costs of sharding

Sharding isn't free — it's a trade you make when you've actually outgrown a single database:
- **Cross-shard joins and transactions become expensive or impossible.** If two related rows live on different shards, you can't just `JOIN` them in one query anymore — you need application-level joins, denormalization, or a distributed transaction protocol (slow, complex).
- **Resharding is operationally painful**, especially with plain hash-mod-N.
- **Hot shards** can still occur even with a good key, if your traffic pattern shifts (e.g., one celebrity user's posts hammering their one shard).

**Interview habit**: don't reach for sharding as a default. It's a significant complexity cost. The right sequence is usually: vertical scaling → read replicas → caching → *then* sharding, once you've exhausted the cheaper options and a single primary genuinely can't keep up with writes or storage.

## 5. The CAP theorem, properly this time

Recall from Day 1: in a distributed system, you pick two of three guarantees — but really, the honest framing is sharper than that.

- **Consistency (C)**: every read receives the most recent write (or an error). All nodes see the same data at the same time.
- **Availability (A)**: every request receives a (non-error) response — without guaranteeing it's the most recent data.
- **Partition tolerance (P)**: the system keeps operating even when network failures split it into groups that can't talk to each other.

**The key insight most people miss**: partitions are not optional. In any real distributed system spanning multiple machines (let alone data centers), network partitions *will* happen eventually — a cable gets cut, a switch fails, a region goes dark. **P is not a choice you get to opt out of.** So the real, unavoidable choice CAP describes is: **when a partition happens, do you sacrifice Consistency or Availability?**

- **CP systems**: during a partition, nodes that can't confirm they have the latest data refuse to answer (sacrificing availability) rather than risk returning stale/wrong data. Examples: ZooKeeper, etcd, traditional RDBMS configured with synchronous replication, HBase.
- **AP systems**: during a partition, every node keeps answering with whatever data it has, even if it might be stale (sacrificing consistency). Examples: Cassandra, DynamoDB, DNS.

**Concrete example**: a bank's core ledger balance is usually CP — better to show an error than let you overdraw because two partitioned nodes each thought you had the full balance. A social media like-count is usually AP — showing a slightly stale count is far better than the app refusing to load.

## 6. Bonus: PACELC (worth knowing for credibility)

CAP only describes behavior *during* a partition. **PACELC** extends the idea: **P**artition — choose **A** or **C**; **E**lse (no partition) — choose **L**atency or **C**onsistency. Even when the network is healthy, synchronous replication (favoring consistency) costs latency, while async replication (favoring speed) risks staleness. This is exactly the sync-vs-async replication trade-off from Day 4, generalized. Mentioning PACELC in an interview signals you understand CAP isn't the whole picture.

---

## Exercise (before Day 6)

You're sharding a social media app's `posts` table by `user_id`, using hash-based sharding across 8 shards.

1. A new celebrity user joins and posts go viral — millions of likes/comments pour in for their posts specifically. What problem does this create, and does changing the *sharding strategy* fix it, or is this a different problem entirely?
2. You need to go from 8 shards to 12 to handle growth. What's painful about this with hash-mod-N sharding specifically?
3. One shard's replica set loses network contact with the rest of the cluster for 30 seconds. For a social media feed, would you rather that shard go CP (refuse reads/writes until reconnected) or AP (keep serving, possibly stale, data)? Justify it the way you would in an interview — state the trade-off, don't just pick a side.

---

## Where we go next (Day 6)
Caching — client/CDN/application/database cache layers, cache invalidation strategies, and common Redis patterns.
