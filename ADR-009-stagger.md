# ADR-009 — Staggered first runs (+ H1 correction)

Status: accepted (2026-09-26).

## Context

`next_run_at` (migration 005) fixed drift-from-completion, but every monitor
created together still shared `next_run_at = now()` (repo forced it; bench
seed left it NULL = due immediately). Result: permanent synchronized waves,
visible in BENCHMARK wave gaps (60-90s instead of steady 60s).

## Decision

Spread first runs deterministically across `[now, now+interval)`:
- App: `domain.StaggerInitialRunAt` (fnv32a of monitor ID mod interval);
  `Create` stores it, repo INSERT honors the entity value (was hardcoded `now()`).
- Existing rows: migration `006_stagger_backfill.sql` (PG `hashtext`, same spread).
- Bench seed: same rule in SQL so the proof measures realistic phasing.
- Go fnv vs PG hashtext differ per system by design; both spread, both
  reproducible within their system.

## Correction to ADR-007

"H1 shared HTTP client — SKIPPED" is stale: a shared `http.Transport` pool
shipped in the same hardening cycle (Fase 5b). H1 is implemented; this record
stands as the correction (ADRs are not rewritten).

## Consequence

Expected: wave gaps converge to ~60s steady-state; BENCHMARK honest note
updated with measured numbers. No contract change (`next_run_at` stays internal).
