# Club Manager

## Status
**Active development**

## Overview
Web application for managing swim clubs. Complex codebase with graphify knowledge graph.

## Stack
- Frontend: React / Next.js
- Database: Supabase (with RLS policies using Clerk JWT)
- Auth: Clerk
- Hosting: TBD

## Graphify
- Knowledge graph exists for this project
- Graph outputs in the club-manager project directory
- Use `/graphify --update` to refresh after major changes

## Project Files
- Code: `C:\Users\Greg\Documents\club-manager\`
- Project CLAUDE.md with custom skills registered

## Project-Specific Skills
- `/new-page` — scaffolds admin pages with sidebar and Supabase imports
- `/supabase-migrate` — RLS policies with club_id + Clerk auth
- `/ui-consistency` — enforces button/card/pill class patterns
- `/audit-feature` — traces all files touching a feature

## Notes
- Recently cleaned up global CLAUDE.md — moved project-specific skills out of global scope into club-manager's `.claude/skills/` directory
