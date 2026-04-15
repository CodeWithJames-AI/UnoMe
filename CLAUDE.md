# UnoMe — "You Know Me"

*Unified AI platform. BuildingAI + OpenWork. Two self-contained services, one seamless UI.*

## Context Directory

Read these files only when the condition matches:

- IF starting a new task → read `docs/BLUEPRINT.md` for phase plan and architecture
- IF coding backend → read `docs/BLUEPRINT.md#api-contract` for endpoint specs
- IF coding frontend → read `docs/BLUEPRINT.md#production-topology` and use z.ai MCP for OpenWork component patterns
- IF writing tests → read `docs/BLUEPRINT.md#acceptance-criteria` for the current phase
- IF integrating UI → query OpenWork via z.ai MCP surgically, do NOT load OpenWork code into workspace
- IF tests fail → re-read the relevant source files before fixing, never guess
- IF scope changes → update `docs/PRD.md`, `docs/BLUEPRINT.md`, and this file, commit together
- IF context feels bloated → stop, consolidate rules, remove contradictions before continuing
- IF unsure about a decision → read `docs/BLUEPRINT.md#rejected-alternatives` for context on why

## Rules

- Every PR must pass: `pnpm lint && pnpm typecheck && pnpm test && pnpm build`
- No CI bypass (`--no-verify`) unless explicitly approved
- TypeScript strict — no `any` without justification
- Neutral prompts: don't bias toward an outcome (e.g. "review this code" not "find bugs in this code")
- Separate research from implementation: research first with fresh context, then implement with fresh context
- Tests are the completion milestone: unless tests pass, task is NOT complete. Do NOT edit tests to pass them.
- For UI work: screenshot + verify design/behavior before claiming complete

## Architecture

- **BuildingAI** (host): NestJS 11 + TypeORM + PostgreSQL 17 + Redis + Vue 3 + Tauri
- **OpenWork v0.11.207** (orchestrator): OpenCode SDK + Express + sessions/skills/permissions
- **Adapter**: one NestJS proxy module + Vue components in BuildingAI's vertical nav bar
- Both self-contained, own their state, update independently
- OpenWork features in nav bar: workflows, skills, messaging, permissions

## Key Commands

```
pnpm install --frozen-lockfile       pnpm lint
pnpm typecheck                       pnpm --filter @buildingai/api test
pnpm build                           docker-compose up -d
```

## Environment

- Node 24, pnpm 10.20, bun 1.3, Docker 29.4, Docker Compose 5.1.3
- Use `docker-compose` (not `docker compose`)
- gh CLI authenticated as CodeWithJames-AI
- DevContainer: `.devcontainer/devcontainer.json`

## Limits

- CLAUDE.md < 150 lines | AGENTS.md < 300 | Adapter < 20 files | New Vue < 15 files
- No package > 50 source files | No duplicate abstractions | No redundant markdown

## Reference

- Repo: https://github.com/CodeWithJames-AI/UnoMe
- PRD: `docs/PRD.md` | Blueprint: `docs/BLUEPRINT.md`
- BuildingAI: https://github.com/BidingCC/BuildingAI
- OpenWork: https://github.com/different-ai/openwork
