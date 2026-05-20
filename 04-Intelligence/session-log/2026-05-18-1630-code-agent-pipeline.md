---
date: 2026-05-18
time: 16:30
tool: code
project: agent-pipeline
duration_minutes: 60
---

## Summary

Built a 3-agent SDK-based coding pipeline (Architect=Claude → Engineer=GPT/Codex → Reviewer=DeepSeek) at a new project root outside Futures Analytics. Added codex CLI as a swappable Engineer backend behind `ENGINEER_BACKEND` env var. Pipeline syntactically verified, deps installed in venv, codex CLI installed and authenticated — but never run end-to-end. Greg wrapped before populating `.env` with API keys.

## Files touched

- `C:\Users\Greg\Documents\Claude\Projects\agent-pipeline\pipeline.py` (created) — 3-agent async orchestrator with codex-vs-openai-SDK Engineer switch
- `C:\Users\Greg\Documents\Claude\Projects\agent-pipeline\requirements.txt` (created) — anthropic, openai, python-dotenv
- `C:\Users\Greg\Documents\Claude\Projects\agent-pipeline\.env.example` (created, then updated with codex options)
- `C:\Users\Greg\Documents\Claude\Projects\agent-pipeline\.env` (created from template, `ENGINEER_BACKEND=codex` set; **keys NOT populated**)
- `C:\Users\Greg\Documents\Claude\Projects\agent-pipeline\.gitignore` (created)
- `C:\Users\Greg\Documents\Claude\Projects\agent-pipeline\.venv\` (created via `python -m venv`; anthropic 0.102.0, openai 2.37.0, python-dotenv 1.2.2 installed)
- Global: `npm install -g @openai/codex` — codex-cli 0.131.0 installed; `codex login status` reports "Logged in using ChatGPT"

## Decisions made

- **SDK route over CLI subprocess** for the primary architecture — DeepSeek has no first-party agentic CLI; uniform abstraction across 3 providers via OpenAI SDK (DeepSeek piggybacks via `base_url="https://api.deepseek.com"`).
- **Pipeline pattern: Architect → Engineer → cheap-Reviewer loop** with max 3 attempts — DeepSeek is ~50x cheaper than Opus per output token, so iterating reviewer is cheap; Claude reserved for high-leverage bookends (architect, arbiter).
- **Codex CLI as swappable Engineer backend** (`ENGINEER_BACKEND=codex`) rather than as a 4th agent — lets Greg A/B-compare codex vs raw OpenAI SDK GPT with one env var flip.
- **Codex sandbox flags**: `--ephemeral --skip-git-repo-check -s read-only -a never -o <tempfile>` — codex stays contained (read-only sandbox), no session persistence, no approval prompts to stall pipeline, clean final output via tempfile (discards codex's progress chatter on stdout).
- **Project placed outside Futures Analytics** at `C:\Users\Greg\Documents\Claude\Projects\agent-pipeline` — keeps experimental orchestration code separate from the locked Futures Analytics trading baseline.
- **Codex auth via ChatGPT login** (not API key) — affects cost bucket: codex calls draw from ChatGPT plan credits, NOT `OPENAI_API_KEY` billing. `OPENAI_API_KEY` is only consumed if `ENGINEER_BACKEND=openai`.

## Open questions

- **Pricing claim from Greg** — third-party explanation he pasted, that `claude -p` moves to a $100/mo Max credit bucket on June 15 2026. I could not verify (my knowledge cutoff is Jan 2026). Treating as unverified. If true, the SDK route is right long-term regardless.
- **Codex output format** untested — does `codex exec -o <file>` write clean fenced code, or include reasoning prose? First run will reveal. Reviewer step (DeepSeek) may flag if format is off.
- **DeepSeek base URL** (`https://api.deepseek.com`) never actually hit this session.

## Next steps

1. Greg populates `.env` with `ANTHROPIC_API_KEY` and `DEEPSEEK_API_KEY` (OpenAI key not needed in codex mode).
2. Smoke test: `.venv\Scripts\python.exe pipeline.py "Write a Python function that returns the nth Fibonacci number iteratively"` — small/cheap first task.
3. Watch step `[2/3]` for codex agent-loop output quirks — may need to tighten `ENGINEER_SYS` prompt or adjust sandbox flag.
4. If codex output is messy, fall back to `ENGINEER_BACKEND=openai` for an apples-to-apples baseline.
5. Future expansion: 4th agent (Tester or Documenter), or swap to fan-out+judge pattern for genuine model-diversity wins.

## Memory promotion candidates

- **Project memory**: agent-pipeline experimentation lives at `C:\Users\Greg\Documents\Claude\Projects\agent-pipeline` — a separate sandbox for multi-agent orchestration patterns, distinct from Futures Analytics.
- **Reference memory**: Codex CLI installed globally (`@openai/codex` 0.131.0), authenticated via ChatGPT login. Calls draw from ChatGPT plan, not `OPENAI_API_KEY`.
