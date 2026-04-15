# Phase 0 Contract: CI/CD + DevContainer + Baseline Audit

**Phase**: 0
**Created**: 2026-04-16

## Deliverables

1. GitHub Actions CI workflow (`.github/workflows/ci.yml`) with lint, typecheck, test, build jobs
2. Verified devcontainer builds and runs successfully
3. Baseline performance benchmark recorded
4. Audit report: BuildingAI test coverage, OpenWork orchestrator API surface

## Acceptance Tests

```bash
# CI workflow exists and is valid
test -f .github/workflows/ci.yml

# BuildingAI builds successfully
cd buildingai && pnpm install --frozen-lockfile && pnpm lint && pnpm typecheck && pnpm build

# BuildingAI API tests pass (if test DB available)
cd buildingai && pnpm --filter @buildingai/api test

# DevContainer config is valid
devcontainer build --workspace-folder .
```

## Regression Gate

```bash
# No existing features broken — BuildingAI starts
cd buildingai && docker-compose -f ../docker-compose.yml up -d
sleep 30
curl -f http://localhost:4090/consoleapi/health
```

## Visual Verification
- [ ] BuildingAI loads in browser at localhost:4090
- [ ] All existing nav items visible and functional

## Definition of Done
- [ ] All acceptance tests pass (DO NOT edit tests to make them pass)
- [ ] All regression tests pass
- [ ] CI workflow runs green on push
- [ ] DevContainer builds without errors
- [ ] No orphan files or dead code introduced
- [ ] Docs updated if scope changed
- [ ] Git tag created: `git tag -a unome/phase0 -m "Phase 0: CI/CD + DevContainer + audit"`

## Do NOT
- Add features beyond what's specified
- Refactor code not touched by this contract
- Create helper utilities for one-time operations
- Leave TODO comments or stubs
- Modify BuildingAI source code (this phase is audit + infrastructure only)
