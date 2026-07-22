# Day 6 — Caching

## 1. Why caching exists

Everything so far (replicas, sharding) scales the database itself. Caching takes a different approach: **avoid hitting the database at all** for data you've already fetched recently. It's usually the single highest-leverage performance change you can make — orders of magnitude faster than any database, at the cost of the same thing we've been trading away all week: freshness.

## 2. The cache layers, in order of distance from the user

Caching happens at multiple layers simultaneously in a real system — not just "add Redis":

- **Client-side (browser) cache**: controlled by HTTP headers (`Cache-Control`, `ETag`, `Last-Modified`). The browser may not even make a request if it already has a valid cached copy. Zero network cost when it hits.
- **CDN (Content Delivery Network)**: caches content at edge servers geographically close to users — mainly static assets (images, JS, CSS, video), but increasingly API responses too. Cuts latency (shorter round trip) *and* cuts load on your origin servers entirely for cached requests.
- **Application-level cache**: an in-memory store (Redis, Memcached) sitting between your app servers and your database, caching computed results, DB query results, session data, rendered fragments — anything expensive to recompute.
- **Database-level cache**: most databases have their own internal caching (buffer pool / page cache) keeping hot pages in memory automatically. You don't manage this directly, but it's why a "hot" database can already be much faster than a "cold" one.

Each layer removes load from everything behind it. A well-cached system might serve 95% of requests without ever reaching the database.

## 3. Caching patterns — how the cache and the database stay in sync

- **Cache-aside (lazy loading)** — the most common pattern. The app checks the cache first; on a miss, it reads from the DB and *writes the result into the cache* itself. Simple, and only caches what's actually requested. Risk: the first request after a miss pays full DB latency, and popular-key expiry can cause a stampede (see below).
- **Read-through** — same idea, but the cache itself (not the app) is responsible for loading from the DB on a miss. The app only ever talks to the cache.
- **Write-through** — writes go to the cache *and* the DB together, synchronously. Cache is always fresh, but every write pays extra latency.
- **Write-behind (write-back)** — writes go to the cache immediately and are flushed to the DB asynchronously later. Very fast writes, but real risk: if the cache crashes before the flush happens, that data is gone.

**Interview habit**: cache-aside is the sane default for read-heavy data. Write-through/write-behind are specialized choices you justify with a specific latency or consistency requirement — don't reach for them by default.

## 4. Cache invalidation — genuinely one of the hard problems

Phil Karlton's famous line: "There are only two hard things in Computer Science: cache invalidation and naming things." The core question: **when the underlying data changes, how does the cache find out?**

- **TTL (time-to-live)**: simplest option — every cached entry just expires after N seconds, forcing a refresh. You're explicitly accepting up to N seconds of staleness in exchange for simplicity. This is the default choice unless you have a specific reason not to.
- **Explicit invalidation on write**: when the app writes new data, it also deletes (or updates) the corresponding cache entry. Fresher, but only works if *every* code path that writes the data remembers to also invalidate the cache — a common source of subtle bugs.
- **Event-based invalidation**: a change event (via pub/sub) notifies all interested caches to invalidate the relevant key. Necessary once you have multiple app servers each with their own local cache, so a write on one server doesn't leave stale data sitting in another server's cache.

## 5. Eviction — what happens when the cache is full

Caches have limited memory, so when full, something has to go:
- **LRU (Least Recently Used)** — evict whatever hasn't been accessed in the longest time. The default in almost every real cache (including Redis's default policy).
- **LFU (Least Frequently Used)** — evict whatever is accessed least often overall, useful when access frequency matters more than recency.
- **FIFO** — evict the oldest inserted entry regardless of usage. Simple but usually a worse fit than LRU.

## 6. Cache stampede (a.k.a. thundering herd)

A specific, very real failure mode: a popular cache key expires, and a burst of concurrent requests all miss at the same instant — all of them then hit the database simultaneously to recompute the same thing, potentially overwhelming it right when load is highest.

**Mitigations**:
- **Locking/mutex on refresh**: only the first request that misses actually queries the DB and repopulates the cache; other concurrent requests wait briefly for that result instead of all querying the DB themselves.
- **Jittered TTLs**: instead of every related key expiring at exactly the same time, add small random variation to expiry times so misses spread out instead of clustering.
- **Stale-while-revalidate**: serve the (slightly) stale cached value immediately while asynchronously refreshing it in the background — the user never waits, and the DB only gets one refresh request, not a burst.

## 7. Common Redis patterns worth knowing by name

Redis shows up constantly in system design interviews beyond "just a cache":
- **Cache-aside with `SETEX`**: set a key with a built-in TTL in one call — the standard caching workhorse.
- **Session store**: centralizing session data outside app server memory (this is exactly the fix for the "sticky sessions" anti-pattern from Day 2).
- **Rate limiting counters**: atomic increment operations make Redis a natural fit for tracking request counts per user/IP within a time window (full rate-limiter design is its own interview walkthrough later in this series).
- **Leaderboards**: Redis's sorted sets give O(log n) ranked inserts/lookups — a natural fit for real-time rankings.
- **Pub/sub for cross-server cache invalidation**: ties directly back to point 4 above — when one app server invalidates a key, it can publish that event so every other server's local cache invalidates too.

---

## Exercise (before Day 7)

You're caching a product page for an e-commerce site.

1. Which caching pattern (cache-aside, write-through, or write-behind) fits best here, and why?
2. The product's price just changed. How do you make sure the cache doesn't keep serving the old price — and what's the risk if you rely on TTL alone versus explicit invalidation?
3. This exact product goes viral and its cache entry expires at the worst possible moment — a huge spike in concurrent requests. What's the specific failure mode here, and which mitigation would you reach for first?

---

## Where we go next (Day 7)
Async systems — message queues, pub/sub, and Kafka basics: how services communicate without waiting on each other directly.
