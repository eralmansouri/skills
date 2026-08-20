---
name: service-boundaries
description: Use for service boundaries, bounded capabilities, modular monoliths, microservices, monolith decomposition, data ownership, shared databases, API coupling, anti-corruption layers, and architecture topology decisions.
---

# Service Boundaries

Design logical ownership before physical topology. A service is the authority
over a cohesive set of business capabilities and the data needed to perform
them. Repositories, containers, processes, and databases do not prove that a
useful boundary exists.

## Decision Workflow

1. Name the measurable outcome: release autonomy, failure isolation, security, throughput, team ownership, or cost.
2. Map business capabilities, workflows, language, rules, state transitions, and authoritative decisions.
3. Identify candidate boundaries by cohesion, lifecycle, ownership, and reasons to change.
4. Verify that commands can usually be decided with locally owned data.
5. Map dependencies in both directions and classify their coupling.
6. Test whether a monolith plus ordinary scaling solves the stated constraint.
7. Price distribution: latency, partial failure, contracts, deployment, observability, consistency, and support.
8. Choose monolith, modular monolith, services, or a hybrid per capability.
9. Extract only a boundary with a concrete beneficiary and sufficiently stable ownership.
10. Record the context, alternatives, consequences, and triggers for reconsideration.

## Boundary Rules

- Start with behavior and business decisions, not tables or entity nouns.
- One logical owner controls each state transition and authoritative write.
- Ownership includes meaning, rules, lifecycle, behavior, and supporting data.
- The same identity may have different purpose-specific models in different boundaries.
- Keep logical and physical boundaries separate; several modules may share a deployment or database instance.
- Prevent modules from reading or writing one another's internal schemas by default.
- Use public APIs and events as translated contracts, not serialized domain or persistence models.
- Use an anti-corruption layer when external semantics differ from the receiving context.
- Treat all communication styles as forms of coupling with different costs.
- Prefer incremental, reversible extraction over wholesale decomposition.

## Validate A Candidate Boundary

A strong boundary:

- Owns cohesive behavior with a narrow business reason to change.
- Owns its rules, lifecycle, state transitions, and authoritative writes.
- Can decide most commands using local data.
- Exposes explicit contracts rather than its internal schema.
- Can evolve without coordinated changes across many consumers.
- Participates in wider workflows through explicit handoffs.
- Can change deployment topology without redefining its purpose.

Repeated synchronous lookups, shared mutable entities, growing event payloads,
cross-boundary transactions, and coordinated releases are warning signals.

## Topology Heuristic

| Situation | Default |
| --- | --- |
| Small team, uncertain domain, frequent boundary changes | Monolith |
| Clear capability ownership without proven deployment need | Modular monolith |
| Proven independent release, scale, isolation, or ownership need | Independently deployed service |
| Different needs by capability | Hybrid |

## Communication Choices

| Mechanism | Use when | Main cost |
| --- | --- | --- |
| In-process call | Same deployment and immediate result | Technology and behavioral coupling |
| HTTP or gRPC | Immediate answer or external integration | Temporal, latency, availability, and contract coupling |
| Queue command | Deferred directed work or buffering | Delivery, idempotency, failure, and operations |
| Pub/sub event | Independent reactions to a fact | Semantic, schema, broker, and publication coupling |
| Shared database | Deliberately coupled local system | Strong schema and change coupling |

No mechanism removes coupling. Choose the form whose failure and evolution
costs match the business requirement.

## Data And Evolution

For a query, decide among client composition, a backend-for-frontend, owner
APIs, or a purpose-built projection. For a command decision, first try to move
the decision and required data into one ownership and transaction boundary.

Treat replicated state as a cache or projection, never co-authority. Define its
freshness, rebuild path, and failure behavior. A synchronous lookup can still be
stale before the caller commits.

Sharing a physical database is acceptable when schemas have clear owners and
permissions enforce access. Separate databases add value for isolation,
scaling, technology, security, backup, or organizational ownership; they are
not a ceremonial requirement.

Evolve incrementally: discover boundaries in the current deployment, establish
module contracts and owner-only writes, select one capability with a proven
deployment benefit, map dependencies, move communication and data gradually,
and protect old and new semantics through translation. Consolidation is also a
valid architectural evolution.

## Avoid

- One service or module per table/entity.
- A universal customer, product, or employee model.
- Cross-schema joins and writes hidden inside a modular monolith.
- HTTP or a broker used to disguise a distributed dependency graph.
- RPC chains across independently deployed services.
- Copying entire authoritative tables into every consumer.
- Coordinated releases among supposedly autonomous services.
- Infrastructure for hypothetical scale without a named beneficiary.

## Review Checklist

- What measurable outcome does this topology serve?
- Are logical and physical boundaries documented separately?
- What capability, rules, and data does each boundary own?
- Can each command be decided with locally authoritative data?
- Who may read and write each schema?
- Which calls create latency or availability chains?
- What consistency does the business actually require?
- What coupling was removed, and what coupling replaced it?
- Are public events and APIs stable translated contracts?
- Can a capability change without coordinated releases?
- Does independent deployment repay its operational cost?
- Is extraction incremental and reversible?
- What evidence would trigger extraction, consolidation, or redesign?
