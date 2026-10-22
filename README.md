# FAANG System Design Bootcamp --- Days 1--7

## Quick Revision Notes, Key Points, Trade-offs, and Mitigations

> Purpose: A practical revision guide for system design interviews,
> using e-commerce and Magento-style examples.
>
> **How to use:** For each design question, clarify requirements,
> estimate scale, draw the high-level architecture, identify bottlenecks
> and failure modes, compare trade-offs, then explain monitoring and
> recovery.

------------------------------------------------------------------------

# Day 1 --- Foundations

## Lesson 1: System Design Fundamentals

### Key points

-   System design defines components, responsibilities, interfaces, data
    flow, and operational behavior.
-   Start with **functional requirements** (what the system does) and
    **non-functional requirements** (latency, availability, consistency,
    security, scalability, cost).
-   Identify users, core workflows, APIs, data stores, external
    dependencies, and failure cases.
-   A common high-level flow: client → CDN/load balancer → application
    services → cache/database/queue → external services.
-   Distinguish architectural decisions from implementation details.

### Advantages

-   Makes complexity manageable by separating concerns.
-   Surfaces bottlenecks and risks before implementation.
-   Helps teams reason about scale, failure, and operational ownership.

### Limitations / risks

-   Overdesigning before requirements are clear wastes time and
    increases complexity.
-   A diagram can hide important details such as data consistency,
    retries, and failure handling.
-   A design that works at small scale may fail under realistic traffic.

### Ways to overcome limitations

-   State assumptions explicitly and validate them with stakeholders.
-   Begin with the simplest design that meets requirements; evolve it
    when evidence supports change.
-   Include data flow, failure behavior, security boundaries, and
    observability---not only boxes and arrows.

### Interview checklist

1.  Clarify scope and exclusions.
2.  Estimate traffic and data volume.
3.  Define APIs and data model.
4.  Draw the architecture.
5.  Explain bottlenecks, trade-offs, and failure recovery.

## Lesson 2: Latency, Throughput, Availability, and Capacity Estimation

### Key points

-   **Latency:** time taken by an operation.
-   **Throughput:** operations processed per unit time.
-   **Availability:** proportion of time a service is usable.
-   **Capacity:** load a system can sustain within its performance
    targets.
-   Measure p50, p95, and p99 latency; averages can hide slow requests.
-   Little's Law: `L = λ × W`, where `L` is average work in the system,
    `λ` is arrival rate, and `W` is average time in the system.
-   Estimate average request rate as daily requests / 86,400; estimate
    peak traffic using observed peak-to-average ratios.

### Advantages of estimation

-   Gives an initial basis for server, database, cache, and queue
    sizing.
-   Makes assumptions visible and helps compare options.

### Limitations / risks

-   Estimates can be misleading if request mix, cache hit rate,
    burstiness, or dependency latency is ignored.
-   A theoretical server count is not a substitute for load testing.

### Ways to overcome limitations

-   Use production telemetry and realistic load tests.
-   Include peak traffic, retries, background jobs, headroom, and
    failure scenarios.
-   Revisit assumptions as the workload changes.

## Lesson 3: Requirements and Problem Scoping

### Key points

-   Separate must-have features from optional features.
-   Define users, critical journeys, data ownership, and business rules.
-   Ask about scale, geographic reach, compliance, latency,
    availability, retention, and recovery targets.
-   Clarify whether the system needs real-time behavior, eventual
    consistency, or strict correctness.
-   Define measurable SLIs and targets early.

### Advantages

-   Prevents building the wrong system.
-   Makes scope and trade-offs explicit.

### Limitations / risks

-   Vague requirements lead to untestable designs.
-   Too many questions can consume interview time.

### Ways to overcome limitations

-   Ask the highest-impact questions first.
-   State reasonable assumptions when details are unavailable.
-   Prioritize requirements by business impact and risk.

## Lesson 4: High-Level Architecture and Core Components

### Key points

-   Common components: clients, CDN, load balancer, application
    services, cache, database, search, message broker, object storage,
    and observability.
-   Keep responsibilities clear and define communication patterns.
-   Use synchronous requests for immediate responses and asynchronous
    messaging for work that can be completed later.
-   Design trust boundaries and failure behavior between components.

### Advantages

