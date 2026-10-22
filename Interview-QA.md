# System Design Interview Q&A --- Days 1--7

## 24 Lessons \| Core Questions, Model Answers, Trade-offs, and Practical Examples

These are concise model answers for interview revision. Adapt the
examples and depth to the question being asked. In an interview, clarify
requirements and state assumptions before proposing a design.

# Day 1 --- System Design Foundations

## Lesson 1: System Design Fundamentals

### Q1. What is system design?

**Answer:** System design is the process of defining a system's
architecture, components, APIs, data models, communication patterns, and
operational behavior to meet functional and non-functional requirements.
A good design explains not only the happy path but also bottlenecks,
security, failure handling, and recovery.

### Q2. What is the difference between functional and non-functional requirements?

**Answer:** Functional requirements describe what the system does, such
as creating an order or searching products. Non-functional requirements
describe how well it must operate, such as latency, availability,
scalability, security, consistency, and cost. Both influence
architecture.

### Q3. How would you approach an open-ended system design interview?

**Answer:** Clarify scope and users; establish functional and
non-functional requirements; estimate traffic and data; define APIs and
data models; draw the high-level architecture; walk through critical
flows; identify bottlenecks and failures; compare trade-offs; then cover
security, monitoring, and disaster recovery.

### Q4. Why not start with microservices for every new system?

**Answer:** Microservices provide independent deployment and scaling but
introduce network failures, distributed transactions, operational
overhead, and more complex debugging. A modular monolith is often
simpler initially. Split services when ownership, deployment, scaling,
or fault-isolation needs justify the cost.

## Lesson 2: Latency, Throughput, Availability, and Capacity

### Q1. What is the difference between latency and throughput?

**Answer:** Latency is the time one operation takes; throughput is the
number of operations completed per unit time. A system may have high
throughput but poor tail latency. Measure both, usually including p50,
p95, and p99 latency.

### Q2. What is Little's Law, and when is it useful?

**Answer:** Little's Law is `L = λ × W`, where `L` is average work in
the system, `λ` is arrival rate, and `W` is average time in the system.
At 500 requests/second and 0.2 seconds average time, average in-flight
work is about 100 requests. It estimates concurrency, not server count
by itself.

### Q3. How do you estimate requests per second?

**Answer:** Divide daily requests by 86,400 to estimate average RPS,
then apply a measured or assumed peak-to-average factor. For example, 1
million requests/day is about 11.6 average RPS. Use production patterns
and load tests to validate the peak estimate.

### Q4. Why is average latency insufficient?

**Answer:** Averages hide slow requests. If most requests take 100 ms
but a small fraction take several seconds, the average may look
acceptable while customers experience timeouts. Percentiles such as p95
and p99 reveal tail behavior.

## Lesson 3: Requirements and Problem Scoping

### Q1. What questions should you ask before designing a system?

**Answer:** Ask who the users are, the main workflows, expected traffic
and data growth, read/write mix, latency and availability targets,
consistency needs, security/compliance constraints, retention,
integrations, and recovery objectives. Prioritize questions that change
the architecture.

### Q2. How do you handle missing requirements during an interview?

**Answer:** State reasonable assumptions explicitly, explain why they
are plausible, and continue. Avoid pretending uncertain numbers are
facts. Identify which assumptions would need validation in a real
project.

### Q3. How do you prioritize requirements?

**Answer:** Separate must-have functionality from optional features.
Prioritize critical user journeys, correctness and safety invariants,
legal/security obligations, and measurable performance targets. Defer
features that do not affect the core objective.

## Lesson 4: High-Level Architecture and Core Components

### Q1. What are the common components of a web architecture?

**Answer:** Clients, CDN, load balancer, application services, cache,
database, search engine, message broker, object storage, external
dependencies, and observability. Not every system needs every component;
choose based on requirements and measured bottlenecks.

### Q2. When should work be synchronous versus asynchronous?

**Answer:** Use synchronous calls when the user needs an immediate
result or decision. Use asynchronous queues for work that can complete
later, is bursty, or should be decoupled, such as notifications,
indexing, or ERP synchronization. Communicate pending states and handle
eventual consistency.

### Q3. What are the disadvantages of adding more services?

**Answer:** More services create network latency, partial failures,
deployment and monitoring overhead, security boundaries, and distributed
data consistency challenges. Mitigate with clear ownership, contracts,
timeouts, tracing, and by avoiding unnecessary service decomposition.

