---
name: domain-driven-design
description: Use for domain-driven design, domain modeling, ubiquitous language, bounded contexts, aggregates, invariants, entities, value objects, domain services, domain events, and deciding whether a rich domain model is warranted.
---

# Domain-Driven Design

Model business behavior before choosing tactical patterns. DDD addresses
inherent business complexity: rules, decisions, workflows, language, and
ownership. It is not a required folder structure or a checklist of entities,
repositories, factories, and events.

## Workflow

1. State the business objective, actors, use cases, constraints, and desired outcome.
2. Model the real workflow collaboratively with domain experts. Capture actions, facts, decisions, failures, handoffs, and pain points.
3. Build context-specific ubiquitous language. Record ambiguous terms and different meanings explicitly.
4. Find boundaries through capabilities, ownership, lifecycle, language shifts, consistency needs, and reasons to change.
5. Classify core, supporting, and generic areas. Spend rich-modeling effort where complexity or competitive value warrants it.
6. For each use case, name the command by intent and identify the business facts it can produce.
7. Enumerate invariants and the state required to enforce each one reliably.
8. Choose the smallest implementation: transaction script, database constraint, policy, value object, or aggregate.
9. Keep read composition separate from write-side consistency decisions.
10. Test allowed and rejected transitions, then revise the model as understanding changes.

## Strategic Model

A bounded context owns the meaning, behavior, rules, lifecycle, authority, and
supporting data for a cohesive capability. It is a logical and linguistic
boundary, not inherently a team, repository, process, database, or microservice.

The same real-world identity can have several legitimate models. An employee in
HR and a user in access management, or a product in sales and warehouse, share
identity without sharing all state and behavior.

Strategic DDD can clarify any large system. Rich tactical modeling is useful
only where local behavior is sufficiently complex.

## Choosing A Rule Mechanism

| Situation | Prefer |
| --- | --- |
| Static shape, range, or format | Request validation or a validated type |
| Simple state change within one transaction | Transaction script and database guarantees |
| Rule shared across state transitions | Domain behavior or policy |
| Multiple objects must remain consistent together | Aggregate, if the state is practical to load and transact |
| Very large collection or external state | Transactional query, database constraint, reservation, or redesigned ownership |
| Caller permission | Application authorization |
| Business eligibility of a participant | Domain rule |
| Display and reporting | Purpose-built query/read model |

## Tactical Rules

- Treat an aggregate as a consistency boundary, not an object graph.
- Include only state required to make a write decision and enforce invariants.
- Make the aggregate root the behavioral gateway; do not expose mutable internals.
- Construct domain objects validly and preserve validity through behavior.
- Use a separate draft model when incomplete planning data is a legitimate lifecycle.
- Use value objects for identityless concepts with value equality, valid construction, and relevant behavior.
- Put caller identity, roles, and permissions in the application layer. Keep business-valid participant rules in the domain.
- Use named policies when behavior does not fit naturally on one entity.
- Keep domain, persistence, read, and public contract models distinct when their different purposes justify the mapping cost.
- Keep internal events private by default. Translate stable, consumer-oriented integration events at the boundary.

## Aggregate Questions

- What invariant must always hold?
- Which command can violate it?
- What is the smallest state needed to decide?
- Can that state be loaded and committed atomically?
- Can another caller bypass the root?
- Would a database constraint or transaction be more reliable?
- Is the aggregate being enlarged only for navigation or a screen?

## Tradeoffs

- An anemic data model is acceptable for explicit transaction scripts. It is a problem when complex behavior is scattered while the design claims to be a rich model.
- Persistence ignorance is a spectrum. Separate persistence models when the ORM distorts behavior, not merely to satisfy purity.
- A policy or specification can be a meaningful domain dependency. Loggers, databases, and transport clients are infrastructure dependencies.
- Domain events and integration events may share a shape only when the concept is deliberately stable for both audiences. Separation is the safe default.

## Avoid

- Starting from nouns, tables, or relationships.
- Building one canonical model for a concept used differently across contexts.
- Creating an entity service that owns every behavior involving a noun.
- Calling private setters and property-assignment methods a rich domain model.
- Loading large collections only to enforce a count or membership rule.
- Forcing all validation, authorization, and orchestration into aggregates.
- Applying tactical DDD uniformly to simple supporting CRUD.

## Review Checklist

- Are goals, actors, workflows, constraints, and use cases explicit?
- Did relevant domain experts participate?
- Does each context have current, unambiguous language?
- Are boundaries based on capabilities and ownership rather than entities?
- Is core-domain effort distinguished from supporting CRUD?
- Are commands named by intent and events as facts?
- Is each rule enforced where the necessary data and consistency exist?
- Are aggregate boundaries derived from invariants?
- Can callers mutate aggregate internals?
- Are valid construction and rejected transitions tested?
- Could a simpler transaction or database constraint replace the model?
- Are query needs kept from inflating write models?
- Are public contracts insulated from internal domain and persistence types?
- Can one context evolve without forcing unrelated contexts to change?