-   Separates concerns and allows independent scaling.
-   Queues and caches can reduce load on critical services.

### Limitations / risks

-   Every additional service creates network, deployment, monitoring,
    and failure complexity.
-   Microservices can add operational overhead and distributed
    consistency problems.

### Ways to overcome limitations

-   Start with a modular monolith when it is sufficient.
-   Split services around clear ownership and scaling needs.
-   Define timeouts, contracts, versioning, and observability for
    service boundaries.

------------------------------------------------------------------------

# Day 2 --- Data and Consistency

## Lesson 5: SQL vs. NoSQL and Data Modeling

### Key points

-   **SQL:** relational structure, joins, constraints, and transactions;
    good for orders, payments, and inventory transactions.
-   **NoSQL:** flexible data models and workload-specific scaling;
    options include document, key-value, wide-column, and graph
    databases.
-   Model data around access patterns and business invariants.
-   Use primary keys, indexes, uniqueness constraints, and appropriate
    normalization.
-   Denormalization can improve reads but introduces update consistency
    work.

### SQL advantages

-   Strong transactional capabilities and constraints.
-   Flexible queries and relationships.

### SQL disadvantages

-   Scaling write-heavy workloads across shards can be complex.
-   Poor indexes and large joins can create bottlenecks.

### NoSQL advantages

-   Can fit high-volume key-based or document-oriented access patterns.
-   Some systems offer straightforward horizontal partitioning.

### NoSQL disadvantages

-   Query flexibility and transaction semantics vary by product.
-   Duplicated data and eventual consistency can complicate updates.

### Ways to overcome limitations

-   Choose by access pattern, correctness requirements, operational
    expertise, and scale---not fashion.
-   Add indexes based on measured queries.
-   Use transactional constraints for critical invariants.
-   If denormalizing, define how updates propagate and how
    inconsistencies are repaired.

## Lesson 6: Replication, Partitioning, and Sharding

### Key points

-   **Replication:** copies data to multiple nodes for availability
    and/or read scaling.
-   **Partitioning:** splits data into manageable subsets.
-   **Sharding:** distributes partitions across independent database
    nodes.
-   Choose a shard key with good cardinality and balanced distribution.
-   Consider hot keys, cross-shard queries, rebalancing, and shard
    growth.
-   Replicas can lag behind the primary.

### Advantages

-   Replication can improve read throughput and failover options.
-   Sharding can increase storage and write capacity beyond one node.

### Disadvantages / risks

-   Replication lag can cause stale reads.
-   Sharding complicates transactions, joins, global uniqueness,
    migrations, and operations.
-   A poor shard key creates hot partitions and uneven load.

### Ways to overcome limitations

-   Pick shard keys from real access patterns and distribution data.
-   Monitor replication lag and route correctness-sensitive reads
    appropriately.
-   Use stable IDs and carefully plan rebalancing.
-   Keep cross-shard transactions rare; use workflows, sagas, or
    reconciliation when appropriate.

## Lesson 7: Consistency, Transactions, and CAP

### Key points

-   **Strong/linearizable consistency:** reads observe the latest
    completed write according to the system's guarantees.
-   **Eventual consistency:** replicas may temporarily disagree but
    converge if updates stop and propagation succeeds.
-   **ACID:** atomicity, consistency, isolation, durability.
-   **CAP:** during a network partition, a distributed system cannot
    guarantee both linearizable consistency and successful responses
    from every side of the partition.
-   CAP is not a claim that systems simply choose any two all the time;
    partition behavior is the key trade-off.
-   Inventory, payment, and order state need explicit correctness rules.

### Strong consistency advantages

-   Easier reasoning about critical state.
-   Reduces stale-read risks.

### Strong consistency disadvantages

-   Coordination can increase latency or reduce availability during
    partitions.

### Eventual consistency advantages

-   Can support responsive reads and continued operation in some
    partition scenarios.
-   Useful for catalogs, analytics, counters, and derived views where
    temporary staleness is acceptable.

### Eventual consistency disadvantages

-   Clients may see stale or conflicting values.
-   Requires conflict resolution, read repair, or reconciliation.

### Ways to overcome limitations

-   Choose consistency per operation, not globally.
-   Use atomic database updates and constraints for critical invariants.
-   Make asynchronous propagation observable and repairable.
-   Define how conflicts are resolved and how clients behave when data
    is stale.