# Day 2 --- Data, Replication, and Consistency

## Lesson 5: SQL vs. NoSQL and Data Modeling

### Q1. When would you choose SQL over NoSQL?

**Answer:** SQL is a strong fit for relational data, joins, constraints,
and transactional workflows such as orders, payments, and stock
reservations. NoSQL can be a better fit for access patterns such as
high-volume key-value lookups or flexible documents. Choose based on
data model, access patterns, consistency, scale, and operational skills.

### Q2. What are the advantages and disadvantages of denormalization?

**Answer:** Denormalization can make reads faster by storing data in a
query-friendly shape, but duplicates information and makes updates
harder to keep consistent. Use it for measured read bottlenecks, define
the source of truth, and maintain derived copies through transactions,
events, or reconciliation.

### Q3. How do indexes improve performance, and what do they cost?

**Answer:** Indexes let the database locate rows without scanning the
entire table. They consume storage and add work to inserts, updates, and
deletes. Create indexes to match real filters, joins, and sort orders,
and verify them with query plans and measurements.

### Q4. How would you model an order system?

**Answer:** Use an order table with a stable primary key and
status/timestamps, order-item rows linked by foreign key, and separate
payment/shipping records as appropriate. Enforce important uniqueness
and integrity constraints. Keep an immutable audit/event trail where
business requirements demand it.

## Lesson 6: Replication, Partitioning, and Sharding

### Q1. What is the difference between replication and sharding?

**Answer:** Replication copies data to multiple nodes, often for read
scaling or failover. Sharding partitions different subsets of data
across nodes to increase total storage and write capacity. A system can
use both.

### Q2. What makes a good shard key?

**Answer:** A good shard key distributes load evenly and aligns with
common access patterns. It should avoid hot keys and minimize
cross-shard queries. Evaluate cardinality, growth, tenant distribution,
and rebalancing needs before choosing it.

### Q3. What is replication lag, and how do you handle it?

**Answer:** Replication lag is the delay before a replica reflects
primary changes. It can cause stale reads. Monitor lag, route
correctness-sensitive reads to an appropriate source, use
read-your-writes mechanisms when available, and design workflows that
tolerate staleness where acceptable.

### Q4. What problems does sharding introduce?

**Answer:** Sharding complicates joins, transactions, global uniqueness,
rebalancing, migrations, and operational troubleshooting. Mitigate with
stable shard keys, carefully designed IDs, shard-aware APIs, limited
cross-shard transactions, and tested rebalancing procedures.

## Lesson 7: Consistency, Transactions, and CAP

### Q1. What is the difference between strong and eventual consistency?

**Answer:** Strong consistency, under a defined model such as
linearizability, makes reads observe the latest completed write in a
single coherent order. Eventual consistency permits temporary stale or
divergent reads but expects replicas to converge if updates stop and
propagation succeeds. Use stronger guarantees for critical invariants
and weaker ones where temporary staleness is acceptable.

### Q2. Explain ACID.

**Answer:** Atomicity means all-or-nothing; consistency means
transactions preserve defined constraints; isolation controls
interactions between concurrent transactions; durability means committed
changes survive the promised failure model. ACID is important for
orders, payments, and inventory.

### Q3. What does CAP theorem say?

**Answer:** During a network partition, a distributed system cannot
guarantee both linearizable consistency and successful responses from
every side of the partition. Real systems must handle partitions; the
design chooses which operations can proceed and which must fail or wait
safely.

### Q4. How would you prevent Magento from overselling the last item?

**Answer:** Use an atomic conditional stock update or reservation
transaction, enforce business invariants in the database or inventory
service, and avoid relying on cached catalog stock as the source of
truth. Make order processing idempotent and reconcile asynchronous
inventory updates.

# Day 3 --- Performance and Traffic Management

## Lesson 8: Caching and Performance

### Q1. Explain the cache-aside pattern.

**Answer:** The application checks the cache first. On a miss, it reads
from the source of truth, populates the cache, and returns the result.
It is simple and effective for read-heavy workloads, but needs an
explicit TTL and invalidation policy.

### Q2. What is a cache stampede, and how do you prevent it?

**Answer:** A cache stampede occurs when many requests miss or expire
the same key at once and overwhelm the origin. Mitigations include
request coalescing, locks, early refresh, TTL jitter,
stale-while-revalidate where safe, and rate-limiting origin fetches.

