---
date: 2026-05-19
time: 11:45
tool: code
project: FuturesAnalytics
duration_minutes: ~240
---

## Summary

Closed the bot-observability gaps. Shipped stdout/stderr file logging via a tee class (every launch writes `logs/live_<TIMESTAMP>.log`), then added four crash-resilient data captures to dashboard.db: broker fill detail (`filled_price`, `filled_ts_utc`, `slippage_ticks`, `broker_fill_payload_json`), MFE/MAE per trade in R-multiples, per-gate decision detail (new `signal_gates` table, top-3 rejecting gates instrumented with margin + regex fallback for the rest), and run-level config + git snapshot (new `runs` table joins every signal/order to the exact code+config that produced it). Ran a Tier 3 parallel audit (three subagents) after the four-capture phase signaled done — caught 1 BLOCKER + 7 IMPORTANT + 20 nits; applied 4 cheap fixes and committed them; the rest is queued in HANDOFF.md backlog. Bot is now running on the audit-fixed code (`a999b523`, run_id=3) capturing the full pipeline.

## Files touched

### Created
- `C:\Users\Greg\Documents\Claude\Projects\Futures Analytics\logs\README.md` (created) — convention + PowerShell grep recipes for the new per-session log files
- `C:\Users\Greg\Documents\Claude\Projects\Futures Analytics\logs\live_20260519_100747.log` (created by bot, run_id=2 session, 45 KB)
- `C:\Users\Greg\Documents\Claude\Projects\Futures Analytics\logs\live_20260519_113202.log` (created by bot, run_id=3 current, growing)
- `C:\Users\Greg\Documents\Claude\Projects\Futures Analytics\logs\live_20260519_160217.log` (created by first bot today before run-table support, 35 KB)
- `C:\Users\Greg\.claude\projects\C--Users-Greg-Documents-Claude-Projects-Futures-Analytics\memory\reference_projectx_stale_backfill.md` (created) — ProjectX REST history can return delayed-mode backfill while SignalR is live; restart cures it

### Modified
- `C:\Users\Greg\Documents\Claude\Projects\Futures Analytics\system\broker\live_runner.py` (modified) — `_StreamTee` class + `_setup_stdout_logging()` helper; `_capture_run_snapshot()` helper (git SHA + 22-key matcher-constants snapshot); new banner lines (Log file path + Run id); `_check_positions` gained fill-detail block (paper-mode `filled_price=entry, slippage=0` with JSON source flag) + MFE/MAE per-bar update block (R-multiples, clamped at 0, uses `o.get("entry_price")` directly post-audit); `start_run` call at main() startup, `end_run` on clean shutdown AND on kill-switch path (per audit fix); 3 write_signal/write_manual_order call sites thread `run_id=self.run_id`; signal-gate verdicts persisted via `write_signal_gates` with regex fallback `r'(UR\d+)'` for non-instrumented gates
- `C:\Users\Greg\Documents\Claude\Projects\Futures Analytics\system\dashboard_writer.py` (modified) — SCHEMA_SQL adds `runs` table + `signal_gates` table + indexes; `_init_db` idempotent ALTER block extended with 7 columns on manual_orders (`filled_price`, `filled_ts_utc`, `slippage_ticks`, `broker_fill_payload_json`, `mfe_R`, `mae_R`, `run_id`) + `run_id` on signals; new methods `update_order_fill`, `update_order_mfe_mae` (SQL CASE + scalar `MAX(a,b)`), `write_signal_gates` (bulk via executemany), `start_run`, `end_run`; `write_manual_order` and `write_signal` accept `run_id`; `update_order_mfe_mae` docstring states explicit non-negative-magnitude CONTRACT
- `C:\Users\Greg\Documents\Claude\Projects\Futures Analytics\system\matcher.py` (modified) — `dataclass, field` import; `SetupSignal.gate_verdicts: list = field(default_factory=list)`; `_log_gate` helper; UR132 / UR194 / UR212 reject sites instrumented (UR194 margin = ATR-distance from nearest Fab 4 edge, UR212 margin = swing magnitude in ATR, UR132 binary with categorical detail)
- `C:\Users\Greg\Documents\Claude\Projects\Futures Analytics\HANDOFF.md` (modified) — added "## 2026-05-19 Capture additions" section (durable layer summary) and "## 2026-05-19 Audit backlog" section (Tier 3 findings + deferred items)
- `C:\Users\Greg\Documents\Claude\Projects\Futures Analytics\.gitignore` (modified) — `logs/*.log` added
- `C:\Users\Greg\.claude\projects\C--Users-Greg-Documents-Claude-Projects-Futures-Analytics\memory\MEMORY.md` (modified) — added `reference_projectx_stale_backfill` index line (25 entries total)

