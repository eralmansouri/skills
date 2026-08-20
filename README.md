# Software Architecture & Design Skills

A collection of agent skills for practical software architecture and design.
The repository currently covers domain-driven design, vertical slice
architecture, service boundaries, event-driven systems, persistence design,
and event sourcing.

Each skill is self-contained under `skills/`:

| Skill | Focus |
| --- | --- |
| `domain-driven-design` | Domain discovery, language, bounded contexts, aggregates, and invariants |
| `vertical-slice-architecture` | Feature-oriented code, CQRS, handlers, pipelines, and testing |
| `service-boundaries` | Modular monoliths, microservices, ownership, coupling, and decomposition |
| `event-driven-systems` | Commands, events, messaging workflows, reliability, and operations |
| `persistence-design` | Transactions, queries, concurrency, caching, migrations, and multi-tenancy |
| `event-sourcing` | Event streams, aggregates, projections, replay, and event evolution |

## Claude Code

This repository is both a Claude Code plugin and a plugin marketplace. After
publishing it to GitHub, install it with:

```text
/plugin marketplace add <github-owner>/my-skills
/plugin install architecture-skills@my-skills
```

For local development or testing:

```bash
claude --plugin-dir ~/Repositories/my-skills
```

Claude exposes these under the `architecture-skills` namespace and can invoke
them automatically when their descriptions match a task.

## Other Agents

The `skills/<name>/SKILL.md` layout follows the common Agent Skills format and
can be registered directly with tools that scan custom skill paths.

## Credit

The architecture knowledge in these skills was distilled from the excellent
videos published by [CodeOpinion on YouTube](https://www.youtube.com/@CodeOpinion).
Many thanks to Derek Comartin for sharing years of practical architecture and
software design guidance.
