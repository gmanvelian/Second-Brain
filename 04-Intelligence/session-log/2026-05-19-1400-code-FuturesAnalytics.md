---
date: 2026-05-19
time: 14:00
tool: code
project: FuturesAnalytics
duration_minutes: 240
---

## Summary

Mid-day session of the first multi-TF PRAC paper run. Shipped two flagged dashboard bugs (ACCOUNT card mode-routing + bar markers across all TFs), ran a Tier 3 three-agent audit of the bar classifier, and shipped Option B (the dashboard-decoupled subset) of the audit's methodology fixes. Option A (matcher-coupled threshold tightening + backtest re-lock) queued for after market close. Bot was kept on the locked-baseline classifier in-memory the entire session; tripwire documented in HANDOFF.md.

## Files touched

### Modified
- `C:\Users\Greg\Documents\Claude\Projects\Futures Analytics\dashboard\app.py` (modified) — three iterations on the bar-marker fix: (1) LEFT JOIN inside `_archive_payload` [later reverted because archive is stale]; (2) `_enrich_labels_inplace` minute-prefix post-process + `_bars_table_payload` for live 1m [superseded]; (3) final pass — `_classify_bars_inplace()` runs the bot's `classify_bar` server-side on every `/api/chart` and `/api/history` response so all TFs (ticks / sec / 1m / 5m / 1h / 1d / 1W / 1M / 1Y) get markers regardless of source (CONT / archive / REST). Also: ACCOUNT card mode-routing — new `_pick_account_by_mode()` + per-mode cache in `_account_info_cache`; `_fetch_account_info(mode)` accepts mode; `_account_payload()` reads mode from control_state. Response now includes a `mode` field.
- `C:\Users\Greg\Documents\Claude\Projects\Futures Analytics\system\detectors\bars.py` (modified) — Option B classifier fixes: `is_bull_180`/`is_bear_180` now require range engulfment (`curr.high > prior.high` / `curr.low < prior.low` per SRC-006 #43/#50) + prior-bar elephant (per SRC-043 #85-91 "two elephants"). `is_color_change_bull`/`is_color_change_bear` now require takeout (per SRC-005 #122 + SRC-076 #34-38). `classify_bar` suppresses BULL_ELEPHANT/BEAR_ELEPHANT when BULL_180/BEAR_180 already fired (mirrors existing COLOR_CHANGE suppression).
- `C:\Users\Greg\Documents\Claude\Projects\Futures Analytics\system\models.py` (modified) — `Bar.is_doji` switched from literal `abs(close-open) < 1e-9` (essentially never fired) to range-relative `body / range < 0.10` (ten_bar_alphabet B11 + SRC-020 #113 "indecision" intent). Affects classify_bar DOJI fallback + UR131 leg counting via `context.py:89`.
- `C:\Users\Greg\Documents\Claude\Projects\Futures Analytics\HANDOFF.md` (modified) — "Known issues" section flipped to "fixed 2026-05-19 morning" with three-pass narrative; new "2026-05-19 Classifier methodology audit — Option B shipped, Option A queued" section with full per-label distribution shift table, tripwire ("do NOT restart live_runner.py"), and Option A backlog with file:line targets.

### Created
- `C:\Users\Greg\.claude\projects\C--Users-Greg-Documents-Claude-Projects-Futures-Analytics\memory\reference_bars_ts_utc_dual_format.md` (created) — `bars.ts_utc` carries two formats (3-digit ms + 6-digit µs) with duplicate rows per minute; match by `SUBSTR(ts_utc, 1, 16)` not byte-exact. Surfaced during the bar-markers diagnosis (the byte-exact JOIN under-matched).
- `C:\Users\Greg\.claude\projects\C--Users-Greg-Documents-Claude-Projects-Futures-Analytics\memory\MEMORY.md` (modified) — added 1 line for the dual-format reference (24th entry).

## Decisions made

- **Shipped Option B (dashboard-decoupled subset) mid-session instead of Option A (full audit fixes + backtest re-lock).** Reason: Greg's paper-mode tape is live today; A would require a bot restart and a 2-month backtest. B exploits Python's no-hot-reload — code edits to `bars.py`/`models.py` don't reach the running bot's matcher until restart. Dashboard restart picks them up immediately so chart markers reflect methodology-clean labels today, while matcher continues on the locked baseline.
- **Server-side classifier `_classify_bars_inplace()` over a hybrid label-enrich approach.** Reason: first two passes (JOIN + enrich-from-bars-table) only delivered labels on bars present in the live `bars` table — anything else (CONT splice, higher TFs, REST fallback, sub-minute, tick) stayed empty. Running `classify_bar` on the returned OHLC works at every TF and contract uniformly with no DB lookup. Tradeoff: 70%+ density on long views could feel busy; mitigated by the existing per-label toggles in the side panel.
- **Tier 3 three-agent audit** (methodology / code correctness / output distribution) instead of inline single-pass. Reason: bar classifier feeds both live matcher dispatch AND Greg's discretionary chart read; blast radius warrants triangulation. All three converged on Y-CONDITIONAL with overlapping findings (BULL_180/BEAR_180 range engulfment, DUAL_TAIL 0.50 vs 0.85, COLOR_CHANGE takeout missing, DOJI literal vs range-relative).
- **Tripwire framing for the staged ship** — HANDOFF.md flagged "do NOT restart `live_runner.py`" with explicit fallback if the bot crashes (log under "Option B effective"). Reason: keeps the daily-review boundary clean so today's tape is attributable to the locked baseline.

## Open questions

- **Whether dashboard's `_classify_bars_inplace(live_bars)` should skip rather than overwrite** in the `_bars_table_payload` path. Currently overwrites — fine because Option B classifier matches bot's in-memory classifier today, but after bot restart they'll match too (same code, same env vars). Audit recommended gating on `b['labels'] == []`; queued as Option A item #4.
- **Whether the `is_doji` change is too loose.** New threshold (body/range < 0.10) emits DOJI at ~3% of bars across TFs vs the prior ~0%. UR131 leg counting will see more streak breaks after bot restart — leg counts smaller → fewer late-leg rejections. Methodology-correct per SRC sources but the empirical matcher impact won't be known until the Option A backtest.
- **What time today's session log should slot in.** Wrote 1400 ET as the timestamp; if the session ran longer or shorter, downstream session-log readers might want to adjust.

## Next steps

- **Greg: restart only the dashboard window** to see new chart markers. Bot stays running.
- **Greg: do NOT restart `live_runner.py`** until Option A is shipped + backtest re-locked. If bot crashes, note the time in `reviews/2026-05-19.md` under "Option B effective from <minute>" so the daily review partitions the tape.
- **Tonight after market close (~2 PM ET → ~16:00 ET):** ship Option A in this order:
  1. `DUAL_TAIL_WICK_SHARE` 0.50 → 0.85 (`bars.py:17`); kills the 30/56k symmetric-doji co-fire edge case.
  2. `BULL_TAIL`/`BEAR_TAIL` thresholds 33/60 → 15/85 (`bars.py:117-141`); per SRC-026 strict 85/15 rule.
  3. Env-var unification for ELEPHANT_BODY_SHARE + DUAL_TAIL_WICK_SHARE (single config source + startup logging on both bot and dashboard).
  4. `_classify_bars_inplace` skip live-1m overwrite path in `dashboard/app.py`.
  5. `write_historical_bar` call `classify_bar` on backfill rows (currently 82% of M26 `bars` rows have `labels_json='[]'` on disk).
  6. `OUTSIDE_BAR` strictness toggle (low pri).
  7. Single-tail body-color sensitivity (low pri, SRC-026 #75).
  8. `python -m system.backtest --fill-mode pessimistic --months 2` — compare per-setup vs `dfcb5a1` locked baseline; if within ±10% lock new baseline, else investigate divergent setup.
- **EOD review pipeline unchanged:** `python -m system.daily_review --et` + read `reviews/detector_hits_2026-05-19.txt` + scan logs for `[diagnostic]` + `[dashboard_writer] realized_pnl_today failed:`.

## Memory promotion candidates

- **Already saved this session:** `reference_bars_ts_utc_dual_format.md` (added to MEMORY.md). Worth keeping.
- **Proposed (awaiting Greg approval at session close):** `feedback_python_no_hotreload_staging.md` — the Option B/A staging pattern. Generalizable lesson: methodology fixes affecting a running bot's hot path can be staged via Python's no-hot-reload + a "do not restart" tripwire. Sibling to [[feedback-exempt-list-invalidates-baseline]].
- **NOT promoted:** matcher.py:340/479 + context.py:89 label-string consumption details — code-derivable via grep. Three-agent audit pattern — covered by existing [[feedback-audit-cycle-tiers]] memory.
