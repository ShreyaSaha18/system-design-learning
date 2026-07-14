# Day 3 — Load Balancers & Scaling the Compute Layer

## 1. What a load balancer actually does

A load balancer (LB) sits in front of a pool of servers and distributes incoming requests across them. Purpose:
- Spread load so no single server is overwhelmed
- Detect and route around unhealthy/dead servers
- Enable horizontal scaling (add/remove servers without clients noticing)

This only works cleanly if the app servers are **stateless** (Day 2) — any server can handle any request.

## 2. L4 vs L7 load balancing

- **L4 (transport layer)**: routes based on IP + port, doesn't look at the actual HTTP content. Fast, simple, protocol-agnostic (works for TCP/UDP generally).
- **L7 (application layer)**: understands HTTP — can route based on URL path, headers, cookies (e.g., `/api/*` → service A, `/images/*` → service B). Slower (more processing) but far more flexible.

Interview signal: if you need path-based or header-based routing (common in microservices), you need L7. If you just need raw fast distribution of any TCP traffic, L4 is enough.

## 3. Load balancing algorithms

| Algorithm | How it works | Good for |
|---|---|---|
| Round robin | Requests distributed in fixed rotation | Servers are roughly equal in capacity |
| Weighted round robin | Rotation, but stronger servers get more requests | Servers have different capacities |
| Least connections | Send to the server with fewest active connections | Requests have variable duration |
| IP hash | Same client IP always → same server | Simple session affinity (use sparingly — see Day 2's sticky session warning) |
| Consistent hashing | Hash-based routing that minimizes reshuffling when servers are added/removed | Distributed caches, sharded systems (full day dedicated to this later) |

## 4. Health checks

The LB periodically pings each server (e.g., `GET /health`). If a server fails checks, the LB stops routing to it until it recovers. This is what makes the system **fault tolerant** at the compute layer — one dead server doesn't cause user-facing failures, it just quietly drops out of rotation.

## 5. Where load balancers actually sit (layered, not just one box)

Real large systems have multiple layers of load balancing:
1. **DNS-level** (Day 2) — routes users to the nearest region/data center.
2. **Global/hardware LB** — in front of a whole data center.
3. **Internal LBs** — between microservices, load balancing traffic to the various backend services.

It's rarely "one load balancer" — think of it as a hierarchy funneling traffic down to smaller and smaller pools.

## 6. Load Balancer vs Reverse Proxy vs API Gateway (quick disambiguation)

These get conflated constantly:
- **Load balancer**: distributes traffic across *identical* server instances.
- **Reverse proxy**: sits in front of one or more servers, can do caching/SSL termination/compression — a load balancer is technically a kind of reverse proxy, but the term is often used even with a single backend.
- **API Gateway**: a reverse proxy with extra smarts — auth, rate limiting, request transformation, routing to *different* services based on the API called. (Full day on this later, once we cover microservices.)

## 7. Horizontal scaling in practice: auto-scaling

Instead of provisioning a fixed number of servers for peak load (wasteful most of the time), cloud systems **auto-scale**:
- Monitor a metric (CPU %, request rate, queue depth)
- If it crosses a threshold, spin up more instances; register them with the LB
- Scale back down when load drops

This is why statelessness matters so much again: new servers can join the pool and immediately start serving traffic with zero special setup.

## 8. Single point of failure (SPOF) — a running theme

Notice: the load balancer itself is now a SPOF if there's only one. Real systems run **multiple LB instances** (often behind DNS or a virtual IP with failover) so the LB layer itself has no single point of failure. This "what if *this* component dies" question is one you should ask about *every* box you draw from now on.

---

## Exercise (before Day 4)

You're designing the **URL shortener** (short.ly). 100M redirects/day, write-to-read ratio is roughly 1:100 (few people create short links, many people click them).

1. Would you use L4 or L7 load balancing here, and why?
2. Which load balancing algorithm would you pick, and why?
3. Given the 1:100 read/write ratio, does that change anything about how you'd scale reads vs writes? (We'll go deep on this in Day 4 — databases — just start forming an intuition.)

---

## Where we go next (Day 4)
Databases I — SQL vs NoSQL, indexing, and replication (read replicas) — directly answering the read-scaling question above.
