# be-semantic

A methodology for designing applications from **meaning**, not from technical constraints.

**be-semantic** is a domain-first design workflow that bridges user intent and implementation through semantics. It combines the [Be Framework](https://github.com/be-framework/be), [ALPS](https://alps.io/), and data-driven schema design into a coherent process.

## The Problem

Most API design starts with technology:
> "We need a REST API. Let's design endpoints and schemas."

This leads to schemas that reflect database tables or framework conventions — not the actual domain.

**be-semantic starts differently:**
> "What does the user want to do? What does that *mean*?"

## The Workflow

```
Story → ALPS → Fake → Agreement → Schema → Be Implementation
```

| Step | What | Why |
|------|------|-----|
| **Story** | User stories in natural language | Ground the design in real user intent |
| **ALPS** | Application vocabulary and state transitions | Define what the app *can do*, not how |
| **Fake** | Generate 50+ realistic fake data entries | Observe reality before prescribing constraints |
| **Agreement** | Review fake data with stakeholders | Align on shared understanding |
| **Schema** | JSON Schema derived from observations | Record agreed reality as constraints |
| **Be** | Implement as Being/Final in Be Framework | Code that mirrors the domain, not the technology |

## Key Insight

> Constraints should be derived from observed reality, not prescribed from abstract rules.

When you generate 50 fake entries and look at them, you discover things:
- The longest title anyone would actually write is 41 characters (not "255 because VARCHAR")
- Some fields are always present, others rarely used
- Edge cases you never thought of appear naturally

The schema becomes a *record of observation*, not a *guess*.

## Quick Start

```bash
git clone https://github.com/koriym/be-semantic.git
cd be-semantic
```

Explore the `example/` directory — a complete Todo app built with this workflow:

- [`example/story/`](example/story/) — User stories
- [`example/alps/`](example/alps/) — ALPS profile
- [`example/fake/`](example/fake/) — 50 fake data entries
- [`example/schema/`](example/schema/) — Derived JSON schemas

## Documentation

- [**Tutorial**](docs/en/tutorial.md) — Build a Be app from scratch (start here)
- [Concept](docs/en/concept.md) — Philosophy and motivation
- [Workflow](docs/en/workflow.md) — Step-by-step guide
- [Insights](docs/en/insights.md) — Observations and reflections

日本語ドキュメント:
- [コンセプト](docs/ja/concept.md)
- [ワークフロー](docs/ja/workflow.md)
- [考察](docs/ja/insights.md)

## Related

- [Be Framework](https://github.com/be-framework/be)
- [ALPS Specification](https://alps.io/)
- [Semantic-Ex Method](https://koriym.github.io/blog/2025/08/10/semantic-method-en)
- [app-state-diagram](https://github.com/koriym/app-state-diagram)
