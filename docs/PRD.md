# UnoMe — Product Requirements Document

**Version:** 1.0.0
**Date:** 2026-04-16
**Status:** Approved (Ralplan Consensus v4)

---

## 1. Overview

UnoMe is an enterprise-grade AI platform that unifies the capabilities of two open-source projects:

- **BuildingAI** — An enterprise AI agent platform with visual configuration, RAG knowledge bases, MCP integration, model management, billing, and user management.
- **OpenWork** — An agentic workflow platform with skill management, session orchestration, permissions, messaging bridges (Slack/Telegram), and OpenCode-powered execution.

The final product, **UnoMe**, presents a single unified application to end users. No one should be able to tell it originated from two separate platforms.

---

## 2. Source Repositories

| Component | Repository | License | Version |
|-----------|-----------|---------|---------|
| BuildingAI (host platform) | https://github.com/BidingCC/BuildingAI | Apache 2.0 | 26.0.1 |
| OpenWork (workflow engine) | https://github.com/different-ai/openwork | MIT | 0.11.207 |

---

## 3. Target Users

- **AI developers** building enterprise AI applications
- **AI entrepreneurs** deploying AI-powered products
- **Organizations** seeking a unified AI workflow and agent platform
- **IT administrators** (Bob) who configure workflows for their team
- **End users** (Susan) who consume AI-powered automations

---

## 4. Core Features

### 4.1 From BuildingAI (Preserved)

- **AI Conversations** — LLM-powered chat with multimodal model support
- **AI Agents** — Autonomous agents with memory, goals, and tool usage
- **Knowledge Base (RAG)** — Document ingestion, vector search, retrieval-augmented generation
- **MCP Integration** — Call MCP tools via SSE and Streamable HTTP protocols
- **Model Management** — Unified API for multiple LLM providers
- **Extension Mechanism** — Installable extensions for expanding capabilities
- **Billing & Payments** — Membership management, compute billing, payment processing
- **Desktop App** — Tauri-based desktop application

### 4.2 From OpenWork (Integrated)

- **Agentic Workflow Engine** — Session lifecycle, prompt management, event streaming
- **Skill Manager** — Install, import, list, and execute workflow skills
- **Permission System** — Allow-once, allow-session, deny with audit trail
- **Messaging Bridges** — Slack and Telegram integration via OpenWork Router
- **Session Templates** — Save and re-run common workflows
- **Workflow Timeline** — Visual execution plan with step-level progress

### 4.3 Unified Experience

- **Single UI** — All features accessible through one Vue 3 web application
- **Vertical Navigation** — Workflow, skills, messaging integrated into BuildingAI's nav bar
- **Seamless Auth** — Single sign-on across all features
- **Unified Settings** — One settings panel for all platform configuration

---

## 5. Technical Architecture

### 5.1 Stack

| Layer | Technology |
|-------|-----------|
| Backend (BuildingAI) | NestJS 11, TypeORM, PostgreSQL 17, Redis, BullMQ |
| Backend (OpenWork) | OpenWork Orchestrator, OpenCode SDK, Express |
| Frontend | Vue 3, NuxtUI 3, React (existing admin), Vite 7, Tailwind CSS |
| Desktop | Tauri |
| Monorepo | pnpm workspaces, Turborepo |
| Language | TypeScript 5.x |
| Containers | Docker, Docker Compose |

### 5.2 Architecture Pattern

Two self-contained services connected via a thin adapter layer:

```
┌─────────────────────────────────────────────────────┐
│                   UnoMe UI (Vue 3)                   │
│  ┌──────┬──────┬──────────┬──────────┬────────────┐  │
│  │ Chat │Agents│Knowledge │ Workflow │ Messaging   │  │
│  │      │      │   Base   │ Skills   │ Slack/TG   │  │
│  └──┬───┴──┬───┴────┬─────┴────┬─────┴─────┬──────┘  │
│     │      │        │          │           │         │
│  ───┼──────┼────────┼──────────┼───────────┼─────── │ ← thin adapter
│     │      │        │          │           │         │
│  ┌──▼──────▼────────▼──┐  ┌───▼───────────▼──────┐  │
│  │  BuildingAI API     │  │ OpenWork Orchestrator │  │
│  │  NestJS Backend     │  │ (OpenCode + Sessions  │  │
│  │  AI SDK, RAG,       │  │  Skills, Permissions) │  │
│  │  Billing, Agents    │  │                       │  │
│  └─────────┬───────────┘  └───────────┬───────────┘  │
│            │                          │              │
│  ┌─────────▼──────────┐   ┌──────────▼───────────┐  │
│  │  PostgreSQL 17     │   │  OpenWork State       │  │
│  │  (BuildingAI)      │   │  (own storage)        │  │
│  └────────────────────┘   └──────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

- BuildingAI and OpenWork remain self-contained with their own state
- Integration via thin adapter: one NestJS proxy module + Vue components
- Both services cherry-pick updates from upstream independently

---

## 6. Quality Requirements

### 6.1 CI/CD

- **CRITICAL**: BuildingAI has no existing CI/CD pipeline. Must be created.
- GitHub Actions workflow: lint, typecheck, test, build
- Run on every PR and push to main

### 6.2 Lint, Syntax, and LSP

- BuildingAI has ESLint, TypeScript strict mode, and Prettier configured
- TypeScript 5.x provides full LSP support
- CI must enforce: `pnpm lint && pnpm typecheck`

### 6.3 Regression Prevention

- **Backend**: Scripted integration tests (Jest + supertest)
- **Frontend**: Playwright + Chrome DevTools MCP for interactive testing
- Regression tests run before every merge
- Existing features must never break when adding new ones

### 6.4 Checkpoint and Rollback

- Every phase produces an annotated git tag with: build hash, test results, lockfile, .env template
- Checkpoints must include all runtime state for full recreation
- Rollback procedure: `docker compose down, git checkout <tag>, docker compose up -d, smoke test`

### 6.5 Context Bloat Prevention

- Adapter module: < 20 files
- New Vue components: < 15 files total
- CLAUDE.md: < 200 lines
- No package > 50 source files
- No duplicate abstractions or redundant markdown files

### 6.6 Repository Hygiene

- Strict issue tickets and PR practices
- Best git commit conventions (conventional commits)
- No orphan files, no dead code, no unused dependencies
- Clean AGENTS.md with minimal instructions

---

## 7. Development Environment

- **DevContainer** with Node 22, pnpm, bun, Rust toolchain, PostgreSQL, Redis
- **z.ai MCP** for surgical OpenWork codebase queries (no context loading)
- **Docker Compose** for local deployment and testing

---

## 8. Rebranding

All user-facing artifacts must be rebranded from BuildingAI/OpenWork to UnoMe:

- Application name and logo
- Package namespaces (`@buildingai/*` remain internal; external-facing is `@unome/*`)
- Docker image names, environment variable prefixes
- Documentation and README
- Favicons, banners, screenshots

---

## 9. Non-Goals

- Replacing OpenCode's CLI/TUI
- Merging the two codebases into one monolith
- Extracting OpenWork features without the orchestrator
- Supporting Windows desktop initially (macOS/Linux first)
- Creating bespoke capabilities that don't map to existing platform APIs

---

## 10. Success Criteria

1. UnoMe runs as a single unified application
2. All BuildingAI features (chat, agents, knowledge base, MCP, billing) work unchanged
3. OpenWork features (workflows, skills, messaging, permissions) are accessible from the nav bar
4. Users cannot tell the app originated from two separate platforms
5. CI/CD pipeline enforces quality gates on every PR
6. Full regression test suite passes
7. Performance within 2x of BuildingAI baseline
8. Clean rollback to any phase checkpoint
