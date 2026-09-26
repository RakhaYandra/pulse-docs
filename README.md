# Pulse Docs

[![docs](https://github.com/RakhaYandra/pulse-docs/actions/workflows/docs-to-pdf.yml/badge.svg)](https://github.com/RakhaYandra/pulse-docs/releases)

> Ecosystem: [api](https://github.com/RakhaYandra/pulse) · [web](https://github.com/RakhaYandra/pulse-web) · [docs](https://github.com/RakhaYandra/pulse-docs/releases)

Official documentation for **Pulse**, an API monitoring & incident platform
(English). This repo is the source of requirements and design; source code
lives in the application repos (see Fact sources).

| Document | Content |
|---|---|
| [BRD.md](BRD.md) | Business Requirements Document — problem, objective, scope, success criteria ([PDF](https://github.com/RakhaYandra/pulse-docs/releases/latest)) |
| [PRD.md](PRD.md) | Product Requirements Document — functional + non-functional requirements ([PDF](https://github.com/RakhaYandra/pulse-docs/releases/latest)) |
| [ARCHITECTURE.md](ARCHITECTURE.md) | System architecture, layers, data flow ([PDF](https://github.com/RakhaYandra/pulse-docs/releases/latest)) |
| [BENCHMARK.md](BENCHMARK.md) | Measured load results, method, honest reads (markdown only) |
| ADR-001…008 | Architecture Decision Records (markdown only, living docs) |
| POSTMORTEM-001/002 | Incident postmortems (markdown only) |

Fact sources (truth per artifact):

| Repo | Referenced artifacts |
|---|---|
| `pulse` | `qa/collection.json` (22 requests), migrations, `bench/` rig, `/metrics` series |
| `pulse-web` | dashboard, detail + sparkline, Reports tab, e2e suite |

## PDF

Each `v*` tag triggers the `docs-to-pdf` workflow → BRD, PRD, ARCHITECTURE
PDFs upload as **release assets**. Download the formal versions on the
[Releases](../../releases) page. BENCHMARK, ADRs, and postmortems stay
markdown-only (living docs that change faster than releases).

## Versioning

| Version | Date | Content |
|---|---|---|
| v1.0.0 | 2026-09-26 | Initial release: BRD, PRD, ARCHITECTURE + PDFs |

## Roadmap (docs)

Done: BRD, PRD, ARCHITECTURE, BENCHMARK, ADR-001…008, postmortems, PDF pipeline.
In progress: FSD, SRS (this release cycle).
Frozen: translations (Indonesian sibling-style docs — English is canonical here).