------------------------------------------------------------------------

# Day 3 --- Performance and Traffic Management

## Lesson 8: Caching and Performance

### Key points

-   Cache hot, expensive, frequently requested data.
-   Common layers: browser, CDN, reverse proxy, application, Redis, and
    database buffer cache.
-   Cache-aside pattern: read cache; on miss read source, then populate
    cache.
-   Define TTL, invalidation, cache key, and stale-data policy.
-   Watch hit ratio, eviction rate, memory, latency, and origin load.
-   Cache keys must include every dimension that changes the response,
    such as store, currency, customer group, locale, or query parameters
    where relevant.

### Advantages

-   Reduces latency and database load.
-   Improves throughput for repeated reads.

### Disadvantages / risks

-   Stale data and difficult invalidation.
-   Cache stampedes after expiration.
-   Memory cost and cache-key mistakes can expose incorrect or
    personalized data.

### Ways to overcome limitations

-   Use explicit invalidation or bounded TTLs.
-   Use request coalescing, locks, early refresh, or jittered TTLs to
    reduce stampedes.
-   Cache only safe response variants; include relevant dimensions in
    keys.
-   Use stale-while-revalidate only where business rules permit it.

## Lesson 9: CDN and Static Content Delivery

### Key points

-   A CDN serves cacheable content from edge locations closer to users.
-   Good candidates: versioned JS/CSS, images, fonts, and public static
    assets.
-   Cache-control headers and asset versioning govern freshness.
-   Dynamic and personalized content needs careful cache policy.
-   Cache keys and invalidation rules are critical.

### Advantages

-   Lower latency and origin traffic.
-   Better performance during traffic spikes.
-   Can improve resilience to some origin load problems.

### Disadvantages / risks

-   Invalidation can be slow or inconsistent.
-   Incorrectly caching personalized responses can leak data.
-   Cache misses and mass invalidations can overload the origin.

### Ways to overcome limitations

-   Use fingerprinted/versioned assets with long TTLs.
-   Set appropriate `Cache-Control` and `Vary` behavior.
-   Purge selectively; avoid unnecessary global invalidations.
-   Use origin shielding, request collapsing, and capacity protection.
-   Test cache behavior for logged-in users, currencies, stores, and
    customer groups.

## Lesson 10: Pagination and Query Optimization

### Key points

-   Avoid returning unbounded result sets.
-   **Offset pagination:** simple, but large offsets can be expensive
    and concurrent writes can shift results.
-   **Cursor/keyset pagination:** continues from a stable ordering;
    often better for large, changing datasets.
-   Index filter and sort columns based on query patterns.
-   Inspect execution plans and measure slow queries.
-   Use stable tie-breakers, e.g. `(created_at, entity_id)`.

### Offset advantages / disadvantages

-   Advantage: easy to implement and supports jumping to a page.
-   Disadvantage: large offsets can scan many rows; pages may duplicate
    or skip rows as data changes.

### Cursor advantages / disadvantages

-   Advantage: efficient continuation and better behavior on changing
    datasets.
-   Disadvantage: harder to jump to arbitrary pages; cursor design and
    ordering must be stable.

### Ways to overcome limitations

-   Use cursor pagination for large feeds and frequently changing lists.
-   Add indexes that match filters and ordering.
-   Select only required columns and avoid N+1 queries.
-   Use bounded page sizes and monitor query plans.

## Lesson 11: Rate Limiting and Throttling

### Key points

-   Protect services from abuse, bursts, and overload.
-   Algorithms: fixed window, sliding window, token bucket, and leaky
    bucket.
-   Token bucket permits controlled bursts while limiting the long-term
    rate.
-   Apply limits per user, API key, IP, tenant, or endpoint as
    appropriate.
-   Return `429 Too Many Requests`; include `Retry-After` when useful.

### Advantages

-   Protects capacity and improves fairness.
-   Limits damage from buggy clients and traffic spikes.

### Disadvantages / risks

-   Shared counters can become bottlenecks.
-   IP-based limits can punish users behind shared NAT.
-   Poor limits can block legitimate traffic.

### Ways to overcome limitations

-   Use distributed counters or a gateway where appropriate.
-   Apply tiered limits and separate expensive endpoints.
-   Add burst allowances, clear client guidance, and monitoring.
-   Keep emergency protection separate from normal quota policy.