### Q3. How do you avoid serving the wrong cached content?

**Answer:** Build cache keys from every dimension that changes the
response, such as store, locale, currency, customer group, permissions,
and relevant query parameters. Do not publicly cache personalized
responses unless the cache policy safely separates users or variants.

### Q4. What are cache invalidation trade-offs?

**Answer:** Short TTLs reduce stale data but increase origin load;
explicit invalidation improves freshness but adds complexity and can
fail. Use bounded TTLs, event-driven invalidation where useful,
versioned keys for immutable data, and reconciliation for critical
derived state.

## Lesson 9: CDN and Static Content Delivery

### Q1. What is a CDN, and why use it?

**Answer:** A CDN serves cacheable content from edge locations near
users. It reduces latency and origin traffic and can absorb traffic
spikes. It is particularly useful for images, fonts, JavaScript, CSS,
and other public static assets.

### Q2. What should be cached at the CDN?

**Answer:** Cache public static assets and carefully selected public
responses. Avoid caching personalized account, cart, or checkout data
without a rigorously designed cache policy. Use correct `Cache-Control`,
`Vary`, and cache-key rules.

### Q3. How do you avoid an origin surge after cache invalidation?

**Answer:** Prefer versioned/fingerprinted assets and selective purges.
Use origin shielding, request collapsing, prewarming where appropriate,
and rate/capacity controls. Avoid mass invalidation unless necessary and
test cache-miss behavior.

### Q4. What are the risks of CDN caching?

**Answer:** Stale content, delayed invalidation, cache misses, and
accidental leakage of personalized responses. Mitigate through explicit
cache policies, safe cache keys, automated tests for logged-in and
anonymous sessions, and monitoring cache-hit ratio and origin load.

## Lesson 10: Pagination and Query Optimization

### Q1. Compare offset and cursor pagination.

**Answer:** Offset pagination is simple and supports jumping to a page,
but large offsets can be expensive and concurrent changes can cause
skipped or repeated rows. Cursor/keyset pagination uses a stable
ordering and continuation token; it is often better for large or
changing datasets but makes arbitrary page jumps harder.

### Q2. How do you design a reliable cursor?

**Answer:** Use a stable indexed ordering and include a unique
tie-breaker, such as `(created_at, entity_id)`. Encode the last-seen
values in an opaque cursor and validate it. Ensure the next-page
predicate matches the sort direction and index.

### Q3. How do you investigate a slow SQL query?

**Answer:** Capture the query and timings, inspect the execution plan,
check indexes and row estimates, identify scans, expensive joins, sorts,
locks, and N+1 behavior, then test a targeted change. Verify with
representative data and production-like load.

### Q4. What other query optimizations are useful?

**Answer:** Select only required columns, index real filter/sort
patterns, batch repeated lookups, avoid N+1 queries, limit result sizes,
cache safe repeated reads, and archive or partition data when justified.
Measure before and after.

## Lesson 11: Rate Limiting and Throttling

### Q1. Compare fixed window, sliding window, token bucket, and leaky bucket.

**Answer:** Fixed window is simple but permits boundary bursts. Sliding
window smooths limits but may cost more to track. Token bucket allows
controlled bursts while limiting average rate. Leaky bucket processes at
a steadier rate. Choose based on burst tolerance, fairness, and
implementation cost.

### Q2. How should an API respond when rate-limited?

**Answer:** Return HTTP `429 Too Many Requests`; include `Retry-After`
when a useful retry time is known. Document limits and ensure clients
use backoff rather than retrying continuously.

### Q3. Where should rate limits be applied?

**Answer:** Apply limits at a gateway and/or service level based on IP,
user, API key, tenant, and endpoint cost. Protect expensive operations
separately and account for shared NATs and legitimate burst patterns.

### Q4. What are common rate-limiter pitfalls?

**Answer:** A centralized counter can become a bottleneck; IP-only
limits can unfairly group users; overly strict quotas can reject
legitimate traffic. Use distributed counters or gateways, tiered limits,
burst allowances, and monitoring of throttled requests.

## Lesson 12: Load Balancing and High Availability

### Q1. What does a load balancer do?

**Answer:** It distributes requests across backend instances and can
stop sending traffic to unhealthy nodes. Algorithms include round robin,
weighted round robin, least connections, and consistent hashing. The
choice depends on workload and state.

