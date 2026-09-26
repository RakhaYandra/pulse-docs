# Pulse — FSD (Functional Specification Document)

Functional spec per endpoint and background job. References PRD `FR-xx`;
verified by `qa/collection.json` (Newman) in the `pulse` repo.

Conventions: base `/api/v1`, auth `Authorization: Bearer <JWT>`,
success `{"data": ...}`, errors `{"error": msg}` with 400/401/404/409/429/500.

## FS-01 Register — `POST /auth/register` (FR-01)

Input: email, password (min 8), name. Behavior: bcrypt hash, UUID id,
409 `email already registered` on duplicate. Output: `{token, user}`.
Rate limit: auth class (10/min/IP).

## FS-02 Login — `POST /auth/login` (FR-01)

Input: email, password. Wrong credentials → 401 `email/password salah`
(identical message both cases, no user enumeration). Output: `{token, user}`.

## FS-03 Me — `GET /auth/me` (FR-01)

Bearer required (401 `missing bearer token` / `invalid token`). Output: user.

## FS-04 Create monitor — `POST /monitors` (FR-02)

Input: name (required), url (http/https; literal private/loopback/link-local
rejected; DNS re-checked at dial time, redirects never followed),
interval 60-3600s (default 300), timeout 1-60s and < interval (default 5),
failure/recovery thresholds ≥1 (defaults 3/2). Sets `next_run_at = now`.
Output: full monitor, status UNKNOWN.

## FS-05 List/Get monitors (FR-02)

`GET /monitors` (own, newest first), `GET /monitors/:id` (404 if not owned).

## FS-06 Update monitor — `PATCH /monitors/:id` (FR-02)

True PATCH: only provided keys applied (pointer fields), omitted kept,
re-validated. Changing interval resets `next_run_at = now + interval`;
other edits never shift the schedule. 404 if not owned.

## FS-07 Delete monitor (FR-02)

Cascades checks + incidents (FK). Output: `{deleted: id}`.

## FS-08 Pause/Resume — `POST /monitors/:id/pause|resume` (FR-02)

Toggles `is_active`. Scheduler skips paused; in-flight jobs finish normally.

## FS-09 Check execution (FR-03)

Scheduler tick (default 10s, `SCHED_TICK_SECONDS`): due scan
(`next_run_at elapsed`) → Redis enqueue with Lua dedup claim
(`pulse:queued:<id>`, TTL 3x interval) → `MarkScheduled(now + interval)`.
Worker pool (`WORKER_CONCURRENCY`, default NumCPU): pop → HTTP GET with
per-attempt timeout → ≤3 attempts (backoff 1s, 2s) collapse into ONE result:
UP (2xx) | DOWN | TIMEOUT | ERROR + code + wall-clock ms.

## FS-10 Incident lifecycle (FR-04)

Trailing streak evaluated per check (`EvalStreak`): fail streak ≥
failure_threshold → OPEN (reason, failure_count); while OPEN, success streak
≥ recovery_threshold → RESOLVED (recovery_count, resolved_at). Transitions
returned to delivery, which notifies (Telegram on OPEN/RESOLVED only).

## FS-11 Reads (FR-05)

`GET /monitors/:id/checks?limit=` (default 20, max 200, newest first);
`GET /monitors/:id/incidents`; `GET /incidents` (all own, newest first, with
`monitor_id` for click-through); `GET /dashboard/summary` (totals, up/down,
active, 24h uptime); `GET /reports/reliability?days=` (1-90, default 30:
per-monitor incidents, MTTR over resolved, uptime %, checks total).

## FS-12 Observability endpoints

`GET /health` (liveness), `GET /api/v1/health` (db + redis),
`GET /metrics` per process (api :9101, workers :9102-04/06, scheduler :9105).
Prometheus + Grafana + 2 alerts via `docker-compose.observability.yml`.
