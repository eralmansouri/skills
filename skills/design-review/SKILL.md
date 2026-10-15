---
name: design-review
description: Design review of a codebase area, branch, or PR. Maps its bounded contexts, vertical slices, and modules, then judges where rules live and whether flexibility sits where change actually happens. Use when the user asks for a design or architecture review, or when a review should judge structure rather than bugs or style.
---

# Design Review

Review the **shape** of code: which context owns each rule, how use cases are cut into slices, which modules are deep or shallow, and whether the design bends where the domain actually changes. Bugs and style belong to a code review. This one judges design, and changes no code.

Two rules carry the review:

- **No map, no findings.** Map the design as it is, traced through real use cases, before judging any of it. A review without a map is a review of file names.
- **Evidence over taste.** Every finding cites code you opened, and every move it proposes passes the change test or the deletion test (see **Fit**). "This should use CQRS" is taste; "the refund window is checked in three places, with two different cut-offs" is evidence.

## Vocabulary

Domain nouns come from the project's `GLOSSARY.md` (one per context, or one at the root): "the Refund rules", not "RefundServiceImpl". Architecture nouns come from this table, one word per scale. Call the Skill tool with "codebase-design" for the rest of the module vocabulary (interface, depth, adapter, leverage, locality) and the deletion test.

| Say | Meaning | Instead of |
| --- | --- | --- |
| **Context** | A bounded context: a capability that owns its language, rules, and data | domain, service |
| **Slice** | One use case end to end, from entry point to persistence and side effects | endpoint, handler |
| **Module** | Anything with an interface and an implementation, at any scale | component, layer, service |
| **Seam** | Where a module's interface lives, and where behaviour can vary | boundary (that word belongs to contexts) |
| **Invariant** | A rule that must hold after every command | validation, check |

Say **service** only for an independently deployed unit. A class called `OrderService` is a module, and often a suspect one.

## Fit

The judgement this review exists for: is each part of the design as heavy as its problem, and flexible exactly where it changes?

**Proportion.** Design weight tracks domain complexity. A core area, where rules are many, shared across use cases, and costly to get wrong, earns a rich model: aggregates, value objects, policies. A supporting area earns transaction scripts and plain CRUD. Both mismatches are findings:

- **Under-built**: a core rule scattered across handlers, duplicated with drift, or enforced where its data isn't.
- **Over-built**: aggregates, factories, repositories, and mappers wrapped around a form that saves a row.

**Axes of change.** An axis of change is a direction the domain actually varies along: a new payment provider, a new tenant tier, a new order status. Find axes in history (the same kind of change, commit after commit), in code (a `switch` on the same type repeated across slices), and in the glossary. Then match seams to axes:

| | Seam | No seam |
| --- | --- | --- |
| **On an axis** | Flexible: keep it | Rigid: each new case edits many places |
| **Off every axis** | Speculative: indirection nothing varies through | Simple: keep it |

**The change test.** Take a likely change along an axis and walk it through the code, listing every file, module, and context it touches. If it lands in one slice or one module, the design holds. If it spreads across contexts, runs through every horizontal layer, or repeats one edit in several places, name the missing seam or the misplaced boundary. The change test asks whether the design absorbs change; the deletion test asks whether a module earns its keep. Every proposed move passes whichever one applies.

## Process

### 1. Scope

Take the target the user named: a path, a context, a branch, or a PR. For a branch or PR, the scope is the diff **plus every slice and context it touches**, since a design problem rarely sits inside the changed lines. With no target, start from the hot spots, because design that keeps changing is where a review pays back:

```bash
git log --since="6 months ago" --name-only --format= | sort | uniq -c | sort -rn | head -30
```

Read `GLOSSARY.md` (or `GLOSSARY-MAP.md`) and the ADRs in `docs/adr/` that cover the scope. ADRs record decisions this review doesn't re-litigate.

Done when the scope is a list of paths, and you've read the glossary and ADRs that cover it or confirmed there are none.

### 2. Map

Describe the design as it is; judge nothing yet. Write the map down: every later step, and every sub-agent, works from it.

- **Contexts**: each one, its folders, what it owns, and how it reaches the others (import, shared table, HTTP, event).
- **Slices**: trace at least three use cases end to end, picked from the hot spots; for a diff, every use case it touches. Each hop gets a `file:line`: entry point, validation, rule, persistence, side effects.
- **Invariants**: each one you meet, where it's enforced, and every other place it's checked.
- **Axes of change**: each one with its evidence.
- **Co-change**: for the two or three hottest files, what changes with them:

  ```bash
  git log --format=%H -n 50 -- <file> | xargs -n1 git show --name-only --format= | sort | uniq -c | sort -rn | head -15
  ```

  The file tops its own list with its commit count. A file from another context near the top is coupled to it, whatever the imports say.

Done when every slice in the map is traced hop by hop, and every context, invariant, and axis rests on code you opened. A map built from file and folder names is a guess.

