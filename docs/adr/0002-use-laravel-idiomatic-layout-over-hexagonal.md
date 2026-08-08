---
status: Accepted
owner: "Architect"
reviewers: []
updated_at: "2026-08-08"
feature_size: "N/A (foundation)"
ticket: "N/A"
---

# 0002 — Use Laravel's idiomatic console-command layout instead of hexagonal layering

- **Status:** Accepted
- **Date:** 2026-08-08
- **Deciders:** user (during `survey` greenfield foundation session)

## Context

With the stack fixed to Laravel ([[0001-use-php-laravel-for-the-cli-tool]]), the module/folder
structure needs to be picked. SDD's default heuristic favors hexagonal layering for modular
services, but this is a single CLI tool, not a service with multiple inbound adapters.

## Decision drivers

- Small, single-actor surface (one CLI, one datastore) — no adapter multiplicity to justify ports/adapters.
- Laravel's own conventions (`app/Console/Commands`, `app/Models`) are well documented and immediately
  legible to anyone who knows the framework.
- Course/test-scale project — minimizing structural ceremony keeps it easy to extend per feature.

## Considered options

1. **Laravel idiomatic layout** — commands in `app/Console/Commands/`, models in `app/Models/`, no extra layering.
2. **Hexagonal (`domain → app → infra → ports`)** — stronger isolation, but adds ports/adapters with only one real adapter (the console) and one real infra target (SQLite).
3. **Flat single-file scripts (no framework structure)** — rejected: throws away Laravel's autodiscovery and testing conventions for no benefit.

## Decision outcome

**Chosen:** Option 1. The project's actual complexity (one CLI surface, one datastore) doesn't warrant
hexagonal's extra indirection; Laravel's own layout already separates entry points (commands) from
data (models) cleanly enough for this scale.

## Consequences

**Positive**
- Zero unused abstraction — every folder maps to something the project actually has.
- Anyone familiar with Laravel can navigate the repo without reading a bespoke layering doc.

**Negative**
- If the tool later grows a second inbound surface (e.g. an HTTP API), some re-layering may be needed
  to isolate domain logic from the console layer.

**Neutral**
- Revisiting this if/when a second adapter appears is a normal, bounded refactor — not a rewrite.

## Links

- Spec: N/A (foundation decision, precedes any feature spec)
- SAD: N/A
- Related ADR: [[0001-use-php-laravel-for-the-cli-tool]]