### Q2. What is the difference between liveness and readiness checks?

**Answer:** Liveness indicates whether a process should be restarted;
readiness indicates whether it should receive traffic. Readiness can
consider startup and required dependencies, but overly broad dependency
checks may remove every node during a shared outage.

### Q3. How do you scale a stateful web application horizontally?

**Answer:** Keep application nodes stateless where possible; externalize
sessions, shared cache, and durable work; store files in shared/object
storage; and use a load balancer with health checks. Avoid sticky
sessions as a substitute for a robust shared-state design unless
justified.

### Q4. How do you prevent a load balancer from becoming a single point of failure?

**Answer:** Use a redundant or managed load-balancing service,
health-check its dependencies, and test failover. Also configure
graceful shutdown, connection draining, and safe retry behavior for
clients.

# Day 4 --- Queues, Events, and Resilience

## Lesson 13: Message Queues --- RabbitMQ and Kafka

### Q1. Why use a message queue?

**Answer:** A queue decouples producers from consumers, absorbs bursts,
and allows asynchronous processing. It is useful for notifications,
search indexing, and ERP integration. The trade-off is eventual
consistency and additional operational complexity.

### Q2. Compare RabbitMQ and Kafka.

**Answer:** RabbitMQ is commonly used for task queues, acknowledgments,
and flexible routing. Kafka is commonly used for durable event streams,
high-throughput partitioned logs, and replay. Both are capable systems;
choose based on routing, retention/replay, ordering, throughput,
consumer model, and team expertise.

### Q3. How do you prevent message loss?

**Answer:** Use durable queues/topics and persistent messages where
supported, publisher confirms, appropriate acknowledgments, replication,
and tested recovery. Acknowledge only after the intended processing is
safely committed. End-to-end guarantees depend on producer, broker,
consumer, and storage configuration.

### Q4. How do you handle duplicate message delivery?

**Answer:** Assume at-least-once delivery can produce duplicates. Give
each event a stable ID, make consumer effects idempotent, and enforce
uniqueness or processed-event tracking transactionally. Do not assume a
broker alone provides exactly-once business effects.

## Lesson 14: Event-Driven Architecture and Background Workers

### Q1. What is event-driven architecture?

**Answer:** Components publish events describing facts that happened,
and consumers react asynchronously. It reduces direct coupling and
supports independent consumers, but introduces delay, duplicates,
ordering concerns, and more complex debugging.

### Q2. What is the transactional outbox pattern?

**Answer:** The application writes the business change and an outbox
event in the same database transaction. A relay later publishes the
event. This avoids the dual-write problem where the database commit
succeeds but message publication fails. Consumers must still handle
duplicate publication.

### Q3. What is a saga?

**Answer:** A saga coordinates a multi-step business workflow across
services using local transactions and, when needed, compensating
actions. For example, if inventory is reserved but payment fails, the
workflow can release the reservation. Compensation is a new business
action, not a perfect rollback of history.

### Q4. How do you make background workers reliable?

**Answer:** Use bounded retries, idempotent handlers, durable messages,
appropriate acknowledgments, dead-letter handling, schema versioning,
correlation IDs, and monitoring. Scale consumers based on queue arrival
rate and downstream capacity, not just queue size.

## Lesson 15: Retries, Idempotency, and Dead-Letter Queues

### Q1. Why is a timeout an ambiguous outcome?

**Answer:** The request may have reached the remote service and
committed even though the response was lost. A timeout means the client
does not know the outcome; it does not prove the operation failed.
Reconcile using a stable business ID or idempotency key before retrying
uncertain side effects.

### Q2. How do exponential backoff and jitter help?

**Answer:** Exponential backoff increases the delay after repeated
failures. Jitter randomizes the delay so many workers do not retry
simultaneously. Use a cap, bounded attempts, and respect `Retry-After`
where supplied.

### Q3. Which failures should be retried?

**Answer:** Transient failures such as temporary service unavailability,
network errors, and some throttling responses may be retried. Validation
errors such as an invalid tax ID usually require correction, not
retries. Authentication failures require credential or permission
repair. Retry policy should depend on status, operation semantics, and
downstream guidance.

### Q4. What is a dead-letter queue?

