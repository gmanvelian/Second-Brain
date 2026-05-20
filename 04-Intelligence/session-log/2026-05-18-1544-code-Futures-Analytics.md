---
date: 2026-05-18
time: 15:44
tool: code
project: Futures-Analytics
duration_minutes: 60
---

## Summary

Investigated Greg's "memory usage is high" concern. Disambiguated two separate things being conflated: Claude Code context window percentage (the 36% — token budget on Anthropic's servers, unrelated to local RAM) vs. Windows physical RAM usage (12.25 GB / 38% of 32 GB — verified real, accounted for). Did a full system audit: removed 6 Xbox AppX packages (~100 MB + Game Bar background activity), audited Avira logs (zero detections across 6 monthly smart scans Nov 2025–Apr 2026; quarantine folder never created), and discussed Pop!_OS migration as a future option since Greg has it set up.

## Files touched

- C:\Users\Greg\Documents\Claude\Projects\Futures Analytics\BOOT_PROMPT.md (read — phase-3 worktree archive)
- C:\Users\Greg\Documents\Claude\Projects\Futures Analytics\.claude\settings.local.json (read — permission allowlist, 10 KB)
- C:\Users\Greg\.claude.json (inspected — 40 KB; biggest keys are cachedGrowthBookFeatures and projects)
- Xbox AppX packages (removed): Microsoft.XboxApp, Microsoft.GamingApp, Microsoft.XboxGameOverlay, Microsoft.XboxGamingOverlay, Microsoft.XboxSpeechToTextOverlay, Microsoft.Xbox.TCUI
- C:\Users\Greg\.claude\projects\C--Users-Greg-Documents-Claude-Projects-Futures-Analytics\memory\feedback_windows_keep_convenience_apps.md (created)
- C:\Users\Greg\.claude\projects\C--Users-Greg-Documents-Claude-Projects-Futures-Analytics\memory\project_popos_portability.md (created)
- C:\Users\Greg\.claude\projects\C--Users-Greg-Documents-Claude-Projects-Futures-Analytics\memory\MEMORY.md (modified — added two new index lines)
- C:\Users\Greg\.claude\CLAUDE.md (modified — two edits to Session Start + Session End protocols; see Decisions for rationale)

## Decisions made

- **Keep Grammarly, Phone Link, Bing Wallpaper.** Greg uses and likes all three. Stop pitching their removal in cleanup suggestions.
- **Remove Avira (planned, not yet executed).** Six months of monthly smart scans returned zero malware; quarantine folder never created. Defender is currently disabled because Avira registered as primary AV — uninstalling Avira will auto-reactivate Defender. Greg will do the uninstall manually via Settings → Apps (Avira's uninstaller wants UI interaction; not suitable for programmatic removal).
- **Defer Pop!_OS migration.** No urgent driver — the 75% RAM incident was transient (pagefile peak only 87 MB, no low-memory events). Pop is ready as a future experiment but not this week.
- **Kept 3 system-bridging Xbox packages**: XboxIdentityProvider (MS account auth), XboxGameCallableUI (system UI), GamingServices (Store dependency). Removing these would risk Store / login functionality for minimal RAM gain.
- **MCP cleanup approach** — pending. User said "yes to disconnecting MCP" but didn't specify which connectors (Canva/Clerk/Figma/Gmail/Google_Drive/Supabase/Vercel/Zapier all live in claude.ai web UI; stitch/clockify are user-scope `.claude.json` togglable via `~/.claude/scripts/mcp-toggle.js`). Punted to a future session.
- **Re-enabled Second Brain auto-read at session start.** Mid-session reflection on whether the session-log protocol earned its weight. Empirically tested by reading 4 prior logs — surfaced ~10 operational items (live paper-mode config overrides, tripwires, queued line-ref fixes, open methodology questions on S004 reversal vs continuation) that MEMORY.md + HANDOFF.md alone don't preserve. Greg confirmed his original "always read Second Brain" rule was correct; dropping it for perceived time cost was a regression. Updated global CLAUDE.md Session Start protocol with new step 4: read 1–2 most recent matching session logs from the vault.
- **Added explicit staleness sweep to Session End protocol.** Existing Staleness Prevention rule fires per-turn ("before ending your turn"), not session-end-specific. Session close is the natural hindsight checkpoint — this session itself was the example: neither protocol edit would have surfaced without explicit reflection. New Session End is a 2-step ordered protocol: (1) staleness sweep with required "no new gaps" statement if nothing surfaced (so it can't be skipped silently); (2) write session-log summary.

## Open questions

- Which specific MCP connectors does Greg actually want to disconnect? (Pending.)
- What did "phase tick rate" refer to when Greg said it didn't help at 75%? (Asked, no answer yet — likely was tested on the matcher polling cadence or similar, which wouldn't affect RAM directly.)

## Next steps

- **Next session topic**: "Multi agents in one terminal" — Greg's intended next quest.
- **Pending action**: Greg uninstalls Avira via Settings UI (VPN first, then Optimizer, then Security last; then reboot; then verify `Get-MpComputerStatus` shows `AntivirusEnabled=True` and `RealTimeProtectionEnabled=True`).
- **Pending action**: Greg decides which claude.ai connectors to detach from the claude.ai → Settings → Connectors UI.
- **Future option**: Pop!_OS migration test. ProjectX is REST-API only so broker side is fully portable; SQLite + Python + Node + WezTerm Lua config all cross-platform. Estimated 2–3 hours end-to-end. See `project_popos_portability.md`.
- **Verify new protocol on next session boot**: confirm step 4 (recent session-log auto-read) actually fires and produces a useful read; confirm Session End step 1 (staleness sweep) runs even when there are no gaps. Filename-slug matching may need tuning given inconsistent project names across logs (`FuturesAnalytics` / `Futures-Analytics` / `FuturesAnalytics-phase-2-trifecta` / `WezTermSetup`).

## Memory promotion candidates

Already promoted this session:
- `feedback_windows_keep_convenience_apps.md` — don't pitch Grammarly/Phone Link/Bing removal
- `project_popos_portability.md` — Pop!_OS readiness + futures-analytics portability surface

Nothing further worth promoting from this session.
