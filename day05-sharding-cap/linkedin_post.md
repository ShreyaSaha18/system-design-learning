Day 5 of learning system design, out loud.

Today's lesson had a number that genuinely stopped me: growing a hash-sharded cluster from 8 shards to 12 doesn't move roughly a third of your data. It moves about 90% of it.

Here's why. The standard way to shard is `shard = hash(key) % N`. Simple, evenly distributed — right up until you need to change N. The moment you resize, the divisor changes, and `%` has no memory of the old assignment. Almost every key gets reshuffled to a different shard, even though most of the physical shards you started with still exist. You just have to move nearly everything anyway.

I built a small simulator to see the actual damage instead of taking it on faith: drag "shards before" and "shards after," and watch a sample of keys light up red (moved) or green (stayed put).

Default numbers — 8 shards growing to 12 — landed at 90% of keys needing migration. That's not a rounding error, that's most of your dataset getting shipped across the network during a routine capacity increase.

There's a fix for this (consistent hashing), and it's specifically designed to only move the minimum number of keys necessary when the cluster resizes — but that's dense enough to deserve its own full lesson, coming later in this series.

Try your own before/after numbers: [link to resharding-simulator artifact]

Day 5 of a series where I share what I'm actually learning, with a small interactive tool for each concept. Days 1-4 covered capacity estimation, HTTP/1.1 vs HTTP/2, load balancer algorithms, and read replicas, if you missed them: [link to hub]

Anyone here lived through a resharding migration in production? How bad was it really?

#SystemDesign #SoftwareEngineering #LearningInPublic