## Lesson 12: Load Balancing and High Availability

### Key points

-   Distribute traffic across healthy instances.
-   Common algorithms: round robin, weighted round robin, least
    connections, and consistent hashing.
-   Health checks should test meaningful readiness, not only whether a
    process exists.
-   Stateless app servers are easier to scale horizontally.
-   Plan for connection draining, retries, session handling, and
    dependency failure.

### Advantages

-   Higher capacity and better fault isolation.
-   Supports rolling deployments and failover.

### Disadvantages / risks

-   Bad health checks can route traffic to unhealthy nodes or remove
    healthy ones.
-   Stateful sessions can pin traffic to one server.
-   A single load balancer can itself become a failure point.

### Ways to overcome limitations

-   Use redundant managed or clustered load balancers.
-   Store sessions in a shared service where appropriate.
-   Use readiness checks, graceful shutdown, and connection draining.
-   Test failover and avoid retrying non-idempotent requests blindly.

------------------------------------------------------------------------

# Day 4 --- Asynchronous Systems and Resilience

## Lesson 13: Message Queues --- RabbitMQ and Kafka

### Key points

-   Queues decouple producers from consumers and absorb bursts.
-   RabbitMQ is commonly used for task queues and routing; Kafka is
    commonly used for durable event streams and replayable logs. Both
    have broader capabilities.
-   Acknowledge messages only after the intended work is safely
    completed.
-   Design consumers for duplicate delivery.
-   Monitor queue depth, oldest-message age, consumer lag, retries, and
    dead-letter counts.

### Advantages

-   Smooths traffic bursts and isolates slow dependencies.
-   Supports asynchronous work and independent scaling.
-   Durable messaging can preserve work across service interruptions.

### Disadvantages / risks

-   Adds eventual consistency and operational complexity.
-   Messages can be delivered more than once.
-   Backlogs can grow beyond recovery capacity.
-   Ordering and replay semantics require care.

### Ways to overcome limitations

-   Make consumers idempotent.
-   Use durable queues/messages, publisher confirms, and appropriate
    acknowledgments.
-   Bound retries and route poison messages to a dead-letter queue.
-   Apply backpressure and capacity limits.
-   Version event schemas and monitor consumer lag.

## Lesson 14: Event-Driven Architecture and Background Workers

### Key points

-   An event records something that happened; a command requests an
    action.
-   Events allow services to react asynchronously and reduce synchronous
    coupling.
-   A worker consumes events and updates downstream systems.
-   The **transactional outbox** stores a business update and its event
    record in the same database transaction; a separate publisher sends
    the event.
-   Use sagas for multi-step workflows that span services.

### Advantages

-   Decouples services and improves burst handling.
-   Allows independent consumer scaling and event-driven extensions.

### Disadvantages / risks

-   Events may be delayed, duplicated, or processed out of order.
-   Distributed workflows can partially succeed.
-   Debugging is harder without correlation IDs and tracing.

### Ways to overcome limitations

-   Use an outbox to avoid the database-commit/message-publish
    dual-write gap.
-   Make handlers idempotent and define ordering requirements.
-   Use saga compensations for reversible business actions, while
    recognizing compensation is not a true rollback.
-   Add reconciliation for irreversible or ambiguous outcomes.

## Lesson 15: Retries, Idempotency, and Dead-Letter Queues

### Key points

-   A timeout means the outcome may be unknown; the remote service may
    already have completed the operation.
-   Retry transient failures with bounded exponential backoff and
    jitter.
-   Respect `Retry-After` for HTTP 429/503 when supplied.
-   Do not repeatedly retry permanent errors such as invalid input.
-   Idempotency keys must be stable and enforced atomically by the
    receiver.
-   A DLQ stores messages that cannot be processed after the configured
    policy; replay must be controlled.

### Advantages

-   Retries recover from temporary network or service failures.
-   Idempotency makes safe retry behavior possible.
-   DLQs prevent poison messages from blocking normal processing.

### Disadvantages / risks

-   Retry storms can overwhelm a recovering dependency.
-   Duplicate side effects occur without idempotency.
-   DLQs can become unnoticed data graveyards.

### Ways to overcome limitations

