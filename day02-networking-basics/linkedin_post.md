Day 2 of learning system design, out loud.

Here's a question I'd used HTTP for a decade without ever actually asking: why does the exact same webpage sometimes feel instant and sometimes crawl, on the exact same network?

The answer isn't your wifi. It's your HTTP version.

Your browser caps how many requests it'll run in parallel per domain — usually 6. HTTP/1.1 can only have 6 requests in flight at a time; everything past that queues up behind them, wave after wave. HTTP/2 multiplexes every request over a single connection: no queue, no per-connection limit.

I built a small interactive tool to make that gap visible instead of just describing it. Drag the number of requests on the page, the connection limit, and the network latency — and watch two waterfalls race: one visibly queueing in waves, one finishing in a single pass.

Default numbers for a fairly typical modern page — 36 requests, 6 connections, 100ms round-trip: HTTP/1.1 finishes in 750ms. HTTP/2 finishes in 320ms. Same page, same network. 2.3x purely from the protocol.

The part that actually surprised me: HTTP/2 isn't faster because any single request gets quicker. Each request still takes the same 100ms round-trip either way. It's faster purely because requests stop waiting in line for a free connection.

Try your own numbers here: [link to http-waterfall artifact]

Day 2 of a series where I share what I'm actually learning, with a small interactive tool to go with each concept. Day 1 was on capacity estimation, if you missed it: [link to hub/day1]

If you've ever chased a "why is this page slow" ticket — was the protocol ever actually the culprit, or was it always something else?

#SystemDesign #SoftwareEngineering #LearningInPublic
