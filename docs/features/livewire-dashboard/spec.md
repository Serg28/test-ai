---
status: Draft
owner: "serg"
reviewers: ["Tech Lead"]
updated_at: "2026-08-08"
feature_size: "M"
---

# Spec — livewire-dashboard

> **Glossary:** [CONTEXT](./CONTEXT.md)
> **Reference module / docs / channels used:** None — only the interview + CONTEXT + `docs/architecture-map.md`.

## 1. Context

Developers who run test-ai CLI commands (AI test executions) currently have no way to check what happened during a run — or whether one is still in progress — without querying the database directly or re-reading raw CLI output. As the number of runs grows this becomes increasingly inconvenient, and it means the tool has no shareable, at-a-glance view of its own activity.

The repository has already moved past a pure command-line footprint: a web-serving surface (routes, views, front-end build tooling) is already present on disk, even though it wasn't part of the original CLI-only foundation. A simple, read-only dashboard page is the natural first use of that surface, giving the CLI tool's own developer immediate value without committing to a larger web application.

The committed approach: a single web page that lists the most recent recorded CLI runs with their status, plus a detail view for one run, open to any developer who can reach the running application — no login, no ability to trigger or manage runs from the page.

Traceability: `docs/architecture-map.md` (status: current, mode: greenfield-bootstrap) still fixes the project foundation as CLI-only with "no HTTP layer" / "no frontend" — that statement is now stale relative to the checked-out code and should be reconciled by re-running `survey` alongside or after `design`. Separately, no "CLI run" data entity exists in the codebase yet (only the default `User` model/migrations are present); this feature's `data-model` stage is where that entity gets its schema — this spec only fixes what a CLI run means at the business level (see `CONTEXT.md`).

## 2. Goals

- A developer can check the outcome of any recorded CLI run without querying the database or re-reading raw CLI output.
- The current/latest status of a CLI run is visible within seconds of opening the dashboard.
- The dashboard becomes the default place a developer looks first to check run status, instead of ad-hoc DB access.

## 3. Non-goals

- No authentication, authorization, or per-user permissions in this feature — the tool is course/demo-scale and any developer with network access is treated as trusted (revisited in §8 if that changes).
- No ability to trigger, retry, or cancel CLI runs from the dashboard — the feature is view-only by explicit scope decision.
- No filtering, search, sorting, or pagination controls — the list shows a fixed, bounded window of the most recent runs (see AC-01), keeping the MVP to the smallest useful list view.
- No real-time/live-updating view (auto-refresh or push updates) — the dashboard reflects state as of page load; live updates are a possible later feature, not part of this scope.

## 4. User stories

### US-01: View run history

**As a** developer
**I want** to see a list of past CLI runs
**So that** I can check what happened without querying the database

### US-02: See current run status

**As a** developer
**I want** to see the status of a CLI run at a glance
**So that** I know whether it's still running, succeeded, or failed

### US-03: See run result / failure details

**As a** developer
**I want** the dashboard to reflect the actual result recorded by the CLI execution
**So that** I can trust what I see without cross-checking the database myself

### US-04: See an empty dashboard state

**As a** developer
**I want** a clear message when no runs have been recorded yet
**So that** I know the dashboard is working correctly and not broken

### US-05: Access the dashboard without logging in

**As a** developer
**I want** to open the dashboard directly without authenticating
**So that** I can quickly check status in a course/demo setting

### US-06: View a single run's detail

**As a** developer
**I want** to open one run and see its full detail
**So that** I can diagnose a failure or confirm a success in depth

## 5. Acceptance criteria

### AC-01 (US-01) — happy path

**Given** at least one CLI run has been recorded
**When** a developer opens the dashboard
**Then** the system displays the 50 most recent runs newest-first, each showing its command, status, and start/finish time

### AC-02 (US-06) — error

**Given** a developer opens the detail view for a run identifier that does not exist
**When** the page loads
**Then** the system tells the developer plainly that no such run exists, instead of showing a blank or broken page

### AC-03 (US-05) — authorization

