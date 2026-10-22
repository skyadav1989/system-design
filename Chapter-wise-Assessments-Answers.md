# System Design Bootcamp --- Chapter-wise Assessments & Model Answers

## Days 1--7 \| All 24 Lessons

This file collects the assessment questions that were presented during
the bootcamp and provides model answers. Some assessments were skipped
when moving to the next lesson; the answers below are study references,
not claims about the user's submitted answers or scores.

------------------------------------------------------------------------

# Day 1 --- Foundations

## Lesson 1: System Design Fundamentals

### Assessment question

A growing e-commerce store is experiencing slow product pages and
occasional checkout failures. How would you start designing an improved
system?

### Model answer

1.  **Clarify requirements:** Identify affected journeys, peak traffic,
    latency/availability targets, data correctness, and business
    priorities.
2.  **Measure first:** Collect p95/p99 latency, error rates, throughput,
    traces, and database/cache metrics.
3.  **Map the architecture:** Client/CDN → load balancer → application →
    cache/database/search/queues → external services.
4.  **Find bottlenecks:** Determine whether latency comes from
    application CPU, database queries, cache misses, search, or external
    calls.
5.  **Improve incrementally:** Optimize queries, cache safe reads, use a
    CDN for public assets, move suitable slow work to queues, and scale
    the bottleneck.
6.  **Protect correctness:** Keep order/payment/inventory invariants
    transactional or enforce them with explicit idempotency and
    reservation logic.
7.  **Validate:** Load-test changes and monitor SLOs.

## Lesson 2: Latency, Throughput, Availability & Capacity

### Assessment question

A service receives 600 requests/second at peak. Average response time is
250 ms. Each application server can sustain 100 requests/second at the
target latency. Calculate concurrency and the theoretical minimum server
count. Why provision extra capacity?

### Model answer

-   Little's Law: `L = λ × W = 600 × 0.25 = 150` requests in flight on
    average.
-   Theoretical minimum servers: `600 / 100 = 6`.
-   Six servers provide no headroom. Provision additional capacity for
    bursts, deployments, uneven traffic, and instance failures.
-   Validate with realistic load tests; sustainable throughput depends
    on request mix and downstream capacity.

## Lesson 3: Requirements & Problem Scoping

### Assessment question

Before designing a new e-commerce API, what questions would you ask, and
how would you proceed if some requirements were unknown?

### Model answer

-   Clarify users, core operations, read/write mix, peak RPS, data
    volume/growth, latency, availability, consistency,
    security/compliance, retention, and RTO/RPO.
-   Separate must-have requirements from optional features.
-   State assumptions when answers are unavailable and explain their
    impact.
-   Design around business-critical flows and measurable SLOs; validate
    assumptions with stakeholders and load tests.

## Lesson 4: High-Level Architecture & Core Components

### Assessment question

Design the high-level flow for a product page and an order submission.
Which operations should be synchronous and which asynchronous?

### Model answer

-   Product page: client → CDN/cache → load balancer → application →
    cache/search/database as required.
-   Checkout: client → application/API → validate customer/cart/pricing
    → reserve inventory and create order transactionally as appropriate
    → initiate payment according to the payment workflow.
-   Keep user-critical decisions synchronous when the user needs an
    immediate result.
-   Use queues for notifications, search indexing, analytics, and ERP
    synchronization that can complete asynchronously.
-   Add timeouts, idempotency, retries, monitoring, and explicit
    pending/failure states.

------------------------------------------------------------------------

# Day 2 --- Data, Replication & Consistency

## Lesson 5: SQL vs. NoSQL & Data Modeling

### Assessment question

An e-commerce platform needs reliable orders and payments, flexible
product attributes, and high-volume session/cache lookups. Which storage
technologies would you choose and why?

### Model answer

-   Use a relational database for orders, payments, and transactional
    inventory because constraints and transactions protect business
    invariants.
-   Use a document or search-oriented store only where flexible
    attributes or search access patterns justify it; keep a clear source
    of truth.
-   Use Redis or another key-value store for cache/session workloads
    where appropriate.
-   Model data around access patterns, use indexes and unique
    constraints, and avoid choosing NoSQL simply because the system is
    expected to grow.
-   If data is denormalized, define update propagation and
    reconciliation.

## Lesson 6: Replication, Partitioning & Sharding

### Assessment question

A single database is approaching its write and storage limits. Explain
replication vs. sharding, the risks of each, and how you would decide
whether to shard.

### Model answer

-   Replication keeps copies of the same data on multiple nodes,
    improving read scaling or failover options.
-   Sharding splits different data subsets across nodes, increasing
    total storage/write capacity.
