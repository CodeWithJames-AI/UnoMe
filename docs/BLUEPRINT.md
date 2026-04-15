# UnoMe — Implementation Blueprint

**UnoMe — "You Know Me"**

*A platform that truly knows you well. A unified platform that houses the most powerful AI agent tools to unlock your productivity.*

**Version:** 1.0.0
**Date:** 2026-04-16
**Status:** Approved (Ralplan Consensus v4 — Critic APPROVED)
**Consensus:** Planner + Architect + Critic (3 iterations)

---

## 1. Decision Record

### Chosen Architecture: Self-Contained Services + Thin Adapter + Unified UI

BuildingAI and OpenWork remain self-contained. Integration is at the UI layer and API proxy layer only. Users see one unified application.

### Rejected Alternatives

| Option | Rejection Rationale |
|--------|-------------------|
| **Parallel-App Merge** | Two competing UI stacks (Vue vs SolidJS), dual AI SDKs in same process, context explosion. Violates context bloat requirements. |
| **Fork-and-Replace** | Discards BuildingAI's mature v26 backend, Vue/NuxtUI frontend, and 20+ shared packages. Highest regression risk. |
| **Adopt OpenCode SDK** | BuildingAI has its own `@buildingai/ai-sdk`. Replacing it breaks all AI features, agents, and RAG pipeline. Creates vendor lock-in. |
| **Clean Rewrite** | 3-5x effort vs adapter pattern. Reimplementing OpenWork concepts without its SDK means reimplementing OpenCode's session/event model. Highest bug risk. |
| **Feature Extraction** | OpenWork is architecturally coupled to OpenCode SDK. Extracting features without the SDK requires reimplementation; keeping it creates dual-SDK bloat. |

### Key Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| SDK reconciliation | OpenWork keeps OpenCode SDK, BuildingAI keeps its own AI SDK | Each service owns its stack; no cross-contamination |
| State management | Separate state per service, no shared DB | Avoids distributed state complexity |
| Frontend framework | Vue 3 for all new UI (SolidJS → Vue rewrite proven) | Consistency with BuildingAI's stack |
| Runtime engine | OpenWork's orchestrator + OpenCode | Proven session/permission/event model |
| Integration layer | Thin NestJS proxy module + Vue components | Minimal code footprint, easy to maintain |
| Deployment | Docker Compose with both services | Single command to start entire platform |
| Development | DevContainer + z.ai MCP | Consistent env, surgical OpenWork queries |

---

## 2. Pre-Mortem Analysis

| # | Failure Scenario | Prevention | Detection | Recovery |
|---|-----------------|-------------|-----------|----------|
| 1 | OpenWork orchestrator unavailable | Health checks + retry with exponential backoff | BuildingAI API proxy returns 502 | Docker restart policy; nav bar shows "Workflows unavailable" |
| 2 | OpenCode SDK breaking change | Pin orchestrator to minor version; contract tests in CI | Contract test failures | Pin to last working version; create migration task |
| 3 | Database migration fails (BuildingAI) | Test against snapshot before apply | Migration script exit code | Restore from backup; checkout previous tag |
| 4 | Frontend can't reach workflow API | Circuit breaker in BuildingAI API proxy | Frontend error boundary | Graceful degradation: hide workflow nav items |
| 5 | Auth token exchange fails | Token validation on both sides | 401/403 spike in Gateway logs | Verify JWT_SECRET match across services |
| 6 | Performance >2x baseline | Benchmarks in CI pipeline | CI threshold breach | Profile bottleneck; add caching or merge hot paths |
| 7 | Context bloat from integration | File/line count guardrails enforced in CI | CI lint rule violation | Enforce limits; prune dead code |

---

## 3. Production Deployment Topology

```yaml
# docker-compose.yml (UnoMe)
services:
  postgres:        # BuildingAI PostgreSQL 17
  redis:           # BuildingAI Redis
  unome-api:       # BuildingAI NestJS backend (port 4090)
  unome-web:       # Vue 3 frontend (served via NestJS serve-static)
  openwork:        # OpenWork orchestrator + server (internal only)
```

- OpenWork service is **not exposed to host** — only `unome-api` communicates with it via internal Docker network
- Health check: `curl -f http://openwork:4091/health`
- Restart policy: `on-failure:3`
- **Note**: Use `docker-compose` (standalone) not `docker compose` (plugin)

---

## 4. Authentication Architecture

