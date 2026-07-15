Day 1 of learning system design, out loud.

I'm a full-stack dev picking up system design from scratch, with an AI tutor that doesn't let me get away with sloppy answers (turns out "high latency" and "low latency" are not the same thing — ask me how I know).

Lesson 1 wasn't databases or load balancers. It was numbers.

Before you design anything, you estimate: "10 million daily active users" — is that a job for one server, or fifty? That single number, requests per second, decides almost every architecture decision that follows.

So instead of just writing about it, I built a small interactive tool: plug in your own DAU, requests/user/day, and an availability target — it shows you the QPS, a rough server count, and exactly how many minutes of downtime a year "99.9%" actually allows.

Try your own numbers: [link to capacity-calculator artifact]

This is Day 1 of a series where I'll share what I'm actually learning, session by session — mistakes included — with a small tool to go with each concept.

If you're learning system design too: what tripped you up first?

#SystemDesign #SoftwareEngineering #LearningInPublic
