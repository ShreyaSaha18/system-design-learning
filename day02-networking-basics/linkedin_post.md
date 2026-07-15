Day 2 of learning system design, out loud.

Today's lesson was about a question I'd never actually stopped to think about as a full-stack dev: when your browser loads a page with 36 requests (images, JS, CSS, fonts), why does it feel instant sometimes and sluggish other times — even on the same network?

The answer is the HTTP version.

HTTP/1.1 can only run a handful of requests in parallel per domain (browsers cap it, usually around 6). Everything past that queues up in waves. HTTP/2 multiplexes every request over a single connection — no queueing, no per-connection limit.

So I built a small interactive tool to make that difference visible instead of just describing it: drag the number of requests, the connection limit, and the network latency, and watch two waterfalls race — one queueing in visible waves, one finishing in a single pass.

At default numbers (36 requests, 6 connections, 100ms latency): HTTP/1.1 finishes in 750ms. HTTP/2 finishes in 320ms. Same network, same requests — 2.3x faster purely from the protocol.

Try it yourself: [link to http-waterfall artifact]

This is Day 2 of a series where I share what I'm actually learning, session by session, with a small tool to go with each concept. Day 1 was on capacity estimation, if you missed it.

If you've ever debugged a "why is this page slow" ticket — was it ever actually the protocol, or always something else?

#SystemDesign #SoftwareEngineering #LearningInPublic
