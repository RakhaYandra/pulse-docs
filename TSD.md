# Pulse — Technical Specification

Version: v1.0 · Date: 2026-09-28 · Status: Approved (as-built)
Author: Rakha Putra Pebri Yandra
Revision history: v1.0 initial (as-built from shipped code + measured runs).
Related: BRD, PRD, FSD, SRS, ARCHITECTURE, BENCHMARK, ADR-001…009 (+FE-CA).

## C. Introduction & Context

Problem: developers running APIs lack a self-hosted, honest availability
monitor — SaaS costs, vendor lock-in, and dashboards that hide how numbers
are produced. Business context: Pulse is a portfolio-grade product proving
backend craft (scheduling, queueing, incident lifecycle, observability).
Scope: local-first monolith (API + worker + scheduler), React SPA,
Telegram notifications, Prometheus/Grafana. Out of scope: HA/multi-region,
mobile apps, billing/teams, SMS/pager channels.

## D. Requirements

Functional (15 API routes + engine): register/login/me; monitor CRUD (true
PATCH semantics), pause/resume, per-monitor checks + incidents; global
incidents; dashboard summary; reliability report (days 1–90, default 30,
with per-monitor `monitor_id` click-through); health; Prometheus metrics.
Flows: register → create monitor → scheduler enqueues → worker checks →
threshold breach OPENS incident → Telegram → recovery RESOLVEs → Telegram.
Non-functional: check latency p95 within 30s timeout envelope; scheduler
tick 10s; throughput budget 100 monitors/2 workers with queue 0; availability
best-effort single node (no SLA beyond honest reporting); OWASP-aware input
handling (SSRF deny-by-default, JWT auth, rate limits).

## E. System Architecture

```
browser ──REST──▶ gin API ──enqueue──▶ redis ──dequeue──▶ 4 workers ──check──▶ target URLs
     │                 │                    │                     └─results──▶ postgres
     │                 └─dashboard/reports──┴──incidents──▶ telegram
     └─prometheus scrapes /metrics; grafana dashboards (5 panels, 2 alerts)
```

Components: scheduler (10s tick, claims due `next_run_at`); Redis queue
(dedup-bounded); workers (NumCPU default, pool); API (auth, CRUD, reports);
Telegram notifier (retry 3×). Data flow per check: claim → enqueue →
HTTP GET (≤30s) → persist check → evaluate consecutive-failure threshold →
incident OPEN/RESOLVED → notify. Integration points: Postgres 16, Redis 7,
Telegram Bot API, Prometheus/Grafana.

## F. Database Design

Tables (`migrations/001–006`): users (id, email unique, password_hash);
monitors (id, user_id FK, name, url, interval_seconds, timeout, threshold,
paused, `next_run_at`, stagger slot); checks (id, monitor_id FK, ts,
status_code, latency_ms, ok, error); incidents (id, monitor_id FK, opened_at,
resolved_at, status OPEN/RESOLVED). Retention: checks pruned by policy
(configurable; bench artifacts cleaned via `clean.sql` regexes).
Backup/recovery: `ops/BACKUP.md` (pg_dump procedure, restore tested path).

## G. API Specifications

Base `/api/v1`, JSON, Bearer JWT (except register/login/health/metrics).
Endpoints (from `router.go`): POST /auth/register, POST /auth/login
(auth limiter 10/min), GET /auth/me, GET|POST /monitors, GET|PATCH
/monitors/:id, POST /monitors/:id/pause|resume, GET /monitors/:id/checks,
GET /monitors/:id/incidents, GET /incidents, GET /dashboard/summary,
GET /reports/reliability?days=, GET /health, GET /metrics.
Envelope: data + meta; errors: mapped status codes (4xx validation/auth,
5xx with safe messages). Rate limits: auth 10/min/IP, API 100/min/IP
(Redis token bucket, fail-open, `TRUST_PROXY` honored). Contract pinned by
Newman collection (22 assertions, green).

## H. UI/UX Specifications

React 19 + Vite 6 SPA, WattVision clinical theme, CSS tokens, dark/light
toggle. Routes: login/register, dashboard (summary cards + monitor table +
sparkline bars), monitor detail (checks chart, incidents, click-through
`monitor_id`), Reports (reliability bars), incidents. Responsive: table →
cards on mobile; a11y: semantic landmarks, focus-visible, keyboard-operable
tabs. Screenshots: `dashboard.png`, `detail.png` (fresh per UI change).

