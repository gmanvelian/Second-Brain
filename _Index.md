# Second Brain

Welcome home. This is your central knowledge base — browse it in Obsidian, and AI agents read it too.

## Active Projects

- [[02-Projects/BodylineSwimShop/README|Bodyline Swim Shop]] — Website redesign on Shopify
- [[02-Projects/ClubManager/README|Club Manager]] — Web app development
- [[02-Projects/Sandpipers/README|Sandpipers]] — Coaching schedule & team ops
- [[02-Projects/FuturesAnalytics/README|Futures Analytics]] — Velez-methodology futures trading bot (Topstep Combine)

## Quick Links

- [[01-Context/profile|About Me]]
- [[01-Context/preferences|Working Preferences]]
- [[01-Context/tools-and-stack|Tools & Stack]]
- [[03-SOPs/website-build|Website Build SOP]]
- [[04-Intelligence/meetings/|Meeting Notes]]

## Recently Updated

- **2026-05-19** — [[02-Projects/FuturesAnalytics/README|FuturesAnalytics]]: methodology DB **v1.29** shipped with 6 new sheets incorporating Greg's video-review findings + YouTube/transcript links for the 12 wired matcher setups; repo + Second Brain pushed to GitHub (private) with Windows↔Linux dual-boot sync conventions in `PLATFORM_NOTES.md`. Companion docs in repo: `docs/greg_video_review_findings.md` (Greg's canonical interpretations), `docs/top_rejectors_methodology.md` (gate-by-gate source map), `docs/v1.29_design.md` (Live Funded semi-auto execution + intra-bar matcher cadence) — sequenced AFTER gate-tightening (HANDOFF backlog #1/#2/#8).

## Agent Routing

```yaml
- project: BodylineSwimShop
  vault_folder: 02-Projects/BodylineSwimShop
  cwd_paths:
    - C:\Users\Greg\Documents\Claude\Projects\BodylineSwimShop
  context_paths:
    - C:\Users\Greg\Documents\Claude\Projects\BodylineSwimShop\CLAUDE.md
    - C:\Users\Greg\Documents\Second Brain\02-Projects\BodylineSwimShop\README.md
  description: Shopify theme for Bodyline Swim Shop
- project: ClubManager
  vault_folder: 02-Projects/ClubManager
  cwd_paths:
    - C:\Users\Greg\Documents\club-manager
    - C:\Users\Greg\Documents\Claude\Projects\ClubManager
  context_paths:
    - C:\Users\Greg\Documents\club-manager\CLAUDE.md
    - C:\Users\Greg\Documents\Second Brain\02-Projects\ClubManager\README.md
  description: Web app for swim club management (has graphify output)
- project: Sandpipers
  vault_folder: 02-Projects/Sandpipers
  cwd_paths:
    - C:\Users\Greg\Documents\Claude\Projects\Sandpipers
  context_paths:
    - C:\Users\Greg\Documents\Claude\Projects\Sandpipers\CLAUDE.md
    - C:\Users\Greg\Documents\Second Brain\02-Projects\Sandpipers\README.md
  description: Coaching schedule and team ops
- project: FuturesAnalytics
  vault_folder: 02-Projects/FuturesAnalytics
  cwd_paths:
    - C:\Users\Greg\Documents\Claude\Projects\Futures Analytics
  context_paths:
    - C:\Users\Greg\Documents\Claude\Projects\Futures Analytics\HANDOFF.md
    - C:\Users\Greg\Documents\Second Brain\02-Projects\FuturesAnalytics\README.md
  description: Velez-methodology futures trading bot on Topstep Combine (MES primary)
```