-   Use bounded exponential backoff with jitter.
-   Add circuit breakers, rate limits, concurrency limits, and timeouts.
-   Reconcile ambiguous timeouts by querying the downstream system using
    a stable business ID.
-   Store idempotency records durably and enforce uniqueness in a
    transaction.
-   Alert on DLQ count and age; document safe replay and repair
    procedures.

## Lesson 16: Distributed Locks, Leader Election, and Coordination

### Key points

-   A distributed lock coordinates exclusive access across processes or
    machines.
-   Leader election chooses one coordinator from multiple candidates.
-   Redis lock acquisition often uses an atomic
    `SET key token NX PX ttl`.
-   Release only if the ownership token still matches.
-   A lease can expire while a slow worker is still running.
-   A fencing token is a monotonically increasing number that downstream
    writes can check to reject stale owners.

### Advantages

-   Prevents overlapping scheduled jobs and coordinates shared work.
-   Leases can allow recovery after a process crashes.

### Disadvantages / risks

-   TTL expiry can allow two workers to overlap.
-   Network pauses and clock issues complicate assumptions.
-   A lock alone does not guarantee exactly-once business effects.

### Ways to overcome limitations

-   Use unique ownership tokens and compare-and-delete release.
-   Choose lease duration based on measured work and renew safely where
    supported.
-   Use fencing tokens or version checks enforced by the resource being
    modified.
-   Add idempotency and database constraints as independent safeguards.
-   Prefer well-understood coordination primitives over custom
    protocols.

## Lesson 17: Fault Tolerance and Disaster Recovery

### Key points

-   Fault tolerance continues essential service despite component
    failures.
-   High availability minimizes downtime; disaster recovery restores
    after major incidents.
-   **RTO:** target maximum recovery time.
-   **RPO:** target maximum tolerable data loss, measured in time.
-   Backups, replication, and point-in-time recovery serve different
    purposes.
-   Include database, media, configuration, secrets, queues, search
    indexes, and integrations in recovery planning.

### Advantages

-   Reduces business impact and makes recovery predictable.
-   Tested recovery procedures reduce dependence on improvisation.

### Disadvantages / risks

-   Cross-region infrastructure and replication cost money.
-   Replication can copy corruption or accidental deletion.
-   Untested backups may be unusable.
-   A replica may lag beyond the required RPO.

### Ways to overcome limitations

-   Define RTO/RPO with business owners.
-   Keep immutable/off-site backups and preserve transaction logs for
    point-in-time recovery.
-   Test restoration regularly and measure actual recovery time and data
    loss.
-   Document promotion, traffic switching, validation, and
    reconciliation.
-   Prevent split-brain by fencing or isolating the old primary before
    writes resume.

## Lesson 18: Quorum Reads/Writes and Advanced Consistency

### Key points

-   With `N` replicas, `W` write acknowledgments, and `R` replicas
    consulted for reads, `R + W > N` guarantees read/write quorum
    overlap in the usual quorum model.
-   `2W > N` guarantees overlap between any two write quorums.
-   Quorum overlap alone does not guarantee linearizability; version
    resolution, read/write protocols, and replica behavior matter.
-   Replicas may lag and return stale values.
-   Critical inventory correctness needs atomic reservation and
    business-level safeguards.

### Advantages

-   Quorums can balance consistency, availability, and latency.
-   Multiple replicas can tolerate some node failures.

### Disadvantages / risks

-   Larger quorums increase latency and may fail when too few replicas
    are reachable.
-   Concurrent writes can conflict.
-   Stale reads remain possible in systems with weaker protocols.

### Ways to overcome limitations

-   Understand the exact database consistency guarantees.
-   Use appropriate versioning, conflict resolution, and read repair.
-   Use atomic conditional updates or reservations for stock.
-   Test behavior during partitions and replica lag.
-   Do not treat quorum arithmetic as a substitute for application
    invariants.

------------------------------------------------------------------------

# Day 5 --- Coordination and Reliability Concepts

> These topics reinforce earlier lessons and are frequently combined in
> architecture interviews.

## Distributed coordination: choosing the right tool

-   Use a lock for exclusive access to a task/resource.
-   Use leader election for one active coordinator.
-   Use a queue for durable work distribution.
-   Use database constraints and transactions for business invariants.
-   Use idempotency for duplicate delivery and retries.
-   These mechanisms are complementary, not interchangeable.

