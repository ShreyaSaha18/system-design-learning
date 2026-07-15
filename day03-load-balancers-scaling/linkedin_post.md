Day 3 of learning system design, out loud.

Today's rabbit hole: "just use round robin" is the default answer everyone gives for load balancing. It's also wrong more often than I expected.

Round robin gives every server the exact same NUMBER of requests. What it never checks is how long any of them take. Feed it a realistic mix of fast and slow requests, and the busiest server can end up doing 30%+ more actual work than the quietest — same request count, wildly different load.

I built a small simulation to see this instead of just reading about it: the same stream of requests, routed by round robin, weighted round robin, and least connections, with live bars showing how much real work lands on each server.

The most interesting bug I hit building this: my first version of "least connections" was actually worse than round robin. Turned out my tie-breaking always favored the lowest-numbered server whenever two servers looked equally busy — and with requests arriving faster than they complete, that happens constantly. It silently piled extra load onto Server 1 every time. Fixed by rotating which server wins ties, the way real load balancers do — and the numbers flipped to what the theory actually predicts.

Try your own traffic pattern here: [link to lb-algorithm-race artifact]

Day 3 of a series where I share what I'm actually learning, with a small tool for each concept — mistakes included, since that one was a genuinely useful mistake to make. Days 1 and 2 covered capacity estimation and HTTP/1.1 vs HTTP/2, if you missed them: [link to hub]

Has "just round robin it" ever bitten you in production?

#SystemDesign #SoftwareEngineering #LearningInPublic