-   Replication risks include lag and stale reads; sharding risks
    include cross-shard joins/transactions, hot partitions, global
    uniqueness, and rebalancing complexity.
-   First optimize queries, indexes, schema, and workload. Consider
    sharding only after measurement shows a single node is the
    bottleneck.
-   Choose a high-cardinality, well-distributed shard key aligned with
    access patterns and plan rebalancing.

## Lesson 7: Consistency, Transactions & CAP

### Assessment question

Two customers try to buy the last unit of a product. Different replicas
show different stock quantities. How would you prevent overselling, and
how does CAP affect the design?

### Model answer

-   Do not rely on a possibly stale replica or cache for the final stock
    decision.
-   Use an atomic conditional update or inventory reservation against
    the authoritative store, for example updating stock only when
    available quantity is sufficient.
-   Make reservation/order operations idempotent and enforce
    uniqueness/constraints.
-   Strong consistency can increase latency or reject/wait for
    operations during partitions; eventual consistency improves some
    availability/latency trade-offs but may expose stale stock.
-   CAP's key trade-off applies during a network partition: linearizable
    consistency and successful responses from every partition side
    cannot both be guaranteed.
-   Reconcile asynchronous inventory views.

------------------------------------------------------------------------

# Day 3 --- Performance & Traffic Management

## Lesson 8: Caching & Performance

### Assessment question

A product page is slow because the database repeatedly loads the same
product information. Design a cache strategy and explain how to prevent
stale data and cache stampedes.

### Model answer

-   Use cache-aside: read cache, fetch from the source on miss, populate
    the cache.
-   Choose keys containing every response-changing dimension, such as
    store, currency, locale, and customer group where relevant.
-   Use TTL plus explicit invalidation for important updates.
-   Prevent stampedes with request coalescing, locking, early refresh,
    TTL jitter, or stale-while-revalidate when stale data is safe.
-   Monitor hit ratio, eviction, cache latency, origin load, and
    stale-data incidents.

## Lesson 9: CDN & Static Content Delivery

### Assessment question

After a deployment, a CDN cache purge causes origin traffic to spike and
some customers see stale or incorrect content. How would you fix the
design?

### Model answer

-   Use fingerprinted/versioned asset filenames and long TTLs for
    immutable static assets.
-   Purge selectively instead of invalidating everything.
-   Configure correct `Cache-Control`, `Vary`, and cache-key dimensions.
-   Never accidentally share personalized cart/account responses across
    users.
-   Use origin shielding, request collapsing, prewarming where useful,
    and capacity protection.
-   Monitor cache-hit ratio, origin RPS, miss rate, and response
    variants for logged-in vs. anonymous users.

## Lesson 10: Pagination & Query Optimization

### Assessment question

A product API has millions of records. Large offset pages are slow and
records occasionally repeat or disappear as products change. What would
you do?

### Model answer

-   Prefer cursor/keyset pagination for large, frequently changing
    datasets.
-   Use a stable indexed order with a unique tie-breaker, such as
    `(created_at, entity_id)`.
-   Offset pagination is acceptable for small datasets or when arbitrary
    page jumps are important, but large offsets may be expensive.
-   Inspect execution plans, add indexes matching filters and sorting,
    select only required columns, and avoid N+1 queries.
-   Bound page sizes and measure query latency before and after changes.

## Lesson 11: Rate Limiting & Throttling

### Assessment question

An API receives sudden bursts from clients and one tenant can overwhelm
the service. Design a rate-limiting strategy.

### Model answer

-   Apply limits by tenant/API key/user and separately protect expensive
    endpoints.
-   Token bucket is useful when controlled bursts are allowed; fixed
    window is simpler but can permit boundary bursts; sliding window
    smooths behavior at greater implementation cost.
-   Return HTTP 429 and `Retry-After` where useful.
-   Use distributed counters or a gateway where needed; account for
    users behind shared IPs.
-   Monitor throttling, latency, rejected requests, and per-tenant
    usage; keep emergency overload controls distinct from contractual
    quotas.

## Lesson 12: Load Balancing & High Availability

### Assessment question

Three application nodes sit behind a load balancer. One node becomes
unhealthy, sessions are stored locally, and the load balancer's health
checks only test whether the process is running. What should change?

### Model answer

-   Use meaningful readiness checks, not only process liveness.
-   Remove unhealthy nodes from traffic and use redundant load-balancing
    infrastructure.
-   Externalize sessions and shared state or deliberately justify
    session affinity; local sessions impede failover.
-   Use graceful shutdown and connection draining during deployment.
-   Configure safe retry behavior; do not blindly retry non-idempotent
    requests.