### Common failure modes

-   Lock expires while work continues.
-   Leader loses connectivity but still performs writes.
-   Queue redelivers a message.
-   Replica lag exposes stale state.
-   Retry logic overwhelms a dependency.

### Mitigations

-   Fencing tokens, leases, idempotency, unique constraints, bounded
    retries, backpressure, and reconciliation.
-   Instrument lock contention, leader changes, queue age, duplicate
    detection, and failed recovery attempts.

------------------------------------------------------------------------

# Day 6 --- Networking, API Design, and Security

## Lesson 19: Networking --- DNS, TCP, TLS, and HTTP

### Key points

-   DNS maps domain names to IP addresses; TTL and caches affect how
    quickly changes propagate.
-   TCP provides a reliable, ordered byte stream, but cannot tell a
    client whether an application operation committed before a
    connection dropped.
-   TLS provides encryption, integrity, and server authentication for
    HTTPS.
-   HTTP status codes describe outcomes; `429` may include
    `Retry-After`, and `5xx` often indicates a server-side problem.
-   HTTP/1.1 and HTTP/2 generally use TCP; HTTP/3 uses QUIC over UDP.
-   Connection reuse and DNS/TLS caching reduce repeated setup costs.

### Advantages

-   TCP simplifies reliable data transfer.
-   TLS protects data in transit.
-   DNS decouples service names from addresses.

### Disadvantages / risks

-   Connection setup adds latency.
-   DNS changes may take time to be reflected due to caching.
-   A dropped response leaves operation outcome ambiguous.
-   TLS does not fix application-level authorization or validation
    defects.

### Ways to overcome limitations

-   Reuse connections and optimize network round trips.
-   Use sensible timeouts and monitor DNS/TLS/connect/server timings.
-   Use idempotency keys for retryable state-changing requests.
-   Manage certificates and DNS TTLs carefully.
-   Diagnose with logs, metrics, tracing, and network-level telemetry.

## Lesson 20: API Design --- REST, GraphQL, Versioning, and Pagination

### Key points

-   REST models resources and uses HTTP methods; GraphQL lets clients
    request specific fields and related data.
-   Use consistent response and error formats.
-   Offset pagination is simple; cursor/keyset pagination often scales
    better for large, changing datasets.
-   Version APIs deliberately and prefer backward-compatible changes.
-   Idempotency keys can make POST operations safe to retry when
    enforced durably.
-   Apply authentication, authorization, validation, rate limits, and
    request-size limits.

### REST advantages / disadvantages

-   Advantage: familiar resource semantics, simple tooling, and
    conventional HTTP caching.
-   Disadvantage: clients may need multiple calls or receive extra
    fields.

### GraphQL advantages / disadvantages

-   Advantage: flexible field selection and related data in one query.
-   Disadvantage: query cost, authorization, caching, and N+1
    performance require careful controls.

### Ways to overcome limitations

-   Select REST or GraphQL based on client needs and operational
    capability.
-   Use query complexity/depth limits and batching or DataLoader
    patterns where appropriate.
-   Use stable cursors and indexed sort fields.
-   Document deprecations and monitor old-version usage.
-   Persist idempotency key, request identity/hash, and outcome
    atomically with the business operation.

## Lesson 21: Authentication, Authorization, and API Security

### Key points

-   Authentication asks "who are you?"; authorization asks "what may you
    do?"
-   JWTs commonly have header, payload, and signature; a signed JWT is
    not necessarily encrypted.
-   Validate signature, algorithm, issuer, audience, expiry, and
    required claims.
-   OAuth 2.0 supports delegated authorization; OpenID Connect adds an
    identity layer.
-   PKCE protects authorization-code flows for public clients.
-   RBAC assigns permissions to roles; ABAC makes decisions using
    attributes.
-   Multi-tenant systems must enforce tenant boundaries on every
    operation.

### Advantages

-   Strong identity and access controls reduce unauthorized actions.
-   Fine-grained permissions support least privilege and auditing.

### Disadvantages / risks

-   Tokens can be stolen, misconfigured, or accepted for the wrong
    audience.
-   Overly broad permissions increase blast radius.
-   Caching authorization decisions across tenants can cause data leaks.

### Ways to overcome limitations