```
Client → BuildingAI API (JWT validation via NestJS guard)
       → Internal REST proxy to OpenWork (JWT exchanged for session token)
       → OpenWork validates session token

Client → BuildingAI API → SSE proxy → OpenWork SSE stream
       (JWT in Authorization header, exchanged server-side)
```

- Both services share `JWT_SECRET` environment variable
- BuildingAI API proxies `/api/workflow/*` to OpenWork internally
- SSE connections: BuildingAI validates JWT, then proxies SSE stream server-side
- No JWT in URLs or query parameters (security requirement)

---

## 5. API Contract (Adapter Layer)

BuildingAI proxies these endpoints to OpenWork orchestrator:

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/workflow/health` | GET | OpenWork health check (proxied) |
| `/api/workflow/sessions` | GET/POST | List/create sessions |
| `/api/workflow/sessions/:id` | GET | Get session detail |
| `/api/workflow/sessions/:id/prompt` | POST | Send prompt to session |
| `/api/workflow/sessions/:id/abort` | POST | Abort running session |
| `/api/workflow/sessions/:id/events` | GET (SSE) | Stream session events |
| `/api/workflow/skills` | GET | List installed skills |
| `/api/workflow/skills/import` | POST | Import skill from directory |
| `/api/workflow/permissions/pending` | GET | List pending permission requests |
| `/api/workflow/permissions/:id/reply` | POST | Respond to permission request |

---

## 6. Implementation Phases

### Phase 0: CI/CD + DevContainer + Baseline Audit

**PR:** #1
**Deliverables:**
- GitHub Actions workflow (`.github/workflows/ci.yml`): lint, typecheck, test, build
- DevContainer configuration (`.devcontainer/`) with Node 22, pnpm, bun, Rust, PostgreSQL, Redis
- Baseline performance benchmark
- Audit report: BuildingAI test coverage, OpenWork orchestrator API surface

**Acceptance:**
```bash
pnpm lint && pnpm typecheck && pnpm --filter @buildingai/api test && pnpm build
```

**Checkpoint:** `git tag -a unome/phase0 -m "Phase 0: CI/CD + DevContainer + audit"`

**Rollback:** Tag is the baseline — checkout and rebuild.

---

### Phase 1: OpenWork Orchestrator Integration

**PR:** #2
**Deliverables:**
- OpenWork as Docker service in `docker-compose.yml`
- BuildingAI API proxy module (`packages/api/src/modules/workflow-proxy/`)
- Auth bridge: BuildingAI JWT to OpenWork session token exchange
- Health check endpoint for orchestrator

**Surgical scope:** One new NestJS module, one new Docker service.

**Acceptance:**
```bash
docker compose up -d
curl -f http://localhost:4091/health                  # OpenWork direct
curl -f http://localhost:4090/api/workflow/health      # Proxied through BuildingAI
```

**Checkpoint:** `git tag -a unome/phase1 -m "Phase 1: orchestrator integration"`

**Rollback:**
```bash
docker compose down
git checkout unome/phase0
docker compose up -d
sleep 30
curl -f http://localhost:4090/consoleapi/health
```

---

### Phase 2: Unified UI — Vertical Nav Bar

**PR:** #3
**Deliverables:**
- New Vue 3 components in BuildingAI client:
  - Workflow timeline panel
  - Skills manager panel
  - Extensions manager
  - Messaging integration (Slack/Telegram via OpenWork Router)
  - Permission dialog (allow-once / allow-session / deny)
- All SolidJS UI patterns rewritten to Vue 3 (proven approach)
- Playwright test suite for new features
- Chrome DevTools MCP tests for interactive verification

**Acceptance:**
```bash
# New features work
pnpm --filter buildingai-client exec playwright test --grep "workflow|messaging|skills"
# Existing features unaffected (regression)
pnpm --filter buildingai-client exec playwright test --grep "chat|agent|knowledge"
```

**Checkpoint:** `git tag -a unome/phase2 -m "Phase 2: unified UI"`

**Rollback:** Checkout `unome/phase1`, rebuild frontend only.

---

### Phase 3: Rebrand to UnoMe

**PR:** #4
**Deliverables:**
- All `BuildingAI` / `buildingai` strings replaced with `UnoMe` / `unome` in user-facing code
- Package namespaces: `@buildingai/*` remain internally; external references use `@unome/*`
- Assets: logos, favicons, banners replaced
- Docker image names and environment variable prefixes updated
- Documentation and README updated

**Acceptance:**
```bash
# No BuildingAI references in user-facing files
! grep -r "BuildingAI" packages/client/src/ packages/api/src/
# Visual verification via Playwright screenshot comparison
pnpm --filter buildingai-client exec playwright test --grep "branding"
```

**Checkpoint:** `git tag -a unome/phase3 -m "Phase 3: rebrand to UnoMe"`

---

### Phase 4: Final Verification & Hardening

**PR:** #5
**Deliverables:**
- Full regression suite (backend + frontend)
- Performance benchmark: <2x Phase 0 baseline
- Security review: `pnpm audit` + manual auth flow review
- Dead code removal: unused packages, orphan files
- Repository hygiene pass

**Acceptance:**
```bash
pnpm lint && pnpm typecheck && pnpm test && pnpm build
pnpm --filter @buildingai/api test:e2e
pnpm --filter buildingai-client exec playwright test
pnpm audit
# Performance
autocannon -c 100 -d 10 http://localhost:4090/consoleapi/health
```

**Final tag:** `git tag -a unome-v1.0.0 -m "UnoMe v1.0.0 release"`

---

## 7. Context Bloat Guardrails

| Metric | Limit | Enforcement |
|--------|-------|-------------|
| Adapter module files | < 20 | CI check |
| New Vue components | < 15 total | CI check |
| CLAUDE.md line count | < 200 lines | CI check |
| Package source files | < 50 per package | CI check |
| Duplicate abstractions | 0 | Code review |
| Redundant markdown files | 0 | Code review |

---

## 8. Development Workflow

### DevContainer Setup

Configured at `.devcontainer/devcontainer.json`:
- Base image: `mcr.microsoft.com/devcontainers/typescript-node:22`
- Rust toolchain (for Tauri desktop builds)
- Forwarded ports: 4090 (API), 4091 (OpenWork), 5432 (PostgreSQL), 6379 (Redis)
- VS Code extensions: Volar, ESLint, Prettier, Tailwind CSS, Playwright
- Post-create: installs pnpm and bun, runs `pnpm install`

### Environment

| Tool | Version |
|------|---------|
| Node.js | 24.14.1 |
| pnpm | 10.20.0 |
| bun | 1.3.12 |
| Docker | 29.4.0 |
| Docker Compose | 5.1.3 (standalone: `docker-compose`) |
| GitHub CLI | 2.89.0 |

**UnoMe repo**: https://github.com/CodeWithJames-AI/UnoMe

### z.ai MCP Usage

Use z.ai MCP server to query OpenWork's codebase surgically during development. This keeps the agent context window focused on BuildingAI changes only — no need to load OpenWork code into the workspace.

### Git Practices

- Conventional commits: `feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `test:`
- Issue tickets for every phase
- PR for every phase with template
- Annotated tags for every checkpoint

---

## 9. Warnings (from Requirements Audit)

| Check | Status | Detail |
|-------|--------|--------|
| CI/CD Pipeline | **NOT FOUND** | BuildingAI has no GitHub Actions. Must create in Phase 0. |
| Lint Support | **CONFIRMED** | ESLint + Prettier configured via Turbo (`pnpm lint`, `pnpm lint:fix`) |
| Syntax Check | **CONFIRMED** | TypeScript 5.x strict mode (`pnpm typecheck`) |
| LSP Support | **CONFIRMED** | TypeScript 5.x provides full LSP |
| Unit/Integration Tests | **PARTIAL** | Jest configured in API package. No CI to run them. |

---

## 10. Rollback Strategy

Every phase produces a verified checkpoint that includes:

- Annotated git tag
- Build hash
- Test results
- Dependency lockfile (`pnpm-lock.yaml`)
- Environment template (`.env.template`)
- Docker Compose configuration

**Rollback procedure (per phase):**
```bash
# 1. Stop all services
docker-compose down

# 2. Revert to previous checkpoint
git checkout unome/phase<N-1>

# 3. Restart services
docker-compose up -d

# 4. Wait for startup
sleep 30

# 5. Verify health
curl -f http://localhost:4090/consoleapi/health

# 6. Monitor logs
docker compose logs -f --tail=50  # monitor 5 minutes
```

---

*This blueprint was produced by the Ralplan consensus workflow: Planner (initial plan) → Architect (structural review, ITERATE) → Critic (REJECT v2, REJECT v3, APPROVE v4).*