-   Test node and load-balancer failover.

------------------------------------------------------------------------

# Day 4 --- Queues, Events & Resilience

## Lesson 13: Message Queues --- RabbitMQ & Kafka

### Assessment question

An order event is published to a queue, but a consumer may crash after
updating the database and before acknowledging the message. How do you
prevent message loss and duplicate business effects?

### Model answer

-   Use durable messaging and suitable publisher
    confirms/acknowledgments.
-   Acknowledge only after the intended database effect commits.
-   Expect redelivery if the consumer crashes before acknowledgment.
-   Give each event a stable ID and make processing idempotent, backed
    by a unique constraint or processed-event record in the same
    transaction as the business change.
-   Monitor consumer lag, queue age, failures, and redelivery rates.

## Lesson 14: Event-Driven Architecture & Background Workers

### Assessment question

Magento reserves inventory successfully, but payment authorization fails
permanently. Why is a distributed transaction difficult here, and how
should a saga handle the failure?

### Model answer

-   The database and payment provider are independent systems and
    usually cannot participate in one atomic ACID transaction.
-   Model the workflow as a saga with durable state and idempotent
    steps.
-   If payment fails permanently, execute a compensating action such as
    releasing the inventory reservation.
-   Record each transition, make compensation retryable/idempotent, and
    reconcile incomplete workflows.
-   Compensation is not a perfect rollback; it is a new business
    operation that restores an acceptable state.

## Lesson 15: Retries, Idempotency & Dead-Letter Queues

### Q1. SAP creates an order but Magento receives a timeout. What should happen?

**Model answer:** Treat the outcome as unknown. Query SAP using a stable
order ID/idempotency key before retrying. If retrying, reuse the same
key. SAP should enforce the key atomically with order creation and
return the original result for duplicates.

### Q2. A thousand workers retry SAP every second while SAP is recovering. What is the failure pattern and how do you prevent it?

**Model answer:** This is a retry storm/thundering herd: synchronized
retries overwhelm the recovering dependency. Use exponential backoff
with jitter, bounded retries, circuit breakers, rate limits, and
concurrency limits. Respect `Retry-After` where supplied.

### Q3. How should the worker handle these responses?

-   **HTTP 429 with `Retry-After: 30`:** Retry after the specified
    delay, with jitter if appropriate.
-   **HTTP 400 invalid tax ID:** Stop automatic retries; route for
    correction.
-   **HTTP 503:** Retry with bounded backoff and jitter, respecting
    `Retry-After`.
-   **Timeout with unknown outcome:** Reconcile by order ID/idempotency
    key before retrying.
-   **Duplicate prevention:** Enforce a stable idempotency key
    atomically in the downstream system.

### Q4. What is a dead-letter queue and how should replay work?

**Model answer:** A DLQ holds messages that exhaust the retry policy or
cannot be processed. Monitor count and age, inspect and correct the
cause, validate payload/schema, and replay in a controlled way with
idempotency protection. Do not blindly replay poison messages.

### Q5. Design a reliable Magento-to-SAP order integration.

**Model answer:** Commit the Magento order and outbox event in one
database transaction; publish through a durable broker with confirms;
consume with a stable order ID/idempotency key; retry transient failures
with bounded backoff/jitter; reconcile ambiguous timeouts; send
exhausted messages to a monitored DLQ; and monitor queue age, SAP
latency/errors, retries, and duplicates.

## Lesson 16: Distributed Locks, Leader Election & Coordination

### Assessment question

A Magento cron obtains a Redis lock with a 60-second TTL, but the job
runs for 90 seconds. Another server acquires the lock and starts a
second job. Why did this happen, and how do you prevent stale writes?

### Model answer

-   The lease expired while the first worker was still running, so a
    second worker could acquire the lock.
-   Possible mitigations: choose/renew a lease safely, split the job
    into bounded units, and make operations idempotent.
-   Release the lock only if the unique ownership token still matches.
-   Use a monotonically increasing fencing token or version check that
    the protected resource validates, rejecting writes from stale
    owners.
-   A lock alone does not guarantee exactly-once effects.

## Lesson 17: Fault Tolerance & Disaster Recovery

### Assessment question

Magento runs in Region A with a replicated standby in Region B. Region A
fails; the standby is three minutes behind, but the RPO is one minute.
The old primary's status is uncertain and RabbitMQ contains pending
order events. What do you do?

### Model answer

-   Do not claim the RPO is met: the standby is behind the one-minute
    target, so promotion may lose recent writes.
-   Follow the incident/recovery policy and quantify the business
    impact; preserve the old primary's data if possible.
