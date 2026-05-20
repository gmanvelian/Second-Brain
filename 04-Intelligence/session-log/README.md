# Session Log

Structured summaries written at the end of every Claude session (Code or Cowork). Read at the start of the next session in either tool to maintain context continuity.

## File naming

`<YYYY-MM-DD>-<HHMM>-<tool>-<project-or-general>.md`

Example: `2026-04-19-1545-cowork-BodylineSwimShop.md`, `2026-04-19-1612-code-general.md`

Sortable by filename. Tool and project visible at a glance.

## File format

Each session-log file has a YAML frontmatter block followed by Markdown body sections.

### Frontmatter

```yaml
---
date: 2026-04-19
time: 15:45
tool: cowork           # or code, codex
project: BodylineSwimShop  # or "general" if vault-mode or unscoped
duration_minutes: 45   # optional, best estimate
---
```

### Body (Markdown sections)

- `## Summary` — two to three sentences capturing the essence of the session. What was the user trying to do? What was accomplished?
- `## Files touched` — bulleted list with absolute Windows paths; annotate `(modified)`, `(created)`, or `(deleted)`.
- `## Decisions made` — bulleted list of decisions with one-line reasons.
- `## Open questions` — optional; things unresolved.
- `## Next steps` — optional; what to pick up next.
- `## Memory promotion candidates` — optional; facts worth promoting to long-term memory (via Code's auto-memory or Cowork's `MEMORY.md`). Leave empty if none.

## Read window for session-start

The Phase 1 protocol's tier 3 read pulls the most recent entries from this folder. Default: last 5 entries OR last 7 days, whichever is fewer. Older entries stay archived as historical record.

## Promotion vs logging

Session-logs capture *what happened*. Memory captures *what matters long-term*. Don't auto-promote every log entry to memory — that bloats memory and makes it less useful. Promote selectively: durable preferences, architectural decisions, constraints, things that will still matter in 30 days.
