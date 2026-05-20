---
date: 2026-05-15
time: 17:01
tool: code
project: FuturesAnalytics
duration_minutes: 75
---

## Summary

Picked up Greg's Cash Command Center build mid-stream after yesterday's afternoon session ran out of context with the live execution path broken. Diagnosed and fixed all blockers preventing tomorrow's 9:30 AM ET autonomous trading: HTTP 422 on every Pydantic-body endpoint (root cause: `from __future__ import annotations` + locally-scoped BaseModels — FastAPI couldn't resolve types via `get_type_hints()`), missing `tzdata` for `zoneinfo.ZoneInfo("America/New_York")`, ProjectX `/api/Order/place` payload format wrong (was sending `type="MARKET"` / `side="BUY"` strings, needed numeric enums at top level — NOT wrapped in `{"request": ...}`), and zero broker-side protective legs on manual live orders. End state: API path verified by ProjectX's clean "instrument not active" rejection (proves auth + payload format work; just blocked by Topstep Combine after-hours window).

## Files touched

- `C:\Users\Greg\Documents\Claude\Projects\Futures Analytics\dashboard\app.py` (modified) — removed `from __future__ import annotations`; moved `ControlUpdate` and `OrderRequest` BaseModel classes from inside `_build_app()` to module scope (the actual 422 root cause); manual `/api/order` live branch now places full bracket (MARKET entry + opposite STOP + opposite LIMIT) with broker_stop_id and broker_target_id captured into manual_orders row
- `C:\Users\Greg\Documents\Claude\Projects\Futures Analytics\system\broker\orders.py` (modified) — `place_order` no longer calls `raise_for_status()` (was hiding ProjectX's actual error body); maps `type`/`side` to numeric enums (LIMIT=1, MARKET=2, STOP=4, STOP_LIMIT=5; BUY=0/Bid, SELL=1/Ask); added `flatten_position()` (uses /api/Position/searchOpen + opposite MARKET order) and `cancel_open_orders()` (uses /api/Order/searchOpen + cancel loop)
- `C:\Users\Greg\Documents\Claude\Projects\Futures Analytics\system\broker\live_runner.py` (modified) — EOD path in `_check_positions` now calls `cancel_open_orders` + `flatten_position` against the broker when `exit_reason=="eod"` (was only updating local DB record); kill switch path flattens broker position before `os._exit(0)`
- `C:\Users\Greg\Documents\Claude\Projects\Futures Analytics\dashboard\index.html` (modified) — `_submitOrderImpl` now renders broker outcome with color: red `BROKER REJECTED #N: <error>`, green `LIVE FILLED #N broker=<id> stop=<id> target=<id>`, yellow `submitted but no broker confirmation yet`. Greg couldn't tell whether clicks were succeeding before this fix.
- `C:\Users\Greg\.claude\projects\C--Users-Greg-Documents-Claude-Projects-Futures-Analytics\memory\feedback_cli_responsiveness.md` (created) — Greg's CLI freezes on long Claude turns; keep tool batches tight; ask steering questions in plain text, not AskUserQuestion
- `C:\Users\Greg\.claude\projects\C--Users-Greg-Documents-Claude-Projects-Futures-Analytics\memory\MEMORY.md` (created) — index pointer
- Python env: `pip install tzdata` (Python 3.14 on Windows has no bundled IANA tz database)

## Decisions made

- **Moved Pydantic models to module scope instead of just removing `from __future__ import annotations`.** Either fix works, but module-scope models are the FastAPI-recommended pattern and also work under PEP 563. The bug was silent — `/api/control` had been 422-ing all day yesterday too, so Greg's kill switch and mode toggle were dead UI buttons no one noticed.
- **Captured broker error response body instead of raising.** Yesterday's HTTP 400 was swallowed by `raise_for_status()` so we couldn't see ProjectX's actual complaint. After the change, ProjectX walked us through three layers of validation: "deserialization failed" → "request field required" (turned out to be a misleading FluentValidation message — payload goes at top level) → "instrument not active" (the clean response that means the auth + format are right).
- **TYPE_MAP / SIDE_MAP added inline in `place_order`.** Could have changed the OrderRequest dataclass `Literal` types but that ripples through every caller. Mapping at submission boundary is cheaper.
- **Manual live orders now place a FULL broker bracket, not just entry.** Without broker-side stop/target, a 422-style local-only state machine would orphan real positions when stop/target hit and the cancel-opposite-leg code targets `broker_stop_id` that was never set. Autonomous matcher signals already had this via `submit_signal()`.
- **EOD + kill flatten broker position before exit.** Local DB close + cancel-stop/target was leaving the actual broker position unmanaged. `flatten_position()` reads `/api/Position/searchOpen`, finds the matching contract row, fires an opposite MARKET order. Heuristic field detection for LONG vs SHORT (`type==1` / `size>0` / `side=="BUY"`) — may need tuning after seeing a real position payload tomorrow.
- **UI now color-codes broker outcome.** Greg couldn't differentiate "local row written" from "broker accepted" — the only feedback was `BUY #N (live)` regardless of outcome. Red/green/yellow + showing broker_order_id makes the distinction obvious. The dashboard reloads `index.html` on every request so this took effect with just a hard browser refresh (no server restart).

## Open questions

- **ProjectX numeric enum mapping is my best-guess from common conventions** (LIMIT=1, MARKET=2, STOP=4, STOP_LIMIT=5; BUY=0, SELL=1). If tomorrow's 9:30 AM ET orders return "OrderType value X invalid" we have one more layer to peel. The fact that we got past payload parsing into "instrument not active" suggests the enums are accepted, but only a real fill will confirm.
- **`flatten_position()` field detection is heuristic** — `type` / `side` / `size` fields on a ProjectX position response weren't observed live. If tomorrow's flatten fails, the position payload from `/api/Position/searchOpen` will tell us the actual schema.
- **Topstep Combine after-hours order window unconfirmed.** Tonight's rejection "Trading is currently unavailable" suggests RTH-only restriction, but I don't have authoritative docs. Tomorrow 9:30 AM ET is when we find out for real.
- **Auto-flatten for autonomous PENDING orders at EOD.** When the matcher fires a signal, `submit_signal` places 3 broker orders simultaneously (entry STOP, stop STOP, target LIMIT). If the entry STOP never triggers and EOD hits, only the "open"-status branch of `_check_positions` runs the cancel/flatten — pending orders may leave working stops/targets at the broker overnight. Low-priority because Velez setups typically trigger within minutes or never, and the orders go away at session end automatically — but worth a defensive cancel-all-working-orders on shutdown.

## Next steps

- **Tomorrow 9:30 AM ET:** leave dashboard + bot running, mode is persisted as `live` in control_state. Autonomous matcher fires during 9:30–1:30 ET prime hours. Manual override anytime via Buy/Sell Mkt — now with broker bracket and color-coded feedback.
- **Watch for "OrderType value invalid"** on the first real signal. If it appears, the numeric enums need re-mapping — read the response body for valid values.
- **Watch for `flatten_position` issues** if a real position needs flattening. Likely fields: `size` (positive = LONG by convention) or `type` (1=LONG, 2=SHORT).
- **Backtest tuning deferred** until Combine starts producing real numbers — was always Greg's plan post-shake-out.
- **Multi-instrument dashboard tabs** still deferred from HANDOFF; not blocking.

## Memory promotion candidates

- **FastAPI + `from __future__ import annotations` + locally-scoped Pydantic models = silent 422 on every endpoint that uses them.** FastAPI's `get_type_hints()` resolves annotations against the function's `__globals__`, so locally-defined BaseModels become unresolvable forward refs and the parameter falls back to "query" param mode. Worth a defensive-pattern memory: when adding new endpoints in this codebase, define BaseModels at module scope.
- **ProjectX `/api/Order/place` payload shape:** top-level `accountId, contractId, type, side, size, [limitPrice, stopPrice]` with numeric enums (LIMIT=1, MARKET=2, STOP=4, STOP_LIMIT=5; BUY=0, SELL=1). NOT wrapped in `{"request": ...}` despite the misleading "request field is required" error from FluentValidation. Worth pinning to project context — saves the next round-trip debug.
- **Capture HTTP error response bodies, never `raise_for_status()` when a broker is involved.** ProjectX (and most exchange APIs) put the actionable error message in the response body of 4xx responses. `raise_for_status()` drops it entirely.
