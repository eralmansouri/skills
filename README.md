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

### OpenCode

Copy the skills you want into `.opencode/skills/` in your project (or
`~/.opencode/skills/` for global access).

### Claude Code

```text
/plugin marketplace add eralmansouri/skills
/plugin install architecture-skills@skills
```

Or copy the skills you want into `.claude/skills/` in your project (or
`~/.claude/skills/` for global access).

## Credit

The architecture knowledge in these skills was distilled from the excellent
videos published by [CodeOpinion on YouTube](https://www.youtube.com/@CodeOpinion).
Many thanks to Derek Comartin for sharing years of practical architecture and
software design guidance.
