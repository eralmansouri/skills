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

## Installation

### Skills CLI

Install with the [Skills CLI](https://github.com/vercel-labs/skills):

```bash
npx skills add eralmansouri/skills
```

### Claude Code

```text
/plugin marketplace add eralmansouri/skills
/plugin install architecture-skills@skills
```

### Other

The `skills/<name>/SKILL.md` layout follows the common Agent Skills format and
can be registered directly with tools that scan custom skill paths.

## Credit

The architecture knowledge in these skills was distilled from the excellent
videos published by [CodeOpinion on YouTube](https://www.youtube.com/@CodeOpinion).
Many thanks to Derek Comartin for sharing years of practical architecture and
software design guidance.
