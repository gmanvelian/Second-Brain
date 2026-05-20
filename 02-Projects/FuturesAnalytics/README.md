# Futures Analytics — Cash Command Center

## Status
**Active live trading** — autonomous bot goes live on Topstep $50K Combine ($MES primary) starting 2026-05-18 09:30 ET.

## Overview
Velez-methodology-based futures trading system. Three layers:
1. **Methodology database** — Velez setups/rules extracted from YouTube content (v1.29 as of 2026-05-19: 59 setups, 403+ universal rules, 82 sources, 74 sheets; v1.29 adds 6 new sheets capturing Greg's video-review findings — three_types_surges, three_types_pauses, elephant_control_regime, bull_move_halves, walk_down_mechanics, no_ft_major_bottom)
2. **Live trading bot** — Python 3.14, ProjectX/TopstepX API, real-time signal generation via matcher
3. **Custom dashboard** — localhost web UI ("Cash Command Center") because CME's single-session rule forces one consumer per data feed

## Cross-platform sync (Windows ↔ Linux dual-boot)
- **Canonical remote:** https://github.com/gmanvelian/Futures-Analytics (private)
- **Working clones:** Windows at `C:\Users\Greg\Documents\Claude\Projects\Futures Analytics` (canonical for live trading), Linux at `~/path/to/Futures-Analytics` (dev/lower-memory sessions)
- **Pickup-on-Linux:** `cd ~/path/to/Futures-Analytics && git fetch origin && git status && git pull origin master` — then read `HANDOFF.md` for current state
- **Pickup-on-Windows:** `cd "C:\Users\Greg\Documents\Claude\Projects\Futures Analytics" && git fetch origin && git pull origin master`
- **Full conventions, what's gitignored, how to recreate `.env` / `dashboard.db` / `bars_archive`:** see [`PLATFORM_NOTES.md`](../../../Claude/Projects/Futures%20Analytics/PLATFORM_NOTES.md) at the repo root
- **Cardinal rules:** commit before switching sides; only one side runs the live bot at a time (CME single-session rule); never commit `.env`, `dashboard/dashboard.db*`, `logs/`, `reviews/`, or `.claude/`

## Stack
- Language: Python 3.14 (`C:\Users\Greg\AppData\Local\Python\bin\python.exe`)
- Broker: ProjectX (TopstepX gateway), JWT auth, SignalR live feed
- Storage: SQLite (`dashboard/dashboard.db`) as the system of record
- UI: FastAPI server, TradingView Lightweight Charts, inline CSS/JS
- Risk limits: $500/trade, $2,000/day (Topstep $50K Combine guidelines)

## Graphify
- **Planned** — scoped to methodology side: `database/` + `sources/` + matcher.py rule references
- Will surface setup ↔ universal-rule ↔ source linkages and v1.11 methodology gaps
- Not yet generated — queued for post-Monday-open once live data starts informing iteration

## Project Files
- Working dir (operational source of truth): `C:\Users\Greg\Documents\Claude\Projects\Futures Analytics\`
- Live handoff (always-current): `…\Futures Analytics\HANDOFF.md`
- Methodology DB: `…\Futures Analytics\database\` (xlsx versions v0.1 → v1.14)
- Velez sources: `…\Futures Analytics\sources\`
- Bot: `system/broker/live_runner.py`
- Dashboard: `dashboard/app.py` + `dashboard/index.html`

## v1.29 Design (next architectural cycle)
Two architecturally significant items captured 2026-05-19 — full design in [`docs/v1.29_design.md`](../../../Claude/Projects/Futures%20Analytics/docs/v1.29_design.md) inside the repo.
1. **Semi-auto execution path** for Topstep Live Funded readiness (Live Funded prohibits API order placement; need human-click confirmation between signal and order). ~6–8h.
2. **Intra-bar / event-driven matcher cadence** — currently matcher fires only at 2m bar close, leaving anticipation entries (taught in SRC-026 / SRC-030) and intra-bar gate re-evaluation (UR194 trap-zone exit) unimplemented. Suspected cause of 13/13 S008 rejections in paper mode. ~8–12h.
**Sequencing:** tighten existing gates (HANDOFF backlog #1, #2, #8) FIRST → then Item 1 (semi-auto) → then Item 2 (anticipation). Anticipation on mis-tuned gates just lets bad signals fire faster.

## Notes
- HANDOFF.md in the project directory is the live source of truth — this README is intentionally thin to avoid stale duplication.
- Methodology iteration is active (Cowork FTC cycles in flight). Don't mirror the database here; refer to working dir.
- Account: Topstep Combine ID 22804300 (confirmed 2026-05-17).
