Day 4 of learning system design, out loud.

The scenario that clicked for me today: someone creates a URL shortener link. Their friend clicks it 200 milliseconds later. Everything should just work — but "should" is doing a lot of work in that sentence.

Read replicas are the standard fix for read-heavy systems: one primary absorbs writes, and you add as many replicas as you need to absorb the reads, scaling each side independently. But replication isn't instant. There's a real, physical delay between "the primary has the write" and "every replica has the write" — and if a reader hits a replica during that window, they get stale data. In the URL shortener's case: a 404 for a link that very much exists.

I built a small calculator to make that trade-off concrete instead of hand-wavy: set your peak read QPS, write QPS, and per-server capacity to see how many replicas you actually need — then set a replication lag and a "time between write and read," and watch which replicas would still be catching up when that read arrives.

Default numbers: 5,000 peak reads, 50 writes, 1,000 req/s per server → 5 replicas needed, primary barely breaking a sweat at 5% load. But with a 300ms worst-case replication lag and someone reading just 200ms after the write — about 20% of replicas are still stale at that moment.

That's not a bug. That's the actual, named trade-off behind "eventually consistent." Async replication buys you fast writes and horizontal read scaling, and the price is a real window where some readers see the past.

Try your own numbers: [link to replica-lag-calculator artifact]

Day 4 of a series where I share what I'm actually learning, with a small interactive tool for each concept. Days 1-3 covered capacity estimation, HTTP/1.1 vs HTTP/2, and load balancer algorithms, if you missed them: [link to hub]

Have you ever chased a bug that turned out to just be replication lag wearing a disguise?

#SystemDesign #SoftwareEngineering #LearningInPublic