-   Use short-lived tokens, secure secret storage, key rotation, and
    appropriate revocation strategies.
-   Validate tenant, resource ownership, action, and scope server-side
    on every request.
-   Isolate tenant credentials and cache keys.
-   Audit sensitive operations; require approvals for high-impact
    actions.
-   Treat AI tool selection as untrusted input and enforce authorization
    outside the model.

------------------------------------------------------------------------

# Day 7 --- Scale, Observability, and Reliability

## Lesson 22: Scalability and Capacity Planning

### Key points

-   **Vertical scaling:** add resources to a machine.
-   **Horizontal scaling:** add more instances.
-   Estimate request rate, peak factor, data size, concurrency, and
    queue processing capacity.
-   Little's Law: `L = λ × W`.
-   Identify bottlenecks before adding capacity.
-   Background workers must be scaled against both queue arrival rate
    and downstream capacity.

### Vertical scaling advantages / disadvantages

-   Advantage: simple operational model.
-   Disadvantage: hardware limits and potential single points of
    failure.

### Horizontal scaling advantages / disadvantages

-   Advantage: more capacity and redundancy.
-   Disadvantage: requires stateless design or shared state, load
    balancing, and coordination.

### Ways to overcome limitations

-   Load-test realistic traffic mixes and failure scenarios.
-   Add headroom for bursts and node loss.
-   Use autoscaling with sensible limits and warm-up behavior.
-   Avoid scaling consumers beyond what databases or downstream APIs can
    handle.
-   Track saturation, not only utilization.

## Lesson 23: Observability --- Logs, Metrics, Tracing, SLI/SLO/SLA

### Key points

-   **Logs:** detailed event records.
-   **Metrics:** numeric trends and rates.
-   **Traces:** follow a request across services.
-   **SLI:** measured indicator; **SLO:** target; **SLA:** external
    commitment, often contractual.
-   Monitor latency percentiles, success rate, throughput, saturation,
    queue age, and dependency health.
-   Use correlation/trace IDs across Magento, queues, and external APIs.

### Advantages

-   Faster diagnosis and better incident decisions.
-   SLOs make reliability measurable and help prioritize work.

### Disadvantages / risks

-   High-cardinality metrics and verbose logs can be expensive.
-   Too many alerts cause fatigue.
-   Metrics without context may not explain root cause.

### Ways to overcome limitations

-   Define actionable alerts tied to user impact.
-   Redact secrets and personal data from telemetry.
-   Sample traces appropriately while retaining useful error traces.
-   Set retention and cardinality budgets.
-   Use runbooks and review incidents to improve alert quality.

## Lesson 24: Reliability, Failover, and Disaster Recovery

### Key points

-   Plan for component, server, zone, region, data, and operator
    failures.
-   **Active-active:** multiple sites serve traffic; can improve
    capacity but complicates writes and consistency.
-   **Active-passive:** a standby takes over; simpler in some systems
    but failover can take longer.
-   Failover includes detection, fencing, promotion, routing,
    validation, and reconciliation.
-   Backups are not the same as replication.
-   Cascading failures are mitigated with timeouts, bounded retries,
    circuit breakers, bulkheads, and backpressure.

### Advantages

-   Reduces downtime and improves recovery predictability.
-   Independent failure domains protect against wider outages.

### Disadvantages / risks

-   Redundant infrastructure costs more.
-   Failover can cause split-brain or data loss if not coordinated.
-   Recovery may fail if dependencies or data were omitted from the
    plan.

### Ways to overcome limitations

-   Match architecture to RTO/RPO and business impact.
-   Fence the old primary before allowing a new writer.
-   Test regional recovery and restore from backups.
-   Validate checkout, payments, inventory, media, queues, and
    integrations after recovery.
-   Reconcile ambiguous or duplicate orders before replaying queued
    work.

------------------------------------------------------------------------

# Cross-Lesson Interview Cheat Sheet

