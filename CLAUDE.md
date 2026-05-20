# Second Brain — AI Navigation Index

This is Greg's central knowledge base. It contains persistent context about who he is, how he works, active projects, standard operating procedures, research, and resources.

## How to use this vault

1. **Start here.** Read this file first to orient yourself.
2. **Check the project folder** if the user's request relates to a specific project — each has a `README.md` with current status, key contacts, and decisions.
3. **Check `01-Context/`** for who Greg is, his preferences, and his tooling.
4. **Check `03-SOPs/`** before doing any recurring workflow — there may already be a documented process.
5. **Update as you learn.** If a conversation reveals new project decisions, contacts, preferences, or processes, update the relevant file here.

## Folder Map

```
01-Context/       → About Greg: roles, preferences, tools, working style
02-Projects/      → One subfolder per active project with README.md
03-SOPs/          → Repeatable workflows and processes
04-Intelligence/  → Meeting notes, research, decisions
05-Resources/     → Reference material, templates, guides
06-Onboarding/    → For onboarding team members or clients
```

## Active Projects (quick reference)

| Project | Folder | Status | Key Contact |
|---------|--------|--------|-------------|
| Bodyline Swim Shop | `02-Projects/BodylineSwimShop/` | Building Shopify store | Ron (owner) |
| Club Manager | `02-Projects/ClubManager/` | Active development | — |
| Sandpipers | `02-Projects/Sandpipers/` | Coaching schedule rebuild | — |
| Futures Analytics | `02-Projects/FuturesAnalytics/` | Live trading (Topstep Combine, MES) | — |

## Rules

- Keep files concise — bullet points and short paragraphs
- Use `README.md` in each project folder as the entry point
- Date-stamp meeting notes: `YYYY-MM-DD-topic.md`
- When updating, note what changed and when
- Don't duplicate info — link between files using `[[wikilinks]]` or relative paths

## Session Rules

- **Start of session:** Read this file and the relevant project README before doing any work.
- **End of session:** Before closing, update the vault with anything learned:
  - New decisions → `04-Intelligence/decisions/`
  - New contacts or requirements → project README
  - New processes discovered → `03-SOPs/`
  - Meeting notes → `04-Intelligence/meetings/`
  - Status changes → project README and the Active Projects table above
  - New preferences or corrections → `01-Context/preferences.md`
- **Never skip the end-of-session update.** Even if the session was short, capture what changed.

## Agent Routing (machine-targeted)

This vault is the orientation hub for every Claude session — both Claude Code and Cowork treat it as the first-class source of context. The routing table lives in `_Index.md` as a YAML block under the "Agent Routing" heading; each row binds a project to its `cwd_paths`, `context_paths`, and vault folder.

At session start, the agent matches its current working directory (Code) or mounted folder (Cowork) against `cwd_paths` — longest prefix wins — and reads every file in the matched row's `context_paths` in order.

Code and Cowork are symmetric consumers of this vault, with two reach asymmetries worth knowing:

1. **Filesystem scope.** Code has full Windows filesystem access, so every path in the YAML is directly reachable. Cowork is sandboxed Linux with only explicitly mounted folders reachable — if a `context_paths` entry isn't under a mount, Cowork silently skips it and notes the skip in the session-start report.
2. **Skills surface.** Code enumerates skills by scanning BOTH `~/.claude/skills/` and `<cwd>/.claude/skills/` on disk. Cowork's skills are plugin-based and enumerated in the system prompt at session start. Both produce a skills count for the report line; the mechanism differs.

MCP availability is noted from the tool list in both environments.

Project-specific rules (graphify reads, domain glossaries, per-project conventions) belong in each project's own `CLAUDE.md` — not here and not in the global `~/.claude/CLAUDE.md`.

## Session-End Protocol (machine-targeted)

Sessions terminate with a write to `04-Intelligence/session-log/`. File naming is `<YYYY-MM-DD>-<HHMM>-<tool>-<project>.md`. Format spec lives in `04-Intelligence/session-log/README.md`.

Reads at session start (Phase 1 tier 3 extension): the 5 most recent entries. This gives every new session a window into the last day or two of work across all tools.

Cowork without vault mount: writes session-log to the current mount; user or next Code session promotes it to the vault. This is a graceful degradation, not a parity failure.

Logs are append-only. Old entries archive in place — don't delete or rewrite. If the folder grows beyond ~500 files, consider monthly subfolders (`session-log/2026-04/`) but not before.
