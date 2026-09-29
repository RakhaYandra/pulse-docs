# Pulse — Development Roadmap

Version: v1.1 · Published: 2026-09-28, updated 2026-09-29 · Period: Q4 2026 – Q1 2027
Status: living document (updated per release, not per sprint — solo dev)

## B. Executive Summary

Vision: the most honest self-hosted API monitor a solo developer can run —
every number measured, every failure documented, no fabricated claims.
Goals this period: (1) close all verification gaps (done: Telegram live
2026-09-28), (2) keep the 6-repo portfolio coherent, (3) ship only what is
measured. Expected outcomes: production-ready local stack, docs that match
code, and a deploy decision unblocked or explicitly kept frozen.

## C. Product Overview

Pulse monitors API availability and response performance: scheduled checks,
threshold incident lifecycle (OPEN/RESOLVED), Telegram notifications,
React dashboard, Prometheus/Grafana observability. Existing: Go/Gin API,
React/Vite UI, Postgres + Redis, DuckDB analytics (4 marts, API cross-check
MATCH), Newman 22/22, Playwright e2e 9/9, benchmarked 1,000 monitors / 4
workers (peak 28/s, queue 0, 50/50 timeouts). Target: developers/small teams
self-hosting. Position: clinical ops console, not SaaS.

## D. Goals & Objectives (SMART)

1. Zero unverified chapters by end of period (measurable: every README claim
   has a run behind it; Telegram was the last, closed 2026-09-28).
2. Docs↔code zero drift (measurable: per-release grep audit; current: clean).
3. Deploy decision resolved one way or the other (frozen since start; a
   documented "no" beats silent limbo).
4. `pulse-data` pipeline kept green (CI + MATCH on every change).

## E. Feature Roadmap (phase-based, source: GitHub issues)

| Item | Priority | Status | Impact |
|---|---|---|---|
| Telegram live verification | High | DONE 2026-09-28 | closes last unverified chapter |
| Stagger deterministik (ADR-009) | High | DONE | wave spacing ~60s steady |
| Incident click-through + Reports | High | DONE (e2e 2.5m proof) | UX core loop closed |
| Single-VPS deploy | High | FROZEN (owner) | unlocks all public value |
| MTTR/SLA reports | Medium | DONE | dashboard Reports tab |
| Self-monitoring | Medium | DONE (procedure verified) | dogfooding proof |
| Per-replica rate limit | Low | OPEN issue #2, by design | matters only at scale-out |
| FE overhaul (audit→Sesi A/B→visual→light→routing→pagination) | High | DONE 2026-09-29 | 12 findings closed, DESIGN.md, e2e 9/9 |
| oxlint+oxfmt side-by-side tooling | Medium | DONE 2026-09-29 | 17x/175x measured, zero diff, 1 finding fixed |
| `pulse-data` upkeep | Ongoing | CI green | analytics stays truthful |

Dependencies: deploy unblocks nothing technical (stack is prod-shaped);
Telegram needed nothing but a token (done). No cross-feature blockers remain.

## F. Technology & Infrastructure

Stack frozen: Go 1.27/Gin, React 19/Vite 6 + react-router, PostgreSQL 16,
Redis 7, Prometheus v3/Grafana 12, Docker Compose. No new tech planned
(react-router added 2026-09-29 for deep links; oxlint+oxfmt for gates).
Infra: single-node by design (no HA/multi-region — explicit non-goal).
Security posture: SSRF deny-by-default + allowlist, Redis rate limits
(auth 10/min, API 100/min, fail-open), JWT fail-fast, CORS env-driven.
Perf: worker pool (NumCPU default), dedup-bounded queue, 10s scheduler tick.

## G. Resource & Budget

Solo developer, AI-assisted. No team, no hires, no vendors, no budget —
stated explicitly instead of a fictional org chart. Training needs: none
open (observability + benchmark tooling already operated live).

## H. Risk Assessment

| Risk | Likelihood / Impact | Mitigation |
|---|---|---|
| Maintainer bandwidth (solo) | Med / Med | Small phases, gates per phase, docs follow code |
| Scope creep (docs!) | High / Low | md-only rule for living docs; tag only formal changes |
| Env drift across 6 repos | Med / Med | Ecosystem bars, cross-repo grep audits |
| Secret leak (Telegram token) | Low / High | gitignored `.env` only, secret-scan before push, revoke procedure |

## I. Dependencies & Constraints

External: Docker Hub images (pinned), Go modules proxy, Telegram Bot API
(notify path), GitHub (releases/CI). Internal: 6 repos share contract
(Newman JSON) + compose in ops. Constraints: single node, local-first data,
English docs canonical.

## J. Timeline & Milestones

- 2026-09-26: v1.0.0 (BRD/PRD/ARCH + PDFs), v1.1.0 (FSD/SRS + PDFs).
- 2026-09-27/28: WattVision theme, header/layout pass, Telegram live proof.
- 2026-09-29: FE overhaul + tooling (design audit closed, light mode,
  routing, pagination, oxlint/oxfmt, e2e 9/9, unit 27).
- Next: deploy decision (frozen) → would trigger VPS runbook + DNS + TLS work.
- No fake Gantt: solo cadence is phase-gated, dates are release tags.

## K. Communication & Approval

Owner/approver: Rakha Putra Pebri Yandra (all areas). No committee, no
signatures to forge — recorded honestly. Channel: this repo + issues.
