# Day 1 — Foundations of System Design

## 1. What "system design" actually means

As a full-stack dev, you already design systems every day — you just do it at the scale of one service. System design (the discipline, and the interview) is the same thinking applied at scale: multiple services, multiple databases, multiple servers, spread across machines/regions, serving millions of users instead of a handful.

The core skill being tested: **given a vague problem, ask the right questions, then make defensible trade-offs.** There is rarely one "correct" architecture — there's a set of trade-offs, and a good architect can explain *why* they picked one over another.

## 2. Requirements: Functional vs Non-Functional

Every design starts here, before drawing a single box.

**Functional requirements** — what the system does. E.g., for a URL shortener: "user submits a long URL, gets a short URL back; visiting the short URL redirects to the long URL."

**Non-functional requirements (NFRs)** — the qualities the system must have while doing that. This is where system design actually lives:

| NFR | Question it answers |
|---|---|
| **Scalability** | Can it handle 10x/100x more load without falling over? |
| **Availability** | What % of the time is it up and responding? (99.9%? 99.99%?) |
| **Reliability** | Does it keep working correctly, even when parts fail? |
| **Latency** | How fast is a single response? |
| **Throughput** | How many requests/sec can it handle? |
| **Consistency** | Do all users see the same data at the same time? |
| **Fault tolerance** | If a server/disk/region dies, does the system survive? |
| **Maintainability** | Can engineers safely change it later? |

**Key interview habit**: the very first thing you do is clarify requirements out loud — don't assume. "Is this read-heavy or write-heavy? How many users? Do we need strong consistency or is eventual consistency fine?" Interviewers are explicitly grading whether you ask this before designing.

## 3. Scalability: Vertical vs Horizontal

- **Vertical scaling (scale up)**: bigger machine — more CPU/RAM on the same box. Simple, but has a hard ceiling, and it's a single point of failure.
- **Horizontal scaling (scale out)**: more machines, sharing the load. This is how virtually every large system scales — but it introduces new problems: how do you split the work? how do machines agree on data? how do you route a request to the right machine?

Almost everything else in system design (load balancers, caching, sharding, consistent hashing, replication) exists **to make horizontal scaling actually work.**

## 4. Availability, in numbers ("the nines")

Availability = uptime / total time.

| Availability | Downtime/year |
|---|---|
| 99% | ~3.65 days |
| 99.9% | ~8.76 hours |
| 99.99% | ~52.6 minutes |
| 99.999% | ~5.26 minutes |

Interviewers like it when you can translate "five nines" into "5 minutes of downtime a year" — it shows you understand what the target *costs* to achieve (redundancy, failover, monitoring — all expensive).

## 5. CAP theorem (preview — full day dedicated to this later)

In a distributed system, when a network partition happens, you must choose:
- **Consistency** (every read gets the latest write), or
- **Availability** (every request gets *a* response, even if not the latest data)

You can't have both during a partition. This single idea explains a huge fraction of database and architecture decisions you'll see later (why Cassandra ≠ traditional Postgres setups, etc). We'll go deep on this in a later day — for now just know the name and the shape of the trade-off.

## 6. Back-of-the-envelope estimation

A core interview skill: given rough numbers, estimate load. This grounds your design in reality instead of guessing.

**Example**: Design a system with 10 million daily active users (DAU), each making 5 requests/day.
- Total requests/day = 10M × 5 = 50M
- Requests/sec (average) = 50M / 86,400s ≈ **~580 req/sec**
- Peak (assume 3x average for peak hours) ≈ **~1,750 req/sec**

Useful numbers to memorize (rough, approximate):
- 1 day ≈ 86,400 seconds (round to ~100,000 for quick mental math)
- 1 million requests/day ≈ ~12 requests/sec average
- Storage: 1 char ≈ 1 byte; a typical text row ≈ 100 bytes–1KB; an image ≈ 100KB–a few MB

We'll use this constantly once we get to designing real systems (Day 9+).

---

## Exercise (do this before Day 2)

Pick **Twitter** and answer, in your own words, no research:
1. List 3 functional requirements and 3 non-functional requirements.
2. Is it read-heavy or write-heavy? Why?
3. Assume 200M daily active users, each loading their timeline 10 times/day. Estimate requests/sec (average and peak).

Bring your answers next session — we'll critique them together, then use this exact example as we introduce load balancers and databases.

---

## Roadmap (subject to reorder as we go)

1. **Foundations** — requirements, scalability, availability, estimation ← *you are here*
2. Networking basics — DNS, HTTP/HTTPS, TCP vs UDP, REST/gRPC
3. Scaling the compute layer — load balancers, stateless services, horizontal scaling in practice
4. Databases I — SQL vs NoSQL, indexing, replication
5. Databases II — sharding/partitioning, CAP theorem deep dive
6. Caching — client/CDN/app/DB cache layers, cache invalidation, Redis patterns
7. Async systems — message queues, pub/sub, Kafka basics
8. Consistent hashing & partitioning strategies
9. Interview walkthrough #1 — Design a URL Shortener
10. Interview walkthrough #2 — Design a Twitter-like feed
11. Interview walkthrough #3 — Design a chat system (WhatsApp)
12. Interview walkthrough #4 — Design a rate limiter / distributed lock
(more added as needed — API gateways, microservices, observability, security)
