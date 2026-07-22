Day 6 of learning system design, out loud.

Today I learned there's a name for a failure mode I've definitely caused by accident: cache stampede (also called thundering herd). A popular cache key expires, and every concurrent request that was relying on it misses at the exact same instant — all of them then hit the database simultaneously to recompute the same thing, right when load is highest.

I built a small simulator to see how bad this actually gets, and how different fixes compare — not just in theory, but in real numbers.

With 150 concurrent requests and a 150ms database query: no mitigation sends all 150 queries at the database at once, and because the DB is now contended, each one slows down too — average latency balloons to over a second. A "locking" fix (only the first request queries the DB, everyone else waits for it) drops that to a single query, but everyone still waits close to the full 150ms. The strategy that actually wins on both fronts — "stale-while-revalidate" (serve the slightly-old value instantly, refresh it in the background) — also drops to a single query, but users see a response in about 5ms. Roughly 225x faster than doing nothing, while sending 150x less load at the database.

The trade nobody mentions when they say "just cache it": stale-while-revalidate means every single user in that window sees data that's a few seconds old. For a product page, nobody notices. For a bank balance, that's the wrong default.

Try your own numbers: [link to cache-stampede-simulator artifact]

Day 6 of a series where I share what I'm actually learning, with a small interactive tool for each concept. Days 1-5 covered capacity estimation, HTTP/1.1 vs HTTP/2, load balancers, read replicas, and sharding, if you missed them: [link to hub]

Has a cache expiry ever taken down a service you were on call for?

#SystemDesign #SoftwareEngineering #LearningInPublic
