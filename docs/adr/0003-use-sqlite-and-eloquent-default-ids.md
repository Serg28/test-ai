---
status: Accepted
owner: "Architect"
reviewers: []
updated_at: "2026-08-08"
feature_size: "N/A (foundation)"
ticket: "N/A"
---

# 0003 — Use SQLite + Eloquent with Laravel's default auto-increment IDs

- **Status:** Accepted
- **Date:** 2026-08-08
- **Deciders:** user (during `survey` greenfield foundation session)

## Context

The tool needs a persistence choice fixed before any feature can add data. It's a locally-run CLI
tool for a course/test context, not a deployed multi-instance service.

## Decision drivers

- Single local process, single writer — no concurrent-write or multi-node ID-collision concern that
  would justify time-sortable IDs (ULID/UUID).
- Zero external setup: SQLite needs no server, fits a course/test-scale tool.
- Laravel's migration tooling (`php artisan migrate`) and Eloquent both default to auto-increment IDs.

## Considered options

1. **SQLite + Eloquent auto-increment integer IDs** — Laravel's out-of-the-box default; zero setup.
2. **SQLite + app-generated ULIDs** — SDD's generic default heuristic (time-sortable IDs), but adds
   complexity (ULID generation, larger keys) with no concurrency/merge scenario to justify it here.
3. **A server-based RDBMS (MySQL/Postgres)** — rejected: unnecessary operational overhead for a local
   single-user CLI tool.

## Decision outcome

**Chosen:** Option 1. No concurrent-write or distributed-ID scenario exists for a local CLI tool, so
the simplest, framework-default choice wins over the generic time-sortable-ID heuristic.

## Consequences

**Positive**
- No extra dependency or ID-generation code; matches every Laravel tutorial and default migration stub.
- SQLite requires no server process — the tool runs anywhere PHP runs.

**Negative**
- Auto-increment IDs are not globally unique or externally mergeable; if the tool ever needs to sync
  records across machines, IDs would need to be revisited.

**Neutral**
- Switching to ULIDs later is a contained migration (one column type change + a backfill), not a rewrite.

## Links

- Spec: N/A (foundation decision, precedes any feature spec)
- SAD: N/A
- Related ADR: [[0001-use-php-laravel-for-the-cli-tool]]
