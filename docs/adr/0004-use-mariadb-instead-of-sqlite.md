---
status: Accepted
owner: "Architect"
reviewers: []
updated_at: "2026-08-08"
feature_size: "N/A (foundation)"
ticket: "N/A"
---

# 0004 — Use MariaDB (via the existing Laradock stack) instead of SQLite

- **Status:** Accepted
- **Date:** 2026-08-08
- **Deciders:** user (correction immediately after [[0003-use-sqlite-and-eloquent-default-ids]] was recorded)

## Context

[[0003-use-sqlite-and-eloquent-default-ids]] picked SQLite for zero-setup local persistence. The user
already runs a Laradock Docker Compose stack for this project (`laradock-php-fpm-1` container, project
mounted at `/var/www/AI tests/course/test-ai`), which provisions MariaDB. SQLite would add a second,
redundant datastore instead of using what's already running.

## Decision drivers

- The user's existing dev environment (Laradock) already runs MariaDB — using it avoids standing up
  a second, unused datastore.
- All Artisan/Composer commands already run inside `laradock-php-fpm-1` via `docker exec`, so there's
  no added operational step to reach MariaDB from there.
- Auto-increment ID reasoning from [[0003-use-sqlite-and-eloquent-default-ids]] is unaffected — still a
  single local writer, no concurrency/merge scenario.

## Considered options

1. **MariaDB via the existing Laradock stack** — reuses infrastructure the user already has running.
2. **SQLite** (the original 0003 choice) — zero-setup, but redundant given Laradock's MariaDB is
   already available.
3. **A separate standalone MariaDB/MySQL container outside Laradock** — rejected: duplicates what
   Laradock already provides.

## Decision outcome

**Chosen:** Option 1 (MariaDB via Laradock). Reuses the user's existing running infrastructure instead
of introducing a second datastore; keeps the "commands run inside `laradock-php-fpm-1`" convention
consistent for DB access too.

## Consequences

**Positive**
- One datastore, matching what the user already runs — no SQLite file to keep in sync or forget to migrate away from.
- Local dev and any future non-CLI environment share the same engine (MariaDB), reducing dialect drift.

**Negative**
- Tests and the CLI now depend on the Laradock stack being up; the tool cannot simply run bare-PHP
  from the host without it (this constraint is already recorded in the architecture map).
- CI needs its own MariaDB service container (Laradock isn't available there) — flagged in the
  architecture map's Constraints and in scaffold task S4.

**Neutral**
- The ID strategy (Eloquent auto-increment) carries over unchanged from 0003.

## Links

- Spec: N/A (foundation decision, precedes any feature spec)
- SAD: N/A
- Related ADR: [[0003-use-sqlite-and-eloquent-default-ids]] (superseded), [[0001-use-php-laravel-for-the-cli-tool]]
