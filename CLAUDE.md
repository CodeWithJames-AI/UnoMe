# UnoMe — Project Instructions

UnoMe integrates OpenWork into BuildingAI as a unified AI platform. Two self-contained services connected via a thin adapter layer. Users see one app.

## Architecture

- **BuildingAI** (host): NestJS 11 + TypeORM + PostgreSQL 17 + Redis + Vue 3 frontend + Tauri desktop
- **OpenWork** (orchestrator): OpenCode SDK + Express + session/skill/permission management
- **Adapter**: One NestJS proxy module (`packages/api/src/modules/workflow-proxy/`) + Vue components
- Both services self-contained, own their state, update independently via Docker images

## Development Rules

### Quality Gates
- Every PR must pass: `pnpm lint && pnpm typecheck && pnpm test && pnpm build`
- No CI bypass (`--no-verify`) unless explicitly approved
- TypeScript strict mode — no `any` without justification

### Regression Prevention
- Backend: Jest + supertest integration tests
- Frontend: Playwright + Chrome DevTools MCP for interactive testing
- All new features must include regression tests for existing features
- If a new feature breaks an old one, fix the regression before merging

### Context Bloat Prevention
- CLAUDE.md: < 200 lines (this file)
- AGENTS.md: < 300 lines
- Adapter module: < 20 files
- New Vue components: < 15 files total
- No package > 50 source files
- No duplicate abstractions, no redundant markdown files
- Use z.ai MCP to query OpenWork codebase surgically — don't load OpenWork code into workspace

### Git Practices
- Conventional commits: `feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `test:`
- Issue ticket for every phase/task
- PR for every phase with template
- Annotated tags for every checkpoint: `git tag -a unome/phase<N> -m "description"`

### Checkpoints and Rollback
- Each phase produces annotated git tag with: build hash, test results, lockfile, .env template
- Checkpoints must recreate full app behavior — no weak revert-only checkpoints
- Rollback: `docker compose down && git checkout <tag> && docker compose up -d && smoke test`

### Repository Hygiene
- No orphan files, no dead code, no unused dependencies
- Keep instructions minimal — agents should read code, not docs
- Don't create helper utilities for one-time operations
- Don't add features, refactor code, or make improvements beyond what was asked

## Key Commands

```bash
pnpm install --frozen-lockfile    # Install dependencies
pnpm lint                         # Lint all packages
pnpm typecheck                    # TypeScript type checking
pnpm --filter @buildingai/api test  # Run backend tests
pnpm build                        # Build all packages
docker compose up -d              # Start all services
docker compose down               # Stop all services
```

## Integration Points

- BuildingAI API proxies `/api/workflow/*` to OpenWork orchestrator (internal Docker network)
- Auth: shared JWT_SECRET, BuildingAI exchanges JWT for OpenWork session token server-side
- No JWT in URLs or query parameters
- OpenWork features in BuildingAI vertical nav bar: workflows, skills, messaging, permissions

## Warnings

- BuildingAI has NO CI/CD pipeline — must be created (Phase 0)
- Lint/TypeScript/Prettier configured but not enforced by CI
- Jest configured in API package only — frontend tests need Playwright setup

## Reference

- PRD: `docs/PRD.md`
- Blueprint: `docs/BLUEPRINT.md`
- BuildingAI repo: https://github.com/BidingCC/BuildingAI
- OpenWork repo: https://github.com/different-ai/openwork
