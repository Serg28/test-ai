---
status: Accepted
owner: "serg"
reviewers: ["Tech Lead"]
updated_at: "2026-08-08"
feature_size: "M"
ticket: "N/A — course project, no tracker"
---

# 0001 — Use Livewire full-page components for the dashboard

- **Status:** Accepted
- **Date:** 2026-08-08
- **Deciders:** Architect (serg) + Tech Lead, during the `design` Socratic walk

## Context

The dashboard is a single, read-only web-frontend surface (sad.md §4 Target-surface decision): a run-list page and a run-detail page, server-rendered directly against MariaDB via Eloquent, no separate API. Spec §8's carried-forward open question asked design to decide how the pages are built — explicitly inviting Filament's architecture (a Livewire-based admin-panel toolkit) as *inspiration* for the resource/table/action structuring, while keeping the resulting code this project's own rather than an installed copy of that toolkit. Livewire itself is not yet a dependency of this Laravel 13.8 project.

## Decision drivers

- Spec §8 (carried from CONTEXT.md interview): "the design stage should draw on Filament's architecture as inspiration... but the resulting code should read as this project's own implementation."
- Spec §3 non-goal: "No real-time/live-updating view... live updates are a possible later feature, not part of this scope" — the door is explicitly left open, not closed.
- Spec §6 NFR: p95 ≤ 500 ms list/detail load — any rendering approach must stay server-rendered and simple enough to hit that without a caching layer (spec §8 rules out caching for now).
- sad.md §2 Constraints: no existing layering convention to inherit beyond Laravel defaults; Livewire is not yet a composer dependency (a real cost, not free).

## Considered options

1. **Livewire full-page components (chosen)** — two Livewire classes (`RunList`, `RunDetail`) wired directly as route targets, each owning its own query + pagination, structured after Filament's Resource/Table pattern but hand-written (no `filament/filament` dependency).
2. **Plain Blade controllers** — two controller actions (`DashboardController@index`, `@show`) returning Blade views from an Eloquent query; zero new dependencies, the Laravel default.

## Decision outcome

**Chosen:** Option 1, Livewire full-page components. It satisfies the spec's explicit Filament-inspired-but-not-Filament instruction directly, and it sets up cleanly for the live-updating door spec §3/§8 leaves open — extending a Livewire component to poll or push later is additive, where retrofitting Livewire onto plain Blade controllers later would touch every view and route again. The one new dependency (`livewire/livewire`) is judged worth it against that avoided rewrite.

## Consequences

**Positive**
- Matches spec §8's stated design-stage preference directly — no re-litigation needed.
- A future auto-refresh/live-status feature (spec §3's explicitly-open door) extends `RunList`/`RunDetail` in place; no rendering-model rewrite.
- Each component owns one query/pagination concern, giving the "Filament-inspired resource" structure without installing Filament.

**Negative**
- Adds `livewire/livewire` as a new composer dependency, and the accompanying JS asset Livewire injects into the page.
- A Livewire component carries more ceremony (`mount()`/`render()` lifecycle, a paired Blade view under `resources/views/livewire/`) than a bare controller action for what is, today, a fully static read-only page.

**Neutral**
- If the live-updating door in spec §3/§8 is never opened, this is one layer more than a plain-Blade page strictly needed — an accepted bet, not a regret in isolation.

## Links

- Spec: [[../spec.md]]
- SAD: [[../sad.md]] §4
- Related ADR: none yet
