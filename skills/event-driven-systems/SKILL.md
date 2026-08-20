---
name: event-driven-systems
description: Use for event-driven or message-driven architecture, commands and events, queues, pub/sub, Kafka, choreography, orchestration, sagas, outbox, idempotency, retries, ordering, backpressure, consumer lag, and messaging operations.
---

# Event-Driven Systems

Use messaging to solve explicit workflow, temporal-decoupling, buffering, or
integration needs. It changes coupling rather than removing it and introduces
delivery, consistency, failure, and operational responsibilities.

## Design Workflow

1. Establish capability boundaries and ownership before choosing messages or transport.
2. Map business intent, facts, handoffs, decisions, deadlines, and failure outcomes.
3. Use synchronous request-response for an immediate answer, a command for directed intent, and an event for an independently consumable fact.
4. Decide whether temporal decoupling is valuable enough to change completed into accepted for later processing.
5. Use choreography while the workflow remains understandable; use orchestration or a stateful policy when progress and compensation must be explicit.
6. Define delivery, duplicate, ordering, timeout, retry, dead-letter, and reconciliation behavior.
7. Choose queue, pub/sub, or retained log by semantics rather than product preference.
8. Design external messages as stable APIs and keep internal events private by default.
9. Design tracing, lag monitoring, outbox operations, dead-letter replay, and capacity before rollout.
10. Choose deployment topology last; messaging does not require microservices.

## Message Semantics

- Command: imperative intent directed to one logical owning consumer.
- Event: past-tense fact, one logical publisher, zero or more consumers.
- Queue: distributes directed work to one logical consumer across competing instances.
- Pub/sub: gives each subscription or consumer group an independent copy.
- Retained log: retains records and tracks offsets; partitioning controls ordering and parallelism.
- Event-sourcing event, notification event, and state-transfer event are different concepts.

## Workflow Choice

Choreography lets boundaries react to facts without a central director, but the
end-to-end process becomes implicit. Use it for small, comprehensible handoffs.

An orchestrator persists progress, receives outcomes, and sends commands for
the next step. It adds explicit participant coupling but improves visibility,
deadlines, and compensation. A policy can record prerequisite events and act
once all are present, removing unnecessary arrival-order dependencies.

## Reliability Patterns

| Need | Pattern | Critical detail |
| --- | --- | --- |
| Atomic state and publication | Transactional outbox | Relay can publish duplicates |
| Duplicate internal work | Inbox/deduplication | Record ID and state change in one transaction |
| External side effect | Idempotency key and reconciliation | Local inbox cannot prevent duplicate remote effects |
| Unknown event order | Stateful policy | Record prerequisites and trigger once |
| Required per-key order | Partition/message group | Ordering limits concurrency |
| Complex workflow | Orchestrator/saga | Persist progress and model compensation |
| Future action | Delayed message | Recheck current state on delivery |
| Large payload | Claim check | Secure shared payload storage |
| New projection | Replay, snapshot, or lazy seed | Question broad data copying |

## Failure Responses

| Failure | Response |
| --- | --- |
| State commits but publication fails | Outbox |
| Acknowledgement is lost | Idempotent consumer or inbox |
| Remote request times out | Query by reference or reconcile |
| Poison message repeats | Bounded retries, inspectable DLQ, controlled replay |
| Slow dependency occupies workers | Timeouts, isolation, circuit/fail-fast where appropriate |
| Production exceeds consumption | Optimize, scale consumers, reduce inflow, or bound capacity |
| Hot partition | Revisit partition key; extra consumers cannot help that partition |
| Contract breaks consumers | Compatible evolution or a new version/type |
| Read model lags after a write | Primary read, wait/poll by version, or push completion |

Assume at-least-once delivery. Make outcomes idempotent, expect outbox relay
duplicates, prefer order-independent policies, bound retries by failure type,
and treat timeouts as uncertainty rather than proof of failure.

## Contracts And Operations

External events are public APIs. Prefer additive changes, but verify serializer,
schema-registry, and consumer behavior because unknown fields are not universally
safe. Use replacement events for semantic changes and translate from internal
state. Thin events work when consumers own what they need. State-carrying events
are valid when data distribution is the purpose, with explicit staleness,
seeding, and schema-coupling costs.

- Propagate trace and correlation context across producers, brokers, consumers, databases, and external calls.
- Track production and completion rate, queue depth, oldest age, lead time, processing duration, and lag by partition or group.
- Track retries, timeout categories, circuit state, dead-letter moves, and fallback usage.
- Monitor outbox age, unpublished rows, relay failures, and throughput.
- Expose workflow progress, deadlines, compensation, completion, and reconciliation state.
- Capacity-test databases and third parties before adding consumers.
- Provide inspectable messages and controlled replay with clear ownership.
- Perform a separate security and privacy review for sensitive fields, access controls, encryption, retention, deletion, dead letters, claim checks, and replay.

## Avoid

- Making every interaction asynchronous or calling every message an event.
- Using Kafka as a command queue without accepting its log semantics.
- CRUD over pub/sub with generic created, updated, and deleted events.
- Thin events followed by synchronous callbacks for required data.
- Fat events expanded for every consumer.
- Assuming FIFO delivery means handlers finish in order.
- Believing inbox deduplication makes external effects exactly once.
- Blanket retries and circuit breakers without modeled failure behavior.
- Treating a dead-letter queue as a solution without inspection and replay tooling.
- Using sagas as the source of truth for external systems.
