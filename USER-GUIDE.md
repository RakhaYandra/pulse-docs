# Pulse — User Guide

Docs version: v1.0 · Updated: 2026-09-28 · Audience: developers (end user +
local admin). Product version: local compose stack (no release tags on code).

## Product Overview

Pulse watches your APIs: it checks URLs on a schedule, opens incidents when
a monitor fails N times in a row, notifies via Telegram, and shows
everything on a React dashboard. Use it when you want self-hosted uptime
monitoring with honest numbers. Requirements: Docker + Compose, a Telegram
bot token (optional, for notifications).

## Getting Started

Prereqs: Docker, Compose, ports 8080 (api) / 5173 (web) free.
1. `cd pulse-ops && docker compose up` (starts api, worker, scheduler,
   postgres, redis, web).
2. Open the web UI → Register → Login (JWT; sessions via Bearer token).
3. Create your first monitor: name + URL + interval (seconds) + timeout +
   failure threshold. Start unpaused.
4. Watch checks land on the dashboard; pause/resume anytime.
Troubleshooting setup: port clash → change compose mapping; DB errors →
check `DATABASE_URL`; no checks flowing → scheduler logs + Redis (`queue
depth` panel in Grafana).

## User Guide

Dashboard: summary cards (totals, up/down, active, 24h uptime), monitor
table with status + sparkline, click a row for detail. Monitor detail:
recent checks chart, incident list, per-monitor `monitor_id` links.
Incidents: OPEN on threshold breach, RESOLVED on recovery — both pushed to
Telegram if configured. Reports tab: reliability bars over N days
(`?days=` 1–90, default 30). Pause: stops scheduling without deleting
history; resume re-arms `next_run_at`.

## Admin Guide

Config (`pulse-ops/.env`, gitignored): `DATABASE_URL`, `REDIS_ADDR`,
`JWT_SECRET` (required in api mode), `TELEGRAM_BOT_TOKEN` + `TELEGRAM_CHAT_ID`,
`PULSE_ALLOW_HOSTS` (SSRF allowlist — needed if targets resolve to private
IPs, e.g. bench rigs), `TRUST_PROXY` (set when behind a proxy so rate
limits see real IPs). Backup: `ops/BACKUP.md` pg_dump procedure. Upgrade:
pull, rebuild, migrate up (down-stubs exist for bench rollback only).
Logs: `docker compose logs api worker scheduler`.

## API Quick Reference

Base `/api/v1` (Bearer JWT except register/login/health/metrics): POST
/auth/register, POST /auth/login (10/min), GET /auth/me, GET|POST
/monitors, GET|PATCH /monitors/:id, POST /monitors/:id/pause|resume,
GET /monitors/:id/checks, GET /monitors/:id/incidents, GET /incidents,
GET /dashboard/summary, GET /reports/reliability?days=, GET /health,
GET /metrics. Limits: API 100/min/IP (fail-open). Full contract: FSD §G +
Newman collection (executable).

## Troubleshooting

| Symptom | Cause → Fix |
|---|---|
| 429 on login/register | auth limiter 10/min → wait, or `TRUST_PROXY` if behind proxy |
| Monitor never checks | paused, or SSRF block on private IP → `PULSE_ALLOW_HOSTS` |
| 401 everywhere | expired/missing JWT → login again |
| No Telegram message | token/chat-id unset or wrong → verify bot + chat id, check worker logs |
| Queue growing | too few workers vs monitors → raise worker count (bench: 4 handles 1,000) |

## Security Notes

Passwords bcrypt-hashed; JWT required for all data routes; SSRF
deny-by-default; secrets live only in `.env` (never commit). Local-first:
your data stays in your Postgres.

## FAQ

Q: Single node only? A: Yes, by design — no HA claims.
Q: How fresh is the dashboard? A: Scheduler ticks every 10s; per-monitor
pace = its interval + stagger slot.
Q: Data retention? A: Checks pruned per policy; incidents kept.
Q: Can I monitor internal hosts? A: Yes via `PULSE_ALLOW_HOSTS`.

## Changelog

- 2026-09-28: Telegram verified live; WattVision theme; ROADMAP/TSD/
  PROGRESS/USER-GUIDE added.
- 2026-09-26: docs v1.1.0 (FSD/SRS + PDFs); v1.0.0 (BRD/PRD/ARCH + PDFs).

## Glossary

Check (one HTTP probe), incident (OPEN→RESOLVED lifecycle), threshold
(consecutive failures to OPEN), cadence (checks/min), stagger (spread
slots), dedup (one queue entry per monitor).