**Answer:** A DLQ stores messages that cannot be processed after a
configured retry policy or because they are permanently invalid. It
prevents poison messages from cycling indefinitely. Monitor count and
age, investigate the cause, and replay only after correcting the problem
and verifying idempotency.

### Q5. How would you design a reliable Magento-to-SAP order integration?

**Answer:** Commit the order and an outbox event atomically; publish
through a durable broker with confirms; process using a stable order
ID/idempotency key; retry transient failures with bounded backoff and
jitter; reconcile ambiguous timeouts; move exhausted messages to a DLQ;
and monitor queue age, retries, SAP errors, and duplicate detection.

## Lesson 16: Distributed Locks, Leader Election, and Coordination

### Q1. What is a distributed lock?

**Answer:** It coordinates exclusive access to a shared task or resource
across multiple processes or machines. Unlike an in-process mutex, it
requires a shared coordination mechanism such as Redis, a database, or a
coordination service.

### Q2. How can a Redis lock be acquired and released safely?

**Answer:** A common acquisition is
`SET lock-key unique-token NX PX ttl`. Release must atomically check
that the stored token matches before deleting the key. Otherwise an old
owner could delete a newer owner's lock after its own lease expires.

### Q3. What happens if a lock expires while the worker is still running?

**Answer:** Another worker may acquire the lock while the original
worker continues, causing overlapping work. Use suitable leases/renewal,
make the operation idempotent, and use fencing tokens or version checks
enforced by the protected resource to reject stale writes.

### Q4. What is leader election, and how is it different from a lock?

**Answer:** A lock grants temporary exclusive access to a resource or
task. Leader election chooses one process to act as coordinator over a
period, with a replacement elected after failure. Both require
protection against stale leaders and split-brain behavior.

## Lesson 17: Fault Tolerance and Disaster Recovery

### Q1. Define RTO and RPO.

**Answer:** RTO is the target time to restore service. RPO is the target
maximum amount of data loss, measured in time. A 60-minute RTO and
5-minute RPO mean the recovery plan targets service restoration within
an hour and data recovery to within five minutes of the failure.

### Q2. Why are backups and replication both needed?

**Answer:** Replication can support failover, but it may also replicate
corruption or accidental deletion. Backups preserve earlier recovery
points. Point-in-time recovery can combine a backup with complete
transaction logs to restore to a selected time.

### Q3. How would you design disaster recovery for Magento?

**Answer:** Define business-approved RTO/RPO; maintain tested database
backups and transaction logs; copy media and configuration to
independent storage; replicate data as appropriate; automate
infrastructure recovery and traffic switching; and test checkout,
payment, inventory, queues, search, and integrations after restoration.

### Q4. What should you consider before promoting a lagging database replica?

**Answer:** Compare its lag with the RPO and business impact. Promotion
may lose recent writes. Prevent the old primary from accepting writes,
validate the new primary, route traffic carefully, and reconcile missing
or ambiguous transactions. Never assume a replica is current without
measuring it.

## Lesson 18: Quorum Reads/Writes and Advanced Consistency

### Q1. What do N, R, and W mean in quorum systems?

**Answer:** `N` is the number of replicas, `R` is the number consulted
for a read, and `W` is the number that must acknowledge a write. In the
common quorum model, `R + W > N` ensures read/write quorum overlap;
`2W > N` ensures write/write overlap.

### Q2. Does quorum overlap guarantee linearizability?

**Answer:** No. Overlap alone does not guarantee that the latest value
is returned or that concurrent writes are correctly ordered. The
database's read/write protocol, version resolution, leader behavior, and
conflict handling determine the actual consistency guarantees.

### Q3. What are the trade-offs of larger quorums?

**Answer:** Larger quorums can improve overlap and consistency
properties but increase latency and make operations fail when too few
replicas are available. Select settings based on the database's
documented semantics and the application's consistency requirements.

### Q4. How would you prevent overselling in a replicated inventory system?

**Answer:** Do not rely only on reading a quorum or cached quantity. Use
atomic conditional updates or a reservation mechanism enforced by the
authoritative inventory store, idempotent order handling, and
reconciliation. Quorum settings are not a replacement for business
invariants.

# Day 6 --- Networking, APIs, and Security

## Lesson 19: DNS, TCP, TLS, and HTTP

### Q1. What happens when a user opens an HTTPS website?

