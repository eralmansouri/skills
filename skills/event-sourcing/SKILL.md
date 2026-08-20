---
name: event-sourcing
description: Use for event sourcing, event streams, event-sourced aggregates, business intent events, projections, replay, snapshots, optimistic stream concurrency, event schema evolution, and deciding whether event sourcing fits.
---

# Event Sourcing

Use ordered, immutable business facts as the source of truth only when history,
intent, temporal reconstruction, auditability, or future reinterpretation has
enough value to repay permanent schema and operational costs.

## Qualify The Use Case

- Does the business care how and why state reached its current value?
- Are transitions meaningful domain facts rather than generic property changes?
- Is temporal reconstruction or first-class audit valuable?
- Do commands depend on prior state and enforce meaningful invariants?
- Does the process have a coherent lifecycle and stream boundary?
- Can the team own replay compatibility, projections, concurrency, and lag?
- Can event sourcing remain local to the capability that needs it?

Prefer current-state persistence for form-centric CRUD, generic created/updated/
deleted streams, defensive logs that will not be interpreted, or proposals
driven only by scale, Kafka, CQRS, DDD, EDA, or microservices.

## Concept Separation

- CQRS separates command and query code paths.
- Event sourcing persists ordered facts instead of only current state.
- DDD models language, rules, workflows, and boundaries.
- EDA coordinates reactions through messages.
- Microservices apply independent deployment topology to logical capability boundaries.

These patterns often compose but imply none of one another.

## Modeling Workflow

1. Select one business capability.
2. Map its workflow, decisions, invariants, lifecycle, and checkpoints.
3. Design task-oriented commands that capture intent.
4. Name events as past-tense business facts with enough data and metadata to replay and explain the transition.
5. Choose stream identity by consistency unit and lifecycle, not entity habit alone.
6. Load events, replay state, invoke behavior, record new events, and append with the expected version.
7. Design caller-specific projections and choose inline, live, or asynchronous consistency.
8. Separate private source events from stable integration contracts.
9. Test replay, decisions, projections, concurrency, and historical event versions.
10. Add snapshots only after measuring replay cost and revisiting stream design.

## Write Path And Aggregate Rules

1. Read the stream and retain its version.
2. Apply historical events without marking them uncommitted.
3. Invoke a command against reconstructed state.
4. Reject invalid behavior without producing events.
5. Apply and record each new event immediately.
6. Append only if the expected stream version still matches.
7. Publish integration events through an explicit reliable boundary.

- Use an event-sourced aggregate as a write-side decision model.
- Derive only the state required for future decisions.
- Persist uncommitted events, not a current-state object graph.
- Enforce immediate invariants from aggregate state, not a lagging projection.
- Retry version conflicts only when repeating the command is semantically safe.

## Projection Timing

| Mode | Use when | Tradeoff |
| --- | --- | --- |
| Inline | Event and read state must commit together | Higher write latency and coupling |
| Live/on-demand | Historical or occasional calculation | Replay cost at query time |
| Asynchronous | Independent scaling or expensive aggregation | Lag, idempotency, rebuild, and UX complexity |

Build narrow models for screens, reports, totals, and lookup needs. Maintain
ordinary query models incrementally, track checkpoints, make at-least-once
projectors idempotent, and expose asynchronous lag honestly. Prefer
single-stream projections where practical and asynchronous cross-stream
aggregation when it avoids contention.

## Snapshots And Stream Boundaries

First look for natural lifecycle boundaries such as accounting periods, stock
counts, closure, or driver-day assignments. Starting a new stream at a domain
checkpoint is modeling; a snapshot is merely a replay optimization.

If measured replay remains expensive, persist aggregate state at a stream
version and replay subsequent events. Keep original events authoritative and
budget for snapshot schema evolution.

## Evolution And Testing

- Prefer additive optional fields for shared and external contracts, then verify serializer and consumer compatibility.
- Upcast old events when current code requires a newer in-memory shape.
- Create a new event type when semantics change materially.
- Do not append both old and replacement events for one fact in the source stream.
- Ensure old and new application versions can read events written during rolling deployment.
- Rebuild projections when their logic changes.
- For fundamental changes, migrate side by side and catch up before switching writes.
- Given historical events, when a command executes, assert new events or rejection.
- Test each event application and verify full replay equals incremental projection results.
- Keep fixtures for historical event versions and test stale expected versions.

## Operations

- Monitor stream and projection position, lag, queue length, throughput, and processing time.
- Detect poison events and provide explicit repair or skip procedures.
- Make asynchronous projectors idempotent and ordered within a stream.
- Maintain replay and zero-downtime projection rebuild procedures.
- Surface projection freshness in user flows that depend on it.
- Record version, order, timestamp, correlation, and causation metadata.
- Test representative production events against new deployments.
- Perform a separate security and privacy review for immutable history, backup, recovery, retention, deletion, replay, and sensitive fields.

## Avoid

- CRUD sourcing and property-change events without business meaning.
- Applying event sourcing to every table or service.
- Infinite streams caused by missing lifecycle boundaries.
- Snapshots used to conceal bad stream design.
- Querying raw streams for routine UI needs.
- Treating private event-store records as public integration APIs.
- Retrying every optimistic-concurrency conflict automatically.
- Ignoring projection duplicates, poison events, lag, and rebuild time.
