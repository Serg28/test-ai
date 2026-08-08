---
status: Accepted
owner: "Architect"
reviewers: []
updated_at: "2026-08-08"
feature_size: "N/A (foundation)"
ticket: "N/A"
---

# 0001 — Use PHP 8.3 + Laravel for the CLI tool

- **Status:** Accepted
- **Date:** 2026-08-08
- **Deciders:** user (during `survey` greenfield foundation session)

## Context

The repo is empty and needs a stack fixed before any feature work can start. The project is a CLI
tool for course/AI-testing work; the user explicitly asked for PHP with Laravel.

## Decision drivers

- Explicit user constraint: PHP + Laravel.
- Laravel ships Artisan, a mature console-command framework, so no extra CLI library is needed.
- Laravel's ecosystem (Eloquent, migrations, Pest, Pint) covers persistence, tests, and lint out of the box.

## Considered options

1. **PHP + Laravel (Artisan console commands)** — the user's stated stack; full framework, only the console kernel is used.
2. **Plain PHP + a lightweight CLI library (e.g. Symfony Console standalone)** — smaller footprint, no framework overhead.
3. **A different language entirely (Go/Rust/Node)** — rejected: contradicts the explicit PHP/Laravel constraint.

## Decision outcome

**Chosen:** Option 1 (PHP + Laravel). Matches the explicit user constraint and gets Artisan, Eloquent,
migrations, and testing tooling for free instead of assembling them piecemeal.

## Consequences

**Positive**
- One coherent, well-documented ecosystem for CLI, ORM, migrations, and tests.
- No framework-selection ambiguity for future features.

**Negative**
- Laravel is heavier than a bare CLI library for a tool that never uses HTTP routing — some framework
  surface (routes/web.php, service container bootstrapping) goes unused.

**Neutral**
- Switching to a lighter CLI library later is possible but would mean re-porting commands/models.

## Links

- Spec: N/A (foundation decision, precedes any feature spec)
- SAD: N/A
- Related ADR: [[0002-use-laravel-idiomatic-layout-over-hexagonal]], [[0003-use-sqlite-and-eloquent-default-ids]]
