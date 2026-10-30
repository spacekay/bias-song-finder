[🇰🇷 한국어](ko/architecture.md)

# Architecture

## System Overview

Bias Song Finder is a full-stack web application with a separately deployed backend and frontend.

```
User Browser
    │
    ▼
Cloudflare Workers (Next.js)
    │  API proxy
    ▼
fly.io (Spring Boot)
    ├── PostgreSQL (schema-per-tenant)
    └── Redis (session + rate limiting)
```

---

## Backend: Gradle Multi-Module

The backend is split into three Gradle modules, each with a distinct responsibility:

| Module | Role |
|--------|------|
| `:core` | Shared domain logic, entities, services, schema migrations, multi-tenancy infrastructure |
| `:api` | REST API server (deployed) — authentication, rate limiting, API documentation |
| `:admin` | Admin UI server (local only) — data management interface |

Both `:api` and `:admin` depend on `:core`.

---

## Application Boundaries

HTTP adapters delegate to domain services, with shared persistence and infrastructure in `:core`. Within the quiz engine, question selection policies, option builders, answer processing, state transitions, and result assembly have separate responsibilities. This makes individual rules easier to test without putting the entire flow in one service.

---

## Multi-Tenancy: Schema-per-Tenant

Each artist group (e.g., SHINee, f(x)) gets its own isolated PostgreSQL schema containing all domain tables, while shared metadata lives in a common schema.

Key aspects:
- **Multi-layer tenant resolution** — resolves tenant context for active sessions and persisted shared links after session expiry
- **Validated schema routing** — tenant identifiers are validated before SQL construction; tenant connections do not fall through to shared domain tables
- **Connection cleanup** — the schema is reset before connection reuse; connections that fail setup or cleanup are discarded

---

## Session-Based Game Flow

No user login required. A session key acts as the user identifier.

The flow follows a simple cycle:
1. **Session creation** — initializes candidate pool from the tenant's song catalog
2. **Question loop** — backend selects the next question, user answers, candidates are filtered
3. **Result** — when the candidate pool is narrowed to a single song, the quiz ends

All session state is stored externally (not in-memory), making the backend horizontally scalable.

### Graceful Fallback

Answer submission returns explicit outcomes for a match, no match, or exhausted questions, with similarity recommendations where applicable. When questions run out, users can reopen the last question or continue to a no-match result. Next-question retrieval retains its error responses for empty or exhausted flows.

### State Consistency

Versioned Redis updates compare the expected session version atomically while preserving its remaining TTL. Stale updates and repeated answers return a conflict instead of overwriting newer progress. Result creation uses a database transaction lock and a uniqueness constraint so concurrent or repeated requests retain one active result per session.

See [Engineering Decisions](engineering-decisions.md) for the problems and tradeoffs behind these boundaries.

---

## Deployment

| Environment | Backend | Frontend |
|-------------|---------|----------|
| Development | fly.io | Cloudflare Workers |
| Production | fly.io | Cloudflare Workers |

### Observability (Local)
- **Loki + Promtail + Grafana** — structured session log ingestion and dashboard
- **Prometheus + Spring Boot Actuator** — application metrics scraped into Grafana
- Grafana dashboards track: session funnels, answer distributions, result satisfaction, per-session journey