-   Fence or isolate the old primary before allowing the standby to
    accept writes, preventing split brain.
-   Promote only after a controlled decision; validate data and
    application behavior.
-   Process queued events idempotently and reconcile order IDs against
    Magento and SAP to avoid duplicate/missing orders.
-   Communicate the recovery trade-off and measure actual RTO/RPO.

## Lesson 18: Quorum Reads/Writes & Advanced Consistency

### Assessment question

There are three replicas (`N=3`). A write succeeds on A and B (`W=2`),
but a read from C returns stale inventory. What is the minimum R
satisfying `R + W > N`, and is overlap enough to prevent overselling?

### Model answer

-   Need `R + 2 > 3`, so `R > 1`; minimum integer `R = 2`.
-   A read from C alone can be stale because C has not received the
    write.
-   Quorum overlap alone does not guarantee linearizability; the
    database's protocol and version/conflict resolution matter.
-   Prevent overselling with an atomic conditional stock update or
    reservation at the authoritative inventory service, idempotent order
    handling, and reconciliation.

------------------------------------------------------------------------

# Day 5 --- Coordination & Consistency Review

## Review assessment: Choose the right mechanism

### Question

For each requirement, choose the primary mechanism and explain why: 1.
Prevent two cron jobs from running the same critical task concurrently.
2. Deliver order events durably to background consumers. 3. Prevent
duplicate order creation after a client timeout. 4. Preserve recoverable
historical data after accidental deletion. 5. Prevent a stale former
lock owner from overwriting new state.

### Model answer

1.  **Distributed lock/coordination**, with leases and careful
    ownership.
2.  **Durable message queue or event log**, plus consumer
    acknowledgments and idempotency.
3.  **Idempotency key** enforced atomically with the business operation.
4.  **Backups and point-in-time recovery**; replication alone may copy
    deletion.
5.  **Fencing token or resource-enforced version check**; a lock by
    itself is insufficient.

------------------------------------------------------------------------

# Day 6 --- Networking, APIs & Security

## Lesson 19: DNS, TCP, TLS & HTTP

### Assessment question

A customer submits `POST /orders`. Magento creates the order, but the
connection drops before the response arrives. The customer retries.
Also, a DNS record has just changed, but some users still reach the old
server. Explain both issues.

### Model answer

-   TCP confirms reliable byte transfer, not whether the business
    operation committed. The order may exist despite the missing
    response.
-   Use a stable idempotency key tied to the logical checkout/order
    attempt and atomically store the key/outcome with order creation;
    return the existing result on retry.
-   DNS answers are cached by clients and recursive resolvers according
    to TTL and implementation behavior. Lower TTL before planned
    migration, verify routing, and allow cached answers to expire; DNS
    change is not instant global invalidation.

## Lesson 20: API Design --- REST, GraphQL, Versioning & Pagination

### Assessment question

Magento receives `POST /api/v1/orders` with key `order-xyz`; it creates
order 50123, but the response times out. A retry arrives with the same
key, followed by a request with the same key but a different cart
payload. What should happen, and how should storage be designed?

### Model answer

-   Same key and same logical request: return the original order/result
    rather than creating another order.
-   Same key with a different payload: reject with a clear
    conflict/validation error and log it; do not silently reinterpret
    the key.
-   Store a scoped unique key, request hash, status, and result
    reference.
-   Enforce uniqueness and create the order in one database transaction
    or equivalent atomic workflow so concurrent requests cannot both
    win.
-   Consider recovery for an in-progress operation and never rely only
    on an expiring cache entry.

## Lesson 21: Authentication, Authorization & API Security

### Assessment question

A Store A administrator has a valid JWT with `orders:read` and requests
an order belonging to Store B. What must the Magento MCP gateway check,
and how do you avoid cross-tenant leakage?

### Model answer

-   JWT signature and scope are not enough: the gateway must also
    validate issuer, audience, expiry, identity-to-tenant mapping, and
    resource ownership.
-   Derive tenant context from trusted identity/credential mapping, not
    from an untrusted request parameter alone.
-   Verify the order belongs to the caller's tenant before returning it.
-   Scope cache keys and authorization decisions by tenant; isolate
    tenant credentials and data access.
-   Audit access and test cross-tenant negative cases.

------------------------------------------------------------------------

# Day 7 --- Scalability, Observability & Reliability

## Lesson 22: Scalability & Capacity Planning

### Assessment question

A Magento store has 600 peak backend requests/second, 250 ms average
response time, and each app server can sustain 100 requests/second at
the target latency. Calculate concurrency and minimum servers, and
explain why extra capacity is needed.

### Model answer