## I. Integration & Dependencies

External: Telegram Bot API (notify, verified live 2026-09-28);
Docker Hub pinned images; Go module proxy. Third-party runtime:
PostgreSQL 16, Redis 7, Prometheus v3, Grafana 12. Internal: 6 repos share
the Newman contract + compose files in `pulse-ops`; analytics reads
Postgres via `pulse-data` ETL into DuckDB. Sync: scheduler↔workers via
Redis; API↔DB direct; no cross-service eventual consistency to manage.

## J. Security Specifications

Auth: bcrypt passwords, JWT (fail-fast in api mode; POSTMORTEM-001).
SSRF: deny-by-default egress (private/link-local/metadata blocked) +
`PULSE_ALLOW_HOSTS` (BENCHMARK + POSTMORTEM-002). Rate limiting per §G.
Secrets: `.env` gitignored, never logged, compose-only. Audit: incident
lifecycle timestamps + check history immutable. Testing: SSRF guard cases
in `test-plan.md`, rate-limit unit tests (`ratelimit_test.go`).

## K. Performance Specifications

Cache: none server-side by design (freshness > speed for monitoring);
browser caches static SPA. DB: indexes on (monitor_id, ts); prune policy.
Load: single node, worker pool scales with CPU; measured: 1,000 monitors /
4 workers → peak 28/s, queue max 0, latencies valid (50/50 timeout split).
Monitoring: RED-style Grafana (5 panels) + 2 alerts + self-monitoring
procedure (Pulse watches itself). Alerting: Telegram on incidents.

## L. Testing Strategy

Unit: Go (`go test`, incl. ratelimit) + vitest 24 (FE utils/hooks).
Contract: Newman 22/22 (seed → CRUD → checks → incidents → reports →
error paths). E2E: Playwright 7/7 (auth, CRUD, pause/resume, charts,
Reports click-through). Bench: `bench/run.sh` N=50/200/1000 + validity
guards (`run_id` tagged, 50/50 split, duration ≥90s, no qa/e2e rows,
updated_at=frozen). UAT: manual flows against local compose before merge.

## M. Deployment & DevOps

Local: `docker compose up` (api, worker, scheduler, postgres, redis, web).
Environments: dev/staging/prod collapsed to one compose (documented
simplification). CI: docs-PDF + DuckDB cross-check + markdown contract.
Rollback: image tag pin + `migrate down` stubs (`clean.sql` for bench).
Logging: structured API logs + compose streams; metrics via Prometheus.
DR: `ops/BACKUP.md` + single-VPS deploy issue (frozen, owner decision).

## N. Documentation Requirements

Code: exported Go symbols + key flows commented; FE feature READMEs where
non-obvious. API: FSD §G + Newman collection as executable contract.
User: `USER-GUIDE.md`. Ops: RUNBOOK/SLA/TROUBLESHOOTING/SELFMONITORING.

## O. Implementation Timeline

Historical (factual): MVP → hardening (dedup/pool/tick) → security
(SSRF/rate-limit/CORS) → observability → analytics → full CA refactors →
WattVision → Telegram live 2026-09-28. Milestones: v1.0.0 + v1.1.0 docs
(2026-09-26). Forward: per ROADMAP.md (deploy decision is the gate).

## P. Acceptance Criteria

Done = Newman 22/22 + e2e 7/7 + bench validity guards pass + zero
`bench_%`/`qa_%`/`e2e_%` rows in prod data + docs↔code grep audit clean.
Quality bar: every README number traceable to a run; every incident
postmortemed (001, 002 closed).

## Q. Appendices

Glossary: cadence (checks/min delivered), wave (one scheduler pass),
dedup (one queue entry per monitor), stagger (deterministic slot spread),
OPEN/RESOLVED (incident states). Config reference: `PULSE_ALLOW_HOSTS`,
`TRUST_PROXY`, `DATABASE_URL`, `REDIS_ADDR`, `TELEGRAM_*`, `JWT_SECRET`.
Key commands: `make test`, `make bench N=1000`, `docker compose up`,
`./run.sh` (qa e2e).
