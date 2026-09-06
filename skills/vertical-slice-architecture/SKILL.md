---
name: vertical-slice-architecture
description: Use for vertical slice architecture, feature folders, task-based APIs and UI, CQRS commands and queries, handlers, request pipelines, controller design, code structure, cohesion, and feature-level testing.
---

# Vertical Slice Architecture

Decompose software around business capabilities, workflows, tasks, and explicit
use cases. A vertical slice is a logical unit of change, not a mandatory folder
layout, endpoint style, framework, or deployment. It may span UI, HTTP, workers,
persistence, and message consumers.

## Add Or Change A Feature

1. Name the business intent with an action such as `PlaceOrder` or `DispatchShipment`.
2. Identify the capability that owns the behavior and affected state.
3. Classify the use case as a command, query, or workflow.
4. Define request and response contracts for this use case only.
5. Choose the simplest implementation that enforces its rules.
6. Place input validation, authorization, business rules, and orchestration deliberately.
7. Add pipeline steps only for demonstrated cross-cutting needs.
8. Handle cross-boundary effects through explicit APIs, commands, or events.
9. Co-locate the endpoint, contract, handler, validation, and slice-specific persistence.
10. Test behavior, persistence, and the externally visible contract at useful seams.

## Design Rules

- Organize primarily by feature, not controllers, services, repositories, DTOs, or entities.
- Keep implementation choices local. One slice can use direct SQL while another uses an aggregate.
- Treat CQRS as separate command and query code paths. It does not require messaging, event sourcing, separate databases, or eventual consistency.
- Project queries directly into caller-specific responses. Do not load aggregates for display.
- Load only the consistency boundary required for commands.
- Share domain behavior among slices in the same lifecycle; vertical slices do not mean share nothing.
- Keep unrelated capabilities from sharing mutable domain models.
- Use controllers, route handlers, and minimal APIs as entry points, not architectural identities.
- Add a request/handler boundary for multiple entry points, a useful pipeline, or framework isolation.
- Use interfaces for meaningful contracts or substitution seams, not by reflex.

## Place Responsibilities

| Concern | Typical location |
| --- | --- |
| Required fields, shape, format, range | Request boundary or validated type |
| Caller identity, claims, role, permission | Application/request layer |
| Business-valid transition and invariant | Domain model when complexity warrants it |
| Loading, saving, integration, sequencing | Handler/application flow |
| Repeated technical mechanics | Focused pipeline component |
| Caller-specific read shape | Direct query/projection |

Start with a transaction script when logic is local and simple. Extract a
domain model when rules repeat across use cases, correctness depends on a
lifecycle, handlers call one another to reuse behavior, or conditionals and
dependencies obscure the use case.

## Typical Structure

```text
src/
  features/
    shipping/
      dispatch/
        Endpoint
        Command
        Handler
      deliver/
        Endpoint
        Command
        Handler
      domain/
        Shipment
      data/
        ShippingStore
```

File extensions, casing, and the name of the persistence type follow whatever
the stack already uses. The shape is what matters, one folder per use case,
with the domain and persistence the slices share sitting next to them.

One file containing a complete small slice is equally valid. Optimize for
locality and discoverability, not file count.

For a modular monolith, hosts compose modules, modules do not reference one
another's implementation, cross-module dependencies use narrow contracts or
messages, and contract modules contain no shared implementation. Modules may
share physical infrastructure while preserving ownership.

## Sharing And Testing

Share code when it has coherent ownership, stable meaning, and demonstrated
reuse. Good candidates include tracing primitives, error contracts, outbox
mechanics, and a domain model shared by related lifecycle operations. Do not
share canonical business entities across unrelated capabilities.

- Assert outcomes, transitions, emitted facts, and public contracts.
- Test invariants directly with given state, action, and expected result.
- Verify ORM and query behavior against realistic persistence where practical.
- Substitute at the real seam: HTTP handler, time provider, delegate, or stable interface.
- Test pipeline components individually and add one integration test for order and short-circuiting.
- Do not write tests whose only value is proving wrappers call wrappers.

## Avoid

- Generic CRUD operations when business intent matters.
- Entity services that gather unrelated behavior around a noun.
- Identical layers, repositories, and handlers in every feature.
- Thin-handler theater that moves one call behind one-use wrappers.
- Generic repositories for use-case-specific queries.
- Pipelines with hidden mutable state or ordering dependencies.
- A global `Shared` module that accumulates unrelated domain meaning.
- Inferring permitted actions in the frontend from raw state when the server owns the rules.

## Review Checklist

- Does the feature name describe an action or capability?
- Is the logical owner clear?
- Does structure reveal what the system does?
- Are request and response contracts use-case-specific?
- Is CQRS being inflated into unrelated infrastructure?
- Does the query fetch only required data?
- Does the command enforce invariants consistently?
- Are validation, authorization, business rules, and orchestration distinguished?
- Is behavior duplicated across handlers?
- Is shared state limited to related features?
- Does another module depend on implementation instead of a contract?
- Is each abstraction justified by reuse, volatility, or a meaningful seam?
- Is asynchronous communication justified by actual workflow needs?
- Can the feature change without navigating unrelated technical layers?