## Common patterns and when to use them

  ---------------------------------------------------------------------------
  Pattern           Use it for        Main limitation   Mitigation
  ----------------- ----------------- ----------------- ---------------------
  Cache-aside       Speed up repeated Stale data /      TTL, invalidation,
                    reads             stampede          request coalescing

  CDN               Deliver           Invalidation and  Versioned assets,
                    static/public     cache leakage     correct cache keys
                    content near                        and headers
                    users                               

  Load balancer     Distribute        Bad health checks Redundancy, readiness
                    requests and      / single point of checks, draining
                    remove unhealthy  failure           
                    nodes                               

  Message queue     Buffer            Duplicates and    Idempotent consumers,
                    asynchronous work growing backlog   DLQ, backpressure

  Transactional     Atomically        Publisher lag and Relay monitoring,
  outbox            persist business  duplicate         idempotent consumers
                    update and event  publication       
                    intent                              

  Retry with        Recover from      Retry storm       Bounded attempts,
  backoff + jitter  transient errors                    circuit breaker,
                                                        concurrency limits

  Idempotency key   Make repeated     Key loss, scope   Durable atomic
                    operations safe   errors, payload   enforcement and
                                      mismatch          request hash

  Distributed lock  Coordinate        Lease expiry /    Fencing tokens,
                    exclusive work    stale owner       compare-and-delete,
                                                        idempotency

  Replication       Read scaling and  Replica lag and   Lag monitoring,
                    failover          replicated        backups, recovery
                                      corruption        tests

  Sharding          Scale data beyond Cross-shard       Good shard key,
                    one node          complexity and    rebalancing plan,
                                      hot keys          limit cross-shard
                                                        work

  Quorum            Coordinate        Latency and       Understand actual
                    reads/writes      protocol          consistency
                    across replicas   complexity        guarantees

  Circuit breaker   Stop calls to     May temporarily   Controlled half-open
                    failing           reject requests   probes and fallback
                    dependency                          behavior

  Rate limiter      Protect service   False throttling  Tiered policies,
                    capacity          / shared-counter  burst allowance,
                                      cost              observability

  Saga              Coordinate        Compensation is   Idempotent steps,
                    multi-service     not true rollback durable state,
                    workflow                            reconciliation

  Active-passive DR Recover after     Recovery time and Tested runbooks,
                    site failure      replication lag   RTO/RPO monitoring

  Active-active     Serve from        Conflict          Partition ownership,
                    multiple sites    resolution and    conflict strategy,
                                      write             carefully scoped
                                      coordination      writes
  ---------------------------------------------------------------------------

## Useful formulas

-   Average requests per second: `daily requests / 86,400`
-   Peak RPS estimate: `average RPS × peak-to-average factor`
-   Little's Law: `concurrency = arrival rate × average time in system`
-   Availability downtime budget:
    `period duration × (1 − availability target)`
-   Quorum read/write overlap condition: `R + W > N`
-   Quorum write/write overlap condition: `2W > N`
-   Base62 combinations for code length `k`: `62^k`
-   Worker count lower bound:
    `arrival rate / sustainable throughput per worker`; add headroom and
    validate by load testing.

## A repeatable system design interview structure

1.  **Clarify requirements:** users, core flows, scope, consistency,
    security, and SLAs.
2.  **Estimate scale:** average and peak traffic, data growth,
    read/write ratio, and concurrency.
3.  **Define APIs and data model:** keys, constraints, indexes,
    pagination, idempotency.
4.  **Draw high-level architecture:** client, edge, application, cache,
    database, queue, dependencies.
5.  **Walk through critical flows:** success path, retry path, timeout,
    duplicate request, and dependency outage.
6.  **Find bottlenecks:** database, cache, worker throughput, network,
    and downstream limits.
7.  **Compare trade-offs:** latency, consistency, availability,
    complexity, and cost.
8.  **Explain security:** authentication, authorization, tenant
    isolation, secrets, and audit.
9.  **Explain observability:** SLIs/SLOs, logs, metrics, traces, alerts,
    and runbooks.
10. **Explain recovery:** backups, failover, RTO/RPO, replay, and
    reconciliation.

## Final reminders

-   Do not claim "exactly once" merely because a queue or lock is used.
    Design for duplicate delivery and idempotent effects.
-   A timeout does not prove the operation failed.
-   Replication is not a substitute for backup.
-   A cache is not the source of truth for critical business invariants.
-   More workers can overload a dependency; scale based on end-to-end
    capacity.
-   Quorum overlap alone does not guarantee linearizability.
-   A healthy server does not necessarily mean a healthy user journey.
-   Prefer the simplest architecture that satisfies the measured
    requirements, and explain how it can evolve.