### DB schema migration applied to live `dashboard/dashboard.db`
- Tables added: `runs`, `signal_gates`
- Columns added: 7 on `manual_orders`, 1 on `signals`
- Migration ran cleanly against the running bot via `DashboardWriter._init_db()` invocation (PRAGMA-gated ALTER block is idempotent)

## Decisions made

- **Tier 3 parallel audit triggered when the 4-capture phase signaled done.** Three parallel general-purpose subagents (persistence / hot path / matcher) returned in ~6min wall clock. Confirmed Tier 3 ROI for 3-file blast radius + hot path + schema (already captured in `feedback_audit_cycle_tiers`).
- **Scope per-gate detail to 3 gates with margin + regex fallback for the rest.** Pass-side logging deferred — only reject-side instrumented. Accepted-signal margin invisibility is documented as a known limitation in the audit backlog.
- **Paper-mode broker fill capture only.** Live-mode broker-side fill query (would require `/api/Position/searchOpen` per fill) deferred with a `TODO` comment in `_check_positions`. Paper mode records `filled_price=entry, slippage=0` honestly + a JSON source flag (`"source":"local_sim_from_bar_OHLC"`) so the provenance is queryable.
- **Defer crash-resilience work to next session.** Bot has been stable across 3 launches today, all clean exits; paper mode means no money at risk. The known leak (DETECTOR_HITS lost on unclean exit; `runs.ended_utc=NULL` on unhandled exception) is annoying but not data-corruption. Three changes queued: try/finally wrap of `main()` shutdown, incremental DETECTOR_HITS persistence, `bot_state.is_running` convention shift.
- **The 7-hour stale-backfill anomaly from this morning** resolved on restart, root cause uninvestigated. Documented as a memory + HANDOFF backlog note (recognition pattern in the banner line) rather than blocking session on a deep debug.

## Open questions

- Root cause of the 7-hour stale REST backfill — ProjectX delay-mode flap vs contract-entitlement quirk vs gateway issue. Memory captures the recognition pattern; investigate if it recurs.
- The same-bar fill-and-stop BLOCKER affects ~1-5% of trades. Frequency under PRAC tape is empirically unknown — would need a join of `manual_orders` exits vs entry bar's high/low to count actual occurrences before / after a fix.
- Pre-existing 151 signals with `run_id=NULL` — synthesize retroactive runs rows via gap detection vs. leave as-is and start fresh? Backfill scripts queued either way.

## Next steps

- **Crash resilience** (try/finally shutdown wrap; DETECTOR_HITS persistence to dashboard.db; `bot_state.is_running` convention shift to "freshness from last_update_utc")
- **Same-bar fill-and-stop BLOCKER fix** — restructure pending→open path to fall through to open branch on the same bar
- **Tag 5 rejection strings with synthesized codes** so they stop bucketing to `OTHER` in `signal_gates`
- **Pure-DB backfills** — historical MFE/MAE on 2026-05-18's 13 closed trades; `signal_gates` for 151 NULL-run_id signals; synthesize `runs` rows for pre-new-code sessions
- **Pass-side gate instrumentation** for UR194/UR212/UR132 (accept-path margin logging)
- **Optional Tier 2 SHOULD-HAVEs**: heartbeats table; system metrics at heartbeat cadence; top-of-book bid/ask snapshot at signal/fill/exit

## Memory promotion candidates

- `reference_projectx_stale_backfill` — already saved this session. No other promotion-worthy lessons surfaced today beyond what's already in memory (audit ROI is captured in `feedback_audit_cycle_tiers`; persistence-layer patterns are project-specific not general).