**Answer:** The client resolves the domain through DNS, establishes or
reuses a transport connection, negotiates TLS as needed, sends an HTTP
request, and receives a response. The request may pass through a CDN,
load balancer, application, cache, and database. Caches and connection
reuse can skip repeated setup work.

### Q2. What is the difference between TCP and UDP?

**Answer:** TCP provides a reliable, ordered byte stream. UDP is
connectionless and does not provide built-in delivery or ordering
guarantees. HTTP/1.1 and HTTP/2 commonly use TCP; HTTP/3 uses QUIC over
UDP.

### Q3. Why can't TCP tell the client whether an order was created?

**Answer:** TCP handles transport, not business transaction state. The
server can commit an order and the response can be lost afterward. The
client then cannot infer the outcome from the connection alone; use
idempotency keys and order-status reconciliation.

### Q4. What does TLS provide, and what does it not provide?

**Answer:** TLS provides encryption in transit, integrity, and server
authentication (and client authentication when configured, such as
mTLS). It does not fix application authorization errors, insecure
business logic, or compromised endpoints.

### Q5. Why might DNS changes not be immediate?

**Answer:** Recursive resolvers, operating systems, and browsers may
cache DNS answers for their TTLs, and some resolvers may behave
differently. Lowering TTL ahead of a planned migration can help, but it
does not instantly clear existing caches.

## Lesson 20: API Design --- REST, GraphQL, Versioning, and Pagination

### Q1. Compare REST and GraphQL.

**Answer:** REST commonly exposes resource-oriented endpoints with
predefined representations and works naturally with HTTP caching.
GraphQL lets clients select fields and related data in a query, which
can reduce over-fetching or round trips. GraphQL needs controls for
query cost, depth, authorization, and N+1 queries; neither approach is
always faster.

### Q2. How do offset and cursor pagination differ?

**Answer:** Offset pagination is simple and supports arbitrary page
numbers, but large offsets can be expensive and changing data can cause
duplicates or skips. Cursor pagination continues from a stable indexed
position and is often more efficient for large feeds, but arbitrary page
jumps are harder.

### Q3. How should APIs be versioned?

**Answer:** Prefer backward-compatible changes when possible. Use a
documented strategy such as URL or header versioning when breaking
changes are needed. Provide deprecation notices, migration guidance,
usage monitoring, and a retirement plan.

### Q4. How do idempotency keys make POST requests safer?

**Answer:** The client sends a stable key for one logical operation. The
server durably records the key, request identity/hash, and outcome
atomically with the business operation. A repeated key with the same
request returns the original result; reuse with a different payload
should be rejected.

### Q5. What makes a good API error response?

**Answer:** It provides a stable machine-readable error code, a safe
human-readable message, relevant validation details, and a
correlation/request ID. It must not leak secrets or internal stack
traces. Status codes should match the failure category.

## Lesson 21: Authentication, Authorization, and API Security

### Q1. What is the difference between authentication and authorization?

**Answer:** Authentication verifies identity; authorization determines
which actions and resources that identity may access. A valid token does
not automatically grant access to every endpoint or record.

### Q2. What should a server validate in a JWT?

**Answer:** Verify the signature using an allowlisted algorithm, then
validate required claims such as issuer, audience, expiry, and
not-before where used. Check scopes and authorization server-side. A
signed JWT is not necessarily encrypted, so do not put secrets in its
payload.

### Q3. Compare RBAC and ABAC.

**Answer:** RBAC grants permissions through roles, such as Catalog
Manager or Support Agent. ABAC evaluates attributes such as tenant,
resource, action, ownership, and context. Multi-tenant systems often
combine role permissions with tenant and resource checks.

### Q4. How do you secure a multi-tenant Magento MCP gateway?

**Answer:** Authenticate the caller, validate token claims, map the
identity to a tenant, and enforce tenant and resource authorization on
every tool invocation. Isolate store credentials, cache keys, and
authorization decisions by tenant; audit sensitive actions; and require
approval for high-impact operations.

### Q5. What additional security controls should an API have?

**Answer:** HTTPS, least-privilege access, input validation, rate and
request-size limits, secure secret storage, audit logging, and
protections against injection, CSRF where applicable, and SSRF. Redact
tokens and sensitive customer data from logs.

# Day 7 --- Scalability, Observability, and Reliability

## Lesson 22: Scalability and Capacity Planning

### Q1. Compare vertical and horizontal scaling.