**Given** the dashboard has no login requirement
**When** a developer who can reach the running application opens the dashboard
**Then** the system displays it without asking for credentials

### AC-04 (US-02) — domain invariant

**Given** a CLI run is shown on the dashboard
**When** the developer looks at its status
**Then** the system always shows exactly one status from the fixed set (running / succeeded / failed) — never a blank or ambiguous status

### AC-05 (US-03) — cross-context

**Given** a CLI run was recorded by the console-command execution (a different part of the system than the dashboard's web module)
**When** the developer opens the dashboard or a run's detail
**Then** the system reflects that run's latest recorded status and result exactly, without the developer needing to inspect the database directly

### AC-06 (US-04) — empty state

**Given** no CLI runs have been recorded yet
**When** a developer opens the dashboard
**Then** the system shows a clear empty-state message instead of an empty or broken table

## 6. Non-functional requirements

| Aspect | Target | Measurement |
|---|---|---|
| Latency p95 dashboard list load | ≤ 500 ms | manual timing / CI smoke test |
| Latency p95 run detail load | ≤ 500 ms | manual timing / CI smoke test |
| Throughput | ≥ 5 req/s per instance | smoke test in CI |
| Availability | 99% successful page loads | % of successful dashboard/detail loads over total loads, tracked in application logs |
| Status accuracy | 0% stale-status reads | displayed status compared against the record's current value at the moment of that same load |

## 6.1 Security / privacy

- **Data classification:** internal — course/demo data, no customer data.
- **Personal data touched:** none — a CLI run is expected to carry only command name, status, timestamps, and a result/error summary.
- **AuthZ/AuthN impact:** none — no permission checks are added; the dashboard is open by explicit decision (see §3, §8).
- **Abuse cases:**
  - unrestricted access to run history/results: accepted for this course/demo scope — rationale: no PII, no production traffic (§8 revisits this before any production deployment).
  - direct/guessed run identifiers in the detail view: the system validates the run exists before rendering and reports "not found" otherwise (AC-02) rather than leaking a stack trace or raw error.
  - data leak through displayed fields: N/A — no sensitive fields are displayed.
  - spam / write abuse: N/A — the page is read-only, no create/update actions to rate-limit.
- **Security review:** N/A — no PII, no new authorization boundary, internal course tool.

## 7. Metrics / KPIs

- **Manual DB queries to check run status** — baseline: 100% of status checks require direct DB/CLI-output access today (no dashboard exists), target: 0% within one sprint of release.
- **Dashboard load success rate** — baseline: N/A (feature doesn't exist yet), target: ≥ 99% successful loads (no unhandled errors) over the first month of use.
- **Time to see latest run status** — baseline: N/A, target: ≤ 5 seconds from opening the dashboard URL.

## 8. Open questions

- [ ] Should the dashboard eventually support triggering new CLI runs directly from the UI (not just viewing)? Default now: no — view-only for this feature. — owner: PM, due: before a follow-up feature is proposed
- [ ] Should login/access control be added once this leaves the course/demo context (e.g. deployed somewhere reachable by non-trusted users)? Default now: no auth, open access. — owner: Tech Lead, due: before any production-facing deployment
- [ ] Where does this dashboard actually run and who can reach it? `docs/architecture-map.md` currently fixes the whole project inside a single local Docker dev container with no network exposure, which §3/AC-03's "any developer who can reach the app" outgrows. Default now: local dev/course environment only (e.g. `php artisan serve` on the developer's machine or inside the dev container) — no shared/networked deployment assumed. — owner: Tech Lead, due: at `sdd:design livewire-dashboard`
- [ ] Confirmed with the user mid-interview: the design stage should draw on Filament's architecture (an admin-panel toolkit built on Livewire — its resource/table/action structuring patterns) as inspiration, but the resulting code should read as this project's own implementation, not an installed copy/reskin of that toolkit for a third-party developer reading the code. Default now: carried forward as a design-stage preference, not a spec-level decision. — owner: Tech Lead, due: at `sdd:design livewire-dashboard`
