---
name: codebase-design
description: Use for code-level module design, deep modules, interfaces, seams, adapters, testability, refactoring shallow modules, and finding architecture improvements in an existing codebase.
---

# Codebase Design

Design deep modules: a lot of behavior behind a small interface, placed at a
seam where something actually varies, and tested through that interface. Depth
gives callers leverage and maintainers locality.

## Vocabulary

Use these terms consistently. Reserve *boundary* for bounded contexts and
service ownership; at code level, say *seam*.

- Module: anything with an interface and an implementation, at any scale: function, class, package, or vertical slice.
- Interface: everything a caller must know to use the module correctly: signature, invariants, ordering, error modes, configuration, and performance characteristics.
- Depth: behavior a caller or test can exercise per unit of interface learned. A shallow module's interface is nearly as complex as its implementation.
- Seam: a place where behavior can change without editing that place; where an interface lives.
- Adapter: a concrete implementation filling a seam, such as a Postgres store or an in-memory fake.
- Leverage: what callers gain from depth. One implementation pays back across every call site and test.
- Locality: what maintainers gain from depth. Change, bugs, and verification concentrate in one place.

## Principles

- Deletion test: imagine deleting the module. If complexity vanishes, it was a pass-through. If complexity reappears across callers, it earns its keep.
- The interface is the test surface. Callers and tests cross the same seam; needing to test past it signals the wrong shape.
- One adapter is a hypothetical seam; two adapters make a real one. Production plus a test fake counts as two.
- Depth belongs to the interface. A deep module may contain small internal parts and internal seams; keep them out of its interface.
- Ask of every interface: can it have fewer entry points, simpler parameters, or hide more?
- At external seams, expose one operation per remote capability rather than a generic `send(request)`, so fakes stay simple and usage stays visible.

## Dependencies And Tests

| Dependency | Example | Seam and test approach |
| --- | --- | --- |
| In-process | Pure logic, in-memory state | Merge and test through the interface; no adapter |
| Local-substitutable | Database, filesystem | Run a local stand-in such as a container or embedded engine; the seam stays internal |
| Remote but owned | Your other services | Port at the seam; transport adapter in production, in-memory adapter in tests |
| True external | Payment, email, SMS providers | Injected port; fake adapter in tests |

- Assert observable outcomes through the interface. A test that changes when only the implementation changes is testing past it.
- Replace, don't layer: once tests exist at a deepened interface, delete the old tests of its shallow parts.

## Improve An Existing Codebase

1. Take the user's named area; otherwise find hotspots in recent git history. Deepening pays off where change keeps happening.
2. Read the glossary and ADRs for that area. Reopen a recorded decision only when the friction is real, and say so.
3. Explore for the friction signals below, then apply the deletion test to each suspect.
4. Present each candidate in domain language with files, problem, proposed change, leverage and locality gains, test changes, and strength: Strong, Worth exploring, or Speculative. Close with the candidate to tackle first.
5. Wait for the user to choose a candidate before designing its interface.
6. Design it twice: draft two or three radically different interfaces, such as minimal entry points, trivial common case, or ports and adapters, in parallel subagents when available. Compare depth, locality, and seam placement, then recommend one or a hybrid.

Friction signals:

- Understanding one concept requires bouncing between many small modules.
- An interface is nearly as complex as its implementation.
- Pure functions were extracted for testability while bugs hide in how they are called.
- Modules leak across their seams.
- Code is untested or hard to test through its current interface.
- A bug cannot be locked down by a regression test at a seam that reproduces it.

## Avoid

- Pass-through wrappers, managers, and one-call handlers that fail the deletion test.
- Ports and interfaces with a single adapter.
- Widening an interface to expose internals for tests.
- Keeping shallow-module tests alongside tests at the deepened interface.
- Refactoring stable code that nobody needs to change.
