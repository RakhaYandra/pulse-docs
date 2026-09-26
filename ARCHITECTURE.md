# Pulse — Architecture

Full Clean Architecture (ADR-005), one Go binary with 3 run modes.

```
pulse-web (React)
   │  JSON contract (Newman-guarded)
   ▼
delivery/http ──► usecase ◄── delivery/worker, delivery/scheduler
                      │  ▲
                      ▼  │ ports owned by usecase
               infrastructure
               (postgres, security, httpcheck, queue, notify)
                      │
                      ▼
        PostgreSQL · Redis (pulse:jobs) · target APIs · Telegram
```

Dependency rule: delivery → infrastructure → usecase → domain.
Domain: stdlib only. Usecase: domain + stdlib + uuid.
Inner layers never import delivery/infrastructure (grep-verified).

Use-case flow (ProcessCheck): ActiveMonitor → Checker.Check → CheckRepo.Append
→ RecordStatus → RecentStatuses → EvalStreak → Open/Resolve → Transition.
Delivery notifies only on non-nil Transition. One bad monitor can't kill
workers (per-job recover in runner).

Repos: [pulse](https://github.com/RakhaYandra/pulse) (this) ·
[pulse-web](https://github.com/RakhaYandra/pulse-web) (dashboard).

## Scheduling (`next_run_at`)

Due is computed from scheduled time, not completion time: `monitors.next_run_at`
(indexed), advanced by the scheduler on each successful enqueue
(`MarkScheduled(now + interval)`). Create sets `next_run_at = now` (check
ASAP); changing the interval resets it. Result: steady ~60s cadence instead
of drift with drain time (see BENCHMARK.md).
Dedup claims (`pulse:queued:<id>`, TTL 3x interval, released on completion)
bound the queue even when workers lag.
