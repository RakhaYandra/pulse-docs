# Pulse — SRS (Software Requirements Specification)

Abridged IEEE 830. Functional IDs (`FR-xx`) defined in PRD; each maps to
FSD functions (`FS-xx`) and a Newman verification where applicable.

## 1. Introduction

Pulse is a self-hosted API monitoring and incident platform for developers
and small teams. Scope: availability + response performance + incident
detection + history. Non-goals: HA, multi-region, teams/billing.

## 2. Overall description

Single Go binary (api/worker/scheduler modes), PostgreSQL (state),
Redis (job queue + rate budgets), React dashboard. Users own isolated
monitors; background workers execute checks outside the request lifecycle.

## 3. Functional requirements

| ID | Requirement | FSD | Verified by |
|---|---|---|---|
| FR-01 | Auth (register/login/me, bcrypt, JWT 24h, per-user isolation) | FS-01…03 | Newman auth (5) |
| FR-02 | Monitor CRUD + pause/resume, URL/threshold validation, true PATCH | FS-04…08 | Newman monitors (10) |
| FR-03 | Scheduled checks, retry-collapse, result store | FS-09 | engine E2E live |
| FR-04 | Threshold incident open/resolve + Telegram on transitions | FS-10 | engine E2E live |
| FR-05 | Dashboard, history, reliability report | FS-11 | Newman reads (5) + e2e |
| FR-06 | Health, metrics, Prometheus/Grafana/alerts | FS-12 | targets up + Grafana shot |

## 4. Non-functional requirements (measurable)

- NFR-01 Throughput: sustain N/60 checks/s to N=1000 (measured 9.28/s avg,
  28/s peak, queue 0 — see BENCHMARK.md).
- NFR-02 Correctness: 0 false incidents on healthy targets; 100% timeout
  detection (measured 50/50).
- NFR-03 API latency: p95 local < 100ms (Newman avg ~10ms).
- NFR-04 Security: SSRF deny-by-default + allowlist; auth 10/min/IP, API
  100/min/IP, fail-open on Redis outage; JWT required, no defaults.
- NFR-05 Reliability: per-job isolation (`recover`), dedup-bounded queue,
  crash-safe claims (TTL re-queue).

## 5. Traceability

PRD `FR-xx` → FSD `FS-xx` (§3 table) → `qa/collection.json` assertions /
engine E2E runs / benchmark tables. Gaps: none open (see GitHub issues for
accepted residual risks: cadence stretch, per-replica limits, single-node).