**Answer:** Vertical scaling adds resources to one machine; it is
simpler but limited by hardware and may retain a single failure point.
Horizontal scaling adds instances and capacity but requires load
balancing, shared-state handling, coordination, and failure-aware
design.

### Q2. How do you estimate required application servers?

**Answer:** Divide measured peak throughput by the sustainable
per-server throughput at the target latency, then add headroom for
bursts, deployments, and instance failures. This is only an initial
estimate; validate it with realistic load tests and dependency
monitoring.

### Q3. Why can adding workers make a system slower?

**Answer:** More workers may overload a shared database or external
dependency, increase contention, or trigger throttling and retries.
Scale based on end-to-end throughput and downstream capacity, with
concurrency limits and backpressure.

### Q4. How do you identify the true bottleneck?

**Answer:** Measure latency, throughput, saturation, queue age, and
dependency timings. Inspect PHP-FPM saturation, database locks and slow
queries, cache behavior, search latency, and downstream API errors.
Scale or optimize the limiting component rather than blindly adding
application servers.

## Lesson 23: Observability --- Logs, Metrics, Tracing, SLI/SLO/SLA

### Q1. Explain logs, metrics, and traces.

**Answer:** Logs record discrete events; metrics summarize numeric
behavior over time; traces follow a request across services and show
where time is spent. Together they help detect, localize, and diagnose
incidents.

### Q2. Define SLI, SLO, and SLA.

**Answer:** An SLI is a measured indicator, such as checkout success
rate. An SLO is a target for that indicator, such as 99.9% over a
defined period. An SLA is an external commitment, often contractual,
with terms and possible remedies.

### Q3. Why monitor p95 and p99 latency?

**Answer:** They expose slow-tail behavior that averages hide. A high
p99 may indicate queueing, lock contention, slow queries, or dependency
delays even if most requests are fast.

### Q4. What would you monitor for a Magento production system?

**Answer:** Customer-facing success rate and latency; HTTP errors;
PHP-FPM workers and queueing; MySQL query/lock/connection health; Redis
latency and evictions; OpenSearch health and query latency; RabbitMQ
depth and oldest-message age; and SAP errors, timeouts, and retries.

### Q5. How should alerts be designed?

**Answer:** Alert on actionable customer impact or imminent risk, such
as checkout SLO violations, growing queue age, exhausted PHP workers, or
critical disk pressure. Pair alerts with runbooks, avoid sensitive data
in telemetry, and reduce noisy alerts that do not require action.

## Lesson 24: Reliability, Failover, and Disaster Recovery

### Q1. Compare active-active and active-passive architectures.

**Answer:** Active-active sites serve traffic simultaneously, improving
capacity but complicating data consistency and writes. Active-passive
uses a standby that takes over after failure, which can be simpler but
may have slower recovery and unused capacity. Choose based on RTO/RPO,
write patterns, cost, and operational capability.

### Q2. What is split brain?

**Answer:** Split brain occurs when two nodes or sites both believe they
are the authoritative writer. This can create divergent data. Use
leader/fencing mechanisms, reliable quorum/coordination, and ensure the
old primary cannot accept writes before promoting a new one.

### Q3. What is the difference between replication and backup?

**Answer:** Replication maintains another copy for availability or
scaling, but may replicate accidental deletion or corruption. Backups
preserve recovery points. Tested backups and transaction logs are
essential for point-in-time recovery.

### Q4. What should happen during disaster recovery?

**Answer:** Detect and declare the incident, contain risk, fence the old
writer, promote or restore the recovery environment, switch traffic,
validate data and dependencies, and reconcile orders and queued events.
Measure actual recovery time and data loss against RTO/RPO.

### Q5. How do you prevent cascading failures?

**Answer:** Use timeouts, bounded retries with jitter, circuit breakers,
bulkheads, rate limits, backpressure, and graceful degradation. Protect
critical business invariants and avoid retrying non-idempotent
operations without safe deduplication.

# Cross-Lesson Scenario Questions

## Q1. SAP creates an order, but Magento receives a timeout. What do you do?

**Answer:** Treat the result as unknown, not as a definite failure.
Query SAP by stable order ID or idempotency key. If retrying is
necessary, use the same key and ensure SAP atomically enforces
uniqueness. Record the outcome and reconcile any mismatch.

## Q2. All workers retry every second after SAP recovers. What is the issue?

