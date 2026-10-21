# Distributed Systems: Concepts and Interview Questions

**A practical guide to what distributed-systems concepts mean, when to use them, and what tradeoffs they introduce.**

A distributed system runs parts of an application across multiple processes or machines that communicate over a network. This can improve scale and resilience, but introduces latency, partial failure, coordination, and consistency challenges.

Try to answer each question before opening it. For every design choice, ask: What requirement does this solve? What failure mode does it add? How will the system detect and recover from that failure?

## Contents

| Level | Focus |
|---|---|
| [1. Foundations](#level-1-distributed-systems-foundations) | Definition, use cases, latency, and partial failures |
| [2. Scale and Service Goals](#level-2-scale-availability-and-reliability) | Scalability, availability, reliability, CAP, and consistency |
| [3. Data Distribution](#level-3-replication-partitioning-and-caching) | Replication, partitioning, sharding, and caches |
| [4. Traffic and Asynchronous Work](#level-4-load-balancing-messaging-and-events) | Load balancing, queues, event-driven systems, Kafka, and Pub/Sub |
| [5. Resilience Patterns](#level-5-retries-idempotency-and-protection) | Idempotency, retries, backoff, circuit breakers, and rate limiting |
| [6. Distributed Transactions](#level-6-distributed-transactions) | Consistency across services, sagas, and outbox pattern |
| [Reference Tables](#reference-tables) | Pattern selection, design checklist, and example flow |

## Level 1: Distributed Systems Foundations

<details>
<summary>1. What is a distributed system?</summary>

**Answer:** A distributed system is a collection of independent processes or machines that communicate over a network and cooperate to provide a service. To a user it may look like one application, even though requests and data are handled by multiple components.

</details>

<details>
<summary>2. Why would a team use a distributed system?</summary>

**Answer:** Distribution can provide more capacity, isolate workloads, place services closer to users, and improve resilience through redundancy. It also adds operational and design complexity, so use it when requirements justify the cost rather than splitting a system by default.

</details>

<details>
<summary>3. What is a partial failure?</summary>

**Answer:** A partial failure occurs when one component or network path fails while other parts of the system continue working. A caller may time out even though the server completed the operation. Distributed systems must handle uncertainty, not just total up/down states.

</details>

<details>
<summary>4. Why is network communication different from an in-process function call?</summary>

**Answer:** A network call can be delayed, lost, duplicated, rejected, or completed after the caller times out. It also crosses process and security boundaries. Design remote calls with deadlines, bounded retries, authentication, observability, and safe handling of duplicate operations.

</details>

<details>
<summary>5. What is a timeout, and why should every remote call have one?</summary>

**Answer:** A timeout bounds how long a caller waits for a response. Without a deadline, requests can occupy threads or connections indefinitely and resource exhaustion can spread. Choose timeouts based on the operation's latency objective and propagate an overall request deadline where possible.

</details>

<details>
<summary>6. What is a cascading failure?</summary>

**Answer:** A cascading failure occurs when an overloaded or failing component causes dependent components to become overloaded or fail too. Timeouts, load shedding, bounded queues, bulkheads, circuit breakers, and capacity planning can limit propagation.

</details>

## Level 2: Scale, Availability, and Reliability

<details>
<summary>7. What is scalability?</summary>

**Answer:** Scalability is a system's ability to handle increased workload while meeting its performance and cost goals. Vertical scaling adds resources to a machine; horizontal scaling adds more instances. Stateless services are generally easier to scale horizontally, while shared data and coordination can become bottlenecks.

</details>

<details>
<summary>8. What is the difference between vertical and horizontal scaling?</summary>

**Answer:** Vertical scaling increases resources such as CPU or memory on one machine. Horizontal scaling adds machines or service instances. Vertical scaling is often simpler but has hardware and availability limits; horizontal scaling can add capacity and redundancy but requires distribution-aware design.

</details>

<details>
<summary>9. What is availability?</summary>

**Answer:** Availability is the degree to which a service is usable when needed. It is commonly measured over a time period, but the exact definition should include which user-visible operations count as available and what latency or correctness thresholds apply.

</details>

<details>
<summary>10. What is reliability?</summary>

**Answer:** Reliability is the ability of a system to perform its intended function correctly and consistently over time. A service can be reachable but unreliable if it returns incorrect results, loses writes, or frequently violates its latency objective.

</details>

<details>
<summary>11. How are availability and reliability different?</summary>

**Answer:** Availability asks whether the service is usable at a given time. Reliability is broader: it includes correctness and dependable behavior over time. Measuring only whether a process responds can miss incorrect or degraded service.

</details>

<details>
<summary>12. What are an SLI, SLO, and SLA?</summary>

**Answer:** An SLI is a measured indicator of service behavior, such as successful-request rate or latency. An SLO is an internal target for that indicator over a window. An SLA is an agreement with users or customers, often with consequences if commitments are missed.

</details>

<details>
<summary>13. What is the CAP theorem?</summary>

**Answer:** CAP describes a tradeoff that becomes relevant when a network partition prevents nodes from communicating: a distributed data system cannot guarantee both a single consistent view and a successful response from every side of the partition. The choice concerns behavior during a partition; it does not mean a system simply picks any two properties at all times.

</details>

<details>
<summary>14. What does consistency mean in distributed systems?</summary>

**Answer:** Consistency describes what values and ordering clients can observe across replicas or operations. Strong consistency may make acknowledged updates immediately visible according to a defined model; eventual consistency allows replicas to temporarily disagree but converge if updates stop and propagation succeeds.

</details>

<details>
<summary>15. What is eventual consistency, and when might it be acceptable?</summary>

**Answer:** Eventual consistency means replicas may return different values temporarily but are expected to converge. It can suit feeds, analytics, and other workflows where brief staleness is acceptable. It may be unsuitable for decisions that require an immediately authoritative balance, permission, or inventory count.

</details>

<details>
<summary>16. What is a consistency model?</summary>

**Answer:** A consistency model defines the guarantees clients can expect about reads and writes, including visibility and ordering. Examples include eventual consistency and linearizability. Select a model based on product requirements and explain which operations need the stronger guarantee.

</details>

## Level 3: Replication, Partitioning, and Caching

<details>
<summary>17. What is replication?</summary>

**Answer:** Replication stores copies of data on multiple nodes. It can improve read capacity and resilience, but creates replica lag, failover, and consistency questions. Replication is not a backup by itself because accidental deletion or corruption may also be copied.

</details>

<details>
<summary>18. What is the difference between synchronous and asynchronous replication?</summary>

**Answer:** With synchronous replication, a write waits for acknowledgments from one or more replicas before it is considered committed, which can improve durability or read consistency but increases latency and availability dependencies. Asynchronous replication acknowledges earlier but may lose recent changes during some failures and can expose stale reads.

</details>

<details>
<summary>19. What is partitioning?</summary>

**Answer:** Partitioning divides a dataset into smaller pieces based on a rule, such as date range or key range. Partitions may exist within one database or across multiple machines. The partition key and query patterns determine whether operations remain efficient.

</details>

<details>
<summary>20. What is sharding?</summary>

**Answer:** Sharding distributes partitions across separate database nodes or clusters. It can increase storage and write capacity, but makes cross-shard queries, transactions, rebalancing, and operational work more complex.

</details>

<details>
<summary>21. How do partitioning and sharding differ?</summary>

**Answer:** Partitioning is the general act of splitting data into subsets. Sharding usually means distributing those subsets across independent database servers or clusters. A database can partition tables internally without the application operating multiple shards.

</details>

<details>
<summary>22. How do you choose a shard key?</summary>

**Answer:** Choose a key that distributes load and storage evenly while keeping common queries and transactions within a shard when possible. Avoid low-cardinality or highly skewed keys, and plan for growth, rebalancing, and changes to the key strategy.

</details>

<details>
<summary>23. What is a cache?</summary>

**Answer:** A cache stores reusable data closer to the caller or computation to reduce latency and load on the source. Caches can be in-process, on a host, or shared. They introduce freshness, invalidation, memory, and failure-mode concerns.

</details>

<details>
<summary>24. When should you use caching?</summary>

**Answer:** Cache data when it is expensive to compute or fetch, requested often, and can tolerate a defined freshness policy. Measure the bottleneck first and decide what happens on a cache miss or cache outage; caching is not a substitute for fixing an inefficient source query.

</details>

<details>
<summary>25. What are cache-aside and write-through patterns?</summary>

**Answer:** In cache-aside, the application checks the cache, loads from the source on a miss, and then populates the cache. In write-through, writes update the cache and source as part of the write path. Each pattern has different consistency and failure behavior.

</details>

<details>
<summary>26. What is cache invalidation?</summary>

**Answer:** Cache invalidation removes or refreshes entries when source data changes or expires. Common strategies include a time-to-live, explicit invalidation after writes, and versioned keys. Define acceptable staleness and handle failures between updating the source and invalidating the cache.

</details>

<details>
<summary>27. What is a cache stampede?</summary>

**Answer:** A cache stampede occurs when many requests miss or expire the same popular key and all recompute or fetch the value at once. Techniques such as request coalescing, staggered expiration, and controlled refresh can reduce the burst.

</details>

## Level 4: Load Balancing, Messaging, and Events

<details>
<summary>28. What is load balancing?</summary>

**Answer:** Load balancing distributes requests or connections across multiple backends. It can improve capacity and availability, but needs health checks, appropriate timeouts, capacity-aware routing, and a plan for what happens when all backends are unhealthy.

</details>

<details>
<summary>29. What is the difference between layer 4 and layer 7 load balancing?</summary>

**Answer:** Layer 4 balancing routes based on transport-level information such as IP addresses and ports. Layer 7 balancing can use application details such as HTTP host, path, or headers. Layer 7 routing is more flexible but requires understanding application protocols and may add processing overhead.

</details>

<details>
<summary>30. What is a health check?</summary>

**Answer:** A health check reports whether an instance should receive traffic or whether a dependency is functioning. Liveness checks ask whether a process should be restarted; readiness checks ask whether it can serve traffic. Checks should avoid turning a temporary dependency issue into unnecessary restart loops.

</details>

<details>
<summary>31. What is a message queue?</summary>

**Answer:** A message queue lets producers send work that consumers process asynchronously. It can smooth bursts and decouple services, but introduces delivery, ordering, duplication, retry, dead-letter, and queue-backlog considerations.

</details>

<details>
<summary>32. What is event-driven architecture?</summary>

**Answer:** Event-driven architecture uses events to communicate that something happened, allowing producers and consumers to be loosely coupled. It can support independent scaling and asynchronous workflows, but makes tracing, ordering, schema evolution, and eventual consistency more important.

</details>

<details>
<summary>33. What is Apache Kafka?</summary>

**Answer:** Kafka is a distributed event-streaming platform. It stores ordered records in partitioned topics for a configured retention period, and consumer groups can process partitions independently. Kafka is useful for durable streams and replay, but requires attention to partitions, offsets, retention, ordering, and operations.

</details>

<details>
<summary>34. What is Pub/Sub?</summary>

**Answer:** Publish/Subscribe is a messaging pattern where publishers send messages to a topic or channel and subscribers receive matching messages. It decouples publishers from subscribers. Delivery retention, replay, ordering, and acknowledgment guarantees depend on the Pub/Sub product or broker.

</details>

<details>
<summary>35. How do Kafka and Pub/Sub compare?</summary>

**Answer:** Kafka is a specific event-streaming platform with retained, partitioned logs and consumer offsets. Pub/Sub is a general messaging pattern and is also the name of particular products. A Kafka-like log is often chosen for replayable streams; a managed Pub/Sub service may be chosen for simpler event distribution. Compare the actual guarantees and operational needs rather than names alone.

</details>

<details>
<summary>36. What is a consumer group in Kafka?</summary>

**Answer:** A consumer group lets multiple consumers share work from a topic. Partitions are assigned among group members, so a group cannot process more partitions in parallel than the topic has. Rebalancing changes assignments when members join or leave.

</details>

<details>
<summary>37. What is message delivery semantics?</summary>

**Answer:** Delivery semantics describe how often a message may be delivered and when it is considered processed. At-most-once can lose messages; at-least-once can redeliver; exactly-once claims are scoped to a system and workflow. Consumers should generally tolerate duplicates and make side effects safe.

</details>

## Level 5: Retries, Idempotency, and Protection

<details>
<summary>38. What is idempotency?</summary>

**Answer:** An operation is idempotent when repeating it has the same intended effect as performing it once. For example, setting a user's status to `active` is naturally idempotent; charging a payment twice is not. Idempotency keys or deduplication records can make retried requests safe.

</details>

<details>
<summary>39. How do idempotency keys work?</summary>

**Answer:** A client supplies a unique key for one logical operation. The server records the key and result atomically enough to return the same outcome for retries rather than repeating the side effect. Define key scope, expiry, collision behavior, and what happens when the same key is reused with different input.

</details>

<details>
<summary>40. When should a distributed system retry a request?</summary>

**Answer:** Retry only failures likely to be temporary, such as selected timeouts or transient service errors, and only when repeating the operation is safe. Set an attempt limit and an overall deadline. Do not retry permanent validation or authorization failures.

</details>

<details>
<summary>41. What is exponential backoff?</summary>

**Answer:** Exponential backoff increases the wait between retry attempts, commonly multiplying the delay after each failure. Add jitter, or randomized variation, so many clients do not retry at the same instant. Cap the delay and total retry duration.

</details>

<details>
<summary>42. What is a circuit breaker?</summary>

**Answer:** A circuit breaker stops sending calls to a dependency that is failing, giving it time to recover and preventing callers from wasting resources. It commonly has closed, open, and half-open states. Define thresholds, reset behavior, and a safe fallback if one exists.

</details>

<details>
<summary>43. How is a circuit breaker different from a retry?</summary>

**Answer:** A retry repeats an individual operation after a potentially temporary failure. A circuit breaker suppresses calls to a dependency after failures cross a threshold. Unbounded retries can worsen an outage; a breaker can limit pressure while the dependency recovers.

</details>

<details>
<summary>44. What is rate limiting?</summary>

**Answer:** Rate limiting restricts how many requests an identity, client, or resource can make during a period. It protects capacity and helps enforce fair use. Define the limit key, response behavior, burst allowance, and whether limits are per instance or global.

</details>

<details>
<summary>45. What are token bucket and fixed-window rate limits?</summary>

**Answer:** A token bucket replenishes tokens over time and permits a configured burst while controlling the long-term rate. A fixed window counts requests in discrete intervals and is simple, but traffic can spike across a window boundary. The algorithm should match the fairness and burst requirements.

</details>

<details>
<summary>46. What is a bulkhead pattern?</summary>

**Answer:** A bulkhead isolates resources, such as thread pools or connection pools, so one slow dependency or workload cannot consume all capacity. It limits the blast radius of failures, but capacity partitions need monitoring and sensible sizing.

</details>

<details>
<summary>47. Why should retries use a budget?</summary>

**Answer:** A retry budget limits the extra traffic generated by retries over time. Without a budget, retries can amplify load during an incident and create a feedback loop. Coordinate retry limits across clients, services, and queues.

</details>

## Level 6: Distributed Transactions

<details>
<summary>48. What is a distributed transaction?</summary>

**Answer:** A distributed transaction coordinates changes across multiple independent services or data stores. Unlike a local database transaction, it cannot generally rely on one database's atomic commit. Network failure, partial completion, and recovery must be addressed explicitly.

</details>

<details>
<summary>49. What is two-phase commit?</summary>

**Answer:** Two-phase commit coordinates participants through a prepare phase and a commit or abort phase. It can provide atomicity across participating resource managers, but requires coordination and can leave participants waiting if the coordinator fails. It may be unsuitable across independent services with high availability needs.

</details>

<details>
<summary>50. What is the Saga pattern?</summary>

**Answer:** A saga models a long-running business operation as a sequence of local transactions. If a later step fails, the workflow runs compensating actions for completed steps. Compensation is a new business action, not necessarily a literal rollback, so workflows need explicit failure and recovery handling.

</details>

<details>
<summary>51. What is the difference between choreography and orchestration in a Saga?</summary>

**Answer:** In choreography, services react to events and coordinate without a central workflow controller. In orchestration, a coordinator directs the sequence and handles outcomes. Choreography reduces a central dependency but can obscure the overall flow; orchestration makes the workflow explicit but adds a coordinator to operate.

</details>

<details>
<summary>52. What is the transactional outbox pattern?</summary>

**Answer:** The outbox pattern writes a business change and an event record in the same local database transaction. A separate publisher sends outbox events to a broker and marks them as published. Consumers still need to tolerate duplicates, and the publisher needs retry and monitoring.

</details>

<details>
<summary>53. How can a system keep data consistent across services without a global transaction?</summary>

**Answer:** Use local transactions, durable events or commands, idempotent consumers, explicit workflow state, retries with bounds, and compensating actions where the business process allows them. Define which data may be temporarily stale and how failures are reconciled.

</details>

<details>
<summary>54. Why is exactly-once processing difficult in distributed systems?</summary>

**Answer:** A sender may not know whether a receiver committed an operation before a connection failed. Most practical designs combine at-least-once delivery with idempotent processing, deduplication, and atomic state changes. Any exactly-once guarantee is limited to particular components and boundaries.

</details>

## Reference Tables

### Pattern Selection

| Pattern | Use it when | Main tradeoff to plan for |
|---|---|---|
| Horizontal scaling | Work can be spread across instances | Coordination, state sharing, and skew |
| Replication | Reads, resilience, or geographic proximity need copies | Lag, failover, and consistency |
| Partitioning/sharding | Data volume or throughput exceeds one node's practical capacity | Cross-partition queries and rebalancing |
| Caching | Repeated reads are expensive and bounded staleness is acceptable | Invalidation and stale data |
| Load balancing | Multiple healthy instances can serve a workload | Health checks, uneven load, and shared state |
| Message queue | Work can be processed asynchronously or smoothed over bursts | Duplicates, backlog, ordering, and retries |
| Event-driven design | Producers and consumers should evolve independently | Tracing and eventual consistency |
| Retry with backoff | A failure may be temporary and repeating is safe | Extra load and longer completion time |
| Circuit breaker | A dependency is failing and needs protection | Fallback behavior and recovery tuning |
| Rate limiting | Capacity or fair-use policy must be enforced | Rejected or delayed requests |
| Saga/outbox | A business workflow spans services | Compensation, duplicate events, and reconciliation |

### A Practical Design Checklist

| Step | Question |
|---:|---|
| 1 | What are the functional requirements and user-visible outcomes? |
| 2 | What traffic, data volume, latency, and availability targets must be met? |
| 3 | What are the data owners, consistency needs, and failure boundaries? |
| 4 | Which components can scale independently, and where is state stored? |
| 5 | What happens on timeouts, duplicate requests, delayed events, and dependency failure? |
| 6 | How are overload, retries, and rate limits bounded? |
| 7 | How will operators observe, recover, and reconcile partial failures? |
| 8 | Which complexity is justified now, and what can remain a simpler design? |

### Example: Order Processing Flow

1. An API validates the request and uses an idempotency key to avoid creating duplicate orders on safe retries.
2. The order service commits the order and an outbox event in one local database transaction.
3. A publisher delivers the event to a message broker; consumers acknowledge and process it idempotently.
4. Inventory and payment services perform local steps and report outcomes through events.
5. A workflow records progress and applies a business compensation if a later step cannot complete.
6. Monitoring tracks queue lag, failures, retries, and end-to-end order completion against service objectives.

This pattern improves decoupling but does not make the workflow instantly consistent. The product must define which states users can see while processing is in progress.

## Interview Answer Framework

For a distributed-systems question:

1. State the requirement the pattern addresses.
2. Explain how it works with a small example.
3. Describe its failure modes and tradeoffs.
4. Explain how you would observe and recover from failures.
5. Mention a simpler alternative if the added complexity is not yet justified.