-   Concurrency: `600 × 0.25 = 150` average requests in flight.
-   Theoretical minimum: `600 / 100 = 6` servers.
-   Add headroom for bursts, uneven distribution, maintenance,
    deployments, and a node failure.
-   Load-test realistic traffic and verify that MySQL, Redis,
    OpenSearch, and SAP can support the load. More application nodes do
    not fix a downstream bottleneck.

## Lesson 23: Observability --- Logs, Metrics, Tracing, SLI/SLO/SLA

### Assessment question

At 10:00 AM, CPU is 35%, checkout p95 is 4 seconds (normally 500 ms),
5xx is 8% (normally below 1%), RabbitMQ queue depth is growing, and SAP
latency is high. What do you investigate, and what alerts/SLO actions do
you take?

### Model answer

-   CPU alone does not diagnose the issue; requests may be waiting on
    SAP or queued behind slow dependency calls.
-   Inspect traces from checkout through Magento and SAP; correlate logs
    using trace/request/order IDs; inspect SAP latency/error codes,
    PHP-FPM busy/queued workers, database waits, and queue
    arrival/completion rates.
-   Alert on checkout success-rate/SLO burn and queue oldest-message age
    or SAP dependency errors.
-   Apply incident response: protect the dependency with circuit
    breaker/concurrency/rate limits, bound retries, and preserve safe
    checkout behavior.
-   Use error-budget burn to prioritize response; pause risky releases
    or nonessential work when the reliability budget is being consumed.

## Lesson 24: Reliability, Failover & Disaster Recovery

### Assessment question

A region fails. The standby database is three minutes behind when the
RPO is one minute; the old primary may still be active, and RabbitMQ
contains pending order events. Should you promote, how do you avoid
split brain, and how do you process the backlog?

### Model answer

-   The standby does not meet the RPO; quantify and communicate possible
    data loss and follow the recovery policy rather than claiming
    compliance.
-   Fence/isolate the old primary before promoting a new writer.
-   Validate recovered database state and dependent services before
    routing full traffic.
-   Consume messages idempotently, compare order IDs and downstream SAP
    status, and reconcile ambiguous operations before replay.
-   Record actual recovery time/data loss and improve the runbook after
    the incident.

------------------------------------------------------------------------

# Additional Architecture Challenge: URL Shortener

### Question

Design a URL shortener serving 10 million redirects/day and 100,000
creations/day. Explain average redirect RPS, code generation,
uniqueness, and caching.

### Model answer

-   Average redirects RPS: `10,000,000 / 86,400 ≈ 115.7 RPS`. Peak
    traffic can be much higher; estimate using measured peak-to-average
    factors.
-   Use `POST /urls` to create a mapping and `GET /{code}` to redirect.
-   Store `short_code → original_url` durably with a unique constraint
    and an index on `short_code`.
-   Generate codes using a unique numeric ID encoded in Base62, or
    random codes with collision detection and retry. Six Base62
    characters yield `62^6 ≈ 56.8 billion` possible codes.
-   Cache hot mappings in Redis; the database remains the durable source
    of truth.
-   Use 302 for temporary redirects when destination changes or cache
    control matters; 301 may be cached more aggressively.
-   Add expiration, rate limits, destination validation, abuse
    protection, and asynchronous analytics.

------------------------------------------------------------------------

# Final Assessment Review --- High-Value Rules

1.  **Timeout ≠ failure:** reconcile before retrying uncertain side
    effects.
2.  **Retries:** bounded exponential backoff + jitter; respect
    `Retry-After`.
3.  **Idempotency:** stable key, durable atomic enforcement, payload
    mismatch detection.
4.  **Queues:** expect duplicate delivery; use durable publishing,
    acknowledgments, and idempotent consumers.
5.  **DLQ:** monitor, diagnose, fix, then replay carefully.
6.  **Distributed locks:** lease/token ownership plus fencing/version
    checks for stale writers.
7.  **Replication:** helps failover but can copy corruption; maintain
    tested backups.
8.  **RTO/RPO:** define with business stakeholders and test recovery.
9.  **Quorum:** overlap formulas are not a complete consistency
    guarantee.
10. **Caching/CDN:** correct cache keys and invalidation prevent stale
    or cross-user data.
11. **Capacity:** use measured peak workload and load tests; protect
    downstream dependencies.
12. **Observability:** combine symptoms (metrics), context (logs), and
    request paths (traces).
13. **Security:** authenticate, authorize per tenant/resource, and
    isolate credentials/cache.
14. **System design interview:** requirements → estimates → APIs/data →
    architecture → bottlenecks/trade-offs → security/observability →
    failure/recovery.