### 3. Lenses

A lens is one design discipline, backed by its own skill. Run the three core lenses on every review, and a conditional lens only when the map shows the code has what it covers.

| Lens | Skill | Runs when | Hunts for |
| --- | --- | --- | --- |
| Domain | `domain-driven-design` | Always | Names drifting from the glossary; one model shared across contexts; invariants scattered or enforced away from their data; aggregates sized by screens instead of invariants |
| Slices | `vertical-slice-architecture` | Always | Use cases smeared across horizontal layers; entity services; generic repositories serving queries; contracts shared between use cases; handlers calling handlers |
| Modules | `codebase-design` | Always | Shallow modules and pass-throughs; seams with one adapter; leaks across seams; tests reaching past the interface |
| Boundaries | `service-boundaries` | Several deployables, or a modular monolith | Reads and writes into another context's tables; synchronous call chains; changes that need coordinated releases |
| Messaging | `event-driven-systems` | Queues, pub/sub, integration events | Commands dressed as events; CRUD events; state and messages that can disagree |
| Persistence | `persistence-design` | Read models, caches, hand-written data access | Decision reads outside the write transaction; unbounded queries; caches with no invalidation path |
| Event sourcing | `event-sourcing` | An event store | Property-change events; streams with no lifecycle; projections treated as truth |

Give each lens its own sub-agent, all in parallel, so one lens's findings don't steer another's. Where sub-agents aren't available, run the lenses yourself, one after another. Each brief carries:

- The scope (with the diff command, for a diff), the map, and the **Fit** and **Finding format** sections of this skill, pasted in full: the sub-agent has no other access to them.
- The lens's row from the table.
- The brief:

  > Call the Skill tool with "<skill>". Hunt the scope for what your lens covers, opening every file you cite. Answer from the code, and skip any question the code can't answer. Return findings in the finding format, strongest first, under 500 words. No findings is a valid answer.

Done when every lens that applies has reported, and every lens that doesn't has a one-line reason in the map ("no messaging: no queue or bus in scope").

### 4. Judge

Lens reports are raw material. Before anything reaches the reader:

- **Merge** findings that are one problem seen through two lenses. Agreement across lenses makes a finding stronger.
- **Test** every move. A removal or merge passes the deletion test; a new seam or boundary sits on an axis of change and passes the change test.
- **Weigh proportion.** Richer modeling for a supporting area needs evidence of real rule complexity there; simplifying a core area names the invariants it keeps.
- **Check ADRs.** A finding that contradicts one survives only when the friction is real, and says so: _"contradicts ADR-0007, worth reopening because…"_
- **Trim to the change.** For a diff, keep what the change introduces or entrenches, and drop what merely sits nearby.
- **Grade** each survivor:
  - **Strong**: evidence in the code, the test agrees, and it sits on a hot spot or an axis of change.
  - **Worth exploring**: the evidence is real, but the payoff depends on where the code goes next.
  - **Speculative**: plausible, on thin evidence.

Collect **Holds up** as you go: designs a reviewer would be tempted to change but shouldn't, each with its reason. _"Invoicing is a transaction script and should stay one: supporting area, no shared invariants."_ No findings and a solid Holds up list is a valid review.

Done when every surviving finding carries evidence, a passed test, and a grade.

### 5. Report

Findings first, strongest first: full entries for Strong and Worth exploring, one line each for Speculative. Then Holds up. Close with the **top recommendation**: the one move to make first, and why.

Return Markdown when you're a sub-agent or the review feeds a PR. When a human asked for the review directly, also render it as an HTML report: see [REPORT.md](REPORT.md).

Then stop. A sub-agent returns its report; with a human, ask which finding to dig into. When the user picks one, call the Skill tool with "codebase-design" and design it twice. When they reject one for a reason a future review would need, offer to record it as an ADR.

## Finding format

```md
### Give the refund window one home in Order · Strong

- **Lens**: Domain
- **Where**: `src/ordering/request-refund/handler.ts:41`, `src/ordering/cancel-order/handler.ts:88`, `web/orders/RefundButton.tsx:12`
- **Evidence**: all three check the refund window, two as 30 days and one as 31; the change test for "window varies by customer tier" edits all three.
- **Problem**: the refund window is an Order invariant with no single owner, and its copies already disagree.
- **Move**: enforce the window in `Order.requestRefund()`; both handlers call it, and the UI reads `canRequestRefund` from the order query.
- **Wins**: one owner for the invariant · locality: one place to change the window · one test covers it
- **Cost**: `cancel-order` loads the Order aggregate instead of a row.
```

- The **title** names the move in domain language.
- **Problem** and **Move** are one sentence each. A move is a concrete change, not a pattern name: "enforce the window in `Order.requestRefund()`", never "apply DDD".
- **Wins** use the vocabulary: locality, leverage, depth, one owner per invariant. "Cleaner" and "easier to maintain" name no gain.
- **Cost** names what the move adds or closes off.