**Answer:** This is a retry storm or thundering herd. Use exponential
backoff with jitter, bounded retries, circuit breaking, rate limits, and
concurrency limits. Respect `Retry-After` where supplied.

## Q3. RabbitMQ queue depth is growing. What should you investigate?

**Answer:** Compare message arrival rate with consumer completion rate;
inspect oldest-message age, worker errors, retry counts, dependency
latency, and consumer capacity. Scale workers only if downstream systems
can handle more traffic. Use backpressure and a DLQ for poison messages.

## Q4. Cache hit ratio drops sharply after a deployment. What could be wrong?

**Answer:** The deployment may have changed cache keys, invalidated many
entries, altered TTLs, or introduced new response variants. Check cache
metrics and origin load, verify key dimensions and headers, and use
selective invalidation, prewarming, or request coalescing where
appropriate.

## Q5. The website is slow but CPU usage is low. How do you investigate?

**Answer:** Look for waiting rather than computation: slow database
queries, lock waits, saturated connection pools, external API latency,
PHP-FPM queueing, network delays, and growing message queues. Use traces
to find the slow span and logs to inspect the failure context.

## Q6. A replica returns stale inventory. What should you do?

**Answer:** Understand the database's consistency guarantees and
replication lag. Route correctness-sensitive reads appropriately and
enforce stock reservations through atomic updates or an authoritative
inventory service. Do not rely only on a stale catalog value to accept
an order.

## Q7. A Redis distributed lock expires while a job is running. How do you prevent stale writes?

**Answer:** The old worker may continue after another worker acquires
the lock. Use safe lease renewal where supported, fencing tokens or
version checks enforced by the protected resource, and idempotent
operations. A lock alone is not enough.

## Q8. A region fails and the standby database is behind the RPO. What is the decision?

**Answer:** Quantify the potential data loss and business impact. Do not
claim the RPO is met if it is not. Follow the recovery policy, fence the
old primary, communicate the trade-off, restore service safely, and
reconcile missing transactions from logs or downstream systems where
possible.

## Q9. How would you design a URL shortener?

**Answer:** Provide an API to create links and a redirect endpoint.
Store a unique short code mapped to the original URL in a durable
database, cache hot mappings in Redis, use indexed lookups, rate-limit
creation, validate destinations, and record analytics asynchronously.
Use a unique constraint and collision handling for generated codes.

## Q10. How do you design a resilient Magento-to-SAP integration?

**Answer:** Write the order and outbox event in one database
transaction; publish to a durable queue; use idempotent consumers and
stable order IDs; apply bounded retries with backoff and jitter;
reconcile ambiguous timeouts; send exhausted messages to a monitored
DLQ; and instrument queue age, error rates, latency, and duplicate
detection.

# Formula and Pattern Quick Reference

-   Average RPS = daily requests / 86,400.
-   Peak RPS ≈ average RPS × measured peak factor.
-   Little's Law: `L = λ × W`.
-   Availability downtime budget = period duration ×
    `(1 − availability target)`.
-   Quorum overlap: `R + W > N`; write/write overlap: `2W > N` (these do
    not alone prove linearizability).
-   Base62 codes of length `k` have `62^k` possible combinations.
-   Minimum worker estimate = arrival rate / sustainable throughput per
    worker; add headroom and load-test.
-   Safe retry pattern = bounded attempts + exponential backoff +
    jitter + idempotency + reconciliation.
-   Safe recovery = RTO/RPO targets + tested backups + fencing +
    validation + reconciliation.

# Interview Answer Structure

1.  Clarify requirements and assumptions.
2.  Estimate traffic, data, and concurrency.
3.  Define APIs and data model.
4.  Draw the high-level architecture.
5.  Explain the critical success and failure paths.
6.  Identify bottlenecks and scaling strategy.
7.  Compare trade-offs and limitations.
8.  Cover security and tenant isolation.
9.  Define observability, SLIs/SLOs, and alerts.
10. Explain recovery, RTO/RPO, and reconciliation.

## Final principles to remember

-   A timeout does not mean an operation failed.
-   A queue, lock, or quorum does not automatically guarantee
    exactly-once business effects.
-   Replication is not a backup.
-   Cache is not the source of truth for critical invariants.
-   More workers can overload a downstream dependency.
-   Quorum overlap alone does not guarantee linearizability.
-   Prefer measured, incremental complexity over premature architecture.
