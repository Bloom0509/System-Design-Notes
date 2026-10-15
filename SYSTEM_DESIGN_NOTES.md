# System Design Revision Notes

> Goal: revise the fundamentals quickly, understand when to use each component, and explain how the pieces fit into a scalable architecture.
>

## Contents
1. [A mental model for system design](#1-a-mental-model-for-system-design)
2. [DNS and HTTP/HTTPS](#2-dns-and-httphttps)
3. [Vertical vs horizontal scaling](#3-vertical-vs-horizontal-scaling)
4. [Load balancers: L4 vs L7](#4-load-balancers-l4-vs-l7)
5. [CDN architecture](#5-cdn-architecture)
6. [Redis and caching strategies](#6-redis-and-caching-strategies)
7. [Database indexing](#7-database-indexing)
8. [Replication](#8-replication)
9. [Sharding](#9-sharding)
10. [Consistent hashing](#10-consistent-hashing)
11. [Message queues and event-driven architecture](#11-message-queues-and-event-driven-architecture)
12. [Kafka vs RabbitMQ](#12-kafka-vs-rabbitmq)
13. [Blob/object storage (S3)](#13-blobobject-storage-s3)
14. [Rate limiting](#14-rate-limiting)
15. [Polling vs WebSockets](#15-polling-vs-websockets)
16. [Idempotency](#16-idempotency)
17. [CAP theorem](#17-cap-theorem)
18. [End-to-end scalable architecture](#18-end-to-end-scalable-architecture)
19. [Trade-off cheat sheet](#19-trade-off-cheat-sheet)
20. [Interview revision checklist](#20-interview-revision-checklist)

---

## 1. A mental model for system design

System design is about choosing components to meet requirements for **latency, throughput, availability, durability, consistency, security, and cost**.

A useful pattern for interviews:

1. Clarify functional requirements and non-functional requirements.
2. Estimate scale: users, requests/second, read/write ratio, data size, payload size, and growth.
3. Draw the simplest design that works.
4. Identify the first bottleneck or failure mode.
5. Add a component that addresses that specific problem.
6. State the trade-off and how you would monitor it.

**Remember:** don't add Kafka, Redis, sharding, or microservices just because they are popular. Introduce each only when there is a concrete reason.

## 2. DNS and HTTP/HTTPS

### DNS
DNS (Domain Name System) resolves a hostname such as `app.example.com` to an IP address. A resolver may use cached records; DNS records have a TTL that influences how long answers may be cached.

Typical request path:
1. User enters `https://app.example.com/products/42`.
2. DNS resolution finds an IP address.
3. The client establishes a network connection to the destination.
4. For HTTPS, TLS authenticates the server and establishes encryption.
5. The client sends an HTTP request; the server returns an HTTP response.

DNS does **not** send the web page itself; it helps the client find where to connect.

### HTTP vs HTTPS
- **HTTP** defines request/response semantics: methods, headers, status codes, and bodies.
- **HTTPS** is HTTP protected by TLS. HTTP is still used in development, internal networks, and some controlled contexts; public production traffic should generally use HTTPS.
- HTTP/1.1, HTTP/2, and HTTP/3 differ in transport and multiplexing details. HTTP/3 uses QUIC over UDP; don't assume every HTTP request is always carried over TCP.

Example request:
```http
GET /api/products/42 HTTP/1.1
Host: app.example.com
Accept: application/json
```

Example response:
```http
HTTP/1.1 200 OK
Content-Type: application/json

{"id": 42, "name": "Keyboard"}
```

Useful status codes: `200` success, `201` created, `400` invalid request, `401` unauthenticated, `403` unauthorized, `404` not found, `429` rate limited, `500` server error, `503` temporarily unavailable.

## 3. Vertical vs horizontal scaling

### Vertical scaling (scale up)
Increase the resources of one machine: CPU, RAM, disk, or network capacity.

**Example:** upgrade an API server from 2 vCPU/4 GB RAM to 8 vCPU/32 GB RAM.

- Pros: simple; often no application-level distribution changes.
- Cons: hardware/instance ceiling, potentially expensive upgrades, and a larger single failure domain.

### Horizontal scaling (scale out)
Add more instances and distribute traffic between them.

**Example:** run 6 stateless API instances behind a load balancer.

- Pros: more aggregate capacity, resilience to individual instance failures, and incremental scaling.
- Cons: requires traffic distribution, health checks, observability, and careful handling of shared state.

**Important:** horizontal scaling works best when app instances are stateless. Put shared sessions in a shared store or use signed client-side tokens rather than relying on one server's local memory.

## 4. Load balancers: L4 vs L7

A load balancer provides a stable entry point and distributes connections/requests to healthy backends.

### Common balancing algorithms
- **Round robin:** rotate through servers; works well when requests and server capacities are similar.
- **Least connections:** prefer a server with fewer active connections; useful when connection durations vary.
- **IP hash:** map a client IP to a backend. This can support affinity, but NAT, changing IPs, and uneven client populations can cause imbalance.
- **Weighted routing:** send more traffic to higher-capacity instances.

### L4 vs L7

| Feature | Layer 4 | Layer 7 |
|---|---|---|
| Typical layer | TCP/UDP | HTTP/HTTPS |
| Routing inputs | IP, port, transport connection | Host, path, headers, cookies, HTTP properties |
| Main strength | Low overhead; protocol-level forwarding | Application-aware routing and richer policies |
| Example use | TCP service, high-throughput connection routing | Route `/api` to APIs and `/images` to an image service |

L7 can route `/api/orders` to an order service and `/api/users` to a user service. Both types can support health checks depending on the product.

### Health checks and failure handling
The load balancer probes backends and stops sending new traffic to unhealthy instances. Deploy multiple load-balancer nodes or use a managed, highly available service so the load balancer itself does not become a single point of failure.

**Trade-off:** sticky sessions can simplify legacy stateful apps, but reduce flexibility. Prefer stateless instances or shared session storage where practical.

## 5. CDN architecture

A Content Delivery Network (CDN) caches and serves content from edge locations closer to users.

**Typical flow:**
1. A user requests an image, stylesheet, JavaScript bundle, video segment, or cacheable response.
2. The nearby edge checks its cache.
3. On a **cache hit**, it returns the object.
4. On a **cache miss**, it fetches from the origin, caches according to policy, and returns it.

### Why use a CDN?
- Lower latency for geographically distributed users.
- Less bandwidth and load on the origin.
- Better delivery of large static assets and media.
- Some CDN products also offer TLS termination, DDoS protection, and edge rules.

### Cache controls to remember
- Cache-Control headers, TTL, cache keys, invalidation/purging, and versioned asset filenames.
- Do not publicly cache personalized or private responses unless the cache key and policy explicitly make it safe.
- A CDN is not a replacement for database caching or application-level caching; it solves edge delivery.

**Example:** serve `/assets/app.abc123.js` from the CDN, while `GET /api/me` goes to the application and is not shared-cached.

## 6. Redis and caching strategies

Redis is an in-memory data store often used for low-latency caching, counters, rate limits, ephemeral state, and certain queues/coordination patterns. It is not automatically the source of truth for durable business data.

### Cache-aside (lazy loading)
1. Application checks cache.
2. On hit, return the cached value.
3. On miss, query the database.
4. Store the result in cache with a TTL and return it.

Example (Python-like pseudocode):
```python
def get_product(product_id):
    key = f"product:{product_id}"
    value = redis.get(key)
    if value is not None:
        return value

    value = database.fetch_product(product_id)
    if value is not None:
        redis.set(key, serialize(value), ex=300)  # 5-minute TTL
    return value
```

### Common cache write strategies

| Strategy | Write path | Benefit | Risk / cost |
|---|---|---|---|
| Cache-aside | App updates DB and invalidates/refreshes cache | Simple, flexible | Stale values or cache stampede if poorly managed |
| Write-through | Write cache and backing store as part of write path | Cache stays warm and more consistent by design | Added write latency; coordinating failures matters |
| Write-back / write-behind | Write cache first; persist asynchronously | Very fast writes and can batch persistence | Data-loss risk if not durably logged; ordering/recovery complexity |
| Read-through | Cache layer loads missing values from backing store | Simplifies callers | Depends on cache/provider support |

### Eviction and expiration
- **LRU:** evict least recently used items.
- **LFU:** evict least frequently used items.
- **TTL:** expire entries after a configured duration.
- TTL is expiration, not exactly the same as an eviction algorithm; real stores may combine expiration and memory-pressure policies.

### Common cache problems
- **Cache stampede/thundering herd:** many requests miss/expire the same hot key at once and overload the DB. Use request coalescing/single-flight locks, jittered TTLs, stale-while-revalidate, or controlled refresh.
- **Cold cache:** a new/emptied cache sends many misses to the database. Consider warming known hot keys and ramping traffic carefully.
- **Stale data:** define invalidation rules, short TTLs, or versioned keys based on business needs.
- **Hot key:** one key receives disproportionate traffic. Consider local in-process caching, replication, key splitting, or request coalescing.
- **Cache outage:** use timeouts, circuit breakers, and controlled fallback; avoid every request stampeding the DB.

**Interview example:** product catalogue pages can tolerate a short stale window; account balance or authorization decisions may need stronger consistency and should not blindly trust a stale cache.

## 7. Database indexing

An index is an auxiliary data structure that helps the database find rows without scanning the entire table. B-tree indexes are common; other index types support different query patterns.

Example:
```sql
CREATE INDEX idx_orders_customer_created
ON orders (customer_id, created_at DESC);

SELECT id, created_at, total
FROM orders
WHERE customer_id = 42
ORDER BY created_at DESC
LIMIT 20;
```

A suitable composite index can help filter by customer and return recent orders in order. Index usefulness depends on the query plan, selectivity, sort, and database engine.

### Trade-offs
- Faster matching reads and sometimes sorting/joins.
- Extra disk/storage and memory use.
- Inserts, updates, and deletes must maintain indexes, so too many indexes slow writes.
- Column order matters for composite indexes. An index on `(customer_id, created_at)` is not generally interchangeable with one on `(created_at, customer_id)`.

Use `EXPLAIN` / `EXPLAIN ANALYZE` (carefully in production) to inspect query plans. An index is not a magic fix for every slow query.

## 8. Replication

Replication copies data from one database node to another.

### Primary/replica pattern
- Primary handles writes.
- Replicas receive changes and can serve reads.
- Failover may promote a replica if the primary fails, using an election/managed failover process.

**Useful when:** read traffic is much higher than write traffic and read capacity or availability needs to grow.

### Key trade-offs
- Replication is not necessarily instantaneous. **Replication lag** means a replica can return stale data.
- If a user updates a record and immediately reads it, route that read to the primary or use a consistency mechanism when read-your-writes behavior is required.
- Failover needs safe leader election, fencing, and recovery procedures to avoid split-brain or lost acknowledged writes.
- Replicas improve read scale and resilience; they do not automatically increase write throughput.

## 9. Sharding

Sharding partitions one logical dataset across multiple database nodes. Each shard stores a subset of the data.

### Common shard-key strategies
- **Range-based:** IDs 1–1M on one shard, 1M–2M on another. Easy range queries, but hot ranges and uneven growth are possible.
- **Hash-based:** hash a key and map it to a shard. Often distributes data more evenly, but range queries become harder.
- **Directory-based:** a lookup service maps keys/tenants to shards. Flexible, but the directory needs availability and scale.

Example concept:
```python
shard_number = hash(user_id) % shard_count
```
This simple modulo example illustrates the idea, but changing `shard_count` can remap many keys. Consistent hashing or an explicit shard map can reduce or control movement.

### Sharding trade-offs
- More storage and write capacity across machines.
- More operational complexity: migrations, rebalancing, cross-shard joins/transactions, unique IDs, and hot shards.
- Choose a shard key with high cardinality and traffic distribution in mind. Avoid keys that send most traffic to one shard.

**Replication and sharding solve different problems:** replication copies a shard for availability/read scale; sharding splits the dataset for capacity. Real systems often use both.

## 10. Consistent hashing

Consistent hashing maps keys and servers onto a conceptual ring. A key is assigned to the next server encountered in ring order. Virtual nodes represent each physical server multiple times to improve distribution.

### Why it helps
With simple `hash(key) % N`, changing the number of servers can remap a large fraction of keys. With consistent hashing, adding or removing a server usually moves only the keys in the affected ring ranges (exact behavior depends on implementation).

Used in distributed caches, partition assignment, and some sharded systems.

**Trade-offs and caveats:**
- Virtual nodes improve balance but do not guarantee perfect balance.
- A good hash function and rebalancing plan matter.
- Consistent hashing does not itself replicate data, guarantee consistency, or automatically handle failed requests; those are separate concerns.

## 11. Message queues and event-driven architecture

A queue decouples the request handler from work that can happen asynchronously.

### Without a queue
A photo upload request may synchronously upload the file, resize it, generate thumbnails, update metadata, and send email. The user waits for every operation; slow work increases latency and failures become tightly coupled.

### With a queue
1. API validates the request and stores the durable input/metadata.
2. API publishes a job/event to a broker.
3. API returns an accepted response (often `202 Accepted`) with a job ID.
4. Workers consume jobs independently: thumbnail worker, email worker, analytics worker, etc.
5. Failed jobs can be retried; repeatedly failing jobs may go to a dead-letter queue.

Example event:
```json
{
  "event_id": "evt_123",
  "type": "photo.uploaded",
  "photo_id": "photo_456",
  "user_id": "user_42"
}
```

### Benefits
- Smooths traffic spikes (the queue buffers work).
- Lets worker pools scale independently.
- Isolates slow or failing downstream operations.
- Supports retries and asynchronous processing.

### Important design details
- A broker acknowledgement does not necessarily mean all business processing is complete.
- Consumers should be idempotent because messages may be delivered more than once.
- Set retry limits and backoff; use dead-letter queues and alerts.
- Monitor queue depth, oldest-message age, processing time, retry rate, and consumer lag.
- Avoid dual-write loss: if DB update succeeds but event publication fails, use an outbox pattern or equivalent reliable publishing strategy.
- Queues improve decoupling, but introduce eventual consistency and operational complexity.

## 12. Kafka vs RabbitMQ

Both are messaging technologies, but their common strengths differ.

| Dimension | Apache Kafka | RabbitMQ |
|---|---|---|
| Core model | Distributed append-only event log with topics and partitions | Message broker with exchanges, queues, and routing |
| Typical fit | Event streams, analytics pipelines, replay, high-throughput fan-out | Task queues, flexible routing, work distribution, request/job messaging |
| Replay | Consumers can often reread retained events by offset | Queue messages are typically removed/acknowledged; replay needs separate design |
| Ordering | Within a partition | Depends on queue and consumer setup; multiple consumers/requeueing can affect order |
| Scaling | Partitioning and consumer groups | Queues, exchanges, clustering/federation patterns; topology matters |
| Key idea | Retained event stream | Broker routes messages to queues |

Neither is universally better. Choose based on delivery semantics, ordering, replay, throughput, routing, latency, operations, and ecosystem.

**Interview shortcut:** Kafka is often a natural fit for durable event streams and replay; RabbitMQ is often a natural fit for routed work queues. Validate against the actual requirements.

## 13. Blob/object storage (S3)

Object storage is designed for files and binary objects: images, videos, documents, backups, and generated artifacts. Store object bytes in S3-like storage and keep metadata/reference/ownership in the database.

### Why not store large files directly in a relational DB?
It can be done, but often increases database size, backup/restore cost, and pressure on database connections and I/O. Object storage separates large binary delivery from transactional metadata.

### Direct upload with a pre-signed URL
1. Client asks API for permission to upload a file.
2. API authenticates/authorizes the user and creates a short-lived pre-signed URL for a specific object and operation.
3. Client uploads directly to object storage.
4. API records metadata after validating upload completion; a storage event or queue can start processing.
5. CDN may serve public/cacheable content; private downloads can use authorization and short-lived URLs.

Example metadata:
```json
{
  "file_id": "file_123",
  "owner_id": "user_42",
  "object_key": "uploads/user_42/file_123.jpg",
  "content_type": "image/jpeg",
  "status": "uploaded"
}
```

Security checklist: private bucket by default, short URL expiry, restrict object key/method/content type where supported, verify size and actual file type, scan untrusted uploads, and avoid exposing secrets. A pre-signed URL is a bearer credential until it expires.

## 14. Rate limiting

Rate limiting protects services from accidental overload, abusive clients, and unfair resource consumption.

### Common algorithms
- **Fixed window:** allow N requests per fixed time window. Simple, but bursts at window boundaries can exceed the intuitive limit.
- **Sliding window/log:** more precise rolling count, with higher state or compute cost depending on implementation.
- **Sliding window counter:** approximate rolling window using current and previous window counts.
- **Token bucket:** tokens refill at a configured rate; requests spend tokens. Supports controlled bursts.
- **Leaky bucket:** smooths output/request rate, often useful to enforce a steady processing pace.

Example policy: allow 100 requests per API key per minute, with a separate stricter limit for expensive endpoints.

### Where to enforce it
- CDN/API gateway for broad edge protection.
- Application for user/tenant-specific business limits.
- Redis can maintain shared counters across multiple app instances.

### Design questions
- Limit by IP, user ID, API key, tenant, or route?
- What happens when the limit is exceeded? Usually return `429 Too Many Requests`, optionally with `Retry-After`.
- How are counters shared across instances?
- What is the policy if the rate-limit store is unavailable?
- Protect against spoofed client IP headers; trust forwarded headers only from known proxies.

## 15. Polling vs WebSockets

### Polling
Client repeatedly sends HTTP requests asking whether anything changed.

- Simple, stateless, and widely compatible.
- Can waste requests when nothing changes and updates may be delayed by the polling interval.
- Use for infrequent updates or when simplicity is more important than immediacy.

### Long polling
Client sends a request and server holds it until an update or timeout, then the client reconnects. Fewer empty responses, but still has request lifecycle and connection management costs.

### WebSockets
Provides a persistent, bidirectional connection after an HTTP upgrade handshake. Either side can send messages over the connection.

- Good for chat, live collaboration, presence, multiplayer state, and real-time dashboards.
- Requires connection management, heartbeats, authentication, reconnect logic, and horizontal fan-out/pub-sub when many application instances are involved.
- Load balancers/proxies must support the connection behavior; long-lived connections consume resources.

| Need | Likely starting point |
|---|---|
| Data changes rarely | Normal HTTP request |
| Updates every 30–60 seconds are acceptable | Polling |
| Updates are sporadic and client needs low delay | Long polling or WebSockets |
| Continuous bidirectional low-latency updates | WebSockets |

WebSockets do not automatically solve message persistence, delivery guarantees, or broadcasting across servers.

## 16. Idempotency

An operation is idempotent when repeating the same operation has the same intended effect as performing it once.

This matters when clients retry after a timeout: the server may have completed the first request even though the response never reached the client.

### Example: payment API
Client sends:
```http
POST /payments
Idempotency-Key: order-987-payment
Content-Type: application/json

{"order_id":"order-987","amount":2500,"currency":"INR"}
```

Server should atomically associate the key with the request/result. If the same key is retried with the same payload, return the original result instead of charging again. If the same key is reused with a different payload, reject it.

Implementation checklist:
- Scope the key to the customer/operation.
- Store request fingerprint, status, and response/result durably.
- Enforce uniqueness atomically (e.g., unique DB constraint); don't rely only on an in-memory check.
- Define retention/expiry based on retry and business windows.
- Handle concurrent duplicate requests and in-progress status.
- Make downstream side effects idempotent too where possible.

HTTP methods such as PUT and DELETE are defined with idempotent semantics, but a POST endpoint can also implement idempotency with a key. Idempotency is not the same as exactly-once delivery.

## 17. CAP theorem

CAP describes a distributed data system under a network partition: it cannot guarantee both **linearizable consistency (C)** and **availability (A)** for every request while also tolerating that partition (**P**).

- **Consistency:** every successful read observes the latest successful write according to the chosen strong consistency model.
- **Availability:** every request to a non-failing node receives a non-error response, though the response may not reflect the latest write.
- **Partition tolerance:** the system continues operating despite communication failures between nodes.

When a partition occurs, a design may prefer:
- **CP behavior:** reject, delay, or restrict some operations to preserve strong consistency.
- **AP behavior:** keep responding and allow temporary divergence/stale results, reconciling later.

This is a simplification for interviews; real systems can make different choices per operation, and consistency models are more nuanced than a single label.

**Example:** a social media view count may tolerate temporary stale reads; a bank transfer or inventory reservation may require stronger coordination to avoid incorrect outcomes.

Do not say “choose any two of three” without mentioning the partition scenario. In a distributed system, partitions are a reality to design for.

## 18. End-to-end scalable architecture

Imagine designing a photo-sharing service.

```text
Client
  |
  v
DNS -> CDN / Edge protection (static and cacheable content)
  |
  v
Highly available Load Balancer / API Gateway
  |
  v
Stateless API instances
  |             \
  |              +--> Redis cache (hot, safe-to-cache reads)
  |
  +--> Primary DB for writes / read replicas for eligible reads
  |         |
  |         +--> Shards when one DB cannot handle data/throughput
  |
  +--> Durable event/outbox -> Message broker -> Worker pools
  |                                      |--> thumbnail/image processing
  |                                      |--> email/notifications
  |                                      +--> analytics
  |
  +--> Object storage (uploads/downloads via short-lived pre-signed URLs)
```

### Request walkthrough: upload a photo
1. Client resolves the service hostname through DNS and connects over HTTPS.
2. Client requests an upload authorization from the API.
3. API checks identity, tenant permissions, file constraints, and rate limits.
4. API returns a short-lived pre-signed URL; client uploads bytes directly to object storage.
5. Upload completion is verified. Metadata is recorded in the database.
6. A reliable event is published (often through a transactional outbox) to the queue/broker.
7. Workers resize the image, generate thumbnails, and send notifications independently.
8. Worker retries handle transient errors; idempotency prevents duplicate side effects.
9. CDN delivers eligible images/derivatives near users. Private content uses authorization-aware delivery.
10. Metrics and alerts track latency, error rate, queue lag, cache hit ratio, database saturation, and object processing failures.

### Why each component exists

| Problem | Component/pattern | What it improves | Trade-off |
|---|---|---|---|
| Hostname must resolve | DNS | Discovery | Cached answers may take time to update |
| Too much traffic for one app server | Horizontal scale + LB | Throughput and instance resilience | More operations and shared-state concerns |
| Unhealthy instance receives requests | Health checks | Availability | Detection is not instantaneous |
| Distant users have slow static delivery | CDN | Latency and origin bandwidth | Invalidation and cache policy complexity |
| Repeated reads overload DB | Redis/cache | Read latency and DB load | Staleness and cache failure modes |
| Slow query scans many rows | Index | Query efficiency | Storage and write overhead |
| Too much data on one DB | Sharding | Capacity and distributed load | Cross-shard complexity |
| DB node failure/read pressure | Replication | Read scale and failover options | Lag and failover complexity |
| Slow secondary work blocks response | Queue + workers | Async processing and buffering | Eventual consistency, retries, duplicates |
| Large binary payloads burden app/DB | Object storage | Durable, scalable file storage | Access control and lifecycle management |
| Abuse or traffic spikes | Rate limiter | Fairness and overload protection | False positives and shared counter overhead |
| Need live updates | WebSockets/pub-sub | Low-latency two-way updates | Connection/fan-out complexity |
| Client retries duplicate a side effect | Idempotency key | Safe retries | Durable key tracking and retention |

### Important caveats
- A queue acknowledgement should not be confused with a completed business workflow. Return a job/status ID if work continues asynchronously.
- Use a database transaction/outbox or another reliable publishing pattern to avoid losing events between a DB write and a broker publish.
- Replication does not guarantee zero data loss; understand acknowledgement policy, failover, and recovery objectives.
- Don't shard early. Indexing, query improvements, vertical scaling, and read replicas may be sufficient first.
- Keep private files private. CDN caching and pre-signed URLs must follow the authorization model.
- Add observability from the beginning: structured logs, metrics, tracing, SLOs, dashboards, and alerts.

## 19. Trade-off cheat sheet

| Concept A | Concept B | Main distinction |
|---|---|---|
| Vertical scaling | Horizontal scaling | Bigger machine vs more machines |
| L4 LB | L7 LB | Transport routing vs HTTP-aware routing |
| CDN | Redis | Edge delivery for content vs application/data cache |
| Indexing | Sharding | Faster access within a DB vs partitioning data across DB nodes |
| Replication | Sharding | Copies of data vs subsets of data |
| Cache-aside | Write-through | Load on cache miss vs keep cache updated in write path |
| Kafka | RabbitMQ | Retained partitioned event stream vs flexible broker/queue routing |
| Polling | WebSockets | Repeated client requests vs persistent bidirectional connection |
| Synchronous work | Queue + workers | Wait for work vs acknowledge and process asynchronously |
| Idempotency | Exactly-once processing | Safe repeated effect vs much broader end-to-end delivery/processing guarantee |

## 20. Interview revision checklist

Before you finish a design, make sure you can answer:

- [ ] What are the main functional and non-functional requirements?
- [ ] What is the estimated read/write traffic, payload size, and data growth?
- [ ] What is the request flow from DNS to the backend and response?
- [ ] What is the first bottleneck, and why is the chosen component appropriate?
- [ ] Which data can be cached, for how long, and how is it invalidated?
- [ ] What are the index and query patterns? Do we actually need sharding?
- [ ] What consistency guarantees do reads need after writes?
- [ ] What happens when a server, database node, cache, broker, or worker fails?
- [ ] Are retries safe? Are consumers idempotent? Is there a dead-letter path?
- [ ] How are large/private files uploaded and delivered securely?
- [ ] What is the rate-limit policy and its failure behavior?
- [ ] What are the key metrics, SLOs, alerts, and recovery objectives?
- [ ] What are the costs and trade-offs of each component?

### A concise answer framework
**Requirement → bottleneck → design choice → request/data flow → failure handling → trade-offs → metrics.**

If you can explain not only *what* a component is but *which problem made you add it*, you have the right mental model for system design interviews.
