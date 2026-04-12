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

## Hexagonal Architecture

Each module follows hexagonal (ports and adapters) architecture, cleanly separating the HTTP layer from domain logic and cross-cutting infrastructure concerns.

---

## Multi-Tenancy: Schema-per-Tenant

Each artist group (e.g., SHINee, f(x)) gets its own isolated PostgreSQL schema containing all domain tables, while shared metadata lives in a common schema.

Key aspects:
- **Multi-layer tenant resolution** — multiple fallback strategies ensure tenant context is always available, including for shared links after session expiry
- **Connection-level schema routing** — tenant context is applied at the database connection level with proper cleanup to prevent cross-tenant data leakage

---

## Session-Based Game Flow

No user login required. A session key acts as the user identifier.

The flow follows a simple cycle:
1. **Session creation** — initializes candidate pool from the tenant's song catalog
2. **Question loop** — backend selects the next question, user answers, candidates are filtered
3. **Result** — when the candidate pool is narrowed to a single song, the quiz ends

All session state is stored externally (not in-memory), making the backend horizontally scalable.

---

## Deployment

| Environment | Backend | Frontend |
|-------------|---------|----------|
| Development | fly.io | Cloudflare Workers |
| Production | fly.io | Cloudflare Workers |

### Observability (Local)
- **Loki + Promtail + Grafana** — structured session log ingestion and dashboard
- Grafana dashboards track: session funnels, answer distributions, result satisfaction, per-session journey
