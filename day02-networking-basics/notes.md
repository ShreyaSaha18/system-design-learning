# Day 2 — Networking Basics (system-design lens)

You already know HTTP/REST as a full-stack dev. This day is *not* "what is an HTTP request" — it's "which networking decisions actually change your architecture at scale."

## 1. What happens when a request is made (the full path)

1. **DNS lookup** — browser asks "what's the IP for api.example.com?" DNS resolves a domain name to an IP address.
2. **TCP handshake** — client and server do a 3-way handshake (SYN, SYN-ACK, ACK) to establish a reliable connection.
3. **TLS handshake** (if HTTPS) — negotiate encryption keys.
4. **HTTP request/response** — the actual data exchange.

Why this matters for system design: **DNS is your first layer of load distribution.** A single domain name can resolve to *multiple* IPs (round-robin DNS), or to the nearest data center (GeoDNS / Anycast). Big systems route users to the nearest region before a single line of application code runs.

## 2. TCP vs UDP — the core trade-off

| | TCP | UDP |
|---|---|---|
| Guarantees delivery | Yes (retransmits lost packets) | No |
| Ordering | Yes | No |
| Overhead | Higher (handshake, ACKs) | Lower |
| Use when | Correctness matters (web APIs, file transfer, chat) | Speed matters more than perfection (video calls, live streaming, gaming) |

**Interview signal**: if you're designing a video call app or live-streaming system and default to "just use TCP/HTTP," that's a flag. A dropped/late video frame is invisible; a video call that stalls waiting to retransmit an old frame is worse. UDP (or protocols built on it, like WebRTC/QUIC) is the right call there.

## 3. HTTP evolution — why it matters at scale

- **HTTP/1.1**: one request per TCP connection at a time (unless pipelining, which is barely used). Browsers open multiple parallel connections to compensate — wasteful.
- **HTTP/2**: multiplexes many requests over a *single* TCP connection (no more opening 6 connections per domain). Big win for latency under load.
- **HTTP/3 (QUIC)**: built on UDP instead of TCP, eliminates a problem called **head-of-line blocking** (one lost packet in HTTP/2 stalls *all* multiplexed streams on that TCP connection, since TCP guarantees order). QUIC handles each stream independently.

Why you should care: at high scale, the protocol version genuinely affects latency and server connection counts. It's a legitimate lever, not just trivia.

## 4. Statelessness — the single most important networking-adjacent idea in this whole course

**HTTP is stateless by design**: each request carries everything the server needs (or a token to look it up) — the server doesn't need to "remember" you between requests.

This matters enormously for horizontal scaling: if your *application server* is also stateless (no session stored in server memory), then **any** request can be routed to **any** server behind a load balancer. This is what makes horizontal scaling of the compute layer actually work.

The opposite — **sticky sessions** (routing a user to the same server every time because their session lives in that server's memory) — is a scaling anti-pattern. It works, but it means that server becomes a soft single point of failure for that user, and your load balancer can't freely spread load. The fix: move session state out of the app server and into a shared store (Redis, a database, or a client-side JWT). We'll hit this again when we cover load balancers.

## 5. Communication styles: REST vs gRPC vs WebSockets

| | REST (HTTP+JSON) | gRPC | WebSockets |
|---|---|---|---|
| Model | Request/response | Request/response (+ streaming) | Persistent bidirectional connection |
| Format | JSON (text, human-readable) | Protobuf (binary, compact, faster) | Any (often JSON) |
| Best for | Public APIs, browser clients | Internal service-to-service calls (microservices) | Real-time features: chat, live notifications, collaborative editing |

**Interview signal**: when two of *your own* microservices talk to each other internally, gRPC (binary, strongly typed, faster) is often a better fit than REST. When a *browser* needs to talk to your backend, REST/JSON (or GraphQL) is more practical because of tooling and human-debuggability. When you need the *server* to push data to the client without the client asking (chat message arrives, notification, live score) — that's WebSockets (or Server-Sent Events for one-directional server→client push).

---

## Exercise (before Day 3)

For a **chat application** (like WhatsApp):
1. Which protocol would you use for sending a message from client → server? Why?
2. Which protocol/mechanism would you use to *push* an incoming message to the recipient's phone in real time?
3. Would you want session state stored in the app server's memory, or externally? Why?

---

## Where we go next (Day 3)
Scaling the compute layer: load balancers (algorithms, health checks), stateless services in practice, and horizontal scaling patterns.
