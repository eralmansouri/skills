---
name: persistence-design
description: Use for database and persistence design, data ownership, transaction boundaries, repositories, CQRS read models, query performance, optimistic concurrency, caching, projections, schema migrations, partitioning, and multi-tenancy.
---

# Persistence Design

Start with business capability, ownership, and required consistency. Database
technology, repositories, schemas, caches, and messages follow those decisions.

## Decision Workflow

1. Classify the operation as query, command, simple CRUD, or a business workflow.
2. For a command, identify the data required to decide and whether it can change concurrently.
3. Choose the smallest consistency boundary that can enforce the rule reliably.
4. Design the read path independently around the caller's required shape.
5. Define acceptable staleness and whether read-your-own-write is required.
6. Estimate production growth, result bounds, lifecycle, and partitioning needs.
7. Choose tenant isolation based on the resource and risk being isolated, then enforce tenant context through every data path.
8. Plan backward-compatible schema and application evolution.

## Consistency Choices

| Need | Mechanism |
| --- | --- |
| Atomic local decision and mutation | Database transaction with required isolation |
| Aggregate invariant | Load, decide, and save the aggregate as a unit |
| Infrequent write conflict | Optimistic version or ETag |
| Short serialized database operation | Pessimistic row lock |
| Finite resource in a long workflow | Reserve, confirm, and expire |
| State plus outgoing message | Transactional outbox |
| Multiple independent owners | Explicit eventual workflow or redesigned ownership |

A transaction alone is not enough; isolation must cover the reads and writes
that determine correctness. Do not hold database transactions open through a
long-running business process.

## Write Guidance

- Use direct CRUD for simple supporting data without meaningful workflows.
- Use task-oriented commands when broad updates hide business intent.
- Use an aggregate when behavior needs a coherent consistency boundary.
- Keep aggregate repositories narrow: load by identity and save.
- Use use-case-specific access for queries rather than a generic repository.
- Keep decision reads and writes in one transaction with the required isolation level.
- Use optimistic versions or ETags to reject stale mutations.
- Use reservations for finite resources across long-running workflows.
- Use an outbox for atomic state change and publication intent.
- Reconsider ownership before accepting distributed consistency complexity.

## Read Guidance

- CQRS means separate command and query code paths, not separate databases.
- Fetch only the shape the caller needs.
- Use direct SQL, ORM projection, views, documents, or projections per use case.
- Use runtime composition when freshness matters and fan-out is acceptable.
- Use precomputation when repeated composition is expensive and bounded staleness is acceptable.
- Treat replicated data as a purpose-specific projection or cache, not co-authority.
- Make asynchronous projections replayable or reconcilable; missed updates make them incorrect, not merely stale.

## Query Checklist

- Is every result bounded by ID, time, limit, or pagination?
- What is its maximum size after years of growth?
- Does it load children or columns the caller never uses?
- Are indexes based on actual query plans and production distributions?
- Would a purpose-specific projection avoid repeated joins or cross-service calls?
- Could precomputation remain in the same database before adding infrastructure?
- Is data volatile enough that runtime composition is preferable?
- Is there a natural lifecycle, time, tenant, or business partition?
- Can finalized data move from hot to warm or archival storage?
- Can the database absorb fallback load when a cache is unavailable?
- Can every mutation path invalidate or update the cache?
- Can projections be replayed or reconciled after a missed update?

## Caching, Migrations, And Tenancy

Optimize queries first and consider a read replica or purpose-built read model.
If caching remains justified, define its update strategy, TTL, stale behavior,
ownership, and database fallback load. Cache useful transformations rather than
blindly mirroring rows.

For schema changes, expand with compatible structures, deploy code that handles
old and new shapes, backfill, tighten constraints, then remove obsolete
structures in a later release. Ensure old and new application versions can
coexist during rolling deployment.

Choose shared, pooled, dedicated, or hybrid tenant storage after identifying
the isolation target: data leakage, noisy neighbors, deployment, failure radius,
compliance, residency, or dedicated capacity. Storage topology does not enforce
tenancy by itself. Establish trusted tenant context at entry points, carry it
through queries, writes, messages, background work, caches, and projections,
and test that cross-tenant access fails.

## Avoid

- A generic repository for aggregates, arbitrary filtering, lists, and projections.
- Abstracting a database solely to claim it can be swapped later.
- Shared-database access without schema ownership.
- One service per table or entity.
- Assuming synchronous RPC provides consistency.
- Copying whole authoritative tables to consumers.
- Publishing raw CDC schemas as public contracts.
- Adding a cache before measuring and optimizing queries.
- Breaking schemas before old application versions stop using them.
- Event sourcing used as a CRUD log or generic scaling strategy.

## Ownership Review

- Which capability makes each write decision?
- Does it own the data required for that decision?
- Can another component bypass it and write the same state?
- Is copied data clearly a projection with a freshness contract?
- Is a cross-service call being mistaken for a consistency guarantee?
- Are domain, persistence, query, and public contract shapes separated only where useful?
